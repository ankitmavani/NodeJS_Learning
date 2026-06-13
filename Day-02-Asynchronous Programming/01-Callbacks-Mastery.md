# Callbacks Mastery Guide (Node.js)

## Table of Contents

1. What is a Callback?
2. Why Callbacks Exist
3. Synchronous Callback
4. Asynchronous Callback
5. Callback Function vs Higher Order Function
6. Node.js Callback Pattern
7. Error-First Callback Pattern
8. Callback Signature (err, result)
9. Callback Conventions
10. Callback Hell
11. Pyramid of Doom
12. Multiple Nested Callbacks
13. Error Propagation Issues
14. Parallel Callbacks
15. Sequential Callbacks
16. Callback Queue
17. Callback Best Practices
18. Real-World Examples
19. Interview Questions
20. Mastery Checklist

---

# 1. What is a Callback?

A callback is a function passed as an argument to another function and executed later.

Example:

```js
function greet(name, callback) {
  console.log(`Hello ${name}`);
  callback();
}

function done() {
  console.log("Finished");
}

greet("Ankit", done);
```

Output:

```text
Hello Ankit
Finished
```

Here:

```text
done()
```

is the callback.

---

# Why Callbacks Exist

JavaScript is single-threaded.

Long-running operations such as:

* File Reading
* Database Queries
* HTTP Requests
* DNS Lookups

cannot block execution.

Instead:

```text
Start Operation
      ↓
Continue Execution
      ↓
Callback Executes Later
```

---

# 2. Synchronous Callback

Executes immediately.

Example:

```js
const numbers = [1,2,3];

numbers.forEach(num => {
  console.log(num);
});
```

Output:

```text
1
2
3
```

The callback runs immediately during iteration.

---

Another Example

```js
[1,2,3].map(x => x * 2);
```

Callback:

```js
x => x * 2
```

runs synchronously.

---

# 3. Asynchronous Callback

Runs later.

Example:

```js
setTimeout(() => {
  console.log("Executed");
}, 1000);
```

Output:

```text
Executed
```

after 1 second.

---

Execution Flow

```text
Main Thread
      ↓
setTimeout()
      ↓
Timer Registered
      ↓
Continue Execution
      ↓
Callback Queue
      ↓
Event Loop
      ↓
Callback Executes
```

---

# 4. Callback Function vs Higher Order Function

Interview Favorite.

---

## Callback Function

Passed into another function.

```js
function sayHi() {
  console.log("Hi");
}
```

Used as:

```js
greet(sayHi);
```

---

## Higher Order Function

Accepts a function or returns a function.

```js
function greet(callback) {
  callback();
}
```

Here:

```text
greet()
```

is Higher Order Function.

```text
sayHi()
```

is Callback Function.

---

Example:

```js
function process(callback) {
  callback();
}

function run() {
  console.log("Running");
}

process(run);
```

---

# 5. Node.js Callback Pattern

Node heavily relies on callbacks.

Example:

```js
fs.readFile("file.txt", callback);
```

Internally:

```text
Read File
     ↓
Finish
     ↓
Invoke Callback
```

---

Example:

```js
const fs = require("fs");

fs.readFile("file.txt", (err, data) => {
  console.log(data.toString());
});
```

---

# 6. Error-First Callback Pattern

Node standard convention.

Format:

```js
function callback(err, result) {}
```

---

Success:

```js
callback(null, data);
```

Failure:

```js
callback(error, null);
```

---

Example:

```js
fs.readFile("file.txt", (err, data) => {

  if (err) {
    console.error(err);
    return;
  }

  console.log(data);

});
```

---

Why Important?

Consistent API design across Node ecosystem.

Used by:

```text
fs
dns
crypto
zlib
many npm packages
```

---

# 7. Callback Signature (err, result)

Standard Signature:

```js
(err, result)
```

Example:

```js
function getUser(id, callback) {

  const user = {
    id,
    name: "John"
  };

  callback(null, user);

}
```

Usage:

```js
getUser(1, (err, user) => {

  if (err) return;

  console.log(user);

});
```

---

# 8. Callback Conventions

Rule 1

Error first.

```js
(err, data)
```

---

Rule 2

Error OR Result.

Never both.

Good:

```js
callback(null, data);
```

or

```js
callback(error);
```

Bad:

```js
callback(error, data);
```

---

Rule 3

Call callback once.

Bad:

```js
callback(null);
callback(null);
```

Creates bugs.

---

# 9. Callback Hell

Most famous callback problem.

Example:

```js
getUser(id, user => {

  getOrders(user.id, orders => {

    getPayments(orders, payments => {

      getInvoice(payments, invoice => {

        console.log(invoice);

      });

    });

  });

});
```

Shape:

```text
└──
   └──
      └──
         └──
```

Known as:

```text
Callback Hell
```

---

Problems

