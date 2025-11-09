# XMLHttpRequest

XMLHttpRequest (XHR) - это встроенный в браузер объект, который позволяет выполнять HTTP-запросы из JavaScript. Несмотря на наличие более современного Fetch API, XMLHttpRequest все еще широко используется и имеет свои особенности.

## Основы XMLHttpRequest

### Создание и настройка запроса

```javascript
// Создание объекта XMLHttpRequest
const xhr = new XMLHttpRequest();

// Настройка запроса
xhr.open('GET', 'https://api.example.com/data', true); // метод, URL, асинхронный

// Установка заголовков
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('Authorization', 'Bearer ' + token);

// Отправка запроса
xhr.send();

// Обработка ответа
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
        console.log('Ответ получен:', xhr.responseText);
    }
};
```

### Состояния запроса (readyState)

```javascript
// Состояния XMLHttpRequest
const readyStates = {
    0: 'UNSENT',        // объект создан, метод open() еще не вызван
    1: 'OPENED',        // метод open() вызван
    2: 'HEADERS_RECEIVED', // метод send() вызван, получены заголовки
    3: 'LOADING',       // загружается тело ответа
    4: 'DONE'           // запрос завершен
};

const xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    console.log(`Состояние: ${readyStates[xhr.readyState]}`);
    
    if (xhr.readyState === 4) {
        if (xhr.status === 200) {
            console.log('Успешный ответ:', xhr.responseText);
        } else {
            console.error(`Ошибка: ${xhr.status} ${xhr.statusText}`);
        }
    }
};
```

## Основные методы и свойства

### Методы

```javascript
const xhr = new XMLHttpRequest();

// Открытие соединения
xhr.open('GET', 'https://api.example.com/data');

// Установка заголовков
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.setRequestHeader('X-Custom-Header', 'value');

// Отправка запроса (с необязательным телом)
xhr.send(null); // для GET запросов
xhr.send(JSON.stringify(data)); // для POST запросов

// Отмена запроса
xhr.abort();
```

### Свойства

```javascript
const xhr = new XMLHttpRequest();

xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        // Основные свойства ответа
        console.log('Статус:', xhr.status);
        console.log('Статус текст:', xhr.statusText);
        console.log('Ответ (текст):', xhr.responseText);
        console.log('Ответ (XML):', xhr.responseXML);
        console.log('Готовность:', xhr.readyState);
        
        // Заголовки ответа
        console.log('Все заголовки:', xhr.getAllResponseHeaders());
        console.log('Конкретный заголовок:', xhr.getResponseHeader('Content-Type'));
    }
};
```

## Типы ответа

### Управление типом ответа

```javascript
const xhr = new XMLHttpRequest();

// Текстовый ответ (по умолчанию)
xhr.responseType = 'text';
xhr.onload = function() {
    console.log('Текст:', xhr.response);
};

// JSON ответ (требует ручного парсинга)
xhr.responseType = 'text'; // responseType не поддерживает 'json'
xhr.onload = function() {
    const data = JSON.parse(xhr.response);
    console.log('JSON:', data);
};

// ArrayBuffer для бинарных данных
xhr.responseType = 'arraybuffer';
xhr.onload = function() {
    console.log('ArrayBuffer:', xhr.response);
};

// Blob для файлов
xhr.responseType = 'blob';
xhr.onload = function() {
    console.log('Blob:', xhr.response);
};

// Документ (XML или HTML)
xhr.responseType = 'document';
xhr.onload = function() {
    console.log('Документ:', xhr.response);
};
```

## Обработка событий

### Современные события

```javascript
const xhr = new XMLHttpRequest();

// Событие загрузки завершена
xhr.onload = function() {
    if (xhr.status >= 200 && xhr.status < 300) {
        console.log('Успешный ответ:', xhr.response);
    } else {
        console.error('Ошибка:', xhr.status, xhr.statusText);
    }
};

// Событие ошибки сети
xhr.onerror = function() {
    console.error('Сетевая ошибка');
};

// Событие отмены запроса
xhr.onabort = function() {
    console.log('Запрос отменен');
};

// Событие таймаута
xhr.ontimeout = function() {
    console.error('Таймаут запроса');
};

// Прогресс загрузки
xhr.upload.onprogress = function(event) {
    if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        console.log(`Загрузка: ${percentComplete.toFixed(2)}%`);
    }
};

// Прогресс скачивания
xhr.onprogress = function(event) {
    if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        console.log(`Скачивание: ${percentComplete.toFixed(2)}%`);
    }
};

xhr.open('POST', 'https://api.example.com/upload');
xhr.send(formData);
```

