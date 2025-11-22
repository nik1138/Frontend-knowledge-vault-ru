# Безопасность в JavaScript

## Введение

Безопасность - критически важный аспект разработки веб-приложений на JavaScript. Современные веб-приложения подвержены множеству угроз, включая межсайтовый скриптинг (XSS), подделку межсайтовых запросов (CSRF), внедрение кода и другие атаки. Понимание принципов безопасности и применение лучших практик помогает создавать надежные и защищенные приложения.

## Основные угрозы безопасности

### 1. Межсайтовый скриптинг (XSS)

XSS - это уязвимость, при которой злоумышленник внедряет вредоносный код в веб-страницы, который затем выполняется в браузерах других пользователей.

#### Типы XSS:

1. **Reflected XSS** - вредоносный скрипт отражается от веб-сервера
2. **Stored XSS** - вредоносный скрипт сохраняется на сервере
3. **DOM-based XSS** - уязвимость в клиентском JavaScript коде

```javascript
// Плохой пример - уязвимость к XSS
function displayUserInput() {
  const userInput = document.getElementById('userInput').value;
  // Уязвимость! Пользовательский ввод вставляется напрямую в DOM
  document.getElementById('output').innerHTML = userInput;
}

// Хороший пример - защита от XSS
function displayUserInputSafely() {
  const userInput = document.getElementById('userInput').value;
  // Безопасная вставка текста
  document.getElementById('output').textContent = userInput;
}

// Еще лучше - использование шаблонизатора с автоматическим экранированием
function displayUserInputWithTemplate() {
  const userInput = document.getElementById('userInput').value;
  const template = document.createElement('div');
  template.textContent = userInput; // Автоматическое экранирование
  document.getElementById('output').appendChild(template);
}

// Функция для безопасного экранирования HTML
function escapeHtml(text) {
  const map = {
    '&': '&',
    '<': '<',
    '>': '>',
    '"': '"',
    "'": '&#039;'
  };
  
  return text.replace(/[&<>"']/g, function(m) {
    return map[m];
  });
}

// Использование экранирования
function displayUserInputEscaped() {
  const userInput = document.getElementById('userInput').value;
  document.getElementById('output').innerHTML = escapeHtml(userInput);
}
```

### 2. Подделка межсайтовых запросов (CSRF)

CSRF - это атака, при которой злоумышленник заставляет жертву выполнить нежелательные действия на веб-сайте, на котором она аутентифицирована.

```javascript
// Защита от CSRF на клиенте
class CSRFProtection {
  constructor() {
    this.token = this.getCSRFToken();
  }
  
  getCSRFToken() {
    // Получение токена из мета-тега или cookie
    return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content') ||
           this.getCookie('csrf-token');
  }
  
  getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) return parts.pop().split(';').shift();
  }
  
  // Добавление CSRF токена к запросам
  async fetchWithCSRF(url, options = {}) {
    const defaultOptions = {
      headers: {
        'X-CSRF-Token': this.token,
        'Content-Type': 'application/json'
      },
      credentials: 'same-origin'
    };
    
    const mergedOptions = {
      ...defaultOptions,
      ...options,
      headers: {
        ...defaultOptions.headers,
        ...options.headers
      }
    };
    
    return fetch(url, mergedOptions);
  }
}

// Использование
const csrf = new CSRFProtection();

// Безопасный POST запрос
csrf.fetchWithCSRF('/api/user/update', {
  method: 'POST',
  body: JSON.stringify({ name: 'John' })
});
```

### 3. Внедрение JavaScript (JavaScript Injection)

Внедрение JavaScript происходит, когда недоверенные данные используются в контексте выполнения JavaScript.

```javascript
// Плохой пример - уязвимость к внедрению
function dangerousEval(userInput) {
  // Никогда не используйте eval с пользовательским вводом!
  return eval(userInput);
}

function dangerousFunctionConstructor(userInput) {
  // Function constructor тоже опасен
  return new Function('return ' + userInput)();
}

// Хороший пример - безопасная обработка
function safeJsonParse(jsonString) {
  try {
    // JSON.parse безопасен для парсинга JSON
    return JSON.parse(jsonString);
  } catch (error) {
    console.error('Invalid JSON:', error);
    return null;
  }
}

// Валидация и очистка пользовательского ввода
function validateAndSanitizeInput(input) {
  // Проверка типа
  if (typeof input !== 'string') {
    throw new Error('Input must be a string');
  }
  
  // Ограничение длины
  if (input.length > 1000) {
    throw new Error('Input too long');
  }
  
  // Удаление потенциально опасных символов
  return input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
              .replace(/javascript:/gi, '')
              .replace(/vbscript:/gi, '')
              .trim();
}
```

