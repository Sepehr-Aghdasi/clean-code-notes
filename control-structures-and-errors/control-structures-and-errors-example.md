# Control Structures & Errors — Before & After Example

A single realistic example showing deep nesting, negative checks, and synthetic (code-based) errors, then the same code cleaned up per [control-structures-and-errors.md](./control-structures-and-errors.md).

---

## ❌ Before

```typescript
function processRefund(order: any) {
  if (order) {
    if (!order.isCancelled) {
      if (order.paymentMethod) {
        if (order.paymentMethod === 'creditCard') {
          const success = refundCreditCard(order);
          if (!success) {
            return { code: 1, message: 'Refund failed' };
          }
        } else if (order.paymentMethod === 'paypal') {
          const success = refundPayPal(order);
          if (!success) {
            return { code: 1, message: 'Refund failed' };
          }
        }
        return { code: 0, message: 'Refund successful' };
      }
    }
  }
  return { code: 2, message: 'Invalid order' };
}
```

The four nested `if` statements are hard to follow, the `!order.isCancelled` check is harder to read than a positive check, and failures return a `{ code, message }` object instead of throwing a real error — so callers must remember to check `.code` instead of just using `try/catch`.

---

## ✅ After

```typescript
function processRefund(order: Order) {
  if (!order || order.isCancelled || !order.paymentMethod) {
    throw new Error('Invalid order!');
  }

  const refund = getRefundHandler(order.paymentMethod);
  refund(order);
}

function getRefundHandler(paymentMethod: string): (order: Order) => void {
  const handlers: Record<string, (order: Order) => void> = {
    creditCard: refundCreditCard,
    paypal: refundPayPal,
  };

  const handler = handlers[paymentMethod];

  if (!handler) {
    throw new Error(`Unsupported payment method: ${paymentMethod}!`);
  }

  return handler;
}

function refundCreditCard(order: Order) {
  const success = chargeProvider.refund(order);

  if (!success) {
    throw new Error('Credit card refund failed!');
  }
}

function refundPayPal(order: Order) {
  const success = paypalProvider.refund(order);

  if (!success) {
    throw new Error('PayPal refund failed!');
  }
}
```

**What changed:**
- Nested `if` statements → one guard clause that checks everything up front and exits early
- `paymentMethod === 'creditCard' / 'paypal'` branching → a factory (`getRefundHandler`) that maps each payment method to its own function
- `{ code, message }` return values → real thrown `Error`s, so callers can use `try/catch`
- Unknown payment methods now throw a clear error instead of crashing later with a confusing one
