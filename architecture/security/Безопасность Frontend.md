# Безопасность Frontend

Безопасность frontend — это критически важный аспект разработки веб-приложений, направленный на защиту пользовательских данных, предотвращение атак и обеспечение доверия пользователей. Frontend безопасность охватывает широкий спектр уязвимостей и методов защиты, которые должны учитываться на всех этапах разработки.

## Что такое Безопасность Frontend

Безопасность frontend — это совокупность практик, технологий и архитектурных решений, направленных на защиту клиентской части веб-приложений от различных угроз и атак. В отличие от backend безопасности, frontend безопасность фокусируется на защите пользовательского интерфейса и данных, обрабатываемых в браузере.

## Основные Угрозы и Уязвимости

### 1. Межсайтовый Скриптинг (XSS)

XSS — одна из самых распространенных уязвимостей, при которой злоумышленник внедряет вредоносный код в веб-страницы, которые затем выполняются в браузерах других пользователей.

#### Типы XSS:

##### Reflected XSS
Вредоносный скрипт отражается от веб-сервера, например, в сообщении об ошибке или результате поиска.

Пример уязвимости:
```javascript
// Небезопасный код
const searchTerm = getQueryParam('search');
document.getElementById('results').innerHTML = `Результаты поиска для: ${searchTerm}`;
```

Защита:
```javascript
// Безопасный код
const searchTerm = getQueryParam('search');
const sanitizedTerm = sanitizeHTML(searchTerm);
document.getElementById('results').textContent = `Результаты поиска для: ${sanitizedTerm}`;
```

##### Stored XSS
Вредоносный скрипт сохраняется на сервере и передается другим пользователям.

Пример уязвимости:
```javascript
// Небезопасный код
const comment = getUserComment();
document.getElementById('comments').innerHTML += `<div>${comment}</div>`;
```

Защита:
```javascript
// Безопасный код
const comment = getUserComment();
const commentElement = document.createElement('div');
commentElement.textContent = comment;
document.getElementById('comments').appendChild(commentElement);
```

##### DOM-based XSS
Уязвимость возникает в клиентском коде, когда данные из недоверенного источника передаются в_sink_, который может выполнить код.

Пример уязвимости:
```javascript
// Небезопасный код
const hash = window.location.hash.substring(1);
document.getElementById('content').innerHTML = decodeURIComponent(hash);
```

Защита:
```javascript
// Безопасный код
const hash = window.location.hash.substring(1);
const content = decodeURIComponent(hash);
document.getElementById('content').textContent = content;
```

### 2. Подделка Межсайтовых Запросов (CSRF)

CSRF — атака, при которой злоумышленник заставляет жертву выполнить нежелательные действия на веб-сайте, на котором она аутентифицирована.

Пример атаки:
```html
<!-- Вредоносный сайт -->
<img src="http://bank.com/transfer?to=attacker&amount=1000" width="0" height="0">
```

Защита:
```javascript
// Использование CSRF токенов
class CSRFProtection {
  async getCSRFToken() {
    const response = await fetch('/api/csrf-token', {
      credentials: 'include'
    });
    const { token } = await response.json();
    return token;
  }
  
  async request(url, options = {}) {
    const csrfToken = await this.getCSRFToken();
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'X-CSRF-Token': csrfToken,
      },
      credentials: 'include'
    });
  }
}
```

### 3. Clickjacking

Clickjacking — атака, при которой пользователь обманом заставляется кликнуть на что-то отличное от того, что он видит.

Защита с помощью X-Frame-Options:
```html
<!-- В HTTP заголовках -->
X-Frame-Options: DENY
<!-- Или -->
X-Frame-Options: SAMEORIGIN
```

Защита с помощью Content Security Policy:
```html
<!-- В HTTP заголовках -->
Content-Security-Policy: frame-ancestors 'none';
<!-- Или -->
Content-Security-Policy: frame-ancestors 'self';
```

### 4. Утечка Данных

