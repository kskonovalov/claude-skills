# Examples — TypeScript

```ts
// FAIL: магия + невнятные имена + вложенность
function calc(d, t) { if (t == 1) { return d * 0.9; } else { if (t == 2) { return d * 0.8; } } }
// FIX: говорящие имена + константы + плоско
const DISCOUNT = { silver: 0.9, gold: 0.8 } as const;
function applyDiscount(price: number, tier: Tier) { return price * DISCOUNT[tier]; }
```
