# Связи между паттернами TypeScript

## Архитектурная карта связей между паттернами

```
                    ┌─────────────────────────────────────────────────────────┐
                    │         Pattern Relations in TypeScript               │
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
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │       Functional Patterns       │
        │                           │                               │
        │                           │ - Pure Functions                │
        │                           │ - Immutability                  │
        │                           │ - Composition                   │
        │                           │ - Functor/Monad Patterns        │
        │                           │ - Currying & Partial Application│
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                         Architectural Patterns                      │
        │                           │                                                                 │
        │                           │ - Clean Architecture                                              │
        │                           │ - Hexagonal Architecture                                          │
        │                           │ - MVC/MVVM/MVP                                                  │
        │                           │ - Microservices                                                 │
        │                           │ - Layered Architecture                                            │
        └───────────────────────────┤ - Dependency Injection/HOI                                        │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Подробные связи между паттернами

### 1. Creational ↔ Structural Patterns
```typescript
// Factory + Decorator
interface Component {
  render(): string;
}

class ConcreteComponent implements Component {
  render(): string {
    return "Base component";
  }
}

// Декоратор как создаваемый через фабрику
interface ComponentDecorator extends Component {
  component: Component;
}

class LoggingDecorator implements ComponentDecorator {
  constructor(public component: Component) {}
  
  render(): string {
    const result = this.component.render();
    console.log("Rendering component:", result);
    return result;
  }
}

// Фабрика с декорированием
class ComponentFactory {
  static create(type: 'basic' | 'logged'): Component {
    const component = new ConcreteComponent();
    
    if (type === 'logged') {
      return new LoggingDecorator(component);
    }
    
    return component;
  }
}

const basicComponent = ComponentFactory.create('basic');    // Component
const loggedComponent = ComponentFactory.create('logged');  // Component (with logging)
```

### 2. Creational ↔ Behavioral Patterns
```typescript
// Factory + Strategy
interface PaymentStrategy {
  pay(amount: number): boolean;
}

class CreditCardStrategy implements PaymentStrategy {
  pay(amount: number): boolean {
    console.log(`Paying ${amount} with credit card`);
    return true;
  }
}

class PayPalStrategy implements PaymentStrategy {
  pay(amount: number): boolean {
    console.log(`Paying ${amount} with PayPal`);
    return true;
  }
}

// Фабрика стратегий
class PaymentStrategyFactory {
  static create(type: 'credit' | 'paypal'): PaymentStrategy {
    switch (type) {
      case 'credit':
        return new CreditCardStrategy();
      case 'paypal':
        return new PayPalStrategy();
      default:
        throw new Error(`Unknown payment type: ${type}`);
    }
  }
}

// Контекст стратегии
class PaymentProcessor {
  private strategy: PaymentStrategy;
  
  constructor(strategyType: 'credit' | 'paypal') {
    this.strategy = PaymentStrategyFactory.create(strategyType);
  }
  
  processPayment(amount: number): boolean {
    return this.strategy.pay(amount);
  }
}
```

### 3. Structural ↔ Behavioral Patterns
```typescript
// Observer + Decorator
class Subject {
  private observers: Array<(data: any) => void> = [];
  
  attach(observer: (data: any) => void): void {
    this.observers.push(observer);
  }
  
  notify(data: any): void {
    this.observers.forEach(observer => observer(data));
  }
}

// Декоратор для логирования уведомлений
class LoggingSubject extends Subject {
  constructor(private subject: Subject) {
    super();
  }
  
  notify(data: any): void {
    console.log("Before notification:", data);
    this.subject.notify(data);
    console.log("After notification:", data);
  }
}

// Использование
const subject = new Subject();
subject.attach(data => console.log("Received:", data));

const loggingSubject = new LoggingSubject(subject);
loggingSubject.notify({ message: "Hello" });
```

## Интеграция с функциональным программированием

### 4. OOP Patterns ↔ Functional Patterns
```typescript
// Функциональный подход к стратегии
type PaymentFunction = (amount: number) => boolean;

const creditCardPayment: PaymentFunction = (amount: number) => {
  console.log(`Paying ${amount} with credit card (FP)`);
  return true;
};

const paypalPayment: PaymentFunction = (amount: number) => {
  console.log(`Paying ${amount} with PayPal (FP)`);
  return true;
};

// Фабрика функций
const createPaymentProcessor = (strategy: PaymentFunction) => ({
  processPayment: (amount: number): boolean => strategy(amount)
});

