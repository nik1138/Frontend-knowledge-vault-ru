# React

React — это популярная JavaScript-библиотека для создания пользовательских интерфейсов, разработанная Facebook. React позволяет создавать сложные интерактивные веб-приложения с использованием компонентного подхода.

## Основные характеристики

### Компонентный подход
React основан на компонентах — переиспользуемых строительных блоках пользовательского интерфейса.

```jsx
// Функциональный компонент
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Компонент с состоянием
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### Virtual DOM
React использует Virtual DOM для оптимизации обновлений реального DOM, что повышает производительность.

```jsx
// React автоматически обновляет только измененные части
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

### Однонаправленный поток данных
React следует принципу однонаправленного потока данных, что делает приложения более предсказуемыми.

```jsx
// Данные передаются сверху вниз через props
function App() {
  const [user, setUser] = useState({ name: 'John' });
  
  return (
    <UserCard user={user} onUpdate={setUser} />
  );
}
```

## Основные концепции

### JSX
JSX — это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобный код в JavaScript.

```jsx
// JSX
const element = (
  <div className="container">
    <h1>Hello, world!</h1>
    <p>This is JSX syntax</p>
  </div>
);

// То же самое в чистом JavaScript
const element = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Hello, world!'),
  React.createElement('p', null, 'This is JSX syntax')
);
```

### Хуки
Хуки — это функции, которые позволяют использовать состояние и другие возможности React без написания классов.

```jsx
// useState - хук состояния
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// useEffect - хук эффекта
function DataFetcher() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []); // Пустой массив зависимостей - выполнить один раз
  
  return <div>{data ? data.content : 'Loading...'}</div>;
}
```

### Контекст
Контекст предоставляет способ передачи данных через дерево компонентов без необходимости передавать props на каждом уровне.

```jsx
// Создание контекста
const ThemeContext = createContext('light');

// Провайдер контекста
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

// Использование контекста
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button 
      className={theme}
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      Toggle Theme
    </button>
  );
}
```

## Преимущества React

1. **Компонентная архитектура** — Повторное использование кода и лучшая организация
2. **Высокая производительность** — Virtual DOM и оптимизации рендеринга
3. **Богатая экосистема** — Множество библиотек и инструментов
4. **Сильное сообщество** — Активная поддержка и обновления
5. **Гибкость** — Может использоваться как для простых, так и для сложных приложений

## Связанные концепции

- [[Композиция]] - React поощряет композицию компонентов вместо наследования
- [[Разделение ответственности]] - React компоненты способствуют разделению ответственности
- [[Чистый код]] - React поощряет написание чистого, деклартивного кода
- [[DRY (Don't Repeat Yourself)]] - Компоненты React способствуют избеганию дублирования
- [[KISS (Keep It Simple, Stupid)]] - React поощряет простоту через компонентный подход
- [[Интерфейсы]] - React компоненты определяют интерфейсы через props
- [[Дженерики]] - React активно использует дженерики в TypeScript версиях
- [[Полиморфизм]] - React использует полиморфизм через компоненты

## В других технологиях

### В [[ts]]
React с TypeScript обеспечивает типобезопасность компонентов:

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onEdit(user)}>Edit</button>
    </div>
  );
};
```

### В [[js]]
React может использоваться с чистым JavaScript:

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input }]);
      setInput('');
    }
  };
  
  return (
    <div>
      <input 
        value={input} 
        onChange={e => setInput(e.target.value)} 
        onKeyPress={e => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

### В [[html]]
React может быть интегрирован в существующие HTML-страницы:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
    <div id="root"></div>
    
    <script type="text/babel">
        function App() {
            return <h1>Hello from React!</h1>;
        }
        
        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
```

## Теги
#react #javascript #frontend #components #ui-library