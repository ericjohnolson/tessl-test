# Task: Refactor with Atomic Commits

Refactor the following code to improve clarity. Use the refactor skill (`/refactor`).

## Code to Refactor

**File: `src/services/orderProcessor.ts`**

```typescript
export function processOrder(items: any[], customer: any, discountCode: string | null) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }

  // apply discount
  if (discountCode !== null && discountCode !== undefined && discountCode !== '') {
    if (discountCode === 'SAVE10') {
      total = total * 0.9;
    } else if (discountCode === 'SAVE20') {
      total = total * 0.8;
    } else if (discountCode === 'HALF') {
      total = total * 0.5;
    }
  }

  const tax = total * 0.08;
  total = total + tax;

  const result = {
    customerId: customer.id,
    customerName: customer.firstName + ' ' + customer.lastName,
    items: items,
    subtotal: total - tax,
    tax: tax,
    total: total,
    discount: discountCode,
    processed: true,
    timestamp: new Date().toISOString()
  };

  return result;
}
```

**File: `tests/services/orderProcessor.test.ts`** (existing tests pass)

The code has obvious refactoring opportunities: extract method for total calculation, extract method for discount application, simplify the null/undefined/empty check, use descriptive variable names, and extract the discount map.
