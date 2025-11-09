# v-for

## Определение
`v-for` - это дирекива в Vue, которая позволяет отображать список элементов на основе массива или объекта. Она использует специальный синтаксис: `item in items`, где `items` - исходный массив, а `item` - переменная, содержащая элемент массива.

## Использование с массивами

### Простой список
```vue
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>

<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, name: 'Яблоко' },
  { id: 2, name: 'Банан' },
  { id: 3, name: 'Апельсин' }
])
</script>
```

### С индексом
```vue
<template>
  <li v-for="(item, index) in items" :key="item.id">
    {{ index }} - {{ item.name }}
  </li>
</template>
```

### С деструктуризацией
```vue
<template>
  <li v-for="{ name, id } in items" :key="id">
    {{ name }}
  </li>
</template>
```

## Использование с объектами

```vue
<template>
  <!-- Итерация по значениям -->
  <div v-for="value in object" :key="value">
    {{ value }}
  </div>
  
  <!-- Итерация по ключу и значению -->
  <div v-for="(value, key) in object" :key="key">
    {{ key }}: {{ value }}
  </div>
  
  <!-- Итерация с индексом -->
  <div v-for="(value, key, index) in object" :key="key">
    {{ index }}. {{ key }}: {{ value }}
  </div>
</template>

<script setup>
import { ref } from 'vue'

const object = ref({
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2021-04-10'
})
</script>
</template>
```

## Использование с диапазонами

```vue
<template>
  <!-- Итерация от 1 до count -->
  <span v-for="n in count" :key="n">{{ n }}</span>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(10)
</script>
</template>
```

## Использование на элементе <template>

```vue
<template>
  <ul>
    <template v-for="item in items" :key="item.id">
      <li>{{ item.name }}</li>
      <li class="divider" aria-hidden="true"></li>
    </template>
  </ul>
</template>
```

## v-for и v-if

### Важный нюанс
`v-for` имеет более высокий приоритет, чем `v-if`, что означает:
- `v-if` будет выполняться для каждого элемента списка

```vue
<!-- v-if будет выполнено для каждого элемента -->
<li v-for="user in users" v-if="user.isActive" :key="user.id">
  {{ user.name }}
</li>
```

### Для производительности
```vue
<!-- Лучший подход - обернуть в <template> -->
<template v-for="user in users" :key="user.id">
  <li v-if="user.isActive">
    {{ user.name }}
  </li>
</template>

<!-- Или использовать отдельный вычисляемый массив -->
<template>
  <li v-for="user in activeUsers" :key="user.id">
    {{ user.name }}
  </li>
</template>

<script setup>
import { computed } from 'vue'

const users = ref([...])

// Фильтрация на уровне данных
const activeUsers = computed(() => users.value.filter(user => user.isActive))
</script>
```

## Ключи (keys)

### Важность ключей
```vue
<!-- ПЛОХО: без ключа Vue использует индексы -->
<li v-for="item in items">{{ item.name }}</li>

<!-- ХОРОШО: с уникальным ключом -->
<li v-for="item in items" :key="item.id">{{ item.name }}</li>
```

### Правила ключей
- Должны быть уникальными в контексте списка
- Предпочтительно использовать стабильные ID
- Не используйте `index` как ключ (кроме особых случаев)

```vue
<!-- ПЛОХО: использование индекса как ключа -->
<li v-for="(item, index) in items" :key="index">{{ item.name }}</li>

<!-- ХОРОШО: использование ID как ключа -->
<li v-for="item in items" :key="item.id">{{ item.name }}</li>
```

## Сортировка и фильтрация

### В template (не рекомендуется)
```vue
<!-- Не рекомендуется: логика в template -->
<li v-for="item in items.filter(i => i.isActive).sort()" :key="item.id">
  {{ item.name }}
</li>
```

### Лучший подход: computed
```vue
<template>
  <li v-for="item in filteredAndSortedItems" :key="item.id">
    {{ item.name }}
  </li>
</template>

<script setup>
import { computed } from 'vue'

const items = ref([...])

const filteredAndSortedItems = computed(() => {
  return items.value
    .filter(item => item.isActive)
    .sort((a, b) => a.name.localeCompare(b.name))
})
</script>
```

## Обновление списков

### Изменение массива
```vue
<template>
  <div>
    <button @click="addItem">Добавить элемент</button>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, name: 'Элемент 1' },
  { id: 2, name: 'Элемент 2' }
])

const addItem = () => {
  items.value.push({
    id: Date.now(), // Уникальный ID
    name: `Элемент ${items.value.length + 1}`
  })
}
</script>
```

### Мутации массива
Vue отслеживает следующие методы мутации:
- `push()`, `pop()`, `shift()`, `unshift()`
- `splice()`, `sort()`, `reverse()`

### Замена массива
```javascript
// Эти методы создают новый массив
items.value = items.value.filter(item => item.id !== 1)
items.value = [...items.value.slice(0, 2), newItem, ...items.value.slice(2)]
```

## Связь с компонентами
```vue
<!-- ChildComponent.vue -->
<template>
  <div class="item">
    {{ item.name }}
  </div>
</template>

<script setup>
defineProps(['item'])
</script>
```

```vue
<!-- Parent.vue -->
<template>
  <ChildComponent 
    v-for="item in items" 
    :key="item.id" 
    :item="item"
  />
</template>
```

## Тестирование списков
```javascript
import { mount } from '@vue/test-utils'
import ListComponent from '@/components/ListComponent.vue'

test('отображает элементы списка', () => {
  const wrapper = mount(ListComponent, {
    data() {
      return {
        items: [
          { id: 1, name: 'Item 1' },
          { id: 2, name: 'Item 2' }
        ]
      }
    }
  })
  
  expect(wrapper.findAll('li')).toHaveLength(2)
})
```

## Лучшие практики
- Всегда используйте `:key` с v-for
- Используйте уникальные ID, а не индексы
- Выносите сложную логику фильтрации в computed
- Избегайте сложных выражений в v-for
- Не используйте v-for и v-if на одном элементе

## Связь с другими концепциями
- [[directives/Conditional]] - комбинация с v-if
- [[reactivity/Reactivity System]] - реактивность списков
- [[components/Components]] - использование с компонентами
- [[performance/Performance]] - оптимизация производительности