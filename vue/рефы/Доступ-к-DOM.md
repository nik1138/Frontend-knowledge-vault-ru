---
aliases: ["Доступ к DOM", "Работа с DOM"]
---

# Доступ-к-DOM

## Обзор

Доступ к DOM в Vue осуществляется через template refs. Это позволяет напрямую взаимодействовать с элементами DOM, что может быть полезно для фокусировки, измерения размеров, анимаций и других операций, требующих прямого доступа к DOM.

## Примеры использования

### Фокусировка на элементе

```vue
<template>
  <input ref="input" />
  <button @click="focusInput">Фокус на input</button>
</template>

<script setup>
import { ref } from 'vue'

const input = ref(null)

const focusInput = () => {
  input.value.focus()
}
</script>
```

### Измерение размеров элемента

```vue
<template>
  <div ref="box" class="box">Контент</div>
  <p>Ширина: {{ width }}px, Высота: {{ height }}px</p>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const box = ref(null)
const width = ref(0)
const height = ref(0)

onMounted(() => {
  width.value = box.value.offsetWidth
  height.value = box.value.offsetHeight
})
</script>
```

## Важные моменты

- Доступ к DOM возможен только после монтирования компонента
- Используйте `onMounted` или `nextTick()` для обеспечения доступности DOM-элемента
- Избегайте прямого манипулирования DOM, когда это возможно, в пользу реактивных данных

## Связанные темы

- [[Template-refs]]
- [[Кастомные-рефы]]
- [[Вызов-методов-компонента]]

#vue #refs #frontend #dom #composition-api