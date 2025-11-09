# Браузерные API

Браузерные API (Application Programming Interfaces) предоставляют JavaScript доступ к различным возможностям браузера и операционной системы. Эти API позволяют создавать богатые интерактивные веб-приложения с расширенным функционалом.

## Классификация браузерных API

### API доступа к данным

#### Fetch API

```javascript
// Основное использование Fetch API
async function fetchData(url) {
    try {
        const response = await fetch(url);
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Ошибка при получении данных:', error);
        throw error;
    }
}

// POST запрос с заголовками
async function postData(url, data) {
    try {
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
    } catch (error) {
        console.error('Ошибка при отправке данных:', error);
        throw error;
    }
}

// Загрузка файла
async function uploadFile(url, file) {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await fetch(url, {
        method: 'POST',
        body: formData
    });
    
    return await response.json();
}
```

#### XMLHttpRequest (устаревший, но все еще используется)

```javascript
// Классический XMLHttpRequest
function xhrRequest(url, method = 'GET', data = null) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.open(method, url);
        xhr.setRequestHeader('Content-Type', 'application/json');
        
        xhr.onload = function() {
            if (xhr.status >= 200 && xhr.status < 300) {
                resolve(JSON.parse(xhr.responseText));
            } else {
                reject(new Error(`HTTP error! status: ${xhr.status}`));
            }
        };
        
        xhr.onerror = function() {
            reject(new Error('Network error'));
        };
        
        xhr.send(data ? JSON.stringify(data) : null);
    });
}
```

### API хранения данных

#### localStorage и sessionStorage

```javascript
// localStorage - данные сохраняются до удаления пользователем
const storage = {
    set(key, value) {
        try {
            localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
            console.error('Ошибка сохранения в localStorage:', error);
        }
    },
    
    get(key) {
        try {
            const item = localStorage.getItem(key);
            return item ? JSON.parse(item) : null;
        } catch (error) {
            console.error('Ошибка получения из localStorage:', error);
            return null;
        }
    },
    
    remove(key) {
        localStorage.removeItem(key);
    },
    
    clear() {
        localStorage.clear();
    }
};

// sessionStorage - данные сохраняются до закрытия вкладки
const session = {
    set(key, value) {
        sessionStorage.setItem(key, JSON.stringify(value));
    },
    
    get(key) {
        const item = sessionStorage.getItem(key);
        return item ? JSON.parse(item) : null;
    }
};
```

#### IndexedDB

```javascript
// Пример использования IndexedDB
class IndexedDBManager {
    constructor(dbName, version = 1) {
        this.dbName = dbName;
        this.version = version;
        this.db = null;
    }
    
    async init() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                this.db = request.result;
                resolve(this.db);
            };
            
            request.onupgradeneeded = (event) => {
                this.db = event.target.result;
                
                // Создание объектного хранилища
                if (!this.db.objectStoreNames.contains('items')) {
                    const objectStore = this.db.createObjectStore('items', { keyPath: 'id', autoIncrement: true });
                    objectStore.createIndex('name', 'name', { unique: false });
                    objectStore.createIndex('date', 'date', { unique: false });
                }
            };
        });
    }
    
    async add(item) {
        const transaction = this.db.transaction(['items'], 'readwrite');
        const store = transaction.objectStore('items');
        return store.add(item);
    }
    
    async get(id) {
        const transaction = this.db.transaction(['items'], 'readonly');
        const store = transaction.objectStore('items');
        return store.get(id);
    }
    
    async getAll() {
        const transaction = this.db.transaction(['items'], 'readonly');
        const store = transaction.objectStore('items');
        return store.getAll();
    }
    
    async delete(id) {
        const transaction = this.db.transaction(['items'], 'readwrite');
        const store = transaction.objectStore('items');
        return store.delete(id);
    }
}

// Использование
const dbManager = new IndexedDBManager('myDatabase');
await dbManager.init();
await dbManager.add({ name: 'Тестовый элемент', date: new Date() });
```

### API мультимедиа

#### Работа с аудио и видео

