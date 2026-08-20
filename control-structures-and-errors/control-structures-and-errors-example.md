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

Four levels of nesting force the reader to hold every outer condition in their head just to understand the innermost line. The negative `!order.isCancelled` check adds friction, and failures are reported as `{ code, message }` objects instead of real errors — so every caller has to remember to check `.code` instead of using `try/catch`.

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

  return handlers[paymentMethod];
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
- Four nested `if` statements → one guard clause at the top that fails fast on any invalid state
- The `paymentMethod === 'creditCard' / 'paypal'` branching (which would only grow with more payment methods) was replaced with a factory (`getRefundHandler`) so each payment method's logic lives in its own function
- `{ code, message }` return values → real thrown `Error`s, so callers can use standard `try/catch` instead of checking a `code` field