// Использование
const creditProcessor = createPaymentProcessor(creditCardPayment);
creditProcessor.processPayment(100); // "Paying 100 with credit card (FP)"
```

### 5. Functional + Behavioral Patterns
```typescript
// Функциональный Observer
type Observer<T> = (value: T) => void;
type Subscription = () => void;

class Observable<T> {
  private observers: Observer<T>[] = [];
  
  subscribe(observer: Observer<T>): Subscription {
    this.observers.push(observer);
    
    // Возвращаем функцию отписки
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  notify(value: T): void {
    this.observers.forEach(observer => observer(value));
  }
  
  // Композиция наблюдателей (функциональный подход)
  pipe<U>(transform: (value: T) => U): Observable<U> {
    const newObservable = new Observable<U>();
    this.subscribe(value => newObservable.notify(transform(value)));
    return newObservable;
  }
}

// Использование
const subject = new Observable<string>();
const subscription = subject.subscribe(value => console.log("Observer 1:", value));
subject.subscribe(value => console.log("Observer 2:", value.toUpperCase()));

subject.notify("hello"); // "Observer 1: hello", "Observer 2: HELLO"

// Отписка
subscription();
subject.notify("world"); // только "Observer 2: WORLD"

// Цепочка трансформаций
const transformed = subject.pipe(value => value.length);
transformed.subscribe(length => console.log("Length:", length)); // "Length: 5" for "hello"
```

## Архитектурные интеграции

### 6. Архитектура + Порождающие паттерны
```typescript
// Clean Architecture + Factory
interface UserRepository {
  findById(id: string): Promise<User | null>;
}

interface UserUseCase {
  execute(id: string): Promise<User | null>;
}

// Фабрика Use Case с внедрением зависимостей
class UseCaseFactory {
  static createUserUseCase(repository: UserRepository): UserUseCase {
    return new GetUserUseCase(repository);
  }
}

class GetUserUseCase implements UserUseCase {
  constructor(private repository: UserRepository) {}
  
  async execute(id: string): Promise<User | null> {
    return await this.repository.findById(id);
  }
}

// Использование в контроллере (Presentation layer)
class UserController {
  private useCase: UserUseCase;
  
  constructor(userRepository: UserRepository) {
    this.useCase = UseCaseFactory.createUserUseCase(userRepository);
  }
  
  async handleGetUser(id: string) {
    return await this.useCase.execute(id);
  }
}
```

### 7. Архитектура + Структурные паттерны
```typescript
// Hexagonal Architecture + Adapter
interface OrderPort {
  placeOrder(order: Order): Promise<boolean>;
}

// Primary Adapter (Driving)
class OrderController {
  constructor(private orderService: OrderPort) {}
  
  async handlePlaceOrder(orderData: any) {
    const order = this.mapToDomain(orderData);
    return await this.orderService.placeOrder(order);
  }
  
  private mapToDomain(data: any): Order {
    // преобразование данных запроса в доменную модель
    return new Order(data);
  }
}

// Secondary Adapter (Driven)
class DatabaseOrderAdapter implements OrderPort {
  async placeOrder(order: Order): Promise<boolean> {
    // сохранение заказа в базу данных
    console.log("Saving order to database:", order.id);
    return true;
  }
}

// Использование в Composition Root
const databaseAdapter = new DatabaseOrderAdapter();
const controller = new OrderController(databaseAdapter);
```

### 8. Архитектура + Поведенческие паттерны
```typescript
// Clean Architecture + State Pattern
interface OrderState {
  process(): OrderState;
  cancel(): OrderState;
  ship(): OrderState;
}

class PendingState implements OrderState {
  process(): OrderState { return new ProcessingState(); }
  cancel(): OrderState { return new CancelledState(); }
  ship(): OrderState { throw new Error("Cannot ship pending order"); }
}

class ProcessingState implements OrderState {
  process(): OrderState { throw new Error("Order already processing"); }
  cancel(): OrderState { return new CancelledState(); }
  ship(): OrderState { return new ShippedState(); }
}

class ShippedState implements OrderState {
  process(): OrderState { throw new Error("Already shipped"); }
  cancel(): OrderState { throw new Error("Cannot cancel shipped order"); }
  ship(): OrderState { throw new Error("Already shipped"); }
}

class CancelledState implements OrderState {
  process(): OrderState { throw new Error("Cannot process cancelled order"); }
  cancel(): OrderState { throw new Error("Already cancelled"); }
  ship(): OrderState { throw new Error("Cannot ship cancelled order"); }
}

// Доменная сущность
class Order {
  private state: OrderState = new PendingState();
  
