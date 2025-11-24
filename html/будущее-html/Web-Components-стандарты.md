---
aliases: ["Веб-компоненты", "Custom Elements", "Shadow DOM", "HTML Components"]
tags: ["#web-components", "#custom-elements", "#frontend", "#standards", "#modularity"]
---

# Web Components стандарты

## Обзор

Web Components - это набор веб-стандартов, позволяющих создавать переиспользуемые пользовательские элементы с инкапсулированным функционалом. В 2025 году эти стандарты становятся ключевым элементом будущего HTML, обеспечивая модульность и переиспользуемость компонентов без зависимости от фреймворков.

## Основные технологии Web Components

### Custom Elements

Custom Elements позволяют разработчикам определять новые HTML-теги с собственным поведением:

```javascript
class UserProfile extends HTMLElement {
  constructor() {
    super();
    
    // Создаем теневой DOM
    const shadow = this.attachShadow({mode: 'open'});
    
    // Шаблон компонента
    const template = document.createElement('template');
    template.innerHTML = `
      <style>
        :host {
          display: block;
          border: 1px solid #ccc;
          border-radius: 8px;
          padding: 16px;
        }
        
        .avatar {
          width: 50px;
          height: 50px;
          border-radius: 50%;
        }
        
        .name {
          font-weight: bold;
          margin-left: 10px;
        }
      </style>
      
      <img class="avatar" src="" alt="Avatar">
      <span class="name"></span>
    `;
    
    shadow.appendChild(template.content.cloneNode(true));
  }
  
  static get observedAttributes() {
    return ['name', 'avatar'];
  }
  
  attributeChangedCallback(name, oldValue, newValue) {
    const shadow = this.shadowRoot;
    
    switch(name) {
      case 'name':
        shadow.querySelector('.name').textContent = newValue;
        break;
      case 'avatar':
        shadow.querySelector('.avatar').src = newValue;
        break;
    }
  }
}

customElements.define('user-profile', UserProfile);
```

Использование в HTML:

```html
<user-profile name="Иван Петров" avatar="/images/ivan.jpg"></user-profile>
```

### Shadow DOM

Shadow DOM обеспечивает инкапсуляцию стилей и DOM-элементов компонента:

- Изолирует внутреннюю структуру компонента от внешнего CSS
- Предотвращает конфликты стилей
- Позволяет создавать независимые компоненты

> [!tip]
> В российских веб-проектах Shadow DOM особенно важен для соблюдения стандартов визуального оформления в государственных и корпоративных системах.

### HTML Templates

Элемент `<template>` позволяет создавать инертные блоки разметки, которые могут быть клонированы и использованы позже:

```html
<template id="card-template">
  <style>
    .card {
      border: 1px solid #ddd;
      border-radius: 4px;
      padding: 16px;
      margin: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  </style>
  <div class="card">
    <h3><slot name="title"></slot></h3>
    <p><slot name="content"></slot></p>
  </div>
</template>
```

## Современные стандарты Web Components

### ES6 Классы

Современные веб-компоненты используют ES6 классы для определения пользовательских элементов:

```javascript
class NotificationBanner extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.render();
  }
  
  render() {
    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
          position: fixed;
          top: 20px;
          right: 20px;
          z-index: 1000;
        }
        
        .banner {
          padding: 12px 16px;
          border-radius: 4px;
          color: white;
          background-color: var(--notification-bg, #007bff);
        }
      </style>
      
      <div class="banner">
        <slot name="message"></slot>
      </div>
    `;
  }
  
  connectedCallback() {
    setTimeout(() => this.remove(), 3000); // Автоматическое скрытие через 3 сек
  }
}

