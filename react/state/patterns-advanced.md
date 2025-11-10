---
aliases: ["React продвинутые паттерны состояния", "Управление состоянием React"]
tags: 
  - #react
  - #state-management
  - #patterns
  - #frontend
---

# Продвинутые паттерны управления состоянием в React

## Глобальное vs локальное состояние

**Решение о типе состояния** основывается на его области видимости и частоте использования:

- **Локальное состояние**: используется только в одном компоненте
- **Глобальное состояние**: используется несколькими компонентами на разных уровнях

```jsx
// Локальное состояние
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}

// Глобальное состояние через Context
const UserContext = createContext();
```

> [!tip] Совет
> Используйте локальное состояние по умолчанию. Повышайте до глобального только при необходимости.

## Поднятие vs опускание состояния

### Поднятие состояния (Lifting State Up)

Когда несколько компонентов нуждаются в одном и том же состоянии, поднимите его до ближайшего общего предка:

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <Display count={count} />
      <Controls setCount={setCount} />
    </div>
  );
}
```

### Опускание состояния (Moving State Down)

Для лучшей инкапсуляции и производительности опускайте состояние к компонентам, которые его используют:

```jsx
function OptimizedList({ items }) {
  return items.map(item => (
    <MemoizedItem key={item.id} item={item} />
  ));
}
```

## Стратегии распределения состояния

1. **Централизованное**: все состояние в одном месте (Redux)
2. **Децентрализованное**: состояние ближе к компонентам
3. **Гибридное**: комбинация подходов в зависимости от потребностей

## Выбор между useState, useReducer, Context и внешними библиотеками

| Паттерн | Использование |
|--------|--------------|
| `useState` | Простое локальное состояние |
| `useReducer` | Сложная логика обновления состояния |
| `Context` | Глобальное состояние без частых обновлений |
| Внешние библиотеки | Сложные приложения с глубокой структурой |

```jsx
// useState для простого состояния
const [visible, setVisible] = useState(true);

// useReducer для сложной логики
const [state, dispatch] = useReducer(reducer, initialState);
```

## Нормализация состояния

Храните данные в нормализованной форме, как в базе данных:

```jsx
// Плохо: дублирование данных
const badState = [
  { id: 1, user: { id: 1, name: 'John' } },
  { id: 2, user: { id: 1, name: 'John' } }
];

// Хорошо: нормализованное состояние
const goodState = {
  users: { 1: { id: 1, name: 'John' } },
  posts: { 1: { id: 1, userId: 1, title: 'Post' } }
};
```

## Неизменяемое vs изменяемое состояние

React рекомендует неизменяемый подход:

```jsx
// Неизменяемый (рекомендуется)
const newState = [...prevState, newItem];
const updatedState = { ...prevState, field: newValue };

// Изменяемый (не рекомендуется)
prevState.push(newItem); // Плохо
prevState.field = newValue; // Плохо
```

## Стратегии сохранения состояния

### localStorage
```jsx
const [data, setData] = useState(() => {
  const saved = localStorage.getItem('data');
  return saved ? JSON.parse(saved) : initialState;
});

useEffect(() => {
  localStorage.setItem('data', JSON.stringify(data));
}, [data]);
```

### useReducer с localStorage
```jsx
const stateReducer = (state, action) => {
  const newState = reducer(state, action);
  localStorage.setItem('appState', JSON.stringify(newState));
  return newState;
};
```

## Управление асинхронным состоянием

```jsx
function useAsync(initialState) {
  const [state, setState] = useState(initialState);
  
  const execute = async (asyncFunction) => {
    setState({ ...state, loading: true, error: null });
    try {
      const result = await asyncFunction();
      setState({ data: result, loading: false });
    } catch (error) {
      setState({ ...state, error, loading: false });
    }
  };
  
  return { ...state, execute };
}
```

## Управление состоянием в иерархиях компонентов

Используйте комбинацию подходов:

- Локальное состояние для UI-контролов
- Context для данных, необходимых нескольким компонентам
- Внешние библиотеки для сложной бизнес-логики

## Синхронизация состояния между компонентами

```jsx
// С помощью useEffect
useEffect(() => {
  if (parentValue !== localValue) {
    setLocalValue(parentValue);
  }
}, [parentValue]);
```

## Принципы локализации состояния

- **Состояние должно быть ближе к компонентам, которые его используют**
- Минимизируйте поднятие состояния
- Используйте колбэки для передачи данных наверх

## Производительность при управлении состоянием

- Используйте `React.memo` для предотвращения ненужных рендеров
- Оптимизируйте обновления с помощью `useCallback`
- Избегайте создания новых объектов в каждом рендере

## Границы ошибок и управление состоянием

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }
    return this.props.children;
  }
}
```

## Тестирование управления состоянием

```jsx
// Тестирование кастомного хука
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount(count + 1);
  return { count, increment };
}

// В тесте
test('should increment count', () => {
  const { result } = renderHook(() => useCounter());
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(1);
});
```

## Связи с другими файлами

- [[react/hooks/useReducer]] - для понимания сложных состояний
- [[react/context-api]] - для работы с глобальным состоянием
- [[react/performance-optimization]] - для оптимизации рендеров
- [[react/testing]] - для тестирования состояния
- [[react/state-management-libraries]] - для сравнения с внешними библиотеками