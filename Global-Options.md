These are options that apply to the gateway itself independent of any backend type.  When available, the shorthand option can be used in place of the long option.
***
```
   --version, -v           list versitygw version (default: false)
```
The version option will print the current binary version and exit. For example:
```bash
$ ./versitygw --version
Version  : v0.1
Build    : a881893
BuildTime: 2023-05-29_05:16:34AM
```
***
```
   --port value, -p value  gateway listen address <ip>:<port> or :<port> (default: ":7070")
```
The port option will specify the listening port for the S3 server.  This option can use either the form `<ip>:<port>` which will listen only on the network interface that matches the IP on the specified port, or `:<port>` which will listen on all network interfaces on the specified port.  The `<ip>` spec can either be IP dotted notation or a resolvable hostname.  The `<port>` spec can either be a numeric port or the service name typically in `/etc/services`.

This option will determine the S3 client endpoint to use.  For example `--port 192.168.0.1:6000` would require the S3 client to configure `http://192.168.0.1:6000` as the server endpoint.
***
```
   --access value          root user access key [$ROOT_ACCESS_KEY_ID, $ROOT_ACCESS_KEY]
   --secret value          root user secret access key [$ROOT_SECRET_ACCESS_KEY, $ROOT_SECRET_KEY]
   --region value          s3 region string (default: "us-east-1")
```
The `access` and `secret` options will specify the root account credentials. The root account is granted full authorization to all API requests after authentication. This is generally useful for creating user level buckets and assigning ACL grants. The `access` and `secret` options can be specified through environment variables (access: ROOT_ACCESS_KEY_ID or ROOT_ACCESS_KEY) (secret: ROOT_SECRET_ACCESS_KEY or ROOT_SECRET_KEY) or with the command line options. The environment variables can help to hide the credentials from process listings. The region is an optional argument, and will default to `us-east-1` if not specified.
***
```
   --cert value            TLS cert file
   --key value             TLS key file
```
The `cert` and `key` values are optional. When not specified, the server will not use TLS. To enable TLS connections, both `cert` and `key` must be provided. The value for these options are the filenames for the respective options.
***
```
   --access-log value                      enable server access logging to specified file [$LOGFILE]
```
The access-log value is optional. When defined, the server will write s3 server access log output to the specified file. It is suggested to use absolute paths for the server log file because the server may chdir into the backend root directory and change locations for relative paths. This option can also be set through LOGFILE env var. This option can only be set if log-webhook-url is not set. 
***
```
   --log-webhook-url value                 webhook url to send the audit logs [$WEBHOOK]
```
The log-webhook-url is optional.  When defined, the server will send s3 server access log entries to the provided webhook URL formatted as json. This option can also be set through WEBHOOK env var . This option can only be set if access-log is not set.
***
```
   --event-kafka-url value, --eku value    kafka server url to send the bucket notifications.
   --event-kafka-topic value, --ekt value  kafka server pub-sub topic to send the bucket notifications to
   --event-kafka-key value, --ekk value    kafka server put-sub topic key to send the bucket notifications to
```
Bucket events can be sent to a kafka message bus. When event-kafka-url, event-kafka-topic, and optionally event-kafka-key are specified, all bucket events will be sent to the kafka service.
***
```
   --event-nats-url value, --enu value     nats server url to send the bucket notifications
   --event-nats-topic value, --ent value   nats server pub-sub topic to send the bucket notifications to
```
Bucket events can be sent to a NATS messaging service. When event-nats-url and event-nats-topic are specified, all bucket events will be sent to the the NATS messaging service.
***
```
   --debug                   enable debug output (default: false)
```
The debug option will print debug output like request signing information.
***
```
   --help, -h              show help
```
The help option prints the command usage and exits.