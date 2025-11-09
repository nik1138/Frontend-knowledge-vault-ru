# toRef

## Определение
`toRef` - это утилитарная функция в Vue 3 Composition API, которая создает реактивную ссылку (ref) для одного свойства реактивного объекта. Созданный ref синхронизируется с исходным свойством объекта.

## Синтаксис
```javascript
import { toRef } from 'vue'

const ref = toRef(reactiveObject, key)
```

## Параметры
- `source` - реактивный объект
- `key` - имя свойства, для которого нужно создать ref

## Примеры использования

### Базовое использование
```javascript
import { reactive, toRef } from 'vue'

export default {
  setup() {
    const state = reactive({
      count: 0,
      name: 'Vue'
    })
    
    // Создаем ref для конкретного свойства
    const countRef = toRef(state, 'count')
    
    const increment = () => {
      countRef.value++ // изменяет state.count
    }
    
    return {
      countRef,
      increment
    }
  }
}
```

### Синхронизация изменений
```javascript
import { reactive, toRef } from 'vue'

const state = reactive({ count: 0 })

const countRef = toRef(state, 'count')

// Изменение через ref
countRef.value = 5
console.log(state.count) // 5

// Изменение через исходный объект
state.count = 10
console.log(countRef.value) // 10
```

### Использование с примитивными значениями
```javascript
import { toRef } from 'vue'

// toRef также работает с примитивными значениями
const normalObject = { count: 0 }
const countRef = toRef(normalObject, 'count')

// Создает ref, который не является реактивным сам по себе,
// но отслеживает изменения в обычном объекте
```

### Сравнение с ref
```javascript
import { reactive, ref, toRef } from 'vue'

const state = reactive({ count: 0 })

// toRef - синхронизируется с исходным объектом
const countRef1 = toRef(state, 'count')

// ref - независимое значение
const countRef2 = ref(state.count)

state.count = 5

console.log(countRef1.value) // 5 (синхронизировано)
console.log(countRef2.value) // 0 (независимое)
```

## toRef vs toRefs
- `toRef` - создает один ref для одного свойства
- `toRefs` - создает refs для всех свойств объекта

```javascript
import { reactive, toRef, toRefs } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// toRef: одно свойство
const countRef = toRef(state, 'count')

// toRefs: все свойства
const { count, name } = toRefs(state)
```

## Практическое применение
- Передача отдельных свойств в компоненты
- Создание refs для свойств нормальных (не реактивных) объектов
- Синхронизация между различными частями приложения

```javascript
import { reactive, toRef } from 'vue'

export default {
  setup() {
    const user = reactive({ name: 'John', age: 30 })
    
    // Передаем только name в дочерний компонент
    const nameRef = toRef(user, 'name')
    
    return {
      nameRef
    }
  }
}
```

## Связь с другими концепциями
- [[ref]] - создает независимый ref
- [[toRefs]] - создает refs для всех свойств
- [[reactive]] - исходный реактивный объект
- [[unref]] - получение значения из ref

## Примечания
- `toRef` создает синхронизированный ref с исходным объектом
- Работает как с реактивными, так и с обычными объектами
- Полезно при передаче отдельных свойств в компоненты