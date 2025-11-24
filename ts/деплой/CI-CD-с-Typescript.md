---
aliases: [Непрерывная интеграция и доставка TypeScript, CI/CD для TypeScript]
tags: [ci, cd, typescript, github-actions, jenkins, gitlab-ci, деплой, автоматизация]
---

# CI/CD с TypeScript

## Обзор

CI/CD (Continuous Integration/Continuous Deployment) - это практика разработки программного обеспечения, при которой изменения кода регулярно объединяются в общую ветку и автоматически тестируются и развертываются. Для TypeScript-проектов CI/CD особенно важен, так как позволяет автоматизировать компиляцию, тестирование и деплой приложений с типобезопасностью.

## Основы CI/CD для TypeScript

### Что такое CI/CD

- **Continuous Integration (CI)**: Практика частого объединения изменений кода в общую ветку с автоматической проверкой
- **Continuous Deployment (CD)**: Автоматическое развертывание кода в production после успешного прохождения всех проверок

### Преимущества CI/CD для TypeScript

- **Раннее обнаружение ошибок**: Типизация позволяет находить ошибки на этапе компиляции
- **Автоматизация тестирования**: Убедитесь, что все тесты проходят перед деплоем
- **Быстрое развертывание**: Автоматический деплой после успешного CI
- **Консистентность**: Одинаковые процессы сборки и тестирования для всех разработчиков
- **Улучшенное качество кода**: Автоматические проверки стиля и линтинга

## Основные этапы CI/CD для TypeScript

### 1. Установка зависимостей

```bash
npm install
# или
yarn install
```

### 2. Линтинг кода

```bash
npm run lint
# или
yarn lint
```

### 3. Компиляция TypeScript

```bash
npm run build
# или
yarn build
```

### 4. Тестирование

```bash
npm test
# или
yarn test
```

### 5. Деплой

```bash
npm run deploy
# или
yarn deploy
```

## Популярные инструменты CI/CD

### GitHub Actions

GitHub Actions - это платформа CI/CD, встроенная в GitHub. Вот пример конфигурации для TypeScript-проекта:

**.github/workflows/ci.yml**:
```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
    - uses: actions/checkout@v4

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Lint code
      run: npm run lint

    - name: Compile TypeScript
      run: npm run build

    - name: Run tests
      run: npm test -- --coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
```

**.github/workflows/cd.yml**:
```yaml
name: CD

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build application
      run: npm run build

    - name: Deploy to production
      run: |
        # Замените на ваш процесс деплоя
        echo "Deploying to production..."
        # npm run deploy
```

### GitLab CI

GitLab CI использует файл `.gitlab-ci.yml` для определения конвейера:

```yaml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

before_script:
  - apt-get update -qq && apt-get install -y -qq git
  - node --version
  - npm --version

install_dependencies:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm ci
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

lint:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm run lint

type_check:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm run type-check

test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm test -- --coverage
  coverage: '/Lines\s*:\s*(\d+.\d+\%)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: node:$NODE_VERSION
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

deploy:
  stage: deploy
  image: node:$NODE_VERSION
  script:
    - echo "Deploying to production..."
    - npm run deploy
  only:
    - main
  when: manual
```

### Jenkins

Jenkins требует установки плагинов для работы с Node.js и TypeScript. Пример pipeline:

```groovy
pipeline {
    agent any

    tools {
        nodejs "NodeJS-18"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Type Check') {
            steps {
                sh 'npm run type-check'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'npm run deploy'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

## Практические примеры

### Пример package.json для CI/CD

```json
{
  "name": "typescript-ci-cd-example",
  "version": "1.0.0",
  "scripts": {
    "prebuild": "npm run clean",
    "build": "tsc --project tsconfig.prod.json",
    "build:dev": "tsc --project tsconfig.dev.json",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "type-check": "tsc --noEmit",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "clean": "rimraf dist",
    "start": "node dist/index.js",
    "start:dev": "nodemon --exec ts-node src/index.ts",
    "deploy": "echo 'Deploying application...'"
  },
  "devDependencies": {
    "@types/jest": "^29.0.0",
    "@types/node": "^18.0.0",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "eslint": "^8.0.0",
    "jest": "^29.0.0",
    "nodemon": "^2.0.0",
    "rimraf": "^3.0.0",
    "ts-jest": "^29.0.0",
    "ts-node": "^10.0.0",
    "typescript": "^4.8.0"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### Пример конфигурации TypeScript для CI/CD

**tsconfig.json**:
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
    "**/*.spec.ts",
    "**/*.test.ts"
  ]
}
```

**tsconfig.prod.json**:
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "removeComments": true,
    "sourceMap": false
  },
  "exclude": [
    "node_modules",
    "dist",
    "**/*.spec.ts",
    "**/*.test.ts",
    "**/*.mock.ts"
  ]
}
```

