---
aliases: [OWASP, Security Vulnerabilities, Безопасность OWASP]
tags: [security, frontend, vulnerabilities]
---

# OWASP Top 10: Уязвимости и меры защиты фронтенд-приложений

## Обзор

OWASP Top 10 - это список из 10 наиболее критических веб-уязвимостей, опубликованный Open Web Application Security Project. Этот список обновляется каждые несколько лет и служит важным руководством для разработчиков и специалистов по безопасности. Понимание этих уязвимостей особенно важно для фронтенд-разработчиков, так как клиентская часть приложения часто становится целью атак.

## 1. Инъекции (Injection)

### Описание
Инъекции происходят, когда ненадежные данные отправляются в интерпретатор в качестве команды или запроса. Это может привести к потере данных, отказу в обслуживании или компрометации системы.

### Фронтенд-импликации
Хотя инъекции чаще всего происходят на серверной стороне, фронтенд может быть связан:
- XSS атаки через инъекции HTML/JavaScript
- SQL-инъекции, если клиент отправляет непроверенные данные

### Меры защиты
```javascript
// Правильная обработка пользовательского ввода
function sanitizeInput(input) {
  // Экранирование HTML
  return input.replace(/[<>&"']/g, function(match) {
    switch(match) {
      case '<': return '&lt;';
      case '>': return '&gt;';
      case '&': return '&amp;';
      case '"': return '&quot;';
      case "'": return '&#x27;';
      default: return match;
    }
  });
}

// Использование безопасных методов рендеринга
function safeRender(content) {
  const element = document.createElement('div');
  element.textContent = content; // Использует textContent вместо innerHTML
  return element.innerHTML;
}
```

## 2. Уязвимости аутентификации (Broken Authentication)

### Описание
Плохо реализованные механизмы аутентификации и управления сессиями могут позволить злоумышленникам компрометировать пароли, ключи или токены.

### Фронтенд-импликации
- Хранение токенов в небезопасных местах
- Неправильное управление сессиями
- Отсутствие проверок безопасности токенов

### Меры защиты
```javascript
// Безопасное хранение токенов
class SecureTokenStorage {
  constructor() {
    this.tokenKey = 'auth_token';
  }

  // Хранение токена в httpOnly куках (предпочтительно на сервере)
  // или в памяти (не в localStorage для чувствительных данных)
  setToken(token) {
    // Не использовать localStorage для токенов доступа
    // использовать sessionStorage или память приложения
    this.token = token;
  }

  getToken() {
    return this.token;
  }

  clearToken() {
    this.token = null;
  }
}

// Проверка срока действия токена
function isTokenExpired(token) {
  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    const currentTime = Math.floor(Date.now() / 1000);
    return payload.exp < currentTime;
  } catch (error) {
    return true; // Если токен поврежден, считаем его просроченным
  }
}

// Обновление токена
async function refreshToken() {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      credentials: 'include', // Отправка httpOnly куки
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if (response.ok) {
      const { accessToken } = await response.json();
      tokenStorage.setToken(accessToken);
      return accessToken;
    }
  } catch (error) {
    console.error('Ошибка обновления токена:', error);
    // Перенаправление на страницу входа
    window.location.href = '/login';
  }
}
```

## 3. Чувствительные данные в открытом доступе (Sensitive Data Exposure)

### Описание
Неадекватная защита конфиденциальных данных, таких как финансовая информация, здоровье, персональные данные.

### Фронтенд-импликации
- Отображение чувствительных данных в открытом виде
- Небезопасная передача данных
- Хранение чувствительных данных в клиентском коде

### Меры защиты
```javascript
// Маскировка чувствительных данных
function maskSensitiveData(data) {
  if (typeof data === 'string') {
    // Маскировка номеров кредитных карт
    data = data.replace(/(\d{4})\d{4}(\d{4})/g, '$1****$2');
    // Маскировка email
    data = data.replace(/(\w{2})\w+(@\w+)/g, '$1***$2');
  }
  return data;
}

// Шифрование данных на клиенте (дополнительно к серверному шифрованию)
import CryptoJS from 'crypto-js';

function encryptData(data, secretKey) {
  return CryptoJS.AES.encrypt(JSON.stringify(data), secretKey).toString();
}

function decryptData(ciphertext, secretKey) {
  const bytes = CryptoJS.AES.decrypt(ciphertext, secretKey);
  return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
}

// Пример использования
const sensitiveData = {
  creditCard: '1234567890123456',
  ssn: '123-45-6789'
};

const encrypted = encryptData(sensitiveData, 'my-secret-key');
console.log('Зашифрованные данные:', encrypted);
```

