# Functions & Methods — Before & After Example

A single realistic example showing too many parameters, mixed levels of abstraction, and an unexpected side effect, then the same code cleaned up per [functions-and-methods.md](./functions-and-methods.md).

---

## ❌ Before

```typescript
function createOrder(
  userId: string,
  productId: string,
  quantity: number,
  discountCode: string,
  giftWrap: boolean,
  expressShipping: boolean
) {
  if (quantity <= 0) {
    console.log('Invalid quantity!');
    return null;
  }

  const product = database.find('products', productId);
  const price = product.price * quantity;
  const discount = discountCode === '' ? 0 : price * 0.1;
  const total = price - discount;

  const order = { userId, productId, quantity, total, giftWrap, expressShipping };
  database.insert('orders', order);

  sendEmail(userId, 'Order confirmed'); // unexpected side effect - not implied by the function name

  return order;
}
```

Six positional parameters make every call site (`createOrder('u1', 'p1', 2, '', false, true)`) unreadable without hovering over the function. The body also mixes levels of abstraction (raw validation logic, price math, and a database call side by side) and quietly sends an email — a side effect nobody would expect from a function called `createOrder`.

---

## ✅ After

```typescript
interface OrderRequest {
  userId: string;
  productId: string;
  quantity: number;
  discountCode?: string;
  giftWrap: boolean;
  expressShipping: boolean;
}

function createOrder(orderRequest: OrderRequest): Order {
  const { quantity } = orderRequest;
  validateQuantity(quantity);

  const total = calculateTotal(orderRequest);
  const order = buildOrder(orderRequest, total);

  saveOrder(order);
  return order;
}

function validateQuantity(quantity: number) {
  if (quantity <= 0) {
    throw new Error('Invalid quantity!');
  }
}

function calculateTotal({ productId, quantity, discountCode }: OrderRequest): number {
  const product = database.find('products', productId);
  const price = product.price * quantity;
  const discount = discountCode ? price * 0.1 : 0;
  return price - discount;
}

function buildOrder(orderRequest: OrderRequest, total: number): Order {
  return { ...orderRequest, total };
}

function saveOrder(order: Order) {
  database.insert('orders', order);
}

// The caller decides when a side effect like sending an email should happen -
// createOrder() itself no longer does it silently.
function handleCheckoutRequest(orderRequest: OrderRequest) {
  const order = createOrder(orderRequest);
  sendEmail(order.userId, 'Order confirmed');
  return order;
}
```

**What changed:**
- Six positional parameters → one `orderRequest` object — the call site now reads as `createOrder(orderRequest)` instead of six unlabeled values
- Split into `validateQuantity()`, `calculateTotal()`, `buildOrder()`, `saveOrder()` — each does one thing, on one level of abstraction
- Moved `sendEmail(...)` out of `createOrder()` into the caller — no more unexpected side effect hidden behind the name
- Replaced the synthetic `console.log` + `return null` error path with a real thrown error
