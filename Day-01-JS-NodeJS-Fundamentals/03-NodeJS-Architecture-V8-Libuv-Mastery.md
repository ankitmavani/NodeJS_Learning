# Node.js Architecture (V8 + Libuv) Mastery Guide

## Table of Contents

1. Node.js Internals
2. Node.js Runtime Architecture
3. Components of Node.js
4. Request Lifecycle
5. V8 Engine
6. JavaScript Compilation Process
7. JIT Compilation
8. TurboFan Optimizer
9. Hidden Classes
10. Garbage Collection
11. Memory Management
12. Libuv
13. Event Loop Implementation
14. Thread Pool
15. Async I/O Operations
16. Default Thread Pool Size
17. UV_THREADPOOL_SIZE
18. File System Operations
19. DNS Operations
20. Crypto Operations
21. Worker Threads
22. Cluster Module
23. Child Processes
24. Performance Bottlenecks
25. Interview Questions
26. Mastery Checklist

---

# 1. Node.js Internals

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine.

Node.js consists of:

* V8 Engine
* Libuv
* Node Core APIs
* Event Loop
* Thread Pool
* Operating System APIs

High-Level Architecture:

```text
Application Code
        │
        ▼
 Node.js APIs
        │
 ┌──────┴──────┐
 ▼             ▼
V8          Libuv
 │             │
 ▼             ▼
JS Engine   Event Loop
            Thread Pool
            Async I/O
                 │
                 ▼
            Operating System
```

---

# 2. Node.js Runtime Architecture

The runtime architecture consists of multiple layers:

```text
User Code
   │
   ▼
Node Core APIs
   │
   ▼
V8 + Libuv
   │
   ▼
Operating System
```

Responsibilities:

### V8

* Parse JavaScript
* Compile JavaScript
* Execute JavaScript
* Garbage Collection

### Libuv

* Event Loop
* Thread Pool
* Networking
* File System
* Timers

---

# 3. Components of Node.js

## V8 Engine

Responsible for executing JavaScript.

## Libuv

Responsible for asynchronous operations.

## Node Core APIs

Examples:

```js
fs
http
https
stream
buffer
crypto
events
path
```

## Operating System

Provides:

* TCP/IP
* File System
* DNS
* Process Scheduling

---

# 4. Request Lifecycle

Example:

```js
app.get("/users", async (req,res)=>{
   const users = await db.users.findMany();
   res.json(users);
});
```

Flow:

```text
Client Request
      │
      ▼
Node HTTP Server
      │
      ▼
Route Handler
      │
      ▼
Database Query
      │
      ▼
Libuv
      │
      ▼
Operating System
      │
      ▼
Database
      │
      ▼
Callback Returned
      │
      ▼
Event Loop
      │
      ▼
Response Sent
```

---

# 5. V8 Engine

## What is V8?

V8 is Google's open-source JavaScript engine.

Responsibilities:

* Parsing
* Compilation
* Execution
* Memory Management
* Garbage Collection

Used by:

* Chrome Browser
* Node.js

---

# 6. JavaScript Compilation Process

Pipeline:

```text
JavaScript Source
       │
       ▼
Lexer
       │
       ▼
Tokens
       │
       ▼
Parser
       │
       ▼
AST
       │
       ▼
Ignition
       │
       ▼
Bytecode
       │
       ▼
TurboFan
       │
       ▼
Machine Code
```

---

# 7. JIT Compilation

JIT = Just-In-Time Compilation

Traditional:

```text
Source
   │
   ▼
Machine Code
```

V8:

```text
Source
   │
   ▼
Bytecode
   │
   ▼
Optimized Machine Code
```

Benefits:

* Faster startup
* Better runtime optimization

---

# 8. TurboFan Optimizer

TurboFan optimizes frequently executed code.

Example:

```js
function add(a,b){
   return a+b;
}

for(let i=0;i<1000000;i++){
   add(1,2);
}
```

TurboFan detects:

```text
Hot Function
```

And generates highly optimized machine code.

---

# 9. Hidden Classes

Important V8 optimization.

Bad:

```js
const user = {};
user.name = "John";
user.age = 20;
```

Good:

```js
const user = {
  name:"John",
  age:20
};
```

Why?

Objects with same structure share Hidden Classes.

Example:

```js
user1
user2
user3
```

Shape:

```text
name
age
```

Shared optimization improves performance.

---

# 10. Garbage Collection

Automatic memory cleanup.

Example:

```js
let user = {
   name:"John"
};

user = null;
```

Object becomes unreachable.

Garbage Collector removes it.

---

## Mark and Sweep Algorithm

### Step 1

Mark reachable objects.

### Step 2

Remove unreachable objects.

```text
Reachable → Keep

Unreachable → Delete
```

---

## Generational Garbage Collection

Memory divided into:

```text
Young Generation

Old Generation
```

Most objects die young.

Optimization:

```text
Clean young generation frequently.
```

---

# 11. Memory Management

Two memory areas:

## Stack Memory

Stores:

* Execution Contexts
* Function Calls

Example:

```js
function add(){}
```

---

## Heap Memory

Stores:

* Objects
* Arrays
* Functions

Example:

```js
const user = {
  name:"John"
};
```

---

# 12. Libuv

## What is Libuv?

