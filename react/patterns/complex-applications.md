---
aliases: ["React паттерны для сложных приложений", "Комплексные React паттерны"]
tags: [react, patterns, architecture, frontend]
---

# Паттерны React для сложных приложений

## Введение

Разработка сложных React-приложений требует применения специфических паттернов, которые помогают управлять состоянием, структурировать код и обеспечивать масштабируемость. В этой статье рассмотрим ключевые паттерны, используемые при создании крупномасштабных React-приложений.

## Основные паттерны компонентов

### Container/Presentational паттерн

Паттерн Container/Presentational разделяет компоненты на два типа:

- **Container** - отвечает за логику получения данных и состояние
- **Presentational** - отвечает за визуальное представление

```jsx
// Presentational компонент
const UserList = ({ users, loading }) => {
  if (loading) return <div>Загрузка...</div>;
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};

// Container компонент
const UserListContainer = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then(data => {
      setUsers(data);
      setLoading(false);
    });
  }, []);

  return <UserList users={users} loading={loading} />;
};
```

### Higher-Order Components (HOC)

HOC - это функции, которые принимают компонент и возвращают новый компонент с дополнительной функциональностью.

```jsx
const withAuth = (WrappedComponent) => {
  return (props) => {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    
    useEffect(() => {
      checkAuth().then(auth => setIsAuthenticated(auth));
    }, []);

    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }

    return <WrappedComponent {...props} />;
  };
};

const ProtectedComponent = withAuth(MyComponent);
```

### Render Props

Паттерн Render Props позволяет делиться кодом между компонентами, используя проп, который является функцией.

```jsx
class DataFetcher extends React.Component {
  state = { data: null, loading: true };

  componentDidMount() {
    this.props.fetchData().then(data => {
      this.setState({ data, loading: false });
    });
  }

  render() {
    return this.props.render(this.state);
  }
}

const MyComponent = () => (
  <DataFetcher
    fetchData={() => fetch('/api/data')}
    render={({ data, loading }) => {
      if (loading) return <div>Загрузка...</div>;
      return <div>{data && data.title}</div>;
    }}
  />
);
```

### Compound Components

Compound Components позволяют компонентам взаимодействовать друг с другом без передачи данных через пропсы.

```jsx
const Tabs = ({ children, activeTab, onTabChange }) => (
  <div className="tabs">
    {React.Children.map(children, child =>
      React.cloneElement(child, { activeTab, onTabChange })
    )}
  </div>
);

Tabs.TabList = ({ children }) => <div className="tab-list">{children}</div>;
Tabs.Tab = ({ id, title, activeTab, onTabChange }) => (
  <button
    className={id === activeTab ? 'active' : ''}
    onClick={() => onTabChange(id)}
  >
    {title}
  </button>
);
Tabs.TabPanels = ({ children }) => <div className="tab-panels">{children}</div>;
Tabs.TabPanel = ({ id, activeTab, children }) => (
  <div className={id === activeTab ? 'active' : 'hidden'}>
    {children}
  </div>
);
```

## Паттерны управления состоянием

### State Reducers

Паттерн State Reducers позволяет внешним потребителям переопределять поведение внутреннего состояния компонента.

```jsx
const stateReducer = (state, action) => {
  switch (action.type) {
    case 'INPUT_CHANGE':
      return { ...state, value: action.value };
    case 'RESET':
      return { ...state, value: '' };
    default:
      return state;
  }
};

const InputWithReducer = ({ initialState = {}, reducer = stateReducer }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <input
      value={state.value}
      onChange={(e) => dispatch({ type: 'INPUT_CHANGE', value: e.target.value })}
    />
  );
};
```

### Provider Pattern

Provider Pattern используется для предоставления данных и функций всем компонентам в дереве.

```jsx
const AppContext = createContext();

export const AppProvider = ({ children, initialState }) => {
  const [state, setState] = useState(initialState);
  
  const updateUser = (user) => {
    setState(prev => ({ ...prev, user }));
  };

  return (
    <AppContext.Provider value={{ state, updateUser }}>
      {children}
    </AppContext.Provider>
  );
};

export const useAppContext = () => useContext(AppContext);
```

### Custom Hooks

Custom Hooks позволяют извлекать логику компонентов в повторно используемые функции.

```jsx
const useApi = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
};
```

## Паттерны организации кода

### Component Composition

