# Fetch API

Fetch API предоставляет современный интерфейс для выполнения сетевых запросов. Он использует промисы и предоставляет более мощный и гибкий функционал по сравнению с XMLHttpRequest.

## Основы Fetch API

### Простой GET запрос

```javascript
// Простой GET запрос
fetch('https://api.example.com/data')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return response.json();
    })
    .then(data => {
        console.log('Данные получены:', data);
    })
    .catch(error => {
        console.error('Ошибка при выполнении запроса:', error);
    });

// Использование async/await
async function fetchData() {
    try {
        const response = await fetch('https://api.example.com/data');
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        console.log('Данные получены:', data);
        return data;
    } catch (error) {
        console.error('Ошибка при выполнении запроса:', error);
        throw error;
    }
}
```

### POST запрос с телом

```javascript
// POST запрос с JSON данными
async function postData(url, data) {
    const response = await fetch(url, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer ' + token
        },
        body: JSON.stringify(data)
    });
    
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
}

// Пример использования
const userData = { name: 'Иван', email: 'ivan@example.com' };
const result = await postData('https://api.example.com/users', userData);
```

### Заголовки и параметры запроса

```javascript
// Сложный запрос с различными заголовками и параметрами
async function complexRequest(url, options = {}) {
    const defaultHeaders = {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    };
    
    const config = {
        method: options.method || 'GET',
        headers: {
            ...defaultHeaders,
            ...options.headers
        }
    };
    
    if (options.body) {
        config.body = typeof options.body === 'object' 
            ? JSON.stringify(options.body)
            : options.body;
    }
    
    // Добавление параметров к URL
    if (options.params) {
        const urlObj = new URL(url);
        Object.keys(options.params).forEach(key => {
            urlObj.searchParams.append(key, options.params[key]);
        });
        url = urlObj.toString();
    }
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response;
}

// Пример использования
const response = await complexRequest('https://api.example.com/search', {
    method: 'POST',
    headers: {
        'Authorization': 'Bearer token123',
        'X-Custom-Header': 'value'
    },
    body: { query: 'javascript', limit: 10 },
    params: { sort: 'date', order: 'desc' }
});
```

## Объект Response

### Свойства Response

```javascript
async function analyzeResponse(url) {
    const response = await fetch(url);
    
    // Основные свойства
    console.log('Статус:', response.status);
    console.log('Статус текст:', response.statusText);
    console.log('URL:', response.url);
    console.log('Ok (200-299):', response.ok);
    
    // Заголовки
    console.log('Content-Type:', response.headers.get('Content-Type'));
    console.log('Все заголовки:');
    for (const [key, value] of response.headers.entries()) {
        console.log(`${key}: ${value}`);
    }
    
    return response;
}
```

### Методы Response

```javascript
async function handleDifferentResponseTypes(response) {
    // JSON
    const jsonData = await response.json();
    
    // Текст
    const textData = await response.text();
    
    // Blob (для файлов)
    const blobData = await response.blob();
    
    // ArrayBuffer (для бинарных данных)
    const arrayBufferData = await response.arrayBuffer();
    
    // FormData
    const formData = await response.formData();
    
    return { jsonData, textData, blobData, arrayBufferData, formData };
}
```

## Типы контента

### Работа с JSON

