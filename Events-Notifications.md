Event notifications can be sent to a messaging service. The currently implemented services are Kafka and NATS.

The gateway events work similar to AWS S3 events as described in https://docs.aws.amazon.com/AmazonS3/latest/userguide/EventNotifications.html and the event structure is documented in https://docs.aws.amazon.com/AmazonS3/latest/userguide/notification-content-structure.html. The gateway event schema can be found in https://github.com/versity/versitygw/blob/main/s3event/event.go defined as EventSchema.

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