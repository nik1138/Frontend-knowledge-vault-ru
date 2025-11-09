# Производительность CSS-фреймворков

Производительность CSS-фреймворков критически важна для скорости загрузки веб-страниц и общего пользовательского опыта. В этом руководстве мы рассмотрим стратегии оптимизации производительности различных CSS-фреймворков.

## Основы производительности CSS

### Влияние CSS на производительность

CSS влияет на производительность веб-страниц следующими способами:

1. **Размер файла**: Чем больше CSS-файл, тем дольше он загружается
2. **Парсинг**: Браузер должен распарсить весь CSS перед отображением страницы
3. **Рендеринг**: Сложные CSS-правила могут замедлить процесс рендеринга
4. **Память**: Большое количество CSS-правил потребляет больше памяти

### Метрики производительности

- **First Contentful Paint (FCP)**: Время до первого отображения контента
- **Largest Contentful Paint (LCP)**: Время до отображения самого большого элемента
- **Cumulative Layout Shift (CLS)**: Стабильность макета
- **Time to Interactive (TTI)**: Время до интерактивности страницы

## Оптимизация Bootstrap

### 1. Импорт только нужных компонентов

Вместо импорта всего Bootstrap, импортируйте только нужные компоненты:

```scss
// bootstrap-custom.scss
// 1. Функции и переменные
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/mixins";

// 2. Утилиты
@import "bootstrap/scss/utilities";

// 3. Сетка
@import "bootstrap/scss/containers";
@import "bootstrap/scss/grid";

// 4. Компоненты (только нужные)
@import "bootstrap/scss/buttons";
@import "bootstrap/scss/card";
@import "bootstrap/scss/nav";
@import "bootstrap/scss/navbar";
@import "bootstrap/scss/forms";
@import "bootstrap/scss/modal";

// 5. API утилит (обязательно в конце)
@import "bootstrap/scss/utilities/api";
```

### 2. Оптимизация через конфигурацию

```scss
// custom-bootstrap.scss
// Отключение неиспользуемых компонентов
$enable-caret: false;
$enable-rounded: true;
$enable-shadows: false;
$enable-gradients: false;
$enable-transitions: true;
$enable-reduced-motion: true;
$enable-grid-classes: true;
$enable-container-classes: true;
$enable-cssgrid: false;
$enable-negative-margins: false;
$enable-deprecation-messages: false;

// Кастомизация для уменьшения размера
$border-radius: 0.25rem;
$border-radius-lg: 0.5rem;
$border-radius-sm: 0.25rem;

// Импорт только нужных компонентов
@import "bootstrap/scss/bootstrap";
```

### 3. Удаление неиспользуемых CSS (PurgeCSS)

```javascript
// purgecss.config.js
module.exports = {
  content: [
    './src/**/*.html',
    './src/**/*.js',
    './src/**/*.jsx',
    './src/**/*.ts',
    './src/**/*.tsx',
    './public/**/*.html'
  ],
  css: [
    './dist/css/bootstrap-custom.css'
  ],
  // Специфичные для Bootstrap селекторы, которые не должны быть удалены
  safelist: [
    // Классы для анимаций
    /^show$/,
    /^fade$/,
    /^collapsing$/,
    // Классы для состояний
    /^btn-/,
    /^alert-/,
    /^form-/,
    // Классы для адаптивности
    /^col-/,
    /^row/,
    /^d-/,
    // Классы для модальных окон
    /^modal/,
    // Классы для навигации
    /^nav/,
    /^navbar/,
    // Классы для валидации форм
    /^is-/,
    /^was-/
  ],
  variables: true
};
```

### 4. Lazy loading компонентов