## Класс для работы с XMLHttpRequest

```javascript
class XhrClient {
    constructor(baseURL = '', defaultHeaders = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = defaultHeaders;
    }
    
    request(url, options = {}) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            
            // Полный URL
            const fullUrl = this.baseURL + url;
            
            // Метод и асинхронность
            const method = options.method || 'GET';
            const async = options.async !== false; // по умолчанию true
            
            // Открытие запроса
            xhr.open(method, fullUrl, async);
            
            // Установка таймаута
            if (options.timeout) {
                xhr.timeout = options.timeout;
            }
            
            // Установка заголовков
            const headers = { ...this.defaultHeaders, ...options.headers };
            for (const [key, value] of Object.entries(headers)) {
                xhr.setRequestHeader(key, value);
            }
            
            // Тип ответа
            if (options.responseType) {
                xhr.responseType = options.responseType;
            }
            
            // Обработка успешного завершения
            xhr.onload = () => {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(this.handleResponse(xhr));
                } else {
                    reject(new XhrError(xhr.status, xhr.statusText, xhr.response));
                }
            };
            
            // Обработка ошибок
            xhr.onerror = () => {
                reject(new XhrError(0, 'Network error', null));
            };
            
            xhr.ontimeout = () => {
                reject(new XhrError(0, 'Request timeout', null));
            };
            
            xhr.onabort = () => {
                reject(new XhrError(0, 'Request aborted', null));
            };
            
            // Прогресс загрузки
            if (options.onUploadProgress && xhr.upload) {
                xhr.upload.onprogress = options.onUploadProgress;
            }
            
            // Прогресс скачивания
            if (options.onDownloadProgress) {
                xhr.onprogress = options.onDownloadProgress;
            }
            
            // Отправка запроса
            const body = this.prepareBody(options.body, headers['Content-Type']);
            xhr.send(body);
        });
    }
    
    handleResponse(xhr) {
        // Автоматический парсинг JSON
        const contentType = xhr.getResponseHeader('Content-Type');
        
        if (contentType && contentType.includes('application/json')) {
            try {
                return JSON.parse(xhr.responseText);
            } catch (e) {
                console.warn('Не удалось распарсить JSON:', e);
                return xhr.responseText;
            }
        }
        
        // Возвращаем в зависимости от типа ответа
        if (xhr.responseType === 'json') {
            return xhr.response;
        }
        
        return xhr.responseText;
    }
    
    prepareBody(body, contentType) {
        if (!body) return null;
        
        if (typeof body === 'string') {
            return body;
        }
        
        if (body instanceof FormData) {
            return body;
        }
        
        if (contentType && contentType.includes('application/json')) {
            return JSON.stringify(body);
        }
        
        return body;
    }
    
    // Удобные методы
    get(url, options = {}) {
        return this.request(url, { ...options, method: 'GET' });
    }
    
    post(url, data, options = {}) {
        return this.request(url, { ...options, method: 'POST', body: data });
    }
    
    put(url, data, options = {}) {
        return this.request(url, { ...options, method: 'PUT', body: data });
    }
    
    delete(url, options = {}) {
        return this.request(url, { ...options, method: 'DELETE' });
    }
}

// Класс для обработки ошибок
class XhrError extends Error {
    constructor(status, statusText, response) {
        super(`${status} ${statusText}`);
        this.status = status;
        this.statusText = statusText;
        this.response = response;
        this.name = 'XhrError';
    }
}

// Использование
const client = new XhrClient('https://api.example.com');

// Простой GET запрос
try {
    const data = await client.get('/users');
    console.log('Пользователи:', data);
} catch (error) {
    console.error('Ошибка запроса:', error);
}

// POST запрос с данными
const newUser = await client.post('/users', {
    name: 'Иван',
    email: 'ivan@example.com'
});
```

