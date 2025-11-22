---
aliases: []
tags: [vue, components, slots]
---

# Slots and Scoped Slots

Slots и Scoped Slots - это мощные механизмы Vue.js для создания гибких и переиспользуемых компонентов. Они позволяют передавать содержимое от родительского компонента дочернему, обеспечивая возможность создания компонентов с переменным содержимым.

## Базовые слоты

Простой слот позволяет передать произвольное содержимое в компонент:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="card">
    <slot></slot>
  </div>
</template>

<!-- Использование -->
<template>
  <ChildComponent>
    <p>Это содержимое будет отображено внутри слота</p>
  </ChildComponent>
</template>
```

## Именованные слоты

Именованные слоты позволяют создавать несколько слотов в одном компоненте:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<!-- Использование -->
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

## Слоты по умолчанию

Можно задать содержимое по умолчанию для слота:

```vue
<template>
  <button class="btn">
    <slot>Кнопка</slot>
  </button>
</template>
```

## Scoped Slots

Scoped slots позволяют передавать данные из дочернего компонента обратно в область видимости родительского компонента:

```vue
<!-- ChildComponent.vue -->
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      <slot :item="item" :index="item.index" :text="item.text"></slot>
    </li>
  </ul>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Элемент 1', index: 0 },
        { id: 2, text: 'Элемент 2', index: 1 }
      ]
    }
  }
}
</script>

<!-- Использование -->
<template>
  <ChildComponent v-slot="slotProps">
    <span :class="slotProps.item.text === 'Элемент 1' ? 'highlight' : ''">
      {{ slotProps.item.index }} - {{ slotProps.item.text }}
    </span>
  </ChildComponent>
</template>
```

## Composition API и слоты

В Composition API можно использовать слоты так же, как и в Options API, но с дополнительными возможностями:

```vue
<template>
  <div>
    <slot name="header" :user="user" :status="status" />
    <slot :update-status="updateStatus" />
  </div>
</template>

<script setup>
import { ref } from 'vue'

const user = ref({ name: 'John', id: 1 })
const status = ref('active')

function updateStatus(newStatus) {
  status.value = newStatus
}
</script>
```

## Абсолютные слоты (v-slot)

Синтаксис `v-slot` предоставляет более гибкий способ работы со слотами:

```vue
<template>
  <ChildComponent>
    <template v-slot:header="slotProps">
      <h1>{{ slotProps.title }}</h1>
    </template>
    
    <template v-slot:default="slotProps">
      <p>{{ slotProps.content }}</p>
    </template>
  </ChildComponent>
</template>
```

## Практические применения

Слоты особенно полезны при создании:

1. Компонентов-контейнеров (карточки, панели, модальные окна)
2. Компонентов с переменным содержимым (списки, таблицы)
3. Переиспользуемых UI-компонентов
4. Компонентов с вариативной структурой

См. также: [[Vue компоненты]], [[Single File Components]], [[Props and Events]]