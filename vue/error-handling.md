# Обработка ошибок в Vue.js приложениях

Обработка ошибок - критически важный аспект архитектуры любых приложений. В этой статье мы рассмотрим стратегии и паттерны обработки ошибок в Vue.js приложениях, включая как технические, так и архитектурные аспекты.

## Введение

Обработка ошибок в Vue.js включает в себя несколько уровней:

1. Обработка ошибок на уровне компонентов
2. Обработка асинхронных ошибок
3. Обработка ошибок API
4. Глобальная обработка ошибок
5. Логирование ошибок

## Обработка ошибок на уровне компонентов

### 1. Обработка ошибок в методах компонентов

В Composition API ошибки можно обрабатывать с помощью стандартных блоков try/catch:

```vue
<template>
  <div>
    <button @click="loadData">Загрузить данные</button>
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error" class="error">
      {{ error }}
    </div>
    <div v-else>
      <!-- Отображение данных -->
      <pre>{{ data }}</pre>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const data = ref(null)
const loading = ref(false)
const error = ref<string | null>(null)

const loadData = async () => {
  loading.value = true
  error.value = null
  
  try {
    // Предполагаем, что fetchApiData может выбросить ошибку
    data.value = await fetchApiData()
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Произошла неизвестная ошибка'
  } finally {
    loading.value = false
  }
}
</script>

<style>
.error {
  color: red;
  padding: 10px;
  border: 1px solid red;
  background-color: #ffe6e6;
}
</style>
```

### 2. Использование onErrorCaptured для перехвата ошибок дочерних компонентов

Composition API предоставляет хук `onErrorCaptured`, который позволяет перехватывать ошибки от дочерних компонентов:

```vue
<template>
  <div>
    <div v-if="error" class="error-boundary">
      Произошла ошибка: {{ error }}
      <button @click="resetError">Попробовать снова</button>
    </div>
    <div v-else>
      <slot />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

const error = ref<string | null>(null)

onErrorCaptured((err, instance, info) => {
  // Сохраняем информацию об ошибке
  error.value = err.message
  
  // Логируем ошибку с дополнительной информацией
  console.error('Ошибка в компоненте:', instance?.type.name)
  console.error('Ошибка:', err)
  console.error('Информация:', info)
  
  // Возвращаем false, чтобы остановить распространение ошибки
  return false
})

const resetError = () => {
  error.value = null
}
</script>
```

## Обработка асинхронных ошибок

### 1. Создание композабла для обработки асинхронных операций

```typescript
// composables/useAsyncOperation.ts
import { ref, Ref } from 'vue'

interface AsyncOperationResult<T> {
  data: Ref<T | null>
  loading: Ref<boolean>
  error: Ref<string | null>
  execute: () => Promise<void>
}

export function useAsyncOperation<T>(asyncFn: () => Promise<T>): AsyncOperationResult<T> {
  const data = ref<T | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)

  const execute = async () => {
    loading.value = true
    error.value = null
    
    try {
      data.value = await asyncFn()
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Произошла неизвестная ошибка'
    } finally {
      loading.value = false
    }
  }

  return {
    data,
    loading,
    error,
    execute
  }
}
```

### 2. Использование композабла в компоненте

```vue
<template>
  <div>
    <button @click="loadUserData">Загрузить профиль</button>
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error" class="error">
      {{ error }}
      <button @click="loadUserData">Повторить</button>
    </div>
    <div v-else-if="data">
      <h2>{{ data.name }}</h2>
      <p>{{ data.email }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useAsyncOperation } from '@/composables/useAsyncOperation'

const { data, loading, error, execute } = useAsyncOperation(() => 
  fetch('/api/user/profile').then(res => res.json())
)

const loadUserData = () => {
  execute()
}
</script>
```

## Обработка ошибок API

### 1. Создание сервиса для обработки API ошибок

```typescript
// services/apiService.ts
import { AxiosError } from 'axios'

export interface ApiError {
  status: number
  message: string
  details?: Record<string, any>
}

class ApiService {
  async request<T>(url: string, options?: RequestInit): Promise<T> {
    try {
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options?.headers
        },
        ...options
      })

      if (!response.ok) {
        const errorData = await response.json().catch(() => ({}))
        throw new Error(`HTTP ${response.status}: ${errorData.message || response.statusText}`)
      }

      return await response.json()
    } catch (error) {
      // Обработка различных типов ошибок
      if (error instanceof TypeError) {
        // Сетевая ошибка (например, нет подключения)
        throw new Error('Нет подключения к интернету')
      } else if (error instanceof Error) {
        // Ошибка с сообщением от сервера
        throw error
      } else {
        // Неизвестная ошибка
        throw new Error('Произошла неизвестная ошибка')
      }
    }
  }
}

// Глобальный экземпляр сервиса
export const apiService = new ApiService()
```

