# Node.js Streams Mastery Guide

## Table of Contents

1. What are Streams?
2. Why Streams are Important
3. Streams vs Buffers
4. Types of Streams
5. Readable Streams
6. Reading Data
7. Stream Events
8. Flowing Mode
9. Paused Mode
10. Writable Streams
11. Writing Data
12. Backpressure
13. Drain Event
14. Pipe Mechanism
15. Stream Chaining
16. Error Handling
17. Custom Streams
18. Transform Streams
19. Object Mode
20. Stream Pipeline
21. HighWaterMark
22. Backpressure Handling
23. Real World Examples
24. Performance Optimization
25. Interview Questions
26. Mastery Checklist

---

# 1. What are Streams?

A Stream is a mechanism for processing data piece by piece (chunks) rather than loading the entire data into memory at once.

Instead of:

```text
Entire File
     ↓
Memory
     ↓
Process
```

Streams work like:

```text
File
 ↓
Chunk 1
 ↓
Chunk 2
 ↓
Chunk 3
 ↓
Process
```

---

## Real World Analogy

Imagine drinking water.

### Buffer Approach

```text
Fill Entire Tank
       ↓
Drink
```

### Stream Approach

```text
Drink While Water Flows
```

Streams are more memory efficient.

---

# 2. Why Streams are Important

Suppose you have:

```text
10 GB Video File
```

Bad:

```js
fs.readFile("movie.mp4");
```

Flow:

```text
10 GB File
     ↓
10 GB RAM
     ↓
Process
```

May crash your application.

---

Good:

```js
fs.createReadStream("movie.mp4");
```

Flow:

```text
10 GB File
     ↓
64 KB Chunk
     ↓
Process
     ↓
Next Chunk
```

Memory stays small.

---

# 3. Streams vs Buffers

## Buffer

Loads data into memory.

```js
fs.readFile("large.zip");
```

Memory:

```text
Entire File
      ↓
RAM
```

---

## Stream

Processes incrementally.

```js
fs.createReadStream("large.zip");
```

Memory:

```text
Chunk
Chunk
Chunk
```

---

Comparison:

| Feature      | Buffer                 | Stream    |
| ------------ | ---------------------- | --------- |
| Memory Usage | High                   | Low       |
| Speed        | Slower for large files | Faster    |
| Scalability  | Poor                   | Excellent |
| Large Files  | Risky                  | Safe      |

---

# 4. Types of Streams

Node provides four stream types.

```text
Readable
Writable
Duplex
Transform
```

---

## Readable Stream

Can read data.

Examples:

```js
fs.createReadStream()
http.IncomingMessage
process.stdin
```

Flow:

```text
Source
  ↓
Application
```

---

## Writable Stream

Can write data.

Examples:

```js
fs.createWriteStream()
process.stdout
http.ServerResponse
```

Flow:

```text
Application
      ↓
Destination
```

---

## Duplex Stream

Readable + Writable.

Examples:

```js
net.Socket
TCP Connections
WebSockets
```

Flow:

```text
Read
 ↑
 │
 ↓
Write
```

---

## Transform Stream

Duplex stream that modifies data.

Examples:

```js
zlib.createGzip()
crypto.createCipheriv()
```

Flow:

```text
Input
  ↓
Transform
  ↓
Output
```

---

# 5. Readable Streams

Example:

```js
const fs = require("fs");

const stream =
  fs.createReadStream("data.txt");
```

Node starts reading chunks.

---

Internal Flow:

```text
File
 ↓
Kernel
 ↓
Buffer
 ↓
Readable Stream
 ↓
Application
```

---

# 6. Reading Data

Example:

```js
stream.on("data", chunk => {
   console.log(chunk);
});
```

Chunk:

```text
Buffer
```

Example Output:

```text
<Buffer 48 65 6c 6c 6f>
```

---

Convert to string:

```js
stream.on("data", chunk => {
   console.log(chunk.toString());
});
```

