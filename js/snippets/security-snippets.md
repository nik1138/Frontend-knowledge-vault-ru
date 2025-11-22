---
aliases: ["Сниппеты безопасности", "JS Security Snippets"]
tags: [javascript, security, frontend, snippets]
---

# Сниппеты для обеспечения безопасности

Набор готовых сниппетов для обеспечения безопасности в JavaScript приложениях. Эти сниппеты помогут защитить ваш код от распространенных уязвимостей.

## Защита от XSS (Cross-Site Scripting)

### 1. Экранирование HTML

```javascript
/**
 * Экранирует HTML символы для предотвращения XSS
 * @param {string} str - строка для экранирования
 * @returns {string} - заэкранированная строка
 */
function escapeHtml(str) {
  if (typeof str !== 'string') return '';
  
  const escapeMap = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  };
  
  return str.replace(/[&<>"'\/]/g, (match) => escapeMap[match]);
}

// Пример использования
const userInput = '<script>alert("XSS")</script>';
const safeOutput = escapeHtml(userInput);
console.log(safeOutput); // &lt;script&gt;alert("XSS")&lt;/script&gt;

// Безопасное встраивание в DOM
function safeInsertHtml(element, htmlString) {
  element.textContent = htmlString; // Используем textContent вместо innerHTML
}

// Или с разрешением HTML, но безопасной очисткой
function sanitizeAndInsertHtml(element, htmlString) {
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = escapeHtml(htmlString);
  element.innerHTML = tempDiv.innerHTML;
}
```

### 2. Безопасная вставка HTML с разрешенными тегами

```javascript
/**
 * Санитизация HTML с разрешенными тегами и атрибутами
 * @param {string} html - HTML строка для санитизации
 * @param {object} options - опции санитизации
 * @returns {string} - очищенная HTML строка
 */
function sanitizeHtml(html, options = {}) {
  // Разрешенные теги и атрибуты
  const allowedTags = options.allowedTags || ['p', 'br', 'strong', 'em', 'u', 'ol', 'ul', 'li', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6'];
  const allowedAttributes = options.allowedAttributes || ['href', 'title', 'alt'];
  
  // Создаем временный DOM элемент
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = html;
  
  function sanitizeNode(node) {
    if (node.nodeType === Node.TEXT_NODE) {
      // Текстовые узлы не нуждаются в санитизации
      return node.cloneNode();
    }
    
    if (node.nodeType === Node.ELEMENT_NODE) {
      const tagName = node.tagName.toLowerCase();
      
      // Проверяем, разрешен ли тег
      if (!allowedTags.includes(tagName)) {
        // Если тег не разрешен, возвращаем его дочерние элементы
        const fragment = document.createDocumentFragment();
        for (const child of node.childNodes) {
          fragment.appendChild(sanitizeNode(child));
        }
        return fragment;
      }
      
      // Создаем новый элемент с разрешенными атрибутами
      const newNode = document.createElement(tagName);
      
      for (const attr of node.attributes) {
        if (allowedAttributes.includes(attr.name)) {
          // Дополнительная проверка для href и src атрибутов
          if ((attr.name === 'href' || attr.name === 'src') && 
              !isValidUrl(attr.value)) {
            continue; // Пропускаем потенциально опасные URL
          }
          newNode.setAttribute(attr.name, attr.value);
        }
      }
      
      // Рекурсивно санитизируем дочерние узлы
      for (const child of node.childNodes) {
        newNode.appendChild(sanitizeNode(child));
      }
      
      return newNode;
    }
    
    return document.createDocumentFragment();
  }
  
  const sanitizedFragment = sanitizeNode(tempDiv);
  const resultDiv = document.createElement('div');
  resultDiv.appendChild(sanitizedFragment);
  
  return resultDiv.innerHTML;
}

/**
 * Проверяет, является ли URL безопасным
 * @param {string} url - URL для проверки
 * @returns {boolean} - true если URL безопасен
 */
function isValidUrl(url) {
  try {
    const parsedUrl = new URL(url, window.location.origin);
    // Разрешаем только http, https и относительные URL
    return ['http:', 'https:', ''].includes(parsedUrl.protocol) || 
           url.startsWith('/') || url.startsWith('#') || url.startsWith('?');
  } catch (e) {
    return false;
  }
}

// Пример использования
const unsafeHtml = '<p>Безопасный текст</p><script>alert("XSS")</script><a href="javascript:alert(1)">Ссылка</a>';
const safeHtml = sanitizeHtml(unsafeHtml);
console.log(safeHtml); // <p>Безопасный текст</p><a>Ссылка</a>
```

