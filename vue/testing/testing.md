# Testing

Тестирование в Vue.js - это практика проверки корректности работы компонентов, логики и пользовательского интерфейса. Включает в себя различные типы тестов для обеспечения качества кода.

## Типы тестирования

### [Unit Testing](unit-testing.md)
Тестирование отдельных компонентов и функций изолированно

### [Component Testing](component-testing.md)
Тестирование Vue компонентов с их DOM представлением

### [Integration Testing](integration-testing.md)
Тестирование взаимодействия между компонентами

### [E2E Testing](e2e-testing.md)
Сквозное тестирование пользовательских сценариев

## Основные инструменты

### [Vitest](vitest.md)
Современный тестовый раннер для Vue и JavaScript

### [Vue Test Utils](vue-test-utils.md)
Официальные утилиты для тестирования Vue компонентов

### [Cypress](cypress.md)
Инструмент для End-to-End тестирования

### [Jest](jest.md)
Популярный фреймворк для unit тестирования (менее предпочтителен в новых проектах)

## Основные концепции тестирования

### Мокирование
```javascript
// Пример мокирования
import { mount } from '@vue/test-utils'
import { vi } from 'vitest'

const mockFunction = vi.fn()

const wrapper = mount(Component, {
  props: {
    onClick: mockFunction
  }
})
```

### Фикстуры
```javascript
// Использование тестовых данных
const testData = {
  user: { id: 1, name: 'John' },
  posts: [{ id: 1, title: 'Post 1' }]
}
```

## Примеры тестов

### Простой unit тест
```javascript
import { describe, it, expect } from 'vitest'
import { mount } from '@vue/test-utils'
import MyComponent from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('отображает приветствие', () => {
    const wrapper = mount(MyComponent, {
      props: { message: 'Привет' }
    })
    
    expect(wrapper.text()).toContain('Привет')
  })
})
```

### Тест с асинхронными операциями
```javascript
it('загружает данные', async () => {
  const wrapper = mount(AsyncComponent)
  
  // Ждем завершения асинхронной операции
  await wrapper.vm.loadData()
  
  expect(wrapper.find('.data').exists()).toBe(true)
})
```

## Best Practices

### Тестирование компонентов
1. Тестируйте поведение, не реализацию
2. Используйте стабильные селекторы
3. Проверяйте пользовательские взаимодействия
4. Изолируйте компонент от внешних зависимостей

### Структура тестов (AAA)
- **Arrange** - подготовка теста
- **Act** - выполнение действия
- **Assert** - проверка результата

## Тестирование с Composition API

```javascript
import { setActivePinia, createPinia } from 'pinia'
import { mount } from '@vue/test-utils'

beforeEach(() => {
  setActivePinia(createPinia())
})
```

## Связь с другими концепциями
- [[components/Components]] - тестирование компонентной логики
- [[composables/Composables]] - тестирование логики за пределами компонентов
- [[state-management/State Management]] - тестирование глобального состояния