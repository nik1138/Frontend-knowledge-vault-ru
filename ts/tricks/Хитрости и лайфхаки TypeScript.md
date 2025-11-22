---
tags: [typescript, frontend, tricks, tips, advanced]
aliases: [Хитрости TypeScript, Лайфхаки TypeScript]
---

# Хитрости и лайфхаки TypeScript

## Введение

TypeScript предоставляет множество возможностей, которые могут значительно улучшить опыт разработки. В этом разделе мы рассмотрим различные хитрости и лайфхаки, которые помогут вам писать более чистый, типобезопасный и эффективный код.

## Полезные трюки с типами

### Использование `satisfies` оператора (TypeScript 4.9+)

```ts
// Оператор satisfies позволяет проверить соответствие типа
// без потери информации о конкретных значениях

// Без satisfies - теряется информация о конкретных ключах
const userWithoutSatisfies = {
  name: "John",
  age: 30,
  preferences: {
    theme: "dark",
    language: "en"
  }
} as const; // Тип: { readonly name: "John"; readonly age: 30; ... }

// С satisfies - сохраняется информация о конкретных значениях
const userWithSatisfies = {
  name: "John",
  age: 30,
  preferences: {
    theme: "dark",
    language: "en"
  }
} satisfies Record<string, unknown>;

// Можем получить конкретные ключи
type UserKeys = keyof typeof userWithSatisfies; // "name" | "age" | "preferences"

// Использование с интерфейсом
interface ApiResponse {
  data: {
    [key: string]: string | number | boolean;
  };
  status: number;
}

const response = {
  data: {
    id: 123,
    name: "Product",
    inStock: true,
    price: 29.99
  },
  status: 200
} satisfies ApiResponse;
```

### Утилита `NonNullable<T>`

```ts
// Удаление null и undefined из типа
type StringOrNull = string | null | undefined;
type OnlyString = NonNullable<StringOrNull>; // string

// Практическое применение
function processUser(id: number | null | undefined) {
  if (id === null || id === undefined) {
    return null;
  }
  
  // Тип id теперь NonNullable<number | null | undefined> = number
  return fetchUser(id);
}

// Или с помощью утилиты
function processUserWithUtility(id: number | null | undefined) {
  const validId = id as NonNullable<typeof id>; // number
  return fetchUser(validId);
}
```

### Извлечение типов из сложных структур

```ts
// Извлечение типа из Promise
type PromiseType<T> = T extends Promise<infer U> ? U : T;

type AsyncResult = Promise<string>;
type UnwrappedResult = PromiseType<AsyncResult>; // string

// Извлечение типа из массива
type ArrayType<T> = T extends (infer U)[] ? U : T;

type StringArray = string[];
type ElementType = ArrayType<StringArray>; // string

// Извлечение типа возврата функции
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

const getUser = () => ({ id: 1, name: "John" });
type User = ReturnType<typeof getUser>; // { id: number; name: string; }

// Извлечение параметров функции
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

const updateUser = (id: number, name: string) => ({ id, name });
type UpdateParams = Parameters<typeof updateUser>; // [number, string]
```

## Продвинутые паттерны типизации

### Discriminated Unions (Tagged Unions)

```ts
// Использование discriminant для безопасной работы с различными типами

interface LoadingState {
  status: 'loading';
}

interface SuccessState<T> {
  status: 'success';
  data: T;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type ApiState<T> = LoadingState | SuccessState<T> | ErrorState;

// Функция, которая безопасно работает с различными состояниями
function handleApiState<T>(state: ApiState<T>): string {
  switch (state.status) {
    case 'loading':
      return 'Loading...';
    case 'success':
      // TypeScript знает, что в этом случае у state есть свойство data
      return `Data: ${JSON.stringify(state.data)}`;
    case 'error':
      // TypeScript знает, что в этом случае у state есть свойство error
      return `Error: ${state.error}`;
  }
}

// Извлечение данных из успешного состояния
function extractData<T>(state: ApiState<T>): T | null {
  if (state.status === 'success') {
    // TypeScript автоматически сужает тип до SuccessState<T>
    return state.data;
  }
  return null;
}
```

### Conditional Types с дистрибутивностью

```ts
// Дистрибутивные условные типы
type FilterFlags<Base, Condition> = {
  [Key in keyof Base]: Base[Key] extends Condition ? Key : never;
};

type BooleanPropsNames = FilterFlags<User, boolean>; // Типы, значения которых являются boolean

interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
  createdAt: Date;
}

type BooleanKeys = FilterFlags<User, boolean>; // "isActive"
type StringKeys = FilterFlags<User, string>;   // "name" | "email"
type DateKeys = FilterFlags<User, Date>;       // "createdAt"

// Извлечение ключей с определенным типом
type AllowedBooleanProps = Pick<User, BooleanKeys extends string ? BooleanKeys : never>;
// { isActive: boolean; }

// Утилита для создания типа только с определенными типами свойств
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};

type BooleanProps = PickByType<User, boolean>; // { isActive: boolean; }
type PrimitiveProps = PickByType<User, string | number | boolean>; // { id: number; name: string; email: string; isActive: boolean; }
```

