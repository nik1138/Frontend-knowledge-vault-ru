# Продвинутые паттерны TypeScript

Продвинутые паттерны TypeScript включают в себя сложные техники типизации, которые позволяют создавать более гибкие, безопасные и выразительные API. Эти паттерны особенно полезны при создании библиотек, фреймворков и сложных приложений.

## 1. Паттерн "Брендинг типов" (Branding Pattern)

```typescript
// Создание уникальных типов для предотвращения путаницы
type Brand<T, B> = T & { __brand: B };

type UserId = Brand<number, 'UserId'>;
type Email = Brand<string, 'Email'>;
type Password = Brand<string, 'Password'>;

// Функции для создания "брендированных" типов
function createUserId(id: number): UserId {
  return id as UserId;
}

function createEmail(email: string): Email {
  return email as Email;
}

function createPassword(password: string): Password {
  if (password.length < 8) {
    throw new Error('Пароль должен быть не менее 8 символов');
  }
  return password as Password;
}

// Использование
const userId = createUserId(123);
const email = createEmail('user@example.com');
const password = createPassword('strongpassword');

// Следующее будет ошибкой компиляции:
// const invalid: UserId = email; // Ошибка: Email не может быть присвоен UserId
```

## 2. Паттерн "Дискриминированный объединенный тип с прокси" (Discriminated Union with Proxy)

```typescript
interface LoadingState {
  status: 'loading';
  progress?: number;
}

interface SuccessState<T> {
  status: 'success';
  data: T;
}

interface ErrorState {
  status: 'error';
  error: string;
}

type ApiResponse<T> = LoadingState | SuccessState<T> | ErrorState;

// Прокси для безопасного доступа к состоянию
class ApiResponseProxy<T> {
  constructor(private response: ApiResponse<T>) {}
  
  isLoading(): this is ApiResponseProxy<LoadingState extends ApiResponse<T> ? T : T> {
    return this.response.status === 'loading';
  }
  
  isSuccess(): this is ApiResponseProxy<SuccessState<T>> {
    return this.response.status === 'success';
  }
  
  isError(): this is ApiResponseProxy<ErrorState> {
    return this.response.status === 'error';
  }
  
  getData(): T | null {
    if (this.response.status === 'success') {
      return this.response.data;
    }
    return null;
  }
  
  getError(): string | null {
    if (this.response.status === 'error') {
      return this.response.error;
    }
    return null;
  }
  
  getProgress(): number | null {
    if (this.response.status === 'loading') {
      return this.response.progress ?? null;
    }
    return null;
  }
}

// Использование
const response: ApiResponse<string> = { status: 'success', data: 'Hello' };
const proxy = new ApiResponseProxy(response);

if (proxy.isSuccess()) {
  console.log(proxy.getData()); // Тип: string
}
```

## 3. Паттерн "Функциональный конструktor" (Functional Constructor Pattern)

```typescript
// Создание объектов с валидацией через функции
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly age: number;
}

type UserError = 
  | { type: 'invalid_email'; email: string }
  | { type: 'invalid_age'; age: number };

// Функциональный конструктор с валидацией
function createUser(name: string, email: string, age: number): User | UserError {
  // Валидация email
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return { type: 'invalid_email', email };
  }
  
  // Валидация возраста
  if (age < 0 || age > 150) {
    return { type: 'invalid_age', age };
  }
  
  // Создание объекта
  return {
    id: Date.now(), // или генерация UUID
    name,
    email,
    age
  };
}

// Улучшенная версия с Result типом
type Result<T, E> = { success: true; value: T } | { success: false; error: E };

function createUserSafe(name: string, email: string, age: number): Result<User, UserError> {
  // Валидация email
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(email)) {
    return { 
      success: false, 
      error: { type: 'invalid_email', email } 
    };
  }
  
  // Валидация возраста
  if (age < 0 || age > 150) {
    return { 
      success: false, 
      error: { type: 'invalid_age', age } 
    };
  }
  
  return {
    success: true,
    value: {
      id: Date.now(),
      name,
      email,
      age
    }
  };
}

// Использование
const result = createUserSafe('John', 'john@example.com', 30);
if (result.success) {
  console.log('Пользователь создан:', result.value);
} else {
  console.error('Ошибка создания пользователя:', result.error);
}
```

