## Speeding up multipart uploads with copy_file_range() and move_blocks()

In VersityGW with `posix` backend, multipart uploads are initiated with a create-multipart-upload API call which returns a unique upload ID. On the backend, the directory `/<gateway root>/<bucket>/.sgwtmp/multipart/<objectname hash>/<upload id>` is created that will contain all the parts as they are uploaded.  Each upload-part API call creates a new file within the upload ID directory with the data for that part. Once all parts are uploaded, the complete-multipart-upload API call is used to create the final object from the part data. The gateway copies the data from the parts into the correct offset for the file object file. This allows a consistent posix view of the completed object file. However, if the filesystem is not optimized for copying data between parts and the final object file, this can result in writing the entire object twice. Once for the initial part uploads, and a second time when copying the data from the parts to the final object file. This would give the impression of half of the storage system capable performance to the client. So is there a way to do multipart uploads without having to write the data to the filesystem twice?

Some filesystems have an optimization for copying data between files within the filesystem. This article will dive into the details of whats needed to take advantage of this optimization when running VersityGW.

Summary of optimizations:
| Filesystem | multipart-upload optimized | Notes |
| :----------: | :--------------------------: | ----- |
| XFS | ✅ | |
| Btrfs | ✅ | |
| ZFS | | |
| ext4 | | |
| NFS 4.2 | ✅ | only if exported filesystem is optimized |
| ScoutFS | ✅ | must use `scoutfs` backend |

### Some background:
The copy_file_range() Linux system call was introduced in 2015 based on work that Versity’s own Principle Filesystem Engineer, Zach Brown, started while working on Btrfs. See https://lwn.net/Articles/659523/. It has been significantly enhanced and aviable in mainline kernel in its current capability since Linux 5.19. It has also been backported to most major enterprise kernel distros as well.

The Go standard library started calling copy_file_range() by default under the hood of io.Copy() in go 1.15: https://github.com/golang/go/issues/36817. This means that developers can make use of copy_file_range() with very standard Go implementations of copying data between files.

### The copy_file_range() details:
First let’s look at the documentation for copy_file_range():
```
The copy_file_range() system call performs an in-kernel copy
       between two file descriptors without the additional cost of
       transferring data from the kernel to user space and then back
       into the kernel.
```

This is great to allow the copy to happen completely within the kernel, but there is more. This basically informs the filesystem of our intent to copy data between files, and opens up the opportunity for the filesystem to optimize this action even further. Some filesystems have a capability to reference the same data from multiple inodes. This is generally done by creating a shared reference to the data, and then copying modifications to a new location so that these modifications are only visible on the file being modified. The two main Linux filesystems that support this are Btrfs and XFS. Unfortunately Btrfs is no longer supported by RedHat, so this article will compare XFS and ext4 performance.

The maintainer of XFS, Darrick Wong, wrote a great blog post describing how XFS supports the shared data blocks that copy_file_range() makes use of: https://blogs.oracle.com/linux/post/xfs-data-block-sharing-reflink.

### Test setup:
AlmaLinux release 9.3 (Shamrock Pampas Cat)
kernel version 5.14.0-362.13.1.el9_3.x86_64

A [previous post](./ZFS-Use-Case) examined the ZFS use case with raidz for reliability. For a comparable test with the other filesystems, a software raid5 device is configured using mdraid:
mdadm --create /dev/md0 --level=5 --raid-devices=4 /dev/vdb /dev/vdc /dev/vdd /dev/vde

The /dev/md0 raid5 device will be used for the block device for other filesystems while testing. This was not strictly necessary, but will hopefully give readers an idea of what is possible if looking for ZFS alternatives.

### Verifying io.Copy():
The first test will use an xfs filesystem:
```
mkfs.xfs -f /dev/md0
mkdir /mnt/xfs
mount /dev/md0 /mnt/xfs
```

To verify that the Go standard library was working as advertised, a test program was used to copy data between files using io.Copy():