Libuv is a C library that powers Node.js asynchronous behavior.

Responsibilities:

* Event Loop
* Thread Pool
* File System
* Networking
* Timers

Without Libuv:

```text
No asynchronous Node.js
```

---

# 13. Event Loop Implementation

Libuv implements:

```text
Timers
Pending Callbacks
Idle/Prepare
Poll
Check
Close
```

The Event Loop continuously cycles through these phases.

---

# 14. Thread Pool

Libuv maintains worker threads.

Purpose:

```text
Execute blocking tasks
outside JavaScript thread
```

Used by:

* File System
* DNS
* Crypto
* Compression

---

# 15. Async I/O Operations

Example:

```js
fs.readFile("test.txt");
```

Flow:

```text
JavaScript Thread
        │
        ▼
Libuv
        │
        ▼
Thread Pool
        │
        ▼
Operating System
        │
        ▼
Callback Queue
```

---

# 16. Default Thread Pool Size

Default:

```text
4 Threads
```

Check:

```js
console.log(process.env.UV_THREADPOOL_SIZE);
```

---

# 17. UV_THREADPOOL_SIZE

Increase pool size:

```bash
UV_THREADPOOL_SIZE=16 node app.js
```

Maximum:

```text
128
```

Use carefully.

More threads ≠ always faster.

---

# 18. File System Operations

Examples:

```js
fs.readFile()
fs.writeFile()
fs.appendFile()
```

Flow:

```text
JS Thread
   │
   ▼
Libuv Thread Pool
   │
   ▼
Operating System
```

---

# 19. DNS Operations

Example:

```js
dns.lookup()
```

Uses Libuv Thread Pool.

Reason:

```text
DNS lookup can block.
```

---

# 20. Crypto Operations

Examples:

```js
crypto.pbkdf2()
bcrypt.hash()
```

These operations use the Thread Pool.

Heavy usage can saturate available threads.

---

# 21. Worker Threads

Designed for CPU-intensive work.

Example:

```js
const { Worker } = require("worker_threads");
```

Each worker has:

```text
Own V8 Engine
Own Event Loop
Own Memory
```

Use Cases:

* Image Processing
* AI Inference
* Data Analytics
* Video Encoding

---

# 22. Cluster Module

Purpose:

```text
Utilize Multiple CPU Cores
```

Example:

```js
const cluster = require("cluster");
```

8-core machine:

```text
Worker 1
Worker 2
Worker 3
Worker 4
Worker 5
Worker 6
Worker 7
Worker 8
```

Load distributed across workers.

---

# 23. Child Processes

Create separate OS processes.

Methods:

```js
fork()
spawn()
exec()
execFile()
```

Use Cases:

* Python Scripts
* FFmpeg
* Shell Commands
* ImageMagick

---

## Worker Threads vs Cluster vs Child Process

| Feature             | Worker Thread | Cluster | Child Process |
| ------------------- | ------------- | ------- | ------------- |
| CPU Intensive Tasks | Yes           | No      | Yes           |
| Multi-Core Usage    | Yes           | Yes     | Yes           |
| Separate Memory     | Yes           | Yes     | Yes           |
| Fast Communication  | Yes           | Medium  | Medium        |
| HTTP Scaling        | No            | Yes     | No            |

---

# 24. Performance Bottlenecks

## Event Loop Blocking

Bad:

```js
while(true){}
```

Server freezes.

---

## CPU Intensive Work

Bad:

```js
for(let i=0;i<1e10;i++){}
```

Solution:

```text
Worker Threads
```

---

## Thread Pool Saturation

Example:

```js
1000 bcrypt.hash()
```

Default pool:

```text
4 threads
```

996 tasks wait.

---

## Memory Leaks

Bad:

```js
global.cache.push(data);
```

Memory continuously grows.

---

## Excessive Garbage Collection

Symptoms:

* High CPU
* Slow Requests
* Long GC Pauses

---

# 25. Interview Questions

## Basic

1. What is Node.js Architecture?
2. What is V8?
3. What is Libuv?
4. What is the Event Loop?
5. What is the Thread Pool?

## Intermediate

6. Explain the JavaScript compilation pipeline.
7. What are Hidden Classes?
8. Explain Garbage Collection.
9. Difference between Stack and Heap?
10. Explain UV_THREADPOOL_SIZE.

## Advanced

11. How does V8 optimize JavaScript?
12. Explain TurboFan.
13. Explain Thread Pool Saturation.
14. Worker Threads vs Cluster?
15. Child Process vs Worker Thread?
16. How would you scale Node.js on an 8-core machine?
17. Explain memory leaks in production.

---

# 26. Mastery Checklist

* Node.js Runtime Architecture
* V8 Internals
* AST
* Ignition Interpreter
* TurboFan Optimizer
* Hidden Classes
* Garbage Collection
* Heap vs Stack
* Libuv Internals
* Event Loop Implementation
* Thread Pool
* Async I/O
* File System Operations
* DNS Operations
* Crypto Operations
* UV_THREADPOOL_SIZE
* Worker Threads
* Cluster Module
* Child Processes
* Performance Bottlenecks
* Production Optimization

Mastering all these topics gives you a deep understanding of how Node.js executes JavaScript, performs asynchronous operations, scales across CPUs, and handles high-concurrency workloads in production.
