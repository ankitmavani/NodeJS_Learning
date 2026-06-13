
# Pipe & Stream Pipeline Mastery Guide (Node.js)

## Table of Contents

1. What is Pipe?
2. Why Pipe Exists
3. Stream Chaining
4. .pipe() Method
5. .unpipe() Method
6. File Copying Example
7. Compression Pipeline
8. Data Processing Pipeline
9. Pipe Error Handling
10. Resource Cleanup
11. Multiple Pipes
12. Stream Pipelines
13. pipeline() API
14. Internal Architecture
15. Real World Examples
16. Interview Questions
17. Mastery Checklist

---

# 1. What is Pipe?

Pipe is a mechanism that connects:

```text
Readable Stream
        ↓
Writable Stream
```

Automatically transferring data.

Example:

```js
readStream.pipe(writeStream);
```

Data flows automatically.

---

# 2. Why Pipe Exists

Without pipe:

```js
readStream.on(
  "data",
  chunk => {

    writeStream.write(
      chunk
    );

  }
);
```

Manual handling:

- Read
- Write
- Backpressure
- Cleanup

Complex.

---

With pipe:

```js
readStream.pipe(
  writeStream
);
```

Everything becomes simpler.

---

# 3. Stream Chaining

Pipe allows streams to be connected.

Example:

```text
File
↓
Transform
↓
Transform
↓
File
```

Each stream processes data and passes it forward.

---

Example

```js
streamA
 .pipe(streamB)
 .pipe(streamC);
```

Flow

```text
A
↓
B
↓
C
```

---

# 4. .pipe() Method

Syntax

```js
readable.pipe(
  writable
);
```

---

Example

```js
const fs =
 require("fs");

const readStream =
 fs.createReadStream(
   "input.txt"
 );

const writeStream =
 fs.createWriteStream(
   "output.txt"
 );

readStream.pipe(
 writeStream
);
```

---

Flow

```text
input.txt
↓
Read Stream
↓
Pipe
↓
Write Stream
↓
output.txt
```

---

Return Value

```js
const result =
 readStream.pipe(
   writeStream
 );
```

Returns:

```text
Destination Stream
```

Allows chaining.

---

# Pipe Lifecycle

```text
open
↓
data
↓
data
↓
data
↓
end
↓
finish
↓
close
```

---

# Automatic Backpressure

Most important benefit.

Pipe automatically:

```text
Pause Readable
```

when:

```text
Writable Buffer Full
```

and

```text
Resume Readable
```

when:

```text
drain Event
```

occurs.

---

# 5. .unpipe() Method

Disconnects streams.

Example

```js
readStream.unpipe(
 writeStream
);
```

---

Flow

Before

```text
Readable
↓
Writable
```

After

```text
Readable

Writable
```

No data transfer.

---

# Selective Unpipe

```js
readStream.unpipe(
 specificStream
);
```

Only disconnects one destination.

---

# 6. File Copying Example

Most common use case.

```js
const fs =
 require("fs");

fs.createReadStream(
 "source.txt"
)
.pipe(
 fs.createWriteStream(
   "copy.txt"
 )
);
```

---

Flow

```text
source.txt
↓
Read Stream
↓
Pipe
↓
Write Stream
↓
copy.txt
```

---

Benefits

```text
Low Memory Usage
Large File Support
Automatic Backpressure
```

---

# 7. Compression Pipeline

Example

```js
const fs =
 require("fs");

const zlib =
 require("zlib");

fs.createReadStream(
 "file.txt"
)
.pipe(
 zlib.createGzip()
)
.pipe(
 fs.createWriteStream(
   "file.txt.gz"
 )
);
```

---

Flow

```text
File
↓
Read Stream
↓
Gzip Transform
↓
Write Stream
↓
Compressed File
```

---

# Why Compression Uses Pipe

Chunks are compressed while flowing.

```text
Chunk
↓
Compress
↓
Write
```

No need to load entire file.

---

# 8. Data Processing Pipeline

Example

```text
CSV
↓
CSV Parser
↓
JSON Transformer
↓
Database Writer
```

---

Implementation

```js
readStream
 .pipe(csvParser)
 .pipe(jsonTransform)
 .pipe(dbWriter);
```

---

Benefits

```text
Modular
Reusable
Scalable
```

---

# 9. Pipe Error Handling

Common mistake:

```js
readStream.pipe(
 writeStream
);
```

without error handlers.

---

Bad

```text
Error
↓
Unhandled
↓
Crash
```

---

Correct

```js
readStream.on(
 "error",
 console.error
);

writeStream.on(
 "error",
 console.error
);
```

---

Transform Streams

```js
transform.on(
 "error",
 console.error
);
```

---

Always handle:

```text
Readable Errors
Writable Errors
Transform Errors
```

---

# Common Pipe Errors

```text
ENOENT
```

File missing.

---

```text
EACCES
```

Permission denied.

---

```text
ENOSPC
```

Disk full.

---

```text
ERR_STREAM_DESTROYED
```

Destination closed.

---

# 10. Resource Cleanup

Important for production.

Example

```js
readStream.destroy();

writeStream.destroy();
```

---

Flow

```text
Error
↓
Destroy Streams
↓
Release Resources
```

---

Without Cleanup

```text
Open File Handles
Memory Leaks
Resource Leaks
```

---

# 11. Multiple Pipes

Pipe supports multiple stages.

Example

```js
streamA
 .pipe(streamB)
 .pipe(streamC)
 .pipe(streamD);
```

---

Flow

```text
A
↓
B
↓
C
↓
D
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
Store
```

---

# Fan Out Example

```js
readStream.pipe(
 writeStream1
);

readStream.pipe(
 writeStream2
);
```

---

Flow

```text
Readable
↓ ↘
A   B
```

One source.

Multiple destinations.

---

# 12. Stream Pipelines

Pipeline is an advanced pipe pattern.

Traditional

```js
streamA
 .pipe(streamB)
 .pipe(streamC);
```

Works.

But error handling is difficult.

---

Pipeline provides:

```text
Automatic Cleanup
Better Error Handling
Safer Production Usage
```

---

# 13. pipeline() API

Node API

```js
const {
 pipeline
} = require(
 "stream"
);
```

---

Example

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
Read
↓
Transform
↓
Write
```

---

Benefits

```text
Automatic Error Propagation
Automatic Cleanup
Production Safe
```

---

Pipeline Error Flow

```text
Error
↓
Destroy Streams
↓
Callback
```

---

# pipeline() vs pipe()

pipe()

```text
Simple
```

But:

```text
Manual Errors
Manual Cleanup
```

---

pipeline()

```text
Production Ready
```

Handles both.

---

Comparison

| Feature | pipe() | pipeline() |
|----------|----------|----------|
| Easy | Yes | Yes |
| Error Handling | Manual | Automatic |
| Cleanup | Manual | Automatic |
| Production | Good | Best |

---

# 14. Internal Architecture

Pipe Internals

```text
Readable
↓
Internal Buffer
↓
Writable
```

---

Backpressure Flow

```text
Readable
↓
write()
↓
false
↓
pause()
↓
drain
↓
resume()
```

Pipe handles this automatically.

---

# 15. Real World Examples

## File Copy

```js
readStream.pipe(
 writeStream
);
```

---

## Compression

```js
readStream
 .pipe(gzip)
 .pipe(writeStream);
```

---

## Encryption

```js
readStream
 .pipe(cipher)
 .pipe(writeStream);
```

---

## Video Processing

```text
Video
↓
Resize
↓
Compress
↓
Store
```

---

## ETL Systems

```text
CSV
↓
Parser
↓
Transform
↓
Database
```

---

# 16. Interview Questions

## Basic

1. What is pipe()?
2. Why use pipe()?
3. What does pipe() return?
4. What is unpipe()?

---

## Intermediate

5. How does pipe() work?
6. Why is pipe better than manual reads?
7. What is stream chaining?
8. What is automatic backpressure?

---

## Advanced

9. Difference between pipe() and pipeline()?
10. How does pipe handle backpressure?
11. Why is pipeline safer?
12. Explain multiple pipes.
13. Explain cleanup during failures.
14. What happens internally during pipe()?

---

# 17. Mastery Checklist

- What is Pipe
- Why Pipe Exists
- Stream Chaining
- .pipe()
- .unpipe()
- File Copying
- Compression Pipeline
- Data Processing Pipeline
- Pipe Error Handling
- Resource Cleanup
- Multiple Pipes
- Stream Pipelines
- pipeline()
- Automatic Backpressure
- Internal Architecture
- Real World Usage
- Interview Preparation
