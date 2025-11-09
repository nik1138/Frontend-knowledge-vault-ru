# Service Workers

Service Workers - это скрипты, которые выполняются в фоне и позволяют создавать оффлайн-приложения, управлять кэшированием и реализовывать функции Push-уведомлений. Они действуют как прокси между веб-приложением, браузером и сетью.

## Основы Service Workers

### Регистрация Service Worker

```javascript
// Регистрация Service Worker в основном скрипте
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
            .then((registration) => {
                console.log('Service Worker зарегистрирован с областью:', registration.scope);
            })
            .catch((error) => {
                console.error('Service Worker не зарегистрирован:', error);
            });
    });
}

// Проверка статуса Service Worker
navigator.serviceWorker.ready
    .then((registration) => {
        console.log('Service Worker готов к работе');
    })
    .catch((error) => {
        console.error('Service Worker не готов:', error);
    });
```

```javascript
// Основной Service Worker файл (sw.js)
// Обработчик установки
self.addEventListener('install', (event) => {
    console.log('Service Worker устанавливается');
    
    // Ожидание установки
    event.waitUntil(
        caches.open('my-app-v1')
            .then((cache) => {
                return cache.addAll([
                    '/',
                    '/index.html',
                    '/styles/main.css',
                    '/scripts/app.js',
                    '/images/logo.png'
                ]);
            })
    );
});

// Обработчик активации
self.addEventListener('activate', (event) => {
    console.log('Service Worker активирован');
    
    // Очистка старых кэшей
    event.waitUntil(
        caches.keys().then((cacheNames) => {
            return Promise.all(
                cacheNames.map((cacheName) => {
                    if (cacheName !== 'my-app-v1') {
                        return caches.delete(cacheName);
                    }
                })
            );
        })
    );
});

// Обработчик сетевых запросов
self.addEventListener('fetch', (event) => {
    console.log('Service Worker перехватывает запрос:', event.request.url);
    
    event.respondWith(
        caches.match(event.request)
            .then((response) => {
                // Возврат кэшированного ответа или выполнение сетевого запроса
                return response || fetch(event.request);
            })
    );
});
```

### Жизненный цикл Service Worker

```javascript
// Полный жизненный цикл Service Worker
class ServiceWorkerLifecycle {
    // 1. Регистрация
    static register() {
        if ('serviceWorker' in navigator) {
            return navigator.serviceWorker.register('/sw.js', {
                scope: '/' // Область действия
            });
        }
        return Promise.reject('Service Worker не поддерживается');
    }
    
    // 2. Установка
    static onInstall(event) {
        console.log('Service Worker устанавливается');
        
        // Кэширование ресурсов
        event.waitUntil(
            caches.open('app-v1')
                .then((cache) => {
                    return cache.addAll([
                        '/',
                        '/offline.html',
                        '/styles/app.css',
                        '/scripts/app.js'
                    ]);
                })
        );
    }
    
    // 3. Активация
    static onActivate(event) {
        console.log('Service Worker активирован');
        
        // Очистка старых кэшей
        event.waitUntil(
            self.clients.claim() // Захват всех клиентов
        );
    }
    
    // 4. Обработка запросов
    static onFetch(event) {
        event.respondWith(
            caches.match(event.request)
                .then((cachedResponse) => {
                    // Возврат кэшированного ответа
                    if (cachedResponse) {
                        return cachedResponse;
                    }
                    
                    // Выполнение сетевого запроса
                    return fetch(event.request)
                        .then((networkResponse) => {
                            // Кэширование ответа для будущего использования
                            if (networkResponse && networkResponse.status === 200) {
                                const responseToCache = networkResponse.clone();
                                
                                caches.open('app-v1')
                                    .then((cache) => {
                                        cache.put(event.request, responseToCache);
                                    });
                            }
                            
                            return networkResponse;
                        })
                        .catch(() => {
                            // Возврат оффлайн-страницы
                            return caches.match('/offline.html');
                        });
                })
        );
    }
}

// Использование обработчиков в Service Worker
self.addEventListener('install', ServiceWorkerLifecycle.onInstall);
self.addEventListener('activate', ServiceWorkerLifecycle.onActivate);
self.addEventListener('fetch', ServiceWorkerLifecycle.onFetch);
```

