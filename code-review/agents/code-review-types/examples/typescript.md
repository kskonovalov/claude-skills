# Examples — TypeScript

```ts
// FAIL: каст ответа без проверки → ложная типобезопасность
const user = (await res.json()) as User;
// FIX: рантайм-схема + вывод типа
const user = UserSchema.parse(await res.json()); // zod: тип = z.infer<typeof UserSchema>

// FAIL: non-null без гарантии
const el = document.querySelector('.x')!; el.focus();
// FIX
const el = document.querySelector('.x'); if (!el) return; el.focus();
```
