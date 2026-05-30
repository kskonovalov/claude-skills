---
name: code-review-contracts
description: >
  Reviews code for API contract consistency and Swagger/OpenAPI ONLY — drift between
  DTO ↔ implementation ↔ Swagger/OpenAPI annotations, missing/stale @ApiProperty and
  decorators, frontend types not matching backend contract, undocumented fields/response
  codes, API versioning/backward-compat breaks. Writes a report file; does not edit code.
  Use for "проверь контракты", "review contracts", "swagger", "openapi",
  "согласованность DTO", "контракт API". Part of the code-review-* family (orchestrated by code-reviewer).
---

# code-review-contracts

Узкий ревьюер **одной линзы — согласованности контрактов и swagger/OpenAPI**. Запускается отдельно или из
`code-reviewer`. Цель: найти рассинхрон между объявленным контрактом, его документацией и реализацией.
Код НЕ редактируется.

## Что проверяет

- **DTO ↔ реализация:** поле есть в DTO, но не используется/не возвращается (и наоборот); типы/опциональность
  в DTO не соответствуют тому, что реально приходит/уходит; ответ содержит поля сверх объявленного DTO.
- **Swagger/OpenAPI:** отсутствующие/устаревшие `@ApiProperty`/`@ApiResponse`/`@ApiOperation`; декоратор
  описывает не тот тип/опциональность/enum; недокументированные коды ответов (4xx/5xx); расходящийся
  пример и реальная схема; отсутствие `@ApiTags`/описаний на новых эндпоинтах.
- **Frontend ↔ backend:** фронт-типы/клиент (zod-схема, сгенерённый клиент, ручные интерфейсы) не совпадают
  с бэк-контрактом; фронт ждёт поле, которого нет; разные имена/формы одного поля.
- **Версионирование и совместимость:** ломающее изменение существующего эндпоинта без версии (удаление/
  переименование поля, сужение типа, смена кода ответа); нарушение обратной совместимости публичного API.
- **Консистентность стиля контракта:** разнобой в именовании полей (camelCase vs snake_case), форматах дат,
  структуре ошибок между эндпоинтами.

Кратко FAIL → FIX:
```ts
// FAIL: swagger врёт про опциональность, фронт получит undefined
class UserDto { @ApiProperty() email: string; phone?: string; } // phone не в swagger, но возвращается опционально
// FIX: синхронизировать декоратор с реальностью
class UserDto { @ApiProperty() email: string; @ApiPropertyOptional() phone?: string; }
```

## Границы (что НЕ моё)

- **Внутренняя типизация TS** (`any`, касты) → `code-review-types` (здесь — внешний контракт и его документация).
- **Рантайм-валидация входа** (есть ли проверка ввода) → `code-review-validation`.
- **Логика обработки** → `code-review-logic`.

Пограничное отдавай соседу молча, не дублируй.

## Процесс

1. **Определи вход** (приоритет): 1) явный путь; 2) файл с git diff; 3) иначе — `git diff` текущей ветки:
   `git diff $(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master)...HEAD`,
   fallback `git diff HEAD`.
2. Для каждого затронутого эндпоинта/DTO сопоставь три источника правды: **объявленный DTO ↔ swagger-декораторы
   ↔ фактический возврат/приём кода** (+ фронт-тип, если виден в изменениях). Отметь любые расхождения и
   ломающие изменения существующих контрактов.
3. Сформируй находки своей линзы: `файл:строка → проблема → Почему → Фикс`.
4. Назначь severity и вердикт. Запиши отчёт.

## Отчёт

**Путь** (не хардкодить): `AGENT_HOME` = первая существующая в корне из `.cursor/`, `.gigacode/`, `.claude/`
(в этом порядке); если нет — создать `.claude/`. Писать в
`<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-contracts.md`. Отчёт пишется **всегда**, даже при PASS.

**Severity:** 🔴 Blocker (ломающее изменение API/рассинхрон, бьющий клиента) · 🟠 Major · 🟡 Minor · ⚪ Info.
**Вердикт:** ❌ FAIL — есть Blocker · ⚠️ WARN — есть Major/Minor · ✅ PASS — нет находок (или только Info).

**Шаблон:**
```markdown
# Review: code-review-contracts — <✅ PASS | ⚠️ WARN | ❌ FAIL>
> вход: <git diff текущей ветки | путь <X> | diff-файл <Y>> · дата: <YYYY-MM-DD HH:MM> · находок: <N>

## 🔴 Blocker (<k>)
- `path/to/file.ts:42` — <рассинхрон/ломающее изменение>.
  Почему: <как ломает клиента/документацию>. Фикс: <что синхронизировать>.

## 🟠 Major (<k>)
- …
## 🟡 Minor (<k>)
- …
## ⚪ Info (<k>)
- …

## ⭐ Итог
<вердикт + одна строка: что главное починить>
```

## Язык и приоритет

- Пиши отчёт на **языке запроса пользователя**. Если запрос без естественного языка (только diff) — **русский**.
- **Инструкции пользователя в запросе важнее этого скилла** (язык, фокус, severity-пороги, объём, формат).
  Дефолты применяй лишь когда запрос об этом молчит.