```javascript
// Воспроизведение аудио
class AudioManager {
    constructor() {
        this.audioContext = null;
        this.tracks = new Map();
    }
    
    async init() {
        try {
            this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
        } catch (error) {
            console.error('Web Audio API не поддерживается:', error);
        }
    }
    
    async playAudio(url) {
        try {
            const response = await fetch(url);
            const arrayBuffer = await response.arrayBuffer();
            const audioBuffer = await this.audioContext.decodeAudioData(arrayBuffer);
            
            const source = this.audioContext.createBufferSource();
            source.buffer = audioBuffer;
            source.connect(this.audioContext.destination);
            source.start();
            
            return source;
        } catch (error) {
            console.error('Ошибка воспроизведения аудио:', error);
        }
    }
}

// Управление видео
class VideoManager {
    constructor(videoElement) {
        this.video = videoElement;
    }
    
    async loadVideo(src) {
        this.video.src = src;
        return new Promise((resolve, reject) => {
            this.video.addEventListener('loadedmetadata', resolve);
            this.video.addEventListener('error', reject);
        });
    }
    
    play() {
        return this.video.play();
    }
    
    pause() {
        this.video.pause();
    }
    
    setVolume(volume) {
        this.video.volume = volume;
    }
    
    setPlaybackRate(rate) {
        this.video.playbackRate = rate;
    }
    
    async captureFrame() {
        const canvas = document.createElement('canvas');
        canvas.width = this.video.videoWidth;
        canvas.height = this.video.videoHeight;
        
        const ctx = canvas.getContext('2d');
        ctx.drawImage(this.video, 0, 0, canvas.width, canvas.height);
        
        return canvas.toDataURL('image/png');
    }
}
```

#### Доступ к камере и микрофону

```javascript
// Доступ к медиаустройствам
class MediaManager {
    constructor() {
        this.stream = null;
        this.constraints = {
            video: { width: 1280, height: 720 },
            audio: true
        };
    }
    
    async startCamera() {
        try {
            this.stream = await navigator.mediaDevices.getUserMedia(this.constraints);
            return this.stream;
        } catch (error) {
            console.error('Ошибка доступа к камере:', error);
            throw error;
        }
    }
    
    async stopCamera() {
        if (this.stream) {
            this.stream.getTracks().forEach(track => track.stop());
            this.stream = null;
        }
    }
    
    async switchCamera() {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const videoDevices = devices.filter(device => device.kind === 'videoinput');
        
        if (videoDevices.length > 1) {
            this.constraints.video.deviceId = { exact: videoDevices[1].deviceId };
            await this.stopCamera();
            return await this.startCamera();
        }
    }
    
    async takePhoto(videoElement) {
        const canvas = document.createElement('canvas');
        canvas.width = videoElement.videoWidth;
        canvas.height = videoElement.videoHeight;
        
        const ctx = canvas.getContext('2d');
        ctx.drawImage(videoElement, 0, 0, canvas.width, canvas.height);
        
        return canvas.toDataURL('image/jpeg');
    }
}
```

### API геолокации

```javascript
// Работа с геолокацией
class GeolocationManager {
    constructor() {
        this.watchId = null;
    }
    
    async getCurrentPosition(options = {}) {
        return new Promise((resolve, reject) => {
            if (!navigator.geolocation) {
                reject(new Error('Геолокация не поддерживается'));
                return;
            }
            
            const defaultOptions = {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 0
            };
            
            navigator.geolocation.getCurrentPosition(
                position => resolve(position),
                error => reject(error),
                { ...defaultOptions, ...options }
            );
        });
    }
    
    watchPosition(callback, options = {}) {
        if (!navigator.geolocation) {
            throw new Error('Геолокация не поддерживается');
        }
        
        const defaultOptions = {
            enableHighAccuracy: true,
            timeout: 10000,
            maximumAge: 60000
        };
        
        this.watchId = navigator.geolocation.watchPosition(
            callback,
            error => console.error('Ошибка геолокации:', error),
            { ...defaultOptions, ...options }
        );
        
        return this.watchId;
    }
    
    clearWatch() {
        if (this.watchId !== null) {
            navigator.geolocation.clearWatch(this.watchId);
            this.watchId = null;
        }
    }
    
    async getFormattedAddress(lat, lng) {
        try {
            // Использование стороннего API для получения адреса по координатам
            const response = await fetch(
                `https://api.opencagedata.com/geocode/v1/json?q=${lat}+${lng}&key=YOUR_API_KEY`
            );
            const data = await response.json();
            return data.results[0].formatted;
        } catch (error) {
            console.error('Ошибка получения адреса:', error);
            return null;
        }
    }
}
```

### API уведомлений

```javascript
// Работа с веб-уведомлениями
class NotificationManager {
    constructor() {
        this.permission = Notification.permission;
    }
    
