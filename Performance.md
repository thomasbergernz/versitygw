# Testing Goals
The Versity S3 Gateway stateless architecture allows linear scaling of multiple instances across a cluster for increased aggregate performance. Additionally, load balancers can ensure even distribution of requests across the Gateway instances. This investigation focused on a single Gateway's performance to get a better understanding of the building block capability. The frontend performance is achieved with the use of [Fiber](https://github.com/gofiber/fiber#-benchmarks), an Express inspired web framework built on top of Fasthttp, the fastest HTTP engine for Go.

# Test Setup
In the Versity lab, the team executed comprehensive tests to evaluate the performance of accessing an object storage system through the Versity S3 Gateway comparing performance of direct access to the object storage system. The team also implemented a new "[noop](https://github.com/versity/versitygw/tree/ben/noop)" backend that could test the performance of the request handling without needing a backend storage system capable of the anticipated peak performance. This was done to characterize the performance capability of the Gateway request handlers.

## Test node specs
* Dell R750
* dual socket Xeon(R) Silver 4314 16C/32T CPU @ 2.40GHz
* 256GB memory
* Centos 7.9 3.10.0-1160.105.1.el7.x86_64

## Test environment
The testing environment comprised a single system running the s3bench client benchmark utility and a Versity S3 Gateway instance. The setup was designed to test the Gateway's performance against a 10-server S3 service cluster. To distribute requests across servers, a localhost pound proxy was used for balanced load across S3 servers.

S3 service direct test:<br>
s3bench -> pound -> S3 service<br>
Gateway proxy to S3 service test:<br>
s3bench -> versitygw -> pound -> S3 service<br>

For all tests, standard S3 v4 authentication request signing was used. This means that the Gateway must use a SHA256 checksum for the incoming data payload to verify the signature. Even with the tests that used the noop backend and did not store data, the entire upload data must be read and checksummed to verify the request signature. So it is expected that significant upload request load would cause significant CPU use due to the need to checksum all of the data payloads. The Gateway will stream the data to the backend service while uploading and defer the final signature validation until the incoming data stream ends.

While running tests, the gateway memory and cpu profile was captured. The output “Maximum resident set size (kbytes)” displays the peak memory used by the gateway during the test, and “Percent of CPU this job got” gives some idea of the CPU utilization of the gateway under the various test load scenarios. This was captured by running the gateway using `/usr/bin/time -v`.

With no client workload running, the gateway did not consume much CPU and only allocated about 12 MB of memory. This was captured as a baseline control.
```
	User time (seconds): 0.00
	System time (seconds): 0.00
	Percent of CPU this job got: 0%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:03.31
	Maximum resident set size (kbytes): 11908
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 1564
	Voluntary context switches: 133
	Involuntary context switches: 29
	Page size (bytes): 409
```

# S3 direct vs Gateway Proxy
## Multiple 1 MB Object Load Test
The first test is to test the scaling of simultaneous object PUTs. The objects are 1 MiB in size and are uploaded with a single PUT. The lines are comparing the benchmark workload directly to the S3 service and through the gateway. The horizontal axis is the number of simultaneous uploads, and the vertical axis is the aggregate throughput. The tests were run multiple times and the average was taken for the graph value. There was some variability in performance across the runs with the same settings. So the difference in performance at some values appears to be mostly due to test run variability and not attributed to any significant gateway performance overhead. This is mainly confirmed due to the crossover in performance at the 350 simultaneous uploads test.

