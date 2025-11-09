# PWA API

## Введение

Progressive Web Apps (PWA) - это веб-приложения, которые используют современные веб-технологии для предоставления пользовательского опыта, сопоставимого с нативными приложениями. В этой главе мы рассмотрим все аспекты PWA: Service Workers, манифест приложения, кэширование, оффлайн-функциональность и другие возможности.

## Содержание

- [[Service Workers]]
- [[Web App Manifest]]
- [[Оффлайн функциональность]]
- [[Push-уведомления]]
- [[Кэширование ресурсов]]
- [[Установка PWA]]
- [[Производительность PWA]]
- [[Безопасность PWA]]
- [[Анализ и мониторинг PWA]]

## Service Workers

Service Worker - это скрипт, который запускается в отдельном потоке и позволяет веб-приложению использовать функции, такие как кэширование, push-уведомления и фоновая синхронизация.

### Основы Service Workers

```javascript
// sw.js - Service Worker скрипт
const CACHE_NAME = 'my-pwa-v1.0.0';
const urlsToCache = [
    '/',
    '/styles/main.css',
    '/scripts/app.js',
    '/images/logo.png',
    '/offline.html'
];

// Событие установки
self.addEventListener('install', (event) => {
    console.log('Service Worker устанавливается');
    
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then((cache) => {
                console.log('Кэширование файлов');
                return cache.addAll(urlsToCache);
            })
            .then(() => {
                console.log('Service Worker успешно установлен');
                return self.skipWaiting(); // Активировать немедленно
            })
            .catch((error) => {
                console.error('Ошибка кэширования:', error);
            })
    );
});

// Событие активации
self.addEventListener('activate', (event) => {
    console.log('Service Worker активируется');
    
    event.waitUntil(
        caches.keys().then((cacheNames) => {
            return Promise.all(
                cacheNames.map((cacheName) => {
                    // Удалить старые кэши
                    if (cacheName !== CACHE_NAME) {
                        console.log('Удаление старого кэша:', cacheName);
                        return caches.delete(cacheName);
                    }
                })
            );
        })
        .then(() => {
            return self.clients.claim(); // Взять под контроль все страницы
        })
    );
});

// Событие fetch
self.addEventListener('fetch', (event) => {
    // Обработка только GET запросов
    if (event.request.method !== 'GET') {
        return;
    }
    
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                // Возврат из кэша или сетевой запрос
                if (response) {
                    console.log('Ответ из кэша:', event.request.url);
                    return response;
                }
                
                // Сетевой запрос
                return fetch(event.request)
                    .then((response) => {
                        // Кэширование ответа, если он успешный
                        if (response && response.status === 200) {
                            const responseToCache = response.clone();
                            caches.open(CACHE_NAME)
                                .then((cache) => {
                                    cache.put(event.request, responseToCache);
                                });
                        }
                        return response;
                    })
                    .catch((error) => {
                        console.error('Ошибка сети:', error);
                        
                        // Возврат оффлайн страницы для HTML запросов
                        if (event.request.headers.get('accept').includes('text/html')) {
                            return caches.match('/offline.html');
                        }
                        
                        // Для других запросов возврат ошибки
                        return new Response('Сеть недоступна', {
                            status: 503,
                            headers: { 'Content-Type': 'text/plain' }
                        });
                    });
            })
    );
});

// Обработка сообщений от основного потока
self.addEventListener('message', (event) => {
    const { type, data } = event.data;
    
    switch (type) {
        case 'CACHE_ADD':
            handleCacheAdd(data);
            break;
        case 'CACHE_DELETE':
            handleCacheDelete(data);
            break;
        case 'CLEAR_CACHE':
            handleClearCache();
            break;
        case 'SYNC_DATA':
            handleDataSync();
            break;
        default:
            console.warn('Неизвестный тип сообщения:', type);
    }
});

// Вспомогательные функции
function handleCacheAdd(data) {
    caches.open(CACHE_NAME).then((cache) => {
        cache.add(data.url);
    });
}

function handleCacheDelete(data) {
    caches.open(CACHE_NAME).then((cache) => {
        cache.delete(data.url);
    });
}

function handleClearCache() {
    caches.keys().then((cacheNames) => {
        return Promise.all(
            cacheNames.map(name => caches.delete(name))
        );
    });
}

function handleDataSync() {
    // Реализация фоновой синхронизации
    console.log('Запуск фоновой синхронизации');
    
    // Проверка подключения
    if (navigator.onLine) {
        // Выполнение синхронизации
        console.log('Синхронизация данных...');
    }
}
```

### Продвинутый Service Worker

