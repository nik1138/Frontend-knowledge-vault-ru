---
aliases: [Service Workers, Сервис-воркеры, Офлайн-функциональность]
tags: [html, browser-api, service-workers, javascript]
---

# Service Workers

## Обзор

Service Workers - это скрипты, которые выполняются в отдельном потоке и могут перехватывать и обрабатывать сетевые запросы, кэшировать ресурсы и обеспечивать офлайн-функциональность. Они действуют как прокси между веб-приложением, браузером и сетью.

## Основные события

- `install` - происходит при установке сервис-воркера.
- `activate` - происходит после установки, когда воркер становится активным.
- `fetch` - происходит при сетевых запросах.

## Практические рекомендации

### Регистрация Service Worker

```javascript
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(function(registration) {
      console.log('Service Worker зарегистрирован с scope:', registration.scope);
    })
    .catch(function(error) {
      console.log('Service Worker регистрация не удалась:', error);
    });
}
```

### Кэширование ресурсов

```javascript
// sw.js
const CACHE_NAME = 'my-site-cache-v1';
const urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        return cache.addAll(urlsToCache);
      })
  );
});

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
```

## Ограничения и особенности

- Service Workers работают только по HTTPS (кроме localhost).
- Они не имеют доступа к DOM.
- Service Workers могут быть обновлены, но требуют специального подхода к управлению версиями.

## Ссылки

- [[Web Workers]]
- [[Browser API]]
- [[Cache API]]

## Теги

#html #browser-api #service-workers #javascript #web-development