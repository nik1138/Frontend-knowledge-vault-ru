---
aliases: ["CSS препроцессоры", "Препроцессоры CSS"]
tags: 
  - css
  - preprocessors
  - sass
  - less
  - stylus
  - frontend
---

# Глубокое погружение в CSS препроцессоры

CSS препроцессоры — это надмножества CSS, которые добавляют дополнительные возможности, недоступные в обычном CSS. Они компилируются в обычный CSS для использования в браузерах.

## Основные препроцессоры

### Sass (SCSS и indented syntax)

Sass (Syntactically Awesome Style Sheets) — один из самых популярных CSS препроцессоров. Имеет два синтаксиса:

- **SCSS** — синтаксис, похожий на CSS с фигурными скобками
- **Indented syntax** — отступы вместо скобок

```scss
// SCSS
$primary-color: #3498db;

.container {
  background-color: $primary-color;
  
  .header {
    color: darken($primary-color, 20%);
  }
}
```

```sass
// Indented syntax
$primary-color: #3498db

.container
  background-color: $primary-color
  
  .header
    color: darken($primary-color, 20%)
```

[[css/sass-basics]] [[css/sass-advanced]]

### LESS

LESS — препроцессор, похожий на Sass, но с собственными особенностями:

```less
@primary-color: #3498db;

.container {
  background-color: @primary-color;
  
  .header {
    color: darken(@primary-color, 20%);
  }
}
```

[[css/less-introduction]]

### Stylus

Stylus — гибкий препроцессор с возможностью различных стилей написания:

```stylus
primary-color = #3498db

.container
  background-color: primary-color
  
  .header
    color: darken(primary-color, 20%)
```

## Основные возможности

### Переменные и область видимости

Все препроцессоры поддерживают переменные, но с разным синтаксисом:

```scss
// SCSS
$global-color: #333;
.container {
  $local-color: #fff; // локальная переменная
  color: $global-color;
  background: $local-color;
}
```

[[css/variables-best-practices]]

### Миксины и функции

Миксины позволяют создавать переиспользуемые блоки кода:

```scss
// SCSS миксин
@mixin button-style($bg-color) {
  background-color: $bg-color;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
}

.btn-primary {
  @include button-style(#007bff);
}
```

Функции возвращают значения:

```scss
@function calculate-opacity($color, $alpha) {
  @return rgba($color, $alpha);
}

.element {
  color: calculate-opacity(#333, 0.8);
}
```

### Наследование и расширение

```scss
%button-base {
  border: none;
  padding: 10px;
}

.btn-primary {
  @extend %button-base;
  background-color: #007bff;
}
```

### Вложенные правила

```scss
.navbar {
  background: #333;
  
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
    
    li { 
      display: inline-block;
      
      a {
        display: block;
        padding: 10px;
        text-decoration: none;
      }
    }
  }
}
```

### Условия и циклы

```scss
// Условия
@if $theme == dark {
  .element { color: white; }
} @else {
  .element { color: black; }
}

// Циклы
@for $i from 1 through 5 {
  .col-#{$i} {
    width: 20% * $i;
  }
}
```

## Модули и импорты

Препроцессоры поддерживают импорт файлов для организации кода:

```scss
// _variables.scss
$primary-color: #007bff;

// main.scss
@import 'variables';
@import 'components/buttons';
```

[[css/modular-css]]

## Компиляция и инструменты

### Процесс компиляции

Препроцессоры компилируются в обычный CSS:

```
SCSS файл -> Препроцессор -> CSS файл
```

### Интеграция с инструментами

#### Webpack

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: ['style-loader', 'css-loader', 'sass-loader']
      }
    ]
  }
};
```

#### Vite

```javascript
// vite.config.js
export default {
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "src/styles/variables.scss";`
      }
    }
  }
};
```

[[build-tools/webpack]] [[build-tools/vite]]

## Отладка и производительность

### Source maps

Source maps позволяют отлаживать исходные препроцессорные файлы:

```scss
// Включить в настройках компиляции
sass input.scss output.css --sourcemap
```

### Производительность

- Избегайте чрезмерного вложения
- Оптимизируйте использование миксинов
- Минимизируйте дублирование кода

## Сравнение препроцессоров

| Особенность | Sass | LESS | Stylus |
|-------------|------|------|--------|
| Синтаксис | SCSS/Indented | CSS-подобный | Гибкий |
| Переменные | `$` | `@` | `$` или `@` |
| Миксины | `@mixin` | `.mixin()` | `()` |
| Расширение | `@extend` | `:extend()` | `@extends` |

## Заключение

CSS препроцессоры значительно упрощают написание и поддержку CSS кода, добавляя программные концепции в стили. Выбор препроцессора зависит от проектных требований и предпочтений команды.

[[css/best-practices]] [[css/architecture]]