## Работа с файлами

### Загрузка файлов

```javascript
class FileUploadManager {
    constructor() {
        this.uploads = new Map();
    }
    
    uploadFile(url, file, onProgress) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            const uploadId = this.generateId();
            
            // Сохраняем xhr для возможности отмены
            this.uploads.set(uploadId, xhr);
            
            // Прогресс загрузки
            xhr.upload.onprogress = (event) => {
                if (event.lengthComputable && onProgress) {
                    const percentComplete = (event.loaded / event.total) * 100;
                    onProgress(percentComplete, uploadId);
                }
            };
            
            xhr.onload = () => {
                this.uploads.delete(uploadId);
                
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(JSON.parse(xhr.responseText));
                } else {
                    reject(new Error(`HTTP error! status: ${xhr.status}`));
                }
            };
            
            xhr.onerror = () => {
                this.uploads.delete(uploadId);
                reject(new Error('Network error'));
            };
            
            xhr.onabort = () => {
                this.uploads.delete(uploadId);
                reject(new Error('Upload cancelled'));
            };
            
            // Подготовка данных
            const formData = new FormData();
            formData.append('file', file);
            
            xhr.open('POST', url);
            xhr.send(formData);
            
            // Возвращаем ID для отмены
            return uploadId;
        });
    }
    
    cancelUpload(uploadId) {
        const xhr = this.uploads.get(uploadId);
        if (xhr) {
            xhr.abort();
            this.uploads.delete(uploadId);
        }
    }
    
    generateId() {
        return Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
}

// Использование
const uploadManager = new FileUploadManager();

const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];

const uploadId = await uploadManager.uploadFile('/api/upload', file, (progress) => {
    console.log(`Загрузка: ${progress}%`);
});
```

### Скачивание файлов

```javascript
class FileDownloadManager {
    async downloadFile(url, filename) {
        const xhr = new XMLHttpRequest();
        
        return new Promise((resolve, reject) => {
            xhr.open('GET', url);
            xhr.responseType = 'blob';
            
            xhr.onload = () => {
                if (xhr.status === 200) {
                    const blob = xhr.response;
                    this.saveBlobAsFile(blob, filename);
                    resolve(blob);
                } else {
                    reject(new Error(`HTTP error! status: ${xhr.status}`));
                }
            };
            
            xhr.onerror = () => {
                reject(new Error('Network error'));
            };
            
            xhr.send();
        });
    }
    
    saveBlobAsFile(blob, filename) {
        const url = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        
        link.href = url;
        link.download = filename || 'download';
        
        document.body.appendChild(link);
        link.click();
        
        // Очистка
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);
    }
    
    async downloadWithProgress(url, filename, onProgress) {
        const xhr = new XMLHttpRequest();
        
        return new Promise((resolve, reject) => {
            xhr.open('GET', url);
            xhr.responseType = 'blob';
            
            xhr.onprogress = (event) => {
                if (event.lengthComputable && onProgress) {
                    const percentComplete = (event.loaded / event.total) * 100;
                    onProgress(percentComplete);
                }
            };
            
            xhr.onload = () => {
                if (xhr.status === 200) {
                    const blob = xhr.response;
                    this.saveBlobAsFile(blob, filename);
                    resolve(blob);
                } else {
                    reject(new Error(`HTTP error! status: ${xhr.status}`));
                }
            };
            
            xhr.onerror = () => {
                reject(new Error('Network error'));
            };
            
            xhr.send();
        });
    }
}
```

## Продвинутые возможности

### Аутентификация

