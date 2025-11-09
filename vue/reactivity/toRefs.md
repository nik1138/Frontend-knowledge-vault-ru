# toRefs

## Определение
`toRefs` - это утилитарная функция в Vue 3 Composition API, которая преобразует реактивный объект в набор ref'ов, где каждый ref представляет собой свойство исходного объекта. Это позволяет деструктурировать реактивный объект, сохраняя реактивность.

## Синтаксис
```javascript
import { toRefs } from 'vue'

const refs = toRefs(reactiveObject)
```

## Проблема, которую решает
Без `toRefs` деструктуризация реактивного объекта теряет реактивность:

```javascript
import { reactive } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// Так реактивность теряется!
const { count, name } = state 

// С toRefs реактивность сохраняется
const { count, name } = toRefs(state)
```

## Примеры использования

### Базовое использование
```javascript
import { reactive, toRefs } from 'vue'

export default {
  setup() {
    const state = reactive({
      count: 0,
      name: 'Vue',
      items: []
    })
    
    // Сохраняем реактивность при деструктуризации
    const { count, name, items } = toRefs(state)
    
    const increment = () => {
      count.value++
    }
    
    return {
      count, // ref
      name,  // ref
      items, // ref
      increment
    }
  }
}
```

### В шаблоне
```vue
<template>
  <div>
    <p>Счетчик: {{ count }}</p>
    <p>Название: {{ name }}</p>
    <button @click="increment">Увеличить</button>
  </div>
</template>
```

### С computed значениями
```javascript
import { reactive, toRefs, computed } from 'vue'

export default {
  setup() {
    const state = reactive({
      firstName: 'John',
      lastName: 'Doe'
    })
    
    const fullName = computed(() => `${state.firstName} ${state.lastName}`)
    
    return {
      ...toRefs(state),
      fullName
    }
  }
}
```

## toRefs vs toRef
- `toRefs` - преобразует все свойства объекта в refs
- `toRef` - создает один ref из одного свойства объекта

```javascript
import { reactive, toRefs, toRef } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// toRefs: все свойства
const { count, name } = toRefs(state)

// toRef: одно свойство
const countRef = toRef(state, 'count')
```

## Внутренняя реализация
`toRefs` создает ref для каждого свойства, используя getter/setter для синхронизации с исходным объектом:

```javascript
// Упрощенная реализация
function toRefs(object) {
  const result = {}
  for (const key in object) {
    result[key] = toRef(object, key)
  }
  return result
}
```

## Связь с другими концепциями
- [[reactive]] - исходный реактивный объект
- [[ref]] - результат преобразования
- [[toRef]] - преобразование одного свойства
- [[setup]] - использование в Composition API

## Примечания
- `toRefs` сохраняет реактивность при деструктуризации
- Полезно для передачи свойств в компоненты
- Работает только с реактивными объектами