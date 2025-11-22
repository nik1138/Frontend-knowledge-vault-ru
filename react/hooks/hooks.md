---
aliases: ["React Hooks", "Хуки React"]
tags: 
  - react
  - hooks
  - javascript
  - functional-components
---

# React Hooks: Полное руководство

## Введение в React Hooks

React Hooks - это функции, которые позволяют использовать состояние и другие возможности React в функциональных компонентах. До появления хуков эти возможности были доступны только в классовых компонентах.

> [!info] Информация
> Хуки были представлены в React 16.8 и полностью изменили подход к созданию компонентов, сделав их более читаемыми и переиспользуемыми.

## Основные хуки

### useState

Хук `useState` позволяет добавить состояние в функциональные компоненты:

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

[[react/state-management]] для более подробной информации о управлении состоянием.

### useEffect

Хук `useEffect` объединяет методы жизненного цикла `componentDidMount`, `componentDidUpdate` и `componentWillUnmount`:

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    // Выполняется после каждого рендера
    fetchUser(userId).then(setUser);
    
    // Функция очистки
    return () => {
      // Очистка ресурсов
    };
  }, [userId]); // Зависимости

  return <div>{user ? user.name : 'Загрузка...'}</div>;
}
```

### useContext

Хук `useContext` позволяет подписываться на контекст React:

```jsx
import React, { useContext } from 'react';
import { ThemeContext } from './theme-context';

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.color }}>
      Кнопка в теме
    </button>
  );
}
```

[[react/context-api]] для более подробной информации о контексте.

### useReducer

Хук `useReducer` - альтернатива `useState`, полезен для сложного состояния:

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
    <div>
      <p>Счетчик: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Сброс</button>
    </div>
  );
}
```

[[react/state-management]] для сравнения с другими подходами управления состоянием.

## Пользовательские хуки

Пользовательские хуки позволяют извлекать логику компонентов в повторно используемые функции:

```jsx
import { useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

// Использование в компоненте
function StatusBar() {
  const isOnline = useOnlineStatus();

  return <p>{isOnline ? 'В сети' : 'Не в сети'}</p>;
}
```

[[react/custom-hooks]] для более подробной информации о создании пользовательских хуков.

## Оптимизационные хуки

### useMemo

Хук `useMemo` позволяет мемоизировать вычисления:

```jsx
import React, { useMemo } from 'react';

function ExpensiveComponent({ items, multiplier }) {
  const expensiveValue = useMemo(() => {
    // Дорогостоящее вычисление
    return items.map(item => item * multiplier);
  }, [items, multiplier]);

  return <div>{expensiveValue.join(', ')}</div>;
}
```

### useCallback

Хук `useCallback` мемоизирует функции:

```jsx
import React, { useCallback, useState } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Пустой массив зависимостей

  return (
    <div>
      <p>Счетчик: {count}</p>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

const ChildComponent = React.memo(({ onClick }) => {
  console.log('Child рендерится');
  return <button onClick={onClick}>Нажми меня</button>;
});
```

### useRef

Хук `useRef` предоставляет доступ к DOM-элементам и позволяет хранить изменяемые значения:

```jsx
import React, { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
}
```

## Расширенные хуки

### useImperativeHandle

Позволяет настраивать экземпляр, передаваемый через ref:

```jsx
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    getValue: () => inputRef.current.value,
    setValue: (val) => inputRef.current.value = val
  }));

  return <input ref={inputRef} type="text" />;
});

// Использование
function Parent() {
  const fancyInputRef = useRef();

  const handleFocus = () => {
    fancyInputRef.current.focus();
  };

  return (
    <div>
      <FancyInput ref={fancyInputRef} />
      <button onClick={handleFocus}>Фокус на инпут</button>
    </div>
  );
}
```

### useLayoutEffect

Похож на `useEffect`, но срабатывает синхронно после всех изменений DOM:

```jsx
import React, { useLayoutEffect, useRef } from 'react';

function MeasureExample() {
  const divRef = useRef();

  useLayoutEffect(() => {
    const { width, height } = divRef.current.getBoundingClientRect();
    console.log(`Размеры: ${width}x${height}`);
  }, []);

  return <div ref={divRef}>Пример измерения</div>;
}
```

### useDebugValue

Помогает отображать метку для пользовательских хуков в React DevTools:

```jsx
import React, { useDebugValue } from 'react';

function useFriendStatus(friendID) {
  const isOnline = useSomeCustomHook(friendID);

  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

## Правила использования хуков

1. **Вызывайте хуки только на верхнем уровне** - не вызывайте хуки внутри циклов, условий или вложенных функций
2. **Вызывайте хуки только из React-функций** - не вызывайте хуки из обычных JavaScript-функций

[[react/rules-of-hooks]] для более подробной информации о правилах.

## Переход от классовых компонентов

Преимущества перехода к функциональным компонентам с хуками:
- Более простое повторное использование логики
- Более компактный код
- Лучшее понимание зависимостей

## Рассмотрение производительности

- Используйте `React.memo()` для предотвращения ненужных рендеров
- Применяйте `useMemo` и `useCallback` для мемоизации
- Избегайте создания новых объектов/функций в рендере

## Распространенные ошибки

1. **Забытые зависимости в массиве зависимостей** - может привести к устаревшим замыканиям
2. **Неправильное использование объектов/массивов в зависимостях** - может вызвать бесконечные циклы
3. **Избыточное мемоизирование** - может ухудшить производительность

## Тестирование хуков

Для тестирования хуков используйте React Testing Library:

```jsx
import { renderHook, act } from '@testing-library/react-hooks';
import { useCounter } from './useCounter';

test('должен увеличивать значение', () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

## Заключение

React Hooks предоставляют мощный способ управления состоянием и логикой в функциональных компонентах. Правильное использование хуков делает код более читаемым и переиспользуемым.

## См. также

- [[Типизация компонентов React в TypeScript]] - Компоненты React
- [[react/state-management]] - Управление состоянием
- [[react/lifecycle]] - Жизненный цикл компонентов
- [[react/testing]] - Тестирование React-приложений