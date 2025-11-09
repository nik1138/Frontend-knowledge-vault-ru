# Типизация компонентов React в TypeScript

Типизация компонентов React в TypeScript позволяет создавать строго типизированные и безопасные React-приложения. Эта секция охватывает все аспекты типизации различных видов компонентов.

## Содержание

1. [Function Components](#function-components)
2. [Class Components](#class-components)
3. [Generic Components](#generic-components)
4. [Forward Ref Components](#forward-ref-components)
5. [Higher-Order Components](#higher-order-components)
6. [Compound Components](#compound-components)
7. [Render Props](#render-props)
8. [Лучшие практики](#лучшие-практики)
9. [Распространенные ошибки](#распространенные-ошибки)

## Function Components

### Базовая типизация

```tsx
// Интерфейс для пропсов
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
}

// Function Component с явной типизацией
const Button: React.FC<ButtonProps> = ({ 
  children, 
  onClick, 
  variant = 'primary', 
  disabled = false 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

// Альтернативный способ без React.FC
const ButtonAlt = ({ 
  children, 
  onClick, 
  variant = 'primary', 
  disabled = false 
}: ButtonProps) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

### Пропсы с children

```tsx
// Различные способы типизации children

// 1. React.ReactNode (рекомендуется)
interface LayoutProps {
  children: React.ReactNode;
}

// 2. React.ReactElement (только один элемент)
interface CardProps {
  children: React.ReactElement;
}

// 3. React.ReactNode[] (массив элементов)
interface ListProps {
  children: React.ReactNode[];
}

// 4. Специфичные children
interface FormProps {
  children: React.ReactElement<HTMLFormElement> | React.ReactElement<HTMLInputElement>[];
}

// Использование
const Layout: React.FC<LayoutProps> = ({ children }) => {
  return (
    <div className="layout">
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
};
```

### Опциональные и обязательные пропсы

```tsx
interface UserCardProps {
  // Обязательные пропсы
  id: number;
  name: string;
  
  // Опциональные пропсы
  email?: string;
  avatar?: string;
  
  // Пропсы с значениями по умолчанию
  isActive?: boolean;
  role?: 'user' | 'admin' | 'moderator';
}

const UserCard: React.FC<UserCardProps> = ({ 
  id, 
  name, 
  email, 
  avatar, 
  isActive = true, 
  role = 'user' 
}) => {
  return (
    <div className={`user-card ${isActive ? 'active' : 'inactive'}`}>
      {avatar && <img src={avatar} alt={name} />}
      <h3>{name}</h3>
      {email && <p>{email}</p>}
      <span className={`role role-${role}`}>{role}</span>
    </div>
  );
};
```

### Пропсы с функциями

```tsx
// Пропсы с callback функциями
interface TodoItemProps {
  todo: {
    id: number;
    text: string;
    completed: boolean;
  };
  onToggle: (id: number) => void;
  onDelete: (id: number) => void;
  onUpdate: (id: number, text: string) => void;
}

const TodoItem: React.FC<TodoItemProps> = ({ 
  todo, 
  onToggle, 
  onDelete, 
  onUpdate 
}) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);

  const handleSave = () => {
    onUpdate(todo.id, editText);
    setIsEditing(false);
  };

  return (
    <div className={`todo-item ${todo.completed ? 'completed' : ''}`}>
      {isEditing ? (
        <>
          <input
            type="text"
            value={editText}
            onChange={e => setEditText(e.target.value)}
          />
          <button onClick={handleSave}>Save</button>
          <button onClick={() => setIsEditing(false)}>Cancel</button>
        </>
      ) : (
        <>
          <span onClick={() => onToggle(todo.id)}>{todo.text}</span>
          <button onClick={() => setIsEditing(true)}>Edit</button>
          <button onClick={() => onDelete(todo.id)}>Delete</button>
        </>
      )}
    </div>
  );
};
```

## Class Components

### Базовая типизация

```tsx
// Интерфейсы для пропсов и состояния
interface UserProfileProps {
  userId: number;
  onUserLoad?: (user: User) => void;
}

interface UserProfileState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

class UserProfile extends React.Component<UserProfileProps, UserProfileState> {
  constructor(props: UserProfileProps) {
    super(props);
    this.state = {
      user: null,
      loading: true,
      error: null
    };
  }

  async componentDidMount() {
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const user: User = await response.json();
      
      this.setState({ user, loading: false });
      
      if (this.props.onUserLoad) {
        this.props.onUserLoad(user);
      }
    } catch (error) {
      this.setState({
        error: error instanceof Error ? error.message : 'Failed to load user',
        loading: false
      });
    }
  }

  render() {
    const { user, loading, error } = this.state;

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    if (!user) return <div>User not found</div>;

    return (
      <div className="user-profile">
        {user.avatar && <img src={user.avatar} alt={user.name} />}
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    );
  }
}
```

### Методы жизненного цикла

```tsx
interface TimerProps {
  initialSeconds: number;
  onComplete?: () => void;
}

interface TimerState {
  seconds: number;
  isRunning: boolean;
}

class Timer extends React.Component<TimerProps, TimerState> {
  private intervalId: NodeJS.Timeout | null = null;

  constructor(props: TimerProps) {
    super(props);
    this.state = {
      seconds: props.initialSeconds,
      isRunning: false
    };
  }

  // Монтирование
  componentDidMount() {
    this.startTimer();
  }

  // Обновление
  componentDidUpdate(prevProps: TimerProps, prevState: TimerState) {
    // Логика при обновлении
    if (prevState.seconds !== this.state.seconds && this.state.seconds === 0) {
      this.stopTimer();
      if (this.props.onComplete) {
        this.props.onComplete();
      }
    }
  }

  // Размонтирование
  componentWillUnmount() {
    this.stopTimer();
  }

  startTimer = () => {
    if (!this.state.isRunning) {
      this.intervalId = setInterval(() => {
        this.setState(prevState => ({
          seconds: prevState.seconds > 0 ? prevState.seconds - 1 : 0,
          isRunning: prevState.seconds > 0
        }));
      }, 1000);
      
      this.setState({ isRunning: true });
    }
  };

  stopTimer = () => {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
      this.setState({ isRunning: false });
    }
  };

  resetTimer = () => {
    this.stopTimer();
    this.setState({ 
      seconds: this.props.initialSeconds,
      isRunning: false
    });
  };

  render() {
    return (
      <div className="timer">
        <div className="time">{this.state.seconds}s</div>
        <button onClick={this.startTimer} disabled={this.state.isRunning}>
          Start
        </button>
        <button onClick={this.stopTimer} disabled={!this.state.isRunning}>
          Stop
        </button>
        <button onClick={this.resetTimer}>Reset</button>
      </div>
    );
  }
}
```

### Статические методы и свойства

```tsx
interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

interface ModalState {
  isAnimating: boolean;
}

class Modal extends React.Component<ModalProps, ModalState> {
  // Статические свойства
  static defaultProps: Partial<ModalProps> = {
    title: 'Modal'
  };

  // Статические методы
  static getDerivedStateFromProps(props: ModalProps, state: ModalState) {
    if (props.isOpen && !state.isAnimating) {
      return { isAnimating: true };
    }
    return null;
  }

  constructor(props: ModalProps) {
    super(props);
    this.state = {
      isAnimating: false
    };
  }

  // Статический метод для создания экземпляра
  static createInstance(props: ModalProps): Modal {
    return new Modal(props);
  }

  componentDidUpdate(prevProps: ModalProps) {
    if (prevProps.isOpen && !this.props.isOpen) {
      setTimeout(() => {
        this.setState({ isAnimating: false });
      }, 300);
    }
  }

  render() {
    const { isOpen, onClose, title, children } = this.props;
    const { isAnimating } = this.state;

    if (!isOpen && !isAnimating) return null;

    return (
      <div className={`modal-overlay ${isOpen ? 'open' : 'closing'}`}>
        <div className="modal-content">
          <div className="modal-header">
            <h2>{title}</h2>
            <button onClick={onClose}>×</button>
          </div>
          <div className="modal-body">
            {children}
          </div>
        </div>
      </div>
    );
  }
}
```

## Generic Components

### Базовые generic компоненты

```tsx
// Generic компонент для списка
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor?: (item: T, index: number) => string | number;
}

function List<T>({ 
  items, 
  renderItem, 
  keyExtractor = (item, index) => index 
}: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item, index)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Использование с различными типами
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
  keyExtractor={(user) => user.id}
/>;

interface Product {
  id: string;
  title: string;
  price: number;
}

const products: Product[] = [
  { id: '1', title: 'Product 1', price: 100 },
  { id: '2', title: 'Product 2', price: 200 }
];

<List<Product>
  items={products}
  renderItem={(product) => (
    <div>
      <h3>{product.title}</h3>
      <p>${product.price}</p>
    </div>
  )}
  keyExtractor={(product) => product.id}
/>;
```

### Сложные generic компоненты

```tsx
// Generic компонент с ограничениями
interface DataTableProps<T extends Record<string, any>> {
  data: T[];
  columns: Array<{
    key: keyof T;
    title: string;
    render?: (value: T[keyof T], item: T) => React.ReactNode;
  }>;
  onRowClick?: (item: T) => void;
  className?: string;
}

function DataTable<T extends Record<string, any>>({
  data,
  columns,
  onRowClick,
  className = ''
}: DataTableProps<T>) {
  return (
    <table className={`data-table ${className}`}>
      <thead>
        <tr>
          {columns.map((column, index) => (
            <th key={index}>{column.title}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((item, rowIndex) => (
          <tr 
            key={rowIndex} 
            onClick={() => onRowClick?.(item)}
            className={onRowClick ? 'clickable' : ''}
          >
            {columns.map((column, colIndex) => (
              <td key={colIndex}>
                {column.render 
                  ? column.render(item[column.key], item)
                  : String(item[column.key])
                }
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Использование
interface Employee {
  id: number;
  name: string;
  department: string;
  salary: number;
  hireDate: string;
}

const employees: Employee[] = [
  { id: 1, name: 'John Doe', department: 'Engineering', salary: 75000, hireDate: '2020-01-15' },
  { id: 2, name: 'Jane Smith', department: 'Marketing', salary: 65000, hireDate: '2019-03-22' }
];

<DataTable<Employee>
  data={employees}
  columns={[
    { key: 'name', title: 'Name' },
    { key: 'department', title: 'Department' },
    { 
      key: 'salary', 
      title: 'Salary',
      render: (value) => `$${value.toLocaleString()}`
    },
    { 
      key: 'hireDate', 
      title: 'Hire Date',
      render: (value) => new Date(value).toLocaleDateString()
    }
  ]}
  onRowClick={(employee) => console.log('Clicked:', employee)}
/>;
```

### Generic компоненты с множественными параметрами

```tsx
// Generic компонент с двумя параметрами
interface FormFieldProps<T, K extends keyof T> {
  name: K;
  value: T[K];
  onChange: (name: K, value: T[K]) => void;
  label: string;
  type?: string;
}

function FormField<T, K extends keyof T>({
  name,
  value,
  onChange,
  label,
  type = 'text'
}: FormFieldProps<T, K>) {
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = type === 'number' 
      ? (parseFloat(e.target.value) as T[K])
      : (e.target.value as T[K]);
    
    onChange(name, newValue);
  };

  return (
    <div className="form-field">
      <label htmlFor={name.toString()}>{label}</label>
      <input
        id={name.toString()}
        type={type}
        value={value as string | number}
        onChange={handleChange}
      />
    </div>
  );
}

// Использование
interface UserForm {
  name: string;
  age: number;
  email: string;
}

const UserFormComponent: React.FC = () => {
  const [formData, setFormData] = useState<UserForm>({
    name: '',
    age: 0,
    email: ''
  });

  const handleChange = <K extends keyof UserForm>(
    name: K,
    value: UserForm[K]
  ) => {
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  return (
    <form>
      <FormField<UserForm, 'name'>
        name="name"
        value={formData.name}
        onChange={handleChange}
        label="Name"
      />
      
      <FormField<UserForm, 'age'>
        name="age"
        value={formData.age}
        onChange={handleChange}
        label="Age"
        type="number"
      />
      
      <FormField<UserForm, 'email'>
        name="email"
        value={formData.email}
        onChange={handleChange}
        label="Email"
        type="email"
      />
    </form>
  );
};
```

## Forward Ref Components

### Базовая типизация forwardRef

```tsx
// Интерфейс для ref
interface ButtonRef {
  focus: () => void;
  blur: () => void;
}

// Пропсы компонента
interface CustomButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  onClick?: () => void;
}

// Компонент с forwardRef
const CustomButton = forwardRef<ButtonRef, CustomButtonProps>((props, ref) => {
  const buttonRef = useRef<HTMLButtonElement>(null);

  // Реализация методов для ref
  useImperativeHandle(ref, () => ({
    focus: () => {
      buttonRef.current?.focus();
    },
    blur: () => {
      buttonRef.current?.blur();
    }
  }));

  return (
    <button
      ref={buttonRef}
      className={`btn btn-${props.variant || 'primary'}`}
      onClick={props.onClick}
    >
      {props.children}
    </button>
  );
});

// Использование
const ParentComponent: React.FC = () => {
  const buttonRef = useRef<ButtonRef>(null);

  const handleFocus = () => {
    buttonRef.current?.focus();
  };

  return (
    <div>
      <CustomButton ref={buttonRef} onClick={() => console.log('Clicked')}>
        Click me
      </CustomButton>
      <button onClick={handleFocus}>Focus custom button</button>
    </div>
  );
};
```

### Сложные forwardRef компоненты

```tsx
// Интерфейс для сложного ref
interface FormInputRef {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
  setValue: (value: string) => void;
}

// Пропсы компонента
interface FormInputProps {
  name: string;
  label: string;
  type?: string;
  placeholder?: string;
  initialValue?: string;
  onValueChange?: (name: string, value: string) => void;
}

// Компонент с forwardRef
const FormInput = forwardRef<FormInputRef, FormInputProps>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  const [value, setValue] = useState(props.initialValue || '');

  // Реализация методов для ref
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current?.focus();
    },
    clear: () => {
      setValue('');
      if (inputRef.current) {
        inputRef.current.value = '';
      }
    },
    getValue: () => {
      return value;
    },
    setValue: (newValue: string) => {
      setValue(newValue);
      if (inputRef.current) {
        inputRef.current.value = newValue;
      }
    }
  }));

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    setValue(newValue);
    props.onValueChange?.(props.name, newValue);
  };

  return (
    <div className="form-input">
      <label htmlFor={props.name}>{props.label}</label>
      <input
        ref={inputRef}
        id={props.name}
        type={props.type || 'text'}
        placeholder={props.placeholder}
        value={value}
        onChange={handleChange}
      />
    </div>
  );
});

// Использование
const FormComponent: React.FC = () => {
  const firstNameRef = useRef<FormInputRef>(null);
  const lastNameRef = useRef<FormInputRef>(null);

  const handleSubmit = () => {
    const firstName = firstNameRef.current?.getValue() || '';
    const lastName = lastNameRef.current?.getValue() || '';
    
    console.log('Form data:', { firstName, lastName });
  };

  const clearForm = () => {
    firstNameRef.current?.clear();
    lastNameRef.current?.clear();
  };

  return (
    <form>
      <FormInput
        ref={firstNameRef}
        name="firstName"
        label="First Name"
        placeholder="Enter first name"
      />
      
      <FormInput
        ref={lastNameRef}
        name="lastName"
        label="Last Name"
        placeholder="Enter last name"
      />
      
      <button type="button" onClick={handleSubmit}>Submit</button>
      <button type="button" onClick={clearForm}>Clear</button>
    </form>
  );
};
```

## Higher-Order Components

### Базовые HOC

```tsx
// Типы для HOC
interface WithLoadingProps {
  loading: boolean;
}

// HOC для добавления состояния загрузки
function withLoading<P extends WithLoadingProps>(
  WrappedComponent: React.ComponentType<P>
) {
  return function WithLoadingComponent(props: Omit<P, keyof WithLoadingProps>) {
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      // Симуляция загрузки данных
      const timer = setTimeout(() => {
        setLoading(false);
      }, 2000);

      return () => clearTimeout(timer);
    }, []);

    if (loading) {
      return <div>Loading...</div>;
    }

    return <WrappedComponent {...(props as P)} loading={loading} />;
  };
}

// Компонент для оборачивания
interface UserProfileProps extends WithLoadingProps {
  user: { name: string; email: string };
}

const UserProfile: React.FC<UserProfileProps> = ({ user, loading }) => {
  if (loading) return null;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

// Обернутый компонент
const UserProfileWithLoading = withLoading(UserProfile);

// Использование
const App: React.FC = () => {
  const user = { name: 'John Doe', email: 'john@example.com' };
  
  return <UserProfileWithLoading user={user} />;
};
```

### Сложные HOC

```tsx
// Типы для сложного HOC
interface WithAuthProps {
  isAuthenticated: boolean;
  user: User | null;
}

interface User {
  id: number;
  name: string;
  role: string;
}

// HOC для аутентификации
function withAuth<P extends WithAuthProps>(
  WrappedComponent: React.ComponentType<P>,
  requiredRole?: string
) {
  return function WithAuthComponent(props: Omit<P, keyof WithAuthProps>) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      // Проверка аутентификации
      checkAuth().then(authData => {
        setIsAuthenticated(authData.isAuthenticated);
        setUser(authData.user);
        setLoading(false);
      });
    }, []);

    if (loading) {
      return <div>Checking authentication...</div>;
    }

    if (!isAuthenticated) {
      return <div>Please log in to view this content</div>;
    }

    if (requiredRole && user && user.role !== requiredRole) {
      return <div>Access denied. Insufficient permissions.</div>;
    }

    return (
      <WrappedComponent 
        {...(props as P)} 
        isAuthenticated={isAuthenticated} 
        user={user} 
      />
    );
  };
}

// Асинхронная функция проверки аутентификации
async function checkAuth(): Promise<{ isAuthenticated: boolean; user: User | null }> {
  // Симуляция API вызова
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        isAuthenticated: true,
        user: { id: 1, name: 'Admin User', role: 'admin' }
      });
    }, 1000);
  });
}

