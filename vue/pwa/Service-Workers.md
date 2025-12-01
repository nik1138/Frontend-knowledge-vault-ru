# Service Workers в PWA на Vue.js

**Alias**: [[Сервис-воркеры]]

**Tags**: #vue #pwa #service-workers #web-development #performance #offline

Service Workers - это скрипты, которые браузер запускает в фоновом режиме, отдельно от веб-страницы. Они позволяют PWA работать автономно, управлять кэшированием и обеспечивать оффлайн-функциональность.

## Основные функции Service Workers

- Перехват и кэширование сетевых запросов
- Обеспечение оффлайн-доступа к контенту
- Управление фоновыми синхронизациями
- Отправка push-уведомлений

## Интеграция в Vue.js

Для использования Service Workers в Vue.js приложениях рекомендуется использовать плагин `@vue/cli-plugin-pwa`, который автоматически настраивает базовую конфигурацию.

```javascript
// vue.config.js
module.exports = {
  pwa: {
    workboxOptions: {
      skipWaiting: true,
      clientsClaim: true
    }
  }
}
```

## Работа с кэшированием

Service Workers позволяют реализовать различные стратегии кэширования:
- Cache First
- Network First
- Cache Only
- Network Only

## Особенности в российском контексте (2025)

- Учет ограничений на доступ к иностранным сервисам
- Локализация кэшируемых ресурсов
- Работа с отечественными CDN

## Связанные темы

- [[Web-App-Manifest]]
- [[Оффлайн-функционал]]
- [[Push-уведомления]]
- [[Установка-PWA]]

## Подробная реализация

Service Worker - это JavaScript файл, который работает в отдельном потоке и не имеет доступа к DOM. Он может перехватывать сетевые запросы, кэшировать ресурсы и управлять фоновыми задачами.

При регистрации Service Worker, важно учитывать следующие аспекты:

1. **Область действия (scope)** - Service Worker может управлять только страницами, находящимися в той же директории или поддиректориях, где находится сам файл SW.

2. **Жизненный цикл** - Service Worker проходит через несколько состояний: installing, installed, activating, activated, и может быть остановлен браузером при неактивности.

3. **Обновление** - при изменении файла Service Worker, браузер загружает новую версию, но старая продолжает обрабатывать запросы до тех пор, пока все вкладки с приложением не будут закрыты.

## Пример регистрации Service Worker

```javascript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((registration) => {
        console.log('SW registered: ', registration);
      })
      .catch((registrationError) => {
        console.log('SW registration failed: ', registrationError);
      });
  });
}
```

## Кэширование ресурсов

Service Worker может кэшировать ресурсы при установке:

```javascript
const CACHE_NAME = 'my-pwa-v1';
const urlsToCache = [
  '/',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        return cache.addAll(urlsToCache);
      })
  );
});
```

## Обработка сетевых запросов

Service Worker может перехватывать и обрабатывать сетевые запросы:

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((response) => {
        // Возвращаем кэшированный ответ или делаем сетевой запрос
        return response || fetch(event.request);
      }
    )
  );
});
```

## Лучшие практики

- Используйте Workbox для упрощения работы с Service Workers
- Обрабатывайте ошибки и обеспечивайте fallback для оффлайн-режима
- Регулярно обновляйте кэшируемые ресурсы
- Тестируйте поведение при медленном интернете и оффлайн