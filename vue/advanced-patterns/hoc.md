# Higher-Order Components (HOC)

## Определение
Higher-Order Components (HOC) - это паттерн, при котором функция принимает компонент и возвращает новый компонент с дополнительной функциональностью. Это популярный паттерн в React, но может быть реализован и в Vue, хотя в Vue Composition API часто предпочтительнее использовать композаблы.

## Реализация HOC в Vue

### Простой HOC
```javascript
// withLoading.js
import { h, ref, onMounted } from 'vue'

export function withLoading(WrappedComponent, loaderComponent) {
  return {
    props: WrappedComponent.props || {},
    
    setup(props) {
      const loading = ref(true)
      const data = ref(null)
      
      onMounted(async () => {
        // Симуляция загрузки данных
        await new Promise(resolve => setTimeout(resolve, 1000))
        loading.value = false
      })
      
      return { loading, data, ...props }
    },
    
    render() {
      if (this.loading) {
        return h(loaderComponent || 'div', 'Загрузка...')
      }
      
      // Передаем пропсы и слоты оригинальному компоненту
      return h(WrappedComponent, this.$props, this.$slots)
    }
  }
}

// Использование
const MyComponentWithLoading = withLoading(MyComponent, MyLoader)
```

### HOC с аутентификацией
```javascript
// withAuth.js
import { h, inject } from 'vue'

export function withAuth(WrappedComponent, redirect = '/login') {
  return {
    setup() {
      const currentUser = inject('currentUser', null)
      
      return { currentUser }
    },
    
    render() {
      if (!this.currentUser) {
        // Редирект на страницу логина
        this.$router.push(redirect)
        return h('div', 'Перенаправление...')
      }
      
      return h(WrappedComponent, this.$props, this.$slots)
    }
  }
}
```

### HOC с логированием
```javascript
// withLogging.js
import { h, getCurrentInstance } from 'vue'

export function withLogging(WrappedComponent) {
  return {
    name: `Logged${WrappedComponent.name || 'Component'}`,
    
    created() {
      console.log(`Компонент ${this.$options.name} создан`)
    },
    
    updated() {
      console.log(`Компонент ${this.$options.name} обновлен`)
    },
    
    render() {
      return h(WrappedComponent, this.$props, this.$slots)
    }
  }
}
```

## Практические примеры

### HOC для управления состоянием
```javascript
// withState.js
import { h, ref, reactive } from 'vue'

export function withState(WrappedComponent, initialState = {}) {
  return {
    props: WrappedComponent.props || {},
    
    setup(props) {
      const state = reactive({ ...initialState })
      
      const setState = (newState) => {
        Object.assign(state, newState)
      }
      
      return { state, setState, ...props }
    },
    
    render() {
      // Передаем состояние как дополнительные пропсы
      return h(WrappedComponent, {
        ...this.$props,
        state: this.state,
        setState: this.setState
      }, this.$slots)
    }
  }
}
```

### HOC для обработки ошибок
```javascript
// withErrorBoundary.js
import { h, onErrorCaptured, ref } from 'vue'

export function withErrorBoundary(WrappedComponent, fallbackComponent) {
  return {
    setup() {
      const hasError = ref(false)
      const error = ref(null)
      
      onErrorCaptured((err) => {
        hasError.value = true
        error.value = err
        console.error('Ошибка в компоненте:', err)
        return false // Не передаем ошибку выше
      })
      
      return { hasError, error }
    },
    
    render() {
      if (this.hasError) {
        return h(fallbackComponent || 'div', `Ошибка: ${this.error.message}`)
      }
      
      return h(WrappedComponent, this.$props, this.$slots)
    }
  }
}
```

## Сравнение с Composition API

### HOC vs Композаблы
```javascript
// HOC подход (менее рекомендуемый в Vue)
const EnhancedComponent = withLogging(
  withAuth(
    withLoading(MyComponent, Loader)
  )
)
```

```javascript
// Composition API подход (рекомендуемый)
<script setup>
import { useAuth } from '@/composables/useAuth'
import { useLoading } from '@/composables/useLoading'

const { currentUser, isAuthenticated } = useAuth()
const { loading, data } = useLoading()
</script>
```