```javascript
// lazy-bootstrap.js
// Загрузка компонентов по требованию
class LazyBootstrap {
  static async loadModal() {
    const { Modal } = await import('bootstrap/dist/js/bootstrap.bundle.min.js');
    return Modal;
  }
  
  static async loadTooltip() {
    const { Tooltip } = await import('bootstrap/dist/js/bootstrap.bundle.min.js');
    return Tooltip;
  }
  
  static async loadPopover() {
    const { Popover } = await import('bootstrap/dist/js/bootstrap.bundle.min.js');
    return Popover;
  }
}

// Использование
document.addEventListener('DOMContentLoaded', function() {
  // Загрузка модального окна только при необходимости
  document.querySelectorAll('[data-bs-toggle="modal"]').forEach(button => {
    button.addEventListener('click', async function() {
      const Modal = await LazyBootstrap.loadModal();
      const target = document.querySelector(this.getAttribute('data-bs-target'));
      new Modal(target).show();
    });
  });
});
```

## Оптимизация Tailwind CSS

### 1. Конфигурация для продакшена

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{html,js,jsx,ts,tsx}',
    './public/index.html',
  ],
  theme: {
    extend: {
      // Только нужные расширения
    },
  },
  plugins: [
    // Только нужные плагины
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
  // Оптимизация для продакшена
  corePlugins: {
    // Отключение неиспользуемых плагинов
    container: false,
  }
}
```

### 2. Использование JIT режима

```javascript
// tailwind.config.js
module.exports = {
  mode: 'jit', // Включает JIT компиляцию
  content: [
    './src/**/*.{html,js,jsx,ts,tsx}',
  ],
  // ... остальная конфигурация
}
```

### 3. Safelist для динамических классов

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './src/**/*.{html,js,jsx,ts,tsx}',
  ],
  safelist: [
    // Динамические цвета
    {
      pattern: /^(bg|text|border)-(red|blue|green|yellow|purple|pink)-(100|200|300|400|500|600|700|800|900)$/,
    },
    // Адаптивные классы
    {
      pattern: /^(sm|md|lg|xl|2xl):/,
    },
    // Классы для состояний
    {
      pattern: /^(hover|focus|active|group-hover|focus-within):/,
    },
    // Классы для анимаций
    /^animate-/,
    // Классы для специфичных компонентов
    {
      pattern: /btn-(primary|secondary|success|danger)/,
    },
  ],
  // ... остальная конфигурация
}
```

### 4. Оптимизация HTML с динамическими классами

```jsx
// Вместо этого:
function Button({ variant, size, isLoading }) {
  return (
    <button className={`btn ${variant ? 'btn-' + variant : 'btn-primary'} ${size ? 'btn-' + size : ''} ${isLoading ? 'loading' : ''}`}>
      {children}
    </button>
  );
}

// Лучше так:
function Button({ variant = 'primary', size, isLoading, children }) {
  const baseClasses = 'btn inline-flex items-center justify-center rounded-md font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500',
    success: 'bg-green-600 text-white hover:bg-green-700 focus:ring-green-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg',
  };
  
  const loadingClasses = isLoading ? 'opacity-75 cursor-not-allowed' : '';
  
  const className = `${baseClasses} ${variantClasses[variant]} ${size ? sizeClasses[size] : ''} ${loadingClasses}`;
  
  return (
    <button className={className} disabled={isLoading}>
      {isLoading && (
        <svg className="animate-spin -ml-1 mr-2 h-4 w-4 text-current" fill="none" viewBox="0 0 24 24">
          <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4"></circle>
          <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      )}
      {children}
    </button>
  );
}
```

## Оптимизация других фреймворков

### Bulma

```scss
// custom-bulma.scss
// Отключение неиспользуемых компонентов
$bulma: (
  'form': (
    'file': false,
    'input': true,
    'textarea': true,
  ),
  'component': (
    'breadcrumb': false,
    'card': true,
    'dropdown': false,
    'level': true,
  ),
  'grid': (
    'column-gap': 1rem,
  ),
);

// Импорт только нужных компонентов
@import "bulma/sass/utilities/_all";
@import "bulma/sass/base/_all";
@import "bulma/sass/elements/button";
@import "bulma/sass/elements/container";
@import "bulma/sass/elements/title";
@import "bulma/sass/grid/columns";
@import "bulma/sass/components/card";
@import "bulma/sass/components/navbar";
```