## Кэширование с Service Workers

### Стратегии кэширования

```javascript
// Различные стратегии кэширования
class CacheStrategies {
    // 1. Cache First (сначала кэш, затем сеть)
    static async cacheFirst(request) {
        const cachedResponse = await caches.match(request);
        if (cachedResponse) {
            return cachedResponse;
        }
        
        const networkResponse = await fetch(request);
        if (networkResponse && networkResponse.status === 200) {
            const responseToCache = networkResponse.clone();
            const cache = await caches.open('dynamic-v1');
            await cache.put(request, responseToCache);
        }
        
        return networkResponse;
    }
    
    // 2. Network First (сначала сеть, затем кэш)
    static async networkFirst(request) {
        try {
            const networkResponse = await fetch(request);
            if (networkResponse && networkResponse.status === 200) {
                const responseToCache = networkResponse.clone();
                const cache = await caches.open('dynamic-v1');
                await cache.put(request, responseToCache);
            }
            return networkResponse;
        } catch (error) {
            const cachedResponse = await caches.match(request);
            if (cachedResponse) {
                return cachedResponse;
            }
            throw error;
        }
    }
    
    // 3. Cache Only (только кэш)
    static async cacheOnly(request) {
        const cachedResponse = await caches.match(request);
        if (cachedResponse) {
            return cachedResponse;
        }
        throw new Error('Ресурс не найден в кэше');
    }
    
    // 4. Network Only (только сеть)
    static async networkOnly(request) {
        return await fetch(request);
    }
    
    // 5. Stale While Revalidate (старый сначала, обновление в фоне)
    static async staleWhileRevalidate(request) {
        // Возврат кэшированного ответа (даже если он устарел)
        const cachedResponse = await caches.match(request);
        
        // Обновление кэша в фоне
        const networkResponsePromise = fetch(request)
            .then(async (networkResponse) => {
                if (networkResponse && networkResponse.status === 200) {
                    const responseToCache = networkResponse.clone();
                    const cache = await caches.open('dynamic-v1');
                    await cache.put(request, responseToCache);
                }
                return networkResponse;
            });
        
        // Возврат кэшированного ответа, если он есть, иначе сетевой
        return cachedResponse || networkResponsePromise;
    }
}

// Использование стратегий в Service Worker
self.addEventListener('fetch', (event) => {
    const url = new URL(event.request.url);
    
    if (url.pathname.startsWith('/api/')) {
        // Для API запросов используем Network First
        event.respondWith(CacheStrategies.networkFirst(event.request));
    } else if (url.pathname.match(/\.(jpg|jpeg|png|gif|svg)$/)) {
        // Для изображений используем Cache First
        event.respondWith(CacheStrategies.cacheFirst(event.request));
    } else {
        // Для других ресурсов используем Stale While Revalidate
        event.respondWith(CacheStrategies.staleWhileRevalidate(event.request));
    }
});
```

### Управление кэшами

