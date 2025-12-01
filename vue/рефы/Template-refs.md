---
aliases: ["Шаблонные рефы", "Refs шаблона"]
---

# Template-refs

## Обзор

Template refs в Vue позволяют получить прямой доступ к элементу DOM или экземпляру компонента из шаблона. Это особенно полезно, когда нужно взаимодействовать с DOM напрямую, например, для фокусировки на input, измерения размеров элемента или вызова методов дочернего компонента.

## Создание Template Ref

Для создания template ref используется директива `ref` в шаблоне. В Composition API реф можно объявить с помощью `ref()`:

```vue
<template>
  <div ref="root">Этот div будет доступен через ref</div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const root = ref(null)

onMounted(() => {
  // Доступ к элементу DOM
  console.log(root.value) // <div>
})
</script>
```

## Использование в Options API

В Options API template refs доступны через `this.$refs`:

```vue
<template>
  <div ref="root">Этот div будет доступен через this.$refs.root</div>
</template>

<script>
export default {
  mounted() {
    console.log(this.$refs.root) // <div>
  }
}
</script>
```

## Особенности работы с Template Refs

- Рефы устанавливаются до вызова `onMounted`
- В случае с `v-for`, реф будет массивом элементов
- Для получения рефа в `setup()` используйте `onMounted` или `nextTick()`, чтобы убедиться, что DOM обновлен

## Связанные темы

- [[Доступ-к-DOM]]
- [[Component-refs]]
- [[Кастомные-рефы]]

#vue #refs #frontend #composition-api #options-api