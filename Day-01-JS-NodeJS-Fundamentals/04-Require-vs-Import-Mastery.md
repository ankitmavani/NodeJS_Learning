# Require vs Import Mastery (Node.js)

## Table of Contents

1. Introduction
2. Module System Basics
3. CommonJS (require)
4. module.exports
5. exports
6. Loading Modules
7. Module Caching
8. ES Modules (import)
9. Named Exports
10. Default Exports
11. Dynamic Imports
12. Import Assertions
13. CommonJS vs ES Modules
14. Synchronous vs Asynchronous Loading
15. Tree Shaking
16. Interoperability
17. Mixed Module Systems
18. Package.json "type": "module"
19. .mjs vs .cjs
20. Module Resolution Algorithm
21. Interview Questions
22. Mastery Checklist

---

# 1. Introduction

JavaScript originally had no built-in module system.

As applications grew, developers needed a way to:

* Split code into files
* Reuse code
* Avoid global variables
* Manage dependencies

Two major module systems emerged:

## CommonJS

```js
const fs = require("fs");
```

Used by Node.js historically.

---

## ES Modules (ESM)

```js
import fs from "fs";
```

Official JavaScript standard.

---

# 2. Module System Basics

A module is simply a file.

Example:

```js
// math.js

function add(a, b) {
  return a + b;
}
```

To use it elsewhere:

```js
// app.js

// import module
```

Benefits:

* Reusability
* Encapsulation
* Maintainability
* Dependency Management

---

# 3. CommonJS (require)

Node.js originally adopted CommonJS.

Import syntax:

```js
const fs = require("fs");
```

Custom module:

```js
const math = require("./math");
```

Characteristics:

* Synchronous loading
* Runtime resolution
* Node.js default for years

---

# How require() Works Internally

```js
const math = require("./math");
```

Node performs:

```text
1. Resolve path
2. Read file
3. Wrap code
4. Execute module
5. Store exports
6. Cache module
7. Return exports
```

Internal wrapper:

```js
(function(exports, require, module, __filename, __dirname) {

  // module code

});
```

This wrapper is automatically created by Node.

---

# 4. module.exports

Every CommonJS module exposes values using:

```js
module.exports
```

Example:

```js
function add(a, b) {
  return a + b;
}

module.exports = add;
```

Usage:

```js
const add = require("./math");

console.log(add(1, 2));
```

Output:

```text
3
```

---

# Export Multiple Values

```js
module.exports = {
  add,
  subtract,
  multiply
};
```

Usage:

```js
const math = require("./math");

math.add();
math.subtract();
```

---

# 5. exports

Node provides a shortcut:

```js
exports
```

Example:

```js
exports.add = add;
exports.subtract = subtract;
```

Equivalent to:

```js
module.exports.add = add;
module.exports.subtract = subtract;
```

---

# Important Interview Question

This works:

```js
exports.add = add;
```

This breaks:

```js
exports = add;
```

Why?

Because:

```js
exports
```

is merely a reference to:

```js
module.exports
```

When reassigned:

```js
exports = add;
```

the reference breaks.

Node still returns:

```js
module.exports
```

---

# 6. Loading Modules

Example:

```js
const math = require("./math");
```

Node follows:

```text
Resolve
Read
Compile
Execute
Cache
Return
```

Loading occurs only once.

---

# 7. Module Caching

Example:

```js
// counter.js

console.log("Loaded");

module.exports = {};
```

```js
require("./counter");
require("./counter");
require("./counter");
```

Output:

```text
Loaded
```

Only once.

Why?

Because Node caches modules.

Cache location:

```js
require.cache
```

Inspect:

```js
console.log(require.cache);
```

---

# Cache Benefits

* Faster loading
* Better performance
* Shared state

Example:

```js
let count = 0;

module.exports.increment = () => count++;
```

All imports share same instance.

---

# 8. ES Modules (import)

Modern JavaScript module system.

Import:

```js
import fs from "fs";
```

Custom module:

```js
import { add } from "./math.js";
```

Advantages:

* Standardized
* Static analysis
* Tree shaking
* Better optimization

---

# 9. Named Exports

Export:

```js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}
```

Import:

```js
import { add, subtract } from "./math.js";
```

---

# Alias Imports

```js
import { add as sum } from "./math.js";
```

Usage:

```js
sum(1, 2);
```

---

# 10. Default Exports

Export:

```js
export default function add(a, b) {
  return a + b;
}
```

Import:

```js
import add from "./math.js";
```

No braces required.

---

# Named vs Default Export

Named:

```js
export const name = "Ankit";
```

Import:

```js
import { name } from "./file.js";
```

Default:

```js
export default "Ankit";
```

Import:

```js
import userName from "./file.js";
```

---

# 11. Dynamic Imports

ESM supports runtime loading.

```js
const module = await import("./math.js");
```

Useful for:

* Lazy Loading
* Code Splitting
* Performance Optimization

Example:

```js
if (isAdmin) {
  const admin = await import("./admin.js");
}
```

---

# 12. Import Assertions

Used when importing non-JavaScript files.

Example:

```js
import config from "./config.json" with {
  type: "json"
};
```

Ensures correct module type.

---

# 13. CommonJS vs ES Modules

| Feature         | CommonJS       | ES Modules |
| --------------- | -------------- | ---------- |
| Syntax          | require        | import     |
| Export          | module.exports | export     |
| Loading         | Synchronous    | Static     |
| Tree Shaking    | No             | Yes        |
| Browser Support | No             | Yes        |
| Standard        | No             | Yes        |

---

# 14. Synchronous vs Asynchronous Loading

CommonJS:

```js
const math = require("./math");
```

Loads synchronously.

Execution pauses until module loads.

---

ES Modules:

```js
import { add } from "./math.js";
```

Statically analyzed before execution.

Enables better optimization.

---

# 15. Tree Shaking

Tree Shaking removes unused code.

Example:

```js
export function add(){}
export function subtract(){}
export function multiply(){}
```

Import:

```js
import { add } from "./math.js";
```

Bundler keeps:

```js
add()
```

Removes:

```js
subtract()
multiply()
```

CommonJS cannot do this reliably.

---

# 16. Interoperability

Import CommonJS into ESM:

```js
import express from "express";
```

Node creates synthetic default export.

---

Use ESM from CommonJS:

```js
async function load() {
  const module = await import("./math.js");
}
```

Dynamic import required.

---

# 17. Mixed Module Systems

Large projects often contain both:

```text
CommonJS
ES Modules
```

Example:

```text
legacy/
modern/
```

Migration strategy:

```text
CommonJS → ESM
```

gradually.

---

# 18. Package.json "type": "module"

Without:

```json
{
}
```

Default:

```text
CommonJS
```

---

Enable ESM:

```json
{
  "type": "module"
}
```

Now:

```js
import fs from "fs";
```

works.

---

# 19. .mjs vs .cjs

Explicit module type.

## .mjs

Always:

```text
ES Module
```

Example:

```text
app.mjs
```

---

## .cjs

Always:

```text
CommonJS
```

Example:

```text
app.cjs
```

---

Useful when both systems coexist.

---

# 20. Module Resolution Algorithm

Interview Favorite.

Example:

```js
require("express");
```

Node searches:

```text
1. Built-in Modules
2. node_modules
3. Parent node_modules
4. Continue upward
```

---

Example:

```js
require("./math");
```

Search order:

```text
math.js
math.json
math.node
math/index.js
```

---

ESM Resolution

```js
import "./math.js";
```

Requires:

```text
Exact Extension
```

Usually:

```js
import "./math.js";
```

not

```js
import "./math";
```

---

# CommonJS Resolution Flow

```text
require()
   │
   ▼
Resolve Filename
   │
   ▼
Check Cache
   │
   ▼
Load File
   │
   ▼
Compile
   │
   ▼
Execute
   │
   ▼
Cache
   │
   ▼
Return Exports
```

---

# ES Module Resolution Flow

```text
import
   │
   ▼
Parse Dependency Graph
   │
   ▼
Resolve Modules
   │
   ▼
Instantiate
   │
   ▼
Evaluate
```

---

# Interview Questions

### Basic

1. What is a module?
2. What is CommonJS?
3. What is ES Module?
4. Difference between exports and module.exports?

### Intermediate

5. How does require() work internally?
6. What is module caching?
7. What are named exports?
8. What are default exports?
9. What are dynamic imports?

### Advanced

10. Explain CommonJS vs ESM.
11. What is Tree Shaking?
12. How does Node resolve modules?
13. Explain module caching internals.
14. How does interoperability work?
15. .mjs vs .cjs?
16. Package.json type: module?
17. Explain Node's module resolution algorithm.

---

# Mastery Checklist

* Module System Basics
* CommonJS
* require()
* module.exports
* exports
* Module Loading
* Module Caching
* ES Modules
* Named Exports
* Default Exports
* Dynamic Imports
* Import Assertions
* CommonJS vs ESM
* Tree Shaking
* Interoperability
* Mixed Module Systems
* Package.json type: module
* .mjs vs .cjs
* Module Resolution Algorithm
* Production Best Practices

After mastering these topics, you'll understand exactly how Node.js loads, caches, resolves, and executes modules internally, which is essential for senior Node.js interviews and large-scale application architecture.
