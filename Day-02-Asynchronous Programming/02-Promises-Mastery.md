# Promises Mastery Guide (Node.js & JavaScript)

## Table of Contents

1. What is a Promise?
2. Why Promises Exist
3. Promise States

   * Pending
   * Fulfilled
   * Rejected
4. Promise Lifecycle
5. Creating Promises
6. Promise Constructor
7. Resolve
8. Reject
9. Consuming Promises
10. .then()
11. .catch()
12. .finally()
13. Promise Chaining
14. Sequential Execution
15. Returning Values
16. Returning Promises
17. Promise.all()
18. Promise.allSettled()
19. Promise.race()
20. Promise.any()
21. Promise Queue
22. Microtask Queue
23. Promise Resolution Procedure
24. Internal Implementation
25. Real World Examples
26. Interview Questions
27. Mastery Checklist

---

# 1. What is a Promise?

A Promise is an object representing the eventual completion or failure of an asynchronous operation.

Before Promises:

```text
Async Operation
      ↓
Callback
      ↓
Callback Hell
```

After Promises:

```text
Async Operation
      ↓
Promise
      ↓
then()
catch()
finally()
```

---

# Real World Analogy

Food Delivery

```text
Order Placed
     ↓
Promise Created
     ↓
Preparing
     ↓
Delivered OR Failed
```

Promise tracks future results.

---

# 2. Why Promises Exist

Callbacks created problems:

* Callback Hell
* Pyramid of Doom
* Difficult Error Handling
* Inversion of Control

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

Promises solve this.

---

Promise Version

```js
getUser(id)
  .then(getOrders)
  .then(getPayments)
  .then(getInvoice)
  .then(console.log)
  .catch(console.error);
```

Much cleaner.

---

# 3. Promise States

A Promise can only be in one state.

```text
Pending
Fulfilled
Rejected
```

---

# Pending

Initial state.

Example:

```js
const promise = new Promise(() => {});
```

State:

```text
Pending
```

No result yet.

---

# Fulfilled

Operation succeeded.

```js
resolve("Success");
```

State:

```text
Fulfilled
```

Value available.

---

# Rejected

Operation failed.

```js
reject(new Error("Failed"));
```

State:

```text
Rejected
```

Error available.

---

# State Transition

```text
           Pending
           /     \
          /       \
         ▼         ▼
   Fulfilled    Rejected
```

Important:

```text
State changes only once.
```

---

# 4. Promise Lifecycle

Example:

```js
const promise = new Promise((resolve) => {

  setTimeout(() => {
    resolve("Done");
  },1000);

});
```

Lifecycle:

```text
Create Promise
      ↓
Pending
      ↓
Async Task
      ↓
resolve()
      ↓
Fulfilled
```

---

# 5. Creating Promises

Syntax:

```js
const promise = new Promise(
  (resolve, reject) => {

  }
);
```

Executor function runs immediately.

---

Example:

```js
const promise = new Promise(
  (resolve, reject) => {

    resolve("Success");

  }
);
```

---

# 6. Promise Constructor

Signature:

```js
new Promise((resolve, reject) => {

});
```

Parameters:

```text
resolve
reject
```

Purpose:

```text
resolve() → Success

reject() → Failure
```

---

Example:

```js
const promise = new Promise(
  (resolve, reject) => {

    const success = true;

    if(success)
      resolve("OK");
    else
      reject("FAILED");

  }
);
```

---

# 7. Resolve

Marks Promise as fulfilled.

Example:

```js
resolve("Hello");
```

---

Consumer:

```js
promise.then(value => {
  console.log(value);
});
```

Output:

```text
Hello
```

---

# Resolve with Object

```js
resolve({
  id:1,
  name:"John"
});
```

---

# 8. Reject

Marks Promise as rejected.

Example:

```js
reject(
  new Error("Database Error")
);
```

Consumer:

```js
promise.catch(err => {
  console.error(err);
});
```

---

Output:

```text
Error: Database Error
```

---

# 9. Consuming Promises

Methods:

```text
then()
catch()
finally()
```

---

# 10. .then()

Handles success.

Example:

```js
promise.then(result => {
   console.log(result);
});
```

---

Example:

```js
Promise.resolve("Hello")
  .then(data => {

    console.log(data);

  });
```

Output:

```text
Hello
```

---

# Multiple then()

```js
Promise.resolve(5)

  .then(x => x * 2)

  .then(x => x * 3)

  .then(console.log);
```

Output:

```text
30
```

---

# 11. .catch()

Handles errors.

Example:

```js
Promise.reject("Failed")
  .catch(console.error);
```

Output:

```text
Failed
```

---

Equivalent to:

```js
then(null, errorHandler);
```

---

# Error Propagation

```js
Promise.resolve()

  .then(() => {
      throw new Error("Oops");
  })

  .catch(console.error);
```

Output:

```text
Error: Oops
```

---

# 12. .finally()

Runs regardless of success or failure.

Example:

```js
promise
  .then(...)
  .catch(...)
  .finally(() => {

      console.log("Cleanup");

  });
```

---

Use Cases:

```text
Close Database
Close File
Hide Loader
Release Resource
```

---

# 13. Promise Chaining

Most important Promise feature.

Example:

```js
Promise.resolve(5)

  .then(x => x * 2)

  .then(x => x + 10)

  .then(console.log);
```

Output:

```text
20
```

---

Flow:

```text
Promise
   ↓
then
   ↓
then
   ↓
then
```

---

