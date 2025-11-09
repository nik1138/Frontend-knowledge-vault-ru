# Продвинутые возможности CSS: современные фичи и техники

## Введение

Современный CSS предоставляет мощные возможности для создания сложных, интерактивных и адаптивных пользовательских интерфейсов. В этой статье мы рассмотрим продвинутые возможности CSS, включая современные фичи, которые значительно улучшают разработку веб-приложений.

## Кастомные CSS свойства (CSS Variables)

### Основы CSS переменных

CSS переменные (кастомные свойства) позволяют хранить значения, которые можно повторно использовать в разных частях CSS-кода.

```css
/* Определение кастомных свойств */
:root {
    /* Цветовая палитра */
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --accent-color: #e74c3c;
    --text-color: #2c3e50;
    --background-color: #ecf0f1;
    
    /* Размеры */
    --border-radius: 8px;
    --spacing-unit: 8px;
    --font-size-base: 16px;
    
    /* Тени */
    --shadow-small: 0 2px 4px rgba(0, 0, 0, 0.1);
    --shadow-medium: 0 4px 8px rgba(0, 0, 0, 0.15);
    --shadow-large: 0 8px 16px rgba(0, 0, 0, 0.2);
}

/* Использование кастомных свойств */
.button {
    background-color: var(--primary-color);
    color: white;
    border-radius: var(--border-radius);
    padding: calc(var(--spacing-unit) * 1.5) calc(var(--spacing-unit) * 2);
    font-size: var(--font-size-base);
    box-shadow: var(--shadow-medium);
}

.card {
    background-color: white;
    border-radius: var(--border-radius);
    box-shadow: var(--shadow-small);
    padding: calc(var(--spacing-unit) * 2);
}
```

### Динамические CSS переменные

```css
/* Переменные, изменяющиеся по состоянию */
.button {
    --bg-color: var(--primary-color);
    --text-color: white;
    background-color: var(--bg-color);
    color: var(--text-color);
    transition: all 0.3s ease;
}

.button:hover {
    --bg-color: #2980b9;
    transform: translateY(-2px);
}

.button:active {
    --bg-color: #21618c;
    transform: translateY(0);
}

/* Темизация с использованием CSS переменных */
.theme-dark {
    --primary-color: #3498db;
    --text-color: #ffffff;
    --background-color: #2c3e50;
    --card-bg: #34495e;
}

.theme-light {
    --primary-color: #3498db;
    --text-color: #2c3e50;
    --background-color: #ffffff;
    --card-bg: #f8f9fa;
}

/* Переменные с fallback значениями */
.element {
    /* Синтаксис: var(--custom-property, fallback-value) */
    color: var(--text-color, #333);
    background-color: var(--card-bg, #fff);
    border-color: var(--border-color, var(--primary-color));
}
```

### Продвинутое использование CSS переменных

```css
/* Переменные для расчетов */
:root {
    --columns: 12;
    --gutter: 20px;
    --column-width: calc((100% - ((var(--columns) - 1) * var(--gutter))) / var(--columns));
}

.grid-item {
    width: var(--column-width);
    margin-right: var(--gutter);
}

.grid-item:last-child {
    margin-right: 0;
}

/* Переменные для анимаций */
.animated-element {
    --animation-duration: 0.5s;
    --animation-delay: 0s;
    --animation-easing: ease-in-out;
    
    animation: slideIn var(--animation-duration) var(--animation-easing) var(--animation-delay);
}

/* Адаптивные переменные */
@media (max-width: 768px) {
    :root {
        --spacing-unit: 6px;
        --font-size-base: 14px;
        --border-radius: 4px;
    }
}

@media (min-width: 1200px) {
    :root {
        --spacing-unit: 12px;
        --font-size-base: 18px;
        --border-radius: 12px;
    }
}
```

## Современные возможности CSS

### Container Queries (экспериментально, с полифилом)

Container Queries позволяют применять стили на основе размера или стиля родительского контейнера, а не всего окна браузера.

