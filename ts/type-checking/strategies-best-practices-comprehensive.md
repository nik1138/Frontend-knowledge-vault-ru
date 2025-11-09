# Стратегии проверки типов и лучшие практики в TypeScript

Эффективное использование TypeScript требует понимания различных стратегий проверки типов и следования лучшим практикам. Это позволяет создавать надежный, поддерживаемый и типобезопасный код.

## Стратегии проверки типов

### 1. Строгая проверка типов

```typescript
// Включаем строгие настройки в tsconfig.json:
// {
//   "compilerOptions": {
//     "strict": true,
//     "noImplicitAny": true,
//     "strictNullChecks": true,
//     "strictFunctionTypes": true,
//     "strictBindCallApply": true,
//     "strictPropertyInitialization": true,
//     "noImplicitThis": true,
//     "alwaysStrict": true
//   }
// }

// Пример кода с строгой проверкой
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Необязательное поле
}

function processUser(user: User) {
  // TypeScript гарантирует, что user.id и user.name существуют
  console.log(`Processing user: ${user.name} (${user.id})`);
  
  // TypeScript проверяет, что age может быть undefined
  if (user.age !== undefined) {
    console.log(`Age: ${user.age}`);
  }
  
  // Ошибка: обращение к potentially undefined без проверки
  // console.log(user.age.toFixed(2)); // Ошибка в strict mode
}

// Правильное использование
function safeProcessUser(user: User) {
  if (user.age !== undefined) {
    console.log(user.age.toFixed(2)); // OK: проверка на undefined
  }
}
```

### 2. Проверка типов во время выполнения

```typescript
// Защитники типов (Type Guards)
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isUser(obj: any): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    typeof obj.id === "number" &&
    typeof obj.name === "string" &&
    typeof obj.email === "string"
  );
}

function processValue(value: unknown) {
  if (isString(value)) {
    // value теперь типизирован как string
    console.log(value.toUpperCase());
  } else if (isNumber(value)) {
    // value теперь типизирован как number
    console.log(value.toFixed(2));
  }
}

function handleApiResponse(response: unknown) {
  if (isUser(response)) {
    // response теперь типизирован как User
    console.log(`Пользователь: ${response.name}`);
  } else {
    console.error("Неверный формат ответа");
  }
}
```

### 3. Использование as const для сохранения литеральных типов

```typescript
// Без as const TypeScript расширяет типы
const config1 = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
};
// тип: { apiUrl: string; timeout: number; retries: number; }

// С as const TypeScript сохраняет литеральные типы
const config2 = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
} as const;
// тип: { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000; readonly retries: 3; }

// Это полезно для создания типов на основе значений
type ApiUrl = typeof config2.apiUrl; // "https://api.example.com"
type Timeout = typeof config2.timeout; // 5000
```

## Лучшие практики типизации

### 1. Использование конкретных типов вместо any

```typescript
// Плохо: используем any
function processData(data: any): any {
  return data.map((item: any) => item.value * 2);
}

// Лучше: определяем конкретные типы
interface DataItem {
  value: number;
  id: string;
}

function processTypedData(data: DataItem[]): DataItem[] {
  return data.map(item => ({
    ...item,
    value: item.value * 2
  }));
}

// Или используем обобщения для универсальности
function processGenericData<T extends { value: number }>(data: T[]): T[] {
  return data.map(item => ({
    ...item,
    value: item.value * 2
  })) as T[];
}
```

### 2. Создание переиспользуемых утилит типов

```typescript
// Утилита для создания частичного типа с обязательными полями
type PartialWithRequired<T, K extends keyof T> = Partial<T> & Required<Pick<T, K>>;

interface UserForm {
  id: number;
  name: string;
  email: string;
  age: number;
  avatar?: string;
}

// Создаем тип для формы редактирования пользователя
// где id, name и email обязательны, а остальные опциональны
type EditUserForm = PartialWithRequired<UserForm, "id" | "name" | "email">;

// Утилита для создания типа только для чтения до указанной глубины
type DeepReadonly<T, Depth extends number = 5> = 
  Depth extends 0 
    ? T 
    : T extends object 
      ? { readonly [K in keyof T]: DeepReadonly<T[K], Prev[Depth]> } 
      : T;

type Prev = [never, 0, 1, 2, 3, 4, 5];
```

### 3. Использование дискриминированных объединений

