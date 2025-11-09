# Деплой TypeScript приложений

Деплой TypeScript приложений включает в себя компиляцию TypeScript в JavaScript, управление зависимостями, оптимизацию кода и развертывание на различных платформах. В этом документе рассмотрены стратегии и лучшие практики деплоя TypeScript приложений.

## Процесс деплоя TypeScript приложений

### Основные этапы

1. **Компиляция TypeScript** - преобразование .ts файлов в .js
2. **Управление зависимостями** - установка продакшен зависимостей
3. **Оптимизация** - минификация, дерево утилизации (tree-shaking)
4. **Тестирование** - выполнение тестов перед деплоем
5. **Развертывание** - загрузка на целевую платформу

### Типичная конфигурация сборки

```json
// package.json
{
  "scripts": {
    "build": "tsc --project tsconfig.prod.json",
    "build:dev": "tsc --watch",
    "predeploy": "npm run build",
    "deploy": "gh-pages -d dist"
  }
}
```

```json
// tsconfig.prod.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": false,
    "removeComments": true,
    "declaration": false
  },
  "exclude": [
    "**/*.test.ts",
    "**/*.spec.ts",
    "tests"
  ]
}
```

## Компиляция и подготовка к деплою

### Компиляция TypeScript

```bash
# Компиляция с использованием конфигурации для продакшена
tsc --project tsconfig.prod.json

# Или с помощью скрипта
npm run build
```

### Генерация объявлений типов

Для библиотек важно также генерировать .d.ts файлы:

```json
// tsconfig.json для библиотеки
{
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": false,
    "outDir": "./dist"
  }
}
```

## Платформы развертывания

### Веб-приложения

#### В традиционные хостинги

Для фронтенд приложений:

```bash
# Сборка для веба
npm run build
# Результат в папке dist/, который затем загружается на хостинг
```

#### Netlify

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"
  environment = { NODE_VERSION = "18" }
```

#### Vercel

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/node",
      "config": { "includeFiles": ["dist/**"] }
    }
  ],
  "routes": [
    { "src": "/(.*)", "dest": "/dist/$1" }
  ]
}
```

#### GitHub Pages

```bash
# Установка gh-pages
npm install -g gh-pages

# Деплой
npm run deploy
```

### Node.js приложения

#### На собственные серверы

```bash
# На сервере
git pull origin main
npm install --production
npm run build
pm2 start dist/server.js
```

#### Heroku

```yaml
# Procfile
web: node dist/server.js

# package.json
{
  "scripts": {
    "prebuild": "npm install --production=false",
    "build": "tsc",
    "postbuild": "npm install --production=true"
  }
}
```

#### Docker

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

#### AWS (ECS, Lambda, etc.)

```typescript
// serverless.ts
import type { AWS } from '@serverless/typescript';

const serverlessConfiguration: AWS = {
  service: 'my-ts-service',
  frameworkVersion: '3',
  plugins: ['serverless-esbuild', 'serverless-offline'],
  provider: {
    name: 'aws',
    runtime: 'nodejs18.x',
    apiGateway: {
      minimumCompressionSize: 1024,
      shouldStartNameWithService: true,
    },
    environment: {
      AWS_NODEJS_CONNECTION_REUSE_ENABLED: '1',
    },
  },
  functions: {
    api: {
      handler: 'dist/lambda.handler',
    },
  },
};

module.exports = serverlessConfiguration;
```

### Cloudflare Workers

```typescript
// wrangler.toml
name = "my-ts-worker"
type = "javascript"
account_id = "your-account-id"
workers_dev = true
route = ""
zone_id = ""

# Компиляция и деплой
# npm run build && wrangler publish
```

## Управление зависимостями

### Продакшен vs разработка

```json
// package.json
{
  "dependencies": {
    "express": "^4.18.0",
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "typescript": "^4.8.0",
    "ts-node": "^10.0.0",
    "@types/express": "^4.17.13",
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0"
  }
}
```

При деплое устанавливайте только продакшен зависимости:

```bash
npm ci --only=production
```

### Управление версиями

```dockerfile
# Использование специфичных версий для воспроизводимости
FROM node:18.17.0-alpine
```

## Оптимизация перед деплоем

### Удаление неиспользуемого кода (Tree Shaking)

```javascript
// webpack.config.js
module.exports = {
  mode: 'production',
  optimization: {
    usedExports: true,
    sideEffects: false
  }
};

// Используйте именованные импорты для лучшего tree-shaking
// Вместо: import * as _ from 'lodash'
// Используйте: import { debounce, throttle } from 'lodash';
```

### Минификация

```javascript
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [new TerserPlugin({
      terserOptions: {
        compress: {
          drop_console: true, // Удаление console.log
          drop_debugger: true
        }
      }
    })]
  }
};
```

