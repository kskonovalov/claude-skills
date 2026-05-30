---
name: code-review-error-handling
description: >
  Reviews code for error-handling and failure resilience ONLY — swallowed errors,
  empty catch blocks, unhandled promise rejections, missing cleanup (finally/dispose),
  lost error context, inconsistent error shapes/codes, missing timeouts/retries on I/O,
  leaking internal details in responses. Writes a report file; does not edit code.
  Use for "проверь обработку ошибок", "review error handling", "error handling review",
  "как обрабатываются сбои". Part of the code-review-* family (orchestrated by code-reviewer).
---

# code-review-error-handling

Узкий ревьюер **одной линзы — устойчивости к сбоям**. Запускается отдельно или из `code-reviewer`.
Цель: найти места, где сбой не обрабатывается, теряется или обрабатывается неверно. Код НЕ редактируется.

## Что проверяет

- **Проглоченные ошибки:** пустой `catch {}`, `catch (e) {}` без логирования/проброса, `.catch(() => {})`,
  возврат «успеха» при пойманной ошибке.
- **Необработанные промисы:** «висящие» промисы без `await`/`.catch`, отсутствие обработки rejection,
  `Promise.all` там, где нужен `allSettled`.
- **Cleanup:** не закрытые ресурсы (соединения, файлы, транзакции, таймеры, подписки) при ошибке;
  отсутствие `finally`/`try…finally`/`using`/`dispose`.
- **Потеря контекста:** перезапись оригинальной ошибки без `cause`, `throw new Error('failed')` без деталей,
  потеря стек-трейса при логировании.
- **Консистентность:** разные форматы/коды ошибок в разных хендлерах, отсутствие единого error-middleware
  (NestJS exception filter), HTTP-статусы не соответствуют типу ошибки.
- **Надёжность I/O:** нет таймаутов на сетевые/БД вызовы, нет ретраев на транзиентные сбои, нет idempotency
  на повторных попытках.
- **Утечка деталей:** стек/SQL/внутренние сообщения уходят клиенту в теле ответа.

Кратко FAIL → FIX:
```ts
// FAIL: проглоченная ошибка
try { await save(x); } catch (e) {}
// FIX
try { await save(x); } catch (e) { logger.error('save failed', { e }); throw e; }

// FAIL: ресурс не освобождён при ошибке
const c = await pool.connect(); const r = await c.query(sql); c.release(); return r;
// FIX
const c = await pool.connect(); try { return await c.query(sql); } finally { c.release(); }
```

## Границы (что НЕ моё)

- **Логика ветвления и корректность** (что код вычисляет) → `code-review-logic`.
- **Валидация входных данных** (проверка внешнего ввода) → `code-review-validation`.
- **Утечка PII/секретов как уязвимость** → `code-review-security` (здесь — только техническая утечка
  деталей ошибки в ответ).

Пограничное отдавай соседу молча, не дублируй. Здесь только «как обрабатывается путь сбоя».

## Процесс

1. **Определи вход** (приоритет):
   1. Явный путь из запроса. 2. Файл с git diff из запроса. 3. Иначе — `git diff` текущей ветки:
   `git diff $(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master)...HEAD`,
   fallback `git diff HEAD`.
2. Для каждого `try/catch`, `await`, `.then/.catch`, работы с ресурсами и внешних вызовов спроси:
   «что будет при сбое — ошибка обработана, проброшена, залогирована, ресурс закрыт?».
3. Сформируй находки своей линзы: `файл:строка → проблема → Почему → Фикс`.
4. Назначь severity и вердикт. Запиши отчёт.

## Отчёт

**Путь** (не хардкодить): `AGENT_HOME` = первая существующая в корне из `.cursor/`, `.gigacode/`, `.claude/`
(в этом порядке); если нет — создать `.claude/`. Писать в
`<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-error-handling.md`. Отчёт пишется **всегда**, даже при PASS.

**Severity:** 🔴 Blocker · 🟠 Major · 🟡 Minor · ⚪ Info.
**Вердикт:** ❌ FAIL — есть Blocker · ⚠️ WARN — есть Major/Minor · ✅ PASS — нет находок (или только Info).

**Шаблон:**
```markdown
# Review: code-review-error-handling — <✅ PASS | ⚠️ WARN | ❌ FAIL>
> вход: <git diff текущей ветки | путь <X> | diff-файл <Y>> · дата: <YYYY-MM-DD HH:MM> · находок: <N>

## 🔴 Blocker (<k>)
- `path/to/file.ts:42` — <проблема обработки ошибок>.
  Почему: <риск/последствие при сбое>. Фикс: <конкретное действие>.

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