```javascript
// advanced-sw.js - Продвинутый Service Worker
class ServiceWorkerManager {
    constructor() {
        this.cacheName = 'advanced-pwa-cache-v1';
        this.apiCacheName = 'api-cache-v1';
        this.runtimeCacheName = 'runtime-cache-v1';
        this.version = '1.0.0';
    }
    
    // Инициализация
    async init() {
        await this.setupCaches();
        await this.setupEventListeners();
    }
    
    async setupCaches() {
        // Создание необходимых кэшей
        await Promise.all([
            caches.open(this.cacheName),
            caches.open(this.apiCacheName),
            caches.open(this.runtimeCacheName)
        ]);
    }
    
    async setupEventListeners() {
        self.addEventListener('install', this.handleInstall.bind(this));
        self.addEventListener('activate', this.handleActivate.bind(this));
        self.addEventListener('fetch', this.handleFetch.bind(this));
        self.addEventListener('sync', this.handleSync.bind(this));
        self.addEventListener('push', this.handlePush.bind(this));
        self.addEventListener('notificationclick', this.handleNotificationClick.bind(this));
        self.addEventListener('message', this.handleMessage.bind(this));
    }
    
    async handleInstall(event) {
        console.log('Установка Service Worker');
        
        event.waitUntil(
            (async () => {
                const cache = await caches.open(this.cacheName);
                
                // Кэширование критических ресурсов
                const criticalAssets = [
                    '/',
                    '/index.html',
                    '/styles/app.css',
                    '/scripts/app.js',
                    '/images/icon-192x192.png',
                    '/offline.html'
                ];
                
                await cache.addAll(criticalAssets);
                await self.skipWaiting();
            })()
        );
    }
    
    async handleActivate(event) {
        console.log('Активация Service Worker');
        
        event.waitUntil(
            (async () => {
                // Удаление старых кэшей
                const cacheNames = await caches.keys();
                const currentCaches = [this.cacheName, this.apiCacheName, this.runtimeCacheName];
                
                await Promise.all(
                    cacheNames
                        .filter(name => !currentCaches.includes(name))
                        .map(name => caches.delete(name))
                );
                
                await self.clients.claim();
            })()
        );
    }
    
    async handleFetch(event) {
        const { request } = event;
        
        // Пропуск не-GET запросов
        if (request.method !== 'GET') {
            return;
        }
        
        // Определение типа запроса
        const url = new URL(request.url);
        const isApiRequest = url.pathname.startsWith('/api/');
        const isImageRequest = request.destination === 'image';
        const isHtmlRequest = request.destination === 'document' || request.headers.get('accept').includes('text/html');
        
        event.respondWith(
            (async () => {
                try {
                    if (isApiRequest) {
                        return await this.handleApiRequest(request);
                    } else if (isImageRequest) {
                        return await this.handleImageRequest(request);
                    } else {
                        return await this.handleStaticRequest(request);
                    }
                } catch (error) {
                    console.error('Ошибка обработки запроса:', error);
                    
                    if (isHtmlRequest) {
                        return await caches.match('/offline.html') || this.createOfflineResponse();
                    }
                    
                    return this.createErrorResponse();
                }
            })()
        );
    }
    
    async handleApiRequest(request) {
        // Попытка получить из кэша
        let response = await caches.match(request, { cacheName: this.apiCacheName });
        
        if (response && !this.isCacheExpired(response)) {
            console.log('API ответ из кэша:', request.url);
            return response;
        }
        
        // Сетевой запрос
        try {
            response = await fetch(request);
            
            if (response.ok) {
                // Кэширование успешного ответа
                const responseToCache = response.clone();
                const cache = await caches.open(this.apiCacheName);
                await cache.put(request, responseToCache);
            }
            
            return response;
        } catch (error) {
            // Возврат кэшированного ответа при ошибке сети
            if (response) {
                console.log('Возврат устаревшего кэшированного ответа:', request.url);
                return response;
            }
            
            throw error;
        }
    }
    
    async handleImageRequest(request) {
        // Попытка получить из кэша
        let response = await caches.match(request, { cacheName: this.runtimeCacheName });
        
        if (response) {
            return response;
        }
        
        // Сетевой запрос
        try {
            response = await fetch(request);
            
            if (response.ok) {
                // Кэширование изображения
                const responseToCache = response.clone();
                const cache = await caches.open(this.runtimeCacheName);
                await cache.put(request, responseToCache);
            }
            
            return response;
        } catch (error) {
            // Возврат запасного изображения
            return await caches.match('/images/placeholder.png');
        }
    }
    
    async handleStaticRequest(request) {
        // Попытка получить из основного кэша
        let response = await caches.match(request, { cacheName: this.cacheName });
        
        if (response) {
            return response;
        }
        
        // Сетевой запрос
        try {
            response = await fetch(request);
            
            if (response.ok) {
                // Кэширование статических ресурсов
                const responseToCache = response.clone();
                const cache = await caches.open(this.cacheName);
                await cache.put(request, responseToCache);
            }
            
            return response;
        } catch (error) {
            throw error;
        }
    }
    
    isCacheExpired(response) {
        const dateHeader = response.headers.get('date');
        if (!dateHeader) return false;
        
        const cacheTime = new Date(dateHeader).getTime();
        const now = Date.now();
        const maxAge = 5 * 60 * 1000; // 5 минут
        
        return (now - cacheTime) > maxAge;
    }
    
    async handleSync(event) {
        if (event.tag === 'data-sync') {
            event.waitUntil(this.performDataSync());
        }
    }
    
    async performDataSync() {
        // Синхронизация данных в фоновом режиме
        console.log('Фоновая синхронизация данных...');
        
        const clients = await self.clients.matchAll();
        for (const client of clients) {
            client.postMessage({ type: 'SYNC_START' });
        }
        
        try {
            // Выполнение синхронизации
            await this.syncPendingChanges();
            
            for (const client of clients) {
                client.postMessage({ type: 'SYNC_COMPLETE' });
            }
        } catch (error) {
            console.error('Ошибка синхронизации:', error);
            
            for (const client of clients) {
                client.postMessage({ type: 'SYNC_ERROR', error: error.message });
            }
        }
    }
    
    async syncPendingChanges() {
        // Логика синхронизации изменений
        // Например, отправка данных, накопленных в IndexedDB
    }
    
    async handlePush(event) {
        const data = event.data.json();
        
        event.waitUntil(
            self.registration.showNotification(data.title, {
                body: data.body,
                icon: data.icon || '/images/icon-192x192.png',
                badge: '/images/badge-72x72.png',
                tag: data.tag,
                data: data.customData
            })
        );
    }
    
    async handleNotificationClick(event) {
        event.notification.close();
        
        event.waitUntil(
            self.clients.openWindow(event.notification.data.url || '/')
        );
    }
    
    async handleMessage(event) {
        const { type, data } = event.data;
        
        switch (type) {
            case 'UPDATE_CACHE':
                await this.updateCache(data);
                break;
            case 'CLEAR_ALL_CACHES':
                await this.clearAllCaches();
                break;
            case 'GET_CACHE_STATS':
                const stats = await this.getCacheStats();
                event.source.postMessage({ type: 'CACHE_STATS', data: stats });
                break;
        }
    }
    
    async updateCache(data) {
        const cache = await caches.open(data.cacheName || this.cacheName);
        await cache.put(data.url, data.response);
    }
    
    async clearAllCaches() {
        const cacheNames = await caches.keys();
        await Promise.all(cacheNames.map(name => caches.delete(name)));
    }
    
    async getCacheStats() {
        const cacheNames = await caches.keys();
        const stats = {};
        
        for (const name of cacheNames) {
            const cache = await caches.open(name);
            const requests = await cache.keys();
            stats[name] = {
                size: requests.length,
                requests: requests.map(r => r.url)
            };
        }
        
        return stats;
    }
    
    createOfflineResponse() {
        return new Response(
            '<html><body><h1>Оффлайн</h1><p>Приложение работает в оффлайн режиме</p></body></html>',
            {
                status: 200,
                headers: { 'Content-Type': 'text/html' }
            }
        );
    }
    
    createErrorResponse() {
        return new Response('Сеть недоступна', {
            status: 503,
            headers: { 'Content-Type': 'text/plain' }
        });
    }
}

// Инициализация менеджера Service Worker
const swManager = new ServiceWorkerManager();
swManager.init().catch(console.error);
```

## Web App Manifest

Web App Manifest - это JSON-файл, который предоставляет информацию о веб-приложении (например, имя, иконки, цвета и т.д.) для установки на домашний экран устройства.