### Управление размером бандла

```bash
# Анализ размера бандла
npm install -g webpack-bundle-analyzer
npx webpack-bundle-analyzer dist/static/js/*.js
```

## CI/CD для TypeScript приложений

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Build
      run: npm run build
    
    - name: Deploy
      run: |
        # Скрипт деплоя на ваш хостинг
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm ci
    - npm test

build:
  stage: build
  image: node:$NODE_VERSION
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:
  stage: deploy
  image: node:$NODE_VERSION
  dependencies:
    - build
  script:
    - echo "Deploying application..."
    # Скрипт деплоя
  only:
    - main
```

## Мониторинг и логирование

### Логирование в продакшене

```typescript
// logger.ts
interface Logger {
  info(message: string, meta?: any): void;
  error(message: string, meta?: any): void;
  warn(message: string, meta?: any): void;
  debug(message: string, meta?: any): void;
}

class ProductionLogger implements Logger {
  info(message: string, meta?: any): void {
    console.log(`[INFO] ${message}`, meta);
  }

  error(message: string, meta?: any): void {
    console.error(`[ERROR] ${message}`, meta);
    // Отправка в систему мониторинга
  }

  // и т.д.
}
```

### Обработка ошибок в продакшене

```typescript
// error-handler.ts
import express from 'express';

export const errorHandler: express.ErrorRequestHandler = (err, req, res, next) => {
  console.error(err.stack);
  
  // В продакшене отправка ошибки в систему трассировки
  if (process.env.NODE_ENV === 'production') {
    // reportErrorToMonitoringService(err);
  }

  res.status(500).json({
    message: 'Internal server error',
    ...(process.env.NODE_ENV !== 'production' && { stack: err.stack })
  });
};
```

## Специфические особенности разных сред

### Frontend приложения

Для фронтенд приложений TypeScript компилируется в браузерный JavaScript:

```json
// tsconfig.json для фронтенда
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "es6"],
    "module": "esnext",
    "moduleResolution": "node",
    "jsx": "react-jsx",
    "sourceMap": true,
    "removeComments": false,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

### Backend приложения

Для бэкенда TypeScript компилируется для Node.js:

```json
// tsconfig.json для бэкенда
{
  "compilerOptions": {
    "target": "ES2018", 
    "module": "commonjs",
    "lib": ["es2018"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### Библиотеки

Для библиотек важно обеспечить совместимость:

```json
// tsconfig.json для библиотеки
{
  "compilerOptions": {
    "target": "ES2018",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["es2018", "dom"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "stripInternal": true
  }
}
```

## Современные тенденции

### Edge deployment

Развертывание TypeScript приложений на edge-серверах:

```typescript
// Cloudflare Worker
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    // TypeScript код выполняется на edge
    return new Response('Hello World');
  },
} satisfies ExportedHandler<Env>;
```

### Serverless

Использование TypeScript с serverless архитектурами:

```bash
# Установка serverless framework
npm install -g serverless

# Создание TypeScript функции
serverless create --template aws-nodejs-typescript --path my-service
```

### Micro-frontends

Развертывание независимых фронтенд модулей:

```typescript
// Микрофронтенд с типами
interface UserModule {
  mount(element: HTMLElement): void;
  unmount(): void;
}

export const userModule: UserModule = {
  mount(element) {
    // Логика монтирования
  },
  unmount() {
    // Логика демонтирования
  }
};
```

## Лучшие практики

1. **Разделяйте конфигурации** для разработки и продакшена
2. **Используйте Docker** для воспроизводимости среды
3. **Оптимизируйте размер бандла** с помощью tree-shaking и минификации
4. **Настройте CI/CD** для автоматического тестирования и деплоя
5. **Используйте семантическое версионирование** для библиотек
6. **Добавьте health check endpoint** для мониторинга
7. **Логируйте ошибки** в централизованную систему
8. **Используйте переменные окружения** для конфигурации

## Устранение неполадок

### Распространенные ошибки при деплое

1. **Отсутствующие зависимости**:
   - Убедитесь, что все зависимости указаны в `dependencies`, а не `devDependencies`

2. **Неправильные пути**:
   - Проверьте настройки `baseUrl` и `paths` в `tsconfig.json`

3. **Проблемы с типами**:
   - Убедитесь, что .d.ts файлы корректно сгенерированы и включены в пакет

4. **Размер бандла**:
   - Используйте анализаторы бандла для выявления "тяжелых" зависимостей

## Связь с другими концепциями

- [[tooling-workflows/deployment-workflows]] - рабочие процессы деплоя
- [[configuration/production-config]] - конфигурация для продакшена
- [[modules/bundling]] - упаковка модулей для деплоя
- [[error-handling/production-errors]] - обработка ошибок в продакшене