// Компонент для оборачивания
interface AdminPanelProps extends WithAuthProps {
  onUserAction: (action: string) => void;
}

const AdminPanel: React.FC<AdminPanelProps> = ({ user, isAuthenticated, onUserAction }) => {
  if (!isAuthenticated || !user) return null;
  
  return (
    <div>
      <h1>Admin Panel</h1>
      <p>Welcome, {user.name}!</p>
      <button onClick={() => onUserAction('delete')}>Delete User</button>
      <button onClick={() => onUserAction('ban')}>Ban User</button>
    </div>
  );
};

// Обернутый компонент с требованием роли
const AdminPanelWithAuth = withAuth(AdminPanel, 'admin');

// Использование
const App: React.FC = () => {
  const handleUserAction = (action: string) => {
    console.log('User action:', action);
  };
  
  return <AdminPanelWithAuth onUserAction={handleUserAction} />;
};
```

## Compound Components

### Базовые compound компоненты

```tsx
// Интерфейсы для compound компонентов
interface TabsProps {
  children: React.ReactNode;
  defaultActiveTab?: string;
}

interface TabProps {
  id: string;
  label: string;
  children: React.ReactNode;
}

interface TabContextType {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

// Контекст для compound компонента
const TabContext = createContext<TabContextType | undefined>(undefined);

// Основной компонент
const Tabs: React.FC<TabsProps> & {
  Tab: React.FC<TabProps>;
} = ({ children, defaultActiveTab = '' }) => {
  const [activeTab, setActiveTab] = useState(defaultActiveTab);

  return (
    <TabContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">
        {children}
      </div>
    </TabContext.Provider>
  );
};

// Дочерний компонент Tab
const Tab: React.FC<TabProps> = ({ id, label, children }) => {
  const context = useContext(TabContext);
  
  if (!context) {
    throw new Error('Tab must be used within Tabs');
  }
  
  const { activeTab, setActiveTab } = context;

  return (
    <div className="tab">
      <button
        className={activeTab === id ? 'active' : ''}
        onClick={() => setActiveTab(id)}
      >
        {label}
      </button>
      {activeTab === id && <div className="tab-content">{children}</div>}
    </div>
  );
};

// Привязка дочернего компонента
Tabs.Tab = Tab;

// Использование
const App: React.FC = () => {
  return (
    <Tabs defaultActiveTab="profile">
      <Tabs.Tab id="profile" label="Profile">
        <p>Profile content</p>
      </Tabs.Tab>
      <Tabs.Tab id="settings" label="Settings">
        <p>Settings content</p>
      </Tabs.Tab>
      <Tabs.Tab id="help" label="Help">
        <p>Help content</p>
      </Tabs.Tab>
    </Tabs>
  );
};
```

### Сложные compound компоненты

```tsx
// Интерфейсы для сложного compound компонента
interface FormProps {
  children: React.ReactNode;
  onSubmit: (values: Record<string, any>) => void;
}

interface FormFieldProps {
  name: string;
  children: React.ReactNode;
}

interface FormContextType {
  values: Record<string, any>;
  errors: Record<string, string>;
  setValue: (name: string, value: any) => void;
  setError: (name: string, error: string) => void;
  validateField: (name: string, value: any) => string | null;
}

// Контекст для формы
const FormContext = createContext<FormContextType | undefined>(undefined);

// Основной компонент формы
const Form: React.FC<FormProps> & {
  Field: React.FC<FormFieldProps>;
  Input: typeof FormInputComponent;
  Select: typeof FormSelectComponent;
  SubmitButton: typeof FormSubmitButton;
} = ({ children, onSubmit }) => {
  const [values, setValues] = useState<Record<string, any>>({});
  const [errors, setErrors] = useState<Record<string, string>>({});

  const setValue = (name: string, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
    // Очищаем ошибку при изменении значения
    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  };

  const setError = (name: string, error: string) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  };