```javascript
// Универсальная функция для работы с JSON API
class JsonApiClient {
    constructor(baseURL, defaultHeaders = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            ...defaultHeaders
        };
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const config = {
            headers: { ...this.defaultHeaders, ...options.headers },
            ...options
        };
        
        // Добавление тела запроса, если есть
        if (options.body && typeof options.body === 'object') {
            config.body = JSON.stringify(options.body);
        }
        
        const response = await fetch(url, config);
        
        // Проверка статуса
        if (!response.ok) {
            const errorData = await response.json().catch(() => ({}));
            throw new ApiError(response.status, errorData.message || response.statusText);
        }
        
        // Возврат JSON данных
        return await response.json();
    }
    
    async get(endpoint, params = {}) {
        let url = `${this.baseURL}${endpoint}`;
        
        if (Object.keys(params).length > 0) {
            const urlObj = new URL(url);
            Object.keys(params).forEach(key => {
                urlObj.searchParams.append(key, params[key]);
            });
            url = urlObj.toString();
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

// Класс для обработки ошибок API
class ApiError extends Error {
    constructor(status, message) {
        super(message);
        this.status = status;
        this.name = 'ApiError';
    }
}

// Использование
const apiClient = new JsonApiClient('https://api.example.com');
const users = await apiClient.get('/users', { limit: 10, offset: 0 });
const newUser = await apiClient.post('/users', { name: 'Иван', email: 'ivan@example.com' });
```

### Работа с файлами

```javascript
// Загрузка файлов с помощью Fetch API
class FileUploader {
    constructor(uploadUrl) {
        this.uploadUrl = uploadUrl;
    }
    
    async uploadFile(file, onProgress) {
        const formData = new FormData();
        formData.append('file', file);
        
        const response = await fetch(this.uploadUrl, {
            method: 'POST',
            body: formData,
            // Прогресс загрузки можно отслеживать через XMLHttpRequest
            // Fetch API не предоставляет встроенный способ отслеживания прогресса загрузки
        });
        
        if (!response.ok) {
            throw new Error(`Ошибка загрузки: ${response.status}`);
        }
        
        return await response.json();
    }
    
    async uploadWithProgress(file, onProgress) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            
            xhr.upload.addEventListener('progress', (event) => {
                if (event.lengthComputable) {
                    const percentComplete = (event.loaded / event.total) * 100;
                    onProgress(percentComplete);
                }
            });
            
            xhr.addEventListener('load', () => {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(JSON.parse(xhr.responseText));
                } else {
                    reject(new Error(`HTTP error! status: ${xhr.status}`));
                }
            });
            
            xhr.addEventListener('error', () => {
                reject(new Error('Ошибка сети'));
            });
            
            const formData = new FormData();
            formData.append('file', file);
            
            xhr.open('POST', this.uploadUrl);
            xhr.send(formData);
        });
    }
    
    async downloadFile(url, filename) {
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error(`Ошибка загрузки: ${response.status}`);
        }
        
        const blob = await response.blob();
        const downloadUrl = window.URL.createObjectURL(blob);
        
        // Создание ссылки для скачивания
        const link = document.createElement('a');
        link.href = downloadUrl;
        link.download = filename || 'download';
        document.body.appendChild(link);
        link.click();
        
        // Очистка
        document.body.removeChild(link);
        window.URL.revokeObjectURL(downloadUrl);
    }
}
```

## Обработка ошибок

### Стратегии обработки ошибок

```javascript
// Универсальный обработчик ошибок
class FetchErrorHandler {
    static async handleResponse(response) {
        if (!response.ok) {
            let errorMessage = `HTTP error! status: ${response.status}`;
            let errorData = null;
            
            try {
                errorData = await response.json();
                errorMessage = errorData.message || errorMessage;
            } catch (e) {
                // Если не удалось распарсить JSON, используем текст
                try {
                    const text = await response.text();
                    errorMessage = text || errorMessage;
                } catch (e2) {
                    // Если не удалось получить текст, используем статус
                }
            }
            
            throw new FetchError(response.status, errorMessage, errorData);
        }
        
        return response;
    }
    
    static async handleNetworkError(error) {
        if (error instanceof TypeError && error.message.includes('fetch')) {
            throw new NetworkError('Сетевая ошибка: невозможно подключиться к серверу');
        }
        throw error;
    }
}

class FetchError extends Error {
    constructor(status, message, data) {
        super(message);
        this.status = status;
        this.data = data;
        this.name = 'FetchError';
    }
}

class NetworkError extends Error {
    constructor(message) {
        super(message);
        this.name = 'NetworkError';
    }
}

// Использование обработчика ошибок
async function safeFetch(url, options = {}) {
    try {
        const response = await fetch(url, options);
        return await FetchErrorHandler.handleResponse(response);
    } catch (error) {
        await FetchErrorHandler.handleNetworkError(error);
        throw error;
    }
}
```

