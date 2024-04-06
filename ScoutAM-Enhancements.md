# ScoutAM Specific Enhancements

The Versity Gateway works alongside [ScoutAM](https://www.versity.com/products/scoutam/), Versity's commercial mass storage data management product, to achieve a on premise Glacier like workflow. This combination allows users to store, retrieve, and manage large volumes of data across different storage systems including tape storage systems. The integration of Versity Gateway and ScoutAM allows deployment of massive archive systems on premise with the compatibility of cloud archive APIs.

## Optimized Writes
With the Versity Gateway, multipart upload parts are written once to the underlying storage then combined into a single file during the complete multipart upload call. Using the scoutfs move blocks interface, the data from one file can be moved to another file by updating the metadata records only without needing to read or write the data again. This optimization is very impactful because it eliminates one full read/write cycle during final multipart object assembly. This can double the performance for large object ingest.

Optimized writes leverage a ScoutFS specific interface when using the Versity [ScoutFS](https://www.versity.com/products/scoutfs/) filesystem as the backend storage platform.

## S3 Glacier Mode 
The Versity Gateway supports "Glacier Mode" which is a cold storage feature for data archiving and long-term data retention. In Glacier Mode, data may be stored on a separate tier (such as local tape systems) using Versity’s [ScoutAM](https://www.versity.com/products/scoutam/) platform. This capability allows organizations to manage their storage costs more effectively by moving the majority of the archive data capacity to lower cost storage. The Versity Gateway's support for Glacier Mode ensures that organizations can seamlessly integrate their existing data lifecycle client workflows with ScoutAM’s powerful file storage and management capabilities.

The following behavior is enabled with Glacier Mode:
```
GET object:  if file offline, return invalid object state

HEAD object: if file offline, set obj storage class to GLACIER
             if file offline and staging, x-amz-restore: ongoing-request="true"
             if file offline and not staging, x-amz-restore: ongoing-request="false"
             if file online, x-amz-restore: ongoing-request="false", expiry-date="Fri, 2 Dec 2050 00:00:00 GMT"
             note: this expiry-date is not used but provided for client glacier compatibility

ListObjects: if file offline, set obj storage class to GLACIER

RestoreObject: add batch stage request to file
```

![Versity Gateway and ScoutAM Diagram](https://github.com/versity/versitygw/blob/assets/assets/scoutam_gateway.png)
