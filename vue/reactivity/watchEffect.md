# watchEffect

## Определение
`watchEffect` - это функция в Composition API Vue 3, которая немедленно выполняет переданную ей функцию и отслеживает реактивные зависимости автоматически. Выполняет побочный эффект при изменении зависимостей.

## Синтаксис
```javascript
import { watchEffect } from 'vue'

watchEffect(effect, options?)
```

## Особенности
- Автоматически отслеживает зависимости при первом выполнении
- Немедленно выполняется при создании
- Перезапускается при изменении отслеживаемых зависимостей
- Не нуждается в явном указании источника для отслеживания

## Примеры использования

### Простой пример
```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const count = ref(0)
    
    watchEffect(() => {
      console.log(`Счетчик: ${count.value}`)
    })
    
    return { count }
  }
}
```

### Асинхронные операции
```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const searchQuery = ref('')
    const results = ref([])
    
    watchEffect(async () => {
      if (searchQuery.value) {
        results.value = await fetchData(searchQuery.value)
      }
    })
    
    return { searchQuery, results }
  }
}
```

### Отмена побочных эффектов
```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const id = ref(1)
    const data = ref(null)
    
    watchEffect((onInvalidate) => {
      let cancelled = false
      
      const promise = fetchData(id.value)
      
      onInvalidate(() => {
        cancelled = true
      })
      
      promise.then(result => {
        if (!cancelled) {
          data.value = result
        }
      })
    })
    
    return { id, data }
  }
}
```

## Опции
- `flush: 'pre' | 'post' | 'sync'` - момент выполнения эффекта
- `onTrack` - колбэк для отладки (при отслеживании зависимости)
- `onTrigger` - колбэк для отладки (при срабатывании эффекта)

### Пример с опциями
```javascript
watchEffect(
  () => {
    console.log('Эффект выполнен')
  },
  {
    flush: 'post', // выполнить после обновления DOM
    onTrack: (event) => console.log('Отслеживается:', event),
    onTrigger: (event) => console.log('Сработал:', event)
  }
)
```

## Возвращаемая функция
`watchEffect` возвращает функцию для остановки отслеживания:

```javascript
const stopEffect = watchEffect(() => {
  console.log(count.value)
})

// Позже
stopEffect() // прекращает наблюдение
```

## Сравнение с watch
- `watchEffect` автоматически отслеживает зависимости
- `watch` требует явного указания источника
- `watchEffect` запускается сразу при создании
- `watch` запускается только при изменении источника (без immediate)

## Связь с другими концепциями
- [[watch]] - отслеживание конкретных источников
- [[ref]] - реактивные зависимости
- [[reactive]] - реактивные объекты
- [[computed]] - вычисляемые значения

## Примечания
- Используйте `watchEffect` для автоматического отслеживания зависимостей
- Хорош для асинхронных побочных эффектов
- Для отладки используйте колбэки `onTrack` и `onTrigger`