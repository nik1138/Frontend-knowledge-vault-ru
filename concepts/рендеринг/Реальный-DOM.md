---
aliases: [Real DOM, Реальный DOM, Browser DOM]
tags: [frontend, dom, browser, performance, javascript]
---

# Реальный DOM

## Определение

**Реальный DOM (Document Object Model)** - это программный интерфейс для HTML и XML документов, представляющий структуру документа в виде дерева объектов. Каждый узел дерева соответствует элементу или фрагменту текста в документе.

## Структура реального DOM

DOM представляет HTML-документ в виде иерархической структуры:

```
#document
├── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── div
        │   ├── h1
        │   └── p
        └── script
```

## Особенности работы с реальным DOM

### 1. Манипуляции напрямую
```javascript
// Прямое манипулирование DOM
const element = document.getElementById('myElement');
element.style.color = 'red';
element.textContent = 'Новый текст';
```

### 2. Событийная модель
```javascript
// Обработка событий через DOM
const button = document.querySelector('button');
button.addEventListener('click', function() {
  console.log('Кнопка нажата');
});
```

### 3. Атрибуты и свойства
```javascript
const input = document.querySelector('input');
input.value = 'Новое значение'; // Изменение свойства
input.setAttribute('data-value', 'данные'); // Изменение атрибута
```

## Производительность реального DOM

### Проблемы производительности
1. **Высокая стоимость обновлений** - Каждое изменение DOM может вызвать перерисовку (reflow) и перерасчет стилей (repaint)
2. **Синхронные операции** - Обновления DOM происходят синхронно, блокируя основной поток
3. **Сложность оптимизации** - Требуется глубокое понимание браузерных механизмов для эффективной работы

### Измерение производительности
```javascript
// Измерение времени обновления DOM
console.time('DOM update');
const container = document.getElementById('container');

for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Элемент ${i}`;
  container.appendChild(div);
}

console.timeEnd('DOM update');
```

## Оптимизации работы с реальным DOM

### 1. Документные фрагменты
```javascript
// Использование DocumentFragment для минимизации рефлов
const fragment = document.createDocumentFragment();
const container = document.getElementById('container');

for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Элемент ${i}`;
  fragment.appendChild(div);
}

container.appendChild(fragment); // Одно обновление DOM
```

### 2. Batch-обновления
```javascript
// Группировка обновлений для минимизации перерисовок
function batchUpdates(updates) {
  const raf = requestAnimationFrame || setTimeout;
  raf(() => {
    updates.forEach(update => update());
  });
}

// Использование
const updates = [
  () => element1.style.display = 'none',
  () => element2.style.opacity = '0.5',
  () => element3.textContent = 'Новый текст'
];

batchUpdates(updates);
```

### 3. Делегирование событий
```javascript
// Делегирование событий для эффективного управления
document.getElementById('container').addEventListener('click', function(e) {
  if (e.target.classList.contains('button')) {
    console.log('Клик по кнопке');
  }
});
```

## Сравнение с виртуальным DOM

| Аспект | Реальный DOM | Виртуальный DOM |
|--------|--------------|-----------------|
| Местоположение | В браузере | В памяти JavaScript |
| Производительность при частых обновлениях | Низкая | Высокая |
| Сложность управления | Высокая | Низкая |
| Оптимизации | Вручную | Автоматически |
| Прямой доступ | Да | Нет (через фреймворк) |

## Современные API для работы с DOM

### 1. Web Components
```javascript
// Создание кастомного элемента
class MyComponent extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
          color: blue;
        }
      </style>
      <div>Привет из Web Component!</div>
    `;
  }
}

customElements.define('my-component', MyComponent);
```

### 2. Intersection Observer API
```javascript
// Эффективное определение видимости элементов
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Элемент виден');
    }
  });
});

observer.observe(document.getElementById('target'));
```

### 3. Resize Observer API
```javascript
// Отслеживание изменений размера элементов
const resizeObserver = new ResizeObserver(entries => {
  for (let entry of entries) {
    console.log('Размер изменился:', entry.contentRect);
  }
});

resizeObserver.observe(document.getElementById('resizable'));
```

## Практические рекомендации

### 1. Минимизация DOM-запросов
```javascript
// Плохо - множественные запросы к DOM
for (let i = 0; i < 100; i++) {
  document.getElementById('container').innerHTML += `<div>${i}</div>`;
}

// Хорошо - кэширование ссылки
const container = document.getElementById('container');
let html = '';
for (let i = 0; i < 100; i++) {
  html += `<div>${i}</div>`;
}
container.innerHTML = html;
```

### 2. Использование CSS-классов вместо прямых стилей
```javascript
// Плохо - прямое изменение стилей
element.style.color = 'red';
element.style.backgroundColor = 'blue';

// Хорошо - использование CSS-классов
element.classList.add('highlighted');
```

### 3. Оптимизация стилей
```javascript
// Избегать триггеров Reflow/Layout
// Плохо
element.style.left = '10px';
console.log(element.offsetLeft); // Вызовет Reflow
element.style.top = '10px';
console.log(element.offsetTop); // Вызовет Reflow

// Хорошо - группировка стилей
element.style.cssText = 'left: 10px; top: 10px;';
console.log(element.offsetLeft); // Один Reflow
console.log(element.offsetTop);
```

## Современные паттерны работы с DOM

### 1. Shadow DOM
```javascript
// Инкапсуляция стилей и DOM
const shadow = element.attachShadow({ mode: 'closed' });
shadow.innerHTML = `
  <style>
    p { color: red; }
  </style>
  <p>Этот текст стилизован изолированно</p>
`;
```

### 2. Template Elements
```html
<template id="card-template">
  <div class="card">
    <h3 class="title"></h3>
    <p class="content"></p>
  </div>
</template>

<script>
  const template = document.getElementById('card-template');
  const clone = template.content.cloneNode(true);
  clone.querySelector('.title').textContent = 'Заголовок';
  clone.querySelector('.content').textContent = 'Содержимое';
  document.body.appendChild(clone);
</script>
```

## Заключение

Реальный DOM является основой для всех веб-приложений, и понимание его работы критически важно для разработчиков. Несмотря на появление абстракций вроде виртуального DOM, знание основ работы с реальным DOM помогает принимать обоснованные решения при оптимизации производительности.

## См. также

- [[Виртуальный DOM]]
- [[Рендеринг в React]]
- [[Рендеринг в Vue]]
- [[Рендеринг в Svelte]]
- [[Оптимизация производительности]]
- [[Web Components]]