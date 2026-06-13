# Call Stack Mastery (JavaScript & Node.js)

## Table of Contents

1. What is Call Stack?
2. Why Call Stack Exists
3. Stack Frames
4. Function Invocation Flow
5. Execution Context
6. Global Execution Context
7. Function Execution Context
8. Scope Chain
9. Lexical Environment
10. Stack Overflow Error
11. Recursive Calls
12. Memory Consumption
13. Call Stack vs Heap Memory
14. Debugging Call Stack
15. Event Loop and Call Stack
16. Interview Questions
17. Mastery Checklist

---

# 1. What is Call Stack?

The Call Stack is a data structure used by the JavaScript engine to keep track of function execution.

It follows the **LIFO (Last In, First Out)** principle.

## Stack Example

```text
Top
┌───────┐
│ Plate3│ ← Removed First
├───────┤
│ Plate2│
├───────┤
│ Plate1│
└───────┘
Bottom
```

JavaScript works exactly the same way.

Example:

```js
function greet() {
    console.log("Hello");
}

greet();
```

Execution:

```text
Call Stack

┌─────────┐
│ greet() │
├─────────┤
│ Global  │
└─────────┘
```

After execution:

```text
Call Stack

┌─────────┐
│ Global  │
└─────────┘
```

---

# 2. Why Call Stack Exists

Without a Call Stack:

```js
function a() {
    b();
}

function b() {
    c();
}

function c() {
    console.log("Hello");
}
```

JavaScript would not know:

* Which function is currently executing
* Where to return after a function completes
* Which variables belong to which function

The Call Stack manages all of this automatically.

---

# 3. Stack Frames

Every function call creates a Stack Frame.

A Stack Frame contains:

```text
Function Name
Arguments
Local Variables
Return Address
Execution Context
```

Example:

```js
function add(a, b) {
    let sum = a + b;
    return sum;
}

add(10, 20);
```

Stack Frame:

```text
┌──────────────────┐
│ Function: add    │
│ a = 10           │
│ b = 20           │
│ sum = 30         │
│ return address   │
└──────────────────┘
```

---

# 4. Function Invocation Flow

Example:

```js
function one() {
    two();
}

function two() {
    three();
}

function three() {
    console.log("Done");
}

one();
```

## Step 1

```text
Stack

┌──────────┐
│ Global   │
└──────────┘
```

## Step 2

```text
Stack

┌──────────┐
│ one()    │
├──────────┤
│ Global   │
└──────────┘
```

## Step 3

```text
Stack

┌──────────┐
│ two()    │
├──────────┤
│ one()    │
├──────────┤
│ Global   │
└──────────┘
```

## Step 4

```text
Stack

┌──────────┐
│ three()  │
├──────────┤
│ two()    │
├──────────┤
│ one()    │
├──────────┤
│ Global   │
└──────────┘
```

## Step 5

```text
Stack

┌──────────┐
│ two()    │
├──────────┤
│ one()    │
├──────────┤
│ Global   │
└──────────┘
```

## Step 6

```text
Stack

┌──────────┐
│ Global   │
└──────────┘
```

---

# 5. Execution Context

Every function execution creates an Execution Context.

Execution Context is the environment where JavaScript code executes.

Contains:

```text
Variables
Functions
this
Scope Information
```

Every Stack Frame contains an Execution Context.

```text
Call Stack
     │
     ▼
Execution Context
```

---

# 6. Global Execution Context (GEC)

Created automatically when JavaScript starts.

Example:

```js
let name = "Ankit";

function greet() {}

console.log(name);
```

Global Execution Context contains:

```text
name
greet()
this
```

## Memory Creation Phase

```js
var a = 10;

function test() {}
```

Before execution:

```text
a → undefined

test → function
```

## Execution Phase

```text
a → 10
```

---

# 7. Function Execution Context

Whenever a function executes:

```js
function greet(name) {
    let age = 25;
}

greet("Ankit");
```

A new Function Execution Context is created.

Contains:

```text
Arguments
Variables
this
Outer Scope Reference
```

## Creation Phase

```text
name → undefined
age → undefined
```

## Execution Phase

```text
name → "Ankit"
age → 25
```

## Destruction Phase

```text
Execution Context Removed
Stack Frame Popped
```

---

# 8. Scope Chain

The Scope Chain helps JavaScript locate variables.

Example:

```js
let a = 10;

function outer() {

    let b = 20;

    function inner() {

        let c = 30;

        console.log(a);
        console.log(b);
        console.log(c);
    }

    inner();
}
```

