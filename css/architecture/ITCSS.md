---
aliases: [ITCSS, Inverted Triangle CSS]
tags: [css, architecture, methodology, scalable]
---

# ITCSS (Inverted Triangle CSS)

ITCSS (Inverted Triangle CSS) — это методология архитектуры CSS, разработанная Саймоном Синеком (Simon Sinca) и Джонатаном Снупом (Jonathan Snook), которая помогает организовать CSS-код в масштабируемых проектах. Она основана на концепции перевернутого треугольника, где стили становятся всё более специфичными по мере продвижения вниз по структуре.

## Общая концепция

ITCSS разбивает CSS-код на шесть слоев, расположенных от самого общего (наименее специфичного) к самому конкретному (наиболее специфичному). Это помогает избежать проблем с каскадом и переопределением стилей.

```
|=======================|  ← Специфичность
|      Settings         |  ← Определения переменных (цвета, размеры и т.д.)
|-----------------------|
|      Tools            |  ← Функции и миксины
|-----------------------|
|      Generic          |  ← Сбросы, нормализация, box-sizing
|-----------------------|
|      Elements         |  ← Стили для HTML-элементов
|-----------------------|
|      Objects          |  ← Объекты/оболочки (без визуального оформления)
|-----------------------|
|      Components       |  ← Компоненты с визуальным оформлением
|-----------------------|
|      Utilities        |  ← Вспомогательные классы (с высокой специфичностью)
|=======================|
```

## Слои ITCSS

### 1. Settings
Содержит переменные и настройки проекта (цвета, размеры, шрифты и т.д.), которые используются в других слоях.

```scss
// settings.colors.scss
$color-primary: #007bff;
$color-secondary: #6c757d;
$color-success: #28a745;
$color-danger: #dc3545;
$color-warning: #ffc107;
$color-info: #17a2b8;
$color-light: #f8f9fa;
$color-dark: #343a40;

// settings.dimensions.scss
$dimension-small: 0.5rem;
$dimension-medium: 1rem;
$dimension-large: 1.5rem;
$dimension-xlarge: 2rem;
```

### 2. Tools
Содержит функции, миксины и другие вспомогательные инструменты, которые могут быть использованы в других слоях.

```scss
// tools.mixins.scss
@mixin respond-to($breakpoint) {
  @if $breakpoint == small {
    @media (min-width: 576px) { @content; }
  } @else if $breakpoint == medium {
    @media (min-width: 768px) { @content; }
  } @else if $breakpoint == large {
    @media (min-width: 992px) { @content; }
  } @else if $breakpoint == xlarge {
    @media (min-width: 1200px) { @content; }
  }
}

@mixin button-variant($bg, $border, $color: color-contrast($bg)) {
  background-color: $bg;
  border-color: $border;
  color: $color;

  &:hover,
  &:focus {
    background-color: shade-color($bg, 10%);
    border-color: shade-color($border, 10%);
    color: $color;
  }
}
```

### 3. Generic
Содержит сбросы стилей, нормализацию и базовые установки для box-sizing.

```css
/* generic.reset.css */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* generic.normalize.css */
html {
  line-height: 1.15;
  -webkit-text-size-adjust: 100%;
}

body {
  margin: 0;
}

/* generic.box-sizing.css */
html {
  box-sizing: border-box;
}

*, *::before, *::after {
  box-sizing: inherit;
}
```

### 4. Elements
Содержит стили для HTML-элементов без классов (например, h1, h2, p, a и т.д.).

```css
/* elements.headings.css */
h1 {
  font-size: 2.5rem;
  font-weight: 700;
  line-height: 1.2;
  margin-top: 0;
  margin-bottom: 0.5rem;
}

h2 {
  font-size: 2rem;
  font-weight: 600;
  line-height: 1.3;
  margin-top: 0.5rem;
  margin-bottom: 0.75rem;
}

/* elements.links.css */
a {
  color: #007bff;
  text-decoration: none;
  background-color: transparent;
}

a:hover {
  color: #0056b3;
  text-decoration: underline;
}
```

### 5. Objects
Содержит объекты, которые определяют структурные шаблоны без визуального оформления. Обычно используются префиксы `.o-`.

