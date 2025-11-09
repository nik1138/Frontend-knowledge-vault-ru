# Advanced Patterns

Продвинутые паттерны Vue - это архитектурные подходы и методы, которые позволяют создавать более сложные, гибкие и поддерживаемые приложения.

## Основные паттерны

### [Renderless Components](renderless.md)
Компоненты без UI, предоставляющие логику другим компонентам

### [Higher-Order Components](hoc.md)
Компоненты, возвращающие другие компоненты с дополнительной функциональностью

### [Mixin Pattern](mixins.md)
Повторное использование логики между компонентами (устаревший подход в пользу Composition API)

### [Scoped Slots Pattern](scoped-slots.md)
Мощный паттерн композиции компонентов с передачей данных

### [Provide/Inject Pattern](provide-inject.md)
Передача данных глубоко в дереве компонентов

### [Template Refs Pattern](template-refs.md)
Управление DOM элементами и экземплярами компонентов

## Архитектурные паттерны

### [Container/Presenter Pattern](container-presenter.md)
Разделение логики и представления

### [State Machine Pattern](state-machines.md)
Управление сложными состояниями компонентов

### [Event Bus Pattern](event-bus.md)
Коммуникация между компонентами через глобальный эмиттер

## Паттерны организации кода

### [Module Pattern](modules.md)
Организация кода в модули для лучшей структуры

### [Factory Pattern](factories.md)
Создание компонентов и объектов с помощью фабричных функций

### [Composition Functions](composition-functions.md)
Повторно используемая логика в Composition API

## Примеры сложных паттернов

### Композабл для пагинации
```javascript
import { ref, computed } from 'vue'

export function usePagination(items, itemsPerPage = 10) {
  const currentPage = ref(1)
  
  const totalPages = computed(() => Math.ceil(items.value.length / itemsPerPage))
  
  const paginatedItems = computed(() => {
    const start = (currentPage.value - 1) * itemsPerPage
    return items.value.slice(start, start + itemsPerPage)
  })
  
  const goToPage = (page) => {
    currentPage.value = Math.max(1, Math.min(page, totalPages.value))
  }
  
  return {
    currentPage,
    totalPages,
    paginatedItems,
    goToPage
  }
}
```

### Рендерлесс компонент
```vue
<!-- DataFetcher.vue -->
<template>
  <slot 
    :data="data" 
    :loading="loading" 
    :error="error"
    :refetch="refetch"
  />
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps(['url'])
const data = ref(null)
const loading = ref(true)
const error = ref(null)

const fetcher = async () => {
  try {
    loading.value = true
    error.value = null
    const response = await fetch(props.url)
    data.value = await response.json()
  } catch (err) {
    error.value = err
  } finally {
    loading.value = false
  }
}

const refetch = () => fetcher()

onMounted(fetcher)
</script>
```

## Связь с другими концепциями
- [[composables/Composables]] - модернизированный подход к повторному использованию логики
- [[components/Components]] - основа для паттернов компонентов
- [[state-management/State Management]] - управление сложными состояниями