## Лучшие практики безопасности

### 1. Защита DOM-манипуляций

```javascript
// Безопасная работа с DOM
class SafeDOMManipulation {
  // Безопасная вставка HTML контента
  static safeInsertHTML(element, htmlContent) {
    // Создание временного элемента для очистки
    const temp = document.createElement('div');
    temp.innerHTML = htmlContent;
    
    // Очистка потенциально опасного контента
    const scripts = temp.getElementsByTagName('script');
    while (scripts.length > 0) {
      scripts[0].parentNode.removeChild(scripts[0]);
    }
    
    element.innerHTML = temp.innerHTML;
  }
  
  // Безопасное создание элементов
  static createElementSafely(tag, attributes = {}, content = '') {
    // Проверка разрешенных тегов
    const allowedTags = ['div', 'span', 'p', 'strong', 'em', 'a', 'img'];
    if (!allowedTags.includes(tag.toLowerCase())) {
      throw new Error(`Tag ${tag} is not allowed`);
    }
    
    const element = document.createElement(tag);
    
    // Безопасная установка атрибутов
    for (const [key, value] of Object.entries(attributes)) {
      if (this.isSafeAttribute(key, value)) {
        element.setAttribute(key, value);
      }
    }
    
    // Безопасная установка контента
    if (content) {
      element.textContent = content;
    }
    
    return element;
  }
  
  // Проверка безопасных атрибутов
  static isSafeAttribute(name, value) {
    // Запрещенные атрибуты
    const forbiddenAttributes = ['onload', 'onerror', 'onclick', 'onmouseover'];
    if (forbiddenAttributes.includes(name.toLowerCase())) {
      return false;
    }
    
    // Проверка значений атрибутов
    if (typeof value === 'string') {
      const dangerousPatterns = [
        /javascript:/i,
        /vbscript:/i,
        /data:/i,
        /<script/i
      ];
      
      return !dangerousPatterns.some(pattern => pattern.test(value));
    }
    
    return true;
  }
}

// Использование
try {
  const safeElement = SafeDOMManipulation.createElementSafely('div', {
    'class': 'user-content',
    'data-id': '123'
  }, 'Безопасный контент');
  
  document.body.appendChild(safeElement);
} catch (error) {
  console.error('Ошибка создания элемента:', error);
}
```

### 2. Безопасная работа с cookies

```javascript
// Класс для безопасной работы с cookies
class SecureCookies {
  // Установка безопасного cookie
  static setSecureCookie(name, value, options = {}) {
    const defaults = {
      path: '/',
      secure: true,        // Только по HTTPS
      httpOnly: true,      // Недоступен для JavaScript
      sameSite: 'Strict',  // Защита от CSRF
      maxAge: 3600         // 1 час
    };
    
    const finalOptions = { ...defaults, ...options };
    
    let cookieString = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
    
    if (finalOptions.domain) {
      cookieString += `; Domain=${finalOptions.domain}`;
    }
    
    cookieString += `; Path=${finalOptions.path}`;
    
    if (finalOptions.expires) {
      cookieString += `; Expires=${finalOptions.expires.toUTCString()}`;
    }
    
    if (finalOptions.maxAge) {
      cookieString += `; Max-Age=${finalOptions.maxAge}`;
    }
    
    if (finalOptions.secure) {
      cookieString += '; Secure';
    }
    
    if (finalOptions.httpOnly) {
      cookieString += '; HttpOnly';
    }
    
    if (finalOptions.sameSite) {
      cookieString += `; SameSite=${finalOptions.sameSite}`;
    }
    
    document.cookie = cookieString;
  }
  
  // Получение cookie
  static getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${encodeURIComponent(name)}=`);
    if (parts.length === 2) {
      return decodeURIComponent(parts.pop().split(';').shift());
    }
    return null;
  }
  
  // Удаление cookie
  static removeCookie(name, path = '/') {
    document.cookie = `${encodeURIComponent(name)}=; Path=${path}; Expires=Thu, 01 Jan 1970 00:00:01 GMT;`;
  }
}

