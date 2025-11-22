---
aliases: ["Управление состоянием", "State Management", "Состояние приложения"]
tags: [js, state, frontend, practical-examples]
---

# Управление состоянием приложения

## Введение в управление состоянием

Управление состоянием - это подход к хранению, обновлению и отслеживанию данных в приложении. Правильное управление состоянием делает приложение более предсказуемым, тестируемым и масштабируемым.

## Простые паттерны управления состоянием

### Глобальный объект состояния

```javascript
// Простой глобальный объект состояния
const AppState = {
  user: null,
  todos: [],
  filters: {
    showCompleted: true,
    searchQuery: ''
  },
  
  // Методы для обновления состояния
  setUser(user) {
    this.user = user;
    this.notifySubscribers('user');
  },
  
  addTodo(todo) {
    this.todos.push(todo);
    this.notifySubscribers('todos');
  },
  
  updateTodo(id, updates) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      Object.assign(todo, updates);
      this.notifySubscribers('todos');
    }
  },
  
  // Уведомление подписчиков об изменениях
  subscribers: {},
  
  subscribe(event, callback) {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
  },
  
  notifySubscribers(event) {
    if (this.subscribers[event]) {
      this.subscribers[event].forEach(callback => callback(this[event]));
    }
  }
};
```

### Класс для управления состоянием

```javascript
class StateManager {
  constructor(initialState = {}) {
    this.state = { ...initialState };
    this.subscribers = [];
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.notifySubscribers();
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers() {
    this.subscribers.forEach(callback => callback(this.getState()));
  }
  
  // Метод для обновления части состояния
  patchState(updates) {
    this.setState(updates);
  }
  
  // Метод для получения конкретного свойства
  get(key) {
    return this.state[key];
  }
  
  // Метод для установки конкретного свойства
  set(key, value) {
    this.setState({ [key]: value });
  }
}

// Использование
const appState = new StateManager({
  user: null,
  todos: [],
  loading: false
});

// Подписка на изменения
const unsubscribe = appState.subscribe((newState) => {
  console.log('Состояние изменилось:', newState);
});

// Обновление состояния
appState.set('user', { id: 1, name: 'John' });
appState.patchState({ loading: true });
```

## Паттерн "Хранилище" (Store Pattern)

```javascript
class Store {
  constructor(reducer, initialState) {
    this.reducer = reducer;
    this.state = initialState;
    this.subscribers = [];
  }
  
  getState() {
    return this.state;
  }
  
  dispatch(action) {
    this.state = this.reducer(this.state, action);
    this.notifySubscribers();
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers() {
    const state = this.getState();
    this.subscribers.forEach(callback => callback(state));
  }
}

// Пример использования с редьюсером
function todoReducer(state = { todos: [], filter: 'all' }, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
    default:
      return state;
  }
}

const store = new Store(todoReducer, { todos: [], filter: 'all' });

// Подписка на изменения
store.subscribe((state) => {
  console.log('Новое состояние:', state);
});
```

## Управление состоянием с помощью Proxy

```javascript
class ReactiveState {
  constructor(initialState) {
    this.subscribers = [];
    this.state = this.createProxy(initialState);
  }
  
  createProxy(obj, path = '') {
    return new Proxy(obj, {
      set(target, property, value) {
        const oldValue = target[property];
        target[property] = value;
        
        // Уведомление подписчиков об изменении
        const fullPath = path ? `${path}.${property}` : property;
        this.notifySubscribers(fullPath, value, oldValue);
        
        // Если значение - объект, оборачиваем его в Proxy
        if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
          target[property] = this.createProxy(value, fullPath);
        }
        
        return true;
      }
    });
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers(path, newValue, oldValue) {
    this.subscribers.forEach(callback => callback(path, newValue, oldValue));
  }
  
  getState() {
    return this.state;
  }
}

// Использование
const reactiveState = new ReactiveState({
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark', notifications: true }
});

reactiveState.subscribe((path, newValue, oldValue) => {
  console.log(`Путь ${path} изменился с ${oldValue} на ${newValue}`);
});

// При изменении свойств автоматически вызываются подписчики
reactiveState.state.user.name = 'Jane'; // Вызовет подписчиков
```

## Управление состоянием с LocalStorage

```javascript
class PersistentState {
  constructor(key, initialState) {
    this.key = key;
    this.state = this.loadState() || initialState;
    this.subscribers = [];
  }
  
  loadState() {
    try {
      const serializedState = localStorage.getItem(this.key);
      return serializedState ? JSON.parse(serializedState) : null;
    } catch (error) {
      console.error('Ошибка при загрузке состояния:', error);
      return null;
    }
  }
  
  saveState() {
    try {
      const serializedState = JSON.stringify(this.state);
      localStorage.setItem(this.key, serializedState);
    } catch (error) {
      console.error('Ошибка при сохранении состояния:', error);
    }
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.saveState();
    this.notifySubscribers();
  }
  
  subscribe(callback) {
    this.subscribers.push(callback);
    return () => {
      this.subscribers = this.subscribers.filter(sub => sub !== callback);
    };
  }
  
  notifySubscribers() {
    this.subscribers.forEach(callback => callback(this.getState()));
  }
  
  // Метод для сброса состояния
  reset() {
    this.state = {};
    localStorage.removeItem(this.key);
    this.notifySubscribers();
  }
}

// Использование
const userPreferences = new PersistentState('userPreferences', {
  theme: 'light',
  language: 'ru',
  notifications: true
});
```

