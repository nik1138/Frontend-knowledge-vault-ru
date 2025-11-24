---
aliases: ["Сервис-воркеры", "Service Worker", "SW"]
tags: ["#pwa", "#typescript", "#web", "#offline", "#caching"]
---

# Service Workers

Service Workers - это скрипты, которые запускаются в фоне браузера, отдельно от веб-страницы, и действуют как прокси-сервер между веб-приложением, браузером и сетью. Они позволяют реализовать функциональность офлайн-приложений, push-уведомлений и кэширования ресурсов.

## Основные возможности Service Workers

- **Кэширование ресурсов**: Позволяет кэшировать статические ресурсы (HTML, CSS, JS, изображения) для быстрого доступа и работы офлайн.
- **Офлайн-функциональность**: Обеспечивает работу приложения без подключения к интернету.
- **Push-уведомления**: Поддержка получения и отображения push-уведомлений.
- **Перехват сетевых запросов**: Возможность перехватывать и изменять сетевые запросы, реализуя кастомную логику кэширования.

## Жизненный цикл Service Worker

Service Worker проходит через несколько этапов жизненного цикла:

1. **Регистрация**: Скрипт Service Worker регистрируется из основного контекста JavaScript.
2. **Установка**: Браузер загружает и устанавливает новый Service Worker.
3. **Активация**: После установки Service Worker активируется и получает контроль над страницей.
4. **Обслуживание**: Активный Service Worker перехватывает сетевые запросы и обрабатывает события.

## Регистрация Service Worker

Для регистрации Service Worker в основном JavaScript-файле приложения необходимо использовать метод `navigator.serviceWorker.register()`:

```typescript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((registration) => {
        console.log('ServiceWorker зарегистрирован с scope:', registration.scope);
      })
      .catch((error) => {
        console.error('ServiceWorker регистрация не удалась:', error);
      });
  });
}
```

## Основной Service Worker (sw.js)

Создадим базовый Service Worker с кэшированием ресурсов:

```typescript
const CACHE_NAME = 'my-pwa-cache-v1';
const urlsToCache = [
  '/',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
];

self.addEventListener('install', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('Открытие кэша');
        return cache.addAll(urlsToCache);
      })
  );
});

self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Возвращаем кэшированный ресурс или делаем сетевой запрос
        if (response) {
          return response;
        }
        return fetch(event.request);
      }
    )
  );
});
```

## Стратегии кэширования

Существует несколько стратегий кэширования, которые можно реализовать в Service Worker:

### 1. Cache First (Кэш первым)

Приоритетно возвращать ресурс из кэша, а в случае отсутствия - делать сетевой запрос и кэшировать результат.

```typescript
self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        return response || fetch(event.request)
          .then((networkResponse) => {
            // Кэшируем сетевой ответ
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, networkResponse.clone());
              });
            return networkResponse;
          });
      })
  );
});
```

### 2. Network First (Сеть первым)

Сначала делается сетевой запрос, и только при его неудаче возвращается кэшированный ресурс.

```typescript
self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    fetch(event.request)
      .then((networkResponse) => {
        // Кэшируем сетевой ответ
        caches.open(CACHE_NAME)
          .then((cache) => {
            cache.put(event.request, networkResponse.clone());
          });
        return networkResponse;
      })
      .catch(() => {
        // Возвращаем кэшированный ресурс в случае ошибки сети
        return caches.match(event.request);
      })
  );
});
```

### 3. Stale-While-Revalidate (Устаревшее при обновлении)

Возвращается кэшированный ресурс, но параллельно делается сетевой запрос для обновления кэша.

```typescript
self.addEventListener('fetch', (event: FetchEvent) => {
  event.respondWith(
    caches.match(event.request)
      .then((cachedResponse) => {
        const networkRequest = fetch(event.request)
          .then((networkResponse) => {
            // Кэшируем сетевой ответ
            caches.open(CACHE_NAME)
              .then((cache) => {
                cache.put(event.request, networkResponse.clone());
              });
            return networkResponse;
          });

        return cachedResponse || networkRequest;
      })
  );
});
```

## Обновление Service Worker

При изменении кода Service Worker необходимо обновить кэш. Это можно сделать, изменив `CACHE_NAME` и добавив логику удаления старых кэшей:

```typescript
const CACHE_NAME = 'my-pwa-cache-v2'; // Обновленная версия

self.addEventListener('activate', (event: ExtendableEvent) => {
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
```

## Типизация в TypeScript

Для корректной типизации событий Service Worker в TypeScript рекомендуется использовать соответствующие типы:

```typescript
declare const self: ServiceWorkerGlobalScope;

self.addEventListener('install', (event: ExtendableEvent) => {
  // Обработка установки
});

self.addEventListener('fetch', (event: FetchEvent) => {
  // Обработка fetch-события
});

self.addEventListener('activate', (event: ExtendableEvent) => {
  // Обработка активации
});
```

## Полезные ресурсы

- [Официальная документация MDN по Service Workers](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API)
- [[Web-App-Manifest]] - для полной настройки PWA
- [[Офлайн-работа]] - стратегии работы без подключения к интернету
- [[Push-уведомления]] - реализация уведомлений
- [[Установка-PWA]] - процесс установки приложения

## Заключение

Service Workers являются ключевым компонентом Progressive Web Apps, обеспечивая офлайн-функциональность, кэширование и push-уведомления. Использование TypeScript помогает избежать ошибок и улучшает поддерживаемость кода Service Worker.