  const validateField = (name: string, value: any): string | null => {
    // Базовая валидация
    if (value === '' || value === null || value === undefined) {
      return 'This field is required';
    }
    return null;
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // Валидация всех полей
    let hasErrors = false;
    const newErrors: Record<string, string> = {};
    
    Object.keys(values).forEach(name => {
      const error = validateField(name, values[name]);
      if (error) {
        newErrors[name] = error;
        hasErrors = true;
      }
    });
    
    if (hasErrors) {
      setErrors(newErrors);
      return;
    }
    
    onSubmit(values);
  };

  return (
    <FormContext.Provider value={{ 
      values, 
      errors, 
      setValue, 
      setError, 
      validateField 
    }}>
      <form onSubmit={handleSubmit}>
        {children}
      </form>
    </FormContext.Provider>
  );
};

// Компонент поля формы
const FormField: React.FC<FormFieldProps> = ({ name, children }) => {
  const context = useContext(FormContext);
  
  if (!context) {
    throw new Error('FormField must be used within Form');
  }
  
  const { errors } = context;
  const error = errors[name];

  return (
    <div className={`form-field ${error ? 'error' : ''}`}>
      {children}
      {error && <span className="error-message">{error}</span>}
    </div>
  );
};

// Компонент input
const FormInputComponent: React.FC<{
  name: string;
  type?: string;
  placeholder?: string;
}> = ({ name, type = 'text', placeholder }) => {
  const context = useContext(FormContext);
  
  if (!context) {
    throw new Error('FormInput must be used within Form');
  }
  
  const { values, setValue } = context;
  const value = values[name] || '';

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(name, e.target.value);
  };

  return (
    <input
      type={type}
      name={name}
      value={value}
      onChange={handleChange}
      placeholder={placeholder}
    />
  );
};

