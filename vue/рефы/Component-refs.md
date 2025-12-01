---
aliases: ["Рефы компонентов", "Refs компонентов"]
---

# Component-refs

## Обзор

Component refs позволяют получить доступ к экземпляру дочернего компонента из родительского компонента. Это полезно для вызова методов дочернего компонента или доступа к его свойствам.

## Создание Component Ref

Для создания component ref используется директива `ref` на компоненте:

```vue
<template>
  <ChildComponent ref="child" />
  <button @click="callChildMethod">Вызвать метод дочернего компонента</button>
</template>

<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const child = ref(null)

const callChildMethod = () => {
  child.value.childMethod()
}
</script>
```

## В дочернем компоненте

Для того чтобы родительский компонент мог получить доступ к методам дочернего, нужно использовать `defineExpose`:

```vue
<script setup>
const childMethod = () => {
  console.log('Метод дочернего компонента вызван!')
}

defineExpose({
  childMethod
})
</script>
```

## Особенности работы с Component Refs

- Компоненты по умолчанию не экспортируют свои внутренние свойства
- Используйте `defineExpose` для явного экспорта свойств/методов
- В Options API доступ к методам осуществляется через `this.$refs.child`

## Связанные темы

- [[Вызов-методов-компонента]]
- [[Template-refs]]
- [[Кастомные-рефы]]

#vue #refs #frontend #composition-api #options-api #components