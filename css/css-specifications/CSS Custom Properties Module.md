---
aliases: [CSS Custom Properties Specification, CSS Variables, CSS Custom Properties Module, CSS Variable System]
tags: [css, variables, custom-properties, specification, css-variables]
---

# CSS Custom Properties Module: Кастомные свойства (переменные) - Полное руководство

## Введение

CSS Custom Properties Module (также известный как CSS Variables) позволяет определять кастомные свойства, значения которых могут быть использованы в других объявлениях CSS. Это позволяет создавать более гибкие и поддерживаемые стили, а также динамически изменять значения через JavaScript.

## CSS Custom Properties Module Level 1

Level 1 определяет основные возможности кастомных свойств.

### Определение кастомных свойств

Кастомные свойства определяются с префиксом `--` и могут быть установлены на любом элементе:

```css
/* Определение кастомных свойств */
:root {
  /* В корневом элементе для глобальной доступности */
  --main-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size: 16px;
  --border-radius: 4px;
  --spacing-unit: 8px;
  --shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.component {
  /* Определение в конкретном компоненте */
  --component-bg: #f8f9fa;
  --component-text: #333;
}

/* Кастомные свойства с дефолтными значениями */
.element {
  --dynamic-color: initial;         /* Значение по умолчанию */
  --dynamic-size: inherit;          /* Наследуемое значение */
}
```

### Использование кастомных свойств

Для использования кастомных свойств используется функция `var()`:

```css
.usage-examples {
  /* Базовое использование */
  color: var(--main-color);
  background-color: var(--secondary-color);
  font-size: var(--font-size);
  border-radius: var(--border-radius);
  
  /* С использованием fallback значений */
  color: var(--main-color, #000000); /* #000000 если --main-color не определена */
  
  /* В составных свойствах */
  margin: calc(var(--spacing-unit) * 2) var(--spacing-unit);
  box-shadow: var(--shadow);
  
  /* В функциях */
  background: linear-gradient(45deg, var(--main-color), var(--secondary-color));
}
```

### Наследование кастомных свойств

```css
.inheritance-parent {
  --text-color: #333;
  --bg-color: #fff;
}

.inheritance-child {
  /* Наследует --text-color и --bg-color от родителя */
  color: var(--text-color);
  background-color: var(--bg-color);
}

.inheritance-override {
  /* Переопределяет значение для этого элемента */
  --text-color: #666;
  color: var(--text-color);         /* Будет #666 */
}
```

## CSS Custom Properties Module Level 2

Level 2 добавляет дополнительные возможности, особенно в области каскада и наследования.

### Каскадные кастомные свойства

```css
.cascading-props {
  /* Использование каскадных значений в кастомных свойствах */
  --theme-color: var(--primary-color, var(--fallback-color, #000));
  
  /* Сложные значения с несколькими fallback вариантами */
  --dynamic-value: var(
    --high-priority-value, 
    var(
      --medium-priority-value, 
      var(--low-priority-value, default-value)
    )
  );
}
```

## Практические применения

### Система цветов

```css
:root {
  /* Основная палитра */
  --color-primary: #3498db;
  --color-primary-dark: #2980b9;
  --color-primary-light: #5dade2;
  
  --color-secondary: #9b59b6;
  --color-secondary-dark: #8e44ad;
  --color-secondary-light: #bb8fce;
  
  --color-success: #2ecc71;
  --color-warning: #f39c12;
  --color-danger: #e74c3c;
  --color-info: #3498db;
  
  /* Цвета фона и текста */
  --color-bg: #ffffff;
  --color-bg-alt: #f8f9fa;
  --color-text: #2c3e50;
  --color-text-secondary: #7f8c8d;
  --color-border: #e0e0e0;
}

/* Использование в компонентах */
.button {
  padding: 12px 24px;
  border: 1px solid var(--color-border);
  border-radius: var(--border-radius);
  background-color: var(--color-bg);
  color: var(--color-text);
  cursor: pointer;
  transition: all 0.2s ease;
}

.button--primary {
  background-color: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

.button--primary:hover {
  background-color: var(--color-primary-dark);
  border-color: var(--color-primary-dark);
}
```

