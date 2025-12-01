---
aliases: ["watchEffect", "реактивные эффекты", "автоматические наблюдатели"]
tags: ["#vue", "#reactivity", "#watchEffect", "#side-effects", "#composition-api"]
---

# WatchEffect (Реактивные эффекты)

## Введение

`watchEffect` - это современный подход к наблюдению за реактивными данными в Vue 3, представленный в Composition API. В отличие от традиционных наблюдателей, `watchEffect` автоматически отслеживает зависимости и запускает эффект при их изменении, без необходимости явно указывать, за какими значениями наблюдать.

## Основы использования

`watchEffect` принимает функцию-эффект, которая автоматически отслеживает все реактивные зависимости, используемые внутри неё:

```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const doubleCount = ref(0)

    // watchEffect автоматически отслеживает использование count
    watchEffect(() => {
      doubleCount.value = count.value * 2
      console.log(`Счетчик: ${count.value}, удвоенное значение: ${doubleCount.value}`)
    })

    // При изменении count эффект будет запущен автоматически
    const increment = () => {
      count.value++
    }

    return {
      count,
      doubleCount,
      increment
    }
  }
}
```

## Автоматическое отслеживание зависимостей

Одним из главных преимуществ `watchEffect` является автоматическое отслеживание зависимостей. В отличие от `watch`, вам не нужно явно указывать, за какими значениями наблюдать:

```javascript
import { ref, reactive, watchEffect } from 'vue'

export default {
  setup() {
    const user = reactive({ name: 'Иван', age: 30 })
    const message = ref('')

    // watchEffect автоматически отслеживает зависимости
    watchEffect(() => {
      // Отслеживаются user.name и user.age
      message.value = `Привет, ${user.name}! Тебе ${user.age} лет.`
    })

    const updateAge = () => {
      user.age += 1 // При изменении age эффект будет запущен автоматически
    }

    return {
      message,
      updateAge
    }
  }
}
```

## Сравнение с watch

| Характеристика | watch | watchEffect |
|----------------|-------|-------------|
| Отслеживание зависимостей | Явное | Автоматическое |
| Кеширование | Нет | Нет |
| Старые и новые значения | Да | Нет |
| Гибкость | Выше | Ниже |
| Удобство использования | Для сложных сценариев | Для простых сценариев |

### Пример с watch
```javascript
watch(count, (newVal, oldVal) => {
  console.log(`Старое значение: ${oldVal}, новое значение: ${newVal}`)
})
```

### Пример с watchEffect
```javascript
watchEffect(() => {
  console.log(`Текущее значение: ${count.value}`)
})
```

## Опции и настройка

`watchEffect` поддерживает несколько опций для настройки поведения:

```javascript
watchEffect(
  () => {
    // эффект
  },
  {
    flush: 'post', // 'pre', 'post', 'sync'
    onTrack: (event) => {
      console.log('Отслеживание:', event)
    },
    onTrigger: (event) => {
      console.log('Срабатывание:', event)
    }
  }
)
```

### Режимы срабатывания (flush)

- `pre` - эффект выполняется перед обновлением DOM (по умолчанию для watchEffect)
- `post` - эффект выполняется после обновления DOM
- `sync` - эффект выполняется синхронно при изменении зависимости

```javascript
watchEffect(
  () => {
    // Будет выполнено после обновления DOM
    console.log('DOM обновлён')
  },
  { flush: 'post' }
)
```

## Остановка наблюдения

`watchEffect` возвращает функцию для остановки наблюдения:

```javascript
import { watchEffect } from 'vue'

const stop = watchEffect(() => {
  console.log('Этот эффект будет остановлен через 5 секунд')
})

// Остановка эффекта через 5 секунд
setTimeout(() => {
  stop()
  console.log('Наблюдение остановлено')
}, 5000)
```

## Практические применения

### Асинхронные операции

```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const searchQuery = ref('')
    const results = ref([])
    const loading = ref(false)

    watchEffect(async (onInvalidate) => {
      if (!searchQuery.value.trim()) {
        results.value = []
        return
      }

      loading.value = true
      
      // Функция для отмены предыдущего запроса
      let cancelled = false
      onInvalidate(() => {
        cancelled = true
      })

      try {
        const data = await fetchSearchResults(searchQuery.value)
        if (!cancelled) {
          results.value = data
        }
      } catch (error) {
        if (!cancelled) {
          console.error('Ошибка поиска:', error)
        }
      } finally {
        if (!cancelled) {
          loading.value = false
        }
      }
    })

    return {
      searchQuery,
      results,
      loading
    }
  }
}
```

