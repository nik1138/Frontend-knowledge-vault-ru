---
aliases: ["Примеры с Fetch API и WebSocket", "Сетевые запросы в JavaScript"]
tags: [js, practical-examples, fetch, websocket, frontend]
---

# Практические примеры с Fetch API и WebSocket

Практические примеры работы с сетевыми протоколами в JavaScript: Fetch API для HTTP-запросов и WebSocket для двусторонней связи в реальном времени.

## 1. Основы Fetch API

```javascript
// Простой GET запрос
async function simpleGetRequest(url) {
    try {
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка при выполнении запроса:', error);
        throw error;
    }
}

// POST запрос с JSON
async function postJsonData(url, data) {
    try {
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Ошибка при отправке данных:', error);
        throw error;
    }
}

// PUT запрос
async function updateResource(url, data) {
    try {
        const response = await fetch(url, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(data)
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Ошибка при обновлении ресурса:', error);
        throw error;
    }
}

// DELETE запрос
async function deleteResource(url) {
    try {
        const response = await fetch(url, {
            method: 'DELETE'
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return response.status === 204; // No Content
    } catch (error) {
        console.error('Ошибка при удалении ресурса:', error);
        throw error;
    }
}

// Пример использования
// simpleGetRequest('https://jsonplaceholder.typicode.com/posts/1')
//     .then(data => console.log(data))
//     .catch(error => console.error(error));
```

## 2. Продвинутые примеры Fetch API

### Управление заголовками и аутентификацией

```javascript
class ApiClient {
    constructor(baseURL, defaultHeaders = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json',
            ...defaultHeaders
        };
        this.token = null;
    }
    
    // Установка токена аутентификации
    setToken(token) {
        this.token = token;
        if (token) {
            this.defaultHeaders['Authorization'] = `Bearer ${token}`;
        } else {
            delete this.defaultHeaders['Authorization'];
        }
    }
    
    // Универсальный метод для выполнения запросов
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        
        const config = {
            headers: {
                ...this.defaultHeaders,
                ...options.headers
            },
            ...options
        };
        
        // Добавляем токен, если он есть
        if (this.token && !config.headers['Authorization']) {
            config.headers['Authorization'] = `Bearer ${this.token}`;
        }
        
        try {
            const response = await fetch(url, config);
            
            // Обработка специфичных статусов
            if (response.status === 401) {
                // Токен истек, можно вызвать коллбэк для перенаправления на логин
                this.handleUnauthorized();
                throw new Error('Требуется аутентификация');
            }
            
            if (response.status === 403) {
                throw new Error('Недостаточно прав для выполнения операции');
            }
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            // Определяем, есть ли тело ответа
            const contentType = response.headers.get('content-type');
            if (contentType && contentType.includes('application/json')) {
                return await response.json();
            } else if (contentType && contentType.includes('text/')) {
                return await response.text();
            } else {
                return response;
            }
        } catch (error) {
            console.error(`Ошибка запроса к ${url}:`, error);
            throw error;
        }
    }
    
    // Обработка неавторизованного доступа
    handleUnauthorized() {
        console.log('Требуется аутентификация');
        // Здесь можно выполнить выход из системы или перенаправление
    }
    
    // Методы для разных HTTP-методов
    async get(endpoint, params = {}) {
        const queryString = new URLSearchParams(params).toString();
        const url = queryString ? `${endpoint}?${queryString}` : endpoint;
        
        return this.request(url, { method: 'GET' });
    }
    
    async post(endpoint, data, options = {}) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data),
            ...options
        });
    }
    
    async put(endpoint, data, options = {}) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data),
            ...options
        });
    }
    
    async delete(endpoint, options = {}) {
        return this.request(endpoint, {
            method: 'DELETE',
            ...options
        });
    }
}

// Пример использования
const apiClient = new ApiClient('https://api.example.com');

// apiClient.setToken('your-jwt-token');
// 
// apiClient.get('/users/123')
//     .then(user => console.log(user))
//     .catch(error => console.error(error));
```

### Загрузка файлов

