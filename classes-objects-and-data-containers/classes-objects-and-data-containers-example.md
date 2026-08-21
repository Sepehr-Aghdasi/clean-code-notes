# Classes, Objects & Data Containers — Before & After Example

A single realistic example showing a class with too many responsibilities and a Law of Demeter violation, then the same code cleaned up per [classes-objects-and-data-containers.md](./classes-objects-and-data-containers.md).

---

## ❌ Before

```typescript
class User {
  public name: string;
  public email: string;
  public password: string;
  public lastPurchase: { date: Date; amount: number };

  constructor(name: string, email: string, password: string) {
    this.name = name;
    this.email = email;
    this.password = password;
  }

  login(password: string): boolean {
    return this.password === password;
  }

  refund(amount: number) {
    // payment logic doesn't belong on User (violates SRP)
    paymentGateway.refund(this.email, amount);
  }

  sendReceipt() {
    // email logic doesn't belong on User either
    mailer.send(this.email, 'Receipt', '...');
  }
}

// Elsewhere in the code - reaching through User into Purchase (Law of Demeter violation):
function notifyLastPurchase(user: User) {
  console.log(`Last purchase: ${user.lastPurchase.date}`);
}
```

`User` has three reasons to change: user account rules, payment/refund rules, and email formatting. It also exposes `password` and `lastPurchase` publicly, which is what lets `notifyLastPurchase()` reach through `user.lastPurchase.date` — a chain that breaks the moment `Purchase` is restructured.

---

## ✅ After

```typescript
interface Purchase {
  date: Date;
  amount: number;
}

class User {
  public name: string;
  public email: string;
  private password: string;
  private lastPurchase: Purchase;

  constructor(name: string, email: string, password: string) {
    this.name = name;
    this.email = email;
    this.password = password;
  }

  login(password: string): boolean {
    return this.password === password;
  }

  getLastPurchaseDate(): Date {
    return this.lastPurchase.date;
  }
}

class PaymentService {
  refund(user: User, amount: number) {
    paymentGateway.refund(user.email, amount);
  }
}

class ReceiptMailer {
  send(user: User) {
    mailer.send(user.email, 'Receipt', '...');
  }
}

// Elsewhere in the code - Law of Demeter respected, no chaining into Purchase:
function notifyLastPurchase(user: User) {
  console.log(`Last purchase: ${user.getLastPurchaseDate()}`);
}
```

**What changed:**
- `refund()` and `sendReceipt()` moved out of `User` into `PaymentService` and `ReceiptMailer` — `User` now has a single responsibility (SRP)
- `password` and `lastPurchase` are now `private`, exposed only through the public API (`login()`, `getLastPurchaseDate()`)
- `user.lastPurchase.date` chaining replaced with `user.getLastPurchaseDate()` — callers no longer depend on `Purchase`'s internal shape (Law of Demeter)
