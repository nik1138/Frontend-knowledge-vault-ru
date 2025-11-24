---
aliases: [K8s для TypeScript, Оркестрация TypeScript]
tags: [kubernetes, k8s, typescript, деплой, оркестрация, контейнеры]
---

# Kubernetes для TypeScript-приложений

## Обзор

Kubernetes (часто сокращаемый до K8s) - это открытая система оркестрации контейнеров, разработанная Google. Она автоматизирует развертывание, масштабирование и управление контейнеризованными приложениями. Для TypeScript-приложений Kubernetes особенно полезен, так как позволяет эффективно управлять жизненным циклом приложения в контейнерах Docker.

## Основы Kubernetes для TypeScript

### Архитектура Kubernetes

Kubernetes состоит из нескольких ключевых компонентов:

- **Master Node**: Управляет кластером
- **Worker Nodes**: Запускают контейнеры с приложениями
- **etcd**: Хранилище конфигурации кластера
- **API Server**: Интерфейс для взаимодействия с кластером
- **Controller Manager**: Управляет различными контроллерами
- **Scheduler**: Назначает контейнеры на узлы

### Ключевые концепции

- **Pod**: Наименьшая развертываемая единица в Kubernetes
- **Deployment**: Определяет желаемое состояние приложения
- **Service**: Обеспечивает сетевой доступ к Pod'ам
- **ConfigMap/Secret**: Хранение конфигурационных данных и секретов
- **Namespace**: Логическое разделение ресурсов

## Подготовка TypeScript-приложения для Kubernetes

### 1. Контейнеризация приложения

Перед развертыванием в Kubernetes TypeScript-приложение должно быть упаковано в Docker-образ. Убедитесь, что у вас есть готовый `Dockerfile` как описано в [[Docker]].

### 2. Создание Docker-образа

```bash
docker build -t my-ts-app:latest .
```

### 3. Загрузка образа в реестр

```bash
docker tag my-ts-app:latest registry.example.com/my-ts-app:latest
docker push registry.example.com/my-ts-app:latest
```

## Основные манифесты Kubernetes для TypeScript

### Deployment

Deployment управляет состоянием приложения и обеспечивает желаемое количество реплик:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: typescript-app
  labels:
    app: typescript-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: typescript-app
  template:
    metadata:
      labels:
        app: typescript-app
    spec:
      containers:
      - name: app
        image: registry.example.com/my-ts-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Service

Service обеспечивает доступ к Pod'ам через стабильный IP и DNS-имя:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: typescript-app-service
spec:
  selector:
    app: typescript-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP  # или LoadBalancer, NodePort в зависимости от требований
```

### ConfigMap

ConfigMap позволяет хранить конфигурационные данные:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: typescript-app-config
data:
  database_url: "mongodb://mongo:27017/myapp"
  api_timeout: "5000"
  log_level: "info"
```

### Secret

Secret используется для хранения чувствительных данных:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: typescript-app-secrets
type: Opaque
data:
  db_password: <base64-encoded-password>
  jwt_secret: <base64-encoded-secret>
```

## Практические примеры

### Пример полного развертывания

Создадим полный пример развертывания TypeScript-приложения:

**1. Namespace** (опционально):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: typescript-app
```

**2. ConfigMap**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: typescript-app
data:
  NODE_ENV: "production"
  API_URL: "https://api.example.com"
  LOG_LEVEL: "info"
```

**3. Secret**:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: typescript-app
type: Opaque
data:
  DATABASE_URL: <base64-encoded-db-url>
  JWT_SECRET: <base64-encoded-jwt-secret>
```

**4. Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: typescript-app
  namespace: typescript-app
  labels:
    app: typescript-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: typescript-app
  template:
    metadata:
      labels:
        app: typescript-app
    spec:
      containers:
      - name: server
        image: registry.example.com/my-ts-app:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: NODE_ENV
        - name: API_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: API_URL
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DATABASE_URL
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: JWT_SECRET
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

**5. Service**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: typescript-app-service
  namespace: typescript-app
spec:
  selector:
    app: typescript-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

**6. Ingress** (для доступа извне):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: typescript-app-ingress
  namespace: typescript-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: typescript-app-service
            port:
              number: 80
```

## Настройка TypeScript-приложения для Kubernetes

### Зависимости

Для TypeScript-приложений, развертываемых в Kubernetes, рекомендуется использовать следующие зависимости:

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "kubernetes-client": "^9.0.0",  // для взаимодействия с API Kubernetes
    "config": "^3.3.0",             // для управления конфигурацией
    "winston": "^3.8.0"             // для логирования
  }
}
```

### Код TypeScript-приложения с поддержкой Kubernetes

