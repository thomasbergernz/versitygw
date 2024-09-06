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

As with **s3**, **versitygw** can be communicated with with the same REST API commands.  Below is an example of the step-by-step process.  Note that if doing something besides reproducing REST bugs, it's a very good idea to write a script that does this or find and copy the code in the versitygw codebase that does this automatically for the command needed, since it's a long process.

## Generate Payload Hash

With a bash terminal, execute the following:  `echo -n "<payload>" | sha256sum | awk '{print $1}'`.  This will generate the hash payload.

With larger payloads, the data can be stored in a file, and the payload can be generated with `cat <file> | sha256sum | awk '{print $1}'`.

For empty payloads, this value is always `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`.

## Generate Canonical Request Hash

S3 documentation has examples of what the canonical request should look like.  For example, for list-buckets (probably the easiest REST command to send), the format is:

```
GET
/

host:s3.amazonaws.com
x-amz-content-sha256:<payload hash, in this case, "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855">
x-amz-date:<current time in ISO8601 format>

host;x-amz-content-sha256;x-amz-date
<payload hash, in this case, "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855">
```

To get the ISO8601 date, the following bash command can be used: `date -u +"%Y%m%dT%H%M%SZ"`.  Also note that this date is valid for 15 minutes after it is generated.

After this request is created, the hash for this request can be generated.  This can be done in a bash file, e.g.:

```
#/usr/bin/env bash

canonical_request="GET
/

host:s3.amazonaws.com
x-amz-content-sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
x-amz-date:20240906T163800Z

host;x-amz-content-sha256;x-amz-date
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"

creq_hash="$(echo -n "$canonical_request" | openssl dgst -sha256 | awk '{print $2}')"
echo $creq_hash
```



