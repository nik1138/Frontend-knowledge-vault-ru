---
aliases: ["Примеры прогрессивных веб-приложений", "PWA Examples"]
tags: [javascript, pwa, progressive-web-app, frontend]
---

# Примеры прогрессивных веб-приложений

Прогрессивные веб-приложения (PWA) — это веб-приложения, которые используют современные веб-технологии для обеспечения нативного пользовательского опыта. В этом документе представлены практические примеры создания PWA.

## Основы PWA

### 1. Манифест веб-приложения

```json
{
  "name": "Мое PWA Приложение",
  "short_name": "PWA App",
  "description": "Пример прогрессивного веб-приложения",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "orientation": "portrait",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### 2. Регистрация Service Worker

```javascript
// main.js - основной файл приложения
class PWAService {
  constructor() {
    this.isSupported = this.checkSupport();
    this.registration = null;
  }
  
  /**
   * Проверяет поддержку PWA возможностей
   * @returns {boolean} - true если PWA поддерживается
   */
  checkSupport() {
    return 'serviceWorker' in navigator && 'PushManager' in window && 'caches' in window;
  }
  
  /**
   * Регистрирует Service Worker
   */
  async registerServiceWorker() {
    if (!this.isSupported) {
      console.warn('PWA возможности не поддерживаются в этом браузере');
      return;
    }
    
    try {
      this.registration = await navigator.serviceWorker.register('/sw.js');
      console.log('Service Worker зарегистрирован:', this.registration);
      
      // Проверяем обновления
      this.registration.addEventListener('updatefound', () => {
        const newWorker = this.registration.installing;
        newWorker.addEventListener('statechange', () => {
          if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
            console.log('Доступно обновление PWA');
            this.showUpdateNotification();
          }
        });
      });
      
      // Проверяем, контролирует ли Service Worker страницу
      if (navigator.serviceWorker.controller) {
        console.log('PWA уже активен');
      } else {
        console.log('PWA будет активирован при следующей загрузке');
      }
    } catch (error) {
      console.error('Ошибка регистрации Service Worker:', error);
    }
  }
  
  /**
   * Показывает уведомление об обновлении
   */
  showUpdateNotification() {
    if ('Notification' in window && Notification.permission === 'granted') {
      new Notification('Доступно обновление', {
        body: 'Доступна новая версия приложения. Обновить сейчас?',
        icon: '/icons/icon-192x192.png'
      });
    } else {
      // Показываем уведомление в интерфейсе
      this.showUIUpdateNotification();
    }
  }
  
  /**
   * Показывает уведомление об обновлении в интерфейсе
   */
  showUIUpdateNotification() {
    const updateBanner = document.createElement('div');
    updateBanner.className = 'pwa-update-banner';
    updateBanner.innerHTML = `
      <div class="pwa-update-content">
        <span>Доступно обновление приложения</span>
        <button id="pwa-update-btn">Обновить</button>
        <button id="pwa-dismiss-btn">Позже</button>
      </div>
    `;
    
    document.body.appendChild(updateBanner);
    
    // Обработчики событий
    document.getElementById('pwa-update-btn').addEventListener('click', () => {
      this.updateApp();
    });
    
    document.getElementById('pwa-dismiss-btn').addEventListener('click', () => {
      updateBanner.remove();
    });
  }
  
  /**
   * Обновляет приложение
   */
  async updateApp() {
    if (this.registration && this.registration.waiting) {
      this.registration.waiting.postMessage({ type: 'SKIP_WAITING' });
      window.location.reload();
    }
  }
  
  /**
   * Проверяет статус сети
   * @returns {object} - объект с информацией о статусе сети
   */
  getNetworkStatus() {
    return {
      online: navigator.onLine,
      connection: navigator.connection ? navigator.connection.effectiveType : 'unknown',
      downlink: navigator.connection ? navigator.connection.downlink : null
    };
  }
  
  /**
   * Подписывается на изменения статуса сети
   * @param {function} callback - функция обратного вызова
   */
  onNetworkChange(callback) {
    window.addEventListener('online', () => callback({ online: true }));
    window.addEventListener('offline', () => callback({ online: false }));
  }
}

// Инициализация PWA сервиса
const pwaService = new PWAService();

// Регистрация Service Worker при загрузке
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    pwaService.registerServiceWorker();
  });
}
```

### 3. Service Worker (sw.js)

```javascript
// sw.js - Service Worker для PWA
const CACHE_NAME = 'pwa-v1.0.0';
const urlsToCache = [
  '/',
  '/index.html',
  '/styles/main.css',
  '/js/main.js',
  '/js/utils.js',
  '/images/logo.png',
  '/manifest.json'
];

// Событие установки Service Worker
self.addEventListener('install', (event) => {
  console.log('Service Worker устанавливается...');
  
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('Кэширование файлов');
        return cache.addAll(urlsToCache);
      })
      .catch((error) => {
        console.error('Ошибка кэширования:', error);
      })
  );
});

// Событие активации Service Worker
self.addEventListener('activate', (event) => {
  console.log('Service Worker активируется...');
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME) {
            console.log('Удаление старого кэша:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

// Событие fetch - перехват сетевых запросов
self.addEventListener('fetch', (event) => {
  // Не кэшируем запросы к API
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          // Клонируем ответ для кэширования
          const responseToCache = response.clone();
          
          caches.open(CACHE_NAME)
            .then((cache) => {
              cache.put(event.request, responseToCache);
            });
          
          return response;
        })
        .catch(() => {
          // Если запрос к API не удался, пытаемся получить из кэша
          return caches.match(event.request);
        })
    );
    return;
  }
  
  // Для остальных запросов используем стратегию "сначала кэш, потом сеть"
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Возвращаем кэшированный ответ, если он есть
        if (response) {
          return response;
        }
        
        // Иначе делаем сетевой запрос
        return fetch(event.request)
          .then((response) => {
            // Проверяем валидность ответа
            if (!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }
            
            // Клонируем ответ для кэширования
            const responseToCache = response.clone();
            
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, responseToCache);
              });
            
            return response;
          });
      })
  );
});

