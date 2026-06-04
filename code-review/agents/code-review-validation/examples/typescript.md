# Examples — TypeScript

```ts
// FAIL: тело принимается как есть
@Post() create(@Body() body: any) { return this.svc.create(body); }
// FIX: схема + типы + границы
class CreateUserDto { @IsEmail() email: string; @Length(1, 100) name: string; @Min(0) @Max(150) age: number; }
@Post() create(@Body() dto: CreateUserDto) { return this.svc.create(dto); } // + global ValidationPipe({ whitelist: true })
```
