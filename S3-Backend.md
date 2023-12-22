The S3 backend redirects object requests to another S3 endpoint. The IAM accounts are still handled within the versitygw, and all requests to the external S3 service use the provided account credentials in the backend configuration.

# Example Command Invocation
```
./versitygw --access rootuser --secret rootpass s3 --access backenduser --secret backendpass --endpoint http://my.s3server.com
```

The `--access/--secret` after the s3 defines the credentials to use for accessing the backend S3 system.

# S3 Service Compatibility
The S3 API requests are validated on the frontend and then relayed to the backend.  So the backend must support any incoming service request to versitygw. All backend requests are regenerated using the configured backend credentials, and requests to the backend are signed using standard v4 authentication.

Some on-premise S3 systems may not allow certain APIs such as CreateBucket, etc. These would also not be allowed through the gateway.

# Configuration Args
```
   --access value, -a value  s3 proxy server access key id
   --secret value, -s value  s3 proxy server secret access key
   --endpoint value          s3 service endpoint, default AWS if not specified
   --region value            s3 service region, default 'us-east-1' if not specified (default: "us-east-1")
   --disable-checksum        disable gateway to server object checksums (default: false)
   --ssl-skip-verify         skip ssl cert verification for s3 service (default: false)
   --debug                   output extra debug tracing (default: false)
   --help, -h                show help
```

The `--access/--secret/--region/--endpoint` should be configured based on the required access to the backend S3 system.  All access to the backend system will use this single account.

The `--disable-checksum` option currently has no effect on PutObject/PutPart due to [#345](https://github.com/versity/versitygw/issues/345).  All streaming data requests have content checksums disabled.

The `--ssl-skip-verify` option is sometimes needed for on-premise S3 object storage systems that use self signed SSL certs.

# IAM Compatibility Notes
The internal versitygw IAM service is unrelated to the backend storage. All frontend accounts are still functional based on the [configured IAM service](https://github.com/versity/versitygw/wiki/Multi-Tenant). Multi-user is still functional with the s3 backend service, but all access to the backend is done solely with the configured single backend access account.

# Use Cases
* The original use case was for a performance comparison of direct S3 access vs through the gateway with a client side benchmark utility.
* Another use case might be for local gateway account management with a single backend storage system account. This setup lets the main administrator access all the stored data, while allowing multiple users to securely access and use the data through the gateway.