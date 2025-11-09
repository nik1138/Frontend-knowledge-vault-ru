# CSRF (Межсайтовая подделка запроса)

CSRF (Cross-Site Request Forgery) - это тип атаки, при которой злоумышленник заставляет доверенного пользователя выполнить нежелательные действия на веб-сайте, на котором пользователь аутентифицирован. В отличие от XSS, при CSRF злоумышленник не пытается украсть данные напрямую, а заставляет пользователя выполнить нежелательные действия.

## Как работает CSRF

### Принцип атаки

CSRF-атака возможна, когда:

1. Пользователь аутентифицирован на сайте (например, банк)
2. Пользователь посещает вредоносный сайт
3. Вредоносный сайт отправляет запрос на целевой сайт от имени пользователя
4. Браузер автоматически отправляет куки аутентификации с запросом
5. Целевой сайт обрабатывает запрос как законный

```html
<!-- Пример вредоносной страницы -->
<!DOCTYPE html>
<html>
<head>
    <title>Безопасный сайт</title>
</head>
<body>
    <h1>Загрузка...</h1>
    
    <!-- Скрытый iframe, который отправляет запрос -->
    <iframe style="display:none" name="csrf-frame"></iframe>
    
    <form method="POST" action="https://bank.example.com/transfer" target="csrf-frame" id="csrf-form">
        <input type="hidden" name="toAccount" value="attacker-account">
        <input type="hidden" name="amount" value="1000">
        <input type="hidden" name="description" value="CSRF Transfer">
    </form>
    
    <script>
        // Автоматическая отправка формы при загрузке страницы
        document.getElementById('csrf-form').submit();
    </script>
</body>
</html>
```

## Типы CSRF атак

### 1. GET-based CSRF

```javascript
// Уязвимый GET запрос (плохая практика)
// http://bank.example.com/transfer?toAccount=attacker&amount=1000

// Злоумышленник может просто вставить изображение:
// <img src="http://bank.example.com/transfer?toAccount=attacker&amount=1000" style="display:none">
```

### 2. POST-based CSRF

```javascript
// Уязвимый POST запрос без защиты
// Форма на сайте банка:
/*
<form method="POST" action="/transfer">
    <input type="text" name="toAccount" value="">
    <input type="number" name="amount" value="">
    <input type="submit" value="Перевести">
</form>
*/

// Злоумышленник создает свою форму:
/*
<form method="POST" action="https://bank.example.com/transfer">
    <input type="hidden" name="toAccount" value="attacker-account">
    <input type="hidden" name="amount" value="1000">
    <input type="submit" value="Кликни для подарка!">
</form>
*/
```

### 3. JSON POST CSRF

```javascript
// Уязвимый JSON API запрос
fetch('https://api.example.com/settings', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        email: 'attacker@example.com',
        password: 'newpassword'
    })
});
```

## Защита от CSRF

### 1. CSRF токены

```javascript
// Генерация CSRF токена на сервере
function generateCSRFToken(sessionId) {
    const crypto = require('crypto');
    const token = crypto.randomBytes(32).toString('hex');
    
    // Сохраняем токен в сессии
    sessions[sessionId].csrfToken = token;
    return token;
}

// Проверка CSRF токена
function validateCSRFToken(receivedToken, sessionId) {
    const storedToken = sessions[sessionId].csrfToken;
    return receivedToken === storedToken;
}

// Пример использования в Express.js
app.use((req, res, next) => {
    if (req.method === 'POST' || req.method === 'PUT' || req.method === 'DELETE') {
        const token = req.body._csrf || req.headers['x-csrf-token'];
        if (!validateCSRFToken(token, req.sessionID)) {
            return res.status(403).send('CSRF токен недействителен');
        }
    }
    next();
});
```

