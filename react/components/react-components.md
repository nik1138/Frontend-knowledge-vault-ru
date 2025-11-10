---
aliases: ["React компоненты", "Компоненты React", "React Component"]
tags: [react, components, frontend, javascript]
---

# Компоненты React: Полное руководство

## Что такое компоненты React

Компоненты React — это переиспользуемые части пользовательского интерфейса, которые позволяют разбивать сложные приложения на небольшие, управляемые элементы. Компоненты могут принимать входные данные (props) и возвращать элементы React, описывающие, что должно отображаться на экране.

Компоненты являются основой архитектуры React-приложений и позволяют создавать масштабируемые и поддерживаемые интерфейсы. [[component-architecture]] [[react]]

## Типы компонентов

### Функциональные компоненты

Функциональные компоненты — это JavaScript-функции, которые принимают props и возвращают элементы React:

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Современный функциональный компонент с хуками
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счет: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}
```

### Классовые компоненты

Классовые компоненты определяются как ES6-классы, расширяющие `React.Component`:

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  render() {
    return (
      <div>
        <p>Счет: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Увеличить
        </button>
      </div>
    );
  }
}
```

## Жизненный цикл компонентов

### Фаза монтирования (Mounting)

Это когда компонент создается и вставляется в DOM:

- `constructor()`: Инициализация состояния и привязка методов
- `render()`: Возврат элементов для отображения
- `componentDidMount()`: Выполнение после монтирования

### Фаза обновления (Updating)

Когда компонент перерисовывается из-за изменений props или state:

- `componentDidUpdate()`: Выполняется после обновления
- `getSnapshotBeforeUpdate()`: Возврат значения перед обновлением

### Фаза размонтирования (Unmounting)

Когда компонент удаляется из DOM:

- `componentWillUnmount()`: Очистка ресурсов, отписка от подписок

## Хуки жизненного цикла (для функциональных компонентов)

Функциональные компоненты используют хуки вместо методов жизненного цикла:

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  // Эквивалент componentDidMount
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  // Эквивалент componentDidUpdate
  useEffect(() => {
    document.title = user ? user.name : 'Загрузка...';
  }, [user]);

  // Эквивалент componentWillUnmount
  useEffect(() => {
    return () => {
      // Очистка ресурсов
      console.log('Компонент будет размонтирован');
    };
  }, []);

  return user ? <div>{user.name}</div> : <div>Загрузка...</div>;
}
```

## Обработка props

Props (свойства) передаются компонентам как атрибуты и неизменяемы:

```jsx
function Button({ onClick, disabled, children, variant = 'primary' }) {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
}

// Использование
<Button variant="success" onClick={handleClick}>
  Нажми меня
</Button>
```

Для проверки типов props используется PropTypes:

```jsx
import PropTypes from 'prop-types';

Button.propTypes = {
  onClick: PropTypes.func.isRequired,
  disabled: PropTypes.bool,
  children: PropTypes.node.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'success'])
};
```

## Управление состоянием

### Внутреннее состояние (useState)

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, { id: Date.now(), text: inputValue, completed: false }]);
      setInputValue('');
    }
  };

  return (
    <div>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Добавить</button>
      <ul>
        {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
      </ul>
    </div>
  );
}
```

### Контекст (Context API)

Для передачи данных через дерево компонентов без пропс-дрilling:

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <header className={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Переключить тему
      </button>
    </header>
  );
}
```

## Паттерны композиции компонентов

### Контейнеры и презентационные компоненты

Разделение логики и представления:

```jsx
// Презентационный компонент
function UserCard({ user, onEdit }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={onEdit}>Редактировать</button>
    </div>
  );
}

