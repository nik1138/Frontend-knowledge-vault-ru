# shallowReactive

## Определение
`shallowReactive` - это функция в Composition API Vue 3, которая создает поверхностно реактивный объект. В отличие от `reactive`, `shallowReactive` делает реактивными только корневые свойства объекта, оставляя вложенные объекты не реактивными.

## Синтаксис
```javascript
import { shallowReactive } from 'vue'

const state = shallowReactive({
  // данные объекта
})
```

## Особенности
- Реактивны только корневые свойства
- Вложенные объекты остаются не реактивными
- Производительность лучше, чем у `reactive` для глубоких структур
- Подходит для объектов с фиксированной структурой

## Примеры использования

### Сравнение с reactive
```javascript
import { reactive, shallowReactive } from 'vue'

// reactive: глубокая реактивность
const deepState = reactive({
  count: 0,
  user: {
    name: 'John',
    age: 30
  }
})

// shallowReactive: поверхностная реактивность
const shallowState = shallowReactive({
  count: 0,
  user: {
    name: 'John',
    age: 30
  }
})

// Это вызовет обновление реактивности
deepState.count++
shallowState.count++

// Это вызовет обновление для deepState, но не для shallowState
deepState.user.name = 'Jane' // Реактивно
shallowState.user.name = 'Jane' // Не реактивно!
```

### Когда использовать
```javascript
import { shallowReactive } from 'vue'

export default {
  setup() {
    // Когда структура объекта известна и не будет меняться
    const state = shallowReactive({
      title: 'My App',
      config: {
        apiUrl: 'https://api.example.com',
        theme: 'dark'
      }
    })
    
    return { state }
  }
}
```

### Сравнение с ref
```javascript
import { ref, shallowReactive, reactive } from 'vue'

// ref с объектом (поверхностная реактивность)
const objRef = ref({ name: 'Vue' })
objRef.value.name = 'Changed' // Не реактивно!

// shallowReactive (поверхностная реактивность, но без .value)
const shallowObj = shallowReactive({ name: 'Vue' })
shallowObj.name = 'Changed'  // Реактивно!

// reactive (глубокая реактивность)
const deepObj = reactive({ name: 'Vue' })
deepObj.name = 'Changed'     // Реактивно!
```

## Преимущества
- Лучшая производительность для глубоких структур данных
- Подходит для фиксированных структур
- Меньше "оверхеда" по сравнению с `reactive`

## Недостатки
- Не реагирует на изменения вложенных объектов
- Ограниченная реактивность

## Связь с другими концепциями
- [[reactive]] - глубокая реактивность
- [[shallowRef]] - поверхностная реактивность для refs
- [[ref]] - реактивные ссылки
- [[isReactive]] - проверка реактивности

## Практические применения
- Конфигурационные объекты
- Объекты с известной структурой
- Кэширование данных, где вложенные объекты не меняются
- Оптимизация производительности при работе с большими объектами

## Примечания
- Используйте, когда не нужно отслеживать изменения вложенных объектов
- Подходит для объектов с фиксированной структурой
- Может улучшить производительность в некоторых случаях