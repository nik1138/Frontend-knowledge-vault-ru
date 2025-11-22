---
aliases: ["CSS @layer", "CSS Layer", "CSS Слои", "CSS Cascade Layers"]
tags: [css, cascade, layer, modern-css, web-development, frontend]
---

# CSS @layer

## Введение

CSS @layer — это современная функция CSS, которая позволяет явно контролировать каскад (cascade) стилей, создавая именованные слои с определенным приоритетом. Это позволяет разработчикам более предсказуемо управлять тем, какие стили применяются, когда есть конфликты между различными источниками CSS.

## Основные концепции

### Что такое CSS @layer

CSS @layer позволяет группировать CSS-правила в именованные слои, которые затем можно упорядочить по приоритету. Более низкие слои имеют меньший приоритет, чем более высокие слои. Это дает разработчикам контроль над каскадом без необходимости использовать `!important` или усложнять селекторы.

### Зачем использовать @layer

- **Предсказуемость каскада**: Четкое определение приоритета стилей
- **Модульность**: Организация CSS в логические блоки
- **Контроль над сторонними библиотеками**: Возможность переопределять стили библиотек
- **Уменьшение использования !important**: Более чистый подход к приоритезации

## Основы CSS @layer

### Создание слоев

```css
/* Создание слоев с определенными именами */
@layer theme {
  /* Стили для темы */
  body {
    color: var(--text-color, #333);
    background-color: var(--bg-color, #fff);
  }
}

@layer base {
  /* Базовые стили */
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
}

@layer components {
  /* Стили компонентов */
  .button {
    padding: 0.5rem 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
}

@layer utilities {
  /* Утилитарные классы */
  .text-center {
    text-align: center;
  }
  
  .bg-primary {
    background-color: #007bff;
  }
}
```

### Определение порядка слоев

```css
/* Определение порядка слоев (должно быть в начале CSS) */
@layer base, theme, components, utilities;

/* Слои будут применяться в этом порядке:
   1. base (наименьший приоритет)
   2. theme
   3. components
   4. utilities (наивысший приоритет)
*/
```

### Вложенность слоев

```css
/* Создание вложенных слоев */
@layer framework {
  @layer reset {
    /* Сброс стилей */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
  }
  
  @layer base {
    /* Базовые стили фреймворка */
    body {
      font-family: Arial, sans-serif;
    }
  }
}

/* Использование вложенных слоев */
@layer framework.reset, framework.base, custom;
```

## Практические примеры

### 1. Организация CSS проекта по слоям

```css
/* Определение порядка слоев в начале файла */
@layer reset, base, theme, components, utilities;

/* Сброс стилей */
@layer reset {
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  body {
    line-height: 1.5;
  }
}

/* Базовые стили */
@layer base {
  body {
    font-family: system-ui, sans-serif;
    color: #333;
    background-color: #fff;
  }
  
  h1, h2, h3, h4, h5, h6 {
    margin-bottom: 0.5em;
    font-weight: 600;
  }
  
  a {
    color: #007bff;
    text-decoration: none;
  }
}

/* Тема */
@layer theme {
  :root {
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --text-color: #212529;
    --bg-color: #ffffff;
  }
  
  /* Темная тема */
  [data-theme="dark"] {
    --text-color: #e9ecef;
    --bg-color: #212529;
  }
}

/* Компоненты */
@layer components {
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
    border: 1px solid transparent;
    border-radius: 0.25rem;
    transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out;
  }
  
  .btn-primary {
    color: #fff;
    background-color: var(--primary-color);
    border-color: var(--primary-color);
  }
  
  .card {
    position: relative;
    display: flex;
    flex-direction: column;
    min-width: 0;
    word-wrap: break-word;
    background-color: var(--bg-color);
    background-clip: border-box;
    border: 1px solid rgba(0, 0, 0, 0.125);
    border-radius: 0.25rem;
  }
}

/* Утилиты */
@layer utilities {
  .d-flex { display: flex; }
  .d-none { display: none; }
  
  .text-primary { color: var(--primary-color); }
  .text-secondary { color: var(--secondary-color); }
  
  .bg-primary { background-color: var(--primary-color); }
  .bg-light { background-color: #f8f9fa; }
  
  .m-0 { margin: 0; }
  .m-1 { margin: 0.25rem; }
  .m-2 { margin: 0.5rem; }
  
  .p-0 { padding: 0; }
  .p-1 { padding: 0.25rem; }
  .p-2 { padding: 0.5rem; }
}
```