// Контейнерный компонент
function UserListContainer() {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);

  const handleEdit = (user) => {
    // Логика редактирования
  };

  return (
    <div>
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user} 
          onEdit={() => handleEdit(user)} 
        />
      ))}
    </div>
  );
}
```

### Высокоуровневые компоненты (HOC)

Функции, которые принимают компонент и возвращают новый компонент:

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    
    useEffect(() => {
      checkAuth().then(setIsAuthenticated);
    }, []);

    if (!isAuthenticated) {
      return <div>Требуется авторизация</div>;
    }

    return <WrappedComponent {...props} />;
  };
}

const ProtectedProfile = withAuth(Profile);
```

## Взаимодействие между компонентами

### Родитель → Потомок: Props

```jsx
function Parent() {
  const [message, setMessage] = useState('Привет из родителя');

  return <Child message={message} />;
}

function Child({ message }) {
  return <div>{message}</div>;
}
```

### Потомок → Родитель: Callback-функции

```jsx
function Parent() {
  const [childData, setChildData] = useState('');

  const handleDataFromChild = (data) => {
    setChildData(data);
  };

  return <Child onData={handleDataFromChild} />;
}

function Child({ onData }) {
  const [input, setInput] = useState('');

  const sendData = () => {
    onData(input);
  };

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={sendData}>Отправить данные</button>
    </div>
  );
}
```

### Соседние компоненты: Context или глобальное состояние

## Принципы проектирования компонентов

1. **Единственная ответственность**: Компонент должен решать одну задачу
2. **Предсказуемость**: Одинаковые props → одинаковый результат
3. **Переиспользуемость**: Компонент должен работать в разных контекстах
4. **Тестируемость**: Компонент должен быть легко тестируемым
5. **Масштабируемость**: Компонент должен легко расширяться

## Повторное использование компонентов

### Пользовательские хуки

Для повторного использования логики между компонентами:

```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
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

// Использование в компоненте
function TodoApp() {
  const [todos, setTodos] = useLocalStorage('todos', []);

  return (
    <div>
      {/* Рендеринг todo-элементов */}
    </div>
  );
}
```

## Библиотеки компонентов

Популярные библиотеки компонентов:
- [[react-bootstrap]]
- [[material-ui]]
- [[ant-design]]
- [[chakra-ui]]

## Архитектура компонентов

Компоненты организуются в иерархии, где:
- Корневой компонент управляет состоянием приложения
- Контейнерные компоненты управляют данными и логикой
- Презентационные компоненты отвечают за отображение

[[component-architecture]]

## Рассмотрения производительности

### Мемоизация компонентов

```jsx
const MemoizedComponent = memo(function Component({ value }) {
  return <div>{value}</div>;
});

// Или для функций рендера
const ExpensiveComponent = ({ items }) => {
  const expensiveValue = useMemo(() => 
    items.map(item => heavyCalculation(item)), 
    [items]
  );

  return <div>{expensiveValue}</div>;
};
```

### Оптимизация рендеринга

- Использование `React.memo()` для предотвращения ненужных рендеров
- Использование `useCallback()` для стабильных функций
- Использование `useMemo()` для дорогостоящих вычислений

## Тестирование компонентов

Для тестирования компонентов используются библиотеки:
- Jest
- React Testing Library
- Enzyme

```jsx
import { render, screen, fireEvent } from '@testing-library/react';

test('Кнопка увеличивает счетчик при клике', () => {
  render(<Counter />);
  
  const button = screen.getByText('Увеличить');
  const counterDisplay = screen.getByText('Счет: 0');
  
  fireEvent.click(button);
  
  expect(screen.getByText('Счет: 1')).toBeInTheDocument();
});
```

## Связи с другими файлами React

- [[react-props]] - Подробное руководство по работе с props
- [[react-state]] - Управление состоянием в React
- [[react-hooks]] - Использование хуков в функциональных компонентах
- [[react-context]] - Передача данных через дерево компонентов
- [[react-performance]] - Оптимизация производительности React-приложений
- [[react-testing]] - Тестирование React-компонентов
- [[react-architecture]] - Архитектурные шаблоны в React
- [[react-lifecycle]] - Жизненный цикл компонентов в деталях