## 4. XML External Entities (XXE)

### Описание
Уязвимости XXE происходят при обработке XML-данных с внешними сущностями.

### Фронтенд-импликации
Хотя XXE в основном серверная уязвимость, фронтенд может быть затронут при:
- Парсинге XML-данных в браузере
- Отправке XML-данных на сервер

### Меры защиты
```javascript
// Безопасный парсинг XML
function safeParseXML(xmlString) {
  // Отключение внешних сущностей в DOMParser
  const parser = new DOMParser();
  // В современных браузерах внешние сущности отключены по умолчанию
  return parser.parseFromString(xmlString, 'text/xml');
}

// Альтернативный подход - использование безопасных библиотек
import { parse } from 'fast-xml-parser';

function safeXMLParse(xmlString) {
  const options = {
    ignoreDeclaration: true,
    ignoreAttributes: false,
    allowBooleanAttributes: false,
    parseTagValue: false,
    parseAttributeValue: false,
    trimValues: true,
    // Отключение внешних сущностей
    processEntities: false
  };
  
  return parse(xmlString, options);
}
```

## 5. Нарушение контроля доступа (Broken Access Control)

### Описание
Ограничения доступа к функциям и данным не применяются должным образом.

### Фронтенд-импликации
- Отображение защищенных элементов интерфейса непривилегированным пользователям
- Отправка запросов к защищенным ресурсам без проверки прав

### Меры защиты
```javascript
// Управление доступом на фронтенде
class AccessControl {
  constructor(userRole) {
    this.userRole = userRole;
    this.permissions = {
      admin: ['read', 'write', 'delete', 'manage_users'],
      user: ['read', 'write'],
      guest: ['read']
    };
  }

  hasPermission(action) {
    const userPermissions = this.permissions[this.userRole] || [];
    return userPermissions.includes(action);
  }

  canAccess(resource, action) {
    // Проверка доступа к ресурсу
    return this.hasPermission(action);
  }

  // Отображение элементов интерфейса в зависимости от прав
  renderIfAllowed(permission, component) {
    return this.hasPermission(permission) ? component : null;
  }
}

// Пример использования в React-компоненте
import React from 'react';

const AdminPanel = ({ userRole }) => {
  const accessControl = new AccessControl(userRole);

  return (
    <div>
      <h2>Панель управления</h2>
      
      {accessControl.renderIfAllowed('read', (
        <div>Просмотр данных</div>
      ))}
      
      {accessControl.renderIfAllowed('write', (
        <button>Редактировать</button>
      ))}
      
      {accessControl.renderIfAllowed('delete', (
        <button>Удалить</button>
      ))}
      
      {accessControl.renderIfAllowed('manage_users', (
        <div>Управление пользователями</div>
      ))}
    </div>
  );
};
```

## 6. Межсайтовый скриптинг (Cross-Site Scripting - XSS)

### Описание
XSS позволяет злоумышленникам вставлять вредоносный скрипт в доверенные веб-сайты, которые просматривают другие пользователи.

### Фронтенд-импликации
- Внедрение скриптов через формы
- Отображение непроверенного пользовательского контента
- Использование небезопасных методов рендеринга

### Меры защиты
```javascript
// Защита от XSS
class XSSProtection {
  // Экранирование HTML
  static escapeHtml(text) {
    const map = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#039;'
    };
    
    return text.replace(/[&<>"']/g, m => map[m]);
  }

  // Безопасное отображение пользовательского контента
  static safeDisplay(content) {
    const temp = document.createElement('div');
    temp.textContent = content; // Использует textContent для экранирования
    return temp.innerHTML;
  }

  // Проверка и очистка HTML (используя библиотеку sanitize-html)
  static sanitizeHTML(html) {
    // В реальном приложении использовать библиотеку вроде DOMPurify
    if (typeof DOMPurify !== 'undefined') {
      return DOMPurify.sanitize(html);
    }
    return html; // Резервный вариант
  }

  // Безопасное создание URL
  static safeUrl(url) {
    try {
      const parsedUrl = new URL(url);
      // Проверка протокола
      if (!['http:', 'https:', 'mailto:'].includes(parsedUrl.protocol)) {
        throw new Error('Небезопасный протокол');
      }
      return parsedUrl.href;
    } catch (error) {
      console.error('Небезопасный URL:', error);
      return '#';
    }
  }
}

// Пример использования
const userInput = '<script>alert("XSS")</script>Привет, мир!';
const safeOutput = XSSProtection.escapeHtml(userInput);
console.log(safeOutput); // &lt;script&gt;alert("XSS")&lt;/script&gt;Привет, мир!

// В React компоненте
const SafeComponent = ({ userContent }) => {
  return (
    <div 
      dangerouslySetInnerHTML={{ 
        __html: XSSProtection.sanitizeHTML(userContent) 
      }} 
    />
  );
};
```

