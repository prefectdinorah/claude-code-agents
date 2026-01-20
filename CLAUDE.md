# Claude Code - Инструкции проекта

## Технологический стек

- **Frontend:** Next.js 14+ (App Router, Server Components)
- **Backend:** Supabase (Auth, Database, RLS, Storage)
- **Styling:** Tailwind CSS + shadcn/ui
- **Language:** TypeScript (strict mode)
- **Deploy:** Vercel
- **Язык документации:** Русский

## Агенты (База знаний)

Инструкции для разных типов задач находятся в `.claude/agents/`:

| Файл | Когда использовать |
|------|-------------------|
| `project-documentation-engineer.md` | Документация, CHANGELOG, архитектура |
| `git-workflow-specialist.md` | Git операции, ветки, коммиты |
| `nextjs-supabase-specialist.md` | Backend, Server Actions, Supabase |
| `ui-ux-designer-nextjs.md` | UI компоненты, формы, Tailwind |
| `code-reviewer-nextjs.md` | Code review перед мержем |
| `test-engineer-nextjs.md` | Unit, integration, E2E тесты |
| `vercel-deployment-specialist.md` | Деплой, CI/CD, мониторинг |

### Как работают агенты

Это **не автозапуск**. Это база знаний, которую я использую:

```
Пользователь: "Создай форму авторизации"

Claude Code:
1. Читает ui-ux-designer-nextjs.md + nextjs-supabase-specialist.md
2. Применяет паттерны из этих инструкций
3. Может запустить параллельные Task для ускорения
```

### Явный вызов агента

Если хочешь использовать конкретного агента:
```
"Используя инструкции из git-workflow-specialist.md, создай feature ветку для авторизации"
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

## Task Dashboard

**URL:** https://claude-task-dashboard-seven.vercel.app/

Real-time Kanban дашборд для визуализации задач проекта. Каждый проект должен иметь свой дашборд для отслеживания прогресса работы.

### Настройка для проекта

1. **Создать проект на дашборде:**
   - Перейди на https://claude-task-dashboard-seven.vercel.app/
   - Вкладка "Create New"
   - Введи название проекта
   - Скопируй Webhook URL

2. **Настроить webhook (пока вручную):**
   - После каждого обновления задач отправляй POST на webhook URL
   - Формат данных:
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

### Мои обязанности (Claude Code)

**ВАЖНО:** При работе над задачами я должен:

1. **Использовать TodoWrite** для всех нетривиальных задач
2. **Отслеживать прогресс** через todo list
3. **Синхронизировать статусы** с Task Dashboard через webhook
4. **Группировать задачи** логически с использованием тегов

Это помогает владельцу проекта видеть прогресс в реальном времени через красивый UI.

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
