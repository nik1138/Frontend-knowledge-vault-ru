---
aliases: [CSS Scoping Specification, CSS Scoping Module, CSS Isolation, CSS Encapsulation]
tags: [css, scoping, encapsulation, specification, css-modules]
---

# CSS Scoping Module: Области видимости и изоляция CSS - Полное руководство

## Введение

CSS Scoping Module определяет способы изоляции CSS стилей, чтобы предотвратить конфликты между различными частями приложения. Модуль предоставляет инструменты для создания изолированных областей видимости стилей, что особенно важно при создании компонентных архитектур и библиотек компонентов.

## CSS Scoping Module Level 1

Level 1 определяет основные концепции изоляции CSS.

### Пространства имен CSS

```css
/* Определение пространства имен */
@namespace "my-app";
@namespace svg url(http://www.w3.org/2000/svg);

/* Использование пространства имен */
my-app|button {
  /* Стили только для элементов button в пространстве имен my-app */
  background-color: blue;
}

svg|circle {
  /* Стили для SVG кругов */
  fill: red;
}
```

### Контекст стилей

```css
/* Определение изолированного контекста */
@scope (.component) {
  /* Стили применяются только внутри элементов с классом .component */
  
  :scope {
    /* Стили для самого элемента .component */
    padding: 20px;
    border: 1px solid #ccc;
  }
  
  .header {
    /* Стили для .header внутри .component */
    font-size: 1.5em;
    color: #333;
  }
  
  .button {
    /* Стили для .button внутри .component */
    background-color: #007bff;
    color: white;
    border: none;
    padding: 8px 16px;
  }
}
```

### Контекст с ограничениями

```css
/* Контекст с явным селектором ограничения */
@scope (.card) to (.dashboard) {
  /* Стили .card применяются только внутри .dashboard */
  
  :scope {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  }
  
  .title {
    font-weight: bold;
    margin-bottom: 10px;
  }
}
```

## CSS Scoping Module Level 2

Level 2 добавляет расширенные возможности изоляции стилей.

### Улучшенные возможности изоляции

```css
/* Расширенный синтаксис @scope */
@scope (:not(.global-styles)) {
  /* Стили, которые не применяются к .global-styles и его потомкам */
  
  .component {
    /* Эти стили будут изолированы от .global-styles */
    padding: 15px;
  }
}

/* Вложенные области видимости */
@scope (.outer-container) {
  .nested-scope {
    @scope (.inner-component) {
      /* Стили для .inner-component внутри .outer-container .nested-scope */
      background-color: #f8f9fa;
    }
  }
}
```

### Модули CSS

```css
/* Пример модульной структуры */
@scope (.button-module) {
  :scope {
    display: inline-block;
    position: relative;
  }
  
  .btn {
    /* Базовые стили кнопки */
    padding: 10px 20px;
    border: 1px solid transparent;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  
  .btn--primary {
    /* Стили первичной кнопки */
    background-color: #007bff;
    color: white;
  }
  
  .btn--secondary {
    /* Стили вторичной кнопки */
    background-color: #6c757d;
    color: white;
  }
  
  .btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
}
```

## Практические применения

### Компонентная архитектура

```css
/* Карточка пользователя */
@scope (.user-card) {
  :scope {
    display: flex;
    flex-direction: column;
    width: 300px;
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
  
  .avatar {
    width: 100%;
    height: 200px;
    object-fit: cover;
  }
  
  .info {
    padding: 20px;
  }
  
  .name {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 5px;
  }
  
  .email {
    color: #666;
    font-size: 0.9rem;
  }
  
  .actions {
    padding: 0 20px 20px;
    display: flex;
    gap: 10px;
  }
}

/* Кнопка в карточке пользователя */
@scope (.user-card) {
  .btn {
    flex: 1;
    padding: 10px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    text-align: center;
    text-decoration: none;
    transition: background-color 0.2s;
  }
  
  .btn--primary {
    background-color: #007bff;
    color: white;
  }
  
  .btn--secondary {
    background-color: #6c757d;
    color: white;
  }
}
```

### Темизация компонентов

```css
/* Базовая тема компонентов */
@scope (.ui-component) {
  :scope {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #17a2b8;
    
    --border-radius: 4px;
    --box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    --transition: all 0.2s ease;
  }
  
  .card {
    background: white;
    border-radius: var(--border-radius);
    box-shadow: var(--box-shadow);
    padding: 1rem;
  }
  
  .button {
    padding: 0.5rem 1rem;
    border-radius: var(--border-radius);
    border: none;
    cursor: pointer;
    transition: var(--transition);
  }
  
  .button--primary {
    background-color: var(--primary-color);
    color: white;
  }
  
  .button--secondary {
    background-color: var(--secondary-color);
    color: white;
  }
}

/* Темная тема */
@scope (.ui-component[data-theme="dark"]) {
  :scope {
    --primary-color: #0d6efd;
    --secondary-color: #6c757d;
    --background-color: #212529;
    --text-color: #f8f9fa;
  }
  
  .card {
    background: var(--background-color);
    color: var(--text-color);
  }
}
```

