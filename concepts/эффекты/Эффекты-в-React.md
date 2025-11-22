---
aliases: [React Effects, React useEffect, Эффекты в Реакт]
tags: [react, frontend, effects, hooks]
---

# Эффекты-в-React

## Введение

Эффекты в React позволяют выполнять [[Побочные-эффекты]] в функциональных компонентах. Это основной способ работы с операциями, которые не должны происходить во время рендеринга компонента, таких как подписки, таймеры, сетевые запросы и манипуляции с DOM.

## Основы useEffect

React предоставляет хук `useEffect`, который объединяет функциональность методов жизненного цикла из классовых компонентов (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`).

```javascript
import React, { useEffect, useState } from 'react';

function ExampleComponent() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Вы нажали ${count} раз`;
  });

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
}
```

### Чистка эффектов

Для предотвращения утечек памяти и нежелательных побочных эффектов, важно правильно очищать ресурсы:

```javascript
useEffect(() => {
  const subscription = subscribeToSomeData();

  return () => {
    // Очистка при размонтировании
    subscription.unsubscribe();
  };
}, []); // Пустой массив зависимостей означает выполнение только при монтировании
```

## Типы эффектов

### 1. Эффекты без зависимостей

Выполняются после каждого рендера:

```javascript
useEffect(() => {
  console.log('Этот эффект выполняется после каждого рендера');
});
```

### 2. Эффекты с зависимостями

Выполняются только при изменении указанных зависимостей:

```javascript
useEffect(() => {
  console.log('Этот эффект выполняется при изменении count');
}, [count]);
```

### 3. Эффекты с пустым массивом зависимостей

Выполняются только при монтировании компонента:

```javascript
useEffect(() => {
  console.log('Этот эффект выполняется только при монтировании');
}, []);
```

## Практические примеры

### Работа с API

```javascript
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Ошибка загрузки пользователя:', error);
      } finally {
        setLoading(false);
      }
    }

    if (userId) {
      fetchUser();
    }
  }, [userId]); // Перезагружаем при изменении userId

  if (loading) return <div>Загрузка...</div>;
  if (!user) return <div>Пользователь не найден</div>;

  return <div>Профиль: {user.name}</div>;
}
```

### Подписка на события

```javascript
import { useState, useEffect } from 'react';

function WindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return (
    <div>
      Ширина: {windowSize.width}, Высота: {windowSize.height}
    </div>
  );
}
```

## Оптимизация эффектов

### useLayoutEffect

Для синхронного выполнения эффектов после обновления DOM:

```javascript
import { useLayoutEffect, useState } from 'react';

function MeasureExample() {
  const [height, setHeight] = useState(0);
  const ref = useRef();

  useLayoutEffect(() => {
    setHeight(ref.current.getBoundingClientRect().height);
  }, []);

  return (
    <div ref={ref}>
      <p>Этот элемент имеет высоту: {height}px</p>
    </div>
  );
}
```

## Распространенные ошибки

### 1. Забытые зависимости

```javascript
// ПЛОХО - эффект не обновляется при изменении userId
useEffect(() => {
  fetchData(userId);
}, []); // userId не включен в зависимости

// ХОРОШО - эффект обновляется при изменении userId
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

### 2. Неправильная очистка

```javascript
// ПЛОХО - возврат не функции
useEffect(() => {
  subscribe();
  return unsubscribe; // Неправильно!
}, []);

// ХОРОШО - возврат функции очистки
useEffect(() => {
  subscribe();
  return () => {
    unsubscribe();
  };
}, []);
```

## Альтернативы useEffect

### useReducer для сложной логики

```javascript
import { useReducer, useEffect } from 'react';

const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  useEffect(() => {
    const timer = setInterval(() => {
      dispatch({ type: 'increment' });
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return (
    <div>
      <p>Счетчик: {state.count}</p>
      <button onClick={() => dispatch({ type: 'setStep', payload: 2 })}>
        Увеличить шаг до 2
      </button>
    </div>
  );
}
```

## Заключение

Эффекты в React - мощный инструмент для работы с побочными эффектами. Правильное использование `useEffect` позволяет создавать компоненты, которые реагируют на изменения и управляют внешними ресурсами без утечек памяти.

См. также:
- [[Побочные-эффекты]]
- [[Управление-эффектами]]
- [[Эффекты-в-Vue]]
- [[Эффекты-в-Svelte]]