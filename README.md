# cpus
Check current OS cpus

> npm i loadtest -g
> node cpus.js
> loadtest -n 500 http://localhost:3000 (send 500 request to localhost:3000 server)

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
