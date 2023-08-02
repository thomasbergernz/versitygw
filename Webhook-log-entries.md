Webhook logs will send json encoded logs with the same fields as defined in AWS S3 server logs: https://docs.aws.amazon.com/AmazonS3/latest/userguide/LogFormat.html.

To enable webhook logs, use the --log-webhook-url option to specify the webhook url where logs should be sent.

Below is a simple example webhook server that will listen on port 8080 for /s3logs:
```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"strings"
)

func main() {
	http.HandleFunc("/s3logs", func(w http.ResponseWriter, r *http.Request) {
		defer r.Body.Close()
		buf := new(strings.Builder)
		io.Copy(buf, r.Body)
		fmt.Println(buf.String())
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
```
go build -o webhooktest
./webhooktest
```

If the above is built and run, and the gateway is started like the following to send logs to our running webhook service:
```
./versitygw -a user -s password --log-webhook-url http://localhost:8080/s3logs posix /tmp/gw
```

Then an initial empty message is sent to check connectivity and all following access will print to stdout from the webhook service. For example:
```

{"BucketOwner":"acct1","Bucket":"bucket1","Time":"2023-08-01T21:25:10.004812-07:00","RemoteIP":"127.0.0.1","Requester":"user","RequestID":"ACA601CDB88E80CA","Operation":"ListObjectsV2","Key":"","RequestURI":"http://127.0.0.1:7070/bucket1?list-type=2\u0026prefix=\u0026delimiter=%2F\u0026encoding-type=url","HttpStatus":200,"ErrorCode":"","BytesSent":359,"ObjectSize":0,"TotalTime":1,"TurnAroundTime":1,"Referer":"","UserAgent":"aws-cli/2.2.25 Python/3.8.8 Darwin/22.5.0 exe/x86_64 prompt/off command/s3.ls","VersionID":"","HostID":"","SignatureVersion":"SigV4","CipherSuite":"","AuthenticationType":"AuthHeader","HostHeader":"s3.us-east-1.amazonaws.com","TLSVersion":"","AccessPointARN":"arn:aws:s3:::/bucket1","AclRequired":"Yes"}
```