---
aliases: ["MobX", "MobX State Management", "Мобикс"]
tags: ["#frontend", "#state-management", "#mobx", "#javascript", "#react"]
---

# MobX

MobX - это мощная библиотека для реактивного управления состоянием, которая позволяет легко создавать масштабируемые приложения. В отличие от Redux, MobX использует подход, основанный на реактивном программировании и наблюдаемых данных.

## Основные принципы

MobX следует следующим принципам:

- **Прозрачность**: Изменения состояния автоматически распространяются по зависимостям
- **Простота**: Минимум бойлерплейта, интуитивно понятный API
- **Реактивность**: Автоматическое отслеживание зависимостей между состоянием и представлением

## Установка

```bash
npm install mobx mobx-react-lite
```

## Базовые концепции

### Observable (Наблюдаемые данные)

Это данные, которые могут быть отслежены на предмет изменений:

```javascript
import { makeAutoObservable } from 'mobx'

class CounterStore {
  count = 0

  constructor() {
    makeAutoObservable(this)
  }

  increment() {
    this.count++
  }

  decrement() {
    this.count--
  }
}

const counterStore = new CounterStore()
```

### Actions (Действия)

Функции, которые изменяют наблюдаемое состояние:

```javascript
class TodoStore {
  todos = []

  constructor() {
    makeAutoObservable(this)
  }

  addTodo(title) {
    this.todos.push({
      id: Date.now(),
      title,
      completed: false
    })
  }

  toggleTodo(id) {
    const todo = this.todos.find(todo => todo.id === id)
    if (todo) {
      todo.completed = !todo.completed
    }
  }

  removeTodo(id) {
    this.todos.remove(todo => todo.id === id)
  }
}
```

### Computed Values (Вычисляемые значения)

Значения, которые автоматически вычисляются на основе наблюдаемого состояния:

```javascript
class TodoStore {
  todos = []

  constructor() {
    makeAutoObservable(this)
  }

  // Вычисляемое значение
  get completedTodosCount() {
    return this.todos.filter(todo => todo.completed).length
  }

  get activeTodosCount() {
    return this.todos.filter(todo => !todo.completed).length
  }

  get allCompleted() {
    return this.todos.length > 0 && this.todos.every(todo => todo.completed)
  }
}
```

## Использование в React

### React Context для передачи хранилища

```jsx
import React, { createContext, useContext } from 'react'
import { TodoStore } from './stores/TodoStore'

const StoreContext = createContext()

export const StoreProvider = ({ children }) => {
  const store = new TodoStore()
  return (
    <StoreContext.Provider value={store}>
      {children}
    </StoreContext.Provider>
  )
}

export const useStore = () => useContext(StoreContext)
```

### Компонент, использующий хранилище

```jsx
import React from 'react'
import { observer } from 'mobx-react-lite'
import { useStore } from './store'

const TodoList = observer(() => {
  const store = useStore()

  return (
    <div>
      <h2>Задачи: {store.activeTodosCount} активных, {store.completedTodosCount} завершенных</h2>
      <button onClick={() => store.addTodo(`Задача ${Date.now()}`)}>
        Добавить задачу
      </button>
      
      {store.todos.map(todo => (
        <div key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => store.toggleTodo(todo.id)}
          />
          <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            {todo.title}
          </span>
          <button onClick={() => store.removeTodo(todo.id)}>Удалить</button>
        </div>
      ))}
    </div>
  )
})

export default TodoList
```

## Продвинутые возможности

### Reactions (Реакции)

Функции, которые выполняются при изменении наблюдаемых данных:

```javascript
import { reaction, when, autorun } from 'mobx'

// reaction - выполняется когда изменяется зависимое значение
const disposer1 = reaction(
  () => store.todos.length,
  (length, previousLength) => {
    console.log(`Количество задач изменилось с ${previousLength} на ${length}`)
  }
)

// autorun - автоматически запускается при каждом изменении зависимостей
const disposer2 = autorun(() => {
  console.log(`Активных задач: ${store.activeTodosCount}`)
})

// when - выполняется один раз, когда условие становится истинным
const disposer3 = when(
  () => store.allCompleted,
  () => {
    alert('Все задачи завершены!')
  }
)
```

