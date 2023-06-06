The Versity Gateway uses a stateless, distributed architecture that can enhance data throughput and overall system performance. Here's a general workflow:

**Initiating Commands:** Any command, such as initiating a multipart upload, can be sent to any gateway (for example, Gateway1) without worrying about sequence or hierarchy.

**Transferring Data:** Parts of the data being uploaded can be transferred to any other gateways (such as Gateway2 or Gateway3). This data transfer is not bound by a sequence and can be sent freely.

**Completing the Upload:** The multipart upload can be completed on yet another gateway (such as Gateway4).

The above steps are possible as long as all gateways are linked to the same underlying storage system. This allows you to treat these gateways as a distributed, load-balanced service cluster, enhancing the overall performance.

**Distributing Commands:** A load balancer can distribute commands across all these gateways, enhancing the command execution efficiency.

**Hosting Gateways:** These gateways can be hosted on different servers, adding another layer of flexibility to the system.

**Leveraging Data Throughput:** By spreading a single multipart upload session across all gateways, you can leverage the combined data throughput of all nodes, enhancing the overall performance of the system.

![Versity Gateway distributed architecture diagram](https://www.versity.com/wp-content/uploads/2023/06/Versity-GW-Stateless-3.png)