    async requestPermission() {
        if (Notification.permission === 'default') {
            this.permission = await Notification.requestPermission();
        }
        return this.permission;
    }
    
    showNotification(title, options = {}) {
        if (this.permission !== 'granted') {
            console.warn('Разрешение на уведомления не получено');
            return null;
        }
        
        return new Notification(title, {
            body: options.body || '',
            icon: options.icon || '',
            tag: options.tag || '',
            badge: options.badge || '',
            vibrate: options.vibrate || [200, 100, 200],
            ...options
        });
    }
    
    async showWithActions(title, body, actions = []) {
        const registration = await navigator.serviceWorker.ready;
        
        registration.showNotification(title, {
            body: body,
            actions: actions,
            tag: 'actionable-notification'
        });
    }
}

// Использование
const notificationManager = new NotificationManager();
await notificationManager.requestPermission();
notificationManager.showNotification('Новое сообщение', {
    body: 'У вас новое сообщение от пользователя',
    icon: '/icon.png'
});
```

### API производительности

```javascript
// Работа с API производительности
class PerformanceManager {
    static measureFunction(fn, name) {
        const start = performance.now();
        const result = fn();
        const end = performance.now();
        
        console.log(`${name} выполнена за ${end - start} мс`);
        return result;
    }
    
    static async measureAsyncFunction(asyncFn, name) {
        const start = performance.now();
        const result = await asyncFn();
        const end = performance.now();
        
        console.log(`${name} выполнена за ${end - start} мс`);
        return result;
    }
    
    static getMemoryInfo() {
        if ('memory' in performance) {
            return performance.memory;
        }
        return null;
    }
    
    static markAndMeasure(markName, measureName) {
        performance.mark(markName);
        
        // Выполнение измеряемого кода
        // ...
        
        performance.mark(`${markName}-end`);
        performance.measure(measureName, markName, `${markName}-end`);
        
        const measure = performance.getEntriesByName(measureName)[0];
        console.log(`${measureName}: ${measure.duration} мс`);
        
        // Очистка записей
        performance.clearMarks(markName);
        performance.clearMarks(`${markName}-end`);
        performance.clearMeasures(measureName);
    }
    
    static observePerformance() {
        // Наблюдение за производительностью
        if ('PerformanceObserver' in window) {
            const observer = new PerformanceObserver((list) => {
                list.getEntries().forEach((entry) => {
                    console.log(`${entry.name}: ${entry.duration} мс`);
                });
            });
            
            observer.observe({ entryTypes: ['measure', 'navigation', 'resource'] });
        }
    }
}
```

### API доступности

```javascript
// Работа с API доступности
class AccessibilityManager {
    static isHighContrastMode() {
        return window.matchMedia('(prefers-contrast: high)').matches;
    }
    
    static prefersReducedMotion() {
        return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    }
    
    static isDarkMode() {
        return window.matchMedia('(prefers-color-scheme: dark)').matches;
    }
    
    static setupAccessibilityListeners() {
        // Слушатель для изменения предпочтений пользователя
        window.matchMedia('(prefers-color-scheme: dark)')
            .addEventListener('change', (e) => {
                if (e.matches) {
                    document.body.classList.add('dark-theme');
                } else {
                    document.body.classList.remove('dark-theme');
                }
            });
        
        window.matchMedia('(prefers-reduced-motion: reduce)')
            .addEventListener('change', (e) => {
                if (e.matches) {
                    document.body.classList.add('reduce-motion');
                } else {
                    document.body.classList.remove('reduce-motion');
                }
            });
    }
    
