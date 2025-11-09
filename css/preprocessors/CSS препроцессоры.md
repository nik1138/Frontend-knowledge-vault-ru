# CSS препроцессоры: Sass, Less, Stylus и современные альтернативы

## Введение

CSS препроцессоры расширяют возможности стандартного CSS, добавляя функции программирования, такие как переменные, вложенные правила, миксины, функции и операции. Они помогают создавать более организованный, масштабируемый и поддерживаемый CSS-код. В этой статье мы рассмотрим популярные препроцессоры и современные альтернативы.

## Популярные CSS препроцессоры

### 1. Sass/SCSS

Sass (Syntactically Awesome Style Sheets) - один из самых популярных CSS препроцессоров с двумя синтаксисами: SCSS (Sassy CSS) и оригинальный Sass.

#### Основные возможности SCSS

```scss
// Переменные
$primary-color: #3498db;
$secondary-color: #2ecc71;
$font-size-base: 16px;
$border-radius: 4px;

// Вложенные правила
.navbar {
  background-color: $primary-color;
  border-radius: $border-radius;
  
  .nav-item {
    display: inline-block;
    padding: 1rem;
    
    &:hover {
      background-color: darken($primary-color, 10%);
    }
    
    a {
      color: white;
      text-decoration: none;
      
      &.active {
        font-weight: bold;
      }
    }
  }
}

// Миксины
@mixin button-style($bg-color, $text-color: white) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  border-radius: $border-radius;
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    background-color: darken($bg-color, 10%);
  }
}

// Использование миксина
.btn-primary {
  @include button-style($primary-color);
}

.btn-secondary {
  @include button-style($secondary-color);
}

// Функции
@function calculate-rem($px-value) {
  $base-font-size: 16px;
  @return ($px-value / $base-font-size) * 1rem;
}

.element {
  font-size: calculate-rem(18px);
  margin: calculate-rem(16px);
}

// Условия
@mixin responsive-button($size) {
  @if $size == small {
    padding: 0.25rem 0.5rem;
    font-size: calculate-rem(14px);
  } @else if $size == large {
    padding: 0.75rem 1.5rem;
    font-size: calculate-rem(18px);
  } @else {
    padding: 0.5rem 1rem;
    font-size: calculate-rem(16px);
  }
}

.btn-small {
  @include responsive-button(small);
}

// Циклы
.grid {
  @for $i from 1 through 12 {
    &.col-#{$i} {
      width: percentage($i / 12);
    }
  }
}

// Каждый элемент списка
$colors: red, blue, green, yellow;

@each $color in $colors {
  .text-#{$color} {
    color: $color;
  }
}
```

#### Продвинутые возможности Sass

```scss
// Расширение/наследование
%button-base {
  border: none;
  border-radius: $border-radius;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn {
  @extend %button-base;
  padding: 0.5rem 1rem;
}

// Операции
.container {
  $width: 1200px;
  $gutter: 20px;
  $columns: 12;
  
  width: $width;
  padding: $gutter / 2;
  
  .column {
    width: ($width - ($gutter * ($columns + 1))) / $columns;
    margin-right: $gutter;
    
    &:last-child {
      margin-right: 0;
    }
  }
}

// Математические функции
.element {
  transform: rotate(45deg) scale(1.2);
  opacity: 0.8;
}

// Работа со строками
$class-name: "primary";
.element-#{$class-name} {
  background-color: $primary-color;
}

// Словари/карты
$theme-colors: (
  primary: #007bff,
  secondary: #6c757d,
  success: #28a745,
  danger: #dc3545,
  warning: #ffc107,
  info: #17a2b8
);

@each $name, $color in $theme-colors {
  .btn-#{$name} {
    @include button-style($color);
  }
  
  .text-#{$name} {
    color: $color;
  }
  
  .bg-#{$name} {
    background-color: $color;
  }
}
```

### 2. Less

Less - это CSS препроцессор, который добавляет возможности программирования в CSS.

