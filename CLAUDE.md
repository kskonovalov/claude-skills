# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Назначение репозитория

Это **не** проект с кодом, а **каталог-кладовка скиллов** (Claude Code Agent Skills), скачанных из разных
публичных репозиториев. Цель — отобрать из этой массы те скиллы, которые подходят под стек владельца:
**React, TypeScript, Node.js, PostgreSQL**, и собрать из них личный набор.

Никакого build / lint / test тут нет. «Работа» в этом репозитории = **читать `SKILL.md`, оценивать,
классифицировать и копировать нужные скиллы наружу** (в `~/.claude/skills/` или в `.claude/skills/` целевого проекта).

---

## Собранные наборы (готовы к установке)

Отобранные и адаптированные инструменты живут в корневых папках (не в `sources/`):

### `code-review/` — гранулярное код-ревью

```
code-review/
├── skills/
│   └── code-reviewer/          ← slash-команда /code-reviewer; оркестратор
└── agents/                     ← промпты для параллельных фоновых агентов
    ├── code-review-logic/
    ├── code-review-error-handling/
    ├── code-review-validation/
    ├── code-review-security/
    ├── code-review-types/
    ├── code-review-performance/
    ├── code-review-architecture/
    ├── code-review-readability/
    └── code-review-contracts/
```

**Как работает:** `/code-reviewer` запускает 9 агентов параллельно (`Agent` tool, `run_in_background: true`,
каждый со своим `subagent_type`), каждый пишет отчёт в `.claude/reports/<YYYY.MM.DD>/`. Сводка — в `code-reviewer.md`.

**Установка:** скопировать `code-review/skills/code-reviewer/` в `.claude/skills/` проекта.
Папку `agents/` — рядом или в то же место; оркестратор читает их содержимое и передаёт агентам как промпты.

---

### `planning/` — планирование и поэтапная реализация

```
planning/
└── skills/
    ├── brainstorming/      ← уточняет требования (вопросы по одному), пишет спек
    ├── writing-plans/      ← поэтапный план с TDD-шагами и проверкой после каждого
    └── executing-plans/    ← исполняет план, стоп при блокере, чекпоинты
```

**Цепочка:** `brainstorming` → `writing-plans` → `executing-plans`

- `brainstorming`: HARD-GATE перед любым кодом — сначала уточняет все детали по одному, предлагает 2-3 подхода, получает одобрение спека.
- `writing-plans`: создаёт план с задачами по 2-5 минут, каждая с тестом → запуск → реализация → запуск → коммит.
- `executing-plans`: исполняет план шаг за шагом, не пропускает проверки, останавливается при блокере.

Спеки сохраняются в `docs/specs/`, планы — в `docs/plans/`.

**Установка:** скопировать все три папки в `.claude/skills/` проекта.

---

### Анатомия скилла
Каждый скилл — это папка с файлом `SKILL.md` (YAML-фронтматтер `name` + `description`, далее инструкции).
Некоторые тянут за собой `references/`, `scripts/`, `assets/`. Установка = скопировать всю папку скилла
в директорию скиллов. Зависимостей между скиллами обычно нет (кроме «meta»-скиллов, ссылающихся друг на друга).

**Конвенция: каждый собранный скилл должен содержать `metadata.md`** рядом с `SKILL.md`. Формат:

```markdown
# metadata

- **Название:** <name>
- **Автор:** <автор / источник, адаптация если есть>
- **Версия:** <semver>
- **Описание:** Что делает скилл — 1–2 предложения.
- **Применимость:** Когда использовать, когда не использовать. Ссылки на смежные скиллы.
- **Использование:** Как запустить, какие аргументы принимает, где сохраняет артефакты.
```

При добавлении нового скилла в этот репозиторий — создать `metadata.md` сразу.

> ⚠️ **Безопасность:** это сторонний код из интернета. `SKILL.md` может содержать инструкции, которые Claude
> выполнит (prompt-injection / вызовы `Bash`). **Читай скилл целиком перед установкой.** В каталоге есть скиллы
> для самопроверки: `skill-security-auditor`, `security-scan` (ECC) — прогоняй ими кандидатов из недоверенных источников.

## Источники (папка `sources/`)

| Репозиторий | Скиллов | Профиль | Релевантность стеку |
|---|---:|---|---|
| `claude-skills-main` | ~178 | Широкие «эксперт/pro» персоны по языкам и фреймворкам + meta/engineering | **Высокая** |
| `ECC-main` (Everything Claude Code) | 249 | Узкие композируемые `*-patterns` / `*-testing` / workflow-скиллы | **Высокая** |
| `superpowers-main` | 14 | Инженерный workflow (TDD, планы, дебаг, ревью, worktrees) — стек-агностично | **Высокая** |
| `awesome-claude-skills-master` | ~896 | Сборная солянка; **832 из них — `composio-skills` (API-автоматизации)** | Низкая (кроме нескольких) |
| `engineering-team` (внутри claude-skills) | 32 | «senior-*» роли + интеграции (Stripe, email) | Средняя |
| `andrej-karpathy-skills-main` | 1 | `karpathy-guidelines` — анти-оверинжиниринг | Полезно всем |
| `awesome-claude-code-main` | 0 SKILL.md | Это **список ссылок/ресурсов**, не скиллы. Только для справки. | — |

