
# Transform Streams Mastery Guide (Node.js)

## Table of Contents

1. What are Transform Streams?
2. Why Transform Streams Exist
3. Input → Process → Output
4. Data Transformation
5. Compression Example
6. Encryption Example
7. CSV to JSON Conversion
8. Uppercase Transformer
9. _transform() Method
10. _flush() Method
11. Chaining Multiple Transforms
12. Object Mode Transform Streams
13. Internal Architecture
14. Real World Examples
15. Interview Questions
16. Mastery Checklist

---

# 1. What are Transform Streams?

A Transform Stream is a special type of Duplex Stream.

It:

- Reads Data
- Transforms Data
- Outputs New Data

Flow:

Input
↓
Transform
↓
Output

---

# 2. Why Transform Streams Exist

Many applications need to modify data while it is flowing.

Examples:

- Compression
- Encryption
- Decryption
- CSV Parsing
- JSON Parsing
- Video Processing
- Log Processing

Without transforms:

Read Entire File
↓
Modify
↓
Write Entire File

With transforms:

Read Chunk
↓
Transform Chunk
↓
Write Chunk

---

# 3. Input → Process → Output

Core Transform Pattern

```text
Input
 ↓
 Process
 ↓
 Output
```

Example

```text
hello world
```

Transform:

```text
UPPERCASE
```

Output:

```text
HELLO WORLD
```

---

# 4. Data Transformation

Transform streams sit between source and destination.

```text
File
↓
Transform
↓
File
```

or

```text
HTTP Request
↓
Transform
↓
Database
```

---

# Example

Input

```text
john
```

Transform

```text
Uppercase
```

Output

```text
JOHN
```

---

# 5. Compression Example

Node provides zlib.

```js
const zlib =
 require("zlib");
```

Gzip Transform

```js
const gzip =
 zlib.createGzip();
```

Pipeline

```js
readStream
 .pipe(gzip)
 .pipe(writeStream);
```

Flow

```text
File
↓
Gzip
↓
Compressed File
```

---

# Why Compression Uses Transform

Input:

```text
100 MB
```

Output:

```text
20 MB
```

Data changes while moving.

---

# 6. Encryption Example

Encryption is another transform.

Input:

```text
hello
```

Transform:

```text
AES Encrypt
```

Output:

```text
x8af9k...
```

---

Crypto Example

```js
const crypto =
 require("crypto");

const cipher =
 crypto.createCipheriv(
   algorithm,
   key,
   iv
 );
```

Pipeline

```js
readStream
 .pipe(cipher)
 .pipe(writeStream);
```

---

# 7. CSV to JSON Conversion

Input CSV

```csv
name,age
john,25
```

Transform

```text
CSV Parser
```

Output

```json
{
  "name":"john",
  "age":25
}
```

---

Pipeline

```text
CSV File
↓
Transform
↓
JSON Objects
```

Used in ETL systems.

---

# 8. Uppercase Transformer

Simple Example

```js
const {
 Transform
} = require(
 "stream"
);

class UpperCase
 extends Transform {

 _transform(
   chunk,
   encoding,
   callback
 ) {

   this.push(
     chunk
       .toString()
       .toUpperCase()
   );

   callback();

 }

}
```

---

Usage

```js
readStream
 .pipe(
   new UpperCase()
 )
 .pipe(writeStream);
```

Input

```text
hello
```

Output

```text
HELLO
```

---

# 9. _transform() Method

Most important Transform method.

Signature

```js
_transform(
 chunk,
 encoding,
 callback
)
```

Parameters

```text
chunk
```

Current input data.

```text
encoding
```

Input encoding.

```text
callback
```

Signals completion.

---

Example

```js
_transform(
 chunk,
 encoding,
 callback
){

 const result =
   chunk
    .toString()
    .toUpperCase();

 this.push(
   result
 );

 callback();

}
```

---

Flow

Input Chunk
↓
_transform()
↓
push()
↓
Output Chunk

---

# this.push()