### Foundation

```scss
// custom-foundation.scss
// Отключение неиспользуемых компонентов
$global-flexbox: true;
$print-transparent-backgrounds: false;

// Кастомизация для уменьшения размера
$global-radius: 0.25rem;
$button-padding: 0.75rem 1rem;
$button-margin: 0 0 1rem 0;

// Импорт только нужных компонентов
@import 'foundation-sites/scss/util/util';
@import 'foundation-sites/scss/global';
@import 'foundation-sites/scss/grid/grid';
@import 'foundation-sites/scss/typography/typography';
@import 'foundation-sites/scss/elements/button';
@import 'foundation-sites/scss/elements/card';
@import 'foundation-sites/scss/components/accordion';
```

## Общие стратегии оптимизации

### 1. Удаление неиспользуемых стилей

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
  ],
  optimization: {
    minimizer: [
      new CssMinimizerPlugin({
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true },
            },
          ],
        },
      }),
    ],
  },
};
```

### 2. Критический CSS

```javascript
// critical-css.js
// Извлечение критического CSS для быстрой загрузки
const penthouse = require('penthouse');

async function generateCriticalCSS(url, width, height) {
  const criticalCss = await penthouse({
    url,
    width,
    height,
    strict: false,
    timeout: 30000,
    renderWaitTime: 2000,
    blockJSRequests: false
  });
  
  return criticalCss;
}

// Использование в HTML
/*
<head>
  <style>
    /* Встроенный критический CSS */
    .header{display:flex}.btn{padding:1rem}
  </style>
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
*/
```

### 3.Ленивая загрузка CSS

```html
<!-- Ленивая загрузка CSS -->
<link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="non-critical.css"></noscript>

<script>
// Альтернативный метод через JavaScript
function loadCSS(href) {
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = href;
  document.head.appendChild(link);
}

// Загрузка после загрузки страницы
window.addEventListener('load', function() {
  loadCSS('non-critical.css');
});
</script>
```

### 4. Сжатие и минификация

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer'),
    ...(process.env.NODE_ENV === 'production' ? [
      require('cssnano')({
        preset: 'default',
      })
    ] : [])
  ]
}
```

## Профилирование и аудит производительности

### 1. Использование Chrome DevTools

```javascript
// Аудит CSS-покрытия
// 1. Откройте DevTools (F12)
// 2. Перейдите в "Coverage" вкладку
// 3. Нажмите "Reload" для анализа
// 4. Просмотрите процент использования CSS

// Аудит производительности
// 1. Перейдите в "Performance" вкладку
// 2. Нажмите "Record"
// 3. Взаимодействуйте со страницей
// 4. Остановите запись и проанализируйте результаты
```

### 2. Автоматические инструменты

```json
// package.json
{
  "scripts": {
    "audit:css": "purgecss --config purgecss.config.js",
    "audit:performance": "lighthouse http://localhost:3000 --output html --output-path ./reports/lighthouse.html",
    "analyze": "webpack-bundle-analyzer dist/stats.json"
  },
  "devDependencies": {
    "purgecss": "^5.0.0",
    "lighthouse": "^10.0.0",
    "webpack-bundle-analyzer": "^4.7.0"
  }
}
```

### 3. Мониторинг производительности

