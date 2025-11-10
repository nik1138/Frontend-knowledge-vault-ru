---
aliases: ["React State", "React Состояние"]
tags: 
  - react
  - state
  - javascript
  - frontend
  - programming
---

# Подробное руководство по состоянию в React

## Введение в состояние React

Состояние (state) в React — это объект, который содержит данные, которые могут изменяться в течение жизненного цикла компонента. В отличие от пропсов (props), которые передаются извне и неизменяемы для компонента, состояние управляется самим компонентом и может изменяться.

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

Состояние является основой для реактивности в React: когда состояние изменяется, компонент перерисовывается с новыми данными.

## Локальное состояние против глобального состояния

### Локальное состояние

Локальное состояние принадлежит одному компоненту и не доступно для других компонентов. Это самая распространенная форма состояния в React-приложениях.

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(userId).then(setUser).finally(() => setLoading(false));
  }, [userId]);

  if (loading) return <div>Загрузка...</div>;
  return <div>{user.name}</div>;
}
```

### Глобальное состояние

Глобальное состояние доступно для нескольких компонентов в приложении. Используется для данных, которые должны быть доступны в разных частях приложения (например, аутентификация, настройки пользователя и т.д.).

Для управления глобальным состоянием используются библиотеки такие как [[react/state-management]] Redux, Zustand, или контекст React.

## setState в классовых компонентах

В классовых компонентах состояние управляется через метод `setState()`:

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  // Альтернативный способ с функцией
  incrementAsync = () => {
    this.setState((prevState) => ({
      count: prevState.count + 1
    }));
  };

  render() {
    return (
      <div>
        <p>Счетчик: {this.state.count}</p>
        <button onClick={this.increment}>Увеличить</button>
      </div>
    );
  }
}
```

> [!NOTE]
> Важно использовать функциональную форму setState при обновлении состояния на основе предыдущего состояния, чтобы избежать проблем с асинхронностью.

## useState в функциональных компонентах

Хук `useState` позволяет добавлять состояние в функциональные компоненты:

```jsx
import { useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, { id: Date.now(), text: inputValue, completed: false }]);
      setInputValue('');
    }
  };

  return (
    <div>
      <input 
        value={inputValue} 
        onChange={(e) => setInputValue(e.target.value)} 
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Добавить</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Неизменяемость состояния

Состояние в React должно быть неизменяемым (immutable). Это означает, что мы не должны изменять объекты или массивы напрямую, а создавать новые экземпляры:

```jsx
// НЕПРАВИЛЬНО
const updateTodo = (id) => {
  todos[id].completed = true; // ПЛОХО: изменение напрямую
  setTodos(todos);
};

// ПРАВИЛЬНО
const updateTodo = (id) => {
  setTodos(todos.map(todo => 
    todo.id === id ? { ...todo, completed: true } : todo
  ));
};
```

## Паттерны управления состоянием

### Поднятие состояния (Lifting State Up)

Когда несколько компонентов нуждаются в доступе к одному и тому же состоянию, его следует поднять к ближайшему общему предку:

```jsx
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  return (
    <fieldset>
      <legend>Введите температуру в {scaleNames[scale]}:</legend>
      <input 
        value={temperature} 
        onChange={(e) => onTemperatureChange(e.target.value)} 
      />
    </fieldset>
  );
}

function Calculator() {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');

  return (
    <div>
      <TemperatureInput 
        scale="c" 
        temperature={temperature}
        onTemperatureChange={setTemperature} 
      />
      <TemperatureInput 
        scale="f" 
        temperature={temperature}
        onTemperatureChange={setTemperature} 
      />
    </div>
  );
}
```

### Локализация состояния (State Colocation)

Состояние должно быть как можно ближе к компонентам, которые его используют. Это упрощает понимание и поддержку кода.

## Комплексное состояние с useReducer

Для сложной логики изменения состояния рекомендуется использовать `useReducer`:

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
    case 'reset':
      return initialState;
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Счетчик: {state.count}</p>
      <p>Шаг: {state.step}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <input 
        type="number" 
        value={state.step} 
        onChange={(e) => dispatch({ 
          type: 'setStep', 
          payload: Number(e.target.value) 
        })}
      />
      <button onClick={() => dispatch({ type: 'reset' })}>Сброс</button>
    </div>
  );
}
```

## Сохранение состояния

Для сохранения состояния между сессиями можно использовать localStorage:

```jsx
function useLocalStorageState(key, defaultValue) {
  const [state, setState] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error(error);
      return defaultValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(state));
    } catch (error) {
      console.error(error);
    }
  }, [key, state]);

  return [state, setState];
}
```

## Валидация состояния

Состояние может потребовать валидации перед обновлением:

```jsx
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Валидация при изменении
    const newErrors = validate({ ...values, [name]: value });
    setErrors(newErrors);
  };

  return { values, errors, handleChange };
}
```

## Синхронизация состояния

Для синхронизации состояния между компонентами можно использовать `useContext`:

```jsx
const StateContext = createContext();

function StateProvider({ children, initialState }) {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <StateContext.Provider value={{ state, dispatch }}>
      {children}
    </StateContext.Provider>
  );
}

function useAppState() {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error('useAppState must be used within StateProvider');
  }
  return context;
}
```

## Отладка состояния

### Инструменты разработчика React

React DevTools позволяют просматривать состояние компонентов, отслеживать изменения и анализировать производительность.

### Логирование изменений состояния

```jsx
function useStateWithLogging(initialValue) {
  const [state, setState] = useState(initialValue);

  const setStateWithLogging = (newState) => {
    console.log('Предыдущее состояние:', state);
    const newStateValue = typeof newState === 'function' ? newState(state) : newState;
    console.log('Новое состояние:', newStateValue);
    setState(newStateValue);
  };

  return [state, setStateWithLogging];
}
```

## Сравнение библиотек управления состоянием

| Библиотека | Преимущества | Недостатки |
|------------|--------------|------------|
| Redux | Предсказуемость, инструменты разработчика | Сложность настройки |
| Zustand | Простота использования | Меньше инструментов разработчика |
| Context API | Встроена в React | Может вызвать лишние перерисовки |

## Производительность и состояние

### Оптимизация перерисовок

Использование `React.memo`, `useMemo` и `useCallback` помогает избежать ненужных перерисовок:

```jsx
const TodoItem = memo(({ todo, onUpdate }) => {
  return (
    <div>
      <span>{todo.text}</span>
      <button onClick={() => onUpdate(todo.id, !todo.completed)}>
        {todo.completed ? 'Отменить' : 'Выполнить'}
      </button>
    </div>
  );
});
```

### Разделение состояния

Разделение сложного состояния на несколько хуков `useState` может улучшить производительность:

```jsx
// Вместо одного объекта состояния
const [state, setState] = useState({ user: null, loading: false, error: null });

// Лучше использовать несколько useState
const [user, setUser] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
```

## Связи с другими файлами

- [[react/hooks]] - Подробнее о хуках React
- [[react/context]] - Контекст для передачи состояния
- [[react/state-management]] - Управление состоянием в приложениях
- [[react/performance]] - Оптимизация производительности
- [[react/lifecycle]] - Жизненный цикл компонентов

## Заключение

Состояние в React — это фундаментальный концепт, который позволяет создавать интерактивные приложения. Понимание различных подходов к управлению состоянием, их преимуществ и ограничений, помогает создавать более эффективные и поддерживаемые приложения.