// Обработка сообщений от клиента
self.addEventListener('message', (event) => {
  if (event.data && event.data.type === 'SKIP_WAITING') {
    self.skipWaiting();
  }
});

// Обработка пуш-уведомлений
self.addEventListener('push', (event) => {
  if (event.data) {
    const data = event.data.json();
    const options = {
      body: data.body || 'У вас новое сообщение',
      icon: data.icon || '/icons/icon-192x192.png',
      badge: '/icons/badge.png',
      tag: data.tag || 'pwa-push'
    };
    
    event.waitUntil(
      self.registration.showNotification(data.title || 'Уведомление', options)
    );
  }
});

// Обработка клика по уведомлению
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  event.waitUntil(
    clients.openWindow(event.notification.data.url || '/')
  );
});
```

## Кэширование и оффлайн функциональность

### 1. Расширенное кэширование

```javascript
// advanced-cache.js - расширенное кэширование для PWA
class AdvancedCacheManager {
  constructor() {
    this.cacheStrategies = {
      IMMUTABLE: 'immutable',      // Неизменяемые ресурсы (библиотеки, изображения)
      STALE_WHILE_REVALIDATE: 'stale-while-revalidate', // Стратегия "старый, но обновляемый"
      CACHE_FIRST: 'cache-first',  // Сначала кэш, потом сеть
      NETWORK_FIRST: 'network-first' // Сначала сеть, потом кэш
    };
    
    this.caches = new Map();
  }
  
  /**
   * Инициализирует кэш с заданной стратегией
   * @param {string} name - имя кэша
   * @param {string} strategy - стратегия кэширования
   * @param {object} options - опции кэширования
   */
  async initCache(name, strategy, options = {}) {
    const cache = await caches.open(name);
    this.caches.set(name, { cache, strategy, options });
    
    console.log(`Кэш ${name} инициализирован со стратегией ${strategy}`);
  }
  
  /**
   * Получает ресурс с использованием стратегии кэширования
   * @param {Request} request - запрос
   * @param {string} cacheName - имя кэша
   * @returns {Response} - ответ
   */
  async get(request, cacheName) {
    const cacheInfo = this.caches.get(cacheName);
    if (!cacheInfo) {
      throw new Error(`Кэш ${cacheName} не найден`);
    }
    
    const { cache, strategy } = cacheInfo;
    
    switch (strategy) {
      case this.cacheStrategies.CACHE_FIRST:
        return this.cacheFirst(cache, request);
        
      case this.cacheStrategies.NETWORK_FIRST:
        return this.networkFirst(cache, request);
        
      case this.cacheStrategies.STALE_WHILE_REVALIDATE:
        return this.staleWhileRevalidate(cache, request);
        
      case this.cacheStrategies.IMMUTABLE:
        return this.immutableCache(cache, request);
        
      default:
        return fetch(request);
    }
  }
  