```json
{
  "name": "Мое Progressive Web App",
  "short_name": "MyPWA",
  "description": "Пример Progressive Web App с полной функциональностью",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "categories": ["productivity", "utilities"],
  "lang": "ru-RU",
  "dir": "ltr",
  "icons": [
    {
      "src": "/images/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/images/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "screenshots": [
    {
      "src": "/images/screenshot-1.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    },
    {
      "src": "/images/screenshot-2.png",
      "sizes": "750x1334",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ],
  "features": [
    "fast-reliable",
    "adjustable-text-size",
    "night-mode"
  ],
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.example.app",
      "id": "com.example.app"
    }
  ],
  "prefer_related_applications": false,
  "shortcuts": [
    {
      "name": "Новое сообщение",
      "short_name": "Сообщение",
      "description": "Создать новое сообщение",
      "url": "/messages/new",
      "icons": [{ "src": "/images/new-message-icon.png", "sizes": "192x192" }]
    },
    {
      "name": "Мои задачи",
      "short_name": "Задачи",
      "description": "Просмотреть мои задачи",
      "url": "/tasks",
      "icons": [{ "src": "/images/tasks-icon.png", "sizes": "192x192" }]
    }
  ],
  "protocol_handlers": [
    {
      "protocol": "web+task",
      "url": "/handle-task?uri=%s"
    }
  ]
}
```

## Оффлайн функциональность

### Оффлайн-менеджер данных

```javascript
// offline-manager.js
class OfflineManager {
    constructor() {
        this.dbName = 'PWA_Offline_DB';
        this.version = 1;
        this.db = null;
        this.pendingChanges = [];
        this.syncInProgress = false;
        this.setupDatabase();
    }
    
    async setupDatabase() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                this.db = request.result;
                resolve(this.db);
            };
            
            request.onupgradeneeded = (event) => {
                this.db = event.target.result;
                
                // Создание хранилища для оффлайн изменений
                if (!this.db.objectStoreNames.contains('pendingChanges')) {
                    const store = this.db.createObjectStore('pendingChanges', { 
                        keyPath: 'id', 
                        autoIncrement: true 
                    });
                    store.createIndex('timestamp', 'timestamp', { unique: false });
                    store.createIndex('type', 'type', { unique: false });
                }
                
                // Создание хранилища для оффлайн данных
                if (!this.db.objectStoreNames.contains('offlineData')) {
                    const store = this.db.createObjectStore('offlineData', { 
                        keyPath: 'id' 
                    });
                    store.createIndex('type', 'type', { unique: false });
                    store.createIndex('synced', 'synced', { unique: false });
                }
            };
        });
    }
    
    // Сохранение данных для оффлайн использования
    async saveOfflineData(type, data, id = null) {
        const transaction = this.db.transaction(['offlineData'], 'readwrite');
        const store = transaction.objectStore('offlineData');
        
        const record = {
            id: id || Date.now().toString(),
            type,
            data,
            synced: false,
            timestamp: Date.now()
        };
        
        return store.add(record);
    }
    
    // Получение оффлайн данных
    async getOfflineData(type, synced = false) {
        const transaction = this.db.transaction(['offlineData'], 'readonly');
        const store = transaction.objectStore('offlineData');
        const index = store.index('type');
        
        return new Promise((resolve, reject) => {
            const request = index.getAll(type);
            
            request.onsuccess = () => {
                const records = request.result;
                if (synced !== null) {
                    resolve(records.filter(r => r.synced === synced));
                } else {
                    resolve(records);
                }
            };
            
            request.onerror = () => reject(request.error);
        });
    }
    
    // Добавление оффлайн изменения
    async addPendingChange(operation, data, resourceType, resourceId = null) {
        const transaction = this.db.transaction(['pendingChanges'], 'readwrite');
        const store = transaction.objectStore('pendingChanges');
        
        const change = {
            operation, // 'create', 'update', 'delete'
            resourceType,
            resourceId,
            data,
            timestamp: Date.now(),
            status: 'pending'
        };
        
        return store.add(change);
    }
    
    // Получение ожидаемых изменений
    async getPendingChanges() {
        const transaction = this.db.transaction(['pendingChanges'], 'readonly');
        const store = transaction.objectStore('pendingChanges');
        
        return new Promise((resolve, reject) => {
            const request = store.getAll();
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }
    
    // Выполнение оффлайн синхронизации
    async syncWithServer() {
        if (this.syncInProgress) return;
        
        this.syncInProgress = true;
        
        try {
            const changes = await this.getPendingChanges();
            
            for (const change of changes) {
                try {
                    await this.executeChange(change);
                    
                    // Удаление успешно синхронизированного изменения
                    await this.removePendingChange(change.id);
                    
                    // Отметка данных как синхронизированных
                    if (change.resourceId) {
                        await this.markAsSynced(change.resourceType, change.resourceId);
                    }
                } catch (error) {
                    console.error('Ошибка синхронизации изменения:', error);
                    // Оставить изменение в очереди для следующей попытки
                    await this.markChangeAsFailed(change.id, error.message);
                }
            }
        } finally {
            this.syncInProgress = false;
        }
    }
    
    async executeChange(change) {
        const { operation, resourceType, resourceId, data } = change;
        
        let url, method, body;
        
        switch (operation) {
            case 'create':
                url = `/api/${resourceType}`;
                method = 'POST';
                body = data;
                break;
                
            case 'update':
                url = `/api/${resourceType}/${resourceId}`;
                method = 'PUT';
                body = data;
                break;
                
            case 'delete':
                url = `/api/${resourceType}/${resourceId}`;
                method = 'DELETE';
                break;
                
            default:
                throw new Error(`Неизвестная операция: ${operation}`);
        }
        
        const response = await fetch(url, {
            method,
            headers: {
                'Content-Type': 'application/json',
                'X-Requested-With': 'XMLHttpRequest'
            },
            body: body ? JSON.stringify(body) : undefined
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return response.json();
    }
    
    async removePendingChange(changeId) {
        const transaction = this.db.transaction(['pendingChanges'], 'readwrite');
        const store = transaction.objectStore('pendingChanges');
        return store.delete(changeId);
    }
    
    async markChangeAsFailed(changeId, error) {
        const transaction = this.db.transaction(['pendingChanges'], 'readwrite');
        const store = transaction.objectStore('pendingChanges');
        
        const request = store.get(changeId);
        
        request.onsuccess = () => {
            const change = request.result;
            change.status = 'failed';
            change.error = error;
            store.put(change);
        };
    }
    
    async markAsSynced(type, id) {
        const transaction = this.db.transaction(['offlineData'], 'readwrite');
        const store = transaction.objectStore('offlineData');
        
        const request = store.get(id);
        
        request.onsuccess = () => {
            const record = request.result;
            if (record) {
                record.synced = true;
                store.put(record);
            }
        };
    }
    
    // Проверка подключения и автоматическая синхронизация
    setupConnectionMonitoring() {
        window.addEventListener('online', () => {
            console.log('Соединение восстановлено, запуск синхронизации');
            this.syncWithServer();
        });
        
        window.addEventListener('offline', () => {
            console.log('Соединение потеряно, переход в оффлайн-режим');
        });
    }
}

// Использование оффлайн-менеджера
const offlineManager = new OfflineManager();
offlineManager.setupConnectionMonitoring();

// Пример использования в приложении
async function saveUserData(userData) {
    if (navigator.onLine) {
        // Прямая отправка на сервер
        await fetch('/api/users', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(userData)
        });
    } else {
        // Сохранение в оффлайн-режиме
        await offlineManager.addPendingChange('create', userData, 'users');
        await offlineManager.saveOfflineData('user', userData);
        showOfflineNotification('Данные сохранены локально, будут синхронизированы при подключении');
    }
}
```