```go
package main

import (
	"io"
	"os"
)

func main() {
	if len(os.Args) < 3 {
		println("Usage: cptest <source> <destination>")
		return
	}

	source := os.Args[1]
	destination := os.Args[2]

	sourceFile, err := os.Open(source)
	if err != nil {
		println("Error opening source file:", err)
		return
	}
	defer sourceFile.Close()

	destinationFile, err := os.Create(destination)
	if err != nil {
		println("Error creating destination file:", err)
		return
	}
	defer destinationFile.Close()

	_, err = io.Copy(destinationFile, sourceFile)
	if err != nil {
		println("Error copying file:", err)
		return
	}
}
```

This code opens an existing file, creates the destination file, and then copies the data between the two. The strace command can be used to see the system calls being invoked.

First create a file with some data:
```
# dd if=/dev/zero of=data bs=1024k count=10240
10240+0 records in
10240+0 records out
10737418240 bytes (11 GB, 10 GiB) copied, 18.009 s, 596 MB/s
```

Then run the cptest under strace to observe the copy_file_range() calls:
```
# time strace -T -e trace=copy_file_range -e signal='!SIGURG' /root/cptest ./data ./test
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004459>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004309>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004316>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004321>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004331>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004406>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004313>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004326>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004290>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <0.004348>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 0 <0.000008>
+++ exited with 0 +++

real	0m0.058s
user	0m0.001s
sys	0m0.011s
```

This test not only confirms that the Go code is invoking the copy_file_range() system call, but also demonstrates the power of this system call on an optimized filesystem such as XFS with reflinks resulting in copying a 10GiB file to a new file in 0.06 seconds. It’s able to do this because there is no new data being written to the filesystem. This is only needing to update some metadata for the block references.

Another popular Linux filesystem is ext4, but this filesystem does not offer any specific optimizations for copy_file_range() other than the VFS doing the copy completely in kernel. This is the same test as above on ext4:
```
mkfs.ext4 /dev/md0
mkdir /mnt/ext4
mount /dev/md0 /mnt/ext4
```

```
# time strace -T -e trace=copy_file_range -e signal='!SIGURG' /root/cptest ./data ./test
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <1.605877>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <2.018944>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <4.022477>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <2.907710>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <3.521956>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <3.052205>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <3.513699>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <3.362166>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <2.898603>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 1073741824 <3.698291>
copy_file_range(3, NULL, 7, NULL, 1073741824, 0) = 0 <0.000036>
+++ exited with 0 +++

real	0m31.076s
user	0m0.004s
sys	0m18.302s
```

This shows that each call to copy_file_range() is taking significantly longer than it did on XFS because ext4 is actually making a copy of the file data within each call. So the entire file to file copy that took 0.06 seconds on XFS is now taking 31 seconds on ext4.

### Verifying VersityGW:
The test program is compelling, but how can we really test the performance characteristics of the gateway on these filesystems here? The following is a script that separates out the create-multipart-upload, upload-part, and complete-multipart-upload so that we can focus on the time it takes to run just the complete-multipart-upload for comparisons.

```
#!/bin/bash

# make sure bucket exists
aws --endpoint-url http://127.0.0.1:7070 s3 mb s3://testbucket

# Create multipart upload and extract UploadId
echo "create multipart upload"
upload_response=$(aws --endpoint-url http://127.0.0.1:7070 s3api create-multipart-upload --bucket testbucket --key testkey)
upload_id=$(echo $upload_response | jq -r '.UploadId')

# Upload parts
for i in {1..5}; do
    echo "uploading part $i"
    aws --endpoint-url http://127.0.0.1:7070 s3api upload-part --bucket testbucket --key testkey --upload-id $upload_id --part-number $i --body part.data
done

# Complete multipart upload
multipart_upload='{"Parts": ['
for i in {1..5}; do
    if [ $i -eq 5 ]; then
        multipart_upload+='{"PartNumber": '$i',"ETag": "cd573cfaace07e7949bc0c46028904ff"}'
    else
        multipart_upload+='{"PartNumber": '$i',"ETag": "cd573cfaace07e7949bc0c46028904ff"},'
    fi
done
multipart_upload+=']}'

echo "complete multipart upload"
time aws --endpoint-url http://127.0.0.1:7070 s3api complete-multipart-upload --bucket testbucket --key testkey --upload-id $upload_id --multipart-upload "$multipart_upload"
```

