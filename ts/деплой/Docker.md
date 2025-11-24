---
aliases: [Docker для TypeScript, Контейнеризация TypeScript]
tags: [docker, typescript, деплой, контейнеризация, devops]
---

# Docker для TypeScript-приложений

## Обзор

Docker - это платформа для разработки, доставки и запуска приложений в контейнерах. Для TypeScript-приложений Docker особенно полезен, так как позволяет создавать изолированные, воспроизводимые среды выполнения, что критически важно для стабильного деплоя.

## Основы контейнеризации TypeScript

TypeScript - это язык программирования, который компилируется в JavaScript. При создании Docker-образов для TypeScript-приложений важно учитывать, что компиляция может происходить как внутри контейнера, так и вне его. Каждый подход имеет свои преимущества.

### Преимущества использования Docker для TypeScript

- **Воспроизводимость**: Одинаковое поведение приложения в разных средах
- **Изоляция**: Каждое приложение работает в своей изолированной среде
- **Упрощение деплоя**: Легковесные контейнеры легко развертываются
- **Управление зависимостями**: Все зависимости упакованы в образ
- **Масштабируемость**: Легко масштабировать приложение с помощью orchestration-инструментов

## Создание Dockerfile для TypeScript-приложения

### Базовый Dockerfile

```dockerfile
# Используем официальный образ Node.js
FROM node:18-alpine

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем package.json и yarn.lock (если используется)
COPY package.json yarn.lock* ./

# Устанавливаем зависимости
RUN yarn install --frozen-lockfile

# Копируем исходный код
COPY . .

# Компилируем TypeScript
RUN yarn build

# Экспонируем порт
EXPOSE 3000

# Запускаем приложение
CMD ["yarn", "start"]
```

### Оптимизированный Dockerfile с многоступенчатой сборкой

Для уменьшения размера конечного образа рекомендуется использовать многоступенчатую сборку:

```dockerfile
# Стадия компиляции
FROM node:18-alpine AS builder

WORKDIR /app
COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile

COPY . .
RUN yarn build

# Стадия запуска
FROM node:18-alpine AS runtime

WORKDIR /app

# Устанавливаем зависимости только для продакшена
COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile --production

# Копируем скомпилированный код из стадии builder
COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

## Настройка TypeScript для работы в Docker

### tsconfig.json для контейнера

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "newLine": "lf",
    "preserveConstEnums": true,
    "noEmitOnError": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.spec.ts"
  ]
}
```

## Практические примеры

### Пример Express-приложения на TypeScript в Docker

Создадим простое Express-приложение:

**src/index.ts**:
```typescript
import express, { Request, Response } from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req: Request, res: Response) => {
  res.json({ message: 'Hello from TypeScript Docker app!' });
});

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

**package.json**:
```json
{
  "name": "typescript-docker-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node src/index.ts"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/node": "^18.0.0",
    "typescript": "^4.7.0",
    "ts-node": "^10.0.0"
  }
}
```

### Docker Compose для разработки

Для разработки можно использовать Docker Compose, чтобы упростить запуск приложения с зависимостями:

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: [
      "/wait-for-it.sh",
      "db:5432",
      "--",
      "yarn",
      "dev"
    ]

  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

**Dockerfile.dev** для разработки:
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile

COPY . .

# Устанавливаем nodemon для горячей перезагрузки
RUN yarn add --dev nodemon ts-node

# Копируем утилиту ожидания сервисов
COPY wait-for-it.sh .
RUN chmod +x wait-for-it.sh

EXPOSE 3000

CMD ["yarn", "dev"]
```

## Лучшие практики

### 1. Использование .dockerignore

Создайте файл `.dockerignore`, чтобы исключить ненужные файлы из контекста сборки:

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.env.local
.env.development.local
.env.test.local
.env.production.local
```

### 2. Использование специфичных тегов образов

Всегда используйте специфичные теги образов (например, `node:18-alpine`) вместо `latest`, чтобы обеспечить воспроизводимость.

### 3. Минимизация размера образа

- Используйте Alpine-образы, когда это возможно
- Удаляйте временные файлы после установки зависимостей
- Используйте многоступенчатую сборку

### 4. Безопасность

- Запускайте приложение от не-рутового пользователя
- Регулярно обновляйте базовые образы
- Используйте инструменты для сканирования уязвимостей

## Отладка TypeScript-приложений в Docker

Для отладки TypeScript-приложений в Docker можно настроить отображение исходных файлов и source maps:

**Dockerfile для отладки**:
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile

COPY . .

# Устанавливаем ts-node для отладки
RUN yarn add --dev ts-node typescript

EXPOSE 3000
EXPOSE 9229

CMD ["node", "--inspect=0.0.0.0:9229", "-r", "ts-node/register", "src/index.ts"]
```

## CI/CD с Docker

При интеграции с CI/CD системами Docker позволяет создавать унифицированные конвейеры сборки и деплоя. Вот пример GitHub Actions для сборки и пуша Docker-образа:

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ secrets.DOCKER_USERNAME }}/my-ts-app:latest
```

## Сравнение с другими подходами

| Подход | Преимущества | Недостатки |
|--------|--------------|------------|
| Docker | Воспроизводимость, изоляция, масштабируемость | Сложность первоначальной настройки |
| Традиционный деплой | Простота понимания | Зависимость от среды, сложности с зависимостями |
| Виртуальные машины | Полная изоляция | Высокие накладные расходы |

## Заключение

Docker предоставляет мощный инструмент для деплоя TypeScript-приложений, обеспечивая консистентность, изоляцию и масштабируемость. Правильная настройка Docker-контейнеров позволяет значительно упростить процесс развертывания и управления приложениями.

Следуя лучшим практикам, описанным в этом руководстве, вы сможете эффективно использовать Docker для своих TypeScript-проектов.

## См. также

- [[Kubernetes]]
- [[CI-CD-с-Typescript]]
- [[Облачные-сервисы]]
- [[Автоматизация-деплоя]]
- [[Node.js-с-TypeScript]]