```javascript
// Класс для управления кэшами
class CacheManager {
    constructor() {
        this.staticCacheName = 'static-v1';
        this.dynamicCacheName = 'dynamic-v1';
        this.apiCacheName = 'api-v1';
    }
    
    // Инициализация кэширования
    async initialize() {
        await this.createStaticCache();
        await this.cleanupOldCaches();
    }
    
    // Создание статического кэша
    async createStaticCache() {
        const cache = await caches.open(this.staticCacheName);
        await cache.addAll([
            '/',
            '/index.html',
            '/styles/main.css',
            '/scripts/app.js',
            '/images/logo.png',
            '/manifest.json'
        ]);
    }
    
    // Очистка старых кэшей
    async cleanupOldCaches() {
        const cacheNames = await caches.keys();
        const validCacheNames = [
            this.staticCacheName,
            this.dynamicCacheName,
            this.apiCacheName
        ];
        
        return Promise.all(
            cacheNames
                .filter(name => !validCacheNames.includes(name))
                .map(name => caches.delete(name))
        );
    }
    
    // Кэширование динамического контента
    async cacheDynamicContent(request, response) {
        if (response.ok) {
            const cache = await caches.open(this.dynamicCacheName);
            await cache.put(request, response.clone());
        }
    }
    
    // Получение из кэша
    async getCachedResponse(request) {
        // Проверка во всех кэшах
        const cacheNames = await caches.keys();
        
        for (const cacheName of cacheNames) {
            const response = await caches.match(request, { cacheName });
            if (response) {
                return response;
            }
        }
        
        return null;
    }
    
    // Удаление из кэша
    async removeFromCache(request) {
        const cacheNames = await caches.keys();
        
        const promises = cacheNames.map(cacheName => 
            caches.delete(cacheName).then(() => 
                caches.open(cacheName).then(cache => 
                    cache.delete(request)
                )
            )
        );
        
        await Promise.all(promises);
    }
    
    // Очистка всех кэшей
    async clearAllCaches() {
        const cacheNames = await caches.keys();
        await Promise.all(cacheNames.map(name => caches.delete(name)));
    }
    
    // Получение информации о кэшах
    async getCacheInfo() {
        const cacheNames = await caches.keys();
        const info = {};
        
        for (const name of cacheNames) {
            const cache = await caches.open(name);
            const requests = await cache.keys();
            info[name] = {
                requestCount: requests.length,
                size: await this.estimateCacheSize(name)
            };
        }
        
        return info;
    }
    
    // Оценка размера кэша
    async estimateCacheSize(cacheName) {
        // Оценка размера кэша (приблизительно)
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
        
        return totalSize;
    }
}

// Использование в Service Worker
const cacheManager = new CacheManager();

self.addEventListener('install', (event) => {
    event.waitUntil(cacheManager.initialize());
});
```

## Оффлайн функциональность

### Оффлайн страница

```javascript
// Создание оффлайн страницы
function createOfflinePage() {
    return new Response(`
        <!DOCTYPE html>
        <html>
        <head>
            <title>Оффлайн</title>
            <style>
                body { 
                    font-family: Arial, sans-serif; 
                    text-align: center; 
                    padding: 50px; 
                    background-color: #f5f5f5;
                }
                .offline-icon { 
                    font-size: 48px; 
                    margin-bottom: 20px; 
                }
            </style>
        </head>
        <body>
            <div class="offline-icon">📱</div>
            <h1>Вы в оффлайн-режиме</h1>
            <p>Подключитесь к интернету, чтобы получить доступ к полному функционалу.</p>
            <button onclick="window.location.reload()">Проверить подключение</button>
        </body>
        </html>
    `, {
        headers: { 'Content-Type': 'text/html' }
    });
}

// Кэширование оффлайн страницы
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open('offline-v1')
            .then((cache) => {
                return cache.put('/offline.html', createOfflinePage());
            })
    );
});
```

### Оффлайн данные

```javascript
// Управление оффлайн данными
class OfflineDataManager {
    constructor() {
        this.storageKey = 'offline-data';
    }
    
    // Сохранение данных для оффлайн использования
    async saveOfflineData(data) {
        try {
            const existingData = await this.getOfflineData();
            const updatedData = { ...existingData, ...data };
            
            if ('storage' in navigator && 'setStorage' in navigator.storage) {
                // Использование Storage API если доступно
                await navigator.storage.setStorage(updatedData);
            } else {
                // Резервный вариант с localStorage
                localStorage.setItem(this.storageKey, JSON.stringify(updatedData));
            }
        } catch (error) {
            console.error('Ошибка сохранения оффлайн данных:', error);
        }
    }
    
    // Получение оффлайн данных
    async getOfflineData() {
        try {
            if ('storage' in navigator && 'getStorage' in navigator.storage) {
                return await navigator.storage.getStorage();
            } else {
                const data = localStorage.getItem(this.storageKey);
                return data ? JSON.parse(data) : {};
            }
        } catch (error) {
            console.error('Ошибка получения оффлайн данных:', error);
            return {};
        }
    }
    
    // Синхронизация данных при возвращении в онлайн
    async syncOfflineData() {
        const offlineData = await this.getOfflineData();
        
        if (Object.keys(offlineData).length === 0) {
            return; // Нет данных для синхронизации
        }
        
        try {
            // Отправка оффлайн данных на сервер
            const response = await fetch('/api/sync', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(offlineData)
            });
            
            if (response.ok) {
                // Очистка оффлайн данных после успешной синхронизации
                localStorage.removeItem(this.storageKey);
                console.log('Оффлайн данные синхронизированы');
            }
        } catch (error) {
            console.error('Ошибка синхронизации оффлайн данных:', error);
        }
    }
}

// Использование в Service Worker
const offlineManager = new OfflineDataManager();

// Обработка оффлайн запросов
self.addEventListener('fetch', async (event) => {
    if (event.request.method !== 'GET') {
        // Для не-GET запросов сохраняем в оффлайн хранилище
        event.respondWith(
            fetch(event.request)
                .catch(async () => {
                    // Сохраняем запрос для последующей синхронизации
                    const clonedRequest = event.request.clone();
                    const body = await clonedRequest.text();
                    
                    await offlineManager.saveOfflineData({
                        [event.request.url]: {
                            method: event.request.method,
                            body: body,
                            timestamp: Date.now()
                        }
                    });
                    
                    return new Response('Запрос сохранен для синхронизации', { 
                        status: 200 
                    });
                })
        );
    }
});
```