## 4. Паттерн "Типобезопасный билдер" (Type-Safe Builder Pattern)

```typescript
// Продвинутый билдер с типобезопасными шагами
interface UserBuilderState {
  name?: string;
  email?: string;
  age?: number;
  phone?: string;
  address?: string;
}

class UserBuilder {
  private state: UserBuilderState = {};
  
  name(name: string): this {
    this.state.name = name;
    return this;
  }
  
  email(email: string): this {
    this.state.email = email;
    return this;
  }
  
  age(age: number): this {
    this.state.age = age;
    return this;
  }
  
  phone(phone: string): this {
    this.state.phone = phone;
    return this;
  }
  
  address(address: string): this {
    this.state.address = address;
    return this;
  }
  
  build(): UserBuilderState {
    // Валидация перед построением
    if (!this.state.name) {
      throw new Error('Имя обязательно');
    }
    if (!this.state.email) {
      throw new Error('Email обязателен');
    }
    
    return { ...this.state };
  }
}

// Использование
const user = new UserBuilder()
  .name('John Doe')
  .email('john@example.com')
  .age(30)
  .phone('+1234567890')
  .address('123 Main St')
  .build();

// Продвинутый билдер с обязательными полями
interface RequiredUserFields {
  name: string;
  email: string;
}

interface OptionalUserFields {
  age?: number;
  phone?: string;
  address?: string;
}

class AdvancedUserBuilder<R extends RequiredUserFields, O extends OptionalUserFields = {}> {
  private required: R;
  private optional: O = {} as O;
  
  constructor(required: R) {
    this.required = required;
  }
  
  withAge(age: number): AdvancedUserBuilder<R, O & { age: number }> {
    return this.setField('age', age);
  }
  
  withPhone(phone: string): AdvancedUserBuilder<R, O & { phone: string }> {
    return this.setField('phone', phone);
  }
  
  withAddress(address: string): AdvancedUserBuilder<R, O & { address: string }> {
    return this.setField('address', address);
  }
  
  private setField<K extends string, V>(
    key: K, 
    value: V
  ): AdvancedUserBuilder<R, O & { [P in K]: V }> {
    (this.optional as any)[key] = value;
    return this as any;
  }
  
  build(): R & O {
    return { ...this.required, ...this.optional };
  }
}

// Использование
const advancedUser = new AdvancedUserBuilder({ name: 'Jane', email: 'jane@example.com' })
  .withAge(25)
  .withPhone('+0987654321')
  .build();

// advancedUser типизирован как { name: string; email: string; age: number; phone: string }
```

## 5. Паттерн "Типобезопасная миграция" (Type-Safe Migration Pattern)