### Оффлайн-страница

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Оффлайн - Мое PWA</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
            color: #333;
            text-align: center;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        
        .container {
            max-width: 500px;
            background: white;
            padding: 40px;
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }
        
        .icon {
            font-size: 64px;
            margin-bottom: 20px;
            color: #666;
        }
        
        h1 {
            color: #333;
            margin-bottom: 10px;
        }
        
        p {
            color: #666;
            margin-bottom: 30px;
            line-height: 1.6;
        }
        
        .status {
            margin: 20px 0;
            padding: 10px;
            background-color: #e3f2fd;
            border-radius: 5px;
            color: #1976d2;
        }
        
        .actions {
            display: flex;
            gap: 10px;
            justify-content: center;
            flex-wrap: wrap;
        }
        
        button {
            padding: 12px 24px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        
        .primary-btn {
            background-color: #1976d2;
            color: white;
        }
        
        .primary-btn:hover {
            background-color: #1565c0;
        }
        
        .secondary-btn {
            background-color: #f5f5f5;
            color: #333;
            border: 1px solid #ddd;
        }
        
        .secondary-btn:hover {
            background-color: #e0e0e0;
        }
        
        .cached-content {
            margin-top: 30px;
            text-align: left;
        }
        
        .cached-content h3 {
            margin-top: 20px;
            color: #333;
        }
        
        .cached-item {
            padding: 10px;
            margin: 5px 0;
            background-color: #f9f9f9;
            border-radius: 5px;
            border-left: 3px solid #1976d2;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="icon">📶</div>
        <h1>Вы в оффлайн-режиме</h1>
        <p>Похоже, вы не подключены к интернету. Приложение работает в оффлайн-режиме. Ваши данные будут синхронизированы, как только подключение восстановится.</p>
        
        <div class="status" id="connectionStatus">
            Статус подключения: <span id="statusText">Оффлайн</span>
        </div>
        
        <div class="actions">
            <button class="primary-btn" onclick="checkConnection()">Проверить подключение</button>
            <button class="secondary-btn" onclick="showCachedContent()">Показать сохраненные данные</button>
        </div>
        
        <div class="cached-content" id="cachedContent" style="display: none;">
            <h3>Сохраненные данные:</h3>
            <div id="cachedItems"></div>
        </div>
    </div>

    <script>
        // Проверка подключения
        async function checkConnection() {
            const statusText = document.getElementById('statusText');
            const connectionStatus = document.getElementById('connectionStatus');
            
            if (navigator.onLine) {
                statusText.textContent = 'Онлайн';
                connectionStatus.style.backgroundColor = '#e8f5e8';
                connectionStatus.style.color = '#2e7d32';
                
                // Попытка синхронизации
                try {
                    const response = await fetch('/api/ping');
                    if (response.ok) {
                        statusText.textContent = 'Подключен';
                        showNotification('Подключение восстановлено! Запуск синхронизации...');
                        
                        // Уведомить Service Worker о подключении
                        if ('serviceWorker' in navigator && navigator.serviceWorker.controller) {
                            navigator.serviceWorker.controller.postMessage({ type: 'CONNECTION_RESTORED' });
                        }
                    }
                } catch (error) {
                    statusText.textContent = 'Подключение, но сервер недоступен';
                }
            } else {
                statusText.textContent = 'Оффлайн';
                connectionStatus.style.backgroundColor = '#ffebee';
                connectionStatus.style.color = '#c62828';
            }
        }
        
        // Показать сохраненные данные
        async function showCachedContent() {
            const cachedContent = document.getElementById('cachedContent');
            const cachedItems = document.getElementById('cachedItems');
            
            try {
                // Попытка получить данные из IndexedDB
                if ('indexedDB' in window) {
                    const dbRequest = indexedDB.open('PWA_Offline_DB', 1);
                    
                    dbRequest.onsuccess = function(event) {
                        const db = event.target.result;
                        const transaction = db.transaction(['offlineData'], 'readonly');
                        const store = transaction.objectStore('offlineData');
                        const getAllRequest = store.getAll();
                        
                        getAllRequest.onsuccess = function() {
                            const data = getAllRequest.result;
                            displayCachedData(data);
                        };
                    };
                } else {
                    cachedItems.innerHTML = '<p>IndexedDB не поддерживается в этом браузере</p>';
                }
                
                cachedContent.style.display = 'block';
            } catch (error) {
                cachedItems.innerHTML = `<p>Ошибка загрузки сохраненных данных: ${error.message}</p>`;
                cachedContent.style.display = 'block';
            }
        }
        
        function displayCachedData(data) {
            const cachedItems = document.getElementById('cachedItems');
            
            if (data.length === 0) {
                cachedItems.innerHTML = '<p>Нет сохраненных данных</p>';
                return;
            }
            
            cachedItems.innerHTML = data.map(item => `
                <div class="cached-item">
                    <strong>${item.type}</strong> (ID: ${item.id})
                    <div style="margin-top: 5px; font-size: 14px; color: #666;">
                        ${JSON.stringify(item.data, null, 2)}
                    </div>
                </div>
            `).join('');
        }
        
        function showNotification(message) {
            // Простая реализация уведомления
            const notification = document.createElement('div');
            notification.style.cssText = `
                position: fixed;
                top: 20px;
                right: 20px;
                background: #4caf50;
                color: white;
                padding: 15px;
                border-radius: 5px;
                z-index: 1000;
            `;
            notification.textContent = message;
            document.body.appendChild(notification);
            
            setTimeout(() => {
                document.body.removeChild(notification);
            }, 3000);
        }
        
        // Мониторинг подключения
        window.addEventListener('online', () => {
            checkConnection();
        });
        
        window.addEventListener('offline', () => {
            document.getElementById('statusText').textContent = 'Оффлайн';
        });
        
        // Проверка подключения при загрузке
        window.addEventListener('load', checkConnection);
    </script>
</body>
</html>
```

## Push-уведомления

### Управление push-уведомлениями

```javascript
// push-manager.js
class PushManager {
    constructor() {
        this.publicKey = 'BDO_8V...'; // Ваш VAPID public key
        this.subscription = null;
        this.isSupported = 'serviceWorker' in navigator && 'PushManager' in window;
    }
    
    async initialize() {
        if (!this.isSupported) {
            throw new Error('Push-уведомления не поддерживаются в этом браузере');
        }
        
        // Регистрация Service Worker
        const registration = await navigator.serviceWorker.register('/sw.js');
        
        // Получение существующей подписки
        this.subscription = await registration.pushManager.getSubscription();
        
        return this.subscription;
    }
    
    async subscribe() {
        if (!this.isSupported) {
            throw new Error('Push-уведомления не поддерживаются');
        }
        
        const registration = await navigator.serviceWorker.ready;
        
        // Подписка на push-уведомления
        this.subscription = await registration.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: this.urlB64ToUint8Array(this.publicKey)
        });
        
        // Отправка подписки на сервер
        await this.sendSubscriptionToServer(this.subscription);
        
        return this.subscription;
    }
    
    async unsubscribe() {
        if (!this.subscription) {
            return false;
        }
        
        const success = await this.subscription.unsubscribe();
        
        if (success) {
            // Уведомление сервера об отписке
            await this.removeSubscriptionFromServer(this.subscription);
            this.subscription = null;
        }
        
        return success;
    }
    
    async sendSubscriptionToServer(subscription) {
        const response = await fetch('/api/subscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(subscription)
        });
        
        if (!response.ok) {
            throw new Error('Не удалось отправить подписку на сервер');
        }
    }
    
    async removeSubscriptionFromServer(subscription) {
        const response = await fetch('/api/unsubscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ endpoint: subscription.endpoint })
        });
        
        if (!response.ok) {
            console.error('Не удалось удалить подписку с сервера');
        }
    }
    
    // Преобразование URL-безопасного base64 в Uint8Array
    urlB64ToUint8Array(base64String) {
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
            .replace(/\-/g, '+')
            .replace(/_/g, '/');
        
        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);
        
        for (let i = 0; i < rawData.length; ++i) {
            outputArray[i] = rawData.charCodeAt(i);
        }
        
        return outputArray;
    }
    
    // Проверка разрешения на уведомления
    getNotificationPermission() {
        return Notification.permission;
    }
    
    // Запрос разрешения на уведомления
    async requestNotificationPermission() {
        if (Notification.permission === 'default') {
            return await Notification.requestPermission();
        }
        
        return Notification.permission;
    }
    
    // Отправка тестового уведомления
    async sendTestNotification() {
        if (this.subscription) {
            // В реальном приложении отправка на сервер для отправки push-уведомления
            console.log('Тестовое уведомление отправлено');
        }
    }
}