```less
// Переменные
@primary-color: #3498db;
@font-size-base: 16px;
@border-radius: 4px;

// Вложенные правила
.navbar {
  background-color: @primary-color;
  border-radius: @border-radius;
  
  .nav-item {
    display: inline-block;
    padding: 1rem;
    
    &:hover {
      background-color: darken(@primary-color, 10%);
    }
    
    a {
      color: white;
      text-decoration: none;
      
      &.active {
        font-weight: bold;
      }
    }
  }
}

// Миксины
.button-style(@bg-color, @text-color: white) {
  background-color: @bg-color;
  color: @text-color;
  border: none;
  border-radius: @border-radius;
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    background-color: darken(@bg-color, 10%);
  }
}

.btn-primary {
  .button-style(@primary-color);
}

// Параметризованные миксины
.transition(@duration: 0.3s, @property: all, @easing: ease) {
  transition: @property @duration @easing;
}

.element {
  .transition(0.5s, transform, ease-in-out);
}

// Условия
.loop(@counter) when (@counter > 0) {
  .col-@{counter} {
    width: percentage(@counter / 12);
  }
  .loop(@counter - 1);
}
.loop(12); // Генерирует col-1 до col-12

// Функции
.calculate-rem(@px-value) {
  @base-font-size: 16px;
  return: unit(@px-value / @base-font-size, rem);
}

.element {
  font-size: .calculate-rem(18px)[return];
}

// Операции
.container {
  @width: 1200px;
  @gutter: 20px;
  
  width: @width;
  padding: @gutter / 2;
  
  .column {
    width: (@width - (@gutter * 3)) / 2; // Для 2 колонок
    margin-right: @gutter;
  }
}
```

### 3. Stylus

Stylus - это гибкий CSS препроцессор с возможностью опускать скобки, точки с запятой и использовать отступы для вложенности.

```stylus
// Переменные
primary-color = #3498db
font-size-base = 16px
border-radius = 4px

// Вложенные правила (без скобок)
navbar
  background-color primary-color
  border-radius border-radius

  .nav-item
    display inline-block
    padding 1rem

    &:hover
      background-color darken(primary-color, 10%)

    a
      color white
      text-decoration none

      &.active
        font-weight bold

// Миксины
button-style(bg-color, text-color = white)
  background-color bg-color
  color text-color
  border none
  border-radius border-radius
  padding 0.5rem 1rem
  cursor pointer
  transition all 0.3s ease

  &:hover
    background-color darken(bg-color, 10%)

.btn-primary
  button-style(primary-color)

// Условия
responsive-button(size)
  if size == 'small'
    padding 0.25rem 0.5rem
    font-size 0.875rem
  else if size == 'large'
    padding 0.75rem 1.5rem
    font-size 1.25rem
  else
    padding 0.5rem 1rem
    font-size 1rem

.btn-small
  responsive-button('small')

// Циклы
for i in (1..12)
  .col-{i}
    width (i / 12) * 100%

// Операции
container-width = 1200px
gutter = 20px

.container
  width container-width
  padding gutter / 2

  .column
    width (container-width - (gutter * 3)) / 2
    margin-right gutter
```

## Современные альтернативы препроцессорам

### 1. PostCSS

PostCSS - это инструмент для преобразования CSS с помощью плагинов. Он может заменить многие функции препроцессоров.

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),        // Импорт файлов
    require('postcss-nested'),        // Вложенные правила
    require('postcss-custom-properties'), // Кастомные свойства
    require('postcss-calc'),          // Вычисления
    require('autoprefixer'),          // Вендорные префиксы
    require('cssnano')                // Минификация
  ]
};
```

```css
/* CSS с PostCSS плагинами */
:root {
  --primary-color: #3498db;
  --font-size-base: 16px;
}

/* Вложенные правила (с postcss-nested) */
.navbar {
  background-color: var(--primary-color);
  
  .nav-item {
    display: inline-block;
    
    &:hover {
      background-color: shade(var(--primary-color), 10%);
    }
  }
}

/* Математические вычисления (с postcss-calc) */
.container {
  width: calc(100% - 2rem);
  margin: calc(var(--font-size-base) * 2) auto;
}
```

### 2. CSS-in-JS библиотеки

```javascript
// styled-components
import styled from 'styled-components';

