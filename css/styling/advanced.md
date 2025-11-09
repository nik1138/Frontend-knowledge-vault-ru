# Продвинутые техники стилизации в CSS

## Введение

Современная CSS-стилизация выходит за рамки базовых свойств и включает продвинутые техники, кастомные свойства, визуальные эффекты и адаптивные подходы. В этой статье мы рассмотрим передовые методы стилизации, используемые в промышленной разработке.

## Кастомные CSS свойства (CSS Variables)

### Основы CSS переменных

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

## Продвинутые визуальные эффекты

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

/* Градиенты как маска */
.gradient-mask {
    background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
    -webkit-mask: linear-gradient(90deg, black 60%, transparent 100%);
    mask: linear-gradient(90deg, black 60%, transparent 100%);
}
```

### Тени и глубина

```css
/* Многослойные тени */
.card-multi-shadow {
    box-shadow: 
        0 1px 3px rgba(0, 0, 0, 0.12),
        0 1px 2px rgba(0, 0, 0, 0.24),
        0 4px 8px rgba(0, 0, 0, 0.08),
        0 8px 16px rgba(0, 0, 0, 0.04);
}

/* Внутренние тени */
.card-inset {
    box-shadow: 
        inset 0 2px 4px rgba(0, 0, 0, 0.06),
        0 1px 3px rgba(0, 0, 0, 0.12);
}

/* Текстовые тени */
.text-shadow {
    text-shadow: 
        2px 2px 4px rgba(0, 0, 0, 0.3),
        -1px -1px 2px rgba(255, 255, 255, 0.1);
}

/* Объемные эффекты */
.3d-effect {
    position: relative;
}

.3d-effect::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: linear-gradient(135deg, rgba(255,255,255,0.1) 0%, transparent 50%);
    border-radius: inherit;
    pointer-events: none;
}

/* Неоновые эффекты */
.neon-effect {
    color: #fff;
    text-shadow: 
        0 0 5px #fff,
        0 0 10px #fff,
        0 0 15px #fff,
        0 0 20px #0ff,
        0 0 35px #0ff,
        0 0 40px #0ff;
    box-shadow: 
        0 0 5px #fff,
        inset 0 0 5px #fff,
        0 0 20px #0ff,
        inset 0 0 20px #0ff,
        0 0 30px #0ff,
        inset 0 0 30px #0ff;
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

## Современные подходы к стилизации

### Container Queries (экспериментально, с полифилом)

```css
/* Стилизация на основе размера контейнера */
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

```css
/* Логические свойства для международной поддержки */
.text-container {
    /* Вместо margin-left/right */
    margin-inline-start: 1rem;
    margin-inline-end: 1rem;
    
    /* Вместо margin-top/bottom */
    margin-block-start: 0.5rem;
    margin-block-end: 0.5rem;
    
    /* Padding */
    padding-inline: 1rem;
    padding-block: 0.5rem;
    
    /* Border */
    border-inline-start: 2px solid var(--primary-color);
    
    /* Text alignment */
    text-align: start; /* вместо left/right */
}

/* Размеры */
.layout-box {
    /* Вместо width/height */
    inline-size: 100%;
    block-size: 200px;
    
    /* Min/max размеры */
    min-inline-size: 300px;
    max-block-size: 400px;
}
```

### Custom Highlight API (экспериментально)

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

## Продвинутые техники стилизации компонентов

### Карточки с интерактивными эффектами

```css
.interactive-card {
    position: relative;
    background: white;
    border-radius: var(--border-radius);
    overflow: hidden;
    transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    box-shadow: var(--shadow-small);
}

.interactive-card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 4px;
    background: linear-gradient(90deg, var(--primary-color), var(--secondary-color));
    transform: scaleX(0);
    transform-origin: left;
    transition: transform 0.3s ease;
}

.interactive-card:hover {
    transform: translateY(-8px) scale(1.02);
    box-shadow: var(--shadow-large);
}

.interactive-card:hover::before {
    transform: scaleX(1);
}

