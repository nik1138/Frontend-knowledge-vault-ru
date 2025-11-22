---
aliases: []
tags: [vue, testing, components]
---

# Vue Testing

Тестирование Vue-компонентов - это важная часть разработки приложений, обеспечивающая надежность, стабильность и предсказуемость кода. Vue предоставляет различные подходы и инструменты для тестирования компонентов, включая юнит-тесты, интеграционные тесты и тесты пользовательских интерфейсов.

## Подходы к тестированию

### Юнит-тестирование
- Тестирование отдельных функций и методов компонента
- Проверка реактивности и вычисляемых свойств
- Тестирование логики без отрисовки компонента

### Интеграционное тестирование
- Тестирование взаимодействия между компонентами
- Проверка передачи данных через props и события
- Тестирование сложных сценариев использования

### Тестирование пользовательского интерфейса
- Тестирование полного пользовательского опыта
- Имитация действий пользователя (клики, ввод данных)
- Проверка визуальных изменений и состояний

## Инструменты для тестирования

### Vue Test Utils
- Официальная библиотека для тестирования Vue-компонентов
- Предоставляет API для монтирования компонентов
- Поддерживает все возможности Vue

### Vitest
- Современный тестовый фреймворк, разработанный для Vite
- Быстрое выполнение тестов
- Отличная интеграция с экосистемой Vue

### Cypress
- Инструмент для E2E тестирования
- Позволяет тестировать приложение в реальном браузере
- Отлично подходит для тестирования пользовательских сценариев

### Jest
- Популярный тестовый фреймворк
- Широкие возможности для мокирования
- Хорошая интеграция с Vue Test Utils

## Основные концепции Vue Test Utils

### Монтирование компонента

```javascript
import { mount } from '@vue/test-utils'
import MyComponent from '@/components/MyComponent.vue'

const wrapper = mount(MyComponent)
```

### Проверка содержимого

```javascript
// Проверка текста
expect(wrapper.text()).toContain('Hello World')

// Проверка элемента
expect(wrapper.find('h1').exists()).toBe(true)

// Проверка атрибутов
expect(wrapper.attributes('class')).toBe('my-class')
```

### Взаимодействие с компонентом

```javascript
// Клик по элементу
await wrapper.find('button').trigger('click')

// Ввод данных
await wrapper.find('input').setValue('new value')

// Проверка состояния
expect(wrapper.vm.count).toBe(1)
```

## Тестирование Props

```javascript
import { mount } from '@vue/test-utils'
import UserProfile from '@/components/UserProfile.vue'

describe('UserProfile', () => {
  test('отображает имя пользователя', () => {
    const wrapper = mount(UserProfile, {
      props: {
        name: 'John Doe'
      }
    })
    
    expect(wrapper.text()).toContain('John Doe')
  })
})
```

## Тестирование событий

```javascript
import { mount } from '@vue/test-utils'
import Counter from '@/components/Counter.vue'

test('увеличивает счетчик при клике', async () => {
  const wrapper = mount(Counter)
  
  await wrapper.find('button').trigger('click')
  
  expect(wrapper.emitted()).toHaveProperty('increment')
  expect(wrapper.vm.count).toBe(1)
})
```

## Тестирование с Composition API

```javascript
import { mount } from '@vue/test-utils'
import { ref, computed } from 'vue'
import UserCard from '@/components/UserCard.vue'

test('вычисляемое свойство правильно работает', () => {
  const wrapper = mount(UserCard, {
    props: {
      firstName: 'John',
      lastName: 'Doe'
    }
  })
  
  expect(wrapper.vm.fullName).toBe('John Doe')
})
```

## Mocking в тестах

```javascript
import { mount } from '@vue/test-utils'
import api from '@/api'
import UserList from '@/components/UserList.vue'

jest.mock('@/api')

test('загружает и отображает пользователей', async () => {
  api.getUsers.mockResolvedValue([{ id: 1, name: 'John' }])
  
  const wrapper = mount(UserList)
  await wrapper.vm.$nextTick()
  
  expect(wrapper.findAll('.user')).toHaveLength(1)
})
```

## Тестирование асинхронных операций

```javascript
test('отображает данные после загрузки', async () => {
  const wrapper = mount(AsyncComponent)
  
  // Изначально показывается спиннер
  expect(wrapper.find('.spinner').exists()).toBe(true)
  
  // Ждем завершения асинхронной операции
  await wrapper.vm.$nextTick()
  await wrapper.vm.$nextTick()
  
  // После загрузки спиннер исчезает, появляются данные
  expect(wrapper.find('.spinner').exists()).toBe(false)
  expect(wrapper.find('.data').exists()).toBe(true)
})
```

## Лучшие практики тестирования

1. Тестировать поведение, а не реализацию
2. Писать понятные и описательные названия тестов
3. Использовать фикстуры для подготовки данных
4. Изолировать компоненты при тестировании
5. Покрывать как успешные, так и ошибочные сценарии
6. Тестировать граничные условия и ошибки

## Организация тестов

Тесты обычно размещаются в папке `tests` или рядом с компонентами:

```
src/
├── components/
│   ├── Button.vue
│   └── Button.test.js
├── views/
│   ├── Home.vue
│   └── Home.test.js
└── tests/
    ├── unit/
    └── e2e/
```

См. также: [[Vue компоненты]], [[Composition API]], [[Props and Events]]