## Push-уведомления

### Регистрация для Push-уведомлений

```javascript
// Класс для управления Push-уведомлениями
class PushNotificationManager {
    constructor() {
        this.publicKey = 'YOUR_VAPID_PUBLIC_KEY';
    }
    
    // Подписка на Push-уведомления
    async subscribe() {
        try {
            const registration = await navigator.serviceWorker.ready;
            
            // Проверка разрешения на уведомления
            const permission = await Notification.requestPermission();
            if (permission !== 'granted') {
                throw new Error('Разрешение на уведомления не получено');
            }
            
            // Подписка к Push-серверу
            const subscription = await registration.pushManager.subscribe({
                userVisibleOnly: true,
                applicationServerKey: this.urlBase64ToUint8Array(this.publicKey)
            });
            
            // Отправка подписки на сервер
            await this.sendSubscriptionToServer(subscription);
            
            return subscription;
        } catch (error) {
            console.error('Ошибка подписки на Push-уведомления:', error);
            throw error;
        }
    }
    
    // Отмена подписки
    async unsubscribe() {
        try {
            const registration = await navigator.serviceWorker.ready;
            const subscription = await registration.pushManager.getSubscription();
            
            if (subscription) {
                await subscription.unsubscribe();
                await this.removeSubscriptionFromServer(subscription.endpoint);
            }
        } catch (error) {
            console.error('Ошибка отмены подписки:', error);
            throw error;
        }
    }
    
    // Отправка подписки на сервер
    async sendSubscriptionToServer(subscription) {
        const response = await fetch('/api/subscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(subscription)
        });
        
        if (!response.ok) {
            throw new Error('Ошибка отправки подписки на сервер');
        }
    }
    
    // Удаление подписки с сервера
    async removeSubscriptionFromServer(endpoint) {
        await fetch('/api/unsubscribe', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ endpoint })
        });
    }
    
    // Преобразование VAPID ключа
    urlBase64ToUint8Array(base64String) {
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
    
    // Проверка статуса подписки
    async isSubscribed() {
        const registration = await navigator.serviceWorker.ready;
        const subscription = await registration.pushManager.getSubscription();
        return !!subscription;
    }
}

// Использование в основном скрипте
const pushManager = new PushNotificationManager();

document.getElementById('subscribe-btn').addEventListener('click', async () => {
    try {
        await pushManager.subscribe();
        console.log('Успешно подписан на Push-уведомления');
    } catch (error) {
        console.error('Ошибка подписки:', error);
    }
});
```

