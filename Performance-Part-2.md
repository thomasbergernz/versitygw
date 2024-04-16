
This is cross posted on the [Versity Blog](https://www.versity.com/unlocking-the-power-of-scalability-analyzing-versity-gateways-scale-out-performance/) as well.

# Testing Goals
The Versity S3 Gateway stateless architecture allows linear scaling of multiple instances across a cluster for increased aggregate performance. Additionally, load balancers can ensure even distribution of requests across the Gateway instances. The previous [Performance](./Performance) investigation focused on a single Gateway's performance to get a better understanding of the building block capability. Now this set of tests will explore the scale out capability of the gateway.

# Test Setup
The testing environment comprised a five-system cluster running a Versity S3 Gateway instance on each system. The setup was designed to test the Gateway’s performance against the distributed ScoutFS backend filesystem.

For these tests, the raw performance numbers are highly dependent on the ScoutFS backend filesystem, primarily the storage hardware connected to the test cluster. The important characterization that we are looking for is the increase in aggregate performance as we increase the total number of gateways. So, the best case would be each gateway contributing 100% of the case of the single gateway as the number of gateway systems increases.

## Test node specs
* Dell R750
* dual socket Xeon(R) Silver 4314 16C/32T CPU @ 2.40GHz
* 256GB memory
* Centos 7.9 3.10.0-1160.105.1.el7.x86_64

## 10G Large Object PUT
![10G large object PUT](https://github.com/versity/versitygw/assets/2184287/b9d517b5-3152-4020-8e38-13463cef48dc)

First, the Gateway’s large object upload performance was tested using a multipart upload of a 10GiB object, segmented into 64MiB chunks, with varying thread counts. Tests were conducted using 2, 4, 8, 16, and 32 threads to measure how these configurations impacted upload speeds. The tests collected scaling performance using one to five gateways all on separate systems.

Results show a consistent increase in performance as gateways are added using smaller thread counts. The larger thread counts can scale up to 3 gateways linearly but start saturating the backend storage system. So while the gateway performance is expected to keep scaling linearly, the backend system was not quite capable enough to keep up with this.

## 10G Large Object GET
![10G large object GET](https://github.com/versity/versitygw/assets/2184287/9337e531-f1cf-4f12-995d-08514284f5d3)

After uploading, the next test was how well the gateway could retrieve a large 10GiB object, broken down into 64MiB download chunks, and examined across varying thread counts and gateways. Just like in the previous test, the team utilized 1, 2, 4, 8, and 16 threads and one to five gateways to understand the scaling capability of download speeds. 

The results demonstrated consistent scaling while adding more gateway systems. The larger thread counts were able to achieve a higher overall performance, and both the smaller and large thread counts were able to achieve close to linear scaling with the addition of more gateway systems. The performance between 4 and 5 gateways was slightly less than perfect scaling once again primarily due to backend storage saturation.

## 1 MB Small Object PUTs
![1MB small object PUTs](https://github.com/versity/versitygw/assets/2184287/a1a5c229-285a-46b1-babb-a8faeeb66794)

Next, the team examined the upload performance of the Versity gateway for 1MB small objects, with each client thread continuously uploading a single 1MB object. The tests evaluated how the performance varied with the addition of gateways, ranging from one to five. 

Results showed that upload speeds increased significantly with each additional gateway.

## HEAD Object Requests
![HEAD object requests_s](https://github.com/versity/versitygw/assets/2184287/434c6b25-e0c4-4ffb-9b6e-2c9232fbc5b7)

Finally, the team’s last test assessed the performance of the Versity gateway for object requests per second.  The request was a simple HEAD object that returns a given object's metadata. They measured the number of requests per second across different gateway configurations, ranging from one to five gateways.

The results showcased a substantial increase in request handling capacity as more gateways were added. This performance scales linearly with each additional gateway. This test demonstrates the gateway's impressive ability to scale up.

## Summary
In conclusion, the zero communication stateless design of the gateway allows nearly perfect scalability up to the limits of the backend storage system capability. The series of tests conducted on the Versity S3 Gateway powerfully highlights its scalability and efficiency in processing large-scale data transfers. These evaluations demonstrated that the Gateway's performance significantly enhances as additional gateways are deployed and the number of concurrent threads increases. The ability to scale up through multiple gateway instances and to boost throughput by increasing thread counts underpins the system's superior design for handling rapid and large data operations.

This dual scalability is crucial for environments that require robust throughput and quick data management, where the addition of gateways and the adjustment of thread counts can lead to significant increases in performance. The results from the tests confirm that the Versity S3 Gateway excels in scalable architecture, making it an ideal solution for organizations anticipating growth and demanding data transfer needs.