### 2. Интеграция сторонних библиотек

```css
/* Определение порядка слоев */
@layer external, base, components, overrides;

/* Стили из внешней библиотеки */
@layer external {
  /* Стили из библиотеки, например Tailwind */
  .btn {
    /* Стили библиотеки */
  }
}

/* Переопределение стилей библиотеки */
@layer overrides {
  /* Наши собственные стили, которые переопределяют библиотеку */
  .btn {
    border-radius: 8px; /* Переопределит стиль из библиотеки */
  }
}
```

### 3. Темизация с использованием слоев

```css
@layer defaults, theme, overrides;

@layer defaults {
  :root {
    --primary-hue: 210;
    --primary-saturation: 100%;
    --primary-lightness: 50%;
    --border-radius: 4px;
    --transition-speed: 0.2s;
  }
}

@layer theme {
  :root {
    --primary-color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
    --primary-dark: hsl(var(--primary-hue), var(--primary-saturation), calc(var(--primary-lightness) - 20%));
    --primary-light: hsl(var(--primary-hue), var(--primary-saturation), calc(var(--primary-lightness) + 20%));
  }
  
  /* Тема для светлого режима */
  .light-theme {
    --text-color: #333;
    --bg-color: #fff;
  }
  
  /* Тема для темного режима */
  .dark-theme {
    --text-color: #f0f0f0;
    --bg-color: #1a1a1a;
  }
}

@layer overrides {
  /* Переопределения для конкретных компонентов */
  .special-button {
    --primary-color: #ff6b6b;
  }
}
```

## Продвинутые техники

### Условные слои

```css
/* Слои с медиа-условиями */
@layer responsive {
  @media (max-width: 768px) {
    .container {
      padding: 0 1rem;
    }
    
    .nav-menu {
      flex-direction: column;
    }
  }
  
  @media (min-width: 769px) {
    .nav-menu {
      flex-direction: row;
    }
  }
}
```

### Слои с CSS-переменными

```css
@layer tokens {
  :root {
    /* Пространство имен для токенов */
    --color-primary-50: #e7f4ff;
    --color-primary-100: #d1e9ff;
    --color-primary-200: #a5d4fd;
    --color-primary-300: #74baf9;
    --color-primary-400: #4098d7;
    --color-primary-500: #1e7bc9;
    --color-primary-600: #1363af;
    --color-primary-700: #0f4d8d;
    --color-primary-800: #0f3f6f;
    --color-primary-900: #103555;
  }
}

@layer components {
  .button {
    --button-bg: var(--color-primary-500);
    --button-text: white;
    --button-border: var(--color-primary-600);
    
    background-color: var(--button-bg);
    color: var(--button-text);
    border: 1px solid var(--button-border);
  }
}
```

### Динамические слои

```css
/* Хотя слои не могут быть динамически созданы с помощью JavaScript,
   можно использовать CSS-переменные для изменения стилей в слоях */

@layer dynamic {
  .dynamic-element {
    background-color: var(--dynamic-bg, #ccc);
    color: var(--dynamic-text, #000);
    border-color: var(--dynamic-border, #999);
    transition: all var(--dynamic-transition, 0.3s);
  }
}

/* Эти переменные могут быть изменены с помощью JavaScript */
/* document.documentElement.style.setProperty('--dynamic-bg', '#ff0000'); */
```

## Совместимость и резервные варианты

### Проверка поддержки

```css
/* Проверка поддержки @layer */
@supports (animation: name) {
  /* Все браузеры, которые поддерживают CSS анимации, 
     также поддерживают @layer */
  
  @layer base, theme, components, utilities;
  
  @layer base {
    /* Базовые стили */
  }
  
  @layer theme {
    /* Тема */
  }
}

/* Для браузеров без поддержки @layer */
@supports not (animation: name) {
  /* Альтернативная стратегия организации CSS */
  /* Использование комментариев и соглашений об именовании */
}
```