* Hard to Read
* Hard to Debug
* Hard to Maintain
* Difficult Error Handling

---

# 10. Pyramid of Doom

Visual structure created by callback hell.

Example:

```js
a(() => {
  b(() => {
    c(() => {
      d(() => {
        e(() => {

        });
      });
    });
  });
});
```

Looks like:

```text
       a
      /
     b
    /
   c
  /
 d
/
e
```

---

# 11. Multiple Nested Callbacks

Real Example:

```js
login(user, () => {

  fetchProfile(() => {

    fetchOrders(() => {

      fetchPayments(() => {

      });

    });

  });

});
```

Each level increases complexity.

---

# 12. Error Propagation Issues

Example:

```js
step1((err, data) => {

  if(err)
    return handle(err);

  step2((err, result) => {

    if(err)
      return handle(err);

    step3((err, output) => {

      if(err)
        return handle(err);

    });

  });

});
```

Error checks repeated everywhere.

---

Problems:

```text
Duplicate Logic
Nested Code
Missed Errors
```

---

# 13. Parallel Callbacks

Execute multiple operations simultaneously.

Example:

```js
let completed = 0;

function done() {

  completed++;

  if(completed === 3) {
     console.log("Finished");
  }

}

task1(done);
task2(done);
task3(done);
```

Flow:

```text
Task1 ─┐
Task2 ─┼──► Complete
Task3 ─┘
```

---

Use Case:

```text
Multiple API Calls
Database Queries
File Reads
```

---

# 14. Sequential Callbacks

Operations execute one after another.

Example:

```js
task1(() => {

  task2(() => {

    task3(() => {

      console.log("Done");

    });

  });

});
```

Flow:

```text
Task1
  ↓
Task2
  ↓
Task3
```

---

Use Case:

When order matters.

---

# 15. Callback Queue

Very important Event Loop concept.

Example:

```js
setTimeout(() => {
   console.log("Timer");
},0);
```

Flow:

```text
setTimeout
     ↓
Timer Phase
     ↓
Callback Queue
     ↓
Event Loop
     ↓
Call Stack
```

---

Callback Queue contains:

```text
setTimeout
setInterval
I/O Callbacks
```

waiting for execution.

---

# 16. Callback Best Practices

---

## Keep Callbacks Small

Bad:

```js
callback => {

  // 300 lines

}
```

Good:

```js
callback => {

  processData();

}
```

---

## Handle Errors First

```js
if(err){
   return callback(err);
}
```

---

## Avoid Deep Nesting

Bad:

```text
10 nested callbacks
```

Good:

```text
Promises
Async/Await
```

---

## Use Named Functions

Bad:

```js
fs.readFile(file, (err,data)=>{
});
```

Good:

```js
fs.readFile(file, handleFile);
```

---

## Never Call Callback Twice

Bad:

```js
callback();
callback();
```

Can crash logic.

---

# 17. Real-World Examples

## File Reading

```js
fs.readFile(
  "data.txt",
  (err, data) => {

    if(err)
      return console.error(err);

    console.log(data.toString());

  }
);
```

---

## HTTP Request

```js
http.get(url, res => {

  console.log(res.statusCode);

});
```

---

## Database Query

```js
db.query(sql, (err, rows) => {

   if(err)
      return console.error(err);

   console.log(rows);

});
```

---

# 18. Why Promises Replaced Callbacks

Callbacks Problems:

```text
Callback Hell
Error Handling
Inversion of Control
Poor Readability
```

Promises Solve:

```text
Chaining
Centralized Errors
Cleaner Code
```

Example:

```js
getUser()
  .then(getOrders)
  .then(getPayments)
  .catch(console.error);
```

Much cleaner.

---

# Interview Questions

## Basic

1. What is a callback?
2. Why are callbacks used?
3. Synchronous vs Asynchronous callbacks?
4. Callback vs Higher Order Function?

---

## Intermediate

5. Explain Error-First Callback Pattern.
6. What is callback signature?
7. What is callback queue?
8. Explain callback conventions.

---

## Advanced

9. What is callback hell?
10. What is Pyramid of Doom?
11. How do callbacks interact with Event Loop?
12. Why are callbacks difficult to maintain?
13. How would you execute callbacks in parallel?
14. How would you execute callbacks sequentially?
15. Why did Promises replace callbacks?

---

# Mastery Checklist

* What is Callback
* Synchronous Callback
* Asynchronous Callback
* Higher Order Functions
* Node Callback Pattern
* Error First Callbacks
* Callback Signature
* Callback Conventions
* Callback Hell
* Pyramid of Doom
* Nested Callbacks
* Error Propagation
* Parallel Execution
* Sequential Execution
* Callback Queue
* Best Practices
* Real-World Examples
* Event Loop Relationship

After mastering callbacks, the next topic is **Promises**, which were introduced to solve the major problems associated with callback-based asynchronous programming.
