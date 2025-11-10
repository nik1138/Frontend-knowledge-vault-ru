---
aliases: ["React архитектурные паттерны", "React шаблоны архитектуры"]
tags: 
  - "#react"
  - "#architecture"
  - "#patterns"
  - "#frontend"
---

# Паттерны архитектуры React

## Обзор

Паттерны архитектуры React помогают организовать код приложения, сделать его более предсказуемым, тестируемым и поддерживаемым. В этой статье рассмотрим ключевые архитектурные паттерны, используемые в React-приложениях.

## Паттерны компонентов

### Container/Presentational (Контейнер/Представление)

Разделяет логику и представление компонентов:

```jsx
// Presentational Component
const UserList = ({ users, loading }) => (
  <div>
    {loading ? <div>Loading...</div> : 
      users.map(user => <div key={user.id}>{user.name}</div>)}
  </div>
);

// Container Component
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

[[React компоненты]]

### Higher-Order Components (HOC)

Функции, которые принимают компонент и возвращают новый компонент с дополнительной функциональностью:

```jsx
const withAuth = (WrappedComponent) => {
  return (props) => {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    
    useEffect(() => {
      checkAuth().then(setIsAuthenticated);
    }, []);
    
    if (!isAuthenticated) return <div>Access Denied</div>;
    
    return <WrappedComponent {...props} />;
  };
};

const ProtectedComponent = withAuth(MyComponent);
```

### Render Props

Паттерн, позволяющий делиться кодом между компонентами с помощью функции в props:

```jsx
class MouseTracker extends Component {
  state = { x: 0, y: 0 };
  
  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };
  
  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

<MouseTracker render={({ x, y }) => (
  <div>Position: {x}, {y}</div>
)} />
```

### Compound Components

Компоненты, которые работают вместе, разделяя неявное состояние:

```jsx
const Select = ({ children, value, onChange }) => (
  <div className="select">
    {Children.map(children, child =>
      cloneElement(child, { value, onChange })
    )}
  </div>
);

Select.Option = ({ value, children, onChange, selectedValue }) => (
  <div 
    onClick={() => onChange(value)}
    className={value === selectedValue ? 'selected' : ''}
  >
    {children}
  </div>
);

// Использование
<Select value={selected} onChange={setSelected}>
  <Select.Option value="1">Option 1</Select.Option>
  <Select.Option value="2">Option 2</Select.Option>
</Select>
```

### State Reducers

Позволяет внешнему коду контролировать внутреннее состояние компонента:

```jsx
const stateReducer = (state, action) => {
  switch (action.type) {
    case 'INPUT_CHANGE':
      return { ...state, inputValue: action.value };
    default:
      return state;
  }
};

const Input = ({ stateReducer }) => {
  const [state, dispatch] = useReducer(
    stateReducer || ((s, a) => s),
    { inputValue: '' }
  );
  
  return (
    <input
      value={state.inputValue}
      onChange={(e) => dispatch({
        type: 'INPUT_CHANGE',
        value: e.target.value
      })}
    />
  );
};
```

## Паттерны управления состоянием

### Provider Pattern

Используется для передачи данных через дерево компонентов без пропсов:

```jsx
const ThemeContext = createContext();

const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Использование
const App = () => (
  <ThemeProvider>
    <Header />
    <Main />
  </ThemeProvider>
);
```

[[React Context API]]

### Custom Hooks Architecture

Инкапсулирует логику в переиспользуемых хуках:

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

## Архитектурные подходы

### Feature-Based Architecture

Организация по функциональным возможностям:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── index.js
│   └── profile/
│       ├── components/
│       ├── hooks/
│       └── services/
```

### Ducks Pattern

Компоненты, действия и редьюсеры в одном файле:

```javascript
// ducks/users.js
// Actions
const LOAD_USERS = 'users/LOAD_USERS';
const LOAD_USERS_SUCCESS = 'users/LOAD_USERS_SUCCESS';

// Reducer
export default (state = { items: [], loading: false }, action) => {
  switch (action.type) {
    case LOAD_USERS:
      return { ...state, loading: true };
    case LOAD_USERS_SUCCESS:
      return { ...state, loading: false, items: action.payload };
    default:
      return state;
  }
};

// Action creators
export const loadUsers = () => ({
  type: LOAD_USERS
});

export const loadUsersSuccess = (users) => ({
  type: LOAD_USERS_SUCCESS,
  payload: users
});
```

### Feature-Sliced Design

Архитектура, разделяющая приложение на слои и срезы:

```
src/
├── app/          # Входная точка
├── processes/    # Бизнес-процессы
├── pages/        # Страницы
├── widgets/      # Виджеты
├── features/     # Функции
├── entities/     # Сущности
└── shared/       # Общие компоненты
```

### Atomic Design Principles

Разбиение интерфейса на иерархию компонентов:

- **Atoms** - базовые элементы (кнопки, инпуты)
- **Molecules** - комбинации атомов
- **Organisms** - сложные компоненты
- **Templates** - структура страниц
- **Pages** - конкретные реализации шаблонов

## Архитектурные стратегии

### Presentation/Domain/Data Layers

Разделение ответственности:

- **Presentation Layer** - компоненты пользовательского интерфейса
- **Domain Layer** - бизнес-логика
- **Data Layer** - работа с API и базами данных

### Code Splitting Strategies

Разделение кода для оптимизации загрузки:

```jsx
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));

const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <Routes>
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/profile" element={<Profile />} />
    </Routes>
  </Suspense>
);
```

### Feature Toggles

Управление функциональностью через конфигурацию:

```jsx
const FeatureToggle = ({ featureName, children }) => {
  const isEnabled = useFeatureFlag(featureName);
  
  return isEnabled ? children : null;
};

// Использование
<FeatureToggle featureName="newCheckout">
  <NewCheckoutFlow />
</FeatureToggle>
```

### Plugin Architectures

Архитектура, позволяющая расширение через плагины:

```jsx
// Plugin interface
const PluginInterface = {
  register: (plugin) => {
    // Регистрация плагина
  },
  execute: (hook, data) => {
    // Выполнение плагинов
  }
};

// Пример плагина
const analyticsPlugin = {
  name: 'analytics',
  hooks: {
    onPageView: (page) => trackPageView(page)
  }
};
```

## Архитектурное тестирование

Проверка архитектурных решений:

- **Структурные тесты** - проверка компонентной структуры
- **Тесты зависимостей** - проверка связей между модулями
- **Тесты паттернов** - проверка соответствия архитектурным паттернам

## Связи с другими файлами

- [[React best practices]] - рекомендации по разработке
- [[React performance optimization]] - оптимизация производительности
- [[React state management]] - управление состоянием
- [[React testing strategies]] - стратегии тестирования
- [[Component architecture]] - архитектура компонентов

## Заключение

Выбор архитектурных паттернов зависит от размера приложения, команды и требований. Важно выбирать паттерны, которые улучшают читаемость, тестируемость и поддерживаемость кода.

> [!tip] 
> Начинайте с простых паттернов и постепенно добавляйте более сложные по мере роста приложения.

> [!warning]
> Избегайте преждевременной архитектурной сложности. Паттерны должны решать реальные проблемы, а не создавать новые.