Sends transformed data forward.

Example

```js
this.push(
 transformedData
);
```

Without push:

```text
No Output
```

---

# 10. _flush() Method

Called when stream finishes.

Useful for:

- Cleanup
- Remaining Data
- Final Processing

---

Example

```js
_flush(
 callback
){

 this.push(
   "END"
 );

 callback();

}
```

---

Flow

```text
Last Chunk
↓
_flush()
↓
Final Output
↓
End
```

---

# Example Use Case

CSV Parser

Partial Data

```text
john,25
mary
```

Remaining buffer processed inside:

```js
_flush()
```

---

# 11. Chaining Multiple Transforms

Transforms can be chained.

Example

```js
readStream
 .pipe(transformA)
 .pipe(transformB)
 .pipe(transformC)
 .pipe(writeStream);
```

---

Flow

```text
Input
↓
Transform A
↓
Transform B
↓
Transform C
↓
Output
```

---

Example

```text
hello
```

Transform A

```text
uppercase
```

Result

```text
HELLO
```

Transform B

```text
reverse
```

Result

```text
OLLEH
```

---

# Why Chaining Matters

Unix-style pipelines.

```text
Data
↓
Filter
↓
Convert
↓
Compress
↓
Store
```

---

# 12. Object Mode Transform Streams

Default Streams:

```text
Buffer
String
```

Object Mode:

```text
JavaScript Objects
```

---

Enable

```js
new Transform({
 objectMode: true
});
```

---

Example

Input

```js
{
 id:1,
 name:"John"
}
```

Transform

```js
{
 id:1,
 name:"JOHN"
}
```

---

Example

```js
const {
 Transform
} = require(
 "stream"
);

const upperCaseNames =
 new Transform({

   objectMode:true,

   transform(
     obj,
     enc,
     cb
   ){

     obj.name =
       obj.name
          .toUpperCase();

     cb(
       null,
       obj
     );

   }

 });
```

---

Used for:

- CSV Parsers
- Database Records
- ETL Pipelines
- JSON Processing

---

# Object Mode HighWaterMark

Default:

```text
16 Objects
```

Instead of:

```text
16 KB
64 KB
```

---

# 13. Internal Architecture

Transform Stream

```text
Readable
+
Writable
```

---

Flow

```text
Input
↓
Writable Buffer
↓
_transform()
↓
Readable Buffer
↓
Output
```

---

Internal Components

```text
Writable Queue
Readable Queue
Transform Logic
Backpressure Logic
```

---

# Backpressure

If output side is slow:

```text
Readable Buffer Fills
↓
Pause Input
```

Transform streams automatically support backpressure.

---

# 14. Real World Examples

## Gzip Compression

```js
zlib.createGzip()
```

---

## Encryption

```js
crypto.createCipheriv()
```

---

## CSV Parser

```js
csv-parser
```

---

## JSON Processor

```js
JSON Transform
```

---

## Image Processing

```text
Image
↓
Resize
↓
Compress
↓
Store
```

---

## Log Processing

```text
Logs
↓
Filter
↓
Analyze
↓
Store
```

---

# 15. Interview Questions

## Basic

1. What is a Transform Stream?
2. Difference between Duplex and Transform?
3. Why use Transform Streams?

---

## Intermediate

4. What does _transform() do?
5. What does this.push() do?
6. What is _flush()?
7. Why chain transforms?

---

## Advanced

8. Explain Transform Stream internals.
9. How does backpressure work?
10. Explain Object Mode.
11. How do compression streams work?
12. Why are gzip streams Transform Streams?
13. Explain CSV to JSON conversion using transforms.

---

# 16. Mastery Checklist

- Transform Streams
- Input → Process → Output
- Data Transformation
- Compression
- Encryption
- CSV to JSON
- Uppercase Transformer
- _transform()
- _flush()
- this.push()
- Transform Pipeline
- Chaining Multiple Transforms
- Object Mode
- Internal Architecture
- Backpressure
- Real World Usage
- Interview Preparation
