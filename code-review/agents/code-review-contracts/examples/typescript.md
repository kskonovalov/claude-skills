# Examples — TypeScript

```ts
// FAIL: swagger врёт про опциональность, фронт получит undefined
class UserDto { @ApiProperty() email: string; phone?: string; } // phone не в swagger, но возвращается опционально
// FIX: синхронизировать декоратор с реальностью
class UserDto { @ApiProperty() email: string; @ApiPropertyOptional() phone?: string; }
```