## Защита от CSRF (Cross-Site Request Forgery)

### 1. Генерация и проверка CSRF токенов

```javascript
/**
 * Генерирует CSRF токен
 * @returns {string} - сгенерированный токен
 */
function generateCsrfToken() {
  // В реальном приложении токен должен генерироваться на сервере
  // и храниться в сессии пользователя
  return Array.from(crypto.getRandomValues(new Uint8Array(32)))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}

/**
 * Проверяет CSRF токен
 * @param {string} token - токен для проверки
 * @param {string} expectedToken - ожидаемый токен
 * @returns {boolean} - true если токен действителен
 */
function validateCsrfToken(token, expectedToken) {
  // Используем времянную константу для предотвращения timing атак
  return token && expectedToken && 
         token.length === expectedToken.length && 
         token === expectedToken;
}

/**
 * Добавляет CSRF токен к форме
 * @param {HTMLFormElement} form - форма для добавления токена
 * @param {string} token - CSRF токен
 */
function addCsrfTokenToForm(form, token) {
  // Проверяем, существует ли уже поле с токеном
  let tokenInput = form.querySelector('input[name="csrf_token"]');
  
  if (!tokenInput) {
    tokenInput = document.createElement('input');
    tokenInput.type = 'hidden';
    tokenInput.name = 'csrf_token';
    form.appendChild(tokenInput);
  }
  
  tokenInput.value = token;
}

/**
 * Добавляет CSRF токен к заголовкам fetch запроса
 * @param {object} headers - объект заголовков
 * @param {string} token - CSRF токен
 * @returns {object} - обновленный объект заголовков
 */
function addCsrfTokenToHeaders(headers = {}, token) {
  return {
    ...headers,
    'X-CSRF-Token': token,
    'X-Requested-With': 'XMLHttpRequest'
  };
}

// Пример использования
const csrfToken = generateCsrfToken();

// Добавление токена к форме
const form = document.getElementById('myForm');
addCsrfTokenToForm(form, csrfToken);

// Использование токена в fetch запросе
fetch('/api/data', {
  method: 'POST',
  headers: addCsrfTokenToHeaders({
    'Content-Type': 'application/json'
  }, csrfToken),
  body: JSON.stringify({ data: 'example' })
});
```

## Безопасная работа с данными

### 1. Валидация и очистка пользовательского ввода

