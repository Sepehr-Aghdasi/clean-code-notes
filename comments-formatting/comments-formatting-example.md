# Comments & Formatting — Before & After Example

A single realistic example showing bad comments and poor formatting, then the same code cleaned up per [comments-formatting.md](./comments-formatting.md).

---

## ❌ Before

```typescript
// !!!!!!!!!! VALIDATION !!!!!!!!!!
function validateSignup(email: string, password: string) {
  // check the input
  if (!email.includes('@') || !/^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/.test(password)) { throw new Error('Invalid input!'); }
  // create the session
  const existingUser = findUserByEmail(email);
  // if (existingUser) {
  //   console.log('found a duplicate, weird case, ignoring for now');
  // }
  if (existingUser) {
    throw new Error('Email already taken!');
  }
}
```

This has a redundant divider comment, a redundant "check the input" comment that adds nothing, a misleading "create the session" comment sitting above code that does nothing of the sort, dead commented-out code, one very long unreadable line, and no vertical spacing between the two unrelated concerns (validating the format vs. checking for a duplicate).

---

## ✅ After

```typescript
function validateSignup(email: string, password: string) {
  if (!isEmail(email) || !isStrongPassword(password)) {
    throw new Error('Invalid input!');
  }

  const existingUser = findUserByEmail(email);

  if (existingUser) {
    throw new Error('Email already taken!');
  }
}

function isEmail(email: string): boolean {
  return email.includes('@');
}

// Min. 8 characters, at least: one letter, one number, one special character
function isStrongPassword(password: string): boolean {
  const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/;
  return passwordRegex.test(password);
}
```

**What changed:**
- Removed the divider comment, the redundant comment, the misleading comment, and the commented-out code entirely
- Kept exactly one comment — a "required explanation" for the regex, which is genuinely hard to read at a glance
- Extracted `isEmail()` / `isStrongPassword()` so the long, unreadable condition became two short, descriptive checks
- Added a blank line between the two unrelated concerns (input validation vs. duplicate check) — vertical distance