```css
.card-container {
    container-type: inline-size;
    container-name: card;
}

@container card (min-width: 300px) {
    .card-content {
        display: grid;
        grid-template-columns: 1fr 2fr;
        gap: 1rem;
    }
}

@container card (max-width: 299px) {
    .card-content {
        display: block;
    }
}

/* Стилизация на основе стиля контейнера */
@container card (width > 400px) {
    .card-title {
        font-size: 1.5rem;
    }
}
```

### Logical Properties

Logical Properties позволяют писать CSS, который адаптируется к направлению текста (LTR/RTL) и другим параметрам локализации.

```css
/* Вместо margin-left/right */
.text-container {
    margin-inline-start: 1rem;    /* Вместо margin-left */
    margin-inline-end: 1rem;      /* Вместо margin-right */
    margin-block-start: 0.5rem;   /* Вместо margin-top */
    margin-block-end: 0.5rem;     /* Вместо margin-bottom */
    
    /* Padding */
    padding-inline: 1rem;         /* Вместо padding-left и padding-right */
    padding-block: 0.5rem;        /* Вместо padding-top и padding-bottom */
    
    /* Border */
    border-inline-start: 2px solid var(--primary-color);  /* Вместо border-left */
    
    /* Text alignment */
    text-align: start;            /* Вместо left/right */
}

/* Размеры */
.layout-box {
    /* Вместо width/height */
    inline-size: 100%;            /* Вместо width */
    block-size: 200px;            /* Вместо height */
    
    /* Min/max размеры */
    min-inline-size: 300px;       /* Вместо min-width */
    max-block-size: 400px;        /* Вместо max-height */
}
```

### Scroll-Driven Animations (экспериментально)

Scroll-driven animations позволяют анимировать элементы в зависимости от прокрутки.

```css
/* Анимации, управляемые прокруткой */
.scroll-animation {
    animation: slideIn linear;
    animation-timeline: view();
    animation-range: entry 0% entry 100%;
}

@keyframes slideIn {
    from { transform: translateX(-100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
}

/* Анимации с разными диапазонами */
.scroll-fade {
    animation: fade linear;
    animation-timeline: view();
    animation-range: contain 0% contain 50%;
}

@keyframes fade {
    from { opacity: 0; }
    to { opacity: 1; }
}

/* Параллакс-эффект с анимациями */
.parallax-container {
    height: 200vh;
    overflow-y: auto;
}

.parallax-element {
    position: sticky;
    top: 0;
    height: 100vh;
    animation: parallax linear;
    animation-timeline: scroll();
    animation-range: entry 0% entry 100%;
}

@keyframes parallax {
    from { transform: translateY(0) scale(1); }
    to { transform: translateY(-50%) scale(1.2); }
}
```

### CSS Nesting (экспериментально)

CSS Nesting позволяет вкладывать CSS-правила, что делает код более читаемым и организованным.

```css
/* Традиционный подход */
.card { background: white; }
.card__header { padding: 1rem; }
.card__title { font-size: 1.2rem; }
.card__content { padding: 1rem; }

/* С использованием nesting (гипотетический пример) */
.card {
    background: white;
    
    &__header {
        padding: 1rem;
        
        &__title {
            font-size: 1.2rem;
        }
    }
    
    &__content {
        padding: 1rem;
    }
}
```

### CSS Subgrid (экспериментально)

Subgrid позволяет дочерним элементам участвовать в родительской сетке.

```css
.parent-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: auto;
    gap: 1rem;
}

.child-grid {
    display: grid;
    grid-column: span 2; /* Занимает 2 колонки родительской сетки */
    grid-template-columns: subgrid; /* Использует колонки родителя */
    grid-template-rows: auto;
}
```

## Продвинутые техники стилизации

### Градиенты

