The versitygw project has both client based integration tests (primarily for testing backends) and unit tests (primarily for testing the api handlers).

# Client
The versitygw command has a built in client test suite. This can be invoked against any running S3 service to check for S3 compatibility. This is useful when developing new backends to verify implementation. Run the gateway with whatever backend desired for testing.

For example, running with posix:
```
ROOT_ACCESS_KEY=myaccess ROOT_SECRET_KEY=mysecret ./versitygw posix /tmp/gw
```
Substitute posix and args for desired backend to test.

Then run the client side test suite:
```
./versitygw test -a myaccess -s mysecret -e http://127.0.0.1:7070 full-flow
```
The full-flow option will run through all of the tests in the [integration tests].(https://github.com/versity/versitygw/blob/main/integration/tests.go)

There is also a performance benchmark test to test upload and download speeds with various parameters. This can be invoked with the `bench` option.
```
% ./versitygw test bench -h       
NAME:
   versitygw test bench - Runs download/upload performance test on the gateway

USAGE:
   versitygw test bench [command options] [arguments...]

DESCRIPTION:
   Uploads/downloads some number(specified by flags) of files with some capacity(bytes).
         Logs the results to the console

OPTIONS:
   --files value        Number of objects to read/write (default: 1)
   --objsize value      Uploading object size (default: 0)
   --prefix value       Object name prefix
   --upload             Upload data to the gateway (default: false)
   --download           Download data to the gateway (default: false)
   --bucket value       Destination bucket name to read/write data
   --partSize value     Upload/download size per thread (default: 67108864)
   --concurrency value  Upload/download threads per object (default: 1)
   --pathStyle          Use Pathstyle bucket addressing (default: false)
   --checksumDis        Disable server checksum (default: false)
   --help, -h           show help
```

# Unit
Invoke unit tests with the following:
```
go test ./...
```

If the [backend.Backend](https://github.com/versity/versitygw/blob/main/backend/backend.go#L28) interface definition changes, then the moq test definitions need to be re-generated:

Install moq if not done already:
```
go install github.com/matryer/moq@latest
```
Regenerate moq test definitions:
```
cd backend
go generate
```
This will regenerate this file:
```
/s3api/controllers/backend_moq_test.go
```
The tests in /s3api may need to be updated for any interface changes.

If the [auth.IAMService](https://github.com/versity/versitygw/blob/main/auth/iam.go#L31) interface definition changes, then the moq test definitions need to be re-generated:
```
cd auth
go generate
```
This will regenerate this file:
```
/s3api/controllers/iam_moq_test.go
```
The tests in /s3api may need to be updated for any interface changes.

# REST

As with **s3**, **versitygw** can be communicated with with the same REST API commands.  Below is an example of a script for the list-buckets command, probably the easiest command to send in REST format:

```
#!/usr/bin/env bash

# sample script for s3 list-buckets REST command

# Fields

payload=""
host="localhost:7070"
aws_region="<AWS region>"
aws_access_key_id="<key ID>"
aws_secret_access_key="<access key>"

# Step 1:  generate payload hash

payload_hash="$(echo -n "$1" | sha256sum | awk '{print $1}')"

# Step 2:  generate canonical hash

current_date_time=$(date -u +"%Y%m%dT%H%M%SZ")

canonical_request="GET
/

host:$host
x-amz-content-sha256:$payload_hash
x-amz-date:$current_date_time

host;x-amz-content-sha256;x-amz-date
$payload_hash"

canonical_request_hash="$(echo -n "$canonical_request" | openssl dgst -sha256 | awk '{print $2}')"

# Step 3:  create STS data string

year_month_day="$(echo "$current_date_time" | cut -c1-8)"

sts_data="AWS4-HMAC-SHA256
$current_date_time
$year_month_day/$aws_region/s3/aws4_request
$canonical_request_hash"

# Step 4:  generate signature

date_key=$(echo -n "$year_month_day" | openssl dgst -sha256 -mac HMAC -macopt key:"AWS4${aws_secret_access_key}" | awk '{print $2}')
date_region_key=$(echo -n "$aws_region" | openssl dgst -sha256 -mac HMAC -macopt hexkey:"$date_key" | awk '{print $2}')
date_region_service_key=$(echo -n "s3" | openssl dgst -sha256 -mac HMAC -macopt hexkey:"$date_region_key" | awk '{print $2}')
signing_key=$(echo -n "aws4_request" | openssl dgst -sha256 -mac HMAC -macopt hexkey:"$date_region_service_key" | awk '{print $2}')
signature=$(echo -n "$sts_data" | openssl dgst -sha256 \
                 -mac HMAC \
                 -macopt hexkey:"$signing_key" | awk '{print $2}')

# Step 5:  send curl command

curl -ks https://$host/ \
       -H "Authorization: AWS4-HMAC-SHA256 Credential=$aws_access_key_id/$year_month_day/$aws_region/s3/aws4_request,SignedHeaders=host;x-amz-content-sha256;x-amz-date,Signature=$signature" \
       -H "x-amz-content-sha256: $payload_hash" \
       -H "x-amz-date: $current_date_time"
```

**NOTES**:
* If sending a REST command with a payload, it needs to be added, and can be added to the **payload** parameter.  And as with the canonical request and STS strings, ensure that no extra space or unnecessary newlines are in the payload.
* If sending a message direct to S3 for comparison purposes, change the **host** parameter to `s3.amazonaws.com`, or `<bucket>.s3.amazonaws.com` if sending a REST command to a specific bucket
* Add the region, AWS user ID, and secret key
* The canonical request string varies by command.  To find the info/params for the command, S3 has documentation, e.g. https://docs.aws.amazon.com/AmazonS3/latest/API/API_ListBuckets.html.  And info on how to convert to a canonical request string:  https://docs.aws.amazon.com/AmazonS3/latest/userguide/RESTAuthentication.html.
* If using `http` rather than `https`, change the protocol in the curl command.
* If there's a mistake, and sending directly to S3, S3 will return the expected correct signatures for the payload and the canonical request.  These can be used to debug.

