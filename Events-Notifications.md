Event notifications can be sent to a messaging service. The currently implemented services are Kafka and NATS.

The gateway events work similar to AWS S3 events as described in:<br>
https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html and the event structure is documented in https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-content-structure.html. The gateway event schema can be found in https://github.com/versity/versitygw/blob/main/s3event/event.go defined as EventSchema. The events are globally enabled for all buckets when set.  See below for filter configuration on what events get notification sent.

Kafka is configured with the following options:
```
   --event-kafka-url value, --eku value    kafka server url to send the bucket notifications.
   --event-kafka-topic value, --ekt value  kafka server pub-sub topic to send the bucket notifications to
   --event-kafka-key value, --ekk value    kafka server put-sub topic key to send the bucket notifications to
```

NATS is configured with the following options:
```
   --event-nats-url value, --enu value     nats server url to send the bucket notifications
   --event-nats-topic value, --ent value   nats server pub-sub topic to send the bucket notifications to
```

WEBHOOK is configured with the following options:
```
   --event-webhook-url value, --ewu value  webhook url to send bucket notifications [$VGW_EVENT_WEBHOOK_URL]
```
see [Webhook Logs](./Webhook-log-entries) for example webhook service.

### Filters
By default, all event types will have a notification sent to the configured service. You can specify a filter to configure only specific event types to trigger a notification. Run the following to generate a default filter rule file in /tmp:
```
versitygw utils gen-event-filter-config --path /tmp
```

This will create the file `/tmp/event_config.json` with all notification types set to true:
```
{
  "s3:ObjectAcl:Put": true,
  "s3:ObjectCreated:*": true,
  "s3:ObjectCreated:CompleteMultipartUpload": true,
  "s3:ObjectCreated:Copy": true,
  "s3:ObjectCreated:Post": true,
  "s3:ObjectCreated:Put": true,
  "s3:ObjectRemoved:Delete": true,
  "s3:ObjectRestore:*": true,
  "s3:ObjectRestore:Completed": true,
  "s3:ObjectRestore:Post": true,
  "s3:ObjectTagging:*": true,
  "s3:ObjectTagging:Delete": true,
  "s3:ObjectTagging:Put": true
}
```

Modify the file to only set the desired event notifications to true, and all others to false. Copy the file to a good permanent location such as `/etc/versitygw.d/event_config.json`. Then use the following to enable the filter, where the value is the full path to the config file:
```
   --event-filter value, --ef value        bucket event notifications filters configuration file path [$VGW_EVENT_FILTER]
```