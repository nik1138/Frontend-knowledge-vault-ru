---
aliases: [CSRF, Подделка межсайтовых запросов, Защита от CSRF]
tags: [security, csrf, frontend-security, authentication]
---

# CSRF-защита

## Введение

Подделка межсайтовых запросов (Cross-Site Request Forgery, CSRF) — это тип атаки, при котором злоумышленник заставляет браузер пользователя выполнить нежелательное действие на сайте, где пользователь аутентифицирован. В отличие от XSS, при CSRF-атаке злоумышленник не пытается украсть данные напрямую, а заставляет пользователя выполнить нежелательное действие, например, изменить настройки аккаунта или совершить транзакцию.

## Как работает CSRF-атака

1. Пользователь аутентифицирован на сайте (например, банк)
2. Пользователь посещает вредоносный сайт
3. Вредоносный сайт отправляет запрос на сайт банка от имени пользователя
4. Браузер автоматически отправляет cookies аутентификации
5. Сервер банка обрабатывает запрос как легитимный

### Пример CSRF-атаки

```html
<!-- Вредоносная страница -->
<html>
  <body>
    <h1>Загрузка...</h1>
    <img src="https://bank.com/transfer?to=attacker&amount=1000" 
         style="display:none;" />
  </body>
</html>
```

Когда пользователь посещает эту страницу, браузер отправляет запрос на перевод средств, и если пользователь аутентифицирован в банке, транзакция может быть успешной.

## Методы защиты от CSRF

### 1. CSRF-токены

Самый распространенный метод защиты - использование CSRF-токенов. Сервер генерирует уникальный токен для каждой сессии или формы и проверяет его при обработке запросов.

#### Реализация на бэкенде с использованием Express.js

```javascript
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

// Middleware для защиты
app.use(csrfProtection);

// Отправка токена клиенту
app.get('/form', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Проверка токена при POST-запросе
app.post('/submit', csrfProtection, (req, res) => {
  // Обработка запроса
});
```

#### Реализация на фронтенде

```javascript
// Получение CSRF-токена
async function getCSRFToken() {
  const response = await fetch('/api/csrf-token');
  const data = await response.json();
  return data.csrfToken;
}

// Использование токена в запросах
async function submitForm(formData) {
  const token = await getCSRFToken();
  
  const response = await fetch('/api/submit', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-CSRF-Token': token
    },
    body: JSON.stringify(formData)
  });
  
  return response.json();
}
```

### 2. Double Submit Cookie

Метод, при котором CSRF-токен отправляется как в заголовке запроса, так и в cookie. Сервер проверяет, совпадают ли токены.

```javascript
// Установка CSRF-токена в cookie на сервере
app.get('/page', (req, res) => {
  const csrfToken = generateCSRFToken();
  res.cookie('csrf-token', csrfToken, { httpOnly: false }); // Не httpOnly, чтобы JS мог получить токен
  res.render('page', { csrfToken });
});

// Проверка токена на сервере
app.post('/action', (req, res) => {
  const cookieToken = req.cookies['csrf-token'];
  const headerToken = req.headers['x-csrf-token'];
  
  if (cookieToken !== headerToken) {
    return res.status(403).send('CSRF token mismatch');
  }
  
  // Обработка запроса
});
```

### 3. SameSite-атрибут cookies

Атрибут SameSite для cookies позволяет предотвратить отправку cookies в межсайтовых запросах.

```javascript
// Установка cookie с SameSite
res.cookie('session', sessionId, { 
  httpOnly: true, 
  secure: true,
  sameSite: 'strict' // или 'lax'
});
```

Типы SameSite:
- `strict` - cookie отправляется только при запросах на тот же сайт
- `lax` - cookie отправляется при GET-запросах с других сайтов (например, по ссылке)
- `none` - cookie отправляется всегда (требует Secure-флага)

### 4. Проверка Referer-заголовка

Сервер может проверять, откуда пришел запрос, используя заголовок Referer. Однако этот метод не всегда надежен, так как:

- Заголовок может быть отключен пользователем
- Заголовок может быть подделан в некоторых браузерах
- Не работает при переходе с HTTPS на HTTP

```javascript
app.use((req, res, next) => {
  const referer = req.get('Referer');
  const allowedOrigin = 'https://yourdomain.com';
  
  if (referer && !referer.startsWith(allowedOrigin)) {
    return res.status(403).send('Invalid referer');
  }
  
  next();
});
```

## Практические рекомендации для фронтенд-разработчиков

### 1. Использование CSRF-токенов в формах

