# Лучшие практики Vue 3 Composition API

Composition API - это мощный инструмент для организации логики в Vue.js приложениях. В этом руководстве мы рассмотрим лучшие практики и паттерны для эффективного использования Composition API.

## Введение

Composition API был представлен в Vue 3 как альтернатива Options API и предоставляет более гибкий способ организации логики компонентов. Он особенно полезен для:

- Повторного использования логики между компонентами
- Организации кода в больших компонентах
- Улучшения типизации в TypeScript
- Лучшего понимания потоков данных

## Основные принципы

### 1. Используйте композаблы для логики, специфичной для компонента

Композаблы (composables) - это функции, которые используют реактивные API Vue для инкапсуляции и переиспользования состояния и логики.

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initialValue
  
  const isEven = computed(() => count.value % 2 === 0)
  
  return {
    count,
    increment,
    decrement,
    reset,
    isEven
  }
}
```

```vue
<!-- Counter.vue -->
<template>
  <div>
    <p>Счетчик: {{ count }}</p>
    <p>Четное: {{ isEven ? 'да' : 'нет' }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="reset">Сброс</button>
  </div>
</template>

<script setup>
import { useCounter } from '@/composables/useCounter'

const { count, increment, decrement, reset, isEven } = useCounter(10)
</script>
```

### 2. Организуйте логику по функциональным группам

Вместо группировки по типу (data, methods, computed), группируйте по бизнес-логике:

```javascript
// composables/useUserManagement.js
import { ref, computed, onMounted } from 'vue'
import { getUser, updateUser, deleteUser } from '@/api/users'

export function useUserManagement(userId) {
  // Логика загрузки пользователя
  const user = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  const loadUser = async () => {
    loading.value = true
    error.value = null
    try {
      user.value = await getUser(userId)
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }
  
  // Логика обновления пользователя
  const updating = ref(false)
  const updateUserProfile = async (data) => {
    updating.value = true
    try {
      const updated = await updateUser(userId, data)
      Object.assign(user.value, updated)
    } catch (err) {
      error.value = err.message
    } finally {
      updating.value = false
    }
  }
  
  // Логика удаления пользователя
  const deleting = ref(false)
  const removeUser = async () => {
    deleting.value = true
    try {
      await deleteUser(userId)
      user.value = null
    } catch (err) {
      error.value = err.message
    } finally {
      deleting.value = false
    }
  }
  
  onMounted(() => {
    loadUser()
  })
  
  return {
    user,
    loading,
    error,
    updating,
    deleting,
    loadUser,
    updateUserProfile,
    removeUser
  }
}
```

### 3. Следуйте соглашениям об именовании

Используйте префикс `use` для всех композаблов: `useCounter`, `useApi`, `useLocalStorage`, и т.д.

### 4. Правильно обрабатывайте асинхронные операции

```javascript
// composables/useAsyncData.js
import { ref, onMounted } from 'vue'

export function useAsyncData(fetchFunction) {
  const data = ref(null)
  const loading = ref(true)
  const error = ref(null)
  
  const execute = async () => {
    loading.value = true
    error.value = null
    try {
      data.value = await fetchFunction()
    } catch (err) {
      error.value = err
    } finally {
      loading.value = false
    }
  }
  
  onMounted(() => {
    execute()
  })
  
  return {
    data,
    loading,
    error,
    execute,
    refresh: execute
  }
}
```

## Паттерны композаблов

### 1. Паттерн "Хук" (Hook Pattern)

Композабл, который работает с реактивным состоянием компонента:

```javascript
// composables/useVModel.js
export function useVModel(props, emit) {
  return {
    get value() {
      return props.modelValue
    },
    set value(value) {
      emit('update:modelValue', value)
    }
  }
}

// Использование в компоненте
// <MyInput v-model="inputValue" />
```

### 2. Паттерн "Состояние + Actions" (State + Actions Pattern)

Композабл, который инкапсулирует состояние и действия с ним:

```javascript
// composables/useForm.js
import { reactive } from 'vue'

export function useForm(initialData, validationRules) {
  const state = reactive({
    data: { ...initialData },
    errors: {},
    isValid: true,
    isSubmitting: false
  })
  
  const validate = () => {
    state.errors = {}
    let isValid = true
    
    for (const field in validationRules) {
      const rule = validationRules[field]
      const value = state.data[field]
      
      if (rule.required && !value) {
        state.errors[field] = 'Обязательное поле'
        isValid = false
      }
      
      if (rule.pattern && !rule.pattern.test(value)) {
        state.errors[field] = rule.message
        isValid = false
      }
    }
    
    state.isValid = isValid
    return isValid
  }
  
  const submit = async (onSubmit) => {
    if (!validate()) return false
    
    state.isSubmitting = true
    try {
      await onSubmit(state.data)
      return true
    } catch (error) {
      console.error('Ошибка отправки формы:', error)
      return false
    } finally {
      state.isSubmitting = false
    }
  }
  
  const updateField = (field, value) => {
    state.data[field] = value
    // Очистить ошибку при изменении поля
    if (state.errors[field]) {
      delete state.errors[field]
    }
  }
  
  return {
    state,
    validate,
    submit,
    updateField
  }
}
```

### 3. Паттерн "События" (Event Pattern)

Композабл, который управляет событиями и подписками:

```javascript
// composables/useEventListener.js
import { onMounted, onUnmounted } from 'vue'

export function useEventListener(target, event, handler, options) {
  onMounted(() => {
    target.addEventListener(event, handler, options)
  })
  
  onUnmounted(() => {
    target.removeEventListener(event, handler, options)
  })
}

// Использование
// const { window } = globalThis
// useEventListener(window, 'resize', handleResize)
```

## TypeScript с Composition API

### 1. Типизация композаблов

```typescript
// composables/useCounter.ts
import { ref, computed, Ref } from 'vue'

interface UseCounterOptions {
  initialValue?: number
  step?: number
}

interface UseCounterReturn {
  count: Ref<number>
  increment: () => void
  decrement: () => void
  reset: () => void
  isEven: Ref<boolean>
}

export function useCounter(options: UseCounterOptions = {}): UseCounterReturn {
  const {
    initialValue = 0,
    step = 1
  } = options
  
  const count = ref<number>(initialValue)
  
  const increment = () => count.value += step
  const decrement = () => count.value -= step
  const reset = () => count.value = initialValue
  
  const isEven = computed(() => count.value % 2 === 0)
  
  return {
    count,
    increment,
    decrement,
    reset,
    isEven
  }
}
```

### 2. Типизация с defineComponent

```vue
<!-- Counter.vue -->
<template>
  <div>
    <p>Счетчик: {{ counter.count }}</p>
    <p>Четное: {{ counter.isEven ? 'да' : 'нет' }}</p>
    <button @click="counter.increment">+</button>
    <button @click="counter.decrement">-</button>
    <button @click="counter.reset">Сброс</button>
  </div>
</template>

<script setup lang="ts">
import { useCounter } from '@/composables/useCounter'

const counter = useCounter({ initialValue: 10, step: 2 })
</script>
```

## Лучшие практики тестирования

### 1. Тестируйте композаблы отдельно от компонентов

```javascript
// composables/useCounter.test.js
import { describe, it, expect } from 'vitest'
import { useCounter } from '@/composables/useCounter'

describe('useCounter', () => {
  it('should initialize with correct value', () => {
    const { count } = useCounter({ initialValue: 5 })
    expect(count.value).toBe(5)
  })
  
  it('should increment counter', () => {
    const { count, increment } = useCounter({ initialValue: 0 })
    increment()
    expect(count.value).toBe(1)
  })
  
  it('should decrement counter', () => {
    const { count, decrement } = useCounter({ initialValue: 5 })
    decrement()
    expect(count.value).toBe(4)
  })
})
```

### 2. Используйте @vue/test-utils для тестирования компонентов

```javascript
// components/Counter.test.js
import { mount } from '@vue/test-utils'
import { describe, it, expect } from 'vitest'
import Counter from '@/components/Counter.vue'

describe('Counter Component', () => {
  it('should render correctly', () => {
    const wrapper = mount(Counter)
    expect(wrapper.text()).toContain('Счетчик')
  })
  
  it('should increment when button is clicked', async () => {
    const wrapper = mount(Counter)
    const button = wrapper.find('button')
    await button.trigger('click')
    expect(wrapper.text()).toContain('Счетчик: 1')
  })
})
```

## Распространенные ошибки и как их избежать

### 1. Неправильное использование реактивности

**Плохо:**
```javascript
// Установка реактивной ссылки напрямую
const { count } = useCounter()
count = ref(42) // Неправильно!
```

**Хорошо:**
```javascript
// Изменение значения реактивной ссылки
const { count } = useCounter()
count.value = 42 // Правильно
```

### 2. Забытые подписки на события

**Плохо:**
```javascript
// Утечка памяти из-за подписки на глобальные события
onMounted(() => {
  window.addEventListener('resize', handleResize)
}) // Никогда не отписываемся!
```

**Хорошо:**
```javascript
// useEventListener композабл заботится об отписке
import { useEventListener } from '@/composables/useEventListener'

const { window } = globalThis
useEventListener(window, 'resize', handleResize)
```

### 3. Неправильное управление жизненным циклом

**Плохо:**
```javascript
// Неправильный контроль асинхронных операций
const loading = ref(false)
const fetchData = async () => {
  loading.value = true
  const data = await api.getData()
  // Если компонент был разрушен, это приведет к ошибке
  someRef.value = data
}
```

**Хорошо:**
```javascript
// Проверка, что компонент все еще активен
import { onMounted, onUnmounted } from 'vue'

let cancelled = false

const fetchData = async () => {
  loading.value = true
  try {
    const data = await api.getData()
    if (!cancelled) {
      someRef.value = data
    }
  } finally {
    loading.value = false
  }
}

onUnmounted(() => {
  cancelled = true
})
```

## Когда использовать Composition API

Composition API особенно полезен в следующих случаях:

1. **Компоненты со сложной логикой** - когда у вас есть много связанной логики, которую сложно организовать в Options API

2. **Повторное использование логики** - когда вы хотите разделить логику между несколькими компонентами

3. **Типобезопасность** - TypeScript лучше работает с Composition API

4. **Компоненты, которые должны быть гибкими** - когда компонент должен адаптироваться к различным сценариям использования

5. **Работа с асинхронными данными** - когда компоненту нужно управлять несколько асинхронных операций

## Заключение

Composition API предоставляет мощные инструменты для организации и повторного использования логики в Vue.js приложениях. Следуя этим лучшим практикам, вы сможете создавать более читаемый, поддерживаемый и масштабируемый код.

Помните, что Composition API - это не замена Options API, а дополнительный инструмент. Выбирайте подходящий инструмент для конкретной задачи и не забывайте о согласованности в вашем коде.

## Связанные темы

- [[../reactivity|Vue3 Reactivity System]]
- [[useApi]]
- [[../components/index|Vue3 Components]]
- [[../lifecycle|Vue3 Lifecycle Hooks]]
- [[../../ts/generics/index|Обобщения в TypeScript]]

## Теги

#vuejs #composition-api #composables #reactivity #frontend #typescript