```javascript
// performance-monitor.js
class PerformanceMonitor {
  static measureFCP() {
    new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        console.log('FCP:', entry.startTime);
        // Отправка данных в аналитику
        this.sendPerformanceData('FCP', entry.startTime);
      }
    }).observe({ entryTypes: ['paint'] });
  }
  
  static measureLCP() {
    new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        console.log('LCP:', entry.startTime);
        this.sendPerformanceData('LCP', entry.startTime);
      }
    }).observe({ entryTypes: ['largest-contentful-paint'] });
  }
  
  static measureCLS() {
    let clsValue = 0;
    new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        if (!entry.hadRecentInput) {
          clsValue += entry.value;
          console.log('Current CLS:', clsValue);
          this.sendPerformanceData('CLS', clsValue);
        }
      }
    }).observe({ entryTypes: ['layout-shift'] });
  }
  
  static sendPerformanceData(metric, value) {
    // Отправка данных в аналитическую систему
    if ('sendBeacon' in navigator) {
      navigator.sendBeacon('/api/performance', JSON.stringify({
        metric,
        value,
        url: window.location.href,
        timestamp: Date.now()
      }));
    }
  }
}

// Инициализация мониторинга
PerformanceMonitor.measureFCP();
PerformanceMonitor.measureLCP();
PerformanceMonitor.measureCLS();
```

## Лучшие практики

### 1. Архитектура файлов

```
project/
├── css/
│   ├── base/
│   │   ├── reset.css          # Сброс стилей
│   │   └── typography.css     # Базовая типографика
│   ├── components/
│   │   ├── buttons.css        # Кнопки
│   │   ├── cards.css          # Карточки
│   │   └── forms.css          # Формы
│   ├── layout/
│   │   ├── header.css         # Шапка
│   │   ├── footer.css         # Футер
│   │   └── grid.css           # Сетка
│   ├── themes/
│   │   └── dark.css          # Темная тема
│   ├── vendor/               # Внешние фреймворки
│   │   └── bootstrap.min.css
│   ├── critical.css          # Критические стили
│   └── main.css              # Основной файл
```

### 2. Использование CSS-модулей

```css
/* Button.module.css */
.button {
  @apply px-4 py-2 rounded font-medium transition-colors;
}

.primary {
  @apply bg-blue-600 text-white hover:bg-blue-700;
}

.secondary {
  @apply bg-gray-200 text-gray-800 hover:bg-gray-300;
}

.large {
  @apply px-6 py-3 text-lg;
}
```

```jsx
// Button.jsx
import styles from './Button.module.css';

function Button({ variant = 'primary', size, children, ...props }) {
  const buttonClasses = [
    styles.button,
    styles[variant],
    size === 'large' && styles.large
  ].filter(Boolean).join(' ');

  return (
    <button className={buttonClasses} {...props}>
      {children}
    </button>
  );
}
```

### 3. Оптимизация анимаций

```css
/* Оптимизированные анимации */
.optimized-animation {
  /* Используйте transform и opacity для плавных анимаций */
  transform: translateZ(0); /* Включает аппаратное ускорение */
  will-change: transform; /* Подсказка браузеру о предстоящих изменениях */
  backface-visibility: hidden; /* Оптимизация для 3D-трансформаций */
}

.slide-in {
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    transform: translateX(100%) translateY(0) translateZ(0);
    opacity: 0;
  }
  to {
    transform: translateX(0) translateY(0) translateZ(0);
    opacity: 1;
  }
}

/* Избегайте анимации layout-свойств */
.bad-animation {
  /* Плохо: анимация width, height, margin, padding */
  animation: badAnimation 1s ease-in-out;
}

.good-animation {
  /* Хорошо: анимация transform, opacity */
  animation: goodAnimation 1s ease-in-out;
}
```

## Заключение

Производительность CSS-фреймворков требует комплексного подхода, включающего:

1. **Оптимизацию размера**: Использование только нужных компонентов, PurgeCSS
2. **Критический CSS**: Встраивание критических стилей
3. **Ленивая загрузка**: Отложенная загрузка неиспользуемых стилей
4. **Минификация**: Сжатие и оптимизация CSS
5. **Мониторинг**: Постоянный аудит и профилирование производительности

Следуя этим практикам, можно значительно улучшить производительность веб-приложений при использовании CSS-фреймворков, обеспечивая быструю загрузку и плавное взаимодействие с пользователем.

#programming #css #frameworks #performance #web-development #frontend #optimization #bundle-size