```typescript
interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  error: string;
}

type ApiResponse<T> = LoadingState | SuccessState<T> | ErrorState;

function handleResponse<T>(response: ApiResponse<T>) {
  switch (response.status) {
    case "loading":
      // response имеет тип LoadingState
      console.log(`Загрузка... ${response.progress || 0}%`);
      break;
    case "success":
      // response имеет тип SuccessState<T>
      console.log("Данные:", response.data);
      break;
    case "error":
      // response имеет тип ErrorState
      console.error("Ошибка:", response.error);
      break;
  }
}
```

### 4. Типобезопасная работа с API

```typescript
// Определение типов для API
interface ApiRequest<T> {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  url: string;
  data?: T;
  headers?: Record<string, string>;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  headers: Record<string, string>;
}

// Типизированная функция для выполнения запросов
async function apiCall<T, R = T>(
  request: ApiRequest<T>
): Promise<ApiResponse<R>> {
  const response = await fetch(request.url, {
    method: request.method,
    headers: {
      'Content-Type': 'application/json',
      ...request.headers
    },
    body: request.data ? JSON.stringify(request.data) : undefined
  });
  
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  }
  
  const data = await response.json();
  return {
    data,
    status: response.status,
    headers: Object.fromEntries(response.headers.entries())
  };
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<User> {
  const response = await apiCall<void, User>({
    method: 'GET',
    url: `/api/users/${id}`
  });
  return response.data;
}
```

## Продвинутые стратегии

### 1. Условная типизация

```typescript
// Утилита для извлечения типа элемента массива
type ArrayElementType<T> = T extends (infer U)[] ? U : T;

type StrArray = string[];
type NumArray = number[];
type MixedArray = (string | number)[];

type StrElement = ArrayElementType<StrArray>;    // string
type NumElement = ArrayElementType<NumArray>;    // number
type MixedElement = ArrayElementType<MixedArray>; // string | number

// Утилита для извлечения типа разрешаемого значения промиса
type PromiseType<T> = T extends Promise<infer U> ? U : T;

type ResolvedString = PromiseType<Promise<string>>;    // string
type ResolvedNumber = PromiseType<Promise<number>>;    // number
type NotPromise = PromiseType<string>;                 // string
```

### 2. Сопоставленные типы

```typescript
// Утилита для создания типа с префиксами к ключам
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

interface User {
  name: string;
  age: number;
}

type PrefixedUser = Prefixed<User, "api">;
// { apiName: string; apiAge: number; }

// Утилита для выбора свойств определенного типа
type PropertiesOfType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};

type StringProperties = PropertiesOfType<User, string>;
// { name: string; }
```

### 3. Рекурсивные типы

```typescript
// Тип для глубокого объединения объектов
type DeepMerge<T, U> = {
  [K in keyof T | keyof U]: 
    K extends keyof T 
      ? K extends keyof U 
        ? T[K] extends object 
          ? U[K] extends object 
            ? DeepMerge<T[K], U[K]> 
            : T[K] | U[K]
          : T[K] | U[K]
        : T[K]
      : K extends keyof U 
        ? U[K] 
        : never
};

interface ConfigA {
  server: {
    host: string;
    port: number;
  };
  features: {
    logging: boolean;
  };
}

interface ConfigB {
  server: {
    timeout: number;
  };
  features: {
    caching: boolean;
  };
}

type MergedConfig = DeepMerge<ConfigA, ConfigB>;
// {
//   server: {
//     host: string;
//     port: number;
//     timeout: number;
//   };
//   features: {
//     logging: boolean;
//     caching: boolean;
//   };
// }
```

## Практические примеры из реальной разработки

### 1. Система валидации форм

