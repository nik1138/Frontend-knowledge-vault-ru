---
aliases: ["CI-CD-Security", "Continuous-Integration-Security"]
tags: [security, ci-cd, devops-security]
---

# Безопасность CI/CD

## Введение

Безопасность непрерывной интеграции и доставки (CI/CD) - это критически важный аспект разработки программного обеспечения, охватывающий защиту всех этапов автоматизированного процесса от коммита кода до развертывания в продакшен. CI/CD пайплайны обрабатывают исходный код, зависимости, секреты и конечные артефакты, что делает их привлекательной целью для атакующих.

## Архитектура CI/CD и угрозы

### 1. Компоненты CI/CD пайплайна

Типичный CI/CD пайплайн включает следующие компоненты:

- **Система контроля версий** (Git, SVN)
- **Сервер CI/CD** (GitHub Actions, GitLab CI, Jenkins, CircleCI)
- **Репозиторий артефактов** (Docker Registry, npm registry)
- **Тестовая среда**
- **Продакшен среда**
- **Мониторинг и алертинг**

### 2. Классификация угроз CI/CD

Основные категории угроз безопасности CI/CD:

- **Угрозы аутентификации**: Компрометация учетных данных
- **Угрозы авторизации**: Несанкционированный доступ к ресурсам
- **Угрозы целостности**: Внедрение вредоносного кода
- **Утечка данных**: Раскрытие секретов и конфиденциальной информации
- **Угрозы инфраструктуры**: Компрометация серверов CI/CD

## Управление секретами в CI/CD

### 1. Хранение секретов

Секреты никогда не должны храниться в коде или конфигурационных файлах:

```yaml
# .github/workflows/deploy.yml - ПЛОХОЙ ПРИМЕР
name: Deploy
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          curl -X POST https://api.example.com/deploy \
            -H "Authorization: Bearer ${{ secrets.SECRET_TOKEN }}" \
            -d "key=hardcoded_secret"  # НИКОГДА НЕ ДЕЛАЙТЕ ТАК!
```

### 2. Безопасное использование секретов

Правильное использование секретов в CI/CD:

```yaml
# .github/workflows/deploy.yml - ХОРОШИЙ ПРИМЕР
name: Deploy
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write  # Для OIDC
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Deploy to production
      run: |
        echo "${{ secrets.DEPLOY_KEY }}" > deploy_key
        chmod 600 deploy_key
        # Использование секрета в безопасной манере
        ssh -i deploy_key user@server 'deploy-script.sh'
      env:
        API_KEY: ${{ secrets.API_KEY }}
```

### 3. Управление секретами с помощью внешних хранилищ

Использование внешних хранилищ секретов:

```yaml
# Использование AWS Secrets Manager
name: Secure Build
on: [push]

jobs:
  secure-build:
    runs-on: ubuntu-latest
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: ${{ secrets.AWS_ROLE }}
        aws-region: us-east-1
    
    - name: Get secrets from AWS Secrets Manager
      id: secrets
      run: |
        echo "DB_PASSWORD=$(aws secretsmanager get-secret-value --secret-id myapp/db --query SecretString --output text)" >> $GITHUB_OUTPUT
      env:
        AWS_DEFAULT_REGION: us-east-1
    
    - name: Use secret in build
      run: echo "Using database password..."
      env:
        DB_PASSWORD: ${{ steps.secrets.outputs.DB_PASSWORD }}
```

## Безопасная конфигурация CI/CD

### 1. GitHub Actions

Безопасная настройка GitHub Actions:

```yaml
# .github/workflows/secure-build.yml
name: Secure Build
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Ограничение разрешений
permissions:
  contents: read  # Только чтение содержимого
  packages: write # Только необходимые разрешения

jobs:
  build:
    runs-on: ubuntu-latest
    
    # Использование конкретных версий действий
    steps:
    - uses: actions/checkout@v4  # Указание конкретной версии
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies securely
      run: npm ci --only=production
    
    - name: Audit dependencies
      run: |
        npm audit --audit-level moderate
        if [ $? -ne 0 ]; then
          echo "Vulnerabilities found!"
          exit 1
        fi
    
    - name: Run security scan
      run: npx snyk test --severity-threshold=medium
```