// Использование
SecureCookies.setSecureCookie('sessionId', 'abc123', {
  maxAge: 3600,
  secure: true,
  httpOnly: true,
  sameSite: 'Strict'
});
```

### 3. Защита от атак типа "грубой силы"

```javascript
// Класс для защиты от атак типа "грубой силы"
class BruteForceProtection {
  constructor(maxAttempts = 5, lockoutTime = 300000) { // 5 минут
    this.maxAttempts = maxAttempts;
    this.lockoutTime = lockoutTime;
    this.attempts = new Map();
    this.lockouts = new Map();
  }
  
  // Проверка, заблокирован ли IP
  isLockedOut(ip) {
    const lockout = this.lockouts.get(ip);
    if (!lockout) return false;
    
    if (Date.now() - lockout.timestamp > this.lockoutTime) {
      // Время блокировки истекло
      this.lockouts.delete(ip);
      this.attempts.delete(ip);
      return false;
    }
    
    return true;
  }
  
  // Регистрация неудачной попытки
  registerFailedAttempt(ip) {
    if (!this.attempts.has(ip)) {
      this.attempts.set(ip, 0);
    }
    
    const attempts = this.attempts.get(ip) + 1;
    this.attempts.set(ip, attempts);
    
    if (attempts >= this.maxAttempts) {
      this.lockouts.set(ip, {
        timestamp: Date.now(),
        attempts: attempts
      });
    }
  }
  
  // Сброс счетчика после успешной попытки
  resetAttempts(ip) {
    this.attempts.delete(ip);
    this.lockouts.delete(ip);
  }
  
  // Получение времени до разблокировки
  getLockoutTimeRemaining(ip) {
    const lockout = this.lockouts.get(ip);
    if (!lockout) return 0;
    
    const elapsed = Date.now() - lockout.timestamp;
    return Math.max(0, this.lockoutTime - elapsed);
  }
}

// Использование на клиенте
const bruteForceProtection = new BruteForceProtection();

async function login(username, password) {
  const ip = 'client'; // В реальном приложении IP определяется сервером
  
  if (bruteForceProtection.isLockedOut(ip)) {
    const timeRemaining = bruteForceProtection.getLockoutTimeRemaining(ip);
    throw new Error(`Слишком много неудачных попыток. Попробуйте через ${Math.ceil(timeRemaining / 60000)} минут`);
  }
  
  try {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ username, password })
    });
    
    if (response.ok) {
      bruteForceProtection.resetAttempts(ip);
      return await response.json();
    } else {
      bruteForceProtection.registerFailedAttempt(ip);
      throw new Error('Неверные учетные данные');
    }
  } catch (error) {
    bruteForceProtection.registerFailedAttempt(ip);
    throw error;
  }
}
```

### 4. Защита от кликджекинга

```javascript
// Защита от кликджекинга
class ClickjackingProtection {
  // Проверка, загружено ли приложение во фрейме
  static isInFrame() {
    try {
      return window.self !== window.top;
    } catch (e) {
      // Если доступ к window.top запрещен, значит мы во фрейме
      return true;
    }
  }
  
  // Защита от загрузки во фрейме
  static preventFraming() {
    if (this.isInFrame()) {
      // Попытка выйти из фрейма
      if (window.top) {
        window.top.location = window.location;
      }
    }
  }
  
  // Добавление заголовков безопасности
  static addSecurityHeaders() {
    // В реальном приложении эти заголовки устанавливаются сервером
    // Но можно добавить мета-теги для дополнительной защиты
    const meta = document.createElement('meta');
    meta.httpEquiv = 'X-Frame-Options';
    meta.content = 'DENY';
    document.head.appendChild(meta);
  }
}

// Инициализация защиты
ClickjackingProtection.preventFraming();
ClickjackingProtection.addSecurityHeaders();
```

## Безопасность API и AJAX запросов

### 1. Защита AJAX запросов

```javascript
// Класс для безопасных AJAX запросов
class SecureAjax {
  constructor() {
    this.csrfToken = this.getCSRFToken();
  }
  