  /**
   * Стратегия "сначала кэш"
   * @param {Cache} cache - объект кэша
   * @param {Request} request - запрос
   * @returns {Response} - ответ
   */
  async cacheFirst(cache, request) {
    const cachedResponse = await cache.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    const networkResponse = await fetch(request);
    if (networkResponse.ok) {
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  }
  
  /**
   * Стратегия "сначала сеть"
   * @param {Cache} cache - объект кэша
   * @param {Request} request - запрос
   * @returns {Response} - ответ
   */
  async networkFirst(cache, request) {
    try {
      const networkResponse = await fetch(request);
      if (networkResponse.ok) {
        cache.put(request, networkResponse.clone());
        return networkResponse;
      }
    } catch (error) {
      console.warn('Сетевой запрос не удался, используем кэш:', error);
    }
    
    const cachedResponse = await cache.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    throw new Error('Сеть недоступна и кэш пуст');
  }
  
  /**
   * Стратегия "старый, но обновляемый"
   * @param {Cache} cache - объект кэша
   * @param {Request} request - запрос
   * @returns {Response} - ответ
   */
  async staleWhileRevalidate(cache, request) {
    const cachedResponse = await cache.match(request);
    
    // Выполняем сетевой запрос в фоне
    const networkPromise = fetch(request)
      .then(networkResponse => {
        if (networkResponse.ok) {
          cache.put(request, networkResponse.clone());
        }
        return networkResponse;
      })
      .catch(error => {
        console.error('Ошибка обновления кэша:', error);
        return cachedResponse; // Возвращаем кэшированный ответ при ошибке
      });
    
    // Возвращаем кэшированный ответ, если есть, иначе сетевой
    return cachedResponse || networkPromise;
  }
  
  /**
   * Стратегия для неизменяемых ресурсов
   * @param {Cache} cache - объект кэша
   * @param {Request} request - запрос
   * @returns {Response} - ответ
   */
  async immutableCache(cache, request) {
    // Для неизменяемых ресурсов всегда сначала проверяем кэш
    const cachedResponse = await cache.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    
    // Загружаем и кэшируем навсегда
    const networkResponse = await fetch(request);
    if (networkResponse.ok) {
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  }
  
  /**
   * Удаляет устаревшие кэши
   * @param {Array} validCacheNames - массив допустимых имен кэшей
   */
  async cleanupCaches(validCacheNames) {
    const cacheNames = await caches.keys();
    
    return Promise.all(
      cacheNames
        .filter(name => !validCacheNames.includes(name))
        .map(name => {
          console.log('Удаление устаревшего кэша:', name);
          return caches.delete(name);
        })
    );
  }
  
  /**
   * Очищает конкретный кэш
   * @param {string} cacheName - имя кэша
   */
  async clearCache(cacheName) {
    const cacheInfo = this.caches.get(cacheName);
    if (cacheInfo) {
      await cacheInfo.cache.addAll();
      this.caches.delete(cacheName);
      console.log(`Кэш ${cacheName} очищен`);
    }
  }
  
  /**
   * Получает информацию о кэше
   * @param {string} cacheName - имя кэша
   * @returns {object} - информация о кэше
   */
  async getCacheInfo(cacheName) {
    const cacheInfo = this.caches.get(cacheName);
    if (!cacheInfo) {
      return null;
    }
    
    const keys = await cacheInfo.cache.keys();
    const size = await this.calculateCacheSize(cacheInfo.cache);
    
    return {
      name: cacheName,
      strategy: cacheInfo.strategy,
      size: size,
      entries: keys.length,
      requests: keys.map(key => key.url)
    };
  }
  
  /**
   * Вычисляет размер кэша
   * @param {Cache} cache - объект кэша
   * @returns {number} - размер в байтах
   */
  async calculateCacheSize(cache) {
    const keys = await cache.keys();
    let totalSize = 0;
    
    for (const request of keys) {
      const response = await cache.match(request);
      if (response) {
        const blob = await response.blob();
        totalSize += blob.size;
      }
    }
    
    return totalSize;
  }
}

// Глобальный экземпляр менеджера кэша
window.advancedCacheManager = new AdvancedCacheManager();
```

### 2. Оффлайн страница

```javascript
// offline-manager.js - менеджер оффлайн режима
class OfflineManager {
  constructor() {
    this.isOffline = !navigator.onLine;
    this.offlineQueue = [];
    this.syncCallbacks = [];
    
    this.initialize();
  }
  
  /**
   * Инициализирует менеджер оффлайн режима
   */
  initialize() {
    // Слушаем изменения статуса сети
    window.addEventListener('online', () => this.goOnline());
    window.addEventListener('offline', () => this.goOffline());
    
    // Проверяем начальный статус
    this.isOffline = !navigator.onLine;
    
    if (this.isOffline) {
      this.showOfflineUI();
    }
  }
  
  /**
   * Переходит в онлайн режим
   */
  goOnline() {
    if (this.isOffline) {
      console.log('Соединение восстановлено');
      this.isOffline = false;
      this.hideOfflineUI();
      this.processOfflineQueue();
    }
  }
  
  /**
   * Переходит в оффлайн режим
   */
  goOffline() {
    if (!this.isOffline) {
      console.log('Соединение потеряно');
      this.isOffline = true;
      this.showOfflineUI();
    }
  }
  
  /**
   * Показывает UI оффлайн режима
   */
  showOfflineUI() {
    // Создаем оверлей оффлайн режима
    const offlineOverlay = document.createElement('div');
    offlineOverlay.id = 'offline-overlay';
    offlineOverlay.className = 'offline-overlay';
    offlineOverlay.innerHTML = `
      <div class="offline-message">
        <div class="offline-icon">📶</div>
        <h3>Оффлайн режим</h3>
        <p>Подключение к интернету отсутствует</p>
        <div class="offline-actions">
          <button id="retry-connection">Повторить</button>
        </div>
      </div>
    `;
    
    document.body.appendChild(offlineOverlay);
    
    // Обработчик кнопки повтора
    document.getElementById('retry-connection').addEventListener('click', () => {
      if (navigator.onLine) {
        this.goOnline();
      }
    });
  }
  
  /**
   * Скрывает UI оффлайн режима
   */
  hideOfflineUI() {
    const offlineOverlay = document.getElementById('offline-overlay');
    if (offlineOverlay) {
      offlineOverlay.remove();
    }
  }
  
  /**
   * Добавляет запрос в очередь оффлайн
   * @param {object} request - объект запроса
   * @param {string} method - HTTP метод
   * @param {string} url - URL запроса
   * @param {object} data - данные запроса
   */
  addToOfflineQueue(request, method, url, data) {
    const queueItem = {
      id: Date.now(),
      timestamp: Date.now(),
      method,
      url,
      data,
      attempts: 0,
      maxAttempts: 3
    };
    
    this.offlineQueue.push(queueItem);
    console.log(`Запрос добавлен в оффлайн очередь: ${method} ${url}`);
    
    // Сохраняем очередь в localStorage
    this.saveOfflineQueue();
  }
  
  /**
   * Обрабатывает очередь оффлайн запросов
   */
  async processOfflineQueue() {
    console.log(`Обработка очереди из ${this.offlineQueue.length} запросов`);
    
    for (const queueItem of [...this.offlineQueue]) {
      if (queueItem.attempts >= queueItem.maxAttempts) {
        console.error(`Превышено количество попыток для запроса: ${queueItem.url}`);
        this.offlineQueue = this.offlineQueue.filter(item => item.id !== queueItem.id);
        continue;
      }
      
      try {
        const response = await fetch(queueItem.url, {
          method: queueItem.method,
          headers: {
            'Content-Type': 'application/json'
          },
          body: queueItem.data ? JSON.stringify(queueItem.data) : undefined
        });
        
        if (response.ok) {
          console.log(`Запрос успешно выполнен: ${queueItem.url}`);
          this.offlineQueue = this.offlineQueue.filter(item => item.id !== queueItem.id);
        } else {
          throw new Error(`HTTP ошибка: ${response.status}`);
        }
      } catch (error) {
        console.error(`Ошибка выполнения запроса: ${queueItem.url}`, error);
        queueItem.attempts++;
        
        if (queueItem.attempts >= queueItem.maxAttempts) {
          console.error(`Превышено количество попыток для запроса: ${queueItem.url}`);
          this.offlineQueue = this.offlineQueue.filter(item => item.id !== queueItem.id);
        }
      }
    }
    
    this.saveOfflineQueue();
    
    // Вызываем колбэки синхронизации
    this.syncCallbacks.forEach(callback => callback(this.offlineQueue.length === 0));
  }
  
  /**
   * Сохраняет очередь оффлайн в localStorage
   */
  saveOfflineQueue() {
    try {
      localStorage.setItem('offlineQueue', JSON.stringify(this.offlineQueue));
    } catch (error) {
      console.error('Ошибка сохранения очереди оффлайн:', error);
    }
  }
  
  /**
   * Загружает очередь оффлайн из localStorage
   */
  loadOfflineQueue() {
    try {
      const queueData = localStorage.getItem('offlineQueue');
      if (queueData) {
        this.offlineQueue = JSON.parse(queueData);
        console.log(`Загружена очередь из ${this.offlineQueue.length} запросов`);
      }
    } catch (error) {
      console.error('Ошибка загрузки очереди оффлайн:', error);
      this.offlineQueue = [];
    }
  }
  
  /**
   * Подписывается на события синхронизации
   * @param {function} callback - функция обратного вызова
   */
  onSync(callback) {
    this.syncCallbacks.push(callback);
  }
  
  /**
   * Получает статус оффлайн режима
   * @returns {object} - объект статуса
   */
  getStatus() {
    return {
      isOffline: this.isOffline,
      queueLength: this.offlineQueue.length,
      networkStatus: navigator.connection ? navigator.connection.effectiveType : 'unknown'
    };
  }
}

// Инициализация оффлайн менеджера
window.offlineManager = new OfflineManager();
window.offlineManager.loadOfflineQueue(); // Загружаем очередь при старте
```

## Пуш-уведомления

### 1. Менеджер пуш-уведомлений

```javascript
// push-notification-manager.js - менеджер пуш-уведомлений
class PushNotificationManager {
  constructor() {
    this.isSupported = this.checkSupport();
    this.subscription = null;
    this.vapidPublicKey = null; // VAPID public key для Web Push
  }
  
  /**
   * Проверяет поддержку пуш-уведомлений
   * @returns {boolean} - true если поддерживается
   */
  checkSupport() {
    return 'serviceWorker' in navigator && 'PushManager' in window && 'Notification' in window;
  }
  
  /**
   * Запрашивает разрешение на уведомления
   * @returns {Promise} - промис с результатом
   */
  async requestPermission() {
    if (!this.isSupported) {
      throw new Error('Пуш-уведомления не поддерживаются');
    }
    
    const permission = await Notification.requestPermission();
    
    if (permission === 'granted') {
      console.log('Разрешение на уведомления получено');
      return true;
    } else if (permission === 'denied') {
      console.log('Разрешение на уведомления отклонено');
      return false;
    } else {
      console.log('Разрешение на уведомления не принято/отклонено');
      return false;
    }
  }
  
  /**
   * Подписывается на пуш-уведомления
   * @param {string} vapidPublicKey - VAPID public key
   * @returns {object} - объект подписки
   */
  async subscribe(vapidPublicKey) {
    if (!await this.requestPermission()) {
      throw new Error('Разрешение на уведомления не получено');
    }
    
    this.vapidPublicKey = vapidPublicKey;
    
    const registration = await navigator.serviceWorker.ready;
    
    // Получаем существующую подписку или создаем новую
    this.subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: this.urlB64ToUint8Array(vapidPublicKey)
    });
    
    console.log('Подписка на пуш-уведомления создана');
    
    // Отправляем подписку на сервер
    await this.sendSubscriptionToServer(this.subscription);
    
    return this.subscription;
  }
  
