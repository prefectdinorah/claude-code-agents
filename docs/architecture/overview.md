# Архитектура Task Dashboard

## Обзор

Task Dashboard использует клиент-серверную архитектуру с real-time обновлениями через WebSocket.

## Компоненты системы

### 1. WebSocket Server (`server/index.ts`)

Отдельный Node.js процесс, который:
- Слушает WebSocket подключения на порту 3051
- Следит за изменениями `tasks.json` через Chokidar
- Рассылает обновления всем подключённым клиентам

**Протокол сообщений:**

```typescript
// Типы сообщений
type WSMessage = {
  type: 'init' | 'update' | 'ping' | 'pong'
  data?: TasksData
  timestamp: string
}

// При подключении клиента
{ type: 'init', data: TasksData, timestamp: string }

// При изменении tasks.json
{ type: 'update', data: TasksData, timestamp: string }

// Keep-alive
{ type: 'ping' } → { type: 'pong', timestamp: string }
```

### 2. Next.js Frontend (`app/`, `components/`)

React приложение с Server Components (где возможно) и Client Components для интерактивности.

**Структура компонентов:**

```
app/
├── layout.tsx          # Root layout (Server Component)
├── page.tsx            # Главная страница (Client Component)
└── globals.css         # Глобальные стили

components/
├── Header.tsx          # Шапка с статусом
├── Board.tsx           # Kanban-доска
├── Column.tsx          # Колонка задач
└── TaskCard.tsx        # Карточка задачи
```

### 3. WebSocket Hook (`hooks/useWebSocket.ts`)

Кастомный React hook для управления WebSocket соединением:
- Автоматическое подключение при монтировании
- Автореконнект при разрыве соединения (3 сек)
- Ping-pong каждые 30 сек для keep-alive
- Возвращает: `{ data, isConnected, lastUpdate }`

## Поток данных

```
┌───────────────────────────────────────────────────────────────┐
│                                                               │
│   Claude Code CLI                                             │
│        │                                                      │
│        │ TodoWrite tool                                       │
│        ▼                                                      │
│   ┌─────────────┐                                            │
│   │ tasks.json  │ ◄─── Chokidar watch                        │
│   └──────┬──────┘                                            │
│          │                                                    │
│          │ file change event                                  │
│          ▼                                                    │
│   ┌──────────────────┐                                       │
│   │ WebSocket Server │                                       │
│   │    (port 3051)   │                                       │
│   └────────┬─────────┘                                       │
│            │                                                  │
│            │ broadcast({ type: 'update', data })             │
│            ▼                                                  │
│   ┌──────────────────┐      ┌──────────────────┐            │
│   │ Browser Client 1 │      │ Browser Client N │            │
│   │   useWebSocket   │      │   useWebSocket   │            │
│   └────────┬─────────┘      └──────────────────┘            │
│            │                                                  │
│            │ setData(message.data)                           │
│            ▼                                                  │
│   ┌──────────────────┐                                       │
│   │      Board       │                                       │
│   │   ┌──────────┐   │                                       │
│   │   │ Column[] │   │                                       │
│   │   │TaskCard[]│   │                                       │
│   │   └──────────┘   │                                       │
│   └──────────────────┘                                       │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

## Стилизация

### Дизайн-система

Тёмная тема в стиле Linear с палитрой:

| Цвет | Значение | Использование |
|------|----------|---------------|
| background | #0a0a0b | Фон страницы |
| surface | #141415 | Фон карточек |
| border | #27272a | Границы |
| status-todo | #71717a | Задачи To Do |
| status-progress | #3b82f6 | Задачи In Progress |
| status-done | #22c55e | Выполненные задачи |

### Анимации

- `fadeIn` — плавное появление элементов
- `slideUp` — появление снизу вверх
- `pulseSubtle` — пульсация для активных задач
- `layout` (Framer Motion) — анимация при перемещении
