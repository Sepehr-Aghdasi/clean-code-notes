# Control Structures & Errors

No matter which kind of application you're building - you will most likely also use control structures in your code: `if` statements, `for` loops, maybe also `while` loops or `switch-case` statements.

Control structures are extremely important to control code flow, and of course you should use them.

But control structures can also lead to bad or poor code, so they are important for writing clean code.

There are three main areas of improvement, you should be aware of:

1. Prefer positive checks
2. Avoid deep nesting
3. Handle errors

---

## Prefer Positive Checks

This is a simple one. It can make sense to use positive wording in your `if` checks instead of negative wording.

Though - in my opinion at least - sometimes a short negative phrase is better than a constructed positive one.

Consider this example:

```javascript
if (isEmpty(blogContent)) {
  // throw error
}

if (!hasContent(blogContent)) {
  // throw error
}
```

The first snippet is quite readable and requires zero thinking.

The second snippet uses the `!` operator to check for the opposite - slightly more thinking and understanding is required from the reader.

Hence option #1 is preferable.

However, sometimes, I do prefer the negative version:

```javascript
if (!isOpen(transaction)) {
  // throw error
}

if (isClosed(transaction)) {
  // throw error
}
```

On first look, it looks like option #2 is better.

And it generally might be. But what if we didn't just have 'Open' and 'Closed' transactions? What if we also had 'Unknown'?

```javascript
if (!isOpen(transaction)) {
  // throw error
}

if (isClosed(transaction) || isUnknown(transaction)) {
  // throw error
}
```

This quickly adds up! The more possible options we have, the more checks we need to combine.

Or we simply check for the opposite - in this example, simply for the transaction NOT being open.

---

## Avoid Deep Nesting

This is very important! You should absolutely avoid deeply nested control structures since such code is highly unreadable, hard to maintain, and also likely to have errors.

There are a couple of techniques that can help you with getting rid of deeply nested control structures and code duplication:

1. Use guards and fail fast
2. Extract control structures and logic into separate functions
3. Polymorphism & Factory Functions
4. Replace if checks with errors

---

### Use Guards & Fail Fast

Guards are a great concept! Often, you can extract a nested `if` check and move it right to the start of a function to fail fast if some condition is (not) met and only continue with the rest of the code otherwise.

Here's an example without a guard:

```javascript
function messageUser(user, message) {
  if (user) {
    if (message) {
      if (user.acceptsMessages) {
        const success = user.sendMessage(message);
        if (success) {
          console.log('Message sent!');
        }
      }
    }
  }
}
```

Here's the improved version, using a guard and failing fast:

```javascript
function messageUser(user, message) {
  if (!user || !message || !user.acceptsMessages) {
    return;
  }

  user.sendMessage(message);

  if (success) {
    console.log('Message sent!');
  }
}
```

By combining the conditions, we can replace three if checks with one. The function stops if any condition fails.

Guards are these `if` checks right at the start of your functions.

Take your nested checks, reverse the logic, and check for the problem first. Then return or throw an error so the rest of the function does not run.

---

### Extract Control Structures & Logic Into New Functions

We already learned that splitting functions and keeping functions small is important. Applying this knowledge is always great, it also helps with removing deeply nested control structures.

Consider this example:

```javascript
function connectDatabase(uri) {
  if (!uri) {
    throw new Error('An URI is required!');
  }

  const db = new Database(uri);
  let success = db.connect();

  if (!success) {
    if (db.fallbackConnection) {
      return db.fallbackConnectionDetails;
    } else {
      throw new Error('Could not connect!');
    }
  }

  return db.connectionDetails;
}
```

This code could be improved by applying what we learned about functions:

```javascript
function connectDatabase(uri) {
  validateUri(uri);
  const db = new Database(uri);
  let success = db.connect();
  let connectionDetails;

  if (success) {
    connectionDetails = db.connectionDetails;
  } else {
    connectionDetails = connectFallbackDatabase(db);
  }

  return connectionDetails;
}

function validateUri(uri) {
  if (!uri) {
    throw new Error('An URI is required!');
  }
}

function connectFallbackDatabase(db) {
  if (db.fallbackConnection) {
    return db.fallbackConnectionDetails;
  } else {
    throw new Error('Could not connect!');
  }
}
```

You might be able to optimize this code even more, but you can already see that the nested control structure was removed by extracting a separate `connectFallbackDatabase()` function.

---

### Polymorphism & Factory Functions

Sometimes, you end up with duplicated `if` statements and duplicated checks just because the code inside of these statements differs slightly.

In such cases, polymorphism and factory functions can help you.

