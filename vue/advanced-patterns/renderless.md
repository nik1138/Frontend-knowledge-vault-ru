# Renderless Components

## Определение
Renderless компоненты (без отрисовки) - это компоненты, которые не отображают собственный UI, а вместо этого предоставляют логику, состояние и поведение для использования другими компонентами через scoped slots или слоты с данными.

## Основной паттерн

### Простой renderless компонент
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

const fetchData = async () => {
  try {
    loading.value = true
    error.value = null
    const response = await fetch(props.url)
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    
    data.value = await response.json()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const refetch = () => fetchData()

onMounted(fetchData)
</script>
```

```vue
<!-- Использование -->
<template>
  <DataFetcher url="/api/users">
    <template #default="{ data, loading, error, refetch }">
      <div v-if="loading">Загрузка...</div>
      <div v-else-if="error">Ошибка: {{ error }}</div>
      <div v-else>
        <ul>
          <li v-for="user in data.users" :key="user.id">
            {{ user.name }}
          </li>
        </ul>
        <button @click="refetch">Обновить</button>
      </div>
    </template>
  </DataFetcher>
</template>
```

## Практические примеры

### Компонент для пагинации
```vue
<!-- Paginator.vue -->
<template>
  <slot
    :currentPage="currentPage"
    :totalPages="totalPages"
    :hasPrev="hasPrev"
    :hasNext="hasNext"
    :goToPage="goToPage"
    :prevPage="prevPage"
    :nextPage="nextPage"
  />
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  totalItems: { type: Number, required: true },
  itemsPerPage: { type: Number, default: 10 },
  initialPage: { type: Number, default: 1 }
})

const currentPage = ref(props.initialPage)

const totalPages = computed(() => Math.ceil(props.totalItems / props.itemsPerPage))

const hasPrev = computed(() => currentPage.value > 1)
const hasNext = computed(() => currentPage.value < totalPages.value)

const goToPage = (page) => {
  currentPage.value = Math.max(1, Math.min(page, totalPages.value))
}

const prevPage = () => {
  if (hasPrev.value) {
    currentPage.value--
  }
}

const nextPage = () => {
  if (hasNext.value) {
    currentPage.value++
  }
}
</script>
```

```vue
<!-- Использование пагинации -->
<template>
  <Paginator :total-items="totalItems" :items-per-page="10">
    <template #default="{ currentPage, totalPages, goToPage, hasPrev, hasNext, prevPage, nextPage }">
      <div>
        <div class="content">
          <!-- Отображение элементов для текущей страницы -->
          <ul>
            <li v-for="item in currentPageItems" :key="item.id">
              {{ item.name }}
            </li>
          </ul>
        </div>
        
        <div class="pagination-controls">
          <button @click="prevPage" :disabled="!hasPrev">Предыдущая</button>
          <button v-for="page in totalPages" 
                  :key="page" 
                  @click="goToPage(page)" 
                  :class="{ active: page === currentPage }">
            {{ page }}
          </button>
          <button @click="nextPage" :disabled="!hasNext">Следующая</button>
        </div>
      </div>
    </template>
  </Paginator>
</template>
```

### Компонент управления формой
```vue
<!-- FormManager.vue -->
<template>
  <slot
    :form="form"
    :errors="errors"
    :valid="isValid"
    :submitting="submitting"
    :handleSubmit="handleSubmit"
    :updateField="updateField"
    :resetForm="resetForm"
  />
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  initialData: { type: Object, required: true },
  validate: { type: Function, required: true },
  onSubmit: { type: Function, required: true }
})

const form = ref({ ...props.initialData })
const errors = ref({})
const submitting = ref(false)

const isValid = computed(() => Object.keys(errors.value).length === 0)

const updateField = (field, value) => {
  form.value[field] = value
  // Валидация только при изменении конкретного поля
  const fieldError = props.validate({ [field]: value }, form.value)
  errors.value[field] = fieldError ? fieldError[field] : null
}

const resetForm = () => {
  form.value = { ...props.initialData }
  errors.value = {}
}

