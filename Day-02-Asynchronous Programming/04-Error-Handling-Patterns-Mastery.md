# Error Handling Patterns Mastery Guide (Node.js)

## Table of Contents

1. Introduction to Error Handling
2. Why Error Handling Matters
3. Types of Errors in JavaScript
4. Basic Error Handling

   * try/catch
   * throw
   * finally
5. Promise Error Handling

   * .catch()
   * Async/Await Error Handling
6. Callback Error Handling
7. Error-First Callback Pattern
8. Operational Errors vs Programmer Errors
9. Global Error Handling
10. process.on('uncaughtException')
11. process.on('unhandledRejection')
12. Creating Custom Error Classes
13. Error Codes
14. Error Metadata
15. Centralized Error Handling
16. Error Logging
17. Error Wrapping
18. Retry Logic
19. Production Error Handling Architecture
20. Real World Examples
21. Interview Questions
22. Mastery Checklist

---

# 1. Introduction to Error Handling

Error handling is the process of detecting, managing, and recovering from unexpected situations.

Without error handling:

```text
Application
     ↓
Unexpected Error
     ↓
Crash
```

With proper error handling:

```text
Application
     ↓
Error
     ↓
Log
     ↓
Recover / Fail Gracefully
```

---

# 2. Why Error Handling Matters

Production systems must:

* Prevent crashes
* Protect data
* Provide meaningful messages
* Enable debugging
* Improve reliability

Bad:

```js
const user = null;

console.log(user.name);
```

Output:

```text
TypeError
```

Application may crash.

---

# 3. Types of Errors in JavaScript

## Syntax Error

```js
if(true {
}
```

Output:

```text
SyntaxError
```

---

## Reference Error

```js
console.log(user);
```

Output:

```text
ReferenceError
```

---

## Type Error

```js
null.toString();
```

Output:

```text
TypeError
```

---

## Range Error

```js
new Array(-1);
```

Output:

```text
RangeError
```

---

## Custom Error

```js
throw new Error("Custom Error");
```

---

# 4. Basic Error Handling

## try/catch

Example:

```js
try {

   const user = null;

   console.log(user.name);

}
catch(err){

   console.error(err);

}
```

Output:

```text
TypeError
```

Application continues.

---

## Execution Flow

```text
try
 ↓
Error?
 ↓
catch
```

---

## throw

Used to create errors manually.

Example:

```js
function divide(a,b){

   if(b === 0){

      throw new Error(
         "Division by zero"
      );

   }

   return a / b;

}
```

---

## finally

Always executes.

Example:

```js
try{

   connectDB();

}
catch(err){

   console.error(err);

}
finally{

   closeDB();

}
```

---

Useful for:

* Database Connections
* File Handles
* Cleanup

---

# 5. Promise Error Handling

## .catch()

Example:

```js
Promise.reject(
  new Error("Failed")
)
.catch(err => {

   console.error(err);

});
```

---

## Error Propagation

```js
Promise.resolve()

 .then(() => {

    throw new Error(
      "Something Failed"
    );

 })

 .catch(console.error);
```

---

Errors automatically move down the chain.

---

## Async/Await Error Handling

Example:

```js
try {

   const user =
      await getUser();

}
catch(err){

   console.error(err);

}
```

---

Equivalent:

```js
getUser()
 .catch(console.error);
```

---

# 6. Callback Error Handling

Before Promises:

```js
fs.readFile(
  "file.txt",
  (err,data) => {

      if(err){

         console.error(err);
         return;

      }

      console.log(data);

  }
);
```

---

Flow:

```text
Operation
    ↓
Error?
 ┌──┴──┐
Yes   No
 ↓     ↓
Handle Continue
```

---

# 7. Error-First Callback Pattern

Node.js Standard.

Signature:

```js
(err, result)
```

Success:

```js
callback(null, data);
```

Failure:

```js
callback(error);
```

---

Example:

```js
function getUser(id, callback){

   if(!id){

      return callback(
         new Error("Invalid ID")
      );

   }

   callback(null, {
      id,
      name:"John"
   });

}
```

---

Usage:

```js
getUser(1,(err,user)=>{

   if(err)
      return console.error(err);

   console.log(user);

});
```

---

# 8. Operational Errors vs Programmer Errors

Very important Node.js interview topic.

---

## Operational Errors

Expected runtime failures.

Examples:

```text
Database Down
File Not Found
Network Timeout
Invalid Input
API Failure
```

Example:

```js
fs.readFile("missing.txt");
```

These should be handled.

---

## Programmer Errors

Bugs in code.

Examples:

```js
null.toString();

undefined.name;

Infinite Recursion;
```

Example:

```js
const user = null;

user.name;
```

These should be fixed, not recovered.

---

Rule:

```text
Operational Errors → Handle

Programmer Errors → Fix
```

---

# 9. Global Error Handling

Some errors escape local handlers.

Need global protection.

Example:

```js
throw new Error("Crash");
```

Without handler:

```text
Node Process Crashes
```

---

# 10. process.on('uncaughtException')

Captures uncaught synchronous exceptions.

Example:

```js
process.on(
  "uncaughtException",
  err => {

     console.error(err);

  }
);
```

---

Example:

```js
setTimeout(() => {

   throw new Error("Boom");

},1000);
```

Caught globally.

---

Important:

Do NOT continue running.

Recommended:

```js
process.exit(1);
```