.interactive-card-content {
    padding: calc(var(--spacing-unit) * 2);
    position: relative;
    z-index: 1;
}

/* Карточки с эффектом параллакса */
.parallax-card {
    position: relative;
    overflow: hidden;
}

.parallax-card::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: radial-gradient(circle, transparent 20%, rgba(255,255,255,0.1) 80%);
    opacity: 0;
    transition: opacity 0.3s ease;
    pointer-events: none;
}

.parallax-card:hover::before {
    opacity: 1;
    animation: rotate 10s linear infinite;
}

@keyframes rotate {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}
```

### Кнопки с продвинутыми эффектами

```css
/* Кнопки с эффектом волны */
.wave-button {
    position: relative;
    overflow: hidden;
    border: none;
    background: var(--primary-color);
    color: white;
    padding: 12px 24px;
    border-radius: var(--border-radius);
    cursor: pointer;
    transition: all 0.3s ease;
}

.wave-button::before {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.3);
    transform: translate(-50%, -50%);
    transition: width 0.6s, height 0.6s;
}

.wave-button:active::before {
    width: 300px;
    height: 300px;
}

/* Кнопки с градиентной анимацией */
.gradient-button {
    position: relative;
    background: var(--primary-color);
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: var(--border-radius);
    overflow: hidden;
    transition: all 0.3s ease;
}

.gradient-button::before {
    content: '';
    position: absolute;
    top: -100%;
    left: -100%;
    width: 200%;
    height: 200%;
    background: linear-gradient(
        45deg,
        transparent,
        rgba(255, 255, 255, 0.3),
        transparent
    );
    transition: all 0.5s ease;
    z-index: 0;
}

.gradient-button:hover::before {
    top: -50%;
    left: -50%;
}

.gradient-button span {
    position: relative;
    z-index: 1;
}
```

### Формы с современными стилями

```css
/* Современные инпуты */
.modern-input {
    position: relative;
    margin-bottom: 1.5rem;
}

.modern-input input {
    width: 100%;
    padding: 12px 16px;
    border: 2px solid #e1e5e9;
    border-radius: var(--border-radius);
    font-size: 1rem;
    transition: all 0.3s ease;
    background: white;
}

.modern-input input:focus {
    outline: none;
    border-color: var(--primary-color);
    box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
}

.modern-input label {
    position: absolute;
    left: 16px;
    top: 50%;
    transform: translateY(-50%);
    color: #95a5a6;
    transition: all 0.3s ease;
    pointer-events: none;
    background: white;
    padding: 0 4px;
}

.modern-input input:focus + label,
.modern-input input:not(:placeholder-shown) + label {
    top: -10px;
    left: 12px;
    font-size: 0.875rem;
    color: var(--primary-color);
}

/* Переключатели */
.toggle-switch {
    position: relative;
    display: inline-block;
    width: 60px;
    height: 34px;
}

.toggle-switch input {
    opacity: 0;
    width: 0;
    height: 0;
}

.toggle-switch-slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    transition: .4s;
    border-radius: 34px;
}

.toggle-switch-slider:before {
    position: absolute;
    content: "";
    height: 26px;
    width: 26px;
    left: 4px;
    bottom: 4px;
    background-color: white;
    transition: .4s;
    border-radius: 50%;
}

input:checked + .toggle-switch-slider {
    background: linear-gradient(45deg, var(--primary-color), var(--secondary-color));
}

input:checked + .toggle-switch-slider:before {
    transform: translateX(26px);
}
```

## Лучшие практики стилизации

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

/* Адаптивные утилиты */
@media (min-width: 768px) {
    .md\:rounded-lg { border-radius: calc(var(--border-radius) * 1.5); }
    .md\:shadow-lg { box-shadow: var(--shadow-large); }
}
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

## Примеры из промышленной разработки

### Система уведомлений

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

### Адаптивная сетка

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

## Ссылки на связанные темы
- [[methodology]] - Методологии CSS
- [[animations]] - Анимации CSS
- [[responsive]] - Адаптивный дизайн
- [[html/semantics]] - Семантическая верстка

#programming #css #styling #best-practices #web-design