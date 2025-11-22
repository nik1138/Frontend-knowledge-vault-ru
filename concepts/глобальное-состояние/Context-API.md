---
aliases: ["Context API", "React Context", "Контекст API"]
tags: ["#frontend", "#state-management", "#react", "#context", "#javascript"]
---

# Context API

Context API - это встроенный механизм React для передачи данных через дерево компонентов без необходимости передавать пропсы на каждом уровне. Он особенно полезен для управления глобальным состоянием приложения.

## Основные концепции

Context API состоит из трех основных компонентов:

1. **Context** - объект, который хранит значение
2. **Provider** - компонент, который предоставляет значение контекста дочерним компонентам
3. **Consumer** - компонент, который получает значение из контекста (альтернатива - хук `useContext`)

## Создание и использование контекста

### Базовое создание контекста

```javascript
import React, { createContext, useContext, useReducer } from 'react'

// Создание контекста с начальным значением
const AppContext = createContext()

// Редьюсер для управления состоянием
const appReducer = (state, action) => {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload }
    case 'SET_THEME':
      return { ...state, theme: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
    default:
      return state
  }
}

// Компонент-провайдер
export const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, {
    user: null,
    theme: 'light',
    loading: false
  })

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  )
}

// Хук для использования контекста
export const useAppContext = () => {
  const context = useContext(AppContext)
  if (!context) {
    throw new Error('useAppContext must be used within an AppProvider')
  }
  return context
}
```

### Использование в компонентах

```jsx
import React from 'react'
import { useAppContext } from './AppContext'

const UserProfile = () => {
  const { state, dispatch } = useAppContext()

  const handleLogin = () => {
    dispatch({
      type: 'SET_USER',
      payload: { id: 1, name: 'John Doe', email: 'john@example.com' }
    })
  }

  const handleLogout = () => {
    dispatch({ type: 'SET_USER', payload: null })
  }

  return (
    <div>
      {state.user ? (
        <div>
          <p>Привет, {state.user.name}!</p>
          <button onClick={handleLogout}>Выйти</button>
        </div>
      ) : (
        <button onClick={handleLogin}>Войти</button>
      )}
    </div>
  )
}

export default UserProfile
```

## Продвинутое использование

### Множественные контексты

```jsx
import React, { createContext, useContext } from 'react'

// Контекст для аутентификации
const AuthContext = createContext()
// Контекст для настроек
const SettingsContext = createContext()

// Компонент, объединяющий несколько провайдеров
export const AppProviders = ({ children }) => (
  <AuthContext.Provider value={authValue}>
    <SettingsContext.Provider value={settingsValue}>
      {children}
    </SettingsContext.Provider>
  </AuthContext.Provider>
)

// Компонент, использующий несколько контекстов
const Dashboard = () => {
  const auth = useContext(AuthContext)
  const settings = useContext(SettingsContext)

  return (
    <div>
      <h1>Панель управления</h1>
      <p>Пользователь: {auth.user?.name}</p>
      <p>Тема: {settings.theme}</p>
    </div>
  )
}
```

### Кастомные хуки для контекста

```javascript
import { useContext } from 'react'
import { AppContext } from './AppContext'

// Хуки для конкретных частей состояния
export const useUser = () => {
  const { state, dispatch } = useContext(AppContext)
  return {
    user: state.user,
    setUser: (user) => dispatch({ type: 'SET_USER', payload: user })
  }
}

export const useTheme = () => {
  const { state, dispatch } = useContext(AppContext)
  return {
    theme: state.theme,
    setTheme: (theme) => dispatch({ type: 'SET_THEME', payload: theme })
  }
}

export const useLoading = () => {
  const { state, dispatch } = useContext(AppContext)
  return {
    loading: state.loading,
    setLoading: (loading) => dispatch({ type: 'SET_LOADING', payload: loading })
  }
}
```

## Управление сложным состоянием

### Комбинирование Context API и useReducer

```javascript
import React, { createContext, useContext, useReducer } from 'react'

// Определение начального состояния
const initialState = {
  user: null,
  cart: [],
  products: [],
  loading: false,
  error: null
}

// Определение действий
const actions = {
  SET_USER: 'SET_USER',
  ADD_TO_CART: 'ADD_TO_CART',
  REMOVE_FROM_CART: 'REMOVE_FROM_CART',
  SET_PRODUCTS: 'SET_PRODUCTS',
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR'
}

// Редьюсер
const appReducer = (state, action) => {
  switch (action.type) {
    case actions.SET_USER:
      return { ...state, user: action.payload }
    case actions.ADD_TO_CART:
      return { 
        ...state, 
        cart: [...state.cart, action.payload] 
      }
    case actions.REMOVE_FROM_CART:
      return { 
        ...state, 
        cart: state.cart.filter(item => item.id !== action.payload) 
      }
    case actions.SET_PRODUCTS:
      return { ...state, products: action.payload }
    case actions.SET_LOADING:
      return { ...state, loading: action.payload }
    case actions.SET_ERROR:
      return { ...state, error: action.payload }
    default:
      return state
  }
}

// Контекст
const AppContext = createContext()

// Провайдер
export const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState)

  const value = {
    state,
    dispatch,
    // Вспомогательные функции
    addToCart: (item) => dispatch({ type: actions.ADD_TO_CART, payload: item }),
    removeFromCart: (itemId) => dispatch({ type: actions.REMOVE_FROM_CART, payload: itemId })
  }

  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  )
}

// Использование в компонентах
const ProductList = () => {
  const { state, addToCart } = useContext(AppContext)

  return (
    <div>
      {state.products.map(product => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>{product.price}</p>
          <button onClick={() => addToCart(product)}>Добавить в корзину</button>
        </div>
      ))}
    </div>
  )
}
```

