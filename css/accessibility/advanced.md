# Доступность CSS: создание инклюзивных стилей

## Введение

Доступность CSS (Cascading Style Sheets) - это практика написания стилей, которые поддерживают и не мешают вспомогательным технологиям, таким как скринридеры, увеличители экрана и клавиатурные навигаторы. Правильное использование CSS помогает создавать более инклюзивные веб-приложения, доступные для всех пользователей.

## Основы доступности CSS

### 1. Визуальная индикация фокуса

```css
/* Обязательная визуальная индикация фокуса */
a:focus,
button:focus,
input:focus,
select:focus,
textarea:focus,
[tabindex]:focus {
    outline: 2px solid #007bff;
    outline-offset: 2px;
}

/* Альтернативная стилизация фокуса */
.focus-indicator:focus {
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
    border-color: #007bff;
}

/* Важно: не удаляйте outline полностью */
/* ПЛОХО */
/*
button:focus {
    outline: none;
}
*/

/* ХОРОШО: замена outline на другую визуальную индикацию */
button:focus {
    outline: none;
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
    border-color: #007bff;
}

/* Улучшенная индикация фокуса для темных тем */
[data-theme="dark"] :focus {
    outline: 2px solid #80bdff;
    outline-offset: 2px;
}

/* Индикация фокуса для различных состояний */
.button {
    padding: 0.5rem 1rem;
    border: 2px solid transparent;
    border-radius: 4px;
    transition: all 0.2s ease;
}

.button:focus {
    border-color: #007bff;
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
    outline: none;
}

.button:focus:not(:focus-visible) {
    border-color: transparent;
    box-shadow: none;
}

.button:focus-visible {
    border-color: #007bff;
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
}
```

### 2. Контрастность цветов

```css
/* Обеспечение достаточного контраста */
:root {
    --text-primary: #212529; /* Темный текст */
    --text-secondary: #6c757d; /* Вторичный текст */
    --text-disabled: #adb5bd; /* Отключенный текст */
    --bg-primary: #ffffff; /* Основной фон */
    --bg-secondary: #f8f9fa; /* Вторичный фон */
    
    /* Цвета для ссылок и кнопок */
    --link-color: #007bff;
    --link-hover: #0056b3;
    --btn-primary: #007bff;
    --btn-primary-hover: #0056b3;
}

/* Проверка контраста: минимум 4.5:1 для нормального текста */
.body-text {
    color: var(--text-primary);
    background-color: var(--bg-primary);
}

.link {
    color: var(--link-color);
    text-decoration: underline;
}

.link:hover {
    color: var(--link-hover);
}

/* Цвета для важной информации (минимум 3:1 для крупного текста) */
.high-contrast-text {
    color: #000000;
    background-color: #ffffff;
}

.low-contrast-warning {
    color: #495057; /* Не рекомендуется - низкий контраст */
    background-color: #f8f9fa;
}

/* Темные темы с хорошим контрастом */
[data-theme="dark"] {
    --text-primary: #e9ecef;
    --text-secondary: #adb5bd;
    --bg-primary: #212529;
    --bg-secondary: #343a40;
    --link-color: #6ea8fe;
    --link-hover: #9ec5ff;
}

/* Контрастные цвета для ошибок и предупреждений */
.error-text {
    color: #dc3545;
    background-color: #f8d7da;
}

.warning-text {
    color: #856404;
    background-color: #fff3cd;
}

.success-text {
    color: #155724;
    background-color: #d4edda;
}
```

### 3. Адаптация к предпочтениям пользователя

