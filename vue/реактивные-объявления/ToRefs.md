---
aliases: [Преобразование реактивного объекта в refs, toRefs в Vue]
tags: [vue, реактивность, toRefs, javascript]
---

# ToRefs

`toRefs` - это утилита в Vue 3, которая преобразует реактивный объект в набор ref'ов, где каждый ref представляет собой свойство исходного объекта. Это позволяет деструктурировать реактивный объект, сохраняя реактивность каждого свойства.

## Основное использование

```javascript
import { reactive, toRefs } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue',
  items: []
})

// Преобразуем реактивный объект в refs
const { count, name, items } = toRefs(state)

console.log(count.value) // 0
count.value++ // Изменение отражается в исходном объекте
console.log(state.count) // 1
```

## Проблема деструктуризации

Без `toRefs` деструктуризация теряет реактивность:

```javascript
import { reactive } from 'vue'

const state = reactive({ count: 0 })

// ПЛОХО - теряется реактивность
const { count } = state
count++ // Не вызывает реактивное обновление

// ХОРОШО - сохраняется реактивность
const countRef = state.count // Но не позволяет изменять
```

## Пример в компоненте

```vue
<template>
  <div>
    <p>Счетчик: {{ count }}</p>
    <p>Имя: {{ name }}</p>
    <button @click="increment">Увеличить</button>
  </div>
</template>

<script setup>
import { reactive, toRefs } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue 3'
})

// Используем toRefs для сохранения реактивности
const { count, name } = toRefs(state)

const increment = () => {
  state.count++ // или count.value++
}
</script>
```

## Совместимость с шаблоном

При использовании `toRefs` в шаблоне значения автоматически разворачиваются:

```vue
<template>
  <!-- Не нужно использовать .value в шаблоне -->
  <div>{{ count }} - {{ name }}</div>
</template>
```

## Работа с computed

`toRefs` можно использовать с вычисляемыми свойствами:

```javascript
import { reactive, computed, toRefs } from 'vue'

const state = reactive({
  firstName: 'John',
  lastName: 'Doe'
})

const fullName = computed(() => `${state.firstName} ${state.lastName}`)

// Добавляем computed в refs
const refs = {
  ...toRefs(state),
  fullName
}
```

## Использование с функциями

Полезно при передаче свойств в другие функции:

```javascript
import { reactive, toRefs } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue'
})

const { count, name } = toRefs(state)

function useCounter(counterRef) {
  const increment = () => {
    counterRef.value++
  }
  
  return { increment }
}

const { increment } = useCounter(count)
```

## Связь с toRef

`toRefs` работает с целым объектом, в то время как `toRef` создает ref для отдельного свойства:

```javascript
import { reactive, toRefs, toRef } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// toRefs - для всех свойств
const allRefs = toRefs(state)

// toRef - для отдельного свойства
const countRef = toRef(state, 'count')
```

## Практические рекомендации

- Используйте `toRefs` при деструктуризации реактивных объектов
- Особенно полезно в Composition API для возврата объекта из хука
- Позволяет передавать отдельные свойства в другие функции с сохранением реактивности
- Упрощает работу с шаблонами, позволяя использовать значения напрямую

## Возвращение из хуков

Часто используется в пользовательских хуках:

```javascript
import { reactive, toRefs } from 'vue'

function useUser() {
  const user = reactive({
    name: '',
    email: '',
    isLoggedIn: false
  })
  
  const login = () => {
    user.isLoggedIn = true
  }
  
  // Возвращаем как refs для сохранения реактивности при деструктуризации
  return {
    ...toRefs(user),
    login
  }
}

// Использование
const { name, email, isLoggedIn, login } = useUser()
```

## См. также

- [[Reactive]]
- [[Ref]]
- [[Readonly]]
- [[IsRef]]