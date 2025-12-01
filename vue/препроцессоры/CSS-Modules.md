---
aliases: ["CSS Modules", "Vue CSS Modules", "CSS Modules в Vue"]
tags: ["#vue", "#css-modules", "#frontend", "#styling", "#component-scoped-styles"]
---

# CSS Modules в Vue.js

## Общее описание

**CSS Modules** - это подход к стилизации компонентов, при котором CSS-классы автоматически ограничиваются областью видимости компонента. В отличие от препроцессоров, CSS Modules не добавляют новые возможности к CSS, а обеспечивают изоляцию стилей на уровне модуля, предотвращая конфликты имен классов в глобальной области видимости.

CSS Modules особенно полезны в крупных приложениях Vue.js, где важно избежать конфликтов стилей между различными компонентами.

## Использование в Vue.js

### Настройка CSS Modules

В проектах Vue.js CSS Modules включаются автоматически при использовании `lang="css"` без атрибута `scoped` и с определенной конфигурацией. В большинстве случаев, CSS Modules включаются через настройку сборки:

```javascript
// vite.config.js (для Vite)
export default {
  css: {
    modules: {
      localsConvention: 'camelCase' // позволяет использовать camelCase в JS
    }
  }
}
```

```javascript
// webpack.config.js (для Webpack)
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]__[local]___[hash:base64:5]',
                localsConvention: 'camelCase'
              }
            }
          }
        ]
      }
    ]
  }
}
```

### Пример использования в компоненте Vue

```vue
<template>
  <div :class="$style.container">
    <h1 :class="$style.title">Пример CSS Modules в Vue</h1>
    <p :class="[$style.description, $style.highlight]">Этот текст использует стили из CSS Modules</p>
  </div>
</template>

<script setup>
// Можно импортировать стили как объект
import styles from './MyComponent.module.css'

// Использовать в скрипте
console.log(styles.container) // уникальное имя класса
</script>

<style module>
.container {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
}

.title {
  font-size: 1.5rem;
  color: #42b883;
  margin-bottom: 10px;
}

.description {
  font-size: 16px;
  color: #333;
  line-height: 1.6;
}

.highlight {
  background-color: #fff9c4;
  padding: 2px 4px;
  border-radius: 3px;
}
</style>
```

## Основные возможности CSS Modules в Vue

### 1. Изоляция стилей

Каждый класс в CSS Module автоматически получает уникальное имя, предотвращая конфликты:

```vue
<style module>
.button {
  background-color: #42b883;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}
</style>
```

После компиляции `.button` станет чем-то вроде `.MyComponent_button__123abc`, уникального для компонента.

### 2. Использование в шаблоне

Классы из CSS Module доступны через `$style`:

```vue
<template>
  <!-- Использование одного класса -->
  <div :class="$style.container">Содержимое</div>
  
  <!-- Использование нескольких классов -->
  <div :class="[$style.container, $style.active]">Активный элемент</div>
  
  <!-- Условное применение классов -->
  <div :class="{ [$style.container]: isActive, [$style.inactive]: !isActive }">Условный стиль</div>
</template>
```

### 3. Композиция классов

CSS Modules поддерживают композицию классов:

```vue
<style module>
.baseButton {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primaryButton {
  composes: baseButton;
  background-color: #42b883;
  color: white;
}

.secondaryButton {
  composes: baseButton;
  background-color: #35495e;
  color: white;
}
</style>
```

### 4. Глобальные стили

Иногда необходимо использовать глобальные стили в модуле:

```vue
<style module>
/* Локальный класс */
.container {
  padding: 20px;
}

/* Глобальный класс (не будет переименован) */
:global(.global-class) {
  color: red;
}

/* Глобальный селектор */
:global {
  .some-global-style {
    margin: 0;
  }
}
</style>
```

## Практические рекомендации

### 1. Структура файлов

Для использования CSS Modules в Vue.js можно использовать два подхода:

**Подход 1: Встроенные стили в компоненте (рекомендуется)**
```vue
<template>
  <div :class="$style.container">...</div>
</template>

<style module>
/* Стили компонента */
</style>
```

