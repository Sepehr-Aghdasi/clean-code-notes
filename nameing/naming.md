# Part 1: Naming

Clean code is not about strict rules you must follow every time. It is about writing code that is easy to read, easy to understand, and easy to maintain.

The name of a variable, function, or class should tell you what it does. You should not need to open the function and read the code to understand it.

If a function does something, the name should describe the action:

```ts
function getUserByEmail(email: string): User {
  // ...
}
```

If a function returns true or false, the name should ask a question:

```ts
function emailIsValid(email: string): boolean {
  return email.includes('@');
}
```

A class name should describe the object:

```ts
const product = new Product();
```

Some simple rules I am trying to follow now:

Do not add extra information to a name. "user" is better than "userWithNameAndAge". If you need a long name to explain a variable, the name is not doing its job.

Do not use slang or unclear short forms. Use "remove", not something unclear like "diePlease".

Use different names for different functions. "getDayData" is not clear. "getDataForToday" is better.

Stay consistent. If you use "getUsers" in one place, do not use "fetchUsers" in another place for the same kind of action.