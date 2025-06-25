# Node.js Cluster Module

The Node.js `cluster` module allows you to create child processes (workers) that all share server ports, enabling you to take advantage of multi-core systems. This is especially useful for web servers and applications that need to handle many concurrent connections efficiently.

---

## Why Use the Cluster Module?

A single instance of Node.js runs in a single thread. To utilize multi-core CPUs, you can launch a cluster of Node.js processes to handle the load. Each worker process runs independently and can share server ports, allowing for better load distribution and reliability.

---

## Basic Usage Example

Here is a basic example that creates a cluster of HTTP server workers:

```js
const cluster = require('node:cluster');
const http = require('node:http');
const numCPUs = require('node:os').availableParallelism();
const process = require('node:process');

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

When you run this script, Node.js will share port 8000 between all workers.

---

## How It Works

- Worker processes are spawned using `child_process.fork()` and communicate with the primary process via IPC.
- The primary process manages incoming connections and distributes them to workers (round-robin by default, except on Windows).
- Workers can be killed or respawned as needed, without affecting other workers.
- There is no built-in session sharing; use external storage for sessions if needed.

---

## Key Properties and Methods

### cluster.isPrimary / cluster.isWorker
- `cluster.isPrimary`: `true` if the process is the primary (main) process.
- `cluster.isWorker`: `true` if the process is a worker.

### cluster.fork([env])
- Spawns a new worker process.
- Optionally pass environment variables.
- Returns a `Worker` object.

### cluster.workers
- An object containing all active worker objects, indexed by their IDs.

### cluster.disconnect([callback])
- Gracefully disconnects all workers.
- Optional callback is called when all workers are disconnected.

### cluster.setupPrimary([settings])
- Configures cluster settings (e.g., script, args, stdio) before forking workers.

---

## Worker Class

A `Worker` object contains information and methods about a worker process. In the primary process, you can access all workers via `cluster.workers`. In a worker, you can access the current worker via `cluster.worker`.

### Key Properties
- `worker.id`: Unique ID for the worker.
- `worker.process`: The underlying `ChildProcess` object.
- `worker.exitedAfterDisconnect`: `true` if the worker exited after `.disconnect()`.

### Key Methods
- `worker.send(message[, sendHandle[, options]][, callback])`: Send a message to the worker (or to the primary from a worker).
- `worker.kill([signal])`: Kill the worker process.
- `worker.disconnect()`: Gracefully disconnect the worker.
- `worker.isConnected()`: Returns `true` if the worker is connected to the primary.
- `worker.isDead()`: Returns `true` if the worker's process has terminated.

---

## Cluster Events

### cluster.on('fork', callback)
Emitted when a new worker is forked.

```js
cluster.on('fork', (worker) => {
  console.log(`Worker ${worker.id} forked`);
});
```

### cluster.on('online', callback)
Emitted when a worker is online and ready.

```js
cluster.on('online', (worker) => {
  console.log(`Worker ${worker.id} is online`);
});
```

### cluster.on('listening', callback)
Emitted when a worker starts listening on a server.

```js
cluster.on('listening', (worker, address) => {
  console.log(`Worker ${worker.id} is listening on ${address.address}:${address.port}`);
});
```

### cluster.on('disconnect', callback)
Emitted after a worker IPC channel has disconnected.

```js
cluster.on('disconnect', (worker) => {
  console.log(`Worker ${worker.id} has disconnected`);
});
```

### cluster.on('exit', callback)
Emitted when a worker dies. You can use this to respawn workers if needed.

```js
cluster.on('exit', (worker, code, signal) => {
  console.log(`Worker ${worker.process.pid} died (${signal || code}). Restarting...`);
  cluster.fork();
});
```

### cluster.on('message', callback)
Emitted when the primary receives a message from any worker.

```js
cluster.on('message', (worker, message, handle) => {
  console.log(`Message from worker ${worker.id}:`, message);
});
```

---

## Worker Events

You can also listen to events on individual worker objects:

- `'online'`: Worker is online.
- `'listening'`: Worker is listening on a server.
- `'disconnect'`: Worker has disconnected.
- `'exit'`: Worker has exited.
- `'message'`: Message received from the primary/worker.
- `'error'`: Error occurred in the worker.

Example:

```js
const worker = cluster.fork();
worker.on('exit', (code, signal) => {
  if (signal) {
    console.log(`Worker was killed by signal: ${signal}`);
  } else if (code !== 0) {
    console.log(`Worker exited with error code: ${code}`);
  } else {
    console.log('Worker success!');
  }
});
```

---

## Example: Counting Requests Across Workers

This example shows how to keep a count of HTTP requests in the primary process using messages from workers:

```js
const cluster = require('node:cluster');
const http = require('node:http');
const numCPUs = require('node:os').availableParallelism();
const process = require('node:process');

if (cluster.isPrimary) {
  let numReqs = 0;
  setInterval(() => {
    console.log(`numReqs = ${numReqs}`);
  }, 1000);

  function messageHandler(msg) {
    if (msg.cmd && msg.cmd === 'notifyRequest') {
      numReqs += 1;
    }
  }

  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  for (const id in cluster.workers) {
    cluster.workers[id].on('message', messageHandler);
  }

} else {
  http.Server((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
    process.send({ cmd: 'notifyRequest' });
  }).listen(8000);
}
```

---

## Graceful Shutdown Example

You can gracefully disconnect a worker and force kill it if it doesn't exit in time:

```js
if (cluster.isPrimary) {
  const worker = cluster.fork();
  let timeout;

  worker.on('listening', (address) => {
    worker.send('shutdown');
    worker.disconnect();
    timeout = setTimeout(() => {
      worker.kill();
    }, 2000);
  });

  worker.on('disconnect', () => {
    clearTimeout(timeout);
  });

} else if (cluster.isWorker) {
  const net = require('node:net');
  const server = net.createServer((socket) => {
    // Connections never end
  });

  server.listen(8000);

  process.on('message', (msg) => {
    if (msg === 'shutdown') {
      // Initiate graceful close of any connections to server
    }
  });
}
```

---

## cluster.schedulingPolicy

- `cluster.SCHED_RR`: Round-robin (default except on Windows)
- `cluster.SCHED_NONE`: OS handles scheduling

You can set the policy before forking workers:

```js
cluster.schedulingPolicy = cluster.SCHED_RR;
```

---

## cluster.settings and setupPrimary

You can customize how workers are spawned:

```js
cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'https'],
  silent: true
});
cluster.fork(); // https worker
cluster.setupPrimary({
  exec: 'worker.js',
  args: ['--use', 'http']
});
cluster.fork(); // http worker
```

---

## Notes
- On Windows, named pipe servers cannot be set up in a worker.
- Node.js does not provide built-in session sharing between workers.
- Use external storage (like Redis) for session data if needed.

---

## References
- [Node.js Cluster Module Documentation](https://nodejs.org/api/cluster.html)
