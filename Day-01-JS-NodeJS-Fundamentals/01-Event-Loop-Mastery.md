# Node.js Event Loop Mastery Guide

## Table of Contents

1. What is the Event Loop?
2. Why Node.js is Single Threaded
3. Synchronous vs Asynchronous Execution
4. Blocking vs Non-Blocking Operations
5. Event Loop Phases

   * Timers Phase
   * Pending Callbacks Phase
   * Idle/Prepare Phase
   * Poll Phase
   * Check Phase
   * Close Callbacks Phase
6. Microtasks
7. Promise Queue
8. process.nextTick()
9. Microtask Priority
10. Execution Order
11. Event Loop Visualization
12. Event Loop Starvation
13. CPU Intensive Tasks Impact
14. Event Loop Monitoring
15. Interview Questions

---

# 1. What is the Event Loop?

The Event Loop is the mechanism that enables Node.js to perform non-blocking I/O operations even though JavaScript executes on a single thread.

## High-Level Flow

```text
Call Stack
    ↓
Event Loop
    ↓
Callback Queues
```

Pseudo-code:

```js
while (true) {
  if (callStack.isEmpty()) {
    executeNextCallback();
  }
}
```

Responsibilities:

* Monitor the Call Stack
* Check callback queues
* Move ready callbacks onto the Call Stack

---

# 2. Why Node.js is Single Threaded

JavaScript execution occurs on a single thread.

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

## Then How Can Node Handle Thousands of Requests?

Because work is delegated.

### V8 Engine

Responsible for:

* Parsing JavaScript
* Compiling JavaScript
* Executing JavaScript

### Libuv

Responsible for:

* File System operations
* Timers
* Networking
* DNS
* Thread Pool

```text
JavaScript Thread
       │
       ▼
   Event Loop
       │
       ▼
     Libuv
```

Node.js is:

```text
Single-threaded JavaScript
+
Multi-threaded I/O Handling
```

---

# 3. Synchronous vs Asynchronous Execution

## Synchronous

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

Each operation waits for the previous one.

---

## Asynchronous

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");
```

Output:

```text
A
C
B
```

The timer callback executes later.

---

# 4. Blocking vs Non-Blocking Operations

## Blocking

```js
const data = fs.readFileSync("file.txt");
console.log(data);
```

Execution stops until file reading completes.

---

## Non-Blocking

```js
fs.readFile("file.txt", (err, data) => {
  console.log(data);
});

console.log("Done");
```

Output:

```text
Done
file content
```

Execution continues while I/O is performed in the background.

---

# 5. Event Loop Phases

```text
┌─────────────┐
│ Timers      │
├─────────────┤
│ Pending     │
├─────────────┤
│ IdlePrepare │
├─────────────┤
│ Poll        │
├─────────────┤
│ Check       │
├─────────────┤
│ Close       │
└─────────────┘
```

Node continuously cycles through these phases.

---

# Timers Phase

Handles:

* setTimeout()
* setInterval()

Example:

```js
setTimeout(() => {
  console.log("Timer");
}, 0);
```

Important:

```text
0 ms does not mean immediate execution.
```

It means:

```text
Run after the current iteration.
```

---

# Pending Callbacks Phase

Executes deferred system-level callbacks.

Examples:

* TCP errors
* Network errors
* Some OS-level callbacks

---

# Idle / Prepare Phase

Internal Libuv phase.

Not directly used by developers.

---

# Poll Phase

Most important phase.

Handles:

* File system callbacks
* Database callbacks
* HTTP callbacks
* Socket callbacks

Example:

```js
fs.readFile("test.txt", () => {
  console.log("File Read");
});
```

Poll responsibilities:

1. Execute I/O callbacks
2. Wait for new I/O events
3. Determine next phase

---

# Check Phase

Handles:

```js
setImmediate()
```

Example:

```js
setImmediate(() => {
  console.log("Immediate");
});
```

Executed in the Check phase.

---

# Close Callbacks Phase

Handles:

```js
socket.on("close", () => {
  console.log("Socket Closed");
});
```

Executed during the Close phase.

---

# 6. Microtasks

Microtasks execute immediately after current JavaScript execution finishes.

Higher priority than Event Loop phases.

Types:

1. process.nextTick Queue
2. Promise Queue

---

# 7. Promise Queue

Created by:

```js
Promise.resolve().then(() => {
  console.log("Promise");
});
```

Stored inside the Microtask Queue.

---

# 8. process.nextTick()

Node-specific queue.

Example:

```js
process.nextTick(() => {
  console.log("nextTick");
});
```

Highest priority queue in Node.js.

---

# 9. Microtask Priority

Priority Order:

```text
process.nextTick
        ↓
