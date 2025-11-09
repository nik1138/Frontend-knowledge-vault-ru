# mounted

## Определение
`mounted` - это хук жизненного цикла Vue, который вызывается после того, как экземпляр компонента был смонтирован и DOM обновлен. В этот момент компонент полностью подготовлен к использованию и имеет доступ к DOM.

## Использование

### В Options API
```javascript
export default {
  data() {
    return {
      message: 'Привет, мир!'
    }
  },
  
  mounted() {
    // Доступ к DOM элементам
    console.log('Компонент смонтирован')
    console.log(this.$el) // DOM элемент компонента
    
    // Пример работы с DOM
    const element = this.$el.querySelector('.my-element')
    element.focus()
  }
}
```

### В Composition API
```javascript
import { onMounted, ref } from 'vue'

export default {
  setup() {
    const message = ref('Привет, мир!')
    
    onMounted(() => {
      console.log('Компонент смонтирован')
      console.log('Сообщение:', message.value)
      
      // Работа с DOM
      const element = document.querySelector('.my-element')
      if (element) {
        element.focus()
      }
    })
    
    return {
      message
    }
  }
}
```

## Обычные задачи в mounted

### Инициализация сторонних библиотек
```javascript
onMounted(() => {
  // Инициализация Chart.js, например
  const ctx = document.getElementById('myChart')
  new Chart(ctx, {
    type: 'bar',
    data: chartData
  })
})
```

### Запросы к API
```javascript
import { ref, onMounted } from 'vue'

const data = ref(null)

onMounted(async () => {
  try {
    const response = await fetch('/api/data')
    data.value = await response.json()
  } catch (error) {
    console.error('Ошибка загрузки данных:', error)
  }
})
```

### Установка фокуса
```javascript
onMounted(() => {
  const input = document.querySelector('input[type="text"]')
  if (input) {
    input.focus()
  }
})
```

### Подписка на события
```javascript
onMounted(() => {
  window.addEventListener('resize', handleResize)
})
```

## Важные особенности

### Доступ к DOM
- В `mounted` DOM уже обновлен и доступен
- Можно безопасно взаимодействовать с элементами DOM
- `$el` доступен в Options API

### Асинхронные операции
```javascript
onMounted(async () => {
  // Можно использовать async/await
  await initializeComponent()
  console.log('Компонент полностью инициализирован')
})
```

### Ограничения
- Не вызывается при серверном рендеринге (SSR)
- Для SSR используйте `onMounted` с проверками

## Сравнение с другими хуками

| Хук | Время вызова |
|-----|--------------|
| `created` | После создания экземпляра, до монтирования |
| `mounted` | После монтирования в DOM |
| `updated` | После обновления компонента |

## Связь с другими концепциями
- [[lifecycle/beforeMount]] - состояние до монтирования
- [[lifecycle/updated]] - обновление после изменений
- [[lifecycle/beforeUnmount]] - очистка ресурсов
- [[reactivity/Reactivity System]] - работа с реактивными данными

## Примечания
- Используйте `mounted` для инициализации после монтирования
- Не забудьте отписываться от событий в `beforeUnmount`
- Подходит для взаимодействия с DOM