```javascript
// Клиентский код для работы с CSRF токенами
class CSRFManager {
    constructor() {
        this.token = this.getTokenFromMetaTag();
    }
    
    // Получение токена из meta тега
    getTokenFromMetaTag() {
        const tokenElement = document.querySelector('meta[name="csrf-token"]');
        return tokenElement ? tokenElement.getAttribute('content') : null;
    }
    
    // Получение токена из куки
    getTokenFromCookie() {
        return document.cookie
            .split(';')
            .find(c => c.trim().startsWith('XSRF-TOKEN='))
            ?.split('=')[1];
    }
    
    // Добавление токена к fetch запросам
    async fetch(url, options = {}) {
        options.headers = {
            'X-CSRF-Token': this.token,
            'Content-Type': 'application/json',
            ...options.headers
        };
        
        return await fetch(url, options);
    }
    
    // Добавление токена к формам
    addTokenToForm(form) {
        if (!this.token) return;
        
        // Проверяем, есть ли уже токен в форме
        const existingToken = form.querySelector('input[name="_csrf"]');
        if (!existingToken) {
            const tokenInput = document.createElement('input');
            tokenInput.type = 'hidden';
            tokenInput.name = '_csrf';
            tokenInput.value = this.token;
            form.appendChild(tokenInput);
        }
    }
    
    // Инициализация для всех форм на странице
    initForAllForms() {
        const forms = document.querySelectorAll('form[method="post"], form[method="POST"]');
        forms.forEach(form => this.addTokenToForm(form));
    }
}

// Инициализация
const csrfManager = new CSRFManager();
csrfManager.initForAllForms();
```

### 2. Double Submit Cookie

```javascript
// Реализация Double Submit Cookie
class DoubleSubmitCookieCSRF {
    constructor() {
        this.cookieName = 'csrf-token';
    }
    
    // Генерация и установка CSRF токена в куки
    setToken() {
        if (!this.getToken()) {
            const token = this.generateToken();
            this.setCookie(this.cookieName, token, { 
                path: '/', 
                httpOnly: false, // Должен быть доступен из JavaScript
                secure: window.location.protocol === 'https:',
                sameSite: 'Strict'
            });
        }
    }
    
    // Проверка токена (токен в заголовке должен совпадать с токеном в куки)
    validateToken(token) {
        const cookieToken = this.getToken();
        return token && cookieToken && token === cookieToken;
    }
    
    generateToken() {
        return Array.from(crypto.getRandomValues(new Uint8Array(32)))
            .map(b => b.toString(16).padStart(2, '0'))
            .join('');
    }
    
    getToken() {
        return document.cookie
            .split(';')
            .find(c => c.trim().startsWith(this.cookieName + '='))
            ?.split('=')[1];
    }
    
    setCookie(name, value, options = {}) {
        let cookieString = `${name}=${value}`;
        
        if (options.expires) {
            cookieString += `; expires=${options.expires.toUTCString()}`;
        }
        
        if (options.maxAge) {
            cookieString += `; max-age=${options.maxAge}`;
        }
        
        if (options.domain) {
            cookieString += `; domain=${options.domain}`;
        }
        
        if (options.path) {
            cookieString += `; path=${options.path}`;
        }
        
        if (options.secure) {
            cookieString += '; secure';
        }
        
        if (options.httpOnly) {
            cookieString += '; httpOnly';
        }
        
        if (options.sameSite) {
            cookieString += `; samesite=${options.sameSite}`;
        }
        
        document.cookie = cookieString;
    }
}

// Использование
const csrf = new DoubleSubmitCookieCSRF();
csrf.setToken();

// При отправке запроса
fetch('/api/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrf.getToken()
    },
    body: JSON.stringify({ data: 'example' })
});
```

### 3. SameSite атрибут куки

```javascript
// Установка куки с SameSite атрибутом
function setSecureCookie(name, value, options = {}) {
    const cookieOptions = {
        path: '/',
        secure: window.location.protocol === 'https:',
        sameSite: 'Strict', // или 'Lax' в зависимости от требований
        ...options
    };
    
    let cookieString = `${name}=${encodeURIComponent(value)}`;
    
    if (cookieOptions.expires) {
        cookieString += `; expires=${cookieOptions.expires.toUTCString()}`;
    }
    
    if (cookieOptions.maxAge) {
        cookieString += `; max-age=${cookieOptions.maxAge}`;
    }
    
    if (cookieOptions.domain) {
        cookieString += `; domain=${cookieOptions.domain}`;
    }
    
    if (cookieOptions.path) {
        cookieString += `; path=${cookieOptions.path}`;
    }
    
    if (cookieOptions.secure) {
        cookieString += '; secure';
    }
    
    if (cookieOptions.httpOnly) {
        cookieString += '; httpOnly';
    }
    
    if (cookieOptions.sameSite) {
        cookieString += `; samesite=${cookieOptions.sameSite}`;
    }
    
    document.cookie = cookieString;
}

// SameSite значения:
// 'Strict' - куки отправляются только в рамках того же сайта
// 'Lax' - куки отправляются при навигации с других сайтов (например, по ссылке)
// 'None' - куки отправляются всегда (требует secure флаг)
```

## Современные подходы защиты

### 1. Synchronizer Token Pattern

