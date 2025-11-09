# Кастомизация CSS-фреймворков

Кастомизация CSS-фреймворков - ключевой аспект создания уникальных и брендированных интерфейсов. В этом руководстве мы рассмотрим различные подходы к кастомизации популярных CSS-фреймворков и лучшие практики для их реализации.

## Подходы к кастомизации

### 1. CSS-переменные (CSS Custom Properties)

CSS-переменные - самый гибкий и современный способ кастомизации большинства фреймворков.

```css
/* Кастомизация Bootstrap через CSS-переменные */
:root {
  /* Цвета бренда */
  --bs-primary: #3498db;
  --bs-secondary: #2c3e50;
  --bs-success: #2ecc71;
  --bs-info: #1abc9c;
  --bs-warning: #f39c12;
  --bs-danger: #e74c3c;
  
  /* Типографика */
  --bs-font-sans-serif: "Inter", "Segoe UI", sans-serif;
  --bs-font-size-base: 1rem;
  --bs-line-height-base: 1.6;
  
  /* Радиусы */
  --bs-border-radius: 0.5rem;
  --bs-border-radius-sm: 0.25rem;
  --bs-border-radius-lg: 0.75rem;
  
  /* Отступы */
  --bs-spacer: 1rem;
  --bs-spacer-sm: 0.5rem;
  --bs-spacer-lg: 2rem;
}

/* Кастомизация Tailwind через CSS-переменные */
:root {
  --tw-brand-primary: #3498db;
  --tw-brand-secondary: #2c3e50;
  --tw-brand-accent: #e74c3c;
}

.custom-element {
  background-color: var(--tw-brand-primary);
  color: white;
  padding: var(--tw-spacer, 1rem);
  border-radius: var(--tw-border-radius, 0.25rem);
}
```

### 2. Sass/SCSS кастомизация

Для фреймворков, основанных на Sass (Bootstrap, Foundation), можно кастомизировать переменные до компиляции:

```scss
// custom-bootstrap.scss
// 1. Переопределение переменных
$primary: #3498db;
$secondary: #2c3e50;
$success: #2ecc71;
$info: #1abc9c;
$warning: #f39c12;
$danger: #e74c3c;

// Типографика
$font-family-sans-serif: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
$font-size-base: 1rem;
$line-height-base: 1.6;

// Радиусы
$border-radius: 0.5rem;
$border-radius-lg: 0.75rem;
$border-radius-sm: 0.25rem;

// Отступы
$spacer: 1rem;
$spacers: (
  0: 0,
  1: $spacer * 0.25,
  2: $spacer * 0.5,
  3: $spacer,
  4: $spacer * 1.5,
  5: $spacer * 3,
  6: $spacer * 4,
  7: $spacer * 5,
  8: $spacer * 6,
  9: $spacer * 7,
  10: $spacer * 8,
);

// Компоненты
$navbar-padding-y: 0.75rem;
$card-border-radius: $border-radius;
$btn-border-radius: $border-radius;

// 2. Импорт фреймворка
@import "bootstrap/scss/bootstrap";

// 3. Дополнительные стили
.custom-button {
  @extend .btn;
  @include button-variant($primary, white);
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(52, 152, 219, 0.3);
  }
}
```

## Кастомизация Bootstrap

### 1. Кастомизация цветов и темы

```scss
// custom-bootstrap-theme.scss
// Определение кастомных цветов
$custom-colors: (
  "brand": #3498db,
  "accent": #e74c3c,
  "neutral": #95a5a6,
  "dark-blue": #2c3e50,
  "light-blue": #3498db,
);

// Добавление кастомных цветов к теме
$theme-colors: map-merge($theme-colors, $custom-colors);

// Использование кастомных цветов
@import "bootstrap/scss/bootstrap";

// Дополнительные стили для кастомных цветов
@each $color, $value in $custom-colors {
  .bg-#{$color} {
    background-color: #{$value} !important;
  }
  
  .text-#{$color} {
    color: #{$value} !important;
  }
  
  .border-#{$color} {
    border-color: #{$value} !important;
  }
}
```

### 2. Кастомизация компонентов