```typescript
// Типы для валидации
interface Validator<T> {
  (value: T): { isValid: boolean; error?: string };
}

interface FieldConfig<T> {
  validators: Validator<T>[];
  defaultValue: T;
}

interface FormConfig<T> {
  [K in keyof T]: FieldConfig<T[K]>;
}

class FormValidator<T> {
  private config: FormConfig<T>;
  
  constructor(config: FormConfig<T>) {
    this.config = config;
  }
  
  validate(formData: Partial<T>): { [K in keyof T]?: string[] } {
    const errors: { [key: string]: string[] } = {};
    
    for (const fieldName in this.config) {
      if (fieldName in formData) {
        const value = formData[fieldName];
        const fieldConfig = this.config[fieldName];
        
        const fieldErrors: string[] = [];
        for (const validator of fieldConfig.validators) {
          const result = validator(value as T[typeof fieldName]);
          if (!result.isValid && result.error) {
            fieldErrors.push(result.error);
          }
        }
        
        if (fieldErrors.length > 0) {
          errors[fieldName] = fieldErrors;
        }
      }
    }
    
    return errors as { [K in keyof T]?: string[] };
  }
}

// Использование
interface UserFormData {
  email: string;
  password: string;
  age: number;
}

const emailValidator: Validator<string> = (email) => {
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  return { isValid, error: isValid ? undefined : "Неверный формат email" };
};

const passwordValidator: Validator<string> = (password) => {
  const isValid = password.length >= 8;
  return { isValid, error: isValid ? undefined : "Пароль должен быть не менее 8 символов" };
};

const ageValidator: Validator<number> = (age) => {
  const isValid = age >= 18 && age <= 120;
  return { isValid, error: isValid ? undefined : "Возраст должен быть от 18 до 120" };
};

const userFormValidator = new FormValidator<UserFormData>({
  email: {
    validators: [emailValidator],
    defaultValue: ""
  },
  password: {
    validators: [passwordValidator],
    defaultValue: ""
  },
  age: {
    validators: [ageValidator],
    defaultValue: 18
  }
});

// Валидация данных формы
const formData = { email: "invalid-email", password: "123", age: 15 };
const errors = userFormValidator.validate(formData);
console.log(errors); // { email: ["Неверный формат email"], password: ["Пароль должен быть не менее 8 символов"], age: ["Возраст должен быть от 18 до 120"] }
```

### 2. Типобезопасная система событий

```typescript
// Типизированная система событий
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: { message: string; code: number };
}

type EventKey = keyof EventMap;
type EventData<T extends EventKey> = EventMap[T];

class TypedEventEmitter {
  private listeners: {
    [K in EventKey]?: ((data: EventData<K>) => void)[];
  } = {};
  
  on<T extends EventKey>(event: T, listener: (data: EventData<T>) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener as (data: EventData<any>) => void);
  }
  
  off<T extends EventKey>(event: T, listener: (data: EventData<T>) => void): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      const index = eventListeners.indexOf(listener as (data: EventData<any>) => void);
      if (index > -1) {
        eventListeners.splice(index, 1);
      }
    }
  }
  
  emit<T extends EventKey>(event: T, data: EventData<T>): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }
}

// Использование
const emitter = new TypedEventEmitter();

// Типобезопасная подписка на события
emitter.on('user_login', (data) => {
  console.log(`Пользователь ${data.userId} вошёл в ${data.timestamp}`);
  // data.userId: number, data.timestamp: Date
});

emitter.on('error', (data) => {
  console.error(`Ошибка ${data.code}: ${data.message}`);
  // data.code: number, data.message: string
});

// Типобезопасная отправка событий
emitter.emit('user_login', { 
  userId: 123, 
  timestamp: new Date() 
});

emitter.emit('error', { 
  message: "Что-то пошло не так", 
  code: 500 
});
```

### 3. Управление состоянием с типизацией

```typescript
// Типизированное управление состоянием
interface StateMachine<State extends string, Event extends { type: string }> {
  current: State;
  states: {
    [S in State]: {
      on?: {
        [T in Event['type']]?: State;
      };
    };
  };
  onTransition?: (from: State, to: State, event: Event) => void;
}

class TypedStateMachine<
  States extends string, 
  Events extends { type: string }
> {
  private state: States;
  private config: StateMachine<States, Events>;
  
  constructor(initialState: States, config: StateMachine<States, Events>) {
    this.state = initialState;
    this.config = config;
  }
  
  getState(): States {
    return this.state;
  }
  
  canTransition(eventType: Events['type']): boolean {
    const allowedTransitions = this.config.states[this.state].on;
    return !!(allowedTransitions && eventType in allowedTransitions);
  }
  
  transition(event: Events): boolean {
    const allowedTransitions = this.config.states[this.state].on;
    
    if (allowedTransitions && event.type in allowedTransitions) {
      const newState = allowedTransitions[event.type] as States;
      
      if (this.config.onTransition) {
        this.config.onTransition(this.state, newState, event);
      }
      
      this.state = newState;
      return true;
    }
    
    return false;
  }
}

// Использование
type LoadingStates = 'idle' | 'loading' | 'success' | 'error';
type LoadingEvents = 
  | { type: 'FETCH_START' }
  | { type: 'FETCH_SUCCESS'; data: any }
  | { type: 'FETCH_ERROR'; error: string }
  | { type: 'RESET' };

const loadingMachine = new TypedStateMachine<LoadingStates, LoadingEvents>('idle', {
  current: 'idle',
  states: {
    idle: {
      on: {
        'FETCH_START': 'loading'
      }
    },
    loading: {
      on: {
        'FETCH_SUCCESS': 'success',
        'FETCH_ERROR': 'error'
      }
    },
    success: {
      on: {
        'RESET': 'idle'
      }
    },
    error: {
      on: {
        'RESET': 'idle'
      }
    }
  },
  onTransition: (from, to, event) => {
    console.log(`Переход из ${from} в ${to} по событию ${event.type}`);
  }
});
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// Ошибка 1: Использование any вместо правильной типизации
function badExample(data: any[]) {
  return data.map(item => item.id); // Ошибка во время выполнения, если item не имеет id
}

// Решение: Правильная типизация
function goodExample<T extends { id: any }>(data: T[]) {
  return data.map(item => item.id);
}

// Ошибка 2: Неправильная обработка null/undefined
interface User {
  name: string;
  email?: string;
}

function badNullHandling(user: User) {
  // Это может вызвать ошибку во время выполнения
  // return user.email.toUpperCase(); // Ошибка, если email undefined
}

function goodNullHandling(user: User) {
  // Правильная проверка
  return user.email ? user.email.toUpperCase() : "No email";
}

// Ошибка 3: Неиспользование readonly для неизменяемых данных
interface Config {
  apiUrl: string;
  timeout: number;
}

let config: Config = { apiUrl: "https://api.com", timeout: 5000 };
// config.apiUrl = "changed"; // Это возможно, но может быть нежелательно

// Лучше использовать readonly
const readonlyConfig: Readonly<Config> = { apiUrl: "https://api.com", timeout: 5000 };
// readonlyConfig.apiUrl = "changed"; // Ошибка компиляции
```

