---
aliases: ["Типы React компонентов", "Компоненты в React"]
tags: [react, components, javascript, frontend, programming]
---

# Типы React компонентов

React предоставляет несколько различных способов создания компонентов, каждый из которых имеет свои особенности и области применения. В этом руководстве рассмотрим основные типы компонентов в React и их использование.

## Функциональные компоненты

Функциональные компоненты - это простые JavaScript-функции, которые принимают пропсы и возвращают JSX. С введением хуков в React 16.8, функциональные компоненты стали равноценной альтернативой классовым компонентам.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Или с использованием стрелочной функции
const Welcome = (props) => {
  return <h1>Привет, {props.name}!</h1>;
};
```

Функциональные компоненты идеально подходят для простых компонентов, которые не требуют сложного состояния или методов жизненного цикла. Они проще в тестировании и понимании. Подробнее о хуках можно прочитать в [[react/hooks]].

## Классовые компоненты

Классовые компоненты создаются с использованием JavaScript-классов и наследуются от `React.Component`. Они обеспечивают доступ к состоянию компонента и методам жизненного цикла.

```jsx
class Welcome extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <h1>Привет, {this.props.name}! Текущее время: {this.state.date.toLocaleTimeString()}</h1>
    );
  }
}
```

Классовые компоненты позволяют использовать состояние и методы жизненного цикла, но являются более громоздкими по сравнению с функциональными компонентами. Их по-прежнему можно использовать, но функциональные компоненты с хуками считаются более современным подходом.

## Чистые компоненты (Pure Components)

Чистые компоненты - это оптимизированные классовые компоненты, которые автоматически реализуют `shouldComponentUpdate` с поверхностным сравнением пропсов и состояния.

```jsx
import React, { PureComponent } from 'react';

class Welcome extends PureComponent {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

Использование `PureComponent` помогает избежать ненужных перерисовок, когда пропсы и состояние не изменились. Однако поверхностное сравнение может не подойти для сложных объектов.

## Компоненты с состоянием и без (Stateful vs Stateless)

**Компоненты с состоянием (Stateful)** - это компоненты, которые управляют своим внутренним состоянием. В функциональных компонентах состояние управляется с помощью хука `useState`.

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**Компоненты без состояния (Stateless)** - это компоненты, которые не управляют внутренним состоянием и зависят только от пропсов.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}
```

## Презентационные и контейнерные компоненты

**Презентационные компоненты** отвечают за визуальное отображение и не зависят от логики приложения.

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

**Контейнерные компоненты** управляют данными и логикой, передавая их презентационным компонентам.

```jsx
class UserContainer extends React.Component {
  state = {
    user: null
  };

  componentDidMount() {
    fetchUser().then(user => this.setState({ user }));
  }

  render() {
    return this.state.user ? <UserCard user={this.state.user} /> : <div>Загрузка...</div>;
  }
}
```

## Контролируемые и неконтролируемые компоненты

**Контролируемые компоненты** - это компоненты формы, значения которых контролируются React.

```jsx
function ControlledForm() {
  const [value, setValue] = useState('');

  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**Неконтролируемые компоненты** - это компоненты формы, значения которых управляются DOM.

```jsx
function UncontrolledForm() {
  const inputRef = useRef();

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" ref={inputRef} />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

## Компоненты высшего порядка (HOCs)

Компонент высшего порядка (HOC) - это функция, которая принимает компонент и возвращает новый компонент с дополнительной функциональностью.

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    
    return <WrappedComponent {...props} />;
  };
}

const ProtectedProfile = withAuth(Profile);
```

HOC позволяют повторно использовать логику между компонентами, но могут усложнить отладку. Подробнее о паттернах можно изучить в [[react/patterns]].

## Компаундные компоненты

Компаундные компоненты - это группа связанных компонентов, которые работают вместе как единое целое.

```jsx
function Select({ children, value, onChange }) {
  return (
    <div className="select">
      {React.Children.map(children, child =>
        React.cloneElement(child, { value, onChange })
      )}
    </div>
  );
}

Select.Option = function Option({ value, children, onChange }) {
  return (
    <div 
      onClick={() => onChange(value)}
      className={value === this.props.value ? 'selected' : ''}
    >
      {children}
    </div>
  );
};
```

## Паттерн Render Props

Паттерн render props позволяет делиться кодом между компонентами с помощью пропа, который является функцией.

```jsx
class DataProvider extends React.Component {
  state = { data: null };

  componentDidMount() {
    fetchData().then(data => this.setState({ data }));
  }

  render() {
    return this.props.render(this.state.data);
  }
}

function MyComponent() {
  return (
    <DataProvider
      render={data => (
        <div>{data ? <span>{data}</span> : <span>Загрузка...</span>}</div>
      )}
    />
  );
}
```

## Пользовательские хуки как компоненты

Пользовательские хуки позволяют извлекать логику состояния в переиспользуемые функции.

```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return { count, increment, decrement };
}

function Counter() {
  const { count, increment, decrement } = useCounter(0);

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

Хуки позволяют повторно использовать логику состояния между компонентами. Подробнее о хуках читайте в [[react/hooks]].

## Паттерны компоновки

Компоновка компонентов - это способ создания новых компонентов путем объединения существующих.

```jsx
// Композиция через пропсы
function Page({ header, sidebar, content, footer }) {
  return (
    <div>
      {header && <Header>{header}</Header>}
      <div className="main">
        {sidebar && <Sidebar>{sidebar}</Sidebar>}
        <Content>{content}</Content>
      </div>
      {footer && <Footer>{footer}</Footer>}
    </div>
  );
}

// Использование
<Page
  header={<Navigation />}
  sidebar={<Menu />}
  content={<MainContent />}
  footer={<Copyright />}
/>
```

## Условные компоненты

Условные компоненты отображаются в зависимости от определенных условий.

```jsx
function ConditionalComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    return <Dashboard />;
  }
  
  return <LoginForm />;
}

// Или с использованием тернарного оператора
function ConditionalRender({ user }) {
  return user ? <UserProfile user={user} /> : <GuestMessage />;
}

// Или с логическим И
function LoadingIndicator({ isLoading }) {
  return isLoading && <Spinner />;
}
```

## Заключение

React предоставляет множество способов создания компонентов, каждый из которых подходит для определенных сценариев. Понимание различий между этими типами компонентов помогает писать более эффективный и поддерживаемый код.

Для дальнейшего изучения рекомендуется ознакомиться с:
- [[react/hooks]] - для понимания работы с состоянием в функциональных компонентах
- [[react/patterns]] - для изучения продвинутых паттернов
- [[react/performance]] - для оптимизации производительности компонентов
- [[react/testing]] - для тестирования компонентов