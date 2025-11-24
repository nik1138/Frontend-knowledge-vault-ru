---
aliases: [CI/CD для фронтенд-приложений, Continuous Integration, Continuous Deployment, Непрерывная интеграция и доставка]
tags: [frontend, deployment, architecture, ci-cd, automation, best-practices, github-actions, gitlab-ci]
---

# CI-CD

## Обзор

CI/CD (Continuous Integration/Continuous Deployment) - это практика разработки программного обеспечения, которая заключается в автоматизации процессов интеграции и доставки кода. Для фронтенд-приложений CI/CD обеспечивает быструю и надежную доставку изменений пользователям с минимальным ручным вмешательством.

## Основы CI/CD для фронтенд-приложений

### Что такое CI/CD?

- **Continuous Integration (CI)**: Автоматическая сборка и тестирование кода при каждом изменении
- **Continuous Delivery (CD)**: Автоматическая подготовка приложения к деплою
- **Continuous Deployment (CD)**: Полностью автоматический деплой в продакшн

### Зачем нужен CI/CD для фронтенда?

- **Быстрая обратная связь**: Немедленное уведомление о проблемах в коде
- **Качество кода**: Автоматические проверки стиля, линтинг, тестирование
- **Безопасность**: Сканирование уязвимостей зависимостей
- **Скорость доставки**: Быстрое и надежное развертывание изменений
- **Консистентность**: Одинаковый процесс сборки и деплоя для всех разработчиков

## Архитектура CI/CD пайплайна

### Типичные этапы CI/CD для фронтенд-приложений

1. **Checkout**: Получение исходного кода из репозитория
2. **Install Dependencies**: Установка зависимостей (npm install, yarn install)
3. **Linting**: Проверка стиля кода
4. **Testing**: Запуск юнит-тестов, интеграционных тестов, e2e тестов
5. **Building**: Сборка приложения (npm run build)
6. **Security Scanning**: Проверка уязвимостей в зависимостях
7. **Deployment**: Деплой на предпродакшн или продакшн
8. **Notification**: Уведомление команды о результатах

## Популярные CI/CD инструменты

### GitHub Actions

#### Базовый workflow для фронтенд-приложения

```yaml
# .github/workflows/frontend-ci.yml
name: Frontend CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run tests
      run: npm run test -- --coverage

    - name: Run build
      run: npm run build

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: frontend
        name: frontend-coverage
```

#### Продвинутый workflow с деплоем

```yaml
# .github/workflows/frontend-cd.yml
name: Frontend CD

on:
  push:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test-and-build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.vars.outputs.version }}

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm run test

    - name: Run linting
      run: npm run lint

    - name: Build application
      run: npm run build

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Log in to container registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}

    - name: Extract version
      id: vars
      run: echo "version=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: test-and-build
    runs-on: ubuntu-latest
    environment: staging

    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying version ${{ needs.test-and-build.outputs.version }} to staging"
        # Add your deployment commands here
        # For example, updating Kubernetes deployment
        kubectl set image deployment/frontend-app frontend-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ needs.test-and-build.outputs.version }} --namespace staging

  deploy-production:
    needs: [test-and-build, deploy-staging]
    runs-on: ubuntu-latest
    environment: production
    if: github.ref == 'refs/heads/main'

    steps:
    - name: Deploy to production
      run: |
        echo "Deploying version ${{ needs.test-and-build.outputs.version }} to production"
        # Add your production deployment commands here
        kubectl set image deployment/frontend-app frontend-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest --namespace production
```

### GitLab CI

#### .gitlab-ci.yml для фронтенд-приложения

```yaml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"
  CONTAINER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  CONTAINER_IMAGE_LATEST: $CI_REGISTRY_IMAGE:latest

before_script:
  - apt-get update -qy
  - apt-get install -y curl

.test_template: &test_definition
  image: node:$NODE_VERSION
  stage: test
  cache:
    key: "$CI_COMMIT_REF_SLUG"
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run lint
    - npm run test:coverage
  coverage: '/Statements\s*:\s*([^%]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    expire_in: 1 week
    paths:
      - coverage/

unit_test:
  <<: *test_definition
  script:
    - npm ci
    - npm run lint
    - npm run test:unit

integration_test:
  <<: *test_definition
  script:
    - npm ci
    - npm run test:integration

build_image:
  stage: build
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  variables:
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CONTAINER_IMAGE .
    - docker tag $CONTAINER_IMAGE $CONTAINER_IMAGE_LATEST
    - docker push $CONTAINER_IMAGE
    - docker push $CONTAINER_IMAGE_LATEST
  only:
    - main
    - develop
  except:
    - schedules

deploy_staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - echo "$KUBE_CONFIG" | base64 -d > kubeconfig
    - kubectl --kubeconfig=kubeconfig set image deployment/frontend-app frontend-app=$CONTAINER_IMAGE --namespace staging
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - echo "$KUBE_CONFIG" | base64 -d > kubeconfig
    - kubectl --kubeconfig=kubeconfig set image deployment/frontend-app frontend-app=$CONTAINER_IMAGE_LATEST --namespace production
  environment:
    name: production
  when: manual
  only:
    - main
```

### Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any

    tools {
        nodejs "NodeJS-18"
    }

    environment {
        REGISTRY = 'my-registry.com'
        IMAGE_NAME = 'frontend-app'
        NAMESPACE = 'production'
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
            parallel {
                stage('ESLint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                stage('StyleLint') {
                    steps {
                        sh 'npm run stylelint'
                    }
                }
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit -- --coverage'
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
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level high'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", 'docker-registry-credentials') {
                        docker.image("${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}").push()
                        docker.image("${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl set image deployment/frontend-app frontend-app=${env.REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} --namespace ${env.NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "Something is wrong with ${env.BUILD_URL}",
                to: "dev-team@example.com"
            )
        }
    }
}
```

## Лучшие практики CI/CD для фронтенда

### 1. Используйте кеширование зависимостей

```yaml
# GitHub Actions
- name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: '18'
    cache: 'npm'  # Кеширует node_modules

- name: Install dependencies
  run: npm ci
  if: steps.node-cache.outputs.cache-hit != 'true'
```

### 2. Параллельное выполнение тестов

```yaml
# Разделение тестов для параллельного выполнения
test_unit:
  runs-on: ubuntu-latest
  steps:
    - run: npm run test:unit

test_integration:
  runs-on: ubuntu-latest
  steps:
    - run: npm run test:integration

test_e2e:
  runs-on: ubuntu-latest
  steps:
    - run: npm run test:e2e
```

### 3. Используйте контейнеризацию для консистентности

```dockerfile
# Dockerfile для CI
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
RUN npm install -g npm@latest

COPY . .
RUN npm run build

CMD ["npm", "start"]
```

### 4. Контроль качества кода

```yaml
# Включение проверок качества в CI
- name: Run ESLint
  run: npm run lint

- name: Run StyleLint
  run: npm run stylelint

- name: Run Type Checking
  run: npm run type-check

- name: Run Security Audit
  run: npm audit --audit-level moderate
```

### 5. Проверка производительности

```yaml
# Добавление проверок производительности
- name: Run Lighthouse CI
  run: |
    npx lighthouse-ci action --config=./lighthouserc.json
  env:
    LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

## Продвинутые возможности

### Условные деплои

```yaml
# GitHub Actions - деплой только при изменениях в определенных файлах
- name: Deploy only if frontend files changed
  run: |
    if git diff --name-only ${{ env.PREV_COMMIT }} ${{ env.CURRENT_COMMIT }} | grep -E '\.(js|jsx|ts|tsx|css|scss|html)$'; then
      echo "Frontend files changed, proceeding with deployment"
      # выполнить деплой
    else
      echo "No frontend files changed, skipping deployment"
    fi
```

### Canary деплоймент

```yaml
# Пример canary деплоя в GitHub Actions
- name: Deploy Canary
  run: |
    kubectl apply -f k8s/canary-deployment.yaml
    kubectl rollout status deployment/frontend-canary
    
- name: Run Canary Tests
  run: npm run test:canary
  
- name: Promote Canary to Production
  if: success()
  run: |
    kubectl apply -f k8s/main-deployment.yaml
    kubectl rollout status deployment/frontend-main
```

### Уведомления и мониторинг

```yaml
# Уведомления о статусе CI/CD
- name: Notify Slack on Failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    channel: '#deployments'
    text: 'Frontend deployment failed!'
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Связанные темы

- [[Стратергии-деплоя]]
- [[Контейнеризация]]
- [[Оркестрация]]
- [[Мониторинг-после-деплоя]]

## Теги

#frontend #deployment #architecture #ci-cd #automation #github-actions #gitlab-ci #jenkins #best-practices #testing #security