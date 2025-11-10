---
aliases: ["React с TypeScript", "Типизация в React", "TSX"]
tags: 
  - #react
  - #typescript
  - #frontend
  - #programming
  - #best-practices
---

# React с TypeScript: Полное руководство

## Введение

TypeScript предоставляет строгую типизацию для React-приложений, что позволяет избежать многих ошибок на этапе разработки. Это руководство охватывает все аспекты типизации React-компонентов и приложений.

## Настройка TypeScript с React

Для начала работы с TypeScript в React-проекте необходимо установить необходимые зависимости:

```bash
npm install --save-dev typescript @types/react @types/react-dom
```

Файл `tsconfig.json` должен содержать следующую конфигурацию:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "es6"],
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

## Типизация функциональных компонентов

Функциональные компоненты можно типизировать несколькими способами:

```tsx
// С использованием React.FC (устаревший подход)
const MyComponent: React.FC<{name: string}> = ({name}) => {
  return <div>Hello, {name}!</div>;
};

// Рекомендуемый подход
const MyComponent = ({name}: {name: string}) => {
  return <div>Hello, {name}!</div>;
};

// С расширенной типизацией
interface MyComponentProps {
  name: string;
  age?: number;
}

const MyComponent = ({name, age}: MyComponentProps) => {
  return (
    <div>
      <p>Name: {name}</p>
      {age && <p>Age: {age}</p>}
    </div>
  );
};
```

## Типизация хуков

### useState
```tsx
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);
```

### useEffect
```tsx
useEffect(() => {
  // Логика эффекта
}, [dependency]);
```

### useRef
```tsx
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);
```

## Типизация пропсов и состояния

Для типизации пропсов рекомендуется использовать интерфейсы:

```tsx
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
  };
  isActive: boolean;
  onToggle: () => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, isActive, onToggle }) => {
  return (
    <div className={isActive ? 'active' : 'inactive'}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={onToggle}>
        {isActive ? 'Deactivate' : 'Activate'}
      </button>
    </div>
  );
};
```

## Типизация событий

```tsx
const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
  // Обработка клика
};

const handleInput = (event: React.ChangeEvent<HTMLInputElement>) => {
  const value = event.target.value;
  // Обработка изменения инпута
};

const handleFormSubmit = (event: React.FormEvent<HTMLFormElement>) => {
  event.preventDefault();
  // Обработка отправки формы
};
```

## Типизация Refs

```tsx
const MyComponent = () => {
  const inputRef = useRef<HTMLInputElement>(null);
  const divRef = useRef<HTMLDivElement>(null);

  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };

  return (
    <div ref={divRef}>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
};
```

## Создание определений типов для компонентов

Создайте файл `types/components.ts` для общих типов компонентов:

```ts
export interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
}

export interface InputProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
  error?: string;
}
```

## Утилитарные типы

TypeScript предоставляет несколько полезных утилитарных типов:

### Partial
```ts
// Делает все свойства опциональными
interface User {
  name: string;
  email: string;
  age: number;
}

type PartialUser = Partial<User>; // { name?: string; email?: string; age?: number; }
```

### Pick
```ts
// Выбирает только указанные свойства
type UserPreview = Pick<User, 'name' | 'email'>; // { name: string; email: string; }
```

### Omit
```ts
// Исключает указанные свойства
type UserWithoutId = Omit<User, 'id'>; // { name: string; email: string; age: number; }
```

### Required
```ts
// Делает все свойства обязательными
type RequiredUser = Required<Partial<User>>; // { name: string; email: string; age: number; }
```

## Типизация кастомных хуков

```ts
// Хук для управления формой
interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: (name: keyof T, value: any) => void;
  handleSubmit: () => void;
}

const useForm = <T extends Record<string, any>>(
  initialState: T
): FormState<T> => {
  const [values, setValues] = useState<T>(initialState);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

  const handleChange = (name: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = () => {
    // Логика отправки формы
  };

  return { values, errors, handleChange, handleSubmit };
};
```