### Повторные попытки запросов

```javascript
// Класс для выполнения запросов с повторными попытками
class RetryableFetch {
    constructor(maxRetries = 3, baseDelay = 1000) {
        this.maxRetries = maxRetries;
        this.baseDelay = baseDelay;
    }
    
    async fetch(url, options = {}, retryCount = 0) {
        try {
            const response = await fetch(url, options);
            
            // Повторная попытка для определенных статусов
            if (!response.ok && this.shouldRetry(response.status, retryCount)) {
                await this.delay(this.calculateDelay(retryCount));
                return await this.fetch(url, options, retryCount + 1);
            }
            
            return response;
        } catch (error) {
            // Повторная попытка для сетевых ошибок
            if (retryCount < this.maxRetries && this.isNetworkError(error)) {
                await this.delay(this.calculateDelay(retryCount));
                return await this.fetch(url, options, retryCount + 1);
            }
            
            throw error;
        }
    }
    
    shouldRetry(status, retryCount) {
        // Повтор для 5xx ошибок и некоторых 4xx
        return retryCount < this.maxRetries && 
               (status >= 500 || status === 429 || status === 408);
    }
    
    isNetworkError(error) {
        return error instanceof TypeError || 
               error.message.includes('Failed to fetch') ||
               error.message.includes('NetworkError');
    }
    
    calculateDelay(retryCount) {
        // Экспоненциальная задержка: baseDelay * 2^retryCount
        return this.baseDelay * Math.pow(2, retryCount);
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// Использование
const retryableFetch = new RetryableFetch(3, 1000);
const response = await retryableFetch.fetch('https://api.example.com/data');
```

## Продвинутые возможности

### Middleware для Fetch

```javascript
// Система middleware для Fetch запросов
class FetchMiddleware {
    constructor() {
        this.middlewares = [];
    }
    
    use(middleware) {
        this.middlewares.push(middleware);
        return this;
    }
    
    async execute(url, options = {}) {
        // Применение middleware к опциям
        let modifiedOptions = { ...options };
        
        for (const middleware of this.middlewares) {
            modifiedOptions = await middleware(url, modifiedOptions) || modifiedOptions;
        }
        
        // Выполнение запроса
        return await fetch(url, modifiedOptions);
    }
}

// Примеры middleware
const authMiddleware = async (url, options) => {
    const token = localStorage.getItem('authToken');
    if (token) {
        options.headers = {
            ...options.headers,
            'Authorization': `Bearer ${token}`
        };
    }
    return options;
};

const loggingMiddleware = async (url, options) => {
    console.log(`Выполняется ${options.method || 'GET'} запрос к ${url}`);
    return options;
};

const timingMiddleware = async (url, options) => {
    const startTime = performance.now();
    
    // Добавление колбэка для измерения времени
    options.startTime = startTime;
    
    return options;
};

// Использование
const fetchClient = new FetchMiddleware()
    .use(loggingMiddleware)
    .use(authMiddleware)
    .use(timingMiddleware);

const response = await fetchClient.execute('/api/data', { method: 'GET' });
```

### Кэширование запросов

