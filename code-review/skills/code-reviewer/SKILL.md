---
name: code-reviewer
description: >
  Orchestrates the full code-review-* family: resolves the review target (explicit path,
  a git diff file, or — by default — the current branch's git diff), runs deterministic
  tooling (tsc + eslint), dispatches the 9 focused sub-skills (logic, error-handling,
  validation, security, types, performance, architecture, readability, contracts), and
  writes per-aspect reports plus an aggregated summary. Review-only; does not edit code.
  Use for "сделай ревью", "code review", "review my changes", "code-reviewer /path",
  "проверь код", "полное ревью". Can run a subset if the request names specific aspects.
---

# code-reviewer

Оркестратор гранулярного код-ревью. Определяет цель, прогоняет детерминированный тулинг, запускает выбранные
суб-скиллы `code-review-*` и собирает сводку. **Advisory & review-only:** ничего не правит в коде и не
блокирует мерж — только отчёты.

## Семейство суб-скиллов

`code-review-logic` · `code-review-error-handling` · `code-review-validation` · `code-review-security` ·
`code-review-types` · `code-review-performance` · `code-review-architecture` · `code-review-readability` ·
`code-review-contracts`. Каждый проверяет свою линзу и не дублирует соседей.

## Процесс

### 1. Определи вход (приоритет)
1. **Явный путь** в запросе (`code-reviewer /path/to/code`, файл или папка) → ревьюим его.
2. **Файл с git diff** в запросе (`*.txt`/`*.diff`) → ревьюим его содержимое.
3. **По умолчанию** — `git diff` текущей ветки относительно базовой:
   ```bash
   BASE=$(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null)
   [ -n "$BASE" ] && git diff "$BASE"...HEAD || git diff HEAD   # fallback: незакоммиченные изменения
   ```

Зафиксируй строку «вход» (что именно ревьюится) — она пойдёт во все отчёты.

### 1.5. Прочитай контекст проекта

Прочитай (если есть в корне проекта): CLAUDE.md, package.json, tsconfig.json.
Зафиксируй: реальный стек (NestJS / Express / Next.js / etc.), тест-раннер, слоевую архитектуру.
Эта информация передаётся в промпт каждого субагента и перекрывает дефолтные примеры из `examples/`.

### 2. Выбери набор аспектов
- По умолчанию — **все 9** суб-скиллов.
- Если запрос называет подмножество («только security и types», «без readability») — запусти только их.
- Если запрос задаёт язык/severity-порог/формат — эти инструкции **перекрывают** дефолты и передаются всем
  суб-скиллам.

### 3. Шаг tooling (детерминированно, замена скилла «syntax»)
Если в проекте сконфигурированы инструменты — запусти и собери вывод:
```bash
npx tsc --noEmit            # если есть tsconfig.json
npx eslint .                # если есть eslint-конфиг (или линт изменённых файлов)
```
Сохрани результат в под-отчёт `code-review-tooling.md` (см. путь ниже): список ошибок/предупреждений
`файл:строка → сообщение`, вердикт PASS/FAIL по факту наличия ошибок. Если инструменты не настроены —
отметь это и пропусти шаг (не выдумывай ошибки).

### 4. Запусти суб-агенты

Каждый агент получает: вход (diff/путь), язык отчёта, пользовательские override'ы,
и полные инструкции своего аспекта из соответствующего `SKILL.md`. Каждый пишет свой файл-отчёт сам.

**Определи режим запуска:**
- Если в списке доступных тебе инструментов есть `Agent` с параметром `subagent_type` → **параллельный режим**.
- Иначе (GigaCode, Cursor, Codex, любая среда без `subagent_type`) → **последовательный режим**.

**Параллельный режим (Claude Code)** — запускай все выбранные одним сообщением (`Agent` tool, `run_in_background: true`),
дождись уведомлений об их завершении, затем переходи к шагу 5.

| Аспект | `subagent_type` (Claude Code) |
|---|---|
| `code-review-logic` | `error-debugging:debugger` |
| `code-review-error-handling` | `error-debugging:error-detective` |
| `code-review-validation` | `backend-api-security:backend-security-coder` |
| `code-review-security` | `comprehensive-review:security-auditor` |
| `code-review-types` | `javascript-typescript:typescript-pro` |
| `code-review-performance` | `backend-development:performance-engineer` |
| `code-review-architecture` | `comprehensive-review:architect-review` |
| `code-review-readability` | `code-documentation:code-reviewer` |
| `code-review-contracts` | `backend-development:backend-architect` |

**Последовательный режим (GigaCode / Cursor / другие)** — для каждого аспекта:
1. Найди SKILL.md этого аспекта (рядом с текущим файлом в `../agents/<aspect>/SKILL.md` или в `agents/<aspect>/SKILL.md`) и прочитай его инструментом `Read`.
2. Выполни ревью в текущем контексте строго по инструкциям этого SKILL.md.
3. Запиши отчёт сразу по завершении аспекта, затем переходи к следующему.
Порядок: logic → error-handling → validation → security → types → performance → architecture → readability → contracts.

### 5. Собери сводку
Запиши `code-reviewer.md` (тот же каталог дня):
- строка «вход» и дата;
- таблица: аспект · вердикт · 🔴/🟠/🟡/⚪ счётчики;
- **общий вердикт:** ❌ FAIL если хоть у одного аспекта Blocker; ⚠️ WARN если есть Major/Minor, но нет
  Blocker; ✅ PASS если везде чисто;
- топ-приоритеты (все Blocker'ы списком с указанием аспекта и `файл:строка`);
- ссылки на под-отчёты.

## Путь отчётов (не хардкодить)

`AGENT_HOME` = первая существующая в корне проекта папка из `.cursor/`, `.gigacode/`, `.claude/`
(в этом порядке); если нет ни одной — создать `.claude/`. Создать `<AGENT_HOME>/reports/<YYYY.MM.DD>/`.
Файлы:
- сводка: `<AGENT_HOME>/reports/<YYYY.MM.DD>/code-reviewer.md`
- tooling: `<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-tooling.md`
- по аспектам: `<AGENT_HOME>/reports/<YYYY.MM.DD>/code-review-<aspect>.md`

Отчёты пишутся всегда, даже при PASS. Повторный запуск в тот же день перезаписывает файлы.

## Шаблон сводки

```markdown
# Code Review — <✅ PASS | ⚠️ WARN | ❌ FAIL>
> вход: <git diff текущей ветки | путь <X> | diff-файл <Y>> · дата: <YYYY-MM-DD HH:MM>

| Аспект | Вердикт | 🔴 | 🟠 | 🟡 | ⚪ |
|---|---|--:|--:|--:|--:|
| tooling (tsc/eslint) | … | | | | |
| logic | … | | | | |
| error-handling | … | | | | |
| validation | … | | | | |
| security | … | | | | |
| types | … | | | | |
| performance | … | | | | |
| architecture | … | | | | |
| readability | … | | | | |
| contracts | … | | | | |

## 🔴 Приоритеты (Blocker)
- [security] `path/file.ts:42` — <кратко>
- …

## Отчёты
- [tooling](./code-review-tooling.md)
- [logic](./code-review-logic.md)
- … (только запущенные аспекты)
```

## Язык и приоритет

- Сводка и все под-отчёты — на **языке запроса пользователя**. Если запрос без естественного языка
  (только diff) — **русский**.
- **Инструкции пользователя в запросе важнее этого скилла и суб-скиллов** (язык, набор аспектов,
  severity-пороги, объём входа, формат). Дефолты применяй лишь когда запрос об этом молчит.
