---
aliases: [Event Sourcing, Событийное sourcing]
tags: [архитектура, frontend, events, event-sourcing, programming]
---

# Event Sourcing (Событийное хранение)

## Обзор

Event Sourcing - это архитектурный паттерн, при котором состояние системы сохраняется не в виде конечного результата, а как последовательность событий, которые привели к этому состоянию. Вместо хранения текущего состояния объекта, система хранит все события, которые изменяли это состояние, и может воссоздать текущее состояние, "проиграв" все события в хронологическом порядке.

## Основные принципы

### Хранение событий как истины

В Event Sourcing вся история изменений системы сохраняется в виде неизменяемых событий. Каждое событие представляет собой факт, произошедший в системе в определенный момент времени. Это позволяет:

- Воссоздать состояние системы на любой момент времени
- Понять, как система пришла к текущему состоянию
- Откатить систему к предыдущему состоянию
- Анализировать историю изменений

### Восстановление состояния

Состояние системы воссоздается путем "проигрывания" всех событий, начиная с начального состояния. Этот процесс называется "rehydration" (восстановление).

```javascript
// Пример: банковский счет с использованием Event Sourcing
class BankAccount {
  constructor() {
    this.balance = 0;
    this.events = [];
  }

  // Восстановление состояния из событий
  loadFromHistory(events) {
    this.events = events;
    this.balance = 0;
    
    for (const event of events) {
      this.applyEvent(event);
    }
  }

  applyEvent(event) {
    switch (event.type) {
      case 'AccountCreated':
        this.balance = 0;
        break;
      case 'MoneyDeposited':
        this.balance += event.amount;
        break;
      case 'MoneyWithdrawn':
        this.balance -= event.amount;
        break;
    }
  }

  // Создание новых событий
  deposit(amount) {
    const event = {
      type: 'MoneyDeposited',
      amount: amount,
      timestamp: Date.now()
    };
    
    this.applyEvent(event);
    this.events.push(event);
    this.saveEvent(event);
  }

  withdraw(amount) {
    if (this.balance >= amount) {
      const event = {
        type: 'MoneyWithdrawn',
        amount: amount,
        timestamp: Date.now()
      };
      
      this.applyEvent(event);
      this.events.push(event);
      this.saveEvent(event);
    } else {
      throw new Error('Insufficient funds');
    }
  }

  saveEvent(event) {
    // Сохранение события в хранилище событий
    localStorage.setItem(`event_${Date.now()}`, JSON.stringify(event));
  }
}
```

## Преимущества Event Sourcing

### 1. Полная история изменений

Каждое изменение в системе фиксируется, что позволяет:
- Понимать, как система пришла к текущему состоянию
- Отслеживать аудит изменений
- Анализировать поведение пользователей

### 2. Возможность отката изменений

Поскольку все события сохраняются, можно легко восстановить систему к любому предыдущему состоянию.

### 3. Повторное проигрывание событий

События можно использовать для обновления различных представлений данных (read models) или для интеграции с другими системами.

### 4. Масштабируемость

Event Sourcing хорошо сочетается с архитектурой, ориентированной на события, что позволяет строить масштабируемые системы.

## Применение в фронтенд-разработке

Хотя Event Sourcing традиционно применяется на бэкенде, его принципы могут быть полезны и в фронтенд-разработке:

### 1. Управление состоянием приложения

```javascript
// Redux с Event Sourcing подходом
const eventStore = {
  state: {},
  events: [],
  
  dispatch(event) {
    this.events.push(event);
    this.state = this.rebuildState();
    this.notifySubscribers();
  },
  
  rebuildState() {
    let state = {};
    
    for (const event of this.events) {
      state = this.applyEvent(state, event);
    }
    
    return state;
  },
  
  applyEvent(state, event) {
    switch (event.type) {
      case 'USER_LOGIN':
        return { ...state, user: event.payload };
      case 'USER_LOGOUT':
        return { ...state, user: null };
      case 'TODO_ADDED':
        return {
          ...state,
          todos: [...(state.todos || []), event.payload]
        };
      default:
        return state;
    }
  }
};
```

### 2. История изменений для отладки