// Service Worker для обработки push-уведомлений
// В файле sw.js добавить:
/*
self.addEventListener('push', (event) => {
    const data = event.data.json();
    
    const options = {
        body: data.body,
        icon: data.icon || '/images/icon-192x192.png',
        badge: '/images/badge-72x72.png',
        tag: data.tag || 'default',
        data: data.customData || {},
        actions: data.actions || []
    };
    
    event.waitUntil(
        self.registration.showNotification(data.title, options)
    );
});

self.addEventListener('notificationclick', (event) => {
    event.notification.close();
    
    if (event.action) {
        // Обработка действий уведомления
        console.log('Действие уведомления:', event.action);
    } else {
        // Обработка клика по уведомлению
        event.waitUntil(
            self.clients.openWindow(event.notification.data.url || '/')
        );
    }
});
*/

// Использование
const pushManager = new PushManager();

async function setupPushNotifications() {
    try {
        const permission = await pushManager.requestNotificationPermission();
        
        if (permission === 'granted') {
            await pushManager.initialize();
            
            if (!pushManager.subscription) {
                await pushManager.subscribe();
                console.log('Успешно подписан на push-уведомления');
            } else {
                console.log('Уже подписан на push-уведомления');
            }
        } else {
            console.log('Разрешение на уведомления не получено:', permission);
        }
    } catch (error) {
        console.error('Ошибка настройки push-уведомлений:', error);
    }
}
```

## Кэширование ресурсов

### Продвинутая система кэширования

```javascript
// cache-manager.js
class CacheManager {
    constructor() {
        this.caches = new Map();
        this.cacheStrategies = new Map();
        this.setupDefaultStrategies();
    }
    
    setupDefaultStrategies() {
        // Стратегия "Cache First" - сначала из кэша, потом сеть
        this.cacheStrategies.set('cache-first', async (request, cacheName) => {
            const cachedResponse = await caches.match(request);
            if (cachedResponse) {
                return cachedResponse;
            }
            
            const networkResponse = await fetch(request);
            if (networkResponse.ok) {
                const responseToCache = networkResponse.clone();
                const cache = await caches.open(cacheName);
                await cache.put(request, responseToCache);
            }
            
            return networkResponse;
        });
        
        // Стратегия "Network First" - сначала сеть, потом кэш
        this.cacheStrategies.set('network-first', async (request, cacheName) => {
            try {
                const networkResponse = await fetch(request);
                if (networkResponse.ok) {
                    const responseToCache = networkResponse.clone();
                    const cache = await caches.open(cacheName);
                    await cache.put(request, responseToCache);
                    return networkResponse;
                }
            } catch (error) {
                // Если сеть недоступна, возвращаем из кэша
                const cachedResponse = await caches.match(request);
                if (cachedResponse) {
                    return cachedResponse;
                }
            }
            
            // Если ни сеть, ни кэш не доступны
            throw new Error('Сеть и кэш недоступны');
        });
        
        // Стратегия "Stale While Revalidate" - возвращаем кэш, но обновляем в фоне
        this.cacheStrategies.set('stale-while-revalidate', async (request, cacheName) => {
            const cachedResponse = await caches.match(request);
            
            // В фоне обновляем кэш
            const networkPromise = fetch(request)
                .then(async (networkResponse) => {
                    if (networkResponse.ok) {
                        const responseToCache = networkResponse.clone();
                        const cache = await caches.open(cacheName);
                        await cache.put(request, responseToCache);
                    }
                    return networkResponse;
                })
                .catch(() => cachedResponse); // Игнорируем ошибки сети при обновлении
            
            // Возвращаем кэш, если доступен, иначе сеть
            return cachedResponse || networkPromise;
        });
        
        // Стратегия "Cache Only" - только из кэша
        this.cacheStrategies.set('cache-only', async (request, cacheName) => {
            const cachedResponse = await caches.match(request);
            if (cachedResponse) {
                return cachedResponse;
            }
            throw new Error('Ресурс не найден в кэше');
        });
        
        // Стратегия "Network Only" - только сеть
        this.cacheStrategies.set('network-only', async (request) => {
            return await fetch(request);
        });
    }
    
