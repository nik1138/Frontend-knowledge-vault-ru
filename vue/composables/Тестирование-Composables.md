---
aliases: ["Тестирование Composables", "Тестирование хуков", "Vue Composables Testing"]
tags: ["#vue", "#composables", "#testing", "#unit-testing", "#frontend-development"]
---

# Тестирование Composables в Vue 3

## Введение

Тестирование Composables является критически важным аспектом разработки на Vue 3, особенно в российской практике, где требования к качеству кода и надежности приложений постоянно растут. Composables инкапсулируют бизнес-логику и состояние, что делает их отличной мишенью для модульного тестирования.

## Подходы к тестированию Composables

### 1. Тестирование в изоляции

Composables можно тестировать отдельно от компонентов, что упрощает написание тестов и повышает их надежность:

```javascript
// composables/useCounter.js
import { ref } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initialValue

  return {
    count,
    increment,
    decrement,
    reset
  }
}
```

```javascript
// tests/composables/useCounter.test.js
import { useCounter } from '@/composables/useCounter'

describe('useCounter', () => {
  test('инициализируется с правильным значением', () => {
    const { count } = useCounter(5)
    expect(count.value).toBe(5)
  })

  test('увеличивает значение при вызове increment', () => {
    const { count, increment } = useCounter(0)
    increment()
    expect(count.value).toBe(1)
  })

  test('уменьшает значение при вызове decrement', () => {
    const { count, decrement } = useCounter(5)
    decrement()
    expect(count.value).toBe(4)
  })

  test('сбрасывает значение при вызове reset', () => {
    const { count, increment, reset } = useCounter(0)
    increment()
    increment()
    reset()
    expect(count.value).toBe(0)
  })
})
```

### 2. Использование @vue/test-utils

Для тестирования Composables, которые зависят от контекста Vue, используйте `renderHook` или создайте вспомогательную функцию:

```javascript
// tests/utils/test-utils.js
import { mount } from '@vue/test-utils'
import { defineComponent, h } from 'vue'

export function renderHook(composable) {
  const component = defineComponent({
    setup() {
      return composable()
    },
    render() {}
  })

  const wrapper = mount(component)
  return wrapper.vm
}
```

## Тестирование асинхронных Composables

### Composable с API вызовами

```javascript
// composables/useApiData.js
import { ref } from 'vue'

export function useApiData(apiFunction) {
  const data = ref(null)
  const loading = ref(false)
  const error = ref(null)

  const fetchData = async () => {
    loading.value = true
    error.value = null

    try {
      const result = await apiFunction()
      data.value = result
    } catch (err) {
      error.value = err.message
    } finally {
      loading.value = false
    }
  }

  return {
    data,
    loading,
    error,
    fetchData
  }
}
```

```javascript
// tests/composables/useApiData.test.js
import { useApiData } from '@/composables/useApiData'

describe('useApiData', () => {
  test('загружает данные при вызове fetchData', async () => {
    const mockApi = jest.fn().mockResolvedValue({ id: 1, name: 'Тест' })
    const { data, loading, fetchData } = useApiData(mockApi)

    expect(loading.value).toBe(false)
    
    await fetchData()
    
    expect(loading.value).toBe(false)
    expect(data.value).toEqual({ id: 1, name: 'Тест' })
    expect(mockApi).toHaveBeenCalled()
  })

  test('обрабатывает ошибки', async () => {
    const mockApi = jest.fn().mockRejectedValue(new Error('Ошибка API'))
    const { error, fetchData } = useApiData(mockApi)

    await fetchData()
    
    expect(error.value).toBe('Ошибка API')
  })
})
```

## Тестирование Composables с жизненным циклом

### Composable с onMounted

```javascript
// composables/useAutoLoad.js
import { ref, onMounted } from 'vue'

export function useAutoLoad(loadFunction) {
  const data = ref(null)
  const loading = ref(false)

  const loadData = async () => {
    loading.value = true
    try {
      data.value = await loadFunction()
    } finally {
      loading.value = false
    }
  }

  onMounted(() => {
    loadData()
  })

  return {
    data,
    loading
  }
}
```

Для тестирования таких Composables потребуется использовать `mount` с компонентом:

```javascript
// tests/composables/useAutoLoad.test.js
import { mount } from '@vue/test-utils'
import { defineComponent } from 'vue'
import { useAutoLoad } from '@/composables/useAutoLoad'

test('автоматически загружает данные при монтировании', async () => {
  const mockLoad = jest.fn().mockResolvedValue('тестовые данные')
  
  const TestComponent = defineComponent({
    setup() {
      return useAutoLoad(mockLoad)
    },
    template: '<div></div>'
  })

  const wrapper = mount(TestComponent)
  
  // Проверяем, что данные загружаются
  expect(wrapper.vm.loading).toBe(true)
  
  await wrapper.vm.$nextTick()
  await new Promise(resolve => setTimeout(resolve, 0))
  
  expect(mockLoad).toHaveBeenCalled()
  expect(wrapper.vm.data).toBe('тестовые данные')
  expect(wrapper.vm.loading).toBe(false)
})
```

## Тестирование реактивных зависимостей

### Composable с computed

```javascript
// composables/useFilteredItems.js
import { ref, computed } from 'vue'

export function useFilteredItems(items, searchTerm) {
  const allItems = ref([...items])
  const search = ref(searchTerm)

  const filteredItems = computed(() => {
    if (!search.value) return allItems.value
    return allItems.value.filter(item =>
      item.toLowerCase().includes(search.value.toLowerCase())
    )
  })

  return {
    filteredItems,
    search
  }
}
```

