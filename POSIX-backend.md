The POSIX backend stores S3 objects within a filesystem.

# Functional Operations
✅ - working<br>
❌ - not supported

<table>
<tr>
<td> Bucket </td> <td> Object </td> <td> Multipart Uploads </td>
</tr>
<tr>
<td valign="top">

✅ Create Bucket<br>
✅ Delete Bucket<br>
✅ Get Bucket Headers<br>
✅ List Buckets<br>
✅ Set Bucket ACL<br>
✅ Get Bucket ACL<br>
✅ Set Bucket Tags<br>
✅ Get Bucket Tags<br>
✅ Delete Bucket Tags<br>
✅ Get Bucket Policy<br>
✅ Put Bucket Policy<br>
❌ Put Bucket Versioning<br>
❌ Get Bucket Versioning

</td>
<td valign="top">

✅ Put Object<br>
✅ Delete Object<br>
✅ Delete Objects<br>
✅ Get Object<br>
✅ Get Object Headers<br>
✅ Get Object Attributes<br>
✅ List Objects<br>
✅ List Objects (v2)<br>
✅ Set Object Tags<br>
✅ Get Object Tags<br>
✅ Delete Object Tags<br>
✅ Copy Object<br>
✅ Put Object Lock Configuration<br>
✅ Get Object Lock Configuration<br>
✅ Put Object Legal Hold<br>
✅ Get Object Legal Hold<br>
✅ Put Object Retention<br>
✅ Get Object Retention<br>
❌ Restore Object<br>
❌ Set Object ACL<br>
❌ Get Object ACL<br>
❌ List Object Versions<br>
❌ Select Object Content

</td>
<td valign="top">

✅ Create Multipart Upload<br>
✅ Complete Multipart Upload<br>
✅ Abort Multipart Upload<br>
✅ List Multipart Uploads<br>
✅ List Multipart Upload Parts<br>
✅ Put Object Part<br>
✅ Upload Part Copy

</td>
</tr>
</table>

# Filesystem compatibility
The filesystem must have the ability to store extended attributes.

# Args
The gateway root directory must be specified with the posix subcommand. The gateway root tells the gateway what directory to host as the S3 service.  This can be a filesystem mount point directory, but more commonly would be a sub-directory within the filesystem to store data associated with the S3 service.  For example, if there is an xfs filesystem mounted at `/mnt/datastore`, the gateway could host the entire filesystem by specifying `/mnt/datastore` or a subdirectory such as `/mnt/datastore/gateway1`. Note that the directory must exist before starting the gateway. The specified directory arg is now considered the "gateway root".

# Object name mapping
The buckets are mapped to top level directories under the gateway root directory. These directories do not have to be created by the gateway. Existing directories within the gateway root will be seen as buckets in the list-buckets API. Any files in the gateway root are ignored, since all objects must be within a bucket in S3.

The object names are mapped to directories/files within the filesystem by splitting the object names on the `/` separator. For example, the following object:
```
gateway root: /mnt/fs/gateway
bucket: mybucket
object: 2023/Jan/myobject
```
Would translate to the following file within the filesystem:
```
/mnt/fs/gateway/mybucket/2023/Jan/myobject
```

Object names ending in `/` will create a directory within the filesystem instead of a file.

# Limitations
FS extended attribute support: The gateway uses file/directory xattrs to store metadata. The gateway will attempt to run a validation check on startup for xattr support.

The object listings will not traverse symlinks. An object GET on a symlink will get the file the symlink references, and an object DELETE will delete the symlink.

If the gateway is run as root, the only access permissions enforced are from the bucket and object ACLs.  The gateway user accounts are **not** mapped to local unix accounts.

If the gateway is run as a user, then the gateway will only be able to access/write files based on that user's permissions.