### Пример конфигурации ESLint для TypeScript

**.eslintrc.js**:
```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
  },
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
  ],
  rules: {
    '@typescript-eslint/explicit-function-return-type': 'warn',
    '@typescript-eslint/explicit-module-boundary-types': 'warn',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': 'error',
    'no-console': 'warn',
  },
  env: {
    node: true,
    jest: true,
  },
};
```

### Пример конфигурации Jest для TypeScript

**jest.config.js**:
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/index.ts',
    '!src/types/**/*',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  globals: {
    'ts-jest': {
      tsconfig: 'tsconfig.json',
    },
  },
};
```

## Лучшие практики CI/CD для TypeScript

### 1. Тип-чекинг как часть CI

Всегда включайте проверку типов в процесс CI:

```bash
npm run type-check
```

Это позволяет находить ошибки типизации до запуска тестов.

### 2. Параллельное выполнение задач

Разделяйте задачи на независимые этапы, которые можно выполнять параллельно:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      # ...

  type-check:
    runs-on: ubuntu-latest
    steps:
      # ...

  test:
    runs-on: ubuntu-latest
    steps:
      # ...
```

### 3. Кэширование зависимостей

Кэшируйте зависимости для ускорения сборки:

```yaml
- name: Cache node modules
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

### 4. Использование матриц для тестирования

Тестируйте приложение на разных версиях Node.js:

```yaml
strategy:
  matrix:
    node-version: [16.x, 18.x, 20.x]
```

### 5. Покрытие кода

Отслеживайте покрытие кода тестами и устанавливайте минимальные пороги:

```yaml
- name: Run tests with coverage
  run: npm test -- --coverage --coverage-threshold='{"global":{"lines":80,"statements":80}}'
```

## Безопасность в CI/CD

### Управление секретами

Не храните секреты в коде. Используйте встроенные механизмы CI/CD:

- GitHub Secrets
- GitLab CI/CD Variables
- Jenkins Credentials Store

### Сканирование уязвимостей

Добавьте сканирование уязвимостей в зависимости:

```bash
npm audit
# или
yarn audit
```

### Проверка лицензий

Проверяйте лицензии зависимостей:

```bash
npx license-checker --summary
```

## Мониторинг и уведомления

### Уведомления о статусе сборки

Настройте уведомления о статусе сборки в Slack, Discord или другие мессенджеры.

### Метрики CI/CD

Отслеживайте метрики CI/CD:

- Время выполнения сборки
- Частота сбоев
- Среднее время до исправления (MTTR)

## Интеграция с Docker

### CI/CD с Docker и TypeScript

```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

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
          tags: ${{ secrets.DOCKER_USERNAME }}/my-ts-app:${{ github.sha }}
```

## Заключение

CI/CD для TypeScript-проектов обеспечивает автоматизированный процесс проверки, сборки и развертывания приложений с учетом преимуществ строгой типизации. Правильная настройка CI/CD позволяет улучшить качество кода, ускорить разработку и повысить надежность приложений.

Следуя лучшим практикам, описанным в этом руководстве, вы сможете эффективно настроить CI/CD для своих TypeScript-проектов.

## См. также

- [[Docker]]
- [[Kubernetes]]
- [[Облачные-сервисы]]
- [[Автоматизация-деплоя]]
- [[Тестирование-в-TypeScript]]
- [[Безопасность-TypeScript-приложений]]