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
   --access value          admin access account [$ADMIN_ACCESS_KEY_ID, $ADMIN_ACCESS_KEY]
   --secret value          admin secret access key [$ADMIN_SECRET_ACCESS_KEY, $ADMIN_SECRET_KEY]
   --region value          s3 region string (default: "us-east-1")
```
The `access` and `secret` options will specify the admin account credentials. The admin account is granted full authorization to all API requests after authentication. This is generally useful for creating user level buckets and assigning ACL grants. The `access` and `secret` options can be specified through environment variables (access: ADMIN_ACCESS_KEY_ID or ADMIN_ACCESS_KEY) (secret: ADMIN_SECRET_ACCESS_KEY or ADMIN_SECRET_KEY) or with the command line options. The environment variables can help to hide the credentials from process listings. The region is an optional argument, and will default to `us-east-1` if not specified.
***
```
   --cert value            TLS cert file
   --key value             TLS key file
```
The `cert` and `key` values are optional. When not specified, the server will not use TLS. To enable TLS connections, both `cert` and `key` must be provided. The value for these options are the filenames for the respective options.
***
```
   --help, -h              show help
```
The help option prints the command usage and exits.