```javascript
// Обработка Push-уведомлений в Service Worker
self.addEventListener('push', (event) => {
    let payload = {};
    
    if (event.data) {
        payload = event.data.json();
    }
    
    const options = {
        body: payload.body || 'Новое уведомление',
        icon: payload.icon || '/images/icon-192x192.png',
        badge: payload.badge || '/images/badge-72x72.png',
        tag: payload.tag || 'default',
        data: payload.data || {},
        actions: payload.actions || [],
        vibrate: payload.vibrate || [200, 100, 200]
    };
    
    event.waitUntil(
        self.registration.showNotification(payload.title || 'Уведомление', options)
    );
});

// Обработка клика по уведомлению
self.addEventListener('notificationclick', (event) => {
    event.notification.close();
    
    if (event.action === 'open') {
        // Обработка действия "открыть"
        event.waitUntil(
            clients.openWindow(event.notification.data.url || '/')
        );
    } else {
        // Обработка основного клика по уведомлению
        event.waitUntil(
            clients.openWindow(event.notification.data.url || '/')
        );
    }
});
```

## Background Sync

### Фоновая синхронизация

```javascript
// Класс для фоновой синхронизации
class BackgroundSyncManager {
    constructor() {
        this.syncTag = 'background-sync';
    }
    
    // Регистрация фоновой синхронизации
    async registerSync() {
        try {
            const registration = await navigator.serviceWorker.ready;
            
            // Проверка поддержки Background Sync
            if ('sync' in registration) {
                await registration.sync.register(this.syncTag);
                console.log('Фоновая синхронизация зарегистрирована');
            } else {
                console.warn('Background Sync не поддерживается');
            }
        } catch (error) {
            console.error('Ошибка регистрации фоновой синхронизации:', error);
        }
    }
    
    // Выполнение синхронизации при возвращении в онлайн
    async performSync() {
        // Получение оффлайн данных для синхронизации
        const offlineData = await this.getOfflineData();
        
        if (Object.keys(offlineData).length === 0) {
            return; // Нет данных для синхронизации
        }
        
        try {
            // Отправка всех оффлайн данных
            for (const [url, data] of Object.entries(offlineData)) {
                await fetch(url, {
                    method: data.method,
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: data.body
                });
            }
            
            // Очистка оффлайн данных после успешной синхронизации
            await this.clearOfflineData();
            console.log('Фоновая синхронизация завершена');
        } catch (error) {
            console.error('Ошибка фоновой синхронизации:', error);
            // В случае ошибки повторная попытка будет сделана автоматически
        }
    }
    
    // Получение оффлайн данных
    async getOfflineData() {
        if (localStorage.getItem('offline-requests')) {
            return JSON.parse(localStorage.getItem('offline-requests'));
        }
        return {};
    }
    
    // Очистка оффлайн данных
    async clearOfflineData() {
        localStorage.removeItem('offline-requests');
    }
    
    // Сохранение запроса для фоновой синхронизации
    async saveForSync(url, method, body) {
        const offlineData = await this.getOfflineData();
        offlineData[url] = {
            method: method,
            body: body,
            timestamp: Date.now()
        };
        
        localStorage.setItem('offline-requests', JSON.stringify(offlineData));
    }
}

// Использование в Service Worker
const bgSyncManager = new BackgroundSyncManager();

// Обработка события синхронизации
self.addEventListener('sync', async (event) => {
    if (event.tag === bgSyncManager.syncTag) {
        event.waitUntil(bgSyncManager.performSync());
    }
});

// Сохранение запросов для фоновой синхронизации
self.addEventListener('fetch', async (event) => {
    if (event.request.method !== 'GET' && event.request.url.includes('/api/')) {
        event.respondWith(
            fetch(event.request.clone())
                .catch(async () => {
                    // Сохраняем запрос для фоновой синхронизации
                    const clonedRequest = event.request.clone();
                    const body = await clonedRequest.text();
                    
                    await bgSyncManager.saveForSync(
                        event.request.url,
                        event.request.method,
                        body
                    );
                    
                    // Регистрируем фоновую синхронизацию
                    await bgSyncManager.registerSync();
                    
                    return new Response('Запрос сохранен для синхронизации', { 
                        status: 200 
                    });
                })
        );
    }
});
```

## Продвинутые возможности

### Обновление Service Worker