Утечка данных может происходить через различные каналы:

#### Автозаполнение Форм
```html
<!-- Отключение автозаполнения для чувствительных полей -->
<input type="password" autocomplete="new-password">
<input type="text" autocomplete="off">
```

#### История Навигации
```javascript
// Очистка истории для чувствительных страниц
if (window.history && window.history.replaceState) {
  window.history.replaceState(null, '', window.location.href);
}
```

## Архитектурные Подходы к Безопасности

### 1. Принцип наименьших привилегий

Каждый компонент должен иметь минимально необходимые права и доступы.

Пример:
```javascript
// Ограничение доступа к API на основе ролей
class SecureApiClient {
  constructor(userRole) {
    this.userRole = userRole;
  }
  
  async request(endpoint, options = {}) {
    // Проверка прав доступа
    if (!this.hasPermission(endpoint, options.method)) {
      throw new Error('Access denied');
    }
    
    return fetch(`/api${endpoint}`, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${this.getToken()}`,
      }
    });
  }
  
  hasPermission(endpoint, method) {
    const permissions = {
      'admin': { '/users': ['GET', 'POST', 'PUT', 'DELETE'] },
      'user': { '/profile': ['GET', 'PUT'] }
    };
    
    return permissions[this.userRole]?.[endpoint]?.includes(method) || false;
  }
}
```

### 2. Защита через Изоляцию

Изоляция компонентов и данных для предотвращения несанкционированного доступа.

Пример:
```javascript
// Использование Shadow DOM для изоляции стилей
class SecureComponent extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: 'closed' });
    
    const style = document.createElement('style');
    style.textContent = `
      .secure-content {
        border: 1px solid #ccc;
        padding: 10px;
      }
    `;
    
    const content = document.createElement('div');
    content.className = 'secure-content';
    content.textContent = 'Secure content';
    
    shadow.appendChild(style);
    shadow.appendChild(content);
  }
}