The gateway can be run with strace like the test commands above:
```
# strace -f -T -e trace=copy_file_range -e signal='!SIGURG' versitygw --access test --secret test posix /mnt/xfs
```

The test script can be run with a few env vars set:
```
# export AWS_ACCESS_KEY_ID=test
# export AWS_SECRET_ACCESS_KEY=test
# export AWS_REGION=us-east-1
# ./upload_test.sh 
make_bucket: testbucket
create multipart upload
uploading part 1
{
    "ETag": "cd573cfaace07e7949bc0c46028904ff"
}
uploading part 2
{
    "ETag": "cd573cfaace07e7949bc0c46028904ff"
}
uploading part 3
{
    "ETag": "cd573cfaace07e7949bc0c46028904ff"
}
uploading part 4
{
    "ETag": "cd573cfaace07e7949bc0c46028904ff"
}
uploading part 5
{
    "ETag": "cd573cfaace07e7949bc0c46028904ff"
}
complete multipart upload
{
    "Bucket": "testbucket",
    "Key": "testkey",
    "ETag": "306cca04302861ed2620a328f286346f-5"
}

real	0m0.994s
user	0m0.305s
sys	0m0.066s
```

The output from the server side:
```
18:46:41 | 409 |     507.009µs | 127.0.0.1 | PUT | /testbucket | -
18:46:42 | 200 |   27.822852ms | 127.0.0.1 | POST | /testbucket/testkey | -
18:46:45 | 200 |  8.359281166s | 127.0.0.1 | PUT | /testbucket/testkey | -
18:46:56 | 200 |  7.694646413s | 127.0.0.1 | PUT | /testbucket/testkey | -
18:47:07 | 200 |  7.821324326s | 127.0.0.1 | PUT | /testbucket/testkey | -
18:47:18 | 200 |  7.492377507s | 127.0.0.1 | PUT | /testbucket/testkey | -
18:47:28 | 200 |  7.429137866s | 127.0.0.1 | PUT | /testbucket/testkey | -
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 1073741824 <0.000116>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 0 <0.000061>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 1073741824 <0.000101>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 0 <0.000049>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 1073741824 <0.000076>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 0 <0.000036>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 1073741824 <0.000060>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 0 <0.000022>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 1073741824 <0.614171>
[pid  4346] copy_file_range(10, NULL, 9, NULL, 1073741824, 0) = 0 <0.000089>
18:47:36 | 200 |  620.474804ms | 127.0.0.1 | POST | /testbucket/testkey | -
```

So we can confirm copy_file_range() is being called by the gateway, and sure enough the complete-multipart-upload only took about 1 second.

### Filesystem Capabilities:
With NFS v4.2, server side copy is enabled. This means that copy_file_range() is handled server side, and forwarded to the exported filesystem. So an XFS filesystem exported via NFS 4.2+ has similar behavior for the optimized complete-multipart-upload.

It is presumed that Btrfs would also show the optimized behavior, but it was not explicitly tested for this.

ZFS and ext4 show that copy_file_range() is taking longer due to copying the data from one file to another. So a multipart upload might take twice as long on these filesystems since the data has to be written twice (once for the upload-part, and again for the complete-multipart-upload).

ScoutFS has a similar optimization, but instead of the reflinks it has an ioctl called move_blocks() that moves extents from one file to another. Since this is moving data instead of copying references, this is incompatible with copy_file_range(). The scoutfs optimization is enabled by invoking the `scoutfs` backend instead of `posix`. For this reason, ScoutFS filesystem should always use `scoutfs` backend since the `posix` backend would fall back to the slower performance for multipart uploads.