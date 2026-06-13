
# Pipeline API Mastery Guide (Node.js)

## Table of Contents

1. What is stream.pipeline()?
2. Why Pipeline Exists
3. Problems with pipe()
4. Why Use Pipeline
5. Automatic Error Handling
6. Multiple Stream Connections
7. Promise-based Pipeline
8. stream.finished()
9. Detect Stream Completion
10. Cleanup Resources
11. Internal Architecture
12. Real World Examples
13. Interview Questions
14. Mastery Checklist

---

# 1. What is stream.pipeline()?

Pipeline is a utility provided by Node.js for safely connecting streams.

Import:

```js
const {
  pipeline
} = require("stream");
```

Basic Example:

```js
pipeline(
  readStream,
  writeStream,
  err => {

    if (err) {
      console.error(err);
    }

  }
);
```

---

Pipeline connects streams and manages:

- Errors
- Cleanup
- Backpressure
- Stream Destruction

Automatically.

---

# 2. Why Pipeline Exists

Before pipeline:

```js
readStream
  .pipe(transform)
  .pipe(writeStream);
```

Looks simple.

But:

```text
Error Handling Difficult
Cleanup Manual
Resource Leaks Possible
```

---

Production systems require:

```text
Reliable Error Handling
Automatic Cleanup
Safe Stream Shutdown
```

Pipeline solves these issues.

---

# 3. Problems with pipe()

Example:

```js
readStream
 .pipe(gzip)
 .pipe(writeStream);
```

If:

```text
gzip Fails
```

then:

```text
Other Streams May Remain Open
```

Result:

```text
Memory Leaks
File Descriptor Leaks
Resource Leaks
```

---

Error propagation is manual.

Example:

```js
readStream.on("error", handler);
gzip.on("error", handler);
writeStream.on("error", handler);
```

Complex.

---

# 4. Why Use Pipeline

Pipeline provides:

```text
Automatic Error Handling
Automatic Cleanup
Automatic Stream Destruction
Backpressure Support
```

---

Example:

```js
pipeline(
  readStream,
  gzip,
  writeStream,
  callback
);
```

If any stream fails:

```text
Destroy All Streams
```

Automatically.

---

# Pipeline vs Pipe

| Feature | pipe() | pipeline() |
|----------|----------|----------|
| Easy Setup | Yes | Yes |
| Error Handling | Manual | Automatic |
| Cleanup | Manual | Automatic |
| Production Ready | Good | Excellent |

---

# 5. Automatic Error Handling

Most important feature.

Example:

```js
pipeline(
  readStream,
  gzip,
  writeStream,
  err => {

    if(err){

      console.error(
        err
      );

    }

  }
);
```

---

Flow

```text
Any Stream Error
↓
Pipeline Detects Error
↓
Destroy All Streams
↓
Execute Callback
```

---

Without pipeline:

```text
One Stream Fails
↓
Others Continue Running
```

Bad.

---

# Example

```js
pipeline(
 readStream,
 transform,
 writeStream,
 err => {

   if(err){

     console.log(
       "Pipeline Failed"
     );

   }

 }
);
```

Output:

```text
Pipeline Failed
```

if any stream errors.

---

# 6. Multiple Stream Connections

Pipeline supports multiple stages.

Example:

```js
pipeline(
  readStream,
  transform1,
  transform2,
  transform3,
  writeStream,
  callback
);
```

---

Flow

```text
Input
↓
Transform 1
↓
Transform 2
↓
Transform 3
↓
Output
```

---

Real Example

```text
File
↓
Decrypt
↓
Parse
↓
Transform
↓
Compress
↓
Store
```

---

Pipeline manages every stage.

---

# 7. Promise-based Pipeline

Modern Node.js provides:

```js
const {
 pipeline
} = require(
 "stream/promises"
);
```

---

Usage

```js
await pipeline(
 readStream,
 gzip,
 writeStream
);
```

---

Example

```js
const {
 pipeline
} = require(
 "stream/promises"
);

await pipeline(
 fs.createReadStream(
   "input.txt"
 ),
 zlib.createGzip(),
 fs.createWriteStream(
   "output.gz"
 )
);
```

---

Benefits

```text
Async/Await
Cleaner Code
Easier Error Handling
```

---

