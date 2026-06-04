# Examples — TypeScript

```ts
// FAIL: гонка read-modify-write
const u = await repo.findById(id); u.balance -= amount; await repo.save(u);
// FIX: атомарно
await repo.decrement({ id }, 'balance', amount); // или транзакция с блокировкой строки

// FAIL: потерянный await → необработанный rejection и неверный порядок
items.forEach(async (i) => { await process(i); });
// FIX
for (const i of items) await process(i);   // или await Promise.all(items.map(process))
```
