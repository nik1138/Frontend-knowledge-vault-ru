# React Компоненты

React Компоненты - это независимые и повторно используемые блоки кода, которые возвращают элементы React для отображения в пользовательском интерфейсе. Компоненты позволяют разделить интерфейс на независимые части и работать с каждой из них отдельно.

## Типы компонентов

### Функциональные компоненты

Функциональные компоненты - это JavaScript-функции, которые принимают пропсы (props) и возвращают элементы React.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Современный синтаксис с деструктуризацией
const Welcome = ({ name }) => {
  return <h1>Привет, {name}!</h1>;
};

// Компонент без пропсов
const Greeting = () => {
  return <h1>Привет, мир!</h1>;
};
```

### Классовые компоненты

Классовые компоненты - это ES6-классы, расширяющие `React.Component`.

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

## Пропсы (Props)

Пропсы - это аргументы, передаваемые компонентам. Они доступны только для чтения и позволяют передавать данные от родительского компонента к дочернему.

```jsx
// Родительский компонент
function App() {
  return <Welcome name="Алиса" />;
}

// Дочерний компонент
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}
```

## Состояние (State)

Состояние позволяет компонентам изменять свое поведение и отображение в ответ на действия пользователя или изменения данных.

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

## Дочерние элементы (Children)

Компоненты могут принимать дочерние элементы через специальный пропс `children`.

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Использование
function App() {
  return (
    <Card title="Моя карточка">
      <p>Это содержимое карточки</p>
      <button>Кнопка</button>
    </Card>
  );
}
```

## Композиция компонентов

Компоненты можно комбинировать для создания более сложных интерфейсов.

```jsx
function Page() {
  return (
    <div>
      <Header />
      <Navigation />
      <MainContent />
      <Footer />
    </div>
  );
}
```

## Повторное использование компонентов

Компоненты позволяют создавать повторно используемые элементы интерфейса.

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

function UserList({ users }) {
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}
```

## Пользовательские хуки в компонентах

Пользовательские хуки позволяют извлекать логику компонентов в повторно используемые функции.

```jsx
// Пользовательский хук
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

// Использование в компоненте
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <div className="friend-status">
      {isOnline ? 'Онлайн' : 'Офлайн'}
    </div>
  );
}
```

## Условная отрисовка

Компоненты могут отображать разные элементы в зависимости от условий.

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

// Использование в JSX
function Mailbox({ unreadMessages }) {
  return (
    <div>
      <h1>Привет!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          У вас {unreadMessages.length} непрочитанных сообщений.
        </h2>
      }
    </div>
  );
}
```

## Лучшие практики

1. Компоненты должны быть маленькими и отвечать за одну задачу
2. Используйте осмысленные имена для компонентов
3. Используйте деструктуризацию пропсов
4. Пропсы должны быть неизменяемыми
5. Используйте TypeScript для лучшей типизации компонентов

## Связанные темы

- [[Композиция Компонентов]]
- [[React Хуки]]
- [[React Контекст]]
- [[Компонентная Архитектура]]
- [[Паттерны Проектирования]]
- [[Управление Состоянием]]

## Теги

#react #components #javascript #frontend #jsx #ui