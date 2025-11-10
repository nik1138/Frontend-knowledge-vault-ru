---
tags: [react, hooks, patterns, state-management, performance]
aliases: [react-hooks-advanced, modern-react-hooks]
---

# Современные паттерны React хуков

## Введение

React хуки революционизировали способ управления состоянием и побочными эффектами в функциональных компонентах. Современные паттерны хуков позволяют создавать более чистый, переиспользуемый и производительный код.

## Создание пользовательских хуков

Пользовательские хуки позволяют извлекать логику компонентов в повторно используемые функции. Они всегда начинаются с префикса `use`.

```javascript
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}
```

## Продвинутые паттерны useState

### Функциональное обновление

Используйте функциональное обновление для доступа к предыдущему состоянию:

```javascript
const [state, setState] = useState({ count: 0, name: '' });

const increment = () => {
  setState(prev => ({ ...prev, count: prev.count + 1 }));
};
```

### Ленивая инициализация

Для дорогостоящих вычислений при инициализации:

```javascript
const [state, setState] = useState(() => {
  // дорогостоящая инициализация
  return computeExpensiveValue(props);
});
```

## Продвинутые паттерны useEffect

### Очистка побочных эффектов

Всегда очищайте подписки, таймеры и соединения:

```javascript
useEffect(() => {
  const subscription = dataSource.subscribe(handleData);
  
  return () => {
    subscription.unsubscribe();
  };
}, [dataSource]);
```

### Один эффект - одна цель

Разделяйте разные задачи на разные эффекты:

```javascript
// Неправильно
useEffect(() => {
  document.title = name;
  localStorage.setItem('name', name);
}, [name]);

// Правильно
useEffect(() => {
  document.title = name;
}, [name]);

useEffect(() => {
  localStorage.setItem('name', name);
}, [name]);
```

## Оптимизация с useCallback и useMemo

### useCallback для функций

Предотвращает пересоздание функций при каждом рендере:

```javascript
const handleClick = useCallback(() => {
  // обработка клика
}, [dependency]);
```

### useMemo для значений

Кэширует вычисляемые значения:

```javascript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

## Паттерны useRef за пределами DOM

### Хранение изменяемых значений

```javascript
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

### Отмена асинхронных операций

```javascript
useEffect(() => {
  let cancelled = false;
  
  fetchData().then(result => {
    if (!cancelled) {
      setData(result);
    }
  });
  
  return () => {
    cancelled = true;
  };
}, [id]);
```

## useReducer для сложного состояния

Используйте для сложной логики обновления состояния:

```javascript
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

## useContext для глобального состояния

Комбинируйте с useReducer для глобального состояния:

```javascript
const StateContext = createContext();

export function StateProvider({ children, reducer, initialState }) {
  return (
    <StateContext.Provider value={useReducer(reducer, initialState)}>
      {children}
    </StateContext.Provider>
  );
}

export const useStateValue = () => useContext(StateContext);
```

## Новые хуки React 18

### useTransition

Позволяет обновлять интерфейс без блокировки:

```javascript
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSelectedTab(newTab);
});
```

### useDeferredValue

Откладывает обновление части UI:

```javascript
const deferredValue = useDeferredValue(value);
return <SlowComponent value={deferredValue} />;
```

## Правила хуков

1. **Вызывайте хуки только на верхнем уровне** - не вызывайте хуки внутри циклов, условий или вложенных функций
2. **Вызывайте хуки только из React компонентов** - не вызывайте хуки из обычных JavaScript функций
3. **Эффекты с зависимостями должны быть указаны в массиве зависимостей**

## Поток хуков в компонентах

Хуки вызываются в порядке определения при каждом рендере. Порядок критичен для сохранения состояния между рендерами.

## Ловушки массива зависимостей

### Пропущенные зависимости

```javascript
// Плохо - функция использует count но не указана в зависимостях
useEffect(() => {
  const handler = () => console.log(count);
  window.addEventListener('click', handler);
  return () => window.removeEventListener('click', handler);
}, []); // Пропущен count
```

### Часто изменяющиеся зависимости

```javascript
// Лучше - извлекаем в эффект или используем useCallback
useEffect(() => {
  const handler = () => console.log(count);
  window.addEventListener('click', handler);
  return () => window.removeEventListener('click', handler);
}, [count]);
```

## Лучшие практики пользовательских хуков

- Начинайте с `use`
- Возвращайте структуру данных, а не функции
- Обрабатывайте жизненный цикл компонента
- Используйте деструктуризацию при вызове
- Документируйте API

## Производительность с хуками

- Используйте `React.memo` для предотвращения ненужных рендеров
- Избегайте создания новых объектов/функций в рендере
- Оптимизируйте массивы зависимостей
- Используйте `useCallback` и `useMemo` разумно

## Тестирование пользовательских хуков

Используйте React Hooks Testing Library:

```javascript
import { renderHook, act } from '@testing-library/react-hooks';

test('should increment counter', () => {
  const { result } = renderHook(() => useCounter(0));
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Композиция хуков

Создавайте более сложные хуки из простых:

```javascript
function useApiData(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url).then(res => res.json()).then(setData).finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading };
}

function useUser(userId) {
  return useApiData(`/api/users/${userId}`);
}
```

## Распространенные паттерны пользовательских хуков

- `useForm` для управления формами
- `useLocalStorage` для работы с localStorage
- `useDebounce` для дебаунса
- `useOnClickOutside` для обработки кликов вне элемента
- `useMediaQuery` для медиа запросов

## Связь с другими файлами

- [[react/state-management]] - Управление состоянием в React
- [[react/performance]] - Оптимизация производительности React приложений
- [[react/context-api]] - Глубокое погружение в Context API
- [[react/testing]] - Тестирование React компонентов и хуков
- [[ts/react-types]] - Типизация React хуков с TypeScript

## Заключение

Современные паттерны React хуков позволяют создавать более чистый, переиспользуемый и производительный код. Понимание этих паттернов критично для разработки современных React приложений.