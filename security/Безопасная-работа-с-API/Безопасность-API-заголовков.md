---
aliases: [Заголовки безопасности API, API Security Headers]
tags: [security, api, headers, web-security]
---

# Безопасность API-заголовков

## Введение в безопасность заголовков API

Безопасность заголовков API - это критический аспект веб-безопасности, который часто недооценивается разработчиками. HTTP-заголовки являются важной частью взаимодействия между клиентом и сервером, и неправильная обработка этих заголовков может привести к серьезным уязвимостям. В этой статье мы рассмотрим, как правильно обрабатывать и защищать заголовки в API-приложениях.

## Зачем важна безопасность заголовков

HTTP-заголовки содержат важную информацию о запросе, включая данные аутентификации, информацию о происхождении запроса и параметры безопасности. Неправильная обработка заголовков может привести к следующим проблемам:

- Утечка конфиденциальной информации
- Атаки типа "подделка межсайтового запроса" (CSRF)
- Обход механизмов аутентификации
- Атаки на инъекции через заголовки
- Уязвимости, связанные с CORS

## Распространенные заголовки безопасности

### Заголовки защиты от XSS

- `X-XSS-Protection`: Включает встроенные механизмы защиты от XSS в браузерах
- `Content-Security-Policy`: Определяет политику безопасности контента
- `X-Content-Type-Options`: Предотвращает MIME-type sniffing

### Заголовки защиты от клик-джеркинга

- `X-Frame-Options`: Контролирует, можно ли встраивать страницу в iframe
- `Content-Security-Policy` с директивой `frame-ancestors`: Современная альтернатива X-Frame-Options

### Заголовки безопасности транспорта

- `Strict-Transport-Security`: Обязывает браузер использовать HTTPS
- `Referrer-Policy`: Контролирует, какие реферальные данные отправляются
- `Expect-CT`: Обеспечивает мониторинг и отчетность о сертификатах

## API-специфичные заголовки

API-приложения часто используют специфичные заголовки для различных целей:

- `X-API-Key`: Идентификатор API-ключа для аутентификации
- `X-Request-ID`: Уникальный идентификатор запроса для трассировки
- `X-Client-Version`: Версия клиента для совместимости
- `X-Forwarded-For`: Информация о реальном IP-адресе клиента
- `X-Real-IP`: Альтернативный заголовок для реального IP-адреса

Важно правильно валидировать и обрабатывать такие заголовки, так как они могут быть подделаны.

## Проверка и валидация заголовков

### Серверная валидация

```javascript
function validateHeaders(req, res, next) {
  const requiredHeaders = ['Content-Type', 'Authorization'];
  const missingHeaders = requiredHeaders.filter(header => !req.headers[header.toLowerCase()]);
  
  if (missingHeaders.length > 0) {
    return res.status(400).json({
      error: `Missing required headers: ${missingHeaders.join(', ')}`
    });
  }
  
  // Проверка формата заголовков
  if (req.headers['content-type'] && 
      !req.headers['content-type'].startsWith('application/json')) {
    return res.status(400).json({
      error: 'Content-Type must be application/json'
    });
  }
  
  next();
}
```

### Валидация значений заголовков

Каждый заголовок должен проходить проверку на:

- Формат (например, формат даты в заголовке `If-Modified-Since`)
- Допустимые значения (например, `Accept` заголовок)
- Длину (чтобы избежать атак типа buffer overflow)
- Наличие вредоносных символов или паттернов

## Защита от подделки заголовков

### Использование доверенных заголовков

При работе за reverse proxy (например, nginx) доверяйте только тем заголовкам, которые вы устанавливаете:

```javascript
// НЕ доверяйте заголовкам X-Forwarded-For напрямую
app.use((req, res, next) => {
  // Вместо этого используйте req.ip, который учитывает настройки trust proxy
  const clientIP = req.ip;
  next();
});
```

### Проверка источника заголовков

Для критических заголовков (например, `Authorization`) убедитесь, что они не могут быть подделаны на уровне сети:

- Используйте HTTPS для всех API-запросов
- Не передавайте чувствительные заголовки через ненадежные соединения
- Валидируйте происхождение запросов с помощью CORS