```javascript
// Управление обновлением Service Worker
class ServiceWorkerUpdater {
    constructor() {
        this.updateInterval = 1000 * 60 * 60; // 1 час
    }
    
    // Проверка обновлений
    async checkForUpdates() {
        if ('serviceWorker' in navigator) {
            const registration = await navigator.serviceWorker.getRegistration();
            
            if (registration) {
                // Принудительная проверка обновлений
                registration.update();
                
                registration.addEventListener('updatefound', () => {
                    const newWorker = registration.installing;
                    
                    newWorker.addEventListener('statechange', () => {
                        if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
                            // Доступно обновление
                            this.showUpdateNotification();
                        }
                    });
                });
            }
        }
    }
    
    // Показ уведомления об обновлении
    showUpdateNotification() {
        if (confirm('Доступно обновление приложения. Перезагрузить?')) {
            this.updateApp();
        }
    }
    
    // Обновление приложения
    async updateApp() {
        if ('serviceWorker' in navigator) {
            const registration = await navigator.serviceWorker.getRegistration();
            
            if (registration && registration.waiting) {
                // Отправка сообщения активному Service Worker для перезагрузки
                registration.waiting.postMessage({ type: 'SKIP_WAITING' });
                
                // Перезагрузка страницы после обновления
                navigator.serviceWorker.addEventListener('controllerchange', () => {
                    window.location.reload();
                });
            }
        }
    }
    
    // Автоматическая проверка обновлений
    startAutoUpdateCheck() {
        setInterval(() => {
            this.checkForUpdates();
        }, this.updateInterval);
    }
}

// Использование
const updater = new ServiceWorkerUpdater();
updater.startAutoUpdateCheck();

// Обработка сообщений от Service Worker в основном скрипте
navigator.serviceWorker.addEventListener('message', (event) => {
    if (event.data && event.data.type === 'UPDATE_AVAILABLE') {
        updater.showUpdateNotification();
    }
});
```

```javascript
// Обработка обновлений в Service Worker
self.addEventListener('message', (event) => {
    if (event.data && event.data.type === 'SKIP_WAITING') {
        self.skipWaiting();
    }
});

// Обработка установки нового Service Worker
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open('new-version-v2')
            .then((cache) => {
                return cache.addAll([
                    // Новые ресурсы
                ]);
            })
            .then(() => {
                // Уведомление основного скрипта о доступности обновления
                if (self.clients) {
                    self.clients.matchAll().then((clients) => {
                        clients.forEach((client) => {
                            client.postMessage({ 
                                type: 'UPDATE_AVAILABLE',
                                message: 'Доступно обновление приложения'
                            });
                        });
                    });
                }
            })
    );
});
```

### Коммуникация с основным приложением

```javascript
// Класс для коммуникации между Service Worker и основным приложением
class ServiceWorkerCommunication {
    constructor() {
        this.messageHandlers = new Map();
    }
    
    // Отправка сообщения в Service Worker
    async sendMessage(message) {
        const registration = await navigator.serviceWorker.ready;
        return new Promise((resolve, reject) => {
            const messageChannel = new MessageChannel();
            messageChannel.port1.onmessage = (event) => {
                if (event.data.error) {
                    reject(new Error(event.data.error));
                } else {
                    resolve(event.data.result);
                }
            };
            
            registration.active.postMessage(message, [messageChannel.port2]);
        });
    }
    
    // Получение сообщения от Service Worker
    listenForMessages() {
        navigator.serviceWorker.addEventListener('message', (event) => {
            const { type, data } = event.data;
            
            if (this.messageHandlers.has(type)) {
                this.messageHandlers.get(type)(data);
            }
        });
    }
    
    // Подписка на типы сообщений
    onMessage(type, handler) {
        this.messageHandlers.set(type, handler);
    }
    
    // Отписка от сообщений
    offMessage(type) {
        this.messageHandlers.delete(type);
    }
}

// Использование
const swComm = new ServiceWorkerCommunication();
swComm.listenForMessages();

// Подписка на сообщения
swComm.onMessage('CACHE_INFO', (info) => {
    console.log('Информация о кэше:', info);
});

swComm.onMessage('SYNC_STATUS', (status) => {
    console.log('Статус синхронизации:', status);
});

// Отправка сообщения в Service Worker
async function getCacheInfo() {
    try {
        const cacheInfo = await swComm.sendMessage({
            type: 'GET_CACHE_INFO'
        });
        console.log('Информация о кэше:', cacheInfo);
    } catch (error) {
        console.error('Ошибка получения информации о кэше:', error);
    }
}
```

