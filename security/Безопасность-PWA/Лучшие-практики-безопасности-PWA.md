---
aliases: ["PWA безопасность", "Рекомендации по безопасности PWA"]
tags: [security, pwa, progressive-web-apps, web-security, frontend-security]
---

# Лучшие практики безопасности PWA

## Введение

Прогрессивные веб-приложения (PWA) сочетают в себе лучшие черты веб- и мобильных приложений, но при этом требуют особого внимания к вопросам безопасности. Из-за оффлайн-функциональности, возможностей push-уведомлений и доступа к системным API, PWA имеют уникальные векторы атак, требующие специфических мер защиты.

## Архитектурные аспекты безопасности PWA

### 1. Безопасность манифеста PWA

Манифест PWA (web manifest) должен быть защищен от несанкционированного изменения:

```json
{
  "name": "Мое безопасное PWA",
  "short_name": "SecurePWA",
  "start_url": "/?utm_source=web_app",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ],
  "scope": "/",
  "serviceworker": {
    "src": "/sw.js",
    "scope": "/",
    "type": "module"
  }
}
```

> [!note] Примечание
> Убедитесь, что файл манифеста доступен только по HTTPS и имеет правильные заголовки безопасности

### 2. Защита Service Worker

Service Worker - это мощный компонент PWA, требующий особой защиты:

```javascript
// sw.js - безопасный Service Worker
const CACHE_NAME = 'secure-pwa-v1.2.3';
const FILES_TO_CACHE = [
  '/',
  '/styles/main.css',
  '/scripts/main.js',
  '/offline.html'
];

// Установка кэша
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(FILES_TO_CACHE))
      .then(self.skipWaiting())
  );
});

// Активация Service Worker
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_NAME) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

// Обработка запросов с безопасностью
self.addEventListener('fetch', (event) => {
  // Проверяем, что запрос безопасен
  if (event.request.url.startsWith(self.location.origin)) {
    event.respondWith(
      caches.match(event.request)
        .then((response) => {
          // Возвращаем кэшированный ответ или делаем сетевой запрос
          return response || fetch(event.request);
        })
        .catch(() => {
          // Возвращаем оффлайн-страницу для безопасных URL
          if (event.request.destination === 'document') {
            return caches.match('/offline.html');
          }
        })
    );
  } else {
    // Для сторонних запросов используем только безопасные методы
    event.respondWith(fetch(event.request));
  }
});

// Защита от XSS в сообщениях Service Worker
self.addEventListener('message', (event) => {
  // Проверяем источник сообщения
  if (event.origin !== self.location.origin) {
    return; // Отбрасываем сообщения из других источников
  }
  
  // Санитизируем полученные данные
  const sanitizedData = sanitizeMessageData(event.data);
  
  // Обрабатываем только разрешенные типы сообщений
  handleSecureMessage(sanitizedData);
});

function sanitizeMessageData(data) {
  // Базовая санитизация данных
  if (typeof data === 'object') {
    const sanitized = {};
    for (const [key, value] of Object.entries(data)) {
      if (typeof value === 'string') {
        sanitized[key] = sanitizeString(value);
      } else {
        sanitized[key] = value;
      }
    }
    return sanitized;
  }
  return data;
}

function sanitizeString(str) {
  // Удаляем потенциально опасные символы
  return str.replace(/[<>'"&]/g, (match) => {
    const escapeMap = { '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#x27;', '&': '&amp;' };
    return escapeMap[match];
  });
}

function handleSecureMessage(data) {
  // Обрабатываем только известные типы сообщений
  const allowedMessageTypes = ['cache-update', 'sync-data', 'clear-cache'];
  
  if (allowedMessageTypes.includes(data.type)) {
    switch (data.type) {
      case 'cache-update':
        updateCache(data.payload);
        break;
      case 'sync-data':
        syncDataWithServer(data.payload);
        break;
      case 'clear-cache':
        clearCache();
        break;
    }
  }
}
```

## Безопасность оффлайн-функциональности

### 1. Защита оффлайн-данных

