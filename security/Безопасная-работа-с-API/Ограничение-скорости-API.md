---
tags: [security, api, rate-limiting, best-practices]
aliases: [Rate Limiting, API Rate Limiting, Ограничение запросов]
---

# Ограничение скорости API

## Введение в ограничение скорости API

Ограничение скорости API (Rate Limiting) — это важный механизм безопасности и управления нагрузкой, который ограничивает количество запросов, которые клиент может отправить к API за определенный период времени. Это предотвращает перегрузку сервера, защищает от атак и обеспечивает стабильную работу сервиса для всех пользователей.

## Зачем нужно ограничение скорости

Ограничение скорости необходимо по нескольким причинам:

- **Защита от перегрузки**: Предотвращает чрезмерное потребление ресурсов сервера
- **Предотвращение атак**: Защищает от DDoS, brute force и других атак
- **Управление квотами**: Позволяет управлять доступом к API для разных пользователей
- **Коммерческие цели**: Позволяет реализовать разные тарифные планы
- **Стабильность сервиса**: Обеспечивает равномерное распределение ресурсов

## Типы атак, предотвращаемых ограничением скорости

### DDoS-атаки
Distributed Denial of Service (DDoS) атаки направлены на перегрузку сервера запросами, что делает сервис недоступным для других пользователей. Ограничение скорости помогает смягчить последствия таких атак, ограничивая количество запросов от одного источника.

### Brute Force атаки
При атаках методом перебора злоумышленник пытается подобрать пароль или токен доступа, отправляя множество запросов. Ограничение скорости делает такие атаки неэффективными, замедляя процесс перебора.

### Scraping
Автоматизированные боты могут использовать API для массового сбора данных. Ограничение скорости предотвращает несанкционированный сбор данных и защищает контент от несанкционированного использования.

## Алгоритмы ограничения скорости

### Fixed Window
Алгоритм фиксированного окна (Fixed Window) устанавливает лимит запросов в пределах определенного временного интервала (например, 1000 запросов в час). Недостаток этого алгоритма в том, что он может привести к пикам нагрузки в начале каждого нового окна.

```javascript
// Пример реализации Fixed Window на Node.js
class FixedWindowRateLimiter {
  constructor(limit, windowMs) {
    this.limit = limit;
    this.windowMs = windowMs;
    this.requests = new Map();
  }

  checkLimit(identifier) {
    const now = Date.now();
    const record = this.requests.get(identifier) || { count: 0, start: now };
    
    if (now - record.start > this.windowMs) {
      // Сброс счетчика
      record.count = 1;
      record.start = now;
      this.requests.set(identifier, record);
      return true;
    }
    
    if (record.count >= this.limit) {
      return false; // Лимит превышен
    }
    
    record.count++;
    this.requests.set(identifier, record);
    return true;
  }
}
```

### Sliding Window
Алгоритм скользящего окна (Sliding Window) более точно отслеживает количество запросов, учитывая время каждого запроса. Это позволяет избежать проблемы пиков нагрузки, характерной для Fixed Window.

### Token Bucket
Алгоритм Token Bucket (ведро с токенами) работает следующим образом: в виртуальное "ведро" добавляются токены с постоянной скоростью. Каждый запрос "забирает" один токен. Если токенов нет, запрос отклоняется. Ведро имеет максимальный размер, что позволяет аккумулировать токены в периоды низкой активности.

```javascript
// Пример реализации Token Bucket на Node.js
class TokenBucketRateLimiter {
  constructor(capacity, refillRate) {
    this.capacity = capacity; // Максимальное количество токенов
    this.refillRate = refillRate; // Количество токенов, добавляемых в секунду
    this.buckets = new Map();
  }

  checkLimit(identifier) {
    const now = Date.now();
    let bucket = this.buckets.get(identifier);
    
    if (!bucket) {
      // Создание нового ведра
      bucket = {
        tokens: this.capacity,
        lastRefill: now
      };
      this.buckets.set(identifier, bucket);
    }
    
    // Вычисление количества токенов для добавления
    const timePassed = (now - bucket.lastRefill) / 1000; // в секундах
    const tokensToAdd = Math.floor(timePassed * this.refillRate);
    
    // Обновление количества токенов (не больше capacity)
    bucket.tokens = Math.min(this.capacity, bucket.tokens + tokensToAdd);
    bucket.lastRefill = now;
    
    if (bucket.tokens > 0) {
      bucket.tokens--;
      return true; // Запрос разрешен
    }
    
    return false; // Лимит превышен
  }
}
```

