---
aliases: [Сервис-воркеры, Service Worker, SW, Background Workers]
tags: [javascript, web-api, pwa, caching, offline, frontend, performance]
---

# Service Workers

## Обзор

Service Workers - это скрипты, запускаемые браузером в фоновом режиме, отдельно от веб-страницы. Они действуют как прокси-сервер между веб-приложением, браузером и сетью, позволяя реализовать функции офлайн-работы, кэширования ресурсов, push-уведомлений и другие возможности Progressive Web Apps (PWA).

## Основы Service Workers

### Регистрация Service Worker

```javascript
// main.js
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker.register('/sw.js')
            .then(registration => {
                console.log('Service Worker зарегистрирован с областью:', registration.scope);
            })
            .catch(error => {
                console.error('Регистрация Service Worker не удалась:', error);
            });
    });
}
```

```javascript
// sw.js (Service Worker скрипт)
const CACHE_NAME = 'my-app-v1';
const urlsToCache = [
    '/',
    '/styles/main.css',
    '/scripts/main.js',
    '/images/logo.png',
    '/offline.html'
];

// Событие установки
self.addEventListener('install', event => {
    console.log('Service Worker: Установка');
    
    // Захватываем событие установки для кэширования ресурсов
    event.waitUntil(
        caches.open(CACHE_NAME)
            .then(cache => {
                console.log('Service Worker: Кэширование файлов');
                return cache.addAll(urlsToCache);
            })
            .catch(error => {
                console.error('Service Worker: Ошибка кэширования:', error);
            })
    );
});

// Событие активации
self.addEventListener('activate', event => {
    console.log('Service Worker: Активация');
    
    // Очистка старых кэшей
    event.waitUntil(
        caches.keys().then(cacheNames => {
            return Promise.all(
                cacheNames.map(cacheName => {
                    if (cacheName !== CACHE_NAME) {
                        console.log('Service Worker: Удаление старого кэша', cacheName);
                        return caches.delete(cacheName);
                    }
                })
            );
        })
    );
});

// Событие fetch (перехват сетевых запросов)
self.addEventListener('fetch', event => {
    console.log('Service Worker: Перехват запроса', event.request.url);
    
    event.respondWith(
        caches.match(event.request)
            .then(response => {
                // Возвращаем кэшированный ответ, если он есть
                if (response) {
                    return response;
                }
                
                // Иначе делаем сетевой запрос
                return fetch(event.request);
            })
            .catch(() => {
                // Возвращаем офлайн-страницу при ошибке сети
                return caches.match('/offline.html');
            })
    );
});
```

## Жизненный цикл Service Worker

Service Worker проходит через несколько состояний:

1. **Registration** - регистрация в основном потоке
2. **Installation** - установка (событие `install`)
3. **Waiting** - ожидание активации (если уже есть активный SW)
4. **Active** - активация (событие `activate`)
5. **Controlling** - управление страницами (событие `fetch` и другие)

```javascript
// sw.js - подробная обработка жизненного цикла
self.addEventListener('install', event => {
    console.log('Service Worker: Установка начата');
    
    // Устанавливаем воркер в режим ожидания до завершения установки
    event.waitUntil(
        caches.open('my-cache-v1')
            .then(cache => cache.addAll([
                '/',
                '/styles.css',
                '/app.js',
                '/offline.html'
            ]))
            .then(() => {
                console.log('Service Worker: Установка завершена');
                // Пропускаем стадию ожидания и активируем сразу
                self.skipWaiting();
            })
    );
});

self.addEventListener('activate', event => {
    console.log('Service Worker: Активация начата');
    
    // Запускаем новый воркер немедленно
    event.waitUntil(
        self.clients.claim() // Принимаем контроль над всеми клиентами
    );
});
```

## Кэширование ресурсов

### Стратегии кэширования

#### 1. Cache First (Кэш в первую очередь)

```javascript
// sw.js
self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request)
            .then(cachedResponse => {
                // Возвращаем кэш, если есть
                if (cachedResponse) {
                    return cachedResponse;
                }
                
                // Иначе делаем сетевой запрос и кэшируем результат
                return fetch(event.request)
                    .then(networkResponse => {
                        // Кэшируем только GET запросы и успешные ответы
                        if (event.request.method === 'GET' && 
                            networkResponse.status === 200 &&
                            networkResponse.type === 'basic') {
                            
                            const responseToCache = networkResponse.clone();
                            caches.open('dynamic-cache-v1')
                                .then(cache => {
                                    cache.put(event.request, responseToCache);
                                });
                        }
                        
                        return networkResponse;
                    });
            })
    );
});
```

#### 2. Network First (Сеть в первую очередь)

