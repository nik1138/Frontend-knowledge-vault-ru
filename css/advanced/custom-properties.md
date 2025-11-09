# Продвинутые возможности - Пользовательские свойства CSS

Пользовательские свойства CSS (CSS Custom Properties), также известные как CSS-переменные, позволяют создавать переиспользуемые значения, которые могут быть изменены динамически во время выполнения. Это мощная возможность современного CSS, введенная в спецификации CSS Variables Module Level 1.

## Table of Contents

- [Основы пользовательских свойств](#основы-пользовательских-свойств)
- [Синтаксис и особенности](#синтаксис-и-особенности)
- [Практические примеры](#практические-примеры)
- [Динамическое управление через JavaScript](#динамическое-управление-через-javascript)
- [Продвинутые техники](#продвинутые-техники)
- [Применение в архитектуре CSS](#применение-в-архитектуре-css)
- [Производительность и оптимизация](#производительность-и-оптимизация)
- [Лучшие практики](#лучшие-практики)

## Основы пользовательских свойств

### Объявление и использование

Пользовательские свойства объявляются с префиксом `--` и используются с функцией `var()`:

```css
:root {
  --main-color: #3498db;
  --main-bg: #f8f9fa;
  --border-radius: 8px;
  --spacing-unit: 16px;
}

.component {
  background-color: var(--main-bg);
  color: var(--main-color);
  border-radius: var(--border-radius);
  padding: var(--spacing-unit);
}
```

### Область видимости

Пользовательские свойства наследуются и могут быть переопределены в разных контекстах:

```css
:root {
  --primary-color: #3498db;
}

.container {
  --primary-color: #e74c3c; /* Переопределяет значение для потомков */
}

.component {
  color: var(--primary-color); /* Использует #e74c3c в .container и #3498db в других местах */
}
```

## Синтаксис и особенности

### Значения по умолчанию

Функция `var()` поддерживает значение по умолчанию:

```css
.element {
  /* Использует --main-color, если доступно, иначе #ff0000 */
  color: var(--main-color, #ff0000);

  /* Значение по умолчанию может быть сложным */
  background: var(--background-gradient, linear-gradient(to right, #fff, #000));

  /* Можно использовать другие переменные как значение по умолчанию */
  border-color: var(--border-color, var(--main-color, #000));
}
```

### Валидация и fallback значения

```css
/* CSS проверяет валидность значений при вычислении */
.invalid-example {
  /* Если --invalid-value не является валидным значением, используется fallback */
  color: var(--invalid-value, red);

  /* Валидные значения будут использованы */
  font-size: var(--valid-size, 16px);
}
```

### Вложенные переменные

CSS-переменные могут ссылаться друг на друга:

```css
:root {
  --primary-hue: 220;
  --primary-saturation: 100%;
  --primary-lightness: 50%;
  
  --primary-color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
  --primary-color-light: hsl(var(--primary-hue), var(--primary-saturation), calc(var(--primary-lightness) + 20%));
  --primary-color-dark: hsl(var(--primary-hue), var(--primary-saturation), calc(var(--primary-lightness) - 20%));
}

.button {
  background-color: var(--primary-color);
}

.button:hover {
  background-color: var(--primary-color-light);
}
```

## Практические примеры

### Пример 1: Система цветов

```css
:root {
  /* Основные цвета */
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  --color-warning: #ffc107;
  --color-info: #17a2b8;

  /* Цвета фона */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f8f9fa;
  --color-bg-tertiary: #e9ecef;

  /* Цвета текста */
  --color-text-primary: #212529;
  --color-text-secondary: #6c757d;
  --color-text-muted: #868e96;

  /* Цвета границ */
  --color-border: #dee2e6;
  --color-border-light: #e9ecef;
}

/* Использование цветовой системы */
.card {
  background-color: var(--color-bg-primary);
  border: 1px solid var(--color-border);
  color: var(--color-text-primary);
}

.btn {
  padding: 8px 16px;
  border-radius: 4px;
  border: 1px solid transparent;
  cursor: pointer;
}

.btn--primary {
  background-color: var(--color-primary);
  color: white;
}

.btn--secondary {
  background-color: var(--color-secondary);
  color: white;
}

.btn--success {
  background-color: var(--color-success);
  color: white;
}
```

### Пример 2: Адаптивная типографика

```css
:root {
  /* Базовые размеры */
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-size-xl: 1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;
  --font-size-4xl: 2.25rem;

  /* Адаптивные размеры с использованием clamp() */
  --font-size-responsive-sm: clamp(0.8rem, 2vw, 1rem);
  --font-size-responsive-base: clamp(1rem, 3vw, 1.25rem);
  --font-size-responsive-lg: clamp(1.25rem, 4vw, 1.5rem);
  --font-size-responsive-xl: clamp(1.5rem, 5vw, 2rem);

  /* Интерлиньяж */
  --line-height-tight: 1.25;
  --line-height-snug: 1.375;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.625;
}

/* Использование адаптивной типографики */
.text-xs { font-size: var(--font-size-xs); }
.text-sm { font-size: var(--font-size-sm); }
.text-base { font-size: var(--font-size-base); }
.text-lg { font-size: var(--font-size-lg); }
.text-xl { font-size: var(--font-size-xl); }

.text-responsive-sm { font-size: var(--font-size-responsive-sm); }
.text-responsive-base { font-size: var(--font-size-responsive-base); }
.text-responsive-lg { font-size: var(--font-size-responsive-lg); }
.text-responsive-xl { font-size: var(--font-size-responsive-xl); }

.body-text {
  font-size: var(--font-size-base);
  line-height: var(--line-height-normal);
}

.display-text {
  font-size: var(--font-size-4xl);
  line-height: var(--line-height-tight);
  font-weight: 700;
}
```

### Пример 3: Темизация интерфейса

```css
:root {
  /* Светлая тема по умолчанию */
  --bg-color: #ffffff;
  --text-color: #212529;
  --border-color: #dee2e6;
  --card-bg: #ffffff;
  --card-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  --header-bg: #f8f9fa;
}

/* Темная тема */
[data-theme="dark"] {
  --bg-color: #212529;
  --text-color: #ffffff;
  --border-color: #495057;
  --card-bg: #343a40;
  --card-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
  --header-bg: #343a40;
}

/* Альтернативная тема */
[data-theme="blue"] {
  --bg-color: #e3f2fd;
  --text-color: #1565c0;
  --border-color: #90caf9;
  --card-bg: #bbdefb;
  --card-shadow: 0 4px 6px rgba(21, 101, 192, 0.1);
  --header-bg: #90caf9;
}

/* Использование темы */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
}

.card {
  background-color: var(--card-bg);
  border: 1px solid var(--border-color);
  box-shadow: var(--card-shadow);
  padding: 1.5rem;
  border-radius: 8px;
}

.header {
  background-color: var(--header-bg);
  padding: 1rem;
  border-bottom: 1px solid var(--border-color);
}
```

### Пример 4: Анимации с переменными

```css
:root {
  --animation-duration: 0.3s;
  --animation-easing: ease-in-out;
  --transform-scale: 1.05;
  --shadow-intensity: 0.2;
}

.animated-element {
  transition:
    transform var(--animation-duration) var(--animation-easing),
    box-shadow var(--animation-duration) var(--animation-easing);
}

.animated-element:hover {
  transform: scale(var(--transform-scale));
  box-shadow: 0 8px 25px rgba(0, 0, 0, var(--shadow-intensity));
}

/* Анимация с использованием переменных */
@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.pulse-element {
  animation: pulse var(--animation-duration) var(--animation-easing) infinite;
}
```

## Динамическое управление через JavaScript

### Чтение и изменение переменных

```javascript
// Получение значения переменной
const root = document.documentElement;
const mainColor = getComputedStyle(root).getPropertyValue('--main-color').trim();
console.log(mainColor); // #3498db

// Изменение значения переменной
root.style.setProperty('--main-color', '#e74c3c');

// Изменение значения с использованием JavaScript
const element = document.querySelector('.component');
element.style.setProperty('--spacing-unit', '24px');

// Изменение нескольких переменных
function updateTheme(theme) {
  const root = document.documentElement;
  root.style.setProperty('--primary-color', theme.primary);
  root.style.setProperty('--secondary-color', theme.secondary);
  root.style.setProperty('--background-color', theme.background);
  root.style.setProperty('--text-color', theme.text);
}
```

### Пример интерактивной темы

```html
<div class="theme-controls">
  <button onclick="setTheme('light')">Светлая</button>
  <button onclick="setTheme('dark')">Темная</button>
  <button onclick="setTheme('blue')">Синяя</button>
  <input type="color" id="primary-color-picker" onchange="updatePrimaryColor(this.value)">
</div>

<div class="card">
  <h2>Пример карточки</h2>
  <p>Содержимое карточки с динамической темой</p>
</div>

<script>
function setTheme(themeName) {
  document.documentElement.setAttribute('data-theme', themeName);
}

function updatePrimaryColor(color) {
  document.documentElement.style.setProperty('--color-primary', color);
}

// Инициализация темы из localStorage
document.addEventListener('DOMContentLoaded', function() {
  const savedTheme = localStorage.getItem('theme') || 'light';
  setTheme(savedTheme);
});

// Синхронизация цвета пикера с переменной
function initColorPicker() {
  const picker = document.getElementById('primary-color-picker');
  const root = document.documentElement;
  const currentColor = getComputedStyle(root).getPropertyValue('--color-primary').trim();
  picker.value = currentColor;
}
</script>
```

### Анимации с динамическими переменными

```javascript
// Динамическое изменение анимации
function updateAnimationSpeed(factor) {
  document.documentElement.style.setProperty('--animation-duration', `${0.3 * factor}s`);
}

// Интерактивное изменение цвета с анимацией
function updateColorWithTransition(element, property, newColor) {
  element.style.setProperty('--transition-temp', newColor);
  element.style.transition = `${property} var(--animation-duration) var(--animation-easing)`;
  element.style.setProperty(property, 'var(--transition-temp)');
}
```

## Продвинутые техники

### Переменные в градиентах

```css
:root {
  --gradient-start: #ff0000;
  --gradient-end: #0000ff;
  --gradient-angle: 45deg;
}

.gradient-box {
  /* Использование переменных в градиентах */
  background: linear-gradient(
    var(--gradient-angle),
    var(--gradient-start),
    var(--gradient-end)
  );

  /* Сложные градиенты */
  background:
    linear-gradient(
      0deg,
      transparent 0%,
      var(--gradient-start) 100%
    ),
    linear-gradient(
      90deg,
      var(--gradient-start),
      var(--gradient-end)
    );
}

/* Динамические градиенты */
.dynamic-gradient {
  --start-hue: 0;
  --end-hue: 360;
  background: linear-gradient(
    45deg,
    hsl(var(--start-hue), 100%, 50%),
    hsl(var(--end-hue), 100%, 50%)
  );
}
```

### Переменные в трансформациях

```css
:root {
  --rotate-angle: 45deg;
  --scale-factor: 1.2;
  --translate-x: 10px;
  --translate-y: 20px;
}

.transformed-element {
  transform:
    rotate(var(--rotate-angle))
    scale(var(--scale-factor))
    translate(var(--translate-x), var(--translate-y));
}

/* Анимированные трансформации */
.animated-transform {
  --current-rotation: 0deg;
  transform: rotate(var(--current-rotation));
  transition: transform 0.3s ease;
}
```

### Условные переменные (через CSS-каскад)

```css
:root {
  --is-large: 1;
  --is-small: 0;
}

.container[data-size="small"] {
  --is-large: 0;
  --is-small: 1;
}

.element {
  /* Использование условных переменных */
  padding: calc(var(--is-large) * 20px + var(--is-small) * 10px);
  font-size: calc(var(--is-large) * 1.2em + var(--is-small) * 0.8em);
}

/* Более сложные условные переменные */
.card {
  --card-padding: 1rem;
  --card-padding-condensed: 0.5rem;
  --use-condensed: 0;
}

.card.condensed {
  --use-condensed: 1;
}

.card .content {
  padding: calc(
    var(--use-condensed) * var(--card-padding-condensed) + 
    (1 - var(--use-condensed)) * var(--card-padding)
  );
}
```

### Математические вычисления с переменными

```css
:root {
  --base-size: 16px;
  --multiplier: 1.5;
  --offset: 10px;
}

.calculated-sizes {
  /* Умножение */
  width: calc(var(--base-size) * var(--multiplier));
  
  /* Сложение */
  height: calc(var(--base-size) + var(--offset));
  
  /* Вычитание */
  margin: calc(var(--base-size) - 2px);
  
  /* Сложные вычисления */
  padding: calc((var(--base-size) * var(--multiplier)) + var(--offset));
}

/* Процентные вычисления */
.responsive-calc {
  width: calc(100% - (var(--base-size) * 2));
  max-width: calc(var(--base-size) * 10);
}
```

## Применение в архитектуре CSS

### Система тем с CSS-переменными

```css
/* Система тем */
:root {
  /* Цвета */
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  --color-warning: #ffc107;
  --color-info: #17a2b8;
  --color-light: #f8f9fa;
  --color-dark: #343a40;
  
  /* Цвета фона */
  --color-bg-body: #fff;
  --color-bg-surface: #fff;
  --color-bg-muted: #f8f9fa;
  
  /* Цвета текста */
  --color-text-primary: #212529;
  --color-text-secondary: #6c757d;
  --color-text-muted: #6c757d;
  --color-text-white: #fff;
  
  /* Цвета границ */
  --color-border: #dee2e6;
  --color-border-light: #e9ecef;
  --color-border-dark: #adb5bd;
  
  /* Типографика */
  --font-family-sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  --font-family-serif: Georgia, "Times New Roman", Times, serif;
  --font-family-monospace: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  
  --font-size-xs: 0.75rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.25rem;
  --font-size-xl: 1.5rem;
  --font-size-2xl: 1.75rem;
  --font-size-3xl: 2rem;
  
  --font-weight-light: 300;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  --line-height-xs: 1.25;
  --line-height-sm: 1.4;
  --line-height-base: 1.5;
  --line-height-lg: 1.6;
  
  /* Отступы */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-base: 1rem;
  --spacing-md: 1.5rem;
  --spacing-lg: 2rem;
  --spacing-xl: 3rem;
  
  /* Размеры */
  --border-width: 1px;
  --border-radius: 0.375rem;
  --border-radius-sm: 0.25rem;
  --border-radius-lg: 0.5rem;
  --border-radius-full: 9999px;
  
  /* Тени */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
  
  /* Анимации */
  --transition-duration-fast: 0.15s;
  --transition-duration-normal: 0.3s;
  --transition-duration-slow: 0.5s;
  --transition-timing: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Темная тема */
[data-theme="dark"] {
  /* Цвета фона */
  --color-bg-body: #1a1a1a;
  --color-bg-surface: #2d2d2d;
  --color-bg-muted: #3a3a3a;
  
  /* Цвета текста */
  --color-text-primary: #f0f0f0;
  --color-text-secondary: #b0b0b0;
  --color-text-muted: #8a8a8a;
  
  /* Цвета границ */
  --color-border: #404040;
  --color-border-light: #333;
  --color-border-dark: #555;
  
  /* Тени */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.3);
  --shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.3), 0 1px 2px 0 rgba(0, 0, 0, 0.2);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.3), 0 2px 4px -1px rgba(0, 0, 0, 0.2);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.3), 0 4px 6px -2px rgba(0, 0, 0, 0.2);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.3), 0 10px 10px -5px rgba(0, 0, 0, 0.2);
}
```

### Система отступов и размеров

```css
/* Система отступов */
:root {
  --spacing-scale: 0.25rem;
  --spacing-1: calc(var(--spacing-scale) * 1);  /* 4px */
  --spacing-2: calc(var(--spacing-scale) * 2);  /* 8px */
  --spacing-3: calc(var(--spacing-scale) * 4);  /* 16px */
  --spacing-4: calc(var(--spacing-scale) * 6);  /* 24px */
  --spacing-5: calc(var(--spacing-scale) * 8);  /* 32px */
  --spacing-6: calc(var(--spacing-scale) * 12); /* 48px */
  --spacing-7: calc(var(--spacing-scale) * 16); /* 64px */
  --spacing-8: calc(var(--spacing-scale) * 24); /* 96px */
  --spacing-9: calc(var(--spacing-scale) * 32); /* 128px */
}

/* Система размеров */
:root {
  --size-scale: 1rem;
  --size-1: calc(var(--size-scale) * 0.25);  /* 4px */
  --size-2: calc(var(--size-scale) * 0.5);   /* 8px */
  --size-3: calc(var(--size-scale) * 0.75);  /* 12px */
  --size-4: calc(var(--size-scale) * 1);     /* 16px */
  --size-5: calc(var(--size-scale) * 1.25);  /* 20px */
  --size-6: calc(var(--size-scale) * 1.5);   /* 24px */
  --size-7: calc(var(--size-scale) * 1.75);  /* 28px */
  --size-8: calc(var(--size-scale) * 2);     /* 32px */
  --size-9: calc(var(--size-scale) * 2.25);  /* 36px */
  --size-10: calc(var(--size-scale) * 2.5);  /* 40px */
  --size-12: calc(var(--size-scale) * 3);    /* 48px */
  --size-16: calc(var(--size-scale) * 4);    /* 64px */
  --size-20: calc(var(--size-scale) * 5);    /* 80px */
  --size-24: calc(var(--size-scale) * 6);    /* 96px */
  --size-32: calc(var(--size-scale) * 8);    /* 128px */
}
```

## Производительность и оптимизация

### Оптимизация производительности

CSS-переменные не оказывают значительного влияния на производительность, но есть несколько моментов:

```css
/* Оптимизированные переменные */
:root {
  /* Используйте простые значения, когда это возможно */
  --simple-color: #007bff;
  
  /* Избегайте сложных вычислений в переменных */
  --complex-calc: calc(100% - (var(--spacing-lg) * 2)); /* OK */
  
  /* Не используйте слишком сложные вычисления */
  --too-complex: calc(((var(--spacing-lg) * 2) + var(--spacing-md)) / 3); /* Избегайте */
}

/* Используйте переменные рационально */
.optimized-component {
  /* Хорошо: переменные для часто используемых значений */
  color: var(--text-color);
  background-color: var(--bg-color);
  
  /* Не очень: переменные для уникальных значений */
  border-left-color: var(--special-border-color); /* Только для одного элемента */
}
```

### Использование CSS-переменных для оптимизации анимаций

```css
/* Оптимизированные анимации с переменными */
:root {
  --animation-duration: 0.3s;
  --animation-timing: ease-in-out;
  --transform-scale: 1.05;
}

.optimized-animation {
  /* Используйте transform и opacity для анимаций */
  transition: transform var(--animation-duration) var(--animation-timing),
              opacity var(--animation-duration) var(--animation-timing);
}

.optimized-animation:hover {
  transform: scale(var(--transform-scale));
}
```

### Избегание "переопределения" переменных

```css
/* Плохо: слишком много переопределений */
.element {
  --temp-color: var(--primary-color);
  --temp-bg: var(--secondary-color);
  --temp-border: var(--border-color);
  color: var(--temp-color);
  background: var(--temp-bg);
  border: 1px solid var(--temp-border);
}

/* Лучше: прямое использование переменных */
.element {
  color: var(--primary-color);
  background: var(--secondary-color);
  border: 1px solid var(--border-color);
}
```

## Лучшие практики

### 1. Структурирование переменных

```css
:root {
  /* Цвета */
  --color-primary: #007bff;
  --color-primary-light: #3395ff;
  --color-primary-dark: #0062cc;

  /* Размеры */
  --size-border: 1px;
  --size-radius: 4px;
  --size-shadow: 8px;

  /* Отступы */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  /* Типографика */
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-size-base: 1rem;
  --font-weight-normal: 400;
  --font-weight-bold: 700;

  /* Анимации */
  --transition-duration: 0.3s;
  --transition-timing: ease-in-out;
}
```

### 2. Использование осмысленных имен

```css
/* Хорошие имена */
:root {
  --color-brand-primary: #007bff;
  --color-brand-secondary: #6c757d;
  --size-border-standard: 1px;
  --size-radius-card: 8px;
  --spacing-unit: 1rem;
  --font-size-heading: 1.5rem;
}

/* Плохие имена */
:root {
  --blue: #007bff; /* Слишком общий */
  --big: 2rem;     /* Не описывает назначение */
  --radius: 4px;   /* Не ясно, для чего */
}
```

### 3. Организация по компонентам

```css
/* Переменные для конкретных компонентов */
.card {
  --card-padding: 1.5rem;
  --card-border-radius: 8px;
  --card-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.modal {
  --modal-max-width: 500px;
  --modal-padding: 2rem;
  --modal-border-radius: 8px;
  --modal-backdrop: rgba(0, 0, 0, 0.5);
}

.button {
  --button-padding: 0.5rem 1rem;
  --button-border-radius: 4px;
  --button-font-size: 1rem;
}
```

### 4. Использование префиксов для организации

```css
:root {
  /* Цвета */
  --color-primary: #007bff;
  --color-primary-hover: #0056b3;
  --color-primary-active: #004085;
  
  /* Размеры */
  --size-border: 1px;
  --size-radius: 4px;
  --size-shadow: 8px;
  
  /* Отступы */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  
  /* Анимации */
  --anim-duration: 0.3s;
  --anim-easing: ease-in-out;
}
```

### 5. Использование в комбинации с современными CSS-фичами

```css
:root {
  --primary-hue: 220;
  --primary-saturation: 100%;
  --primary-lightness: 50%;
  --primary-color: hsl(var(--primary-hue), var(--primary-saturation), var(--primary-lightness));
}

/* Использование с clamp() для адаптивности */
.responsive-element {
  --font-size: clamp(1rem, 2.5vw, 2rem);
  font-size: var(--font-size);
}
```

## Совместимость и fallback

### Проверка поддержки

```css
.element {
  /* Fallback для старых браузеров */
  color: #3498db;

  /* Современное значение */
  color: var(--main-color, #3498db);
}

/* Проверка поддержки через CSS */
@supports (--css: variables) {
  .supports-variables {
    color: var(--main-color);
  }
}

@supports not (--css: variables) {
  .no-variables {
    color: #3498db; /* Fallback */
  }
}

/* Проверка поддержки через JavaScript */
if (window.CSS && window.CSS.supports && window.CSS.supports('--custom-property', 'value')) {
  // Поддержка CSS-переменных
  document.documentElement.classList.add('supports-variables');
} else {
  // Отсутствие поддержки
  document.documentElement.classList.add('no-variables');
}
```

## Заключение

Пользовательские свойства CSS - мощный инструмент для создания гибких и поддерживаемых стилевых архитектур. Они позволяют создавать системы тем, упрощать управление значениями и динамически изменять стили. Современные возможности, такие как вложенные переменные, вычисления и динамическое управление через JavaScript, расширяют возможности использования CSS-переменных. Правильное использование CSS-переменных значительно улучшает читаемость и поддерживаемость кода, особенно в больших проектах. Важно следовать лучшим практикам организации переменных и учитывать производительность при создании сложных систем.

#programming #css #custom-properties #variables #web-development #frontend #theming #architecture