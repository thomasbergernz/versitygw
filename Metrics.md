The gateway is capable of sending metrics to various endpoints using StatsD metrics in either the standard style or DataDog's `dogstatsd` format.

## Metrics
Metric keys are prefixed with `versitygw.`

The metrics keys are the following:
```
failed_count             non-success status requests
success_count            success status requests
bytes_written            incremented with content-length values for all PutObject/PutPart requests
object_created_count     incremented with each PutObject/CreateMultipartUpload rquest
bytes_read               incremented with payload size of each GetObject request
object_removed_count     incremented with object count for each DeleteObject/DeleteObjects request
```

All metrics are tagged with the following tags:
```
service                  hostname by default, can be overridden with --metrics-service-name option
action                   API name (ex: PutObject, GetObject, etc)
method                   HTTP method (PUT, GET, POST, DELETE)
api                      api protocol type (ex: s3)
```

## Metrics servers
The gateway supports various metrics service endpoints. When the `--metrics-statsd-servers` option is enabled, the gateway will send standard-style StatsD metrics to the configured endpoint. When the `--metrics-dogstatsd-servers` option is enabled, the gateway will send `dogstatsd`-flavored StatsD to the configured endpoint. Each of these accepts a comma separated list of endpoints, and it is fine to enable both `--metrics-statsd-servers` and `--metrics-dogstatsd-servers` at the same time.

### Prometheus
Prometheus itself does not support listening for StatsD metrics; instead, it scrapes metrics from targeted client systems. To enable Prometheus, a local agent is needed to translate listen for StatsD traffic and make Prometheus-style metrics available for scraping. For this agent, [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) from influxdata is a highly configurable agent with a [Prometheus exporter plugin](https://github.com/influxdata/telegraf/tree/release-1.4/plugins/outputs/prometheus_client), or [statsd_exporter](https://prometheus.io/download/#statsd_exporter) from the Prometheus project can be used.

When using a local agent the following option can be enabled:
```
--metrics-statsd-servers 127.0.0.1:8125
```

### InfluxDB
InfluxDB supports sending metrics directly to the InfluxDB server, or you can use [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) as a local agent that can aggregate node level metrics and forward the metrics to the InfluxDB server.

When sending directly to the server, the following option can be enabled:
```
--metrics-statsd-servers <influx_server>:8125
```
When using a local agent the following option can be enabled:
```
--metrics-statsd-servers 127.0.0.1:8125
```

### DataDog
DataDog uses their own flavor of StatsD. DataDog recommends installing their [Agent](https://docs.datadoghq.com/agent) on the gateway host system, [enabling a dogstatsd listener in the agent](https://docs.datadoghq.com/developers/dogstatsd/?tab=hostagent), and configuring DogStatsD to be sent to the local agent and then forwarded to DataDog.

When using a local agent the following option can be enabled:
```
--metrics-dogstatsd-servers 127.0.0.1:8125
```