customElements.define('secure-component', SecureComponent);
```

### 3. Валидация и Санитизация

Все входные данные должны быть проверены и очищены перед использованием.

Пример:
```javascript
// Библиотека для валидации и санитизации
class InputValidator {
  static sanitizeHTML(input) {
    const div = document.createElement('div');
    div.textContent = input;
    return div.innerHTML;
  }
  
  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  static validatePassword(password) {
    // Минимум 8 символов, хотя бы одна цифра, одна буква и один специальный символ
    const passwordRegex = /^(?=.*[A-Za-z])(?=.*\d)(?=.*[@$!%*#?&])[A-Za-z\d@$!%*#?&]{8,}$/;
    return passwordRegex.test(password);
  }
  
  static sanitizeURL(url) {
    try {
      const parsed = new URL(url);
      // Разрешаем только HTTP и HTTPS протоколы
      if (parsed.protocol === 'http:' || parsed.protocol === 'https:') {
        return parsed.href;
      }
      return null;
    } catch (e) {
      return null;
    }
  }
}
```

## Content Security Policy (CSP)

CSP — мощный механизм защиты от XSS и других атак, позволяющий разработчикам контролировать, какие ресурсы могут быть загружены и выполнены на странице.

Пример CSP заголовка:
```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval' https://apis.google.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: https:;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://api.example.com;
  frame-src 'self' https://accounts.google.com;
  object-src 'none';
  base-uri 'self';
  form-action 'self';
```

Реализация в HTML:
```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline'">
```

## Защита от Атак на Каналы Времени

Атаки на каналы времени могут использоваться для извлечения информации из приложения.

Пример защиты:
```javascript
// Константное время сравнения для предотвращения атак по времени
function timingSafeEqual(a, b) {
  if (a.length !== b.length) {
    // Заполняем случайными данными для маскировки длины
    b = a + Math.random().toString();
  }
  
  let result = 0;
  for (let i = 0; i < a.length; i++) {
    result |= a.charCodeAt(i) ^ b.charCodeAt(i);
  }
  
  return result === 0;
}
```

## Защита Конфиденциальных Данных

### 1. Хранение Данных

#### LocalStorage и SessionStorage
```javascript
// Не храните чувствительные данные в localStorage/sessionStorage
// localStorage.setItem('authToken', token); // НЕПРАВИЛЬНО

// Используйте httpOnly cookies для токенов
class SecureTokenStorage {
  setToken(token) {
    // Токен устанавливается через HTTP заголовки Set-Cookie
    // с флагами httpOnly, secure, sameSite
  }
  
  getToken() {
    // Токен автоматически отправляется браузером
    // и недоступен через JavaScript
    return null;
  }
}
```

#### IndexedDB
```javascript
// Шифрование данных перед сохранением в IndexedDB
class SecureIndexedDB {
  constructor() {
    this.db = null;
  }
  
  async encryptData(data, key) {
    // Реализация шифрования
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(JSON.stringify(data));
    
    const encrypted = await crypto.subtle.encrypt(
      { name: "AES-GCM" },
      key,
      dataBuffer
    );
    
    return new Uint8Array(encrypted);
  }
  
  async decryptData(encryptedData, key) {
    // Реализация дешифрования
    const decrypted = await crypto.subtle.decrypt(
      { name: "AES-GCM" },
      key,
      encryptedData
    );
    
    const decoder = new TextDecoder();
    return JSON.parse(decoder.decode(decrypted));
  }
}
```

### 2. Передача Данных

#### HTTPS
Всегда используйте HTTPS для передачи данных:
```javascript
// Проверка использования HTTPS
if (location.protocol !== 'https:') {
  console.warn('Приложение должно использоваться только через HTTPS');
}
```

#### Защита от Перехвата
```javascript
// Использование Subresource Integrity (SRI)
// <script src="https://example.com/script.js" 
//         integrity="sha384-..." 
//         crossorigin="anonymous"></script>
```

## Защита от Атак на Аутентификацию

### 1. Защита от Брутфорса

```javascript
class LoginProtection {
  constructor() {
    this.attempts = new Map();
    this.blocked = new Set();
  }
  
  async login(username, password) {
    // Проверка блокировки
    if (this.blocked.has(username)) {
      throw new Error('Account temporarily blocked');
    }
    
    // Проверка попыток
    const attempts = this.attempts.get(username) || 0;
    if (attempts >= 5) {
      this.blocked.add(username);
      setTimeout(() => this.blocked.delete(username), 300000); // 5 минут
      throw new Error('Too many failed attempts');
    }
    
    try {
      const result = await this.authenticate(username, password);
      // Сброс попыток при успешной аутентификации
      this.attempts.delete(username);
      return result;
    } catch (error) {
      // Увеличение счетчика попыток
      this.attempts.set(username, attempts + 1);
      throw error;
    }
  }
}
```

### 2. Защита Токенов

```javascript
class TokenManager {
  constructor() {
    this.refreshTimeout = null;
  }
  
  setTokens(accessToken, refreshToken) {
    // Хранение refresh токена в httpOnly cookie
    document.cookie = `refreshToken=${refreshToken}; HttpOnly; Secure; SameSite=Strict`;
    
    // Хранение access токена в памяти (не в localStorage)
    this.accessToken = accessToken;
    
    // Автоматическое обновление токена
    this.scheduleTokenRefresh();
  }
  
  scheduleTokenRefresh() {
    // Обновление токена за 5 минут до истечения срока действия
    const payload = JSON.parse(atob(this.accessToken.split('.')[1]));
    const expirationTime = payload.exp * 1000;
    const refreshTime = expirationTime - Date.now() - 300000;
    
    if (refreshTime > 0) {
      this.refreshTimeout = setTimeout(() => this.refreshToken(), refreshTime);
    }
  }
  
  async refreshToken() {
    try {
      const response = await fetch('/api/auth/refresh', {
        method: 'POST',
        credentials: 'include' // Для отправки cookies
      });
      
      if (response.ok) {
        const { accessToken } = await response.json();
        this.accessToken = accessToken;
        this.scheduleTokenRefresh();
      } else {
        // Перенаправление на страницу входа
        window.location.href = '/login';
      }
    } catch (error) {
      console.error('Token refresh failed:', error);
      window.location.href = '/login';
    }
  }
}
```

## Мониторинг и Логирование

### 1. Обнаружение Подозрительной Активности

```javascript
class SecurityMonitor {
  constructor() {
    this.suspiciousActivities = [];
  }
  
  logActivity(activity, data) {
    const logEntry = {
      timestamp: Date.now(),
      activity,
      data,
      userAgent: navigator.userAgent,
      ip: this.getIP() // Получение IP через сервер
    };
    
    // Отправка в систему мониторинга
    this.sendToMonitoring(logEntry);
    
    // Проверка на подозрительную активность
    this.checkForSuspiciousActivity(logEntry);
  }
  
  checkForSuspiciousActivity(logEntry) {
    // Примеры подозрительной активности:
    // - Много неудачных попыток входа
    // - Необычные паттерны запросов
    // - Попытки XSS
    
    if (this.isSuspicious(logEntry)) {
      this.reportSuspiciousActivity(logEntry);
    }
  }
  
  isSuspicious(logEntry) {
    // Реализация логики обнаружения
    // Может включать машинное обучение или правила
    return false;
  }
}
```

### 2. Отчеты о Безопасности

```javascript
// Content Security Policy violation reports
document.addEventListener('securitypolicyviolation', (event) => {
  const violationReport = {
    documentURI: event.documentURI,
    violatedDirective: event.violatedDirective,
    effectiveDirective: event.effectiveDirective,
    originalPolicy: event.originalPolicy,
    blockedURI: event.blockedURI,
    statusCode: event.statusCode
  };
  
  // Отправка отчета
  fetch('/api/security-violation-report', {
    method: 'POST',
    body: JSON.stringify(violationReport),
    headers: {
      'Content-Type': 'application/json'
    }
  });
});
```

## Тестирование Безопасности

### 1. Автоматизированное Тестирование

```javascript
// Тесты на уязвимости XSS
describe('XSS Protection', () => {
  test('should sanitize user input', () => {
    const maliciousInput = '<script>alert("XSS")</script>';
    const sanitized = InputValidator.sanitizeHTML(maliciousInput);
    expect(sanitized).not.toContain('<script>');
  });
  
  test('should prevent DOM-based XSS', () => {
    // Тестирование с использованием JSDOM
    const maliciousHash = '#<img src=x onerror=alert(1)>';
    window.location.hash = maliciousHash;
    
    // Проверка, что код не выполняется
    expect(window.alert).not.toHaveBeenCalled();
  });
});
```

### 2. Ручное Тестирование

Используйте инструменты для ручного тестирования:
- OWASP ZAP
- Burp Suite
- Browser Developer Tools

## Лучшие Практики

### 1. Регулярные Обновления

```javascript
// Проверка версий зависимостей
class DependencyChecker {
  async checkForVulnerabilities() {
    const dependencies = this.getDependencies();
    const vulnerabilities = await this.scanVulnerabilities(dependencies);
    
    if (vulnerabilities.length > 0) {
      console.warn('Найдены уязвимости:', vulnerabilities);
      // Уведомление команды разработки
      this.notifyTeam(vulnerabilities);
    }
  }
}
```

### 2. Обучение Команды

Регулярно проводите обучение по безопасности для всех членов команды:
- Code reviews с фокусом на безопасность
- Регулярные тренинги
- Анализ инцидентов

## Связанные Концепции

- [[Архитектура]]
- [[Архитектура API и Интеграций]]
- [[Тестирование Архитектуры]]
- [[PWA Архитектура]]

## Теги

#security #frontend #xss #csrf #csp #authentication #authorization #vulnerabilities #web-security