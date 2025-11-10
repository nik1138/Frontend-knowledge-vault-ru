---
aliases: ["Типы компонентов React", "React компоненты", "Компоненты в React"]
tags: ["#react", "#components", "#frontend", "#javascript", "#web-development"]
---

# Типы компонентов React

## Введение

React предоставляет различные типы компонентов, каждый из которых решает определенные задачи и подходит для разных сценариев разработки. Понимание этих типов помогает создавать более эффективные и поддерживаемые приложения.

## Функциональные компоненты

Функциональные компоненты — это простые JavaScript-функции, которые принимают props и возвращают JSX. Они стали предпочтительным способом создания компонентов после появления React Hooks.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Стрелочная функция
const Greeting = ({ name }) => {
  return <div>Здравствуйте, {name}!</div>;
};
```

Функциональные компоненты идеально подходят для простых компонентов, которые не требуют сложного состояния или методов жизненного цикла. Они легче читаются и тестируются. Подробнее о функциональных компонентах можно прочитать в [[react/components/components]].

## Классовые компоненты

Классовые компоненты используют синтаксис ES6 классов и могут содержать состояние и методы жизненного цикла.

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

Классовые компоненты имеют доступ к состоянию (`this.state`), методам жизненного цикла и ссылкам (`refs`). Однако с появлением React Hooks необходимость в классовых компонентах значительно снизилась. Подробнее о жизненном цикле компонентов см. [[react/components/lifecycle]].

## Чистые компоненты (Pure Components)

Чистые компоненты — это оптимизированные классовые компоненты, которые автоматически реализуют `shouldComponentUpdate`, выполняя поверхностное сравнение состояния и props.

```jsx
import React, { PureComponent } from 'react';

class PureCounter extends PureComponent {
  render() {
    return <div>Счетчик: {this.props.count}</div>;
  }
}
```

Использование чистых компонентов может улучшить производительность, избегая ненужных перерисовок. Подробнее о производительности см. [[react/performance/performance-optimization]].

## Компоненты с состоянием и без (Stateful vs Stateless)

### Компоненты с состоянием (Stateful)

Компоненты, которые управляют внутренним состоянием:

```jsx
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

### Компоненты без состояния (Stateless)

Компоненты, которые не управляют внутренним состоянием:

```jsx
function Display({ value }) {
  return <div>Значение: {value}</div>;
}
```

## Презентационные и контейнерные компоненты

### Презентационные компоненты

Фокусируются на визуальном отображении данных:

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### Контейнерные компоненты

Управляют данными и логикой:

```jsx
function UserContainer() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser().then(setUser);
  }, []);

  return user ? <UserCard user={user} /> : <div>Загрузка...</div>;
}
```

Этот паттерн помогает разделить логику и представление. Подробнее о паттернах см. [[react/components/component-patterns]].

## Контролируемые и неконтролируемые компоненты

### Контролируемые компоненты

Значение элемента формы контролируется React:

```jsx
function ControlledForm() {
  const [value, setValue] = useState('');

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

### Неконтролируемые компоненты

Значение элемента формы контролируется DOM:

```jsx
function UncontrolledForm() {
  const inputRef = useRef();

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="значение по умолчанию" />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

Подробнее о работе с формами см. [[react/forms/forms]].

## Компоненты высшего порядка (HOC)

HOC — это функция, которая принимает компонент и возвращает новый компонент с дополнительной функциональностью:

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);

    // Логика аутентификации
    useEffect(() => {
      checkAuth().then(setIsAuthenticated);
    }, []);

    if (!isAuthenticated) {
      return <div>Требуется аутентификация</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

const ProtectedComponent = withAuth(MyComponent);
```

HOC позволяют повторно использовать логику между компонентами. Подробнее см. [[react/components/hoc-patterns]].

## Компоненты с Render Props

Шаблон, при котором компонент использует функцию для рендеринга:

```jsx
class DataProvider extends React.Component {
  state = { data: null };

  render() {
    return this.props.render(this.state.data);
  }
}

function MyComponent() {
  return (
    <DataProvider
      render={(data) => (
        <div>Данные: {data && data.value}</div>
      )}
    />
  );
}
```

## Компаундные компоненты

Компоненты, которые работают вместе как единое целое:

```jsx
function Tabs({ children, activeTab, onTabChange }) {
  return (
    <div className="tabs">
      {React.Children.map(children, (child) =>
        React.cloneElement(child, { activeTab, onTabChange })
      )}
    </div>
  );
}

function Tab({ id, label, activeTab, onTabChange, children }) {
  return (
    <div
      className={`tab ${activeTab === id ? 'active' : ''}`}
      onClick={() => onTabChange(id)}
    >
      {label}
      {activeTab === id && <div className="tab-content">{children}</div>}
    </div>
  );
}

// Использование
function App() {
  return (
    <Tabs activeTab={activeTab} onTabChange={setActiveTab}>
      <Tab id="1" label="Вкладка 1">Содержимое 1</Tab>
      <Tab id="2" label="Вкладка 2">Содержимое 2</Tab>
    </Tabs>
  );
}
```

## Компоненты управления состоянием

Компоненты, специализирующиеся на управлении состоянием:

```jsx
// Простой компонент управления состоянием
function StateManager({ children }) {
  const [state, setState] = useState({});

  return (
    <StateContext.Provider value={{ state, setState }}>
      {children}
    </StateContext.Provider>
  );
}
```

Для сложного управления состоянием рекомендуется использовать внешние библиотеки, такие как Redux или Zustand. Подробнее см. [[react/state/state-management]].

## Пользовательские хуки как компоненты

Хотя хуки не являются компонентами по определению, они могут инкапсулировать логику, аналогичную компонентам:

```jsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return { count, increment, decrement };
}

// Использование в компоненте
function Counter() {
  const { count, increment, decrement } = useCounter(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}
```

Подробнее о хуках см. [[react/hooks/hooks-overview]].

## Паттерны компонентов

### Provider Pattern

```jsx
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### Compound Components Pattern

Паттерн, позволяющий компонентам эффективно взаимодействовать друг с другом, как описано выше в разделе компаундных компонентов.

### Render Props Pattern

Альтернатива HOC для повторного использования логики.

## Архитектура компонентов

Правильная архитектура компонентов включает:

1. **Разделение ответственности**: каждый компонент должен выполнять одну конкретную задачу
2. **Повторное использование**: создание переиспользуемых компонентов
3. **Управление состоянием**: четкое разделение между локальным и глобальным состоянием
4. **Тестирование**: компоненты должны быть легко тестируемыми

Для более глубокого понимания архитектуры см. [[react/architecture/component-architecture]].

## Взаимосвязи с другими файлами

- [[react/components/components]] — основы компонентов React
- [[react/components/lifecycle]] — жизненный цикл компонентов
- [[react/hooks/hooks-overview]] — использование хуков в компонентах
- [[react/state/state-management]] — управление состоянием
- [[react/forms/forms]] — работа с формами
- [[react/components/component-patterns]] — паттерны компонентов
- [[react/performance/performance-optimization]] — оптимизация производительности

## Заключение

Выбор подходящего типа компонента зависит от конкретных требований приложения. Функциональные компоненты с хуками являются современным стандартом, но другие типы компонентов также имеют свои области применения. Понимание различий между типами компонентов позволяет создавать более эффективные и поддерживаемые приложения на React.