```javascript
// Загрузка файла с прогрессом
async function uploadFileWithProgress(file, url, onProgress) {
    const formData = new FormData();
    formData.append('file', file);
    
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        // Отслеживание прогресса загрузки
        xhr.upload.addEventListener('progress', (event) => {
            if (event.lengthComputable) {
                const percentComplete = (event.loaded / event.total) * 100;
                onProgress && onProgress(percentComplete);
            }
        });
        
        xhr.addEventListener('load', () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(`Upload failed with status ${xhr.status}`));
            }
        });
        
        xhr.addEventListener('error', () => {
            reject(new Error('Network error occurred'));
        });
        
        xhr.open('POST', url);
        xhr.send(formData);
    });
}

// Загрузка файла с использованием fetch (без прогресса)
async function uploadFile(file, url) {
    const formData = new FormData();
    formData.append('file', file);
    
    try {
        const response = await fetch(url, {
            method: 'POST',
            body: formData
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Ошибка при загрузке файла:', error);
        throw error;
    }
}

// Загрузка нескольких файлов
async function uploadMultipleFiles(files, url, onProgress) {
    const formData = new FormData();
    
    // Добавляем все файлы в FormData
    Array.from(files).forEach((file, index) => {
        formData.append(`files[${index}]`, file);
    });
    
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        let totalFiles = files.length;
        let completedFiles = 0;
        
        xhr.upload.addEventListener('progress', (event) => {
            if (event.lengthComputable) {
                // Рассчитываем общий прогресс для всех файлов
                const singleFileProgress = (event.loaded / event.total) * 100;
                const totalProgress = ((completedFiles + (singleFileProgress / 100)) / totalFiles) * 100;
                onProgress && onProgress(totalProgress);
            }
        });
        
        xhr.addEventListener('load', () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                completedFiles++;
                if (completedFiles === totalFiles) {
                    resolve(JSON.parse(xhr.response));
                }
            } else {
                reject(new Error(`Upload failed with status ${xhr.status}`));
            }
        });
        
        xhr.addEventListener('error', () => {
            reject(new Error('Network error occurred'));
        });
        
        xhr.open('POST', url);
        xhr.send(formData);
    });
}
```

### Кэширование запросов

```javascript
class CachedApiClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
        this.cache = new Map();
        this.cacheTimeout = 5 * 60 * 1000; // 5 минут
    }
    
    async getWithCache(endpoint, options = {}) {
        const cacheKey = `${endpoint}?${JSON.stringify(options)}`;
        const cached = this.cache.get(cacheKey);
        
        // Проверяем, есть ли кэшированный результат и не истекло ли его время
        if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
            console.log(`Возвращаем данные из кэша для: ${endpoint}`);
            return cached.data;
        }
        
        // Выполняем запрос
        try {
            const response = await fetch(`${this.baseURL}${endpoint}`, {
                method: 'GET',
                ...options
            });
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            const data = await response.json();
            
            // Кэшируем результат
            this.cache.set(cacheKey, {
                data: data,
                timestamp: Date.now()
            });
            
            return data;
        } catch (error) {
            console.error('Ошибка при выполнении запроса:', error);
            throw error;
        }
    }
    
    // Очистка устаревшего кэша
    clearExpiredCache() {
        const now = Date.now();
        for (const [key, value] of this.cache.entries()) {
            if (now - value.timestamp >= this.cacheTimeout) {
                this.cache.delete(key);
            }
        }
    }
    
    // Очистка всего кэша
    clearCache() {
        this.cache.clear();
    }
}

// Пример использования
const cachedApiClient = new CachedApiClient('https://api.example.com');
// cachedApiClient.getWithCache('/users')
//     .then(users => console.log(users))
//     .catch(error => console.error(error));
```

## 3. WebSocket основы

