# Code Review Skills

Набор гранулярного код-ревью: **1 оркестратор + 9 узких скиллов**, каждый проверяет одну линзу.
Заточены под **TS / React / Node (NestJS) / PostgreSQL**. Сделаны узкими специально — чтобы надёжно
работать на слабой модели. **Review-only:** код не правят, пишут отчёт.

## Скиллы

| Скилл | Что ловит |
|---|---|
| `code-reviewer` | **Оркестратор**: определяет вход, гоняет tsc+eslint, запускает суб-скиллы, собирает сводку |
| `code-review-logic` | Корректность: edge cases, null, условия, гонки, async |
| `code-review-security` | Уязвимости (OWASP): инъекции, XSS, authz/IDOR, секреты |
| `code-review-validation` | Валидация внешнего ввода: схемы (zod/DTO), границы, нормализация |
| `code-review-error-handling` | Сбои: проглоченные ошибки, rejection, cleanup, таймауты |
| `code-review-types` | Типобезопасность TS: `any`, касты `as`, `!`, дрейф тип↔рантайм |
| `code-review-contracts` | Согласованность DTO ↔ реализация ↔ swagger/OpenAPI, ломающие изменения API |
| `code-review-performance` | N+1, индексы Postgres, лишние ре-рендеры React, блокирующий I/O |
| `code-review-architecture` | Слои, связность, SOLID, god-объекты, тестируемость (DI) |
| `code-review-readability` | Нейминг, сложность, мёртвый код, магия, дубли |

## Порядок запуска

Лучше всего — запустить **`code-reviewer`**: он сам прогоняет всё в правильном порядке и делает сводку.

Если запускаешь вручную, иди от багов к качеству (сверху вниз):

```
0. tsc + eslint            ← детерминированно, до всего (внутри code-reviewer)
1. logic                   ┐
2. security                │ ловят реальные баги/риски — сначала
3. validation              │
4. error-handling          ┘
5. types                   ┐
6. contracts               │ корректность интерфейсов
7. performance             ┘
8. architecture            ┐ качество/сопровождаемость — в конце
9. readability             ┘
```

## Вход

1. Явный путь: `code-reviewer /path/to/code` (файл или папка).
2. Файл с git diff: `*.txt` / `*.diff`.
3. По умолчанию — `git diff` текущей ветки (vs main/master).

> Запрос пользователя всегда главнее: можно задать язык, набор аспектов, severity-порог, объём.

## Отчёты

Пишутся в `<AGENT_HOME>/reports/<YYYY.MM.DD>/<skill>.md`, где `AGENT_HOME` — первая из
`.cursor` / `.gigacode` / `.claude` (определяется автоматически). На языке запроса (fallback — русский).

- **Severity:** 🔴 Blocker · 🟠 Major · 🟡 Minor · ⚪ Info
- **Вердикт:** ❌ FAIL (есть Blocker) · ⚠️ WARN (Major/Minor) · ✅ PASS

## Установка

Скопируй нужные папки скиллов в `~/.claude/skills/` (или `.cursor` / `.gigacode`). Каждая папка
самодостаточна. `SHARED-SPEC.md` — справочник контракта для сопровождающего, копировать не нужно.