### Template Literal Types для создания типов

```ts
// Использование template literal types для создания новых типов
type EventName = "click" | "hover" | "focus";

// Создание типа для обработчиков событий
type EventHandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onHover" | "onFocus"

// Создание типа для асинхронных функций
type AsyncMethod<T> = {
  [K in keyof T as `${string & K}Async`]: (...args: Parameters<T[K]>) => Promise<Awaited<ReturnType<T[K]>>>
};

interface UserService {
  getUser(id: number): { id: number; name: string };
  updateUser(id: number, data: Partial<User>): { id: number; name: string };
}

type AsyncUserService = AsyncMethod<UserService>;
// {
//   getUserAsync: (id: number) => Promise<{ id: number; name: string }>;
//   updateUserAsync: (id: number, data: Partial<User>) => Promise<{ id: number; name: string }>;
// }

// Преобразование ключей объекта
type AddPrefix<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

type PrefixedConfig = AddPrefix<Config, 'app'>;
// { appApiUrl: string; appTimeout: number; appRetries: number; }
```

## Практические хитрости для frontend разработки

### Типизация для CSS-in-JS

```ts
// Типизация для styled-components или подобных библиотек
interface Theme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
  };
  spacing: {
    small: string;
    medium: string;
    large: string;
  };
  breakpoints: {
    mobile: string;
    tablet: string;
    desktop: string;
  };
}

// Создание типизированного хелпера для медиа-запросов
type Breakpoint = keyof Theme['breakpoints'];

const mediaQuery = (breakpoint: Breakpoint) => (
  literals: TemplateStringsArray,
  ...placeholders: any[]
) => `
  @media (min-width: ${theme.breakpoints[breakpoint]}) {
    ${literals.reduce((acc, literal, i) => 
      `${acc}${literal}${placeholders[i] !== undefined ? placeholders[i] : ''}`, 
      ''
    )}
  }
`;

// Использование в styled-components
import styled from 'styled-components';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

const Button = styled.button<ButtonProps>`
  background-color: ${props => 
    props.variant === 'primary' ? props.theme.colors.primary : props.theme.colors.secondary
  };
  padding: ${props => {
    switch(props.size) {
      case 'small': return props.theme.spacing.small;
      case 'large': return props.theme.spacing.large;
      default: return props.theme.spacing.medium;
    }
  }};
  cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};
  opacity: ${props => props.disabled ? 0.6 : 1};
  
  ${mediaQuery('tablet')} {
    font-size: 1.2em;
  }
`;
```

### Типизация для React Context с безопасностью

```tsx
import React, { createContext, useContext } from 'react';

// Создание типизированного контекста с проверкой использования
interface AuthContextType {
  user: { id: number; name: string } | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
}

// Создание контекста с начальным значением undefined
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Провайдер контекста
interface AuthProviderProps {
  children: React.ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  // Реализация провайдера...
  const value: AuthContextType = {
    user: null,
    login: async () => {},
    logout: () => {},
    loading: false
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

// Хук с проверкой контекста и типизацией
export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  
  return context;
};

// Использование хука с полной типизацией
const UserProfile: React.FC = () => {
  const { user, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>Please log in</div>;
  
  return <div>Welcome, {user.name}!</div>;
};
```

### Типизация для динамических компонентов

```tsx
import React from 'react';

// Типизация компонентов с динамическим тегом
type PolymorphicComponent<Props, DefaultElement extends React.ElementType> = {
  <Element extends React.ElementType = DefaultElement>(
    props: {
      as?: Element;
    } & React.ComponentPropsWithoutRef<Element> & 
    Omit<Props, keyof React.ComponentPropsWithoutRef<Element>>
  ): React.ReactElement | null;
} & Omit<React.NamedExoticComponent<Props>, 'propTypes' | 'contextTypes' | 'displayName'>;

interface BoxOwnProps {
  children: React.ReactNode;
  className?: string;
  padding?: number;
  margin?: number;
}

type BoxProps<T extends React.ElementType> = BoxOwnProps & {
  as?: T;
};

const Box = React.forwardRef(
  <T extends React.ElementType = 'div'>(
    { as, children, className = '', padding = 0, margin = 0, ...props }: BoxProps<T>,
    ref: React.Ref<HTMLElement>
  ) => {
    const Component = as || 'div';
    
    const style = {
      padding: `${padding}px`,
      margin: `${margin}px`,
    };
    
    return (
      <Component
        ref={ref}
        className={className}
        style={style}
        {...props}
      >
        {children}
      </Component>
    );
  }
) as PolymorphicComponent<BoxOwnProps, 'div'>;

// Использование полиморфного компонента
const Example = () => (
  <div>
    <Box as="section" padding={20} margin={10}>
      This is a section
    </Box>
    <Box as="article" padding={15}>
      This is an article
    </Box>
    <Box padding={10} className="custom-class">
      This is a div (default)
    </Box>
  </div>
);
```

## Оптимизация производительности типов

### Использование `satisfies` для предотвращения лишней типизации