```javascript
class WebSocketManager {
    constructor(url, options = {}) {
        this.url = url;
        this.options = {
            reconnectInterval: 5000,
            maxReconnectAttempts: 5,
            heartbeatInterval: 30000,
            ...options
        };
        
        this.ws = null;
        this.reconnectAttempts = 0;
        this.heartbeatTimer = null;
        this.messageHandlers = new Map();
        this.readyState = WebSocket.CLOSED;
        
        this.connect();
    }
    
    connect() {
        try {
            this.ws = new WebSocket(this.url);
            this.setupEventHandlers();
        } catch (error) {
            console.error('Ошибка при создании WebSocket:', error);
            this.handleReconnect();
        }
    }
    
    setupEventHandlers() {
        this.ws.onopen = (event) => {
            console.log('WebSocket соединение установлено');
            this.reconnectAttempts = 0;
            this.readyState = WebSocket.OPEN;
            
            // Запускаем отправку heartbeat
            this.startHeartbeat();
            
            // Вызываем пользовательский обработчик
            this.handleMessage({ type: 'connection', status: 'open' });
        };
        
        this.ws.onmessage = (event) => {
            try {
                const data = JSON.parse(event.data);
                this.handleMessage(data);
            } catch (error) {
                console.error('Ошибка при парсинге сообщения:', error);
                // Обработка не-JSON сообщений
                this.handleMessage({ type: 'raw', data: event.data });
            }
        };
        
        this.ws.onclose = (event) => {
            console.log('WebSocket соединение закрыто:', event.code, event.reason);
            this.readyState = WebSocket.CLOSED;
            this.stopHeartbeat();
            
            // Попытка переподключения
            this.handleReconnect();
        };
        
        this.ws.onerror = (error) => {
            console.error('Ошибка WebSocket:', error);
        };
    }
    
    handleMessage(data) {
        // Обработка системных сообщений
        if (data.type === 'heartbeat') {
            return; // Просто игнорируем heartbeat
        }
        
        // Вызов зарегистрированных обработчиков
        if (this.messageHandlers.has(data.type)) {
            this.messageHandlers.get(data.type).forEach(handler => {
                handler(data);
            });
        } else {
            // Обработка неизвестных типов сообщений
            console.log('Получено сообщение неизвестного типа:', data);
        }
    }
    
    send(data) {
        if (this.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(data));
        } else {
            console.warn('Попытка отправить сообщение при закрытом соединении');
        }
    }
    
    // Регистрация обработчика сообщений
    onMessage(type, handler) {
        if (!this.messageHandlers.has(type)) {
            this.messageHandlers.set(type, []);
        }
        this.messageHandlers.get(type).push(handler);
    }
    
    // Отмена регистрации обработчика
    offMessage(type, handler) {
        if (this.messageHandlers.has(type)) {
            const handlers = this.messageHandlers.get(type);
            const index = handlers.indexOf(handler);
            if (index > -1) {
                handlers.splice(index, 1);
            }
        }
    }
    
    // Попытка переподключения
    handleReconnect() {
        if (this.reconnectAttempts < this.options.maxReconnectAttempts) {
            this.reconnectAttempts++;
            console.log(`Попытка переподключения... (${this.reconnectAttempts}/${this.options.maxReconnectAttempts})`);
            
            setTimeout(() => {
                this.connect();
            }, this.options.reconnectInterval);
        } else {
            console.error('Достигнуто максимальное количество попыток переподключения');
        }
    }
    
    // Отправка heartbeat
    startHeartbeat() {
        this.heartbeatTimer = setInterval(() => {
            if (this.readyState === WebSocket.OPEN) {
                this.send({ type: 'heartbeat', timestamp: Date.now() });
            }
        }, this.options.heartbeatInterval);
    }
    
    stopHeartbeat() {
        if (this.heartbeatTimer) {
            clearInterval(this.heartbeatTimer);
            this.heartbeatTimer = null;
        }
    }
    
    close() {
        this.stopHeartbeat();
        if (this.ws) {
            this.ws.close();
        }
    }
}

// Пример использования
// const wsManager = new WebSocketManager('ws://localhost:8080');
// 
// wsManager.onMessage('chat', (data) => {
//     console.log('Получено сообщение чата:', data.message);
// });
// 
// wsManager.onMessage('notification', (data) => {
//     console.log('Получено уведомление:', data.text);
// });
// 
// // Отправка сообщения
// wsManager.send({
//     type: 'chat',
//     message: 'Привет, мир!',
//     timestamp: Date.now()
// });
```

## 4. Продвинутые примеры WebSocket

### Чат-приложение

