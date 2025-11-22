# TypeScript с React

TypeScript значительно улучшает разработку React-приложений, обеспечивая строгую типизацию компонентов, хуков, контекста и других React-концепций. Эта секция охватывает все аспекты использования TypeScript в React-разработке.

## Содержание

1. [Основы TypeScript с React](#основы-typescript-с-react)
2. [Типизация компонентов](#типизация-компонентов)
3. [Типизация хуков](#типизация-хуков)
4. [Типизация контекста](#типизация-контекста)
5. [Типизация форм](#типизация-форм)
6. [Типизация событий](#типизация-событий)
7. [Практические примеры](#практические-примеры)
8. [Лучшие практики](#лучшие-практики)
9. [Распространенные ошибки](#распространенные-ошибки)

## Основы TypeScript с React

### Настройка проекта

```bash
# Создание нового проекта с TypeScript
npx create-react-app my-app --template typescript

# Или добавление TypeScript в существующий проект
npm install --save typescript @types/node @types/react @types/react-dom @types/jest
```

### Базовая конфигурация

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "es6"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": [
    "src"
  ]
}
```

### Преимущества TypeScript в React

```tsx
// Без TypeScript - потенциальные ошибки
function UserProfile({ user }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <img src={user.avatar} alt={user.name} />
    </div>
  );
}

// С TypeScript - строгая типизация
interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

interface UserProfileProps {
  user: User;
}

function UserProfile({ user }: UserProfileProps) {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      {user.avatar && <img src={user.avatar} alt={user.name} />}
    </div>
  );
}
```

## Типизация компонентов

### 1. Function Components

```tsx
// Базовая типизация
interface Props {
  name: string;
  age: number;
  isActive?: boolean;
}

const UserCard: React.FC<Props> = ({ name, age, isActive = false }) => {
  return (
    <div className={isActive ? 'active' : 'inactive'}>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
};

// Альтернативный способ
const UserCard = ({ name, age, isActive = false }: Props) => {
  return (
    <div className={isActive ? 'active' : 'inactive'}>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
};
```

### 2. Class Components

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

interface Props {
  initialUser: User;
}

interface State {
  user: User;
  loading: boolean;
}

class UserProfile extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      user: props.initialUser,
      loading: false
    };
  }

  updateUser = (newUser: User) => {
    this.setState({ user: newUser });
  };

  render() {
    const { user, loading } = this.state;
    return (
      <div>
        {loading ? (
          <p>Loading...</p>
        ) : (
          <>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
          </>
        )}
      </div>
    );
  }
}
```

### 3. Generic Components

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Использование
interface User {
  id: number;
  name: string;
}

const users: User[] = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

<List<User> 
  items={users} 
  renderItem={(user) => <span>{user.name}</span>} 
/>;
```

## Типизация хуков

### 1. useState

```tsx
// Простые типы
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');

// Объекты
interface User {
  id: number;
  name: string;
  email: string;
}

const [user, setUser] = useState<User | null>(null);

// Массивы
const [users, setUsers] = useState<User[]>([]);

// Сложные состояния
type FormState = {
  name: string;
  email: string;
  age: number;
};

const [form, setForm] = useState<FormState>({
  name: '',
  email: '',
  age: 0
});
```

### 2. useEffect

```tsx
// Базовое использование
useEffect(() => {
  document.title = 'My App';
}, []);

// С асинхронными операциями
useEffect(() => {
  const fetchUser = async () => {
    try {
      const response = await fetch('/api/user');
      const userData: User = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Failed to fetch user:', error);
    }
  };

  fetchUser();
}, []);
```

### 3. useContext

```tsx
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Хук для использования контекста
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 4. useReducer

```tsx
interface State {
  count: number;
  loading: boolean;
}

type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_LOADING'; payload: boolean };

const initialState: State = {
  count: 0,
  loading: false
};

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    default:
      return state;
  }
};

// Использование
const [state, dispatch] = useReducer(reducer, initialState);
```

### 5. Custom Hooks

```tsx
// Типизированный кастомный хук
interface ApiResponse<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useApi<T>(url: string): ApiResponse<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const result: T = await response.json();
        setData(result);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Unknown error');
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const UserList: React.FC = () => {
  const { data: users, loading, error } = useApi<User[]>('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!users) return <div>No users found</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name} - {user.email}</li>
      ))}
    </ul>
  );
};
```

## Типизация контекста

### 1. Базовый контекст

```tsx
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    // Логика аутентификации
    const userData: User = await authenticateUser(email, password);
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  const isAuthenticated = !!user;

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated }}>
      {children}
    </AuthContext.Provider>
  );
};

const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### 2. Контекст с асинхронными операциями

```tsx
interface DataContextType {
  data: Record<string, any>;
  loading: Record<string, boolean>;
  error: Record<string, string | null>;
  fetchData: (key: string, url: string) => Promise<void>;
}

const DataContext = createContext<DataContextType | undefined>(undefined);

export const DataProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [data, setData] = useState<Record<string, any>>({});
  const [loading, setLoading] = useState<Record<string, boolean>>({});
  const [error, setError] = useState<Record<string, string | null>>({});

  const fetchData = async (key: string, url: string) => {
    try {
      setLoading(prev => ({ ...prev, [key]: true }));
      setError(prev => ({ ...prev, [key]: null }));
      
      const response = await fetch(url);
      const result = await response.json();
      
      setData(prev => ({ ...prev, [key]: result }));
    } catch (err) {
      setError(prev => ({ 
        ...prev, 
        [key]: err instanceof Error ? err.message : 'Unknown error' 
      }));
    } finally {
      setLoading(prev => ({ ...prev, [key]: false }));
    }
  };

  return (
    <DataContext.Provider value={{ data, loading, error, fetchData }}>
      {children}
    </DataContext.Provider>
  );
};
```

## Типизация форм

### 1. Базовая типизация форм

```tsx
interface LoginFormValues {
  email: string;
  password: string;
  rememberMe: boolean;
}

const LoginForm: React.FC = () => {
  const [values, setValues] = useState<LoginFormValues>({
    email: '',
    password: '',
    rememberMe: false
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Логика отправки формы
    console.log(values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={values.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        type="password"
        name="password"
        value={values.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <label>
        <input
          type="checkbox"
          name="rememberMe"
          checked={values.rememberMe}
          onChange={handleChange}
        />
        Remember me
      </label>
      <button type="submit">Login</button>
    </form>
  );
};
```

### 2. Использование библиотек форм

```tsx
// С React Hook Form
import { useForm, SubmitHandler } from 'react-hook-form';

interface RegisterFormInputs {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age: number;
  terms: boolean;
}

const RegisterForm: React.FC = () => {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm<RegisterFormInputs>();

  const onSubmit: SubmitHandler<RegisterFormInputs> = data => {
    console.log(data);
  };

  const password = watch('password');

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register('username', { 
          required: 'Username is required',
          minLength: {
            value: 3,
            message: 'Username must be at least 3 characters'
          }
        })}
        placeholder="Username"
      />
      {errors.username && <span>{errors.username.message}</span>}

      <input
        {...register('email', {
          required: 'Email is required',
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: 'Invalid email address'
          }
        })}
        placeholder="Email"
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        {...register('password', {
          required: 'Password is required',
          minLength: {
            value: 8,
            message: 'Password must be at least 8 characters'
          }
        })}
        type="password"
        placeholder="Password"
      />
      {errors.password && <span>{errors.password.message}</span>}

      <input
        {...register('confirmPassword', {
          required: 'Please confirm password',
          validate: value => 
            value === password || 'Passwords do not match'
        })}
        type="password"
        placeholder="Confirm Password"
      />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <input
        {...register('age', {
          required: 'Age is required',
          min: {
            value: 18,
            message: 'You must be at least 18 years old'
          }
        })}
        type="number"
        placeholder="Age"
      />
      {errors.age && <span>{errors.age.message}</span>}

      <label>
        <input
          {...register('terms', {
            required: 'You must accept the terms and conditions'
          })}
          type="checkbox"
        />
        I accept the terms and conditions
      </label>
      {errors.terms && <span>{errors.terms.message}</span>}

      <button type="submit">Register</button>
    </form>
  );
};
```

## Типизация событий

### 1. События мыши

```tsx
interface ButtonProps {
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({ onClick, children }) => {
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    e.preventDefault();
    onClick(e);
  };

  return (
    <button onClick={handleClick}>
      {children}
    </button>
  );
};

// Использование
const MyComponent: React.FC = () => {
  const handleButtonClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Button clicked at:', e.clientX, e.clientY);
  };

  return <Button onClick={handleButtonClick}>Click me</Button>;
};
```

### 2. События клавиатуры

```tsx
interface SearchInputProps {
  onSearch: (query: string) => void;
}

const SearchInput: React.FC<SearchInputProps> = ({ onSearch }) => {
  const [query, setQuery] = useState<string>('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
  };

  const handleKeyPress = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      onSearch(query);
    }
  };

  return (
    <input
      type="text"
      value={query}
      onChange={handleChange}
      onKeyPress={handleKeyPress}
      placeholder="Search..."
    />
  );
};
```

### 3. События формы

```tsx
interface ContactFormValues {
  name: string;
  email: string;
  message: string;
}

const ContactForm: React.FC = () => {
  const [values, setValues] = useState<ContactFormValues>({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Логика отправки формы
    console.log(values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={values.name}
        onChange={handleChange}
        placeholder="Your name"
      />
      <input
        type="email"
        name="email"
        value={values.email}
        onChange={handleChange}
        placeholder="Your email"
      />
      <textarea
        name="message"
        value={values.message}
        onChange={handleChange}
        placeholder="Your message"
      />
      <button type="submit">Send</button>
    </form>
  );
};
```

## Практические примеры

### 1. Todo List Application

```tsx
interface Todo {
  id: number;
  text: string;
  completed: boolean;
  createdAt: Date;
}

interface TodoListState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

const TodoApp: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [inputValue, setInputValue] = useState<string>('');
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');

  const addTodo = () => {
    if (inputValue.trim()) {
      const newTodo: Todo = {
        id: Date.now(),
        text: inputValue.trim(),
        completed: false,
        createdAt: new Date()
      };
      setTodos(prev => [...prev, newTodo]);
      setInputValue('');
    }
  };

  const toggleTodo = (id: number) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  const deleteTodo = (id: number) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };

  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <h1>Todo List</h1>
      <div>
        <input
          type="text"
          value={inputValue}
          onChange={e => setInputValue(e.target.value)}
          onKeyPress={e => e.key === 'Enter' && addTodo()}
          placeholder="Add a new todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <div>
        <button onClick={() => setFilter('all')}>All</button>
        <button onClick={() => setFilter('active')}>Active</button>
        <button onClick={() => setFilter('completed')}>Completed</button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id} style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
            <span>{todo.text}</span>
            <button onClick={() => toggleTodo(todo.id)}>
              {todo.completed ? 'Undo' : 'Complete'}
            </button>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### 2. User Dashboard with API Integration

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'moderator';
  lastLogin: string;
}

interface DashboardState {
  users: User[];
  loading: boolean;
  error: string | null;
  searchTerm: string;
}

const UserDashboard: React.FC = () => {
  const [state, setState] = useState<DashboardState>({
    users: [],
    loading: false,
    error: null,
    searchTerm: ''
  });

  const fetchUsers = async () => {
    try {
      setState(prev => ({ ...prev, loading: true, error: null }));
      const response = await fetch('/api/users');
      const users: User[] = await response.json();
      setState(prev => ({ ...prev, users, loading: false }));
    } catch (error) {
      setState(prev => ({
        ...prev,
        loading: false,
        error: error instanceof Error ? error.message : 'Failed to fetch users'
      }));
    }
  };

  useEffect(() => {
    fetchUsers();
  }, []);

  const filteredUsers = state.users.filter(user =>
    user.name.toLowerCase().includes(state.searchTerm.toLowerCase()) ||
    user.email.toLowerCase().includes(state.searchTerm.toLowerCase())
  );

  if (state.loading) return <div>Loading users...</div>;
  if (state.error) return <div>Error: {state.error}</div>;

  return (
    <div>
      <h1>User Dashboard</h1>
      
      <div>
        <input
          type="text"
          value={state.searchTerm}
          onChange={e => setState(prev => ({ ...prev, searchTerm: e.target.value }))}
          placeholder="Search users..."
        />
        <button onClick={fetchUsers}>Refresh</button>
      </div>

      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Last Login</th>
          </tr>
        </thead>
        <tbody>
          {filteredUsers.map(user => (
            <tr key={user.id}>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user.role}</td>
              <td>{user.lastLogin}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

### 3. Modal Component with Generics

```tsx
interface ModalProps<T> {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
  data?: T;
  onConfirm?: (data: T) => void;
}

function Modal<T>({ 
  isOpen, 
  onClose, 
  title, 
  children, 
  data,
  onConfirm 
}: ModalProps<T>) {
  if (!isOpen) return null;

  const handleConfirm = () => {
    if (data && onConfirm) {
      onConfirm(data);
    }
    onClose();
  };

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose}>×</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
        <div className="modal-footer">
          <button onClick={onClose}>Cancel</button>
          {onConfirm && <button onClick={handleConfirm}>Confirm</button>}
        </div>
      </div>
    </div>
  );
}

// Использование
interface UserFormData {
  name: string;
  email: string;
}

const UserModal: React.FC = () => {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [formData, setFormData] = useState<UserFormData>({ 
    name: '', 
    email: '' 
  });

  const handleConfirm = (data: UserFormData) => {
    console.log('Creating user:', data);
    // Логика создания пользователя
  };

  return (
    <>
      <button onClick={() => setIsModalOpen(true)}>
        Add User
      </button>
      
      <Modal<UserFormData>
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        title="Add New User"
        data={formData}
        onConfirm={handleConfirm}
      >
        <input
          type="text"
          placeholder="Name"
          value={formData.name}
          onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
        />
        <input
          type="email"
          placeholder="Email"
          value={formData.email}
          onChange={e => setFormData(prev => ({ ...prev, email: e.target.value }))}
        />
      </Modal>
    </>
  );
};
```

## Лучшие практики

### 1. Использование Partial и Required

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
  isActive: boolean;
}

// Для форм, где не все поля обязательны
interface UserFormValues extends Partial<User> {
  name: string;
  email: string;
}

// Для API ответов, где все поля обязательны
interface UserApiResponse extends Required<User> {}

// Использование
const UserForm: React.FC = () => {
  const [values, setValues] = useState<UserFormValues>({
    name: '',
    email: ''
    // age и isActive не обязательны
  });
  
  // ...
};
```

### 2. Типизация children

```tsx
// Для компонентов, принимающих children
interface LayoutProps {
  children: React.ReactNode;
}

const Layout: React.FC<LayoutProps> = ({ children }) => {
  return (
    <div className="layout">
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
};

// Для компонентов с определенными children
interface CardProps {
  title: string;
  children: React.ReactElement; // Только один элемент
}

const Card: React.FC<CardProps> = ({ title, children }) => {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div>{children}</div>
    </div>
  );
};
```

### 3. Использование utility types

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

// Omit для исключения полей
interface UserPublic extends Omit<User, 'password'> {}

// Pick для выбора определенных полей
interface UserBasicInfo extends Pick<User, 'id' | 'name' | 'email'> {}

// Partial для опциональных полей
interface UserUpdate extends Partial<User> {
  id: number; // id остается обязательным
}

// Record для объектов с динамическими ключами
interface UserPreferences extends Record<string, boolean> {}

// Использование
const UserProfile: React.FC<{ user: UserPublic }> = ({ user }) => {
  // password недоступен
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

### 4. Типизация асинхронных операций

```tsx
// Интерфейс для API ответов
interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

// Интерфейс для ошибок
interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

// Типизированный хук для API
function useApi<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<ApiError | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url);
      const result: ApiResponse<T> = await response.json();
      
      if (response.ok) {
        setData(result.data);
      } else {
        setError({
          code: response.status.toString(),
          message: result.message || 'Unknown error'
        });
      }
    } catch (err) {
      setError({
        code: 'NETWORK_ERROR',
        message: err instanceof Error ? err.message : 'Network error'
      });
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}
```

## Распространенные ошибки

### 1. Неявная типизация any

```tsx
// Проблема: неявный any
function BadComponent({ data }) {  // data: any
  return <div>{data.name}</div>;  // Нет проверки типов
}

// Исправление: явная типизация
interface Data {
  name: string;
  value: number;
}

interface Props {
  data: Data;
}

function GoodComponent({ data }: Props) {
  return <div>{data.name}</div>;
}
```

### 2. Неправильная типизация событий

```tsx
// Проблема: неправильный тип события
function BadInput({ onChange }) {
  return <input onChange={onChange} />;  // onChange: any
}

// Исправление: правильная типизация
interface InputProps {
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
}

function GoodInput({ onChange }: InputProps) {
  return <input onChange={onChange} />;
}
```

### 3. Проблемы с useState

```tsx
// Проблема: неопределенное состояние
const [user, setUser] = useState();  // user: undefined

// Исправление: явная типизация
interface User {
  id: number;
  name: string;
}

const [user, setUser] = useState<User | null>(null);
```

### 4. Неправильная типизация контекста

```tsx
// Проблема: контекст без проверки
const ThemeContext = createContext();  // any

function BadComponent() {
  const theme = useContext(ThemeContext);  // any
  return <div className={theme} />;  // Нет проверки типов
}

// Исправление: типизированный контекст
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

function GoodComponent() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('Component must be used within ThemeProvider');
  }
  
  const { theme, toggleTheme } = context;
  return <div className={theme} />;
}
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка TypeScript для React
- [[ts/advanced/generics|Дженерики]] - Использование дженериков в React-компонентах
- [[ts/advanced/utility-types|Вспомогательные типы]] - Utility types для типизации React-приложений
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация React-приложений с TypeScript
- [[Тестирование в TypeScript|Тестирование]] - Тестирование React-компонентов с TypeScript

## Рекомендации по изучению

1. Начните с базовой типизации компонентов и хуков
2. Освойте типизацию контекста и пользовательских хуков
3. Практикуйтесь в типизации форм и событий
4. Изучите использование utility types в React
5. Освойте интеграцию с библиотеками форм
6. Практикуйтесь в создании типизированных кастомных хуков

## Следующие шаги

- [[Типизация хуков React в TypeScript|Хуки React]] - Подробное изучение типизации хуков
- [[Типизация компонентов React в TypeScript|Компоненты]] - Типизация различных видов компонентов
- [[Типизация форм в React с TypeScript|Формы]] - Работа с формами в TypeScript
- [[Типизация контекста в React с TypeScript|Контекст]] - Типизация контекста и провайдеров
- [[ts/react/testing|Тестирование]] - Тестирование React-компонентов с TypeScript