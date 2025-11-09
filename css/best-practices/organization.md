# Лучшие практики CSS: организация кода, производительность и поддержка

## Введение

CSS (Cascading Style Sheets) является фундаментальной технологией веб-разработки, но его неправильное использование может привести к проблемам с производительностью, поддержкой и масштабируемостью. В этой статье мы рассмотрим лучшие практики написания CSS-кода, которые помогут создавать чистый, эффективный и легко поддерживаемый код.

## Организация CSS-кода

### 1. Структура файлов и импорт

```css
/* Пример структуры CSS-файлов по методологии ITCSS (Inverted Triangle CSS) */

/* 1. Settings - переменные и настройки */
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  --border-radius: 0.25rem;
  --border-radius-lg: 0.5rem;
  --border-radius-sm: 0.2rem;
  
  --transition-base: 0.15s ease-in-out;
  --transition-fade: 0.15s linear;
}

/* 2. Tools - миксины и функции (в препроцессорах) */
/* Определяются в Sass/Less/Stylus */

/* 3. Generic - сброс стилей и нормализация */
* {
  box-sizing: border-box;
}

*::before,
*::after {
  box-sizing: border-box;
}

html {
  font-size: 16px;
  line-height: 1.5;
}

body {
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-size: var(--font-size-base);
  line-height: 1.5;
  color: #212529;
  background-color: #fff;
}

/* 4. Elements - стили для HTML элементов */
h1, h2, h3, h4, h5, h6 {
  margin-top: 0;
  margin-bottom: 0.5rem;
  font-weight: 500;
  line-height: 1.2;
}

h1 { font-size: calc(1.375rem + 1.5vw); }
h2 { font-size: calc(1.325rem + 0.9vw); }
h3 { font-size: calc(1.3rem + 0.6vw); }
h4 { font-size: calc(1.275rem + 0.3vw); }
h5 { font-size: 1.25rem; }
h6 { font-size: 1rem; }

p {
  margin-top: 0;
  margin-bottom: 1rem;
}

a {
  color: var(--color-primary);
  text-decoration: none;
}

a:hover {
  color: #0056b3;
  text-decoration: underline;
}

/* 5. Objects - OOCSS объекты */
.o-layout {
  display: flex;
}

.o-layout__item {
  flex: 1;
}

.o-media {
  display: flex;
  align-items: flex-start;
}

.o-media__img {
  margin-right: 1rem;
}

.o-media__body {
  flex: 1;
}

/* 6. Components - конкретные компоненты */
.c-button {
  display: inline-block;
  font-weight: 400;
  line-height: 1.5;
  color: #212529;
  text-align: center;
  text-decoration: none;
  vertical-align: middle;
  cursor: pointer;
  user-select: none;
  background-color: transparent;
  border: 1px solid transparent;
  padding: 0.375rem 0.75rem;
  font-size: var(--font-size-base);
  border-radius: var(--border-radius);
  transition: var(--transition-base);
}

.c-button--primary {
  color: #fff;
  background-color: var(--color-primary);
  border-color: var(--color-primary);
}

.c-button--primary:hover {
  color: #fff;
  background-color: #0069d9;
  border-color: #0062cc;
}

.c-card {
  position: relative;
  display: flex;
  flex-direction: column;
  min-width: 0;
  word-wrap: break-word;
  background-color: #fff;
  background-clip: border-box;
  border: 1px solid rgba(0, 0, 0, 0.125);
  border-radius: var(--border-radius);
}

.c-card__body {
  flex: 1 1 auto;
  padding: 1rem 1rem;
}

/* 7. Utilities - вспомогательные классы */
.u-hidden {
  display: none !important;
}

.u-screen-reader-only {
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

.u-text-center { text-align: center !important; }
.u-text-left { text-align: left !important; }
.u-text-right { text-align: right !important; }

.u-float-left { float: left !important; }
.u-float-right { float: right !important; }
.u-float-none { float: none !important; }

.u-clearfix::after {
  display: block;
  clear: both;
  content: "";
}
```

### 2. Именование классов

```css
/* Рекомендуется: БЭМ (Block Element Modifier) */
/* Пример БЭМ нотации */
.menu { }
.menu__item { }
.menu__link { }
.menu--horizontal { }
.menu__item--active { }

/* Хорошо: понятные и описательные имена */
.user-profile { }
.user-profile__avatar { }
.user-profile__name { }
.user-profile--large { }

/* Плохо: неописательные имена */
.a { }
.b { }
.c { }
.red-button { } /* зависит от визуального стиля */
.big-font { }   /* зависит от размера шрифта */
```