### 2. Валидация ответов API

```typescript
// utils/validation.ts
export function validateApiResponse<T>(data: any, schema: any): T {
  // Простая проверка структуры ответа
  if (!data) {
    throw new Error('Ответ API пустой')
  }
  
  // Можно использовать Joi, Yup или Zod для более сложной валидации
  for (const requiredField of schema.requiredFields || []) {
    if (!(requiredField in data)) {
      throw new Error(`Отсутствует обязательное поле: ${requiredField}`)
    }
  }
  
  return data as T
}
```

## Глобальная обработка ошибок

### 1. Глобальные обработчики ошибок в Vue

Vue предоставляет два глобальных обработчика ошибок:

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Глобальный обработчик ошибок
app.config.errorHandler = (err, instance, info) => {
  console.error('Глобальная обработка ошибки:')
  console.error(err)
  console.error('Компонент:', instance)
  console.error('Информация:', info)
  
  // Отправка ошибки в систему мониторинга (например, Sentry)
  // logErrorToService(err, { component: instance?.type.name, info })
}

// Обработчик необработанных промисов
window.addEventListener('unhandledrejection', (event) => {
  console.error('Необработанный промис:', event.reason)
  // logErrorToService(event.reason, { type: 'unhandled-promise-rejection' })
})

app.mount('#app')
```

### 2. Централизованный компонент для отображения ошибок

```vue
<!-- components/GlobalErrorHandler.vue -->
<template>
  <div v-if="hasError" class="global-error-overlay">
    <div class="error-modal">
      <h3>Произошла ошибка</h3>
      <p>{{ currentError }}</p>
      <div class="error-actions">
        <button @click="retry">Повторить</button>
        <button @click="closeError">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const currentError = ref<string | null>(null)
const hasError = ref(false)

const setError = (message: string) => {
  currentError.value = message
  hasError.value = true
}

const closeError = () => {
  hasError.value = false
  currentError.value = null
}

const retry = () => {
  // Сброс ошибки и повтор последнего действия (реализация зависит от контекста)
  closeError()
  // Возможность повтора последнего действия
}

// Обработка ошибок через глобальные события
const handleError = (event: CustomEvent) => {
  setError(event.detail.message)
}

onMounted(() => {
  window.addEventListener('app-error', handleError as EventListener)
})

onUnmounted(() => {
  window.removeEventListener('app-error', handleError as EventListener)
})
</script>

<style scoped>
.global-error-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
}

.error-modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.error-actions {
  margin-top: 15px;
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.error-actions button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.error-actions button:first-child {
  background: #007bff;
  color: white;
}
</style>
```

## Логирование ошибок

### 1. Создание сервиса логирования

```typescript
// services/errorLogger.ts
interface ErrorLog {
  timestamp: Date
  level: 'error' | 'warning' | 'info'
  message: string
  stack?: string
  context?: Record<string, any>
}

class ErrorLogger {
  private logs: ErrorLog[] = []
  private maxLogs = 100 // Максимальное количество логов в памяти

  log(level: ErrorLog['level'], message: string, context?: Record<string, any>) {
    const log: ErrorLog = {
      timestamp: new Date(),
      level,
      message,
      context,
      stack: level === 'error' ? new Error().stack : undefined
    }

    this.logs.push(log)
    
    // Удаляем старые логи, если превышен лимит
    if (this.logs.length > this.maxLogs) {
      this.logs = this.logs.slice(-this.maxLogs)
    }

    // В реальном приложении можно отправлять логи на сервер
    if (level === 'error') {
      console.error(log)
    } else {
      console[level](log)
    }
  }

  error(message: string, context?: Record<string, any>) {
    this.log('error', message, context)
  }

  warn(message: string, context?: Record<string, any>) {
    this.log('warning', message, context)
  }

  info(message: string, context?: Record<string, any>) {
    this.log('info', message, context)
  }

  getLogs() {
    return [...this.logs] // Возврат копии массива
  }

  clearLogs() {
    this.logs = []
  }
}

export const errorLogger = new ErrorLogger()
```

### 2. Интеграция логирования с композаблами

```typescript
// composables/useErrorHandling.ts
import { ref, onErrorCaptured } from 'vue'
import { errorLogger } from '@/services/errorLogger'

export function useErrorHandling() {
  const error = ref<string | null>(null)

  const setError = (message: string, context?: Record<string, any>) => {
    error.value = message
    errorLogger.error(message, context)
  }

  const clearError = () => {
    error.value = null
  }

  // Перехват ошибок в компоненте
  onErrorCaptured((err, instance, info) => {
    setError(err.message, {
      component: instance?.type.name,
      info,
      stack: err.stack
    })
    
    return false // Остановить распространение ошибки
  })

  return {
    error,
    setError,
    clearError
  }
}
```

## Архитектурные шаблоны обработки ошибок

### 1. Шаблон Result

Result - это шаблон, который позволяет явно обрабатывать успешные и неуспешные результаты операций:

```typescript
// types/result.ts
export type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E }