```javascript
// Реализация Synchronizer Token Pattern
class SynchronizerTokenManager {
    constructor() {
        this.tokenEndpoint = '/api/csrf-token';
        this.currentToken = null;
    }
    
    async initialize() {
        try {
            const response = await fetch(this.tokenEndpoint);
            const data = await response.json();
            this.currentToken = data.token;
            this.storeToken(data.token);
        } catch (error) {
            console.error('Ошибка получения CSRF токена:', error);
        }
    }
    
    storeToken(token) {
        sessionStorage.setItem('csrf-token', token);
    }
    
    getStoredToken() {
        return sessionStorage.getItem('csrf-token');
    }
    
    async getValidToken() {
        let token = this.getStoredToken();
        
        if (!token) {
            await this.initialize();
            token = this.getStoredToken();
        }
        
        return token;
    }
    
    async makeSecureRequest(url, options = {}) {
        const token = await this.getValidToken();
        
        options.headers = {
            'X-CSRF-Token': token,
            'Content-Type': 'application/json',
            ...options.headers
        };
        
        const response = await fetch(url, options);
        
        // Если токен устарел, получаем новый и повторяем запрос
        if (response.status === 419) { // Token expired
            await this.initialize();
            const newToken = await this.getValidToken();
            
            options.headers['X-CSRF-Token'] = newToken;
            return await fetch(url, options);
        }
        
        return response;
    }
}

// Использование
const tokenManager = new SynchronizerTokenManager();
await tokenManager.initialize();

const response = await tokenManager.makeSecureRequest('/api/data', {
    method: 'POST',
    body: JSON.stringify({ data: 'example' })
});
```

### 2. Request Origin Verification

```javascript
// Проверка происхождения запроса
class OriginVerifier {
    constructor(allowedOrigins = []) {
        this.allowedOrigins = allowedOrigins;
    }
    
    isOriginAllowed(origin) {
        if (!origin) return false;
        
        try {
            const originUrl = new URL(origin);
            return this.allowedOrigins.some(allowed => 
                originUrl.origin === allowed || 
                originUrl.hostname === allowed
            );
        } catch (e) {
            return false;
        }
    }
    
    verifyRequest(request) {
        // Проверка Origin заголовка
        const origin = request.headers.get('Origin');
        if (origin && !this.isOriginAllowed(origin)) {
            return false;
        }
        
        // Проверка Referer заголовка
        const referer = request.headers.get('Referer');
        if (referer && !this.isOriginAllowed(referer)) {
            return false;
        }
        
        return true;
    }
}

// Использование на сервере
const originVerifier = new OriginVerifier([
    'https://example.com',
    'https://app.example.com'
]);
```

## Практические примеры защиты

### 1. Защита форм

```javascript
// Класс для защиты форм от CSRF
class SecureForm {
    constructor(formSelector, options = {}) {
        this.form = document.querySelector(formSelector);
        this.csrfToken = options.csrfToken || this.getCSRFToken();
        this.tokenFieldName = options.tokenFieldName || '_csrf';
        
        if (this.form) {
            this.addCSRFToken();
            this.setupSubmissionProtection();
        }
    }
    
    getCSRFToken() {
        // Пытаемся получить токен из meta тега
        const metaToken = document.querySelector('meta[name="csrf-token"]');
        if (metaToken) {
            return metaToken.getAttribute('content');
        }
        
        // Или из куки
        return document.cookie
            .split(';')
            .find(c => c.trim().startsWith('XSRF-TOKEN='))
            ?.split('=')[1];
    }
    
    addCSRFToken() {
        if (!this.csrfToken) {
            console.warn('CSRF токен не найден');
            return;
        }
        
        // Проверяем, есть ли уже токен в форме
        const existingToken = this.form.querySelector(`input[name="${this.tokenFieldName}"]`);
        if (!existingToken) {
            const tokenInput = document.createElement('input');
            tokenInput.type = 'hidden';
            tokenInput.name = this.tokenFieldName;
            tokenInput.value = this.csrfToken;
            this.form.appendChild(tokenInput);
        }
    }
    
    setupSubmissionProtection() {
        this.form.addEventListener('submit', (event) => {
            // Дополнительная проверка перед отправкой
            if (!this.validateForm()) {
                event.preventDefault();
                return false;
            }
        });
    }
    
    validateForm() {
        // Проверка наличия CSRF токена
        const tokenInput = this.form.querySelector(`input[name="${this.tokenFieldName}"]`);
        if (!tokenInput || !tokenInput.value) {
            console.error('CSRF токен отсутствует в форме');
            return false;
        }
        
        return true;
    }
}

// Использование
const secureForm = new SecureForm('#transfer-form', {
    tokenFieldName: '_token'
});
```

