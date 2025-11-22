---
aliases: ["Реакт паттерны состояния", "Управление состоянием React"]
tags: 
  - #react
  - #state-management
  - #patterns
  - #frontend
---

# Паттерны управления состоянием в React

## Введение

Управление состоянием — одна из ключевых задач при разработке приложений на React. Правильный выбор паттерна управления состоянием влияет на производительность, масштабируемость и поддерживаемость кода.

## Локальное состояние против глобального состояния

### Локальное состояние (Local State)

Локальное состояние используется внутри компонента и не доступно другим компонентам. Оно подходит для управления состоянием UI, например, состояние формы или открытие/закрытие модального окна.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

[[react/hooks/usestate]] [[react/hooks/usestate]]

### Глобальное состояние (Global State)

Глобальное состояние доступно для всего приложения и используется для данных, которые должны быть доступны в разных частях приложения, например, пользовательские данные, настройки приложения и т.д.

```jsx
// Использование Context API для глобального состояния
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <AppContext.Provider value={{ user, setUser }}>
      {children}
    </AppContext.Provider>
  );
}
```

[[Типизация контекста в React с TypeScript]] [[react/hooks/usecontext]]

## Поднятие состояния (Lifting State Up)

Иногда несколько компонентов нуждаются в одном и том же изменяющемся состоянии. В таких случаях состояние "поднимается" до ближайшего общего предка.

```jsx
function Parent() {
  const [sharedValue, setSharedValue] = useState('');
  
  return (
    <div>
      <ChildA value={sharedValue} onChange={setSharedValue} />
      <ChildB value={sharedValue} />
    </div>
  );
}
```

## Локализация состояния (State Colocation)

Локализация состояния означает размещение состояния как можно ближе к компонентам, которые его используют. Это упрощает понимание потока данных и уменьшает количество пропсов.

[[react/props]]

## useState против useReducer

### useState

Подходит для простых состояний с примитивными значениями или небольшими объектами:

```jsx
const [count, setCount] = useState(0);
const [name, setName] = useState('');
```

### useReducer

Предпочтительнее для сложных состояний с логикой обновления:

```jsx
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Счетчик: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <input 
        type="number" 
        value={state.step}
        onChange={(e) => dispatch({ type: 'setStep', payload: Number(e.target.value) })}
      />
    </div>
  );
}
```

[[react/hooks/usestate]] [[react/hooks/usereducer]]

## Когда использовать Context против библиотек управления состоянием

### Context API

Используйте Context API для:
- Передачи данных через несколько уровней компонентов
- Данных, которые редко изменяются
- Аутентификации, тем, языка интерфейса

### Библиотеки управления состоянием

Используйте Redux, Zustand, Jotai и др. для:
- Сложных состояний приложения
- Необходимости отладки и мониторинга
- Состояния, которое часто изменяется
- Необходимости персистентности состояния

[[Типизация контекста в React с TypeScript]] [[redux]] [[zustand]]

## Нормализация состояния

Нормализация помогает избежать дублирования данных и упрощает обновления. Состояние структурируется так, что каждый фрагмент информации хранится только один раз.

```jsx
// Плохо - дублирование данных
const badState = {
  posts: [
    { id: 1, title: 'Post 1', author: { id: 1, name: 'John' } },
    { id: 2, title: 'Post 2', author: { id: 1, name: 'John' } }
  ]
};

// Хорошо - нормализованное состояние
const goodState = {
  posts: {
    1: { id: 1, title: 'Post 1', authorId: 1 },
    2: { id: 2, title: 'Post 2', authorId: 1 }
  },
  authors: {
    1: { id: 1, name: 'John' }
  }
};
```

## Неизменяемые обновления состояния

React полагается на неизменяемость для определения изменений. Всегда создавайте новые объекты при обновлении состояния:

```jsx
// Плохо
setState(prev => {
  prev.items.push(newItem);
  return prev;
});

// Хорошо
setState(prev => ({
  ...prev,
  items: [...prev.items, newItem]
}));
```

[[react/immutability]]

## Антипаттерны управления состоянием