```ts
// Плохо - TypeScript пытается вывести максимально широкий тип
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  features: {
    logging: true,
    caching: true,
    compression: false
  }
} as const;

// Лучше - используем satisfies для проверки типа без потери конкретных значений
const betterConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  features: {
    logging: true,
    caching: true,
    compression: false
  }
} satisfies {
  apiUrl: string;
  timeout: number;
  retries: number;
  features: {
    logging: boolean;
    caching: boolean;
    compression: boolean;
  };
};

// Тип betterConfig сохраняет конкретные строковые/числовые значения
type ApiUrl = typeof betterConfig.apiUrl; // "https://api.example.com" (не string!)
```

### Использование `const assertions` для уточнения типов

```ts
// Без const assertion - получаем обобщенные типы
const actions = ['create', 'update', 'delete'];
type Action = typeof actions[number]; // string (не то, что нужно)

// С const assertion - получаем точные литеральные типы
const betterActions = ['create', 'update', 'delete'] as const;
type BetterAction = typeof betterActions[number]; // "create" | "update" | "delete"

// Использование в реальном примере
const HTTP_METHODS = ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'] as const;
type HttpMethod = typeof HTTP_METHODS[number];

function makeRequest(url: string, method: HttpMethod) {
  return fetch(url, { method });
}

// Теперь TypeScript будет проверять, что method - это один из разрешенных методов
makeRequest('/api/users', 'GET');    // OK
// makeRequest('/api/users', 'HEAD'); // Ошибка: 'HEAD' не является допустимым методом
```

## Продвинутые лайфхаки

### Создание типобезопасных enum-ов

```ts
// Типобезопасная альтернатива enum
const Status = {
  PENDING: 'pending',
  APPROVED: 'approved',
  REJECTED: 'rejected',
} as const;

type Status = typeof Status[keyof typeof Status];

// Использование
const userStatus: Status = Status.PENDING; // OK
// const invalidStatus: Status = 'invalid'; // Ошибка: не является допустимым значением

// Создание типобезопасного объекта действий
const Actions = {
  FETCH_USERS: 'FETCH_USERS',
  CREATE_USER: 'CREATE_USER',
  UPDATE_USER: 'UPDATE_USER',
  DELETE_USER: 'DELETE_USER',
} as const;

type Action = typeof Actions[keyof typeof Actions];

interface ActionObject {
  type: Action;
  payload?: any;
}

// Функция для создания action-ов
function createAction<T extends Action>(
  type: T,
  payload?: T extends 'FETCH_USERS' | 'CREATE_USER' | 'UPDATE_USER' | 'DELETE_USER' 
    ? any 
    : never
): ActionObject {
  return { type, payload };
}
```

### Использование `Omit` и `Pick` для создания производных типов

```ts
interface User {
  id: number;
  name: string;
  email: string;
  password: string; // чувствительное поле
  createdAt: Date;
  updatedAt: Date;
}

// Создание типа для публичного профиля (без пароля)
type PublicProfile = Omit<User, 'password'>;
// { id: number; name: string; email: string; createdAt: Date; updatedAt: Date; }

// Создание типа для регистрации (только необходимые поля)
type RegistrationData = Pick<User, 'name' | 'email' | 'password'>;
// { name: string; email: string; password: string; }

// Создание типа для обновления (все необязательно)
type UpdateUser = Partial<Omit<User, 'id' | 'createdAt'>>;
// { name?: string; email?: string; password?: string; updatedAt?: Date; }

// Создание типа для создания (без id и дат)
type CreateUser = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
// { name: string; email: string; password: string; }
```

### Типизация для валидации форм

```ts
// Типизация для системы валидации форм
interface ValidationRule<T> {
  condition: (value: T) => boolean;
  message: string;
}

interface ValidationResult<T> {
  isValid: boolean;
  errors: string[];
  value: T;
}

function createValidator<T>(
  rules: ValidationRule<T>[]
) {
  return (value: T): ValidationResult<T> => {
    const errors: string[] = [];
    
    for (const rule of rules) {
      if (!rule.condition(value)) {
        errors.push(rule.message);
      }
    }
    
    return {
      isValid: errors.length === 0,
      errors,
      value
    };
  };
}

// Использование для валидации email
const emailValidator = createValidator<string>([
  { 
    condition: (value) => value.includes('@'), 
    message: 'Email must contain @ symbol' 
  },
  { 
    condition: (value) => /\S+@\S+\.\S+/.test(value), 
    message: 'Email format is invalid' 
  }
]);

const emailResult = emailValidator('test@example.com');
// { isValid: true, errors: [], value: 'test@example.com' }
```

## Заключение

Эти хитрости и лайфхаки помогут вам писать более типобезопасный, читаемый и поддерживаемый код на TypeScript. Регулярное использование этих техник улучшит ваш опыт разработки и снизит количество ошибок в коде.

> [!tip] Совет
> Используйте `satisfies` оператор для проверки типов без потери информации о конкретных значениях.

> [!warning] Важно
> Не переусложняйте типы - чрезмерное использование продвинутых паттернов может затруднить понимание кода.

## Связанные темы

- [[Продвинутые типы и метапрограммирование на типах]]
- [[Утилиты типов TypeScript]]
- [[Дженерики и условные типы]]