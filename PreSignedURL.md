The versitygw has support for using presigned URLs. This allows providing access to objects for a certain duration without the need to share credentials.

See [https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html) for more details.

Example:
```
% aws --endpoint-url http://localhost:7070 s3 presign s3://my-bucket/my-object  --expires-in 604800
http://localhost:7070/my-bucket/my-object?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=test%2F20240309%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240309T180642Z&X-Amz-Expires=604800&X-Amz-SignedHeaders=host&X-Amz-Signature=c93c25defd5477d65b3bed11e36c77f20230e01a07416190fadf74c1a68ef4f2
```

The returned URL can be used to retrieve the object contents until the expiration time.