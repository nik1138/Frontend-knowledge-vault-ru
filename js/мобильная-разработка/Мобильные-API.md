---
aliases: [Мобильные веб-API, Mobile Web APIs]
tags: [javascript, mobile, api, web-apis]
---

# Мобильные API

Мобильные API — это специфические веб-API, которые позволяют веб-приложениям взаимодействовать с функциями мобильных устройств. В 2025 году эти API становятся все более важными для создания полноценных мобильных веб-приложений.

## Основные мобильные API

### Device Orientation API

Позволяет получить информацию об ориентации устройства в пространстве.

```javascript
window.addEventListener('deviceorientation', function(event) {
  var alpha = event.alpha; // вращение вокруг оси Z
  var beta = event.beta;   // вращение вокруг оси X
  var gamma = event.gamma; // вращение вокруг оси Y
});
```

### Geolocation API

Позволяет получить текущее местоположение пользователя.

```javascript
navigator.geolocation.getCurrentPosition(function(position) {
  var latitude = position.coords.latitude;
  var longitude = position.coords.longitude;
});
```

### Vibration API

Позволяет вызвать вибрацию на устройстве.

```javascript
navigator.vibrate(200); // вибрация на 200 мс
```

### Battery Status API

Позволяет получить информацию о состоянии батареи устройства.

```javascript
navigator.getBattery().then(function(battery) {
  console.log('Уровень заряда: ' + battery.level * 100 + '%');
});
```

### Network Information API

Позволяет получить информацию о типе подключения к сети.

```javascript
if ('connection' in navigator) {
  console.log('Тип соединения: ' + navigator.connection.effectiveType);
}
```

## Особенности использования

Большинство мобильных API требуют разрешения пользователя для доступа к данным или функциям устройства. Также важно учитывать, что не все API поддерживаются во всех браузерах.

## Особенности российского рынка

В России важна поддержка офлайн-режима из-за возможных проблем с качеством связи в удаленных регионах. Также учитывайте, что пользователи могут быть чувствительны к расходу заряда батареи, поэтому оптимизация использования API важна.

## См. также

- [[Touch-события]]
- [[PWA]]
- [[Сервис-воркеры]]