### Постепенное внедрение

```css
/* Пример постепенного внедрения @layer в существующий проект */

/* Сначала определяем слои */
@layer base, components, utilities;

/* Затем постепенно перемещаем существующие стили в соответствующие слои */
@layer base {
  /* Перемещаем базовые стили */
  body {
    font-family: Arial, sans-serif;
  }
}

@layer components {
  /* Перемещаем стили компонентов */
  .button {
    /* ... */
  }
}

/* Стили вне слоев будут иметь наивысший приоритет (как и раньше) */
.legacy-style {
  /* Эти стили будут применяться поверх слоев */
}
```

## Производительность и оптимизация

### Оптимизация использования слоев

```css
/* Эффективное использование слоев */
@layer reset, base, theme, components, utilities;

/* Избегайте чрезмерного дробления на слои */
/* Лучше иметь несколько хорошо организованных слоев, чем много мелких */

@layer base {
  /* Объединяем связанные базовые стили */
  *, *::before, *::after {
    box-sizing: border-box;
  }
  
  body {
    font-family: system-ui, sans-serif;
    line-height: 1.5;
  }
}

/* Оптимизация селекторов внутри слоев */
@layer components {
  /* Используем эффективные селекторы */
  .btn { /* Хорошо */ }
  
  /* Избегаем сложных селекторов */
  /* div > span + .btn:hover { - Слишком сложный */ }
}
```

### Слои и CSS-архитектура

```css
/* Интеграция @layer с методологиями CSS */

@layer reset, base, layout, components, utilities;

/* Layout слой для структурных классов */
@layer layout {
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 1rem;
  }
  
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
  }
}

/* Components слой для конкретных компонентов */
@layer components {
  .card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
  }
}
```

## Примеры из промышленной разработки

### Слойная архитектура для крупного приложения

```css
/* Определение архитектуры слоев */
@layer 
  /* Сброс и нормализация */
  reset,
  
  /* Базовые стили */
  base,
  
  /* Стили темы */
  theme,
  
  /* Стили макета */
  layout,
  
  /* Атомарные компоненты */
  atoms,
  
  /* Молекулярные компоненты */
  molecules,
  
  /* Организмы (комбинации компонентов) */
  organisms,
  
  /* Страницы */
  pages,
  
  /* Утилиты */
  utilities,
  
  /* Переопределения */
  overrides;

/* Сброс */
@layer reset {
  /* Сброс стилей браузера */
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
}

/* Базовые стили */
@layer base {
  body {
    font-family: 'Inter', system-ui, sans-serif;
    color: var(--color-text-primary);
    background-color: var(--color-bg-primary);
  }
}

/* Тема */
@layer theme {
  :root {
    /* Цветовая палитра */
    --color-primary-50: #eff6ff;
    --color-primary-100: #dbeafe;
    --color-primary-500: #3b82f6;
    --color-primary-900: #1e3a8a;
    
    /* Цвета текста */
    --color-text-primary: #111827;
    --color-text-secondary: #6b7280;
    
    /* Цвета фона */
    --color-bg-primary: #ffffff;
    --color-bg-secondary: #f9fafb;
    
    /* Пространства */
    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;
  }
  
  [data-theme="dark"] {
    --color-text-primary: #f9fafb;
    --color-text-secondary: #d1d5db;
    --color-bg-primary: #111827;
    --color-bg-secondary: #1f2937;
  }
}

/* Компоненты */
@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: var(--space-sm) var(--space-md);
    font-family: inherit;
    font-size: 0.875rem;
    font-weight: 500;
    line-height: 1.25;
    color: var(--color-text-primary);
    background-color: var(--color-primary-500);
    border: 1px solid var(--color-primary-500);
    border-radius: 0.375rem;
    cursor: pointer;
    transition: all 0.15s ease-in-out;
  }
  
  .button:hover {
    background-color: var(--color-primary-600);
    border-color: var(--color-primary-600);
  }
}

/* Утилиты */
@layer utilities {
  .text-center { text-align: center; }
  .text-left { text-align: left; }
  .text-right { text-align: right; }
  
  .bg-primary { background-color: var(--color-primary-500); }
  .text-white { color: white; }
  
  .flex { display: flex; }
  .grid { display: grid; }
  .hidden { display: none; }
}
```