### Leaky Bucket
Алгоритм Leaky Bucket (протекающее ведро) представляет собой очередь с фиксированной скоростью обработки запросов. Запросы помещаются в очередь, и обрабатываются с постоянной скоростью. Если очередь переполнена, новые запросы отбрасываются.

## Реализация на сервере

На сервере ограничение скорости обычно реализуется с использованием промежуточного программного обеспечения (middleware) в веб-фреймворках. Ниже приведены примеры реализации для популярных фреймворков.

### Express.js
Для Node.js с Express можно использовать библиотеку `express-rate-limit`:

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100, // Ограничение на 100 запросов за windowMs
  message: 'Слишком много запросов с этого IP, попробуйте позже',
  standardHeaders: true, // Возврат информации о лимитах в заголовках
  legacyHeaders: false, // Отключение заголовков 'X-RateLimit-*'
});

// Применение ограничения ко всем маршрутам
app.use(limiter);
```

### Django
В Django можно использовать `django-ratelimit`:

```python
from django_ratelimit.decorators import ratelimit
from django.http import HttpResponse

@ratelimit(key='ip', rate='5/m', method='GET', block=True)
def my_view(request):
    return HttpResponse('Hello World')
```

### Spring Boot
В Spring Boot можно реализовать ограничение скорости с помощью фильтров:

```java
@Component
public class RateLimitFilter implements Filter {
    
    private final Map<String, RateLimitBucket> buckets = new ConcurrentHashMap<>();
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                         FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String clientIP = getClientIP(httpRequest);
        
        RateLimitBucket bucket = buckets.computeIfAbsent(clientIP, 
            ip -> new RateLimitBucket(100, 1.0)); // 100 запросов, 1 в секунду
        
        if (bucket.tryConsume()) {
            chain.doFilter(request, response);
        } else {
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            httpResponse.getWriter().write("Rate limit exceeded");
        }
    }
    
    private String getClientIP(HttpServletRequest request) {
        // Логика определения IP клиента
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (xForwardedFor != null && !xForwardedFor.isEmpty()) {
            return xForwardedFor.split(",")[0];
        }
        return request.getRemoteAddr();
    }
}
```

## Реализация на клиенте

На клиенте ограничение скорости может быть реализовано для управления отправкой запросов к API и предотвращения превышения лимитов.

### JavaScript
```javascript
class RateLimitedClient {
  constructor(maxRequests, timeWindowMs) {
    this.maxRequests = maxRequests;
    this.timeWindowMs = timeWindowMs;
    this.requests = [];
  }

  async makeRequest(url, options = {}) {
    const now = Date.now();
    
    // Удаление старых запросов из окна
    this.requests = this.requests.filter(timestamp => 
      now - timestamp < this.timeWindowMs
    );
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const timeToWait = this.timeWindowMs - (now - oldestRequest);
      
      await new Promise(resolve => setTimeout(resolve, timeToWait));
      
      // Повторная проверка после ожидания
      return this.makeRequest(url, options);
    }
    
    // Добавление текущего запроса
    this.requests.push(now);
    
    try {
      const response = await fetch(url, options);
      return response;
    } catch (error) {
      console.error('Ошибка запроса:', error);
      throw error;
    }
  }
}

// Использование
const client = new RateLimitedClient(10, 60000); // 10 запросов в минуту

async function fetchUserData(userId) {
  const response = await client.makeRequest(`/api/users/${userId}`);
  return response.json();
}
```

## Настройка лимитов

При настройке лимитов важно учитывать следующие факторы:

- **Тип API**: Публичный или частный
- **Тип запросов**: Аутентифицированные или анонимные
- **Тарифные планы**: Разные лимиты для разных категорий пользователей
- **Типы операций**: GET, POST, PUT, DELETE могут иметь разные лимиты
- **Уровень агрегации**: IP, пользователь, API-ключ

Пример настройки разных лимитов:

```javascript
const rateLimiters = {
  anonymous: rateLimit({
    windowMs: 15 * 60 * 1000, // 15 минут
    max: 100, // 100 запросов за 15 минут
  }),
  authenticated: rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 1000, // 1000 запросов за 15 минут
  }),
  premium: rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 10000, // 10000 запросов за 15 минут
  })
};

