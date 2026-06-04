# Examples — TypeScript

```ts
// FAIL: бизнес-логика + доступ к БД в контроллере, скрытая зависимость от времени
@Post() async pay(@Body() b) {
  if (new Date().getHours() < 9) throw new Error('closed');
  return this.pool.query('INSERT ...');           // SQL прямо в контроллере
}
// FIX: контроллер → сервис (логика, инъекция clock) → репозиторий (БД). Время инжектится → тестируемо.
```
