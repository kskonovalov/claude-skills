---
name: code-review-security
description: >
  Reviews code for exploitable security vulnerabilities ONLY (OWASP) — SQL/NoSQL/command
  injection, XSS, broken authn/authz, IDOR, hardcoded secrets, insecure deserialization,
  SSRF, unsafe CORS/headers, PII leakage, risky dependencies. Writes a report file; does
  not edit code. Use for "проверь безопасность", "security review", "уязвимости",
  "OWASP", "проверь на инъекции". Part of the code-review-* family (orchestrated by code-reviewer).
---

# code-review-security

Узкий ревьюер **одной линзы — эксплуатируемых уязвимостей**. Запускается отдельно или из `code-reviewer`.
Цель: найти то, что реально атакуемо. Код НЕ редактируется. Это **defensive review** для своего кода.

## Что проверяет

- **Инъекции:** конкатенация SQL/строк в запросах вместо параметризации; raw-запросы с пользовательским
  вводом; NoSQL-инъекции (объект вместо значения); command/shell-инъекции (`exec`/`spawn` с вводом);
  инъекция в шаблоны/eval.
- **XSS:** `dangerouslySetInnerHTML`, `innerHTML`, неэкранированный вывод пользовательских данных в DOM/HTML.
- **Authn/Authz:** эндпоинт без guard/проверки прав; проверка только на фронте; **IDOR** — доступ к объекту
  по id без проверки владельца; отсутствие проверки роли/скоупа; небезопасный JWT (нет проверки подписи/exp).
- **Секреты:** хардкод ключей/паролей/токенов в коде; секреты в логах; коммит `.env`.
- **Десериализация/парсинг:** небезопасная десериализация недоверенных данных; прототайп-полюшн
  (`Object.assign`/spread непроверенных ключей в чувствительный объект).
- **SSRF/доступ:** запросы по URL из пользователя без allowlist; path traversal (`../`) при работе с файлами.
- **Транспорт/конфиг:** слишком открытый CORS (`*` с credentials), отсутствие security-заголовков,
  отключённая проверка TLS.
- **Зависимости:** добавление пакета с известной уязвимостью/подозрительного происхождения.

Кратко FAIL → FIX:
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

## Границы (что НЕ моё)

- **Отсутствие валидации без вектора эксплуатации** (просто «нет проверки длины») → `code-review-validation`.
  Здесь — только когда есть **атакуемый путь**.
- **Логические баги без security-импакта** → `code-review-logic`.
- **Техническая утечка стека ошибки** (не PII/секрет) → `code-review-error-handling`.

Пограничное отдавай соседу молча, не дублируй.

## Процесс

1. **Определи вход** (приоритет): 1) явный путь; 2) файл с git diff; 3) иначе — `git diff` текущей ветки:
   `git diff $(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master)...HEAD`,
   fallback `git diff HEAD`.
2. Пройди по потоку недоверенных данных к опасным «стокам» (БД, shell, файловая система, DOM, HTTP-клиент,
   ответ клиенту) и проверь наличие защиты. Отдельно проверь authz на каждом новом эндпоинте и секреты.
3. Каждая находка должна иметь **правдоподобный вектор атаки**. Сомнительное — Info «проверить вручную».
4. `файл:строка → проблема → Почему (как эксплуатируется) → Фикс`. Назначь severity, вердикт. Запиши отчёт.

## Отчёт

**Путь** (не хардкодить): `AGENT_HOME` = первая существующая в корне из `.cursor/`, `.gigacode/`, `.claude/`
(в этом порядке); если нет — создать `.claude/`. Писать в
`<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-security.md`. Отчёт пишется **всегда**, даже при PASS.

**Severity:** 🔴 Blocker (эксплуатируемо удалённо/утечка данных) · 🟠 Major (уязвимость с условиями) ·
🟡 Minor (харднинг) · ⚪ Info (наблюдение/проверить вручную).
**Вердикт:** ❌ FAIL — есть Blocker · ⚠️ WARN — есть Major/Minor · ✅ PASS — нет находок (или только Info).

**Шаблон:**
```markdown
# Review: code-review-security — <✅ PASS | ⚠️ WARN | ❌ FAIL>
> вход: <git diff текущей ветки | путь <X> | diff-файл <Y>> · дата: <YYYY-MM-DD HH:MM> · находок: <N>

## 🔴 Blocker (<k>)
- `path/to/file.ts:42` — <уязвимость> (тип: SQLi/XSS/IDOR/…).
  Почему: <как эксплуатируется и что получает атакующий>. Фикс: <конкретная защита>.

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