    // Кэширование ресурсов при установке
    async cacheInstallResources(resources, cacheName) {
        const cache = await caches.open(cacheName);
        return await cache.addAll(resources);
    }
    
    // Кэширование одного ресурса
    async cacheResource(request, cacheName) {
        const response = await fetch(request);
        if (response.ok) {
            const cache = await caches.open(cacheName);
            await cache.put(request, response);
        }
        return response;
    }
    
    // Получение ресурса с использованием стратегии
    async getCachedResource(request, cacheName, strategy = 'cache-first') {
        const strategyFn = this.cacheStrategies.get(strategy);
        if (!strategyFn) {
            throw new Error(`Неизвестная стратегия кэширования: ${strategy}`);
        }
        
        return await strategyFn(request, cacheName);
    }
    
    // Удаление ресурса из кэша
    async removeFromCache(request, cacheName) {
        const cache = await caches.open(cacheName);
        return await cache.delete(request);
    }
    
    // Очистка кэша
    async clearCache(cacheName) {
        return await caches.delete(cacheName);
    }
    
    // Получение статистики кэша
    async getCacheStats(cacheName) {
        const cache = await caches.open(cacheName);
        const requests = await cache.keys();
        
        let totalSize = 0;
        
        for (const request of requests) {
            const response = await cache.match(request);
            if (response) {
                const blob = await response.blob();
                totalSize += blob.size;
            }
        }
        
        return {
            name: cacheName,
            size: requests.length,
            totalSize,
            requests: requests.map(r => r.url)
        };
    }
    
    // Получение всех кэшей
    async getAllCacheStats() {
        const cacheNames = await caches.keys();
        const stats = [];
        
        for (const name of cacheNames) {
            stats.push(await this.getCacheStats(name));
        }
        
        return stats;
    }
    
    // Предварительная загрузка ресурсов
    async preloadResources(resources, cacheName, options = {}) {
        const { concurrency = 3, timeout = 10000 } = options;
        const results = [];
        
        // Разделение ресурсов на группы для параллельной загрузки
        for (let i = 0; i < resources.length; i += concurrency) {
            const group = resources.slice(i, i + concurrency);
            const groupPromises = group.map(async (resource) => {
                try {
                    const controller = new AbortController();
                    const timeoutId = setTimeout(() => controller.abort(), timeout);
                    
                    const response = await fetch(resource, {
                        signal: controller.signal
                    });
                    
                    clearTimeout(timeoutId);
                    
                    if (response.ok) {
                        const cache = await caches.open(cacheName);
                        await cache.put(resource, response.clone());
                        return { url: resource, success: true };
                    } else {
                        return { url: resource, success: false, status: response.status };
                    }
                } catch (error) {
                    return { url: resource, success: false, error: error.message };
                }
            });
            
            const groupResults = await Promise.all(groupPromises);
            results.push(...groupResults);
        }
        
        return results;
    }
    
    // Управление жизненным циклом кэша
    async manageCacheLifecycle(cacheName, maxAge = 24 * 60 * 60 * 1000) { // 24 часа
        const cache = await caches.open(cacheName);
        const requests = await cache.keys();
        
        const now = Date.now();
        const cleanupPromises = [];
        
        for (const request of requests) {
            // Для каждого запроса нужно получить ответ и проверить дату
            // Это требует более сложной реализации, так как кэш не хранит метаданные
            // В реальном приложении лучше использовать кастомные заголовки или IndexedDB для отслеживания
        }
        
        return Promise.all(cleanupPromises);
    }
}

// Использование CacheManager
const cacheManager = new CacheManager();

// Пример использования в Service Worker
/*
self.addEventListener('fetch', (event) => {
    const url = new URL(event.request.url);
    
    if (url.pathname.startsWith('/api/')) {
        // API запросы - network-first стратегия
        event.respondWith(
            cacheManager.getCachedResource(event.request, 'api-cache', 'network-first')
        );
    } else if (url.pathname.match(/\.(png|jpg|jpeg|gif|svg|webp)$/)) {
        // Изображения - stale-while-revalidate стратегия
        event.respondWith(
            cacheManager.getCachedResource(event.request, 'images-cache', 'stale-while-revalidate')
        );
    } else {
        // Статические ресурсы - cache-first стратегия
        event.respondWith(
            cacheManager.getCachedResource(event.request, 'static-cache', 'cache-first')
        );
    }
});
*/

// Пример предварительной загрузки
async function preloadCriticalResources() {
    const criticalResources = [
        '/',
        '/styles/main.css',
        '/scripts/app.js',
        '/images/logo.png',
        '/api/config'
    ];
    
    const results = await cacheManager.preloadResources(
        criticalResources, 
        'critical-cache',
        { concurrency: 2, timeout: 5000 }
    );
    
    console.log('Результаты предварительной загрузки:', results);
}
```

## Примеры из промышленной разработки

### Комплексное PWA приложение

```javascript
// pwa-app.js - комплексное PWA приложение
class PWAApplication {
    constructor() {
        this.isPWA = false;
        this.isOnline = navigator.onLine;
        this.installPrompt = null;
        this.pushManager = null;
        this.cacheManager = null;
        this.offlineManager = null;
        
        this.setupEventListeners();
        this.initializeServices();
    }
    
    setupEventListeners() {
        // События подключения/отключения
        window.addEventListener('online', () => {
            this.isOnline = true;
            this.onConnectivityChange(true);
        });
        
        window.addEventListener('offline', () => {
            this.isOnline = false;
            this.onConnectivityChange(false);
        });
        
        // Событие установки PWA
        window.addEventListener('beforeinstallprompt', (event) => {
            event.preventDefault();
            this.installPrompt = event;
            this.showInstallButton();
        });
        
        // Событие установки
        window.addEventListener('appinstalled', () => {
            console.log('PWA установлен');
            this.isPWA = true;
            this.trackInstallEvent();
        });
    }
    
    async initializeServices() {
        try {
            // Регистрация Service Worker
            if ('serviceWorker' in navigator) {
                const registration = await navigator.serviceWorker.register('/sw.js');
                console.log('Service Worker зарегистрирован:', registration.scope);
            }
            
            // Инициализация менеджеров
            this.cacheManager = new CacheManager();
            this.offlineManager = new OfflineManager();
            this.pushManager = new PushManager();
            
            // Проверка, является ли приложение PWA
            this.isPWA = window.matchMedia('(display-mode: standalone)').matches ||
                        window.navigator.standalone ||
                        document.referrer.includes('android-app://');
                        
            console.log(this.isPWA ? 'Запущено как PWA' : 'Запущено в браузере');
        } catch (error) {
            console.error('Ошибка инициализации PWA сервисов:', error);
        }
    }
    