## 7. Небезопасная десериализация (Insecure Deserialization)

### Описание
Приложения и API, которые десериализуют ненадежные данные, могут быть уязвимы к атакам.

### Фронтенд-импликации
- Десериализация ненадежных данных из localStorage/sessionStorage
- Обработка JSON-данных без проверки

### Меры защиты
```javascript
// Безопасная десериализация
class SafeDeserializer {
  static safeParseJSON(jsonString) {
    try {
      // Проверка на наличие потенциально опасных свойств
      const parsed = JSON.parse(jsonString);
      
      // Проверка на наличие опасных свойств (например, __proto__, constructor)
      this.validateObject(parsed);
      
      return parsed;
    } catch (error) {
      console.error('Ошибка безопасной десериализации:', error);
      throw new Error('Небезопасные данные для десериализации');
    }
  }

  static validateObject(obj, depth = 0) {
    if (depth > 10) { // Защита от рекурсии
      throw new Error('Слишком глубокий объект');
    }

    if (obj !== null && typeof obj === 'object') {
      // Проверка на опасные свойства
      if (obj.hasOwnProperty('__proto__') || obj.hasOwnProperty('constructor')) {
        throw new Error('Обнаружены потенциально опасные свойства');
      }

      // Рекурсивная проверка вложенных объектов
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          this.validateObject(obj[key], depth + 1);
        }
      }
    }
  }

  // Безопасное восстановление из localStorage
  static safeGetFromStorage(key) {
    try {
      const stored = localStorage.getItem(key);
      if (!stored) return null;
      
      return this.safeParseJSON(stored);
    } catch (error) {
      console.error('Ошибка при получении из хранилища:', error);
      localStorage.removeItem(key); // Удаление поврежденных данных
      return null;
    }
  }
}
```

## 8. Использование компонентов с известными уязвимостями

### Описание
Приложения и API, использующие уязвимые компоненты, могут подвергаться атакам.

### Фронтенд-импликации
- Использование устаревших библиотек
- Небезопасные зависимости
- Отсутствие регулярного обновления

### Меры защиты
```javascript
// Проверка уязвимостей в зависимостях
// package.json скрипты
{
  "scripts": {
    "audit": "npm audit",
    "audit:fix": "npm audit fix",
    "outdated": "npm outdated"
  }
}

// Использование Snyk или других инструментов
// .snyk файл конфигурации
module.exports = {
  // Автоматическая проверка уязвимостей
  auto: true,
  // Игнорирование определенных уязвимостей (временно)
  ignore: [
    // {
    //   id: 'SNYK-JS-EXAMPLE-123',
    //   reason: 'Временно игнорируется до фикса'
    // }
  ]
};
```

## 9. Недостаточная защита логов и мониторинга

### Описание
Недостаточные логи и мониторинг позволяют злоумышленникам атаковать системы без обнаружения.

### Фронтенд-импликации
- Отсутствие логирования безопасности на клиенте
- Потенциальная утечка информации через логи