```javascript
// sw.js
self.addEventListener('fetch', event => {
    event.respondWith(
        fetch(event.request)
            .then(networkResponse => {
                // Кэшируем успешный ответ
                if (networkResponse.status === 200) {
                    const responseToCache = networkResponse.clone();
                    caches.open('dynamic-cache-v1')
                        .then(cache => {
                            cache.put(event.request, responseToCache);
                        });
                }
                
                return networkResponse;
            })
            .catch(() => {
                // Если сеть недоступна, возвращаем кэш
                return caches.match(event.request)
                    .then(cachedResponse => {
                        return cachedResponse || caches.match('/offline.html');
                    });
            })
    );
});
```

#### 3. Cache with Network Fallback (Кэш с сетевым фолбэком)

```javascript
// sw.js
self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request)
            .then(cachedResponse => {
                if (cachedResponse) {
                    // Обновляем кэш в фоне
                    fetch(event.request)
                        .then(networkResponse => {
                            if (networkResponse.status === 200) {
                                const responseToCache = networkResponse.clone();
                                caches.open('dynamic-cache-v1')
                                    .then(cache => {
                                        cache.put(event.request, responseToCache);
                                    });
                            }
                        });
                    
                    return cachedResponse;
                }
                
                return fetch(event.request);
            })
    );
});
```

## Продвинутые возможности

### Кэширование API-ответов

```javascript
// sw.js
self.addEventListener('fetch', event => {
    // Обработка только GET запросов к API
    if (event.request.method === 'GET' && 
        event.request.url.includes('/api/')) {
        
        event.respondWith(
            caches.open('api-cache-v1')
                .then(cache => {
                    return cache.match(event.request)
                        .then(cachedResponse => {
                            // Сетевой запрос для обновления кэша
                            const networkFetch = fetch(event.request)
                                .then(networkResponse => {
                                    // Кэшируем только успешные ответы
                                    if (networkResponse.status === 200) {
                                        const responseToCache = networkResponse.clone();
                                        cache.put(event.request, responseToCache);
                                    }
                                    return networkResponse;
                                })
                                .catch(() => {
                                    // Если сеть недоступна, возвращаем кэш
                                    return cachedResponse;
                                });
                            
                            // Возвращаем кэш, если он есть, иначе сетевой запрос
                            return cachedResponse || networkFetch;
                        });
                })
        );
    }
});
```

### Динамическое кэширование изображений

```javascript
// sw.js
self.addEventListener('fetch', event => {
    // Обработка изображений
    if (event.request.destination === 'image') {
        event.respondWith(
            caches.match(event.request)
                .then(cachedResponse => {
                    if (cachedResponse) {
                        return cachedResponse;
                    }
                    
                    return fetch(event.request)
                        .then(response => {
                            if (response.status === 200) {
                                const responseToCache = response.clone();
                                
                                // Кэшируем изображение в отдельном кэше
                                caches.open('images-cache-v1')
                                    .then(cache => {
                                        cache.put(event.request, responseToCache);
                                    });
                            }
                            
                            return response;
                        })
                        .catch(() => {
                            // Возвращаем плейсхолдер при ошибке
                            return caches.match('/images/placeholder.png');
                        });
                })
        );
    }
});
```

### Кастомная логика кэширования

```javascript
// sw.js
class CacheManager {
    constructor() {
        this.staticCacheName = 'static-v1';
        this.dynamicCacheName = 'dynamic-v1';
        this.apiCacheName = 'api-v1';
    }
    
    async cacheRequest(request, response, cacheName) {
        if (request.method !== 'GET' || response.status !== 200) {
            return;
        }
        
        const cache = await caches.open(cacheName);
        await cache.put(request, response.clone());
    }
    
    async getResponse(request) {
        // Проверяем статический кэш
        let response = await caches.match(request, {
            cacheName: this.staticCacheName
        });
        
        if (response) {
            return response;
        }
        
        // Проверяем динамический кэш
        response = await caches.match(request, {
            cacheName: this.dynamicCacheName
        });
        
        if (response) {
            return response;
        }
        
        // Проверяем API кэш
        response = await caches.match(request, {
            cacheName: this.apiCacheName
        });
        
        return response;
    }
    
    async shouldCache(request, response) {
        // Не кэшируем POST запросы
        if (request.method !== 'GET') {
            return false;
        }
        
        // Не кэшируем ошибки
        if (response.status !== 200) {
            return false;
        }
        
        // Не кэшируем запросы к другим доменам
        if (new URL(request.url).origin !== self.location.origin) {
            return false;
        }
        
        return true;
    }
}

const cacheManager = new CacheManager();

self.addEventListener('fetch', async event => {
    event.respondWith(
        (async () => {
            const cachedResponse = await cacheManager.getResponse(event.request);
            
            if (cachedResponse) {
                // Обновляем кэш в фоне
                fetch(event.request)
                    .then(networkResponse => {
                        if (cacheManager.shouldCache(event.request, networkResponse)) {
                            cacheManager.cacheRequest(
                                event.request,
                                networkResponse,
                                getCacheNameForRequest(event.request)
                            );
                        }
                    });
                
                return cachedResponse;
            }
            
            // Делаем сетевой запрос
            const networkResponse = await fetch(event.request);
            
            if (cacheManager.shouldCache(event.request, networkResponse)) {
                cacheManager.cacheRequest(
                    event.request,
                    networkResponse,
                    getCacheNameForRequest(event.request)
                );
            }
            
            return networkResponse;
        })()
    );
});

function getCacheNameForRequest(request) {
    if (request.url.includes('/api/')) {
        return cacheManager.apiCacheName;
    } else if (isStaticAsset(request)) {
        return cacheManager.staticCacheName;
    } else {
        return cacheManager.dynamicCacheName;
    }
}

function isStaticAsset(request) {
    const url = new URL(request.url);
    return /\.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$/i.test(url.pathname);
}
```

