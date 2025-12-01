---
aliases: ["CI/CD для Vue", "Непрерывная интеграция и доставка", "Автоматизация деплоя"]
tags: [vue, ci-cd, deployment, automation, gitlab, github-actions]
---

# CI/CD для Vue.js приложений

## Обзор

CI/CD (Continuous Integration/Continuous Deployment) — это практика автоматизации процесса сборки, тестирования и деплоя приложений. Для Vue.js приложений CI/CD обеспечивает стабильность, быструю доставку изменений и снижение рисков при релизах.

## Архитектура CI/CD пайплайна

### Основные этапы

1. **Сборка (Build)**: Компиляция исходного кода и создание артефактов
2. **Тестирование (Test)**: Запуск юнит-тестов, интеграционных тестов
3. **Анализ качества кода**: ESLint, Stylelint, SonarQube
4. **Сборка Docker-образа**: При использовании контейнеризации
5. **Деплой**: Развертывание на тестовую или продакшн среду
6. **Мониторинг**: Проверка состояния после деплоя

## Популярные CI/CD платформы

### GitHub Actions

GitHub Actions — встроенный CI/CD инструмент GitHub:

```yaml
name: Vue.js CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run unit tests
      run: npm run test:unit

    - name: Run build
      run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20.x'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Build production
      run: npm run build
      env:
        NODE_ENV: production

    - name: Deploy to production
      run: |
        # Скрипт деплоя
        echo "Deploying to production..."
        # rsync, scp или другой способ деплоя
```

### GitLab CI/CD

GitLab CI/CD с использованием `.gitlab-ci.yml`:

```yaml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  DOCKER_LATEST: $CI_REGISTRY_IMAGE:latest

# Кэширование зависимостей
cache:
  paths:
    - node_modules/

before_script:
  - npm ci

test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm run lint
    - npm run test:unit
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

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
    - docker build -t $DOCKER_IMAGE .
    - docker tag $DOCKER_IMAGE $DOCKER_LATEST
    - docker push $DOCKER_IMAGE
    - docker push $DOCKER_LATEST
  only:
    - main

deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache rsync openssh
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $STAGING_HOST >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - rsync -avz --delete dist/ $STAGING_USER@$STAGING_HOST:/var/www/vue-app/
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache docker-cli
    - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
    - docker pull $DOCKER_IMAGE
    - docker tag $DOCKER_IMAGE vue-app:latest
    - docker run -d --name vue-app -p 80:80 --rm vue-app:latest
  environment:
    name: production
  when: manual
  only:
    - main
```

### Jenkins

Pipeline для Jenkins (Jenkinsfile):

```groovy
pipeline {
    agent any
    
    tools {
        nodejs "NodeJS-20"
    }
    
    environment {
        DOCKER_IMAGE = "vue-app:${env.BUILD_NUMBER}"
        REGISTRY = "registry.example.com"
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
        
        stage('Test') {
            steps {
                sh 'npm run test:unit'
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.REGISTRY}/${DOCKER_IMAGE}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://${env.REGISTRY}", 'docker-registry-credentials') {
                        docker.image("${env.REGISTRY}/${DOCKER_IMAGE}").push()
                        docker.image("${env.REGISTRY}/${DOCKER_IMAGE}").push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Production') {
            steps {
                script {
                    // Деплой на продакшн
                    sh '''
                        docker login -u $DOCKER_USER -p $DOCKER_PASS $REGISTRY
                        docker pull $REGISTRY/$DOCKER_IMAGE
                        docker stop vue-app || true
                        docker rm vue-app || true
                        docker run -d --name vue-app -p 80:80 $REGISTRY/$DOCKER_IMAGE
                    '''
                }
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

## Российские особенности CI/CD

### Локальные решения

В 2025 году особенно актуальны локальные CI/CD решения:

- [[GitLab]] CE/EE (самохостинг)
- Jenkins (самохостинг)
- TeamCity
- Bamboo (при доступности)

### Российские облака

- [[Яндекс.Облако]] - Yandex Cloud Functions, Container Registry
- [[МегаФон.Облако]] - CI/CD сервисы
- [[Ростелеком.Облако]] - DevOps инструменты

### Ограничения и обходные пути

- Использование внутренних npm-репозиториев
- Зеркалирование Docker-образов
- Использование proxy-серверов для доступа к внешним ресурсам

## Практические примеры CI/CD

### Для статических Vue-приложений

Пример GitHub Actions для деплоя на GitHub Pages:

```yaml
name: Deploy Vue App to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '20.x'
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Build
      run: |
        npm run build
      env:
        BASE_URL: /${{ github.event.repository.name }}/
        
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist
```

### Для SSR-приложений (Nuxt.js)

Пример GitLab CI для деплоя Nuxt.js на VPS:

```yaml
stages:
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

cache:
  paths:
    - node_modules/

before_script:
  - npm ci

test:
  stage: test
  image: node:$NODE_VERSION
  script:
    - npm run test
    - npm run build
  artifacts:
    paths:
      - .output/
    expire_in: 1 hour

deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh rsync
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $STAGING_HOST >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - rsync -avz --delete .output/ $STAGING_USER@$STAGING_HOST:/var/www/nuxt-app/
    - ssh $STAGING_USER@$STAGING_HOST 'pm2 restart nuxt-app || pm2 start ecosystem.config.js'
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh rsync
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $PRODUCTION_HOST >> ~/.ssh/known_hosts
    - chmod 644 ~/.ssh/known_hosts
    - rsync -avz --delete .output/ $PRODUCTION_USER@$PRODUCTION_HOST:/var/www/nuxt-app/
    - ssh $PRODUCTION_USER@$PRODUCTION_HOST 'pm2 restart nuxt-app || pm2 start ecosystem.config.js'
  environment:
    name: production
  when: manual
  only:
    - main
```

## Мониторинг и алертинг CI/CD

### Уведомления

- Интеграция с Slack, Telegram, Microsoft Teams
- Email-уведомления
- Вебхуки для внешних систем

### Алертинг

- Уведомления о неудачных сборках
- Алерты о долгих сборках
- Уведомления о релизах

## Безопасность CI/CD

### Управление секретами

- Использование зашифрованных переменных
- Хранение секретов в Vault
- Использование внешних систем управления секретами

### Защита от атак

- Проверка изменений в CI/CD конфигурациях
- Ограничение прав доступа к репозиториям
- Использование signed commits

## Лучшие практики CI/CD

### Оптимизация времени выполнения

- Кэширование зависимостей
- Параллельное выполнение задач
- Использование легковесных образов

### Обработка ошибок

- Корректная обработка сбоев
- Автоматический повтор сборок
- Резервные стратегии деплоя

## Связанные темы

- [[Vue.js]] - основы фреймворка
- [[Статический-деплой]] - особенности статического деплоя
- [[SSR-деплой]] - особенности SSR-деплоя
- [[Docker]] - контейнеризация приложений
- [[Мониторинг]] - отслеживание состояния CI/CD

## Заключение

CI/CD для Vue.js приложений обеспечивает автоматизацию процесса разработки и доставки, снижает риски и ускоряет выпуск новых версий. При правильной настройке и учете российских реалий 2025 года, CI/CD становится ключевым элементом успешного процесса разработки.