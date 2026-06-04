# Examples — TypeScript

```ts
// FAIL: N+1
for (const o of orders) o.user = await repo.findUser(o.userId);
// FIX: один запрос
const users = await repo.findUsersByIds(orders.map(o => o.userId)); // затем map по id

// FAIL: нестабильный проп → лишние ре-рендеры
<List style={{ margin: 8 }} onSelect={() => pick(id)} />
// FIX: вынести/мемоизировать style и onSelect (useMemo/useCallback)
```