### Подписка на внешние события

```javascript
import { ref, watchEffect } from 'vue'

export default {
  setup() {
    const onlineStatus = ref(navigator.onLine)

    watchEffect((onInvalidate) => {
      const updateStatus = () => {
        onlineStatus.value = navigator.onLine
      }

      window.addEventListener('online', updateStatus)
      window.addEventListener('offline', updateStatus)

      // Очистка подписки при остановке эффекта
      onInvalidate(() => {
        window.removeEventListener('online', updateStatus)
        window.removeEventListener('offline', updateStatus)
      })
    })

    return { onlineStatus }
  }
}
```

### Интеграция с внешними библиотеками

```javascript
import { ref, watchEffect } from 'vue'
import Chart from 'chart.js/auto'

export default {
  setup() {
    const chartData = ref([])
    const chartRef = ref(null)
    let chartInstance = null

    watchEffect(() => {
      if (chartRef.value && chartData.value.length) {
        if (chartInstance) {
          chartInstance.data = chartData.value
          chartInstance.update()
        } else {
          chartInstance = new Chart(chartRef.value, {
            type: 'bar',
            data: chartData.value
          })
        }
      }
    })

    // Очистка при разрушении компонента
    onUnmounted(() => {
      if (chartInstance) {
        chartInstance.destroy()
      }
    })

    return { chartRef, chartData }
  }
}
```

## Отмена операций (onInvalidate)

Функция `onInvalidate` позволяет отменять предыдущие асинхронные операции:

```javascript
watchEffect((onInvalidate) => {
  const controller = new AbortController()
  
  onInvalidate(() => {
    // Отмена предыдущего запроса при изменении зависимостей
    controller.abort()
  })

  fetch('/api/data', {
    signal: controller.signal
  }).then(response => {
    if (!controller.signal.aborted) {
      // Обработка ответа
    }
  })
})
```

## Интеграция с жизненными циклами

`watchEffect` автоматически останавливается при разрушении компонента, но можно явно управлять его жизненным циклом:

```javascript
import { watchEffect, onUnmounted } from 'vue'

export default {
  setup() {
    const stopEffects = []

    // Несколько эффектов
    stopEffects.push(
      watchEffect(() => {
        // первый эффект
      })
    )

    stopEffects.push(
      watchEffect(() => {
        // второй эффект
      })
    )

    // Явная остановка всех эффектов при разрушении
    onUnmounted(() => {
      stopEffects.forEach(stop => stop())
    })

    return {}
  }
}
```

## Сравнение с другими подходами

| Характеристика | Computed | Watch | WatchEffect |
|----------------|----------|-------|-------------|
| Назначение | Производные данные | Сайд-эффекты | Сайд-эффекты |
| Кеширование | Да | Нет | Нет |
| Отслеживание зависимостей | Автоматическое | Явное | Автоматическое |
| Старые/новые значения | Нет | Да | Нет |
| Асинхронные операции | Не рекомендуется | Да | Да |

## Оптимизация производительности

1. **Избегайте тяжелых вычислений** - `watchEffect` не кеширует результаты, поэтому тяжелые вычисления будут выполняться при каждом изменении зависимостей.

2. **Контролируйте количество зависимостей** - эффект будет запускаться при изменении любой зависимости, что может привести к избыточным срабатываниям.

3. **Используйте правильный режим flush** - для DOM-операций часто требуется `flush: 'post'`.

## Связь с другими концепциями Vue

`watchEffect` тесно связан с другими аспектами реактивности Vue:

- [[Watch]] - традиционный подход к наблюдению
- [[Computed]] - альтернатива для производных данных
- [[Реактивные-объявления]] - основа, на которой строится watchEffect
- [[Побочные-эффекты]] - общая концепция, которую реализует watchEffect

## Заключение

`watchEffect` предоставляет удобный и мощный способ выполнения побочных эффектов в ответ на изменения реактивных данных. Его автоматическое отслеживание зависимостей делает его особенно полезным для асинхронных операций и интеграции с внешними API.

При использовании `watchEffect` важно понимать его особенности и различия с другими инструментами реактивности. В сочетании с подходами, описанными в [[Лучшие-практики]] и [[Оптимизация]], `watchEffect` может значительно упростить создание реактивных приложений.

Для выбора наиболее подходящего инструмента рекомендуется использовать:
- [[Computed]] для производных данных с кешированием
- `watch` для наблюдения за конкретными значениями с доступом к старым/новым значениям
- `watchEffect` для автоматического отслеживания зависимостей и выполнения побочных эффектов