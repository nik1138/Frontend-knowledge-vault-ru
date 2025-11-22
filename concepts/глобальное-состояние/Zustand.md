---
aliases: ["Zustand", "Zustand State Management", "Сустанд"]
tags: ["#frontend", "#state-management", "#zustand", "#javascript", "#react"]
---

# Zustand

Zustand - это легковесная библиотека для управления состоянием в JavaScript-приложениях, особенно популярная в экосистеме React. Она предоставляет простой и интуитивно понятный API без необходимости в сложной настройке.

## Основные особенности

- **Простота**: Нет необходимости в провайдерах, меньше бойлерплейта
- **Легковесность**: Маленький размер пакета (~1KB)
- **Гибкость**: Поддерживает как синхронные, так и асинхронные обновления
- **TypeScript**: Полная поддержка TypeScript
- **Мiddleware**: Возможность расширения через middleware

## Установка

```bash
npm install zustand
```

## Базовое использование

### Создание хранилища

```javascript
import { create } from 'zustand'

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}))
```

### Использование в компонентах

```jsx
import React from 'react'
import useStore from './store'

function Counter() {
  const { count, increment, decrement } = useStore()

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  )
}

export default Counter
```

## Продвинутое использование

### Разделение хранилища на части

```javascript
import { create } from 'zustand'

const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
}))

const useAppStore = create((set) => ({
  theme: 'light',
  setTheme: (theme) => set({ theme }),
  notifications: [],
  addNotification: (notification) => 
    set((state) => ({ 
      notifications: [...state.notifications, notification] 
    })),
}))
```

### Асинхронные действия

```javascript
import { create } from 'zustand'

const useApiStore = create((set) => ({
  data: null,
  loading: false,
  error: null,
  fetchData: async (url) => {
    set({ loading: true, error: null })
    try {
      const response = await fetch(url)
      const data = await response.json()
      set({ data, loading: false })
    } catch (error) {
      set({ error: error.message, loading: false })
    }
  }
}))
```

### Использование селекторов

```javascript
import { create } from 'zustand'

const useComplexStore = create((set) => ({
  users: [],
  addUser: (user) => set((state) => ({ users: [...state.users, user] })),
  getUserById: (id) => (state) => state.users.find(user => user.id === id),
  getActiveUsers: (state) => state.users.filter(user => user.active),
}))

// В компоненте
function UserProfile({ userId }) {
  const getUserById = useComplexStore(state => state.getUserById(userId))
  const user = useComplexStore(getUserById)
  
  return <div>{user?.name}</div>
}
```

## Мидлвары

Zustand поддерживает мидлвары для расширения функциональности:

### Мидлвар для логирования

```javascript
import { create } from 'zustand'

const logger = (config) => (set, get, api) => 
  config(
    (...args) => {
      console.log('Предыдущее состояние:', get())
      set(...args)
      console.log('Новое состояние:', get())
    },
    get,
    api
  )

const useLoggerStore = create(
  logger((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
  }))
)
```

### Мидлвар для сохранения в localStorage

```javascript
import { create } from 'zustand'

const persist = (config, key) => (set, get, api) => 
  config(
    (...args) => {
      set(...args)
      localStorage.setItem(key, JSON.stringify(get()))
    },
    () => {
      const saved = localStorage.getItem(key)
      return saved ? JSON.parse(saved) : {}
    },
    api
  )

const usePersistStore = create(
  persist((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
  }), 'my-storage-key')
)
```

## Сравнение с другими решениями

| Характеристика | Zustand | Redux | Context API |
|----------------|---------|-------|-------------|
| Бойлерплейт | Минимум | Много | Средне |
| Размер | ~1KB | ~10KB | Встроен |
| Кривая обучения | Низкая | Средняя | Низкая |
| Производительность | Высокая | Высокая | Зависит от использования |
| Отладка | Ограниченная | Отличная | Ограниченная |

## Лучшие практики

1. **Разделяйте состояние по фичам**: Создавайте отдельные хранилища для разных частей приложения
2. **Используйте селекторы**: Для получения только нужных данных из состояния
3. **Избегайте избыточных перерисовок**: Используйте селекторы для получения конкретных частей состояния
4. **Типизируйте состояние**: Используйте TypeScript для определения структуры состояния

### Пример с TypeScript

```typescript
import { create } from 'zustand'

interface User {
  id: number
  name: string
  email: string
}

interface UserState {
  users: User[]
  loading: boolean
  error: string | null
  addUser: (user: User) => void
  removeUser: (id: number) => void
  fetchUsers: () => Promise<void>
}

const useUserStore = create<UserState>((set) => ({
  users: [],
  loading: false,
  error: null,
  addUser: (user) => set((state) => ({ users: [...state.users, user] })),
  removeUser: (id) => set((state) => ({ 
    users: state.users.filter(user => user.id !== id) 
  })),
  fetchUsers: async () => {
    set({ loading: true })
    try {
      const response = await fetch('/api/users')
      const users = await response.json()
      set({ users, loading: false })
    } catch (error) {
      set({ error: error.message, loading: false })
    }
  }
}))
```

## Когда использовать Zustand

- Для небольших и средних приложений
- Когда нужна простота и минимализм
- Для прототипирования и MVP
- Когда Redux кажется избыточным

## Связанные концепции

- [[Redux]] - более традиционный подход к управлению состоянием
- [[Context API]] - встроенный механизм React для передачи состояния
- [[MobX]] - реактивное управление состоянием
- [[GraphQL и глобальное состояние]] - альтернативный подход к управлению данными

## Теги

#frontend #javascript #react #state-management #zustand #typescript #architecture