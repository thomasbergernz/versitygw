When the --access-log option is defined, the gateway will append log entries to the specified log file. The log file format follows the AWS S3 log file format documented in https://docs.aws.amazon.com/AmazonS3/latest/userguide/LogFormat.html.

Example output:
```
- - [01/August/2023:21:11:35 -0700] 127.0.0.1 admin 0C0C4A5F8759E883 ListBucket - http://127.0.0.1:7070/ 200 - 481 0 1 1 - aws-cli/2.2.25 Python/3.8.8 Darwin/22.5.0 exe/x86_64 prompt/off command/s3.ls - - SigV4 - AuthHeader s3.us-east-1.amazonaws.com - arn:aws:s3:::/ Yes
acct1 bucket1 [01/August/2023:21:11:52 -0700] 127.0.0.1 acct1 0B9570A292BEE1E5 ListObjectsV2 - http://127.0.0.1:7070/bucket1?list-type=2&prefix=&delimiter=%2F&encoding-type=url 200 - 359 0 0 0 - aws-cli/2.2.25 Python/3.8.8 Darwin/22.5.0 exe/x86_64 prompt/off command/s3.ls - - SigV4 - AuthHeader s3.us-east-1.amazonaws.com - arn:aws:s3:::/bucket1 Yes
```

The log can be rotated by sending SIGHUP to the gateway. This will cause the gateway to close and re-open the logfile without needing to restart the gateway.