const handleSubmit = async () => {
  errors.value = props.validate(form.value) || {}
  
  if (isValid.value) {
    submitting.value = true
    try {
      await props.onSubmit(form.value)
      resetForm()
    } finally {
      submitting.value = false
    }
  }
}
</script>
```

## Сравнение с другими подходами

### Renderless vs Композаблы
```javascript
// Композабл
export function useDataFetcher(url) {
  const data = ref(null)
  const loading = ref(true)
  
  const fetchData = async () => {
    // логика получения данных
  }
  
  return { data, loading, fetchData }
}
```

```vue
<!-- Renderless компонент -->
<template>
  <slot :data="data" :loading="loading" />
</template>
```

**Композаблы лучше для:**
- Простой логики
- Когда не нужен доступ к контексту компонента

**Renderless компоненты лучше для:**
- Сложных состояний
- Когда нужна инкапсуляция UI-логики
- Переиспользуемых паттернов с UI

### Рендер-пропсы (React) vs Scoped slots (Vue)
```vue
<!-- Vue scoped slot -->
<template>
  <DataFetcher url="/api/data">
    <template #default="data">
      <div>{{ data.items }}</div>
    </template>
  </DataFetcher>
</template>
```

## Продвинутые примеры

### Компонент для управления состоянием поиска
```vue
<!-- SearchManager.vue -->
<template>
  <slot
    :query="query"
    :results="results"
    :loading="loading"
    :error="error"
    :search="search"
    :setQuery="setQuery"
    :clear="clear"
  />
</template>

<script setup>
import { ref, debounce } from 'vue'

const props = defineProps({
  searchEndpoint: { type: String, required: true },
  debounceMs: { type: Number, default: 300 }
})

const query = ref('')
const results = ref([])
const loading = ref(false)
const error = ref(null)

const performSearch = async () => {
  if (!query.value.trim()) {
    results.value = []
    return
  }
  
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(`${props.searchEndpoint}?q=${encodeURIComponent(query.value)}`)
    results.value = await response.json()
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const debouncedSearch = debounce(performSearch, props.debounceMs)

const search = (searchQuery) => {
  query.value = searchQuery
  debouncedSearch()
}

const setQuery = (newQuery) => {
  query.value = newQuery
  debouncedSearch()
}

const clear = () => {
  query.value = ''
  results.value = []
  error.value = null
}
</script>
```

## Преимущества renderless компонентов

1. **Повторное использование логики** - инкапсуляция сложной логики
2. **Гибкость UI** - пользователь контролирует отображение
3. **Композиция** - легко комбинировать несколько компонентов
4. **Тестируемость** - изолированная логика легче тестируется

## Ограничения и подводные камни

### Перформанс
```vue
<!-- Потенциальная проблема: переиспользование компонента может создать много экземпляров -->
<DataFetcher url="/api/users">
  <template #default="{ data }">
    <!-- Этот слот будет перерендериваться при каждом изменении данных -->
    <ComplexComponent :data="data" />
  </template>
</DataFetcher>
```

### Отладка
- Сложнее отлаживать из-за абстракций
- Меньше информации в инструментах разработчика

## Тестирование renderless компонентов
```javascript
import { mount } from '@vue/test-utils'
import DataFetcher from '@/components/DataFetcher.vue'

test('передает данные в слот', async () => {
  const mockFetch = jest.spyOn(global, 'fetch')
    .mockResolvedValue({
      ok: true,
      json: () => Promise.resolve({ users: [] })
    })
  
  const wrapper = mount(DataFetcher, {
    props: { url: '/api/test' },
    slots: {
      default: '<div>{{ props.loading }}</div>'
    }
  })
  
  // Проверяем, что данные передаются
  await wrapper.vm.$nextTick()
  expect(wrapper.text()).toContain('true') // loading
  
  mockFetch.mockRestore()
})
```

## Связь с другими концепциями
- [[advanced-patterns/scoped-slots]] - основа для renderless компонентов
- [[composables/Composables]] - альтернативный подход к повторному использованию логики
- [[components/slots]] - использование слотов для передачи данных
- [[reactivity/Reactivity System]] - реактивность внутри компонентов