# beforeUnmount

## Определение
`beforeUnmount` - это хук жизненного цикла Vue, который вызывается сразу перед тем, как экземпляр компонента будет уничтожен и удален из DOM. Это последний момент, когда компонент все еще полностью функционален.

## Использование

### В Options API
```javascript
export default {
  data() {
    return {
      timer: null,
      eventListener: null
    }
  },
  
  mounted() {
    // Установка таймера
    this.timer = setInterval(() => {
      console.log('Тик')
    }, 1000)
    
    // Добавление обработчика события
    this.eventListener = () => console.log('Событие')
    window.addEventListener('custom-event', this.eventListener)
  },
  
  beforeUnmount() {
    console.log('Компонент будет размонтирован')
    
    // Очистка таймера
    if (this.timer) {
      clearInterval(this.timer)
    }
    
    // Удаление обработчика события
    window.removeEventListener('custom-event', this.eventListener)
  }
}
```

### В Composition API
```javascript
import { onMounted, onBeforeUnmount, ref } from 'vue'

export default {
  setup() {
    const timer = ref(null)
    
    onMounted(() => {
      timer.value = setInterval(() => {
        console.log('Тик')
      }, 1000)
    })
    
    onBeforeUnmount(() => {
      console.log('Компонент будет размонтирован')
      
      if (timer.value) {
        clearInterval(timer.value)
      }
    })
  }
}
```

## Обычные задачи в beforeUnmount

### Очистка таймеров
```javascript
let intervalId = null

onMounted(() => {
  intervalId = setInterval(() => {
    // выполняется каждую секунду
  }, 1000)
})

onBeforeUnmount(() => {
  if (intervalId) {
    clearInterval(intervalId)
  }
})
```

### Удаление обработчиков событий
```javascript
const handleResize = () => {
  console.log('Окно изменено')
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
})
```

### Отписка от WebSocket или других соединений
```javascript
let ws = null

onMounted(() => {
  ws = new WebSocket('ws://example.com')
})

onBeforeUnmount(() => {
  if (ws) {
    ws.close()
  }
})
```

### Уничтожение сторонних библиотек
```javascript
let chart = null

onMounted(() => {
  const ctx = document.getElementById('myChart')
  chart = new Chart(ctx, {
    type: 'bar',
    data: chartData
  })
})

onBeforeUnmount(() => {
  if (chart) {
    chart.destroy()
  }
})
```

## Важные особенности

### Последний момент доступа
- Все компонентные функции и данные все еще доступны
- Можно безопасно обращаться к `this` в Options API
- Можно безопасно обращаться к реактивным значениям в Composition API

### Синхронные операции
```javascript
onBeforeUnmount(() => {
  // Синхронные операции
  localStorage.setItem('componentState', JSON.stringify(state))
  
  // Асинхронные операции не гарантируют выполнения
  // await someAsyncOperation() // не рекомендуется
})
```

## beforeUnmount vs unmounted

| Хук | Время вызова | Практическое применение |
|-----|--------------|------------------------|
| `beforeUnmount` | Перед размонтированием | Очистка ресурсов |
| `unmounted` | После размонтирования | Финальные действия |

## Связь с другими концепциями
- [[lifecycle/mounted]] - начальная инициализация
- [[lifecycle/unmounted]] - завершающий хук
- [[composables/Composables]] - управление эффектами в композаблах
- [[reactivity/Reactivity System]] - отписка от реактивных эффектов

## Примечания
- Используйте для очистки ресурсов
- Подписки и таймеры должны быть отменены
- Это лучшее место для очистки эффектов