  /**
   * Отменяет подписку на пуш-уведомления
   */
  async unsubscribe() {
    if (!this.subscription) {
      console.log('Нет активной подписки');
      return;
    }
    
    await this.subscription.unsubscribe();
    console.log('Подписка на пуш-уведомления отменена');
    
    // Удаляем подписку с сервера
    await this.removeSubscriptionFromServer(this.subscription);
    
    this.subscription = null;
  }
  
  /**
   * Отправляет подписку на сервер
   * @param {object} subscription - объект подписки
   */
  async sendSubscriptionToServer(subscription) {
    try {
      await fetch('/api/push/subscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(subscription)
      });
      
      console.log('Подписка отправлена на сервер');
    } catch (error) {
      console.error('Ошибка отправки подписки на сервер:', error);
    }
  }
  
  /**
   * Удаляет подписку с сервера
   * @param {object} subscription - объект подписки
   */
  async removeSubscriptionFromServer(subscription) {
    try {
      await fetch('/api/push/unsubscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(subscription)
      });
      
      console.log('Подписка удалена с сервера');
    } catch (error) {
      console.error('Ошибка удаления подписки с сервера:', error);
    }
  }
  
  /**
   * Преобразует URL-безопасный base64 в Uint8Array
   * @param {string} base64String - base64 строка
   * @returns {Uint8Array} - массив байтов
   */
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
  
  /**
   * Отображает локальное уведомление
   * @param {string} title - заголовок
   * @param {object} options - опции уведомления
   */
  showLocalNotification(title, options = {}) {
    if (Notification.permission === 'granted') {
      new Notification(title, {
        body: options.body || '',
        icon: options.icon || '/icons/icon-192x192.png',
        badge: options.badge || '/icons/badge.png',
        tag: options.tag || 'pwa-notification',
        data: options.data || {}
      });
    }
  }
  
  /**
   * Проверяет статус подписки
   * @returns {object} - статус подписки
   */
  getSubscriptionStatus() {
    return {
      isSupported: this.isSupported,
      isSubscribed: !!this.subscription,
      permission: Notification.permission
    };
  }
  
  /**
   * Получает данные подписки для отправки на сервер
   * @returns {object} - данные подписки
   */
  getSubscriptionData() {
    if (!this.subscription) {
      return null;
    }
    
    const subscriptionObj = this.subscription.toJSON();
    return {
      endpoint: subscriptionObj.endpoint,
      publicKey: subscriptionObj.keys.p256dh,
      auth: subscriptionObj.keys.auth
    };
  }
}