### 2. Лучшие практики

```typescript
// 1. Используйте интерфейсы для публичных API, типы для закрытых
// Интерфейсы могут быть расширены, типы - нет
export interface PublicApi {  // Публичный API
  method(): void;
}

interface InternalType {      // Внутренний тип
  internalProp: string;
}

// 2. Используйте Pick и Omit для производных типов
interface FullUser {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

type UserForRegistration = Pick<FullUser, 'name' | 'email' | 'password'>;
type UserProfile = Omit<FullUser, 'password'>;

// 3. Используйте branded types для дополнительной безопасности
type Brand<T, B> = T & { __brand: B };
type UserId = Brand<number, 'UserId'>;
type Email = Brand<string, 'Email'>;

function createUserId(id: number): UserId {
  return id as UserId;
}

function createEmail(email: string): Email {
  return email as Email;
}

// 4. Используйте utility types для преобразования типов
type PartialUser = Partial<FullUser>;
type RequiredUser = Required<Omit<FullUser, 'id'>>;
type ReadOnlyUser = Readonly<FullUser>;
```

## Рекомендации по настройке компилятора

### 1. Строгие настройки

```json
{
  "compilerOptions": {
    // Основные строгие настройки
    "strict": true,
    
    // Предотвращение неявного any
    "noImplicitAny": true,
    
    // Проверка null и undefined
    "strictNullChecks": true,
    
    // Проверка типов функций
    "strictFunctionTypes": true,
    
    // Проверка вызовов функций
    "strictBindCallApply": true,
    
    // Проверка инициализации свойств
    "strictPropertyInitialization": true,
    
    // Предотвращение неявного this
    "noImplicitThis": true,
    
    // Всегда использовать strict mode
    "alwaysStrict": true,
    
    // Проверка лишних свойств в объектных литералах
    "noImplicitReturns": true,
    
    // Проверка достижимости кода
    "noFallthroughCasesInSwitch": true,
    
    // Проверка импортов
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    
    // Дополнительные проверки
    "forceConsistentCasingInFileNames": true
  }
}
```

## Лучшие практики для командной разработки

1. **Создайте общий стиль типизации** - документируйте подходы к типизации в вашей команде
2. **Используйте линтеры** - ESLint с TypeScript правилами для обеспечения консистентности
3. **Регулярно обновляйте зависимости** - включая TypeScript, чтобы использовать новые возможности
4. **Пишите тесты для типов** - используйте dtslint или tsd для тестирования типов
5. **Документируйте сложные типы** - особенно когда они используют условные или сопоставленные типы
6. **Используйте консистентные имена** - для типов, интерфейсов и переменных

## Связи с другими концепциями

- [[ts/type-system/type-compatibility|Совместимость типов]] - как проверяется совместимость типов
- [[ts/advanced-types/conditional-types|Условные типы]] - продвинутые паттерны типизации
- [[ts/utility-types/index|Утилиты типов]] - встроенные и пользовательские утилиты
- [[ts/type-system/type-inference|Вывод типов]] - как TypeScript автоматически определяет типы
- [[architecture/component-architecture]] - архитектурные аспекты типизации