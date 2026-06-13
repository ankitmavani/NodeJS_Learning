
# Readable Streams Mastery Guide (Node.js)

## Table of Contents

1. What are Readable Streams?
2. Why Readable Streams Exist
3. Creating Read Streams
4. Reading Large Files
5. .read() Method
6. .pause() Method
7. .resume() Method
8. Flowing Mode
9. Paused Mode
10. data Event
11. end Event
12. readable Event
13. HighWaterMark
14. Backpressure Basics
15. Internal Architecture
16. Real World Examples
17. Interview Questions
18. Mastery Checklist

---

# 1. What are Readable Streams?

Readable Streams are streams that allow applications to read data from a source.

Examples:

- Files
- Network Connections
- HTTP Requests
- Process Input
- Compression Streams

Example:

```js
const fs = require("fs");

const stream =
  fs.createReadStream("file.txt");
```

Data is received chunk-by-chunk.

---

# 2. Why Readable Streams Exist

Without streams:

```js
const data =
  await fs.promises.readFile(
    "bigfile.txt"
  );
```

Problem:

Entire file loads into memory.

Example:

```text
10 GB File
↓
10 GB RAM
```

With streams:

```js
fs.createReadStream(
  "bigfile.txt"
);
```

Memory:

```text
64 KB
↓
64 KB
↓
64 KB
```

Very efficient.

---

# 3. Creating Read Streams

Basic Example

```js
const fs = require("fs");

const stream =
  fs.createReadStream(
    "file.txt"
  );
```

Options:

```js
const stream =
  fs.createReadStream(
    "file.txt",
    {
      encoding: "utf8",
      highWaterMark: 65536
    }
  );
```

---

# 4. Reading Large Files

Example:

```js
const stream =
  fs.createReadStream(
    "movie.mp4"
  );

stream.on(
  "data",
  chunk => {

    console.log(
      chunk.length
    );

  }
);
```

Instead of:

```text
Load Entire File
```

Node performs:

```text
Chunk 1
↓
Chunk 2
↓
Chunk 3
```

This is why streams scale.

---

# 5. .read() Method

Reads data manually.

Example:

```js
stream.on(
  "readable",
  () => {

    let chunk;

    while(
      null !== (
        chunk =
        stream.read()
      )
    ) {

      console.log(chunk);

    }

  }
);
```

Returns:

```text
Buffer
or
String
```

depending on encoding.

---

# How .read() Works

Internal Buffer

```text
Chunk
Chunk
Chunk
```

.read()

```text
Take Chunk
↓
Return Chunk
```

Used mostly in:

```text
Paused Mode
```

---

# 6. .pause() Method

Stops automatic data flow.

Example:

```js
stream.pause();
```

Flow:

Before:

```text
Chunk
Chunk
Chunk
```

After:

```text
Stopped
```

Useful when:

```text
Consumer is Slow
```

---

# Example

```js
stream.on(
  "data",
  chunk => {

    stream.pause();

    setTimeout(
      () => {

        console.log(chunk);

        stream.resume();

      },
      1000
    );

  }
);
```

---

# 7. .resume() Method

Restarts flow after pause.

Example:

```js
stream.resume();
```

Flow:

```text
Paused
↓
resume()
↓
Continue Reading
```

Usually paired with:

```js
pause()
resume()
```

---

# 8. Flowing Mode

Automatic reading mode.

Example:

```js
stream.on(
  "data",
  chunk => {

    console.log(chunk);

  }
);
```

Adding:

```js
data
```

listener automatically switches stream into:

```text
Flowing Mode
```

---

Flowing Mode Diagram

```text
Disk
↓
Stream
↓
data
↓
data
↓
data
```

No manual reading needed.

---

# Characteristics

- Automatic
- Fast
- Event Driven

---

# 9. Paused Mode

Manual reading mode.

Default mode.

Example:

```js
stream.on(
  "readable",
  () => {

    let chunk;

    while(
      null !== (
        chunk =
        stream.read()
      )
    ) {

      console.log(chunk);

    }

  }
);
```

---

Flow

```text
Disk
↓
Buffer
↓
read()
↓
Application
```