### 2. Защита API запросов

```javascript
// Класс для безопасных API запросов
class SecureAPI {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL;
        this.csrfToken = options.csrfToken;
        this.tokenHeader = options.tokenHeader || 'X-CSRF-Token';
        this.retryOnTokenExpiration = options.retryOnTokenExpiration !== false;
    }
    
    async request(endpoint, options = {}) {
        // Получение CSRF токена, если не предоставлен
        if (!this.csrfToken) {
            this.csrfToken = await this.fetchCSRFToken();
        }
        
        const url = `${this.baseURL}${endpoint}`;
        
        const config = {
            headers: {
                [this.tokenHeader]: this.csrfToken,
                'Content-Type': 'application/json',
                ...options.headers
            },
            ...options
        };
        
        // Добавление тела запроса, если есть
        if (options.body && typeof options.body === 'object') {
            config.body = JSON.stringify(options.body);
        }
        
        let response = await fetch(url, config);
        
        // Обработка истечения токена
        if (response.status === 419 && this.retryOnTokenExpiration) {
            // Получаем новый токен
            this.csrfToken = await this.fetchCSRFToken();
            config.headers[this.tokenHeader] = this.csrfToken;
            
            // Повторяем запрос
            response = await fetch(url, config);
        }
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return response.json();
    }
    
    async fetchCSRFToken() {
        try {
            const response = await fetch(`${this.baseURL}/csrf-token`);
            const data = await response.json();
            return data.token;
        } catch (error) {
            console.error('Ошибка получения CSRF токена:', error);
            throw error;
        }
    }
    
    // Удобные методы
    async get(endpoint, params = {}) {
        let url = endpoint;
        if (Object.keys(params).length > 0) {
            url += '?' + new URLSearchParams(params).toString();
        }
        return await this.request(url, { method: 'GET' });
    }
    
    async post(endpoint, data) {
        return await this.request(endpoint, {
            method: 'POST',
            body: data
        });
    }
    
    async put(endpoint, data) {
        return await this.request(endpoint, {
            method: 'PUT',
            body: data
        });
    }
    
    async delete(endpoint) {
        return await this.request(endpoint, {
            method: 'DELETE'
        });
    }
}

// Использование
const api = new SecureAPI('https://api.example.com', {
    tokenHeader: 'X-CSRF-TOKEN'
});

try {
    const result = await api.post('/transfer', {
        toAccount: '12345',
        amount: 100
    });
    console.log('Перевод выполнен:', result);
} catch (error) {
    console.error('Ошибка выполнения перевода:', error);
}
```

### 3. Защита загрузки файлов

```javascript
// Безопасная загрузка файлов с CSRF защитой
class SecureFileUploader {
    constructor(uploadUrl, options = {}) {
        this.uploadUrl = uploadUrl;
        this.csrfToken = options.csrfToken;
        this.tokenFieldName = options.tokenFieldName || '_token';
        this.onProgress = options.onProgress || null;
    }
    
    async uploadFile(file, additionalData = {}) {
        if (!this.csrfToken) {
            this.csrfToken = await this.fetchCSRFToken();
        }
        
        const formData = new FormData();
        
        // Добавление CSRF токена
        formData.append(this.tokenFieldName, this.csrfToken);
        
        // Добавление дополнительных данных
        Object.keys(additionalData).forEach(key => {
            formData.append(key, additionalData[key]);
        });
        
        // Добавление файла
        formData.append('file', file);
        
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            
            // Отслеживание прогресса
            if (this.onProgress && xhr.upload) {
                xhr.upload.onprogress = (event) => {
                    if (event.lengthComputable) {
                        const percentComplete = (event.loaded / event.total) * 100;
                        this.onProgress(percentComplete);
                    }
                };
            }
            
            xhr.onload = () => {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(JSON.parse(xhr.responseText));
                } else {
                    reject(new Error(`HTTP error! status: ${xhr.status}`));
                }
            };
            
            xhr.onerror = () => {
                reject(new Error('Network error'));
            };
            
            xhr.onabort = () => {
                reject(new Error('Upload cancelled'));
            };
            
            xhr.open('POST', this.uploadUrl);
            xhr.send(formData);
        });
    }
    
    async fetchCSRFToken() {
        try {
            const response = await fetch('/api/csrf-token');
            const data = await response.json();
            return data.token;
        } catch (error) {
            console.error('Ошибка получения CSRF токена:', error);
            throw error;
        }
    }
}

// Использование
const uploader = new SecureFileUploader('/api/upload', {
    tokenFieldName: '_token',
    onProgress: (progress) => {
        console.log(`Загрузка: ${progress}%`);
    }
});

const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];

try {
    const result = await uploader.uploadFile(file, {
        description: 'Мой файл'
    });
    console.log('Файл загружен:', result);
} catch (error) {
    console.error('Ошибка загрузки файла:', error);
}
```

