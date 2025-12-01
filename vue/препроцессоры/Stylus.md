---
aliases: ["Stylus", "Vue Stylus", "CSS Preprocessor Stylus"]
tags: ["#vue", "#css-preprocessors", "#stylus", "#frontend", "#styling"]
---

# Stylus в Vue.js

## Общее описание

**Stylus** - это мощный, гибкий и экспрессивный CSS-препроцессор, разработанный TJ Holowaychuk (бывшим членом команды Express.js). Stylus отличается от других препроцессоров своей гибкой синтаксической структурой, которая может быть как похожей на CSS (с фигурными скобками и точками с запятой), так и использовать отступы для определения вложенности (как в SASS), а также может обходиться без них вообще.

Stylus предоставляет все основные возможности CSS-препроцессоров: переменные, миксины, вложенность, функции, операции и условные выражения, но с более гибким и кратким синтаксисом.

## Использование в Vue.js

### Установка зависимостей

Для использования Stylus в проекте Vue.js необходимо установить соответствующий препроцессор:

```bash
# Для npm
npm install -D stylus stylus-loader

# Для yarn
yarn add -D stylus stylus-loader

# Для pnpm
pnpm add -D stylus stylus-loader
```

### Пример использования в компоненте Vue

```vue
<template>
  <div class="container">
    <h1 class="title">Пример Stylus в Vue</h1>
    <p class="description">Этот текст использует стили, определенные в Stylus</p>
  </div>
</template>

<script setup>
// Скрипт компонента
</script>

<style lang="stylus" scoped>
// Переменные
primary-color = #42b883
secondary-color = #35495e
text-color = #333
font-size = 16px

// Миксин для адаптивности
responsive-font(size)
  font-size size
  @media (max-width: 768px)
    font-size size * 0.85

.container
  padding 20px
  background-color lighten(primary-color, 45%)
  border-radius 8px
  
  .title
    responsive-font(1.5rem)
    color primary-color
    margin-bottom 10px
  
  .description
    responsive-font(font-size)
    color text-color
    line-height 1.6
</style>
```

## Основные возможности Stylus в Vue

### 1. Переменные

Stylus позволяет определять переменные несколькими способами:

```stylus
// С присваиванием
primary-color = #42b883
font-size-base = 16px
border-radius = 4px

// Без присваивания (альтернативный синтаксис)
secondary-color #35495e
padding-size 10px

// Использование переменных
.button
  background-color primary-color
  font-size font-size-base
  border-radius border-radius
  padding padding-size
```

### 2. Вложенные правила

Stylus поддерживает вложенные правила с использованием отступов:

```stylus
.navbar
  background-color #fff
  
  .nav-item
    display inline-block
    
    .nav-link
      text-decoration none
      
      &:hover
        color primary-color
```

### 3. Миксины (Mixins)

Миксины позволяют создавать многократно используемые блоки стилей:

```stylus
// Определение миксина
button-style(bg-color, text-color)
  background-color bg-color
  color text-color
  border none
  padding 10px 20px
  border-radius 4px
  cursor pointer
  
  &:hover
    opacity 0.8

// Использование миксина
.primary-button
  button-style(#42b883, #fff)

.secondary-button
  button-style(#35495e, #fff)
```

### 4. Операции

Stylus позволяет выполнять математические операции с единицами измерения:

```stylus
.container
  width 100% - 20px
  margin 10px + 5px
  padding 2 * 10px
```

### 5. Функции

Stylus предоставляет встроенные функции и позволяет создавать пользовательские:

```stylus
// Использование встроенных функций
.element
  background-color lighten(#35495e, 20%)
  border-color saturate(#42b883, 30%)
  width percentage(5/12) // преобразует в проценты

// Создание собственной функции
calculate-rem(size)
  return (size / 16px) * 1rem

.text
  font-size calculate-rem(18px)
```

### 6. Условные выражения

Stylus поддерживает условные выражения:

```stylus
button-style(type = 'primary')
  if type == 'primary'
    background-color #42b883
    color #fff
  else if type == 'secondary'
    background-color #35495e
    color #fff
  else
    background-color #f0f0f0
    color #333

.primary-btn
  button-style('primary')

.secondary-btn
  button-style('secondary')
```

## Продвинутые возможности Stylus

### 1. Циклы

Stylus поддерживает циклы для генерации стилей:

```stylus
// Генерация размеров
for num in (1..5)
  .col-{num}
    width (num * 20)%

// Генерация цветов
colors = primary #42b883, secondary #35495e, accent #f0f0f0

for name, color in colors
  .{name}-color
    color color
    background-color lighten(color, 40%)
```

### 2. Работа с цветами

Stylus предоставляет расширенные возможности для работы с цветами:

```stylus
.base-color = #42b883

.color-examples
  color-base .base-color
  color-lighten lighten(.base-color, 20%)
  color-darken darken(.base-color, 20%)
  color-saturate saturate(.base-color, 30%)
  color-desaturate desaturate(.base-color, 30%)
  color-adjust-hue adjust-hue(.base-color, 90deg)
  color-rgba rgba(.base-color, 0.5)
```

