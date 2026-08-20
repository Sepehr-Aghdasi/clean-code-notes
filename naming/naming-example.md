# Naming — Before & After Example

A single realistic example showing how bad naming makes code hard to follow, and how applying the naming rules from [naming.md](./naming.md) fixes it.

---

## ❌ Before

```typescript
class User {
  e: string;
  p: string;

  constructor(e: string, p: string) {
    this.e = e;
    this.p = p;
  }

  save() {
    // ...
  }
}

function proc(d: any) {
  let x = d.e;
  let y = d.p;
  let chk = x.includes('@') && y.length >= 8;

  if (chk) {
    let usr = new User(x, y);
    usr.save();
    return true;
  }

  return false;
}

function fetchUsers() {
  /* ... */
}

function getProducts() {
  /* ... */
} // inconsistent with fetchUsers()
```

Every name here requires the reader to dig into the implementation to figure out what it means: `d`, `x`, `y`, `chk`, `usr` are meaningless, `proc()` is a generic verb that says nothing about what's being processed, and `getProducts()` breaks the `fetch...()` convention already established by `fetchUsers()`.

---

## ✅ After

```typescript
class User {
  email: string;
  password: string;

  constructor(email: string, password: string) {
    this.email = email;
    this.password = password;
  }

  save() {
    // ...
  }
}

function registerUser(credentials: { email: string; password: string }): boolean {
  const isValidInput = isEmail(credentials.email) && hasValidPasswordLength(credentials.password);

  if (!isValidInput) {
    return false;
  }

  const user = new User(credentials.email, credentials.password);
  user.save();
  return true;
}

function isEmail(email: string): boolean {
  return email.includes('@');
}

function hasValidPasswordLength(password: string): boolean {
  return password.length >= 8;
}

function fetchUsers() {
  /* ... */
}

function fetchProducts() {
  /* ... */
} // now consistent with fetchUsers()
```

**What changed:**
- `e` / `p` → `email` / `password` — nouns that describe the data they hold
- `proc(d)` → `registerUser(credentials)` — a verb that says exactly what the function does, with a descriptive parameter
- `chk` → `isValidInput` — a boolean phrased as a yes/no question
- `usr` → `user` — no reason to abbreviate
- `getProducts()` → `fetchProducts()` — consistent with `fetchUsers()`