```javascript
// Отладка с помощью Event Sourcing
class DebuggableState {
  constructor() {
    this.currentState = {};
    this.eventHistory = [];
    this.eventCounter = 0;
  }

  setState(newState, eventDescription = '') {
    const event = {
      id: ++this.eventCounter,
      timestamp: Date.now(),
      description: eventDescription,
      previousState: { ...this.currentState },
      newState: newState,
      action: 'setState'
    };
    
    this.eventHistory.push(event);
    this.currentState = { ...newState };
  }

  // Возврат к предыдущему состоянию
  undo(steps = 1) {
    if (this.eventHistory.length >= steps) {
      const targetEvent = this.eventHistory[this.eventHistory.length - steps - 1];
      this.currentState = targetEvent ? targetEvent.previousState : {};
      // Удаляем последние события
      this.eventHistory = this.eventHistory.slice(0, this.eventHistory.length - steps);
    }
  }

  // Получение истории событий
  getHistory() {
    return this.eventHistory;
  }
}
```

## Реализация Event Sourcing в фронтенде

### Хранилище событий

```javascript
class EventStore {
  constructor() {
    this.events = [];
    this.aggregates = new Map();
  }

  // Сохранение события
  storeEvent(aggregateId, event) {
    const eventWithMetadata = {
      ...event,
      aggregateId,
      timestamp: Date.now(),
      version: this.getNextVersion(aggregateId)
    };
    
    this.events.push(eventWithMetadata);
    this.updateAggregate(aggregateId, eventWithMetadata);
  }

  // Получение событий для агрегата
  getEventsForAggregate(aggregateId) {
    return this.events
      .filter(e => e.aggregateId === aggregateId)
      .sort((a, b) => a.version - b.version);
  }

  // Восстановление агрегата
  getAggregate(aggregateId) {
    if (!this.aggregates.has(aggregateId)) {
      const events = this.getEventsForAggregate(aggregateId);
      const aggregate = this.rebuildAggregate(aggregateId, events);
      this.aggregates.set(aggregateId, aggregate);
    }
    
    return this.aggregates.get(aggregateId);
  }

  getNextVersion(aggregateId) {
    const events = this.getEventsForAggregate(aggregateId);
    return events.length > 0 ? Math.max(...events.map(e => e.version)) + 1 : 1;
  }

  updateAggregate(aggregateId, event) {
    let aggregate = this.aggregates.get(aggregateId) || {};
    aggregate = this.applyEvent(aggregate, event);
    this.aggregates.set(aggregateId, aggregate);
  }

  rebuildAggregate(aggregateId, events) {
    let aggregate = {};
    
    for (const event of events) {
      aggregate = this.applyEvent(aggregate, event);
    }
    
    return aggregate;
  }

  applyEvent(aggregate, event) {
    // Здесь реализуется логика применения события к агрегату
    // В зависимости от типа события
    return {
      ...aggregate,
      [event.type]: event.payload
    };
  }
}
```

### Интеграция с React

```jsx
import React, { createContext, useContext, useReducer, useEffect } from 'react';

// Контекст для Event Sourcing
const EventSourcingContext = createContext();

// Редьюсер для обработки событий
function eventReducer(state, action) {
  switch (action.type) {
    case 'LOAD_FROM_HISTORY':
      return rebuildStateFromEvents(action.events);
    case 'APPLY_EVENT':
      return applyEvent(state, action.event);
    default:
      return state;
  }
}

function rebuildStateFromEvents(events) {
  return events.reduce((state, event) => applyEvent(state, event), {});
}

function applyEvent(state, event) {
  // Применение события к состоянию
  switch (event.type) {
    case 'USER_LOGIN':
      return { ...state, user: event.payload };
    case 'TASK_CREATED':
      return {
        ...state,
        tasks: [...(state.tasks || []), event.payload]
      };
    default:
      return state;
  }
}

// Провайдер Event Sourcing
export function EventSourcingProvider({ children, initialEvents = [] }) {
  const [state, dispatch] = useReducer(eventReducer, {});

  useEffect(() => {
    dispatch({ type: 'LOAD_FROM_HISTORY', events: initialEvents });
  }, [initialEvents]);

  const storeEvent = (event) => {
    dispatch({ type: 'APPLY_EVENT', event });
    // Здесь можно сохранить событие в хранилище
  };

  return (
    <EventSourcingContext.Provider value={{ state, storeEvent }}>
      {children}
    </EventSourcingContext.Provider>
  );
}

// Хук для использования Event Sourcing
export const useEventSourcing = () => {
  const context = useContext(EventSourcingContext);
  if (!context) {
    throw new Error('useEventSourcing must be used within EventSourcingProvider');
  }
  return context;
};
```