## Интеграция с DOM

```javascript
class StatefulComponent {
  constructor(selector, initialState = {}) {
    this.element = document.querySelector(selector);
    this.stateManager = new StateManager(initialState);
    this.render = this.render.bind(this);
    
    // Подписка на изменения состояния
    this.stateManager.subscribe(this.render);
  }
  
  setState(newState) {
    this.stateManager.setState(newState);
  }
  
  getState() {
    return this.stateManager.getState();
  }
  
  // Метод рендеринга должен быть переопределен
  render() {
    // Переопределяется в наследниках
  }
  
  // Метод для обновления DOM
  updateDOM(html) {
    this.element.innerHTML = html;
  }
}

// Пример компонента списка задач
class TodoList extends StatefulComponent {
  constructor(selector) {
    super(selector, {
      todos: [],
      newTodoText: ''
    });
  }
  
  render() {
    const state = this.getState();
    const filteredTodos = state.todos.filter(todo => !todo.completed);
    
    const html = `
      <div class="todo-list">
        <input 
          type="text" 
          placeholder="Добавить задачу" 
          value="${state.newTodoText}"
          oninput="todoList.handleInput(event)"
        >
        <button onclick="todoList.addTodo()">Добавить</button>
        <ul>
          ${filteredTodos.map(todo => `
            <li>
              <span>${todo.text}</span>
              <button onclick="todoList.toggleTodo(${todo.id})">✓</button>
            </li>
          `).join('')}
        </ul>
      </div>
    `;
    
    this.updateDOM(html);
  }
  
  handleInput(event) {
    this.setState({ newTodoText: event.target.value });
  }
  
  addTodo() {
    const state = this.getState();
    if (state.newTodoText.trim()) {
      const newTodo = {
        id: Date.now(),
        text: state.newTodoText,
        completed: false
      };
      
      this.setState({
        todos: [...state.todos, newTodo],
        newTodoText: ''
      });
    }
  }
  
  toggleTodo(id) {
    const state = this.getState();
    const todos = state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    
    this.setState({ todos });
  }
}

// Инициализация компонента
const todoList = new TodoList('#todo-app');
```

## Асинхронное управление состоянием

```javascript
class AsyncStateManager extends StateManager {
  constructor(initialState) {
    super(initialState);
    this.loadingStates = new Set();
  }
  
  async dispatchAsync(action) {
    const { type, payload, meta } = action;
    
    // Установка состояния загрузки
    if (meta && meta.loadingKey) {
      this.setLoading(meta.loadingKey, true);
    }
    
    try {
      const result = await payload();
      this.setState({ [type]: result });
      return result;
    } catch (error) {
      this.setState({ error: error.message });
      throw error;
    } finally {
      // Сброс состояния загрузки
      if (meta && meta.loadingKey) {
        this.setLoading(meta.loadingKey, false);
      }
    }
  }
  
  setLoading(key, value) {
    this.loadingStates.add(key);
    this.set(`${key}Loading`, value);
  }
  
  isLoading(key) {
    return this.get(`${key}Loading`) || false;
  }
}

// Пример использования
const asyncState = new AsyncStateManager({
  users: [],
  posts: []
});

// Асинхронное действие
asyncState.dispatchAsync({
  type: 'FETCH_USERS',
  payload: async () => {
    const response = await fetch('/api/users');
    return await response.json();
  },
  meta: { loadingKey: 'users' }
}).then(users => {
  console.log('Пользователи загружены:', users);
});
```

## Лучшие практики

### Иммутабельность

```javascript
// Плохо - изменение состояния напрямую
state.todos.push(newTodo);

// Хорошо - создание нового состояния
const newState = {
  ...state,
  todos: [...state.todos, newTodo]
};

// Использование Object.freeze для предотвращения изменений
function freezeState(obj) {
  Object.getOwnPropertyNames(obj).forEach(prop => {
    if (typeof obj[prop] === 'object' && obj[prop] !== null) {
      freezeState(obj[prop]);
    }
  });
  return Object.freeze(obj);
}
```

### Организация состояния

```javascript
// Организация состояния по доменам
const appState = {
  auth: {
    user: null,
    isAuthenticated: false,
    token: null
  },
  ui: {
    theme: 'light',
    language: 'ru',
    notifications: true
  },
  data: {
    users: [],
    posts: [],
    comments: []
  }
};
```

## Ссылки по теме

- [[js/state/redux]] - Redux
- [[js/patterns/flux]] - Flux архитектура
- [[js/storage/localstorage]] - LocalStorage
- [[js/patterns/observer]] - Паттерн Наблюдатель