```css
/* Учет предпочтений пользователя */
@media (prefers-reduced-motion: reduce) {
    /* Отключение анимаций для пользователей, чувствительных к движению */
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
    
    .smooth-animation {
        animation: none;
        transition: none;
    }
}

/* Учет предпочтений по контрастности */
@media (prefers-contrast: high) {
    /* Увеличенный контраст для пользователей с нарушениями зрения */
    body {
        color: #000000;
        background-color: #ffffff;
    }
    
    .card {
        border: 2px solid #000000;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
    }
    
    .button {
        border: 2px solid #000000;
        background-color: #ffffff;
        color: #000000;
    }
}

/* Учет предпочтений по свету/темноте */
@media (prefers-color-scheme: dark) {
    body {
        background-color: #121212;
        color: #e0e0e0;
    }
    
    .card {
        background-color: #1e1e1e;
        border: 1px solid #333333;
    }
}

/* Учет увеличения текста */
@media (prefers-reduced-transparency: reduce) {
    .modal-overlay {
        background-color: #000000;
        opacity: 0.9;
    }
}

/* Учет предпочтений по жестам */
@media (any-pointer: coarse) {
    /* Для сенсорных устройств */
    .button {
        min-height: 44px; /* Минимальный размер для сенсорного ввода */
        min-width: 44px;
    }
}

@media (any-pointer: fine) {
    /* Для устройств с точным вводом (мышь) */
    .button {
        min-height: 32px;
        min-width: 32px;
    }
}
```

## Скрытие и отображение контента

### 1. Правильное скрытие элементов

```css
/* Класс для скрытия элементов только визуально (доступно для скринридеров) */
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}

/* Класс для скрытия элементов, но с возможностью фокуса */
.sr-only.focusable:focus,
.sr-only.focusable:active {
    position: static;
    width: auto;
    height: auto;
    margin: 0;
    overflow: visible;
    clip: auto;
    white-space: normal;
}

/* Правильное скрытие для доступности */
.visually-hidden {
    position: absolute !important;
    height: 1px;
    width: 1px;
    overflow: hidden;
    clip: rect(1px, 1px, 1px, 1px);
}

/* Полное скрытие (недоступно для всех) */
.hidden {
    display: none !important;
}

/* Условное скрытие с доступностью */
.accessible-hidden[aria-hidden="true"] {
    display: none;
}

/* Скрытие при загрузке, показ при готовности */
.loading-content[aria-hidden="true"] {
    display: none;
}

/* Плавное скрытие с сохранением доступности */
.fade-out {
    opacity: 0;
    visibility: hidden;
    transition: opacity 0.3s ease, visibility 0.3s ease;
}

.fade-out[aria-hidden="true"] {
    display: none;
}

/* Пример использования */
.skip-link {
    position: absolute;
    top: -40px;
    left: 6px;
    background: #000;
    color: #fff;
    padding: 8px;
    text-decoration: none;
    border-radius: 4px;
    z-index: 1000;
    transition: top 0.3s ease;
}

.skip-link:focus {
    top: 6px;
}
```

### 2. Адаптивные скрытия

```css
/* Адаптивное скрытие для разных размеров экрана */
@media (max-width: 767px) {
    .desktop-only {
        display: none !important;
    }
    
    .mobile-accessible {
        display: block !important;
    }
}

@media (min-width: 768px) {
    .mobile-only {
        display: none !important;
    }
    
    .desktop-accessible {
        display: block !important;
    }
}

/* Скрытие для печати */
@media print {
    .no-print {
        display: none !important;
    }
    
    .print-accessible {
        display: block !important;
    }
}

/* Скрытие для скринридеров при определенных условиях */
@media screen and (prefers-reduced-motion: reduce) {
    .animation-dependent {
        display: none;
    }
    
    .static-alternative {
        display: block;
    }
}
```

## Стилизация форм и интерактивных элементов

### 1. Доступные формы