### Меры защиты
```javascript
// Безопасное логирование
class SecureLogger {
  constructor() {
    this.logLevel = 'info';
    this.sensitiveDataPatterns = [
      /password/gi,
      /token/gi,
      /secret/gi,
      /key/gi,
      /[A-Z0-9]{32,}/gi // потенциальные токены
    ];
  }

  log(level, message, data = {}) {
    if (!this.shouldLog(level)) return;

    // Маскировка чувствительных данных в логах
    const sanitizedData = this.maskSensitiveData(data);
    const sanitizedMessage = this.maskSensitiveDataInString(message);

    // Отправка логов на сервер (если необходимо)
    console[level](`${new Date().toISOString()} [${level.toUpperCase()}] ${sanitizedMessage}`, sanitizedData);
  }

  maskSensitiveDataInString(str) {
    if (typeof str !== 'string') return str;

    let result = str;
    this.sensitiveDataPatterns.forEach(pattern => {
      result = result.replace(pattern, '[REDACTED]');
    });
    return result;
  }

  maskSensitiveData(obj) {
    if (obj === null || typeof obj !== 'object') {
      return obj;
    }

    const result = Array.isArray(obj) ? [] : {};
    
    for (const [key, value] of Object.entries(obj)) {
      if (this.isSensitiveKey(key)) {
        result[key] = '[REDACTED]';
      } else if (typeof value === 'object' && value !== null) {
        result[key] = this.maskSensitiveData(value);
      } else if (typeof value === 'string') {
        result[key] = this.maskSensitiveDataInString(value);
      } else {
        result[key] = value;
      }
    }
    
    return result;
  }

  isSensitiveKey(key) {
    return this.sensitiveDataPatterns.some(pattern => pattern.test(key));
  }

  shouldLog(level) {
    const levels = { error: 0, warn: 1, info: 2, debug: 3 };
    return levels[level] <= levels[this.logLevel];
  }

  // Методы для конкретных уровней логирования
  error(message, data) { this.log('error', message, data); }
  warn(message, data) { this.log('warn', message, data); }
  info(message, data) { this.log('info', message, data); }
  debug(message, data) { this.log('debug', message, data); }
}

// Использование
const logger = new SecureLogger();

logger.info('Пользователь вошел в систему', {
  userId: 123,
  timestamp: new Date(),
  token: 'abc123def456' // будет замаскировано
});
```

## 10. Небезопасная загрузка файлов (Server-Side Request Forgery - SSRF)

### Описание
SSRF позволяет злоумышленникам заставить сервер выполнить запросы к недоверенным ресурсам.

### Фронтенд-импликации
Хотя SSRF в основном серверная уязвимость, фронтенд может быть связан:
- Загрузка файлов с внешних источников
- Предоставление URL для загрузки данных

### Меры защиты
```javascript
// Безопасная загрузка файлов
class SecureFileUpload {
  constructor() {
    this.allowedFileTypes = ['image/jpeg', 'image/png', 'application/pdf'];
    this.maxFileSize = 5 * 1024 * 1024; // 5MB
    this.allowedDomains = ['trusted-domain.com', 'cdn.example.com'];
  }

  async uploadFile(file) {
    // Проверка типа файла
    if (!this.allowedFileTypes.includes(file.type)) {
      throw new Error('Недопустимый тип файла');
    }

    // Проверка размера файла
    if (file.size > this.maxFileSize) {
      throw new Error('Файл слишком большой');
    }

    // Проверка содержимого файла (базовая проверка)
    const isValid = await this.validateFileContent(file);
    if (!isValid) {
      throw new Error('Файл содержит недопустимое содержимое');
    }

    // Загрузка файла
    const formData = new FormData();
    formData.append('file', file);

    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });

    if (!response.ok) {
      throw new Error('Ошибка загрузки файла');
    }

    return response.json();
  }

  async validateFileContent(file) {
    // Проверка сигнатуры файла
    const buffer = await file.arrayBuffer();
    const view = new Uint8Array(buffer);
    
    // Проверка сигнатуры JPEG
    if (file.type === 'image/jpeg') {
      return view[0] === 0xFF && view[1] === 0xD8;
    }
    
    // Проверка сигнатуры PNG
    if (file.type === 'image/png') {
      return view[0] === 0x89 && view[1] === 0x50 && view[2] === 0x4E && view[3] === 0x47;
    }
    
    return true; // Для других типов файлов базовая проверка
  }

  // Безопасная загрузка по URL
  async uploadFromUrl(url) {
    try {
      // Проверка домена
      const urlObj = new URL(url);
      if (!this.allowedDomains.includes(urlObj.hostname)) {
        throw new Error('Недопустимый домен для загрузки');
      }

      // Загрузка файла через сервер (для дополнительной безопасности)
      const response = await fetch('/api/proxy-upload', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ url })
      });

      if (!response.ok) {
        throw new Error('Ошибка загрузки файла по URL');
      }

      return response.json();
    } catch (error) {
      console.error('Ошибка безопасной загрузки:', error);
      throw error;
    }
  }
}
```

## Заключение

OWASP Top 10 представляет собой важный список уязвимостей, с которыми должны быть знакомы все фронтенд-разработчики. Понимание этих уязвимостей и применение соответствующих мер защиты помогает создавать более безопасные веб-приложения. Важно помнить, что безопасность - это не одноразовая задача, а непрерывный процесс, требующий регулярного обновления знаний и внедрения новых практик защиты.

Связанные концепции: [[Frontend Security]], [[Security Testing]], [[Authentication]], [[Input Validation]]