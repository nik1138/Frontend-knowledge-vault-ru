---
aliases: ["Реакт паттерны", "React паттерны", "Продвинутые React паттерны"]
tags: [react, patterns, advanced, frontend]
---

# Продвинутые паттерны React

## Введение

Продвинутые паттерны React позволяют создавать более гибкие, переиспользуемые и масштабируемые компоненты. Эти паттерны решают сложные задачи управления состоянием, композиции компонентов и архитектуры приложений.

## Higher-Order Components (HOCs)

HOC (Higher-Order Component) - это функция, которая принимает компонент и возвращает новый компонент с дополнительной функциональностью.

```javascript
const withAuth = (WrappedComponent) => {
  return (props) => {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    
    useEffect(() => {
      // проверка аутентификации
    }, []);
    
    if (!isAuthenticated) return <div>Доступ запрещен</div>;
    
    return <WrappedComponent {...props} />;
  };
};

const ProtectedComponent = withAuth(MyComponent);
```

## Render Props

Паттерн Render Props позволяет делиться кодом между компонентами с помощью функции в пропсах.

```javascript
const DataProvider = ({ render }) => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  
  return render(data);
};

const MyComponent = () => (
  <DataProvider 
    render={(data) => <div>{data && data.content}</div>} 
  />
);
```

## Compound Components

Compound Components позволяют создавать связанные компоненты, которые взаимодействуют друг с другом через контекст.

```javascript
const Tabs = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) => 
        React.cloneElement(child, { activeTab, setActiveTab, index })
      )}
    </div>
  );
};

const TabList = ({ children, activeTab, setActiveTab, index }) => (
  <div className="tab-list">
    {React.Children.map(children, (child, i) => 
      React.cloneElement(child, { isActive: i === activeTab, onClick: () => setActiveTab(i) })
    )}
  </div>
);

const Tab = ({ isActive, onClick, children }) => (
  <button className={isActive ? 'active' : ''} onClick={onClick}>
    {children}
  </button>
);
```

## Provider Pattern

Provider Pattern использует [[Context API]] для передачи данных по дереву компонентов без пропсов.

```javascript
const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => useContext(ThemeContext);
```

## Контролируемые и неконтролируемые компоненты

[[Controlled vs Uncontrolled Components]] - важное понятие для работы с формами. Контролируемые компоненты получают свое значение из состояния, а неконтролируемые управляют своим состоянием.

```javascript
// Контролируемый
const ControlledInput = () => {
  const [value, setValue] = useState('');
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Неконтролируемый
const UncontrolledInput = () => {
  const inputRef = useRef();
  return <input ref={inputRef} />;
};
```

## Пользовательские хуки

[[Custom Hooks]] позволяют извлекать логику компонентов в переиспользуемые функции.

```javascript
const useApi = (url) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading };
};
```

## Паттерн Container/Presentational

Разделяет логику (Container) от представления (Presentational).

```javascript
// Container
const UserContainer = () => {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);
  
  return <UserList users={users} />;
};

// Presentational
const UserList = ({ users }) => (
  <ul>{users.map(user => <li key={user.id}>{user.name}</li>)}</ul>
);
```

## Hooks Flow

[[React Hooks Flow]] определяет порядок выполнения хуков и жизненный цикл компонента.

## Context Factories

Фабрики контекста позволяют создавать несколько экземпляров одного контекста.

```javascript
const createContextFactory = (initialValue) => {
  const Context = createContext(initialValue);
  
  const Provider = ({ children, value = initialValue }) => (
    <Context.Provider value={value}>{children}</Context.Provider>
  );
  
  return [Provider, () => useContext(Context)];
};
```

## Compound Components с состоянием

Комбинирование паттернов Compound Components и управления состоянием.

```javascript
const Accordion = ({ children }) => {
  const [activeIndex, setActiveIndex] = useState(null);
  
  return (
    <div className="accordion">
      {React.Children.map(children, (child, index) => 
        React.cloneElement(child, { 
          isActive: index === activeIndex,
          onClick: () => setActiveIndex(index === activeIndex ? null : index)
        })
      )}
    </div>
  );
};
```

## Решения проблемы Prop Drilling

Использование [[Context API]] или [[Custom Hooks]] для избежания передачи пропсов через несколько уровней.

## State Colocation

[[State Colocation]] - принцип размещения состояния ближе к компонентам, которые его используют.

## Lifting State Up

[[Lifting State Up]] - поднятие состояния к общему родительскому компоненту для синхронизации дочерних компонентов.

## Compound Components с слотами

Использование слотов для более гибкой композиции компонентов.

## Compound Components с Context

Сочетание Compound Components и Context для передачи данных между компонентами.

## Compound Components с Reducers

Использование `useReducer` для управления сложным состоянием в Compound Components.

```javascript
const tabsReducer = (state, action) => {
  switch(action.type) {
    case 'SET_ACTIVE':
      return { ...state, activeTab: action.index };
    default:
      return state;
  }
};
```

## Provider Pattern с несколькими контекстами

Комбинирование нескольких провайдеров для управления различными аспектами приложения.

```javascript
const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <LocaleProvider>
        {children}
      </LocaleProvider>
    </ThemeProvider>
  </AuthProvider>
);
```

## State Machines в React

Использование [[State Machines]] для управления сложными состояниями компонентов.

## Паттерны Suspense

[[Suspense Patterns]] позволяют отображать заглушку во время загрузки данных или компонентов.

## Паттерны Concurrent Mode

[[Concurrent Mode Patterns]] для создания отзывчивых интерфейсов.

## Паттерн Error Boundaries

[[Error Boundaries]] позволяют перехватывать ошибки в дереве компонентов.

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}
```

## Заключение

Продвинутые паттерны React позволяют создавать более гибкие и масштабируемые приложения. Выбор конкретного паттерна зависит от требований задачи и архитектуры приложения.

## См. также

- [[React Hooks]]
- [[Context API]]
- [[State Management]]
- [[React Performance]]

#react #patterns #frontend #javascript #advanced