// Компонент select
const FormSelectComponent: React.FC<{
  name: string;
  options: Array<{ value: string; label: string }>;
}> = ({ name, options }) => {
  const context = useContext(FormContext);
  
  if (!context) {
    throw new Error('FormSelect must be used within Form');
  }
  
  const { values, setValue } = context;
  const value = values[name] || '';

  const handleChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setValue(name, e.target.value);
  };

  return (
    <select name={name} value={value} onChange={handleChange}>
      {options.map(option => (
        <option key={option.value} value={option.value}>
          {option.label}
        </option>
      ))}
    </select>
  );
};

// Компонент кнопки отправки
const FormSubmitButton: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <button type="submit">{children}</button>;
};

// Привязка компонентов
Form.Field = FormField;
Form.Input = FormInputComponent;
Form.Select = FormSelectComponent;
Form.SubmitButton = FormSubmitButton;

// Использование
const App: React.FC = () => {
  const handleSubmit = (values: Record<string, any>) => {
    console.log('Form submitted:', values);
  };

  return (
    <Form onSubmit={handleSubmit}>
      <Form.Field name="name">
        <Form.Input name="name" placeholder="Enter your name" />
      </Form.Field>
      
      <Form.Field name="email">
        <Form.Input name="email" type="email" placeholder="Enter your email" />
      </Form.Field>
      
      <Form.Field name="country">
        <Form.Select 
          name="country" 
          options={[
            { value: 'us', label: 'United States' },
            { value: 'ca', label: 'Canada' },
            { value: 'uk', label: 'United Kingdom' }
          ]} 
        />
      </Form.Field>
      
      <Form.SubmitButton>Submit</Form.SubmitButton>
    </Form>
  );
};
```

## Render Props

### Базовые render props

```tsx
// Интерфейсы для render props
interface ToggleProps {
  children: (state: {
    isToggled: boolean;
    toggle: () => void;
  }) => React.ReactNode;
  defaultToggled?: boolean;
}

