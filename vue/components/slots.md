# Slots

## Определение
Slots (слоты) - это механизм в Vue, позволяющий авторам компонентов создавать места, куда могут быть вставлены пользователи компонента. Это основной способ композиции компонентов и передачи содержимого дочерних элементов.

## Типы слотов

### Default Slot
```vue
<!-- ChildComponent.vue -->
<template>
  <div class="card">
    <header>Заголовок карточки</header>
    <div class="content">
      <slot></slot> <!-- Место для вставки содержимого -->
    </div>
    <footer>Футер карточки</footer>
  </div>
</template>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <p>Это содержимое будет вставлено в слот</p>
    <button>Кнопка тоже в слоте</button>
  </ChildComponent>
</template>
```

### Named Slots
```vue
<!-- ChildComponent.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot> <!-- Default slot -->
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <template #header>
      <h1>Заголовок</h1>
    </template>
    
    <p>Основное содержимое</p>
    
    <template #footer>
      <p>Футер</p>
    </template>
  </ChildComponent>
</template>
```

### Scoped Slots
```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <slot :user="user" :isActive="user.active"></slot>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const user = ref({
  name: 'John',
  active: true
})
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent v-slot="slotProps">
    <p>{{ slotProps.user.name }}</p>
    <p :class="slotProps.isActive ? 'active' : 'inactive'">
      Статус: {{ slotProps.isActive ? 'Активен' : 'Неактивен' }}
    </p>
  </ChildComponent>
</template>
```

## Слоты с fallback
```vue
<template>
  <div>
    <slot>
      <p>Содержимое по умолчанию, если слот не заполнен</p>
    </slot>
  </div>
</template>
```

## Динамические имена слотов
```vue
<template>
  <ChildComponent>
    <template #[dynamicSlotName]>
      <p>Содержимое для динамического слота</p>
    </template>
  </ChildComponent>
</template>

<script setup>
import { ref } from 'vue'

const dynamicSlotName = ref('header')
</script>
```

## Слоты в Composition API
```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()

// Проверка наличия слота
const hasDefaultSlot = !!slots.default
const hasHeaderSlot = !!slots.header
</script>
```

## Слоты vs Props/Events
- **Slots**: для передачи содержимого и структуры
- **Props**: для передачи данных сверху вниз
- **Events**: для отправки сообщений снизу вверх

## Практические применения
- Создание переиспользуемых компонентов с кастомным содержимым
- Карточки, модальные окна, макеты
- Компоненты с переменной структурой

## Связь с другими концепциями
- [[props]] - альтернатива для передачи простых данных
- [[events]] - для коммуникации между компонентами
- [[composables/Composables]] - для сложной логики компонентов

## Примечания
- Слоты - мощный инструмент композиции компонентов
- Используйте scoped slots для передачи данных из компонента
- Named slots позволяют создавать сложные компоновки