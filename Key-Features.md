The Versity Gateway uses a stateless, distributed architecture that can enhance data throughput and overall system performance. 

Key Features
Scalability
Multiple Versity Gateway instances may be deployed in a cluster to increase aggregate throughput. The Versity Gateway’s stateless architecture allows any request to be serviced by any gateway thereby distributing workloads and enhancing performance. Load balancers may be used to evenly distribute requests across the cluster of gateways for optimal performance. 

High Performance
Developed from scratch in Go, a programming language celebrated for its performance and scalability, the Versity Gateway leverages the capabilities of Go to deliver unmatched speed and efficiency. The Versity Gateway utilizes Go Fiber, a lightweight and high-performance HTTP server framework, to handle incoming requests. Compared to older web frameworks like gorilla/mux, Fiber offers improved performance, resulting in faster processing and response times.

Modular Backend Support
The Versity Gateway is designed with modularity in mind, enabling future extensions to support additional backend storage systems. At present, the Versity Gateway supports any generic POSIX file backend storage and Versity’s open source ScoutFS filesystem.  

![Versity Gateway distributed architecture diagram](https://www.versity.com/wp-content/uploads/2023/06/Versity-GW-Stateless-2.png)