```css
/* Стилизация формы с учетом доступности */
.form-container {
    max-width: 600px;
    margin: 0 auto;
    padding: 1rem;
}

.form-group {
    margin-bottom: 1rem;
}

.form-label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: 500;
    color: var(--text-primary);
}

.form-input,
.form-textarea,
.form-select {
    width: 100%;
    padding: 0.75rem;
    border: 2px solid #ced4da;
    border-radius: 4px;
    font-size: 1rem;
    transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
    background-color: var(--bg-primary);
    color: var(--text-primary);
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
    outline: none;
    border-color: #80bdff;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Индикация ошибок */
.form-input:invalid,
.form-textarea:invalid,
.form-select:invalid {
    border-color: #dc3545;
}

.form-input:valid {
    border-color: #28a745;
}

.form-error {
    color: #dc3545;
    font-size: 0.875rem;
    margin-top: 0.25rem;
    display: block;
}

.form-hint {
    color: var(--text-secondary);
    font-size: 0.875rem;
    margin-top: 0.25rem;
    display: block;
}

/* Стилизация чекбоксов и радио-кнопок */
.checkbox-container,
.radio-container {
    display: flex;
    align-items: center;
    margin-bottom: 0.5rem;
    cursor: pointer;
    position: relative;
    padding-left: 1.5rem;
}

.checkbox-input,
.radio-input {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    height: 0;
    width: 0;
}

.checkbox-custom,
.radio-custom {
    position: absolute;
    left: 0;
    top: 0.25rem;
    height: 1.25rem;
    width: 1.25rem;
    background-color: var(--bg-primary);
    border: 2px solid #ced4da;
    border-radius: 4px;
}

.radio-custom {
    border-radius: 50%;
}

.checkbox-input:checked ~ .checkbox-custom {
    background-color: #007bff;
    border-color: #007bff;
}

.radio-input:checked ~ .radio-custom {
    background-color: #007bff;
    border-color: #007bff;
}

.checkbox-custom::after,
.radio-custom::after {
    content: "";
    position: absolute;
    display: none;
}

.checkbox-input:checked ~ .checkbox-custom::after {
    display: block;
}

.checkbox-custom::after {
    left: 0.4rem;
    top: 0.1rem;
    width: 0.3rem;
    height: 0.6rem;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: rotate(45deg);
}

.radio-custom::after {
    left: 0.4rem;
    top: 0.4rem;
    width: 0.4rem;
    height: 0.4rem;
    border-radius: 50%;
    background: white;
}

/* Фокус на кастомных чекбоксах */
.checkbox-input:focus ~ .checkbox-custom,
.radio-input:focus ~ .radio-custom {
    box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
}
```

### 2. Доступные кнопки и ссылки

```css
/* Стилизация кнопок с доступностью */
.button {
    display: inline-block;
    font-weight: 400;
    text-align: center;
    vertical-align: middle;
    user-select: none;
    border: 1px solid transparent;
    padding: 0.375rem 0.75rem;
    font-size: 1rem;
    line-height: 1.5;
    border-radius: 0.25rem;
    transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
    text-decoration: none;
    cursor: pointer;
    min-height: 44px; /* Минимальный размер для доступности */
    min-width: 44px;
}

.button:focus {
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.button:disabled {
    opacity: 0.65;
    cursor: not-allowed;
}

/* Варианты кнопок */
.button-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}

.button-primary:hover:not(:disabled) {
    background-color: #0069d9;
    border-color: #0062cc;
}

.button-secondary {
    color: #fff;
    background-color: #6c757d;
    border-color: #6c757d;
}

.button-outline {
    color: #007bff;
    background-color: transparent;
    border-color: #007bff;
}

.button-outline:hover:not(:disabled) {
    background-color: #007bff;
    color: #fff;
}

/* Ссылки с хорошей видимостью */
.link {
    color: #007bff;
    text-decoration: underline;
    transition: color 0.15s ease-in-out;
}

.link:hover {
    color: #0056b3;
}

.link:focus {
    outline: 2px solid #007bff;
    outline-offset: 2px;
    border-radius: 2px;
}

/* Улучшенная видимость ссылок */
.link-enhanced {
    padding: 2px 4px;
    border-radius: 4px;
    background-color: rgba(0, 123, 255, 0.1);
}

/* Стилизация кнопок-иконок */
.icon-button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 2.5rem;
    height: 2.5rem;
    border: none;
    background: none;
    border-radius: 50%;
    cursor: pointer;
    transition: background-color 0.15s ease;
}

.icon-button:focus {
    outline: 2px solid #007bff;
    outline-offset: 2px;
}

.icon-button:hover {
    background-color: rgba(0, 0, 0, 0.1);
}
```