### 2. GitLab CI

Безопасная настройка GitLab CI:

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  # Безопасные переменные по умолчанию
  NODE_ENV: production

.test_template: &test_definition
  image: node:18-alpine
  before_script:
    - npm ci --only=production
  script:
    - npm test
    - npm run security-audit

build_job:
  <<: *test_definition
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  # Ограничение доступа к артефактам
  when: on_success

deploy_job:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - echo "Deploying to production..."
  # Запуск только при ручном подтверждении
  when: manual
  environment:
    name: production
    url: https://example.com
```

## Защита от атак в CI/CD

### 1. Защита от вредоносных коммитов

Проверка коммитов на наличие вредоносного кода:

```bash
#!/bin/bash
# pre-commit hook для проверки безопасности

# Проверка на наличие секретов в коде
git diff --cached -U0 | grep -E "(password|secret|token|key)" | grep -E "^\+.*=" > /tmp/secrets_found

if [ -s /tmp/secrets_found ]; then
    echo "Найдены потенциальные секреты в коммите:"
    cat /tmp/secrets_found
    exit 1
fi

# Проверка на наличие вредоносных паттернов
git diff --cached | grep -E "(eval\(|Function\(|setTimeout\([^']*function|setInterval\([^']*function)" > /tmp/malicious_patterns

if [ -s /tmp/malicious_patterns ]; then
    echo "Найдены потенциально вредоносные паттерны:"
    cat /tmp/malicious_patterns
    exit 1
fi

exit 0
```

### 2. Защита от атак с помощью веток

Ограничение выполнения CI/CD для непроверенных веток:

```yaml
# GitHub Actions с ограничением веток
name: Secure Pipeline
on:
  push:
    branches: 
      - main
      - develop
      - release/*
    tags:
      - 'v*'
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

jobs:
  security-check:
    runs-on: ubuntu-latest
    # Ограничение выполнения для внешних форков
    if: github.event.pull_request.head.repo.full_name == github.repository
    steps:
      - uses: actions/checkout@v4
      - name: Security scan
        run: |
          # Проверка безопасности
          echo "Running security checks..."
```

### 3. Защита от DoS атак

Ограничение ресурсов и времени выполнения:

```yaml
# Ограничение ресурсов в GitHub Actions
name: Resource-Constrained Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30  # Ограничение времени выполнения
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Build with resource limits
      run: |
        # Ограничение использования памяти и CPU
        ulimit -v 1048576  # 1GB виртуальная память
        ulimit -t 1800     # 30 минут CPU time
        npm run build
```

## Аудит и мониторинг CI/CD

### 1. Логирование безопасности

Реализация безопасного логирования в CI/CD:

```javascript
// Пример безопасного логирования в скриптах CI/CD
class SecureLogger {
  constructor() {
    this.sensitivePatterns = [
      /api[_-]?key/i,
      /secret/i,
      /password/i,
      /token/i,
      /auth/i,
      /[A-Z_]*KEY[A-Z_]*/i
    ];
  }
  
  log(level, message, data = {}) {
    const safeData = this.sanitizeData(data);
    console.log(`[${new Date().toISOString()}] [${level}] ${message}`, safeData);
  }
  
  sanitizeData(data) {
    if (typeof data !== 'object' || data === null) {
      return data;
    }
    
    const sanitized = Array.isArray(data) ? [] : {};
    
    for (const [key, value] of Object.entries(data)) {
      if (this.isSensitiveKey(key)) {
        sanitized[key] = '[REDACTED]';
      } else if (typeof value === 'string') {
        sanitized[key] = this.sanitizeValue(value);
      } else if (typeof value === 'object') {
        sanitized[key] = this.sanitizeData(value);
      } else {
        sanitized[key] = value;
      }
    }
    
    return sanitized;
  }
  
  sanitizeValue(value) {
    for (const pattern of this.sensitivePatterns) {
      if (pattern.test(value)) {
        return '[REDACTED]';
      }
    }
    return value;
  }
  
  isSensitiveKey(key) {
    return this.sensitivePatterns.some(pattern => pattern.test(key));
  }
}