```javascript
// Система кэширования для Fetch API
class FetchCache {
    constructor(ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
        this.cache = new Map();
        this.ttl = ttl;
        this.cleanupInterval = setInterval(() => this.cleanup(), 60 * 1000); // Очистка каждую минуту
    }
    
    async get(url, options = {}) {
        const cacheKey = this.generateCacheKey(url, options);
        const cached = this.cache.get(cacheKey);
        
        if (cached && (Date.now() - cached.timestamp) < this.ttl) {
            console.log('Данные получены из кэша:', url);
            return cached.data;
        }
        
        // Удаление устаревшего кэша
        if (cached) {
            this.cache.delete(cacheKey);
        }
        
        return null;
    }
    
    async set(url, options, data) {
        const cacheKey = this.generateCacheKey(url, options);
        this.cache.set(cacheKey, {
            data,
            timestamp: Date.now()
        });
    }
    
    generateCacheKey(url, options) {
        // Генерация уникального ключа на основе URL и опций
        const keyData = JSON.stringify({ url, options });
        return btoa(keyData).replace(/[+/=]/g, ''); // Простое кодирование
    }
    
    cleanup() {
        const now = Date.now();
        for (const [key, value] of this.cache.entries()) {
            if ((now - value.timestamp) >= this.ttl) {
                this.cache.delete(key);
            }
        }
    }
    
    clear() {
        this.cache.clear();
    }
    
    destroy() {
        if (this.cleanupInterval) {
            clearInterval(this.cleanupInterval);
        }
    }
}

// Класс для кэшированных запросов
class CachedFetch {
    constructor(ttl = 5 * 60 * 1000) {
        this.cache = new FetchCache(ttl);
    }
    
    async fetch(url, options = {}) {
        // Попытка получения из кэша
        const cached = await this.cache.get(url, options);
        if (cached) {
            return cached;
        }
        
        // Выполнение запроса
        const response = await fetch(url, options);
        
        if (response.ok) {
            const data = await response.json();
            await this.cache.set(url, options, data);
            return data;
        }
        
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    async invalidate(url, options = {}) {
        const cacheKey = this.cache.generateCacheKey(url, options);
        this.cache.cache.delete(cacheKey);
    }
    
    clearCache() {
        this.cache.clear();
    }
}

// Использование
const cachedFetch = new CachedFetch(10 * 60 * 1000); // 10 минут
const data = await cachedFetch.fetch('https://api.example.com/data');
```

## Практические примеры

### Клиент API с аутентификацией

```javascript
// Комплексный клиент API с аутентификацией и обработкой токенов
class ApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
        this.token = null;
        this.refreshToken = null;
        this.tokenExpiry = null;
    }
    
    setTokens(accessToken, refreshToken, expiry) {
        this.token = accessToken;
        this.refreshToken = refreshToken;
        this.tokenExpiry = expiry;
    }
    
    async isAuthenticated() {
        if (!this.token) return false;
        if (this.tokenExpiry && new Date() > new Date(this.tokenExpiry)) {
            return await this.refreshAccessToken();
        }
        return true;
    }
    
    async refreshAccessToken() {
        if (!this.refreshToken) return false;
        
        try {
            const response = await fetch(`${this.baseURL}/auth/refresh`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ refreshToken: this.refreshToken })
            });
            
            if (response.ok) {
                const data = await response.json();
                this.setTokens(data.accessToken, data.refreshToken, data.expiry);
                return true;
            } else {
                // Токен обновить не удалось, нужно заново авторизоваться
                this.logout();
                return false;
            }
        } catch (error) {
            console.error('Ошибка обновления токена:', error);
            this.logout();
            return false;
        }
    }
    
    async request(endpoint, options = {}) {
        // Проверка аутентификации
        if (await this.isAuthenticated()) {
            options.headers = {
                ...options.headers,
                'Authorization': `Bearer ${this.token}`
            };
        }
        
        const response = await fetch(`${this.baseURL}${endpoint}`, options);
        
        // Обработка 401 ошибки (неавторизованный доступ)
        if (response.status === 401) {
            if (await this.refreshAccessToken()) {
                // Повторный запрос с новым токеном
                options.headers = {
                    ...options.headers,
                    'Authorization': `Bearer ${this.token}`
                };
                return await fetch(`${this.baseURL}${endpoint}`, options);
            } else {
                // Перенаправление на страницу логина
                window.location.href = '/login';
                throw new Error('Требуется аутентификация');
            }
        }
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return response;
    }
    
    logout() {
        this.token = null;
        this.refreshToken = null;
        this.tokenExpiry = null;
    }
}
```

