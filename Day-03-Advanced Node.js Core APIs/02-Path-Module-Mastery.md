# Path Module (path) Mastery Guide (Node.js)

## Table of Contents

1. What is the Path Module?
2. Why Path Module Exists
3. Windows vs Linux Paths
4. Importing Path Module
5. path.join()
6. path.resolve()
7. path.normalize()
8. path.parse()
9. path.format()
10. path.basename()
11. path.dirname()
12. path.extname()
13. Relative Paths
14. Absolute Paths
15. Current Working Directory
16. __dirname vs process.cwd()
17. Cross-Platform Path Handling
18. URL Paths vs File Paths
19. Path Security Considerations
20. Internal Implementation
21. Real World Examples
22. Interview Questions
23. Mastery Checklist

---

# What is the Path Module?

The path module is a built-in Node.js module that provides utilities for working with file and directory paths.

```js
const path = require("path");
```

It helps:
- Build paths safely
- Extract file information
- Normalize paths
- Resolve relative paths
- Support Windows and Linux consistently

---

# Why Path Module Exists

Operating systems use different path separators.

Windows:

```text
C:\Users\John\project\file.txt
```

Linux/macOS:

```text
/home/john/project/file.txt
```

Hardcoding paths can break applications.

Bad:

```js
const file = "uploads/" + fileName;
```

Good:

```js
const file = path.join("uploads", fileName);
```

---

# Windows vs Linux Paths

Windows separator:

```text
\
```

Linux/macOS separator:

```text
/
```

Node automatically handles these differences using path methods.

```js
path.join("images", "profile.jpg");
```

Windows:

```text
images\profile.jpg
```

Linux:

```text
images/profile.jpg
```

---

# Importing Path Module

```js
const path = require("path");
```

No npm installation required.

---

# path.join()

Combines path segments safely.

```js
path.join("uploads", "images", "photo.jpg");
```

Result:

```text
uploads/images/photo.jpg
```

Automatically removes duplicate separators.

```js
path.join("uploads/", "/images/", "photo.jpg");
```

Output:

```text
uploads/images/photo.jpg
```

---

# path.resolve()

Creates an absolute path.

```js
path.resolve("uploads", "photo.jpg");
```

Assume current directory:

```text
/project
```

Output:

```text
/project/uploads/photo.jpg
```

Difference:

```js
path.join("a", "b");
```

Result:

```text
a/b
```

```js
path.resolve("a", "b");
```

Result:

```text
/project/a/b
```

---

# path.normalize()

Cleans invalid or messy paths.

```js
path.normalize("/users//images///photo.jpg");
```

Output:

```text
/users/images/photo.jpg
```

Also resolves:

```text
.
..
```

Example:

```js
path.normalize("/users/admin/../images");
```

Output:

```text
/users/images
```

---

# path.parse()

Breaks a path into components.

```js
const info = path.parse("/users/images/photo.jpg");
console.log(info);
```

Output:

```js
{
  root: '/',
  dir: '/users/images',
  base: 'photo.jpg',
  ext: '.jpg',
  name: 'photo'
}
```

---

# path.format()

Reverse operation of parse().

```js
path.format({
  dir: "/users/images",
  name: "photo",
  ext: ".jpg"
});
```

Output:

```text
/users/images/photo.jpg
```

---

# path.basename()

Returns the filename.

```js
path.basename("/users/images/photo.jpg");
```

Output:

```text
photo.jpg
```

Remove extension:

```js
path.basename("/users/images/photo.jpg", ".jpg");
```

Output:

```text
photo
```

---

# path.dirname()

Returns directory portion.

```js
path.dirname("/users/images/photo.jpg");
```

Output:

```text
/users/images
```

---

# path.extname()

Returns file extension.

```js
path.extname("photo.jpg");
```

Output:

```text
.jpg
```

Useful for validation and upload systems.

---

# Relative Paths

Relative paths depend on current location.

```text
./uploads/photo.jpg
```

```text
../config.json
```

Examples:

```js
fs.readFile("./config.json");
```

Resolved relative to process.cwd().

---

# Absolute Paths

Absolute paths start from filesystem root.

Linux:

```text
/home/user/project/file.txt
```

Windows:

```text
C:\Users\User\project\file.txt
```

Safer in production applications.

```js
path.resolve(__dirname, "config.json");
```

---

# Current Working Directory

```js
process.cwd();
```

Returns:

```text
Directory where Node process started
```

Example:

```bash
node app.js
```

from:

```text
/home/project
```

Output:

```text
/ home / project
```

---

# __dirname vs process.cwd()

__dirname:

```js
console.log(__dirname);
```

Returns location of current file.

process.cwd():

```js
console.log(process.cwd());
```

Returns location where process started.

| Feature | __dirname | process.cwd() |
|----------|------------|------------|
| Current File Location | Yes | No |
| Process Start Directory | No | Yes |
| Runtime Changeable | No | Yes |

---

# Cross-Platform Path Handling

Never hardcode separators.

Bad:

```js
"uploads\\images\\file.jpg"
```

Good:

```js
path.join("uploads", "images", "file.jpg");
```

Node provides:

```js
path.win32
path.posix
```

Example:

```js
path.win32.basename("C:\\test\\file.txt");
```

---

# URL Paths vs File Paths

URL:

```text
https://example.com/users/1
```

File Path:

```text
/home/project/file.txt
```

Path module is for filesystem paths.

Do NOT use:

```js
path.join("https://site.com", "users");
```

Use:

```js
new URL("/users", "https://site.com");
```

---

# Path Security Considerations

Directory Traversal Attack:

Input:

```text
../../../etc/passwd
```

Bad:

```js
const file = path.join(uploadDir, userInput);
```

Attacker can escape upload directory.

Safe:

```js
const filePath = path.resolve(uploadDir, userInput);

if (!filePath.startsWith(uploadDir)) {
  throw new Error("Invalid path");
}
```

---

# Internal Implementation

Path module:

```text
Pure JavaScript
```

Does not access disk.

Flow:

```text
Input Path
    ↓
Normalization
    ↓
OS-Specific Handling
    ↓
Output Path
```

Very fast because no filesystem calls occur.

---

# Real World Examples

## Upload System

```js
const uploadPath = path.join(
  __dirname,
  "uploads",
  fileName
);
```

## Static Files

```js
const publicDir = path.resolve(
  __dirname,
  "public"
);
```

## Extension Validation

```js
if (path.extname(file) !== ".jpg") {
  throw new Error("Only JPG allowed");
}
```

## Backup System

```js
const backup = path.join(
  "backup",
  path.basename(file)
);
```

---

# Interview Questions

## Basic

1. What is the path module?
2. Why is path.join() useful?
3. Difference between join() and resolve()?

## Intermediate

4. What does normalize() do?
5. How does parse() work?
6. basename() vs dirname()?
7. extname() use cases?

## Advanced

8. __dirname vs process.cwd()?
9. How does Node support cross-platform paths?
10. What is directory traversal?
11. How do you secure file paths?
12. URL paths vs file paths?
13. Why is resolve() preferred in production?

---

# Mastery Checklist

- Path Module Basics
- Windows vs Linux Paths
- path.join()
- path.resolve()
- path.normalize()
- path.parse()
- path.format()
- path.basename()
- path.dirname()
- path.extname()
- Relative Paths
- Absolute Paths
- process.cwd()
- __dirname
- Cross-Platform Support
- URL vs File Paths
- Directory Traversal Protection
- Path Security
- Real World Usage