```scss
// custom-components.scss
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/mixins";

// Кастомизация кнопок
.btn-brand {
  @include button-variant(map-get($theme-colors, "brand"), white);
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(52, 152, 219, 0.4);
  }
  
  &:active,
  &.active {
    transform: translateY(0);
    box-shadow: 0 2px 4px rgba(52, 152, 219, 0.3);
  }
}

// Кастомизация карточек
.card {
  transition: all 0.3s ease;
  border: none;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15);
  }
  
  &.card-highlighted {
    border-left: 4px solid map-get($theme-colors, "brand");
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
  }
}

// Кастомизация форм
.form-control {
  border-radius: 8px;
  padding: 0.75rem 1rem;
  border: 2px solid $gray-200;
  
  &:focus {
    border-color: map-get($theme-colors, "brand");
    box-shadow: 0 0 0 0.2rem rgba(map-get($theme-colors, "brand"), 0.25);
  }
}

// Кастомизация навигации
.navbar-brand {
  font-weight: 600;
  letter-spacing: 0.5px;
  
  &:hover {
    color: map-get($theme-colors, "brand") !important;
  }
}

.nav-link {
  position: relative;
  
  &::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 0;
    height: 2px;
    background-color: map-get($theme-colors, "brand");
    transition: width 0.3s ease;
  }
  
  &:hover::after {
    width: 100%;
  }
}
```

### 3. Темизация (светлая/темная)

```scss
// themes.scss
// Светлая тема (по умолчанию)
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f8f9fa;
  --text-primary: #212529;
  --text-secondary: #6c757d;
  --border-color: #dee2e6;
}

// Темная тема
[data-theme="dark"] {
  --bg-primary: #212529;
  --bg-secondary: #343a40;
  --text-primary: #f8f9fa;
  --text-secondary: #e9ecef;
  --border-color: #495057;
}

// Применение переменных к компонентам
.card-themed {
  background-color: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
}

.btn-themed {
  background-color: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
  
  &:hover {
    background-color: color-mix(in srgb, var(--bg-primary), var(--text-primary) 10%);
  }
}
```

## Кастомизация Tailwind CSS

### 1. Конфигурация кастомизации

```javascript
// tailwind.config.js
const plugin = require('tailwindcss/plugin');

module.exports = {
  content: [
    './src/**/*.{html,js,jsx,ts,tsx}',
  ],
  theme: {
    extend: {
      // Кастомные цвета
      colors: {
        'brand': {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        },
        'neutral': {
          50: '#fafafa',
          100: '#f5f5f5',
          200: '#e5e5e5',
          300: '#d4d4d4',
          400: '#a3a3a3',
          500: '#737373',
          600: '#525252',
          700: '#404040',
          800: '#262626',
          900: '#171717',
        }
      },
      
      // Кастомные шрифты
      fontFamily: {
        'sans': ['Inter', 'ui-sans-serif', 'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'Helvetica Neue', 'Arial', 'Noto Sans', 'sans-serif'],
        'display': ['Poppins', 'ui-sans-serif', 'system-ui', '-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'Helvetica Neue', 'Arial', 'Noto Sans', 'sans-serif'],
      },
      
      // Кастомные размеры
      spacing: {
        '18': '4.5rem',
        '84': '21rem',
        '96': '24rem',
        '128': '32rem',
      },
      
      // Кастомные брейкпоинты
      screens: {
        'xs': '475px',
        '3xl': '1600px',
      },
      
      // Кастомные тени
      boxShadow: {
        'xl': '0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04)',
        '2xl': '0 25px 50px -12px rgba(0, 0, 0, 0.25)',
        'neon': '0 0 5px theme(colors.blue.400), 0 0 10px theme(colors.blue.300)',
      },
      
      // Кастомные анимации
      animation: {
        'spin-slow': 'spin 3s linear infinite',
        'ping-slow': 'ping 3s cubic-bezier(0,0,0.2,1) infinite',
        'pulse-fast': 'pulse 1s cubic-bezier(0.4, 0, 0.6, 1) infinite',
      },
    },
  },
  plugins: [
    // Кастомный плагин для дополнительных утилит
    plugin(function({ addUtilities, theme, variants }) {
      const customUtilities = {
        '.text-shadow': {
          textShadow: '0 1px 3px rgba(0, 0, 0, 0.2)',
        },
        '.text-shadow-sm': {
          textShadow: '0 1px 2px rgba(0, 0, 0, 0.1)',
        },
        '.text-shadow-lg': {
          textShadow: '0 10px 15px rgba(0, 0, 0, 0.2)',
        },
        '.transform-perspective': {
          transform: 'perspective(1000px)',
        },
        '.transform-rotate-y-45': {
          transform: 'perspective(1000px) rotateY(45deg)',
        },
      };
      
      addUtilities(customUtilities, variants('transform'));
    }),
  ],
}
```