- Мутация состояния напрямую
- Хранение не-данных в состоянии (DOM-элементы, функции)
- Избыточное состояние (состояние, которое можно вычислить из другого состояния)
- Слишком глубокая вложенность состояния

## Рассмотрение производительности

- Используйте `React.memo` для предотвращения ненужных перерисовок
- Оптимизируйте обновления состояния с помощью `useCallback` и `useMemo`
- Избегайте создания новых объектов/массивов при каждом рендере

[[react/performance]]

## Компонентное состояние против состояния приложения

Компонентное состояние используется для локального UI-состояния, тогда как состояние приложения содержит данные, которые нужны нескольким компонентам.

## Персистентность состояния

Состояние может сохраняться в localStorage, sessionStorage или на сервере для восстановления после перезагрузки:

```jsx
function useLocalStorageState(key, defaultValue) {
  const [state, setState] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      return defaultValue;
    }
  });
  
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(state));
    } catch (error) {
      // Обработка ошибки
    }
  }, [key, state]);
  
  return [state, setState];
}
```

## Серверное состояние против клиентского состояния

Серверное состояние — это данные, которые приходят с сервера (например, список пользователей), а клиентское — это данные, которые существуют только в браузере (например, состояние UI).

[[react/server-state-vs-client-state]]

## Синхронизация состояния

Для синхронизации состояния между компонентами и сервером можно использовать библиотеки вроде SWR или React Query.

[[swr]] [[react-query]]

## Шаблоны общего состояния

- Singleton-паттерн для глобального состояния
- Паттерн "контейнер-презентер" для разделения логики и представления
- Паттерн "провайдер" для предоставления состояния дочерним компонентам

## Производное состояние

Производное состояние вычисляется на основе других значений состояния:

```jsx
const [items, setItems] = useState([]);
const [filter, setFilter] = useState('');

// Производное состояние
const filteredItems = useMemo(() => 
  items.filter(item => item.name.includes(filter)), 
  [items, filter]
);
```

[[react/hooks/usememo]]

## Вычисляемое состояние

Аналогично производному состоянию, вычисляемое состояние обновляется только при изменении зависимостей:

```jsx
const total = useMemo(() => items.reduce((sum, item) => sum + item.price, 0), [items]);
```

## Машины состояний в React

Для сложной логики состояния можно использовать библиотеки машин состояний, такие как XState:

```jsx
import { useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: { on: { TOGGLE: 'active' } },
    active: { on: { TOGGLE: 'inactive' } }
  }
});

function Toggle() {
  const [current, send] = useMachine(toggleMachine);
  
  return (
    <button onClick={() => send('TOGGLE')}>
      {current.value === 'inactive' ? 'Off' : 'On'}
    </button>
  );
}
```

[[xstate]]

## Паттерны Redux

Redux предоставляет предсказуемое состояние для JavaScript-приложений:

- Actions: определяют, что произошло
- Reducers: определяют, как состояние обновляется
- Store: содержит состояние приложения

[[redux]]

## Zustand против других решений

Zustand — это легковесная альтернатива Redux с более простым API:

```jsx
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));
```

[[zustand]]

## Атомарное управление состоянием

Атомарное управление позволяет управлять отдельными фрагментами состояния независимо:

```jsx
// В Jotai
const countAtom = atom(0);
const itemsAtom = atom([]);

// В Recoil
const countState = atom({
  key: 'countState',
  default: 0,
});
```

[[jotai]] [[recoil]]

## SWR и React Query для серверного состояния

Эти библиотеки упрощают работу с серверными данными:

```jsx
// SWR
const { data, error } = useSWR('/api/user', fetcher);

// React Query
const { data, isLoading, error } = useQuery(['user', userId], fetchUser);
```

## Заключение

Выбор правильного паттерна управления состоянием зависит от сложности приложения, частоты изменений и архитектурных требований. Важно понимать различия между локальным и глобальным состоянием, а также знать, когда использовать тот или иной инструмент.

## Связанные темы

- [[Типизация хуков React в TypeScript]]
- [[Типизация контекста в React с TypeScript]]
- [[react/performance]]
- [[redux]]
- [[zustand]]
- [[recoil]]
- [[jotai]]
- [[swr]]
- [[react-query]]
- [[xstate]]