```javascript
class ChatClient {
    constructor(webSocketUrl) {
        this.wsManager = new WebSocketManager(webSocketUrl);
        this.messageHistory = [];
        this.users = new Map();
        
        this.setupMessageHandlers();
    }
    
    setupMessageHandlers() {
        // Обработка входящих сообщений
        this.wsManager.onMessage('message', (data) => {
            this.addMessageToHistory(data);
            this.onNewMessage(data);
        });
        
        // Обработка списка пользователей
        this.wsManager.onMessage('userList', (data) => {
            this.users.clear();
            data.users.forEach(user => {
                this.users.set(user.id, user);
            });
            this.onUserListUpdate(this.users);
        });
        
        // Обработка подключений/отключений пользователей
        this.wsManager.onMessage('userJoined', (data) => {
            this.users.set(data.user.id, data.user);
            this.onUserJoined(data.user);
        });
        
        this.wsManager.onMessage('userLeft', (data) => {
            this.users.delete(data.userId);
            this.onUserLeft(data.userId);
        });
        
        // Обработка ошибок
        this.wsManager.onMessage('error', (data) => {
            this.onError(data.message);
        });
    }
    
    // Отправка сообщения
    sendMessage(text, metadata = {}) {
        const message = {
            type: 'message',
            text: text,
            timestamp: Date.now(),
            ...metadata
        };
        
        this.wsManager.send(message);
    }
    
    // Добавление сообщения в историю
    addMessageToHistory(message) {
        this.messageHistory.push(message);
        
        // Ограничиваем размер истории
        if (this.messageHistory.length > 1000) {
            this.messageHistory = this.messageHistory.slice(-500);
        }
    }
    
    // Методы для переопределения в наследниках
    onNewMessage(message) {
        console.log('Новое сообщение:', message);
    }
    
    onUserListUpdate(users) {
        console.log('Список пользователей обновлен:', users.size, 'пользователей');
    }
    
    onUserJoined(user) {
        console.log('Пользователь подключился:', user.name);
    }
    
    onUserLeft(userId) {
        console.log('Пользователь отключился:', userId);
    }
    
    onError(message) {
        console.error('Ошибка чата:', message);
    }
    
    // Получение истории сообщений
    getRecentMessages(count = 50) {
        return this.messageHistory.slice(-count);
    }
    
    // Получение списка пользователей
    getUserList() {
        return Array.from(this.users.values());
    }
    
    // Закрытие соединения
    disconnect() {
        this.wsManager.close();
    }
}

// Пример использования чата
// const chat = new ChatClient('ws://localhost:8080/chat');
// 
// chat.onNewMessage = (message) => {
//     console.log(`${message.user}: ${message.text}`);
// };
// 
// chat.sendMessage('Привет всем в чате!');
```

### Реал-тайм доска

```javascript
class WhiteboardClient {
    constructor(webSocketUrl, canvasId) {
        this.wsManager = new WebSocketManager(webSocketUrl);
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.isDrawing = false;
        this.currentPath = [];
        this.drawingHistory = [];
        
        this.setupCanvasEvents();
        this.setupMessageHandlers();
    }
    
    setupCanvasEvents() {
        this.canvas.addEventListener('mousedown', (e) => {
            this.isDrawing = true;
            const rect = this.canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            this.currentPath = [{ x, y }];
            this.ctx.beginPath();
            this.ctx.moveTo(x, y);
        });
        
        this.canvas.addEventListener('mousemove', (e) => {
            if (!this.isDrawing) return;
            
            const rect = this.canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            
            this.currentPath.push({ x, y });
            
            this.ctx.lineTo(x, y);
            this.ctx.stroke();
            
            // Отправляем действие на сервер
            this.wsManager.send({
                type: 'draw',
                action: 'lineTo',
                x,
                y
            });
        });
        
        this.canvas.addEventListener('mouseup', () => {
            if (this.isDrawing) {
                this.isDrawing = false;
                
                // Отправляем завершенный путь
                this.wsManager.send({
                    type: 'draw',
                    action: 'pathComplete',
                    path: this.currentPath
                });
                
                this.drawingHistory.push([...this.currentPath]);
                this.currentPath = [];
            }
        });
        
        this.canvas.addEventListener('mouseout', () => {
            if (this.isDrawing) {
                this.isDrawing = false;
                this.currentPath = [];
            }
        });
    }
    
    setupMessageHandlers() {
        this.wsManager.onMessage('draw', (data) => {
            this.handleRemoteDraw(data);
        });
        
        this.wsManager.onMessage('clear', () => {
            this.clearCanvas();
        });
        
        this.wsManager.onMessage('sync', (data) => {
            this.syncDrawing(data.history);
        });
    }
    
    handleRemoteDraw(data) {
        if (data.action === 'lineTo') {
            this.ctx.lineTo(data.x, data.y);
            this.ctx.stroke();
        } else if (data.action === 'pathComplete') {
            // Воспроизводим весь путь
            if (data.path && data.path.length > 0) {
                this.ctx.beginPath();
                this.ctx.moveTo(data.path[0].x, data.path[0].y);
                
                for (let i = 1; i < data.path.length; i++) {
                    this.ctx.lineTo(data.path[i].x, data.path[i].y);
                }
                
                this.ctx.stroke();
            }
        }
    }
    
    clearCanvas() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.drawingHistory = [];
    }
    
    syncDrawing(history) {
        this.drawingHistory = history;
        this.redraw();
    }
    
    redraw() {
        this.clearCanvas();
        
        // Воспроизводим всю историю рисования
        this.drawingHistory.forEach(path => {
            if (path.length > 0) {
                this.ctx.beginPath();
                this.ctx.moveTo(path[0].x, path[0].y);
                
                for (let i = 1; i < path.length; i++) {
                    this.ctx.lineTo(path[i].x, path[i].y);
                }
                
                this.ctx.stroke();
            }
        });
    }
    
    // Отправка команды очистки
    sendClearCommand() {
        this.wsManager.send({ type: 'clear' });
    }
    
    // Запрос синхронизации
    requestSync() {
        this.wsManager.send({ type: 'syncRequest' });
    }
}
```