## Push-уведомления

### Регистрация для push-уведомлений

```javascript
// main.js
if ('serviceWorker' in navigator && 'PushManager' in window) {
    navigator.serviceWorker.register('/sw.js')
        .then(registration => {
            return registration.pushManager.subscribe({
                userVisibleOnly: true,
                applicationServerKey: urlB64ToUint8Array('YOUR_PUBLIC_VAPID_KEY')
            });
        })
        .then(subscription => {
            console.log('Подписка на push-уведомления:', subscription);
            
            // Отправляем подписку на сервер
            sendSubscriptionToServer(subscription);
        })
        .catch(error => {
            console.error('Ошибка подписки на push:', error);
        });
}

function urlB64ToUint8Array(base64String) {
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
```

```javascript
// sw.js
self.addEventListener('push', event => {
    const payload = event.data ? event.data.text() : 'Нет данных';
    
    event.waitUntil(
        self.registration.showNotification('Новое уведомление', {
            body: payload,
            icon: '/images/icon-192x192.png',
            badge: '/images/badge-72x72.png',
            vibrate: [200, 100, 200],
            tag: 'push-notification',
            data: {
                dateOfArrival: Date.now(),
                primaryKey: 1
            }
        })
    );
});

self.addEventListener('notificationclick', event => {
    console.log('Клик по уведомлению:', event.notification);
    
    event.notification.close();
    
    // Открываем или фокусируем окно приложения
    event.waitUntil(
        clients.openWindow('/dashboard')
    );
});
```

## Синхронизация в фоне (Background Sync)

```javascript
// main.js
if ('serviceWorker' in navigator && 'SyncManager' in window) {
    navigator.serviceWorker.ready
        .then(registration => {
            // Регистрируем задачу синхронизации
            return registration.sync.register('sync-messages');
        })
        .catch(error => {
            console.error('Ошибка регистрации синхронизации:', error);
        });
}
```

```javascript
// sw.js
self.addEventListener('sync', event => {
    if (event.tag === 'sync-messages') {
        event.waitUntil(
            syncMessages()
                .catch(error => {
                    console.error('Ошибка синхронизации:', error);
                    // Повторная попытка синхронизации
                    setTimeout(() => {
                        self.registration.sync.register('sync-messages');
                    }, 30000); // Повтор через 30 секунд
                })
        );
    }
});

async function syncMessages() {
    // Получаем неотправленные сообщения из IndexedDB
    const unsentMessages = await getUnsentMessages();
    
    for (const message of unsentMessages) {
        try {
            const response = await fetch('/api/messages', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(message)
            });
            
            if (response.ok) {
                // Удаляем сообщение из локального хранилища
                await removeSentMessage(message.id);
            }
        } catch (error) {
            console.error('Ошибка отправки сообщения:', error);
            // Прерываем синхронизацию при ошибке
            throw error;
        }
    }
}
```

## Обработка сообщений от основного потока

```javascript
// main.js
navigator.serviceWorker.ready
    .then(registration => {
        // Отправка сообщения в Service Worker
        registration.active.postMessage({
            type: 'CACHE_URL',
            url: '/special-content.html'
        });
        
        // Обработка сообщений от Service Worker
        navigator.serviceWorker.addEventListener('message', event => {
            console.log('Сообщение от Service Worker:', event.data);
            
            if (event.data.type === 'CACHE_COMPLETE') {
                console.log('Кэширование завершено');
            }
        });
    });
```