```css
/* Линейные градиенты */
.gradient-linear {
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    /* Угол, цвета */
}

.gradient-multistop {
    background: linear-gradient(
        to right,
        #ff9a9e 0%,
        #fecfef 50%,
        #fecfef 50%,
        #fecfef 100%
    );
}

/* Радиальные градиенты */
.gradient-radial {
    background: radial-gradient(circle, #667eea 0%, #764ba2 100%);
}

/* Конические градиенты */
.gradient-conic {
    background: conic-gradient(
        from 0deg at 50% 50%,
        #ff0000,
        #00ff00,
        #0000ff,
        #ff0000
    );
}

/* Градиенты с анимацией */
.animated-gradient {
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4, #ffeaa7);
    background-size: 400% 400%;
    animation: gradientShift 4s ease infinite;
}

@keyframes gradientShift {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}
```

### Фильтры и blend modes

```css
/* CSS фильтры */
.image-filters {
    transition: filter 0.3s ease;
}

.image-filters:hover {
    filter: 
        brightness(1.1)
        contrast(1.1)
        saturate(1.2)
        blur(0.5px)
        hue-rotate(5deg);
}

/* Blend modes */
.blend-overlay {
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    mix-blend-mode: overlay;
}

.blend-multiply {
    background: url('image.jpg');
    background-blend-mode: multiply;
    background-color: #ff6b6b;
}

/* Комбинация фильтров */
.filter-combo {
    filter: 
        drop-shadow(0 4px 8px rgba(0,0,0,0.2))
        contrast(1.2)
        saturate(1.3);
}
```

### CSS Shapes

```css
/* Обтекание элементов */
.shape-wrapper {
    width: 300px;
    height: 300px;
    float: left;
    shape-outside: circle(50% at center);
    shape-margin: 20px;
}

/* Сложные формы */
.complex-shape {
    shape-outside: polygon(0 0, 100% 0, 100% 75%, 75% 75%, 75% 100%, 50% 75%, 0 75%);
    float: left;
}
```

## Адаптивные и отзывчивые техники

### Fluid Typography

```css
/* Адаптивный размер шрифта */
.fluid-text {
    font-size: clamp(1rem, 2.5vw, 2rem);
    /* min, preferred, max */
}

/* Более сложный пример */
.responsive-heading {
    font-size: calc(1.2rem + 1.5vw);
}

@media (min-width: 1200px) {
    .responsive-heading {
        font-size: 2.5rem;
    }
}
```

### Responsive Spacing

```css
/* Адаптивные отступы */
.responsive-container {
    padding: clamp(1rem, 4vw, 3rem);
}

/* Адаптивные размеры элементов */
.responsive-card {
    width: min(90%, 600px);
    margin: 0 auto;
    padding: clamp(1rem, 3vw, 2rem);
}
```

## Продвинутые селекторы

### Псевдоклассы и псевдоэлементы

```css
/* Современные псевдоклассы */
.element:has(> .child) {
    /* Стили для элемента, содержащего .child */
    border: 2px solid blue;
}

.element:not(.disabled):not(.hidden) {
    /* Стили для элемента, не имеющего классов disabled и hidden */
    opacity: 1;
}

/* Псевдоэлементы */
.element::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, rgba(255,255,255,0.1), transparent);
}

.element::marker {
    color: var(--primary-color);
}

/* Стилизация списка */
.list::marker {
    color: var(--accent-color);
}
```

### Сложные селекторы

```css
/* Селекторы соседних элементов */
.element + .sibling {
    /* Соседний элемент сразу после .element */
    margin-top: 1rem;
}

.element ~ .siblings {
    /* Все элементы .siblings после .element */
    opacity: 0.7;
}

/* Селекторы потомков */
.parent .child .grandchild {
    color: var(--text-color);
}

/* Селекторы с атрибутами */
input[type="text"][required] {
    border: 2px solid var(--primary-color);
}

input[name^="user"] {
    /* Элементы, имя которых начинается с "user" */
    background-color: #f0f8ff;
}

input[name$="email"] {
    /* Элементы, имя которых заканчивается на "email" */
    background-color: #fff0f0;
}

input[name*="name"] {
    /* Элементы, имя которых содержит "name" */
    background-color: #f0fff0;
}
```

