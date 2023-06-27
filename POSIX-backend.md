The POSIX backend stores S3 objects within a filesystem.

# Functional Operations
### Bucket:
- [x] Create Bucket
- [x] Delete Bucket
- [x] Get Bucket Headers
- [x] List Buckets
- [x] Set Bucket ACL
- [x] Get Bucket ACL
- [ ] Set Bucket Tags
- [ ] Get Bucket Tags
### Object:
- [x] Put Object
- [x] Delete Object
- [x] Delete Objects
- [x] Get Object
- [x] Get Object Headers
- [x] List Objects
- [x] List Objects (v2)
- [x] Set Object Tags
- [x] Get Object Tags
- [ ] Copy Object
- [ ] Set Object ACL
- [ ] Get Object ACL
### Multipart Uploads:
- [x] Create Multipart Upload
- [x] Complete Multipart Upload
- [x] Abort Multipart Upload
- [x] List Multipart Uploads
- [x] List Multipart Upload Parts
- [x] Put Object Part
- [ ] Copy Object Part
- [ ] Upload Part Copy

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
The object listings will not traverse symlinks. An object GET on a symlink will get the file the symlink references, and an object DELETE will delete the symlink.

If the gateway is run as root, the only access permissions enforced are from the bucket and object ACLs.  The gateway user accounts are **not** mapped to local unix accounts.

If the gateway is run as a user, then the gateway will only be able to access/write files based on that user's permissions.

Multi-part upload parts are written as individual files within the filesystem temporary directory (see advanced topic) for upload-part API, and then copied to the final object file for the complete-multipart-upload API. The copy will try to use optimized filesystem clone extents calls when possible, but this means that multipart upload objects might be effectively written twice to the filesystem with fallback behavior.

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