```typescript
// Паттерн для безопасной миграции между версиями данных
interface UserV1 {
  id: number;
  name: string;
  email: string;
}

interface UserV2 {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  createdAt: string;
}

interface UserV3 {
  id: number;
  profile: {
    firstName: string;
    lastName: string;
  };
  contact: {
    email: string;
  };
  metadata: {
    createdAt: string;
    lastLogin: string;
  };
}

// Интерфейс стратегии миграции
interface MigrationStrategy<From, To> {
  from: string;
  to: string;
  migrate: (data: From) => To;
}

// Реализация стратегий миграции
const v1ToV2: MigrationStrategy<UserV1, UserV2> = {
  from: 'v1',
  to: 'v2',
  migrate: (user) => {
    const [firstName, lastName] = user.name.split(' ');
    return {
      id: user.id,
      firstName: firstName || user.name,
      lastName: lastName || '',
      email: user.email,
      createdAt: new Date().toISOString()
    };
  }
};

const v2ToV3: MigrationStrategy<UserV2, UserV3> = {
  from: 'v2',
  to: 'v3',
  migrate: (user) => {
    return {
      id: user.id,
      profile: {
        firstName: user.firstName,
        lastName: user.lastName
      },
      contact: {
        email: user.email
      },
      metadata: {
        createdAt: user.createdAt,
        lastLogin: user.createdAt
      }
    };
  }
};

// Менеджер миграций
class MigrationManager {
  private strategies: Map<string, MigrationStrategy<any, any>> = new Map();
  
  register<T, U>(strategy: MigrationStrategy<T, U>): void {
    const key = `${strategy.from}-to-${strategy.to}`;
    this.strategies.set(key, strategy);
  }
  
  migrate<T, U>(versionFrom: string, versionTo: string, data: T): U {
    const key = `${versionFrom}-to-${versionTo}`;
    const strategy = this.strategies.get(key);
    
    if (!strategy) {
      throw new Error(`Миграция ${versionFrom} -> ${versionTo} не найдена`);
    }
    
    return strategy.migrate(data) as U;
  }
  
  migrateThrough<T, U>(startVersion: string, endVersion: string, data: T): U {
    // Простой пример для последовательной миграции
    // В реальности может потребоваться более сложный граф зависимостей
    if (startVersion === 'v1' && endVersion === 'v3') {
      const v2Data = this.migrate<UserV1, UserV2>('v1', 'v2', data as any);
      return this.migrate<UserV2, UserV3>('v2', 'v3', v2Data) as any;
    }
    
    return this.migrate(startVersion, endVersion, data) as U;
  }
}

// Использование
const migrationManager = new MigrationManager();
migrationManager.register(v1ToV2);
migrationManager.register(v2ToV3);

const v1User: UserV1 = { id: 1, name: 'John Doe', email: 'john@example.com' };
const v3User = migrationManager.migrateThrough<UserV1, UserV3>('v1', 'v3', v1User);
console.log(v3User);
```

## 6. Паттерн "Типобезопасная конфигурация" (Type-Safe Configuration Pattern)