### 2. Использование @apply для кастомных компонентов

```css
/* components.css */
@layer components {
  /* Кастомные кнопки */
  .btn-brand {
    @apply bg-brand-500 hover:bg-brand-600 text-white font-medium py-2 px-4 rounded-lg transition-all duration-300 shadow-sm hover:shadow-md;
  }
  
  .btn-brand-outline {
    @apply border-2 border-brand-500 text-brand-500 hover:bg-brand-500 hover:text-white font-medium py-2 px-4 rounded-lg transition-colors duration-300;
  }
  
  .btn-brand-sm {
    @apply btn-brand py-1.5 px-3 text-sm;
  }
  
  .btn-brand-lg {
    @apply btn-brand py-3 px-6 text-lg;
  }
  
  /* Кастомные карточки */
  .card-elevated {
    @apply bg-white rounded-xl border border-gray-200 shadow-sm hover:shadow-lg transition-shadow duration-300 overflow-hidden;
  }
  
  .card-featured {
    @apply card-elevated border-l-4 border-brand-500;
  }
  
  /* Кастомные формы */
  .input-brand {
    @apply w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-brand-500 focus:border-brand-500 transition-colors;
  }
  
  .input-brand.error {
    @apply border-red-500 focus:ring-red-500 focus:border-red-500;
  }
  
  /* Кастомные навигационные элементы */
  .nav-link {
    @apply px-4 py-2 text-gray-700 hover:text-brand-500 font-medium rounded-lg transition-colors relative;
  }
  
  .nav-link.active {
    @apply text-brand-500 bg-brand-50;
  }
  
  .nav-link::after {
    @apply absolute bottom-0 left-1/2 w-0 h-0.5 bg-brand-500 transition-all duration-300 content-[''];
  }
  
  .nav-link:hover::after {
    @apply w-3/4 left-1/8;
  }
  
  /* Кастомные тултипы */
  .tooltip {
    @apply absolute z-50 px-3 py-2 text-sm font-medium text-white bg-gray-900 rounded-lg shadow-sm opacity-0 invisible transition-opacity duration-300;
  }
  
  .tooltip.show {
    @apply opacity-100 visible;
  }
}
```

### 3. Темизация с Tailwind

```html
<!DOCTYPE html>
<html lang="en" class="light" data-theme="light">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Темизация с Tailwind</title>
  <script>
    // JavaScript для переключения темы
    function toggleTheme() {
      const html = document.documentElement;
      const isDark = html.classList.contains('dark');
      
      if (isDark) {
        html.classList.remove('dark');
        html.setAttribute('data-theme', 'light');
        localStorage.theme = 'light';
      } else {
        html.classList.add('dark');
        html.setAttribute('data-theme', 'dark');
        localStorage.theme = 'dark';
      }
    }
    
    // Установка темы при загрузке
    if (localStorage.theme === 'dark' || 
        (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
      document.documentElement.classList.add('dark');
      document.documentElement.setAttribute('data-theme', 'dark');
    } else {
      document.documentElement.classList.remove('dark');
      document.documentElement.setAttribute('data-theme', 'light');
    }
  </script>
  <style type="text/tailwindcss">
    @layer base {
      :root {
        --color-bg-primary: 255 255 255;
        --color-bg-secondary: 249 250 251;
        --color-text-primary: 17 24 39;
        --color-text-secondary: 107 114 128;
        --color-border: 229 231 235;
      }
      
      .dark {
        --color-bg-primary: 17 24 39;
        --color-bg-secondary: 31 41 55;
        --color-text-primary: 249 250 251;
        --color-text-secondary: 156 163 175;
        --color-border: 55 65 81;
      }
    }
    
    @layer components {
      .card-themed {
        @apply bg-[rgb(var(--color-bg-secondary))] text-[rgb(var(--color-text-primary))] border border-[rgb(var(--color-border))];
      }
    }
  </style>
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
</head>
<body class="bg-[rgb(var(--color-bg-primary))] text-[rgb(var(--color-text-primary))] transition-colors duration-300">
  <div class="container mx-auto p-4">
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-2xl font-bold">Темизация интерфейса</h1>
      <button 
        onclick="toggleTheme()" 
        class="p-2 rounded-full bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-200 hover:bg-gray-300 dark:hover:bg-gray-600 transition-colors"
      >
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"></path>
        </svg>
      </button>
    </div>
    
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
      <div class="card-themed p-6 rounded-xl">
        <h2 class="text-xl font-semibold mb-4">Светлая тема</h2>
        <p class="mb-4">Это пример карточки с темизацией. Цвета автоматически изменяются при переключении темы.</p>
        <button class="btn-brand">Пример кнопки</button>
      </div>
      
      <div class="card-themed p-6 rounded-xl">
        <h2 class="text-xl font-semibold mb-4">Темная тема</h2>
        <p class="mb-4">Та же карточка, но с темной темой. Обратите внимание на плавный переход между темами.</p>
        <button class="btn-brand">Пример кнопки</button>
      </div>
    </div>
  </div>
</body>
</html>
```