## Оптимизация производительности

### Использование useMemo для значений контекста

```jsx
import React, { createContext, useContext, useReducer, useMemo } from 'react'

const AppContext = createContext()

export const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState)

  // Использование useMemo для предотвращения ненужных перерендеров
  const contextValue = useMemo(() => ({
    state,
    dispatch,
    addToCart: (item) => dispatch({ type: 'ADD_TO_CART', payload: item }),
    removeFromCart: (id) => dispatch({ type: 'REMOVE_FROM_CART', payload: id })
  }), [state])

  return (
    <AppContext.Provider value={contextValue}>
      {children}
    </AppContext.Provider>
  )
}
```

### Разделение контекста для изоляции обновлений

```jsx
// Вместо одного большого контекста
const UserContext = createContext()
const CartContext = createContext()
const ThemeContext = createContext()

// Это позволяет обновлять части состояния независимо
const App = () => (
  <UserContext.Provider value={userState}>
    <CartContext.Provider value={cartState}>
      <ThemeContext.Provider value={themeState}>
        <MainApp />
      </ThemeContext.Provider>
    </CartContext.Provider>
  </UserContext.Provider>
)
```

## Паттерны использования

### Паттерн "Context Module"

```javascript
// context/AppContext.js
import React, { createContext, useContext, useReducer } from 'react'

// Action creators
export const setUser = (user) => ({ type: 'SET_USER', payload: user })
export const setTheme = (theme) => ({ type: 'SET_THEME', payload: theme })

// Context
const AppContext = createContext()

// Provider
export const AppProvider = ({ children, initialState = {} }) => {
  const [state, dispatch] = useReducer(appReducer, { ...initialState })

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  )
}

// Custom hook
export const useApp = () => {
  const context = useContext(AppContext)
  if (!context) {
    throw new Error('useApp must be used within an AppProvider')
  }
  return context
}

// Использование
const SomeComponent = () => {
  const { state, dispatch } = useApp()
  
  return (
    <button onClick={() => dispatch(setUser({ name: 'John' }))}>
      Установить пользователя
    </button>
  )
}
```

## Преимущества Context API

- **Встроен в React**: Не требует дополнительных зависимостей
- **Простота**: Легко начать использовать
- **Гибкость**: Подходит для различных сценариев
- **Интеграция**: Хорошо работает с остальными возможностями React

## Ограничения и недостатки

- **Производительность**: При частых изменениях может вызывать лишние перерендеры
- **Отладка**: Сложнее отлаживать по сравнению с Redux
- **Масштабирование**: Для сложных приложений может потребоваться дополнительная архитектура
- **Тестирование**: Более сложное тестирование компонентов

## Когда использовать Context API

- Для передачи данных, которые нужны нескольким компонентам
- Для управления настройками приложения (тема, язык и т.д.)
- Для аутентификации и управления пользовательским сеансом
- В небольших и средних приложениях
- Когда Redux или другие решения кажутся избыточными

## Когда НЕ использовать Context API

- Для часто изменяющихся данных (может вызвать лишние перерендеры)
- Для сложного глобального состояния с множеством зависимостей
- Когда нужна сложная логика управления состоянием
- Когда необходимы продвинутые возможности отладки

## Лучшие практики

1. **Изолируйте часто изменяющиеся данные**: Разделяйте контексты для разных типов данных
2. **Используйте useMemo**: Для предотвращения ненужных перерендеров
3. **Комбинируйте с другими решениями**: Используйте Context API вместе с Redux или Zustand при необходимости
4. **Организуйте код**: Разделяйте контексты по функциональности

## Сравнение с другими решениями

| Характеристика | Context API | Redux | Zustand | MobX |
|----------------|-------------|-------|---------|------|
| Бойлерплейт | Средний | Высокий | Минимальный | Минимальный |
| Производительность | Зависит от использования | Высокая | Высокая | Высокая |
| Кривая обучения | Низкая | Средняя | Низкая | Средняя |
| Отладка | Ограниченная | Отличная | Средняя | Средняя |
| Размер | Встроен | ~10KB | ~1KB | ~4KB |

## Связанные концепции

- [[Redux]] - более мощное решение для управления состоянием
- [[Zustand]] - легковесная альтернатива
- [[MobX]] - реактивное управление состоянием
- [[GraphQL и глобальное состояние]] - альтернативный подход к управлению данными

## Теги

#frontend #javascript #react #state-management #context-api #architecture #usecontext