// Применение в зависимости от типа пользователя
app.use('/api/', (req, res, next) => {
  if (req.user) {
    if (req.user.isPremium) {
      rateLimiters.premium(req, res, next);
    } else {
      rateLimiters.authenticated(req, res, next);
    }
  } else {
    rateLimiters.anonymous(req, res, next);
  }
});
```

## Обработка превышения лимитов

Когда клиент превышает лимит запросов, сервер должен корректно обработать ситуацию:

- Возвращать соответствующий HTTP-статус
- Указывать время, когда можно повторить запрос
- Предоставлять понятное сообщение об ошибке
- Вести логи превышений для анализа

```javascript
const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 100,
  handler: (req, res, next) => {
    res.status(429).json({
      error: 'Превышен лимит запросов',
      retryAfter: '15 минут',
      message: 'Пожалуйста, попробуйте снова позже'
    });
  }
});
```

## Возврат HTTP-статусов

При превышении лимита запросов сервер должен возвращать HTTP-статус 429 (Too Many Requests). Это стандартный статус, указывающий на то, что клиент отправил слишком много запросов за заданный период.

Дополнительно можно использовать:

- 503 (Service Unavailable) - если сервис временно недоступен из-за ограничений
- 403 (Forbidden) - в некоторых случаях, особенно при подозрительной активности

## Заголовки Rate Limit

HTTP-заголовки позволяют информировать клиента о текущих ограничениях:

- `X-RateLimit-Limit` - максимальное количество запросов в окне
- `X-RateLimit-Remaining` - оставшееся количество запросов
- `X-RateLimit-Reset` - время сброса лимита
- `Retry-After` - время ожидания до следующего запроса

Современные API также могут использовать стандартизированные заголовки:

- `RateLimit-Limit` - максимальное количество запросов
- `RateLimit-Remaining` - оставшееся количество запросов
- `RateLimit-Reset` - время сброса
- `RateLimit-Policy` - политика ограничения

```javascript
// Пример установки заголовков в Express.js
app.use((req, res, next) => {
  // Установка заголовков до обработки запроса
  res.set({
    'X-RateLimit-Limit': 1000,
    'X-RateLimit-Remaining': 999,
    'X-RateLimit-Reset': new Date(Date.now() + 15 * 60 * 1000).toUTCString()
  });
  
  next();
});
```

## Практические примеры

### Пример 1: Ограничение скорости для аутентифицированных пользователей

```javascript
// Ограничение для аутентифицированных пользователей
const authRateLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 час
  max: (req, res) => {
    // Разные лимиты для разных типов пользователей
    if (req.user && req.user.isPremium) {
      return 5000; // Премиум пользователи: 5000 запросов в час
    }
    return 1000; // Обычные пользователи: 1000 запросов в час
  },
  skip: (req, res) => {
    // Пропуск ограничения для администраторов
    return req.user && req.user.isAdmin;
  },
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/authenticated/', authRateLimiter);
```

### Пример 2: Ограничение для конкретных маршрутов

```javascript
// Разные лимиты для разных типов запросов
const getRateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 минут
  max: 1000, // 1000 GET запросов
  message: 'Слишком много GET запросов'
});

const postRateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100, // 100 POST запросов
  message: 'Слишком много POST запросов'
});

app.get('/api/data', getRateLimiter, (req, res) => {
  res.json({ data: 'some data' });
});

app.post('/api/data', postRateLimiter, (req, res) => {
  // Обработка POST запроса
  res.json({ message: 'Data received' });
});
```

### Пример 3: Использование Redis для распределенного ограничения

```javascript
const redis = require('redis');
const client = redis.createClient();

// Асинхронные методы Redis
const getAsync = promisify(client.get).bind(client);
const setAsync = promisify(client.set).bind(client);

class RedisRateLimiter {
  constructor(windowMs, max) {
    this.windowMs = windowMs;
    this.max = max;
  }

  async checkLimit(identifier) {
    const key = `rate_limit:${identifier}`;
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // Использование Redis для хранения временных меток запросов
    const requests = await getAsync(key);
    let requestTimes = [];
    
    if (requests) {
      requestTimes = JSON.parse(requests)
        .filter(time => time > windowStart);
    }
    
    if (requestTimes.length >= this.max) {
      return false; // Лимит превышен
    }
    
    requestTimes.push(now);
    await setAsync(key, JSON.stringify(requestTimes), 'EX', Math.ceil(this.windowMs / 1000));
    
    return true;
  }
}
```

## Инструменты для ограничения скорости

### NGINX
NGINX предоставляет встроенные возможности для ограничения скорости с помощью модулей `ngx_http_limit_req_module` и `ngx_http_limit_conn_module`:

```nginx
# Ограничение запросов
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=mylimit burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

