The ScoutFS backend stores S3 objects within a scoutfs filesystem.

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
- [x] Restore Object
- [x] Set Object Tags
- [x] Get Object Tags
- [x] Copy Object
- [ ] Set Object ACL
- [ ] Get Object ACL
### Multipart Uploads:
- [x] Create Multipart Upload
- [x] Complete Multipart Upload
- [x] Abort Multipart Upload
- [x] List Multipart Uploads
- [x] List Multipart Upload Parts
- [x] Put Object Part
- [x] Upload Part Copy

# Args
The gateway root directory must be specified with the scoutfs subcommand. The gateway root tells the gateway what directory to host as the S3 service.  This can be a filesystem mount point directory, but more commonly would be a sub-directory within the filesystem to store data associated with the S3 service.  For example, if there is a filesystem mounted at `/mnt/datastore`, the gateway could host the entire filesystem by specifying `/mnt/datastore` or a subdirectory such as `/mnt/datastore/gateway1`. Note that the directory must exist before starting the gateway. The specified directory arg is now considered the "gateway root".

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

# Glacier mode
ScoutFS supports an offline file mode where the metadata is resident in the filesystem, but the data is not on local filesystem block devices. To retrieve the data, a stage request is issued to ScoutAM that will copy the data from the archive system back into the local filesystem.

When Glacier mode is enabled in the gateway, the following behavior is enabled:
* The object status is reflected in the storage class. The storage class is set to STANDARD for online objects, and GLACIER for offline objects. The storage class can be queried with the HeadObject API call.
* A GET for an offline object will return InvalidObjectState error that is compatible with Glacier API.
* An offline object can be transitioned from GLACIER to STANDARD with the RestoreObject API.
* Once a restore object is requested, the x-amz-restore header in HeadObject response will reflect ongoing-request=“true” until the recall is completed.
* The object can be transitioned to offline state by setting storage class to GLACIER. 

# Limitations
The object listings will not traverse symlinks. An object GET on a symlink will get the file the symlink references, and an object DELETE will delete the symlink.

If the gateway is run as root, the only access permissions enforced are from the bucket and object ACLs.  The gateway user accounts are **not** mapped to local unix accounts.

If the gateway is run as a user, then the gateway will only be able to access/write files based on that user's permissions.

# Advanced details
## Temporary files
There are two cases where the gateway needs temporary storage. The first is for inflight object uploads (see Atomic actions below), and the second is for in progress multipart uploads.

While PUT and PUT part operation are in progress, the gateway will make use of temporary unnamed files with O_TMPFILE. The advantage of O_TMPFILE is that these are automatically removed by the filesystem when the file handle is closed if the gateway does not complete the upload.

The create multipart upload API will create the following directory:
```
<bucket>/.sgwtmp/multipart/<obj-hash>/<uploadid>
```
The `<obj-hash>` is a sha256 hash of the object name, and the `<uploadid>` is a uuid returned from the create multipart request. The parts are uploaded as files within this directory named by part id.

This allows multiple uploads in progress for the same object name (as supported by AWS S3), listing of all in progress multipart uploads, and listing of multipart upload parts.

The multipart uploads can be removed by the s3 client with the abort-multipart-upload API.

## Atomic actions
Since creating and writing to a file is not atomic, the posix backend uses temporary files to uphold the atomic semantics of an object PUT. The expected behavior of a client uploading two objects with the same name simultaneously is that the last object to complete uploading will be the resulting object in entirety.  Meaning that the object will not be some combination of writes from one and the other with the file. The file will be only the complete object uploaded by one of these sessions, whichever happened to be the last one to complete uploading. This is true for both object PUTs and multipart part PUTs. Since parts can be overwritten in the same way that objects can.

This is achieved using O_TMPFILE unnamed file handles. The object or part is written to the open file handle while the upload is in progress.  This allows simultaneous uploads of the same files because the gateway is writing to different file handles. Once the request is completely uploaded and successful so far, the open file handle is linked into the filesystem namespace as the appropriate name.  This linking is atomic and upholds the atomic upload semantics. If the request is not successful or if the gateway crashes, the filesystem automatically reclaims the storage space when the file handle is closed.