### Библиотека компонентов

```css
/* Изоляция стилей библиотеки */
@scope (.my-component-library) {
  /* Все стили библиотеки изолированы */
  
  .alert {
    padding: 1rem;
    border-radius: 0.375rem;
    margin-bottom: 1rem;
  }
  
  .alert--success {
    background-color: #d4edda;
    color: #155724;
    border-left: 4px solid #28a745;
  }
  
  .alert--error {
    background-color: #f8d7da;
    color: #721c24;
    border-left: 4px solid #dc3545;
  }
  
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0,0,0,0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }
  
  .modal__content {
    background: white;
    border-radius: 8px;
    max-width: 90%;
    max-height: 90%;
    overflow: auto;
  }
  
  .modal__header {
    padding: 1rem;
    border-bottom: 1px solid #eee;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .modal__body {
    padding: 1rem;
  }
  
  .modal__footer {
    padding: 1rem;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: flex-end;
    gap: 0.5rem;
  }
}
```

### Вложенные компоненты

```css
/* Контейнер приложения */
@scope (.app-container) {
  :scope {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    max-width: 1200px;
    margin: 0 auto;
    padding: 1rem;
  }
  
  /* Боковая панель */
  @scope (.sidebar) {
    :scope {
      width: 250px;
      background: #f8f9fa;
      border-right: 1px solid #dee2e6;
      padding: 1rem;
      height: calc(100vh - 2rem);
      position: fixed;
    }
    
    .nav-item {
      display: block;
      padding: 0.5rem;
      text-decoration: none;
      color: #495057;
      border-radius: 4px;
      margin-bottom: 0.25rem;
      transition: background-color 0.2s;
    }
    
    .nav-item:hover {
      background-color: #e9ecef;
    }
  }
  
  /* Основной контент */
  @scope (.main-content) {
    :scope {
      margin-left: 270px; /* Учет ширины боковой панели */
      padding: 1rem 0;
    }
    
    .content-section {
      margin-bottom: 2rem;
    }
  }
}
```

## Современные возможности

### CSS Custom Properties в изоляции

```css
@scope (.themed-component) {
  :scope {
    /* Переменные, доступные только внутри области видимости */
    --primary-hue: 210;
    --primary-saturation: 100%;
    --primary-lightness: 50%;
    --primary-color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
  }
  
  .element {
    background-color: var(--primary-color);
    color: white;
  }
  
  .element--light {
    --primary-lightness: 80%; /* Изменение переменной для конкретного класса */
    background-color: var(--primary-color);
    color: black;
  }
}
```

### Адаптивные области видимости

```css
@scope (.responsive-component) {
  :scope {
    display: flex;
    flex-direction: column;
  }
  
  .item {
    padding: 1rem;
    border-bottom: 1px solid #eee;
  }
  
  @media (min-width: 768px) {
    :scope {
      flex-direction: row;
    }
    
    .item {
      flex: 1;
      border-bottom: none;
      border-right: 1px solid #eee;
    }
    
    .item:last-child {
      border-right: none;
    }
  }
}
```

## Интеграция с другими технологиями

### Связь с CSS Custom Properties

```css
@scope (.dynamic-component) {
  :scope {
    /* Использование кастомных свойств для динамического изменения */
    --component-bg: #ffffff;
    --component-text: #333333;
    --component-border: #dddddd;
  }
  
  .element {
    background-color: var(--component-bg);
    color: var(--component-text);
    border: 1px solid var(--component-border);
    padding: 1rem;
    border-radius: 4px;
  }
}

/* JavaScript может изменять переменные внутри изолированной области */
/* document.querySelector('.dynamic-component').style.setProperty('--component-bg', '#f0f0f0'); */
```

### Работа с Shadow DOM

```css
/* Стили для Shadow DOM компонентов */
@scope (.shadow-component) {
  :scope {
    /* Стили применяются только внутри shadow root */
    display: block;
  }
  
  .internal-element {
    /* Эти стили не повлияют на внешние элементы */
    padding: 10px;
    background: #f8f9fa;
  }
}
```

## Поддержка браузерами

- `@scope` правило: Поддержка в процессе внедрения в современные браузеры
- Пространства имен CSS: Поддержка в большинстве современных браузеров
- Требует полифиллов или преобразования для широкой поддержки

## Связанные темы

- [[CSS Custom Properties Module]] - кастомные свойства
- [[CSS Cascade and Inheritance]] - каскад и наследование
- [[CSS Architecture]] - архитектура CSS

## Заключение

CSS Scoping Module предоставляет мощные инструменты для изоляции стилей и создания модульных компонентов. Понимание областей видимости позволяет создавать более предсказуемые и поддерживаемые стилевые решения.