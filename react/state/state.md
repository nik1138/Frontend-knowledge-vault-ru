---
aliases: ["React state management", "Управление состоянием в React"]
tags: 
  - "#react"
  - "#state-management"
  - "#frontend"
  - "#javascript"
---

# Управление состоянием в React

## Введение

Управление состоянием - это ключевой аспект разработки приложений на React. Состояние определяет, как компоненты реагируют на пользовательские действия, изменения данных и другие события.

## Типы состояния

### Локальное состояние (Local State)

Локальное состояние используется в пределах одного компонента и управляется с помощью хуков [[react/hooks/usestate]] или `this.setState` в классовых компонентах.

```jsx
import React, { useState } from 'react';

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

### Глобальное состояние (Global State)

Глобальное состояние используется приложением в целом и доступно всем компонентам. Для управления глобальным состоянием используются такие инструменты как [[react/context-api]] или внешние библиотеки.

### Серверное состояние (Server State)

Серверное состояние - это данные, полученные с сервера. Для управления этим типом состояния часто используются библиотеки, такие как React Query, SWR или Apollo Client.

## setState и обновления состояния

В функциональных компонентах обновления состояния происходят через вызов функций, возвращаемых хуками. Важно понимать, что обновления состояния могут быть асинхронными и пакетными.

```jsx
// Пакетные обновления
const [count, setCount] = useState(0);

setCount(prev => prev + 1); // count = 0
setCount(prev => prev + 1); // count = 1
setCount(prev => prev + 1); // count = 2
```

## useState хук

Хук `useState` позволяет добавить состояние в функциональные компоненты. Он возвращает пару значений: текущее состояние и функцию для его обновления.

```jsx
const [state, setState] = useState(initialState);
```

## useEffect и побочные эффекты состояния

Хук `useEffect` позволяет выполнять побочные эффекты в функциональных компонентах. Это может быть загрузка данных, подписка на события или манипуляции с DOM.

```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  return user ? <div>{user.name}</div> : <div>Загрузка...</div>;
}
```

## Паттерны управления состоянием

### Поднятие состояния (Lifting State Up)

Когда несколько компонентов нуждаются в одном и том же изменяющемся состоянии, его следует поднять к ближайшему общему предку.

### Соседство состояния (State Colocation)

Состояние должно быть как можно ближе к компонентам, которые его используют. Это упрощает понимание потока данных.

## Context API

[[react/context-api]] позволяет передавать данные через дерево компонентов без необходимости передавать пропсы на каждом уровне.

```jsx
import React, { createContext, useContext } from 'react';

const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Кнопка</button>;
}
```

## useReducer хук

Хук `useReducer` является альтернативой `useState`, особенно полезной для сложных логик обновления состояния.

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

## Библиотеки глобального управления состоянием

### Redux

Redux - это предсказуемый контейнер состояния для JavaScript-приложений. Он помогает писать приложения, которые ведут себя предсказуемо.

> [!NOTE]
> Подробнее о Redux см. в файле [[react/state/redux]]

### Zustand

Zustand - это легковесное решение для управления состоянием, не требующее дополнительных компонентов-оберток.

### MobX

MobX - это простая и масштабируемая библиотека управления состоянием, использующая реактивное программирование.

## Рассмотрение производительности

При управлении состоянием важно учитывать производительность:

- Избегайте ненужных перерисовок с помощью `React.memo`
- Используйте `useMemo` и `useCallback` для оптимизации вычислений
- Нормализуйте состояние для предотвращения дублирования данных

## Нормализация состояния

Нормализация данных помогает избежать дублирования информации и упрощает обновление состояния.

```jsx
// Плохо: дублирование данных
const state = {
  posts: [
    { id: 1, title: 'Пост 1', author: { id: 1, name: 'Иван' } },
    { id: 2, title: 'Пост 2', author: { id: 1, name: 'Иван' } }
  ]
};

// Хорошо: нормализованные данные
const state = {
  posts: {
    1: { id: 1, title: 'Пост 1', authorId: 1 },
    2: { id: 2, title: 'Пост 2', authorId: 1 }
  },
  authors: {
    1: { id: 1, name: 'Иван' }
  }
};
```

## Сохранение состояния

Для сохранения состояния между сессиями можно использовать `localStorage`, `sessionStorage` или другие механизмы сохранения данных.

```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}
```

## Тестирование компонентов со состоянием

При тестировании компонентов со сложным состоянием важно:

- Изолировать логику обновления состояния
- Использовать тестовые утилиты React Testing Library
- Проверять различные состояния компонента

## Заключение

Управление состоянием в React требует понимания различных подходов и инструментов. Выбор подходящей стратегии зависит от сложности приложения и требований к масштабируемости.

> [!TIP]
> Для изучения других аспектов React см. [[react/react-overview]]

> [!INFO]
> Связанные темы: [[react/hooks/usestate]], [[react/hooks/useeffect]], [[react/context-api]], [[react/state/redux]], [[react/performance-optimization]]