// Компонент с render props
const Toggle: React.FC<ToggleProps> = ({ 
  children, 
  defaultToggled = false 
}) => {
  const [isToggled, setIsToggled] = useState(defaultToggled);

  const toggle = useCallback(() => {
    setIsToggled(prev => !prev);
  }, []);

  return <>{children({ isToggled, toggle })}</>;
};

// Использование
const App: React.FC = () => {
  return (
    <div>
      <Toggle defaultToggled={true}>
        {({ isToggled, toggle }) => (
          <div>
            <p>Toggle is {isToggled ? 'ON' : 'OFF'}</p>
            <button onClick={toggle}>
              {isToggled ? 'Turn Off' : 'Turn On'}
            </button>
          </div>
        )}
      </Toggle>
    </div>
  );
};
```

### Сложные render props

```tsx
// Интерфейсы для сложных render props
interface DataFetcherProps<T> {
  url: string;
  children: (state: {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => Promise<void>;
  }) => React.ReactNode;
}

// Компонент с render props для загрузки данных
const DataFetcher = <T,>({ url, children }: DataFetcherProps<T>) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result: T = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return <>{children({ data, loading, error, refetch: fetchData })}</>;
};

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const App: React.FC = () => {
  return (
    <div>
      <DataFetcher<User[]> url="/api/users">
        {({ data, loading, error, refetch }) => {
          if (loading) return <div>Loading users...</div>;
          if (error) return <div>Error: {error}</div>;
          if (!data) return <div>No users found</div>;

          return (
            <div>
              <button onClick={() => refetch()}>Refresh</button>
              <ul>
                {data.map(user => (
                  <li key={user.id}>
                    {user.name} - {user.email}
                  </li>
                ))}
              </ul>
            </div>
          );
        }}
      </DataFetcher>
    </div>
  );
};
```

## Лучшие практики

### 1. Явная типизация пропсов

```tsx
// Хорошо: явная типизация
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ children, onClick, variant = 'primary', disabled = false }) => {
  // Реализация
};

