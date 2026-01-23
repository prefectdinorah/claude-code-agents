# Claude Code - Инструкции проекта

## Технологический стек

- **Frontend:** Next.js 14+ (App Router, Server Components)
- **Backend:** Supabase (Auth, Database, RLS, Storage)
- **Styling:** Tailwind CSS + shadcn/ui
- **Language:** TypeScript (strict mode)
- **Deploy:** Vercel
- **Язык документации:** Русский

## Специализированные агенты

Проект использует систему специализированных агентов для разных типов задач. Агенты находятся в `.claude/agents/` и вызываются через Task tool.

| Агент | Когда использовать | Вызов |
|-------|-------------------|-------|
| `project-initializer` | **Первый запуск!** Создание docs/, настройка Kanban | При начале нового проекта |
| `project-documentation-engineer` | Документация, CHANGELOG, архитектура | Автоматически для задач документирования |
| `git-workflow-specialist` | Git операции, ветки, коммиты, PR | Автоматически для git задач |
| `nextjs-supabase-specialist` | Backend, Server Actions, Supabase | Автоматически для backend задач |
| `ui-ux-designer-nextjs` | UI компоненты, формы, Tailwind | Автоматически для UI/UX задач |
| `code-reviewer-nextjs` | Code review перед мержем | Автоматически перед коммитом |
| `test-engineer-nextjs` | Unit, integration, E2E тесты | Автоматически после написания кода |
| `vercel-deployment-specialist` | Деплой, CI/CD, мониторинг | Автоматически для задач деплоя |

### Как работают агенты

**ВАЖНО:** Агенты вызываются через **Task tool**, а не просто читаются как документация.

**Примеры использования:**

```
Задача: "Создай форму авторизации"

Claude Code:
1. Вызывает ui-ux-designer-nextjs через Task для создания UI
2. Вызывает nextjs-supabase-specialist через Task для backend
3. Может запустить агенты параллельно для ускорения
```

```
Задача: "Задеплой на Vercel"

Claude Code:
1. Сначала вызывает code-reviewer-nextjs для проверки
2. Затем вызывает git-workflow-specialist для коммита
3. Наконец вызывает vercel-deployment-specialist для деплоя
```

### Мои обязанности (Claude Code)

**При работе над задачами я ДОЛЖЕН:**

1. **Проверить инициализацию проекта** — если нет `docs/` или `.claude/config/kanban.json`, вызвать `project-initializer`
2. **Определить тип задачи** и выбрать подходящего агента
3. **Использовать Task tool** для вызова специализированного агента
4. **Запускать агенты параллельно** когда это возможно (UI + Backend)
5. **Всегда вызывать code-reviewer** перед коммитом
6. **Всегда вызывать test-engineer** после написания кода
7. **Синхронизировать задачи с Kanban** — при обновлении TodoWrite отправлять webhook

**Примеры вызовов:**

```typescript
// ПЕРВЫЙ ЗАПУСК — инициализация проекта
Task(subagent_type: "project-initializer",
     prompt: "Инициализируй проект 'НазваниеПроекта'")

// Для UI задачи
Task(subagent_type: "ui-ux-designer-nextjs",
     prompt: "Создай форму регистрации с валидацией")

// Для Backend задачи
Task(subagent_type: "nextjs-supabase-specialist",
     prompt: "Создай Server Action для регистрации")

// Для Git задачи
Task(subagent_type: "git-workflow-specialist",
     prompt: "Создай feature ветку и закоммить изменения")
```

### Явный вызов агента

Пользователь может явно указать агента:
```
"Используй git-workflow-specialist для создания feature ветки"
"Вызови code-reviewer для проверки перед коммитом"
```

## Git Workflow

### Структура веток
- `master` — production (мердж только вручную)
- `dev` — разработка
- `feature/*` — новые фичи
- `hotfix/*` — критические исправления

### Процесс работы

1. **Планирование** → Спецификация в `docs/features/`
2. **Разработка** → Feature ветка из `dev`
3. **Тестирование** → Coverage минимум 80%
4. **Code Review** → Проверка перед мерджем
5. **Мердж в dev** → После одобрения
6. **Документация** → Обновить CHANGELOG

### Формат коммитов

```
feat: добавлена JWT авторизация
fix: исправлена проблема с RLS
docs: обновлена документация API
refactor: рефакторинг auth модуля
test: добавлены тесты для профиля
```

## Структура документации

```
docs/                          # Создаётся агентом документации
├── project.md                 # Описание проекта
├── context.md                 # Текущий контекст работы
├── tasks.md                   # Задачи со статусами
├── CHANGELOG.md               # История изменений
├── architecture/              # Архитектура
├── database/                  # Схемы БД
├── api/                       # API документация
└── features/                  # Спецификации фич
```

## Task Dashboard (Kanban)

**URL:** https://claude-task-dashboard-seven.vercel.app/

Real-time Kanban дашборд для визуализации задач проекта. **Настраивается автоматически агентом `project-initializer`.**

### Как это работает

```
┌─────────────────────────────────────────────────────────────┐
│  1. project-initializer создаёт проект на дашборде          │
│  2. Webhook URL сохраняется в .claude/config/kanban.json    │
│  3. При каждом TodoWrite → данные отправляются на webhook   │
│  4. Владелец видит прогресс в реальном времени              │
└─────────────────────────────────────────────────────────────┘
```

### Конфигурация

После инициализации проекта создаётся файл `.claude/config/kanban.json`:

```json
{
  "projectName": "Название проекта",
  "dashboardUrl": "https://claude-task-dashboard-seven.vercel.app/",
  "webhookUrl": "https://claude-task-dashboard-seven.vercel.app/api/webhook/PROJECT_ID",
  "createdAt": "2025-01-20T00:00:00.000Z"
}
```

### Формат данных webhook

```json
{
  "project": "Название проекта",
  "lastUpdated": "2025-01-20T17:00:00.000Z",
  "tasks": [
    {
      "id": "task-1",
      "content": "Описание задачи",
      "status": "pending | in_progress | completed",
      "activeForm": "Что делается сейчас",
      "createdAt": "2025-01-20T16:00:00.000Z",
      "updatedAt": "2025-01-20T17:00:00.000Z",
      "tags": ["backend", "api"]
    }
  ]
}
```

### Интеграция с агентами

**Все агенты ДОЛЖНЫ:**
- Использовать TodoWrite для отслеживания своих задач
- Группировать задачи тегами (backend, frontend, docs, test, deploy)
- Обновлять статусы задач в реальном времени

**Claude Code ДОЛЖЕН:**
- Проверять наличие `.claude/config/kanban.json` при старте
- Если файла нет → вызвать `project-initializer`
- При каждом TodoWrite → синхронизировать с webhook

## Авто-одобренные команды

См. [.claude/config/auto-approved-commands.md](.claude/config/auto-approved-commands.md)

## Правила

### Всегда

- TypeScript strict mode, нет `any`
- Server Components по умолчанию
- RLS включен для всех таблиц Supabase
- Валидация данных через Zod
- Комментарии и документация на русском
- Тесты для критичной логики

### Никогда

- Не мерджить в `master` напрямую
- Не коммитить секреты (.env, credentials)
- Не пропускать code review
- Не деплоить без тестов

## Уведомления

```bash
npm run notify "Заголовок" "Сообщение"
```

Когда отправлять:
- После завершения этапа разработки
- При критических проблемах
- После успешного деплоя

## Быстрые команды

```bash
npm run dev          # Локальная разработка
npm run build        # Сборка
npm run test         # Тесты
npm run lint         # Линтинг
npm run type-check   # Проверка типов
```
