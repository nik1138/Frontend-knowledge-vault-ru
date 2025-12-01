---
aliases: ["SCSS", "SASS", "Vue SCSS", "Vue SASS"]
tags: ["#vue", "#css-preprocessors", "#scss", "#sass", "#frontend", "#styling"]
---

# SCSS/SASS в Vue.js

## Общее описание

**SCSS** (Sassy CSS) и **SASS** (Syntactically Awesome Style Sheets) - это два синтаксиса одного и того же CSS-препроцессора, который добавляет мощные возможности к стандартному CSS. В контексте Vue.js они позволяют использовать переменные, вложенные правила, миксины, наследование селекторов и другие возможности, что делает стилизацию компонентов более гибкой и поддерживаемой.

### Основные отличия SCSS и SASS

- **SCSS** использует синтаксис, похожий на обычный CSS, но с дополнительными возможностями. Файлы имеют расширение `.scss`
- **SASS** использует отступы для определения вложенности и не требует фигурных скобок и точек с запятой. Файлы имеют расширение `.sass`

В проектах Vue.js чаще всего используется SCSS из-за его схожести с CSS и лучшей читаемости.

## Использование в Vue.js

### Установка зависимостей

Для использования SCSS/SASS в проекте Vue.js необходимо установить соответствующий препроцессор:

```bash
# Для npm
npm install -D sass

# Для yarn
yarn add -D sass

# Для pnpm
pnpm add -D sass
```

### Пример использования в компоненте Vue

```vue
<template>
  <div class="container">
    <h1 class="title">Пример SCSS в Vue</h1>
    <p class="description">Этот текст использует стили, определенные в SCSS</p>
  </div>
</template>

<script setup>
// Скрипт компонента
</script>

<style lang="scss" scoped>
// Переменные
$primary-color: #42b883;
$secondary-color: #35495e;
$text-color: #333;
$font-size: 16px;

// Миксин для адаптивности
@mixin responsive-font($size) {
  font-size: $size;
  @media (max-width: 768px) {
    font-size: $size * 0.85;
  }
}

.container {
  padding: 20px;
  background-color: lighten($primary-color, 45%);
  border-radius: 8px;
  
  .title {
    @include responsive-font(1.5rem);
    color: $primary-color;
    margin-bottom: 10px;
  }
  
  .description {
    @include responsive-font($font-size);
    color: $text-color;
    line-height: 1.6;
  }
}
</style>
```

## Основные возможности SCSS/SASS в Vue

### 1. Переменные

Переменные позволяют хранить значения (цвета, размеры, шрифты и т.д.), которые можно переиспользовать по всему проекту:

```scss
// Определение переменных
$primary-color: #42b883;
$font-size-base: 16px;
$border-radius: 4px;

// Использование переменных
.button {
  background-color: $primary-color;
  font-size: $font-size-base;
  border-radius: $border-radius;
}
```

### 2. Вложенные правила

SCSS позволяет вкладывать CSS-правила друг в друга, что делает код более структурированным:

```scss
.navbar {
  background-color: #fff;
  
  .nav-item {
    display: inline-block;
    
    .nav-link {
      text-decoration: none;
      
      &:hover {
        color: $primary-color;
      }
    }
  }
}
```

### 3. Миксины (Mixins)

Миксины позволяют создавать многократно используемые блоки стилей:

```scss
// Определение миксина
@mixin button-style($bg-color, $text-color) {
  background-color: $bg-color;
  color: $text-color;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
}

// Использование миксина
.primary-button {
  @include button-style(#42b883, #fff);
}

.secondary-button {
  @include button-style(#35495e, #fff);
}
```

### 4. Функции

SCSS предоставляет встроенные функции и позволяет создавать свои:

```scss
// Использование встроенных функций
.element {
  background-color: lighten(#35495e, 20%);
  border-color: desaturate(#42b883, 30%);
  width: percentage(5/12); // преобразует в проценты
}

// Создание собственной функции
@function calculate-rem($size) {
  @return ($size / 16px) * 1rem;
}

.text {
  font-size: calculate-rem(18px);
}
```

### 5. Операции

SCSS позволяет выполнять математические операции с единицами измерения:

```scss
.container {
  width: 100% - 20px;
  margin: 10px + 5px;
  padding: 2 * 10px;
}
```

