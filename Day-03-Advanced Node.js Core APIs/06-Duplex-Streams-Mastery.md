
# Duplex Streams Mastery Guide (Node.js)

## Table of Contents

1. What are Duplex Streams?
2. Why Duplex Streams Exist
3. Read and Write Capability
4. Bidirectional Data Flow
5. Duplex Stream Architecture
6. TCP Socket Example
7. WebSocket Example
8. Network Communication
9. Duplex Stream Lifecycle
10. Duplex Events
11. Custom Duplex Streams
12. Internal Architecture
13. Real World Examples
14. Interview Questions
15. Mastery Checklist

---

# 1. What are Duplex Streams?

A Duplex Stream is a stream that is both:

- Readable
- Writable

at the same time.

Unlike:

```text
Readable Stream
→ Only Read
```

and

```text
Writable Stream
→ Only Write
```

Duplex Streams can do both simultaneously.

---

# 2. Why Duplex Streams Exist

Many systems need two-way communication.

Examples:

- TCP Connections
- WebSockets
- Chat Applications
- Database Connections
- SSH Sessions

Flow:

```text
Client
↓ ↑
Server
```

Data moves in both directions.

---

# 3. Read and Write Capability

Duplex streams inherit:

```text
Readable
+
Writable
```

Capabilities:

```js
stream.write("Hello");

stream.on(
  "data",
  chunk => {
    console.log(chunk);
  }
);
```

Same object can:

- Receive Data
- Send Data

---

# 4. Bidirectional Data Flow

Traditional Readable Stream:

```text
Source
↓
Application
```

Traditional Writable Stream:

```text
Application
↓
Destination
```

Duplex Stream:

```text
Source
↓ ↑
Application
↓ ↑
Destination
```

Both directions are active.

---

# Full Duplex Communication

```text
Client
 ↓ ↑
Socket
 ↓ ↑
Server
```

Read and write happen independently.

---

# 5. Duplex Stream Architecture

Internally:

```text
Readable Side
       ↑
       |
Duplex Stream
       |
       ↓
Writable Side
```

Both sides maintain:

```text
Separate Buffers
Separate States
Separate Queues
```

---

# 6. TCP Socket Example

Most important Duplex Stream example.

Node TCP Server:

```js
const net = require("net");

const server =
 net.createServer(
   socket => {

     socket.write(
       "Welcome"
     );

     socket.on(
       "data",
       data => {

         console.log(
           data.toString()
         );

       }
     );

   }
 );
```

---

Socket can:

```text
Read Incoming Data
```

and

```text
Write Outgoing Data
```

simultaneously.

---

# TCP Flow

```text
Client
 ↓
Socket.read()
 ↓
Server

Server
 ↓
Socket.write()
 ↓
Client
```

---

# 7. WebSocket Example

WebSockets are Duplex Streams conceptually.

Client:

```js
socket.send(
  "Hello"
);
```

Server:

```js
socket.send(
  "Hi"
);
```

Both can send messages anytime.

---

Flow:

```text
Browser
 ↓ ↑
WebSocket
 ↓ ↑
Server
```

---

# Why WebSockets Need Duplex

Chat Application:

```text
User A
↓ ↑
Server
↓ ↑
User B
```

Messages flow both ways.

---

# 8. Network Communication

Most network communication is duplex.

Examples:

```text
HTTP/2
TCP
SSH
FTP
Database Connections
```

---

Database Example

Application:

```text
Query
↓
Database
```

Database:

```text
Results
↓
Application
```

Both directions required.

---

# 9. Duplex Stream Lifecycle

Lifecycle:

```text
Create
↓
Open
↓
Read
↓
Write
↓
Read
↓
Write
↓
End
↓
Close
```

Unlike Readable Streams:

```text
Read
↓
End
```

Duplex streams continuously perform both.

---

# Example Sequence

```text
connect
↓
data
↓
write
↓
data
↓
write
↓
end
↓
close
```

---

# 10. Duplex Events

Because Duplex extends Readable and Writable, it supports:

Readable Events:

```text
data
readable
end
```

Writable Events:

```text
finish
drain
```

Common Events:

```text
close
error
```

---

# data Event

```js
stream.on(
 "data",
 chunk => {}
);
```

Triggered when new data arrives.

---

# finish Event

```js
stream.on(
 "finish",
 () => {}
);
```

Triggered after writes complete.

---

# drain Event

```js
stream.on(
 "drain",
 () => {}
);
```

Triggered when buffer becomes available again.

---

# error Event

```js
stream.on(
 "error",
 err => {}
);
```

Always handle errors.

---

# 11. Custom Duplex Streams

Node provides:

```js
const {
 Duplex
} = require(
 "stream"
);
```

Create:

```js
class MyDuplex
 extends Duplex {

   _read(){

   }

   _write(
     chunk,
     encoding,
     callback
   ){

     callback();

   }

 }
```

---

# Required Methods

```text
_read()
_write()
```

Readable side:

```js
_read(size)
```

Writable side:

```js
_write(
 chunk,
 encoding,
 callback
)
```

---

# Example Echo Stream

```js
const {
 Duplex
} = require(
 "stream"
);

class EchoStream
 extends Duplex {

 _read(){}

 _write(
   chunk,
   enc,
   cb
 ){

   this.push(
     chunk
   );

   cb();

 }

}
```

---

Usage

```js
const stream =
 new EchoStream();

stream.on(
 "data",
 d => {

   console.log(
     d.toString()
   );

 }
);

stream.write(
 "Hello"
);
```

Output:

```text
Hello
```

---

# 12. Internal Architecture

Duplex contains:

```text
Readable Buffer
```

and

```text
Writable Buffer
```

simultaneously.

---

Flow

```text
Incoming Data
↓
Readable Buffer
↓
Application

Application
↓
Writable Buffer
↓
Outgoing Data
```

---

Internal Components

```text
Readable State
Writable State
Read Queue
Write Queue
Backpressure Logic
```

---

# Backpressure in Duplex Streams

Read side:

```text
pause()
resume()
```

Write side:

```text
drain
```

Both operate independently.

---

# 13. Real World Examples

## TCP Server

```js
net.createServer();
```

Returns duplex sockets.

---

## WebSockets

```js
ws
```

Library uses duplex communication.

---

## SSH Connections

```text
Commands
↓ ↑
Responses
```

---

## Database Drivers

```text
Send Query
↓ ↑
Receive Results
```

---

## Chat Applications

```text
User A
↓ ↑
Server
↓ ↑
User B
```

---

# 14. Interview Questions

## Basic

1. What is a Duplex Stream?
2. Difference between Readable and Duplex?
3. Difference between Writable and Duplex?

---

## Intermediate

4. Why are TCP sockets duplex?
5. Why are WebSockets duplex?
6. Explain bidirectional communication.
7. Which events are available on Duplex streams?

---

## Advanced

8. How are Duplex streams implemented internally?
9. Why do Duplex streams need separate buffers?
10. Explain backpressure in Duplex streams.
11. How do you create a custom Duplex stream?
12. Difference between Duplex and Transform streams?
13. Explain _read() and _write().

---

# 15. Mastery Checklist

- Duplex Streams
- Read and Write Capability
- Bidirectional Communication
- TCP Sockets
- WebSockets
- Network Communication
- Duplex Lifecycle
- Duplex Events
- data Event
- finish Event
- drain Event
- error Event
- Custom Duplex Streams
- _read()
- _write()
- Internal Architecture
- Backpressure
- Real World Usage
- Interview Preparation