### Actions с транзакциями

```javascript
import { runInAction } from 'mobx'

class AsyncStore {
  data = null
  loading = false

  constructor() {
    makeAutoObservable(this)
  }

  async fetchData() {
    this.loading = true
    try {
      const response = await fetch('/api/data')
      const data = await response.json()
      
      runInAction(() => {
        this.data = data
        this.loading = false
      })
    } catch (error) {
      runInAction(() => {
        this.loading = false
      })
    }
  }
}
```

## Структура приложения с MobX

### Организация хранилищ

```javascript
// stores/index.js
import { createContext, useContext } from 'react'
import { TodoStore } from './TodoStore'
import { UserStore } from './UserStore'

export class RootStore {
  constructor() {
    this.todoStore = new TodoStore()
    this.userStore = new UserStore()
  }
}

const StoreContext = createContext()

export const StoreProvider = ({ children }) => {
  const store = new RootStore()
  return (
    <StoreContext.Provider value={store}>
      {children}
    </StoreContext.Provider>
  )
}

export const useStore = () => useContext(StoreContext)
```

### Подключение к приложению

```jsx
// App.js
import React from 'react'
import { StoreProvider } from './stores'
import TodoList from './components/TodoList'

function App() {
  return (
    <StoreProvider>
      <TodoList />
    </StoreProvider>
  )
}

export default App
```

## TypeScript с MobX

```typescript
import { makeAutoObservable, observable, action, computed } from 'mobx'

interface Todo {
  id: number
  title: string
  completed: boolean
}

class TodoStore {
  todos: Todo[] = []

  constructor() {
    makeAutoObservable(this)
  }

  @observable todos: Todo[] = []

  @computed get completedTodosCount(): number {
    return this.todos.filter(todo => todo.completed).length
  }

  @action addTodo(title: string): void {
    this.todos.push({
      id: Date.now(),
      title,
      completed: false
    })
  }

  @action toggleTodo(id: number): void {
    const todo = this.todos.find(todo => todo.id === id)
    if (todo) {
      todo.completed = !todo.completed
    }
  }
}
```

## Мидлвары и расширения

### DevTools

```javascript
import { configure } from 'mobx'
import { useStrict } from 'mobx-devtools-mst'

// Включить строгий режим
configure({ enforceActions: 'always' })
```

## Преимущества MobX

- **Простота**: Меньше бойлерплейта по сравнению с Redux
- **Реактивность**: Автоматическое обновление компонентов
- **Производительность**: Эффективное отслеживание изменений
- **Гибкость**: Подходит для сложных сценариев

## Недостатки MobX

- **Кривая обучения**: Требуется понимание реактивного программирования
- **Отладка**: Сложнее отлаживать по сравнению с Redux
- **Нестандартный подход**: Может быть непривычным для разработчиков

## Лучшие практики

1. **Используйте makeAutoObservable**: Для упрощения создания наблюдаемых объектов
2. **Разделяйте хранилища по фичам**: Организуйте состояние по функциональности
3. **Используйте observer**: Для автоматического отслеживания изменений в компонентах
4. **Избегайте сложных вычислений в computed**: Они должны быть быстрыми

## Сравнение с другими решениями

| Характеристика | MobX | Redux | Zustand |
|----------------|------|-------|---------|
| Подход | Реактивный | Функциональный | Простой |
| Бойлерплейт | Минимум | Максимум | Минимум |
| Кривая обучения | Средняя | Средняя | Низкая |
| Производительность | Высокая | Высокая | Высокая |
| Отладка | Сложная | Простая | Средняя |

## Связанные концепции

- [[Redux]] - традиционный подход к управлению состоянием
- [[Zustand]] - простая альтернатива
- [[Context API]] - встроенный механизм React
- [[GraphQL и глобальное состояние]] - альтернативный подход к управлению данными

## Теги

#frontend #javascript #react #state-management #mobx #reactive-programming #architecture