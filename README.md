# cpus
Check current OS cpus

> npm i loadtest -g
> node cpus.js

```javascript
// cpus.js
const cpus = require("os").cpus();
console.log(cpus);
```

Check

```javascript
const cluster = require("cluster");

if (cluster.isMaster) {
  console.log("this is the master cluster", process.pid);
  cluster.fork();
  cluster.fork();
  cluster.fork();
} else {
  console.log("this is the working cluster", process.pid);
}
``` 
 
嘗試看看發500個request進去， 他們會share不同的fork 來處理進來的request

> loadtest -n 500 http://localhost:3000 

```javascript
const http = require("http");
const cluster = require("cluster");
const numCpus = require("os").cpus().length;

if (cluster.isMaster) {
  console.log("this is the master cluster", process.pid);
  for (let i = 0; i < numCpus; i++) {
    cluster.fork();
  }
} else {
  http
    .createServer((req, res) => {
      const message = `Worker: ", ${process.pid}`;
      console.log(message);
      res.end(message);
    })
    .listen(3000);
}
```

```javascript
const http = require("http");
const cluster = require("cluster");
const numCpus = require("os").cpus().length;
// 當kill完 process server 就登出了
if (cluster.isMaster) {
  console.log("this is the master cluster", process.pid);
  for (let i = 0; i < numCpus; i++) {
    cluster.fork();
  }
  cluster.on("exit", (worker) => {
    console.log(`worker process ${process.pid} had died`);
    console.log(`only remaining ${Object.keys(cluster.workers).length}`);
    
    // 當kill後再create fork， 那樣就不會造成kill完process server died了
    console.log('starting new worker');
    cluster.fork()
  });
} else {
  console.log(`started a worker at ${process.pid}`);
  http
    .createServer((req, res) => {
      const message = `Worker process id: ", ${process.pid}`;
      res.end(message);
      if (req.url === "/kill") {
        process.exit();
      } else if (req.url === "/") {
        console.log(`serving from ${process.pid}`);
      }
    })
    .listen(3000);
}
```

