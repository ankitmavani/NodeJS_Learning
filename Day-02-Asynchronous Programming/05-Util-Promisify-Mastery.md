# util.promisify() Mastery Guide (Node.js)

## Table of Contents

1. Introduction
2. What is util.promisify()?
3. Why util.promisify() Exists
4. Callback vs Promise APIs
5. How util.promisify() Works
6. Converting Callback APIs
7. File System Examples
8. Legacy API Conversion
9. util.callbackify()
10. Custom Promisify Functions
11. Promisifying Class Methods
12. Binding Context (this)
13. Batch Promisification
14. Internal Implementation
15. Performance Considerations
16. Real World Examples
17. Interview Questions
18. Mastery Checklist

---

# 1. Introduction

Before Promises and Async/Await, Node.js relied heavily on callbacks.

Example:

```js
const fs = require("fs");

fs.readFile("file.txt", (err, data) => {

   if(err)
      return console.error(err);

   console.log(data.toString());

});
```

Problems:

```text
Nested Callbacks
Error Handling Complexity
Callback Hell
Difficult Composition
```

To modernize old APIs, Node.js introduced:

```js
util.promisify()
```

---

# 2. What is util.promisify()?

util.promisify() converts a Node.js style callback function into a Promise-based function.

Import:

```js
const util = require("util");
```

Usage:

```js
const promiseFn =
   util.promisify(callbackFn);
```

---

Example

Callback Version:

```js
fs.readFile(
  "file.txt",
  callback
);
```

Promisified Version:

```js
const readFile =
   util.promisify(fs.readFile);

await readFile("file.txt");
```

---

# 3. Why util.promisify() Exists

Node.js originally used:

```js
(err, result)
```

callback pattern.

Example:

```js
getUser(id, (err, user) => {

});
```

Modern JavaScript prefers:

```js
const user =
   await getUser(id);
```

util.promisify bridges the gap.

---

Without Promisify

```js
fs.readFile(file, (err,data)=>{

   if(err)
      return reject(err);

   resolve(data);

});
```

Need manual Promise wrapping.

---

With Promisify

```js
const readFile =
  util.promisify(
     fs.readFile
  );
```

Done automatically.

---

# 4. Callback vs Promise APIs

Callback:

```js
function getUser(id, callback){

   callback(null, user);

}
```

Usage:

```js
getUser(1,(err,user)=>{

});
```

---

Promise:

```js
function getUser(id){

   return Promise.resolve(user);

}
```

Usage:

```js
const user =
   await getUser(1);
```

---

Comparison

| Feature        | Callback  | Promise  |
| -------------- | --------- | -------- |
| Error Handling | Manual    | Built-In |
| Chaining       | Difficult | Easy     |
| Async/Await    | No        | Yes      |
| Readability    | Lower     | Higher   |

---

# 5. How util.promisify() Works

Suppose:

```js
function getData(callback){

   callback(
      null,
      "Hello"
   );

}
```

Promisify:

```js
const getDataPromise =
   util.promisify(getData);
```

Usage:

```js
const result =
   await getDataPromise();
```

Output:

```text
Hello
```

---

Internally:

```js
function promisified(...args){

   return new Promise(
     (resolve,reject)=>{

       originalFn(
          ...args,
          (err,result)=>{

             if(err)
                reject(err);

             else
                resolve(result);

          }
       );

     }
   );

}
```

---

# 6. Converting Callback APIs

Original:

```js
function sum(a,b,callback){

   callback(
      null,
      a+b
   );

}
```

Promisify:

```js
const sumAsync =
 util.promisify(sum);
```

Usage:

```js
const result =
 await sumAsync(5,10);

console.log(result);
```

Output:

```text
15
```

---

# 7. File System Examples

Very common interview example.

---

Traditional

```js
const fs = require("fs");

fs.readFile(
 "file.txt",
 "utf8",
 (err,data)=>{

   if(err)
      return console.error(err);

   console.log(data);

 });
```

---

Promisify Version

```js
const fs = require("fs");
const util = require("util");

const readFile =
 util.promisify(
    fs.readFile
 );

const data =
 await readFile(
    "file.txt",
    "utf8"
 );

console.log(data);
```

---

Write File

```js
const writeFile =
 util.promisify(
    fs.writeFile
 );
```

Usage:

```js
await writeFile(
  "test.txt",
  "Hello"
);
```

---

# 8. Legacy API Conversion

Older npm packages often use callbacks.

Example:

```js
legacyApi(
  options,
  callback
);
```

Convert:

```js
const legacyApiAsync =
 util.promisify(
    legacyApi
 );
```

Now:

```js
const result =
 await legacyApiAsync(
   options
 );
```

---

Useful during migration.

```text
Callback API
      ↓
Promisify
      ↓
Async/Await
```

---

# 9. util.callbackify()

Reverse operation.

Converts Promise API into Callback API.

Import:

```js
const util =
 require("util");
```

---

Promise Function