```javascript
// Обработка сообщений в Service Worker
self.addEventListener('message', async (event) => {
    const { type, data } = event.data;
    
    switch (type) {
        case 'GET_CACHE_INFO':
            const cacheInfo = await getCacheInfo();
            event.ports[0].postMessage({ result: cacheInfo });
            break;
            
        case 'CLEAR_CACHE':
            await clearAllCaches();
            event.ports[0].postMessage({ result: 'Кэш очищен' });
            break;
            
        case 'SYNC_NOW':
            await performImmediateSync();
            event.ports[0].postMessage({ result: 'Синхронизация завершена' });
            break;
            
        default:
            event.ports[0].postMessage({ error: `Неизвестный тип сообщения: ${type}` });
    }
});

// Вспомогательные функции для Service Worker
async function getCacheInfo() {
    const cacheNames = await caches.keys();
    const info = {};
    
    for (const name of cacheNames) {
        const cache = await caches.open(name);
        const requests = await cache.keys();
        info[name] = requests.length;
    }
    
    return info;
}

async function clearAllCaches() {
    const cacheNames = await caches.keys();
    await Promise.all(cacheNames.map(name => caches.delete(name)));
}

async function performImmediateSync() {
    // Выполнение немедленной синхронизации
    // Реализация зависит от конкретных требований
    return 'Синхронизация выполнена';
}
```

## Лучшие практики

### 1. Обработка ошибок

```javascript
// Централизованная обработка ошибок в Service Worker
class ServiceWorkerErrorHandler {
    static async handleRequest(request, handler) {
        try {
            return await handler(request);
        } catch (error) {
            console.error('Ошибка обработки запроса:', error);
            
            // Попытка вернуть резервный ответ
            return await this.getFallbackResponse(request, error);
        }
    }
    
    static async getFallbackResponse(request, error) {
        // Попытка вернуть кэшированный ответ
        const cachedResponse = await caches.match(request);
        if (cachedResponse) {
            return cachedResponse;
        }
        
        // Попытка вернуть оффлайн страницу для HTML запросов
        if (request.headers.get('Accept').includes('text/html')) {
            const offlineResponse = await caches.match('/offline.html');
            if (offlineResponse) {
                return offlineResponse;
            }
        }
        
        // Возврат ошибки
        return new Response('Сервер недоступен', { 
            status: 503, 
            statusText: 'Service Unavailable' 
        });
    }
    
    static logError(error, context = '') {
        // Отправка ошибки на сервер для анализа
        if ('sendBeacon' in navigator) {
            navigator.sendBeacon('/api/errors', JSON.stringify({
                error: error.toString(),
                context: context,
                url: self.location.href,
                timestamp: new Date().toISOString()
            }));
        }
    }
}
```

### 2. Управление версиями кэша

```javascript
// Управление версиями кэша
class CacheVersionManager {
    constructor() {
        this.currentVersion = 'v1.2.3';
        this.cachePrefix = 'app-cache';
    }
    
    getCacheName() {
        return `${this.cachePrefix}-${this.currentVersion}`;
    }
    
    async cleanupOldVersions() {
        const cacheNames = await caches.keys();
        const currentCacheName = this.getCacheName();
        
        const oldCaches = cacheNames.filter(name => 
            name.startsWith(this.cachePrefix) && name !== currentCacheName
        );
        
        await Promise.all(oldCaches.map(name => caches.delete(name)));
    }
    
    async updateCache() {
        await this.cleanupOldVersions();
        
        const cache = await caches.open(this.getCacheName());
        await cache.addAll([
            // Ресурсы для текущей версии
        ]);
    }
}
```

Service Workers предоставляют мощные возможности для создания оффлайн-приложений, управления кэшированием и реализации Push-уведомлений. При правильной реализации они могут значительно улучшить пользовательский опыт и производительность веб-приложений.

#javascript #service-workers #pwa #offline #caching #frontend #web-development