### Apache
Apache использует модуль `mod_evasive` или `mod_security` для ограничения скорости:

```apache
# Пример с mod_evasive
<IfModule mod_evasive24.c>
    DOSHashTableSize    2048
    DOSPageCount        2
    DOSSiteCount        50
    DOSPageInterval     1
    DOSSiteInterval     1
    DOSBlockingPeriod   600
</IfModule>
```

### Cloudflare
Cloudflare предоставляет встроенные функции Rate Limiting через правила Firewall:

```javascript
// Пример с Cloudflare Workers
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  const ip = request.headers.get('CF-Connecting-IP');
  
  // Проверка лимита через Cloudflare KV
  const key = `rate_limit:${ip}`;
  const value = await RATES.get(key);
  
  if (value && parseInt(value) > 100) { // 100 запросов в час
    return new Response('Rate limit exceeded', { status: 429 });
  }
  
  // Увеличение счетчика
  await RATES.put(key, value ? parseInt(value) + 1 : 1, { expirationTtl: 3600 });
  
  return fetch(request);
}
```

### AWS API Gateway
AWS API Gateway предоставляет встроенные механизмы Rate Limiting:

```yaml
# Пример CloudFormation шаблона
Resources:
  MyApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: MyApi
      Tags:
        - Key: Name
          Value: MyApi
  
  MyUsagePlan:
    Type: AWS::ApiGateway::UsagePlan
    Properties:
      ApiStages:
        - ApiId: !Ref MyApi
          Stage: !Ref MyApiStage
      Quota:
        Limit: 10000
        Period: DAY
      Throttle:
        BurstLimit: 100
        RateLimit: 50
```

## Мониторинг и логирование

Мониторинг и логирование важны для понимания эффективности системы ограничения скорости:

- Отслеживание частоты превышения лимитов
- Анализ паттернов использования API
- Определение аномальной активности
- Оптимизация лимитов на основе данных

### Логирование в Node.js

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'rate-limit.log' })
  ]
});

const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  handler: (req, res, next) => {
    logger.warn({
      message: 'Rate limit exceeded',
      ip: req.ip,
      url: req.originalUrl,
      userAgent: req.get('User-Agent'),
      timestamp: new Date().toISOString()
    });
    
    res.status(429).json({
      error: 'Превышен лимит запросов'
    });
  },
  skipSuccessfulRequests: true
});
```

### Метрики с Prometheus

```javascript
const promClient = require('prom-client');

// Создание метрики для отслеживания превышений лимита
const rateLimitExceeded = new promClient.Counter({
  name: 'rate_limit_exceeded_total',
  help: 'Total number of rate limit exceeded events',
  labelNames: ['endpoint', 'method', 'client_type']
});

const rateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  handler: (req, res, next) => {
    rateLimitExceeded.inc({
      endpoint: req.originalUrl,
      method: req.method,
      client_type: req.user ? 'authenticated' : 'anonymous'
    });
    
    res.status(429).json({ error: 'Rate limit exceeded' });
  }
});
```

## Ссылки на связанные файлы

- [[Аутентификация-API]] - информация об аутентификации запросов к API
- [[Защита-от-CSRF]] - защита от подделки межсайтовых запросов
- [[Шифрование-данных]] - методы шифрования передаваемых данных
- [[Сессии-и-куки]] - управление сессиями и куки в веб-приложениях
- [[OAuth-2.0]] - протокол авторизации OAuth 2.0
- [[JWT-токены]] - использование JWT для аутентификации
- [[HTTPS-и-SSL]] - настройка безопасного соединения
- [[Логирование-и-аудит]] - системы логирования и аудита безопасности
- [[Сканирование-уязвимостей]] - инструменты и методы сканирования
- [[Обработка-ошибок]] - безопасная обработка ошибок в API

## Заключение

Ограничение скорости API является критически важным компонентом безопасности и стабильности любого веб-сервиса. Правильная реализация позволяет защитить сервер от перегрузки, предотвратить различные виды атак и обеспечить справедливое распределение ресурсов между пользователями.

При проектировании системы ограничения скорости важно учитывать специфику приложения, типы пользователей и требования к производительности. Регулярный мониторинг и анализ данных о лимитах позволяет оптимизировать настройки и улучшить общую производительность API.

## Теги

#security #api #rate-limiting #best-practices #web-security #backend #nodejs #express #django #spring-boot