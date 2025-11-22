---
aliases: [Жизненный цикл React, Методы жизненного цикла React]
tags: [frontend, react, component, lifecycle]
---

# Методы жизненного цикла в React

React предоставляет разработчикам набор методов жизненного цикла, которые позволяют выполнять определенные действия на различных этапах существования компонента. Эти методы дают контроль над монтированием, обновлением и размонтированием компонентов.

## Классовые компоненты и методы жизненного цикла

В классовых компонентах React предоставляет специальные методы, которые вызываются автоматически на разных этапах жизненного цикла.

### Фаза монтирования

Эти методы вызываются при создании компонента и его вставке в DOM:

#### constructor(props)
```javascript
constructor(props) {
  super(props);
  this.state = { count: 0 };
  // Привязка методов
  this.handleClick = this.handleClick.bind(this);
}
```
Конструктор вызывается первым при создании компонента. Здесь инициализируется состояние и привязываются методы.

#### static getDerivedStateFromProps(props, state)
```javascript
static getDerivedStateFromProps(props, state) {
  // Синхронизация состояния с пропсами
  if (props.initialCount !== state.prevInitialCount) {
    return {
      count: props.initialCount,
      prevInitialCount: props.initialCount
    };
  }
  return null;
}
```
Вызывается перед render. Редко используется, позволяет обновить состояние в ответ на изменение пропсов.

#### render()
```javascript
render() {
  return (
    <div>
      <p>Count: {this.state.count}</p>
      <button onClick={this.handleClick}>Increment</button>
    </div>
  );
}
```
Обязательный метод, возвращающий JSX. Должен быть чистой функцией без побочных эффектов.

#### componentDidMount()
```javascript
componentDidMount() {
  // Выполнение после монтирования
  fetch('/api/data')
    .then(response => response.json())
    .then(data => this.setState({ data }));
  
  // Подписка на события
  document.addEventListener('keydown', this.handleKeyDown);
}
```
Вызывается после монтирования компонента. Используется для выполнения операций, требующих DOM-элементов, таких как сетевые запросы или подписки.

### Фаза обновления

Эти методы вызываются при изменении пропсов или состояния:

#### static getDerivedStateFromProps(props, state)
Тот же метод, что и в фазе монтирования, вызывается также перед обновлением.

#### shouldComponentUpdate(nextProps, nextState)
```javascript
shouldComponentUpdate(nextProps, nextState) {
  // Оптимизация перерендеринга
  return this.props.count !== nextProps.count;
}
```
Позволяет контролировать, нужно ли обновлять компонент. Возвращает true/false.

#### render()
Тот же метод, что и в фазе монтирования.

#### getSnapshotBeforeUpdate(prevProps, prevState)
```javascript
getSnapshotBeforeUpdate(prevProps, prevState) {
  // Сохранение информации перед обновлением
  if (prevProps.list.length < this.props.list.length) {
    const list = this.listRef.current;
    return list.scrollHeight - list.scrollTop;
  }
  return null;
}
```
Вызывается перед обновлением DOM. Позволяет захватить информацию из DOM до его изменения.

#### componentDidUpdate(prevProps, prevState, snapshot)
```javascript
componentDidUpdate(prevProps, prevState, snapshot) {
  // Реакция на обновление
  if (prevProps.count !== this.props.count) {
    this.logChange(this.props.count);
  }
  
  // Использование сохраненного снимка
  if (snapshot !== null) {
    const list = this.listRef.current;
    list.scrollTop = list.scrollHeight - snapshot;
  }
}
```
Вызывается после обновления компонента. Используется для выполнения операций, требующих обновленного DOM.

### Фаза размонтирования

#### componentWillUnmount()
```javascript
componentWillUnmount() {
  // Очистка ресурсов
  clearInterval(this.timer);
  document.removeEventListener('keydown', this.handleKeyDown);
  this.subscription.unsubscribe();
}
```
Вызывается перед удалением компонента из DOM. Используется для очистки ресурсов, отписки от событий и т.д.

### Обработка ошибок

#### static getDerivedStateFromError(error)
```javascript
static getDerivedStateFromError(error) {
  // Обновление состояния для отображения fallback UI
  return { hasError: true };
}
```
Вызывается при ошибке в любом дочернем компоненте. Позволяет обновить состояние для отображения запасного интерфейса.

#### componentDidCatch(error, info)
```javascript
componentDidCatch(error, info) {
  // Логирование ошибки
  console.error('Error caught by boundary:', error);
  console.log(info.componentStack);
}
```
Вызывается при ошибке в любом дочернем компоненте. Используется для логирования ошибки и информации о стеке.

## Хуки и методы жизненного цикла

С появлением хуков в React 16.8, функциональные компоненты получили возможность использовать методы жизненного цикла через хуки.

### useEffect и жизненный цикл

```javascript
import React, { useState, useEffect } from 'react';

function MyComponent({ userId }) {
  const [user, setUser] = useState(null);

  // Аналог componentDidMount
  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(error => console.error(error));
  }, []); // Пустой массив зависимостей

  // Аналог componentDidUpdate
  useEffect(() => {
    document.title = user ? user.name : 'Загрузка...';
  }, [user]); // Зависит от user

  // Аналог componentWillUnmount
  useEffect(() => {
    const subscription = subscribeToUserUpdates(userId, setUser);
    
    return () => {
      // Очистка при размонтировании
      subscription.unsubscribe();
    };
  }, [userId]);

  return user ? <div>{user.name}</div> : <div>Загрузка...</div>;
}
```

### Другие хуки

- `useState` - управление состоянием
- `useContext` - доступ к контексту
- `useReducer` - альтернатива useState для сложного состояния
- `useMemo` и `useCallback` - оптимизация производительности

## Практические рекомендации

- Используйте `useEffect` для побочных эффектов в функциональных компонентах
- Не забывайте возвращать функцию очистки в `useEffect`
- Избегайте ненужных перерендеров с помощью `shouldComponentUpdate` или `React.memo`
- Обрабатывайте ошибки с помощью Error Boundaries

## Связанные концепции

- [[Фазы-жизненного-цикла]]
- [[Методы-жизненного-цикла-в-Vue]]
- [[Методы-жизненного-цикла-в-Angular]]
- [[Практическое-применение]]
- [[Хуки]]
- [[Компонентная-архитектура]]

## Заключение

Понимание методов жизненного цикла в React позволяет создавать более эффективные и предсказуемые компоненты. С переходом на хуки, подход к работе с жизненным циклом изменился, но основные принципы остались теми же.