The following examples assume a localhost gateway running on port 7070 with a self-signed cert using the following credentials, adjust these options in the examples for your use: <br>
```
access key: myaccess
secret key: mysecret
endpoint: https://127.0.0.1:7070
```

for example:
```
ROOT_ACCESS_KEY=myaccess ROOT_SECRET_KEY=mysecret ./versitygw --cert $PWD/cert.pem --key $PWD/cert.key posix /tmp/gw
```

# aws cli
link to docs: [https://aws.amazon.com/cli/](https://aws.amazon.com/cli/)

set the following env vars:
```
export AWS_ACCESS_KEY_ID=myaccess
export AWS_SECRET_ACCESS_KEY=mysecret
export AWS_ENDPOINT_URL=https://127.0.0.1:7070
```

use `--no-verify-ssl` option for self signed certs

example:
```
aws --no-verify-ssl s3 ls s3://
```

# s3cmd
link to docs: [https://s3tools.org/s3cmd](https://s3tools.org/s3cmd)

create a configuration file such as `s3cfg.local`:
```
# Setup endpoint
host_base = 127.0.0.1:7070
host_bucket = 127.0.0.1:7070
bucket_location = us-east-1
use_https = True

# Setup access keys
access_key =  myaccess
secret_key = mysecret

# Enable S3 v4 signature APIs
signature_v2 = False
```

use `--no-check-certificate` option for self signed certs <br>
use `-c <file>` option to specify config

example:
```
s3cmd --no-check-certificate -c ./s3cfg.local ls s3://
```

# mc
link to GitHub: [https://github.com/minio/mc](https://github.com/minio/mc)

setup alias, we will call this `local`

```
mc alias s --insecure local https://127.0.0.1:7070 myaccess mysecret
```

use `--insecure` option for self signed certs

example:
```
mc ls --insecure local
```

# Cyberduck
Add new service bookmark by selecting the "+" at the bottom left of the main page.<br>
Select type as "Amazon S3"<br>
Fill out Server, Port, Access Key ID, Secret Access Key<br>

for example:<br>
![cyberduck_config](https://github.com/versity/versitygw/assets/2184287/6557e405-cac7-404a-9ff1-1123ebd9518f)

accept server certificate when asked