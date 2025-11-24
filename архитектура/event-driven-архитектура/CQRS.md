---
aliases: [Command Query Responsibility Segregation, Разделение ответственности команд и запросов]
tags: [архитектура, frontend, cqrs, programming, events]
---

# CQRS (Command Query Responsibility Segregation)

## Обзор

CQRS (Command Query Responsibility Segregation) - это архитектурный паттерн, при котором операции чтения (queries) и операции записи (commands) разделяются на разные модели. Вместо одной модели для чтения и записи данных, CQRS использует отдельные модели для этих операций, что позволяет оптимизировать каждую из них под свои задачи.

## Основные понятия

### Команда (Command)
- Операция, изменяющая состояние системы
- Не возвращает данные
- Может быть асинхронной
- Примеры: создание пользователя, обновление профиля, удаление заказа

### Запрос (Query)
- Операция, возвращающая данные
- Не изменяет состояние системы
- Обычно синхронная
- Примеры: получение списка пользователей, получение профиля, статистика

### Разделение моделей
- **Write Model (Модель записи)** - оптимизирована для операций записи
- **Read Model (Модель чтения)** - оптимизирована для операций чтения

## Архитектура CQRS

```
[UI Layer]
    |
[Command Handler]  [Query Handler]
    |                   |
[Write Model]  <--->  [Read Model]
    |                   |
[Event Store]  <--->  [Read DB]
```

## Применение в фронтенд-разработке

Хотя CQRS традиционно применяется на бэкенде, его принципы могут быть адаптированы для фронтенд-приложений:

### 1. Разделение состояния приложения

```javascript
// Модель записи (commands)
class WriteModel {
  constructor(eventBus) {
    this.eventBus = eventBus;
    this.state = {};
  }

  executeCommand(command) {
    switch (command.type) {
      case 'CREATE_USER':
        this.createUser(command.payload);
        break;
      case 'UPDATE_USER':
        this.updateUser(command.payload);
        break;
      case 'DELETE_USER':
        this.deleteUser(command.payload);
        break;
    }
  }

  createUser(userData) {
    // Логика создания пользователя
    const event = {
      type: 'USER_CREATED',
      payload: { ...userData, id: generateId() }
    };
    
    this.eventBus.publish(event.type, event.payload);
  }

  updateUser(userData) {
    // Логика обновления пользователя
    const event = {
      type: 'USER_UPDATED',
      payload: userData
    };
    
    this.eventBus.publish(event.type, event.payload);
  }

  deleteUser(userId) {
    // Логика удаления пользователя
    const event = {
      type: 'USER_DELETED',
      payload: { userId }
    };
    
    this.eventBus.publish(event.type, event.payload);
  }
}

// Модель чтения (queries)
class ReadModel {
  constructor(eventBus) {
    this.eventBus = eventBus;
    this.users = [];
    this.setupEventHandlers();
  }

  setupEventHandlers() {
    this.eventBus.subscribe('USER_CREATED', (payload) => {
      this.users.push(payload);
    });

    this.eventBus.subscribe('USER_UPDATED', (payload) => {
      const index = this.users.findIndex(u => u.id === payload.id);
      if (index !== -1) {
        this.users[index] = { ...this.users[index], ...payload };
      }
    });

    this.eventBus.subscribe('USER_DELETED', (payload) => {
      this.users = this.users.filter(u => u.id !== payload.userId);
    });
  }

  // Запросы
  getAllUsers() {
    return this.users;
  }

  getUserById(userId) {
    return this.users.find(u => u.id === userId);
  }

  getUsersByRole(role) {
    return this.users.filter(u => u.role === role);
  }
}
```

### 2. Интеграция с Redux

```javascript
// Redux с CQRS подходом
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

// Command actions (изменяют состояние)
const COMMANDS = {
  CREATE_TODO: 'COMMAND_CREATE_TODO',
  UPDATE_TODO: 'COMMAND_UPDATE_TODO',
  DELETE_TODO: 'COMMAND_DELETE_TODO'
};

// Query actions (только для чтения)
const QUERIES = {
  GET_ALL_TODOS: 'QUERY_GET_ALL_TODOS',
  GET_COMPLETED_TODOS: 'QUERY_GET_COMPLETED_TODOS',
  GET_ACTIVE_TODOS: 'QUERY_GET_ACTIVE_TODOS'
};

// Command handlers
function createTodoCommand(payload) {
  return async (dispatch, getState) => {
    try {
      // Выполняем команду
      const response = await api.createTodo(payload);
      
      // Обновляем состояние
      dispatch({
        type: COMMANDS.CREATE_TODO,
        payload: response.data
      });
    } catch (error) {
      console.error('Failed to create todo:', error);
    }
  };
}

// Query handlers
function getAllTodosQuery() {
  return (dispatch, getState) => {
    const state = getState();
    return state.todos.items; // Возвращаем данные из состояния
  };
}

// Редьюсер для модели записи
function todosWriteReducer(state = { items: [] }, action) {
  switch (action.type) {
    case COMMANDS.CREATE_TODO:
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    case COMMANDS.UPDATE_TODO:
      return {
        ...state,
        items: state.items.map(todo => 
          todo.id === action.payload.id ? { ...todo, ...action.payload } : todo
        )
      };
    case COMMANDS.DELETE_TODO:
      return {
        ...state,
        items: state.items.filter(todo => todo.id !== action.payload.id)
      };
    default:
      return state;
  }
}
```

