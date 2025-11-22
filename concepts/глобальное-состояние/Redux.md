---
aliases: ["Redux", "Редакс", "Redux Toolkit", "RTK"]
tags: ["#frontend", "#state-management", "#redux", "#javascript", "#react"]
---

# Redux

Redux - это предсказуемый контейнер состояния для JavaScript-приложений. Он помогает писать приложения, которые ведут себя предсказуемо, работают одинаково во всех средах и легко тестируются.

## Основные концепции

Redux основан на следующих ключевых принципах:

- **Единое состояние (Single Source of Truth)**: Всё состояние вашего приложения хранится в одном дереве состояния в хранилище.
- **Состояние только для чтения (State is Read-Only)**: Единственный способ изменить состояние - это вызвать действие (action).
- **Изменения через чистые функции (Changes are Made with Pure Functions)**: Редьюсеры описывают, как каждое действие изменяет состояние.

### Основные компоненты

1. **Store (Хранилище)**: Объект, который хранит состояние приложения
2. **Actions (Действия)**: Объекты, которые описывают, что произошло
3. **Reducers (Редьюсеры)**: Чистые функции, которые определяют, как состояние изменяется в ответ на действия

## Установка и настройка

```bash
npm install redux react-redux @reduxjs/toolkit
```

Redux Toolkit (RTK) - это рекомендуемый способ использования Redux, который упрощает создание хранилища и работу с ним.

### Пример использования Redux Toolkit

```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit'

// Создание слайса - части состояния с редьюсерами
const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: (state) => {
      state.value += 1
    },
    decrement: (state) => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

// Экспорт действий
export const { increment, decrement, incrementByAmount } = counterSlice.actions

// Создание хранилища
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
})

export default store
```

### Использование в React-компонентах

```jsx
import React from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { increment, decrement, incrementByAmount } from './store/counterSlice'

function Counter() {
  const count = useSelector((state) => state.counter.value)
  const dispatch = useDispatch()

  return (
    <div>
      <div>
        <button aria-label="Increment value" onClick={() => dispatch(increment())}>
          +
        </button>
        <span>{count}</span>
        <button aria-label="Decrement value" onClick={() => dispatch(decrement())}>
          -
        </button>
      </div>
      <button onClick={() => dispatch(incrementByAmount(5))}>
        Добавить 5
      </button>
    </div>
  )
}

export default Counter
```

## Архитектура Redux

### Actions

Действия - это простые JavaScript-объекты, которые описывают, что произошло в приложении. У них обязательно должен быть тип (type):

```javascript
{
  type: 'counter/increment',
  payload: 5
}
```

### Reducers

Редьюсеры - это чистые функции, которые принимают предыдущее состояние и действие, а затем возвращают новое состояние:

```javascript
function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/increment':
      return { value: state.value + 1 }
    case 'counter/decrement':
      return { value: state.value - 1 }
    case 'counter/incrementByAmount':
      return { value: state.value + action.payload }
    default:
      return state
  }
}
```

### Store

Хранилище связывает воедино редьюсеры и действия:

```javascript
import { createStore } from 'redux'
import counterReducer from './reducers/counterReducer'

const store = createStore(counterReducer)
```

## Redux Toolkit

Redux Toolkit включает в себя:

- `configureStore`: Упрощает настройку хранилища с хорошими настройками по умолчанию
- `createSlice`: Создает редьюсеры и действия в одном месте
- `createAsyncThunk`: Упрощает обработку асинхронных операций
- `createSelector`: Реализует мемоизацию селекторов

### Пример асинхронного действия

```javascript
import { createAsyncThunk, createSlice } from '@reduxjs/toolkit'
import { userAPI } from './userAPI'

// Создание асинхронного действия
export const fetchUserById = createAsyncThunk(
  'users/fetchByIdStatus',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await userAPI.fetchById(userId)
      return response.data
    } catch (error) {
      return rejectWithValue(error.response.data)
    }
  }
)

const usersSlice = createSlice({
  name: 'users',
  initialState: { entities: [], loading: 'idle' },
  reducers: {
    // обычные редьюсеры
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state, action) => {
        state.loading = 'pending'
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.loading = 'idle'
        state.entities.push(action.payload)
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.loading = 'idle'
      })
  }
})
```

## Преимущества Redux

- **Предсказуемость**: Поскольку Redux следует строгим принципам, состояние приложения становится предсказуемым
- **Отладка**: Redux DevTools позволяют отслеживать изменения состояния и действия
- **Тестируемость**: Чистые функции легко тестировать
- **Масштабируемость**: Подходит для больших приложений с сложной логикой

## Недостатки Redux

- **Бойлерплейт**: Требуется много кода для простых операций
- **Сложность для новичков**: Кривая обучения может быть крутой
- **Производительность**: При неправильном использовании может привести к проблемам с производительностью

## Лучшие практики

1. **Используйте Redux Toolkit**: Он упрощает большинство задач
2. **Нормализуйте данные**: Используйте `normalizr` для структурирования сложных данных
3. **Организуйте код по фичам**: Группируйте действия, редьюсеры и селекторы по функциональности
4. **Используйте мемоизацию**: Применяйте `createSelector` для оптимизации вычислений

## Связанные концепции

- [[Context API]] - альтернативный способ управления состоянием в React
- [[Zustand]] - современная альтернатива Redux
- [[MobX]] - реактивное управление состоянием
- [[GraphQL и глобальное состояние]] - использование GraphQL для управления состоянием

## Теги

#frontend #javascript #react #state-management #redux #redux-toolkit #rtk #architecture