```typescript
// Паттерн для создания типобезопасной конфигурации
interface ConfigDefinition<T> {
  [K: string]: ConfigProperty<T[K]>;
}

interface ConfigProperty<T> {
  defaultValue: T;
  validator?: (value: any) => value is T;
  description?: string;
  deprecated?: boolean;
}

class ConfigBuilder<T extends Record<string, any>> {
  private definition: ConfigDefinition<T> = {} as ConfigDefinition<T>;
  
  addProperty<K extends keyof T>(
    key: K,
    defaultValue: T[K],
    options?: {
      validator?: (value: any) => value is T[K];
      description?: string;
      deprecated?: boolean;
    }
  ): this {
    this.definition[key] = {
      defaultValue,
      validator: options?.validator,
      description: options?.description,
      deprecated: options?.deprecated
    };
    return this;
  }
  
  build(initialValues?: Partial<T>): Configuration<T> {
    return new Configuration(this.definition, initialValues);
  }
}

class Configuration<T extends Record<string, any>> {
  private values: Partial<T> = {};
  
  constructor(
    private definition: ConfigDefinition<T>,
    initialValues?: Partial<T>
  ) {
    // Установка начальных значений
    if (initialValues) {
      for (const [key, value] of Object.entries(initialValues)) {
        this.set(key as keyof T, value as T[keyof T]);
      }
    }
    
    // Установка значений по умолчанию
    for (const [key, prop] of Object.entries(this.definition)) {
      if (!(key in this.values)) {
        this.values[key] = prop.defaultValue;
      }
    }
  }
  
  set<K extends keyof T>(key: K, value: T[K]): void {
    const prop = this.definition[key];
    
    if (prop.validator && !prop.validator(value)) {
      throw new Error(`Неверное значение для ${String(key)}`);
    }
    
    if (prop.deprecated) {
      console.warn(`Конфигурация ${String(key)} устарела`);
    }
    
    this.values[key] = value;
  }
  
  get<K extends keyof T>(key: K): T[K] {
    return this.values[key]!;
  }
  
  getAll(): T {
    return { ...this.values } as T;
  }
  
  validate(): { valid: true } | { valid: false; errors: string[] } {
    const errors: string[] = [];
    
    for (const [key, prop] of Object.entries(this.definition)) {
      const value = this.values[key as keyof T];
      
      if (prop.validator && !prop.validator(value)) {
        errors.push(`Неверное значение для ${key}`);
      }
    }
    
    return errors.length === 0 
      ? { valid: true } 
      : { valid: false, errors };
  }
}

// Использование
interface AppConfig {
  apiUrl: string;
  timeout: number;
  retries: number;
  debug: boolean;
  allowedDomains: string[];
}

const configBuilder = new ConfigBuilder<AppConfig>()
  .addProperty('apiUrl', 'https://api.example.com', {
    validator: (value: any): value is string => typeof value === 'string' && value.startsWith('https://'),
    description: 'Базовый URL API'
  })
  .addProperty('timeout', 5000, {
    validator: (value: any): value is number => typeof value === 'number' && value > 0,
    description: 'Таймаут запросов в миллисекундах'
  })
  .addProperty('retries', 3, {
    validator: (value: any): value is number => typeof value === 'number' && value >= 0,
    description: 'Количество попыток запроса'
  })
  .addProperty('debug', false, {
    description: 'Режим отладки'
  })
  .addProperty('allowedDomains', ['example.com'], {
    validator: (value: any): value is string[] => 
      Array.isArray(value) && value.every(item => typeof item === 'string'),
    description: 'Разрешенные домены'
  });

const config = configBuilder.build({
  apiUrl: 'https://myapi.com',
  timeout: 10000
});

console.log(config.get('apiUrl')); // 'https://myapi.com'
console.log(config.get('timeout')); // 10000
console.log(config.get('retries')); // 3 (default)
```

## 7. Паттерн "Типобезопасный DSL" (Type-Safe DSL Pattern)