```javascript
class AuthenticatedXhrClient extends XhrClient {
    constructor(baseURL, authProvider) {
        super(baseURL);
        this.authProvider = authProvider;
    }
    
    async request(url, options = {}) {
        // Получение токена аутентификации
        const token = await this.authProvider.getToken();
        
        // Добавление заголовка аутентификации
        options.headers = {
            ...options.headers,
            'Authorization': `Bearer ${token}`
        };
        
        try {
            return await super.request(url, options);
        } catch (error) {
            // Обработка 401 ошибки (неавторизованный доступ)
            if (error.status === 401) {
                // Попытка обновить токен
                const refreshTokenResult = await this.authProvider.refreshToken();
                
                if (refreshTokenResult) {
                    // Повторный запрос с новым токеном
                    options.headers['Authorization'] = `Bearer ${refreshTokenResult}`;
                    return await super.request(url, options);
                } else {
                    // Перенаправление на страницу логина
                    window.location.href = '/login';
                    throw new Error('Требуется аутентификация');
                }
            }
            
            throw error;
        }
    }
}

// Пример провайдера аутентификации
class AuthProvider {
    constructor() {
        this.token = localStorage.getItem('authToken');
        this.refreshToken = localStorage.getItem('refreshToken');
    }
    
    async getToken() {
        if (!this.token) {
            throw new Error('Пользователь не аутентифицирован');
        }
        
        // Проверка срока действия токена (упрощенная)
        const tokenExpiry = localStorage.getItem('tokenExpiry');
        if (tokenExpiry && new Date() > new Date(tokenExpiry)) {
            await this.refreshToken();
        }
        
        return this.token;
    }
    
    async refreshToken() {
        if (!this.refreshToken) {
            return false;
        }
        
        try {
            const response = await fetch('/auth/refresh', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ refreshToken: this.refreshToken })
            });
            
            if (response.ok) {
                const data = await response.json();
                this.token = data.accessToken;
                this.refreshToken = data.refreshToken;
                
                // Сохранение токенов
                localStorage.setItem('authToken', this.token);
                localStorage.setItem('refreshToken', this.refreshToken);
                localStorage.setItem('tokenExpiry', data.expiry);
                
                return this.token;
            }
        } catch (error) {
            console.error('Ошибка обновления токена:', error);
        }
        
        return false;
    }
}
```

### Кэширование запросов

```javascript
class CachedXhrClient extends XhrClient {
    constructor(baseURL, defaultHeaders = {}) {
        super(baseURL, defaultHeaders);
        this.cache = new Map();
        this.cacheTimeout = 5 * 60 * 1000; // 5 минут
    }
    
    async request(url, options = {}) {
        // Только для GET запросов
        if (options.method && options.method !== 'GET') {
            return super.request(url, options);
        }
        
        const cacheKey = this.generateCacheKey(url, options);
        const cached = this.cache.get(cacheKey);
        
        // Проверка кэша
        if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
            console.log('Ответ из кэша:', url);
            return cached.data;
        }
        
        // Выполнение запроса
        const data = await super.request(url, options);
        
        // Сохранение в кэш
        this.cache.set(cacheKey, {
            data,
            timestamp: Date.now()
        });
        
        return data;
    }
    
    generateCacheKey(url, options) {
        // Уникальный ключ на основе URL и параметров
        return `${url}?${JSON.stringify(options)}`;
    }
    
    invalidateCache(url) {
        for (const key of this.cache.keys()) {
            if (key.startsWith(url)) {
                this.cache.delete(key);
            }
        }
    }
    
    clearCache() {
        this.cache.clear();
    }
}
```

## Практические примеры

### Клиент REST API