  process(): void {
    this.state = this.state.process();
  }
  
  cancel(): void {
    this.state = this.state.cancel();
  }
  
  ship(): void {
    this.state = this.state.ship();
  }
}

// Использование в Use Case
class ProcessOrderUseCase {
  async execute(order: Order): Promise<void> {
    order.process();
    // дополнительная бизнес-логика
  }
}
```

## Функциональные архитектурные паттерны

### 9. Функциональное ядро + ООП обертки
```typescript
// Функциональная логика
interface User {
  id: string;
  email: string;
  active: boolean;
}

const validateEmail = (email: string): boolean => 
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

const createUser = (email: string): User | null => {
  if (!validateEmail(email)) {
    return null;
  }
  
  return {
    id: Math.random().toString(36),
    email,
    active: true
  };
};

const activateUser = (user: User): User => ({
  ...user,
  active: true
});

// ООП обертка для инфраструктуры
class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async createUser(email: string): Promise<User | null> {
    const user = createUser(email);
    if (user) {
      await this.userRepository.save(user);
      return user;
    }
    return null;
  }
  
  async activateUser(userId: string): Promise<boolean> {
    const user = await this.userRepository.findById(userId);
    if (user && !user.active) {
      const activatedUser = activateUser(user);
      await this.userRepository.update(activatedUser);
      return true;
    }
    return false;
  }
}
```

## Практические примеры интеграции

### 10. Комплексная система с несколькими паттернами
```typescript
// Использование нескольких паттернов в одной системе
interface Notification {
  id: string;
  type: string;
  content: string;
  recipient: string;
}

// Стратегия отправки уведомлений
interface NotificationStrategy {
  send(notification: Notification): Promise<boolean>;
}

class EmailStrategy implements NotificationStrategy {
  async send(notification: Notification): Promise<boolean> {
    console.log(`Sending email to ${notification.recipient}: ${notification.content}`);
    return true;
  }
}

class SmsStrategy implements NotificationStrategy {
  async send(notification: Notification): Promise<boolean> {
    console.log(`Sending SMS to ${notification.recipient}: ${notification.content}`);
    return true;
  }
}