```typescript
// Создание типобезопасного DSL для валидации
type ValidationRule<T> = (value: T) => { isValid: boolean; error?: string };

interface ValidationDSL<T> {
  required: () => ValidationRule<T>;
  minLength: (length: number) => ValidationRule<T extends string ? T : never>;
  maxLength: (length: number) => ValidationRule<T extends string ? T : never>;
  min: (min: number) => ValidationRule<T extends number ? T : never>;
  max: (max: number) => ValidationRule<T extends number ? T : never>;
  pattern: (regex: RegExp) => ValidationRule<T extends string ? T : never>;
  custom: (validator: ValidationRule<T>) => ValidationRule<T>;
}

class ValidationBuilder<T> implements ValidationDSL<T> {
  required(): ValidationRule<T> {
    return (value: T) => {
      if (value === undefined || value === null || value === '') {
        return { isValid: false, error: 'Поле обязательно' };
      }
      return { isValid: true };
    };
  }
  
  minLength(length: number): ValidationRule<string> {
    return (value: string) => {
      if (value.length < length) {
        return { isValid: false, error: `Минимум ${length} символов` };
      }
      return { isValid: true };
    };
  }
  
  maxLength(length: number): ValidationRule<string> {
    return (value: string) => {
      if (value.length > length) {
        return { isValid: false, error: `Максимум ${length} символов` };
      }
      return { isValid: true };
    };
  }
  
  min(min: number): ValidationRule<number> {
    return (value: number) => {
      if (value < min) {
        return { isValid: false, error: `Минимум ${min}` };
      }
      return { isValid: true };
    };
  }
  
  max(max: number): ValidationRule<number> {
    return (value: number) => {
      if (value > max) {
        return { isValid: false, error: `Максимум ${max}` };
      }
      return { isValid: true };
    };
  }
  
  pattern(regex: RegExp): ValidationRule<string> {
    return (value: string) => {
      if (!regex.test(value)) {
        return { isValid: false, error: 'Неверный формат' };
      }
      return { isValid: true };
    };
  }
  
  custom(validator: ValidationRule<T>): ValidationRule<T> {
    return validator;
  }
}

// Применение валидации к объекту
interface UserForm {
  name: string;
  age: number;
  email: string;
}

class FormValidator<T> {
  private rules: { [K in keyof T]?: ValidationRule<T[K]>[] } = {};
  
  field<K extends keyof T>(field: K) {
    return new FieldValidator<T, K>(field, this);
  }
  
  validate(data: T): { isValid: boolean; errors: { [K in keyof T]?: string[] } } {
    const errors: { [K in keyof T]?: string[] } = {};
    let isValid = true;
    
    for (const [fieldName, fieldRules] of Object.entries(this.rules) as [keyof T, ValidationRule<T[keyof T]>[]][]) {
      const value = data[fieldName];
      const fieldErrors: string[] = [];
      
      for (const rule of fieldRules) {
        const result = rule(value);
        if (!result.isValid && result.error) {
          fieldErrors.push(result.error);
          isValid = false;
        }
      }
      
      if (fieldErrors.length > 0) {
        errors[fieldName] = fieldErrors;
      }
    }
    
    return { isValid, errors };
  }
}

class FieldValidator<T, K extends keyof T> {
  constructor(
    private field: K,
    private validator: FormValidator<T>
  ) {}
  
  validate(...rules: ValidationRule<T[K]>[]): FormValidator<T> {
    if (!this.validator.rules[this.field]) {
      this.validator.rules[this.field] = [];
    }
    
    (this.validator.rules[this.field] as ValidationRule<T[K]>[]).push(...rules);
    return this.validator;
  }
}

// Использование
const validator = new FormValidator<UserForm>();
const validationBuilder = new ValidationBuilder<UserForm[keyof UserForm]>();

validator
  .field('name')
  .validate(
    validationBuilder.required(),
    validationBuilder.minLength(2),
    validationBuilder.maxLength(50)
  )
  .field('age')
  .validate(
    validationBuilder.required(),
    validationBuilder.min(18),
    validationBuilder.max(120)
  )
  .field('email')
  .validate(
    validationBuilder.required(),
    validationBuilder.pattern(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)
  );

const formData: UserForm = { name: 'John', age: 25, email: 'john@example.com' };
const result = validator.validate(formData);
console.log(result); // { isValid: true, errors: {} }
```

## 8. Паттерн "Типобезопасный EventBus" (Type-Safe Event Bus Pattern)

```typescript
// Типобезопасная система событий
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: { message: string; code: number };
  data_update: { table: string; recordId: number };
}

type EventKey = keyof EventMap;
type EventData<T extends EventKey> = EventMap[T];

type EventListener<T extends EventKey> = (data: EventData<T>) => void;

class TypedEventBus {
  private listeners: {
    [K in EventKey]?: EventListener<K>[];
  } = {};
  
  on<T extends EventKey>(event: T, listener: EventListener<T>): () => void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    (this.listeners[event] as EventListener<T>[]).push(listener);
    
    // Возвращаем функцию для отписки
    return () => {
      const eventListeners = this.listeners[event];
      if (eventListeners) {
        const index = eventListeners.indexOf(listener as EventListener<any>);
        if (index > -1) {
          eventListeners.splice(index, 1);
        }
      }
    };
  }
  
  emit<T extends EventKey>(event: T, data: EventData<T>): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }
  
  once<T extends EventKey>(event: T, listener: EventListener<T>): void {
    const unsubscribe = this.on(event, (data) => {
      listener(data);
      unsubscribe();
    });
  }
  
  off<T extends EventKey>(event: T, listener: EventListener<T>): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      const index = eventListeners.indexOf(listener as EventListener<any>);
      if (index > -1) {
        eventListeners.splice(index, 1);
      }
    }
  }
}

// Использование
const eventBus = new TypedEventBus();

// Подписка на события
const unsubscribeLogin = eventBus.on('user_login', (data) => {
  console.log(`Пользователь ${data.userId} вошёл в ${data.timestamp}`);
});

eventBus.on('error', (data) => {
  console.error(`Ошибка ${data.code}: ${data.message}`);
});

// Генерация событий
eventBus.emit('user_login', { 
  userId: 123, 
  timestamp: new Date() 
});

eventBus.emit('error', { 
  message: 'Что-то пошло не так', 
  code: 500 
});

// Отписка
unsubscribeLogin();
```