// Плохо: неявная типизация
const Button = ({ children, onClick, variant = 'primary', disabled = false }: any) => {
  // Реализация
};
```

### 2. Использование utility types

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

// Partial для форм
interface UserFormValues extends Partial<User> {
  name: string; // Обязательное поле
  email: string; // Обязательное поле
}

// Omit для исключения полей
interface UserPublic extends Omit<User, 'password'> {}

// Pick для выбора полей
interface UserBasic extends Pick<User, 'id' | 'name'> {}

// Record для объектов с динамическими ключами
interface UserPreferences extends Record<string, boolean> {}
```

### 3. Типизация children

```tsx
// Для компонентов, принимающих любые children
interface LayoutProps {
  children: React.ReactNode;
}

// Для компонентов с определенными children
interface CardProps {
  title: string;
  children: React.ReactElement; // Только один элемент
}

// Для компонентов с массивом children
interface ListProps {
  items: React.ReactNode[];
}
```

### 4. Типизация событий

```tsx
// Типизация событий мыши
interface ButtonProps {
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

// Типизация событий клавиатуры
interface InputProps {
  onKeyPress: (event: React.KeyboardEvent<HTMLInputElement>) => void;
}

// Типизация событий формы
interface FormProps {
  onSubmit: (event: React.FormEvent<HTMLFormElement>) => void;
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
const handleChange = (e) => {  // e: any
  console.log(e.target.value);  // Нет автозаполнения
};

// Исправление: правильная типизация
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);  // Полная автозаполнение и проверка типов
};
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

### 5. Проблемы с forwardRef

```tsx
// Проблема: неправильная типизация ref
const Button = forwardRef((props, ref) => {  // ref: any
  return <button ref={ref}>Click me</button>;
});

