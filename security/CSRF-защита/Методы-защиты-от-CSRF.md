---
aliases: ["Защита от CSRF", "CSRF-токены", "CSRF-защита"]
tags: [security, csrf, web-security, authentication]
---

# Методы защиты от CSRF

## Введение

CSRF (Cross-Site Request Forgery) — это тип атаки, при которой злоумышленник заставляет аутентифицированного пользователя выполнить нежелательные действия в веб-приложении, в котором он аутентифицирован. Защита от CSRF критически важна для безопасности веб-приложений, особенно для операций, изменяющих состояние.

## Механизм CSRF-атаки

CSRF-атака работает следующим образом:
1. Пользователь аутентифицирован в приложении
2. Пользователь посещает вредоносный сайт
3. Вредоносный сайт отправляет запрос к защищенному приложению
4. Браузер автоматически отправляет аутентификационные куки
5. Запрос выполняется от имени пользователя

## Основные методы защиты

### 1. CSRF-токены (Synchronizer Token Pattern)

Самый распространенный и эффективный метод защиты от CSRF.

#### Реализация на сервере

```javascript
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

// Middleware для защиты
app.use(csrfProtection);

// Отправка токена клиенту
app.get('/form', (req, res) => {
  res.render('form', { csrfToken: req.csrfToken() });
});

// Защита маршрутов
app.post('/transfer', csrfProtection, (req, res) => {
  // Обработка запроса
  transferMoney(req.body.from, req.body.to, req.body.amount);
  res.send('Transfer completed');
});
```

#### Реализация на клиенте

```html
<!-- Форма с CSRF-токеном -->
<form method="POST" action="/transfer">
  <input type="hidden" name="_csrf" value="{{csrfToken}}">
  <input type="text" name="amount" placeholder="Amount">
  <button type="submit">Transfer</button>
</form>
```

```javascript
// AJAX-запросы с CSRF-токеном
$.ajaxSetup({
  beforeSend: function(xhr, settings) {
    if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
      xhr.setRequestHeader("X-CSRF-Token", $('meta[name="csrf-token"]').attr('content'));
    }
  }
});
```

### 2. Double Submit Cookie

Метод, при котором токен отправляется как в куки, так и в теле/заголовке запроса.

```javascript
// Установка CSRF-куки
function setCsrfCookie(req, res, next) {
  if (!req.cookies.csrfToken) {
    const csrfToken = generateSecureToken();
    res.cookie('csrfToken', csrfToken, { 
      httpOnly: false, // Доступен из JavaScript
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });
  }
  next();
}

// Проверка токена
function validateCsrf(req, res, next) {
  const cookieToken = req.cookies.csrfToken;
  const headerToken = req.headers['x-csrf-token'] || req.body._csrf;
  
  if (cookieToken !== headerToken) {
    return res.status(403).send('CSRF token mismatch');
  }
  
  next();
}

app.use(setCsrfCookie);
app.post('/api/data', validateCsrf, (req, res) => {
  // Обработка запроса
});
```

### 3. SameSite-атрибут куки

Один из самых простых методов защиты, но с ограничениями совместимости.

```javascript
// Установка сессионных куки с SameSite
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'lax' // или 'strict'
  }
}));
```

Типы SameSite:
- `strict` — куки отправляются только в рамках одного сайта
- `lax` — куки отправляются при навигации GET-запросами
- `none` — куки отправляются всегда (требует Secure)

### 4. Проверка Referer-заголовка

Менее надежный метод, так как заголовок может быть подделан или отсутствовать.

```javascript
function validateReferer(req, res, next) {
  const referer = req.headers.referer;
  const origin = req.headers.origin || req.headers.host;
  
  const trustedOrigins = [
    'https://yourdomain.com',
    'https://www.yourdomain.com'
  ];
  
  if (referer) {
    const refererOrigin = new URL(referer).origin;
    if (!trustedOrigins.includes(refererOrigin)) {
      return res.status(403).send('Invalid referer');
    }
  }
  
  next();
}
```

### 5. Custom Request Headers

Для AJAX-запросов можно использовать кастомные заголовки, которые браузер не отправляет автоматически.

```javascript
// Клиент
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest',
    'X-Custom-Header': 'value'
  },
  body: JSON.stringify({ amount: 100 })
});

// Сервер
function requireCustomHeader(req, res, next) {
  if (!req.headers['x-custom-header']) {
    return res.status(403).send('Missing custom header');
  }
  next();
}
```

## Современные методы защиты

### 1. Double Cookie Submit с подписью

Более безопасная версия double submit cookie с криптографической подписью.

```javascript
const crypto = require('crypto');

function generateSignedToken(secret) {
  const token = crypto.randomBytes(32).toString('hex');
  const signature = crypto
    .createHmac('sha256', secret)
    .update(token)
    .digest('hex');
  return `${token}.${signature}`;
}

function verifySignedToken(token, secret) {
  const parts = token.split('.');
  if (parts.length !== 2) return false;
  
  const [tokenPart, signaturePart] = parts;
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(tokenPart)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signaturePart, 'hex'),
    Buffer.from(expectedSignature, 'hex')
  );
}
```

### 2. Encrypted Token Pattern

Токены шифруются с использованием серверного ключа.

