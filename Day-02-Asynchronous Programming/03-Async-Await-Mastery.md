# Async/Await Mastery Guide (Node.js & JavaScript)

## Table of Contents

1. What is Async/Await?
2. Why Async/Await Exists
3. What is an Async Function?
4. What does await do?
5. Async Function Return Values
6. Converting Promise Chains
7. Awaiting Multiple Promises
8. Sequential Execution
9. Parallel Execution
10. Async Loops
11. Async Iterators
12. Async Generators
13. Top-Level Await
14. Missing await
15. Unhandled Promise Rejections
16. Blocking Sequential Awaits
17. Using await inside forEach
18. Async/Await Internals
19. Event Loop Relationship
20. Real World Examples
21. Interview Questions
22. Mastery Checklist

---

# 1. What is Async/Await?

Async/Await is syntactic sugar built on top of Promises.

It allows asynchronous code to look synchronous.

Before:

```js
getUser()
  .then(user => getOrders(user.id))
  .then(orders => getPayments(orders))
  .then(console.log)
  .catch(console.error);
```

After:

```js
try {
  const user = await getUser();

  const orders = await getOrders(user.id);

  const payments = await getPayments(orders);

  console.log(payments);

} catch(err) {

  console.error(err);

}
```

Same behavior.

Cleaner syntax.

---

# 2. Why Async/Await Exists

Promise chains become difficult.

Example:

```js
getUser()
  .then(user => {
    return getOrders(user.id);
  })
  .then(orders => {
    return getPayments(orders);
  })
  .then(payments => {
    return getInvoice(payments);
  });
```

As complexity grows:

```text
Hard to Read
Hard to Debug
Hard to Maintain
```

Async/Await solves this.

---

# 3. What is an Async Function?

An async function is a function that automatically returns a Promise.

Syntax:

```js
async function greet() {

}
```

---

Example:

```js
async function greet() {
   return "Hello";
}
```

Output:

```js
console.log(greet());
```

Result:

```text
Promise { "Hello" }
```

---

Equivalent:

```js
function greet() {

   return Promise.resolve(
      "Hello"
   );

}
```

---

Important Rule

Every async function returns:

```text
Promise
```

even if you return:

```js
return 5;
```

Result:

```text
Promise<5>
```

---

# 4. What does await do?

await pauses execution until a Promise settles.

Example:

```js
const result =
  await Promise.resolve("Done");

console.log(result);
```

Output:

```text
Done
```

---

Without await

```js
const result =
  Promise.resolve("Done");

console.log(result);
```

Output:

```text
Promise { "Done" }
```

---

What await does:

```text
Promise
    ↓
Wait
    ↓
Resolved Value
```

---

# Important

await only works inside:

```js
async function
```

Example:

```js
function test() {

  await fetchUser();

}
```

Output:

```text
SyntaxError
```

---

Correct:

```js
async function test() {

  await fetchUser();

}
```

---

# 5. Async Function Return Values

Example:

```js
async function getNumber() {

   return 10;

}
```

Result:

```js
getNumber()
 .then(console.log);
```

Output:

```text
10
```

---

Internally:

```js
async function getNumber() {

  return Promise.resolve(10);

}
```

---

Returning Promise

```js
async function getUser() {

   return fetchUser();

}
```

Promise automatically adopted.

---

# 6. Converting Promise Chains

Promise Version

```js
getUser()

 .then(user => getOrders(user.id))

 .then(orders => getPayments(orders))

 .then(console.log)

 .catch(console.error);
```

---

Async/Await Version

```js
try {

  const user =
    await getUser();

  const orders =
    await getOrders(user.id);

  const payments =
    await getPayments(orders);

  console.log(payments);

}
catch(err){

  console.error(err);

}
```

---

Much easier to read.

---

# 7. Awaiting Multiple Promises

Bad:

```js
const user =
 await getUser();

const orders =
 await getOrders();
```

Runs sequentially.

---

Good:

```js
const [user, orders] =
 await Promise.all([
   getUser(),
   getOrders()
 ]);
```

Runs concurrently.

---

Performance improves significantly.

---

# 8. Sequential Execution

Each await waits.

Example:

```js
const user =
 await getUser();

const orders =
 await getOrders(user.id);

const payments =
 await getPayments(orders);
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

Use When:

```text
Next task depends on previous task
```

---

Example:

```text
Login
 ↓
Get Profile
 ↓
Get Orders
```

---

# 9. Parallel Execution

Independent tasks.

Example:

```js
const userPromise =
 getUser();

const ordersPromise =
 getOrders();

const paymentsPromise =
 getPayments();

const results =
 await Promise.all([
   userPromise,
   ordersPromise,
   paymentsPromise
 ]);
```

Flow:

```text
Task1 ─┐
Task2 ─┼──► Await All
Task3 ─┘
```

---

Much faster.

---

Sequential:

```text
1s + 1s + 1s = 3s
```

Parallel:

```text
1s
```

---

# 10. Async Loops

Common interview topic.

---

Sequential Loop

```js
for(const id of ids){

   const user =
     await getUser(id);

   console.log(user);

}
```

Runs one by one.

---

Flow:

```text
User1
 ↓
User2
 ↓