## Лучшие практики и паттерны

### Система компонентов

```css
/* Базовые стили компонентов */
:root {
    /* Типографика */
    --font-family-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    --font-family-mono: 'Fira Code', Consolas, Monaco, monospace;
    
    /* Размеры шрифтов */
    --font-size-xs: 0.75rem;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
    --font-size-2xl: 1.5rem;
    --font-size-3xl: 1.875rem;
    
    /* Вес шрифта */
    --font-weight-normal: 400;
    --font-weight-medium: 500;
    --font-weight-semibold: 600;
    --font-weight-bold: 700;
    
    /* Линии */
    --line-height-tight: 1.25;
    --line-height-snug: 1.375;
    --line-height-normal: 1.5;
    --line-height-relaxed: 1.625;
    
    /* Отступы */
    --spacing-1: calc(var(--spacing-unit) * 0.25);
    --spacing-2: calc(var(--spacing-unit) * 0.5);
    --spacing-3: calc(var(--spacing-unit) * 0.75);
    --spacing-4: calc(var(--spacing-unit) * 1);
    --spacing-5: calc(var(--spacing-unit) * 1.25);
    --spacing-6: calc(var(--spacing-unit) * 1.5);
    --spacing-8: calc(var(--spacing-unit) * 2);
    --spacing-10: calc(var(--spacing-unit) * 2.5);
    --spacing-12: calc(var(--spacing-unit) * 3);
}

/* Базовые классы */
.text-primary { color: var(--text-color); }
.text-secondary { color: #7f8c8d; }
.text-accent { color: var(--primary-color); }

.bg-primary { background-color: var(--background-color); }
.bg-secondary { background-color: #bdc3c7; }
.bg-accent { background: linear-gradient(45deg, var(--primary-color), var(--secondary-color)); }

.border-primary { border: 1px solid #bdc3c7; }
.border-accent { border: 2px solid var(--primary-color); }

.rounded { border-radius: var(--border-radius); }
.rounded-lg { border-radius: calc(var(--border-radius) * 1.5); }
.rounded-full { border-radius: 9999px; }

.shadow { box-shadow: var(--shadow-small); }
.shadow-md { box-shadow: var(--shadow-medium); }
.shadow-lg { box-shadow: var(--shadow-large); }
```

### Темизация

```css
/* Светлая тема (по умолчанию) */
:root {
    --color-bg: #ffffff;
    --color-text: #2c3e50;
    --color-text-secondary: #7f8c8d;
    --color-border: #ecf0f1;
    --color-surface: #f8f9fa;
    --color-overlay: rgba(0, 0, 0, 0.5);
}

/* Темная тема */
[data-theme="dark"],
.theme-dark {
    --color-bg: #1a1a1a;
    --color-text: #f8f9fa;
    --color-text-secondary: #adb5bd;
    --color-border: #343a40;
    --color-surface: #2d3748;
    --color-overlay: rgba(0, 0, 0, 0.8);
}

/* Автоматическое определение темы */
@media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) {
        --color-bg: #1a1a1a;
        --color-text: #f8f9fa;
        --color-text-secondary: #adb5bd;
        --color-border: #343a40;
        --color-surface: #2d3748;
    }
}

/* Компонент с поддержкой темы */
.themed-card {
    background-color: var(--color-surface);
    color: var(--color-text);
    border: 1px solid var(--color-border);
    border-radius: var(--border-radius);
    padding: var(--spacing-4);
    transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease;
}
```

## Современные инструменты и техники

### CSS Custom Highlight API (экспериментально)

```css
/* Пользовательские выделения */
::highlight(custom-search) {
    background-color: yellow;
    color: black;
}

::highlight(custom-mark) {
    background-color: var(--accent-color);
    color: white;
    border-radius: 2px;
}
```

### CSS Anchor Positioning (экспериментально)

