# Claude Code - Инструкции проекта

## Язык

- Документация и комментарии на русском

## Git Workflow

### Структура веток
- `master` — production (мердж только через PR)
- `dev` — основная ветка разработки
- `feature/*` — новые фичи
- `hotfix/*` — критические исправления

### Формат коммитов

```
feat: добавлена авторизация
fix: исправлена валидация формы
docs: обновлена документация API
refactor: рефакторинг модуля auth
test: добавлены тесты для профиля
chore: обновлены зависимости
```

## Специализированные агенты

Проект использует систему специализированных агентов для разных типов задач. Агенты находятся в `.claude/agents/` и вызываются через Task tool.

| Агент | Когда использовать |
|-------|-------------------|
| `project-initializer` | Первый запуск, создание структуры проекта |
| `project-documentation-engineer` | Документация, CHANGELOG, архитектура |
| `git-workflow-specialist` | Git операции, ветки, коммиты, PR |
| `backend-specialist` | Backend логика, API, база данных |
| `ui-ux-designer` | UI компоненты, формы, стили |
| `code-reviewer` | Code review перед мержем |
| `test-engineer` | Unit, integration, E2E тесты |
| `deployment-specialist` | Деплой, CI/CD, мониторинг |

### Как работают агенты

Агенты вызываются через **Task tool**, а не просто читаются как документация.

```
Задача: "Создай форму авторизации"

1. Вызывает ui-ux-designer через Task для создания UI
2. Вызывает backend-specialist через Task для API
3. Может запустить агенты параллельно для ускорения
```

### Явный вызов агента

Пользователь может явно указать агента:
```
"Используй git-workflow-specialist для создания feature ветки"
"Вызови code-reviewer для проверки перед коммитом"
```

## Task Dashboard (Kanban)

**URL:** https://claude-task-dashboard-seven.vercel.app/

Real-time Kanban дашборд для визуализации задач проекта. Настраивается автоматически агентом `project-initializer`.

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

### ОБЯЗАТЕЛЬНОЕ использование Kanban

**Claude Code ОБЯЗАН при каждой сессии работы:**

1. **Проверить** наличие `.claude/config/kanban.json` при старте. Если файла нет — вызвать `project-initializer`.
2. **Выписать все задачи** текущей сессии в TodoWrite **ДО начала работы**.
3. **Синхронизировать каждое обновление** TodoWrite с webhook из `kanban.json`.
4. **Обновлять статусы в реальном времени** — `pending` → `in_progress` → `completed`.
5. **Никогда не работать без задач в Kanban** — если задача не записана, она не существует.

**Все агенты ОБЯЗАНЫ:**
- Использовать TodoWrite для отслеживания своих подзадач
- Группировать задачи тегами (backend, frontend, docs, test, deploy)
- Обновлять статусы задач при каждом переходе

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
      "tags": ["backend", "api"]
    }
  ]
}
```

## Правила

### Всегда

- Строгая типизация, избегать `any`
- Валидация входных данных на границах системы
- Тесты для критичной логики
- Code review перед мерджем

### Никогда

- Не мерджить в `master` напрямую
- Не коммитить секреты (.env, credentials, ключи)
- Не деплоить без прохождения тестов