User3
```

---

Parallel Loop

```js
await Promise.all(

  ids.map(id =>
    getUser(id)
  )

);
```

Flow:

```text
User1 ─┐
User2 ─┼─► Parallel
User3 ─┘
```

---

# 11. Async Iterators

Introduced for asynchronous streams.

Example:

```js
for await (
  const chunk of stream
){

   console.log(chunk);

}
```

---

Works with:

```text
Streams
Network Data
Async Generators
```

---

Traditional:

```js
stream.on("data");
```

Modern:

```js
for await (...) {}
```

---

# 12. Async Generators

Combine:

```text
async
+
generator
```

Syntax:

```js
async function* numbers(){

}
```

---

Example:

```js
async function* numbers(){

   yield 1;

   yield 2;

   yield 3;

}
```

Usage:

```js
for await(
 const num of numbers()
){

 console.log(num);

}
```

Output:

```text
1
2
3
```

---

Useful for:

```text
Streaming Data
Pagination
Large Datasets
```

---

# 13. Top-Level Await

Modern ES Modules support:

```js
const data =
 await fetchData();
```

outside functions.

---

Before:

```js
async function main(){

   await fetchData();

}

main();
```

---

Now:

```js
await fetchData();
```

directly.

---

Requirement:

```json
{
  "type":"module"
}
```

or

```text
.mjs
```

---

# 14. Missing await

Very common bug.

Bad:

```js
const user =
   getUser();

console.log(user);
```

Output:

```text
Promise {...}
```

---

Correct:

```js
const user =
   await getUser();
```

---

# 15. Unhandled Promise Rejections

Bad:

```js
async function run(){

   await Promise.reject(
      "Failed"
   );

}
```

No error handling.

---

Output:

```text
UnhandledPromiseRejection
```

---

Correct:

```js
try {

   await riskyTask();

}
catch(err){

   console.error(err);

}
```

---

# 16. Blocking Sequential Awaits

Bad:

```js
const a =
 await task1();

const b =
 await task2();

const c =
 await task3();
```

Independent tasks.

Wasteful.

---

Good:

```js
const [a,b,c] =
 await Promise.all([
   task1(),
   task2(),
   task3()
 ]);
```

---

Performance:

```text
3 Seconds
   ↓
1 Second
```

---

# 17. Using await inside forEach

Very common interview trap.

Bad:

```js
users.forEach(
 async user => {

   await saveUser(user);

 });
```

forEach doesn't wait.

---

Output unpredictable.

---

Correct:

```js
for(const user of users){

   await saveUser(user);

}
```

---

Or Parallel:

```js
await Promise.all(

 users.map(
   saveUser
 )

);
```

---

# Why forEach Fails

forEach ignores returned promises.

Example:

```js
users.forEach(async user => {

   await save(user);

});

console.log("Done");
```

Output:

```text
Done
```

before saves complete.

---

# 18. Async/Await Internals

Example:

```js
async function test(){

   const value =
      await fetchData();

   return value;

}
```

Internally similar to:

```js
function test(){

   return fetchData()

      .then(value => {

         return value;

      });

}
```

---

Async/Await is syntax sugar.

Still uses:

```text
Promises
Microtasks
Event Loop
```

under the hood.

---

# 19. Event Loop Relationship

Example:

```js
async function run(){

   console.log("A");

   await Promise.resolve();

   console.log("B");

}

run();

console.log("C");
```

Output:

```text
A
C
B
```

---

Why?

Flow:

```text
A
 ↓
await
 ↓
Microtask Queue
 ↓
C
 ↓
B
```

---

await schedules continuation as a:

```text
Microtask
```

---

# 20. Real World Examples

## Database Query

```js
const users =
 await User.find();
```

---

## File Reading

```js
const data =
 await fs.promises.readFile(
   "file.txt"
 );
```

---

## HTTP Request

```js
const response =
 await fetch(url);

const data =
 await response.json();
```

---

## Multiple APIs

```js
const [
  users,
  posts,
  comments
] = await Promise.all([
  getUsers(),
  getPosts(),
  getComments()
]);
```

---

# 21. Interview Questions

## Basic

1. What is Async/Await?
2. What does async mean?
3. What does await do?
4. Can await be used outside async?

---

## Intermediate

5. What does an async function return?
6. Async/Await vs Promises?
7. Sequential vs Parallel execution?
8. How does error handling work?

---

## Advanced

9. Why does await not block Node.js?
10. How does await interact with the Event Loop?
11. What is Top-Level Await?
12. Why is await inside forEach bad?
13. How do Async Iterators work?
14. What are Async Generators?
15. How is async/await implemented internally?

---

# 22. Mastery Checklist

* Async Functions
* await Keyword
* Async Return Values
* Promise Conversion
* Sequential Execution
* Parallel Execution
* Promise.all()
* Async Loops
* Async Iterators
* Async Generators
* Top-Level Await
* Missing await
* Unhandled Rejections
* Blocking Sequential Awaits
* await inside forEach
* Async/Await Internals
* Event Loop Integration
* Real World Patterns

After mastering Async/Await, you'll be ready for advanced asynchronous patterns such as Promise Combinators, Concurrency Control, Streams with Async Iteration, Worker Threads, and Production-Grade Async Architecture.
