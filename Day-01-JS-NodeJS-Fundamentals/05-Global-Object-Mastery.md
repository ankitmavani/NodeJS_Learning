# Global Object Mastery (Node.js)

## Table of Contents

1. Global Scope
2. What is Global Object?
3. Browser vs Node.js Global Object
4. Node.js Globals
5. global
6. globalThis
7. __dirname
8. __filename
9. process
10. Buffer
11. setTimeout
12. setInterval
13. setImmediate
14. Process Object
15. Environment Variables
16. Process Arguments
17. Process Events
18. Exit Codes
19. Memory Usage
20. Process Information
21. Signal Handling
22. Uncaught Exceptions
23. Interview Questions
24. Mastery Checklist

---

# 1. Global Scope

Global Scope is the outermost scope available in a JavaScript runtime.

Variables declared globally can be accessed from anywhere.

Example:

```js
const appName = "NodeJS";

function show() {
    console.log(appName);
}

show();
```

Output:

```text
NodeJS
```

---

# Browser Global Scope

In browsers:

```js
var name = "John";

console.log(window.name);
```

Output:

```text
John
```

Because:

```text
window === global object
```

---

# Node.js Global Scope

In Node.js:

```js
var name = "John";

console.log(global.name);
```

Output:

```text
undefined
```

Why?

Because every Node.js file is wrapped inside a module.

Internal wrapper:

```js
(function(exports, require, module, __filename, __dirname){

    // your code

});
```

Variables are module-scoped, not global.

---

# 2. What is Global Object?

Global Object is an object that contains globally accessible APIs.

Examples:

```js
setTimeout
setInterval
console
Buffer
process
```

Accessible without importing.

Example:

```js
console.log("Hello");
```

Actually:

```js
global.console.log("Hello");
```

---

# 3. Browser vs Node.js Global Object

| Browser      | Node.js       |           |
| ------------ | ------------- | --------- |
| window       | global        |           |
| document     | Not Available |           |
| localStorage | Not Available |           |
| process      | Not Available |           |
| Buffer       | Not Available |           |
| setTimeout   | Available     | Available |

---

Browser:

```js
window.alert("Hello");
```

Node:

```js
global.setTimeout(() => {
    console.log("Hi");
},1000);
```

---

# 4. Node.js Globals

Available automatically:

```js
global
globalThis
__dirname
__filename
process
Buffer
setTimeout
setInterval
setImmediate
console
```

No import required.

---

# 5. global

Node.js global object.

Example:

```js
global.appName = "MyApp";

console.log(global.appName);
```

Output:

```text
MyApp
```

---

## Global Variables

Bad Practice:

```js
global.dbConnection = db;
```

Why?

```text
Hard to Debug
Hard to Test
Memory Leaks
```

Avoid unless necessary.

---

# Inspect Global Object

```js
console.log(global);
```

Contains:

```text
process
Buffer
console
timers
etc
```

---

# 6. globalThis

Introduced as a universal global object.

Works everywhere:

```js
globalThis
```

Browser:

```js
globalThis === window
```

Node:

```js
globalThis === global
```

Example:

```js
globalThis.name = "Ankit";

console.log(globalThis.name);
```

Output:

```text
Ankit
```

---

# Why globalThis Exists?

Before:

```js
window
global
self
```

Different environments.

Now:

```js
globalThis
```

Works everywhere.

---

# 7. __dirname

Returns current directory path.

Example:

```js
console.log(__dirname);
```

Output:

```text
/home/project/src
```

Useful for:

```text
File Paths
Config Files
Static Assets
```

---

Example:

```js
const path = require("path");

console.log(
  path.join(__dirname, "data.json")
);
```

---

# 8. __filename

Returns current file path.

Example:

```js
console.log(__filename);
```

Output:

```text
/home/project/src/index.js
```

Difference:

```text
__dirname  → Folder

__filename → Full File
```

---

# ESM Limitation

Not available:

```js
import fs from "fs";
```

These fail:

```js
__dirname
__filename
```

Alternative:

```js
import { fileURLToPath } from "url";
```

---

# 9. process

One of the most important globals.

Represents:

```text
Current Node.js Process
```

Example:

```js
console.log(process);
```

Contains:

```text
PID
Arguments
Memory
Environment
Platform
Events
```

---

# Process Information

```js
console.log(process.pid);
```

Output:

```text
12345
```

---

Platform:

```js
console.log(process.platform);
```

Output:

```text
linux
```

---

Node Version:

```js
console.log(process.version);
```

Output:

```text
v24.x.x
```

---

# 10. Buffer

Node-specific binary data structure.

Example:

```js
const buf = Buffer.from("Hello");

console.log(buf);
```

Output:

```text
<Buffer 48 65 6c 6c 6f>
```

---

Why Buffer?

JavaScript originally lacked binary support.

Used for:

```text
Files
Streams
Sockets
Images
Videos
```

---

# Buffer Example

```js
const buf = Buffer.alloc(10);

console.log(buf);
```

Creates:

```text
10 bytes
```

---

# 11. setTimeout

Runs once after delay.

Example:

```js
setTimeout(() => {
    console.log("Hello");
}, 1000);
```

