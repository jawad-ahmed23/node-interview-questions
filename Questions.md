# Node Interview Questions by Toptal

## 1. How does Node.js handle child threads?
Node.js, in its essence, is a single thread process. It does not expose child threads and thread management methods to the developer. Technically, Node.js does spawn child threads for certain tasks such as asynchronous I/O, but these run behind the scenes and do not execute any application JavaScript code, nor block the main event loop. If threading support is desired in a Node.js application, there are tools available to enable it, such as the [ChildProcess](https://nodejs.org/api/child_process.html) module. In fact, [Node.js 12 has experimental support for threads.](https://nodejs.org/docs/latest-v12.x/api/worker_threads.html)

## 2. How does Node.js support multi-processor platforms, and does it fully utilize all processor resources?

Since Node.js is by default a **single** thread application, it will run on a single processor core and will not take full advantage of multiple core resources. However, Node.js provides support for deployment on multiple-core systems, to take greater advantage of the hardware. The [Cluster](https://nodejs.org/api/cluster.html) module is one of the core Node.js modules and it allows running multiple Node.js worker processes that will share the same port.

## 3. What is typically the first argument passed to a Node.js callback handler?

Node.js core modules, as well as most of the community-published ones, follow a pattern whereby the first argument to any callback handler is an optional error object. If there is no error, the argument will be null or undefined.

A typical callback handler could therefore perform error handling as follows:
```
function callback(err, results) {
    // usually we'll check for the error before handling results
    if(err) {
        // handle error somehow and return
    }
    // no error, perform standard callback handling
}
```