## Вызовы и ограничения

### 1. Производительность

При большом количестве событий восстановление состояния может занимать много времени. Решения:
- Использование снимков (snapshots)
- Индексация событий
- Асинхронная обработка

### 2. Сложность реализации

Event Sourcing требует более сложной архитектуры и понимания концепций:
- Агрегаты
- События
- Проекции
- Снимки

### 3. Хранение данных

Необходимость хранения всех событий может привести к увеличению объема данных.

## Практические рекомендации

### 1. Использование снимков

```javascript
class SnapshotEventStore {
  constructor() {
    this.events = [];
    this.snapshots = new Map();
    this.snapshotInterval = 100; // Создавать снимок каждые 100 событий
  }

  storeEvent(aggregateId, event) {
    this.events.push({
      ...event,
      aggregateId,
      timestamp: Date.now(),
      sequence: this.events.length
    });

    // Создание снимка при достижении интервала
    const eventCount = this.getEventCountForAggregate(aggregateId);
    if (eventCount % this.snapshotInterval === 0) {
      this.createSnapshot(aggregateId);
    }
  }

  createSnapshot(aggregateId) {
    const events = this.getEventsForAggregate(aggregateId);
    const state = this.rebuildStateFromEvents(events);
    
    this.snapshots.set(aggregateId, {
      state,
      sequence: events.length - 1,
      timestamp: Date.now()
    });
  }

  getAggregate(aggregateId) {
    const snapshot = this.snapshots.get(aggregateId);
    const events = this.getEventsForAggregate(aggregateId);
    
    if (snapshot) {
      // Восстановление из снимка + оставшихся событий
      const relevantEvents = events.filter(e => e.sequence > snapshot.sequence);
      return this.rebuildStateFromEvents(relevantEvents, snapshot.state);
    } else {
      // Восстановление из всех событий
      return this.rebuildStateFromEvents(events);
    }
  }
}
```

### 2. Типизация событий

```typescript
interface EventBase {
  type: string;
  timestamp: number;
  aggregateId: string;
}

interface UserCreatedEvent extends EventBase {
  type: 'UserCreated';
  payload: {
    userId: string;
    email: string;
    name: string;
  };
}

interface UserUpdatedEvent extends EventBase {
  type: 'UserUpdated';
  payload: {
    userId: string;
    updates: Partial<{ email: string; name: string }>;
  };
}

type UserEvent = UserCreatedEvent | UserUpdatedEvent;

class TypedEventStore {
  private events: UserEvent[] = [];

  storeEvent(event: UserEvent) {
    this.events.push(event);
  }

  getEventsForUser(userId: string): UserEvent[] {
    return this.events.filter(event => 
      'payload' in event && 
      'userId' in event.payload && 
      event.payload.userId === userId
    );
  }
}
```

## Российские реалии и особенности 2025

В 2025 году Event Sourcing в российской разработке становится особенно актуальным в следующих сценариях:

- Финансовые приложения, требующие полной аудиторной следы
- Системы электронного документооборота
- Игровые приложения с важной историей действий
- Приложения с требованиями к откату изменений

Российские компании также сталкиваются с необходимостью хранения данных в соответствии с требованиями законодательства, что делает Event Sourcing привлекательным решением.

## Связанные концепции

- [[Событийная-архитектура]]
- [[Pub-Sub]]
- [[CQRS]]
- [[Тестирование]]

## Заключение

Event Sourcing - мощный паттерн, который предоставляет полную историю изменений в системе. Хотя он требует больше усилий для реализации, он предоставляет уникальные возможности для анализа, отладки и восстановления состояния системы. В контексте фронтенд-разработки он особенно полезен для сложных приложений, где важна история изменений и возможность отката действий.