Output:

```text
Hello
```

after 1 second.

---

# Internal Flow

```text
setTimeout
    │
    ▼
Timer Queue
    │
    ▼
Event Loop
    │
    ▼
Call Stack
```

---

# 12. setInterval

Runs repeatedly.

Example:

```js
setInterval(() => {
    console.log("Running");
},1000);
```

Output:

```text
Running
Running
Running
```

---

Stop:

```js
const id = setInterval(...);

clearInterval(id);
```

---

# 13. setImmediate

Executes during:

```text
Check Phase
```

Example:

```js
setImmediate(() => {
    console.log("Immediate");
});
```

---

Comparison:

```js
setTimeout(fn,0);
setImmediate(fn);
```

Execution order depends on context.

---

# 14. Process Object

Contains everything about current process.

Important APIs:

```js
process.cwd()
process.exit()
process.env
process.argv
process.memoryUsage()
process.uptime()
```

---

# Current Working Directory

```js
console.log(process.cwd());
```

Output:

```text
/project
```

Different from:

```text
__dirname
```

---

# Exit Process

```js
process.exit(0);
```

Terminates process.

---

# 15. Environment Variables

Access:

```js
process.env
```

Example:

```js
console.log(process.env.PORT);
```

---

Using dotenv:

```js
require("dotenv").config();

console.log(process.env.DB_URL);
```

---

Why Use Environment Variables?

Store:

```text
Secrets
API Keys
Passwords
Configuration
```

Outside source code.

---

# 16. Process Arguments

Command:

```bash
node app.js hello world
```

Code:

```js
console.log(process.argv);
```

Output:

```text
[
 node,
 app.js,
 hello,
 world
]
```

---

Example:

```js
const name = process.argv[2];

console.log(name);
```

Run:

```bash
node app.js Ankit
```

Output:

```text
Ankit
```

---

# 17. Process Events

Node process emits events.

Examples:

```js
exit
beforeExit
uncaughtException
SIGINT
SIGTERM
```

---

Exit Event:

```js
process.on("exit", code => {
    console.log(code);
});
```

---

# 18. Exit Codes

Success:

```js
process.exit(0);
```

Failure:

```js
process.exit(1);
```

Convention:

```text
0 → Success

1+ → Error
```

---

# 19. Memory Usage

Important for production monitoring.

```js
console.log(process.memoryUsage());
```

Output:

```js
{
 rss,
 heapTotal,
 heapUsed,
 external
}
```

---

Meaning:

### rss

Total memory allocated.

### heapUsed

Memory actively used.

### heapTotal

Heap size allocated.

---

# Monitoring

```js
setInterval(() => {
    console.log(
      process.memoryUsage()
    );
},5000);
```

Useful for detecting:

```text
Memory Leaks
```

---

# 20. Process Information

Uptime:

```js
console.log(process.uptime());
```

Output:

```text
123.45
```

seconds.

---

CPU Usage:

```js
console.log(process.cpuUsage());
```

---

Resource Usage:

```js
console.log(process.resourceUsage());
```

---

# 21. Signal Handling

OS sends signals.

Examples:

```text
SIGINT
SIGTERM
SIGHUP
```

---

Handle Ctrl+C

```js
process.on("SIGINT", () => {

    console.log("Stopping");

    process.exit(0);

});
```

---

Production Shutdown

```js
process.on("SIGTERM", async () => {

   await db.close();

   process.exit(0);

});
```

Very common in Docker/Kubernetes.

---

# 22. Uncaught Exceptions

Example:

```js
throw new Error("Crash");
```

Process crashes.

---

Handle:

```js
process.on(
  "uncaughtException",
  err => {

    console.error(err);

  }
);
```

---

Promise Rejection

```js
process.on(
  "unhandledRejection",
  err => {

    console.error(err);

  }
);
```

---

Production Recommendation

Log error:

```js
logger.error(err);
```

Then:

```js
process.exit(1);
```

because application state may be corrupted.

---

# Interview Questions

## Basic

1. What is the global object in Node.js?
2. Difference between global and globalThis?
3. What is __dirname?
4. What is __filename?
5. What is process?

## Intermediate

6. Difference between process.cwd() and __dirname?
7. What is Buffer?
8. Explain process.argv.
9. Explain process.env.
10. Explain process.exit().

## Advanced

11. How does Node implement global variables?
12. Why are variables not attached to global?
13. How does setImmediate differ from setTimeout?
14. Explain process memoryUsage().
15. Explain signal handling.
16. Explain uncaughtException.
17. How would you gracefully shut down a Node.js server?

---

# Mastery Checklist

* Global Scope
* Global Object
* Browser vs Node.js Globals
* global
* globalThis
* __dirname
* __filename
* process
* Buffer
* setTimeout
* setInterval
* setImmediate
* Environment Variables
* Process Arguments
* Process Events
* Exit Codes
* Memory Usage
* Process Information
* Signal Handling
* Uncaught Exceptions
* Unhandled Rejections
* Graceful Shutdown

Mastering these topics gives you a strong understanding of the Node.js runtime environment, process lifecycle, memory management, and production-grade server behavior.