### 3. Комментарии и документация

```css
/**
 * Карточка пользователя
 * 
 * Компонент для отображения информации о пользователе
 * 
 * @class user-card
 * @classdesc Основной компонент карточки пользователя
 * 
 * @state user-card--featured - выделенная карточка
 * @state user-card--loading - карточка в состоянии загрузки
 * 
 * @example
 * <div class="user-card user-card--featured">
 *   <img class="user-card__avatar" src="avatar.jpg" alt="Avatar">
 *   <h3 class="user-card__name">Иван Иванов</h3>
 *   <p class="user-card__position">Разработчик</p>
 * </div>
 */

.user-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  border: 1px solid #dee2e6;
  border-radius: 0.375rem;
  background-color: #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.user-card__avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  margin-bottom: 0.5rem;
}

.user-card__name {
  font-size: 1.125rem;
  font-weight: 600;
  margin: 0 0 0.25rem 0;
}

.user-card__position {
  color: #6c757d;
  margin: 0;
}

/* Модификатор: выделенная карточка */
.user-card--featured {
  border-color: #007bff;
  box-shadow: 0 4px 6px rgba(0, 123, 255, 0.15);
}
```

## Производительность CSS

### 1. Эффективные селекторы

```css
/* ПЛОХО: сложные и медленные селекторы */
div#header ul.navigation li a:hover { color: red; }
body div.container section.article div.content p.text span.highlighted { color: yellow; }

/* ХОРОШО: простые и эффективные селекторы */
.nav-link:hover { color: red; }
.text-highlight { color: yellow; }

/* ПЛОХО: избыточная специфичность */
ul.nav li a { color: blue; }

/* ЛУЧШЕ: минимальная специфичность */
.nav-link { color: blue; }

/* ИЗБЕГАЙТЕ: тег-селекторы без классов для сложных структур */
/* Плохо */
.header nav ul li a { }

/* Хорошо */
.main-navigation-link { }
```

### 2. Оптимизация рендеринга

```css
/* Используйте transform и opacity для анимаций (GPU-accelerated) */
.smooth-animation {
  transform: translateX(0);
  opacity: 1;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.smooth-animation:hover {
  transform: translateX(10px);
  opacity: 0.8;
}

/* Избегайте анимации свойств, вызывающих layout/reflow */
.bad-animation {
  /* Плохо: вызывает reflow */
  width: 100px;
  height: 100px;
  margin: 10px;
  transition: width 0.3s ease, height 0.3s ease, margin 0.3s ease;
}

/* Лучше: используйте transform */
.good-animation {
  /* Хорошо: GPU-accelerated */
  transform: scale(1) translate(0, 0);
  transition: transform 0.3s ease;
}

/* Используйте will-change с осторожностью */
.optimized-element {
  will-change: transform, opacity;
  /* Используйте только перед анимацией и удаляйте после */
}

/* Используйте contain для изоляции компонентов */
.component-container {
  contain: layout style paint;
  /* Ограничивает влияние изменений на остальную страницу */
}
```

### 3. Минификация и оптимизация

```css
/* До минификации */
.header {
  background-color: #ffffff;
  padding: 1rem;
  margin: 0;
  border: 1px solid #cccccc;
  border-radius: 4px;
}

/* После минификации */
.header{background-color:#fff;padding:1rem;margin:0;border:1px solid #ccc;border-radius:4px}

/* Использование сокращений */
/* Вместо */
.element {
  margin-top: 10px;
  margin-right: 20px;
  margin-bottom: 10px;
  margin-left: 20px;
}

/* Лучше */
.element {
  margin: 10px 20px; /* сокращение для всех сторон */
}

/* Использование CSS-переменных для поддержки */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --border-radius: 0.25rem;
}

.button {
  background-color: var(--primary-color);
  border-radius: var(--border-radius);
}
```

## Современные возможности CSS

### 1. CSS Custom Properties (переменные)

```css
/* Централизованное управление темами */
:root {
  /* Цветовая палитра */
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
  
  --border-radius: 0.25rem;
  --border-radius-lg: 0.3rem;
  --border-radius-sm: 0.2rem;
  
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Тени */
  --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
  --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
  
  /* Анимации */
  --transition-base: 0.15s ease-in-out;
  --transition-fade: 0.15s linear;
}

/* Использование переменных */
.card {
  background-color: var(--color-bg);
  color: var(--color-text);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow);
  transition: box-shadow var(--transition-base);
}

.card:hover {
  box-shadow: var(--shadow-lg);
}

/* Темизация */
[data-theme="dark"] {
  --color-bg: #212529;
  --color-bg-secondary: #343a40;
  --color-text: #ffffff;
  --color-text-muted: #adb5bd;
}

/* Переменные с fallback значениями */
.element {
  background-color: var(--custom-color, #333);
  /* Если --custom-color не определена, используется #333 */
}
```

