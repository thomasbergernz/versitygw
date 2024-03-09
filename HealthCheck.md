# Health endpoint
The Health Check endpoint can be optionally enabled for load balancer backend health checks. This allows the load balancer to verify the gateway is up and listening on the configured port, and only use gateways that are up and alive.

The health endpoint is configured with the option:
```
   --health value                          health check endpoint path. Health endpoint will be configured on GET http method: GET <health>
                                                   NOTICE: the path has to be specified with '/'. e.g /health [$VGW_HEALTH]
```

When set, the configured endpoint will return 200 status (OK) for **unauthenticated http GET**. This will mask bucket actions at that endpoint. For example, if `--health` is set to `/health` then a bucket called "health" cannot be created or contents listed (if already created).

Example:
```
% curl http://127.0.0.1:7070/health
OK%                
```

S3 client access is expected to fail, since this is not a valid bucket endpoint:
```
% aws s3 ls s3://health

Unable to parse response (syntax error: line 1, column 0), invalid XML received. Further retries may succeed:
b'OK'
```

# Example load balancer config
## HAProxy
The uri option should match the configured health endpoint. Other options may be needed for SSL enabled gateways.
```
backend gateways
  option httpchk
  http-check send meth GET  uri /health
  server server1 192.168.0.1:7070 check
  server server2 192.168.0.2:7070 check
  server server3 192.168.0.3:7070 check
```