## CORS-заголовки безопасности

### Правильная настройка CORS

```javascript
const cors = require('cors');

// Плохо: разрешение всех источников
app.use(cors());

// Хорошо: конкретные источники
app.use(cors({
  origin: ['https://trusted-domain.com', 'https://another-trusted.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key']
}));
```

### Важные CORS-заголовки

- `Access-Control-Allow-Origin`: Определяет, какие источники могут получить доступ к ресурсу
- `Access-Control-Allow-Credentials`: Указывает, можно ли передавать учетные данные
- `Access-Control-Allow-Headers`: Перечисляет разрешенные заголовки
- `Access-Control-Allow-Methods`: Указывает разрешенные HTTP-методы

## Заголовки авторизации

### Типы заголовков авторизации

#### Basic Authentication

```
Authorization: Basic <base64-encoded-credentials>
```

#### Bearer Token

```
Authorization: Bearer <jwt-token>
```

#### API Key

```
Authorization: ApiKey <api-key>
X-API-Key: <api-key>
```

### Проверка токенов авторизации

```javascript
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1]; // Bearer <token>

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: 'Invalid or expired token' });
    }
    req.user = user;
    next();
  });
}
```

## Примеры уязвимостей через заголовки

### 1. Уязвимость через заголовок Host

```javascript
// Плохо: использование заголовка Host напрямую
app.get('/redirect', (req, res) => {
  const redirectUrl = `https://${req.headers.host}/dashboard`;
  res.redirect(redirectUrl); // Уязвимо к атакам
});

// Хорошо: использование белого списка доменов
const ALLOWED_REDIRECTS = ['example.com', 'sub.example.com'];
app.get('/redirect', (req, res) => {
  const host = req.headers.host;
  if (ALLOWED_REDIRECTS.includes(host)) {
    const redirectUrl = `https://${host}/dashboard`;
    res.redirect(redirectUrl);
  } else {
    res.redirect('/dashboard'); // Использовать безопасный путь по умолчанию
  }
});
```

### 2. Подделка заголовка X-Forwarded-For

```javascript
// Плохо: доверие заголовку X-Forwarded-For
const clientIP = req.headers['x-forwarded-for'];

// Хорошо: проверка через доверенные прокси
app.set('trust proxy', 1);
const clientIP = req.ip; // Использует встроенную систему доверия прокси
```

## Лучшие практики

### 1. Минимизация передаваемых заголовков

- Передавайте только необходимые заголовки
- Не используйте кастомные заголовки без необходимости
- Избегайте передачи чувствительной информации в заголовках

### 2. Правильная обработка заголовков

- Всегда проверяйте наличие обязательных заголовков
- Валидируйте формат и значения заголовков
- Используйте белые списки для разрешенных значений
- Логируйте подозрительные заголовки

### 3. Защита от атак

- Устанавливайте строгие политики безопасности
- Используйте HTTPS для всех API-запросов
- Валидируйте заголовки на сервере, независимо от клиента
- Регулярно обновляйте политики безопасности

### 4. Мониторинг и аудит

- Логируйте все заголовки для аудита безопасности
- Мониторьте аномальные паттерны в заголовках
- Регулярно анализируйте логи на наличие потенциальных атак

## Проверка заголовков на сервере

### Пример middleware для проверки заголовков

```javascript
function secureHeaders(req, res, next) {
  // Проверка и очистка заголовков
  const forbiddenHeaders = [
    'x-forwarded-for', 'x-real-ip', 'x-client-ip', 
    'x-originating-ip', 'x-remote-ip', 'x-remote-addr'
  ];

  for (const header of forbiddenHeaders) {
    if (req.headers[header]) {
      // Если заголовок присутствует, проверяем, доверяем ли мы ему
      if (!req.connection.encrypted && !isTrustedProxy(req.connection.remoteAddress)) {
        return res.status(400).json({ error: 'Unexpected proxy header' });
      }
    }
  }

  next();
}