// utils/api.ts
export async function safeApiCall<T>(apiCall: () => Promise<T>): Promise<Result<T>> {
  try {
    const data = await apiCall()
    return { success: true, data }
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error : new Error('Неизвестная ошибка') 
    }
  }
}

// Использование в композабле
async function loadUserData(userId: number) {
  const result = await safeApiCall(() => fetchUserById(userId))
  
  if (result.success) {
    // Обработка успешного результата
    user.value = result.data
  } else {
    // Обработка ошибки
    errorLogger.error(`Ошибка загрузки пользователя: ${result.error.message}`)
    errorMessage.value = result.error.message
  }
}
```

### 2. Шаблон Error Boundary (Границы ошибок)

Границы ошибок - это компоненты, которые перехватывают JavaScript ошибки в дереве компонентов потомков, регистрируют эти ошибки и отображают запасной UI, а не дерево компонентов, в котором произошла ошибка.

```vue
<!-- components/ErrorBoundary.vue -->
<template>
  <component 
    :is="error ? errorComponent : 'div'" 
    v-if="errorComponent && error"
    :error="error"
    :info="info"
    @reset="reset"
  />
  <slot v-else />
</template>

<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

interface Props {
  errorComponent?: any // Компонент для отображения ошибок
}

const props = withDefaults(defineProps<Props>(), {
  errorComponent: undefined
})

const error = ref<Error | null>(null)
const info = ref<string>('')

onErrorCaptured((err, instance, errInfo) => {
  error.value = err
  info.value = errInfo || ''
  return false
})

const reset = () => {
  error.value = null
  info.value = ''
}
</script>
```

## Тестирование обработки ошибок

### 1. Юнит-тесты для композаблов обработки ошибок

```typescript
// composables/useAsyncOperation.test.ts
import { describe, it, expect, vi } from 'vitest'
import { useAsyncOperation } from '@/composables/useAsyncOperation'

describe('useAsyncOperation', () => {
  it('should handle successful operations', async () => {
    const mockAsyncFn = vi.fn().mockResolvedValue('success data')
    const { data, loading, error, execute } = useAsyncOperation(mockAsyncFn)
    
    await execute()
    
    expect(loading.value).toBe(false)
    expect(error.value).toBeNull()
    expect(data.value).toBe('success data')
  })
  
  it('should handle errors', async () => {
    const mockAsyncFn = vi.fn().mockRejectedValue(new Error('Test error'))
    const { data, loading, error, execute } = useAsyncOperation(mockAsyncFn)
    
    await execute()
    
    expect(loading.value).toBe(false)
    expect(error.value).toBe('Test error')
    expect(data.value).toBeNull()
  })
})
```

### 2. Тестирование компонентов с ошибками

```typescript
// components/UserProfile.test.ts
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import UserProfile from '@/components/UserProfile.vue'
import { createTestingPinia } from '@pinia/testing'

describe('UserProfile', () => {
  it('should display error message when API fails', async () => {
    // Мокаем API вызов для возврата ошибки
    const mockFetchUser = vi.fn().mockRejectedValue(new Error('Network error'))
    
    const wrapper = mount(UserProfile, {
      global: {
        plugins: [createTestingPinia()]
      },
      // Мокаем функцию через provide или props
    })
    
    // Имитируем действие, которое вызывает ошибку
    await wrapper.find('button').trigger('click')
    
    // Проверяем, что сообщение об ошибке отображается
    expect(wrapper.text()).toContain('Network error')
  })
})
```

## Заключение

Эффективная обработка ошибок в Vue.js приложениях требует комплексного подхода, включающего:

1. Локальную обработку ошибок в компонентах
2. Централизованную стратегию для асинхронных операций
3. Глобальные обработчики для неожиданных ошибок
4. Систему логирования для мониторинга и анализа
5. Пользовательский интерфейс для информирования пользователей об ошибках

Правильная архитектура обработки ошибок улучшает надежность приложения и用户体验, делая приложение более устойчивым к непредвиденным ситуациям и упрощая отладку и мониторинг.

## Связанные темы

- [[../architecture/error-handling|Архитектура обработки ошибок]]
- [[composables/best-practices|Лучшие практики Vue 3 Composition API]]
- [[Расширенные типы TypeScript|Расширенные типы TypeScript]]
- [[../concepts/error-handling|Фундаментальные концепции обработки ошибок]]
- [[../architecture/logging|Архитектура логирования]]

## Теги

#vuejs #error-handling #frontend #typescript #architecture #testing