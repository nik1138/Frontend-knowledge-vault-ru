# Связи между паттернами проектирования TypeScript

## Архитектурная карта связей

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Design Pattern Relations                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │  Creational     │               │      Structural          │                                    │  Behavioral     │
  │  Patterns       │               │  Patterns                │                                    │  Patterns       │
  │                 │               │                          │                                    │                 │
  │ - Factory       │◄──────────────┤ - Adapter                │─────────────────────────────────────►│ - Observer      │
  │ - Singleton     │               │ - Decorator              │                                    │ - Strategy      │
  │ - Builder       │               │ - Composite              │                                    │ - Command       │
  │ - Abstract      │               │ - Facade                 │                                    │ - State         │
  │   Factory       │               │ - Proxy                  │                                    │ - Iterator      │
  │ - Prototype     │               │ - Bridge                 │                                    │ - Mediator      │
  │                 │               │ - Flyweight              │                                    │ - Memento       │
  │                 │               │                          │                                    │ - Visitor       │
  │                 │               │                          │                                    │ - Chain of Resp │
  │                 │               │                          │                                    │ - Template M.   │
  │                 │               │                          │                                    │ - Interpreter   │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │        Functional              │
        │                           │        Patterns               │
        │                           │                                 │
        │                           │ - Pure Functions                │
        │                           │ - Immutability                  │
        │                           │ - Higher-Order Functions        │
        │                           │ - Currying & Partial Application │
        │                           │ - Composition                   │
        │                           │ - Functors & Monads             │
        │                           │ - Reactive Programming          │
        │                           │ - Functional Data Structures    │
        └───────────────────────────┤ - Functional State Management   │
                                    └─────────────────────────────────┘

                    ┌─────────────────────────────────────────────────────────────────────────────────────────┐
                    │              Architectural Patterns                                                     │
                    │                                                                                         │
                    │ - Clean Architecture                                                                    │
                    │ - MVC/MVP/MVVM                                                                          │
                    │ - Layered Architecture                                                                  │
                    │ - Hexagonal Architecture                                                                │
                    │ - Microservices Architecture                                                            │
                    │ - Event-Driven Architecture                                                             │
                    │ - Component-Based Architecture                                                          │
                    │ - Domain-Driven Design                                                                  │
                    └─────────────────────────────────────────────────────────────────────────────────────────┘
```

## Подробные связи между паттернами

### 1. Creational ↔ Structural Patterns

#### Factory + Adapter
```typescript
// Factory создает адаптеры для различных типов
interface PaymentProcessor {
  process(amount: number): boolean;
}

interface LegacyPaymentProcessor {
  makePayment(amount: number): boolean;
}

class PaymentAdapter implements PaymentProcessor {
  constructor(private legacy: LegacyPaymentProcessor) {}
  
  process(amount: number): boolean {
    return this.legacy.makePayment(amount);
  }
}

class PaymentAdapterFactory {
  static create(type: 'paypal' | 'stripe' | 'legacy'): PaymentProcessor {
    switch (type) {
      case 'paypal':
        return new PaypalPaymentProcessor();
      case 'stripe':
        return new StripePaymentProcessor();
      case 'legacy':
        return new PaymentAdapter(new LegacyPaymentProcessorImpl());
    }
  }
}
```

#### Builder + Facade
```typescript
// Builder может использовать Facade для упрощения сложной конфигурации
class ComplexSystemFacade {
  private subsystems: Subsystem[] = [];
  
  addSubsystem(subsystem: Subsystem): void {
    this.subsystems.push(subsystem);
  }
  
  configure(): void {
    this.subsystems.forEach(sub => sub.initialize());
  }
}

class SystemBuilder {
  private facade: ComplexSystemFacade = new ComplexSystemFacade();
  
  addDatabase(): this {
    this.facade.addSubsystem(new DatabaseSubsystem());
    return this;
  }
  
  addCache(): this {
    this.facade.addSubsystem(new CacheSubsystem());
    return this;
  }
  
  addLogger(): this {
    this.facade.addSubsystem(new LoggerSubsystem());
    return this;
  }
  
  build(): ComplexSystemFacade {
    this.facade.configure();
    return this.facade;
  }
}

// Использование
const system = new SystemBuilder()
  .addDatabase()
  .addCache() 
  .addLogger()
  .build();