FS/Kernel copy_file_range() support: Multi-part upload parts are written as individual files within the filesystem temporary directory  for upload-part API (see advanced topic), and then copied to the final object file for the complete-multipart-upload API. The copy will try to use the optimized filesystem copy_file_range() calls when possible, but will fallback to standard data copy when not available. The optimizations are filesystem dependent, and ideally would just be a metadata update utilizing internal copy on write behavior. Some filesystems may optimize this with still copying the data, but with a single system call. The fallback behavior is just a standard userspace data copy, meaning that multipart uploads will effectively be written twice to storage. Once with the original put part, and a second time with the complete upload copy into place. Choosing an optimized filesystem filesystem can possibly double performance for these cases.

Object PUTs will overwrite files but not directories. If a directory already exists with the same name as the object, the PUT will fail with ExistingObjectIsDirectory.

Object names are translated into directory components by splitting on "/" separator. If a directory within the object name already exists as a file instead of a directory, then the PUT will fail with ObjectParentIsFile.

An object with a trailing "/" is interpreted as a directory.  So put object with trailing "/" will create a directory in the filesystem.  This means that the content length must be 0 for all directory type objects since there is no data payload for directories in the filesystem. It is also and error to create a multipart upload with a trailing "/" in the key. The create multipart upload with trailing "/" or put object with a trailing "/" and non-0 content length the error DirectoryObjectContainsData will be returned.

# Advanced details
## Temporary files
There are two cases where the gateway needs temporary storage. The first is for inflight object uploads (see Atomic actions below), and the second is for in progress multipart uploads.

While PUT and PUT part operation are in progress, the gateway will make use of temporary files. If the filesystem supports O_TMPFILE, this is used when possible to create temporary files that are inflight. The advantage of O_TMPFILE is that these are automatically removed by the filesystem when the file handle is closed if the gateway does not complete the upload. When not available the system will create a unique temp file name within the .sgwtmp directory under the bucket directory, and do a best effort cleanup on failures.

The create multipart upload API will create the following directory:
```
<bucket>/.sgwtmp/multipart/<obj-hash>/<uploadid>
```
The `<obj-hash>` is a sha256 hash of the object name, and the `<uploadid>` is a uuid returned from the create multipart request. The parts are uploaded as files within this directory named by part id.

This allows multiple uploads in progress for the same object name (as supported by AWS S3), listing of all in progress multipart uploads, and listing of multipart upload parts.

The multipart uploads can be removed by the s3 client with the abort-multipart-upload API.

## Atomic actions
Since creating and writing to a file is not atomic, the posix backend uses temporary files to uphold the atomic semantics of an object PUT. The expected behavior of a client uploading two objects with the same name simultaneously is that the last object to complete uploading will be the resulting object in entirety.  Meaning that the object will not be some combination of writes from one and the other with the file. The file will be only the complete object uploaded by one of these sessions, whichever happened to be the last one to complete uploading. This is true for both object PUTs and multipart part PUTs. Since parts can be overwritten in the same way that objects can.

This is achieved in one of two ways. If the filesystem supports O_TMPFILE, then the object or part is written to the open file handle while the upload is in progress.  This allows simultaneous uploads of the same files because the gateway is writing to different file handles. Once the request is completely uploaded and successful so far, the open file handle is linked into the filesystem namespace as the appropriate name.  This linking is atomic and upholds the atomic upload semantics. If the request is not successful or if the gateway crashes, the filesystem automatically reclaims the storage space when the file handle is closed.

If O_TMPFILE is not available within the filesystem, then the gateway will create a temporary unique filename within the temp directory.  For example with object PUT:
```
<bucket>/.sgwtmp/<obj-hash>.<random>
```
The `<obj-hash>` is a sha256 hash of the object name, and `<random>` is a random string. For example mutlipart part PUT:
```
<bucket>/.sgwtmp/multipart/<obj-hash>/<uploadid>/<part>.<random>
```
The `<obj-hash>` is a sha256 hash of the object name, the `<uploadid>` is a uuid returned from the create multipart request, the `<part>` is the integer part number, and `<random>` is a random string.

The gateway will attempt to cleanup these files for unsuccessful upload, but may leave files behind if it crashes.