```css
/* Позиционирование элементов относительно якоря */
.tooltip {
    position: absolute;
    anchor-name: --tooltip-anchor;
    top: anchor(bottom);
    left: anchor(left);
    background: black;
    color: white;
    padding: 0.5rem;
    border-radius: 4px;
}

.tooltip-anchor {
    anchor-name: --tooltip-anchor;
}
```

## Примеры из промышленной разработки

### Адаптивная сетка с CSS Grid

```css
/* Гибкая адаптивная сетка */
.grid-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: var(--spacing-4);
    padding: var(--spacing-4);
}

/* Карточки в сетке */
.grid-card {
    background: var(--color-surface);
    border: 1px solid var(--color-border);
    border-radius: var(--border-radius);
    overflow: hidden;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.grid-card:hover {
    transform: translateY(-4px);
    box-shadow: var(--shadow-medium);
}

.grid-card-image {
    width: 100%;
    height: 200px;
    object-fit: cover;
    display: block;
}

.grid-card-content {
    padding: var(--spacing-4);
}

.grid-card-title {
    font-size: var(--font-size-lg);
    font-weight: var(--font-weight-semibold);
    margin: 0 0 var(--spacing-2) 0;
    color: var(--color-text);
}

.grid-card-description {
    color: var(--color-text-secondary);
    line-height: var(--line-height-relaxed);
    margin: 0;
}

/* Адаптация для мобильных устройств */
@media (max-width: 768px) {
    .grid-container {
        grid-template-columns: 1fr;
        padding: var(--spacing-2);
        gap: var(--spacing-3);
    }
    
    .grid-card {
        border-radius: calc(var(--border-radius) * 1.5);
    }
}

/* Адаптация для планшетов */
@media (min-width: 769px) and (max-width: 1024px) {
    .grid-container {
        grid-template-columns: repeat(2, 1fr);
    }
}
```

### Система уведомлений с современными возможностями

```css
/* Стили для системы уведомлений */
.notifications-container {
    position: fixed;
    top: var(--spacing-4);
    right: var(--spacing-4);
    z-index: 9999;
    display: flex;
    flex-direction: column;
    gap: var(--spacing-2);
    max-width: 400px;
}

.notification {
    --notification-bg: #3498db;
    --notification-text: white;
    
    background: var(--notification-bg);
    color: var(--notification-text);
    padding: var(--spacing-4);
    border-radius: var(--border-radius);
    box-shadow: var(--shadow-large);
    display: flex;
    align-items: center;
    gap: var(--spacing-3);
    transform: translateX(100%);
    opacity: 0;
    transition: transform 0.3s ease, opacity 0.3s ease;
    animation: slideIn 0.3s ease forwards;
}

.notification.show {
    transform: translateX(0);
    opacity: 1;
}

.notification::before {
    content: '';
    width: 4px;
    height: 100%;
    background: var(--notification-bg);
    border-radius: 2px 0 0 2px;
    position: absolute;
    left: 0;
    top: 0;
}

.notification-icon {
    width: 24px;
    height: 24px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.notification-content {
    flex: 1;
    font-size: var(--font-size-sm);
    line-height: var(--line-height-tight);
}

.notification-close {
    background: none;
    border: none;
    color: var(--notification-text);
    cursor: pointer;
    padding: var(--spacing-1);
    border-radius: var(--border-radius);
    opacity: 0.7;
    transition: opacity 0.2s ease;
}

.notification-close:hover {
    opacity: 1;
}

/* Типы уведомлений */
.notification.success { --notification-bg: #2ecc71; }
.notification.error { --notification-bg: #e74c3c; }
.notification.warning { --notification-bg: #f39c12; color: #2c3e50; }
.notification.info { --notification-bg: #3498db; }

@keyframes slideIn {
    from {
        transform: translateX(100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}
```

## Ссылки на связанные темы
- [[methodology]] - Методологии CSS
- [[animations]] - Анимации CSS
- [[responsive]] - Адаптивный дизайн
- [[html/semantics]] - Семантическая верстка

#programming #css #advanced #variables #custom-properties #grid #flexbox #modern-css