```javascript
// sw.js
self.addEventListener('message', event => {
    console.log('Сообщение от основного потока:', event.data);
    
    switch(event.data.type) {
        case 'CACHE_URL':
            cacheSpecificUrl(event.data.url);
            break;
        case 'CLEAR_CACHE':
            clearCache(event.data.cacheName);
            break;
        default:
            console.warn('Неизвестный тип сообщения:', event.data.type);
    }
});

async function cacheSpecificUrl(url) {
    try {
        const response = await fetch(url);
        const cache = await caches.open('special-cache-v1');
        await cache.put(url, response);
        
        // Отправка сообщения обратно в основной поток
        event.source.postMessage({
            type: 'CACHE_COMPLETE',
            url: url
        });
    } catch (error) {
        console.error('Ошибка кэширования URL:', error);
    }
}
```

## Совместимость с браузерами

| Браузер | Поддержка Service Workers |
|---------|---------------------------|
| Chrome | С 40.0 |
| Firefox | С 44.0 |
| Safari | С 11.1 |
| Edge | С 17.0 |
| Internet Explorer | Нет |

## Ограничения и особенности

### Безопасность

> [!caution]
> Service Workers требуют HTTPS (или localhost для разработки) из соображений безопасности.

### Ограничения

- Service Workers не имеют доступа к DOM
- Не работают в web workers
- Ограничения на выполнение тяжелых вычислений (могут быть завершены браузером)
- Ограничения на фоновую активность

### Практические ограничения

```javascript
// sw.js - что НЕЛЬЗЯ делать в Service Worker
// document.querySelector('.element'); // Ошибка - нет DOM
// window.localStorage; // Ограниченный доступ
// alert('message'); // Ошибка - нет UI API

// Доступные API в Service Worker:
fetch('https://api.example.com'); // Сетевые запросы
caches.open('cache-name'); // Кэширование
indexedDB; // Базы данных
setTimeout(() => {}, 1000); // Таймеры
```

## Практические применения в российских реалиях 2025

1. **Электронная коммерция** - офлайн-каталоги товаров, кэширование изображений
2. **Банковские приложения** - быстрый доступ к счетам, push-уведомления
3. **Образовательные платформы** - офлайн-доступ к курсам и материалам
4. **Новостные порталы** - кэширование статей, push-уведомления
5. **Транспортные сервисы** - офлайн-карты, уведомления о поездках
6. **Медицинские приложения** - доступ к данным пациентов без подключения
7. **Игры** - сохранение прогресса, офлайн-режим

## Отладка Service Workers

### Инструменты разработчика

В Chrome DevTools:
1. Перейдите на вкладку Application
2. Раздел Service Workers
3. Здесь можно:
   - Просматривать зарегистрированные SW
   - Принудительно обновлять
   - Активировать/деактивировать
   - Проверять кэши

### Логирование и мониторинг

```javascript
// sw.js - улучшенное логирование
const DEBUG = true;

function log(level, message, data = null) {
    if (DEBUG) {
        console.log(`[SW ${level}] ${message}`, data);
    }
}

self.addEventListener('fetch', event => {
    log('FETCH', `Запрос: ${event.request.method} ${event.request.url}`);
    
    event.respondWith(
        (async () => {
            try {
                const response = await caches.match(event.request);
                if (response) {
                    log('CACHE', 'Ответ из кэша', event.request.url);
                    return response;
                }
                
                const networkResponse = await fetch(event.request);
                log('NETWORK', 'Ответ из сети', event.request.url);
                return networkResponse;
            } catch (error) {
                log('ERROR', 'Ошибка запроса', {
                    url: event.request.url,
                    error: error.message
                });
                return caches.match('/offline.html');
            }
        })()
    );
});
```

## Альтернативы и дополнения

- [[Web Workers]] - для выполнения тяжелых вычислений
- [[Cache API]] - для кэширования (используется внутри SW)
- [[Push API]] - для push-уведомлений
- [[Background Sync]] - для синхронизации данных
- [[Web App Manifest]] - для PWA функций

## Рекомендации по использованию

1. **Используйте HTTPS** - Service Workers требуют безопасное соединение
2. **Реализуйте стратегию кэширования** - в зависимости от типа контента
3. **Обрабатывайте ошибки** - особенно при работе с сетью
4. **Оптимизируйте размер кэша** - для экономии места на устройстве
5. **Тестируйте офлайн-режим** - убедитесь, что приложение работает без сети
6. **Регулярно обновляйте кэши** - для предотвращения использования устаревших данных

## Заключение

Service Workers - мощный инструмент для создания современных Progressive Web Apps, обеспечивающий офлайн-функциональность, push-уведомления и улучшенную производительность. В 2025 году они особенно важны для:

- Создания надежных офлайн-приложений
- Улучшения производительности за счет кэширования
- Реализации push-уведомлений
- Фоновой синхронизации данных

При правильной реализации Service Workers могут значительно улучшить пользовательский опыт, особенно в условиях нестабильного интернет-соединения, что особенно актуально для российских реалий с различным качеством связи в разных регионах.