customElements.define('notification-banner', NotificationBanner);
```

### Lifecycle Callbacks

- `connectedCallback` - вызывается при добавлении элемента в DOM
- `disconnectedCallback` - вызывается при удалении из DOM
- `adoptedCallback` - вызывается при перемещении в новый документ
- `attributeChangedCallback` - вызывается при изменении наблюдаемых атрибутов

## Российские стандарты и практики

### Государственные требования

В 2025 году российские веб-компоненты должны соответствовать:

- Стандартам доступности (Руководство по обеспечению доступности веб-контента, WCAG 2.1)
- Требованиям по защите информации
- Национальным стандартам визуального оформления

### Отечественные фреймворки

Российские компании разрабатывают собственные системы веб-компонентов:

- **РосКомпонент** - библиотека компонентов для государственных проектов
- **СберКомпоненты** - компоненты для внутренних систем Сбербанка
- **Яндекс.Компоненты** - адаптированная версия под российские стандарты

> [!important]
> При разработке веб-компонентов для российского рынка необходимо учитывать требования об обработке персональных данных и требования к хранению данных на территории РФ.

### Локализация и интернационализация

```javascript
class LocalizedButton extends HTMLElement {
  constructor() {
    super();
    this.locale = document.documentElement.lang || 'ru';
    this.attachShadow({ mode: 'open' });
    this.render();
  }
  
  render() {
    const translations = {
      ru: { submit: 'Отправить', cancel: 'Отмена' },
      en: { submit: 'Submit', cancel: 'Cancel' }
    };
    
    const text = translations[this.locale]?.[this.getAttribute('type')] || 'Button';
    
    this.shadowRoot.innerHTML = `
      <style>
        button {
          padding: 8px 16px;
          border: none;
          border-radius: 4px;
          background-color: #007bff;
          color: white;
          cursor: pointer;
        }
      </style>
      
      <button type="${this.getAttribute('type')}">${text}</button>
    `;
  }
}
```

## Практические рекомендации

### Лучшие практики

1. **Используйте префиксы для пользовательских элементов**
   - Всегда используйте дефис в названиях пользовательских элементов
   - Рекомендуется использовать префиксы, например: `rus-`, `gov-`, `sber-`

2. **Обеспечьте доступность**
   - Добавляйте ARIA-атрибуты
   - Поддерживайте клавиатурную навигацию
   - Обеспечьте контрастность цветов

3. **Документируйте компоненты**
   - Создавайте JSDoc-документацию
   - Описывайте атрибуты и события
   - Приводите примеры использования

### Тестирование веб-компонентов

```javascript
// Пример тестирования веб-компонента
describe('user-profile', () => {
  let element;
  
  beforeEach(() => {
    element = document.createElement('user-profile');
    document.body.appendChild(element);
  });
  
  afterEach(() => {
    document.body.removeChild(element);
  });
  
  it('должен отображать имя пользователя', () => {
    element.setAttribute('name', 'Иван Петров');
    const nameElement = element.shadowRoot.querySelector('.name');
    expect(nameElement.textContent).toBe('Иван Петров');
  });
});
```

## Интеграция с фреймворками

### React
```javascript
// Использование веб-компонента в React
import { createComponent } from '@lit-labs/react';

const UserProfile = createComponent(
  React,
  'user-profile',
  window.UserProfile
);

function App() {
  return <UserProfile name="Иван Петров" avatar="/images/ivan.jpg" />;
}
```

### Vue
```javascript
// Использование веб-компонента в Vue
export default {
  name: 'App',
  template: `
    <user-profile :name="userName" :avatar="userAvatar"></user-profile>
  `,
  data() {
    return {
      userName: 'Иван Петров',
      userAvatar: '/images/ivan.jpg'
    };
  }
};
```

## Связанные темы

- [[HTML6-и-новые-функции]]
- [[Интеграция-с-веб-API]]
- [[Совместимость-и-эволюция]]
- [[Российские-перспективы]]

## Заключение

Web Components становятся основой будущего HTML, обеспечивая независимость от фреймворков и возможность создания переиспользуемых компонентов. В российских условиях особенно важно учитывать требования безопасности, доступности и национальные стандарты при разработке веб-компонентов.