function isTrustedProxy(ip) {
  // Проверка, является ли IP доверенным прокси
  const trustedProxies = ['127.0.0.1', '10.0.0.0/8'];
  return trustedProxies.some(trusted => isIPInRange(ip, trusted));
}
```

### Использование библиотек безопасности

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
  hsts: {
    maxAge: 31536000, // 1 год
    includeSubDomains: true,
    preload: true
  }
}));

// Ограничение частоты запросов
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100 // ограничение на 100 запросов за 15 минут
});
app.use(limiter);
```

## Проверка заголовков на клиенте

### Важные замечания

Клиентская проверка заголовков имеет ограниченную ценность, так как:

- Клиент может быть скомпрометирован
- Заголовки могут быть подделаны на клиенте
- Клиентская проверка может быть обойдена

### Когда использовать клиентскую проверку

- Для улучшения UX и предотвращения ошибок
- Для предварительной валидации перед отправкой запроса
- Для отладки и разработки

```javascript
// Пример клиентской проверки (только для UX, не для безопасности)
function validateRequestHeaders(headers) {
  const errors = [];
  
  // Проверка обязательных заголовков
  if (!headers['Content-Type']) {
    errors.push('Content-Type header is required');
  }
  
  // Проверка формата заголовков
  if (headers['Authorization'] && !headers['Authorization'].startsWith('Bearer ')) {
    errors.push('Authorization header must use Bearer scheme');
  }
  
  return errors;
}
```

## Тестирование безопасности заголовков

### Автоматизированное тестирование

```javascript
// Пример теста для проверки безопасности заголовков
const request = require('supertest');
const app = require('../app');

describe('Header Security', () => {
  it('should reject requests without required headers', async () => {
    const response = await request(app)
      .post('/api/data')
      .send({ data: 'test' })
      .expect(400);
    
    expect(response.body.error).toContain('Missing required headers');
  });

  it('should reject requests with malicious headers', async () => {
    const response = await request(app)
      .post('/api/data')
      .set('X-Forwarded-For', 'malicious-ip')
      .send({ data: 'test' })
      .expect(400);
    
    expect(response.body.error).toContain('Unexpected proxy header');
  });

  it('should accept requests with valid headers', async () => {
    const response = await request(app)
      .post('/api/data')
      .set('Content-Type', 'application/json')
      .set('Authorization', 'Bearer valid-token')
      .send({ data: 'test' })
      .expect(200);
    
    expect(response.body).toBeDefined();
  });
});
```

### Ручное тестирование

- Проверка обработки отсутствующих заголовков
- Проверка обработки некорректных значений заголовков
- Проверка обработки заголовков с вредоносным содержимым
- Проверка обработки дублированных заголовков
- Проверка обработки заголовков с нестандартным форматированием

### Инструменты для тестирования

- OWASP ZAP для автоматического сканирования
- Burp Suite для ручного тестирования
- curl для тестирования вручную
- Postman/Newman для автоматизированных тестов

## Ссылки на связанные материалы

- [[Безопасность-HTTP-заголовков]] - Общая информация о безопасности HTTP-заголовков
- [[CORS-безопасность]] - Подробное руководство по безопасности CORS
- [[Аутентификация-и-авторизация]] - Информация об аутентификации и авторизации
- [[XSS-защита]] - Методы защиты от межсайтового скриптинга
- [[CSRF-защита]] - Защита от подделки межсайтовых запросов
- [[HTTPS-и-SSL]] - Информация о безопасном соединении
- [[Rate-Limiting]] - Ограничение частоты запросов
- [[JWT-безопасность]] - Безопасность JSON Web Tokens
- [[API-безопасность-в-целом]] - Общие принципы безопасности API
- [[Сканирование-уязвимостей]] - Инструменты и методы сканирования

## Заключение

Безопасность заголовков API - это критически важный аспект веб-безопасности, требующий внимательного подхода и постоянного мониторинга. Правильная обработка заголовков помогает защитить приложение от множества типов атак и обеспечивает целостность данных и аутентификацию пользователей.

Ключевые моменты, которые нужно помнить:

1. Всегда валидируйте заголовки на сервере
2. Не доверяйте заголовкам, установленным клиентом
3. Используйте строгие политики безопасности
4. Регулярно тестируйте безопасность заголовков
5. Следите за обновлениями в области безопасности

Соблюдение этих принципов поможет создать более безопасные и надежные API-приложения.
