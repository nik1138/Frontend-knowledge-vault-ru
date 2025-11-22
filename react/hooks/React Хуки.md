# React Хуки

React Хуки - это функции, которые позволяют использовать состояние и другие возможности React в функциональных компонентах. Хуки были представлены в React 16.8 и позволяют использовать функциональные компоненты для большинства задач, которые ранее требовали классовые компоненты.

## Основные хуки

### useState
Хук `useState` позволяет добавить состояние в функциональный компонент.

```jsx
import React, { useState } from 'react';

function Example() {
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
Хук `useEffect` позволяет выполнять побочные эффекты в функциональных компонентах.

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Вы нажали ${count} раз`;
  });

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

### useContext
Хук `useContext` позволяет использовать значение контекста без использования компонента-обертки.

```jsx
import React, { useContext } from 'react';

const ThemeContext = createContext();

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Кнопка</button>;
}
```

### useReducer
Хук `useReducer` - альтернатива `useState`, полезная для сложной логики обновления состояния.

```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <div>
      Счетчик: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </div>
  );
}
```

## Пользовательские хуки

Пользовательские хуки позволяют повторно использовать логику состояния между компонентами.

```jsx
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}

function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Загрузка...';
  }
  return isOnline ? 'Онлайн' : 'Офлайн';
}
```

## Правила использования хуков

1. Используйте хуки только на верхнем уровне. Не вызывайте их внутри циклов, условий или вложенных функций.
2. Вызывайте хуки только из функциональных компонентов React или пользовательских хуков.

## Связанные темы

- [[Композиция Компонентов]]
- [[React Компоненты]]
- [[Компонентная Архитектура]]
- [[Управление Состоянием]]
- [[React Контекст]]

## Теги

#react #hooks #javascript #frontend #components