**Дубли:** темы сильно пересекаются между источниками. Общее правило выбора:
- Нужен **узкий, конкретный гайд под фреймворк** → бери `ECC *-patterns` (точечные, композируемые).
- Нужна **широкая «персона-эксперт»** на весь язык/фреймворк → бери `claude-skills *-expert/-pro`.
- Нужен **процесс/дисциплина** (как работать, а не что писать) → бери `superpowers`.

---

## Классификация по релевантности стеку (React / TS / Node / PostgreSQL)

### 🟢 Tier 1 — Прямое попадание в стек (главные кандидаты на установку)

**Frontend / React / TypeScript**
- `claude-skills/skills`: `react-expert`, `typescript-pro`, `javascript-pro`, `nextjs-developer`, `graphql-architect`, `websocket-engineer`
- `ECC/skills`: `react-patterns`, `react-performance`, `react-testing`, `frontend-patterns`, `frontend-a11y`, `nextjs-turbopack`, `vite-patterns`, `error-handling`, `bun-runtime`, `make-interfaces-feel-better`, `frontend-design-direction`, `design-system`, `motion-foundations` / `motion-patterns` / `motion-advanced` / `motion-ui`
- `engineering-team/skills`: `senior-frontend`, `senior-qa`, `email-template-builder`, `stripe-integration-expert`
- `awesome`: `artifacts-builder`, `webapp-testing`, `theme-factory`

**Backend / Node.js / TypeScript**
- `claude-skills/skills`: `nestjs-expert`, `api-designer`, `graphql-architect`, `websocket-engineer`, `microservices-architect`, `mcp-developer`
- `ECC/skills`: `backend-patterns`, `nestjs-patterns`, `mcp-server-patterns`, `api-design`, `redis-patterns`, `deployment-patterns`, `docker-patterns`, `hexagonal-architecture`, `error-handling`
- `engineering/skills`: `api-design-reviewer`, `api-test-suite-builder`, `mcp-server-builder`, `monorepo-navigator`, `performance-profiler`, `observability-designer`

**Database / PostgreSQL**
- `claude-skills/skills`: `postgres-pro`, `database-optimizer`, `sql-pro`
- `ECC/skills`: `postgres-patterns`, `prisma-patterns`, `database-migrations`, `redis-patterns`
- `engineering/skills`: `database-designer`, `database-schema-designer`, `sql-database-assistant`, `migration-architect`

**Тестирование (TS/JS/web)**
- `claude-skills/skills`: `test-master`, `playwright-expert`
- `ECC/skills`: `react-testing`, `e2e-testing`, `tdd-workflow`, `browser-qa`, `ai-regression-testing`
- `superpowers`: `test-driven-development`
- `engineering-team/skills`: `senior-qa`, `tdd-guide`

### 🟡 Tier 2 — Инженерный процесс и качество (стек-агностично, рекомендую почти всё)

**`superpowers-main` (все 14 — ядро рабочего процесса):** `brainstorming`, `writing-plans`, `executing-plans`,
`subagent-driven-development`, `dispatching-parallel-agents`, `systematic-debugging`, `test-driven-development`,
`requesting-code-review`, `receiving-code-review`, `verification-before-completion`, `using-git-worktrees`,
`finishing-a-development-branch`, `writing-skills`, `using-superpowers`.

**Ревью / качество / отладка**
- `karpathy`: `karpathy-guidelines` (анти-оверинжиниринг — ставить в первую очередь)
- `claude-skills/skills`: `code-reviewer`, `debugging-wizard`, `code-documenter`
- `engineering-team/skills`: `adversarial-reviewer`, `code-reviewer`
- `engineering/skills`: `pr-review-expert`, `focused-fix`, `tech-debt-tracker`, `ship-gate`, `dependency-auditor`, `ci-cd-pipeline-builder`, `spec-driven-workflow`, `env-secrets-manager`
- `ECC/skills`: `coding-standards`, `git-workflow`, `github-ops`, `codebase-onboarding`, `documentation-lookup`, `search-first`, `repo-scan`, `production-audit`, `architecture-decision-records`

**Безопасность (web/backend)**
- `claude-skills/skills`: `security-reviewer`, `secure-code-guardian`, `fullstack-guardian`
- `ECC/skills`: `security-review`, `security-scan`, `security-bounty-hunter`
- `engineering-team/skills`: `senior-security`, `senior-secops`, `ai-security`