### Загрузка и обработка большого объема данных

```javascript
// Класс для постраничной загрузки данных
class DataLoader {
    constructor(apiClient) {
        this.apiClient = apiClient;
        this.cache = new Map();
    }
    
    async loadAllData(endpoint, options = {}) {
        const allData = [];
        let page = 1;
        const limit = options.limit || 50;
        
        while (true) {
            try {
                const response = await this.apiClient.request(`${endpoint}?page=${page}&limit=${limit}`);
                const data = await response.json();
                
                if (data.items && data.items.length > 0) {
                    allData.push(...data.items);
                    
                    // Если данных меньше лимита, значит достигли конца
                    if (data.items.length < limit) {
                        break;
                    }
                    
                    page++;
                } else {
                    break;
                }
            } catch (error) {
                console.error(`Ошибка загрузки страницы ${page}:`, error);
                throw error;
            }
        }
        
        return allData;
    }
    
    async loadWithCaching(endpoint, options = {}) {
        const cacheKey = `${endpoint}?${new URLSearchParams(options).toString()}`;
        
        // Проверка кэша
        if (this.cache.has(cacheKey)) {
            const cached = this.cache.get(cacheKey);
            if (Date.now() - cached.timestamp < 5 * 60 * 1000) { // 5 минут
                return cached.data;
            }
        }
        
        // Загрузка данных
        const response = await this.apiClient.request(endpoint, {
            method: 'GET',
            ...options
        });
        
        const data = await response.json();
        
        // Сохранение в кэш
        this.cache.set(cacheKey, {
            data,
            timestamp: Date.now()
        });
        
        return data;
    }
}
```

## Лучшие практики

### 1. Управление ресурсами

```javascript
// Отмена запросов с AbortController
async function cancellableFetch(url, timeout = 5000) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
        const response = await fetch(url, {
            signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        return response;
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log('Запрос отменен по таймауту');
        }
        throw error;
    }
}

// Использование с отменой
let currentRequestController = null;

async function makeRequest(url) {
    // Отмена предыдущего запроса
    if (currentRequestController) {
        currentRequestController.abort();
    }
    
    currentRequestController = new AbortController();
    
    try {
        const response = await fetch(url, {
            signal: currentRequestController.signal
        });
        
        return await response.json();
    } catch (error) {
        if (error.name !== 'AbortError') {
            throw error;
        }
    } finally {
        currentRequestController = null;
    }
}
```

### 2. Обработка больших объемов данных

```javascript
// Потоковая обработка ответа
async function streamResponse(url) {
    const response = await fetch(url);
    
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    let buffer = '';
    
    try {
        while (true) {
            const { done, value } = await reader.read();
            
            if (done) {
                break;
            }
            
            buffer += decoder.decode(value, { stream: true });
            
            // Обработка частей данных по мере получения
            const lines = buffer.split('\n');
            buffer = lines.pop(); // Сохраняем неполные строки
            
            for (const line of lines) {
                if (line.trim()) {
                    try {
                        const data = JSON.parse(line);
                        // Обработка данных
                        console.log('Получена строка данных:', data);
                    } catch (e) {
                        console.error('Ошибка парсинга строки:', line);
                    }
                }
            }
        }
        
        // Обработка оставшихся данных
        if (buffer.trim()) {
            try {
                const data = JSON.parse(buffer);
                console.log('Последняя строка данных:', data);
            } catch (e) {
                console.error('Ошибка парсинга последней строки:', buffer);
            }
        }
    } finally {
        reader.releaseLock();
    }
}
```

Fetch API предоставляет современный и мощный способ выполнения сетевых запросов в JavaScript. При правильном использовании он позволяет создавать надежные и эффективные веб-приложения с хорошей обработкой ошибок и оптимизацией производительности.

#javascript #fetch #api #network #frontend #web-development