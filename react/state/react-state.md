---
aliases: ["Состояние в React", "Управление состоянием", "React State"]
tags: 
  - "#react"
  - "#state-management"
  - "#hooks"
  - "#frontend"
---

# Управление состоянием в React

## Что такое состояние в React

Состояние (state) в React — это объект, который содержит данные, которые могут изменяться в течение жизненного цикла компонента. Когда состояние изменяется, компонент перерисовывается, отражая новые данные.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>Увеличить</button>
    </div>
  );
}
```

Состояние отличается от пропсов тем, что оно локальное и изменяемое, тогда как пропсы передаются из родительского компонента и должны быть неизменяемыми.

## Локальное состояние vs Глобальное состояние

### Локальное состояние
Используется внутри одного компонента и не передается другим компонентам. Управляется с помощью хуков `useState`, `useReducer`, `useRef`.

### Глобальное состояние
Используется для хранения данных, которые нужны нескольким компонентам. Реализуется с помощью контекста [[react/context/context]] или библиотек управления состоянием.

## Основные хуки состояния

### useState
Самый простой способ управления состоянием в функциональных компонентах:

```jsx
const [state, setState] = useState(initialValue);
```

### useReducer
Более сложный, но предпочтительный способ управления сложным состоянием:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}
```

### useContext
Позволяет компонентам получать доступ к глобальному состоянию, созданному с помощью [[react/context/context]]:

```jsx
const CountContext = createContext();

function App() {
  const [count, setCount] = useState(0);
  
  return (
    <CountContext.Provider value={{ count, setCount }}>
      <ChildComponent />
    </CountContext.Provider>
  );
}
```

## Паттерны управления состоянием

### Поднятие состояния (Lifting State Up)
Когда несколько компонентов нуждаются в одних и тех же данных, состояние поднимается к общему родительскому компоненту.

### Локализация состояния (State Colocation)
Состояние должно находиться как можно ближе к компонентам, которые его используют, чтобы упростить понимание и поддержку кода.

## Библиотеки управления состоянием

### Redux
Классическая библиотека с предсказуемым состоянием, основанная на принципах Flux. Использует редьюсеры и действия (actions) для изменения состояния.

### Zustand
Легковесная альтернатива Redux с минимальной настройкой и встроенной поддержкой TypeScript.

### MobX
Реализует реактивное программирование, автоматически отслеживая зависимости состояния и обновляя компоненты при изменениях.

## Нормализация состояния

Для оптимизации производительности и упрощения обновлений, состояние часто нормализуется, особенно при работе с коллекциями данных:

```jsx
// Не нормализованное состояние
const posts = [
  { id: 1, title: 'Post 1', author: { id: 1, name: 'John' } },
  { id: 2, title: 'Post 2', author: { id: 2, name: 'Jane' } }
];

// Нормализованное состояние
const state = {
  posts: { 1: { id: 1, title: 'Post 1', authorId: 1 }, 2: { id: 2, title: 'Post 2', authorId: 2 } },
  authors: { 1: { id: 1, name: 'John' }, 2: { id: 2, name: 'Jane' } }
};
```

## Неизменяемые обновления состояния

В React состояние должно обновляться неизменяемо (immutably), чтобы избежать непредсказуемых результатов:

```jsx
// Правильно
setItems([...items, newItem]);
setItems(items.filter(item => item.id !== id));

// Неправильно
items.push(newItem);
items[0] = newValue;
```

## Производительность и оптимизация

- Используйте `useMemo` и `useCallback` для предотвращения ненужных перерасчетов
- Оптимизируйте рендеринг с помощью `React.memo`
- Разделяйте часто и редко изменяемые данные

## Паттерны управления состоянием

- **Local State**: Для состояния, используемого только в одном компоненте
- **Lifted State**: Для состояния, разделяемого между несколькими компонентами
- **Global State**: Для состояния, используемого во всем приложении
- **Server State**: Для данных, полученных с сервера

## Валидация состояния и обработка ошибок

Состояние может быть проверено на корректность перед обновлением:

```jsx
function validateEmail(email) {
  return email.includes('@');
}

function EmailForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState(null);
  
  const updateEmail = (newEmail) => {
    if (validateEmail(newEmail)) {
      setEmail(newEmail);
      setError(null);
    } else {
      setError('Неверный формат email');
    }
  };
  
  return (
    <div>
      <input onChange={e => updateEmail(e.target.value)} value={email} />
      {error && <div className="error">{error}</div>}
    </div>
  );
}
```

## Связь с другими файлами

- [[react/hooks/hooks]] - Основные концепции хуков
- [[react/context/context]] - Контекстное управление состоянием
- [[react/components/components]] - Компонентный подход
- [[react/performance/performance]] - Оптимизация производительности
- [[react/testing/testing]] - Тестирование компонентов со состоянием

## Заключение

Управление состоянием — ключевой аспект разработки на React. Выбор подходящего метода зависит от сложности приложения, объема данных и требований к производительности. Понимание основных концепций позволяет создавать эффективные и поддерживаемые приложения.