---

Characteristics

- Manual Control
- Better Precision
- Useful for Parsers

---

# Flowing vs Paused Mode

| Feature | Flowing | Paused |
|----------|----------|----------|
| Reading | Automatic | Manual |
| Event | data | readable |
| Control | Low | High |
| Simplicity | Easy | Advanced |

---

# 10. data Event

Most common event.

```js
stream.on(
  "data",
  chunk => {

    console.log(chunk);

  }
);
```

Triggered whenever new data arrives.

Flow:

```text
Chunk
↓
data Event
↓
Chunk
↓
data Event
```

---

# 11. end Event

Triggered when all data has been consumed.

```js
stream.on(
  "end",
  () => {

    console.log(
      "Finished"
    );

  }
);
```

Sequence:

```text
data
↓
data
↓
data
↓
end
```

---

# 12. readable Event

Triggered when data becomes available.

Example:

```js
stream.on(
  "readable",
  () => {

    let chunk;

    while(
      null !== (
        chunk =
        stream.read()
      )
    ) {

      console.log(chunk);

    }

  }
);
```

Difference:

```text
data
↓
Automatic

readable
↓
Manual
```

---

# 13. HighWaterMark

One of the most important stream concepts.

Defines:

```text
Maximum Internal Buffer Size
```

Example:

```js
const stream =
 fs.createReadStream(
   "file.txt",
   {
     highWaterMark:
       1024
   }
 );
```

Meaning:

```text
1 KB Chunks
```

---

Default Values

File Streams:

```text
64 KB
```

Object Mode:

```text
16 Objects
```

---

Large Value

```text
More Memory
Fewer Reads
```

Small Value

```text
Less Memory
More Reads
```

---

# HighWaterMark Visualization

```text
Buffer Capacity
┌───────────────┐
│               │
│    64 KB      │
│               │
└───────────────┘
```

Once filled:

```text
Pause Reading
```

until consumer catches up.

---

# 14. Backpressure Basics

Most important stream topic.

Occurs when:

```text
Producer
↓
Faster
↓
Consumer
```

Example:

```text
Disk
↓
100 MB/s
```

Consumer:

```text
Database
↓
10 MB/s
```

Problem:

```text
Memory Growth
```

---

Backpressure Solution

Readable Stream:

```text
Pause
```

Writable Stream:

```text
Drain
```

---

Flow

```text
Producer
↓
Too Fast
↓
Buffer Fills
↓
Pause Reading
↓
Consumer Catches Up
↓
Resume
```

---

Node handles much of this automatically.

---

# 15. Internal Architecture

```text
Disk
↓
OS
↓
Libuv
↓
Readable Stream
↓
Internal Buffer
↓
Application
```

Chunk Flow:

```text
Disk
↓
64KB
↓
Buffer
↓
data Event
```

---

# 16. Real World Examples

## Log Processing

```js
fs.createReadStream(
  "logs.txt"
);
```

---

## Video Streaming

```js
fs.createReadStream(
  "movie.mp4"
);
```

---

## CSV Processing

```js
fs.createReadStream(
  "users.csv"
);
```

---

## HTTP Requests

```js
req.on(
  "data",
  chunk => {}
);
```

Incoming requests are readable streams.

---

# 17. Interview Questions

## Basic

1. What is a Readable Stream?
2. Why use Readable Streams?
3. What is createReadStream()?

## Intermediate

4. Difference between data and readable?
5. What is Flowing Mode?
6. What is Paused Mode?
7. What does .read() do?
8. What do pause() and resume() do?

## Advanced

9. What is HighWaterMark?
10. Explain Backpressure.
11. What happens internally during createReadStream()?
12. Why are streams memory efficient?
13. How do streams use Libuv?
14. How does Node prevent memory explosions?

---

# 18. Mastery Checklist

- Readable Streams
- createReadStream()
- Reading Large Files
- .read()
- .pause()
- .resume()
- Flowing Mode
- Paused Mode
- data Event
- end Event
- readable Event
- HighWaterMark
- Backpressure Basics
- Internal Architecture
- Real World Usage
- Interview Questions
