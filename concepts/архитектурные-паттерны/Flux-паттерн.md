---
aliases: ["Flux-архитектура", "Facebook-Flux"]
tags: ["#архитектурный-паттерн", "#frontend", "#state-management", "#react"]
---

# Flux-паттерн

## Обзор

Flux - это архитектурный паттерн управления состоянием, разработанный Facebook (ныне Meta) для построения клиентских веб-приложений. Он предоставляет односторонний поток данных, что делает управление состоянием более предсказуемым и легким для отладки.

## Основные компоненты

### Store (Хранилище)
- **Содержит состояние приложения**
- Уведомляет подписчиков об изменениях
- Обновляется только через Actions

### Action (Действие)
- **Объект, содержащий тип и полезную нагрузку**
- Передает информацию из представления в Store
- Обычно создается через Action Creator

### Dispatcher (Диспетчер)
- **Единственный центральный диспетчер**
- Получает Actions и передает их в соответствующие Store
- Обеспечивает односторонний поток данных

### View (Представление)
- **Компоненты пользовательского интерфейса**
- Слушает изменения Store и перерисовывается
- Создает Actions в ответ на действия пользователя

## Диаграмма потока данных

```
[View] -> [Action] -> [Dispatcher] -> [Store] -> [View]
(односторонний поток)
```

## Пример реализации на JavaScript

```javascript
// Простая реализация Flux-архитектуры

// Dispatcher
class Dispatcher {
  constructor() {
    this.callbacks = [];
    this.isDispatching = false;
    this.queue = [];
    this.payload = null;
  }

  register(callback) {
    this.callbacks.push(callback);
    return this.callbacks.length - 1;
  }

  dispatch(payload) {
    if (this.isDispatching) {
      this.queue.push(payload);
      return;
    }

    this.isDispatching = true;
    this.payload = payload;

    for (let i = 0; i < this.callbacks.length; i++) {
      if (this.callbacks[i]) {
        this.callbacks[i](this.payload);
      }
    }

    this.isDispatching = false;
    this.payload = null;

    if (this.queue.length > 0) {
      this.dispatch(this.queue.shift());
    }
  }
}

// Action Types
const ActionTypes = {
  ADD_TODO: 'ADD_TODO',
  TOGGLE_TODO: 'TOGGLE_TODO',
  DELETE_TODO: 'DELETE_TODO'
};

// Action Creators
const TodoActions = {
  addTodo(text) {
    dispatcher.dispatch({
      type: ActionTypes.ADD_TODO,
      text: text
    });
  },
  
  toggleTodo(id) {
    dispatcher.dispatch({
      type: ActionTypes.TOGGLE_TODO,
      id: id
    });
  },
  
  deleteTodo(id) {
    dispatcher.dispatch({
      type: ActionTypes.DELETE_TODO,
      id: id
    });
  }
};

// Store
class TodoStore {
  constructor(dispatcher) {
    this.todos = [];
    this.dispatcherIndex = dispatcher.register(this.handleAction.bind(this));
  }

  handleAction(action) {
    switch (action.type) {
      case ActionTypes.ADD_TODO:
        this.todos.push({
          id: this.todos.length + 1,
          text: action.text,
          completed: false
        });
        this.emitChange();
        break;
        
      case ActionTypes.TOGGLE_TODO:
        const todo = this.todos.find(t => t.id === action.id);
        if (todo) {
          todo.completed = !todo.completed;
          this.emitChange();
        }
        break;
        
      case ActionTypes.DELETE_TODO:
        this.todos = this.todos.filter(t => t.id !== action.id);
        this.emitChange();
        break;
    }
  }

  getAllTodos() {
    return this.todos;
  }

  emitChange() {
    // Уведомление подписчиков об изменении
    if (this.onChange) {
      this.onChange();
    }
  }

  addChangeListener(callback) {
    this.onChange = callback;
  }
}

// Использование
const dispatcher = new Dispatcher();
const todoStore = new TodoStore(dispatcher);

// Подписка представления на изменения Store
todoStore.addChangeListener(() => {
  console.log('Todos обновлены:', todoStore.getAllTodos());
});

// Пример использования Actions
TodoActions.addTodo('Изучить Flux');
TodoActions.addTodo('Создать приложение');
TodoActions.toggleTodo(1);
```

## Современная реализация с использованием Redux (наследник Flux)

```javascript
import { createStore } from 'redux';

// Reducer
function todoReducer(state = { todos: [] }, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: state.todos.length + 1,
            text: action.text,
            completed: false
          }
        ]
      };
      
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.id ? { ...todo, completed: !todo.completed } : todo
        )
      };
      
    default:
      return state;
  }
}

// Store
const store = createStore(todoReducer);

// Subscribe to store changes
store.subscribe(() => {
  console.log('State updated:', store.getState());
});

// Actions
store.dispatch({ type: 'ADD_TODO', text: 'Изучить Redux' });
store.dispatch({ type: 'ADD_TODO', text: 'Создать приложение' });
store.dispatch({ type: 'TOGGLE_TODO', id: 1 });
```

## Преимущества

- **Предсказуемость** - односторонний поток данных делает поведение приложения предсказуемым
- **Отладка** - легче отследить, как данные изменяются и когда
- **Тестируемость** - состояния и изменения легко тестировать изолированно
- **Масштабируемость** - позволяет управлять сложным состоянием приложения

## Недостатки

- **Сложность для простых приложений** - может быть избыточным для небольших проектов
- **Бойлерплейт** - требует создания множества промежуточных компонентов
- **Кривая обучения** - новичкам может быть сложно понять концепцию

## Когда использовать

- В приложениях с сложным состоянием
- Когда необходимо синхронизировать данные между несколькими компонентами
- В приложениях, где состояние часто изменяется
- В сочетании с React (изначально разработан для React)

## Связанные концепции

- [[Архитектурные-паттерны]] - общее понятие о паттернах архитектуры
- [[Redux]] - популярная реализация Flux-паттерна
- [[Глобальное-состояние]] - управление состоянием приложения
- [[React]] - фреймворк, для которого был разработан Flux
- [[MVC-паттерн]] - альтернативный подход к архитектуре
- [[Flux-паттерн]] - более сложный вариант Flux

## Заключение

Flux-паттерн стал основой для многих современных библиотек управления состоянием, таких как Redux. Его концепция одностороннего потока данных помогает создавать более предсказуемые и легко тестируемые приложения.