// Глобальный экземпляр менеджера пуш-уведомлений
window.pushNotificationManager = new PushNotificationManager();
```

## Адаптивный дизайн и производительность

### 1. Адаптивные компоненты для PWA

```javascript
// responsive-components.js - адаптивные компоненты для PWA
class ResponsiveComponentManager {
  constructor() {
    this.breakpoints = {
      xs: 0,
      sm: 576,
      md: 768,
      lg: 992,
      xl: 1200,
      xxl: 1400
    };
    
    this.currentBreakpoint = this.getCurrentBreakpoint();
    this.listeners = [];
    
    this.initialize();
  }
  
  /**
   * Инициализирует менеджер адаптивных компонентов
   */
  initialize() {
    // Слушаем изменения размера окна
    window.addEventListener('resize', this.debounce(() => {
      const newBreakpoint = this.getCurrentBreakpoint();
      if (newBreakpoint !== this.currentBreakpoint) {
        this.currentBreakpoint = newBreakpoint;
        this.notifyListeners(newBreakpoint);
      }
    }, 250));
  }
  
  /**
   * Получает текущую точку останова
   * @returns {string} - имя точки останова
   */
  getCurrentBreakpoint() {
    const width = window.innerWidth;
    
    if (width >= this.breakpoints.xxl) return 'xxl';
    if (width >= this.breakpoints.xl) return 'xl';
    if (width >= this.breakpoints.lg) return 'lg';
    if (width >= this.breakpoints.md) return 'md';
    if (width >= this.breakpoints.sm) return 'sm';
    return 'xs';
  }
  
  /**
   * Проверяет, соответствует ли текущая точка останова заданной
   * @param {string} breakpoint - точка останова
   * @returns {boolean} - true если соответствует
   */
  matchesBreakpoint(breakpoint) {
    return this.currentBreakpoint === breakpoint;
  }
  
  /**
   * Проверяет, больше ли текущая точка останова заданной
   * @param {string} breakpoint - точка останова
   * @returns {boolean} - true если больше
   */
  isBreakpointUp(breakpoint) {
    const breakpointOrder = ['xs', 'sm', 'md', 'lg', 'xl', 'xxl'];
    const currentIdx = breakpointOrder.indexOf(this.currentBreakpoint);
    const targetIdx = breakpointOrder.indexOf(breakpoint);
    
    return currentIdx >= targetIdx;
  }
  
  /**
   * Проверяет, меньше ли текущая точка останова заданной
   * @param {string} breakpoint - точка останова
   * @returns {boolean} - true если меньше
   */
  isBreakpointDown(breakpoint) {
    const breakpointOrder = ['xs', 'sm', 'md', 'lg', 'xl', 'xxl'];
    const currentIdx = breakpointOrder.indexOf(this.currentBreakpoint);
    const targetIdx = breakpointOrder.indexOf(breakpoint);
    
    return currentIdx <= targetIdx;
  }
  
  /**
   * Добавляет слушатель изменения точки останова
   * @param {function} callback - функция обратного вызова
   */
  onBreakpointChange(callback) {
    this.listeners.push(callback);
  }
  
  /**
   * Уведомляет слушателей об изменении точки останова
   * @param {string} newBreakpoint - новая точка останова
   */
  notifyListeners(newBreakpoint) {
    this.listeners.forEach(callback => {
      try {
        callback(newBreakpoint, this.currentBreakpoint);
      } catch (error) {
        console.error('Ошибка в слушателе изменения точки останова:', error);
      }
    });
  }
  
  /**
   * Создает адаптивный компонент
   * @param {object} config - конфигурация компонента
   * @returns {HTMLElement} - элемент компонента
   */
  createResponsiveComponent(config) {
    const component = document.createElement('div');
    component.className = `responsive-component ${config.className || ''}`;
    
    // Рендерим компонент в зависимости от точки останова
    const renderComponent = () => {
      const breakpoint = this.getCurrentBreakpoint();
      const template = config.templates[breakpoint] || config.templates.default;
      
      if (template) {
        component.innerHTML = typeof template === 'function' ? 
          template(config.data || {}) : template;
      }
    };
    
    // Рендерим начальный компонент
    renderComponent();
    
    // Обновляем компонент при изменении точки останова
    this.onBreakpointChange(() => {
      renderComponent();
    });
    
    return component;
  }
  
  /**
   * Дебаунс функции
   * @param {function} func - функция для дебаунса
   * @param {number} wait - время ожидания в мс
   * @returns {function} - дебаунсированная функция
   */
  debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
      const later = () => {
        clearTimeout(timeout);
        func(...args);
      };
      clearTimeout(timeout);
      timeout = setTimeout(later, wait);
    };
  }
}