## Лучшие практики

### 1. Комплексная защита

```javascript
// Комплексный подход к защите от CSRF
class ComprehensiveCSRFProtection {
    constructor(options = {}) {
        this.tokenManager = new SynchronizerTokenManager();
        this.originVerifier = new OriginVerifier(options.allowedOrigins || []);
        this.doubleSubmitCookie = new DoubleSubmitCookieCSRF();
    }
    
    async initialize() {
        await this.tokenManager.initialize();
        this.doubleSubmitCookie.setToken();
    }
    
    async createSecureRequest(url, options = {}) {
        // Получение всех необходимых токенов
        const csrfToken = await this.tokenManager.getValidToken();
        const cookieToken = this.doubleSubmitCookie.getToken();
        
        // Установка заголовков безопасности
        options.headers = {
            'X-CSRF-Token': csrfToken,
            'X-Requested-With': 'XMLHttpRequest',
            'Sec-Fetch-Site': 'same',
            'Sec-Fetch-Mode': 'cors',
            ...options.headers
        };
        
        // Проверка происхождения (если применимо)
        if (!this.originVerifier.isOriginAllowed(window.location.origin)) {
            throw new Error('Недопустимое происхождение запроса');
        }
        
        return await fetch(url, options);
    }
    
    // Установка защиты для всех исходящих запросов
    setupGlobalProtection() {
        // Перехват fetch запросов
        const originalFetch = window.fetch;
        window.fetch = async (url, options = {}) => {
            // Применяем защиту только к подходящим запросам
            if (this.shouldProtectRequest(url, options)) {
                return await this.createSecureRequest(url, options);
            }
            return await originalFetch(url, options);
        };
    }
    
    shouldProtectRequest(url, options) {
        // Защищаем только POST, PUT, DELETE запросы
        const method = (options.method || 'GET').toUpperCase();
        return ['POST', 'PUT', 'DELETE', 'PATCH'].includes(method);
    }
}

// Инициализация комплексной защиты
const csrfProtection = new ComprehensiveCSRFProtection({
    allowedOrigins: ['https://example.com', 'https://app.example.com']
});
await csrfProtection.initialize();
csrfProtection.setupGlobalProtection();
```

### 2. Мониторинг и логирование

```javascript
// Система мониторинга CSRF атак
class CSRFAttackMonitor {
    constructor() {
        this.attackLog = [];
        this.maxLogSize = 1000;
        this.setupRequestInterceptors();
    }
    
    setupRequestInterceptors() {
        // Перехват fetch запросов
        const originalFetch = window.fetch;
        window.fetch = async (url, options = {}) => {
            const startTime = Date.now();
            let result;
            let error = null;
            
            try {
                result = await originalFetch(url, options);
            } catch (err) {
                error = err;
                throw err;
            } finally {
                this.logRequest(url, options, result, error, Date.now() - startTime);
            }
            
            return result;
        };
    }
    
    logRequest(url, options, response, error, duration) {
        const logEntry = {
            timestamp: new Date().toISOString(),
            url: url,
            method: options.method || 'GET',
            headers: this.sanitizeHeaders(options.headers),
            responseStatus: response ? response.status : null,
            error: error ? error.toString() : null,
            duration: duration,
            suspectedCSRF: this.detectCSRFAttempt(url, options)
        };
        
        this.attackLog.push(logEntry);
        
        // Ограничение размера лога
        if (this.attackLog.length > this.maxLogSize) {
            this.attackLog = this.attackLog.slice(-this.maxLogSize);
        }
        
        // Если подозревается CSRF атака
        if (logEntry.suspectedCSRF) {
            this.handleCSRFAlert(logEntry);
        }
    }
    
    detectCSRFAttempt(url, options) {
        // Проверка отсутствия CSRF токена в подозрительных запросах
        const method = (options.method || 'GET').toUpperCase();
        if (['POST', 'PUT', 'DELETE'].includes(method)) {
            // Проверяем наличие токена в заголовках или теле
            const hasCSRFToken = options.headers && (
                options.headers['X-CSRF-Token'] || 
                options.headers['X-CSRF-TOKEN'] ||
                (options.body && options.body.includes && 
                 (options.body.includes('_csrf') || options.body.includes('csrf_token')))
            );
            
            return !hasCSRFToken;
        }
        
        return false;
    }
    
    sanitizeHeaders(headers) {
        if (!headers) return {};
        
        const sanitized = {};
        for (const [key, value] of Object.entries(headers)) {
            if (!['authorization', 'cookie'].includes(key.toLowerCase())) {
                sanitized[key] = value;
            }
        }
        return sanitized;
    }
    
    handleCSRFAlert(logEntry) {
        console.warn('Подозреваемая CSRF атака:', logEntry);
        
        // Здесь можно добавить отправку алерта на сервер
        this.reportCSRFAttempt(logEntry);
    }
    
    async reportCSRFAttempt(logEntry) {
        try {
            await fetch('/api/security/csrf-alert', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    url: logEntry.url,
                    method: logEntry.method,
                    timestamp: logEntry.timestamp
                })
            });
        } catch (error) {
            console.error('Ошибка отправки алерта CSRF:', error);
        }
    }
    
    getAttackLog() {
        return this.attackLog;
    }
}

// Использование
const csrfMonitor = new CSRFAttackMonitor();
```