### Интеграция с CSS-фреймворками

```css
/* Пример интеграции с Tailwind CSS */
@layer 
  /* Стили Tailwind */
  tailwind-base,
  tailwind-components,
  tailwind-utilities,
  
  /* Собственные слои */
  base,
  theme,
  components,
  custom-utilities;

/* Импорт Tailwind */
@import "tailwindcss/base" layer(tailwind-base);
@import "tailwindcss/components" layer(tailwind-components);
@import "tailwindcss/utilities" layer(tailwind-utilities);

/* Собственные слои */
@layer base {
  body {
    font-family: 'Custom Font', system-ui, sans-serif;
  }
}

@layer theme {
  :root {
    --custom-primary: #6366f1;
  }
}

@layer components {
  .custom-button {
    @apply px-4 py-2 rounded bg-indigo-500 text-white hover:bg-indigo-600;
  }
}

/* Пользовательские утилиты поверх Tailwind */
@layer custom-utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

## Лучшие практики

### 1. Структура слоев

```css
/* Рекомендуемая структура слоев */
@layer 
  reset,      /* Сброс стилей */
  base,       /* Базовые стили */
  theme,      /* Переменные темы */
  layout,     /* Макеты */
  components, /* Компоненты */
  utilities,  /* Утилиты */
  overrides;  /* Переопределения */

/* Каждый слой должен иметь четкое назначение */
```

### 2. Именование слоев

```css
/* Хорошие имена слоев */
@layer reset, base, theme, components, utilities;

/* Избегайте неоднозначных имен */
/* @layer styles, css, theme1; */
```

### 3. Учет специфичности

```css
/* Важно понимать, что @layer влияет на каскад, но не на специфичность */
@layer low {
  .my-class {
    color: red; /* Приоритет 1 (слой low) */
  }
}

@layer high {
  .my-class {
    color: blue; /* Приоритет 2 (слой high) - будет применен */
  }
}

/* Но специфичность все еще работает внутри слоев */
@layer high {
  #my-id.my-class {
    color: green; /* Более специфичный селектор - будет применен */
  }
}
```

## Ограничения и подводные камни

### 1. Совместимость с браузерами

CSS @layer поддерживается в современных браузерах, но не во всех. Всегда проверяйте совместимость и предоставляйте резервные варианты для старых браузеров.

### 2. Сложность отладки

Слои могут усложнить отладку CSS, особенно при большом количестве слоев. Используйте понятные имена и документацию.

### 3. Взаимодействие с другими CSS-файлами

При подключении нескольких CSS-файлов, каждый файл должен учитывать порядок слоев, определенный в других файлах.

## Заключение

CSS @layer — это мощный инструмент для управления каскадом CSS, который позволяет создавать более предсказуемые и модульные стили. Он особенно полезен в крупных проектах, где необходимо управлять приоритетами между различными источниками CSS.

При правильном использовании @layer помогает избежать чрезмерного использования `!important`, упрощает интеграцию сторонних библиотек и улучшает общую архитектуру CSS-кода.

## См. также

- [[CSS Cascade]]
- [[CSS Specificity]]
- [[CSS Architecture]]
- [[CSS Methodologies]]
- [[CSS Custom Properties]]
- [[Modern CSS]]
- [[CSS Layout]]
- [[CSS Grid]]
- [[CSS Flexbox]]
- [[CSS Preprocessors]]
- [[CSS Frameworks]]
- [[CSS Modules]]
- [[Component-Based Architecture]]
- [[Design Systems]]
- [[Frontend Architecture]]

## Дополнительные ресурсы

- [MDN Web Docs: CSS @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)
- [CSS Working Group: Cascade 5 Specification](https://www.w3.org/TR/css-cascade-5/)
- [Web.dev: Learn CSS @layer](https://web.dev/learn/css/the-cascade/)
- [CSS-Tricks: CSS @layer](https://css-tricks.com/css-cascade-layers/)