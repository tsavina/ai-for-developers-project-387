# AGENTS.md

## О проекте

Booking-сервис: SPA для бронирования слотов в календаре. Учебный проект Hexlet.
Деплой: Render.com (`Dockerfile`, `render.yaml`).

## Стек

| Слой | Технологии |
|---|---|
| Frontend | React 19 + TypeScript 6 + Vite 8 + Mantine v9 + react-router-dom v7 |
| Backend | Node.js (ESM) + Express 5 (plain JavaScript) |
| API spec | TypeSpec → OpenAPI 3.0 (`calendar-api/`) |
| Тесты | Playwright (API + E2E) |
| CI | GitHub Actions (4 workflows) |

## Команды

```bash
cd frontend && npm run dev     # Vite dev + Prism mock API
cd frontend && npm run build   # tsc + vite build
cd backend && npm run dev      # Express с --watch
cd backend && npm start        # Express production
cd tests && npm test           # Playwright (API + E2E)
cd tests && npm run test:ui    # Playwright UI mode
```

## Конвенции кода

### Frontend
- Компоненты: `export default function`, PascalCase файлы, `interface Props` в начале
- Импорты: React → Mantine → react-router → локальные компоненты → API → `import type`
- Типы: `import type { ... } from '../types'`, не `import { type ... }`
- Дёрганье данных: `.then()/.catch()/.finally()`, не async/await в useEffect
- UI-текст на русском, код/комментарии на английском
- Страницы админки — в `pages/admin/`, импорты через `../../`

### Backend
- ES Modules: `import`/`export default`
- Маршруты: Express `Router`, раздельные файлы для public/admin
- Ошибки: `next({ status, code, message, suggestedNext? })`
- Валидация полей — manual, без библиотек

### Тесты
- `test.describe` для группировки, `beforeEach` с `DELETE /api/admin/reset`
- API-тесты через `{ request }`, E2E через `{ page }`

## Архитектурные инварианты

### Frontend: направление зависимостей

```
App → pages/ → api/ + components/ → types/
```

- `types/` — базовый слой, не импортирует ничего внутри проекта
- `api/` — импортирует ТОЛЬКО `types/`, ничего из components/pages
- `components/` — импортирует ТОЛЬКО `types/` и библиотеки
- `pages/` — может импортировать `components/`, `api/`, `types/`
- `App.tsx` — импортирует `pages/` и `components/` (не api/ или types/ напрямую)

**Запрещённые зависимости (фронтенд):**
- ❌ `pages/*` не импортируют друг друга
- ❌ `components/*` не импортируют `pages/` или `api/`
- ❌ `api/*` не импортирует `components/` или `pages/`
- ❌ `types/` не импортирует внутренние модули

### Backend: направление зависимостей

```
index.js → app.js → routes/ → store.js + slotService.js
                  ↗ middleware/errorHandler.js
```

- `store.js` — зависит только от `uuid` (leaf-модуль данных)
- `slotService.js` — чистые функции, без импортов из проекта (leaf-модуль бизнес-логики)
- `middleware/errorHandler.js` — без импортов из проекта (горизонтальный слой)
- `routes/` — могут импортировать `store.js` и `slotService.js`
- `app.js` — импортирует `routes/` и `middleware/`, НЕ импортирует `store.js` или `slotService.js` напрямую

**Запрещённые зависимости (бэкенд):**
- ❌ `store.js` не импортирует `routes/`, `slotService.js`, `middleware/`
- ❌ `slotService.js` не импортирует `store.js`, `routes/`, `middleware/`
- ❌ `middleware/` не импортирует `store.js`, `routes/`, `slotService.js`
- ❌ `routes/admin.js` и `routes/public.js` не импортируют друг друга
- ❌ `app.js` не импортирует `store.js` или `slotService.js` напрямую

### Cross-cutting

- Фронтенд ↔ Бэкенд: только через HTTP REST API (`/api/*`)
- `api/client.ts` — единственный слой, выполняющий HTTP-запросы
- `types/` на фронтенде зеркалит структуру ответов бэкенда
- Бэкенд раздаёт статику через `express.static`; SPA fallback для не-API запросов

## Важно

- In-memory storage: данные сбрасываются при рестарте бэкенда
- В dev-режиме запущены 2 API: Prism mock (из openapi.yaml) и реальный Express. Реальный бэкенд на порту 4010
- TypeScript — только на фронтенде; бэкенд на чистом JS без сборки
- Не редактировать `hexlet-check.yml` — он auto-generated
