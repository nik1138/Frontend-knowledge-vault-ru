---
aliases: [HTML шаблоны и микрозадачи, templates and microtasks, шаблоны HTML]
tags: [html, javascript, templates, microtasks, web-components, dom]
---

# HTML-шаблоны и микрозадачи

## Обзор

HTML-шаблоны и микрозадачи являются важными компонентами асинхронного программирования в веб-разработке. В 2025 году эти концепции особенно актуальны для создания производительных и интерактивных веб-приложений, соответствующих современным стандартам.

## HTML-шаблоны

### Элемент template

Элемент `<template>` позволяет создавать инертные блоки HTML-кода, которые можно клонировать и вставлять в DOM по мере необходимости:

```html
<template id="card-template">
    <div class="card">
        <h3 class="title"></h3>
        <p class="content"></p>
    </div>
</template>
```

### Использование шаблонов

```javascript
// Получаем шаблон
const template = document.getElementById('card-template');
const clone = template.content.cloneNode(true);

// Заполняем данными
clone.querySelector('.title').textContent = 'Заголовок карточки';
clone.querySelector('.content').textContent = 'Содержимое карточки';

// Вставляем в DOM
document.body.appendChild(clone);
```

## Микрозадачи

### Что такое микрозадачи

Микрозадачи - это функции, которые планируются к выполнению сразу после завершения текущего блока кода и перед следующим циклом событий. Они имеют более высокий приоритет, чем макрозадачи.

### Создание микрозадач

```javascript
// С помощью Promise
Promise.resolve().then(() => {
    console.log('Это микрозадача');
});

// С помощью queueMicrotask
queueMicrotask(() => {
    console.log('Это также микрозадача');
});
```

## Взаимодействие шаблонов и микрозадач

### Асинхронное обновление DOM с использованием шаблонов

```javascript
class TemplateRenderer {
    constructor(templateId) {
        this.template = document.getElementById(templateId).content;
    }

    async render(data) {
        // Создаем микрозадачу для отложенного рендеринга
        return new Promise(resolve => {
            queueMicrotask(() => {
                const fragment = this.template.cloneNode(true);
                
                // Заполняем данными
                const title = fragment.querySelector('.title');
                const content = fragment.querySelector('.content');
                
                if (title) title.textContent = data.title;
                if (content) content.textContent = data.content;
                
                resolve(fragment);
            });
        });
    }
}

// Использование
const renderer = new TemplateRenderer('card-template');
renderer.render({title: 'Новости', content: 'Содержимое новости'})
    .then(fragment => {
        document.body.appendChild(fragment);
    });
```

## Практические рекомендации

### Оптимизация рендеринга

В 2025 году российские разработчики сталкиваются с необходимостью оптимизации под разные сценарии:

1. **Оптимизация для слабых устройств**
   - Использование шаблонов для минимизации перерисовки DOM
   - Применение микрозадач для пакетной обработки изменений

2. **Учет особенностей российского сегмента**
   - Работа с кириллическими шрифтами и текстом
   - Оптимизация под разные провайдеров интернета

### Пример оптимизированного рендеринга

```javascript
class OptimizedRenderer {
    constructor(templateId) {
        this.template = document.getElementById(templateId).content;
        this.pendingTasks = [];
    }

    queueRender(data) {
        return new Promise(resolve => {
            this.pendingTasks.push({data, resolve});
            
            // Планируем выполнение всех задач как одну микрозадачу
            queueMicrotask(() => {
                if (this.pendingTasks.length > 0) {
                    const tasks = this.pendingTasks.splice(0);
                    
                    const fragments = tasks.map(task => {
                        const fragment = this.template.cloneNode(true);
                        
                        // Заполняем данными
                        const title = fragment.querySelector('.title');
                        const content = fragment.querySelector('.content');
                        
                        if (title) title.textContent = task.data.title;
                        if (content) content.textContent = task.data.content;
                        
                        return {fragment, resolver: task.resolve};
                    });
                    
                    // Вставляем все фрагменты за один раз
                    fragments.forEach(({fragment, resolver}) => {
                        document.body.appendChild(fragment);
                        resolver(fragment);
                    });
                }
            });
        });
    }
}
```

## Современные паттерны

### Комбинирование шаблонов с веб-компонентами

```javascript
class CardComponent extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({mode: 'open'});
        
        // Используем шаблон
        const template = document.getElementById('card-template').content;
        this.shadowRoot.appendChild(template.cloneNode(true));
    }
    
    connectedCallback() {
        // Асинхронная инициализация
        queueMicrotask(() => {
            this.updateContent();
        });
    }
    
    updateContent() {
        const title = this.shadowRoot.querySelector('.title');
        const content = this.shadowRoot.querySelector('.content');
        
        if (title) title.textContent = this.getAttribute('title') || '';
        if (content) content.textContent = this.getAttribute('content') || '';
    }
}

customElements.define('card-component', CardComponent);
```

> [!tip] Совет
> Используйте микрозадачи для группировки DOM-операций, чтобы минимизировать перерисовку и улучшить производительность.

## Связанные темы

- [[Асинхронные-скрипты]]
- [[Promise-в-HTML-API]]
- [[Работа-с-таймерами]]
- [[Обработка-ошибок]]

## Источники

- [MDN Web Docs: HTML Template](https://developer.mozilla.org/ru/docs/Web/HTML/Element/template)
- [MDN Web Docs: Microtasks](https://developer.mozilla.org/ru/docs/Web/API/HTML_DOM_API/Microtask_guide)
- [Web Components Specifications](https://www.w3.org/wiki/WebComponents/)