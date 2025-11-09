# v-if, v-else, v-else-if

## Определение
Директивы `v-if`, `v-else`, и `v-else-if` используются для условной отрисовки элементов. Элемент будет отображен в DOM только если условие истинно.

## Использование

### Простой v-if
```vue
<template>
  <div v-if="seen">Теперь ты меня видишь</div>
</template>

<script setup>
import { ref } from 'vue'

const seen = ref(true)
</script>
```

### v-if с v-else
```vue
<template>
  <div v-if="Math.random() > 0.5">
    Больше 0.5
  </div>
  <div v-else>
    Меньше или равно 0.5
  </div>
</template>
```

### v-if с v-else-if
```vue
<template>
  <div v-if="type === 'A'">
    A
  </div>
  <div v-else-if="type === 'B'">
    B
  </div>
  <div v-else-if="type === 'C'">
    C
  </div>
  <div v-else>
    Не A/B/C
  </div>
</template>

<script setup>
import { ref } from 'vue'

const type = ref('A')
</script>
```

## Работа с элементами

### На элементах
```vue
<template>
  <h1 v-if="ok">Добро пожаловать</h1>
</template>
```

### На элементах с v-for
```vue
<template>
  <!-- v-for имеет более высокий приоритет, чем v-if -->
  <li v-for="user in users" v-if="user.isActive" :key="user.id">
    {{ user.name }}
  </li>
</template>

<!-- Для улучшения производительности -->
<template>
  <ul v-if="users.length">
    <li v-for="user in users" :key="user.id">
      {{ user.name }}
    </li>
  </ul>
  <p v-else>Нет пользователей</p>
</template>
```

### На <template>
```vue
<template>
  <template v-if="ok">
    <h1>Заголовок</h1>
    <p>Параграф 1</p>
    <p>Параграф 2</p>
  </template>
</template>
```

## Условия и компоненты
```vue
<template>
  <div>
    <ChildComponent v-if="type === 'child'" />
    <OtherComponent v-else />
  </div>
</template>
```

## Сравнение с v-show

| v-if | v-show |
|------|--------|
| Условная отрисовка | Условное отображение |
| Полное создание/удаление DOM | CSS display: none/block |
| Лучше для редких переключений | Лучше для частых переключений |
| Невозможно использовать с v-else на <template> | Не работает с v-else |

```vue
<template>
  <!-- v-if: дорогая переключение, дешевый начальный рендер -->
  <div v-if="expensiveCondition">
    Сложный компонент
  </div>
  
  <!-- v-show: дешевое переключение, дорогой начальный рендер -->
  <div v-show="expensiveCondition">
    Сложный компонент
  </div>
</template>
```

## Управление анимациями
```vue
<template>
  <div v-if="show" v-transition="fade">
    Анимированный элемент
  </div>
</template>

<script setup>
import { ref } from 'vue'

const show = ref(true)
</script>
```

## Связь с реактивностью
```vue
<template>
  <div v-if="userProfile && userProfile.name">
    <h2>{{ userProfile.name }}</h2>
  </div>
  <div v-else>
    <p>Загрузка профиля...</p>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const userProfile = ref(null)

// Загрузка профиля
const loadProfile = async () => {
  userProfile.value = await fetchUserProfile()
}
</script>
```

## Тестирование условной отрисовки
```javascript
import { mount } from '@vue/test-utils'
import ConditionalComponent from '@/components/ConditionalComponent.vue'

test('показывает элемент когда условие истинно', () => {
  const wrapper = mount(ConditionalComponent, {
    data() {
      return { show: true }
    }
  })
  
  expect(wrapper.find('.conditional').exists()).toBe(true)
})

test('скрывает элемент когда условие ложно', () => {
  const wrapper = mount(ConditionalComponent, {
    data() {
      return { show: false }
    }
  })
  
  expect(wrapper.find('.conditional').exists()).toBe(false)
})
```

## Лучшие практики
- Используйте `v-show` для частых переключений
- Используйте `v-if` для редких переключений
- Не используйте `v-if` и `v-for` на одном элементе
- Для списка с возможным скрытием оберните в `<template>`

## Связь с другими концепциями
- [[directives/v-for]] - условная отрисовка списков
- [[directives/v-show]] - альтернатива условного отображения
- [[components/Slots]] - альтернатива условной отрисовки
- [[reactivity/Reactivity System]] - реактивность условий