# 14. Sequential Execution

Example:

```js
getUser()

 .then(user => {

    return getOrders(user.id);

 })

 .then(orders => {

    return getPayments(orders);

 })

 .then(console.log);
```

Each step waits.

---

Flow:

```text
Task1
  ↓
Task2
  ↓
Task3
```

---

# 15. Returning Values

Example:

```js
Promise.resolve(5)

  .then(x => x * 2)

  .then(console.log);
```

Output:

```text
10
```

Returned value automatically becomes:

```text
Resolved Promise
```

---

Equivalent:

```js
.then(x => {

   return Promise.resolve(
      x * 2
   );

});
```

---

# 16. Returning Promises

Example:

```js
.then(() => {

   return fetchUser();

})
```

Chain waits automatically.

---

Flow:

```text
Promise
   ↓
Promise
   ↓
Promise
```

---

# 17. Promise.all()

Run multiple promises in parallel.

Example:

```js
Promise.all([
   fetchUser(),
   fetchOrders(),
   fetchPayments()
])
.then(console.log);
```

---

Result:

```js
[
  user,
  orders,
  payments
]
```

---

Important:

If one fails:

```text
Entire Promise.all Fails
```

---

Example:

```js
Promise.all([
  Promise.resolve(1),
  Promise.reject("Error"),
  Promise.resolve(3)
]);
```

Result:

```text
Rejected
```

---

# 18. Promise.allSettled()

Waits for all promises.

Example:

```js
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject("Failed")
]);
```

Output:

```js
[
  {
    status:"fulfilled",
    value:1
  },
  {
    status:"rejected",
    reason:"Failed"
  }
]
```

---

Use when:

```text
Need every result
```

---

# 19. Promise.race()

Returns first settled promise.

Example:

```js
Promise.race([
  slowRequest(),
  fastRequest()
]);
```

Winner:

```text
First Completed
```

---

Example:

```js
Promise.race([
  Promise.resolve("Fast"),
  new Promise(r =>
     setTimeout(
       ()=>r("Slow"),
       1000
     )
  )
]);
```

Output:

```text
Fast
```

---

# 20. Promise.any()

Returns first successful promise.

Example:

```js
Promise.any([
   Promise.reject("A"),
   Promise.resolve("B"),
   Promise.reject("C")
]);
```

Output:

```text
B
```

---

Difference:

```text
race → first settle

any → first success
```

---

# 21. Promise Queue

Promise callbacks do not go into Callback Queue.

They go into:

```text
Promise Queue
```

also called:

```text
Microtask Queue
```

---

Example:

```js
Promise.resolve()
  .then(() => {

    console.log("Promise");

  });
```

Stored in:

```text
Microtask Queue
```

---

# 22. Microtask Queue

Priority:

```text
process.nextTick

↓

Promise Queue

↓

Timers

↓

I/O

↓

Check
```

---

Example:

```js
setTimeout(() => {
   console.log("Timer");
},0);

Promise.resolve()
 .then(() => {
    console.log("Promise");
 });
```

Output:

```text
Promise
Timer
```

---

Why?

Microtasks execute before Event Loop phases.

---

# 23. Promise Resolution Procedure

Most advanced Promise topic.

Example:

```js
Promise.resolve(
   Promise.resolve(5)
)
```

Result:

```text
5
```

Not:

```text
Promise<Promise<5>>
```

---

Why?

Promises automatically unwrap nested promises.

---

Example:

```js
resolve(
  Promise.resolve("Hello")
);
```

Final value:

```text
Hello
```

---

This process is called:

```text
Promise Resolution Procedure
```

---

# 24. Internal Implementation

Simplified Promise:

```js
class MyPromise {

  state = "pending";

  value = undefined;

  resolve(value){

     this.state = "fulfilled";

     this.value = value;

  }

  reject(error){

     this.state = "rejected";

     this.value = error;

  }

}
```

Actual implementation is much more complex.

---

# 25. Real World Examples

## File Reading

```js
fs.promises
  .readFile("file.txt")
  .then(console.log);
```

---

## Database Query

```js
User.find()
   .then(users => {
      console.log(users);
   });
```

---

## HTTP Request

```js
fetch(url)
   .then(res => res.json())
   .then(console.log);
```

---

# 26. Interview Questions

## Basic

1. What is a Promise?
2. Why were Promises introduced?
3. What are Promise states?

---

## Intermediate

4. Difference between resolve and reject?
5. Explain then, catch, finally.
6. What is Promise chaining?
7. What happens when a Promise returns a value?

---

## Advanced

8. Promise.all vs Promise.allSettled?
9. Promise.race vs Promise.any?
10. What is Promise Resolution Procedure?
11. Explain Promise Queue.
12. Explain Microtask Queue.
13. Why do Promises execute before setTimeout?
14. How does Promise chaining work internally?

---

# 27. Mastery Checklist

* What is a Promise
* Promise States
* Pending
* Fulfilled
* Rejected
* Promise Constructor
* Resolve
* Reject
* then()
* catch()
* finally()
* Promise Chaining
* Sequential Execution
* Returning Values
* Returning Promises
* Promise.all()
* Promise.allSettled()
* Promise.race()
* Promise.any()
* Promise Queue
* Microtask Queue
* Promise Resolution Procedure
* Internal Mechanics
* Real World Usage

After mastering Promises, the next topic is **Async/Await**, which is syntactic sugar built on top of Promises and is the modern standard for asynchronous programming in Node.js.
