---
aliases: []
tags: [vue, components, sfc]
---

# Single File Components

Single File Components (SFC) - это уникальная особенность Vue.js, позволяющая объединять шаблон, скрипт и стили в одном файле с расширением `.vue`. Этот подход делает компоненты более автономными и легкими для понимания, поскольку все аспекты компонента находятся в одном месте.

## Структура SFC

Каждый файл `.vue` состоит из трех возможных секций:

```vue
<template>
  <!-- HTML-шаблон компонента -->
  <div class="my-component">
    {{ message }}
  </div>
</template>

<script>
// Логика компонента
export default {
  data() {
    return {
      message: 'Hello Vue!'
    }
  }
}
</script>

<style>
/* Стили компонента */
.my-component {
  color: blue;
}
</style>
```

## Атрибуты секций

Секции могут иметь дополнительные атрибуты:

- `lang` - указывает язык для секции (например, `<template lang="pug">`)
- `scoped` - ограничивает стиль только этим компонентом
- `module` - позволяет использовать CSS-модули

## Scoped CSS

Атрибут `scoped` автоматически добавляет уникальный атрибут к элементам компонента и изменяет селекторы CSS, чтобы они применялись только к элементам этого компонента:

```vue
<style scoped>
.example {
  color: red;
}
</style>
```

## Слоты и SFC

SFC поддерживает использование слотов для создания более гибких компонентов:

```vue
<template>
  <div class="card">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

## Асинхронные компоненты

SFC поддерживают ленивую загрузку через асинхронные компоненты:

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./MyComponent.vue')
)
```

## TypeScript в SFC

SFC могут использовать TypeScript для строгой типизации:

```vue
<script setup lang="ts">
interface Props {
  msg: string
}

const props = defineProps<Props>()
</script>
```

См. также: [[Vue компоненты]], [[Props and Events]], [[Slots and Scoped Slots]]