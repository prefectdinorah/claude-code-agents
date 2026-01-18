# Task Dashboard

## Описание проекта

**Task Dashboard** — это real-time дашборд для мониторинга задач Claude Code. Приложение отображает Kanban-доску с задачами, которые автоматически обновляются через WebSocket при изменении файла `tasks.json`.

## Назначение

Дашборд предназначен для визуализации прогресса работы Claude Code агента:
- Отслеживание текущих задач в реальном времени
- Kanban-доска с тремя колонками (To Do, In Progress, Done)
- Автоматические обновления без перезагрузки страницы
- Индикация активных задач с анимацией

## Технологический стек

| Технология | Назначение |
|------------|-----------|
| Next.js 14 | Frontend framework (App Router) |
| React 18 | UI библиотека |
| TypeScript | Типизация |
| Tailwind CSS | Стилизация |
| Framer Motion | Анимации |
| @dnd-kit | Drag & Drop (планируется) |
| WebSocket (ws) | Real-time обновления |
| Chokidar | Отслеживание изменений файла |

## Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                     Task Dashboard                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    WebSocket     ┌─────────────────────┐  │
│  │   Browser   │ ◄──────────────► │   WebSocket Server  │  │
│  │  (Next.js)  │    :3051         │     (Node.js)       │  │
│  │   :3050     │                  │                     │  │
│  └─────────────┘                  └──────────┬──────────┘  │
│                                              │              │
│                                    Chokidar  │ watch       │
│                                              ▼              │
│                                   ┌─────────────────────┐  │
│                                   │     tasks.json      │  │
│                                   │  (Claude Code CLI)  │  │
│                                   └─────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Структура данных

### Task

```typescript
interface Task {
  id: string
  content: string          // Описание задачи
  status: TaskStatus       // pending | in_progress | completed
  activeForm: string       // Что делается сейчас (для in_progress)
  createdAt: string        // ISO timestamp
  updatedAt: string        // ISO timestamp
  project?: string         // Проект (опционально)
  tags?: string[]          // Теги (опционально)
}
```

### tasks.json

```json
{
  "project": "Project Name",
  "lastUpdated": "2025-01-17T03:55:00.000Z",
  "tasks": [
    {
      "id": "task-1",
      "content": "Создать документацию",
      "status": "in_progress",
      "activeForm": "Создаю документацию проекта",
      "createdAt": "2025-01-17T03:00:00.000Z",
      "updatedAt": "2025-01-17T03:55:00.000Z"
    }
  ]
}
```

## Порты

| Сервис | Порт |
|--------|------|
| Next.js (Frontend) | 3050 |
| WebSocket Server | 3051 |

## Запуск

```bash
cd task-dashboard
npm install
npm run dev
```

Откроется дашборд на http://localhost:3050