## 5. Обработка ошибок и отладка

```javascript
// Улучшенный менеджер сетевых запросов с обработкой ошибок
class NetworkManager {
    constructor(options = {}) {
        this.timeout = options.timeout || 10000;
        this.retryAttempts = options.retryAttempts || 3;
        this.retryDelay = options.retryDelay || 1000;
        this.interceptors = {
            request: [],
            response: []
        };
    }
    
    // Добавление перехватчика запросов
    addRequestInterceptor(interceptor) {
        this.interceptors.request.push(interceptor);
    }
    
    // Добавление перехватчика ответов
    addResponseInterceptor(interceptor) {
        this.interceptors.response.push(interceptor);
    }
    
    // Выполнение запроса с повторными попытками
    async fetchWithRetry(url, options = {}, attempt = 1) {
        // Применение перехватчиков запросов
        let modifiedOptions = { ...options };
        for (const interceptor of this.interceptors.request) {
            modifiedOptions = await interceptor(modifiedOptions);
        }
        
        try {
            // Установка таймаута
            const controller = new AbortController();
            const timeoutId = setTimeout(() => controller.abort(), this.timeout);
            
            const response = await fetch(url, {
                ...modifiedOptions,
                signal: controller.signal
            });
            
            clearTimeout(timeoutId);
            
            // Применение перехватчиков ответов
            for (const interceptor of this.interceptors.response) {
                await interceptor(response);
            }
            
            return response;
        } catch (error) {
            if (error.name === 'AbortError') {
                throw new Error(`Таймаут запроса: ${url}`);
            }
            
            // Повторная попытка, если не превышено максимальное количество попыток
            if (attempt < this.retryAttempts) {
                console.log(`Попытка ${attempt} не удалась, повтор через ${this.retryDelay}мс`);
                await new Promise(resolve => setTimeout(resolve, this.retryDelay));
                return this.fetchWithRetry(url, options, attempt + 1);
            }
            
            throw error;
        }
    }
    
    // Универсальный метод для выполнения запросов
    async request(url, options = {}) {
        try {
            const response = await this.fetchWithRetry(url, options);
            
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            
            return response;
        } catch (error) {
            console.error('Ошибка сети:', error);
            throw error;
        }
    }
    
    // Методы для разных HTTP-методов
    async get(url, options = {}) {
        return this.request(url, { method: 'GET', ...options });
    }
    
    async post(url, data, options = {}) {
        return this.request(url, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            body: JSON.stringify(data),
            ...options
        });
    }
    
    async put(url, data, options = {}) {
        return this.request(url, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            body: JSON.stringify(data),
            ...options
        });
    }
    
    async delete(url, options = {}) {
        return this.request(url, { method: 'DELETE', ...options });
    }
}

// Пример перехватчиков
const networkManager = new NetworkManager({
    timeout: 15000,
    retryAttempts: 2
});

// Перехватчик для добавления аутентификации
networkManager.addRequestInterceptor(async (options) => {
    const token = localStorage.getItem('authToken');
    if (token) {
        options.headers = {
            ...options.headers,
            'Authorization': `Bearer ${token}`
        };
    }
    return options;
});

// Перехватчик для логирования ответов
networkManager.addResponseInterceptor(async (response) => {
    console.log(`Запрос к ${response.url} завершен с кодом ${response.status}`);
    return response;
});
```

## 6. Практические примеры использования

### API для управления задачами