Reason:

Application state may be corrupted.

---

# 11. process.on('unhandledRejection')

Captures rejected promises without catch.

Example:

```js
process.on(
  "unhandledRejection",
  err => {

     console.error(err);

  }
);
```

---

Example:

```js
Promise.reject(
  new Error("Failed")
);
```

Without catch:

```text
Unhandled Rejection
```

---

Global handler catches it.

---

# 12. Creating Custom Error Classes

Very common in production.

---

Example:

```js
class ValidationError
extends Error {

   constructor(message){

      super(message);

      this.name =
         "ValidationError";

   }

}
```

Usage:

```js
throw new ValidationError(
   "Email Required"
);
```

---

Benefits:

```text
Readable
Structured
Reusable
```

---

# Multiple Error Types

```js
class NotFoundError
extends Error {}

class DatabaseError
extends Error {}

class AuthError
extends Error {}
```

---

# 13. Error Codes

Production systems use codes.

Example:

```js
class AppError
extends Error {

   constructor(
      code,
      message
   ){

      super(message);

      this.code = code;

   }

}
```

Usage:

```js
throw new AppError(
  "USER_NOT_FOUND",
  "User missing"
);
```

---

Benefits:

```text
Machine Readable
API Friendly
Consistent
```

---

# 14. Error Metadata

Attach additional information.

Example:

```js
class ValidationError
extends Error {

 constructor(
   message,
   field
 ){

   super(message);

   this.field = field;

 }

}
```

Usage:

```js
throw new ValidationError(
  "Invalid Email",
  "email"
);
```

Output:

```js
{
  message:"Invalid Email",
  field:"email"
}
```

---

Useful for APIs.

---

# 15. Centralized Error Handling

Production Best Practice.

Instead of:

```js
try{}
catch{}

try{}
catch{}

try{}
catch{}
```

Use:

```text
Controllers
      ↓
Middleware
      ↓
Error Handler
```

---

Express Example

```js
app.use(
  (err,req,res,next)=>{

     res.status(500).json({

       success:false,
       message:err.message

     });

  }
);
```

---

Benefits:

```text
Single Place
Consistent Response
Easier Maintenance
```

---

# 16. Error Logging

Never ignore errors.

Bad:

```js
catch(err){}
```

---

Good:

```js
catch(err){

   logger.error(err);

}
```

---

Popular Tools:

```text
Winston
Pino
Bunyan
```

---

Log:

```text
Timestamp
Stack Trace
User
Request
Environment
```

---

# 17. Error Wrapping

Add context to errors.

Example:

```js
try {

   await db.query();

}
catch(err){

   throw new Error(
      `Database Query Failed: ${err.message}`
   );

}
```

---

Modern Approach

```js
throw new Error(
  "Database Query Failed",
  {
    cause: err
  }
);
```

---

Benefits:

```text
Original Error Preserved
More Context
Better Debugging
```

---

# 18. Retry Logic

Useful for transient failures.

Examples:

```text
Network Error
Database Timeout
API Rate Limit
```

---

Basic Retry

```js
async function retry(fn,retries){

  while(retries > 0){

     try{

        return await fn();

     }
     catch(err){

        retries--;

     }

  }

}
```

---

Exponential Backoff

```text
1s
2s
4s
8s
```

instead of:

```text
1s
1s
1s
1s
```

---

Used in:

```text
AWS SDK
Google APIs
Stripe
Database Drivers
```

---

# 19. Production Error Handling Architecture

```text
Request
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Error Flow:

```text
Database Error
      ↓
Repository
      ↓
Service
      ↓
Controller
      ↓
Central Error Handler
      ↓
Response
```

---

Recommended Structure

```text
src/

errors/
 ├── AppError.js
 ├── ValidationError.js
 ├── NotFoundError.js

middlewares/
 └── errorHandler.js

utils/
 └── logger.js
```

---

# 20. Real World Examples

## Validation

```js
if(!email){

   throw new ValidationError(
      "Email Required"
   );

}
```

---

## Authentication

```js
if(!token){

   throw new AuthError(
      "Unauthorized"
   );

}
```

---

## Database

```js
if(!user){

   throw new NotFoundError(
      "User Not Found"
   );

}
```

---

# 21. Interview Questions

## Basic

1. What is try/catch?
2. What does throw do?
3. What is finally?

---

## Intermediate

4. Error-first callback pattern?
5. Promise error handling?
6. Async/Await error handling?
7. Operational vs Programmer Errors?

---

## Advanced

8. uncaughtException vs unhandledRejection?
9. Why use custom errors?
10. What is centralized error handling?
11. What is error wrapping?
12. How does retry logic work?
13. Why should uncaught exceptions terminate the process?
14. How would you design production-grade error handling?

---

# 22. Mastery Checklist

* Error Basics
* try/catch
* throw
* finally
* Promise Errors
* Async/Await Errors
* Callback Errors
* Error-First Pattern
* Operational Errors
* Programmer Errors
* uncaughtException
* unhandledRejection
* Custom Errors
* Error Codes
* Error Metadata
* Centralized Handling
* Error Logging
* Error Wrapping
* Retry Logic
* Production Architecture

After mastering Error Handling Patterns, you'll be ready for advanced topics such as Event Emitters, Streams Error Handling, HTTP Error Management, Distributed Systems Resilience, and Production-Grade Node.js Architecture.
