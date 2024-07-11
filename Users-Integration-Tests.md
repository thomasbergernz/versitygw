## Test Matrix

| symbol | meaning |
| :------: | ------- |
| ✅ | tested and passing |
| ❌ | tested and failing |
| ⚪ | not tested |
| ❗ | partially tested |
| ❎ | not applicable |
| ❓ | needs to be verified |

### supported operations (NOTE:  info in process of being added, table is currently inaccurate after delete-objects)
| Role | root | admin | userplus | user | Notes |
| :---: | :------: | :-------: | :-----: | :--: | ----- |
| abort-multipart-upload | ✅ | ⚪ | ⚪ | ⚪ ||
| complete-multipart-upload | ✅ | ⚪ | ⚪ | ⚪ | No way to manually trigger multipart upload in **aws-cli s3**, **s3cmd**, **mc**, will automatically do so if file is above certain size |
| copy-object | ✅ | ⚪ | ⚪ | ⚪ | Includes **aws cli**, **s3cmd**, and **mc** '**cp**' command |
| create-bucket | ✅ | ✅ | ✅ | ❎ |
| create-multipart-upload | ✅ | ✅ | ⚪ | ❌ | **versitygw** has no multipart upload size minimum |
| delete-bucket | ✅ | ⚪ | ⚪ | ❎ |
| delete-bucket-policy | ✅ | ⚪ | ⚪ | ⚪ |
| delete-bucket-tagging | ✅ |  ⚪ |  ⚪ |  ⚪ |
| delete-object | ✅ | ✅ | ❌ | ❌ |
| delete-object-tagging | ✅ | ⚪  |  ⚪ |  ⚪ |
| delete-objects | ✅ | ⚪ | ⚪ | ⚪ | Refers to recursive deletion for **aws-cli s3**, **s3cmd**, and **mc** |
| get-bucket-acl | ❎ | ✅ | ❗ | ❎ | Data successfully retrieved in **s3cmd**, but needs to be parsed and verified **NOTE:** awaiting changes for the underlying command before testing |
| get-bucket-location | ❎ |  ✅ |  ✅ |  ✅ |
| get-bucket-policy | ❎ | ✅ | ✅ | ✅ |
| get-bucket-tagging | ❎ | ✅ | ❎ | ✅ | |
| get-bucket-versioning | ❎ |  ❎ |  ❎ |  ❎ | **As of May 3, 2024, not implemented** |
| get-object | ✅ | ✅ | ✅ | ✅ | Copy commands have been tested, move commands haven't |
| get-object-acl | ❎ |  ❎ |  ❎ |  ❎ | **As of May 13, 2024, not implemented** |
| get-object-attributes | ❎ |  ✅ |  ❎ |  ❎ |
| get-object-legal-hold | ❎ |  ✅ |  ❎ |  ❎ |
| get-object-lock-configuration | ❎ |  ✅ |  ❎ |  ❎ |
| get-object-retention | ❎ |  ✅ |  ❎ |  ❎ |
| get-object-tagging | ❎ | ✅ | ⚪ | ✅ |
| head-bucket | ❎ | ✅ | ✅ | ✅ |
| head-object | ❎ | ✅ | ❓ | ❓ |
| list-buckets | ✅ | ✅ | ✅ | ✅ |
| list-multipart-uploads | ❎ | ✅ | ❎ | ❎ |
| list-object-versions | ❎ |  ❎ |  ❎ |  ❎ | **As of May 20, 2024, currently not implemented, but likely to be added soon** |
| list-objects | ✅ | ✅ | ✅ | ✅ |
| list-objects-v2 | ❎ | ✅ | ❎ | ❎ |
| list-parts | ❎ | ✅ | ❎ | ❎ |
| put-bucket-acl | ❎ |  ✅ |  ❓ |  ❎ | https://github.com/versity/versitygw/issues/561 **NOTE:** this command refers to changing the local bucket ACLs for versitygw users, not the remote ACLs on S3 **ANOTHER NOTE:** awaiting changes for the underlying command before testing |
| put-bucket-policy | ❎ | ✅ | ✅ | ✅ |
| put-bucket-tagging | ❎ | ✅ | ❎ | ✅ |
| put-bucket-versioning | ❎ |  ❎ |  ❎ |  ❎ | **As of May 3, 2024, not implemented** |
| put-object | ✅ | ✅ | ✅ | ✅ |
| put-object-acl | ❎ |  ❎ |  ❎ |  ❎ | **As of May 13, 2024, not implemented** |
| put-object-legal-hold | ❎ |  ✅ |  ❎ |  ❎ |
| put-object-lock-configuration | ❎ |  ✅ |  ❎ |  ❎ |
| put-object-retention | ❎ |  ✅ |  ❎ |  ❎ |
| put-object-tagging | ❎ | ✅ | ❎ | ✅ |
| restore-object | ❎ |  ❎ |  ❎ |  ❎ | **As of May 20, 2024, not implemented** |
| select-object-content | ❎ | ❎ | ❎ | ❎ | **As of May 20, 2024, not implemented** |
| upload-part | ❎ |  ✅ |  ❎ |  ❎ |
| upload-part-copy | ❎ |  ✅ |  ❎ |  ❎ |