Output:

```text
Hello
```

---

# 7. Stream Events

Important events:

```text
data
end
error
close
```

---

## data

Triggered when chunk arrives.

```js
stream.on("data", chunk => {
   console.log(chunk);
});
```

---

## end

Triggered when reading finishes.

```js
stream.on("end", () => {
   console.log("Done");
});
```

---

## error

Triggered when failure occurs.

```js
stream.on("error", err => {
   console.error(err);
});
```

---

# 8. Flowing Mode

Data automatically flows.

Example:

```js
stream.on("data", chunk => {
   console.log(chunk);
});
```

Flow:

```text
Source
 ↓
Auto Push
 ↓
Consumer
```

---

# 9. Paused Mode

Application manually pulls data.

Example:

```js
stream.on("readable", () => {

   let chunk;

   while(
      null !== (chunk = stream.read())
   ){
      console.log(chunk);
   }

});
```

Flow:

```text
Source
 ↓
Wait
 ↓
Manual Read
```

---

# 10. Writable Streams

Create:

```js
const stream =
 fs.createWriteStream("output.txt");
```

Write:

```js
stream.write("Hello");
```

---

Flow:

```text
Application
      ↓
Writable Stream
      ↓
Disk
```

---

# 11. Writing Data

Example:

```js
stream.write("NodeJS");
```

Multiple writes:

```js
stream.write("A");
stream.write("B");
stream.write("C");
```

Finish:

```js
stream.end();
```

---

# 12. Backpressure

Most important stream topic.

Interview favorite.

---

Problem:

Producer faster than consumer.

```text
Producer
 100 MB/s
      ↓
Consumer
 10 MB/s
```

Data accumulates.

Memory grows.

Eventually:

```text
Out Of Memory
```

---

Example:

```js
readable.pipe(writable);
```

Readable:

```text
Fast
```

Writable:

```text
Slow
```

Need regulation.

---

# How Node Handles Backpressure

```js
const ok = stream.write(data);
```

Return:

```js
false
```

Means:

```text
STOP WRITING
```

Buffer is full.

---

# 13. Drain Event

When writable becomes available again:

```js
stream.on("drain", () => {
   console.log("Continue");
});
```

Flow:

```text
Buffer Full
      ↓
Pause
      ↓
Drain
      ↓
Resume
```

---

# 14. Pipe Mechanism

Most powerful stream feature.

Without pipe:

```js
readable.on("data", chunk => {
   writable.write(chunk);
});
```

Manual.

---

With pipe:

```js
readable.pipe(writable);
```

Node automatically:

* Reads
* Writes
* Handles backpressure

---

Flow:

```text
Readable
    │
    ▼
Writable
```

---

# 15. Stream Chaining

Multiple streams together.

Example:

```js
readable
  .pipe(transform1)
  .pipe(transform2)
  .pipe(writable);
```

Flow:

```text
File
 ↓
Transform
 ↓
Transform
 ↓
Destination
```

---

# Gzip Example

```js
fs.createReadStream("file.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("file.gz"));
```

Flow:

```text
File
 ↓
Compression
 ↓
Compressed File
```

---

# 16. Error Handling

Bad:

```js
readable.pipe(writable);
```

No error handling.

---

Good:

```js
readable.on("error", console.error);
writable.on("error", console.error);
```

---

# 17. Custom Streams

Create custom readable stream.

```js
const {
  Readable
} = require("stream");
```

Example:

```js
class NumberStream
extends Readable {

  constructor() {
    super();
    this.count = 1;
  }

  _read() {

    this.push(
      String(this.count++)
    );

    if(this.count > 5){
       this.push(null);
    }

  }

}
```

---

Output:

```text
1
2
3
4
5
```

---

# 18. Transform Streams

Transform data while streaming.

Example:

```js
const {
 Transform
} = require("stream");
```

Uppercase transform:

```js
class UpperCase
extends Transform {

 _transform(
   chunk,
   encoding,
   callback
 ){

   callback(
      null,
      chunk
        .toString()
        .toUpperCase()
   );

 }

}
```

Input:

```text
hello
```

Output:

```text
HELLO
```

---

# 19. Object Mode

Normally:

```text
Streams Handle Buffers
```

Object Mode:

```text
Streams Handle Objects
```

Example:

```js
new Readable({
  objectMode:true
});
```

Can stream:

```js
{
 id:1,
 name:"John"
}
```

instead of Buffers.

---

Useful for:

* Database Records
* JSON Processing
* ETL Pipelines

---

# 20. Stream Pipeline

Modern stream API.

Instead of:

```js
readable
 .pipe(transform)
 .pipe(writable);
```

Use:

```js
const {
 pipeline
} = require("stream");
```

Example:

```js
pipeline(
  readable,
  transform,
  writable,
  err => {

    if(err)
      console.error(err);

  }
);
```

Benefits:

```text
Automatic Cleanup
Automatic Error Handling
```

---

# 21. HighWaterMark

Defines internal buffer size.

Default:

```text
16 KB
```

for most streams.

File streams:

```text
64 KB
```

---

Example:

```js
fs.createReadStream(
  "file.txt",
  {
     highWaterMark:
       1024
  }
);
```

Chunk size:

```text
1 KB
```

---

# Why Important?

Large:

```text
Fewer Reads
More Memory
```

Small:

```text
Less Memory
More Reads
```

Trade-off.

---

# 22. Backpressure Handling

Manual handling:

```js
if(!stream.write(chunk)){

   readable.pause();

   stream.once(
      "drain",
      () => readable.resume()
   );

}
```

Flow:

```text
Write
 ↓
Full?
 ↓
Pause
 ↓
Drain
 ↓
Resume
```

---

# 23. Real World Examples

## File Upload

```text
Browser
 ↓
Readable Stream
 ↓
Server
 ↓
Disk
```

---

## Video Streaming

```text
Video File
 ↓
Readable Stream
 ↓
HTTP Response
 ↓
Browser
```

---

## Compression

```text
File
 ↓
Gzip
 ↓
Compressed File
```

---

## Database Export

```text
Database
 ↓
Readable
 ↓
CSV Transform
 ↓
File
```

---

# 24. Performance Optimization

Avoid:

```js
fs.readFile(
  "10GB-file.zip"
);
```

---

Prefer:

```js
fs.createReadStream(
  "10GB-file.zip"
);
```

---

Avoid:

```text
Huge Buffers
```

Use:

```text
Streaming
```

---

Monitor:

```text
Memory
Backpressure
Throughput
```

---

# Interview Questions

## Basic

1. What is a Stream?
2. Why use Streams?
3. Streams vs Buffers?
4. Types of Streams?

---

## Intermediate

5. Explain Readable Streams.
6. Explain Writable Streams.
7. What is Flowing Mode?
8. What is Paused Mode?
9. How does pipe() work?

---

## Advanced

10. What is Backpressure?
11. How does Node handle Backpressure?
12. What is HighWaterMark?
13. What are Transform Streams?
14. What is Object Mode?
15. Why use pipeline() over pipe()?
16. How would you stream a 100GB file?
17. How do streams improve scalability?

---

# Mastery Checklist

* What are Streams
* Streams vs Buffers
* Readable Streams
* Writable Streams
* Duplex Streams
* Transform Streams
* Reading Data
* Stream Events
* Flowing Mode
* Paused Mode
* Writing Data
* Backpressure
* Drain Event
* pipe()
* Stream Chaining
* Error Handling
* Custom Streams
* Transform Streams
* Object Mode
* Pipeline
* HighWaterMark
* Backpressure Handling
* Performance Optimization

After mastering Streams, you'll be ready for advanced topics such as File System Internals, HTTP Internals, TCP Networking, WebSockets, Compression, ETL Pipelines, and building highly scalable Node.js applications.
