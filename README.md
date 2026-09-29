# telegram-bot

Node.js + TypeScript boilerplate: strict-типизация, линтер, тесты, Docker и CI.

## Стек

| Слой           | Что используется                                                |
| -------------- | --------------------------------------------------------------- |
| Runtime        | Node.js 24 LTS, native ESM (`"type": "module"`)                 |
| Язык           | TypeScript 6, `strict` + дополнительные флаги                   |
| Линтер         | ESLint 10 (flat config) + typescript-eslint `strictTypeChecked` |
| Форматирование | Prettier 3                                                      |
| Тесты          | Vitest 5 + `@vitest/coverage-v8`                                |
| Валидация env  | Zod 4                                                           |
| Логи           | Pino 10 (+ pino-pretty в dev)                                   |
| Контейнер      | multi-stage Dockerfile на `node:24-alpine`, non-root, tini      |

## Быстрый старт

```bash
npm install
cp .env.example .env
npm run dev
```

Сервис поднимется на `http://localhost:3000`, health-check — `GET /health` → `{"status":"ok"}`.

## Скрипты

| Команда                 | Назначение                             |
| ----------------------- | -------------------------------------- |
| `npm run dev`           | Запуск с hot-reload (tsx watch)        |
| `npm run build`         | Компиляция в `dist/`                   |
| `npm start`             | Запуск собранного билда                |
| `npm run typecheck`     | `tsc --noEmit`                         |
| `npm run lint` / `:fix` | ESLint                                 |
| `npm run format`        | Prettier --write                       |
| `npm test`              | Тесты                                  |
| `npm run test:watch`    | Тесты в watch-режиме                   |
| `npm run test:coverage` | Тесты + coverage c порогами            |
| `npm run verify`        | format:check → lint → typecheck → test |
| `npm run clean`         | Удаление `dist/` и `coverage/`         |
| `npm run docker:build`  | Сборка образа                          |
| `npm run docker:run`    | Запуск контейнера с `.env`             |

## Конфигурация

Все переменные окружения валидируются Zod в `src/config.ts` — при ошибке процесс падает
на старте с понятным сообщением, а не в рантайме.

| Переменная            | Тип                                     | По умолчанию   |
| --------------------- | --------------------------------------- | -------------- |
| `NODE_ENV`            | `development` \| `test` \| `production` | `development`  |
| `LOG_LEVEL`           | `trace`…`fatal`                         | `info`         |
| `SERVICE_NAME`        | string                                  | `telegram-bot` |
| `HOST`                | string                                  | `0.0.0.0`      |
| `PORT`                | 1–65535                                 | `3000`         |
| `SHUTDOWN_TIMEOUT_MS` | положительное целое                     | `10000`        |
| `TELEGRAM_BOT_TOKEN`  | string (опционально)                    | —              |

Пример — `.env.example`. Файл `.env` в git не попадает.

## Структура

```
src/
  index.ts    точка входа, graceful shutdown по SIGINT/SIGTERM
  config.ts   схема и валидация env
  logger.ts   фабрика Pino с redact секретов
  server.ts   HTTP-сервер и /health
test/         юнит- и интеграционные тесты
```

## Docker

```bash
npm run docker:build
npm run docker:run
# или
docker compose up --build
```

Образ multi-stage: стадии `deps` → `build` → `prod-deps` → `runtime`.
В рантайме только production-зависимости, непривилегированный пользователь
(`uid 10001`), `tini` как PID 1, встроенный `HEALTHCHECK` и ротация логов в compose.

## CI

`.github/workflows/ci.yml` — три джобы на каждый push и PR в `main`:

1. **verify** — format check, lint, typecheck, тесты с coverage-порогами;
2. **build** — сборка и smoke-тест `node dist/index.js` через `/health`;
3. **docker** — сборка образа с GHA cache и проверка health-check в контейнере.

`concurrency` отменяет предыдущий прогон для того же ref, есть `timeout-minutes`
на каждой джобе. Dependabot обновляет npm-зависимости и Actions раз в неделю.

## Строгий TypeScript

Включены не только `strict`, но и `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`,
`noImplicitOverride`, `noPropertyAccessFromIndexSignature`, `erasableSyntaxOnly`,
`verbatimModuleSyntax`. Это ломает часть кода, который на обычных настройках молча
проходит, — так ошибки не доезжают до прода.

## Лицензия

MIT — см. [LICENSE](./LICENSE).
