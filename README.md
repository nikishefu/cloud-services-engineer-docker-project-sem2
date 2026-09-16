# momo-store

Демо-приложение пельменной: backend на Go (`:8081`) и frontend на Vue 3, собранный в статику и раздаваемый
через nginx (хостовый порт `80`).

## Запуск 

Предусмотрено два профиля в Docker Compose: dev и prod

### dev
Профиль публикует порт backend на хосте, по умолчанию `:8081`
```sh
COMPOSE_PROFILES=dev docker compose up --build -d
```

Завершение работы:
```sh
COMPOSE_PROFILES=dev docker compose down --remove-orphans
```

### prod
В этом профиле backend доступен только по внутренней сети и проксируется через /api
```sh
COMPOSE_PROFILES=prod docker compose up --build -d
```

Также в профиле prod возможно масштабирование:
```sh
COMPOSE_PROFILES=prod docker compose up --build -d --scale backend=3
```

Завершение работы:
```sh
COMPOSE_PROFILES=prod docker compose down --remove-orphans
```

## Образы

Оба Dockerfile — multi-stage: тяжёлый тулчейн (Go-компилятор / Node+npm) остаётся в билд-стадии, в финальный
образ попадает только результат. Backend — статический бинарник поверх `alpine` (~40 МБ), frontend —
собранная статика поверх `nginx-unprivileged` (~76 МБ). Для сравнения, сами неоптимизированные базовые
образы (`golang:1.17`, `node:14`) весят по 1.3+ ГБ каждый.

## Конфигурация

`VUE_APP_API_URL` — build arg для frontend, зашивается в фронтенд на этапе сборки. У backend'а
конфигурируемых параметров пока нет.

Публикуемые хостовые порты настраиваются через `.env` (или переменные окружения):

- `FRONTEND_PORT` (по умолчанию `80`),
- `BACKEND_PORT` (по умолчанию `8081`).

## Безопасность

Оба контейнера: без root, `cap_drop: [ALL]`, `no-new-privileges`, read-only корневая ФС, лимиты CPU/памяти.

Образы собираются и сканируются Trivy в CI (`.github/workflows/deploy.yaml`) перед пушем в DockerHub —
сканирование сейчас информативное, пуш не блокирует.

В самом приложении нет секретов; данные для DockerHub
лежат в GitHub Actions Secrets.