## Пример реализации в React

```jsx
import React, { createContext, useContext, useReducer, useCallback } from 'react';

// Контекст для CQRS
const CQRSContext = createContext();

// Редьюсер для модели записи
function writeReducer(state, action) {
  switch (action.type) {
    case 'USER_CREATED':
      return {
        ...state,
        users: [...state.users, action.payload]
      };
    case 'USER_UPDATED':
      return {
        ...state,
        users: state.users.map(user => 
          user.id === action.payload.id ? { ...user, ...action.payload } : user
        )
      };
    case 'USER_DELETED':
      return {
        ...state,
        users: state.users.filter(user => user.id !== action.payload.userId)
      };
    default:
      return state;
  }
}

// Хук для команд (запись)
function useCommand() {
  const dispatch = useDispatch();

  const createUser = useCallback((userData) => {
    // Выполняем команду
    dispatch({ type: 'CREATE_USER_COMMAND', payload: userData });
  }, [dispatch]);

  const updateUser = useCallback((userData) => {
    dispatch({ type: 'UPDATE_USER_COMMAND', payload: userData });
  }, [dispatch]);

  const deleteUser = useCallback((userId) => {
    dispatch({ type: 'DELETE_USER_COMMAND', payload: { userId } });
  }, [dispatch]);

  return { createUser, updateUser, deleteUser };
}

// Хук для запросов (чтение)
function useQuery() {
  const state = useSelector(state => state);

  const getAllUsers = useCallback(() => {
    return state.users;
  }, [state.users]);

  const getUserById = useCallback((userId) => {
    return state.users.find(user => user.id === userId);
  }, [state.users]);

  const getUsersByRole = useCallback((role) => {
    return state.users.filter(user => user.role === role);
  }, [state.users]);

  return { getAllUsers, getUserById, getUsersByRole };
}

// Компонент с CQRS
function UserManagementComponent() {
  const command = useCommand();
  const query = useQuery();

  const handleCreateUser = (userData) => {
    command.createUser(userData);
  };

  const users = query.getAllUsers();

  return (
    <div>
      <h2>Управление пользователями</h2>
      <UserList users={users} />
      <UserForm onCreate={handleCreateUser} />
    </div>
  );
}
```

## Продвинутая реализация с Event Sourcing

```javascript
class CQRSSystem {
  constructor() {
    this.writeModel = new WriteModel();
    this.readModels = new Map();
    this.eventBus = new EventBus();
    
    // Подписываем read модели на события
    this.setupEventHandlers();
  }

  setupEventHandlers() {
    // Подписываем все read модели на события
    this.eventBus.subscribe('USER_CREATED', (payload) => {
      this.readModels.forEach(model => {
        if (typeof model.onUserCreated === 'function') {
          model.onUserCreated(payload);
        }
      });
    });

    this.eventBus.subscribe('USER_UPDATED', (payload) => {
      this.readModels.forEach(model => {
        if (typeof model.onUserUpdated === 'function') {
          model.onUserUpdated(payload);
        }
      });
    });

    this.eventBus.subscribe('USER_DELETED', (payload) => {
      this.readModels.forEach(model => {
        if (typeof model.onUserDeleted === 'function') {
          model.onUserDeleted(payload);
        }
      });
    });
  }

  registerReadModel(name, model) {
    this.readModels.set(name, model);
  }

  executeCommand(command) {
    // Выполняем команду в модели записи
    this.writeModel.executeCommand(command);
  }

  query(modelName, queryName, params) {
    const model = this.readModels.get(modelName);
    if (model && typeof model[queryName] === 'function') {
      return model[queryName](params);
    }
    throw new Error(`Query ${queryName} not found in model ${modelName}`);
  }
}

// Пример конкретной read модели
class UserReadModel {
  constructor() {
    this.users = [];
  }

  onUserCreated(payload) {
    this.users.push(payload);
  }

  onUserUpdated(payload) {
    const index = this.users.findIndex(u => u.id === payload.id);
    if (index !== -1) {
      this.users[index] = { ...this.users[index], ...payload };
    }
  }

  onUserDeleted(payload) {
    this.users = this.users.filter(u => u.id !== payload.userId);
  }

  getAllUsers() {
    return this.users;
  }

  getUserById(userId) {
    return this.users.find(u => u.id === userId);
  }

  getUsersByRole(role) {
    return this.users.filter(u => u.role === role);
  }
}
```