```javascript
/**
 * Валидатор пользовательского ввода
 */
class InputValidator {
  constructor() {
    this.rules = new Map();
  }
  
  /**
   * Добавляет правило валидации
   * @param {string} field - имя поля
   * @param {object} rule - правило валидации
   */
  addRule(field, rule) {
    this.rules.set(field, rule);
    return this;
  }
  
  /**
   * Валидирует и очищает данные
   * @param {object} data - данные для валидации
   * @returns {object} - объект с результатами { isValid, sanitizedData, errors }
   */
  validate(data) {
    const sanitizedData = {};
    const errors = {};
    let isValid = true;
    
    for (const [field, rule] of this.rules) {
      const value = data[field];
      
      // Применяем санитизацию
      let sanitizedValue = value;
      
      if (rule.sanitize) {
        sanitizedValue = this.sanitizeValue(sanitizedValue, rule.sanitize);
      }
      
      // Применяем валидацию
      if (rule.validate) {
        const validation = this.validateValue(sanitizedValue, rule.validate);
        if (!validation.isValid) {
          errors[field] = validation.errors;
          isValid = false;
          continue; // Не сохраняем некорректное значение
        }
      }
      
      sanitizedData[field] = sanitizedValue;
    }
    
    return { isValid, sanitizedData, errors };
  }
  
  /**
   * Санитизирует значение по заданным правилам
   * @param {*} value - значение для санитизации
   * @param {object} sanitizeRules - правила санитизации
   * @returns {*} - санитизированное значение
   */
  sanitizeValue(value, sanitizeRules) {
    if (value === null || value === undefined) return value;
    
    let result = value;
    
    // Преобразование в строку для текстовых операций
    if (typeof result === 'string' || typeof result === 'number') {
      result = String(result);
      
      // Удаление тегов HTML
      if (sanitizeRules.stripHtml) {
        result = result.replace(/<[^>]*>/g, '');
      }
      
      // Экранирование HTML
      if (sanitizeRules.escapeHtml) {
        result = escapeHtml(result);
      }
      
      // Ограничение длины
      if (sanitizeRules.maxLength) {
        result = result.substring(0, sanitizeRules.maxLength);
      }
      
      // Очистка от лишних пробелов
      if (sanitizeRules.trim) {
        result = result.trim();
      }
      
      // Удаление специальных символов
      if (sanitizeRules.stripSpecialChars) {
        result = result.replace(/[^\w\sа-яА-ЯёЁ]/g, '');
      }
    }
    
    return result;
  }
  
  /**
   * Валидирует значение по заданным правилам
   * @param {*} value - значение для валидации
   * @param {object} validateRules - правила валидации
   * @returns {object} - результат валидации { isValid, errors }
   */
  validateValue(value, validateRules) {
    const errors = [];
    
    // Проверка на обязательность
    if (validateRules.required && 
        (value === null || value === undefined || value === '')) {
      errors.push('Поле обязательно для заполнения');
    }
    
    // Пропускаем другие проверки, если значение пустое и не обязательное
    if ((value === null || value === undefined || value === '') && 
        !validateRules.required) {
      return { isValid: true, errors: [] };
    }
    
    // Проверка типа
    if (validateRules.type) {
      if (validateRules.type === 'email' && !this.isValidEmail(value)) {
        errors.push('Некорректный формат email');
      } else if (validateRules.type === 'url' && !this.isValidUrl(value)) {
        errors.push('Некорректный формат URL');
      } else if (validateRules.type === 'number' && isNaN(value)) {
        errors.push('Значение должно быть числом');
      }
    }
    
    // Проверка длины строки
    if (typeof value === 'string') {
      if (validateRules.minLength && value.length < validateRules.minLength) {
        errors.push(`Минимальная длина: ${validateRules.minLength} символов`);
      }
      if (validateRules.maxLength && value.length > validateRules.maxLength) {
        errors.push(`Максимальная длина: ${validateRules.maxLength} символов`);
      }
    }
    
    // Проверка числового диапазона
    if (typeof value === 'number' || !isNaN(value)) {
      const numValue = Number(value);
      if (validateRules.min !== undefined && numValue < validateRules.min) {
        errors.push(`Минимальное значение: ${validateRules.min}`);
      }
      if (validateRules.max !== undefined && numValue > validateRules.max) {
        errors.push(`Максимальное значение: ${validateRules.max}`);
      }
    }
    
    // Проверка по регулярному выражению
    if (validateRules.pattern && typeof value === 'string') {
      const regex = new RegExp(validateRules.pattern);
      if (!regex.test(value)) {
        errors.push(validateRules.message || 'Некорректный формат значения');
      }
    }
    
    return { isValid: errors.length === 0, errors };
  }
  
  /**
   * Проверяет формат email
   * @param {string} email - email для проверки
   * @returns {boolean} - true если формат корректный
   */
  isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  /**
   * Проверяет формат URL
   * @param {string} url - URL для проверки
   * @returns {boolean} - true если формат корректный
   */
  isValidUrl(url) {
    try {
      new URL(url);
      return true;
    } catch (e) {
      return false;
    }
  }
}

// Пример использования
const validator = new InputValidator();

validator
  .addRule('username', {
    validate: { 
      required: true, 
      minLength: 3, 
      maxLength: 20,
      pattern: '^[a-zA-Z0-9_]+$',
      message: 'Имя пользователя может содержать только латинские буквы, цифры и подчеркивание'
    },
    sanitize: { 
      trim: true, 
      stripSpecialChars: true 
    }
  })
  .addRule('email', {
    validate: { required: true, type: 'email' },
    sanitize: { trim: true }
  })
  .addRule('bio', {
    validate: { maxLength: 500 },
    sanitize: { stripHtml: true, trim: true }
  });

const userData = {
  username: '  user_123<script>  ',
  email: 'user@example.com',
  bio: '<p>О себе <script>alert("XSS")</script></p>'
};

const result = validator.validate(userData);
console.log(result);
// {
//   isValid: true,
//   sanitizedData: {
//     username: 'user_123',
//     email: 'user@example.com',
//     bio: 'О себе '
//   },
//   errors: {}
// }
```

### 2. Безопасная работа с localStorage и sessionStorage

