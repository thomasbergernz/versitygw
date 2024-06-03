Do you need a simple, reliable, performant, easy to deploy S3 server for your applications or test infrastructure? VersityGW combined with ZFS offers an easy to manage S3 service on top of a robust filesystem with built in software RAID protection. ZFS is a powerful file system renowned for its advanced capabilities in storage management. Pairing ZFS with the deployment of VersityGW offers a simple S3 server solution that maximizes single server performance and resilience. Follow along for a setup example.

See [MultiPart Optimizations](./MultiPart-Optimizations) for consideration of other filesystems that might offer a more optimized case for multipart uploads.

The test system is a virtual machine running:
```
AlmaLinux release 9.3 (Shamrock Pampas Cat)
kernel version 5.14.0-362.13.1.el9_3.x86_64
```
These steps or similar are likely to work on any modern linux distro.

### Install ZFS
```
dnf install https://zfsonlinux.org/epel/zfs-release-2-2$(rpm --eval "%{dist}").noarch.rpm
dnf install zfs
```

### Configure ZFS
On this system, we have four block devices (/dev/vdb /dev/vdc /dev/vdd /dev/vde) that will be used to setup ZFS with RAIDZ.
```
modprobe zfs
zpool create mypool raidz /dev/vdb /dev/vdc /dev/vdd /dev/vde
zpool status
mkdir /mnt/zfs
zfs create mypool/myfilesystem 
zfs set mountpoint=/mnt/zfs mypool/myfilesystem
```

### Install VersityGW
For RedHat and Debian based distributions, there are pre-build distro packages available in the latest release assets. At the time of this writing, v0.21 is the latest release. Replace the following URL with URL to latest version.
```
dnf install https://github.com/versity/versitygw/releases/download/v0.21/versitygw_0.21_linux_amd64.rpm
```

### Configure and Run VersityGW
```
mkdir /mnt/zfs/vgw
mkdir /mnt/zfs/accounts
```
Add the following to `/etc/versitygw.d/zfs.conf`, changing options as needed. We will be hosting the directory /mnt/zfs/vgw via S3, and using /mnt/zfs/accounts to store IAM account info.
```
VGW_BACKEND=posix
VGW_BACKEND_ARG=/mnt/zfs/vgw
ROOT_ACCESS_KEY_ID=admin
ROOT_SECRET_ACCESS_KEY=password
VGW_IAM_DIR=/mnt/zfs/accounts
```
Enable the service and have it start immediately.
```
systemctl enable --now versitygw@zfs.service
```

### Test Gateway
We now have a running S3 service on top of the ZFS filesystem. We can install a test client utility to test.
```
dnf install s3cmd
```
Setup the following config in `./s3cfg.local`, modifying the account info as needed.
```
# Setup endpoint
host_base = 127.0.0.1:7070
host_bucket = 127.0.0.1:7070
bucket_location = us-east-1
use_https = False

# Setup access keys
access_key =  admin
secret_key = password

# Enable S3 v4 signature APIs
signature_v2 = False
```

Run example client commands.
```
# s3cmd -c ./s3cfg.local mb s3://testbucket
# echo "testing" >testfile
# s3cmd -c ./s3cfg.local put testfile s3://testbucket/dir/testfile

# s3cmd -c ./s3cfg.local ls s3://testbucket
                          DIR  s3://testbucket/dir/
# s3cmd -c ./s3cfg.local ls s3://testbucket/dir/
2024-04-26 17:55            8  s3://testbucket/dir/testfile
```

This is what the above command will create within the filesystem:
```
# find /mnt/zfs/
/mnt/zfs/
/mnt/zfs/accounts
/mnt/zfs/accounts/users.json
/mnt/zfs/vgw
/mnt/zfs/vgw/testbucket
/mnt/zfs/vgw/testbucket/.sgwtmp
/mnt/zfs/vgw/testbucket/dir
/mnt/zfs/vgw/testbucket/dir/testfile
```
The `/mnt/zfs/accounts/users.json` file will keep track of any IAM accounts created through the gateway admin interfaces. All buckets are directories at the top level `/mnt/zfs/vgw`. The object names are split on "/" and put into the corresponding directory hierarchy. The file `/mnt/zfs/vgw/testbucket/dir/testfile` is the same as what we uploaded in the test:
```
# cat /mnt/zfs/vgw/testbucket/dir/testfile
testing
```

### Conclusion
This example demonstrates the power of a simple to deploy S3 gateway on top of a robust storage system such as ZFS. Storage admins keep the familiarity of the underlying storage system tools and capabilities while adding compatibility to the growing number of S3 client applications.