Search Order:

```text
inner scope
     ↓
outer scope
     ↓
global scope
```

Output:

```text
10
20
30
```

---

# 9. Lexical Environment

Lexical means:

```text
Where code is physically written
```

Example:

```js
let a = 10;

function outer() {

    let b = 20;

    function inner() {
        console.log(a, b);
    }

    inner();
}
```

JavaScript remembers:

```text
inner was created inside outer
```

Not:

```text
where inner was called
```

This behavior is called Lexical Scoping.

---

## Lexical Environment Structure

```text
Lexical Environment
     │
     ├── Environment Record
     │      Variables
     │      Functions
     │
     └── Outer Reference
```

Example:

```js
let a = 10;

function test() {
    let b = 20;
}
```

Global Environment:

```text
a = 10
test()
```

Function Environment:

```text
b = 20

Outer → Global
```

---

# 10. Stack Overflow Error

Example:

```js
function run() {
    run();
}

run();
```

Result:

```text
RangeError:
Maximum call stack size exceeded
```

Reason:

```text
run()
run()
run()
run()
...
```

Stack grows indefinitely until memory is exhausted.

---

# 11. Recursive Calls

Recursion means a function calling itself.

Bad Example:

```js
function count() {
    count();
}
```

Infinite recursion.

Good Example:

```js
function count(n) {

    if (n === 0)
        return;

    console.log(n);

    count(n - 1);
}

count(5);
```

Output:

```text
5
4
3
2
1
```

## Recursive Call Stack

```text
count(3)
count(2)
count(1)
count(0)
```

Then unwinds:

```text
count(0) returns

count(1) returns

count(2) returns

count(3) returns
```

---

# 12. Memory Consumption

Each Stack Frame consumes memory.

Contains:

```text
Arguments
Variables
Closures
Return Address
Execution Context
```

Example:

```js
function heavy() {
    let arr = new Array(1000000);
}
```

Large memory usage.

Many stack frames can significantly increase memory consumption.

---

# 13. Call Stack vs Heap Memory

| Call Stack                | Heap           |
| ------------------------- | -------------- |
| Stores execution contexts | Stores objects |
| Fast access               | Slower access  |
| Fixed size                | Dynamic size   |
| LIFO                      | Random access  |

Example:

```js
let user = {
    name: "Ankit"
};
```

Stack:

```text
user → reference
```

Heap:

```text
{
  name: "Ankit"
}
```

---

# 14. Debugging Call Stack

Example:

```js
function one() {
    two();
}

function two() {
    three();
}

function three() {
    console.trace();
}

one();
```

Output:

```text
three
two
one
global
```

Displays the current call stack.

---

## Error Stack Trace

```js
function one() {
    two();
}

function two() {
    throw new Error("Failed");
}

one();
```

Output:

```text
Error: Failed

at two
at one
at global
```

This is the call stack at the moment of failure.

---

# 15. Event Loop and Call Stack

The Event Loop only executes callbacks when:

```text
Call Stack Empty
```

Example:

```js
setTimeout(() => {
    console.log("Timer");
}, 0);

while(true){}
```

Output:

```text
Nothing
```

Reason:

```text
Call Stack Never Empty
```

The Event Loop cannot move callbacks onto the stack.

---

# 16. Interview Questions

## Basic

1. What is a Call Stack?
2. What is a Stack Frame?
3. What is LIFO?
4. How does JavaScript execute nested functions?

## Intermediate

5. What is Execution Context?
6. Difference between Global and Function Execution Context?
7. What is Scope Chain?
8. What is Lexical Environment?
9. What is Lexical Scoping?

## Advanced

10. Why does Stack Overflow occur?
11. Explain recursion using the Call Stack.
12. How does Event Loop interact with the Call Stack?
13. Why does `while(true){}` block Node.js?
14. Explain memory consumption in the Call Stack.
15. How would you debug Call Stack issues?

---

# 17. Mastery Checklist

* Call Stack
* LIFO Principle
* Stack Frames
* Function Invocation Flow
* Execution Context
* Global Execution Context
* Function Execution Context
* Scope Chain
* Lexical Environment
* Lexical Scoping
* Recursion
* Stack Overflow
* Memory Consumption
* Heap vs Stack
* Error Stack Traces
* Debugging Techniques
* Event Loop + Call Stack Relationship

After mastering these topics, you'll have the foundation required for:

* Closures
* Hoisting
* Async/Await
* Promises
* Event Loop Internals
* React Hooks Internals
* Node.js Runtime Internals