    static ensureKeyboardNavigation() {
        // Улучшение навигации с клавиатуры
        document.addEventListener('keydown', (event) => {
            if (event.key === 'Tab') {
                document.body.classList.add('keyboard-navigation');
            }
        });
        
        document.addEventListener('mousedown', () => {
            document.body.classList.remove('keyboard-navigation');
        });
    }
}
```

## Современные API

### Web Workers

```javascript
// Основное использование Web Workers
class WorkerManager {
    constructor(workerScript) {
        this.worker = new Worker(workerScript);
        this.messageId = 0;
        this.pendingPromises = new Map();
        
        this.worker.onmessage = (event) => {
            const { id, result, error } = event.data;
            const promise = this.pendingPromises.get(id);
            
            if (promise) {
                this.pendingPromises.delete(id);
                if (error) {
                    promise.reject(error);
                } else {
                    promise.resolve(result);
                }
            }
        };
    }
    
    executeTask(task, data) {
        return new Promise((resolve, reject) => {
            const id = ++this.messageId;
            this.pendingPromises.set(id, { resolve, reject });
            
            this.worker.postMessage({ id, task, data });
        });
    }
    
    terminate() {
        this.worker.terminate();
    }
}

// Пример воркера (в отдельном файле)
/*
// worker.js
self.onmessage = function(event) {
    const { id, task, data } = event.data;
    
    try {
        let result;
        
        switch(task) {
            case 'calculate':
                result = performHeavyCalculation(data);
                break;
            case 'processData':
                result = processData(data);
                break;
            default:
                throw new Error(`Неизвестная задача: ${task}`);
        }
        
        self.postMessage({ id, result });
    } catch (error) {
        self.postMessage({ id, error: error.message });
    }
};
*/
```

### Service Workers

```javascript
// Регистрация Service Worker
class ServiceWorkerManager {
    static async registerServiceWorker(swPath) {
        if ('serviceWorker' in navigator) {
            try {
                const registration = await navigator.serviceWorker.register(swPath);
                console.log('Service Worker зарегистрирован:', registration.scope);
                
                // Слушатель обновления
                registration.addEventListener('updatefound', () => {
                    const newWorker = registration.installing;
                    newWorker.addEventListener('statechange', () => {
                        if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                            console.log('Новый Service Worker готов к использованию');
                        }
                    });
                });
                
                return registration;
            } catch (error) {
                console.error('Ошибка регистрации Service Worker:', error);
            }
        } else {
            console.log('Service Worker не поддерживается');
        }
    }
    
    static async checkForUpdates() {
        if ('serviceWorker' in navigator) {
            const registration = await navigator.serviceWorker.ready;
            if (registration.waiting) {
                registration.waiting.postMessage({ type: 'SKIP_WAITING' });
            }
        }
    }
}
```

### WebSockets

```javascript
// Управление WebSocket соединениями
class WebSocketManager {
    constructor(url, options = {}) {
        this.url = url;
        this.options = {
            reconnectInterval: 5000,
            maxReconnectAttempts: 5,
            ...options
        };
        this.ws = null;
        this.reconnectAttempts = 0;
        this.messageQueue = [];
        this.eventHandlers = new Map();
    }
    
    connect() {
        this.ws = new WebSocket(this.url);
        
        this.ws.onopen = () => {
            console.log('WebSocket соединение установлено');
            this.reconnectAttempts = 0;
            
            // Отправка отложенных сообщений
            while (this.messageQueue.length > 0) {
                const message = this.messageQueue.shift();
                this.ws.send(JSON.stringify(message));
            }
        };
        
        this.ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.handleMessage(data);
        };
        
        this.ws.onclose = () => {
            console.log('WebSocket соединение закрыто');
            this.attemptReconnect();
        };
        
