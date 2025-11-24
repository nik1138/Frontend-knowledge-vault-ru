---
aliases: [Publish-Subscribe Pattern, Паттерн издатель-подписчик]
tags: [архитектура, frontend, events, pubsub, programming]
---

# Pub-Sub (Publish-Subscribe)

## Обзор

Паттерн Publish-Subscribe (Pub-Sub) - это шаблон проектирования, при котором отправители сообщений (издатели) не отправляют сообщения напрямую конкретным получателям (подписчикам). Вместо этого они публикуют сообщения в систему, не зная, кто будет их получать. Подписчики интересуются определенными типами сообщений и получают только те, на которые они подписались.

## Основные компоненты

### Издатель (Publisher)
Компонент, который создает и отправляет события в систему. Издатель не знает, кто будет получать события, и не управляет подписками.

### Подписчик (Subscriber)
Компонент, который выражает интерес к определенным типам событий и обрабатывает их. Подписчик не знает, кто является издателем события.

### Шина событий (Event Bus)
Центральный компонент, который принимает события от издателей и доставляет их подписчикам. Может быть реализован как:
- Централизованная шина событий
- Распределенная система сообщений
- Встроенный механизм в фреймворке

## Реализация в JavaScript

### Простая реализация Event Bus

```javascript
class EventBus {
  constructor() {
    this.events = {};
  }

  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
    // Возвращаем функцию для отписки
    return () => {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    };
  }

  publish(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }

  unsubscribe(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
}

// Использование
const eventBus = new EventBus();

const unsubscribe = eventBus.subscribe('USER_LOGIN', (userData) => {
  console.log('Пользователь вошел:', userData);
});

eventBus.publish('USER_LOGIN', { id: 123, name: 'Иван' });

unsubscribe(); // Отписка
```

### Продвинутая реализация с поддержкой асинхронности

```javascript
class AdvancedEventBus {
  constructor() {
    this.events = new Map();
    this.middlewares = [];
  }

  use(middleware) {
    this.middlewares.push(middleware);
  }

  subscribe(event, callback, priority = 0) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    
    const subscribers = this.events.get(event);
    subscribers.push({ callback, priority });
    subscribers.sort((a, b) => b.priority - a.priority); // Сортировка по приоритету
    
    return () => {
      const index = subscribers.findIndex(sub => sub.callback === callback);
      if (index > -1) {
        subscribers.splice(index, 1);
      }
    };
  }

  async publish(event, data) {
    const subscribers = this.events.get(event) || [];
    
    // Применение middleware
    let processedData = data;
    for (const middleware of this.middlewares) {
      processedData = await middleware(event, processedData);
    }

    // Асинхронная обработка событий
    const promises = subscribers.map(sub => 
      Promise.resolve().then(() => sub.callback(processedData))
    );
    
    return Promise.all(promises);
  }
}
```

## Преимущества Pub-Sub

- **Декларативность** - легче понимать, какие компоненты зависят от каких событий
- **Слабая связанность** - издатели и подписчики не зависят друг от друга
- **Гибкость** - легко добавлять новых подписчиков без изменения издателей
- **Масштабируемость** - можно легко масштабировать систему, добавляя новых участников
- **Повторное использование** - компоненты можно использовать в разных контекстах

## Недостатки и риски

- **Сложность отладки** - трудно отследить путь события
- **Утечки памяти** - если не управлять подписками должным образом
- **Переусложнение** - может быть избыточным для простых сценариев
- **Непредсказуемый порядок выполнения** - если несколько подписчиков на одно событие
- **Сложность тестирования** - необходимо учитывать все возможные подписчики

## Лучшие практики

### 1. Четкая номенклатура событий

```javascript
// Хорошо: понятные имена событий
const EVENTS = {
  USER_LOGIN: 'user.login',
  USER_LOGOUT: 'user.logout',
  PRODUCT_ADDED_TO_CART: 'cart.product.added',
  ORDER_CREATED: 'order.created'
};

// Плохо: неинформативные имена
const BAD_EVENTS = {
  EVT_001: 'event.001',
  LOGIN: 'login',
  ADD: 'add'
};
```

### 2. Управление жизненным циклом подписок

