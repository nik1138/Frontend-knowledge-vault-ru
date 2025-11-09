---
aliases: ["Продвинутый HTML", "HTML продвинутые возможности"]
tags: 
  - html
  - web-components
  - canvas
  - svg
  - web-workers
  - service-workers
  - best-practices
---

# Продвинутые темы HTML

## Введение

HTML — это основа веб-разработки, но за пределами базовых тегов существуют мощные возможности, позволяющие создавать интерактивные и высокопроизводительные веб-приложения. В этом руководстве рассмотрим продвинутые темы HTML, включая шаблоны, веб-компоненты, графику, асинхронные операции и технологии оффлайн-работы.

## Шаблоны HTML (template)

Элемент `<template>` позволяет создавать фрагменты HTML, которые не отображаются при загрузке страницы, но могут быть клонированы и использованы позже.

```html
<template id="card-template">
  <div class="card">
    <h3 data-content="title"></h3>
    <p data-content="description"></p>
  </div>
</template>

<script>
  const template = document.getElementById('card-template');
  const clone = template.content.cloneNode(true);
  clone.querySelector('[data-content="title"]').textContent = 'Новая карточка';
  document.body.appendChild(clone);
</script>
```

## Веб-компоненты

Веб-компоненты — это набор веб-технологий, позволяющих создавать переиспользуемые пользовательские элементы с инкапсулированным функционалом.

### Custom Elements

Создание пользовательских HTML-элементов:

```javascript
class CustomCard extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        :host { display: block; border: 1px solid #ccc; }
        h3 { color: #333; }
      </style>
      <h3><slot name="title"></slot></h3>
      <p><slot name="content"></slot></p>
    `;
  }
}

customElements.define('custom-card', CustomCard);
```

```html
<custom-card>
  <span slot="title">Заголовок карточки</span>
  <span slot="content">Описание карточки</span>
</custom-card>
```

### Shadow DOM

Shadow DOM позволяет инкапсулировать стили и разметку, изолируя их от остальной страницы:

```javascript
const shadow = element.attachShadow({ mode: 'closed' }); // или 'open'
shadow.innerHTML = `
  <style>
    p { color: blue; }
  </style>
  <p>Этот текст изолирован от основного DOM</p>
`;
```

## Canvas и SVG

### Canvas

Canvas предоставляет API для рисования графики на лету:

```html
<canvas id="myCanvas" width="400" height="200"></canvas>
<script>
  const canvas = document.getElementById('myCanvas');
  const ctx = canvas.getContext('2d');
  
  ctx.fillStyle = '#FF0000';
  ctx.fillRect(10, 10, 150, 80);
  
  ctx.beginPath();
  ctx.arc(200, 100, 40, 0, 2 * Math.PI);
  ctx.stroke();
</script>
```

### SVG

SVG — масштабируемая векторная графика, идеально подходящая для иконок и диаграмм:

```html
<svg width="100" height="100">
  <circle cx="50" cy="50" r="40" fill="blue" />
  <rect x="10" y="10" width="80" height="80" fill="red" opacity="0.5" />
</svg>
```

## Web Workers

Web Workers позволяют выполнять JavaScript в фоновом потоке, не блокируя UI:

```javascript
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ command: 'start', data: [1, 2, 3, 4] });

worker.onmessage = function(e) {
  console.log('Результат:', e.data);
};

// worker.js
self.onmessage = function(e) {
  const result = e.data.data.map(x => x * x);
  self.postMessage(result);
};
```

## Микроформаты

Микроформаты — это способ добавления семантики к HTML-разметке:

```html
<div class="h-card">
  <h1 class="p-name">Иван Иванов</h1>
  <div class="p-org">Компания</div>
  <a class="u-url" href="https://example.com">Профиль</a>
</div>
```

## Веб-сокеты

WebSocket обеспечивает двустороннюю связь между клиентом и сервером:

```javascript
const socket = new WebSocket('ws://localhost:8080');

socket.onopen = function(event) {
  console.log('Соединение установлено');
  socket.send('Привет, сервер!');
};

socket.onmessage = function(event) {
  console.log('Получено сообщение:', event.data);
};
```

## Drag-and-Drop API

API перетаскивания позволяет создавать интерактивные элементы:

```html
<div id="draggable" draggable="true">Перетащи меня</div>
<div id="dropzone">Сюда</div>

<script>
  const draggable = document.getElementById('draggable');
  const dropzone = document.getElementById('dropzone');
  
  draggable.addEventListener('dragstart', e => {
    e.dataTransfer.setData('text/plain', draggable.id);
  });
  
  dropzone.addEventListener('drop', e => {
    e.preventDefault();
    const id = e.dataTransfer.getData('text/plain');
    dropzone.appendChild(document.getElementById(id));
  });
  
  dropzone.addEventListener('dragover', e => {
    e.preventDefault();
  });
</script>
```

## Storage API

### LocalStorage и SessionStorage

```javascript
// Сохранение данных
localStorage.setItem('user', JSON.stringify({ name: 'Иван', age: 30 }));
sessionStorage.setItem('temp', 'временные данные');

// Получение данных
const user = JSON.parse(localStorage.getItem('user'));
```

### IndexedDB

Более сложное хранилище для структурированных данных:

```javascript
const request = indexedDB.open('MyDB', 1);

request.onupgradeneeded = function(event) {
  const db = event.target.result;
  const objectStore = db.createObjectStore('users', { keyPath: 'id' });
  objectStore.createIndex('name', 'name', { unique: false });
};

request.onsuccess = function(event) {
  const db = event.target.result;
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  store.add({ id: 1, name: 'Иван' });
};
```

## Offline технологии

### Service Workers

Service Worker — это скрипт, который работает в фоне и позволяет создавать оффлайн-приложения:

```javascript
// sw.js
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => {
      return response || fetch(event.request);
    })
  );
});

// Регистрация
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js');
}
```

### Web App Manifest

Файл `manifest.json` позволяет сделать веб-приложение похожим на нативное:

```json
{
  "name": "Мое веб-приложение",
  "short_name": "МВП",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

```html
<link rel="manifest" href="/manifest.json">
```

## Лучшие практики

- Используйте семантические элементы HTML5
- Обеспечивайте доступность (a11y) через ARIA-атрибуты
- Минимизируйте использование inline-стилей
- Проверяйте совместимость с браузерами
- Используйте HTTPS для безопасной работы с API
- Оптимизируйте производительность при использовании Web Workers

## Связи с другими файлами

- [[html/basics/basics]] — основы HTML
- [[css/advanced/advanced]] — продвинутые возможности CSS
- [[js/advanced/advanced]] — продвинутый JavaScript
- [[architecture/component-architecture/components]] — архитектура компонентов
- [[architecture/accessibility/accessibility]] — доступность веб-приложений
- [[architecture/caching/caching]] — стратегии кэширования
- [[architecture/api-integration/api-integration]] — интеграция с API