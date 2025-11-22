---
aliases: []
tags: [vue, components, communication]
---

# Props and Events

Props и Events - это основные механизмы коммуникации между компонентами в Vue.js. Props используются для передачи данных от родительского компонента к дочернему, а Events - для передачи информации от дочернего компонента к родительскому.

## Props

Props - это пользовательские атрибуты, которые можно зарегистрировать в компоненте. Когда значение передается в атрибут пропса, оно становится свойством этого экземпляра компонента.

### Определение Props

```javascript
export default {
  props: {
    title: String,
    likes: Number,
    isActive: {
      type: Boolean,
      default: true
    },
    callback: Function
  }
}
```

### Использование Props в шаблоне

```vue
<template>
  <h1>{{ title }}</h1>
  <p>Нравится: {{ likes }}</p>
  <button v-if="isActive" @click="callback">Нажми меня</button>
</template>
```

### Передача Props родительским компонентом

```vue
<template>
  <child-component 
    :title="postTitle" 
    :likes="postLikes"
    :is-active="postActive"
    :callback="handleCallback"
  />
</template>
```

## Events (Emit)

Events позволяют дочерним компонентам передавать информацию родительским компонентам.

### Генерация событий в дочернем компоненте

```javascript
export default {
  methods: {
    notifyParent() {
      this.$emit('custom-event', 'some data')
    }
  }
}
```

### Использование Composition API

```javascript
import { defineEmits } from 'vue'

const emit = defineEmits(['response', 'update'])

function submitForm() {
  emit('response', { success: true })
}
```

### Обработка событий в родительском компоненте

```vue
<template>
  <child-component @custom-event="handleCustomEvent" />
</template>

<script>
export default {
  methods: {
    handleCustomEvent(data) {
      console.log('Полученные данные:', data)
    }
  }
}
</script>
```

## Валидация Props

Vue предоставляет возможность валидации props:

```javascript
props: {
  height: {
    type: Number,
    default: 100
  },
  name: {
    type: String,
    required: true
  },
  callback: {
    type: Function,
    default: () => {}
  },
  alignment: {
    validator: value => {
      return ['left', 'right', 'center'].includes(value)
    }
  }
}
```

## Лучшие практики

1. Используйте понятные имена для props и событий
2. Всегда определяйте типы props
3. Указывайте значения по умолчанию
4. Используйте kebab-case для имен событий при передаче в шаблоне
5. Используйте camelCase для имен событий в JavaScript

См. также: [[Vue компоненты]], [[Single File Components]], [[Slots and Scoped Slots]]