**Подход 2: Отдельные файлы стилей**
```
MyComponent.vue
MyComponent.module.css
```

```vue
<!-- MyComponent.vue -->
<template>
  <div :class="styles.container">...</div>
</template>

<script setup>
import styles from './MyComponent.module.css'
</script>
```

### 2. Именование классов

Рекомендуется использовать понятные имена классов, так как они будут переименованы в уникальные:

```vue
<style module>
/* Хорошо: понятные имена */
.header { /* ... */ }
.navigationItem { /* ... */ }
.userProfileCard { /* ... */ }

/* Плохо: непонятные сокращения */
.h { /* ... */ }
.n { /* ... */ }
.upc { /* ... */ }
</style>
```

### 3. Работа с динамическими классами

При использовании динамических классов с CSS Modules:

```vue
<template>
  <!-- Правильно: использование computed свойства -->
  <div :class="computedClasses">Элемент</div>
  
  <!-- Правильно: использование метода -->
  <div :class="getClassNames(type)">Элемент</div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps(['isActive', 'type'])

// Использование computed для комбинации классов
const computedClasses = computed(() => ({
  [$style.container]: true,
  [$style.active]: props.isActive,
  [$style.disabled]: !props.isActive
}))

// Использование метода для динамического выбора класса
const getClassNames = (type) => {
  const baseClass = $style.component
  const typeClass = $style[`component--${type}`]
  return [baseClass, typeClass]
}
</script>
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

CSS Modules могут немного увеличить размер сборки из-за генерации уникальных имен классов, поэтому:

- Используйте оптимизацию сборки
- Удаляйте неиспользуемые стили
- Рассмотрите использование Tree Shaking для стилей

## Сравнение с другими подходами

| Характеристика | CSS Modules | SCSS/SASS | LESS | Stylus | Scoped стили |
|----------------|-------------|-----------|------|--------|--------------|
| Изоляция стилей | Высокая | Нет | Нет | Нет | Высокая |
| Переменные | CSS-переменные | Препроцессорные | Препроцессорные | Препроцессорные | Нет |
| Вложенность | Нет | Поддерживается | Поддерживается | Поддерживается | Нет |
| Миксины | Нет | Поддерживается | Поддерживается | Поддерживается | Нет |
| Компиляция | Да | Да | Да | Да | Нет |
| Экосистема | Средняя | Высокая | Средняя | Низкая | Высокая |

## Рекомендации по производительности

### 1. Минимизация уникальных имен

CSS Modules генерируют уникальные имена классов, что может увеличить размер CSS:

```vue
<!-- Плохо: слишком много уникальных классов -->
<div :class="$style.veryLongComponentNameForHeader">...</div>
<div :class="$style.veryLongComponentNameForSubHeader">...</div>
<div :class="$style.veryLongComponentNameForMainContent">...</div>

<!-- Лучше: более короткие имена -->
<div :class="$style.header">...</div>
<div :class="$style.subHeader">...</div>
<div :class="$style.content">...</div>
```

### 2. Использование композиции

Используйте композицию для переиспользования стилей:

```vue
<style module>
/* Базовые стили */
.baseButton {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* Специфические стили */
.primaryButton {
  composes: baseButton;
  background-color: #42b883;
  color: white;
}

.secondaryButton {
  composes: baseButton;
  background-color: #35495e;
  color: white;
}
</style>
```

## Связанные темы

- [[Vue.js стилизация]]
- [[SCSS-SASS]]
- [[LESS]]
- [[Stylus]]
- [[PostCSS]]
- [[Vue компоненты]]
- [[Vue scoped стили]]
- [[CSS-in-JS]]

## Источники и дополнительные ресурсы

- [Официальная документация CSS Modules](https://github.com/css-modules/css-modules)
- [Руководство по CSS Modules в Vue](https://vuejs.org/api/sfc-css-features.html#css-modules)
- [Vue Style Guide](https://vuejs.org/style-guide/)
- [CSS Modules Specification](https://github.com/css-modules/icss)

## Ключевые теги

#vue #css-modules #frontend #styling #web-development #russian-frontend #vue-components #component-styling #scoped-styles