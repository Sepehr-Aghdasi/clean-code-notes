# Naming - Clean Code

## Overview

Naming things (variables, properties, functions, methods, classes) correctly and in an understandable way is an extremely important part of writing clean code.

**Indeed - if poor names are chosen - pretty much all other concepts taught throughout the course will not help that much.**

## Be Descriptive

Names have one simple purpose: They should describe what's stored in a variable or property or what a function or method does. Or what kind of object will be created when instantiating a class.

If you keep that in mind, coming up with good names should actually be straightforward - though coming up with the best name for a given variable/property/function will still require some practice and often multiple iterations. That's normal though - clean code is written by iterating and improving code over time!

---

## Naming Rules

### Variables & Properties

Variables and properties hold data - numbers, text (strings), boolean values, objects, lists, arrays, maps etc.

Hence the name should imply which kind of data is being stored.

**Rules:**
- Typically receive a **noun** as a name
  - Example: `user`, `product`, `customer`, `database`, `transaction`
- Alternatively, use a short phrase with an **adjective** - typically for storing boolean values
  - Example: `isValid`, `didAuthenticate`, `isLoggedIn`, `emailExists`

**Tip:** If you can be more specific, you should be more specific.
- Prefer `customer` over `user` if the code is doing customer-specific operations

---

### Functions & Methods

Functions and methods can be called to execute code. That means they perform tasks and operations.

**Rules:**
- Typically receive a **verb** as a name
  - Example: `login()`, `createUser()`, `database.insert()`, `log()`
- For producing values (especially booleans), use short phrases with adjectives
  - Example: `isValid(...)`, `isEmail(...)`, `isEmpty(...)`

**Avoid:**
- Names like `email()`, `user()` - these sound like properties
- Prefer `getEmail()` instead

**Tip:** If you can be more specific, it makes sense to use such more specific names
- Example: `createUser()` instead of just `create()`

---

### Classes

Classes are used to create objects (unless it's a static class).

**Rules:**
- The class name should describe the kind of object it will create
- Even if it's a static class, describe what container it represents
- Good class names are **nouns**
  - Example: `User`, `Product`, `RootAdministrator`, `Transaction`, `Payment`

---

## Avoid Generic Names

In most situations, you should avoid generic names like:
- `handle()`
- `process()`
- `data`
- `item`

**Exception:** There can be situations where it makes sense, but typically:
- Make names more specific (e.g. `processTransaction()`)
- Or go for a different kind of name (e.g. `product` instead of `item`)

---

## Be Consistent

An important part of using proper names is consistency.

**Example:**
- If you used `fetchUsers()` in one part of your code, you should also use `fetchProducts()` - and not `getProducts()` - in another part of that same code

**General principle:** It doesn't matter if you prefer `fetch...()`, `get...()`, `retrieve...()` or any other term - but you should be consistent!

---

## Summary Checklist

| Element | Name Type | Examples |
|---------|-----------|----------|
| Variables & Properties | Nouns or short phrases with adjectives | `user`, `isValid` |
| Functions & Methods | Verbs or short phrases with adjectives | `login()`, `isEmail()` |
| Classes | Nouns | `User`, `Transaction` |

**Key Principles:**
- Be as specific as necessary and possible
- Use yes/no "questions" for booleans (e.g. `isValid`)
- Avoid misleading names
- Be consistent with your names
- Avoid generic names like `data`, `item`, `process()`