const Button = styled.button`
  background-color: ${props => props.primary ? '#3498db' : '#2ecc71'};
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  
  &:hover {
    background-color: ${props => props.primary ? '#2980b9' : '#27ae60'};
  }
  
  ${props => props.size === 'large' && `
    padding: 0.75rem 1.5rem;
    font-size: 1.25rem;
  `}
`;

// Использование
<Button primary>Primary Button</Button>
<Button size="large">Large Button</Button>
```

### 3. CSS Modules

```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.primary {
  composes: button;
  background-color: #3498db;
  color: white;
}

.secondary {
  composes: button;
  background-color: #2ecc71;
  color: white;
}

.large {
  composes: button;
  padding: 0.75rem 1.5rem;
  font-size: 1.25rem;
}
```

```javascript
// Component.js
import styles from './Button.module.css';

function Button({ variant = 'primary', size, children }) {
  const buttonClass = [
    styles.button,
    styles[variant],
    size && styles[size]
  ].filter(Boolean).join(' ');
  
  return <button className={buttonClass}>{children}</button>;
}
```

## Сравнение препроцессоров

### Таблица сравнения

| Функция | Sass/SCSS | Less | Stylus | PostCSS |
|---------|-----------|------|--------|---------|
| Переменные | `$var: value` | `@var: value` | `var = value` | CSS Custom Properties |
| Вложенные правила | `&` | `&` | `&` | С плагином |
| Миксины | `@mixin` / `@include` | `.mixin()` | `mixin()` | С плагинами |
| Условия | `@if/@else` | `.mixin when` | `if/else` | С плагинами |
| Циклы | `@for/@each` | `.loop` | `for/in` | С плагинами |
| Функции | `@function` | `.function()` | `function()` | С плагинами |
| Операции | `+`, `-`, `*`, `/` | `+`, `-`, `*`, `/` | `+`, `-`, `*`, `/` | С плагинами |
| Скорость компиляции | Быстрая | Быстрая | Быстрая | Зависит от плагинов |
| Сообщество | Большое | Большое | Среднее | Очень большое |

### Производительность и особенности

```scss
// Пример оптимизации в Sass
// Использование %-placeholders для уменьшения дублирования
%sprite-icon {
  display: inline-block;
  background-image: url('sprite.png');
  background-repeat: no-repeat;
}

.icon-home {
  @extend %sprite-icon;
  background-position: 0 0;
  width: 16px;
  height: 16px;
}

.icon-user {
  @extend %sprite-icon;
  background-position: -16px 0;
  width: 16px;
  height: 16px;
}

// Использование @content для продвинутых миксинов
@mixin media-breakpoint-up($name, $breakpoints: (sm: 576px, md: 768px, lg: 992px, xl: 1200px)) {
  $min: map-get($breakpoints, $name);
  @if $min {
    @media (min-width: $min) {
      @content;
    }
  }
}

.element {
  @include media-breakpoint-up(md) {
    display: block;
  }
  
  @include media-breakpoint-up(lg) {
    display: flex;
  }
}
```

## Лучшие практики использования препроцессоров

### 1. Структура проекта

```scss
// Пример структуры для Sass
styles/
├── abstracts/
│   ├── _variables.scss
│   ├── _mixins.scss
│   └── _functions.scss
├── base/
│   ├── _reset.scss
│   ├── _typography.scss
│   └── _base.scss
├── components/
│   ├── _buttons.scss
│   ├── _cards.scss
│   └── _forms.scss
├── layout/
│   ├── _header.scss
│   ├── _footer.scss
│   └── _grid.scss
├── pages/
│   ├── _home.scss
│   └── _about.scss
├── themes/
│   └── _dark.scss
└── main.scss
```

### 2. main.scss

```scss
// main.scss - главный файл
@import 'abstracts/variables';
@import 'abstracts/mixins';
@import 'abstracts/functions';

@import 'base/reset';
@import 'base/typography';
@import 'base/base';

@import 'layout/grid';
@import 'layout/header';
@import 'layout/footer';

@import 'components/buttons';
@import 'components/cards';
@import 'components/forms';

