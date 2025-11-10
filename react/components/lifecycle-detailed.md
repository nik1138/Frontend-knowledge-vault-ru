---
aliases: ["React Lifecycle", "Методы жизненного цикла", "Жизненный цикл компонента"]
tags: ["react", "lifecycle", "components", "javascript", "frontend"]
---

# Методы жизненного цикла React (React Lifecycle Methods)

Методы жизненного цикла React - это специальные функции, которые вызываются на разных этапах существования компонента. Они позволяют выполнять код в определенные моменты жизненного цикла компонента: при его создании, обновлении и удалении.

## Фазы жизненного цикла

Жизненный цикл компонента делится на три основные фазы:

- [[#Монтаж (Mounting)]]
- [[#Обновление (Updating)]]
- [[#Демонтаж (Unmounting)]]

## Монтаж (Mounting)

Фаза монтажа - это когда компонент создается и вставляется в DOM. В этой фазе вызываются следующие методы:

### constructor(props)

Конструктор вызывается первым при создании компонента. Он используется для инициализации состояния и привязки методов.

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }
  
  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return <button onClick={this.handleClick}>{this.state.count}</button>;
  }
}
```

> [!tip] Совет
> В современном React рекомендуется использовать поля класса вместо конструктора для инициализации состояния:
> ```javascript
> state = { count: 0 };
> handleClick = () => {
>   this.setState({ count: this.state.count + 1 });
> };
> ```

### render()

Метод `render()` - это единственный обязательный метод в классовом компоненте. Он должен быть чистой функцией, возвращающей JSX.

### componentDidMount()

Этот метод вызывается сразу после монтирования компонента (вставки в DOM). Это место для выполнения побочных эффектов:

```javascript
class UserProfile extends React.Component {
  state = { user: null };
  
  componentDidMount() {
    fetch(`/api/users/${this.props.userId}`)
      .then(response => response.json())
      .then(user => this.setState({ user }));
  }
  
  render() {
    return this.state.user ? <h1>{this.state.user.name}</h1> : <p>Загрузка...</p>;
  }
}
```

## Обновление (Updating)

Компонент обновляется при изменении пропсов или состояния. В этой фазе вызываются следующие методы:

### static getDerivedStateFromProps(props, state)

Вызывается перед `render()`, когда компонент получает новые пропсы. Редко используется, возвращает обновленное состояние или null.

```javascript
static getDerivedStateFromProps(nextProps, prevState) {
  if (nextProps.userID !== prevState.userID) {
    return {
      userID: nextProps.userID,
      loading: true
    };
  }
  return null;
}
```

### shouldComponentUpdate(nextProps, nextState)

Позволяет указать, должен ли компонент обновляться. Возвращение `false` предотвращает вызов `render()`.

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.color !== this.props.color;
}
```

### getSnapshotBeforeUpdate(prevProps, prevState)

Вызывается сразу перед коммитом изменений в DOM. Позволяет захватить информацию из DOM до обновления.

```javascript
getSnapshotBeforeUpdate(prevProps, prevState) {
  if (prevProps.list.length < this.props.list.length) {
    const list = this.listRef.current;
    return list.scrollHeight - list.scrollTop;
  }
  return null;
}

componentDidUpdate(prevProps, prevState, snapshot) {
  if (snapshot !== null) {
    const list = this.listRef.current;
    list.scrollTop = list.scrollHeight - snapshot;
  }
}
```

### componentDidUpdate(prevProps, prevState, snapshot)

Вызывается сразу после обновления компонента. Подходит для выполнения операций после обновления.

## Демонтаж (Unmounting)

### componentWillUnmount()

Вызывается перед удалением компонента из DOM. Используется для очистки ресурсов:

```javascript
componentWillUnmount() {
  window.removeEventListener('resize', this.handleResize);
  this.timerID && clearInterval(this.timerID);
}
```

## Границы ошибок (Error Boundaries)

### static getDerivedStateFromError(error)

Используется для изменения состояния при ошибке, чтобы показать резервный UI.

### componentDidCatch(error, info)

Используется для логирования информации об ошибке.

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    console.error('Ошибка компонента:', error);
    console.error('Информация:', info);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}
```

## Устаревшие методы жизненного цикла

Некоторые методы были помечены как устаревшие:

- `componentWillMount` → Используйте `constructor` или `useEffect`
- `componentWillReceiveProps` → Используйте `static getDerivedStateFromProps`
- `componentWillUpdate` → Обычно не нужен, используйте `componentDidUpdate`

## Хуки как замена методам жизненного цикла

С появлением хуков в React 16.8, функциональные компоненты могут использовать методы жизненного цикла через хуки:

- `useEffect` → `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`
- `useState` → `this.state`
- `useContext` → `context`

```javascript
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(response => response.json())
      .then(setUser);
    
    return () => {
      // Аналог componentWillUnmount
      console.log('Компонент будет размонтирован');
    };
  }, [userId]); // Зависимости, аналог componentDidUpdate
  
  return user ? <h1>{user.name}</h1> : <p>Загрузка...</p>;
}
```

## Миграция с методов жизненного цикла на хуки

При миграции с классовых компонентов на функциональные с хуками:

1. Замените `constructor` на `useState`
2. Замените `componentDidMount` и `componentDidUpdate` на `useEffect`
3. Замените `componentWillUnmount` на возврат функции из `useEffect`

## Практические рекомендации

- Используйте функциональные компоненты с хуками вместо классовых компонентов
- Избегайте ненужных обновлений с помощью `shouldComponentUpdate` или `React.memo`
- Не мутируйте состояние напрямую
- Используйте `PureComponent` для поверхностного сравнения пропсов и состояния

## Отладка методов жизненного цикла

Для отладки можно использовать:

- `console.log` внутри методов жизненного цикла
- React DevTools для отслеживания обновлений
- React Profiler для анализа производительности

## Связанные темы

- [[react/components/class-vs-functional]] - Сравнение классовых и функциональных компонентов
- [[react/hooks/basics]] - Основы React хуков
- [[react/state-management]] - Управление состоянием в React
- [[react/performance]] - Оптимизация производительности React приложений