```

#### Singleton + Proxy
```typescript
// Singleton может управляться через Proxy для дополнительной логики
class DatabaseConnection {
  private static instance: DatabaseConnection;
  
  private constructor() {}
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
  
  query(sql: string): any[] {
    // выполнение запроса
    return [];
  }
}

// Proxy для добавления логирования/кеширования
class DatabaseConnectionProxy {
  private realConnection: DatabaseConnection;
  
  constructor() {
    this.realConnection = DatabaseConnection.getInstance();
  }
  
  query(sql: string): any[] {
    console.log(`Executing query: ${sql}`);
    const result = this.realConnection.query(sql);
    console.log(`Query executed successfully`);
    return result;
  }
}
```

### 2. Structural ↔ Behavioral Patterns

#### Decorator + Strategy
```typescript
// Декоратор может оборачивать стратегии
interface ValidationStrategy {
  validate(value: any): boolean;
}

class StringValidationStrategy implements ValidationStrategy {
  validate(value: any): boolean {
    return typeof value === 'string' && value.length > 0;
  }
}

class NumberValidationStrategy implements ValidationStrategy {
  validate(value: any): boolean {
    return typeof value === 'number' && !isNaN(value);
  }
}

// Декоратор для логирования валидаций
class LoggingValidationDecorator implements ValidationStrategy {
  constructor(private strategy: ValidationStrategy) {}
  
  validate(value: any): boolean {
    console.log(`Validating value: ${value}`);
    const result = this.strategy.validate(value);
    console.log(`Validation result: ${result}`);
    return result;
  }
}

// Использование
const stringValidator = new LoggingValidationDecorator(new StringValidationStrategy());
const numberValidator = new LoggingValidationDecorator(new NumberValidationStrategy());
```

#### Composite + Iterator
```typescript
// Композитный элемент может быть итерируемым
interface Component {
  operate(): void;
}

interface Iterator<T> {
  next(): { value: T; done: boolean };
  [Symbol.iterator](): Iterator<T>;
}

class Leaf implements Component {
  constructor(private name: string) {}
  
  operate(): void {
    console.log(`Operating on leaf: ${this.name}`);
  }
}

class Composite implements Component {
  private children: Component[] = [];
  
  add(component: Component): void {
    this.children.push(component);
  }
  
  remove(component: Component): void {
    const index = this.children.indexOf(component);
    if (index > -1) {
      this.children.splice(index, 1);
    }
  }
  
  operate(): void {
    for (const child of this) {
      child.operate();
    }
  }
  
  // Реализация итератора
  *[Symbol.iterator](): Iterator<Component> {
    for (const child of this.children) {
      yield child;
    }
  }
}
```

#### Adapter + Observer
```typescript
// Адаптер может преобразовать события из одной системы в другую
interface EventEmitter {
  on(event: string, listener: (...args: any[]) => void): void;
  emit(event: string, ...args: any[]): void;
}

interface ObserverPattern {
  subscribe(observer: Observer): void;
  notify(data: any): void;
}

class ObservableAdapter implements ObserverPattern {
  private emitter: EventEmitter;
  private observers: Observer[] = [];
  
  constructor(emitter: EventEmitter) {
    this.emitter = emitter;
    
    // Адаптируем события emitter к Observer паттерну
    this.emitter.on('data', (data: any) => {
      this.notify(data);
    });
  }
  
  subscribe(observer: Observer): void {
    this.observers.push(observer);
  }
  
  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }
}
```

### 3. Behavioral ↔ Behavioral Patterns

#### Observer + Command
```typescript
// Команды могут быть отправлены через систему событий
interface Command {
  execute(): void;
  undo?(): void;
}

class CommandQueue {
  private commands: Command[] = [];
  private observers: Array<(command: Command) => void> = [];
  
  addCommand(command: Command): void {
    this.commands.push(command);
    this.notifyObservers(command);
    command.execute();
  }
  
  subscribe(observer: (command: Command) => void): void {
    this.observers.push(observer);
  }
  
  private notifyObservers(command: Command): void {
    this.observers.forEach(observer => observer(command));
  }
  
