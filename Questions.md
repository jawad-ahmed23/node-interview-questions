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

## 4. What is REPL? What purpose it is used for?

REPL stands for (READ, EVAL, PRINT, LOOP). Node js comes with bundled REPL environment. This allows for the easy creation of CLI (Command Line Interface) applications.

## 5. Consider the following JavaScript code:

```
console.log("first");
setTimeout(() => {
    console.log("second");
}, 0);
console.log("third");
```

## The output will be:
```
first
third
second
```
## Assuming that this is the desired behavior, how else might we write this code?

Way back when, Node.js version 0.10 introduced setImmediate, which is equivalent to setTimeout(fn, 0), but with some slight advantages.

setTimeout(fn, delay) calls the given callback fn after the given delay has ellapsed (in milliseconds). However, the callback is not executed immediately at this time, but added to the function queue so that it is executed as soon as possible, after all the currently executing and currently queued event handlers have completed. Setting the delay to 0 adds the callback to the queue immediately so that it is executed as soon as all currently-queued functions are finished.

setImmediate(fn) achieves the same effect, except that it doesn’t use the queue of functions. Instead, it checks the queue of I/O event handlers. If all I/O events in the current snapshot are processed, it executes the callback. It queues them immediately after the last I/O handler, somewhat like process.nextTick. This is faster than setTimeout(fn, 0).

So, the above code can be written in Node.js as:

```
console.log("first");
setImmediate(() => {
    console.log("second");
});
console.log("third");
```

## 6. What is the preferred method of resolving unhandled exceptions in Node.js?

Unhandled exceptions in Node.js can be caught at the `Process` level by attaching a handler for `uncaughtException` event.

```
process.on('uncaughtException', (err) => {
  console.log(`Caught exception: ${err}`);
});
```

Unhandled exceptions in Node.js can be caught at the Process level by attaching a handler for uncaughtException event.

```
process.on('uncaughtException', (err) => {
  console.log(`Caught exception: ${err}`);
});
```

However, `uncaughtException` is a very crude mechanism for exception handling and may be removed from Node.js in the future. An exception that has bubbled all the way up to the Process level means that your application, and Node.js may be in an undefined state, and the only sensible approach would be to restart everything.

The preferred way is to add another layer between your application and the Node.js process which is called the [domain](https://nodejs.org/api/domain.html).

Domains provide a way to handle multiple different I/O operations as a single group. So, by having your application, or part of it, running in a separate domain, you can safely handle exceptions at the domain level, before they reach the Process level.

However, domains have been pending deprecation for a few years—since Node.js 4. It’s possible a more future-proof approach would be to use [zones](https://github.com/angular/angular/tree/main/packages/zone.js).

## 7. Consider following code snippet:

```
{
    console.time("loop");
    for (var i = 0; i < 1000000; i += 1){
        // Do nothing
    }
    console.timeEnd("loop");
}
```

## The time required to run this code in Google Chrome is considerably more than the time required to run it in Node.js. Explain why this is so, even though both use the v8 JavaScript Engine.

Within a web browser such as Chrome, declaring the variable i outside of any function’s scope makes it global and therefore binds it as a property of the window object. As a result, running this code in a web browser requires repeatedly resolving the property i within the heavily populated window namespace in each iteration of the for loop.

In Node.js, however, declaring any variable outside of any function’s scope binds it only to the module’s own scope (not the window object) which therefore makes it much easier and faster to resolve.

It’s also worth noting that using let instead of var in the for loop declaration can reduce the loop’s run time by over 50%. But such a change assumes you know [the difference between let and var](https://www.toptal.com/javascript/interview-questions#question-527) and whether this will have an effect on the behavior of your specific loop.

## 8. What is “callback hell” and how can it be avoided?

“Callback hell” refers to heavily nested callbacks that have become unweildy or unreadable.

An example of heavily nested code is below:

```
query("SELECT clientId FROM clients WHERE clientName='picanteverde';", function(id){
  query(`SELECT * FROM transactions WHERE clientId=${id}`, function(transactions){
    transactions.each((transac) => {
      query(`UPDATE transactions SET value = ${transac.value*0.1} WHERE id=${transac.id}`, (error) => {
        if(!error){
          console.log("success!!");
        }else{
          console.log("error");
        }
      });
    });
  });
});
```

At one point, the primary method to fix callback hell was modularization. The callbacks are broken out into independent functions which can be called with some parameters. So the first level of improvement might be:

```
const logError = (error) => {
    if(!error){
      console.log("success!!");
    }else{
      console.log("error");
    }
  },
  updateTransaction = (t) => {
    query(`UPDATE transactions SET value = ${t.value*0.1} WHERE id=${t.id}, logError);
  },
  handleTransactions = (transactions) => {
    transactions.each(updateTransaction);
  },
  handleClient = (id) => {
    query(`SELECT * FROM transactions WHERE clientId=${id}`, handleTransactions);
  };

query("SELECT clientId FROM clients WHERE clientName='picanteverde';",handleClient);
```

Even though this code is much easier to read, and we created some functions that we can even reuse later, in some cases it may be appropriate to use a more robust solution in the form of promises. Promises allow additional desirable behavior such as error propagation and chaining. Node.js includes native support for them.

Additionally, a more supercharged solution to callback hell was provided by generators, as these can resolve execution dependency between different callbacks. However, generators are much more advanced and it might be overkill to use them for this purpose. To read more about generators you can start with this [post](https://strongloop.com/strongblog/how-to-generators-node-js-yield-use-cases/).

However, these approaches are pretty dated at this point. The current solution is to use async/await—an approach that leverages Promises and finally makes it easy to flatten the so-called “pyramid of doom” shown above.