Promise Queue
        ↓
Event Loop Phases
```

Example:

```js
setTimeout(() => {
  console.log("Timer");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise");
});

process.nextTick(() => {
  console.log("nextTick");
});
```

Output:

```text
nextTick
Promise
Timer
```

---

# 10. Execution Order

Example:

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve().then(() => {
  console.log("3");
});

process.nextTick(() => {
  console.log("4");
});

console.log("5");
```

Execution:

```text
1
5
4
3
2
```

Output:

```text
1
5
4
3
2
```

---

# 11. Event Loop Visualization

```text
Call Stack
    │
    ▼
Execute JavaScript
    │
    ▼
nextTick Queue
    │
    ▼
Promise Queue
    │
    ▼
Timers
    │
    ▼
Pending
    │
    ▼
Poll
    │
    ▼
Check
    │
    ▼
Close
```

The cycle repeats continuously.

---

# 12. Event Loop Starvation

Occurs when the Event Loop never reaches later phases.

Example:

```js
function loop() {
  process.nextTick(loop);
}

loop();
```

Result:

```text
nextTick
nextTick
nextTick
...
```

Timers, Poll, and Check phases never execute.

---

Another Example:

```js
while (true) {}
```

The entire process freezes.

---

# 13. CPU Intensive Tasks Impact

Bad Example:

```js
function heavy() {
  for (let i = 0; i < 10000000000; i++) {}
}

heavy();
```

Effects:

* Event Loop blocked
* No requests processed
* No timers executed
* No database callbacks handled

---

Example:

```js
app.get("/", (req, res) => {
  heavy();
  res.send("Done");
});
```

A single request can freeze the server.

---

## Solutions

### Worker Threads

```js
const { Worker } = require("worker_threads");
```

### Child Processes

```js
fork();
spawn();
```

### Job Queues

* BullMQ
* RabbitMQ
* Kafka

---

# 14. Event Loop Monitoring

## Event Loop Lag

Expected:

```js
setTimeout(fn, 100);
```

Actual:

```text
100ms
```

Blocked:

```text
500ms
```

Lag:

```text
400ms
```

---

## Simple Measurement

```js
const start = Date.now();

setTimeout(() => {
  console.log(Date.now() - start);
}, 100);
```

If output is significantly larger than 100, the Event Loop is busy.

---

## monitorEventLoopDelay

```js
const { monitorEventLoopDelay } = require("perf_hooks");

const histogram = monitorEventLoopDelay();

histogram.enable();

setInterval(() => {
  console.log(histogram.mean);
}, 1000);
```

Used to detect performance bottlenecks in production systems.

---

# 15. Most Important Interview Questions

## Basic

1. What is the Event Loop?
2. Why is Node.js called single-threaded?
3. Difference between synchronous and asynchronous execution?
4. Difference between blocking and non-blocking operations?
5. What are Event Loop phases?

---

## Intermediate

6. Explain the Poll phase.
7. Explain Microtasks.
8. Difference between Promise Queue and process.nextTick Queue?
9. Difference between setTimeout and setImmediate?
10. What is Event Loop starvation?

---

## Advanced

11. Why can process.nextTick starve the Event Loop?
12. How does Libuv work internally?
13. How does Node handle file system operations?
14. How does Node handle DNS operations?
15. How would you monitor Event Loop lag in production?
16. How do Worker Threads solve CPU-intensive problems?
17. Explain a real-world Event Loop bottleneck you have faced.

---

# Senior-Level Mastery Checklist

* Event Loop Fundamentals
* Call Stack
* Callback Queue
* Microtasks
* Promise Queue
* process.nextTick
* Event Loop Phases
* Poll Phase
* Timers
* setImmediate
* Event Loop Starvation
* CPU Blocking
* Worker Threads
* Libuv Thread Pool
* Event Loop Monitoring

Mastering these topics covers the majority of Node.js Event Loop interview questions from Mid-Level to Senior-Level positions.