// Глобальный экземпляр менеджера адаптивных компонентов
window.responsiveComponentManager = new ResponsiveComponentManager();
```

### 2. Оптимизация производительности PWA

```javascript
// performance-optimizer.js - оптимизатор производительности PWA
class PerformanceOptimizer {
  constructor() {
    this.metrics = {};
    this.observers = [];
    this.resourceTiming = [];
    
    this.initialize();
  }
  
  /**
   * Инициализирует оптимизатор производительности
   */
  initialize() {
    // Собираем метрики загрузки страницы
    this.collectPageLoadMetrics();
    
    // Начинаем наблюдение за производительностью
    this.startPerformanceObservation();
    
    // Проверяем производительность ресурсов
    this.monitorResourcePerformance();
  }
  
  /**
   * Собирает метрики загрузки страницы
   */
  collectPageLoadMetrics() {
    if (performance.timing) {
      const timing = performance.timing;
      
      this.metrics = {
        // Время загрузки
        navigationStart: timing.navigationStart,
        loadEventEnd: timing.loadEventEnd,
        pageLoadTime: timing.loadEventEnd - timing.navigationStart,
        
        // Время до DOMContentLoaded
        domContentLoaded: timing.domContentLoadedEventEnd - timing.navigationStart,
        
        // Время до первого байта
        ttfb: timing.responseStart - timing.navigationStart,
        
        // Время загрузки домена
        domainLookup: timing.domainLookupEnd - timing.domainLookupStart,
        
        // Время подключения
        connect: timing.connectEnd - timing.connectStart,
        
        // Время ответа сервера
        response: timing.responseEnd - timing.responseStart
      };
    }
    
    console.log('Метрики загрузки страницы:', this.metrics);
  }
  
  /**
   * Начинает наблюдение за производительностью
   */
  startPerformanceObservation() {
    // Наблюдение за метриками производительности
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          if (entry.entryType === 'navigation') {
            console.log('Метрики навигации:', entry);
          } else if (entry.entryType === 'paint') {
            console.log('Paint метрика:', entry);
          }
        });
      });
      
      observer.observe({ entryTypes: ['navigation', 'paint', 'measure'] });
      this.observers.push(observer);
    }
  }
  
  /**
   * Мониторит производительность ресурсов
   */
  monitorResourcePerformance() {
    if ('PerformanceObserver' in window) {
      const resourceObserver = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          this.resourceTiming.push({
            name: entry.name,
            duration: entry.duration,
            size: entry.transferSize,
            type: entry.entryType
          });
        });
      });
      
      resourceObserver.observe({ entryTypes: ['resource'] });
      this.observers.push(resourceObserver);
    }
  }
  
  /**
   * Измеряет производительность функции
   * @param {function} func - функция для измерения
   * @param {string} name - имя метрики
   * @returns {*} - результат выполнения функции
   */
  async measureFunction(func, name) {
    const startMark = `start-${name}`;
    const endMark = `end-${name}`;
    
    performance.mark(startMark);
    
    try {
      const result = await Promise.resolve(func());
      
      performance.mark(endMark);
      performance.measure(name, startMark, endMark);
      
      const measure = performance.getEntriesByName(name)[0];
      console.log(`Измерение ${name}: ${measure.duration}ms`);
      
      return result;
    } catch (error) {
      performance.mark(endMark);
      performance.measure(name, startMark, endMark);
      
      const measure = performance.getEntriesByName(name)[0];
      console.error(`Ошибка в ${name} (${measure.duration}ms):`, error);
      
      throw error;
    }
  }
  
  /**
   * Оптимизирует загрузку изображений
   * @param {HTMLElement} container - контейнер с изображениями
   */
  optimizeImages(container) {
    const images = container.querySelectorAll('img[data-src]');
    
    // Используем Intersection Observer для ленивой загрузки
    if ('IntersectionObserver' in window) {
      const imageObserver = new IntersectionObserver((entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            const img = entry.target;
            img.src = img.dataset.src;
            img.removeAttribute('data-src');
            
            // Убираем наблюдение за этим элементом
            imageObserver.unobserve(img);
          }
        });
      });
      
      images.forEach(img => imageObserver.observe(img));
    } else {
      // Резервный вариант для старых браузеров
      images.forEach(img => {
        img.src = img.dataset.src;
        img.removeAttribute('data-src');
      });
    }
  }
  
  /**
   * Оптимизирует рендеринг списка
   * @param {Array} items - массив элементов
   * @param {function} renderItem - функция рендеринга элемента
   * @param {HTMLElement} container - контейнер для рендеринга
   * @param {object} options - опции оптимизации
   */
  optimizeListRendering(items, renderItem, container, options = {}) {
    const {
      batchSize = 10,    // Количество элементов за раз
      delay = 16,        // Задержка между пакетами (примерно 60 FPS)
      useDocumentFragment = true
    } = options;
    
    let currentIndex = 0;
    const fragment = useDocumentFragment ? document.createDocumentFragment() : null;
    
    const renderBatch = () => {
      const endIndex = Math.min(currentIndex + batchSize, items.length);
      
      for (let i = currentIndex; i < endIndex; i++) {
        const itemElement = renderItem(items[i], i);
        if (fragment) {
          fragment.appendChild(itemElement);
        } else {
          container.appendChild(itemElement);
        }
      }
      
      currentIndex = endIndex;
      
      if (currentIndex < items.length) {
        setTimeout(renderBatch, delay);
      } else {
        // Вставляем фрагмент в DOM за раз
        if (fragment) {
          container.appendChild(fragment);
        }
        console.log(`Рендеринг списка завершен. Всего элементов: ${items.length}`);
      }
    };
    
    renderBatch();
  }
  
  /**
   * Получает отчет о производительности
   * @returns {object} - объект с метриками производительности
   */
  getPerformanceReport() {
    return {
      metrics: this.metrics,
      resourceTiming: this.resourceTiming,
      memory: this.getMemoryInfo(),
      network: this.getNetworkInfo()
    };
  }
  
  /**
   * Получает информацию о памяти (если доступна)
   * @returns {object} - информация о памяти
   */
  getMemoryInfo() {
    if ('memory' in performance) {
      return {
        usedJSHeapSize: performance.memory.usedJSHeapSize,
        totalJSHeapSize: performance.memory.totalJSHeapSize,
        jsHeapSizeLimit: performance.memory.jsHeapSizeLimit
      };
    }
    return null;
  }
  
  /**
   * Получает информацию о сети
   * @returns {object} - информация о сети
   */
  getNetworkInfo() {
    if ('connection' in navigator) {
      return {
        effectiveType: navigator.connection.effectiveType,
        downlink: navigator.connection.downlink,
        rtt: navigator.connection.rtt,
        saveData: navigator.connection.saveData
      };
    }
    return null;
  }
  
  /**
   * Очищает ресурсы
   */
  destroy() {
    // Отключаем наблюдателей
    this.observers.forEach(observer => observer.disconnect());
    this.observers = [];
  }
}