```javascript
const crypto = require('crypto');

class EncryptedCsrfToken {
  constructor(secret) {
    this.secret = secret;
  }
  
  generateToken(userId, timestamp = Date.now()) {
    const data = JSON.stringify({ userId, timestamp });
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher('aes-256-cbc', this.secret);
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    return `${iv.toString('hex')}.${encrypted}`;
  }
  
  validateToken(token, userId) {
    try {
      const [ivHex, encrypted] = token.split('.');
      const iv = Buffer.from(ivHex, 'hex');
      const decipher = crypto.createDecipher('aes-256-cbc', this.secret);
      let decrypted = decipher.update(encrypted, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      const data = JSON.parse(decrypted);
      // Проверка пользователя и времени жизни токена
      return data.userId === userId && (Date.now() - data.timestamp) < 3600000; // 1 час
    } catch (error) {
      return false;
    }
  }
}
```

## Реализация комплексной защиты

### Middleware для комплексной защиты

```javascript
class CsrfProtection {
  constructor(options = {}) {
    this.tokenGenerator = options.tokenGenerator || this.defaultTokenGenerator;
    this.storage = options.storage || new InMemoryStorage();
    this.timeLimit = options.timeLimit || 3600000; // 1 час
  }
  
  defaultTokenGenerator() {
    return crypto.randomBytes(32).toString('hex');
  }
  
  // Генерация токена для сессии
  generateToken(req) {
    const sessionId = req.sessionID || this.getSessionId(req);
    let token = this.storage.get(sessionId);
    
    if (!token) {
      token = this.tokenGenerator();
      this.storage.set(sessionId, token, this.timeLimit);
    }
    
    return token;
  }
  
  // Проверка токена
  validateToken(req) {
    const sessionId = req.sessionID || this.getSessionId(req);
    const storedToken = this.storage.get(sessionId);
    const receivedToken = req.body._csrf || 
                         req.headers['x-csrf-token'] || 
                         req.headers['x-xsrf-token'];
    
    if (!storedToken || !receivedToken) {
      return false;
    }
    
    // Защита от timing attacks
    return crypto.timingSafeEqual(
      Buffer.from(storedToken),
      Buffer.from(receivedToken)
    );
  }
  
  // Middleware
  middleware() {
    return (req, res, next) => {
      // Для GET-запросов - генерация токена
      if (req.method === 'GET') {
        res.locals.csrfToken = this.generateToken(req);
      } 
      // Для других методов - проверка токена
      else if (['POST', 'PUT', 'DELETE', 'PATCH'].includes(req.method)) {
        if (!this.validateToken(req)) {
          return res.status(403).json({ error: 'CSRF token mismatch' });
        }
        // Обновление токена после успешной проверки
        this.generateToken(req);
      }
      
      next();
    };
  }
}

// Использование
const csrfProtection = new CsrfProtection();
app.use(csrfProtection.middleware());
```

## Защита конкретных типов запросов

### Защита форм

```javascript
// Middleware для автоматического добавления CSRF-токена к формам
function addCsrfToken(req, res, next) {
  res.locals.csrfToken = req.csrfToken();
  next();
}

app.use(addCsrfToken);

// В шаблоне
app.get('/transfer-form', (req, res) => {
  res.render('transfer-form', { amount: 100 });
});
```

### Защита API

```javascript
// Защита API-маршрутов
const apiRoutes = express.Router();

apiRoutes.use((req, res, next) => {
  // Проверка заголовка для API-запросов
  if (req.headers['content-type'] === 'application/json') {
    if (req.headers['x-csrf-token'] !== req.session.csrfToken) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
  } else {
    // Проверка токена в теле для форм
    if (req.body._csrf !== req.session.csrfToken) {
      return res.status(403).send('Invalid CSRF token');
    }
  }
  next();
});

app.use('/api', apiRoutes);
```

## Лучшие практики

> [!tip] Лучшие практики CSRF-защиты
> 1. Используйте надежные CSRF-токены для операций, изменяющих состояние
> 2. Применяйте SameSite-атрибут куки как дополнительную защиту
> 3. Не полагайтесь только на проверку Referer
> 4. Обновляйте токены после успешной проверки
> 5. Используйте криптографически стойкие генераторы токенов

### Обработка ошибок

```javascript
// Централизованная обработка CSRF-ошибок
app.use((error, req, res, next) => {
  if (error.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({
      error: 'Session has expired or form has been tampered with',
      code: 'CSRF_INVALID'
    });
  }
  next(error);
});
```

## Связь с другими аспектами безопасности

Методы защиты от CSRF тесно связаны с:
- [[Управление-сессиями]] — правильное управление сессиями критично для CSRF-защиты
- [[HTTP-Security-Headers]] — заголовки безопасности дополняют CSRF-защиту
- [[Распространенные-уязвимости]] — CSRF входит в OWASP Top 10
- [[Управление-доступом]] — контроль доступа дополняет защиту
- [[Тестирование-безопасности]] — методы проверки CSRF-защиты

## Заключение

Защита от CSRF требует комплексного подхода с использованием нескольких методов. CSRF-токены остаются наиболее надежным методом защиты, но должны дополняться другими мерами, такими как SameSite-атрибуты куки. Регулярное тестирование и обновление методов защиты помогут обеспечить безопасность веб-приложения от CSRF-атак.

## Дополнительные ресурсы

- OWASP CSRF Prevention Cheat Sheet
- RFC 7871 (SameSite Attribute)
- CSRF Protection Libraries Documentation