```javascript
// Класс для безопасного хранения оффлайн-данных
class SecureOfflineStorage {
  constructor(encryptionKey) {
    this.encryptionKey = encryptionKey;
    this.dbName = 'SecurePWAData';
    this.version = 1;
  }

  async init() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        // Создаем защищенное хранилище для чувствительных данных
        if (!db.objectStoreNames.contains('encrypted_data')) {
          const store = db.createObjectStore('encrypted_data', { keyPath: 'id' });
          store.createIndex('category', 'category', { unique: false });
          store.createIndex('timestamp', 'timestamp', { unique: false });
        }
      };
    });
  }

  async storeSecureData(category, data) {
    if (!this.db) {
      await this.init();
    }
    
    // Шифруем данные перед сохранением
    const encryptedData = await this.encryptData(JSON.stringify(data));
    
    const transaction = this.db.transaction(['encrypted_data'], 'readwrite');
    const store = transaction.objectStore('encrypted_data');
    
    const record = {
      id: crypto.randomUUID(),
      category: category,
      data: encryptedData,
      timestamp: Date.now()
    };
    
    return new Promise((resolve, reject) => {
      const request = store.add(record);
      request.onsuccess = () => resolve(record.id);
      request.onerror = () => reject(request.error);
    });
  }

  async retrieveSecureData(recordId) {
    if (!this.db) {
      await this.init();
    }
    
    const transaction = this.db.transaction(['encrypted_data'], 'readonly');
    const store = transaction.objectStore('encrypted_data');
    
    return new Promise((resolve, reject) => {
      const request = store.get(recordId);
      
      request.onsuccess = async () => {
        const record = request.result;
        if (record) {
          try {
            // Расшифровываем данные
            const decryptedData = await this.decryptData(record.data);
            resolve(JSON.parse(decryptedData));
          } catch (error) {
            reject(new Error('Ошибка расшифровки данных'));
          }
        } else {
          resolve(null);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }

  async encryptData(data) {
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(data);
    
    // Генерируем случайный IV
    const iv = crypto.getRandomValues(new Uint8Array(12));
    
    // Получаем ключ шифрования
    const key = await this.importKey(this.encryptionKey);
    
    // Шифруем данные
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv: iv },
      key,
      dataBuffer
    );
    
    // Возвращаем зашифрованные данные с IV
    return {
      encrypted: Array.from(new Uint8Array(encrypted)),
      iv: Array.from(iv)
    };
  }

  async decryptData(encryptedPackage) {
    const key = await this.importKey(this.encryptionKey);
    
    const decrypted = await crypto.subtle.decrypt(
      { name: 'AES-GCM', iv: new Uint8Array(encryptedPackage.iv) },
      key,
      new Uint8Array(encryptedPackage.encrypted)
    );
    
    const decoder = new TextDecoder();
    return decoder.decode(decrypted);
  }

  async importKey(keyMaterial) {
    const encoder = new TextEncoder();
    const keyBuffer = encoder.encode(keyMaterial);
    
    return crypto.subtle.importKey(
      'raw',
      keyBuffer,
      { name: 'AES-GCM' },
      false,
      ['encrypt', 'decrypt']
    );
  }
}
```

### 2. Синхронизация данных