// Глобальный экземпляр оптимизатора производительности
window.performanceOptimizer = new PerformanceOptimizer();

// Автоматическая оптимизация при загрузке
document.addEventListener('DOMContentLoaded', () => {
  const container = document.getElementById('main-content');
  if (container) {
    window.performanceOptimizer.optimizeImages(container);
  }
});
```

## Практические примеры использования

### 1. Пример полного PWA приложения

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Пример PWA Приложения</title>
    <link rel="manifest" href="/manifest.json">
    <link rel="apple-touch-icon" href="/icons/icon-192x192.png">
    <meta name="theme-color" content="#000000">
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f5f5f5;
        }
        
        .app-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .offline-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10000;
        }
        
        .offline-message {
            background: white;
            padding: 30px;
            border-radius: 10px;
            text-align: center;
            max-width: 400px;
        }
        
        .offline-icon {
            font-size: 3rem;
            margin-bottom: 15px;
        }
        
        .offline-actions {
            margin-top: 20px;
        }
        
        .offline-actions button {
            padding: 10px 20px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        
        .pwa-update-banner {
            position: fixed;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #333;
            color: white;
            padding: 15px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            gap: 15px;
            z-index: 1000;
        }
        
        .pwa-update-banner button {
            padding: 5px 10px;
            margin-left: 10px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 3px;
            cursor: pointer;
        }
        
        .responsive-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="app-container">
        <header>
            <h1>Пример PWA Приложения</h1>
            <p>Это прогрессивное веб-приложение с оффлайн функциональностью</p>
        </header>
        
        <main id="main-content">
            <div class="responsive-grid" id="content-grid">
                <!-- Контент будет добавлен динамически -->
            </div>
        </main>
    </div>

    <script src="/js/pwa-service.js"></script>
    <script src="/js/advanced-cache.js"></script>
    <script src="/js/offline-manager.js"></script>
    <script src="/js/push-notification-manager.js"></script>
    <script src="/js/responsive-components.js"></script>
    <script src="/js/performance-optimizer.js"></script>
    
    <script>
        // Инициализация PWA приложения
        document.addEventListener('DOMContentLoaded', async () => {
            try {
                // Инициализация всех сервисов
                await pwaService.registerServiceWorker();
                
                // Оптимизация изображений
                performanceOptimizer.optimizeImages(document.getElementById('main-content'));
                
                // Создание адаптивного контента
                createResponsiveContent();
                
                console.log('PWA приложение инициализировано');
            } catch (error) {
                console.error('Ошибка инициализации PWA:', error);
            }
        });
        
        function createResponsiveContent() {
            const grid = document.getElementById('content-grid');
            
            // Создаем адаптивные карточки
            for (let i = 0; i < 6; i++) {
                const card = document.createElement('div');
                card.className = 'card';
                card.innerHTML = `
                    <h3>Карточка ${i + 1}</h3>
                    <p>Это адаптивный контент, который изменяется в зависимости от размера экрана.</p>
                    <img data-src="/images/placeholder-${i % 3}.jpg" alt="Изображение" style="width: 100%; height: 200px; object-fit: cover;">
                `;
                grid.appendChild(card);
            }
            
            // Оптимизируем загрузку изображений
            performanceOptimizer.optimizeImages(grid);
        }
    </script>
</body>
</html>
```

### 2. Интеграция с API и синхронизация данных

