# Download Gateway
## Download release binary
Download a release binary from [Releases](https://github.com/versity/versitygw/releases)

## Build from source
This assumes you have a [Go](https://go.dev) compiler installed
### Clone repo
```
git clone https://github.com/versity/versitygw.git
```
### Build project
```
cd versitygw
make
```

# Run Gateway
### Create backend directory if it doesn't already exist
```
mkdir /tmp/gw
```
### Start Gateway
Set the root account credentials with the ROOT_ACCESS_KEY and ROOT_SECRET_KEY environment variables. These can alternatively be set with `--access` and `--secret` cli options.
```
ROOT_ACCESS_KEY="myaccess" ROOT_SECRET_KEY="mysecret" ./versitygw posix /tmp/gw
```

# Test client interactions
### Using aws cli
Set client credentials to root account.
```
export AWS_ACCESS_KEY_ID=myaccess
export AWS_SECRET_ACCESS_KEY=mysecret
```
Create a bucket. Replace `127.0.0.1` with server IP if gateway running on different host.
```
aws --endpoint-url http://127.0.0.1:7070 s3 mb s3://mybucket
```
Upload a file "localfile" to the S3 gateway calling it "/test/file" within the bucket we just created. 
```
aws --endpoint-url http://127.0.0.1:7070 s3 cp localfile s3://mybucket/test/file
```