```typescript
import express, { Request, Response } from 'express';
import { createLogger, transports, format } from 'winston';

// Создаем логгер с форматом, подходящим для Kubernetes
const logger = createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.splat(),
    format.json()
  ),
  transports: [
    new transports.Console()
  ]
});

const app = express();
const PORT = parseInt(process.env.PORT || '3000', 10);

// Health check endpoint для liveness probe
app.get('/health', (req: Request, res: Response) => {
  // Проверяем критические зависимости (база данных, внешние API и т.д.)
  const healthcheck = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
  };
  
  try {
    res.status(200).send(healthcheck);
    logger.info('Health check passed');
  } catch (error) {
    res.status(503).send({ error: 'Health check failed', details: error });
    logger.error('Health check failed', { error });
  }
});

// Ready check endpoint для readiness probe
app.get('/ready', (req: Request, res: Response) => {
  // Проверяем готовность приложения к обработке запросов
  const readyCheck = {
    message: 'Ready',
    timestamp: Date.now(),
  };
  
  res.status(200).send(readyCheck);
  logger.info('Readiness check passed');
});

// Основной маршрут
app.get('/', (req: Request, res: Response) => {
  logger.info('Received request to root endpoint');
  res.json({ 
    message: 'Hello from TypeScript app in Kubernetes!',
    environment: process.env.NODE_ENV,
    podName: process.env.HOSTNAME
  });
});

// Обработка SIGTERM для корректного завершения
process.on('SIGTERM', () => {
  logger.info('Received SIGTERM, shutting down gracefully');
  server.close(() => {
    logger.info('Process terminated');
  });
});

const server = app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT} in ${process.env.NODE_ENV} mode`);
});
```

## Управление конфигурацией

### Использование ConfigMap и Secret в TypeScript

TypeScript-приложение может использовать библиотеку `config` для управления конфигурацией:

**config/default.ts**:
```typescript
export default {
  app: {
    port: parseInt(process.env.PORT || '3000', 10),
    env: process.env.NODE_ENV || 'development',
    name: process.env.APP_NAME || 'typescript-app'
  },
  database: {
    url: process.env.DATABASE_URL || 'mongodb://localhost:27017/myapp',
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true
    }
  },
  logging: {
    level: process.env.LOG_LEVEL || 'info'
  }
};
```

## Мониторинг и логирование

### Логирование в Kubernetes

Для эффективного логирования в Kubernetes рекомендуется:

1. Использовать структурированные логи в формате JSON
2. Включать контекстные данные (ID запроса, пользователь и т.д.)
3. Использовать соответствующие уровни логирования

### Мониторинг с Prometheus

Для мониторинга TypeScript-приложений в Kubernetes можно использовать экспортеры Prometheus:

```typescript
import client from 'prom-client';

// Создаем метрики
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'code']
});

// Middleware для сбора метрик
export const metricsMiddleware = (req: Request, res: Response, next: NextFunction) => {
  const end = httpRequestDuration.startTimer();
  
  res.on('finish', () => {
    end({
      method: req.method,
      route: req.route?.path || req.path,
      code: res.statusCode
    });
  });
  
  next();
};
```

## CI/CD с Kubernetes

### Helm Charts

Для упрощения развертывания можно использовать Helm - пакетный менеджер для Kubernetes:

**Chart.yaml**:
```yaml
apiVersion: v2
name: typescript-app
description: A Helm chart for TypeScript application
type: application
version: 0.1.0
appVersion: "1.0.0"
```

**values.yaml**:
```yaml
replicaCount: 3

image:
  repository: registry.example.com/my-ts-app
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  type: LoadBalancer
  port: 80

ingress:
  enabled: true
  hosts:
    - host: app.example.com
      paths: ["/"]

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 200m
    memory: 256Mi
```

## Лучшие практики

### 1. Использование Init Containers

Для ожидания готовности зависимостей (баз данных, кэшей и т.д.) используйте Init Containers:

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox:1.35
    command: ['sh', '-c', 'until nc -z database-service 5432; do echo waiting for database; sleep 2; done;']
```

### 2. Правильные Probes

Настройте liveness и readiness probes для корректного мониторинга состояния приложения:

- Liveness probe: проверяет, живо ли приложение
- Readiness probe: проверяет, готово ли приложение принимать трафик

### 3. Resource Limits

Всегда устанавливайте resource requests и limits для предотвращения проблем с ресурсами:

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "200m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 4. Security Context

Используйте security context для повышения безопасности:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
```

## Заключение

Kubernetes предоставляет мощную платформу для развертывания и управления TypeScript-приложениями. Правильная настройка позволяет достичь высокой доступности, масштабируемости и надежности.

Следуя лучшим практикам, описанным в этом руководстве, вы сможете эффективно использовать Kubernetes для своих TypeScript-проектов.

## См. также

- [[Docker]]
- [[CI-CD-с-Typescript]]
- [[Облачные-сервисы]]
- [[Автоматизация-деплоя]]
- [[Мониторинг-и-логирование]]