```html
<!-- Вставка CSRF-токена в форму -->
<form id="transferForm">
  <input type="hidden" name="csrf_token" value="{{csrf_token}}" />
  <input type="text" name="amount" />
  <input type="text" name="to" />
  <button type="submit">Перевести</button>
</form>
```

### 2. Использование токенов в AJAX-запросах

```javascript
// Получение токена из meta-тега
const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');

// Использование токена в fetch-запросах
fetch('/api/transfer', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': csrfToken
  },
  body: JSON.stringify({
    amount: 100,
    to: 'recipient'
  })
});
```

### 3. Автоматическая вставка токенов

```javascript
// Автоматическая установка CSRF-токена для всех запросов
function getCSRFToken() {
  return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content');
}

// Настройка axios с автоматическим добавлением токена
axios.defaults.headers.common['X-CSRF-Token'] = getCSRFToken();

// Или настройка fetch через middleware
const originalFetch = fetch;
window.fetch = function(url, options = {}) {
  const token = getCSRFToken();
  if (token && (options.method === 'POST' || options.method === 'PUT' || options.method === 'DELETE')) {
    options.headers = {
      ...options.headers,
      'X-CSRF-Token': token
    };
  }
  return originalFetch(url, options);
};
```

### 4. Защита SPA-приложений

Для одностраничных приложений важно правильно управлять CSRF-токенами:

```javascript
class CSRFManager {
  constructor() {
    this.token = null;
    this.refreshThreshold = 5 * 60 * 1000; // 5 минут
    this.lastRefresh = 0;
  }

  async getToken() {
    const now = Date.now();
    if (!this.token || (now - this.lastRefresh) > this.refreshThreshold) {
      await this.refreshToken();
    }
    return this.token;
  }

  async refreshToken() {
    try {
      const response = await fetch('/api/csrf-token', {
        method: 'GET',
        credentials: 'include' // Включаем cookies в запрос
      });
      const data = await response.json();
      this.token = data.token;
      this.lastRefresh = Date.now();
    } catch (error) {
      console.error('Не удалось обновить CSRF-токен:', error);
      throw error;
    }
  }

  async makeSecureRequest(url, options = {}) {
    const token = await this.getToken();
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'X-CSRF-Token': token
      }
    });
  }
}

// Использование
const csrfManager = new CSRFManager();
await csrfManager.makeSecureRequest('/api/transfer', {
  method: 'POST',
  body: JSON.stringify({ amount: 100 })
});
```

## Современные подходы к защите

### 1. SameSite-атрибут (предпочтительный метод)

```javascript
// Наиболее безопасная настройка
res.cookie('session', sessionId, { 
  httpOnly: true, 
  secure: true,
  sameSite: 'strict'
});
```

### 2. Samesite с fallback для старых браузеров

```javascript
// Установка как SameSite, так и CSRF-токена для совместимости
app.use((req, res, next) => {
  // Установка SameSite-атрибута
  res.cookie('session', sessionId, { 
    httpOnly: true, 
    secure: true,
    sameSite: 'lax'
  });
  
  // Также генерируем CSRF-токен для дополнительной защиты
  req.csrfToken = generateCSRFToken();
  next();
});
```

### 3. Double Submit Cookie с подписью

```javascript
const crypto = require('crypto');

function generateSignedToken(secret, data) {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(data);
  return data + '.' + hmac.digest('hex');
}

function verifySignedToken(secret, token) {
  const parts = token.split('.');
  if (parts.length !== 2) return false;
  
  const [data, signature] = parts;
  const expectedSignature = crypto.createHmac('sha256', secret)
    .update(data)
    .digest('hex');
    
  return crypto.timingSafeEqual(
    Buffer.from(signature), 
    Buffer.from(expectedSignature)
  );
}
```

## Тестирование CSRF-защиты

### 1. Проверка отсутствия токена

Попробуйте отправить POST-запрос без CSRF-токена - сервер должен отклонить его.

### 2. Проверка неправильного токена

Отправьте запрос с неправильным/устаревшим токеном.

### 3. Проверка SameSite-атрибута

Используйте инструменты разработчика для проверки установки правильного атрибута cookie.

## Заключение

CSRF-атаки остаются серьезной угрозой для веб-приложений, особенно для действий, изменяющих состояние. Использование комбинации методов защиты (CSRF-токены, SameSite-атрибуты, проверка заголовков) позволяет эффективно защищать приложения от этих атак. Важно помнить, что защита от CSRF должна быть реализована как на фронтенде, так и на бэкенде.

## См. также
- [[Основы-веб-безопасности]]
- [[XSS-защита]]
- [[Безопасность-аутентификации]]
- [[HTTP-заголовки-безопасности]]