    onConnectivityChange(isOnline) {
        const statusElement = document.getElementById('connection-status');
        if (statusElement) {
            statusElement.textContent = isOnline ? 'Онлайн' : 'Оффлайн';
            statusElement.className = isOnline ? 'online' : 'offline';
        }
        
        if (isOnline) {
            // Попытка синхронизации оффлайн данных
            this.syncOfflineData();
        }
        
        // Уведомление других компонентов
        this.broadcastConnectivityChange(isOnline);
    }
    
    async syncOfflineData() {
        try {
            await this.offlineManager.syncWithServer();
            console.log('Оффлайн данные синхронизированы');
        } catch (error) {
            console.error('Ошибка синхронизации оффлайн данных:', error);
        }
    }
    
    broadcastConnectivityChange(isOnline) {
        // Отправка события в другие части приложения
        window.dispatchEvent(new CustomEvent('connectivitychange', {
            detail: { isOnline }
        }));
    }
    
    showInstallButton() {
        const installBtn = document.getElementById('pwa-install-btn');
        if (installBtn) {
            installBtn.style.display = 'block';
            installBtn.onclick = () => this.promptInstall();
        }
    }
    
    async promptInstall() {
        if (this.installPrompt) {
            this.installPrompt.prompt();
            
            const { outcome } = await this.installPrompt.userChoice;
            console.log(`Результат установки: ${outcome}`);
            
            this.installPrompt = null;
            this.hideInstallButton();
        }
    }
    
    hideInstallButton() {
        const installBtn = document.getElementById('pwa-install-btn');
        if (installBtn) {
            installBtn.style.display = 'none';
        }
    }
    
    // Методы для работы с приложением
    async loadInitialData() {
        try {
            if (this.isOnline) {
                // Загрузка данных с сервера
                const response = await fetch('/api/initial-data');
                const data = await response.json();
                this.cacheManager.cacheResource('/api/initial-data', 'api-cache');
                return data;
            } else {
                // Загрузка из кэша
                const cachedResponse = await caches.match('/api/initial-data');
                if (cachedResponse) {
                    return await cachedResponse.json();
                }
            }
        } catch (error) {
            console.error('Ошибка загрузки начальных данных:', error);
            // Загрузка дефолтных данных
            return this.getDefaultData();
        }
    }
    
    getDefaultData() {
        return {
            user: null,
            settings: {},
            lastUpdate: null
        };
    }
    
    // Методы для отслеживания производительности
    trackInstallEvent() {
        if ('gtag' in window) {
            gtag('event', 'install', {
                event_category: 'PWA',
                event_label: 'App Installed'
            });
        }
        
        // Отправка на собственный аналитический сервер
        fetch('/api/analytics', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                event: 'pwa_install',
                timestamp: Date.now(),
                userAgent: navigator.userAgent
            })
        }).catch(console.error);
    }
    
    // Методы для обработки ошибок
    setupErrorHandling() {
        window.addEventListener('error', (event) => {
            this.logError(event.error);
        });
        
        window.addEventListener('unhandledrejection', (event) => {
            this.logError(event.reason);
        });
    }
    
    logError(error) {
        console.error('PWA Error:', error);
        
        // Отправка ошибки на сервер для анализа
        if (this.isOnline) {
            fetch('/api/errors', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    message: error.message,
                    stack: error.stack,
                    url: window.location.href,
                    userAgent: navigator.userAgent,
                    timestamp: Date.now()
                })
            }).catch(console.error);
        } else {
            // Сохранение ошибки для отправки при восстановлении соединения
            this.offlineManager.addPendingChange(
                'create', 
                {
                    message: error.message,
                    stack: error.stack,
                    url: window.location.href,
                    timestamp: Date.now()
                }, 
                'errors'
            );
        }
    }
    
    // Методы для обновления приложения
    async checkForUpdates() {
        if ('serviceWorker' in navigator) {
            const registration = await navigator.serviceWorker.getRegistration();
            if (registration) {
                registration.update();
                
                registration.addEventListener('updatefound', () => {
                    const newWorker = registration.installing;
                    newWorker.addEventListener('statechange', () => {
                        if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                            console.log('Доступно обновление приложения');
                            this.showUpdateNotification();
                        }
                    });
                });
            }
        }
    }
    
    showUpdateNotification() {
        if (confirm('Доступно обновление приложения. Перезагрузить?')) {
            window.location.reload();
        }
    }
    
    // Инициализация приложения
    async initialize() {
        this.setupErrorHandling();
        
        // Загрузка начальных данных
        const initialData = await this.loadInitialData();
        this.updateUIWithInitialData(initialData);
        
        // Настройка push-уведомлений
        if ('PushManager' in window) {
            await this.pushManager.initialize();
        }
        
        // Проверка обновлений
        this.checkForUpdates();
        
        console.log('PWA приложение инициализировано');
    }
    
    updateUIWithInitialData(data) {
        // Обновление UI с начальными данными
        console.log('Начальные данные загружены:', data);
    }
}