With try/catch

```js
try {

 await pipeline(
   readStream,
   transform,
   writeStream
 );

}
catch(err){

 console.error(err);

}
```

---

# 8. stream.finished()

Another important API.

Import:

```js
const {
 finished
} = require(
 "stream"
);
```

Purpose:

```text
Detect When Stream Ends
```

---

Example

```js
finished(
 stream,
 err => {

   if(err){

     console.error(err);

   }

   console.log(
     "Done"
   );

 }
);
```

---

Useful when:

```text
Need Notification
Need Cleanup
Need Monitoring
```

---

# Difference Between finish and finished()

finish Event:

```text
Writable Only
```

finished()

```text
Readable
Writable
Duplex
Transform
```

All stream types.

---

# 9. Detect Stream Completion

finished() detects:

```text
end
finish
error
close
```

events.

---

Flow

```text
Stream
↓
Success
↓
finished()
```

or

```text
Stream
↓
Error
↓
finished()
```

---

Example

```js
finished(
 readStream,
 err => {

   if(err){

     console.error(err);

   }

   console.log(
     "Completed"
   );

 }
);
```

---

Promise Version

```js
const {
 finished
} = require(
 "stream/promises"
);

await finished(
 stream
);
```

---

Example

```js
await finished(
 writeStream
);

console.log(
 "Done"
);
```

---

# 10. Cleanup Resources

Production systems must clean resources.

Resources:

```text
File Handles
Sockets
Memory Buffers
Network Connections
```

---

Pipeline automatically performs cleanup.

Example

```text
Error
↓
Destroy Streams
↓
Close Resources
```

---

Without Cleanup

```text
Open Files
Memory Leaks
Socket Leaks
```

---

Manual Cleanup

```js
stream.destroy();
```

---

Pipeline internally calls destroy when needed.

---

# Cleanup Example

```js
pipeline(
 source,
 transform,
 destination,
 err => {

   if(err){

     console.error(err);

   }

 }
);
```

If:

```text
Transform Crashes
```

Pipeline:

```text
Destroy Source
Destroy Transform
Destroy Destination
```

Automatically.

---

# 11. Internal Architecture

Pipeline Flow

```text
Readable
↓
Transform
↓
Transform
↓
Writable
```

---

Backpressure

```text
Writable Full
↓
Pause Previous Stream
↓
Resume After drain
```

Automatically.

---

Error Flow

```text
Error
↓
Pipeline
↓
Destroy Streams
↓
Callback
```

---

Internally

```text
Attach Error Listeners
Attach Close Listeners
Manage Backpressure
Manage Cleanup
```

---

# 12. Real World Examples

## File Compression

```js
await pipeline(
 readStream,
 zlib.createGzip(),
 writeStream
);
```

---

## File Decompression

```js
await pipeline(
 readStream,
 zlib.createGunzip(),
 writeStream
);
```

---

## Video Processing

```text
Video
↓
Transform
↓
Compress
↓
Store
```

---

## CSV Processing

```text
CSV
↓
Parser
↓
Transformer
↓
Database
```

---

## ETL Pipelines

```text
Extract
↓
Transform
↓
Load
```

---

## Log Processing

```text
Logs
↓
Parser
↓
Filter
↓
Store
```

---

# 13. Interview Questions

## Basic

1. What is stream.pipeline()?
2. Why was pipeline introduced?
3. Difference between pipe() and pipeline()?

---

## Intermediate

4. How does pipeline handle errors?
5. What is automatic cleanup?
6. Why use Promise-based pipeline?
7. What does finished() do?

---

## Advanced

8. How does pipeline manage backpressure?
9. What happens if a transform stream fails?
10. Why is pipeline preferred in production?
11. Difference between finish event and finished()?
12. Explain pipeline internals.
13. How does pipeline prevent resource leaks?

---

# 14. Mastery Checklist

- stream.pipeline()
- Why Pipeline Exists
- Problems with pipe()
- Automatic Error Handling
- Multiple Stream Connections
- Promise-based Pipeline
- Async/Await Pipelines
- stream.finished()
- Detect Stream Completion
- Cleanup Resources
- Stream Destruction
- Error Propagation
- Backpressure Support
- Internal Architecture
- Real World Usage
- Interview Preparation