## Адаптивный дизайн и доступность

### 1. Адаптация к размеру текста

```css
/* Использование относительных единиц для доступности */
html {
    font-size: 16px; /* Базовый размер */
}

/* Использование rem для размеров */
.container {
    max-width: 75rem; /* 1200px при базовом 16px */
    padding: 1rem; /* 16px */
    margin: 0 auto;
}

.card {
    padding: 1.5rem; /* 24px */
    border-radius: 0.5rem; /* 8px */
}

/* Адаптация к увеличению текста */
@media screen and (min-width: 1200px) {
    html {
        font-size: 18px; /* Увеличенный базовый размер */
    }
}

/* Гибкие размеры шрифтов */
.responsive-heading {
    font-size: clamp(1.5rem, 4vw, 2.5rem);
    /* min, preferred, max */
}

.responsive-text {
    font-size: clamp(1rem, 2.5vw, 1.25rem);
}

/* Адаптация интервалов */
.spaced-section {
    padding: clamp(1rem, 5vw, 3rem);
    margin: clamp(0.5rem, 2vw, 1rem);
}

/* Гибкие размеры элементов */
.adaptive-card {
    width: min(90%, 600px);
    margin: 0 auto;
    padding: clamp(1rem, 3vw, 2rem);
}
```

### 2. Адаптация к различным условиям

```css
/* Адаптация к разным условиям освещения */
@media (prefers-contrast: high) {
    .card {
        border: 2px solid #000000;
        background-color: #ffffff;
        color: #000000;
    }
    
    .button {
        border: 2px solid #000000;
        background-color: #ffffff;
        color: #000000;
    }
    
    .text {
        color: #000000;
        background-color: #ffffff;
    }
}

/* Адаптация к движению */
@media (prefers-reduced-motion: reduce) {
    .animation-element {
        animation: none;
        transition: none;
    }
    
    .smooth-scroll {
        scroll-behavior: auto;
    }
}

/* Адаптация к темной теме */
@media (prefers-color-scheme: dark) {
    :root {
        --bg-color: #121212;
        --text-color: #e0e0e0;
        --border-color: #333333;
        --card-bg: #1e1e1e;
    }
    
    body {
        background-color: var(--bg-color);
        color: var(--text-color);
    }
    
    .card {
        background-color: var(--card-bg);
        border-color: var(--border-color);
    }
}

/* Адаптация к сенсорным устройствам */
@media (hover: none) and (pointer: coarse) {
    /* Для сенсорных устройств */
    .touch-target {
        min-height: 44px;
        min-width: 44px;
        padding: 12px;
    }
    
    .hover-effect {
        /* Убираем hover-эффекты для сенсорных устройств */
        transition: none;
    }
}

@media (hover: hover) and (pointer: fine) {
    /* Для устройств с мышью */
    .hover-effect:hover {
        transform: translateY(-2px);
    }
}
```

## Современные возможности CSS для доступности

### 1. CSS Custom Properties и доступность