## Тестирование CSRF защиты

### 1. Автоматизированные тесты

```javascript
// Тестирование CSRF защиты
class CSRFSecurityTester {
    static async testEndpoint(url, method = 'POST', data = {}) {
        const tests = [
            {
                name: 'Без CSRF токена',
                headers: {},
                expected: 403, // Ожидаем ошибку
                description: 'Запрос без CSRF токена должен быть отклонен'
            },
            {
                name: 'С неправильным CSRF токеном',
                headers: { 'X-CSRF-Token': 'invalid-token' },
                expected: 403,
                description: 'Запрос с неправильным токеном должен быть отклонен'
            },
            {
                name: 'С правильным CSRF токеном',
                headers: { 'X-CSRF-Token': await this.getValidToken() },
                expected: [200, 201, 204], // Успешные статусы
                description: 'Запрос с правильным токеном должен быть разрешен'
            }
        ];
        
        const results = [];
        
        for (const test of tests) {
            try {
                const response = await fetch(url, {
                    method: method,
                    headers: {
                        'Content-Type': 'application/json',
                        ...test.headers
                    },
                    body: method !== 'GET' ? JSON.stringify(data) : undefined
                });
                
                const expectedStatus = Array.isArray(test.expected) 
                    ? test.expected.includes(response.status)
                    : response.status === test.expected;
                
                results.push({
                    test: test.name,
                    description: test.description,
                    status: response.status,
                    passed: expectedStatus,
                    message: expectedStatus 
                        ? 'Тест пройден' 
                        : `Ожидаемый статус ${test.expected}, получен ${response.status}`
                });
            } catch (error) {
                results.push({
                    test: test.name,
                    description: test.description,
                    status: 'ERROR',
                    passed: false,
                    message: error.message
                });
            }
        }
        
        return results;
    }
    
    static async getValidToken() {
        // Получение валидного токена для тестов
        const response = await fetch('/api/csrf-token');
        const data = await response.json();
        return data.token;
    }
    
    static async runAllTests(apiEndpoints) {
        const allResults = {};
        
        for (const [name, config] of Object.entries(apiEndpoints)) {
            console.log(`Тестирование ${name}...`);
            allResults[name] = await this.testEndpoint(
                config.url, 
                config.method, 
                config.data
            );
        }
        
        return allResults;
    }
}

// Использование (только для тестирования!)
/*
const endpoints = {
    transfer: { url: '/api/transfer', method: 'POST', data: { to: '123', amount: 100 } },
    profile: { url: '/api/profile', method: 'PUT', data: { name: 'John' } }
};

const results = await CSRFSecurityTester.runAllTests(endpoints);
console.log('Результаты тестирования CSRF защиты:', results);
*/
```

CSRF-атаки представляют серьезную угрозу безопасности веб-приложений. Эффективная защита требует комплексного подхода, включающего использование CSRF-токенов, правильную настройку куки с атрибутом SameSite, проверку заголовков запросов и мониторинг подозрительной активности.

#javascript #csrf #security #frontend #web-development #security-best-practices