// Использование безопасного логгера
const logger = new SecureLogger();
logger.log('info', 'Build started', { 
  commit: process.env.GITHUB_SHA,
  branch: process.env.GITHUB_REF_NAME
});
```

### 2. Мониторинг аномалий

Мониторинг аномальных действий в CI/CD:

```yaml
# GitHub Actions для мониторинга аномалий
name: Anomaly Detection
on:
  workflow_run:
    workflows: ["Build and Test"]
    types: [completed]

jobs:
  anomaly-detection:
    runs-on: ubuntu-latest
    if: github.event.workflow_run.event == 'push'
    steps:
    - name: Check for anomalies
      run: |
        # Проверка продолжительности выполнения
        DURATION=$(curl -s -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
          "${{ github.event.workflow_run.url }}" | jq '.run_duration_ms')
        
        if [ $DURATION -gt 3600000 ]; then  # 1 час
          echo "ALERT: Workflow took too long: $DURATION ms"
          # Отправка алерта
        fi
        
        # Проверка количества изменений в коммите
        CHANGES=$(curl -s -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
          "${{ github.event.workflow_run.repository.url }}/commits/${{ github.event.workflow_run.head_sha }}" | \
          jq '.files | length')
        
        if [ $CHANGES -gt 100 ]; then
          echo "ALERT: Unusually large commit: $CHANGES files changed"
        fi
```

## Безопасность образов и контейнеров

### 1. Безопасные Docker-образы

Создание безопасных Docker-образов в CI/CD:

```dockerfile
# Dockerfile для безопасного образа
FROM node:18-alpine AS base

# Создание непривилегированного пользователя
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Установка зависимостей в безопасной среде
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Копирование кода
COPY --from=0 --chown=nextjs:nodejs /app /app
WORKDIR /app

# Переключение на непривилегированного пользователя
USER nextjs

EXPOSE 3000
CMD ["npm", "start"]
```

### 2. Сканирование образов

Сканирование Docker-образов на уязвимости:

```yaml
# GitHub Actions для сканирования образов
name: Image Security Scan
on: [push]

jobs:
  scan-image:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
```

## Лучшие практики CI/CD безопасности

### 1. Принцип наименьших привилегий

Ограничьте разрешения для CI/CD систем:

```yaml
# Пример минимальных необходимых разрешений
permissions:
  contents: read          # Только чтение содержимого
  deployments: write      # Только развертывание
  statuses: write         # Только обновление статусов
  # НЕ используйте permissions: write-all
```

### 2. Использование OIDC для аутентификации

Использование OpenID Connect вместо токенов:

```yaml
# GitHub Actions с OIDC
name: OIDC Deployment
on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # Необходимо для OIDC
      contents: read
    
    steps:
    - name: Assume AWS role with OIDC
      uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: arn:aws:iam::123456789012:role/GitHubActionRole
        aws-region: us-east-1
        role-session-name: GitHubActionSession
    
    - name: Deploy to AWS
      run: |
        aws s3 sync dist/ s3://my-bucket/
```

### 3. Защита артефактов

Обеспечьте защиту и целостность артефактов:

```yaml
# Защита артефактов в GitHub Actions
name: Secure Artifacts
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Build application
      run: npm run build
    
    - name: Create artifact signature
      run: |
        tar -czf app.tar.gz dist/
        openssl dgst -sha256 -sign signing.key -out app.tar.gz.sig app.tar.gz
    
    - name: Upload signed artifact
      uses: actions/upload-artifact@v3
      with:
        name: app-with-signature
        path: |
          app.tar.gz
          app.tar.gz.sig
        retention-days: 7
```

## Заключение

Безопасность CI/CD требует комплексного подхода, включающего:

- Безопасное управление секретами
- Ограничение разрешений
- Мониторинг и аудит
- Защиту от атак
- Регулярные обновления и проверки

> [!tip] Помните
> CI/CD пайплайны - это критическая инфраструктура доставки, и их безопасность должна быть приоритетом наравне с безопасностью конечного продукта.

## Связанные темы

- [[Безопасность-систем-сборки]]
- [[Управление-зависимостями]]
- [[Безопасность-процесса-сборки]]
- [[Аудит-CI-CD-безопасности]]