### Темизация интерфейса

```css
/* Светлая тема (по умолчанию) */
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --header-bg: #f8f9fa;
  --card-bg: #ffffff;
  --border-color: #e0e0e0;
  --shadow: 0 2px 10px rgba(0,0,0,0.1);
}

/* Темная тема */
[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
  --header-bg: #2d2d2d;
  --card-bg: #2a2a2a;
  --border-color: #444444;
  --shadow: 0 2px 10px rgba(0,0,0,0.3);
}

/* Применение темы */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
  transition: background-color 0.3s ease, color 0.3s ease;
}

.header {
  background-color: var(--header-bg);
  border-bottom: 1px solid var(--border-color);
}

.card {
  background-color: var(--card-bg);
  border: 1px solid var(--border-color);
  box-shadow: var(--shadow);
  border-radius: 8px;
  padding: 20px;
}
```

### Адаптивные значения

```css
:root {
  /* Базовые значения */
  --font-size-sm: 14px;
  --font-size-base: 16px;
  --font-size-lg: 18px;
  --font-size-xl: 24px;
  
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  
  /* Значения, изменяющиеся по медиа-запросам */
  --container-max-width: 540px;
}

/* Адаптивные размеры */
@media (min-width: 576px) {
  :root {
    --container-max-width: 540px;
  }
}

@media (min-width: 768px) {
  :root {
    --container-max-width: 720px;
    --font-size-lg: 20px;
    --font-size-xl: 28px;
  }
}

@media (min-width: 992px) {
  :root {
    --container-max-width: 960px;
    --font-size-xl: 32px;
  }
}

@media (min-width: 1200px) {
  :root {
    --container-max-width: 1140px;
  }
}

.container {
  max-width: var(--container-max-width);
  margin: 0 auto;
  padding: 0 var(--spacing-md);
}
```

### Сложные вычисления с calc()

```css
.calculations {
  /* Вычисления с использованием кастомных свойств */
  --base-font-size: 16px;
  --scale-factor: 1.2;
  --header-height: 60px;
  --sidebar-width: 250px;
  
  /* Высота основного контента */
  --main-height: calc(100vh - var(--header-height));
  
  /* Размеры с масштабированием */
  --font-size-lg: calc(var(--base-font-size) * var(--scale-factor));
  --font-size-xl: calc(var(--font-size-lg) * var(--scale-factor));
  
  /* Адаптивные размеры */
  --grid-gap: calc(var(--spacing-md) / 2);
  --border-width: calc(var(--spacing-xs) / 4);
}

.main-content {
  height: var(--main-height);
  margin-top: var(--header-height);
}

.grid {
  display: grid;
  grid-gap: var(--grid-gap);
  grid-template-columns: var(--sidebar-width) 1fr;
}
```

## Использование с JavaScript

Кастомные свойства могут быть изменены через JavaScript:

```javascript
// Получение значения кастомного свойства
const root = document.documentElement;
const mainColor = getComputedStyle(root).getPropertyValue('--main-color').trim();

// Установка значения кастомного свойства
root.style.setProperty('--main-color', '#e74c3c');

// Изменение темы
function toggleTheme() {
  const currentTheme = root.getAttribute('data-theme');
  const newTheme = currentTheme === 'dark' ? 'light' : 'dark';
  root.setAttribute('data-theme', newTheme);
}

// Адаптивные изменения
function updateVariablesForScreen() {
  if (window.innerWidth < 768) {
    root.style.setProperty('--font-size-base', '14px');
  } else {
    root.style.setProperty('--font-size-base', '16px');
  }
}

window.addEventListener('resize', updateVariablesForScreen);
```

## Продвинутые паттерны

### Система отступов

```css
:root {
  /* Базовая сетка отступов */
  --space-factor: 4px;
  --space-1: calc(var(--space-factor) * 1); /* 4px */
  --space-2: calc(var(--space-factor) * 2); /* 8px */
  --space-3: calc(var(--space-factor) * 3); /* 12px */
  --space-4: calc(var(--space-factor) * 4); /* 16px */
  --space-5: calc(var(--space-factor) * 5); /* 20px */
  --space-6: calc(var(--space-factor) * 6); /* 24px */
  --space-8: calc(var(--space-factor) * 8); /* 32px */
  --space-10: calc(var(--space-factor) * 10); /* 40px */
  --space-12: calc(var(--space-factor) * 12); /* 48px */
  --space-16: calc(var(--space-factor) * 16); /* 64px */
}

.component {
  margin: var(--space-4);
  padding: var(--space-3) var(--space-4);
}

.button {
  padding: var(--space-2) var(--space-4);
  margin: var(--space-1);
}
```

