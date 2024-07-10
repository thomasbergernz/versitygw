VersityGW has a comprehensive integration test suite for validating various clients against the S3 service. These tests are automatically run with GitHub Actions on all pull requests before merging and main when updated. This helps to ensure compatible functionality for tested clients with all code changes.

Not all tests are relevant for all backend storage system types. See the backend system specific page for details on supported operations for that backed system. The tables below are more reflective of the gateway frontend handler capabilities, and if the api service is being tested with any backend.

## Test environment

The tests are all written in bash using [bats](https://bats-core.readthedocs.io/en/stable/) test framework. This was selected to easily run the client commands as a user would. This also gives good test output so that these can be run in CI jobs.

## Running the tests

The tests can be run directly if all of the dependencies are available, or can can be run from a docker container. See [tests README](https://github.com/versity/versitygw/tree/main/tests) for more details on running test.

## Test Matrix

| symbol | meaning |
| :------: | ------- |
| ✅ | tested and passing |
| ❌ | tested and failing |
| ⚪ | not tested |
| ❗ | partially tested |
| ❎ | not applicable |
| ❓ | needs to be verified |

### supported operations
| API | aws-cli s3 | aws-cli s3api | s3cmd | mc | Notes |
| :---: | :------: | :-------: | :-----: | :--: | ----- |
| abort-multipart-upload | ❎ | ✅ | ❎ | ❎ ||
| complete-multipart-upload | ✅ | ✅ | ✅ | ✅ | No way to manually trigger multipart upload in **aws-cli s3**, **s3cmd**, **mc**, will automatically do so if file is above certain size |
| copy-object | ✅ | ✅ | ✅ | ✅ | Includes **aws cli**, **s3cmd**, and **mc** '**cp**' command |
| create-bucket | ✅ | ✅ | ✅ | ✅ |
| create-multipart-upload | ❎ | ✅ | ❎ | ❎ | **versitygw** has no multipart upload size minimum |
| delete-bucket | ✅ | ✅ | ✅ | ✅ |
| delete-bucket-policy | ❎ | ✅ | ✅ | ✅ |
| delete-bucket-tagging | ❎ |  ✅ |  ❎ |  ✅ |
| delete-object | ✅ | ✅ | ✅ | ✅ |
| delete-object-tagging | ❎ | ✅  |  ❎ |  ✅ |
| delete-objects | ✅ | ✅ | ✅ | ✅ | Refers to recursive deletion for **aws-cli s3**, **s3cmd**, and **mc** |
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

| commands | s3 | s3api | s3cmd | mc |
| :------: | :-: | :---: | :---: | :-: |
| generate presign URL |  ✅ | ❎ |  ❌ |  ✅ |

### User tests

| User type | aws | s3cmd | mc |
|:---------:|:---:|:-----:|:--:|
| admin | ✅ | ✅ | ⚪ |
| userplus | ✅ | ✅ | ⚪ |
| user | ✅ | ✅ | ⚪ |

### s3 api ops not expected to be used by gateway
(open an issue to request functionality)

| API | aws-cli s3api | s3cmd | mc |
| :---: | :-------: | :-----: | :--: |
| delete-bucket-analytics-configuration |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-cors |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-encryption |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-intelligent-tiering-configuration |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-inventory-configuration |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-lifecycle |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-metrics-configuration |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-ownership-controls |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-replication |  ⚪ |  ⚪ |  ⚪ |
| delete-bucket-website |  ⚪ |  ⚪ |  ⚪ |
| delete-public-access-block |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-accelerate-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-analytics-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-cors |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-encryption |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-intelligent-tiering-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-inventory-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-lifecycle-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-logging |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-metrics-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-notification-configuration |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-ownership-controls |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-policy-status |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-replication |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-request-payment |  ⚪ |  ⚪ |  ⚪ |
| get-bucket-website |  ⚪ |  ⚪ |  ⚪ |
| list-bucket-analytics-configurations |  ⚪ |  ⚪ |  ⚪ |
| list-bucket-intelligent-tiering-configurations |  ⚪ |  ⚪ |  ⚪ |
| list-bucket-inventory-configurations |  ⚪ |  ⚪ |  ⚪ |
| list-bucket-metrics-configurations |  ⚪ |  ⚪ |  ⚪ |
| get-object-torrent |  ⚪ |  ⚪ |  ⚪ |
| get-public-access-block |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-accelerate-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-analytics-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-cors |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-encryption |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-intelligent-tiering-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-inventory-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-lifecycle-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-logging |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-metrics-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-notification-configuration |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-ownership-controls |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-replication |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-request-payment |  ⚪ |  ⚪ |  ⚪ |
| put-bucket-website |  ⚪ |  ⚪ |  ⚪ |
| put-public-access-block |  ⚪ |  ⚪ |  ⚪ |
| write-get-object-response |  ⚪ |  ⚪ |  ⚪ |

## Test failure notes
The s3cmd generate presign URL does not appear to provide all required request headers, see [#478](https://github.com/versity/versitygw/issues/478).