  getCSRFToken() {
    return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content');
  }
  
  // Безопасный GET запрос
  async get(url, options = {}) {
    const defaultOptions = {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json'
      },
      credentials: 'same-origin'
    };
    
    return this.request(url, { ...defaultOptions, ...options });
  }
  
  // Безопасный POST запрос
  async post(url, data, options = {}) {
    const defaultOptions = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': this.csrfToken
      },
      body: JSON.stringify(data),
      credentials: 'same-origin'
    };
    
    return this.request(url, { ...defaultOptions, ...options });
  }
  
  // Универсальный метод запроса
  async request(url, options) {
    // Валидация URL
    if (!this.isValidUrl(url)) {
      throw new Error('Invalid URL');
    }
    
    try {
      const response = await fetch(url, options);
      
      // Проверка статуса ответа
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      // Проверка типа контента
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        return await response.json();
      } else {
        return await response.text();
      }
    } catch (error) {
      console.error('Request failed:', error);
      throw error;
    }
  }
  
  // Валидация URL
  isValidUrl(string) {
    try {
      const url = new URL(string, window.location.origin);
      return url.origin === window.location.origin;
    } catch (_) {
      return false;
    }
  }
  
  // Санитизация данных перед отправкой
  sanitizeData(data) {
    if (typeof data === 'string') {
      return this.sanitizeString(data);
    }
    
    if (typeof data === 'object' && data !== null) {
      const sanitized = {};
      for (const [key, value] of Object.entries(data)) {
        sanitized[key] = this.sanitizeData(value);
      }
      return sanitized;
    }
    
    return data;
  }
  
  // Санитизация строк
  sanitizeString(str) {
    return str.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
              .replace(/javascript:/gi, '')
              .replace(/vbscript:/gi, '');
  }
}

// Использование
const secureAjax = new SecureAjax();

// Безопасный GET запрос
secureAjax.get('/api/users')
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Безопасный POST запрос
secureAjax.post('/api/users', {
  name: 'John',
  email: 'john@example.com'
})
.then(data => console.log(data))
.catch(error => console.error(error));
```

### 2. Защита от атак на уровне данных

```javascript
// Класс для защиты данных
class DataProtection {
  // Шифрование данных (упрощенный пример)
  static encryptData(data, key) {
    // В реальных приложениях используйте библиотеки типа crypto-js
    // или Web Crypto API
    if (typeof data === 'string') {
      return btoa(data); // Базовое кодирование (не шифрование!)
    }
    return data;
  }
  
  // Дешифрование данных
  static decryptData(encryptedData, key) {
    try {
      return atob(encryptedData); // Базовое декодирование
    } catch (error) {
      console.error('Decryption failed:', error);
      return null;
    }
  }
  
  // Хеширование данных
  static async hashData(data) {
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(data);
    const hashBuffer = await crypto.subtle.digest('SHA-256', dataBuffer);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  }
  
  // Валидация email
  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  // Валидация пароля
  static validatePassword(password) {
    // Минимум 8 символов, хотя бы одна цифра, одна буква и один специальный символ
    const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/;
    return passwordRegex.test(password);
  }
  
  // Санитизация HTML
  static sanitizeHTML(html) {
    const temp = document.createElement('div');
    temp.textContent = html;
    return temp.innerHTML;
  }
  
  // Очистка объекта от опасных свойств
  static cleanObject(obj, dangerousKeys = ['__proto__', 'constructor', 'prototype']) {
    if (typeof obj !== 'object' || obj === null) {
      return obj;
    }
    
    const cleaned = Array.isArray(obj) ? [] : {};
    
    for (const [key, value] of Object.entries(obj)) {
      // Пропускаем опасные ключи
      if (dangerousKeys.includes(key)) {
        continue;
      }
      
      // Рекурсивная очистка вложенных объектов
      cleaned[key] = this.cleanObject(value, dangerousKeys);
    }
    
    return cleaned;
  }
}