### Система типографики

```css
:root {
  /* Базовые значения типографики */
  --font-family-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-family-secondary: 'Georgia', serif;
  
  --font-size-xs: 0.75rem;      /* 12px */
  --font-size-sm: 0.875rem;     /* 14px */
  --font-size-base: 1rem;       /* 16px */
  --font-size-lg: 1.125rem;     /* 18px */
  --font-size-xl: 1.25rem;      /* 20px */
  --font-size-2xl: 1.5rem;      /* 24px */
  --font-size-3xl: 1.875rem;    /* 30px */
  --font-size-4xl: 2.25rem;     /* 36px */
  --font-size-5xl: 3rem;        /* 48px */
  --font-size-6xl: 4rem;        /* 64px */
  
  /* Интервалы */
  --line-height-sm: 1.25;
  --line-height-base: 1.5;
  --line-height-lg: 1.75;
  
  /* Жирность шрифта */
  --font-weight-light: 300;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
}

.heading-primary {
  font-size: var(--font-size-4xl);
  font-weight: var(--font-weight-bold);
  line-height: var(--line-height-sm);
}

.text-body {
  font-size: var(--font-size-base);
  line-height: var(--line-height-base);
  color: var(--color-text);
}

.text-small {
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
}
```

### Адаптивные анимации

```css
:root {
  /* Адаптивные значения анимации */
  --animation-speed-fast: 0.1s;
  --animation-speed-normal: 0.3s;
  --animation-speed-slow: 0.5s;
  
  --animation-easing: cubic-bezier(0.4, 0, 0.2, 1);
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --animation-speed-fast: 0.01ms;
    --animation-speed-normal: 0.01ms;
    --animation-speed-slow: 0.01ms;
    --animation-easing: step-start;
  }
}

.animated-element {
  transition: all var(--animation-speed-normal) var(--animation-easing);
}

.button:hover {
  transform: translateY(-2px);
  transition-duration: var(--animation-speed-fast);
}
```

## Поддержка браузерами

- Кастомные свойства: Хорошая поддержка во всех современных браузерах
- Требует префиксов или полифиллов для IE11 и более старых браузеров
- Функция `var()` поддерживается во всех современных браузерах

## Лучшие практики

### Организация кастомных свойств

```css
:root {
  /* Группировка свойств по категориям */
  
  /* Цвета */
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  
  /* Размеры */
  --size-border: 1px;
  --size-radius: 4px;
  --size-shadow: 0 2px 4px;
  
  /* Отступы */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  
  /* Типографика */
  --font-size-base: 16px;
  --font-weight-normal: 400;
  --font-weight-bold: 700;
  
  /* Анимации */
  --transition-fast: 0.1s;
  --transition-normal: 0.3s;
  --transition-slow: 0.5s;
}
```

### Именование кастомных свойств

```css
:root {
  /* Использование понятных имен */
  --brand-color-primary: #007bff;
  --brand-color-secondary: #6c757d;
  
  /* Использование модификаторов */
  --color-primary: #007bff;
  --color-primary-dark: #0056b3;
  --color-primary-light: #80bdff;
  
  /* Семантические имена */
  --color-background: #ffffff;
  --color-text: #212529;
  --color-border: #dee2e6;
}
```

## Связанные темы

- [[CSS Values and Units]] - значения и единицы измерения
- [[CSS Cascade and Inheritance]] - каскад и наследование
- [[CSS Architecture]] - архитектура CSS

## Заключение

CSS Custom Properties Module предоставляет мощный инструмент для создания гибких, поддерживаемых и темизируемых стилей. Понимание возможностей кастомных свойств позволяет создавать более адаптивные и легко изменяемые интерфейсы.