```javascript
class TaskManager {
    constructor(apiBaseUrl) {
        this.api = new ApiClient(apiBaseUrl);
    }
    
    // Получение списка задач
    async getTasks(filters = {}) {
        try {
            const tasks = await this.api.get('/tasks', filters);
            return tasks;
        } catch (error) {
            console.error('Ошибка при получении задач:', error);
            throw error;
        }
    }
    
    // Создание новой задачи
    async createTask(taskData) {
        try {
            const newTask = await this.api.post('/tasks', taskData);
            return newTask;
        } catch (error) {
            console.error('Ошибка при создании задачи:', error);
            throw error;
        }
    }
    
    // Обновление задачи
    async updateTask(taskId, taskData) {
        try {
            const updatedTask = await this.api.put(`/tasks/${taskId}`, taskData);
            return updatedTask;
        } catch (error) {
            console.error('Ошибка при обновлении задачи:', error);
            throw error;
        }
    }
    
    // Удаление задачи
    async deleteTask(taskId) {
        try {
            const result = await this.api.delete(`/tasks/${taskId}`);
            return result;
        } catch (error) {
            console.error('Ошибка при удалении задачи:', error);
            throw error;
        }
    }
    
    // Изменение статуса задачи
    async updateTaskStatus(taskId, status) {
        try {
            const updatedTask = await this.api.put(`/tasks/${taskId}/status`, { status });
            return updatedTask;
        } catch (error) {
            console.error('Ошибка при изменении статуса задачи:', error);
            throw error;
        }
    }
}

// Пример использования
// const taskManager = new TaskManager('https://api.example.com');
// 
// taskManager.getTasks({ status: 'pending' })
//     .then(tasks => console.log('Текущие задачи:', tasks))
//     .catch(error => console.error(error));
```

### Live-обновления с WebSocket

```javascript
class LiveUpdatesManager {
    constructor(webSocketUrl) {
        this.wsManager = new WebSocketManager(webSocketUrl);
        this.updateCallbacks = new Map();
        this.dataCache = new Map();
        
        this.setupMessageHandlers();
    }
    
    setupMessageHandlers() {
        // Обработка обновлений данных
        this.wsManager.onMessage('dataUpdate', (data) => {
            this.handleDataUpdate(data);
        });
        
        // Обработка уведомлений
        this.wsManager.onMessage('notification', (data) => {
            this.handleNotification(data);
        });
        
        // Обработка ошибок
        this.wsManager.onMessage('error', (data) => {
            this.handleError(data);
        });
    }
    
    // Подписка на обновления определенного типа данных
    subscribe(dataType, callback) {
        if (!this.updateCallbacks.has(dataType)) {
            this.updateCallbacks.set(dataType, []);
        }
        
        this.updateCallbacks.get(dataType).push(callback);
        
        // Отправляем последнее закэшированное значение, если оно есть
        if (this.dataCache.has(dataType)) {
            callback(this.dataCache.get(dataType));
        }
    }
    
    // Отписка от обновлений
    unsubscribe(dataType, callback) {
        if (this.updateCallbacks.has(dataType)) {
            const callbacks = this.updateCallbacks.get(dataType);
            const index = callbacks.indexOf(callback);
            if (index > -1) {
                callbacks.splice(index, 1);
            }
        }
    }
    
    handleDataUpdate(data) {
        const { type, payload } = data;
        
        // Кэшируем данные
        this.dataCache.set(type, payload);
        
        // Вызываем все зарегистрированные колбэки
        if (this.updateCallbacks.has(type)) {
            this.updateCallbacks.get(type).forEach(callback => {
                callback(payload);
            });
        }
    }
    
    handleNotification(data) {
        console.log('Получено уведомление:', data);
        // Здесь можно показать уведомление пользователю
    }
    
    handleError(data) {
        console.error('Ошибка в live-обновлениях:', data.message);
    }
    
    // Отправка команды на сервер
    sendCommand(command, data) {
        this.wsManager.send({
            type: 'command',
            command,
            data
        });
    }
    
    // Закрытие соединения
    disconnect() {
        this.wsManager.close();
    }
}

// Пример использования
// const liveUpdates = new LiveUpdatesManager('ws://localhost:8080/live');
// 
// liveUpdates.subscribe('userCount', (count) => {
//     document.getElementById('userCount').textContent = `Пользователей онлайн: ${count}`;
// });
// 
// liveUpdates.subscribe('latestNews', (news) => {
//     displayNews(news);
// });
```

Эти примеры демонстрируют практическое использование Fetch API и WebSocket для создания современных веб-приложений с сетевым взаимодействием.

См. также:
- [[js/practical-examples/canvas-graphics-examples]] - Работа с canvas и графикой
- [[js/practical-examples/web-api-examples]] - Работа с Web API
- [[js/practical-examples/web-workers-examples]] - Примеры использования Web Workers