// Использование
async function secureUserRegistration(userData) {
  // Валидация данных
  if (!DataProtection.validateEmail(userData.email)) {
    throw new Error('Invalid email format');
  }
  
  if (!DataProtection.validatePassword(userData.password)) {
    throw new Error('Password does not meet requirements');
  }
  
  // Санитизация данных
  const sanitizedData = DataProtection.cleanObject({
    name: DataProtection.sanitizeHTML(userData.name),
    email: userData.email.toLowerCase().trim(),
    password: userData.password // Пароль не санитизируется, а хешируется на сервере
  });
  
  // Хеширование пароля (в реальном приложении это делается на сервере)
  const passwordHash = await DataProtection.hashData(sanitizedData.password);
  
  // Отправка данных
  const secureAjax = new SecureAjax();
  return secureAjax.post('/api/register', {
    ...sanitizedData,
    password: passwordHash
  });
}
```

## Content Security Policy (CSP)

Content Security Policy - это дополнительный уровень безопасности, который помогает обнаруживать и смягчать определенные типы атак, включая XSS и внедрение данных.

```javascript
// Класс для работы с CSP
class ContentSecurityPolicy {
  // Генерация_nonce для inline скриптов
  static generateNonce() {
    return Math.random().toString(36).substr(2, 10);
  }
  
  // Добавление nonce к скриптам
  static addNonceToScripts() {
    const nonce = this.generateNonce();
    
    // Добавление nonce к существующим скриптам
    const scripts = document.querySelectorAll('script');
    scripts.forEach(script => {
      if (!script.hasAttribute('src')) {
        script.setAttribute('nonce', nonce);
      }
    });
    
    return nonce;
  }
  
  // Создание безопасного inline скрипта
  static createSecureScript(code, nonce) {
    const script = document.createElement('script');
    script.textContent = code;
    script.setAttribute('nonce', nonce);
    return script;
  }
  
  // Проверка источника скрипта
  static isAllowedSource(src, allowedSources) {
    try {
      const url = new URL(src, window.location.origin);
      return allowedSources.some(allowed => 
        url.origin === allowed || allowed === '*'
      );
    } catch (error) {
      return false;
    }
  }
}

// Использование CSP в приложении
class SecureApplication {
  constructor() {
    this.cspNonce = ContentSecurityPolicy.addNonceToScripts();
    this.allowedScriptSources = [
      window.location.origin,
      'https://apis.google.com'
    ];
  }
  
  // Безопасное добавление скрипта
  addScript(src) {
    if (ContentSecurityPolicy.isAllowedSource(src, this.allowedScriptSources)) {
      const script = document.createElement('script');
      script.src = src;
      script.crossOrigin = 'anonymous';
      document.head.appendChild(script);
    } else {
      console.warn('Script source not allowed:', src);
    }
  }
  
  // Безопасное выполнение inline кода
  executeInlineScript(code) {
    const script = ContentSecurityPolicy.createSecureScript(code, this.cspNonce);
    document.head.appendChild(script);
    // Удаление скрипта после выполнения (опционально)
    setTimeout(() => script.remove(), 0);
  }
}
```

## Лучшие практики и рекомендации

### 1. Общие принципы безопасности

```javascript
// Принцип наименьших привилегий
class SecureModule {
  constructor() {
    // Ограничиваем доступ к глобальным объектам
    this.allowedGlobals = ['console', 'document', 'window'];
  }
  
  // Безопасное выполнение кода
  executeSafely(code, context = {}) {
    // Создание изолированного контекста
    const safeContext = this.createSafeContext(context);
    
    // Использование Function вместо eval (менее опасно)
    const func = new Function(
      ...Object.keys(safeContext),
      `"use strict"; ${code}`
    );
    
    return func(...Object.values(safeContext));
  }
  
  createSafeContext(userContext) {
    const safeContext = {};
    
    // Копируем только разрешенные значения
    for (const [key, value] of Object.entries(userContext)) {
      if (this.isSafeValue(value)) {
        safeContext[key] = value;
      }
    }
    
    return safeContext;
  }
  
  isSafeValue(value) {
    // Разрешаем только примитивные типы и простые объекты
    const allowedTypes = ['string', 'number', 'boolean', 'undefined'];
    if (allowedTypes.includes(typeof value)) {
      return true;
    }
    
    if (value === null) {
      return true;
    }
    
    if (Array.isArray(value)) {
      return value.every(item => this.isSafeValue(item));
    }
    
    if (typeof value === 'object') {
      return Object.values(value).every(val => this.isSafeValue(val));
    }
    
    return false;
  }
}