## Кастомизация других фреймворков

### Bulma

```scss
// custom-bulma.scss
// Переопределение переменных Bulma
$primary: #3498db;
$info: #209cee;
$success: #2ecc71;
$warning: #f39c12;
$danger: #e74c3c;

// Типографика
$family-primary: "Inter", "Segoe UI", sans-serif;
$size-base: 1rem;

// Радиусы
$radius: 0.5rem;
$radius-small: 0.25rem;
$radius-large: 0.75rem;

// Отступы
$gap: 0.75rem;

// Импорт Bulma
@import "bulma/bulma";

// Дополнительные стили
.button.is-brand {
  background-color: $primary;
  border-color: transparent;
  color: white;
  
  &:hover,
  &.is-hovered {
    background-color: darken($primary, 5%);
    border-color: transparent;
    color: white;
  }
  
  &:focus,
  &.is-focused {
    border-color: transparent;
    box-shadow: 0 0 0.5em rgba($primary, 0.25);
  }
}

.card {
  border-radius: $radius;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15);
  }
}
```

### Foundation

```scss
// custom-foundation.scss
// Импорт функций Foundation
@import 'foundation-sites/scss/util/util';

// Переопределение переменных
$primary-color: #3498db;
$secondary-color: #2c3e50;
$success-color: #2ecc71;
$warning-color: #f39c12;
$alert-color: #e74c3c;

// Типографика
$body-font-family: 'Inter', 'Helvetica Neue', Helvetica, Roboto, Arial, sans-serif;
$body-line-height: 1.6;
$global-radius: 0.5rem;

// Грид
$grid-container: rem-calc(1170);
$grid-column-gutter: (
  small: 20px,
  medium: 30px
);

// Импорт Foundation компонентов
@import 'foundation-scss/foundation';

// Кастомизация компонентов
.button {
  border-radius: $global-radius;
  transition: all 0.3s ease;
  
  &:hover {
    transform: translateY(-2px);
  }
}

.card {
  border-radius: $global-radius;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  
  .card-section {
    padding: 1.5rem;
  }
}
```

## Лучшие практики кастомизации

### 1. Модульная архитектура

```
project/
├── scss/
│   ├── abstracts/
│   │   ├── _variables.scss    # Кастомные переменные
│   │   ├── _mixins.scss       # Кастомные миксины
│   │   └── _functions.scss    # Кастомные функции
│   ├── base/
│   │   ├── _reset.scss        # Сброс стилей
│   │   ├── _typography.scss   # Базовая типографика
│   │   └── _base.scss         # Базовые стили
│   ├── components/
│   │   ├── _buttons.scss      # Кастомные кнопки
│   │   ├── _cards.scss        # Кастомные карточки
│   │   ├── _forms.scss        # Кастомные формы
│   │   └── _navigation.scss   # Кастомная навигация
│   ├── layout/
│   │   ├── _header.scss       # Шапка
│   │   ├── _footer.scss       # Футер
│   │   └── _grid.scss         # Сетка
│   ├── pages/
│   │   └── _home.scss         # Специфичные стили страниц
│   ├── themes/
│   │   ├── _light.scss        # Светлая тема
│   │   └── _dark.scss         # Темная тема
│   └── main.scss              # Главный файл
└── tailwind.config.js         # Конфигурация Tailwind (если используется)
```

### 2. Создание дизайн-системы

