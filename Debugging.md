# Debug
Enabling debug mode:
```
   --debug                                 enable debug output (default: false) [$VGW_DEBUG]
```
Will enable the following:
* logging of incoming request headers
* DEBUG Request Signature generation
  * CANONICAL STRING
  * STRING TO SIGN
* logging of incoming request path parameters
* logging of response headers

This is very verbose for each request, so is intended to be run while attempting to triage an issue. This should not be enabled by default for production use.

# Profiling
Profiling can be enabled while running in production, but it is recommended to only listen on localhost or private networks and not expose this endpoint to external networks.

Enabling profiling:
```
   --pprof value                           enable pprof debug on specified port [$VGW_PPROF]
```
The pprof option enables the pprof HTTP server for profiling the gateway while the process is running. See the following for more information:<br>
https://pkg.go.dev/net/http/pprof<br>
To enable, set the pprof option to the listening address for the pprof service. For example, to listen on localhost port 6060, set the option to "localhost:6060".

Assuming the prrof service is running on localhost:6060, here are some useful client commands for profiling.
### Get stack traces for all goroutines:
```
curl 'http://localhost:6060/debug/pprof/goroutine?debug=1'
```
or
```
curl 'http://localhost:6060/debug/pprof/goroutine?debug=2'
```
### Get detailed memory heap stats:
```
curl 'http://localhost:6060/debug/pprof/heap?debug=1'
```
Format for heap allocations is in https://go.dev/src/runtime/pprof/pprof.go
```
    	fmt.Fprintf(w, "heap profile: %d: %d [%d: %d] @ heap/%d\n",
    		total.InUseObjects(), inUseBytes,
    		total.AllocObjects, allocBytes,
    		rate)
    
    	for i := range p {
    		r := &p[i]
    		fmt.Fprintf(w, "%d: %d [%d: %d] @",
    			r.InUseObjects(), r.InUseBytes(),
    			r.AllocObjects, r.AllocBytes)
    		for _, pc := range r.Stack() {
    			fmt.Fprintf(w, " %#x", pc)
    		}
    		fmt.Fprintf(w, "\n")
    		printStackRecord(w, r.Stack(), false)
    	}
```
### Grab and display a 30 second CPU profile on a local webserver on port 9999:
```
go tool pprof -http :9999 'http://localhost:6060/debug/pprof/profile?seconds=30'
```
### Execution traces
```
curl -o trace.out 'http://localhost:6060/debug/pprof/trace?debug=1'
go tool trace trace.out
```
See this link for a good writeup on the execution tracer use: https://sourcegraph.com/blog/go/an-introduction-to-go-tool-trace-rhys-hiltner
### Interactive Browser
Point a browser to http://localhost:6060/debug/pprof to see links for the options:
```
Profile Descriptions:

allocs:       A sampling of all past memory allocations
block:        Stack traces that led to blocking on synchronization primitives
cmdline:      The command line invocation of the current program
goroutine:    Stack traces of all current goroutines.
heap:         A sampling of memory allocations of live objects.
mutex:        Stack traces of holders of contended mutexes
profile:      CPU profile. Use the go tool pprof command to investigate the profile
threadcreate: Stack traces that led to the creation of new OS threads
trace:        A trace of execution of the current program. Use the go tool trace command to investigate the trace.
```