```javascript
class RestApiClient {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json',
            ...options.defaultHeaders
        };
        this.timeout = options.timeout || 10000;
        this.retryAttempts = options.retryAttempts || 3;
    }
    
    async request(endpoint, method = 'GET', data = null, headers = {}) {
        const url = this.baseURL + endpoint;
        const config = {
            method,
            headers: { ...this.defaultHeaders, ...headers },
            timeout: this.timeout
        };
        
        if (data && method !== 'GET' && method !== 'HEAD') {
            config.body = typeof data === 'string' ? data : JSON.stringify(data);
        }
        
        let attempts = 0;
        let lastError;
        
        while (attempts < this.retryAttempts) {
            try {
                const xhr = new XMLHttpRequest();
                
                return await new Promise((resolve, reject) => {
                    xhr.open(config.method, url, true);
                    xhr.timeout = config.timeout;
                    
                    // Установка заголовков
                    for (const [key, value] of Object.entries(config.headers)) {
                        xhr.setRequestHeader(key, value);
                    }
                    
                    xhr.onload = () => {
                        if (xhr.status >= 200 && xhr.status < 300) {
                            let response = xhr.responseText;
                            
                            // Попытка парсинга JSON
                            if (xhr.getResponseHeader('Content-Type')?.includes('application/json')) {
                                try {
                                    response = JSON.parse(response);
                                } catch (e) {
                                    console.warn('Не удалось распарсить JSON:', e);
                                }
                            }
                            
                            resolve(response);
                        } else {
                            reject(new Error(`HTTP error! status: ${xhr.status}`));
                        }
                    };
                    
                    xhr.onerror = () => reject(new Error('Network error'));
                    xhr.ontimeout = () => reject(new Error('Request timeout'));
                    xhr.onabort = () => reject(new Error('Request aborted'));
                    
                    xhr.send(config.body || null);
                });
            } catch (error) {
                lastError = error;
                attempts++;
                
                if (attempts < this.retryAttempts) {
                    // Экспоненциальная задержка
                    await this.delay(Math.pow(2, attempts) * 1000);
                }
            }
        }
        
        throw lastError;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
    
    // Удобные методы
    get(endpoint, params = null) {
        if (params) {
            const queryString = new URLSearchParams(params).toString();
            endpoint = `${endpoint}?${queryString}`;
        }
        return this.request(endpoint, 'GET');
    }
    
    post(endpoint, data) {
        return this.request(endpoint, 'POST', data);
    }
    
    put(endpoint, data) {
        return this.request(endpoint, 'PUT', data);
    }
    
    patch(endpoint, data) {
        return this.request(endpoint, 'PATCH', data);
    }
    
    delete(endpoint) {
        return this.request(endpoint, 'DELETE');
    }
}

// Использование
const api = new RestApiClient('https://api.example.com');

// Получение данных
const users = await api.get('/users', { limit: 10, offset: 0 });

// Создание нового пользователя
const newUser = await api.post('/users', {
    name: 'Иван',
    email: 'ivan@example.com'
});

// Обновление пользователя
const updatedUser = await api.put('/users/123', {
    name: 'Иван Петров',
    email: 'ivan.petrov@example.com'
});
```

### Работа с WebSocket через XHR (для совместимости)

```javascript
// Класс для эмуляции WebSocket через XHR (Long Polling)
class XhrWebSocketEmulator {
    constructor(url, protocols = []) {
        this.url = url;
        this.protocols = protocols;
        this.readyState = 0; // CONNECTING
        this.onopen = null;
        this.onmessage = null;
        this.onerror = null;
        this.onclose = null;
        this.isClosed = false;
        this.pollingInterval = 1000;
        
        this.connect();
    }
    
    connect() {
        this.readyState = 0; // CONNECTING
        this.poll();
        
        // Вызов события open
        if (this.onopen) {
            setTimeout(this.onopen, 0);
        }
    }
    
    poll() {
        if (this.isClosed) return;
        
        const xhr = new XMLHttpRequest();
        
        xhr.open('GET', this.url, true);
        xhr.timeout = this.pollingInterval * 2;
        
        xhr.onload = () => {
            if (xhr.status === 200 && !this.isClosed) {
                if (this.onmessage) {
                    this.onmessage({ data: xhr.responseText });
                }
                
                // Продолжаем опрос
                setTimeout(() => this.poll(), this.pollingInterval);
            } else if (!this.isClosed) {
                this.handleError();
            }
        };
        
        xhr.onerror = () => {
            if (!this.isClosed) {
                this.handleError();
            }
        };
        
        xhr.ontimeout = () => {
            if (!this.isClosed) {
                // Продолжаем опрос при таймауте
                setTimeout(() => this.poll(), this.pollingInterval);
            }
        };
        
        xhr.send();
    }
    
    handleError() {
        this.readyState = 3; // CLOSED
        if (this.onerror) {
            this.onerror(new Error('Connection error'));
        }
        if (this.onclose) {
            this.onclose({ code: 1006, reason: 'Connection failed', wasClean: false });
        }
    }
    
    send(data) {
        if (this.readyState !== 1) { // OPEN
            throw new Error('WebSocket is not open');
        }
        
        const xhr = new XMLHttpRequest();
        xhr.open('POST', this.url, true);
        xhr.setRequestHeader('Content-Type', 'text/plain');
        xhr.send(data);
    }
    
    close() {
        this.isClosed = true;
        this.readyState = 3; // CLOSED
        if (this.onclose) {
            this.onclose({ code: 1000, reason: 'Closed by client', wasClean: true });
        }
    }
}
```