```javascript
/**
 * Безопасная обертка для работы с localStorage/sessionStorage
 */
class SecureStorage {
  constructor(storageType = 'local', encryptionKey = null) {
    this.storage = storageType === 'session' ? sessionStorage : localStorage;
    this.encryptionKey = encryptionKey;
  }
  
  /**
   * Безопасное сохранение данных
   * @param {string} key - ключ
   * @param {*} data - данные для сохранения
   * @param {object} options - опции
   */
  setItem(key, data, options = {}) {
    try {
      // Сериализация данных
      let serializedData = JSON.stringify(data);
      
      // Шифрование, если ключ предоставлен
      if (this.encryptionKey) {
        serializedData = this.encrypt(serializedData);
      }
      
      // Добавляем метаданные для проверки целостности
      const storageData = {
        data: serializedData,
        timestamp: Date.now(),
        expiry: options.expiry || null // Время жизни в миллисекундах
      };
      
      this.storage.setItem(key, JSON.stringify(storageData));
      return true;
    } catch (e) {
      console.error('Ошибка при сохранении в хранилище:', e);
      return false;
    }
  }
  
  /**
   * Безопасное получение данных
   * @param {string} key - ключ
   * @returns {*} - данные или null
   */
  getItem(key) {
    try {
      const stored = this.storage.getItem(key);
      if (!stored) return null;
      
      const storageData = JSON.parse(stored);
      
      // Проверка срока действия
      if (storageData.expiry && 
          Date.now() - storageData.timestamp > storageData.expiry) {
        this.removeItem(key); // Удаляем просроченные данные
        return null;
      }
      
      // Дешифрование, если ключ предоставлен
      let data = storageData.data;
      if (this.encryptionKey) {
        data = this.decrypt(data);
      }
      
      return JSON.parse(data);
    } catch (e) {
      console.error('Ошибка при получении из хранилища:', e);
      return null;
    }
  }
  
  /**
   * Удаление данных
   * @param {string} key - ключ
   */
  removeItem(key) {
    try {
      this.storage.removeItem(key);
    } catch (e) {
      console.error('Ошибка при удалении из хранилища:', e);
    }
  }
  
  /**
   * Шифрование данных (упрощенная реализация)
   * В реальном приложении используйте криптографически стойкие методы
   * @param {string} data - данные для шифрования
   * @returns {string} - зашифрованные данные
   */
  encrypt(data) {
    if (!this.encryptionKey) return data;
    
    // Простое XOR шифрование для демонстрации
    // В реальном приложении используйте Web Crypto API
    let result = '';
    for (let i = 0; i < data.length; i++) {
      result += String.fromCharCode(
        data.charCodeAt(i) ^ this.encryptionKey.charCodeAt(i % this.encryptionKey.length)
      );
    }
    return btoa(result); // Base64 кодирование
  }
  
  /**
   * Расшифровка данных
   * @param {string} encryptedData - зашифрованные данные
   * @returns {string} - расшифрованные данные
   */
  decrypt(encryptedData) {
    if (!this.encryptionKey) return encryptedData;
    
    try {
      const data = atob(encryptedData); // Base64 декодирование
      let result = '';
      for (let i = 0; i < data.length; i++) {
        result += String.fromCharCode(
          data.charCodeAt(i) ^ this.encryptionKey.charCodeAt(i % this.encryptionKey.length)
        );
      }
      return result;
    } catch (e) {
      console.error('Ошибка при расшифровке:', e);
      return encryptedData;
    }
  }
  
  /**
   * Очистка всех данных с проверкой
   */
  clear() {
    // Можно добавить дополнительные проверки безопасности
    this.storage.clear();
  }
}

// Пример использования
const secureStorage = new SecureStorage('local', 'my-secret-key');

// Сохранение данных с ограничением по времени
secureStorage.setItem('userPreferences', { theme: 'dark', language: 'ru' }, { 
  expiry: 24 * 60 * 60 * 1000 // 24 часа
});

// Получение данных
const preferences = secureStorage.getItem('userPreferences');
console.log(preferences); // { theme: 'dark', language: 'ru' }
```

## Безопасные сетевые запросы

### 1. Безопасная обертка для fetch