```js
async function getUser(){

   return {
      name:"John"
   };

}
```

Convert:

```js
const getUserCb =
 util.callbackify(
    getUser
 );
```

Usage:

```js
getUserCb(
 (err,user)=>{

    console.log(user);

 });
```

Output:

```js
{
  name:"John"
}
```

---

Why Needed?

Legacy systems expecting callbacks.

---

# 10. Custom Promisify Functions

Some APIs don't follow:

```js
(err,result)
```

pattern.

Example:

```js
function weirdApi(
 success,
 failure
){

}
```

Normal promisify fails.

---

Solution:

Custom Promise Wrapper

```js
function weirdApiAsync(){

  return new Promise(
    (resolve,reject)=>{

      weirdApi(
        resolve,
        reject
      );

    }
  );

}
```

---

# util.promisify.custom

Node provides custom behavior.

Example:

```js
myFn[
 util.promisify.custom
] = function(){

   return Promise.resolve(
      "Custom"
   );

};
```

Now:

```js
util.promisify(myFn);
```

uses custom implementation.

---

# 11. Promisifying Class Methods

Common Interview Question.

---

Class:

```js
class UserService {

   getUser(id, callback){

      callback(
        null,
        { id }
      );

   }

}
```

---

Wrong:

```js
const getUser =
 util.promisify(
   service.getUser
 );
```

Error:

```text
this lost
```

---

Why?

Method detached from object.

---

# 12. Binding Context (this)

Correct:

```js
const getUser =
 util.promisify(
   service.getUser.bind(
      service
   )
 );
```

Now:

```js
await getUser(1);
```

works.

---

Flow:

```text
Method
   ↓
bind(this)
   ↓
promisify
```

---

Very important in interviews.

---

# 13. Batch Promisification

Large codebase:

```js
fs.readFile
fs.writeFile
fs.stat
fs.unlink
```

Need multiple conversions.

---

Example:

```js
const methods = [

  "readFile",
  "writeFile",
  "stat"

];
```

Loop:

```js
for(const method of methods){

   fs[method] =
      util.promisify(
        fs[method]
      );

}
```

Now:

```js
await fs.readFile();
await fs.writeFile();
```

---

Alternative

Use:

```js
fs.promises
```

Modern Node.js already provides Promise APIs.

---

Example:

```js
const fs =
 require("fs").promises;
```

Usage:

```js
await fs.readFile(
   "file.txt"
);
```

No promisify needed.

---

# 14. Internal Implementation

Simplified version:

```js
function promisify(fn){

   return function(...args){

      return new Promise(
       (resolve,reject)=>{

         fn(
           ...args,

           (
             err,
             result
           )=>{

             if(err)
               reject(err);

             else
               resolve(result);

           }

         );

       }
      );

   };

}
```

Core idea:

```text
Callback
     ↓
Promise
```

---

# 15. Performance Considerations

Promisify adds:

```text
Promise Creation
Closure Creation
Microtask Scheduling
```

Slight overhead.

---

Example:

```js
util.promisify(fn)
```

is slower than:

```js
nativePromiseFn()
```

---

Modern Node:

Prefer:

```js
fs.promises
dns.promises
timers/promises
```

instead of promisify.

---

# 16. Real World Examples

## DNS

```js
const dns =
 require("dns");

const lookup =
 util.promisify(
   dns.lookup
 );

const result =
 await lookup(
   "google.com"
 );
```

---

## Crypto

```js
const pbkdf2 =
 util.promisify(
   crypto.pbkdf2
 );
```

Usage:

```js
const hash =
 await pbkdf2(
   password,
   salt,
   10000,
   64,
   "sha512"
 );
```

---

## Legacy Database Driver

```js
const query =
 util.promisify(
   db.query.bind(db)
 );
```

Usage:

```js
const users =
 await query(
   "SELECT * FROM users"
 );
```

---

# 17. Interview Questions

## Basic

1. What is util.promisify()?
2. Why was it introduced?
3. What kind of functions can be promisified?

---

## Intermediate

4. How does promisify work internally?
5. Difference between promisify and callbackify?
6. Why is promisify useful with Async/Await?
7. Why does Node use Error-First Callbacks?

---

## Advanced

8. Why can class methods fail when promisified?
9. How do you preserve this context?
10. What is util.promisify.custom?
11. How would you batch promisify APIs?
12. Why should fs.promises be preferred over promisify(fs.readFile)?

---

# 18. Mastery Checklist

* What is util.promisify()
* Why it Exists
* Callback to Promise Conversion
* Error-First Callback Pattern
* File System Examples
* Legacy API Migration
* util.callbackify()
* Custom Promisify Logic
* util.promisify.custom
* Promisifying Methods
* Binding this
* Batch Promisification
* Internal Implementation
* Performance Tradeoffs
* Modern Alternatives

After mastering util.promisify(), you'll understand how Node.js evolved from callback-based APIs to Promise-based APIs and how modern Async/Await code interoperates with legacy Node.js modules.