## Совместимость и полифилы

### Полифил для старых браузеров

```javascript
// Полифил для XMLHttpRequest в старых IE
(function(global) {
    if (global.XMLHttpRequest) return; // Уже поддерживается
    
    global.XMLHttpRequest = function() {
        // Попытка создать объект в зависимости от версии IE
        const progIds = [
            'Msxml2.XMLHTTP.6.0',
            'Msxml2.XMLHTTP.3.0',
            'Microsoft.XMLHTTP'
        ];
        
        let xhr;
        
        for (const progId of progIds) {
            try {
                xhr = new global.ActiveXObject(progId);
                break;
            } catch (e) {
                // Пробуем следующий
            }
        }
        
        if (!xhr) {
            throw new Error('XMLHttpRequest не поддерживается в этом браузере');
        }
        
        return xhr;
    };
})(window);
```

## Лучшие практики

### 1. Управление ошибками

```javascript
// Централизованная обработка ошибок
class XhrErrorHandler {
    static handle(xhr, context = '') {
        let error;
        
        if (xhr.status === 0) {
            error = new Error(`${context}: Network error or CORS issue`);
        } else if (xhr.status >= 400) {
            error = new Error(`${context}: HTTP error ${xhr.status} - ${xhr.statusText}`);
        } else {
            error = new Error(`${context}: Unknown error`);
        }
        
        error.status = xhr.status;
        error.statusText = xhr.statusText;
        
        // Логирование ошибки
        console.error('XHR Error:', error);
        
        // Отправка ошибки на сервер для анализа
        this.logError(error, context, xhr);
        
        throw error;
    }
    
    static logError(error, context, xhr) {
        // Отправка данных об ошибке на сервер
        if ('sendBeacon' in navigator) {
            navigator.sendBeacon('/api/errors', JSON.stringify({
                error: error.toString(),
                context: context,
                status: xhr.status,
                statusText: xhr.statusText,
                url: xhr.responseURL,
                timestamp: new Date().toISOString()
            }));
        }
    }
}
```

### 2. Таймауты и отмена

```javascript
// Управление таймаутами и отменой запросов
class XhrManager {
    constructor() {
        this.requests = new Map();
    }
    
    createRequest(id, xhr) {
        this.requests.set(id, xhr);
        return id;
    }
    
    cancelRequest(id) {
        const xhr = this.requests.get(id);
        if (xhr) {
            xhr.abort();
            this.requests.delete(id);
        }
    }
    
    cancelAll() {
        for (const [id, xhr] of this.requests) {
            xhr.abort();
        }
        this.requests.clear();
    }
    
    // Автоматическая очистка завершенных запросов
    cleanup() {
        for (const [id, xhr] of this.requests) {
            if (xhr.readyState === 4) { // DONE
                this.requests.delete(id);
            }
        }
    }
}

// Использование
const xhrManager = new XhrManager();

const xhr = new XMLHttpRequest();
xhrManager.createRequest('request-1', xhr);

xhr.onload = function() {
    xhrManager.cleanup(); // Очистка завершенных запросов
    console.log('Ответ:', xhr.responseText);
};

xhr.open('GET', '/api/data');
xhr.send();
```

XMLHttpRequest остается важной частью веб-разработки, особенно при необходимости поддержки старых браузеров. Несмотря на появление более современных API, понимание XHR важно для полного понимания сетевых взаимодействий в веб-приложениях.

#javascript #xmlhttprequest #xhr #network #frontend #web-development