## 9. Паттерн "Типобезопасный State Machine" (Type-Safe State Machine Pattern)

```typescript
// Типобезопасная машина состояний
interface StateMachineConfig<
  States extends string,
  Events extends { type: string }
> {
  initial: States;
  states: {
    [S in States]: {
      on?: {
        [T in Events['type']]?: States;
      };
    };
  };
  onTransition?: (from: States, to: States, event: Events) => void;
}

class TypedStateMachine<
  States extends string,
  Events extends { type: string }
> {
  private state: States;
  
  constructor(private config: StateMachineConfig<States, Events>) {
    this.state = config.initial;
  }
  
  getState(): States {
    return this.state;
  }
  
  canHandle(eventType: Events['type']): boolean {
    const transitions = this.config.states[this.state].on;
    return !!(transitions && eventType in transitions);
  }
  
  handle(event: Events): boolean {
    const transitions = this.config.states[this.state].on;
    
    if (transitions && event.type in transitions) {
      const newState = transitions[event.type] as States;
      
      if (this.config.onTransition) {
        this.config.onTransition(this.state, newState, event);
      }
      
      this.state = newState;
      return true;
    }
    
    return false;
  }
  
  // Метод для получения всех возможных переходов из текущего состояния
  getPossibleTransitions(): Events['type'][] {
    const transitions = this.config.states[this.state].on;
    return transitions ? Object.keys(transitions) as Events['type'][] : [];
  }
}

// Использование
type LightStates = 'green' | 'yellow' | 'red';
type LightEvents = 
  | { type: 'TIMER_TICK' }
  | { type: 'EMERGENCY_STOP' }
  | { type: 'RESET' };

const lightStateMachine = new TypedStateMachine<LightStates, LightEvents>({
  initial: 'red',
  states: {
    green: {
      on: {
        'TIMER_TICK': 'yellow'
      }
    },
    yellow: {
      on: {
        'TIMER_TICK': 'red'
      }
    },
    red: {
      on: {
        'TIMER_TICK': 'green',
        'EMERGENCY_STOP': 'red',
        'RESET': 'red'
      }
    }
  },
  onTransition: (from, to, event) => {
    console.log(`Переход из ${from} в ${to} по событию ${event.type}`);
  }
});

// Использование
console.log(lightStateMachine.getState()); // 'red'
lightStateMachine.handle({ type: 'TIMER_TICK' });
console.log(lightStateMachine.getState()); // 'green'
lightStateMachine.handle({ type: 'TIMER_TICK' });
console.log(lightStateMachine.getState()); // 'yellow'

// Проверка возможных переходов
console.log(lightStateMachine.getPossibleTransitions()); // ['TIMER_TICK']
```

## 10. Паттерн "Типобезопасный Dependency Injection Container" (Type-Safe DI Container Pattern)