```javascript
// Класс для безопасной синхронизации данных
class SecureSyncManager {
  constructor(apiEndpoint, encryptionKey) {
    this.apiEndpoint = apiEndpoint;
    this.encryptionKey = encryptionKey;
    this.syncQueue = [];
  }

  async syncDataWhenOnline() {
    // Проверяем, есть ли подключение к интернету
    if (!navigator.onLine) {
      return;
    }

    // Проверяем, не выполняется ли уже синхронизация
    if (this.isSyncing) {
      return;
    }

    this.isSyncing = true;

    try {
      // Шифруем данные перед отправкой
      const encryptedQueue = await this.encryptSyncQueue();
      
      const response = await fetch(`${this.apiEndpoint}/sync`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${await this.getAuthToken()}`
        },
        body: JSON.stringify(encryptedQueue)
      });

      if (response.ok) {
        // Очищаем очередь после успешной синхронизации
        this.syncQueue = [];
        await this.clearSyncQueue();
      } else {
        throw new Error('Ошибка синхронизации данных');
      }
    } catch (error) {
      console.error('Ошибка синхронизации:', error);
    } finally {
      this.isSyncing = false;
    }
  }

  async addToSyncQueue(data) {
    // Добавляем данные в очередь синхронизации
    const queueItem = {
      id: crypto.randomUUID(),
      data: data,
      timestamp: Date.now(),
      attempts: 0
    };

    this.syncQueue.push(queueItem);
    await this.saveSyncQueue();
  }

  async saveSyncQueue() {
    // Сохраняем очередь в безопасное хранилище
    const storage = new SecureOfflineStorage(this.encryptionKey);
    await storage.storeSecureData('sync-queue', this.syncQueue);
  }

  async clearSyncQueue() {
    // Очищаем очередь синхронизации
    this.syncQueue = [];
    const storage = new SecureOfflineStorage(this.encryptionKey);
    // В реальности нужно реализовать метод очистки для конкретной категории
  }

  async encryptSyncQueue() {
    // Шифруем всю очередь перед отправкой
    const encryptedItems = [];
    
    for (const item of this.syncQueue) {
      const encryptedData = await this.encryptData(item.data);
      encryptedItems.push({
        id: item.id,
        data: encryptedData,
        timestamp: item.timestamp
      });
    }
    
    return encryptedItems;
  }

  async encryptData(data) {
    // Реализация шифрования данных
    const encoder = new TextEncoder();
    const dataBuffer = encoder.encode(JSON.stringify(data));
    
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const key = await this.importKey(this.encryptionKey);
    
    const encrypted = await crypto.subtle.encrypt(
      { name: 'AES-GCM', iv: iv },
      key,
      dataBuffer
    );
    
    return {
      encrypted: Array.from(new Uint8Array(encrypted)),
      iv: Array.from(iv)
    };
  }

  async importKey(keyMaterial) {
    const encoder = new TextEncoder();
    const keyBuffer = encoder.encode(keyMaterial);
    
    return crypto.subtle.importKey(
      'raw',
      keyBuffer,
      { name: 'AES-GCM' },
      false,
      ['encrypt', 'decrypt']
    );
  }

  async getAuthToken() {
    // Получаем токен аутентификации
    const storage = new SecureOfflineStorage(this.encryptionKey);
    const tokenData = await storage.retrieveSecureData('auth-token');
    return tokenData ? tokenData.token : null;
  }
}

// Используем синхронизацию при восстановлении подключения
window.addEventListener('online', () => {
  const syncManager = new SecureSyncManager('/api', 'your-encryption-key');
  syncManager.syncDataWhenOnline();
});
```

## Безопасность push-уведомлений

### 1. Защита подписки на уведомления

```javascript
// Класс для безопасной работы с push-уведомлениями
class SecurePushManager {
  constructor(vapidPublicKey) {
    this.vapidPublicKey = vapidPublicKey;
  }

  async subscribeToPushNotifications() {
    try {
      // Проверяем поддержку push-уведомлений
      if (!('serviceWorker' in navigator && 'PushManager' in window)) {
        throw new Error('Push-уведомления не поддерживаются этим браузером');
      }

      // Регистрируем Service Worker
      const registration = await navigator.serviceWorker.register('/sw.js');
      
      // Запрашиваем разрешение на показ уведомлений
      const permission = await Notification.requestPermission();
      
      if (permission !== 'granted') {
        throw new Error('Разрешение на уведомления не получено');
      }

      // Подписываемся на push-уведомления
      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: this.urlBase64ToUint8Array(this.vapidPublicKey)
      });

      // Отправляем подписку на сервер
      await this.sendSubscriptionToServer(subscription);
      