```scss
// design-system.scss
// Система отступов
$spacing-scale: (
  'xs': 0.25rem,  // 4px
  'sm': 0.5rem,   // 8px
  'md': 1rem,     // 16px
  'lg': 1.5rem,   // 24px
  'xl': 2rem,     // 32px
  '2xl': 3rem,    // 48px
  '3xl': 4rem,    // 64px
);

// Система цветов
$color-palette: (
  'primary': (
    '50': #eff6ff,
    '100': #dbeafe,
    '200': #bfdbfe,
    '300': #93c5fd,
    '400': #60a5fa,
    '500': #3b82f6,
    '600': #2563eb,
    '700': #1d4ed8,
    '800': #1e40af,
    '900': #1e3a8a,
  ),
  'secondary': (
    '50': #f8fafc,
    '100': #f1f5f9,
    '200': #e2e8f0,
    '300': #cbd5e1,
    '400': #94a3b8,
    '500': #64748b,
    '600': #475569,
    '700': #334155,
    '800': #1e293b,
    '900': #0f172a,
  )
);

// Система теней
$shadow-scale: (
  'sm': 0 1px 2px 0 rgba(0, 0, 0, 0.05),
  'base': 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1),
  'md': 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1),
  'lg': 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1),
  'xl': 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -6px rgba(0, 0, 0, 0.1),
);

// Миксин для применения теней
@mixin shadow($level) {
  box-shadow: map-get($shadow-scale, $level);
}

// Миксин для применения отступов
@mixin spacing($size, $direction: 'all') {
  @if $direction == 'all' {
    margin: map-get($spacing-scale, $size);
  } @else if $direction == 'x' {
    margin-left: map-get($spacing-scale, $size);
    margin-right: map-get($spacing-scale, $size);
  } @else if $direction == 'y' {
    margin-top: map-get($spacing-scale, $size);
    margin-bottom: map-get($spacing-scale, $size);
  }
}
```

### 3. Работа с брендированными компонентами

```html
<!-- Брендированные компоненты -->
<div class="brand-card">
  <div class="brand-card__header">
    <h3 class="brand-card__title">Заголовок карточки</h3>
    <span class="brand-card__badge">Новинка</span>
  </div>
  <div class="brand-card__body">
    <p class="brand-card__text">Содержимое карточки с брендированным стилем</p>
  </div>
  <div class="brand-card__footer">
    <button class="brand-btn brand-btn--primary">Действие</button>
    <button class="brand-btn brand-btn--secondary">Отмена</button>
  </div>
</div>

<style>
@layer components {
  .brand-card {
    @apply bg-white rounded-xl border border-gray-200 shadow-sm overflow-hidden transition-all duration-300;
  }
  
  .brand-card:hover {
    @apply shadow-md transform -translate-y-1;
  }
  
  .brand-card__header {
    @apply px-6 py-4 border-b border-gray-200 bg-gray-50 flex justify-between items-center;
  }
  
  .brand-card__title {
    @apply text-lg font-semibold text-gray-900 m-0;
  }
  
  .brand-card__badge {
    @apply inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-blue-100 text-blue-800;
  }
  
  .brand-card__body {
    @apply p-6;
  }
  
  .brand-card__text {
    @apply text-gray-600;
  }
  
  .brand-card__footer {
    @apply px-6 py-4 bg-gray-50 border-t border-gray-200 flex justify-end space-x-3;
  }
  
  .brand-btn {
    @apply inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2;
  }
  
  .brand-btn--primary {
    @apply bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500;
  }
  
  .brand-btn--secondary {
    @apply bg-white text-gray-700 hover:bg-gray-50 border-gray-300 focus:ring-gray-500;
  }
}
</style>
```

## Заключение

Кастомизация CSS-фреймворков позволяет создавать уникальные и брендированные интерфейсы, сохраняя при этом преимущества готовых решений. Ключевые аспекты успешной кастомизации:

1. **Планирование**: Создание дизайн-системы до начала кастомизации
2. **Архитектура**: Организация кода в модульную структуру
3. **Переменные**: Использование CSS-переменных или Sass-переменных для консистентности
4. **Компоненты**: Создание переиспользуемых кастомных компонентов
5. **Темизация**: Поддержка светлой и темной тем
6. **Производительность**: Оптимизация размера итогового CSS

Правильная кастомизация позволяет сохранить преимущества фреймворков (скорость разработки, адаптивность, доступность) при создании уникального пользовательского опыта.

#programming #css #frameworks #customization #web-development #frontend #design-system