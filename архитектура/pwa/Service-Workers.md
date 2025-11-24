---
aliases: [Service Worker, Service Workers в PWA]
tags: [pwa, service-worker, caching, offline, web-api]
---

# Service Workers

## Обзор

Service Workers - это скрипты, которые выполняются в фоновом режиме и действуют как прокси между веб-приложением и сетью. Они позволяют реализовать оффлайн-функциональность, кэширование ресурсов и управление сетевыми запросами.

## Основные возможности

- **Кэширование ресурсов**: Сохранение статических и динамических ресурсов для оффлайн-доступа
- **Оффлайн-функциональность**: Обеспечение работы приложения при отсутствии интернет-соединения
- **Push-уведомления**: Поддержка уведомлений даже когда вкладка закрыта
- **Перехват сетевых запросов**: Управление тем, как и когда загружаются ресурсы
- **Фоновая синхронизация**: Синхронизация данных когда восстанавливается соединение

## Жизненный цикл Service Worker

Service Worker проходит через несколько этапов жизненного цикла:

1. **Регистрация**: Скрипт регистрируется в главном JavaScript-файле приложения
2. **Установка**: Service Worker устанавливается и готов к работе
3. **Активация**: Service Worker становится активным и может перехватывать события
4. **Обслуживание**: Service Worker перехватывает события fetch и push
5. **Обновление**: При изменении скрипта происходит обновление Service Worker

## Пример регистрации Service Worker

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((registration) => {
        console.log('ServiceWorker зарегистрирован с scope:', registration.scope);
      })
      .catch((error) => {
        console.log('Регистрация ServiceWorker не удалась:', error);
      });
  });
}
```

## Основные события Service Worker

### Install Event
Событие срабатывает при установке Service Worker. Используется для кэширования основных ресурсов:

```javascript
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        return cache.addAll([
          '/',
          '/styles/main.css',
          '/scripts/main.js',
          '/images/logo.png'
        ]);
      })
  );
});
```

### Fetch Event
Событие срабатывает при каждом сетевом запросе. Позволяет управлять кэшированием и оффлайн-доступом:

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Возвращаем кэшированный ответ, если он есть
        if (response) {
          return response;
        }
        // Иначе делаем сетевой запрос
        return fetch(event.request);
      })
  );
});
```

### Activate Event
Событие срабатывает при активации Service Worker. Используется для очистки старых кэшей:

```javascript
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.filter((cacheName) => {
          // Удаляем старые кэши
          return cacheName !== CACHE_NAME;
        }).map((cacheName) => {
          return caches.delete(cacheName);
        })
      );
    })
  );
});
```

## Стратегии кэширования

### Cache First
Сначала проверяется кэш, затем сеть. Подходит для статических ресурсов:

```javascript
const cacheFirst = async (request) => {
  const cachedResponse = await caches.match(request);
  if (cachedResponse) {
    return cachedResponse;
  }
  return fetch(request);
};
```

### Network First
Сначала сеть, затем кэш. Подходит для часто обновляемых данных:

```javascript
const networkFirst = async (request) => {
  try {
    const networkResponse = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, networkResponse.clone());
    return networkResponse;
  } catch (error) {
    const cachedResponse = await caches.match(request);
    if (cachedResponse) {
      return cachedResponse;
    }
    // Возвращаем оффлайн-страницу
    return caches.match('/offline.html');
  }
};
```

### Stale While Revalidate
Возвращает кэшированный ответ, но также обновляет кэш в фоне:

```javascript
const staleWhileRevalidate = async (request) => {
  const cachedResponse = await caches.match(request);
  
  // Выполняем сетевой запрос в фоне
  const networkResponse = await fetch(request);
  const cache = await caches.open(CACHE_NAME);
  cache.put(request, networkResponse.clone());
  
  // Возвращаем кэшированный ответ, если он есть, иначе - свежий
  return cachedResponse || networkResponse;
};
```

## Особенности для российских реалий 2025 года

В 2025 году в России при использовании Service Workers важно учитывать:

- Ограничения на доступ к зарубежным ресурсам, что требует локального хранения критических зависимостей
- Необходимость поддержки различных провайдеров связи с нестабильным интернетом
- Повышенные требования к безопасности и защите персональных данных
- Возможные ограничения на push-уведомления в государственных и корпоративных сетях

Разработчикам необходимо тщательно планировать стратегии кэширования с учетом российской инфраструктуры и законодательства.

## Лучшие практики

1. **Используйте HTTPS**: Service Workers работают только на защищенных соединениях
2. **Оптимизируйте размер кэша**: Избегайте чрезмерного кэширования ресурсов
3. **Обрабатывайте обновления**: Правильно обновляйте Service Workers и кэши
4. **Тестируйте оффлайн-режим**: Убедитесь, что приложение корректно работает без интернета
5. **Мониторьте производительность**: Отслеживайте влияние Service Workers на производительность

## Связанные темы

- [[Архитектура PWA]]
- [[Web App Manifest]]
- [[Оффлайн работа]]
- [[Кэширование]]

## Заключение

Service Workers являются ключевым компонентом архитектуры PWA, обеспечивающим оффлайн-функциональность и улучшенный пользовательский опыт. Правильное использование этих технологий позволяет создавать надежные и быстрые веб-приложения, которые работают даже при нестабильном интернет-соединении.