```javascript
// tests/composables/useFilteredItems.test.js
import { useFilteredItems } from '@/composables/useFilteredItems'

test('фильтрует элементы на основе поискового запроса', () => {
  const items = ['яблоко', 'банан', 'апельсин']
  const { filteredItems, search } = useFilteredItems(items, '')

  expect(filteredItems.value).toEqual(items)

  search.value = 'ябл'
  expect(filteredItems.value).toEqual(['яблоко'])
})
```

## Тестирование с моками и шпионами

### Моки localStorage

```javascript
// composables/useLocalStorage.js
import { ref, watch } from 'vue'

export function useLocalStorage(key, defaultValue) {
  const value = ref(defaultValue)

  // Загрузка из localStorage
  const saved = localStorage.getItem(key)
  if (saved) {
    value.value = JSON.parse(saved)
  }

  // Сохранение в localStorage
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue))
  }, { deep: true })

  return value
}
```

```javascript
// tests/composables/useLocalStorage.test.js
import { useLocalStorage } from '@/composables/useLocalStorage'

describe('useLocalStorage', () => {
  // Мокаем localStorage
  const mockLocalStorage = {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
  }

  beforeEach(() => {
    Object.defineProperty(window, 'localStorage', {
      value: mockLocalStorage,
    })
    jest.clearAllMocks()
  })

  test('сохраняет значение в localStorage при изменении', () => {
    mockLocalStorage.getItem.mockReturnValue(null)
    
    const value = useLocalStorage('test-key', 'default-value')
    
    value.value = 'новое значение'
    
    expect(mockLocalStorage.setItem).toHaveBeenCalledWith(
      'test-key',
      JSON.stringify('новое значение')
    )
  })
})
```

## Интеграционное тестирование Composables

### Тестирование взаимодействия между несколькими Composables

```javascript
// composables/useUserProfile.js
import { useAuth } from './useAuth'
import { useApiResource } from './useApiResource'

export function useUserProfile() {
  const { user, isAuthenticated } = useAuth()
  const { data: profile, loading, error, fetchData } = useApiResource(
    fetchUserProfile,
    computed(() => isAuthenticated.value ? user.value?.id : null)
  )

  return {
    profile,
    loading,
    error,
    isAuthenticated,
    refreshProfile: fetchData
  }
}
```

```javascript
// tests/composables/useUserProfile.test.js
import { useUserProfile } from '@/composables/useUserProfile'

test('загружает профиль пользователя при аутентификации', async () => {
  // Мокаем зависимости
  jest.mock('@/composables/useAuth', () => ({
    useAuth: () => ({
      user: { id: 1, name: 'Иван' },
      isAuthenticated: { value: true }
    })
  }))

  jest.mock('@/composables/useApiResource', () => ({
    useApiResource: (apiFn, userId) => ({
      data: { value: null },
      loading: { value: false },
      error: { value: null },
      fetchData: jest.fn()
    })
  }))

  const { profile, refreshProfile } = useUserProfile()
  
  await refreshProfile()
  
  // Проверяем, что профиль загружен
  expect(profile.value).toBeDefined()
})
```

## Практические рекомендации российских разработчиков

### 1. Структура тестов

Организуйте тесты по следующему принципу:

```
tests/
├── composables/
│   ├── useCounter.test.js
│   ├── useApiData.test.js
│   ├── useLocalStorage.test.js
│   └── auth/
│       ├── useAuth.test.js
│       └── usePermissions.test.js
```

### 2. Покрытие тестами

- Цель: 80-90% покрытия для критических Composables
- Для вспомогательных утилит: 70-80%
- Используйте Istanbul для анализа покрытия

### 3. Тестирование граничных условий

```javascript
test('обрабатывает пустые значения', () => {
  const { items, addItem } = useItemList()
  
  addItem('')
  expect(items.value).toHaveLength(0)
  
  addItem(null)
  expect(items.value).toHaveLength(0)
})

test('обрабатывает максимальное количество элементов', () => {
  const { items, addItem } = useItemList(5) // максимальное 5 элементов
  
  for (let i = 0; i < 10; i++) {
    addItem(`item${i}`)
  }
  
  expect(items.value).toHaveLength(5)
})
```

### 4. Использование фикстур

```javascript
// tests/fixtures/user.js
export const mockUser = {
  id: 1,
  name: 'Иван Петров',
  email: 'ivan@example.com',
  role: 'admin'
}

export const mockUsers = [
  mockUser,
  {
    id: 2,
    name: 'Мария Сидорова',
    email: 'maria@example.com',
    role: 'user'
  }
]
```

## Инструменты для тестирования

### Jest с Vue Test Utils

```javascript
// jest.config.js
module.exports = {
  preset: '@vue/cli-plugin-unit-jest',
  collectCoverageFrom: [
    'src/composables/**/*.{js,vue}',
    '!src/composables/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
}
```

### Vue Testing Library (альтернатива)

```javascript
import { renderHook } from '@testing-library/vue'
import { useCounter } from '@/composables/useCounter'

test('инициализирует счетчик с правильным значением', () => {
  const { result } = renderHook(() => useCounter(5))
  
  expect(result.current.count.value).toBe(5)
})
```

## Заключение

Тестирование Composables в Vue 3 требует особого подхода, отличающегося от тестирования компонентов. В российской практике разработки особое внимание уделяется качеству тестов и покрытию, что позволяет создавать надежные и легко поддерживаемые приложения. Правильное тестирование Composables обеспечивает стабильность бизнес-логики и упрощает рефакторинг в будущем.

## См. также

- [[Создание-кастомных-хуков]]
- [[Повторное-использование]]
- [[Тестирование-компонентов]]
- [[Реактивные-объявления]]
- [[Модульное-тестирование]]