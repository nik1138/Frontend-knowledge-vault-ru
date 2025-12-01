---
aliases: ["Docker для Vue", "Контейнеризация Vue", "Vue Docker"]
tags: [vue, docker, deployment, containerization, container]
---

# Docker для Vue.js приложений

## Обзор

Docker позволяет упаковать Vue.js приложение и все его зависимости в контейнер, что обеспечивает единообразие сред выполнения на различных платформах. Контейнеризация особенно полезна при деплое Vue-приложений, поскольку обеспечивает изоляцию, масштабируемость и простоту развертывания.

## Подготовка Dockerfile для Vue.js

### Для статических приложений

Для стандартных Vue-приложений, которые собираются в статические файлы:

```dockerfile
# Используем официальный образ Node.js
FROM node:20-alpine AS build-stage

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем package.json и package-lock.json
COPY package*.json ./

# Устанавливаем зависимости
RUN npm ci --only=production

# Копируем исходный код
COPY . .

# Собираем приложение
RUN npm run build

# Продукционный сервер (nginx)
FROM nginx:alpine AS production-stage

# Копируем собранные файлы из build-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html

# Копируем конфигурацию nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Открываем порт 80
EXPOSE 80

# Запускаем nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Для SSR-приложений (Nuxt.js)

Для SSR-приложений, таких как Nuxt.js:

```dockerfile
FROM node:20-alpine

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем package.json и package-lock.json
COPY package*.json ./

# Устанавливаем зависимости
RUN npm ci --only=production

# Копируем исходный код
COPY . .

# Собираем приложение
RUN npm run build

# Открываем порт
EXPOSE 3000

# Запускаем приложение
CMD ["node", ".output/server/index.mjs"]
```

## Конфигурация nginx для статических Vue-приложений

Файл `nginx.conf` для контейнера:

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Настройки логирования
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Обработка SPA
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Кэширование статических файлов
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Безопасность
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
    }
}
```

## Docker Compose для разработки и деплоя

Файл `docker-compose.yml` для локальной разработки:

```yaml
version: '3.8'

services:
  # Vue-приложение
  vue-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "80:80"
    volumes:
      - ./dist:/usr/share/nginx/html
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    
  # nginx для проксирования (опционально)
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./dist:/usr/share/nginx/html
    depends_on:
      - vue-app
    restart: unless-stopped

  # Redis для кэширования (для SSR-приложений)
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped

volumes:
  redis-data:
```

### Для SSR-приложений

```yaml
version: '3.8'

services:
  nuxt-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NUXT_HOST=0.0.0.0
      - NUXT_PORT=3000
    restart: unless-stopped
    
  # Обратный прокси
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx-ssr.conf:/etc/nginx/nginx.conf
    depends_on:
      - nuxt-app
    restart: unless-stopped

  # База данных (если требуется)
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: vue_app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres-data:
```

## Российские особенности использования Docker

### Регистри локальных образов

В 2025 году, учитывая санкционные ограничения, рекомендуется использовать локальные Docker-регистры:

- [[Яндекс.Облако]] Container Registry
- [[МегаФон.Облако]] Container Registry
- [[Ростелеком.Облако]] Container Registry
- Внутренние корпоративные регистры

### Альтернативы Docker

- Podman (как замена Docker CLI)
- LXC/LXD
- OpenVZ

### Ограничения и обходные пути

- Использование альтернативных репозиториев образов
- Создание собственных образов на основе локальных зеркал
- Использование proxy-серверов для доступа к Docker Hub

## Оптимизация Docker-образов

### Multi-stage сборка

Использование многоступенчатой сборки для уменьшения размера итогового образа:

```dockerfile
# Stage 1: Сборка
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci && npm cache clean --force

COPY . .
RUN npm run build

# Stage 2: Продукционный образ
FROM nginx:alpine AS production

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Использование Alpine Linux

Использование легковесных базовых образов Alpine:

```dockerfile
FROM node:20-alpine
# или
FROM nginx:alpine
```

### Игнорирование ненужных файлов

Файл `.dockerignore`:

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
.coverage
.coverage/
.DS_Store
```

## CI/CD с Docker

### GitHub Actions

Пример workflow для сборки и публикации Docker-образа:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Login to Registry
        uses: docker/login-action@v3
        with:
          registry: cr.yandex
          username: ${{ secrets.YANDEX_REGISTRY_USERNAME }}
          password: ${{ secrets.YANDEX_REGISTRY_PASSWORD }}
          
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: cr.yandex/your-id/vue-app:${{ github.sha }}
```

## Безопасность Docker-контейнеров

### Пользователи в контейнерах

Запуск приложения не от root-пользователя:

```dockerfile
# Создание пользователя
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Изменение владельца файлов
COPY --chown=nextjs:nodejs . .

# Переключение на пользователя
USER nextjs
```

### Ограничение ресурсов

В docker-compose.yml:

```yaml
services:
  vue-app:
    build: .
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'
```

## Мониторинг контейнеров

### Логирование

Использование централизованного логирования:

- ELK Stack (Elasticsearch, Logstash, Kibana)
- Fluentd
- Graylog

### Метрики

- Prometheus + Grafana
- [[Мониторинг]] с использованием cAdvisor
- Docker Stats

## Связанные темы

- [[Vue.js]] - основы фреймворка
- [[Статический-деплой]] - альтернативный подход
- [[SSR-деплой]] - серверный рендеринг
- [[CI-CD]] - автоматизация процесса деплоя
- [[Мониторинг]] - отслеживание состояния контейнеров

## Заключение

Docker предоставляет мощный инструмент для контейнеризации Vue.js приложений, обеспечивая единообразие сред выполнения, изоляцию и простоту развертывания. При правильной настройке и учете российских реалий 2025 года, контейнеризация становится неотъемлемой частью процесса деплоя современных веб-приложений.