```css
/* Использование CSS переменных для темизации */
:root {
    /* Цвета по умолчанию (светлая тема) */
    --color-bg: #ffffff;
    --color-text: #212529;
    --color-text-secondary: #6c757d;
    --color-border: #dee2e6;
    --color-accent: #007bff;
    --color-accent-hover: #0056b3;
    --color-success: #28a745;
    --color-warning: #ffc107;
    --color-danger: #dc3545;
    
    /* Размеры */
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
    
    /* Интервалы */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;
}

/* Темная тема */
[data-theme="dark"] {
    --color-bg: #212529;
    --color-text: #e9ecef;
    --color-text-secondary: #adb5bd;
    --color-border: #495057;
    --color-accent: #6ea8fe;
    --color-accent-hover: #9ec5ff;
}

/* Контрастная тема */
[data-theme="high-contrast"] {
    --color-bg: #ffffff;
    --color-text: #000000;
    --color-border: #000000;
    --color-accent: #0000ff;
    --color-accent-hover: #0000cc;
}

/* Использование переменных в компонентах */
.card {
    background-color: var(--color-bg);
    color: var(--color-text);
    border: 1px solid var(--color-border);
    border-radius: var(--spacing-sm);
    padding: var(--spacing-md);
    transition: all 0.2s ease;
}

.button {
    background-color: var(--color-accent);
    color: white;
    border: 1px solid var(--color-accent);
    padding: var(--spacing-sm) var(--spacing-md);
    border-radius: var(--spacing-xs);
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.button:hover {
    background-color: var(--color-accent-hover);
}

/* Адаптация к предпочтениям пользователя */
@media (prefers-color-scheme: dark) {
    :root:not([data-theme]) {
        --color-bg: #212529;
        --color-text: #e9ecef;
        --color-text-secondary: #adb5bd;
        --color-border: #495057;
    }
}
```

### 2. Container Queries и доступность

```css
/* Использование Container Queries для адаптивности */
.card-container {
    container-type: inline-size;
    container-name: card-container;
    width: 100%;
    max-width: 400px;
    margin: 0 auto;
}

.card {
    padding: 1rem;
    background: var(--color-bg);
    color: var(--color-text);
    border: 1px solid var(--color-border);
    border-radius: 8px;
    transition: all 0.2s ease;
}

/* Адаптация к размеру контейнера */
@container card-container (min-width: 300px) {
    .card {
        padding: 1.5rem;
    }
    
    .card-header {
        font-size: 1.25rem;
    }
}

@container card-container (min-width: 350px) {
    .card {
        display: grid;
        grid-template-columns: 80px 1fr;
        gap: 1rem;
        align-items: start;
    }
    
    .card-image {
        grid-row: 1 / span 2;
    }
}

/* Учет доступности в Container Queries */
@container card-container (min-width: 400px) and (prefers-reduced-motion: no-preference) {
    .card:hover {
        transform: translateY(-4px);
        box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
    }
}
```

### 3. Logical Properties и RTL

```css
/* Использование Logical Properties для международной доступности */
.text-container {
    margin-inline-start: 1rem;    /* Вместо margin-left */
    margin-inline-end: 1rem;      /* Вместо margin-right */
    padding-inline: 1rem;         /* Вместо padding-left и padding-right */
    text-align: start;            /* Вместо left/right - учитывает направление текста */
}

/* Адаптация размеров */
.layout-box {
    inline-size: 100%;            /* Вместо width - учитывает направление */
    block-size: 200px;            /* Вместо height */
    min-inline-size: 300px;       /* Вместо min-width */
    max-block-size: 400px;        /* Вместо max-height */
}

/* Адаптация границ */
.border-container {
    border-inline-start: 2px solid var(--color-border);  /* Вместо border-left */
    border-inline-end: 2px solid var(--color-border);    /* Вместо border-right */
}

/* Адаптация отступов в сетке */
.grid-container {
    display: grid;
    gap: 1rem;
    padding-inline: 1rem;         /* Вместо padding-left и padding-right */
}

.grid-item {
    padding-block: 1rem;          /* Вместо padding-top и padding-bottom */
    padding-inline: 1rem;         /* Вместо padding-left и padding-right */
}

/* Адаптация позиционирования */
.positioned-element {
    inset-inline-start: 1rem;     /* Вместо left */
    inset-inline-end: 1rem;       /* Вместо right */
    inset-block-start: 1rem;      /* Вместо top */
    inset-block-end: 1rem;        /* Вместо bottom */
}

/* Использование в компонентах */
.component {
    margin-inline: var(--spacing-md);     /* Автоматическая адаптация */
    padding-inline: var(--spacing-lg);
    border-inline-start-width: 2px;      /* Сохранение логики границ */
}
```

