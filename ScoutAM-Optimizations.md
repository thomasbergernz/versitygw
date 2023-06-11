The Versity Gateway integrates effortlessly with [ScoutAM](https://www.versity.com/products/scoutam/), Versity's commercial mass storage data management product. This combination allows users to store, retrieve, and manage large volumes of data across different storage systems including on premises tape storage systems, thereby simplifying data management workflows. The integration of Versity Gateway and ScoutAM delivers enhanced performance and scalability while ensuring seamless access to object storage.

## Optimized Writes
With the Versity Gateway, multi-part upload segments are written once to the underlying storage then combined into a single file with a system call with metadata updates only. This optimization is very impactful because it eliminates one full read/write cycle by reducing the amount of data that needs to be processed during final mutlipart object assembly, which speeds up the process and reduces overall upload time. This is particularly beneficial for handling large files as it improves efficiency and system performance.

Optimized writes leverage a ScoutFS specific interface so are only available when using the Versity [ScoutFS](https://www.versity.com/products/scoutfs/) filesystem as the backend storage platform. However, the community is free to add similar optimizations for any filesystem that would enable such a feature.

## S3 Glacier Mode 
The Versity Gateway supports "Glacier Mode," which is a cold storage feature for data archiving and long-term data retention. In Glacier Mode, data may be stored on a separate tier (such as local tape systems) using Versity’s [ScoutAM](https://www.versity.com/products/scoutam/) platform. This capability allows organizations to manage their storage costs more effectively by accessing low cost storage. The Versity Gateway's support for Glacier Mode ensures that organizations can seamlessly integrate their existing data lifecycle client workflows with ScoutAM’s powerful file storage and management capabilities.

![Versity Gateway and ScoutAM Diagram](https://github.com/versity/versitygw/blob/assets/assets/scoutam_gateway.png)