![1 MiB Object PUT](https://github.com/versity/versitygw/assets/2184287/fd211750-d014-45dc-b1c9-7cca4e891bc1)

test command:
```
for i in 100 150 200 250 300 350; do
    echo "$i objects:"
    for j in `seq 1 5`; do
        ./s3bench -access $access -secret $secret -bucket $bucket -endpoint $endpoint -objectsize 1048576 -pathstyle -prefix benchtest/ -upload -delete -n $i | grep run
    done
done
```

memory and CPU profile:<br>
There is minimal CPU use per object, and the 350 simultaneous objects caused the Gateway to only consume about 35% CPU. The Gateway allocated 6.8 MB over baseline when starting to handle incoming requests, and peaked at 80 MB (68 MB over baseline) with the 350 concurrent object test.

Single 1 MiB PUT:
```
	User time (seconds): 0.01
	System time (seconds): 0.00
	Percent of CPU this job got: 0%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:10.15
	Maximum resident set size (kbytes): 18764
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 2363
	Voluntary context switches: 602
	Involuntary context switches: 29
	Page size (bytes): 4096
```

350 concurrent 1 MiB PUT:
```
	User time (seconds): 1.04
	System time (seconds): 0.68
	Percent of CPU this job got: 35%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:04.84
	Maximum resident set size (kbytes): 79740
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 4688
	Voluntary context switches: 9903
	Involuntary context switches: 639
	Page size (bytes): 4096
```

## 10 GiB Object, Concurrent Parts Load Test
The next test compares multipart upload performance of the S3 service vs going through the gateway. In this test, a single object of size 10 GiB is uploaded with part size 64 MiB and varying the number of concurrent part uploads. Once again there was some variability in test results, so the average actually has the gateway performing better in some cases. This is just due to test result variance with no significant gateway overhead.

![10 GiB Object Multipart PUT](https://github.com/versity/versitygw/assets/2184287/e07aa023-8674-4f04-bef0-1f7f473b8d79)

test command:
```
for i in 1 10 20 30 40 50 60 70 80 90 100; do
    echo "$i concurrency:"
    for j in `seq 1 5`; do
        ./s3bench -access $access -secret $secret -bucket $bucket -endpoint $endpoint -objectsize 10737418240 -pathstyle -prefix benchtest$i$j/ -upload -delete -n 1 -concurrency $i | grep run
    done
done
```

memory and CPU profile:<br>
This test used about 142% CPU primarily due to the amount of incoming data and the need to checksum all of the data. During this test the Gateway had a peak memory allocation of about 40 MB (28 MB over baseline).

100 concurrent parts 10 GiB multipart upload (to S3):
```
	User time (seconds): 7.09
	System time (seconds): 3.41
	Percent of CPU this job got: 142%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:07.37
	Maximum resident set size (kbytes): 36960
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 2965
	Voluntary context switches: 20912
	Involuntary context switches: 275
	Page size (bytes): 4096
```

## Request Rate Load Test
The final S3 service to gateway comparison is request rate. In this test, the benchmark is issuing as many get object headers requests as it can in 10 seconds. The horizontal axis is the number of simultaneous client routines issuing requests. The vertical axis is the aggregate requests per sec for that test. In this test the gateway does show significant overhead. The reason for this is because the gateway is retrieving the bucket ACLs from the server for each incoming request. So each incoming request to the gateway is actually turning into two requests to the backend service. So the gateway is getting effectively half the request rate compared to direct to the S3 server.

![Requests per second](https://github.com/versity/versitygw/assets/2184287/c0f3dd30-d25e-4c1e-b3de-45abc727063f)

test command:
```
for i in 1 5 10 15 20; do
    echo "$i concurrency:"
    ./s3bench -access $access -secret $secret -bucket $bucket -endpoint $endpoint -pathstyle -prefix test/ -query -concurrency $i -sec 10
done
```

# Frontend Capability Load Tests
The S3 direct vs Gateway Proxy test were a good comparison of versitygw and direct to a backend storage system, but the performance in the tests was always limited by the capability of the backend S3 system. This shows the overhead when going through the gateway but does not reflect the peak performance capability of the Gateway. The next set of tests will use the [noop backend](https://github.com/versity/versitygw/tree/ben/noop) to demonstrate the real performance capability of the Gateway. This backend acts like `/dev/null` in linux where all of the data is read on PUT, but not stored anywhere. In these tests the benchmark utility and the Gateway are both running on the same host, so the host CPUs will be split among these. The same client test commands were used as with the S3 service tests.

## Multiple 1 MB Object Load Test
The first test is the 1 MiB object test with simultaneous PUTs. In this test, a single client stream is seeing about 140 MiB/s, and this quickly scales and plateaus at peak gateway performance of about 1400 MiB/s. The throughput here is pretty dependent on object size and CPU performance (mostly for SHA256 checksum).

![1 MiB Object PUT-noop](https://github.com/versity/versitygw/assets/2184287/f6f923d4-17a2-490a-a15d-aa5b96bcb888)

## 10 GiB Object, Concurrent Parts Load Test
The next test is the single 10 GiB object upload with 64 MiB part size and varying the number of concurrent uploads. The single stream starts at about 490 MiB/s and quickly scales and plateaus at over 6100 MiB/s throughput.  This is likely hitting the host platform peak capability.

![10 GiB Object Multipart PUT-noop](https://github.com/versity/versitygw/assets/2184287/03954fac-eff8-432c-b83e-54b4a5cf2d1c)

memory and CPU profile:<br>
The memory and CPU profile for this test was very similar to going to S3 service. The gateway used about 140% CPU and allocated almost 41 MB of memory (29 MB above baseline). This is expected because the Gateway frontend request handlers are doing the same amount of work unrelated to what backend is in use.

100 concurrent parts 10 GiB multipart upload:
```
	User time (seconds): 6.60
	System time (seconds): 1.52
	Percent of CPU this job got: 139%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:05.84
	Maximum resident set size (kbytes): 40868
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 3763
	Voluntary context switches: 8176
	Involuntary context switches: 36
	Page size (bytes): 4096
```

## Request Rate Load Test
The final test is the request rate to see how fast the gateway is capable of responding to many requests. The test is to issue as many get object headers requests in a ten second time period varying the number of concurrent routines issuing the requests. A single routine can see about 3500 requests/s, and this scales to over 10000 requests/s with 20 concurrent client routines. This is also likely hitting the host platform capability.

![Requests per second-noop](https://github.com/versity/versitygw/assets/2184287/13448007-fce7-4dec-b439-93ac68d205fe)

memory and CPU profile:<br>
This test was the most significant CPU use of all the tests. This is due to the large number of incoming requests per second the Gateway is capable of, limited by the CPU capability. In this case it is expected that the Gateway should saturate CPU resources to maintain a high level of request rate performance. The peak memory allocation under load for this test was under 24 MB (12 MB above baseline).
20 concurrent client routines:
```
	User time (seconds): 28.53
	System time (seconds): 18.64
	Percent of CPU this job got: 360%
	Elapsed (wall clock) time (h:mm:ss or m:ss): 0:13.09
	Maximum resident set size (kbytes): 23864
	Major (requiring I/O) page faults: 0
	Minor (reclaiming a frame) page faults: 120661
	Voluntary context switches: 500301
	Involuntary context switches: 944
	Page size (bytes): 4096
```

# Conclusion
The Versity S3 Gateway demonstrates minimal to no significant overhead in most data throughput scenarios. However, the rate of requests is affected by the need to send multiple requests to the backend for every incoming request. To enhance request performance, especially where bucket ACLs are not crucial, their deactivation may be beneficial. The results also show that the Gateway's frontend can effectively scale to meet the demands of various backend storage access requirements.

The memory profiling shows the most significant memory usage by the Versity S3 Gateway occurred during tasks involving the upload of numerous objects simultaneously, peaking at only 80 MB. The tests that demanded the highest CPU usage, notably the noop request rate load test, illustrate the gateway's ability to efficiently utilize multiple CPU cores. This demonstrates  its capacity for handling intensive tasks with effective resource utilization.