```javascript
class UserProfileComponent {
  constructor() {
    this.eventBus = EventBus.getInstance();
    this.subscriptions = [];
  }

  init() {
    // Сохраняем функции отписки
    this.subscriptions.push(
      this.eventBus.subscribe('USER_UPDATED', this.handleUserUpdate.bind(this)),
      this.eventBus.subscribe('USER_PROFILE_PICTURE_CHANGED', this.handleProfilePictureChange.bind(this))
    );
  }

  destroy() {
    // Отписываемся от всех событий
    this.subscriptions.forEach(unsubscribe => unsubscribe());
    this.subscriptions = [];
  }

  handleUserUpdate(userData) {
    this.updateView(userData);
  }

  handleProfilePictureChange(imageUrl) {
    this.updateProfilePicture(imageUrl);
  }
}
```

### 3. Использование TypeScript для типизации событий

```typescript
type EventTypes = {
  'user.login': { userId: string; timestamp: number };
  'user.logout': { userId: string; reason?: string };
  'cart.product.added': { productId: string; quantity: number; price: number };
};

class TypedEventBus {
  private events: { [K in keyof EventTypes]?: Array<(data: EventTypes[K]) => void> } = {};

  subscribe<K extends keyof EventTypes>(event: K, callback: (data: EventTypes[K]) => void) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    (this.events[event] as Array<(data: EventTypes[K]) => void>).push(callback);
  }

  publish<K extends keyof EventTypes>(event: K, data: EventTypes[K]) {
    const callbacks = this.events[event] as Array<(data: EventTypes[K]) => void> | undefined;
    if (callbacks) {
      callbacks.forEach(callback => callback(data));
    }
  }
}
```

## Примеры использования в фронтенд-фреймворках

### React с Context API

```jsx
import React, { createContext, useContext, useReducer } from 'react';

// Определяем события
const EVENTS = {
  ADD_TO_CART: 'ADD_TO_CART',
  REMOVE_FROM_CART: 'REMOVE_FROM_CART',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY'
};

// Создаем контекст событий
const EventContext = createContext();

// Редьюсер для обработки событий
function cartReducer(state, action) {
  switch (action.type) {
    case EVENTS.ADD_TO_CART:
      return {
        ...state,
        items: [...state.items, action.payload]
      };
    case EVENTS.REMOVE_FROM_CART:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload.id)
      };
    default:
      return state;
  }
}

// Провайдер событий
export function EventProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  const publish = (event, data) => {
    dispatch({ type: event, payload: data });
  };

  return (
    <EventContext.Provider value={{ state, publish }}>
      {children}
    </EventContext.Provider>
  );
}

// Хук для подписки на события
export const useEvent = () => {
  const context = useContext(EventContext);
  if (!context) {
    throw new Error('useEvent must be used within EventProvider');
  }
  return context;
};
```

### Vue.js с глобальной шиной событий

```javascript
// Создание глобальной шины событий
import { createApp } from 'vue';

const EventBus = {
  install(app) {
    app.config.globalProperties.$eventBus = {
      events: {},
      
      $on(event, callback) {
        if (!this.events[event]) {
          this.events[event] = [];
        }
        this.events[event].push(callback);
      },
      
      $off(event, callback) {
        if (this.events[event]) {
          if (callback) {
            this.events[event] = this.events[event].filter(cb => cb !== callback);
          } else {
            delete this.events[event];
          }
        }
      },
      
      $emit(event, data) {
        if (this.events[event]) {
          this.events[event].forEach(callback => callback(data));
        }
      }
    };
  }
};

// Использование в компоненте
export default {
  mounted() {
    this.$eventBus.$on('user-logged-in', this.handleLogin);
  },
  
  beforeUnmount() {
    this.$eventBus.$off('user-logged-in', this.handleLogin);
  },
  
  methods: {
    handleLogin(userData) {
      this.user = userData;
    }
  }
};
```

## Российские реалии и особенности 2025

В 2025 году в российской разработке паттерн Pub-Sub особенно актуален в следующих сценариях:

- Интеграция с отечественными API и сервисами
- Разработка приложений с высокой нагрузкой
- Микрофронтенд-архитектуры
- Системы с реальным временем (например, чаты, дашборды)

Компании в России активно используют Pub-Sub для интеграции различных внутренних систем и обеспечения гибкости архитектуры.

## Связанные концепции

- [[Событийная-архитектура]]
- [[Event-sourcing]]
- [[CQRS]]
- [[Тестирование]]

## Заключение

Паттерн Publish-Subscribe является важным инструментом в арсенале фронтенд-разработчика. Он позволяет создавать гибкие, слабосвязанные системы, которые легче масштабировать и поддерживать. При правильном применении этот паттерн значительно улучшает архитектуру приложения.