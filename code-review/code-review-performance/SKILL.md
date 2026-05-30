---
name: code-review-performance
description: >
  Reviews code for performance ONLY — N+1 queries, inefficient/unindexed Postgres queries,
  unnecessary React re-renders (missing memo, unstable props, list keys), work/allocations
  in loops, blocking I/O, missing pagination/streaming, heavy sync operations. Writes a
  report file; does not edit code. Use for "проверь производительность",
  "performance review", "N+1", "оптимизация", "почему медленно". Part of the
  code-review-* family (orchestrated by code-reviewer).
---

# code-review-performance

Узкий ревьюер **одной линзы — производительности**. Запускается отдельно или из `code-reviewer`.
Цель: найти места с лишней работой/задержками под стек React + Node + PostgreSQL. Код НЕ редактируется.

## Что проверяет

- **БД (Postgres):** N+1 запросы (запрос в цикле/на элемент списка вместо join/`IN`/`dataloader`);
  отсутствие индекса под фильтр/сортировку/join; `SELECT *` где нужны поля; запрос без пагинации/лимита;
  `count(*)` на больших таблицах в горячем пути; отсутствие пула/переоткрытие соединений; ORM, грузящий
  лишние связи (eager без надобности).
- **React:** ре-рендеры из-за нестабильных пропсов (инлайн-объекты/функции/массивы), отсутствие
  `useMemo`/`useCallback`/`memo` в горячих местах; `key={index}` в динамических списках; тяжёлые вычисления
  в рендере; контекст, дёргающий всё дерево; загрузка всего списка без виртуализации/пагинации.
- **Циклы/алгоритмы:** O(n²) там, где хватит map/set; работа/аллокации в цикле, выносимые наружу;
  повторный пересчёт одного и того же; лишние копии больших структур.
- **I/O и async:** блокирующие синхронные вызовы (`fs.readFileSync`, тяжёлый `JSON.parse`) в горячем пути;
  последовательные `await` там, где можно `Promise.all`; отсутствие стриминга для больших ответов;
  отсутствие кэширования дорогих повторяемых операций.

Кратко FAIL → FIX:
```ts
// FAIL: N+1
for (const o of orders) o.user = await repo.findUser(o.userId);
// FIX: один запрос
const users = await repo.findUsersByIds(orders.map(o => o.userId)); // затем map по id

// FAIL: нестабильный проп → лишние ре-рендеры
<List style={{ margin: 8 }} onSelect={() => pick(id)} />
// FIX: вынести/мемоизировать style и onSelect (useMemo/useCallback)
```

## Границы (что НЕ моё)

- **Архитектурная связность/слои** → `code-review-architecture` (здесь — конкретные перф-горячие точки).
- **Логические баги** (в т.ч. лишний `await`, ломающий корректность) → `code-review-logic`
  (здесь — лишний `await`, бьющий по скорости, без поломки логики).
- **Безопасность** → `code-review-security`.

Пограничное отдавай соседу молча, не дублируй.

## Процесс

1. **Определи вход** (приоритет): 1) явный путь; 2) файл с git diff; 3) иначе — `git diff` текущей ветки:
   `git diff $(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master)...HEAD`,
   fallback `git diff HEAD`.
2. Ищи горячие пути: циклы с I/O, запросы к БД, рендеры компонентов, обработку коллекций. Оцени стоимость
   на реалистичных объёмах данных (не на 3 элементах).
3. Сформируй находки своей линзы: `файл:строка → проблема → Почему (стоимость/масштаб) → Фикс`.
4. Назначь severity и вердикт. Запиши отчёт.

## Отчёт

**Путь** (не хардкодить): `AGENT_HOME` = первая существующая в корне из `.cursor/`, `.gigacode/`, `.claude/`
(в этом порядке); если нет — создать `.claude/`. Писать в
`<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-performance.md`. Отчёт пишется **всегда**, даже при PASS.

**Severity:** 🔴 Blocker (деградация под нагрузкой/таймауты) · 🟠 Major · 🟡 Minor · ⚪ Info.
**Вердикт:** ❌ FAIL — есть Blocker · ⚠️ WARN — есть Major/Minor · ✅ PASS — нет находок (или только Info).

**Шаблон:**
```markdown
# Review: code-review-performance — <✅ PASS | ⚠️ WARN | ❌ FAIL>
> вход: <git diff текущей ветки | путь <X> | diff-файл <Y>> · дата: <YYYY-MM-DD HH:MM> · находок: <N>

## 🔴 Blocker (<k>)
- `path/to/file.ts:42` — <узкое место>.
  Почему: <как растёт стоимость с объёмом данных>. Фикс: <конкретное действие>.

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