// Инициализация PWA приложения
document.addEventListener('DOMContentLoaded', async () => {
    const app = new PWAApplication();
    await app.initialize();
});
```

### Страница установки PWA

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Установить PWA - Мое Приложение</title>
    <link rel="manifest" href="/manifest.json">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.6;
            color: #333;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .install-container {
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            overflow: hidden;
            max-width: 500px;
            width: 100%;
        }
        
        .install-header {
            background: #4a5568;
            color: white;
            padding: 30px;
            text-align: center;
        }
        
        .app-icon {
            width: 96px;
            height: 96px;
            background: linear-gradient(45deg, #667eea, #764ba2);
            border-radius: 24px;
            margin: 0 auto 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 48px;
        }
        
        .install-content {
            padding: 30px;
        }
        
        .feature-list {
            list-style: none;
            margin: 20px 0;
        }
        
        .feature-list li {
            padding: 10px 0;
            display: flex;
            align-items: center;
        }
        
        .feature-list li::before {
            content: "✓";
            color: #48bb78;
            font-weight: bold;
            margin-right: 10px;
        }
        
        .install-btn {
            background: linear-gradient(45deg, #667eea, #764ba2);
            color: white;
            border: none;
            padding: 15px 30px;
            border-radius: 25px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            width: 100%;
            margin: 20px 0;
            transition: transform 0.2s, box-shadow 0.2s;
        }
        
        .install-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
        }
        
        .install-btn:active {
            transform: translateY(0);
        }
        
        .browser-instructions {
            background: #f7fafc;
            border-radius: 10px;
            padding: 20px;
            margin: 20px 0;
        }
        
        .step {
            display: flex;
            align-items: flex-start;
            margin: 10px 0;
        }
        
        .step-number {
            background: #4299e1;
            color: white;
            width: 24px;
            height: 24px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            font-weight: bold;
            margin-right: 10px;
            flex-shrink: 0;
        }
        
        .connection-status {
            text-align: center;
            padding: 10px;
            margin: 10px 0;
            border-radius: 5px;
        }
        
        .online {
            background: #c6f6d5;
            color: #22543d;
        }
        
        .offline {
            background: #fed7d7;
            color: #742a2a;
        }
    </style>
</head>
<body>
    <div class="install-container">
        <div class="install-header">
            <div class="app-icon">📱</div>
            <h1>Установить приложение</h1>
            <p>Добавьте наше приложение на главный экран</p>
        </div>
        
        <div class="install-content">
            <h2>Преимущества PWA</h2>
            <ul class="feature-list">
                <li>Работает без интернета</li>
                <li>Мгновенный запуск</li>
                <li>Push-уведомления</li>
                <li>Как нативное приложение</li>
                <li>Без установки из магазина</li>
            </ul>
            
            <div id="connectionStatus" class="connection-status offline">
                Статус: Оффлайн
            </div>
            
            <button id="installBtn" class="install-btn" style="display: none;">
                Установить приложение
            </button>
            
            <div class="browser-instructions">
                <h3>Как установить:</h3>
                <div class="step">
                    <div class="step-number">1</div>
                    <div>Откройте меню браузера (три точки или звездочка)</div>
                </div>
                <div class="step">
                    <div class="step-number">2</div>
                    <div>Выберите "Установить приложение" или "Добавить на главный экран"</div>
                </div>
                <div class="step">
                    <div class="step-number">3</div>
                    <div>Подтвердите установку</div>
                </div>
            </div>
            
            <div style="text-align: center; color: #666; font-size: 14px; margin-top: 20px;">
                <p>Приложение займет менее 2 МБ места</p>
                <p>© 2025 Мое PWA Приложение. Все права защищены.</p>
            </div>
        </div>
    </div>

    <script>
        // Проверка поддержки PWA
        const supportsPWA = 'serviceWorker' in navigator && 'PushManager' in window;
        
        // Проверка статуса подключения
        function updateConnectionStatus() {
            const statusElement = document.getElementById('connectionStatus');
            const isOnline = navigator.onLine;
            
            statusElement.textContent = `Статус: ${isOnline ? 'Онлайн' : 'Оффлайн'}`;
            statusElement.className = `connection-status ${isOnline ? 'online' : 'offline'}`;
        }
        
        // Обработка установки PWA
        let installPrompt = null;
        
        window.addEventListener('beforeinstallprompt', (e) => {
            e.preventDefault();
            installPrompt = e;
            
            const installBtn = document.getElementById('installBtn');
            installBtn.style.display = 'block';
            
            installBtn.addEventListener('click', () => {
                installPrompt.prompt();
                
                installPrompt.userChoice.then((choiceResult) => {
                    if (choiceResult.outcome === 'accepted') {
                        console.log('Пользователь принял установку PWA');
                        // Перенаправление на основное приложение
                        window.location.href = '/';
                    } else {
                        console.log('Пользователь отклонил установку PWA');
                    }
                    installPrompt = null;
                    installBtn.style.display = 'none';
                });
            });
        });
        
        // Проверка, установлено ли PWA
        function isStandalonePWA() {
            return window.matchMedia('(display-mode: standalone)').matches ||
                   window.navigator.standalone === true;
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            updateConnectionStatus();
            
            // Если уже установлено как PWA, перенаправить
            if (isStandalonePWA()) {
                window.location.href = '/';
                return;
            }
            
            // Регистрация Service Worker для PWA функций
            if ('serviceWorker' in navigator) {
                navigator.serviceWorker.register('/sw.js')
                    .then(registration => {
                        console.log('SW registered: ', registration);
                    })
                    .catch(registrationError => {
                        console.log('SW registration failed: ', registrationError);
                    });
            }
            
            // Мониторинг подключения
            window.addEventListener('online', updateConnectionStatus);
            window.addEventListener('offline', updateConnectionStatus);
        });
    </script>
</body>
</html>
```

## Лучшие практики

### 1. Оптимизация производительности

```javascript
// Лучшие практики для PWA
class PWAOptimization {
    // Минимизация начального веса приложения
    static optimizeInitialLoad() {
        // Использование code splitting
        // Загрузка только критических ресурсов при запуске
        // Отложенная загрузка несущественных компонентов
    }
    
    // Эффективное кэширование
    static implementSmartCaching() {
        // Кэширование критических ресурсов при установке
        // Использование подходящих стратегий кэширования для разных типов ресурсов
        // Регулярная очистка устаревших кэшей
    }
    
    // Оптимизация изображений
    static optimizeImages() {
        // Использование современных форматов (WebP, AVIF)
        // Lazy loading изображений
        // Responsive изображения
    }
    
    // Эффективная обработка оффлайн режима
    static handleOfflineMode() {
        // Предоставление полезного оффлайн интерфейса
        // Кэширование важных данных
        // Очередь на синхронизацию изменений
    }
}
```

## Безопасность

При разработке PWA важно учитывать безопасность:

```javascript
// Безопасность PWA
class PWASecurity {
    // Проверка подлинности Service Worker
    static validateServiceWorker() {
        // Убедиться, что SW обслуживается по HTTPS
        // Проверка источника SW
    }
    
    // Безопасная обработка push-уведомлений
    static securePushNotifications(subscription, payload) {
        // Валидация подписки
        // Санитизация данных уведомления
        // Проверка источника
    }
    
    // Защита от XSS в PWA
    static preventXSS() {
        // Санитизация данных перед отображением
        // Правильное использование Content Security Policy
        // Защита кэшированных данных
    }
    
    // Безопасная синхронизация данных
    static secureDataSync(data) {
        // Шифрование чувствительных данных
        // Проверка подлинности запросов
        // Валидация данных
    }
}

// Content Security Policy для PWA
/*
Content-Security-Policy: default-src 'self';
                         connect-src 'self' https://api.example.com;
                         img-src 'self' data: https:;
                         script-src 'self' 'unsafe-inline';
                         style-src 'self' 'unsafe-inline';
*/
```

## Теги

#pwa #progressive-web-apps #service-worker #web-app-manifest #offline #push-notifications #caching #web-development #mobile-web