### Преимущества композаблов над HOC
1. **Лучшая читаемость** - ясно видно, что используется
2. **Лучшая типизация** - легче документировать и типизировать
3. **Отладка** - легче отлаживать и тестировать
4. **Производительность** - меньше вложенных компонентов

## Фабрика HOC

### Универсальный HOC
```javascript
// hocFactory.js
import { h } from 'vue'

export function createHOC(enhancementFn) {
  return (WrappedComponent) => {
    return {
      props: WrappedComponent.props || {},
      
      setup(props, { slots }) {
        const enhanced = enhancementFn()
        
        return { ...props, ...enhanced }
      },
      
      render() {
        return h(WrappedComponent, this.$props, slots)
      }
    }
  }
}

// Пример использования
const withWindowSize = createHOC(() => {
  const width = ref(window.innerWidth)
  const height = ref(window.innerHeight)
  
  const handleResize = () => {
    width.value = window.innerWidth
    height.value = window.innerHeight
  }
  
  onMounted(() => {
    window.addEventListener('resize', handleResize)
  })
  
  onUnmounted(() => {
    window.removeEventListener('resize', handleResize)
  })
  
  return { width, height }
})
```

## Комбинирование HOC

```javascript
// Комбинирование нескольких HOC
const enhanceComponent = (Component) => 
  withLogging(
    withAuth(
      withState(
        withLoading(Component, CustomLoader),
        { count: 0 }
      )
    )
  )

const EnhancedComponent = enhanceComponent(MyComponent)
```

## Ограничения HOC

### Проблемы с HOC
1. **Имена пропсов** - конфликты между HOC
2. **Отладка** - сложнее понимать, откуда что пришло
3. **Производительность** - создание множества оберток
4. **Типизация** - сложнее определить типы

## Современные альтернативы

### Composition API
```javascript
// Лучшая альтернатива - композаблы
<script setup>
import { useAuth } from '@/composables/useAuth'
import { useWindowSize } from '@/composables/useWindowSize'
import { useApi } from '@/composables/useApi'

// Все вместе в одном месте
const { user, login, logout } = useAuth()
const { width, height } = useWindowSize()
const { data, loading, error } = useApi('/api/data')
</script>
```

### Renderless компоненты
```vue
<template>
  <AuthWrapper>
    <DataFetcher url="/api/data">
      <WindowSize>
        <template #default="{ user, data, width, height }">
          <MyComponent :user="user" :data="data" :size="{ width, height }" />
        </template>
      </WindowSize>
    </DataFetcher>
  </AuthWrapper>
</template>
```

## Тестирование HOC
```javascript
import { mount } from '@vue/test-utils'
import { withAuth } from '@/hocs/withAuth'
import MyComponent from '@/components/MyComponent.vue'

test('рендерит компонент если пользователь аутентифицирован', () => {
  const WrapperComponent = withAuth(MyComponent)
  const wrapper = mount(WrapperComponent, {
    global: {
      provide: {
        currentUser: { id: 1, name: 'John' }
      }
    }
  })
  
  expect(wrapper.findComponent(MyComponent).exists()).toBe(true)
})

test('редиректит если пользователь не аутентифицирован', () => {
  const mockPush = jest.fn()
  const WrapperComponent = withAuth(MyComponent)
  
  const wrapper = mount(WrapperComponent, {
    global: {
      provide: {
        currentUser: null
      },
      mocks: {
        $router: { push: mockPush }
      }
    }
  })
  
  expect(mockPush).toHaveBeenCalledWith('/login')
})
```

## Когда использовать HOC

**Используйте HOC когда:**
- Нужно применить одинаковую логику к множеству компонентов
- Миграция из React приложения
- В библиотечном коде для предоставления абстракций

**Используйте композаблы когда:**
- Создание нового кода на Vue
- Нужна лучшая читаемость и типизация
- Необходима гибкость и переиспользуемость

## Связь с другими концепциями
- [[composables/Composables]] - современная альтернатива HOC
- [[advanced-patterns/renderless]] - альтернативный паттерн
- [[components/Components]] - база для создания HOC
- [[reactivity/Reactivity System]] - реактивность в HOC