```javascript
// pwa-data-sync.js - синхронизация данных для PWA
class PWADataSync {
  constructor() {
    this.syncQueue = [];
    this.isSyncing = false;
    this.syncInterval = 30000; // 30 секунд
    this.syncTimer = null;
    
    this.initialize();
  }
  
  /**
   * Инициализирует синхронизацию данных
   */
  initialize() {
    // Загружаем очередь синхронизации из localStorage
    this.loadSyncQueue();
    
    // Запускаем периодическую синхронизацию
    this.startPeriodicSync();
    
    // Слушаем события онлайн/оффлайн для мгновенной синхронизации
    window.addEventListener('online', () => {
      if (this.syncQueue.length > 0) {
        this.syncNow();
      }
    });
  }
  
  /**
   * Добавляет операцию в очередь синхронизации
   * @param {string} operation - тип операции (create, update, delete)
   * @param {string} endpoint - API endpoint
   * @param {object} data - данные операции
   * @param {object} options - опции
   */
  addToSyncQueue(operation, endpoint, data, options = {}) {
    const syncItem = {
      id: Date.now() + Math.random(),
      operation,
      endpoint,
      data,
      options,
      timestamp: Date.now(),
      attempts: 0,
      maxAttempts: options.maxAttempts || 3
    };
    
    this.syncQueue.push(syncItem);
    this.saveSyncQueue();
    
    console.log(`Операция добавлена в очередь синхронизации: ${operation} ${endpoint}`);
    
    // Если приложение онлайн, запускаем синхронизацию
    if (navigator.onLine) {
      this.syncNow();
    }
  }
  
  /**
   * Выполняет синхронизацию
   */
  async syncNow() {
    if (this.isSyncing || this.syncQueue.length === 0) {
      return;
    }
    
    this.isSyncing = true;
    
    try {
      console.log(`Начало синхронизации ${this.syncQueue.length} операций`);
      
      for (const syncItem of [...this.syncQueue]) {
        if (syncItem.attempts >= syncItem.maxAttempts) {
          console.error(`Превышено количество попыток для операции: ${syncItem.operation} ${syncItem.endpoint}`);
          this.syncQueue = this.syncQueue.filter(item => item.id !== syncItem.id);
          continue;
        }
        
        try {
          await this.executeSyncOperation(syncItem);
          // Успешно выполненная операция удаляется из очереди
          this.syncQueue = this.syncQueue.filter(item => item.id !== syncItem.id);
          console.log(`Операция выполнена: ${syncItem.operation} ${syncItem.endpoint}`);
        } catch (error) {
          console.error(`Ошибка синхронизации: ${syncItem.operation} ${syncItem.endpoint}`, error);
          syncItem.attempts++;
          
          if (syncItem.attempts >= syncItem.maxAttempts) {
            console.error(`Превышено количество попыток для операции: ${syncItem.operation} ${syncItem.endpoint}`);
            this.syncQueue = this.syncQueue.filter(item => item.id !== syncItem.id);
          }
        }
      }
      
      this.saveSyncQueue();
      console.log('Синхронизация завершена');
    } catch (error) {
      console.error('Критическая ошибка синхронизации:', error);
    } finally {
      this.isSyncing = false;
    }
  }
  
  /**
   * Выполняет операцию синхронизации
   * @param {object} syncItem - элемент очереди синхронизации
   */
  async executeSyncOperation(syncItem) {
    const { operation, endpoint, data } = syncItem;
    
    let method, body;
    
    switch (operation.toLowerCase()) {
      case 'create':
        method = 'POST';
        body = JSON.stringify(data);
        break;
      case 'update':
        method = 'PUT';
        body = JSON.stringify(data);
        break;
      case 'delete':
        method = 'DELETE';
        break;
      case 'patch':
        method = 'PATCH';
        body = JSON.stringify(data);
        break;
      default:
        throw new Error(`Неизвестная операция: ${operation}`);
    }
    
    const response = await fetch(endpoint, {
      method,
      headers: {
        'Content-Type': 'application/json'
      },
      body
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status} ${response.statusText}`);
    }
    
    return response.json();
  }
  
  /**
   * Начинает периодическую синхронизацию
   */
  startPeriodicSync() {
    this.syncTimer = setInterval(() => {
      if (navigator.onLine && this.syncQueue.length > 0) {
        this.syncNow();
      }
    }, this.syncInterval);
  }
  
  /**
   * Останавливает периодическую синхронизацию
   */
  stopPeriodicSync() {
    if (this.syncTimer) {
      clearInterval(this.syncTimer);
      this.syncTimer = null;
    }
  }
  
  /**
   * Сохраняет очередь синхронизации в localStorage
   */
  saveSyncQueue() {
    try {
      localStorage.setItem('pwaSyncQueue', JSON.stringify(this.syncQueue));
    } catch (error) {
      console.error('Ошибка сохранения очереди синхронизации:', error);
    }
  }
  
  /**
   * Загружает очередь синхронизации из localStorage
   */
  loadSyncQueue() {
    try {
      const queueData = localStorage.getItem('pwaSyncQueue');
      if (queueData) {
        this.syncQueue = JSON.parse(queueData);
        console.log(`Загружена очередь синхронизации из ${this.syncQueue.length} операций`);
      }
    } catch (error) {
      console.error('Ошибка загрузки очереди синхронизации:', error);
      this.syncQueue = [];
    }
  }
  
  /**
   * Получает статус синхронизации
   * @returns {object} - статус синхронизации
   */
  getSyncStatus() {
    return {
      isSyncing: this.isSyncing,
      queueLength: this.syncQueue.length,
      nextSync: this.syncTimer ? this.syncInterval : null
    };
  }
  
  /**
   * Очищает очередь синхронизации
   */
  clearSyncQueue() {
    this.syncQueue = [];
    this.saveSyncQueue();
    console.log('Очередь синхронизации очищена');
  }
  
  /**
   * Уничтожает экземпляр класса
   */
  destroy() {
    this.stopPeriodicSync();
    this.clearSyncQueue();
  }
}

// Глобальный экземпляр синхронизатора данных
window.pwaDataSync = new PWADataSync();
```

## Практические советы

- Обязательно тестируйте PWA в разных браузерах и устройствах
- Используйте Web App Manifest для настройки внешнего вида
- Реализуйте стратегию кэширования в зависимости от типа контента
- Обеспечьте корректную работу в оффлайн режиме
- Оптимизируйте изображения и статические ресурсы
- Используйте современные API для улучшения производительности
- Регулярно обновляйте Service Worker для исправления багов

## Связанные темы

- [[service-worker-patterns]]
- [[web-app-manifest-guide]]
- [[offline-first-architecture]]