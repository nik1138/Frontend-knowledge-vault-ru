# Events

## Определение
Events (события) - это механизм в Vue, позволяющий дочерним компонентам отправлять сообщения родительским компонентам. Это основной способ передачи данных из дочернего компонента в родительский.

## Использование событий

### В Options API
```vue
<!-- ChildComponent.vue -->
<template>
  <button @click="notifyParent">Нажми меня</button>
</template>

<script>
export default {
  methods: {
    notifyParent() {
      // Отправка события родителю
      this.$emit('custom-event', 'some data')
    }
  }
}
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent @custom-event="handleEvent" />
</template>

<script>
import ChildComponent from './ChildComponent.vue'

export default {
  components: {
    ChildComponent
  },
  methods: {
    handleEvent(data) {
      console.log('Получены данные:', data)
    }
  }
}
</script>
```

### В Composition API
```vue
<!-- ChildComponent.vue -->
<template>
  <button @click="notifyParent">Нажми меня</button>
</template>

<script setup>
// Определение отправляемых событий
const emit = defineEmits(['custom-event', 'another-event'])

const notifyParent = () => {
  emit('custom-event', 'some data')
}
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent @custom-event="handleEvent" />
</template>

<script setup>
import ChildComponent from './ChildComponent.vue'

const handleEvent = (data) => {
  console.log('Получены данные:', data)
}
</script>
```

## Валидация событий
```javascript
// В Composition API можно указать типы событий
const emit = defineEmits({
  'custom-event': (data) => {
    // Валидация данных события
    if (typeof data !== 'string') {
      console.warn('Ожидается строка')
      return false
    }
    return true
  }
})
```

## Специальные события

### Двусторонняя привязка (v-model)
```vue
<!-- ChildComponent.vue -->
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent v-model="value" />
</template>
```

### Несколько v-model
```vue
<ChildComponent 
  v-model:first-name="firstName" 
  v-model:last-name="lastName" 
/>
```

## Связь с другими концепциями
- [[props]] - для передачи данных в другом направлении
- [[reactivity/Reactivity System]] - управление реактивностью
- [[composables/Composables]] - альтернатива для сложной связи компонентов

## Примечания
- События - основной способ коммуникации от дочерних к родительским компонентам
- Используйте v-model для удобной двусторонней связи
- Валидация событий полезна для публичных компонентов