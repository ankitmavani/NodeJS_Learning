# File System (fs) Mastery Guide (Node.js)

# Table of Contents

1. Introduction to File System Module
2. What is the File System Module?
3. Synchronous vs Asynchronous APIs
4. Callback-Based APIs
5. Promise-Based APIs (fs/promises)
6. Reading Files
7. fs.readFile()
8. fs.readFileSync()
9. Reading Text Files
10. Reading JSON Files
11. Encoding Options
12. Writing Files
13. fs.writeFile()
14. fs.writeFileSync()
15. Overwrite vs Append
16. fs.appendFile()
17. Log File Management
18. Create File
19. Delete File (unlink)
20. Rename File
21. Copy File
22. Move File
23. Create Directory (mkdir)
24. Nested Directories
25. Read Directory (readdir)
26. Delete Directory (rmdir, rm)
27. File Information (stat)
28. File Size
29. File Permissions
30. Created/Modified Dates
31. Watching Files
32. fs.watch()
33. File Change Detection
34. File Descriptors
35. Open/Close Files
36. Large File Handling
37. Temporary Files
38. Internal Architecture
39. Real-World Examples
40. Interview Questions
41. Mastery Checklist

---

# 1. Introduction to File System Module

The File System module allows Node.js applications to interact with files and directories on the operating system.

Import:

```js
const fs = require("fs");
```

Promise Version:

```js
const fs = require("fs/promises");
```

---

# Why File System Matters

Almost every backend application uses files.

Examples:

```text
Configuration Files
Logs
Images
Videos
Documents
Backups
Uploads
Reports
```

Flow:

```text
Application
      ↓
File System Module
      ↓
Operating System
      ↓
Disk
```

---

# 2. What is the File System Module?

The fs module provides APIs for:

```text
Read Files
Write Files
Create Files
Delete Files
Rename Files
Copy Files
Watch Files
Manage Directories
```

Node delegates file operations to:

```text
Libuv Thread Pool
```

Because file I/O is slow.

---

# 3. Synchronous vs Asynchronous APIs

Node provides two versions.

---

## Synchronous APIs

Block execution.

Example:

```js
const data =
 fs.readFileSync("file.txt");
```

Flow:

```text
Read File
     ↓
Wait
     ↓
Continue
```

---

## Asynchronous APIs

Non-blocking.

Example:

```js
fs.readFile(
 "file.txt",
 callback
);
```

Flow:

```text
Start Read
     ↓
Continue Execution
     ↓
Callback Later
```

---

Comparison

| Feature           | Sync  | Async              |
| ----------------- | ----- | ------------------ |
| Blocks Event Loop | Yes   | No                 |
| Performance       | Lower | Higher             |
| Production Safe   | No    | Yes                |
| Simplicity        | Easy  | Better Scalability |

---

# 4. Callback-Based APIs

Classic Node style.

Example:

```js
fs.readFile(
 "file.txt",
 (err,data)=>{

   if(err)
      return console.error(err);

   console.log(data);

 }
);
```

Pattern:

```js
(err,result)
```

Known as:

```text
Error First Callback
```

---

# 5. Promise-Based APIs

Modern Node.js.

Import:

```js
const fs =
 require("fs/promises");
```

Example:

```js
const data =
 await fs.readFile(
   "file.txt",
   "utf8"
 );
```

Benefits:

```text
Cleaner
Async/Await
Better Error Handling
```

---

# 6. Reading Files

Most common operation.

Methods:

```js
fs.readFile()
fs.readFileSync()
```

---

# 7. fs.readFile()

Asynchronous.

Example:

```js
fs.readFile(
 "notes.txt",
 "utf8",
 (err,data)=>{

    if(err)
       return console.error(err);

    console.log(data);

 });
```

Output:

```text
File Contents
```

---

Flow:

```text
Application
      ↓
Libuv Thread Pool
      ↓
Disk
      ↓
Callback
```

---

# 8. fs.readFileSync()

Synchronous version.

Example:

```js
const data =
 fs.readFileSync(
   "notes.txt",
   "utf8"
 );

console.log(data);
```

Blocks Event Loop.

---

Avoid in:

```text
HTTP Servers
Production APIs
```

Use only:

```text
Scripts
Startup Config
CLI Tools
```

---

# 9. Reading Text Files

Example:

```js
const content =
 await fs.readFile(
   "notes.txt",
   "utf8"
 );
```

Output:

```text
Hello World
```

---

Without encoding:

```js
const content =
 await fs.readFile(
   "notes.txt"
 );
```

Output:

```text
Buffer
```

---

# 10. Reading JSON Files

Example:

config.json

```json
{
  "port": 3000
}
```

Read:

```js
const text =
 await fs.readFile(
   "config.json",
   "utf8"
 );

const config =
 JSON.parse(text);
```

Result:

```js
{
  port:3000
}
```

---

# 11. Encoding Options

Common encodings:

```text
utf8
ascii
base64
hex
latin1
```

Example:

```js
fs.readFile(
 "file.txt",
 "utf8",
 callback
);
```

Without encoding:

```text
Buffer
```

With encoding:

```text
String
```

---

# 12. Writing Files

Methods:

```js
fs.writeFile()
fs.writeFileSync()
```

---

# 13. fs.writeFile()

Asynchronous write.

Example:

```js
await fs.writeFile(
  "test.txt",
  "Hello Node"
);
```

Creates file if missing.

---

Overwrite behavior:

```text
Existing Content
      ↓
Removed
      ↓
New Content
```

---

# 14. fs.writeFileSync()

Synchronous version.

Example:

```js
fs.writeFileSync(
 "test.txt",
 "Hello"
);
```

Blocks execution.

---

# 15. Overwrite vs Append

writeFile:

```js
await fs.writeFile(
 "log.txt",
 "Entry"
);
```

Output:

```text
Entry
```

Every call replaces old data.

---

# 16. fs.appendFile()

Adds data at end.

Example:

```js
await fs.appendFile(
 "log.txt",
 "\nNew Entry"
);
```

Result:

```text
Entry
New Entry
```

---

Useful for:

```text
Logs
Audit Trails
Reports
```

---

# 17. Log File Management

Example:

```js
await fs.appendFile(
 "app.log",
 `${Date.now()} Login\n`
);
```

Result:

```text
17100000 Login
17100010 Logout
```

---

Production loggers:

```text
Pino
Winston
Bunyan
```

use append operations.

---

# 18. Create File

Simplest way:

```js
await fs.writeFile(
 "new.txt",
 ""
);
```

Creates empty file.

---

Alternative:

```js
await fs.open(
 "new.txt",
 "w"
);
```

---

# 19. Delete File (unlink)

Example:

```js
await fs.unlink(
 "old.txt"
);
```

Removes file permanently.

---

Error if file missing:

```text
ENOENT
```

---

# 20. Rename File

Example:

```js
await fs.rename(
 "old.txt",
 "new.txt"
);
```

Result:

```text
old.txt
     ↓
new.txt
```

---

# 21. Copy File

Example:

```js
await fs.copyFile(
 "source.txt",
 "backup.txt"
);
```

Result:

```text
source.txt
backup.txt
```

---

# 22. Move File

Move:

```js
await fs.rename(
 "./a.txt",
 "./archive/a.txt"
);
```

Internally:

```text
Rename Path
```

Often faster than copying.

---

# 23. Create Directory (mkdir)

Example:

```js
await fs.mkdir(
 "uploads"
);
```

Creates:

```text
uploads/
```

---

# 24. Nested Directories

Example:

```js
await fs.mkdir(
 "uploads/images/2026",
 {
   recursive:true
 }
);
```

Creates all levels.

---

Without recursive:

```text
ENOENT
```

Possible error.

---

# 25. Read Directory (readdir)

Example:

```js
const files =
 await fs.readdir(
   "./uploads"
 );
```

Output:

```js
[
 "a.txt",
 "b.txt"
]
```

---

Useful for:

```text
File Browsers
Uploads
Reports
```

---

# 26. Delete Directory

Old:

```js
fs.rmdir()
```

Deprecated.

---

Modern:

```js
await fs.rm(
 "./uploads",
 {
   recursive:true,
   force:true
 }
);
```

Deletes entire directory tree.

---

# 27. File Information (stat)

Example:

```js
const stats =
 await fs.stat(
   "test.txt"
 );
```

Returns:

```js
Stats
```

object.

---

# 28. File Size

Example:

```js
console.log(
 stats.size
);
```

Output:

```text
2048
```

bytes.

---

Convert:

```js
stats.size / 1024
```

KB.

---

# 29. File Permissions

Example:

```js
console.log(
 stats.mode
);
```

Output:

```text
33188
```

Unix permissions.

---

Change permissions:

```js
await fs.chmod(
 "file.txt",
 0o644
);
```

---