## Практические рекомендации

### 1. Структура файлов стилей

Рекомендуется использовать следующую структуру для организации SCSS-файлов в проекте Vue:

```
src/
├── styles/
│   ├── abstracts/
│   │   ├── _variables.scss
│   │   ├── _mixins.scss
│   │   └── _functions.scss
│   ├── base/
│   │   ├── _reset.scss
│   │   ├── _typography.scss
│   │   └── _helpers.scss
│   ├── components/
│   │   ├── _buttons.scss
│   │   └── _forms.scss
│   ├── layout/
│   │   ├── _header.scss
│   │   ├── _footer.scss
│   │   └── _sidebar.scss
│   └── main.scss
```

### 2. Глобальные стили

Для подключения глобальных стилей с использованием SCSS в проекте Vue:

```javascript
// main.js
import './styles/main.scss'
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

Файл `main.scss` может импортировать все необходимые SCSS-файлы:

```scss
// styles/main.scss
@import 'abstracts/variables';
@import 'abstracts/mixins';
@import 'abstracts/functions';
@import 'base/reset';
@import 'base/typography';
@import 'components/buttons';
@import 'layout/header';
```

### 3. Компонентные стили

Для компонентов рекомендуется использовать scoped стили или CSS-модули:

```vue
<template>
  <div class="my-component">
    <h2>Заголовок компонента</h2>
    <p>Содержимое компонента</p>
  </div>
</template>

<style lang="scss" scoped>
.my-component {
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
  
  h2 {
    color: $primary-color;
    margin-top: 0;
  }
  
  p {
    line-height: 1.6;
  }
}
</style>
```

## Особенности использования в России в 2025 году

### 1. Альтернативные репозитории и зеркала

В связи с геополитической ситуацией и возможными ограничениями доступа к npm-репозиториям, разработчики в России могут использовать альтернативные зеркала и частные репозитории:

- Использование локальных npm-репозиториев (например, Nexus, Artifactory)
- Настройка npm/yarn для работы с российскими зеркалами
- Использование pnpm как более быстрой альтернативы

### 2. Совместимость с отечественными решениями

При разработке для российского рынка может потребоваться совместимость с отечественными браузерами и системами:

- Учет требований к веб-сайтам по ГОСТ Р 59853-2022 (доступность)
- Поддержка отечественных браузеров (Яндекс.Браузер, Mail.Ru Browser, Astra Browser)
- Использование шрифтов, доступных в российских системах

### 3. Оптимизация производительности

В условиях возможных ограничений на доступ к внешним ресурсам:

- Минимизация внешних зависимостей
- Использование локальных шрифтов и иконок
- Оптимизация размера сборки

## Рекомендации по производительности

### 1. Избегайте чрезмерной вложенности

Старайтесь не использовать более 3-4 уровней вложенности, чтобы не создавать слишком специфичные селекторы:

```scss
// Плохо - слишком глубокая вложенность
.component {
  .section {
    .block {
      .item {
        .element {
          // ...
        }
      }
    }
  }
}

// Лучше - используйте классы для конкретных элементов
.component {
  .component-section { /* ... */ }
  .component-block { /* ... */ }
  .component-item { /* ... */ }
}
```

### 2. Оптимизация импортов

Импортируйте только необходимые миксины и переменные, чтобы избежать лишнего кода в финальной сборке:

```scss
// Вместо импорта всего файла абстракций
@import 'abstracts';

// Импортируйте только необходимые элементы
@import 'abstracts/variables';
@import 'abstracts/mixins';
```

## Связанные темы

- [[Vue.js стилизация]]
- [[CSS Modules]]
- [[PostCSS]]
- [[LESS]]
- [[Stylus]]
- [[Vue компоненты]]
- [[Vue scoped стили]]

## Источники и дополнительные ресурсы

- [Официальная документация SASS](https://sass-lang.com/)
- [Руководство по SCSS в Vue](https://vuejs.org/guide/scaling-up/tooling.html#css-pre-processors)
- [Sass Guidelines (EN)](https://sass-guidelin.es/)
- [Vue Style Guide](https://vuejs.org/style-guide/)

## Ключевые теги

#vue #css-preprocessors #scss #sass #frontend #styling #web-development #russian-frontend #vue-components #component-styling