        this.ws.onerror = (error) => {
            console.error('Ошибка WebSocket:', error);
        };
    }
    
    send(message) {
        if (this.ws && this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(JSON.stringify(message));
        } else {
            // Добавление в очередь при недоступности соединения
            this.messageQueue.push(message);
        }
    }
    
    attemptReconnect() {
        if (this.reconnectAttempts < this.options.maxReconnectAttempts) {
            setTimeout(() => {
                this.reconnectAttempts++;
                console.log(`Попытка переподключения... (${this.reconnectAttempts}/${this.options.maxReconnectAttempts})`);
                this.connect();
            }, this.options.reconnectInterval);
        } else {
            console.error('Достигнуто максимальное количество попыток переподключения');
        }
    }
    
    on(event, handler) {
        if (!this.eventHandlers.has(event)) {
            this.eventHandlers.set(event, []);
        }
        this.eventHandlers.get(event).push(handler);
    }
    
    handleMessage(data) {
        const handlers = this.eventHandlers.get(data.type);
        if (handlers) {
            handlers.forEach(handler => handler(data.payload));
        }
    }
    
    disconnect() {
        if (this.ws) {
            this.ws.close();
        }
    }
}
```

## Лучшие практики использования API

### 1. Проверка поддержки API

```javascript
// Проверка поддержки API перед использованием
function supportsFetch() {
    return typeof fetch !== 'undefined';
}

function supportsWebSockets() {
    return 'WebSocket' in window;
}

function supportsIndexedDB() {
    return 'indexedDB' in window;
}

// Универсальная проверка
function supportsAPI(apiName) {
    const apiMap = {
        'fetch': typeof fetch !== 'undefined',
        'websockets': 'WebSocket' in window,
        'indexeddb': 'indexedDB' in window,
        'geolocation': 'geolocation' in navigator,
        'notifications': 'Notification' in window,
        'webworkers': 'Worker' in window,
        'serviceworkers': 'serviceWorker' in navigator,
        'webaudio': 'AudioContext' in window || 'webkitAudioContext' in window
    };
    
    return apiMap[apiName] || false;
}
```

### 2. Обработка ошибок

```javascript
// Централизованная обработка ошибок API
class APIErrorHandler {
    static handle(error, context = '') {
        console.error(`Ошибка ${context}:`, error);
        
        // Логирование ошибки для анализа
        this.logError(error, context);
        
        // Показ пользовательского сообщения об ошибке
        this.showUserError(error, context);
    }
    
    static logError(error, context) {
        // Отправка ошибки на сервер для анализа
        if ('sendBeacon' in navigator) {
            navigator.sendBeacon('/api/errors', JSON.stringify({
                error: error.toString(),
                context: context,
                url: window.location.href,
                timestamp: new Date().toISOString()
            }));
        }
    }
    
    static showUserError(error, context) {
        // Показ пользовательского интерфейса ошибки
        const errorDiv = document.createElement('div');
        errorDiv.className = 'api-error';
        errorDiv.innerHTML = `
            <p>Произошла ошибка при выполнении операции: ${context}</p>
            <p>${error.message || error}</p>
        `;
        
        document.body.appendChild(errorDiv);
        
        // Автоматическое скрытие через 5 секунд
        setTimeout(() => errorDiv.remove(), 5000);
    }
}
```

### 3. Управление ресурсами

```javascript
// Управление ресурсами API
class ResourceManager {
    constructor() {
        this.resources = new Map();
    }
    
    addResource(name, resource) {
        this.resources.set(name, resource);
    }
    
    removeResource(name) {
        const resource = this.resources.get(name);
        if (resource && typeof resource.destroy === 'function') {
            resource.destroy();
        }
        this.resources.delete(name);
    }
    
    destroyAll() {
        for (const [name, resource] of this.resources) {
            if (resource && typeof resource.destroy === 'function') {
                resource.destroy();
            }
        }
        this.resources.clear();
    }
}

// Использование
const resourceManager = new ResourceManager();

// При создании API-ресурсов
const wsManager = new WebSocketManager('ws://example.com');
resourceManager.addResource('websocket', wsManager);

// При уничтожении компонента
window.addEventListener('beforeunload', () => {
    resourceManager.destroyAll();
});
```

Браузерные API предоставляют мощные возможности для создания современных веб-приложений. Понимание их возможностей и правильное использование позволяет создавать богатые интерактивные интерфейсы с расширенным функционалом.

#javascript #api #frontend #web-development #browser-api