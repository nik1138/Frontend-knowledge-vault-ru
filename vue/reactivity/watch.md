# watch

## Определение
`watch` - это функция в Composition API Vue 3, которая позволяет отслеживать изменения реактивных значений и выполнять побочные эффекты при их изменении. Является аналогом опции `watch` в Options API.

## Синтаксис
```javascript
import { watch } from 'vue'

watch(source, callback, options?)
```

## Параметры
- `source` - источник для отслеживания (ref, reactive, getter функция или массив)
- `callback` - функция, вызываемая при изменении источника
- `options` - опции конфигурации (optional)

## Примеры использования

### Отслеживание ref
```javascript
import { ref, watch } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    watch(count, (newCount, oldCount) => {
      console.log(`Счетчик изменился с ${oldCount} на ${newCount}`)
    })
    
    return { count }
  }
}
```

### Отслеживание reactive объекта
```javascript
import { reactive, watch } from 'vue'

export default {
  setup() {
    const state = reactive({ count: 0 })
    
    // Отслеживание всего объекта
    watch(() => state.count, (newCount, oldCount) => {
      console.log(`Счетчик изменился: ${newCount}`)
    })
    
    // Или отслеживание всего объекта (глубокое наблюдение)
    watch(state, () => {
      console.log('Состояние изменилось')
    })
    
    return { state }
  }
}
```

### Отслеживание нескольких источников
```javascript
import { ref, watch } from 'vue'

export default {
  setup() {
    const firstName = ref('')
    const lastName = ref('')
    
    watch([firstName, lastName], ([newFirstName, newLastName], [oldFirstName, oldLastName]) => {
      console.log(`Имя изменилось: ${oldFirstName} -> ${newFirstName}`)
      console.log(`Фамилия изменилась: ${oldLastName} -> ${newLastName}`)
    })
    
    return { firstName, lastName }
  }
}
```

## Опции
- `immediate: true` - выполнить callback сразу при создании
- `deep: true` - глубокое отслеживание объектов
- `flush: 'pre' | 'post' | 'sync'` - момент выполнения эффекта

### Пример с опциями
```javascript
watch(
  () => state.obj,
  (newObj, oldObj) => {
    console.log('Объект изменился')
  },
  { deep: true, immediate: false }
)
```

## Возвращаемая функция
`watch` возвращает функцию для остановки отслеживания:

```javascript
const stopWatching = watch(count, callback)

// Позже
stopWatching() // прекращает наблюдение
```

## Сравнение с watchEffect
- `watch` позволяет явно указать, что отслеживать
- `watch` не вызывает эффект сразу при создании (без опции `immediate`)
- `watchEffect` автоматически отслеживает зависимости

## Связь с другими концепциями
- [[watchEffect]] - автоматическое отслеживание зависимостей
- [[ref]] - отслеживание реактивных ссылок
- [[reactive]] - отслеживание реактивных объектов
- [[computed]] - вычисляемые значения

## Примечания
- Используйте `watch` когда нужно выполнить побочные эффекты при изменении
- Для асинхронных операций подходит лучше чем `computed`
- Не забывайте останавливать watch при необходимости