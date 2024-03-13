These are options that apply to the gateway itself independent of any backend type.  When available, the shorthand option can be used in place of the long option. Alternatively, the optional environment variable setting is specified in []s. Setting this env var has the same effect as using the command line option. For boolean options set env var value to "true", for example `VGW_QUIET=true`.
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
   --port value, -p value  gateway listen address <ip>:<port> or :<port> (default: ":7070") [$VGW_PORT]
```
The port option will specify the listening port for the S3 server.  This option can use either the form `<ip>:<port>` which will listen only on the network interface that matches the IP on the specified port, or `:<port>` which will listen on all network interfaces on the specified port.  The `<ip>` spec can either be IP dotted notation or a resolvable hostname.  The `<port>` spec can either be a numeric port or the service name typically in `/etc/services`.

This option will determine the S3 client endpoint to use.  For example `--port 192.168.0.1:6000` would require the S3 client to configure a server endpoint that would connect to this server endpoint such as `http://192.168.0.1:6000`.
***
```
   --access value          root user access key [$ROOT_ACCESS_KEY_ID, $ROOT_ACCESS_KEY]
   --secret value          root user secret access key [$ROOT_SECRET_ACCESS_KEY, $ROOT_SECRET_KEY]
   --region value          s3 region string (default: "us-east-1") [$VGW_REGION]
```
The `access` and `secret` options will specify the root account credentials. The root account is granted full authorization to all API requests after authentication. This is generally useful for creating user level buckets and assigning ACL grants. The `access` and `secret` options can be specified through environment variables (access: ROOT_ACCESS_KEY_ID or ROOT_ACCESS_KEY) (secret: ROOT_SECRET_ACCESS_KEY or ROOT_SECRET_KEY) or with the command line options. The environment variables can help to hide the credentials from process listings. The region is an optional argument, and will default to `us-east-1` if not specified.
***
```
   --cert value            TLS cert file [$VGW_CERT]
   --key value             TLS key file [$VGW_KEY]
```
The `cert` and `key` values are optional. When not specified, the server will not use TLS. To enable TLS connections, both `cert` and `key` must be provided. The value for these options are the filenames for the respective options.
***
```
   --admin-port value, --ap value          gateway admin server listen address <ip>:<port> or :<port> [$VGW_ADMIN_PORT]
   --admin-cert value                      TLS cert file for admin server [$VGW_ADMIN_CERT]
   --admin-cert-key value                  TLS key file for admin server [$VGW_ADMIN_CERT_KEY]
```
The admin server endpoint can optionally be set to listen on a different interface or port than the S3 service. This allows for better control of firewall restrictions to the admin endpoint. The certs for this can be different certs than specified for the S3 service.
The default when these are not specified is to have the admin server listen on the same endpoint and the S3 service. 
***
```
   --quiet, -q                             silence stdout request logging output (default: false) [$VGW_QUIET]
```
The quiet option will silence the request info output enabled by default to stdout.
***
```
   --access-log value                      enable server access logging to specified file [$LOGFILE]
```
The access-log value is optional. When defined, the server will write s3 server access log output to the specified file. It is suggested to use absolute paths for the server log file because the server may chdir into the backend root directory and change locations for relative paths. This option can also be set through LOGFILE env var. This option can only be set if log-webhook-url is not set. See [LogFile](./S3-server-access-log) for more details and log format.
***
```
   --health value                          health check endpoint path. Health endpoint will be configured on GET http method: GET <health>
                                                   NOTICE: the path has to be specified with '/'. e.g /health [$VGW_HEALTH]
```
The health options specifies a health check endpoint often used for load balancers to verify gateway is alive. The health endpoint masks any bucket with this setting. For example, if the health endpoint is set to `/health`, the gateway will not allow creating or listing contents of a bucket called `health`. The health endpoint is unauthenticated, and returns a 200 status for GET.
***
```
   --log-webhook-url value                 webhook url to send the audit logs [$WEBHOOK]
```
The log-webhook-url is optional.  When defined, the server will send s3 server access log entries to the provided webhook URL formatted as json. This option can also be set through WEBHOOK env var . This option can only be set if access-log is not set. See [LogFile](./S3-server-access-log) for more details and log format.
***
```
   --event-kafka-url value, --eku value    kafka server url to send the bucket notifications. [$VGW_EVENT_KAFKA_URL]
   --event-kafka-topic value, --ekt value  kafka server pub-sub topic to send the bucket notifications to [$VGW_EVENT_KAFKA_TOPIC]
   --event-kafka-key value, --ekk value    kafka server put-sub topic key to send the bucket notifications to [$VGW_EVENT_KAFKA_KEY]
```
Bucket events can be sent to a kafka message bus. When event-kafka-url, event-kafka-topic, and optionally event-kafka-key are specified, all bucket events will be sent to the kafka service. See [Events-Notifications](./Events-Notifications) for more details and format.
***
```
   --event-nats-url value, --enu value     nats server url to send the bucket notifications [$VGW_EVENT_NATS_URL]
   --event-nats-topic value, --ent value   nats server pub-sub topic to send the bucket notifications to [$VGW_EVENT_NATS_TOPIC]
```
Bucket events can be sent to a NATS messaging service. When event-nats-url and event-nats-topic are specified, all bucket events will be sent to the the NATS messaging service. See [Events-Notifications](./Events-Notifications) for more details and format.
***
```
   --iam-dir value                         if defined, run internal iam service within this directory [$VGW_IAM_DIR]
```
The iam-dir option will enable the internal IAM service with accounts stored in a file under the specified directory. This is provided to minimize dependencies on outside services for basic functionality. The local account files are plain text and only protected with file permissions. This IAM service is added for convenience, but is not considered as secure or scalable as a dedicated IAM service. See [Multi-Tenant](./Multi-Tenant) for more details.
***
```
   --iam-ldap-url value                    ldap server url to store iam data [$VGW_IAM_LDAP_URL]
   --iam-ldap-bind-dn value                ldap server binding dn, example: 'cn=admin,dc=example,dc=com' [$VGW_IAM_LDAP_BIND_DN]
   --iam-ldap-bind-pass value              ldap server user password [$VGW_IAM_LDAP_BIND_PASS]
   --iam-ldap-query-base value             ldap server destination query, example: 'ou=iam,dc=example,dc=com' [$VGW_IAM_LDAP_QUERY_BASE]
   --iam-ldap-object-classes value         ldap server object classes used to store the data. provide it as comma separated string, example: 'top,person' [$VGW_IAM_LDAP_OBJECT_CLASSES]
   --iam-ldap-access-atr value             ldap server user access key id attribute name [$VGW_IAM_LDAP_ACCESS_ATR]
   --iam-ldap-secret-atr value             ldap server user secret access key attribute name [$VGW_IAM_LDAP_SECRET_ATR]
   --iam-ldap-role-atr value               ldap server user role attribute name [$VGW_IAM_LDAP_ROLE_ATR]
```
The ldap options will enable the LDAP IAM service with accounts stored in an external LDAP service. The iam-ldap-access-atr, iam-ldap-secret-atr, and iam-ldap-role-atr define the LDAP attributes that map to access, secret credentials and role respectively. See [Multi-Tenant](./Multi-Tenant) for more details.
***
```
   --s3-iam-access value                   s3 IAM access key [$VGW_S3_IAM_ACCESS_KEY]
   --s3-iam-secret value                   s3 IAM secret key [$VGW_S3_IAM_SECRET_KEY]
   --s3-iam-region value                   s3 IAM region (default: "us-east-1") [$VGW_S3_IAM_REGION]
   --s3-iam-bucket value                   s3 IAM bucket [$VGW_S3_IAM_BUCKET]
   --s3-iam-endpoint value                 s3 IAM endpoint [$VGW_S3_IAM_ENDPOINT]
   --s3-iam-noverify                       s3 IAM disable ssl verification (default: false) [$VGW_S3_IAM_NO_VERIFY]
   --s3-iam-debug                          s3 IAM debug output (default: false) [$VGW_S3_IAM_DEBUG]
```
The S3 IAM service is similar to the internal IAM service, but instead stores the account information JSON encoded in an S3 object. This should use a bucket that is not accessible to general users when using s3 backend to prevent access to account credentials. This IAM service is added for convenience, but is not considered as secure or scalable as a dedicated IAM service. See [Multi-Tenant](./Multi-Tenant) for more details.
***
```
   --iam-cache-disable                     disable local iam cache (default: false) [$VGW_IAM_CACHE_DISABLE]
   --iam-cache-ttl value                   local iam cache entry ttl (seconds) (default: 120) [$VGW_IAM_CACHE_TTL]
   --iam-cache-prune value                 local iam cache cleanup interval (seconds) (default: 3600) [$VGW_IAM_CACHE_PRUNE]
```
The IAM cache is intended to ease the load on the IAM service and increase the Gateway performance by caching accounts and credentials for the TTL time interval. Disabling this will cause a request to the configured IAM service for each incoming request to retrieve the corresponding account credentials. The cache is enabled by default. The TTL specifies how long to cache credentials, and the prune value determines the interval for expired entries to be removed from the cache. Increasing the TTL may lessen the load on the IAM service backend, but may have out of date account info until the next interval. Increasing the prune value may reduce memory use at the cost of added CPU to check cache expirations. See [Multi-Tenant](./Multi-Tenant#iam-cache) for more details.
***
```
   --debug                   enable debug output (default: false) [$VGW_DEBUG]
```
The debug option will print debug output like request signing information.
***
```
   --help, -h              show help
```
The help option prints the command usage and exits.