// Защита от prototype pollution
class PrototypeProtection {
  static safeMerge(target, source) {
    // Защита от опасных ключей
    const dangerousKeys = ['__proto__', 'constructor', 'prototype'];
    
    for (const [key, value] of Object.entries(source)) {
      if (dangerousKeys.includes(key)) {
        continue;
      }
      
      if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
        if (!target[key]) {
          target[key] = {};
        }
        this.safeMerge(target[key], value);
      } else {
        target[key] = value;
      }
    }
    
    return target;
  }
  
  static safeAssign(target, ...sources) {
    for (const source of sources) {
      this.safeMerge(target, source);
    }
    return target;
  }
}
```

### 2. Мониторинг и логирование безопасности

```javascript
// Система мониторинга безопасности
class SecurityMonitor {
  constructor() {
    this.suspiciousActivities = [];
    this.maxLogSize = 100;
  }
  
  // Логирование подозрительной активности
  logSuspiciousActivity(activity, details = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      activity,
      details,
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    this.suspiciousActivities.push(logEntry);
    
    // Ограничение размера лога
    if (this.suspiciousActivities.length > this.maxLogSize) {
      this.suspiciousActivities.shift();
    }
    
    // Отправка на сервер (в реальном приложении)
    this.sendToSecurityServer(logEntry);
    
    console.warn('Suspicious activity detected:', logEntry);
  }
  
  // Отправка на сервер безопасности
  async sendToSecurityServer(logEntry) {
    try {
      // В реальном приложении отправляйте на специальный endpoint
      // await fetch('/api/security/log', {
      //   method: 'POST',
      //   headers: { 'Content-Type': 'application/json' },
      //   body: JSON.stringify(logEntry)
      // });
    } catch (error) {
      console.error('Failed to send security log:', error);
    }
  }
  
  // Проверка подозрительных паттернов
  checkForSuspiciousPatterns() {
    // Проверка на множественные ошибки
    window.addEventListener('error', (event) => {
      this.logSuspiciousActivity('javascript_error', {
        message: event.message,
        filename: event.filename,
        lineno: event.lineno,
        colno: event.colno
      });
    });
    
    // Проверка на неожиданные изменения DOM
    const observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        if (mutation.type === 'childList') {
          mutation.addedNodes.forEach((node) => {
            if (node.nodeType === Node.ELEMENT_NODE) {
              // Проверка на внедрение скриптов
              if (node.tagName === 'SCRIPT' || node.querySelectorAll('script').length > 0) {
                this.logSuspiciousActivity('script_injection', {
                  element: node.tagName,
                  content: node.outerHTML.substring(0, 100)
                });
              }
            }
          });
        }
      });
    });
    
    observer.observe(document.body, {
      childList: true,
      subtree: true
    });
  }
}

// Инициализация мониторинга
const securityMonitor = new SecurityMonitor();
securityMonitor.checkForSuspiciousPatterns();
```

## Заключение

Безопасность в JavaScript - это комплексная задача, требующая внимания к множеству аспектов:

1. **Предотвращение XSS** - экранирование вывода, валидация ввода
2. **Защита от CSRF** - использование токенов, проверка источника
3. **Безопасная работа с DOM** - предотвращение внедрения кода
4. **Защита данных** - шифрование, хеширование, валидация
5. **Content Security Policy** - ограничение источников контента
6. **Мониторинг и логирование** - выявление подозрительной активности

Ключевые принципы:

- **Не доверяй вводу** - всегда валидируй и санируй данные
- **Принцип наименьших привилегий** - давай только необходимые права
- **Защита в глубину** - используй несколько уровней защиты
- **Регулярное обновление** - следи за уязвимостями в зависимостях
- **Обучение** - постоянно изучай новые угрозы и методы защиты

Помните, что безопасность - это процесс, а не одноразовое действие. Регулярный аудит кода, обновление зависимостей и следование лучшим практикам помогут создать надежные и защищенные веб-приложения.

#javascript #security #xss #csrf #csp #web-security #frontend-security