## Типизация контекста

```tsx
// Определение типа контекста
interface UserContextType {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

// Создание контекста
const UserContext = createContext<UserContextType | undefined>(undefined);

// Провайдер контекста
export const UserProvider: React.FC<{children: React.ReactNode}> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = (userData: User) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
};

// Хук для использования контекста
export const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
};
```

## Типизация редьюсера

```ts
interface State {
  count: number;
  loading: boolean;
  error: string | null;
}

type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'RESET':
      return { ...state, count: 0 };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload };
    default:
      return state;
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, { count: 0, loading: false, error: null });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
    </div>
  );
};
```

## Типизация форм

```tsx
interface FormData {
  email: string;
  password: string;
  remember: boolean;
}

const LoginForm = () => {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
    remember: false
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Отправка формы
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <label>
        <input
          type="checkbox"
          name="remember"
          checked={formData.remember}
          onChange={handleChange}
        />
        Remember me
      </label>
      <button type="submit">Login</button>
    </form>
  );
};
```

## Дженерик-компоненты

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

const List = <T,>({ items, renderItem, keyExtractor }: ListProps<T>) => {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
};

// Использование
const users: User[] = [
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com' }
];

<List 
  items={users}
  keyExtractor={user => user.id}
  renderItem={(user, index) => <div>{user.name}</div>}
/>;
```

## Условная типизация

```ts
type ApiResponse<T> = T extends { success: true } 
  ? { data: T } 
  : { error: string };

type SuccessResponse = ApiResponse<{ success: true; data: User }>; // { data: { success: true; data: User } }
type ErrorResponse = ApiResponse<{ success: false; message: string }>; // { error: string }
```

## Работа с типами сторонних библиотек

Для типизации сторонних библиотек:

1. Установите соответствующие @types пакеты: `npm install --save-dev @types/library-name`
2. При необходимости создайте объявления типов:

```ts
// types/library.d.ts
declare module 'some-external-library' {
  export interface LibraryOptions {
    apiKey: string;
    timeout: number;
  }
  
  export function initialize(options: LibraryOptions): void;
}
```

## Миграция с JavaScript на TypeScript в React

1. Добавьте TypeScript в проект
2. Переименуйте файлы с `.js` на `.tsx`
3. Начните с типизации пропсов и возвращаемых значений
4. Используйте `any` временно, затем постепенно уточняйте типы
5. Включите строгую проверку типов

## Лучшие практики

- Используйте интерфейсы для определения форм и данных
- Избегайте избыточной типизации
- Используйте утилитарные типы для создания новых типов
- Создавайте переиспользуемые типы в отдельных файлах
- Включите строгую проверку типов (`strict: true`)

## Частые паттерны

- Типизация компонентов через интерфейсы
- Использование `React.PropsWithChildren`
- Типизация асинхронных операций
- Обработка ошибок с типами

## Избегание утверждений типов

Предпочтительно использовать утверждения типов только когда это действительно необходимо:

```ts
// Не рекомендуется
const element = document.getElementById('myId') as HTMLInputElement;

// Рекомендуется
const element = document.getElementById('myId');
if (element instanceof HTMLInputElement) {
  // Использование element как HTMLInputElement
}
```

## Рассмотрение строгого режима

Включите строгие настройки TypeScript для лучшей безопасности типов:

```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "strictFunctionTypes": true,
  "noImplicitReturns": true
}
```

## Связи с другими файлами

- [[React Hooks]] - для понимания работы хуков с типизацией
- [[TypeScript Basics]] - основы TypeScript
- [[React Components]] - компонентный подход в React
- [[Form Handling]] - обработка форм в React

## Заключение

Использование TypeScript с React значительно повышает надежность и поддерживаемость кода. Правильная типизация помогает избежать многих ошибок и улучшает автодополнение в IDE.