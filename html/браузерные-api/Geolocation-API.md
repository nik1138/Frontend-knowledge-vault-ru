---
aliases: [Geolocation, Геолокация, Определение местоположения]
tags: [html, browser-api, geolocation, javascript]
---

# Geolocation API

## Обзор

Geolocation API позволяет веб-приложениям получать информацию о местоположении пользователя. Это может быть полезно для персонализации контента, предоставления географически ориентированных услуг или отслеживания маршрутов.

## Основные методы

- `getCurrentPosition()` - получает текущее местоположение пользователя.
- `watchPosition()` - отслеживает изменения местоположения пользователя.
- `clearWatch()` - прекращает отслеживание местоположения.

## Практические рекомендации

### Получение текущего местоположения

```javascript
navigator.geolocation.getCurrentPosition(
  function(position) {
    const latitude = position.coords.latitude;
    const longitude = position.coords.longitude;
    console.log(`Широта: ${latitude}, Долгота: ${longitude}`);
  },
  function(error) {
    console.error('Ошибка получения местоположения:', error.message);
  }
);
```

### Отслеживание изменений местоположения

```javascript
const watchId = navigator.geolocation.watchPosition(
  function(position) {
    const latitude = position.coords.latitude;
    const longitude = position.coords.longitude;
    console.log(`Новое местоположение: ${latitude}, ${longitude}`);
  },
  function(error) {
    console.error('Ошибка отслеживания местоположения:', error.message);
  }
);

// Остановка отслеживания
navigator.geolocation.clearWatch(watchId);
```

## Ограничения и особенности

- Пользователь должен предоставить разрешение на доступ к местоположению.
- Точность местоположения может варьироваться в зависимости от устройства и метода определения.
- API не гарантирует точность местоположения, особенно в помещениях или при слабом сигнале GPS.

## Ссылки

- [[Browser API]]
- [[JavaScript]]
- [[HTML5]]

## Теги

#html #browser-api #geolocation #javascript #web-development