# 30. Created/Modified Dates

Example:

```js
console.log(
 stats.birthtime
);

console.log(
 stats.mtime
);
```

Output:

```text
Created Date
Modified Date
```

---

Important Properties

```text
birthtime
mtime
ctime
atime
```

---

# 31. Watching Files

Useful for:

```text
Hot Reload
Build Tools
Monitoring
```

---

# 32. fs.watch()

Example:

```js
fs.watch(
 "file.txt",
 (event,file)=>{

   console.log(
      event
   );

 });
```

---

Events:

```text
change
rename
```

---

# 33. File Change Detection

Example:

```js
fs.watch(
 "./config",
 (event,file)=>{

   console.log(
      `${file} changed`
   );

 });
```

Useful for:

```text
Config Reload
Log Monitoring
Auto Build Systems
```

---

# 34. File Descriptors

Very important interview topic.

When opening a file:

```js
const handle =
 await fs.open(
   "file.txt"
 );
```

Node receives:

```text
File Descriptor
```

Example:

```text
3
4
5
```

---

Operating System tracks file using descriptor.

---

Flow:

```text
File
 ↓
OS
 ↓
Descriptor
 ↓
Node
```

---

# 35. Open/Close Files

Open:

```js
const handle =
 await fs.open(
   "file.txt"
 );
```

Close:

```js
await handle.close();
```

Always close resources.

---

Bad:

```text
Open
Never Close
```

Can cause:

```text
Descriptor Leak
```

---

# 36. Large File Handling

Bad:

```js
await fs.readFile(
 "10GB.zip"
);
```

Loads entire file.

---

Memory:

```text
10GB RAM
```

Potential crash.

---

Correct:

```js
fs.createReadStream(
 "10GB.zip"
);
```

Flow:

```text
Chunk
Chunk
Chunk
```

Memory efficient.

---

# Stream Example

```js
const stream =
 fs.createReadStream(
   "big.mp4"
 );
```

Reads:

```text
64KB Chunks
```

by default.

---

# 37. Temporary Files

Common use:

```text
Uploads
Exports
Processing
Caching
```

Example:

```js
const os =
 require("os");

const tempDir =
 os.tmpdir();
```

Output:

```text
/tmp
```

or

```text
C:\Temp
```

---

Create temporary file:

```js
const path =
 `${os.tmpdir()}/file.tmp`;
```

---

# 38. Internal Architecture

Read File Flow:

```text
Application
      ↓
fs.readFile()
      ↓
Libuv
      ↓
Thread Pool
      ↓
OS
      ↓
Disk
      ↓
Callback
```

---

Important

File operations DO NOT run in Event Loop.

They run inside:

```text
Libuv Thread Pool
```

Default:

```text
4 Threads
```

Configurable:

```bash
UV_THREADPOOL_SIZE=8
```

---

# 39. Real World Examples

## Upload Service

```js
await fs.writeFile(
  filePath,
  buffer
);
```

---

## Backup System

```js
await fs.copyFile(
 source,
 backup
);
```

---

## Log Viewer

```js
const logs =
 await fs.readFile(
   "app.log",
   "utf8"
 );
```

---

## File Explorer

```js
const files =
 await fs.readdir(
   "./uploads"
 );
```

---

# 40. Interview Questions

## Basic

1. What is fs module?
2. Difference between readFile and readFileSync?
3. What is fs/promises?

---

## Intermediate

4. How does appendFile differ from writeFile?
5. How do you read JSON files?
6. How do you create nested directories?
7. Explain fs.watch().

---

## Advanced

8. What is a File Descriptor?
9. Why are file operations asynchronous?
10. How does fs use Libuv?
11. Why should large files use streams?
12. What happens internally during readFile?
13. What is UV_THREADPOOL_SIZE?
14. How would you handle millions of files?

---

# 41. Mastery Checklist

* fs Module Basics
* Sync vs Async APIs
* Callback APIs
* Promise APIs
* readFile
* readFileSync
* JSON Reading
* Encodings
* writeFile
* appendFile
* unlink
* rename
* copyFile
* mkdir
* readdir
* rm
* stat
* File Size
* Permissions
* Dates
* fs.watch
* File Descriptors
* Open/Close
* Streams
* Large Files
* Temporary Files
* Libuv Integration

After mastering File System, the next important topic is **Streams + File System Internals**, where you'll learn how Node.js efficiently processes GB-scale files using Readable Streams, Writable Streams, Backpressure, and Pipelines.
