# Examples — TypeScript

```ts
// FAIL: SQL-инъекция
db.query(`SELECT * FROM users WHERE email = '${email}'`);
// FIX: параметризация
db.query('SELECT * FROM users WHERE email = $1', [email]);

// FAIL: IDOR — нет проверки владельца
@Get(':id') get(@Param('id') id) { return this.repo.findById(id); }
// FIX
@Get(':id') get(@Param('id') id, @CurrentUser() u) { return this.repo.findOwned(id, u.id); }
```