      return subscription;
    } catch (error) {
      console.error('Ошибка подписки на push-уведомления:', error);
      throw error;
    }
  }

  urlBase64ToUint8Array(base64String) {
    const padding = '='.repeat((4 - base64String.length % 4) % 4);
    const base64 = (base64String + padding)
      .replace(/-/g, '+')
      .replace(/_/g, '/');
    
    const rawData = atob(base64);
    const outputArray = new Uint8Array(rawData.length);
    
    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    
    return outputArray;
  }

  async sendSubscriptionToServer(subscription) {
    // Санитизируем данные подписки
    const sanitizedSubscription = this.sanitizeSubscription(subscription);
    
    const response = await fetch('/api/subscribe', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${await this.getAuthToken()}`
      },
      body: JSON.stringify(sanitizedSubscription)
    });

    if (!response.ok) {
      throw new Error('Ошибка отправки подписки на сервер');
    }
  }

  sanitizeSubscription(subscription) {
    // Удаляем потенциально чувствительные данные
    return {
      endpoint: subscription.endpoint,
      keys: {
        auth: subscription.getKey('auth'),
        p256dh: subscription.getKey('p256dh')
      }
    };
  }

  async getAuthToken() {
    // Получаем токен аутентификации из безопасного хранилища
    const storage = new SecureOfflineStorage('your-encryption-key');
    const tokenData = await storage.retrieveSecureData('auth-token');
    return tokenData ? tokenData.token : null;
  }
}
```

### 2. Обработка push-уведомлений

```javascript
// Обработка push-уведомлений в Service Worker
self.addEventListener('push', (event) => {
  let notificationData = {};

  if (event.data) {
    try {
      // Парсим и санитизируем данные уведомления
      notificationData = sanitizePushData(event.data.json());
    } catch (error) {
      console.error('Ошибка обработки push-данных:', error);
      return;
    }
  }

  const title = notificationData.title || 'Новое уведомление';
  const options = {
    body: notificationData.body || '',
    icon: notificationData.icon || '/icon-192x192.png',
    badge: '/badge-72x72.png',
    tag: notificationData.tag || 'default',
    data: {
      url: notificationData.url || '/'
    }
  };

  // Показываем уведомление
  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  // Санитизируем URL перед открытием
  const safeUrl = sanitizeUrl(event.notification.data.url);
  
  event.waitUntil(
    clients.openWindow(safeUrl)
  );
});

function sanitizePushData(data) {
  // Санитизируем все строки в данных уведомления
  const sanitized = {};
  
  for (const [key, value] of Object.entries(data)) {
    if (typeof value === 'string') {
      sanitized[key] = sanitizeString(value);
    } else if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizePushData(value); // Рекурсивная санитизация
    } else {
      sanitized[key] = value;
    }
  }
  
  return sanitized;
}

function sanitizeString(str) {
  // Удаляем потенциально опасные символы
  return str
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '') // Удаляем скрипты
    .replace(/javascript:/gi, '') // Удаляем JavaScript URL
    .replace(/on\w+\s*=/gi, '') // Удаляем обработчики событий
    .trim();
}

function sanitizeUrl(url) {
  // Проверяем, что URL безопасен
  try {
    const urlObj = new URL(url, self.location.origin);
    
    // Проверяем протокол
    if (!['https:', 'http:'].includes(urlObj.protocol)) {
      return self.location.origin; // Возвращаем безопасный URL по умолчанию
    }
    
    // Проверяем домен (если нужно ограничить)
    if (urlObj.hostname !== self.location.hostname) {
      return self.location.origin;
    }
    
    return urlObj.toString();
  } catch (error) {
    console.error('Небезопасный URL:', url);
    return self.location.origin;
  }
}
```

## Современные подходы к безопасности PWA

### 1. Использование Content Security Policy

Для защиты PWA от XSS-атак:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' 'unsafe-eval'; 
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:; 
               media-src 'self' https:;
               connect-src 'self' https://api.example.com;
               font-src 'self' https://fonts.gstatic.com;
               object-src 'none';
               frame-src 'none';
               base-uri 'self';
               form-action 'self';
               manifest-src 'self';">
```

[[Content-Security-Policy]]

### 2. Защита с помощью Subresource Integrity

```html
<!-- Пример использования SRI для внешних ресурсов -->
<script 
  src="https://cdn.example.com/library.js" 
  integrity="sha384-abcdefghijklmnopqrstuvwxyz123456789"
  crossorigin="anonymous">
</script>
```

[[Subresource-Integrity]]

### 3. Использование современных заголовков безопасности

```javascript
// Пример настройки заголовков безопасности в PWA
const securityHeaders = {
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'geolocation=(), microphone=(), camera=()',
  'Cross-Origin-Embedder-Policy': 'require-corp',
  'Cross-Origin-Opener-Policy': 'same-origin',
  'Cross-Origin-Resource-Policy': 'same-origin'
};
```

[[HTTP-Security-Headers]]

## Заключение

Безопасность PWA требует комплексного подхода, включающего:

1. Защиту Service Worker и кэшированных ресурсов
2. Шифрование оффлайн-данных
3. Безопасную синхронизацию данных
4. Защиту push-уведомлений
5. Использование современных заголовков безопасности
6. Регулярные проверки и обновления систем безопасности

[[Безопасность-в-SPA]]
[[Безопасность-в-приложениях-с-офлайн-режимом]]
[[Secure-Storage]]
[[Шифрование-на-клиенте]]
[[Content-Security-Policy]]
[[HTTP-Security-Headers]]