### 2. Logical Properties

```css
/* Использование логических свойств для международной поддержки */
.text-container {
  /* Вместо margin-left/right */
  margin-inline-start: 1rem;    /* Работает с RTL */
  margin-inline-end: 1rem;      /* Работает с RTL */
  margin-block-start: 0.5rem;   /* Вместо margin-top */
  margin-block-end: 0.5rem;     /* Вместо margin-bottom */
  
  /* Padding */
  padding-inline: 1rem;         /* Работает с RTL */
  padding-block: 0.5rem;        /* Работает с LTR/RTL */
  
  /* Border */
  border-inline-start: 2px solid var(--color-border); /* Работает с RTL */
  
  /* Text alignment */
  text-align: start;            /* Работает с LTR/RTL */
}

/* Размеры */
.layout-box {
  /* Вместо width/height */
  inline-size: 100%;            /* Работает с LTR/RTL */
  block-size: 200px;            /* Независимо от направления */
  
  /* Min/max размеры */
  min-inline-size: 300px;       /* Работает с RTL */
  max-block-size: 400px;        /* Независимо от направления */
}
```

### 3. Container Queries (экспериментально)

```css
/* Пример использования Container Queries */
.card-container {
  container-type: inline-size;
  container-name: card;
  width: 100%;
  max-width: 400px;
  margin: 0 auto;
}

.card {
  padding: 1rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Адаптация на основе размера контейнера */
@container card (min-width: 300px) {
  .card {
    padding: 1.5rem;
  }
  
  .card-header {
    font-size: 1.25rem;
  }
}

@container card (min-width: 350px) {
  .card {
    display: grid;
    grid-template-columns: 80px 1fr;
    gap: 1rem;
    align-items: start;
  }
}
```

## Практики поддержки и масштабирования

### 1. Модульность и повторное использование

```css
/* Создание переиспользуемых модулей */
/* Объекты OOCSS */
.media {
  display: flex;
  align-items: flex-start;
}

.media__img {
  margin-right: 1rem;
  flex-shrink: 0;
}

.media__body {
  flex: 1;
}

/* Утилиты */
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

.text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.d-flex { display: flex; }
.d-block { display: block; }
.d-none { display: none; }

.justify-content-center { justify-content: center; }
.align-items-center { align-items: center; }

/* Компоненты */
.btn {
  display: inline-block;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  text-align: center;
  text-decoration: none;
  vertical-align: middle;
  cursor: pointer;
  user-select: none;
  border: 1px solid transparent;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.btn-primary {
  color: #fff;
  background-color: #007bff;
  border-color: #007bff;
}

.btn-primary:hover {
  color: #fff;
  background-color: #0069d9;
  border-color: #0062cc;
}
```

### 2. Версионирование и управление зависимостями

```css
/* Пример версионирования компонентов */
/* v1.0.0 - первая версия */
.card-v1 {
  background: white;
  padding: 1rem;
  border: 1px solid #ddd;
}

/* v2.0.0 - обновленная версия с улучшениями */
.card-v2 {
  background: white;
  padding: 1.5rem;
  border: 1px solid #dee2e6;
  border-radius: 0.375rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

/* Лучше: использовать модификаторы вместо версионирования */
.card {
  background: white;
  padding: 1rem;
  border: 1px solid #dee2e6;
  border-radius: 0.375rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.card--compact {
  padding: 0.75rem;
}

.card--spacious {
  padding: 2rem;
}
```

### 3. Тестирование CSS

