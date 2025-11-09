# Props

## Определение
Props (свойства) - это пользовательские атрибуты, которые можно зарегистрировать в компоненте. Когда значение передается в атрибуте в родительском компоненте, оно становится свойством этого компонента.

## Объявление Props

### В Options API
```vue
<template>
  <h1>{{ title }}</h1>
  <p>{{ likes }}</p>
</template>

<script>
export default {
  props: ['title', 'likes'],
  // или с типами
  props: {
    title: String,
    likes: {
      type: Number,
      default: 0
    }
  }
}
</script>
```

### В Composition API
```vue
<template>
  <h1>{{ title }}</h1>
  <p>{{ likes }}</p>
</template>

<script setup>
// Простое объявление
const props = defineProps(['title', 'likes'])

// С типами (TypeScript)
interface Props {
  title?: string
  likes?: number
}

withDefaults(defineProps<Props>(), {
  likes: 0
})
</script>
```

## Типы Props
- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol
- Custom constructor

### Пример с валидацией
```javascript
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  likes: {
    type: Number,
    default: 0
  },
  status: {
    type: String,
    default: 'draft',
    validator: (value) => {
      return ['draft', 'published', 'archived'].includes(value)
    }
  }
})
```

## Особенности передачи данных

### Односторонний поток данных
Props следуют принципу "одностороннего потока данных":
- Изменения в родительском свойстве автоматически обновляют дочерние
- Изменения в дочернем компоненте НЕ влияют на родительский

### Изменение Props
В дочернем компоненте не рекомендуется изменять props напрямую. Вместо этого:
- Используйте `$emit` для отправки событий родителю
- Используйте `v-model` для двусторонней связи
- Создавайте локальные данные на основе props

```javascript
// Не рекомендуется
props.count = 5 // вызовет предупреждение

// Рекомендуется
const localCount = ref(props.count)
```

## Связь с другими концепциями
- [[events]] - для отправки данных родителю
- [[slots]] - альтернативный способ передачи содержимого
- [[reactivity/Reactivity System]] - работа с реактивными props

## Примечания
- Props - основной способ передачи данных сверху вниз
- Всегда используйте валидацию для публичных компонентов
- Избегайте изменения props напрямую