Before we dive into these concepts, have a look at this example:

```javascript
function processTransaction(transaction) {
  if (isPayment(transaction)) {
    if (usesCreditCard(transaction)) {
      processCreditCardPayment(transaction);
    }
    if (usesPayPal(transaction)) {
      processPayPalPayment(transaction);
    }
  } else {
    if (usesCreditCard(transaction)) {
      processCreditCardRefund(transaction);
    }
    if (usesPayPal(transaction)) {
      processPayPalRefund(transaction);
    }
  }
}
```

In this example, we repeat the `usesCreditCard()` and `usesPayPal()` checks because we run different code depending on whether we have payment or refund.

We can solve this by writing a factory function which returns a polymorphic object:

```javascript
function getProcessors(transaction) {
  let processors = {
    processPayment: null,
    processRefund: null
  };

  if (usesCreditCard(transaction)) {
    processors.processPayment = processCreditCardPayment;
    processors.processRefund = processCreditCardRefund;
  }

  if (usesPayPal(transaction)) {
    processors.processPayment = processPayPalPayment;
    processors.processRefund = processPayPalRefund;
  }

  return processors;
}

function processTransaction(transaction) {
  const processors = getProcessors(transaction);

  if (isPayment(transaction)) {
    processors.processPayment(transaction);
  } else {
    processors.processRefund(transaction);
  }
}
```

The repeated checks for whether a credit card or PayPal was used was now outsourced into the `getProcessors()` function which now only runs these checks once (instead of twice, as before).

`getProcessors()` is a factory function. It creates and returns objects.

The function returns an object with two functions: `processCreditCardPayment` and `processCreditCardRefund`. These functions are not called yet because there are no `()` after their names.

The returned object is polymorphic because we use it in the same way by calling `processPayment()` and `processRefund()`, but the code that runs can be different.

**Note:** The same problem can also be solved by using classes and inheritance.

---

## Handle Errors

Errors are another nice way of getting rid of redundant if checks. They let us use the language's built-in features to handle problems where they should be handled.

Consider this example:

```javascript
function createUser(email, password) {
  const inputValidity = validateInput(email, password);

  if (inputValidity.code === 1 || inputValidity === 2) {
    console.log(inputValidity.message);
    return;
  }

  // ... continue
}

function validateInput(email, password) {
  if (!email.includes('@') || password.length < 7) {
    return { code: 1, message: 'Invalid input' };
  }

  const existingUser = findUserByEmail(email);

  if (existingUser) {
    return { code: 2, message: 'Email is already in use!' };
  }
}
```

Here, the `validateInput()` function does not directly log to the console and also not just return `true` or `false`. Instead, it returns an object / map with more information about the validation result. This is a common situation, but it could be implemented better.

In the end, the example code produces a custom error. But because it's custom, we can't handle it with normal error handling tools. Instead, `if` checks are used.

Here's a better version - using built-in error support which pretty much all programming languages offer:

```javascript
function createUser(email, password) {
  try {
    validateInput(email, password);
  } catch (error) {
    console.log(error.message);
  }

  // ... continue
}

function validateInput(email, password) {
  if (!email.includes('@') || password.length < 7) {
    throw new Error('Input is invalid!');
  }

  const existingUser = findUserByEmail(email);

  if (existingUser) {
    throw new Error('Email is already taken!');
  }
}
```

`throw` is a keyword in JavaScript (and many other languages) which can be used to generate an error.

When an error occurs, it moves up the functions and stops the code until it is handled with `try-catch`.

This removes the need for extra `if` checks and `return` statements.

And we could even move the entire error handling logic out of the `createUser()` function.

```javascript
function handleSignupRequest(request) {
  try {
    createUser(request.email, request.password);
  } catch (error) {
    console.log(error.message);
  }
}

function createUser(email, password) {
  validateInput(email, password);
  // ... continue
}

function validateInput(email, password) {
  if (!email.includes('@') || password.length < 7) {
    throw new Error('Input is invalid!');
  }

  const existingUser = findUserByEmail(email);

  if (existingUser) {
    throw new Error('Email is already taken!');
  }
}
```

Indeed, error handling should typically be considered to be "one thing" (remember: functions should do one thing), so moving it up into a separate function is a good idea.

---

## Summary Checklist

- ✅ Prefer positive checks
- ✅ Avoid deep nesting
- ✅ Consider using "Guard" statements
- ✅ Consider using polymorphism and factory functions
- ✅ Extract control structures into separate functions
- ✅ Consider using "real" errors (with error handling) instead of "fake errors" built with if statements.