```css
/* Стили для тестирования доступности */
/* Псевдоклассы для тестирования состояний */
.btn {
  padding: 0.5rem 1rem;
  border: 1px solid #007bff;
  background: #007bff;
  color: white;
  cursor: pointer;
}

.btn:hover,
.btn:focus {
  background: #0056b3;
  border-color: #0056b3;
}

.btn:focus {
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Проверка контрастности */
.high-contrast:where(:not(.btn, .link)) {
  background: #000 !important;
  color: #fff !important;
}

/* Стили для тестирования масштабирования */
@media screen and (min-width: 1200px) {
  /* Проверка поведения на больших экранах */
  .responsive-element {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

## Лучшие практики для командной разработки

### 1. Соглашения по кодированию

```css
/* Пример .stylelintrc.json для автоматической проверки */
/*
{
  "extends": ["stylelint-config-standard"],
  "rules": {
    "indentation": 2,
    "string-quotes": "single",
    "no-eol-whitespace": true,
    "max-empty-lines": 1,
    "selector-max-id": 0,
    "selector-combinator-space-after": "always",
    "declaration-colon-space-before": "never",
    "declaration-colon-space-after": "always",
    "block-opening-brace-space-before": "always",
    "block-closing-brace-newline-after": "always"
  }
}
*/

/* Порядок свойств по группам */
.component {
  /* Позиционирование */
  position: relative;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 1;
  
  /* Блочная модель */
  display: block;
  float: right;
  width: 100px;
  height: 100px;
  padding: 10px;
  margin: 10px;
  overflow: hidden;
  box-sizing: border-box;
  border: 1px solid #e5e5e5;
  
  /* Типографика */
  font: normal 16px sans-serif;
  line-height: 1.4;
  text-align: center;
  text-transform: none;
  letter-spacing: normal;
  word-wrap: break-word;
  
  /* Визуальные */
  background: #f5f5f5;
  border-radius: 3px;
  opacity: 1;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
  
  /* Анимации */
  transition: all 0.3s ease;
  
  /* Разное */
  appearance: button;
}
```

### 2. Инструменты автоматизации

```json
{
  "name": "css-best-practices-project",
  "scripts": {
    "lint:css": "stylelint 'src/**/*.css'",
    "format:css": "prettier --write 'src/**/*.css'",
    "build:css": "postcss src/css/main.css -o dist/css/main.min.css",
    "optimize:css": "cssnano --preset advanced dist/css/main.min.css dist/css/main.optimized.css"
  },
  "devDependencies": {
    "stylelint": "^15.0.0",
    "stylelint-config-standard": "^33.0.0",
    "prettier": "^3.0.0",
    "postcss-cli": "^10.0.0",
    "cssnano": "^6.0.0",
    "autoprefixer": "^10.4.0"
  }
}
```

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('autoprefixer'), // Автоматические вендорные префиксы
    require('cssnano')({     // Минификация
      preset: [
        'default',
        {
          discardComments: {
            removeAll: true
          }
        }
      ]
    })
  ]
};
```

### 3. Документация компонентов

```css
/**
 * Кнопка
 * 
 * Основной интерактивный элемент для выполнения действий
 * 
 * @module Button
 * @author Frontend Team
 * @version 2.1.0
 * 
 * @example
 * <button class="btn btn--primary">Primary Button</button>
 * <button class="btn btn--secondary">Secondary Button</button>
 * <button class="btn btn--large">Large Button</button>
 * 
 * @modifier btn--primary Основная кнопка
 * @modifier btn--secondary Второстепенная кнопка
 * @modifier btn--large Большая кнопка
 * @modifier btn--disabled Неактивная кнопка
 * 
 * @state :hover Состояние наведения
 * @state :active Состояние активации
 * @state :focus Состояние фокуса
 * @state :disabled Состояние отключения
 * 
 * @property {Color} --btn-color Цвет текста
 * @property {Color} --btn-bg-color Цвет фона
 * @property {Color} --btn-border-color Цвет границы
 * @property {Length} --btn-padding-padding Отступы
 * @property {Length} --btn-border-radius Радиус скругления
 */
.btn {
  --btn-color: #fff;
  --btn-bg-color: #007bff;
  --btn-border-color: #007bff;
  --btn-padding: 0.375rem 0.75rem;
  --btn-border-radius: 0.25rem;
  
  display: inline-block;
  padding: var(--btn-padding);
  font-weight: 400;
  line-height: 1.5;
  color: var(--btn-color);
  text-align: center;
  text-decoration: none;
  vertical-align: middle;
  cursor: pointer;
  user-select: none;
  background-color: var(--btn-bg-color);
  border: 1px solid var(--btn-border-color);
  border-radius: var(--btn-border-radius);
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

/**
 * @modifier btn--secondary Второстепенная кнопка
 */
.btn--secondary {
  --btn-bg-color: #6c757d;
  --btn-border-color: #6c757d;
}

/**
 * @modifier btn--large Большая кнопка
 */
.btn--large {
  --btn-padding: 0.5rem 1rem;
  font-size: 1.25rem;
  line-height: 1.5;
  border-radius: 0.3rem;
}

/**
 * @state :disabled Неактивное состояние
 */
.btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}
```