Композиция компонентов позволяет создавать более гибкие и переиспользуемые интерфейсы.

```jsx
const Layout = ({ sidebar, content, footer }) => (
  <div className="layout">
    <aside>{sidebar}</aside>
    <main>{content}</main>
    <footer>{footer}</footer>
  </div>
);

const App = () => (
  <Layout
    sidebar={<Sidebar />}
    content={<Content />}
    footer={<Footer />}
  />
);
```

### Controlled/Uncontrolled Components

- **Controlled** - состояние компонента управляется React
- **Uncontrolled** - состояние компонента управляется DOM

```jsx
// Controlled
const ControlledInput = () => {
  const [value, setValue] = useState('');
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Uncontrolled
const UncontrolledInput = () => {
  const inputRef = useRef();
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  return <input ref={inputRef} />;
};
```

### Lifting State Up

Поднятие состояния используется для синхронизации данных между компонентами.

```jsx
const ParentComponent = () => {
  const [sharedValue, setSharedValue] = useState('');
  
  return (
    <>
      <ChildA value={sharedValue} onChange={setSharedValue} />
      <ChildB value={sharedValue} />
    </>
  );
};
```

## Решения проблем с пропсами

### Prop Drilling Solutions

Для решения проблемы передачи пропсов через много промежуточных компонентов можно использовать:

- Context API
- Custom hooks
- State management libraries

## Паттерны использования Context

### Лучшие практики с Context

- Используйте несколько контекстов для разных частей состояния
- Избегайте помещения всего состояния в один контекст
- Комбинируйте с memo() для оптимизации ререндеров

```jsx
const UserContext = createContext();
const ThemeContext = createContext();

const App = () => (
  <UserContext.Provider value={user}>
    <ThemeContext.Provider value={theme}>
      <Main />
    </ThemeContext.Provider>
  </UserContext.Provider>
);
```

## Паттерны обработки ошибок и рендеринга

### Error Boundaries

Error Boundary - это компонент, который перехватывает JavaScript ошибки в дереве компонентов потомков.

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}
```

### Portals

Portals позволяют рендерить дочерние элементы в DOM-узел, который находится вне DOM-иерархии родительского компонента.

```jsx
const Modal = ({ children, isOpen, onClose }) => {
  if (!isOpen) return null;
  
  return ReactDOM.createPortal(
    <div className="modal">
      <div className="modal-content">
        {children}
        <button onClick={onClose}>Закрыть</button>
      </div>
    </div>,
    document.getElementById('modal-root')
  );
};
```

### Suspense Patterns

Suspense позволяет "ожидать" что-то во время рендеринга.

```jsx
const App = () => (
  <Suspense fallback={<div>Загрузка...</div>}>
    <ProfileDetails />
    <Suspense fallback={<div>Загрузка комментариев...</div>}>
      <Comments />
    </Suspense>
  </Suspense>
);
```

## Паттерны производительности

### Оптимизация производительности

- Используйте `React.memo()` для предотвращения ненужных ререндеров
- Применяйте `useCallback` и `useMemo` для мемоизации
- Используйте `useCallback` для колбэков, передаваемых в дочерние компоненты

```jsx
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  const memoizedValue = useMemo(() => computeExpensiveValue(data), [data]);
  
  const handleClick = useCallback(() => {
    onUpdate(memoizedValue);
  }, [onUpdate, memoizedValue]);

  return <div onClick={handleClick}>{memoizedValue}</div>;
});
```

## Архитектурные паттерны

### Clean Architecture в React

Clean Architecture в React включает в себя:

- Слои ответственности
- Независимость от фреймворков
- Независимость от UI

[[react/architecture/clean-architecture]]

### Модульная архитектура

Модульная архитектура позволяет организовать код по функциональным модулям.

```
src/
├── modules/
│   ├── user/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types/
│   └── product/
├── shared/
├── ui/
└── app/
```

### Feature-based организация

Организация по фичам группирует все, что относится к определенной функциональности.

[[react/architecture/feature-based-organization]]

## Заключение

Эти паттерны позволяют создавать масштабируемые и поддерживаемые React-приложения. Важно выбирать подходящие паттерны в зависимости от конкретных требований проекта и команды разработчиков.

Для более подробного изучения отдельных паттернов см.:

- [[react/patterns/state-management]]
- [[react/patterns/performance]]
- [[react/architecture/component-organization]]