## Лучшие практики и примеры

### 1. Система компонентов с доступностью

```css
/* Доступная система компонентов */
:root {
    /* Цветовая палитра с доступностью */
    --color-primary: #007bff;
    --color-primary-dark: #0056b3;
    --color-primary-light: #80bdff;
    --color-secondary: #6c757d;
    --color-success: #28a745;
    --color-danger: #dc3545;
    --color-warning: #ffc107;
    --color-info: #17a2b8;
    
    /* Цвета фона и текста */
    --color-bg: #ffffff;
    --color-bg-secondary: #f8f9fa;
    --color-text: #212529;
    --color-text-muted: #6c757d;
    
    /* Размеры */
    --font-size-xs: 0.75rem;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-xl: 1.25rem;
    
    /* Интервалы */
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;
    
    /* Радиусы */
    --border-radius: 0.25rem;
    --border-radius-lg: 0.3rem;
    --border-radius-sm: 0.2rem;
    
    /* Тени */
    --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
    --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
    
    /* Переходы */
    --transition-base: 0.15s ease-in-out;
    --transition-fade: 0.15s linear;
}

/* Базовые стили с доступностью */
* {
    box-sizing: border-box;
}

*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    font-size: var(--font-size-base);
    line-height: 1.5;
    color: var(--color-text);
    background-color: var(--color-bg);
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

/* Доступные компоненты */
.accessible-button {
    display: inline-block;
    padding: 0.375rem 0.75rem;
    font-weight: 400;
    line-height: 1.5;
    text-align: center;
    text-decoration: none;
    vertical-align: middle;
    cursor: pointer;
    user-select: none;
    border: 1px solid transparent;
    border-radius: var(--border-radius);
    transition: var(--transition-base);
    min-height: 44px; /* Для доступности */
    min-width: 44px;
}

.accessible-button:focus {
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.accessible-button--primary {
    color: #fff;
    background-color: var(--color-primary);
    border-color: var(--color-primary);
}

.accessible-button--primary:hover:not(:disabled) {
    background-color: var(--color-primary-dark);
    border-color: var(--color-primary-dark);
}

.accessible-card {
    position: relative;
    display: flex;
    flex-direction: column;
    min-width: 0;
    word-wrap: break-word;
    background-color: var(--color-bg);
    background-clip: border-box;
    border: 1px solid rgba(0, 0, 0, 0.125);
    border-radius: var(--border-radius);
    transition: var(--transition-base);
}

.accessible-card:focus-within {
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.accessible-card__body {
    flex: 1 1 auto;
    padding: var(--spacing-lg);
}

/* Формы с доступностью */
.accessible-form {
    max-width: 600px;
    margin: 0 auto;
    padding: var(--spacing-lg);
}

.accessible-form__group {
    margin-bottom: var(--spacing-md);
}

.accessible-form__label {
    display: block;
    margin-bottom: var(--spacing-xs);
    font-weight: 500;
    color: var(--color-text);
}

.accessible-form__input {
    display: block;
    width: 100%;
    padding: 0.375rem 0.75rem;
    font-size: var(--font-size-base);
    font-weight: 400;
    line-height: 1.5;
    color: var(--color-text);
    background-color: var(--color-bg);
    background-clip: padding-box;
    border: 1px solid #ced4da;
    border-radius: var(--border-radius);
    transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
    min-height: calc(1.5em + 0.75rem + 2px);
}

.accessible-form__input:focus {
    color: var(--color-text);
    background-color: var(--color-bg);
    border-color: #80bdff;
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.accessible-form__input:disabled {
    background-color: #e9ecef;
    opacity: 1;
}

.accessible-form__input:invalid:not(:focus) {
    border-color: var(--color-danger);
}

/* Скрытие с доступностью */
.sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}

.sr-only.focusable:focus,
.sr-only.focusable:active {
    position: static;
    width: auto;
    height: auto;
    margin: 0;
    overflow: visible;
    clip: auto;
    white-space: normal;
}

/* Ссылки с доступностью */
.accessible-link {
    color: var(--color-primary);
    text-decoration: underline;
    transition: color 0.15s ease-in-out;
}

.accessible-link:hover {
    color: var(--color-primary-dark);
    text-decoration: underline;
}

.accessible-link:focus {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
    border-radius: 2px;
}
```