### 3. Селекторы и расширения

Stylus поддерживает расширение селекторов:

```stylus
.message
  padding 10px
  border-radius 5px
  font-weight bold

.success
  @extend .message
  background-color #dff0d8
  color #3c763d

.error
  @extend .message
  background-color #f2dede
  color #a94442
```

## Синтаксические особенности Stylus

### 1. Различные синтаксические стили

Stylus поддерживает несколько синтаксических стилей:

```stylus
/* Стиль 1: Сокращенный синтаксис с отступами */
body
  font 14px/1.5 "Helvetica Neue", Arial, sans-serif
  color #333

/* Стиль 2: С CSS-подобным синтаксисом */
.container {
  width: 100%;
  padding: 20px;
  background: #fff;
}

/* Стиль 3: С опциональными двоеточиями */
.button
  display: block
  width: 100px
  height: 40px
```

### 2. Интерполяция

Stylus поддерживает интерполяцию переменных и выражений:

```stylus
prefix = 'my-app'

.{prefix}-header
  background-color #42b883

// Использование в свойствах
create-gradient(color)
  background linear-gradient(to bottom, color, darken(color, 20%))

.gradient-box
  create-gradient(#42b883)
```

## Практические рекомендации

### 1. Структура файлов стилей

Рекомендуется использовать следующую структуру для организации Stylus-файлов в проекте Vue:

```
src/
├── styles/
│   ├── abstracts/
│   │   ├── _variables.styl
│   │   ├── _mixins.styl
│   │   └── _functions.styl
│   ├── base/
│   │   ├── _reset.styl
│   │   ├── _typography.styl
│   │   └── _helpers.styl
│   ├── components/
│   │   ├── _buttons.styl
│   │   └── _forms.styl
│   ├── layout/
│   │   ├── _header.styl
│   │   ├── _footer.styl
│   │   └── _sidebar.styl
│   └── main.styl
```

### 2. Глобальные стили

Для подключения глобальных стилей с использованием Stylus в проекте Vue:

```javascript
// main.js
import './styles/main.styl'
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

Файл `main.styl` может импортировать все необходимые Stylus-файлы:

```stylus
// styles/main.styl
@import 'abstracts/variables'
@import 'abstracts/mixins'
@import 'abstracts/functions'
@import 'base/reset'
@import 'base/typography'
@import 'components/buttons'
@import 'layout/header'
```

### 3. Компонентные стили

Для компонентов рекомендуется использовать scoped стили:

```vue
<template>
  <div class="my-component">
    <h2>Заголовок компонента</h2>
    <p>Содержимое компонента</p>
  </div>
</template>

<style lang="stylus" scoped>
primary-color = #42b883

.my-component
  padding 20px
  border 1px solid #ddd
  border-radius 4px
  
  h2
    color primary-color
    margin-top 0
  
  p
    line-height 1.6
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

## Сравнение с другими препроцессорами

| Характеристика | Stylus | SCSS/SASS | LESS | PostCSS |
|----------------|--------|-----------|------|---------|
| Синтаксис | Гибкий, может быть разным | Похож на CSS | Похож на CSS | CSS-совместимый |
| Переменные | variable = value | $variable | @variable | Плагины |
| Вложенность | Поддерживается | Поддерживается | Поддерживается | Плагины |
| Миксины | Поддерживается | Поддерживается | Поддерживается | Плагины |
| Компиляция | Быстрая | Средняя | Средняя | Зависит от плагинов |
| Экосистема | Низкая | Высокая | Средняя | Высокая |

## Рекомендации по производительности

### 1. Избегайте чрезмерной вложенности

Старайтесь не использовать более 3-4 уровней вложенности, чтобы не создавать слишком специфичные селекторы:

```stylus
// Плохо - слишком глубокая вложенность
.component
  .section
    .block
      .item
        .element
          // ...

// Лучше - используйте классы для конкретных элементов
.component
  .component-section { /* ... */ }
  .component-block { /* ... */ }
  .component-item { /* ... */ }
```

### 2. Оптимизация импортов

Импортируйте только необходимые миксины и переменные, чтобы избежать лишнего кода в финальной сборке:

```stylus
// Вместо импорта всего файла абстракций
@import 'abstracts'

// Импортируйте только необходимые элементы
@import 'abstracts/variables'
@import 'abstracts/mixins'
```

## Связанные темы

- [[Vue.js стилизация]]
- [[SCSS-SASS]]
- [[LESS]]
- [[CSS Modules]]
- [[PostCSS]]
- [[Vue компоненты]]
- [[Vue scoped стили]]

## Источники и дополнительные ресурсы

- [Официальная документация Stylus](https://stylus-lang.com/)
- [Руководство по Stylus в Vue](https://vuejs.org/guide/scaling-up/tooling.html#css-pre-processors)
- [Stylus Documentation (EN)](https://stylus-lang.com/docs/)
- [Vue Style Guide](https://vuejs.org/style-guide/)

## Ключевые теги

#vue #css-preprocessors #stylus #frontend #styling #web-development #russian-frontend #vue-components #component-styling