  undoLast(): void {
    const lastCommand = this.commands.pop();
    if (lastCommand && lastCommand.undo) {
      lastCommand.undo();
    }
  }
}
```

#### Strategy + State
```typescript
// Состояния могут иметь разные стратегии поведения
interface PlayerState {
  play(): void;
  pause(): void;
  stop(): void;
}

class PlayingState implements PlayerState {
  private strategy: PlaybackStrategy;
  
  constructor(strategy: PlaybackStrategy) {
    this.strategy = strategy;
  }
  
  play(): void {
    console.log("Already playing");
  }
  
  pause(): void {
    console.log("Pausing playback...");
    this.changeState(new PausedState(this.strategy));
  }
  
  stop(): void {
    console.log("Stopping playback...");
    this.changeState(new StoppedState(this.strategy));
  }
  
  private changeState(newState: PlayerState): void {
    // Логика изменения состояния
  }
}

class PausedState implements PlayerState {
  constructor(private strategy: PlaybackStrategy) {}
  
  play(): void {
    console.log("Resuming playback...");
    // изменение состояния на PlayingState
  }
  
  pause(): void {
    console.log("Already paused");
  }
  
  stop(): void {
    console.log("Stopping playback...");
    // изменение состояния на StoppedState
  }
}

interface PlaybackStrategy {
  playNext(): void;
  playPrevious(): void;
  seek(position: number): void;
}
```

#### Template Method + Strategy
```typescript
// Шаблонный метод может использовать стратегии в своих шагах
abstract class DataProcessor {
  // Шаблонный метод
  process(data: any): any {
    this.validate(data);
    const transformed = this.transform(data);
    return this.save(transformed);
  }
  
  protected validate(data: any): void {
    if (!data) {
      throw new Error("Data is required");
    }
  }
  
  // Абстрактный шаг - может быть реализован стратегией
  protected abstract transform(data: any): any;
  
  // Шаг с возможностью замены стратегии
  protected save(data: any): any {
    const strategy = this.getSaveStrategy();
    return strategy.execute(data);
  }
  
  protected abstract getSaveStrategy(): SaveStrategy;
}

class JSONDataProcessor extends DataProcessor {
  protected transform(data: any): any {
    // конкретная реализация трансформации
    return JSON.stringify(data);
  }
  
  protected getSaveStrategy(): SaveStrategy {
    return new JSONSaveStrategy();
  }
}

interface SaveStrategy {
  execute(data: any): any;
}

class JSONSaveStrategy implements SaveStrategy {
  execute(data: any): any {
    // сохранение как JSON
    return data;
  }
}
```

## Функциональные паттерны и ООП

### 4. Functional ↔ OOP Patterns

#### Pure Functions + Singleton
```typescript
// Функциональные утилиты в синглтоне
class FunctionalUtils {
  private static instance: FunctionalUtils;
  
  private constructor() {}
  
  static getInstance(): FunctionalUtils {
    if (!FunctionalUtils.instance) {
      FunctionalUtils.instance = new FunctionalUtils();
    }
    return FunctionalUtils.instance;
  }
  
  // Функциональная утилита
  map<T, U>(array: T[], fn: (item: T) => U): U[] {
    return array.map(fn);
  }
  
  // Функция композиции
  compose<A, B, C>(f: (b: B) => C, g: (a: A) => B): (a: A) => C {
    return (x: A) => f(g(x));
  }
  
  // Чистые функции в "функциональном синглтоне"
  pipe<T>(value: T, ...fns: Array<(arg: any) => any>): any {
    return fns.reduce((acc, fn) => fn(acc), value);
  }
}
```

#### Currying + Strategy
```typescript
// Каррированные чистые функции как стратегии
type ValidationFn = (value: any) => boolean;

const lengthValidator = (min: number, max: number, value: string): boolean => {
  return value.length >= min && value.length <= max;
};

const curryLengthValidator = (min: number) => 
  (max: number) => 
    (value: string): boolean => lengthValidator(min, max, value);

// Создание конкретных стратегий через каррирование
const emailValidator: ValidationFn = (value: string) => 
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);

const nameValidator: ValidationFn = curryLengthValidator(2)(50);

// Использование в стратегическом паттерне
class ValidationContext {
  constructor(private strategy: ValidationFn) {}
  
  execute(value: any): boolean {
    return this.strategy(value);
  }
  
  setStrategy(strategy: ValidationFn): void {
    this.strategy = strategy;
  }
}

