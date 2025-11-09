---
aliases: ["Состояние JavaScript", "Управление состоянием", "State Management"]
tags: ["#javascript", "#state-management", "#react", "#vue", "#angular", "#redux", "#mobx", "#zustand", "#frontend", "#programming"]
---

# Управление состоянием в JavaScript: Комплексное руководство

## Что такое состояние в JavaScript

Состояние (state) в JavaScript - это коллекция данных, которая определяет поведение приложения в любой момент времени. Это могут быть данные пользователя, настройки приложения, результаты API-запросов и т.д.

Состояние определяет, как приложение будет отображаться и вести себя. При изменении состояния приложение должно реагировать соответствующим образом, обновляя пользовательский интерфейс.

## Клиентское состояние против серверного состояния

### Клиентское состояние (Client-side State)

Клиентское состояние хранится в браузере пользователя и включает:

- Состояние пользовательского интерфейса (активные вкладки, открытые меню)
- Временные данные (формы, фильтры)
- Локальные настройки (темы, языки)

```javascript
// Пример клиентского состояния
const [userPreferences, setUserPreferences] = useState({
  theme: 'dark',
  language: 'ru',
  notifications: true
});
```

### Серверное состояние (Server-side State)

Серверное состояние хранится на сервере и включает:

- Пользовательские данные (профили, контент)
- Бизнес-логику
- Данные аутентификации

```javascript
// Пример серверного состояния
const serverState = {
  users: [{ id: 1, name: 'Иван' }],
  permissions: { userId: 1, roles: ['admin'] }
};
```

См. также: [[js/api/data-fetching]] для работы с серверным состоянием.

## Паттерны управления состоянием

### 1. Паттерн "Контейнер-компонент"

Разделяет логику управления состоянием от презентационных компонентов.

### 2. Паттерн "Flux"

Однонаправленный поток данных: действие → диспетчер → хранилище → представление.

### 3. Паттерн "Observer"

Компоненты подписываются на изменения состояния и автоматически обновляются.

### 4. Паттерн "Singleton"

Один глобальный объект состояния, доступный всем компонентам.

## Локальное состояние против глобального состояния

### Локальное состояние

Локальное состояние принадлежит конкретному компоненту и не влияет на другие части приложения.

```javascript
// Локальное состояние в React
function Counter() {
  const [count, setCount] = useState(0);
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### Глобальное состояние

Глобальное состояние доступно из любой части приложения и используется для данных, которые нужны нескольким компонентам.

```javascript
// Глобальное состояние с Redux
const store = createStore(reducer);
```

См. также: [[js/react/react-state-management]] для подробностей о состоянии в React.

## Неизменяемое состояние против изменяемого состояния

### Неизменяемое состояние (Immutable)

При изменении создается новый объект, а не изменяется существующий.

```javascript
// Неизменяемое обновление
const newState = {
  ...currentState,
  items: [...currentState.items, newItem]
};
```

### Изменяемое состояние (Mutable)

Состояние изменяется напрямую.

```javascript
// Изменяемое обновление
currentState.items.push(newItem);
```

Неизменяемость предотвращает неожиданные побочные эффекты и упрощает отладку.

## Библиотеки управления состоянием

### Redux

Redux - это предсказуемый контейнер состояния для JavaScript-приложений.

```javascript
// Определение редьюсера
const counterReducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    default:
      return state;
  }
};

// Создание хранилища
const store = createStore(counterReducer);
```

См. также: [[js/redux/basics]]

### Zustand

Zustand - это легковесная библиотека управления состоянием.

```javascript
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}));
```

### MobX

MobX - это библиотека реактивного программирования.

```javascript
import { makeObservable, observable, action } from 'mobx';

class Store {
  count = 0;
  
  constructor() {
    makeObservable(this, {
      count: observable,
      increment: action
    });
  }
  
  increment = () => {
    this.count++;
  };
}
```

См. также: [[js/mobx/reactive-programming]]

## Состояние компонентов во фреймворках

### React

React предоставляет встроенные хуки для управления состоянием:

- `useState` - для простого состояния
- `useReducer` - для сложной логики
- `useContext` - для глобального состояния

### Vue

Vue использует реактивную систему:

```javascript
// Vue 3 Composition API
import { ref, reactive } from 'vue';

const count = ref(0);
const state = reactive({ name: 'Vue' });
```

### Angular

Angular использует сервисы и RxJS для управления состоянием:

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';

@Injectable()
export class StateService {
  private stateSubject = new BehaviorSubject<any>({});
  public state$ = this.stateSubject.asObservable();
}
```

## Сохранение состояния

Состояние может быть сохранено в:

- `localStorage` / `sessionStorage`
- Cookies
- IndexedDB
- Внешние сервисы

```javascript
// Сохранение в localStorage
const saveState = (state) => {
  localStorage.setItem('appState', JSON.stringify(state));
};

const loadState = () => {
  const state = localStorage.getItem('appState');
  return state ? JSON.parse(state) : {};
};
```

См. также: [[js/storage/local-storage-patterns]]

## Синхронизация состояния

Синхронизация необходима при работе с несколькими вкладками или пользователями:

```javascript
// Синхронизация через StorageEvent
window.addEventListener('storage', (e) => {
  if (e.key === 'sharedState') {
    updateLocalState(JSON.parse(e.newValue));
  }
});
```

## Реактивное управление состоянием

Реактивное программирование позволяет автоматически обновлять зависимости при изменении состояния:

```javascript
// Пример с RxJS
import { BehaviorSubject } from 'rxjs';

const state$ = new BehaviorSubject({ count: 0 });

state$.subscribe((state) => {
  console.log('State changed:', state);
});

// Изменение состояния
state$.next({ count: state$.value.count + 1 });
```

## Рассмотрение производительности

При управлении состоянием важно учитывать:

- Частоту обновлений
- Размер состояния
- Оптимизацию рендеринга
- Избегать ненужных перерисовок

```javascript
// Оптимизация с React.memo
const OptimizedComponent = React.memo(({ data }) => {
  return <div>{data.value}</div>;
});
```

## Отладка состояния

Для отладки состояния используются:

- Инструменты разработчика (React DevTools, Vue DevTools)
- Логирование изменений
- История изменений (Redux DevTools)

```javascript
// Простая отладка состояния
const debugState = (prevState, newState, action) => {
  console.group('State Update');
  console.log('Action:', action);
  console.log('Previous:', prevState);
  console.log('Next:', newState);
  console.groupEnd();
};
```

## Заключение

Управление состоянием - ключевой аспект разработки современных JavaScript-приложений. Правильный выбор паттернов и инструментов управления состоянием влияет на производительность, масштабируемость и поддерживаемость приложения.

---

**См. также:**
- [[js/react/react-state-management]]
- [[js/vue/vuex-patterns]]
- [[js/angular/ngrx-store]]
- [[js/redux/basics]]
- [[js/mobx/reactive-programming]]
- [[js/storage/local-storage-patterns]]
- [[js/api/data-fetching]]

**Теги:** #javascript #state-management #react #vue #angular #redux #mobx #zustand #frontend #programming