### 2. Примеры из промышленной разработки

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступный интерфейс</title>
    <style>
        /* Импортировать стили из примера выше */
        :root {
            --color-primary: #007bff;
            --color-primary-dark: #0056b3;
            --color-bg: #ffffff;
            --color-text: #212529;
            --spacing-md: 1rem;
            --border-radius: 0.25rem;
            --transition-base: 0.15s ease-in-out;
        }

        body {
            margin: 0;
            font-family: system-ui, sans-serif;
            background-color: var(--color-bg);
            color: var(--color-text);
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: var(--spacing-md);
        }

        .accessible-button {
            display: inline-block;
            padding: 0.375rem 0.75rem;
            font-weight: 400;
            line-height: 1.5;
            text-align: center;
            text-decoration: none;
            vertical-align: middle;
            cursor: pointer;
            user-select: none;
            border: 1px solid transparent;
            border-radius: var(--border-radius);
            transition: var(--transition-base);
            min-height: 44px;
            min-width: 44px;
        }

        .accessible-button:focus {
            outline: 0;
            box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
        }

        .accessible-button--primary {
            color: #fff;
            background-color: var(--color-primary);
            border-color: var(--color-primary);
        }

        .accessible-button--primary:hover:not(:disabled) {
            background-color: var(--color-primary-dark);
            border-color: var(--color-primary-dark);
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Доступный интерфейс</h1>
        
        <!-- Ссылка для пропуска навигации -->
        <a href="#main-content" class="sr-only sr-only-focusable">
            Перейти к основному содержимому
        </a>
        
        <nav aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
        
        <main id="main-content">
            <h2>Основной контент</h2>
            
            <form>
                <div class="form-group">
                    <label for="email">Email</label>
                    <input 
                        type="email" 
                        id="email" 
                        name="email"
                        required
                        aria-describedby="email-help email-error">
                    <div id="email-help" class="form-hint">Введите ваш email</div>
                    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <button type="submit" class="accessible-button accessible-button--primary">
                    Отправить
                </button>
            </form>
        </main>
    </div>
    
    <script>
        // Пример обработки формы с доступностью
        document.querySelector('form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const email = document.getElementById('email');
            const errorElement = document.getElementById('email-error');
            
            if (!email.value || !email.validity.valid) {
                email.setAttribute('aria-invalid', 'true');
                errorElement.textContent = 'Пожалуйста, введите корректный email';
                errorElement.hidden = false;
                
                // Фокус на поле с ошибкой
                email.focus();
            } else {
                email.setAttribute('aria-invalid', 'false');
                errorElement.textContent = '';
                errorElement.hidden = true;
                
                // Сообщение об успехе
                const successMessage = document.createElement('div');
                successMessage.setAttribute('role', 'status');
                successMessage.setAttribute('aria-live', 'polite');
                successMessage.textContent = 'Форма успешно отправлена!';
                document.querySelector('form').appendChild(successMessage);
                
                // Удаление сообщения через 5 секунд
                setTimeout(() => {
                    successMessage.remove();
                }, 5000);
            }
        });
    </script>
</body>
</html>
```

## Ссылки на связанные темы
- [[best-practices/best-practices]] - Лучшие практики CSS
- [[html/accessibility]] - Доступность HTML
- [[js/accessibility]] - Доступность JavaScript
- [[web-standards]] - Веб-стандарты

#programming #css #accessibility #a11y #inclusive-design #best-practices