**Архитектура / API-дизайн**
- `claude-skills/skills`: `architecture-designer`, `api-designer`, `microservices-architect`
- `engineering/skills`: `senior-architect`, `feature-flags-architect`, `runbook-generator`, `release-manager`

### 🟠 Tier 3 — Смежное / опционально (DevOps, инфраструктура, meta-инструменты)

- **DevOps/инфра (если касаешься деплоя):** `devops-engineer`, `kubernetes-specialist`, `terraform-engineer`,
  `monitoring-expert`, `sre-engineer`, `chaos-engineer` (claude-skills); `docker-patterns`, `deployment-patterns`,
  `slo-architect`, `observability-designer`, `secrets-vault-manager`, `kubernetes-operator` (ECC/engineering); `senior-devops`.
- **Создание/аудит самих скиллов и MCP:** `skill-creator`, `mcp-builder` (awesome); `writing-skills` (superpowers);
  `skill-scout`, `skill-stocktake`, `skill-comply` (ECC); `skill-tester`, `skill-security-auditor` (engineering).
- **Полезные утилиты:** `changelog-generator`, `domain-name-brainstormer`, `file-organizer`, `image-enhancer` (awesome);
  `content-hash-cache-pattern`, `regex-vs-llm-structured-text`, `cost-tracking` (ECC).

### ⚪ Tier 4 — Другие стеки (НЕ для тебя; игнорировать при отборе)
Python (`python-pro`, `fastapi-expert`, `django-*`), Java/Kotlin/Spring (`spring-boot-*`, `quarkus-*`, `kotlin-*`,
`jpa-patterns`), Go (`golang-*`), Rust (`rust-*`), C/C++ (`cpp-*`), C#/.NET (`dotnet-*`, `csharp-*`), PHP/Laravel,
Ruby/Rails, Swift/iOS (`swift*`, `swiftui-*`), Flutter/Dart, Vue/Nuxt/Angular, мобайл (`react-native-*` — только если будет мобайл),
ML/AI-инфра (`ml-pipeline`, `rag-architect`, `pytorch-*`, `fine-tuning-expert`), embedded, game-dev, WordPress/Shopify/Salesforce, blockchain/DeFi/EVM, сети/homelab (`network-*`, `homelab-*`, `cisco-*`), healthcare/HIPAA.

### ⚫ Tier 5 — Не-инженерные (исключить полностью)
Бизнес/маркетинг/контент/финансы/HR/логистика/research/соцсети из `claude-skills` (`marketing`, `finance`,
`business-*`, `c-level-advisor`, `commercial`, `ra-qm-team`, `compliance-os` и т.п.), ECC ops-скиллы
(`*-ops`, `investor-*`, `lead-intelligence`, `ito-*`, prediction-market, `seo`, `content-engine`, видео/Manim/Remotion),
и из `awesome` — `tailored-resume-generator`, `twitter-algorithm-optimizer`, `meeting-insights-analyzer`, `invoice-organizer` и пр.

### 🚫 Bulk — исключить не глядя
`awesome-claude-skills-master/composio-skills/` — **832** однотипных скилла-обёртки над сторонними API
(Zoho, Fillout, Tomba, Veriphone, Dovetail…). Шум. Не разбирать поштучно.

---

## Рекомендуемый минимальный стартовый набор

Если нужно собрать компактный личный набор «здесь и сейчас», ядро такое:

1. **Процесс:** `superpowers/*` (особенно `brainstorming`, `writing-plans`, `test-driven-development`,
   `systematic-debugging`, `requesting-code-review`, `verification-before-completion`, `using-git-worktrees`) + `karpathy-guidelines`.
2. **React/TS фронт:** ECC `react-patterns` + `react-performance` + `react-testing` + `frontend-a11y`; claude-skills `typescript-pro`.
3. **Node бэк:** ECC `backend-patterns` + `nestjs-patterns` (если NestJS) + `api-design`; claude-skills `api-designer`.
4. **БД:** ECC `postgres-patterns` + `prisma-patterns` + `database-migrations`; claude-skills `database-optimizer`.
5. **Качество:** ECC `git-workflow` + `security-review`; engineering `pr-review-expert` + `tech-debt-tracker`.

## Как двигаться дальше

- Отбор по релевантности уже сделан выше (Tier 1–2 = кандидаты, Tier 3 = опционально, 4/5/bulk = отсев).
- Следующий шаг — **прочитать `SKILL.md` финалистов** (особенно где Tier 1 даёт дубли из двух источников — выбрать
  лучший по качеству текста) и **скопировать выбранные папки** в целевую директорию скиллов.
- Команда для просмотра имени+описания любого набора:
  `find sources/<repo>/skills -name SKILL.md` → читать фронтматтер.