## Примеры из промышленной разработки

### 1. Система компонентов

```css
/* Система компонентов с поддержкой темизации */
:root {
  /* Светлая тема по умолчанию */
  --color-bg: #ffffff;
  --color-text: #212529;
  --color-border: #dee2e6;
  --color-surface: #f8f9fa;
  --color-accent: #007bff;
  --color-accent-hover: #0056b3;
}

/* Темная тема */
[data-theme="dark"] {
  --color-bg: #212529;
  --color-text: #ffffff;
  --color-border: #495057;
  --color-surface: #343a40;
  --color-accent: #007bff;
  --color-accent-hover: #0069d9;
}

/* Базовый компонент */
.c-card {
  background-color: var(--color-bg);
  color: var(--color-text);
  border: 1px solid var(--color-border);
  border-radius: 0.375rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.15s ease-in-out;
}

.c-card__header {
  padding: 1rem 1rem 0.5rem;
  border-bottom: 1px solid var(--color-border);
  font-weight: 600;
}

.c-card__body {
  padding: 1rem;
}

.c-card__footer {
  padding: 0.5rem 1rem;
  border-top: 1px solid var(--color-border);
  font-size: 0.875rem;
  color: var(--color-text);
}

/* Интерактивные элементы в карточке */
.c-card .c-button {
  background-color: var(--color-accent);
  color: white;
  border: none;
  padding: 0.375rem 0.75rem;
  border-radius: 0.25rem;
  cursor: pointer;
  transition: background-color 0.15s ease-in-out;
}

.c-card .c-button:hover {
  background-color: var(--color-accent-hover);
}
```

### 2. Адаптивная сетка

```css
/* Адаптивная сетка с поддержкой промежуточных брейкпоинтов */
.grid {
  display: grid;
  gap: 1rem;
  padding: 1rem;
}

/* 1 колонка на мобильных */
.grid {
  grid-template-columns: 1fr;
}

/* 2 колонки на малых планшетах */
@media (min-width: 576px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 3 колонки на планшетах */
@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* 4 колонки на десктопах */
@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

/* 6 колонок на больших экранах */
@media (min-width: 1400px) {
  .grid {
    grid-template-columns: repeat(6, 1fr);
  }
}

/* Адаптивные карточки */
.grid-item {
  background: var(--color-surface);
  padding: 1rem;
  border-radius: 0.375rem;
  border: 1px solid var(--color-border);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.grid-item:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* Адаптивные отступы */
@media (max-width: 767px) {
  .grid {
    padding: 0.5rem;
    gap: 0.5rem;
  }
  
  .grid-item {
    padding: 0.75rem;
  }
}
```

### 3. Система форм

```css
/* Адаптивная система форм */
.form-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 1rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-row {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
  color: var(--color-text);
}

.form-input,
.form-textarea,
.form-select {
  width: 100%;
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  color: var(--color-text);
  background-color: var(--color-bg);
  background-clip: padding-box;
  border: 1px solid var(--color-border);
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  color: var(--color-text);
  background-color: var(--color-bg);
  border-color: var(--color-accent);
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.form-textarea {
  min-height: 100px;
  resize: vertical;
}

.form-button {
  display: inline-block;
  font-weight: 400;
  text-align: center;
  vertical-align: middle;
  user-select: none;
  border: 1px solid transparent;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  cursor: pointer;
}

.form-button--primary {
  color: #fff;
  background-color: var(--color-accent);
  border-color: var(--color-accent);
}

.form-button--primary:hover {
  background-color: var(--color-accent-hover);
  border-color: var(--color-accent-hover);
}

/* Адаптация для мобильных */
@media (max-width: 767px) {
  .form-row {
    flex-direction: column;
  }
  
  .form-container {
    padding: 0.5rem;
  }
}

/* Адаптация для планшетов и выше */
@media (min-width: 768px) {
  .form-row {
    flex-direction: row;
  }
  
  .form-group {
    flex: 1;
    margin-bottom: 0;
  }
  
  .form-group:not(:first-child) {
    margin-left: 1rem;
  }
}
```

## Ссылки на связанные темы
- [[methodology/methodology]] - Методологии CSS
- [[performance/performance]] - Производительность CSS
- [[responsive/responsive]] - Адаптивный дизайн
- [[html/best-practices]] - Лучшие практики HTML

#programming #css #best-practices #code-organization #performance #maintainability