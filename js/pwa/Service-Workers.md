---
aliases: ["Сервис-воркеры", "Service Worker", "SW"]
tags: ["#pwa", "#javascript", "#web", "#caching", "#offline"]
---

# Service Workers

Service Workers - это скрипты, которые работают в фоновом режиме и позволяют создавать прогрессивные веб-приложения (PWA) с возможностями оффлайн-работы, кэширования и push-уведомлений. Они действуют как прокси-сервер между веб-приложением, браузером и сетью.

## Основные особенности

- **Изолированное выполнение**: Service Worker работает в собственном потоке, независимо от основного потока JavaScript
- **Отсутствие доступа к DOM**: Не может напрямую взаимодействовать с элементами страницы
- **Прокси-функциональность**: Может перехватывать и обрабатывать сетевые запросы
- **Кэширование**: Обеспечивает надежное кэширование ресурсов для оффлайн-доступа
- **Push-уведомления**: Поддерживает получение и отображение уведомлений

## Жизненный цикл Service Worker

Service Worker проходит через несколько этапов жизненного цикла:

1. **Регистрация** - приложение регистрирует Service Worker
2. **Установка** - Service Worker устанавливается и готов к работе
3. **Активация** - Service Worker активируется и готов перехватывать события
4. **Использование** - Service Worker обрабатывает события fetch и сообщения
5. **Обновление** - при изменении файла Service Worker может быть обновлен

```javascript
// Регистрация Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(function(registration) {
      console.log('Service Worker зарегистрирован с scope:', registration.scope);
    })
    .catch(function(error) {
      console.log('Регистрация Service Worker не удалась:', error);
    });
}
```

### Событие установки

```javascript
// В файле sw.js
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open('my-cache-v1')
      .then(function(cache) {
        return cache.addAll([
          '/',
          '/styles/main.css',
          '/scripts/main.js',
          '/images/logo.png',
          '/offline.html'
        ]);
      })
  );
});
```

### Событие активации

```javascript
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.filter(function(cacheName) {
          // Удаляем старые кэши
          return cacheName.startsWith('my-cache-') &&
                 cacheName !== 'my-cache-v1';
        }).map(function(cacheName) {
          return caches.delete(cacheName);
        })
      );
    })
  );
});
```

### Обработка сетевых запросов

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Возвращаем кэшированный ресурс или делаем сетевой запрос
        return response || fetch(event.request);
      })
  );
});
```

## Типы кэширования

### 1. Cache-first стратегия

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        return response || fetch(event.request)
          .then(function(fetchResponse) {
            // Кэшируем ответ для будущих запросов
            caches.open('my-cache-v1')
              .then(function(cache) {
                cache.put(event.request, fetchResponse.clone());
              });
            return fetchResponse;
          });
      })
  );
});
```

### 2. Network-first стратегия

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    fetch(event.request)
      .then(function(fetchResponse) {
        // Кэшируем успешный ответ
        caches.open('my-cache-v1')
          .then(function(cache) {
            cache.put(event.request, fetchResponse.clone());
          });
        return fetchResponse;
      })
      .catch(function() {
        // Возвращаем кэшированный ресурс при ошибке сети
        return caches.match(event.request);
      })
  );
});
```

### 3. Network-only стратегия

```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(fetch(event.request));
});
```

## Push-уведомления

Service Workers позволяют реализовать push-уведомления:

```javascript
// Регистрация для push-уведомлений
self.addEventListener('push', function(event) {
  const options = {
    body: event.data ? event.data.text() : 'Новое уведомление',
    icon: '/images/icon-192x192.png',
    badge: '/images/badge-72x72.png',
    vibrate: [100, 50, 100],
    data: {
      dateOfArrival: Date.now(),
      primaryKey: 1
    }
  };

  event.waitUntil(
    self.registration.showNotification('Мое приложение', options)
  );
});

// Обработка клика по уведомлению
self.addEventListener('notificationclick', function(event) {
  event.notification.close();
  
  event.waitUntil(
    clients.openWindow('/dashboard')
  );
});
```

## Обработка оффлайн-режима

```javascript
self.addEventListener('fetch', function(event) {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request)
        .catch(function() {
          return caches.open('my-cache-v1')
            .then(function(cache) {
              return cache.match('/offline.html');
            });
        })
    );
  }
});
```

## Рекомендации по безопасности

- Service Workers могут быть зарегистрированы только на защищенных соединениях (HTTPS)
- Необходимо использовать HTTPS даже на локальных серверах для разработки
- Service Worker имеет доступ к куки и другим аутентификационным данным
- Рекомендуется использовать Content Security Policy для ограничения возможностей скриптов

## Совместимость и поддержка

Все современные браузеры поддерживают Service Workers:
- Chrome: с версии 40
- Firefox: с версии 44
- Safari: с версии 11.1
- Edge: с версии 17

> [!warning] Важно
> Service Workers не поддерживаются в Internet Explorer. Для поддержки старых браузеров необходимо использовать полифиллы или альтернативные решения.

## Практические рекомендации для российских разработчиков

В 2025 году в России наблюдается рост интереса к PWA-технологиям, особенно в условиях ограничений на традиционные магазины приложений. Service Workers становятся важной частью архитектуры веб-приложений, особенно для:

- Электронной коммерции с оффлайн-каталогами
- Приложений для банковского сектора
- Государственных информационных систем
- Локальных решений, не требующих публикации в магазинах приложений

При разработке Service Workers для российского рынка важно учитывать:
- Особенности работы с кириллическими символами в URL
- Требования к локализации и интернационализации
- Возможные ограничения на push-уведомления
- Необходимость поддержки старых версий браузеров в корпоративных средах

## Связанные темы

- [[Кэширование]]
- [[Оффлайн-работа]]
- [[Web-App-Manifest]]
- [[Браузерные API]]
- [[Современные практики JavaScript]]