@import 'pages/home';
@import 'pages/about';

@import 'themes/dark';
```

### 3. Организация переменных

```scss
// _variables.scss
// Цвета
$colors: (
  primary: #007bff,
  secondary: #6c757d,
  success: #28a745,
  info: #17a2b8,
  warning: #ffc107,
  danger: #dc3545,
  light: #f8f9fa,
  dark: #343a40
);

// Извлечение цветов
@function color($key) {
  @return map-get($colors, $key);
}

// Отступы
$spacers: (
  0: 0,
  1: 0.25rem,
  2: 0.5rem,
  3: 1rem,
  4: 1.5rem,
  5: 3rem
);

@function spacer($size) {
  @return map-get($spacers, $size);
}

// Шрифты
$font-family-sans-serif: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
$font-family-monospace: SFMono-Regular, Menlo, Monaco, Consolas, 'Liberation Mono';

// Размеры шрифтов
$font-sizes: (
  xs: 0.75rem,
  sm: 0.875rem,
  base: 1rem,
  lg: 1.25rem,
  xl: 1.5rem,
  '2xl': 1.75rem,
  '3xl': 2rem
);

// Брейкпоинты
$grid-breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px,
  xxl: 1400px
);
```

### 4. Продвинутые миксины

```scss
// _mixins.scss
// Адаптивные шрифты
@mixin fluid-type($min-vw, $max-vw, $min-font-size, $max-font-size) {
  $u1: unit($min-vw);
  $u2: unit($max-vw);
  $u3: unit($min-font-size);
  $u4: unit($max-font-size);

  @if $u1 == $u2 and $u1 == $u3 and $u1 == $u4 {
    & {
      font-size: $min-font-size;
      
      @media screen and (min-width: $min-vw) {
        font-size: calc(#{$min-font-size} + #{strip-unit($max-font-size - $min-font-size)} * ((100vw - #{$min-vw}) / #{strip-unit($max-vw - $min-vw)}));
      }
      
      @media screen and (min-width: $max-vw) {
        font-size: $max-font-size;
      }
    }
  }
}

// Утилита для удаления единиц измерения
@function strip-unit($number) {
  @if type-of($number) == 'number' and not unitless($number) {
    @return $number / ($number * 0 + 1);
  }
  
  @return $number;
}

// Миксин для адаптивных отступов
@mixin responsive-spacing($property, $spacer-map, $breakpoints: $grid-breakpoints) {
  @each $size, $length in $spacer-map {
    .#{$property}-#{$size} { #{$property}: $length; }
    .#{$property}t-#{$size} { #{$property}-top: $length; }
    .#{$property}r-#{$size} { #{$property}-right: $length; }
    .#{$property}b-#{$property}-#{$size} { #{$property}-bottom: $length; }
    .#{$property}l-#{$size} { #{$property}-left: $length; }
    .#{$property}x-#{$size} { #{$property}-left: $length; #{$property}-right: $length; }
    .#{$property}y-#{$size} { #{$property}-top: $length; #{$property}-bottom: $length; }
    
    @each $name, $breakpoint in $breakpoints {
      @media (min-width: $breakpoint) {
        .#{$name}\:#{$property}-#{$size} { #{$property}: $length; }
        .#{$name}\:#{$property}t-#{$size} { #{$property}-top: $length; }
        .#{$name}\:#{$property}r-#{$size} { #{$property}-right: $length; }
        .#{$name}\:#{$property}b-#{$size} { #{$property}-bottom: $length; }
        .#{$name}\:#{$property}l-#{$size} { #{$property}-left: $length; }
        .#{$name}\:#{$property}x-#{$size} { #{$property}-left: $length; #{$property}-right: $length; }
        .#{$name}\:#{$property}y-#{$size} { #{$property}-top: $length; #{$property}-bottom: $length; }
      }
    }
  }
}
```

## Современные тенденции

### 1. Использование CSS Custom Properties с препроцессорами

```scss
// Комбинация Sass переменных и CSS Custom Properties
:root {
  --primary-color: #{$primary-color};
  --secondary-color: #{$secondary-color};
  --font-size-base: #{$font-size-base};
  --border-radius: #{$border-radius};
}

// Использование CSS Custom Properties в сочетании с Sass
.button {
  background-color: var(--primary-color, #3498db);
  border-radius: var(--border-radius, 4px);
  font-size: var(--font-size-base, 16px);
  
  // Sass логика для генерации тем
  @each $theme, $color in $theme-colors {
    &--#{$theme} {
      --primary-color: #{$color};
    }
  }
}
```

### 2. Интеграция с современными фреймворками

```scss
// Пример для Tailwind CSS с кастомными плагинами
@tailwind base;
@tailwind components;
@tailwind utilities;

// Кастомные компоненты
@layer components {
  .btn {
    @apply px-4 py-2 rounded font-medium transition-colors;
  }
  
  .btn-primary {
    @apply btn bg-blue-500 text-white hover:bg-blue-600;
  }
  
  .btn-secondary {
    @apply btn bg-gray-500 text-white hover:bg-gray-600;
  }
}
```

## Примеры из промышленной разработки

### 1. Система компонентов на Sass

```scss
// _button-system.scss
// Система кнопок с использованием Sass
$button-variants: (
  primary: (bg: #007bff, text: #fff, hover: #0056b3),
  secondary: (bg: #6c757d, text: #fff, hover: #545b62),
  success: (bg: #28a745, text: #fff, hover: #1e7e34),
  danger: (bg: #dc3545, text: #fff, hover: #c82333)
);

$button-sizes: (
  sm: (padding: 0.25rem 0.5rem, font-size: 0.875rem),
  md: (padding: 0.375rem 0.75rem, font-size: 1rem),
  lg: (padding: 0.5rem 1rem, font-size: 1.25rem)
);

// Миксин для генерации кнопок
@mixin generate-buttons() {
  .btn {
    display: inline-block;
    font-weight: 400;
    text-align: center;
    vertical-align: middle;
    user-select: none;
    border: 1px solid transparent;
    transition: all 0.15s ease-in-out;
    
    @each $variant, $colors in $button-variants {
      &--#{$variant} {
        background-color: map-get(map-get($button-variants, $variant), bg);
        color: map-get(map-get($button-variants, $variant), text);
        
        &:hover {
          background-color: map-get(map-get($button-variants, $variant), hover);
          border-color: map-get(map-get($button-variants, $variant), hover);
        }
      }
    }
    
    @each $size, $props in $button-sizes {
      &--#{$size} {
        padding: map-get(map-get($button-sizes, $size), padding);
        font-size: map-get(map-get($button-sizes, $size), font-size);
      }
    }
  }
}

@include generate-buttons();
```

### 2. Адаптивная сетка

```scss
// _grid-system.scss
$grid-columns: 12;
$grid-gutter: 30px;
$breakpoints: (
  xs: 0,
  sm: 576px,
  md: 768px,
  lg: 992px,
  xl: 1200px,
  xxl: 1400px
);

// Миксин для генерации сетки
@mixin make-grid-columns($columns: $grid-columns, $gutter: $grid-gutter, $breakpoints: $breakpoints) {
  .col {
    flex-basis: 0;
    flex-grow: 1;
    max-width: 100%;
    padding-right: calc($gutter / 2);
    padding-left: calc($gutter / 2);
  }

  @each $breakpoint, $value in $breakpoints {
    @media (min-width: $value) {
      .col-#{$breakpoint} {
        flex-basis: 0;
        flex-grow: 1;
        max-width: 100%;
        padding-right: calc($gutter / 2);
        padding-left: calc($gutter / 2);
      }

      @for $i from 1 through $columns {
        .col-#{$breakpoint}-#{$i} {
          flex: 0 0 percentage($i / $columns);
          max-width: percentage($i / $columns);
        }
      }
    }
  }
}

@include make-grid-columns();
```

## Ссылки на связанные темы
- [[best-practices/best-practices]] - Лучшие практики CSS
- [[methodology/methodology]] - Методологии CSS
- [[performance/performance]] - Производительность CSS
- [[build-tools]] - Инструменты сборки

#programming #css #preprocessors #sass #less #stylus #postcss #best-practices