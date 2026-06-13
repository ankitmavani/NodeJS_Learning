
# Compression Streams Mastery Guide (Node.js)

## Table of Contents

1. Introduction to Compression Streams
2. What is the zlib Module?
3. Why Compression Matters
4. Gzip Compression
5. Gunzip Decompression
6. Deflate Compression
7. Inflate Decompression
8. Brotli Compression
9. Compression Stream Architecture
10. Compression Pipelines
11. Log Compression
12. API Response Compression
13. File Storage Optimization
14. Performance Considerations
15. Internal Working of Compression Streams
16. Real World Examples
17. Interview Questions
18. Mastery Checklist

---

# 1. Introduction to Compression Streams

Compression Streams reduce data size while data is flowing through a stream.

Flow:

Input Data
↓
Compression Stream
↓
Compressed Data

Benefits:

- Lower Storage Usage
- Faster Network Transfer
- Reduced Bandwidth
- Better Scalability

---

# 2. What is the zlib Module?

Node.js provides built-in compression support through:

```js
const zlib = require("zlib");
```

No installation required.

Supported algorithms:

- Gzip
- Deflate
- Brotli

zlib streams are Transform Streams.

---

Flow

```text
Readable
↓
zlib Transform
↓
Writable
```

---

# 3. Why Compression Matters

Example

Original File

```text
100 MB
```

Compressed

```text
20 MB
```

Savings

```text
80%
```

---

Benefits

Storage:

```text
More Files
Less Disk Space
```

Network:

```text
Smaller Responses
Faster Downloads
```

---

# 4. Gzip Compression

Most commonly used compression format.

Create Gzip Stream

```js
const gzip =
 zlib.createGzip();
```

---

File Compression Example

```js
const fs = require("fs");
const zlib = require("zlib");

fs.createReadStream("input.txt")
 .pipe(zlib.createGzip())
 .pipe(
   fs.createWriteStream(
     "input.txt.gz"
   )
 );
```

---

Flow

```text
input.txt
↓
Gzip
↓
input.txt.gz
```

---

Gzip Characteristics

```text
High Compression
Widely Supported
Common On Web
```

---

# 5. Gunzip Decompression

Used to restore original data.

Create Stream

```js
const gunzip =
 zlib.createGunzip();
```

---

Example

```js
fs.createReadStream(
 "input.txt.gz"
)
.pipe(
 zlib.createGunzip()
)
.pipe(
 fs.createWriteStream(
   "output.txt"
 )
);
```

---

Flow

```text
Compressed File
↓
Gunzip
↓
Original File
```

---

# 6. Deflate Compression

Another compression algorithm.

Create Stream

```js
const deflate =
 zlib.createDeflate();
```

---

Example

```js
fs.createReadStream(
 "input.txt"
)
.pipe(
 zlib.createDeflate()
)
.pipe(
 fs.createWriteStream(
   "compressed.deflate"
 )
);
```

---

Characteristics

```text
Fast
Compact
Widely Supported
```

---

# 7. Inflate Decompression

Reverse of Deflate.

Create Stream

```js
const inflate =
 zlib.createInflate();
```

---

Example

```js
fs.createReadStream(
 "compressed.deflate"
)
.pipe(
 zlib.createInflate()
)
.pipe(
 fs.createWriteStream(
   "output.txt"
 )
);
```

---

Flow

```text
Deflate Data
↓
Inflate
↓
Original Data
```

---

# 8. Brotli Compression

Modern compression algorithm.

Introduced for:

```text
Web Performance
```

---

Create Brotli Stream

```js
const brotli =
 zlib.createBrotliCompress();
```

---

Example

```js
fs.createReadStream(
 "input.txt"
)
.pipe(
 zlib.createBrotliCompress()
)
.pipe(
 fs.createWriteStream(
   "input.br"
 )
);
```

---

Decompression

```js
zlib.createBrotliDecompress()
```

---

Advantages

```text
Better Compression Ratio
Smaller Output
Modern Browsers Support
```

---

Comparison

| Algorithm | Speed | Compression |
|------------|---------|---------|
| Deflate | Fast | Medium |
| Gzip | Medium | Good |
| Brotli | Slower | Best |

