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