```css
/* objects.wrappers.css */
.o-container {
  width: 100%;
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
}

.o-container-fluid {
  width: 100%;
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
}

/* objects.grid.css */
.o-grid {
  display: flex;
  flex-wrap: wrap;
  margin-right: -15px;
  margin-left: -15px;
}

.o-grid__item {
  position: relative;
  width: 100%;
  padding-right: 15px;
  padding-left: 15px;
}
```

### 6. Components
Содержит стили для конкретных компонентов с визуальным оформлением. Используются префиксы `.c-`.

```css
/* components.buttons.css */
.c-btn {
  display: inline-block;
  font-weight: 400;
  color: #212529;
  text-align: center;
  vertical-align: middle;
  user-select: none;
  background-color: transparent;
  border: 1px solid transparent;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  border-radius: 0.25rem;
  transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, 
              border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.c-btn--primary {
  @extend .c-btn;
  color: #fff;
  background-color: #007bff;
  border-color: #007bff;
}

.c-btn--secondary {
  @extend .c-btn;
  color: #fff;
  background-color: #6c757d;
  border-color: #6c757d;
}

/* components.cards.css */
.c-card {
  position: relative;
  display: flex;
  flex-direction: column;
  min-width: 0;
  word-wrap: break-word;
  background-color: #fff;
  background-clip: border-box;
  border: 1px solid rgba(0, 0, 0, 0.125);
  border-radius: 0.25rem;
}

.c-card__body {
  flex: 1 1 auto;
  min-height: 1px;
  padding: 1.25rem;
}
```

### 7. Utilities
Содержит вспомогательные классы с высокой специфичностью, которые переопределяют все предыдущие слои. Обычно используются префиксы `.u-` или `.is-`.

```css
/* utilities.spacing.css */
.u-margin-top-none { margin-top: 0 !important; }
.u-margin-right-none { margin-right: 0 !important; }
.u-margin-bottom-none { margin-bottom: 0 !important; }
.u-margin-left-none { margin-left: 0 !important; }
.u-padding-top-none { padding-top: 0 !important; }
.u-padding-right-none { padding-right: 0 !important; }
.u-padding-bottom-none { padding-bottom: 0 !important; }
.u-padding-left-none { padding-left: 0 !important; }

/* utilities.display.css */
.u-hidden { display: none !important; }
.u-block { display: block !important; }
.u-inline { display: inline !important; }
.u-inline-block { display: inline-block !important; }

/* utilities.helpers.css */
.u-clearfix::after {
  display: block;
  clear: both;
  content: "";
}
```

## Преимущества ITCSS

1. **Предсказуемость**: Благодаря структурированному подходу, легче понимать, где искать нужные стили
2. **Масштабируемость**: Подходит для проектов любого размера
3. **Управление специфичностью**: Слои организованы так, что предотвращают проблемы с переопределением стилей
4. **Командная разработка**: Позволяет нескольким разработчикам работать с CSS-кодом без конфликтов

## Применение в российских реалиях 2025 года

В российском веб-разработке 2025 года ITCSS особенно актуален для:

- Крупных корпоративных проектов с командами из 10+ разработчиков
- Государственных информационных систем, где важна поддержка и документация
- E-commerce платформ, требующих масштабируемой архитектуры
- Проектов, использующих [[CSS-архитектура]] и [[Atomic-Design]] в сочетании с ITCSS

## Связь с другими методологиями

ITCSS хорошо сочетается с другими методологиями:
- [[Atomic-Design]]: можно использовать ITCSS для организации CSS-кода атомов, молекул и т.д.
- [[БЭМ]]: можно использовать БЭМ для именования классов в слое Components
- [[Организация-файлов]]: ITCSS предоставляет структуру для файлов

## Заключение

ITCSS — мощная методология для создания масштабируемых CSS-архитектур. Он особенно полезен в больших проектах, где важна предсказуемость и управляемость CSS-кода. В российских реалиях 2025 года это один из ключевых подходов для профессиональных CSS-архитекторов.

## См. также

- [[CSS-архитектура]]
- [[Atomic-Design]]
- [[Организация-файлов]]
- [[Масштабируемость]]
- [[БЭМ]]