---

# 9. Compression Stream Architecture

Compression streams are:

```text
Transform Streams
```

Flow

```text
Input
↓
Compression Algorithm
↓
Compressed Output
```

---

Internal Structure

```text
Readable Side
↓
Compression Engine
↓
Writable Side
```

---

# 10. Compression Pipelines

Compression usually uses pipe() or pipeline().

Example

```js
readStream
 .pipe(
   zlib.createGzip()
 )
 .pipe(writeStream);
```

---

Production Version

```js
const {
 pipeline
} = require(
 "stream"
);

pipeline(
 readStream,
 zlib.createGzip(),
 writeStream,
 callback
);
```

---

Benefits

```text
Error Handling
Cleanup
Backpressure
```

---

# 11. Log Compression

Real-world use case.

Application Logs

```text
app.log
```

can become:

```text
10 GB
```

---

Compress Old Logs

```js
fs.createReadStream(
 "app.log"
)
.pipe(
 zlib.createGzip()
)
.pipe(
 fs.createWriteStream(
   "app.log.gz"
 )
);
```

---

Benefits

```text
Less Storage
Cheaper Backups
```

---

# 12. API Response Compression

Very common in production.

Without Compression

```text
Response
500 KB
```

---

With Gzip

```text
Response
50 KB
```

---

Express Example

```js
const compression =
 require(
   "compression"
 );

app.use(
 compression()
);
```

---

Flow

```text
JSON Response
↓
Gzip
↓
Client
```

---

Benefits

```text
Faster APIs
Lower Bandwidth
Better User Experience
```

---

# 13. File Storage Optimization

Large files consume storage.

Example

```text
Reports
Logs
Exports
Backups
```

---

Compression

```text
100 GB
↓
20 GB
```

---

Used In

```text
AWS Backups
Database Dumps
Archive Systems
```

---

# 14. Performance Considerations

Compression uses CPU.

Trade-off:

```text
Less Storage
↓
More CPU
```

---

Gzip

```text
Balanced
```

---

Brotli

```text
Better Compression
More CPU
```

---

For huge workloads:

```text
Measure Compression Ratio
Measure CPU Usage
```

---

# Compression Levels

Example

```js
zlib.createGzip({
 level: 9
});
```

---

Levels

```text
1 = Fast
9 = Best Compression
```

---

# 15. Internal Working of Compression Streams

Flow

```text
Chunk
↓
Compression Engine
↓
Compressed Chunk
```

---

Stream Processing

```text
Chunk 1
↓
Compress
↓
Write

Chunk 2
↓
Compress
↓
Write
```

---

Benefits

```text
Low Memory
Large File Support
```

---

Backpressure supported automatically.

---

# 16. Real World Examples

## Website Assets

```text
HTML
CSS
JS
↓
Gzip/Brotli
↓
Browser
```

---

## Log Rotation

```text
app.log
↓
gzip
↓
app.log.gz
```

---

## Database Backups

```text
dump.sql
↓
gzip
↓
dump.sql.gz
```

---

## API Responses

```text
JSON
↓
Compression
↓
Client
```

---

## Cloud Storage

```text
Files
↓
Compress
↓
Store
```

---

# 17. Interview Questions

## Basic

1. What is zlib?
2. What is Gzip?
3. What is Gunzip?
4. Why compress data?

---

## Intermediate

5. Difference between Gzip and Deflate?
6. What is Brotli?
7. Why are compression streams Transform streams?
8. How do you compress a file using Node.js?

---

## Advanced

9. Explain Compression Stream Internals.
10. How does compression interact with backpressure?
11. Why is Brotli preferred for web assets?
12. Compression vs CPU trade-off?
13. How would you compress multi-GB log files?

---

# 18. Mastery Checklist

- zlib Module
- Gzip Compression
- Gunzip Decompression
- Deflate
- Inflate
- Brotli Compression
- Compression Pipelines
- Log Compression
- API Response Compression
- File Storage Optimization
- Compression Levels
- Performance Considerations
- Stream Internals
- Backpressure Support
- Real World Usage
- Interview Preparation