// Фабрика стратегий
class NotificationStrategyFactory {
  static create(type: string): NotificationStrategy {
    switch (type) {
      case 'email': return new EmailStrategy();
      case 'sms': return new SmsStrategy();
      default: throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Наблюдатель за событиями уведомлений
interface NotificationObserver {
  onNotificationSent(notification: Notification, success: boolean): void;
}

class LoggingObserver implements NotificationObserver {
  onNotificationSent(notification: Notification, success: boolean): void {
    console.log(`Notification ${success ? 'sent' : 'failed'}: ${notification.id}`);
  }
}

// Компоновщик уведомлений
interface NotificationComponent {
  send(): Promise<boolean>;
}

class SingleNotification implements NotificationComponent {
  constructor(
    private notification: Notification,
    private strategy: NotificationStrategy,
    private observers: NotificationObserver[] = []
  ) {}
  
  async send(): Promise<boolean> {
    const success = await this.strategy.send(this.notification);
    
    this.observers.forEach(obs => 
      obs.onNotificationSent(this.notification, success)
    );
    
    return success;
  }
}

class NotificationGroup implements NotificationComponent {
  private components: NotificationComponent[] = [];
  
  add(component: NotificationComponent): void {
    this.components.push(component);
  }
  
  async send(): Promise<boolean> {
    const results = await Promise.all(
      this.components.map(comp => comp.send())
    );
    return results.every(result => result);
  }
}

// Использование
const notification: Notification = {
  id: "notif-1",
  type: "email",
  content: "Welcome aboard!",
  recipient: "user@example.com"
};

// Создание стратегии через фабрику
const strategy = NotificationStrategyFactory.create(notification.type);

// Создание компонента с наблюдателями
const observer = new LoggingObserver();
const notificationComponent = new SingleNotification(notification, strategy, [observer]);

// Отправка уведомления
await notificationComponent.send();
```

## Современные паттерны интеграции

### 11. Dependency Injection + Functional Patterns
```typescript
// Функциональные сервисы с DI
interface HttpService {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
}

const createHttpService = (baseUrl: string, headers?: Record<string, string>): HttpService => ({
  async get<T>(url: string): Promise<T> {
    const response = await fetch(`${baseUrl}${url}`, { 
      headers: { ...headers, 'Content-Type': 'application/json' } 
    });
    return response.json();
  },
  
  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(`${baseUrl}${url}`, {
      method: 'POST',
      headers: { ...headers, 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
});

// Контейнер зависимостей
interface Container {
  get<T>(name: string): T;
  register<T>(name: string, factory: () => T): void;
}

class DIContainer implements Container {
  private services = new Map<string, () => any>();
  
  register<T>(name: string, factory: () => T): void {
    this.services.set(name, factory);
  }
  
  get<T>(name: string): T {
    const factory = this.services.get(name);
    if (!factory) {
      throw new Error(`Service not registered: ${name}`);
    }
    return factory();
  }
}

// Использование
const container = new DIContainer();
container.register('httpService', () => createHttpService('https://api.example.com'));

const httpService = container.get<HttpService>('httpService');
```

### 12. Functional Reactive Programming + Architectural Patterns
```typescript
// FRP + Clean Architecture
interface Stream<T> {
  subscribe(observer: (value: T) => void): () => void;
  map<U>(fn: (value: T) => U): Stream<U>;
  filter(predicate: (value: T) => boolean): Stream<T>;
  scan<U>(fn: (acc: U, value: T) => U, initial: U): Stream<U>;
}

class ObservableStream<T> implements Stream<T> {
  private observers: Array<(value: T) => void> = [];
  
  constructor(private source?: Stream<T>) {
    if (source) {
      source.subscribe(value => this.notify(value));
    }
  }
  
  subscribe(observer: (value: T) => void): () => void {
    this.observers.push(observer);
    
    return () => {
      const index = this.observers.indexOf(observer);
      if (index > -1) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  map<U>(fn: (value: T) => U): Stream<U> {
    const mappedStream = new ObservableStream<U>();
    this.subscribe(value => mappedStream.notify(fn(value)));
    return mappedStream;
  }
  
  filter(predicate: (value: T) => boolean): Stream<T> {
    const filteredStream = new ObservableStream<T>();
    this.subscribe(value => {
      if (predicate(value)) {
        filteredStream.notify(value);
      }
    });
    return filteredStream;
  }
  
  scan<U>(fn: (acc: U, value: T) => U, initial: U): Stream<U> {
    const scannedStream = new ObservableStream<U>();
    let accumulator = initial;
    
    this.subscribe(value => {
      accumulator = fn(accumulator, value);
      scannedStream.notify(accumulator);
    });
    
    // Отправить начальное значение
    scannedStream.notify(initial);
    
    return scannedStream;
  }
  
  private notify(value: T): void {
    this.observers.forEach(observer => observer(value));
  }
}

// Использование в архитектуре
class UserActivityUseCase {
  private activityStream = new ObservableStream<{ userId: string; action: string }>();
  
  getUserActivityStream(userId: string): Stream<number> {
    return this.activityStream
      .filter(activity => activity.userId === userId)
      .scan((count, _) => count + 1, 0);
  }
  
  recordActivity(userId: string, action: string): void {
    this.activityStream.notify({ userId, action });
  }
}
```

## Лучшие практики интеграции

### 13. Сочетание парадигм
```typescript
// Гибридный подход: функциональное ядро + ООП обертки
// 1. Чистые функции для бизнес-логики
const calculateDiscount = (amount: number, customerType: string): number => {
  if (customerType === 'premium') return amount * 0.15;
  if (customerType === 'regular') return amount * 0.05;
  return 0;
};

const validateOrder = (order: Order): boolean => {
  return order.items.length > 0 && 
         order.total > 0 && 
         order.customerId !== '';
};

// 2. ООП для инфраструктуры и состояния
class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentService: PaymentService
  ) {}
  
  async processOrder(order: Order): Promise<boolean> {
    // Используем функциональную логику
    if (!validateOrder(order)) {
      throw new Error('Invalid order');
    }
    
    const discount = calculateDiscount(order.total, order.customer.type);
    const finalAmount = order.total - discount;
    
    // ООП логика для инфраструктуры
    const payment = await this.paymentService.charge(finalAmount);
    if (!payment.success) {
      return false;
    }
    
    await this.orderRepository.save(order);
    return true;
  }
}
```

## Подводные камни и ошибки

### 14. Переусложнение интеграции
```typescript
// ПЛОХО: избыточное использование паттернов
class OverlyComplexNotificationService {
  // Паттерн: Service Locator Anti-pattern
  private locator = new ServiceLocator();
  
  sendNotification(notification: Notification) {
    // Динамическое создание через reflection (плохо для TypeScript)
    const strategy = this.locator.get(
      `${notification.type.charAt(0).toUpperCase() + notification.type.slice(1)}Strategy`
    );
    
    // Необходимо приведение типов и проверки
    return (strategy as NotificationStrategy).send(notification);
  }
}

// ЛУЧШЕ: явное внедрение зависимостей с простыми паттернами
class SimpleNotificationService {
  constructor(
    private emailSender: EmailStrategy,
    private smsSender: SmsStrategy,
    private logger: LoggingObserver
  ) {}
  
  async sendNotification(notification: Notification): Promise<boolean> {
    const sender = notification.type === 'email' ? 
      this.emailSender : this.smsSender;
    
    const result = await sender.send(notification);
    this.logger.onNotificationSent(notification, result);
    return result;
  }
}
```

### 15. Неправильная граница между паттернами
```typescript
// ПЛОХО: смешение ответственностей
class BadOrderProcessor {
  // Нарушение Single Responsibility Principle
  processOrder(order: Order): boolean {
    // Логика валидации
    if (order.total <= 0) return false;
    
    // Логика отправки
    if (order.needsEmailConfirmation) {
      // код отправки email
    }
    
    // Логика сохранения
    // код сохранения в базу
    
    return true;
  }
}

// ХОРОШО: разделение ответственностей
class GoodOrderProcessor {
  constructor(
    private validator: OrderValidator,
    private notifier: NotificationService,
    private repository: OrderRepository
  ) {}
  
  async processOrder(order: Order): Promise<boolean> {
    if (!this.validator.validate(order)) {
      return false;
    }
    
    await this.repository.save(order);
    await this.notifier.sendConfirmation(order);
    
    return true;
  }
}
```

## Современные тенденции

### 16. Composition over Inheritance + Functional Patterns
```typescript
// Современный подход: композиция + функции
type OrderProcessor = (order: Order) => Promise<boolean>;

const createOrderProcessor = (
  validator: (order: Order) => boolean,
  repository: { save: (order: Order) => Promise<void> },
  notifier: { sendConfirmation: (order: Order) => Promise<void> }
): OrderProcessor => {
  return async (order: Order): Promise<boolean> => {
    if (!validator(order)) {
      return false;
    }
    
    await repository.save(order);
    await notifier.sendConfirmation(order);
    
    return true;
  };
};

// Создание специфичных обработчиков
const premiumOrderProcessor = createOrderProcessor(
  (order) => order.customer.type === 'premium' && order.total > 100,
  new DatabaseOrderRepository(),
  new EmailNotificationService()
);
```

## Заключение

Интеграция паттернов в TypeScript позволяет:

1. **Создавать гибкие архитектуры** - комбинируя разные подходы
2. **Обеспечивать тестируемость** - через слабую связанность
3. **Повышать переиспользуемость** - через модульность паттернов  
4. **Улучшать поддерживаемость** - через ясные границы ответственности
5. **Балансировать между парадигмами** - использовать лучшее от каждого подхода

Ключ к успешной интеграции:

- **Понимание целей каждого паттерна**
- **Выбор подходящих паттернов для контекста**
- **Избегание избыточной сложности**
- **Ясные границы ответственности**
- **Тестирование интеграций**

## Рекомендации по выбору комбинаций

| Сценарий | Рекомендуемые комбинации |
|----------|-------------------------|
| CRUD операции | Factory + Repository + DI |
| Асинхронные потоки | Observer + Functional Streams |
| Бизнес-логика | Strategy + State + Functional Core |
| API слой | Facade + Adapter + Observer |
| Компонентный UI | Composite + Decorator + State |
| Формы ввода | Command + Observer + Validation |
| Настройки | Abstract Factory + Singleton + Bridge |

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - совместимость типов при интеграции
- [[../modules/index]] - модульная организация паттернов
- [[../generics/advanced]] - обобщения для паттернов
- [[../functional-programming/index]] - функциональные альтернативы ООП паттернам
- [[Advanced TypeScript Types]] - продвинутые типы для паттернов