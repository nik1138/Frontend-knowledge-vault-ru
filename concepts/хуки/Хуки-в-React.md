---
aliases: [React Hooks, Хуки React, Реакт Хуки]
tags: [react, hooks, frontend, javascript]
---

# Хуки в React

## Введение

Хуки - это функции, которые позволяют использовать состояние и другие возможности React без написания классов. Они были представлены в React 16.8 и значительно изменили подход к созданию компонентов, делая код более читаемым и переиспользуемым.

## Основные хуки

### useState

`useState` - это хук, который позволяет добавить состояние в функциональные компоненты.

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}
```

### useEffect

`useEffect` позволяет выполнять побочные эффекты в функциональных компонентах. Это объединяет функциональность `componentDidMount`, `componentDidUpdate` и `componentWillUnmount`.

```jsx
import React, { useState, useEffect } from 'react';

function UserStatus({ userId }) {
  const [status, setStatus] = useState(null);

  useEffect(() => {
    async function fetchStatus() {
      const response = await fetch(`/api/user/${userId}/status`);
      const data = await response.json();
      setStatus(data.status);
    }

    fetchStatus();

    // Функция очистки
    return () => {
      console.log('Компонент будет размонтирован');
    };
  }, [userId]); // Зависимости

  return <div>Статус пользователя: {status}</div>;
}
```

### useContext

`useContext` позволяет подписываться на React контекст без вложенности компонентов.

```jsx
import React, { useContext, useState } from 'react';

const ThemeContext = React.createContext();

function ThemedComponent() {
  const theme = useContext(ThemeContext);

  return (
    <div style={{ backgroundColor: theme.background, color: theme.color }}>
      Контент с темой
    </div>
  );
}

function App() {
  const [theme, setTheme] = useState({
    background: '#fff',
    color: '#000'
  });

  return (
    <ThemeContext.Provider value={theme}>
      <ThemedComponent />
    </ThemeContext.Provider>
  );
}
```

### useReducer

`useReducer` - альтернатива для `useState`, полезная для сложной логики состояния.

```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      Счетчик: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Сброс</button>
    </>
  );
}
```

## Дополнительные хуки

### useMemo

`useMemo` позволяет мемоизировать вычисления между рендерами компонента.

```jsx
import React, { useState, useMemo } from 'react';

function ExpensiveComponent({ items, multiplier }) {
  const expensiveValue = useMemo(() => {
    console.log('Вычисление ресурсоемкой функции');
    return items.reduce((acc, item) => acc + item.value * multiplier, 0);
  }, [items, multiplier]);

  return <div>Результат: {expensiveValue}</div>;
}
```

### useCallback

`useCallback` мемоизирует функцию между рендерами компонента.

```jsx
import React, { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Пустой массив зависимостей, функция не будет пересоздаваться

  return (
    <div>
      <p>Счетчик: {count}</p>
      <ChildComponent onIncrement={increment} />
    </div>
  );
}

const ChildComponent = React.memo(({ onIncrement }) => {
  console.log('Рендер ChildComponent');
  return <button onClick={onIncrement}>Увеличить</button>;
});
```

### useRef

`useRef` позволяет получить доступ к DOM-элементу или хранить изменяемое значение.

```jsx
import React, { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" placeholder="Фокус при монтировании" />;
}
```

## Правила использования хуков

1. **Вызывайте хуки только на верхнем уровне**: Не вызывайте хуки внутри циклов, условий или вложенных функций.
2. **Вызывайте хуки только из React-компонентов**: Не вызывайте хуки из обычных функций JavaScript.

## Заключение

Хуки значительно упрощают работу с состоянием и побочными эффектами в функциональных компонентах React. Они позволяют создавать более чистый и понятный код, а также способствуют переиспользованию логики между компонентами.

См. также:
- [[Пользовательские-хуки]]
- [[Лучшие-практики-хуков]]
- [[Хуки-в-Vue]]
- [[Хуки-в-Svelte]]