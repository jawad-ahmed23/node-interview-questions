# Node Interview Questions by Toptal

## 1. How does Node.js handle child threads?
Node.js, in its essence, is a single thread process. It does not expose child threads and thread management methods to the developer. Technically, Node.js does spawn child threads for certain tasks such as asynchronous I/O, but these run behind the scenes and do not execute any application JavaScript code, nor block the main event loop. If threading support is desired in a Node.js application, there are tools available to enable it, such as the [ChildProcess](https://nodejs.org/api/child_process.html) module. In fact, [Node.js 12 has experimental support for threads.](https://nodejs.org/docs/latest-v12.x/api/worker_threads.html)
