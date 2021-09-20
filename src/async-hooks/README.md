import async_hooks from 'async_hooks';

```
console.log(Object.keys(async_hooks));
[
  'AsyncLocalStorage',
  
  'createHook',
  'executionAsyncId',
  'triggerAsyncId',
  
  'executionAsyncResource',
  'AsyncResource'
]

```


```
// Return the ID of the current execution context.
const eid = async_hooks.executionAsyncId();

// Return the ID of the handle responsible for triggering the callback of the
// current execution scope to call.
const tid = async_hooks.triggerAsyncId();

/** Create a new async hook */
const asyncHook = async_hooks.createHook({
  init: (asyncId, type, triggerAsyncId, resource) => {
    const eid = async_hooks.executionAsyncId();
    //logger here
  },
  before: () => {
    // ANY
  },
  after: () => {
    // ANY
  },
  destroy: (asyncId) => {
    File.appendContentFile(`[DESTROY] timestamp: ${Date.now()} > asyncId: ${asyncId} \n`);
  },
  promiseResolve: () => {
    // ANY
  },
});

```


```
  // Enable async hook.
  asyncHook.enable();
  
  
 
  // Disabled async hook.
  asyncHook.disable();
```


Example
```
  // Enable async hook.
  asyncHook.enable();

  // Create http server and return status 200 with json.
  const server = http.createServer(async (req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'application/json');
    // Call to external endpoint for get content.
    let result;
    try {
      const request = await httpsRequest(CONFIG.urlRequest);
      result = JSON.parse(request);
    } catch (err) {
      result = 'Wrong value';
    }
    res.end(JSON.stringify({
      result,
    }));
    // Disabled async hook.
    asyncHook.disable();
  });

```


The async_hooks module provides an API to track asynchronous resources in Node.js. An async resource is an object with a callback function associated with it.
An asynchronous resource represents an object with an associated callback.
If Workers are used, each thread has an independent async_hooks interface, and each thread will use a new set of async IDs.