// Использование
const validator = new ValidationContext(emailValidator);
const isEmailValid = validator.execute("test@example.com");
```

#### Function Composition + Chain of Responsibility
```typescript
// Композиция функций как цепочка ответственности
type Handler<T> = (value: T) => T | null;

const addPrefix = (prefix: string) => (value: string): string => `${prefix}${value}`;
const validateEmail = (value: string): string | null => 
  value.includes('@') ? value : null;
const sanitizeInput = (value: string): string => value.trim();

// Цепочка обработчиков через композицию
const emailProcessingChain = (input: string): string | null =>
  [sanitizeInput, validateEmail].reduce((acc: string | null, handler) => {
    if (acc === null) return null;
    return handler(acc);
  }, input) as string | null;

// Использование
const result = emailProcessingChain("  test@example.com  "); // "test@example.com"
const invalid = emailProcessingChain("  invalid-email  "); // null
```

## Архитектурные паттерны и их связи

### 5. Architectural ↔ All Patterns

#### MVC + Observer
```typescript
// Модель использует Observer для уведомления представлений
class Subject {
  private observers: Array<(data: any) => void> = [];
  
  subscribe(observer: (data: any) => void): () => void {
    this.observers.push(observer);
    
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  notify(data: any): void {
    this.observers.forEach(observer => observer(data));
  }
}

class UserModel extends Subject {
  private data: any;
  
  setData(newData: any): void {
    this.data = newData;
    this.notify(newData);
  }
  
  getData(): any {
    return this.data;
  }
}

class UserView {
  constructor(model: UserModel) {
    model.subscribe((data) => {
      this.render(data);
    });
  }
  
  render(data: any): void {
    console.log("Rendering user data:", data);
  }
}
```

#### Clean Architecture + All Patterns
```typescript
// Пример интеграции нескольких паттернов в Clean Architecture

// Domain Layer - использует стратегии и сущности
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

class UserService {
  constructor(private repository: UserRepository) {}
  
  async getUserProfile(userId: string): Promise<UserProfile> {
    const user = await this.repository.findById(userId);
    if (!user) {
      throw new Error("User not found");
    }
    
    // Использование стратегии для форматирования профиля
    const formatter = this.getProfileFormatter(user.type);
    return formatter.format(user);
  }
  
  private getProfileFormatter(userType: string): ProfileFormatter {
    // Фабрика стратегий
    switch (userType) {
      case "admin": return new AdminProfileFormatter();
      case "regular": return new RegularProfileFormatter();
      default: return new DefaultProfileFormatter();
    }
  }
}

interface ProfileFormatter {
  format(user: User): UserProfile;
}

// Use Cases используют Command паттерн
interface UseCase<Request, Response> {
  execute(request: Request): Promise<Response>;
}

class UpdateUserProfileUseCase implements UseCase<UpdateUserRequest, UpdateUserResponse> {
  constructor(private service: UserService) {}
  
  async execute(request: UpdateUserRequest): Promise<UpdateUserResponse> {
    // Command pattern для выполнения
    const command = new UpdateUserCommand(request);
    await command.execute(this.service);
    return { success: true };
  }
}

class UpdateUserCommand {
  constructor(private request: UpdateUserRequest) {}
  
  async execute(service: UserService): Promise<void> {
    // логика команды
  }
}
```

## Практические примеры интеграции

### 6. Комплексная интеграция

```typescript
// Пример: комплексная система с несколькими паттернами

// 1. Creational: Factory + Builder
interface Notification {
  send(): void;
}

class EmailNotification implements Notification {
  constructor(private to: string, private subject: string, private body: string) {}
  
  send(): void {
    console.log(`Sending email to ${this.to}: ${this.subject}`);
  }
}

class PushNotification implements Notification {
  constructor(private deviceId: string, private message: string) {}
  