## Преимущества CQRS

### 1. Оптимизация производительности
- Read модели могут быть оптимизированы для конкретных запросов
- Write модели могут быть оптимизированы для конкретных команд

### 2. Упрощение сложных операций
- Разделение логики чтения и записи упрощает каждую из них
- Легче тестировать отдельные части системы

### 3. Масштабируемость
- Read и write модели могут масштабироваться независимо
- Разные базы данных для чтения и записи при необходимости

### 4. Гибкость
- Возможность иметь разные представления данных для разных потребителей
- Легче вносить изменения в отдельные части системы

## Недостатки и ограничения

### 1. Сложность
- Увеличивает общую сложность архитектуры
- Требует больше времени на реализацию

### 2. Согласованность данных
- Между read и write моделями может быть задержка
- Требуется обработка событий для синхронизации

### 3. Увеличение количества кода
- Нужно поддерживать две модели вместо одной
- Больше кода для синхронизации

## Практические рекомендации

### 1. Когда использовать CQRS

CQRS особенно полезен в следующих случаях:
- Сложные доменные модели с разными требованиями к чтению и записи
- Высоконагруженные системы с разными паттернами доступа к данным
- Системы с требованиями к аудиту и отслеживанию изменений
- Приложения с разными SLA для операций чтения и записи

### 2. Реализация с TypeScript

```typescript
interface Command {
  type: string;
  payload: any;
}

interface Query<T> {
  type: string;
  params: any;
  result: T;
}

interface ICommandHandler<C extends Command> {
  handle(command: C): void | Promise<void>;
}

interface IQueryHandler<Q extends Query<any>> {
  handle(query: Q): Q['result'] | Promise<Q['result']>;
}

class TypedCQRSSystem {
  private commandHandlers: Map<string, ICommandHandler<any>> = new Map();
  private queryHandlers: Map<string, IQueryHandler<any>> = new Map();

  registerCommandHandler<T extends Command>(type: T['type'], handler: ICommandHandler<T>) {
    this.commandHandlers.set(type, handler);
  }

  registerQueryHandler<T extends Query<any>>(type: T['type'], handler: IQueryHandler<T>) {
    this.queryHandlers.set(type, handler);
  }

  async executeCommand<T extends Command>(command: T): Promise<void> {
    const handler = this.commandHandlers.get(command.type);
    if (handler) {
      return await handler.handle(command);
    }
    throw new Error(`No handler found for command: ${command.type}`);
  }

  async executeQuery<T extends Query<any>>(query: T): Promise<T['result']> {
    const handler = this.queryHandlers.get(query.type);
    if (handler) {
      return await handler.handle(query);
    }
    throw new Error(`No handler found for query: ${query.type}`);
  }
}
```

### 3. Интеграция с кешированием

```javascript
class CachedReadModel {
  constructor() {
    this.cache = new Map();
    this.cacheTimeout = 5 * 60 * 1000; // 5 минут
  }

  async getCachedOrFetch(key, fetchFn) {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      return cached.data;
    }

    const data = await fetchFn();
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });

    return data;
  }

  async getUsersWithCache() {
    return await this.getCachedOrFetch('all-users', () => {
      // Логика получения пользователей
      return this.getAllUsers();
    });
  }
}
```

## Российские реалии и особенности 2025

В 2025 году CQRS в российской фронтенд-разработке становится особенно актуальным в следующих сценариях:

- Корпоративные системы с высокими требованиями к производительности
- Финансовые приложения с разными уровнями доступа к данным
- Системы с географически распределенными пользователями
- Приложения с оффлайн-режимом и синхронизацией

Российские компании также сталкиваются с необходимостью интеграции с отечественными системами, где CQRS помогает обеспечить гибкость архитектуры.

## Связанные концепции

- [[Событийная-архитектура]]
- [[Pub-Sub]]
- [[Event-sourcing]]
- [[Тестирование]]

## Заключение

CQRS - мощный паттерн, который помогает разделить ответственность между операциями чтения и записи. Хотя он добавляет сложности, он предоставляет значительные преимущества в производительности, масштабируемости и гибкости для сложных приложений. В контексте фронтенд-разработки он особенно полезен для приложений с высокими требованиями к производительности и сложной бизнес-логикой.