```javascript
/**
 * Безопасная обертка для fetch API
 */
class SecureFetch {
  constructor(baseURL = '', defaultHeaders = {}) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      ...defaultHeaders
    };
  }
  
  /**
   * Выполняет безопасный запрос
   * @param {string} url - URL для запроса
   * @param {object} options - опции запроса
   * @returns {Promise} - промис с результатом
   */
  async request(url, options = {}) {
    // Формирование полного URL
    const fullURL = this.baseURL + url;
    
    // Проверка URL на безопасность
    if (!this.isValidUrl(fullURL)) {
      throw new Error('Небезопасный URL');
    }
    
    // Объединение заголовков
    const headers = {
      ...this.defaultHeaders,
      ...options.headers
    };
    
    // Проверка заголовков на потенциально опасные значения
    this.validateHeaders(headers);
    
    // Проверка тела запроса
    let body = options.body;
    if (body && typeof body === 'object') {
      body = this.sanitizeRequestBody(body);
    }
    
    try {
      const response = await fetch(fullURL, {
        ...options,
        headers,
        body: body ? JSON.stringify(body) : undefined
      });
      
      // Проверка статуса ответа
      if (!response.ok) {
        throw new Error(`HTTP ошибка! статус: ${response.status}`);
      }
      
      // Проверка типа контента
      const contentType = response.headers.get('content-type');
      if (!contentType || !contentType.includes('application/json')) {
        throw new Error('Неподдерживаемый тип контента');
      }
      
      const data = await response.json();
      
      // Санитизация ответа
      return this.sanitizeResponseData(data);
    } catch (error) {
      console.error('Ошибка безопасного запроса:', error);
      throw error;
    }
  }
  
  /**
   * Проверяет URL на безопасность
   * @param {string} url - URL для проверки
   * @returns {boolean} - true если URL безопасен
   */
  isValidUrl(url) {
    try {
      const parsedUrl = new URL(url);
      // Разрешаем только http и https
      return ['http:', 'https:'].includes(parsedUrl.protocol);
    } catch (e) {
      return false;
    }
  }
  
  /**
   * Проверяет заголовки на безопасность
   * @param {object} headers - заголовки для проверки
   */
  validateHeaders(headers) {
    // Запрещаем потенциально опасные заголовки
    const forbiddenHeaders = [
      'connection', 'keep-alive', 'proxy-authenticate', 
      'proxy-authorization', 'te', 'trailers', 'transfer-encoding', 'upgrade'
    ];
    
    for (const header in headers) {
      if (forbiddenHeaders.includes(header.toLowerCase())) {
        throw new Error(`Заголовок ${header} запрещен`);
      }
    }
  }
  
  /**
   * Санитизирует тело запроса
   * @param {object} body - тело запроса
   * @returns {object} - санитизированное тело
   */
  sanitizeRequestBody(body) {
    // Простая санитизация - удаление потенциально опасных полей
    const sanitized = { ...body };
    
    // Удаляем поля, которые могут содержать скрипты
    for (const key in sanitized) {
      if (typeof sanitized[key] === 'string') {
        sanitized[key] = escapeHtml(sanitized[key]);
      }
    }
    
    return sanitized;
  }
  
  /**
   * Санитизирует данные ответа
   * @param {object} data - данные ответа
   * @returns {object} - санитизированные данные
   */
  sanitizeResponseData(data) {
    // Рекурсивная санитизация объектов и массивов
    if (typeof data === 'string') {
      return escapeHtml(data);
    } else if (Array.isArray(data)) {
      return data.map(item => this.sanitizeResponseData(item));
    } else if (data !== null && typeof data === 'object') {
      const sanitized = {};
      for (const key in data) {
        sanitized[key] = this.sanitizeResponseData(data[key]);
      }
      return sanitized;
    }
    return data;
  }
  
  /**
   * GET запрос
   * @param {string} url - URL
   * @param {object} options - опции
   * @returns {Promise} - результат запроса
   */
  get(url, options = {}) {
    return this.request(url, { ...options, method: 'GET' });
  }
  
  /**
   * POST запрос
   * @param {string} url - URL
   * @param {object} data - данные для отправки
   * @param {object} options - опции
   * @returns {Promise} - результат запроса
   */
  post(url, data, options = {}) {
    return this.request(url, { 
      ...options, 
      method: 'POST',
      body: data 
    });
  }
  
  /**
   * PUT запрос
   * @param {string} url - URL
   * @param {object} data - данные для отправки
   * @param {object} options - опции
   * @returns {Promise} - результат запроса
   */
  put(url, data, options = {}) {
    return this.request(url, { 
      ...options, 
      method: 'PUT',
      body: data 
    });
  }
  
  /**
   * DELETE запрос
   * @param {string} url - URL
   * @param {object} options - опции
   * @returns {Promise} - результат запроса
   */
  delete(url, options = {}) {
    return this.request(url, { ...options, method: 'DELETE' });
  }
}

// Пример использования
const secureClient = new SecureFetch('https://api.example.com');

try {
  const userData = await secureClient.get('/user/123');
  console.log(userData);
} catch (error) {
  console.error('Ошибка запроса:', error);
}
```

## Практические советы

- Всегда санитизируйте пользовательский ввод перед использованием
- Используйте Content Security Policy (CSP) для дополнительной защиты
- Проверяйте и валидируйте все данные, поступающие от клиента
- Используйте HTTPS для всех запросов
- Не храните чувствительные данные в localStorage/sessionStorage без шифрования
- Реализуйте защиту от частых запросов (rate limiting)
- Регулярно обновляйте зависимости и проверяйте их на уязвимости

## Связанные темы

- [[frontend-security-basics]]
- [[authentication-security-patterns]]
- [[data-validation-techniques]]