  send(): void {
    console.log(`Sending push to ${this.deviceId}: ${this.message}`);
  }
}

// Фабрика уведомлений
class NotificationFactory {
  create(type: 'email' | 'push', config: any): Notification {
    switch (type) {
      case 'email':
        return new EmailNotification(config.to, config.subject, config.body);
      case 'push':
        return new PushNotification(config.deviceId, config.message);
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Строитель уведомлений
class NotificationBuilder {
  private type: 'email' | 'push' = 'email';
  private config: any = {};
  
  setType(type: 'email' | 'push'): this {
    this.type = type;
    return this;
  }
  
  setEmailConfig(to: string, subject: string, body: string): this {
    this.config = { to, subject, body };
    return this;
  }
  
  setPushConfig(deviceId: string, message: string): this {
    this.config = { deviceId, message };
    return this;
  }
  
  build(): Notification {
    const factory = new NotificationFactory();
    return factory.create(this.type, this.config);
  }
}

// 2. Structural: Decorator для добавления функциональности
class NotificationDecorator implements Notification {
  constructor(protected notification: Notification) {}
  
  send(): void {
    this.notification.send();
  }
}

class LoggingNotificationDecorator extends NotificationDecorator {
  send(): void {
    console.log(`About to send notification...`);
    this.notification.send();
    console.log(`Notification sent`);
  }
}

class RetryNotificationDecorator extends NotificationDecorator {
  async send(): Promise<void> {
    let attempts = 0;
    const maxAttempts = 3;
    
    while (attempts < maxAttempts) {
      try {
        await this.notification.send(); // если send асинхронный
        return;
      } catch (error) {
        attempts++;
        if (attempts >= maxAttempts) {
          throw error;
        }
        // ждать перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, 1000 * attempts));
      }
    }
  }
}

// 3. Behavioral: Observer для отслеживания отправки
interface NotificationObserver {
  onNotificationSent(notification: Notification, success: boolean): void;
}

class AnalyticsObserver implements NotificationObserver {
  onNotificationSent(notification: Notification, success: boolean): void {
    console.log(`Analytics: Notification ${success ? 'successful' : 'failed'}`);
  }
}

class NotificationMediator {
  private observers: NotificationObserver[] = [];
  
  addObserver(observer: NotificationObserver): void {
    this.observers.push(observer);
  }
  
  notifyObservers(notification: Notification, success: boolean): void {
    this.observers.forEach(observer => observer.onNotificationSent(notification, success));
  }
}

// 4. Использование всего вместе
const notificationBuilder = new NotificationBuilder();
const notification = notificationBuilder
  .setType('email')
  .setEmailConfig('user@example.com', 'Welcome!', 'Welcome to our service!')
  .build();

const decoratedNotification = new LoggingNotificationDecorator(
  new RetryNotificationDecorator(notification)
);

const mediator = new NotificationMediator();
mediator.addObserver(new AnalyticsObserver());

try {
  decoratedNotification.send();
  mediator.notifyObservers(decoratedNotification, true);
} catch (error) {
  mediator.notifyObservers(decoratedNotification, false);
}
```

## Функциональные архитектурные паттерны

### 7. Functional Architecture
```typescript
// Функциональная архитектура с чистыми функциями
type DependencyContainer = {
  [key: string]: any;
};

type UseCase<TInput, TOutput> = (
  deps: DependencyContainer
) => (input: TInput) => Promise<TOutput>;

// Чистая бизнес-логика
const validateUser = (userData: any): boolean => {
  return userData.name && userData.email && userData.email.includes('@');
};

const createUserInDB = (db: any) => (userData: any): Promise<any> => {
  return db.users.insert(userData);
};

// Композиция функций для Use Case
const createUserUseCase: UseCase<{ name: string; email: string }, { id: string }> = 
  (deps) => async (input) => {
    if (!validateUser(input)) {
      throw new Error("Invalid user data");
    }
    
    const result = await createUserInDB(deps.db)(input);
    return { id: result.id };
  };

// Использование
const dependencies = {
  db: { /* database instance */ },
  logger: { /* logger instance */ }
};

const useCase = createUserUseCase(dependencies);
// const result = await useCase({ name: "Alice", email: "alice@example.com" });
```

## Современные интеграции

### 8. FRP + Traditional Patterns
```typescript
// Функциональное реактивное программирование с традиционными паттернами
import { Observable, Subject, BehaviorSubject } from 'rxjs';
import { map, filter, withLatestFrom, distinctUntilChanged } from 'rxjs/operators';

// Observable как Observer паттерн на стероидах
class StateManager<T> {
  private state$: BehaviorSubject<T>;
  private actions$ = new Subject<{ type: string; payload?: any }>();
  
  constructor(initialState: T) {
    this.state$ = new BehaviorSubject<T>(initialState);
    
    // Использование Command паттерна через Actions
    this.actions$.pipe(
      withLatestFrom(this.state$),
      map(([action, state]) => this.reduce(state, action))
    ).subscribe(newState => {
      this.state$.next(newState);
    });
  }
  
  getState(): Observable<T> {
    return this.state$.pipe(distinctUntilChanged());
  }
  
  dispatch(action: { type: string; payload?: any }): void {
    this.actions$.next(action);
  }
  
  private reduce(state: T, action: { type: string; payload?: any }): T {
    // как в Redux
    switch (action.type) {
      case 'UPDATE_FIELD':
        return { ...state, [action.payload.field]: action.payload.value } as T;
      default:
        return state;
    }
  }
}
```

## Лучшие практики интеграции

### 1. Используйте подходящий паттерн для задачи
- **Creational**: при создании объектов с сложной логикой
- **Structural**: для организации уже созданных объектов
- **Behavioral**: для определения взаимодействия между объектами
- **Functional**: для чистой логики и трансформаций данных

### 2. Избегайте чрезмерной композиции
```typescript
// ПЛОХО: излишняя сложность
class OverlyComplexSystem {
  // Factory + Builder + Abstract Factory + Prototype + Singleton
  // Decorator + Adapter + Facade + Proxy + Bridge + Composite + Flyweight  
  // Observer + Strategy + Command + State + Iterator + Mediator + Memento + Visitor
}

// ЛУЧШЕ: используйте только нужные паттерны
class WellDesignedSystem {
  // Только необходимые паттерны для решения задачи
}
```

### 3. Документируйте выбор паттернов
```typescript
/**
 * Используем Strategy + Factory для:
 * - Легкого переключения между алгоритмами валидации
 * - Возможности добавления новых стратегий без изменения клиентского кода
 * - Тестируемости каждой стратегии отдельно
 */
class ValidationService {
  // реализация с использованием подходящих паттернов
}
```

### 4. Тестируйте интеграции
```typescript
// Юнит-тесты для комбинированных паттернов
describe('Integrated Patterns', () => {
  test('Factory creates decorated objects correctly', () => {
    const factory = new NotificationFactory();
    const notification = factory.create('email', { to: 'test@example.com' });
    const decorated = new LoggingDecorator(notification);
    
    expect(decorated).toBeInstanceOf(NotificationDecorator);
  });
  
  test('Observer responds to state changes in Subject', () => {
    const subject = new Subject();
    const observerSpy = jest.fn();
    subject.subscribe(observerSpy);
    
    subject.notify("test data");
    
    expect(observerSpy).toHaveBeenCalledWith("test data");
  });
});
```

## Сравнение паттернов

| Категория | Когда использовать | Совместимость |
|-----------|-------------------|---------------|
| Creational | При создании объектов | Хорошо сочетаются с Structural |
| Structural | При организации объектов | Основа для Behavioral паттернов |
| Behavioral | При взаимодействии между объектами | Работают с любыми другими |
| Functional | При обработке данных/чистой логике | Совместимы со всеми, но требуют внимательности к состоянию |

## Заключение

Связи между паттернами проектирования в TypeScript:

1. **Усиливают друг друга** - комбинация паттернов создает более мощные решения
2. **Покрывают разные аспекты** - каждый уровень абстракции решает свою часть проблемы
3. **Обеспечивают гибкость** - комбинирование позволяет адаптироваться к изменениям
4. **Требуют баланса** - слишком много паттернов усложняет код
5. **Улучшают поддерживаемость** - при правильном использовании упрощают сопровождение

Ключ к успешной интеграции - понимание целей каждого паттерна и ситуации, где он наиболее эффективен. Паттерны должны служить решению проблемы, а не быть самоцелью.

## Связь с другими концепциями
- [[TypeScript Type System]] - как типы влияют на паттерны
- [[../generics/index]] - обобщения в паттернах
- [[Advanced TypeScript Types]] - продвинутые типы для паттернов
- [[Модули в TypeScript]] - организация паттернов в модули
- [[../ecosystem/architecture]] - архитектурные подходы
- [[Функциональное программирование в TypeScript]] - функциональные альтернативы ООП паттернам