// Исправление: правильная типизация ref
interface ButtonRef {
  focus: () => void;
}

const Button = forwardRef<ButtonRef, ButtonProps>((props, ref) => {
  const buttonRef = useRef<HTMLButtonElement>(null);
  
  useImperativeHandle(ref, () => ({
    focus: () => buttonRef.current?.focus()
  }));
  
  return <button ref={buttonRef}>Click me</button>;
});
```

## Связи с другими концепциями

- [[ts/react/React_с_TypeScript|TypeScript с React]] - Основы использования TypeScript в React
- [[ts/react/hooks|Хуки]] - Типизация хуков в React
- [[ts/advanced/generics|Дженерики]] - Использование дженериков в компонентах
- [[ts/advanced/utility-types|Вспомогательные типы]] - Utility types для типизации компонентов
- [[ts/react/context|Контекст]] - Типизация контекста и провайдеров

## Рекомендации по изучению

1. Начните с базовой типизации Function Components
2. Освойте типизацию Class Components
3. Практикуйтесь в создании Generic Components
4. Изучите типизацию Forward Ref Components
5. Освойте Higher-Order Components с TypeScript
6. Практикуйтесь в создании Compound Components
7. Изучите паттерн Render Props с типизацией
8. Освойте лучшие практики типизации компонентов

## Следующие шаги

- [[ts/react/forms|Формы]] - Работа с формами в TypeScript
- [[ts/react/context|Контекст]] - Типизация контекста и провайдеров
- [[ts/react/testing|Тестирование]] - Тестирование React-компонентов с TypeScript
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация React-приложений с TypeScript