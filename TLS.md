The gateway can use TLS connections between S3 client and server if the TLS certs are provided. The cert and key must be provided to the gateway to enable TLS connections. If the server is only hosting traffic on a local network, then self signed credentials can be generated. For hosting a public facing service, it is recommended to generate official credentials using [Let's Encrypt](https://letsencrypt.org) or some other CA service.

To enable TLS, see [Global-Options](https://github.com/versity/versitygw/wiki/Global-Options) for specifying TLS cert and key files.

To generate self signed certificates, follow the direction for [mkcert](https://github.com/FiloSottile/mkcert):
```
$ mkcert -install
$ mkcert localhost 127.0.0.1 ::1
```
Add any local IPs and hostnames to the list as needed, or to generate a wildcard cert for the local domain:
```
$ mkcert "*.local.test"
```

Or you can use [openssl](https://www.openssl.org/docs/apps/req.html) to generate the certs, for example:
```
# interactive and 1 year expiration
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 365 -nodes
# non-interactive and 10 years expiration
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -sha256 -days 3650 -nodes -subj "/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=CommonNameOrHostname"
```