```typescript
// Типобезопасный контейнер внедрения зависимостей
interface Token<T> {
  __token__: string;
  __type__: T;
}

function createToken<T>(name: string): Token<T> {
  return { __token__: name, __type__: undefined as any } as Token<T>;
}

interface ServiceDefinition<T> {
  token: Token<T>;
  useFactory?: () => T;
  useClass?: new (...args: any[]) => T;
  useValue?: T;
  deps?: Token<any>[];
  singleton?: boolean;
}

class DIContainer {
  private services = new Map<string, ServiceDefinition<any>>();
  private instances = new Map<string, any>();
  
  register<T>(definition: ServiceDefinition<T>): void {
    this.services.set(definition.token.__token__, definition);
  }
  
  resolve<T>(token: Token<T>): T {
    const definition = this.services.get(token.__token__);
    
    if (!definition) {
      throw new Error(`Сервис ${token.__token__} не зарегистрирован`);
    }
    
    // Если это синглтон и инстанс уже создан
    if (definition.singleton && this.instances.has(token.__token__)) {
      return this.instances.get(token.__token__);
    }
    
    let instance: T;
    
    if (definition.useValue !== undefined) {
      instance = definition.useValue;
    } else if (definition.useClass) {
      if (definition.deps) {
        const dependencies = definition.deps.map(dep => this.resolve(dep));
        instance = new definition.useClass(...dependencies);
      } else {
        instance = new definition.useClass();
      }
    } else if (definition.useFactory) {
      instance = definition.useFactory();
    } else {
      throw new Error(`Неверное определение сервиса для ${token.__token__}`);
    }
    
    if (definition.singleton) {
      this.instances.set(token.__token__, instance);
    }
    
    return instance;
  }
}

// Пример использования
interface Logger {
  log(message: string): void;
}

interface Database {
  query(sql: string): Promise<any>;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

class PostgreSQLDatabase implements Database {
  async query(sql: string): Promise<any> {
    console.log(`Выполнение запроса: ${sql}`);
    return [];
  }
}

// Создание токенов
const LOGGER_TOKEN = createToken<Logger>('logger');
const DATABASE_TOKEN = createToken<Database>('database');

// Создание контейнера и регистрация сервисов
const container = new DIContainer();

container.register({
  token: LOGGER_TOKEN,
  useClass: ConsoleLogger,
  singleton: true
});

container.register({
  token: DATABASE_TOKEN,
  useClass: PostgreSQLDatabase,
  deps: [LOGGER_TOKEN], // Зависимость от логгера
  singleton: true
});

// Использование
const logger = container.resolve(LOGGER_TOKEN);
const database = container.resolve(DATABASE_TOKEN);

logger.log('Приложение запущено');
database.query('SELECT * FROM users');
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// ОШИБКА: Слишком широкие типы
// function processData(data: any): any { return data; }

// ЛУЧШЕ: Конкретные типы
function processData<T>(data: T): T { return data; }

// ОШИБКА: Неправильное использование дженериков
// function badExample<T>(items: T[]): T[] { return items; }

// ЛУЧШЕ: Правильное ограничение дженериков
function goodExample<T extends { id: number }>(items: T[]): T[] { 
  return items; 
}
```

### 2. Лучшие практики

```typescript
// 1. Используйте branded types для предотвращения путаницы
type UserId = Brand<number, 'UserId'>;

// 2. Создавайте переиспользуемые утилиты типов
type NonEmptyArray<T> = [T, ...T[]];

// 3. Используйте conditional types для сложных преобразований
type Awaited<T> = T extends Promise<infer U> ? U : T;

// 4. Документируйте сложные типы
/**
 * Тип для представления результата асинхронной операции
 * @template T Тип успешного результата
 * @template E Тип ошибки
 */
type AsyncResult<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };
```

## Лучшие практики

1. **Используйте branded types** - для предотвращения путаницы между схожими типами
2. **Создавайте типобезопасные DSL** - для описания сложной логики
3. **Применяйте паттерн "Builder"** - для создания сложных объектов
4. **Используйте дискриминированные объединения** - для безопасной работы с разными типами данных
5. **Создавайте типобезопасные системы событий** - для надежной коммуникации между компонентами
6. **Документируйте сложные типы** - особенно когда они используют условные или сопоставленные типы

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - основа для многих продвинутых паттернов
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - для преобразования типов
- [[Утилиты типов TypeScript|Утилиты типов]] - для создания производных типов
- [[ts/type-system/type-inference|Вывод типов]] - как TypeScript определяет типы в паттернах
- [[ts/generics/TS Generics|Обобщения]] - основа для универсальных паттернов