# Architectural Patterns

Архитектурные паттерны в TypeScript описывают структурные решения организации приложений на высоком уровне. Они определяют крупномасштабные компоненты, их роли и отношения, а также способы взаимодействия между ними.

## Основные архитектурные паттерны

### [Model-View-Controller (MVC)](mvc.md)
Разделение приложения на три компонента: модель, представление и контроллер

### [Model-View-ViewModel (MVVM)](mvvm.md)
Шаблон, связывающий модель данных с представлением через ViewModel

### [Model-View-Presenter (MVP)](mvp.md)
Паттерн, где презентер управляет логикой представления

### [Clean Architecture](clean-architecture.md)
Архитектура с чистыми слоями и независимостью от внешних факторов

### [Hexagonal Architecture (Ports and Adapters)](hexagonal.md)
Архитектура, изолирующая бизнес-логику от внешних зависимостей

### [Layered Architecture (n-tier)](layered.md)
Многоуровневая архитектура с четким разделением уровней

### [Microkernel Architecture](microkernel.md)
Архитектура с минимальным ядром и расширяемой функциональностью

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Architectural Patterns                           │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Clean         │               │     Layered              │                                    │   Hexagonal     │
  │  Architecture   │               │  Architecture            │                                    │  Architecture   │
  │                 │               │                          │                                    │                 │
  │ - Entities      │◄──────────────┤ - Presentation Layer     │─────────────────────────────────────►│ - Core Domain   │
  │ - Use Cases     │               │ - Business Logic Layer   │                                    │ - Ports         │
  │ - Interfaces    │               │ - Data Access Layer      │                                    │ - Adapters      │
  │ - Frameworks    │               │ - Persistence Layer      │                                    │ - Independent   │
  │ - Dependency    │               │ - Separation of Concerns │                                    │   Business Logic│
  │   Rule          │               │ - Loose Coupling         │                                    └─────────────────┘
  └─────────────────┘               └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │       MVC / MVVM / MVP          │
        │                           │                               │
        │                           │ - Presentation Layer          │
        │                           │ - Business Logic Layer        │
        │                           │ - Data Layer                  │
        │                           │ - Separation of Concerns      │
        │                           │ - Testability                 │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                         Microservices                             │
        │                           │                                                                 │
        │                           │ - Service Decomposition                                           │
        │                           │ - API Gateway                                                     │
        │                           │ - Service Discovery                                               │
        │                           │ - Load Balancing                                                  │
        │                           │ - Circuit Breakers                                                │
        │                           │ - Event Driven Communication                                      │
        │                           │ - Data Consistency Patterns                                       │
        └───────────────────────────┤ - Resilience & Fault Tolerance                                    │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Clean Architecture

### Основные принципы
```typescript
// Внешние слои (зависимости указывают внутрь)
// ┌─────────────────────────────────────────────────────────┐
// │                    Frameworks & Drivers               │
// │  (UI, DB, Web Framework, External Services)           │
// └───────────────────┬─────────────────────────────────────┘
//                     │
//                     ▼
// ┌─────────────────────────────────────────────────────────┐
// │                      Interface Adapters                │
// │ (Controllers, Presenters, Gateways, Data Mappers)      │
// └───────────────────┬─────────────────────────────────────┘
//                     │
//                     ▼
// ┌─────────────────────────────────────────────────────────┐
// │                    Application Business Rules         │
// │                 (Input/Output Boundaries)             │
// └───────────────────┬─────────────────────────────────────┘
//                     │
//                     ▼
// ┌─────────────────────────────────────────────────────────┐
// │                     Enterprise Business Rules          │
// │                        (Entities)                      │
// └─────────────────────────────────────────────────────────┘

// 1. Внешние слои зависят от внутренних, а не наоборот (Dependency Rule)
// 2. Внутренние слои не знают ничего о внешних

// Слой сущностей (Enterprise Business Rules)
interface UserEntity {
  id: string;
  email: string;
  password: string;
  createdAt: Date;
  validate(): boolean;
}

class User implements UserEntity {
  id: string;
  email: string;
  password: string;
  createdAt: Date;
  
  constructor(email: string, password: string) {
    this.id = this.generateId();
    this.email = email;
    this.password = password;
    this.createdAt = new Date();
  }
  
  validate(): boolean {
    return this.email.includes('@') && this.password.length >= 8;
  }
  
  private generateId(): string {
    // бизнес-логика генерации id
    return Math.random().toString(36);
  }
}
```

### Слой использования (Application Business Rules)
```typescript
// Интерфейсы для входных/выходных границ
interface InputBoundary<TRequest, TResponse> {
  execute(request: TRequest): Promise<TResponse>;
}

interface OutputBoundary<T> {
  present(response: T): void;
}

interface UserRepository {
  save(user: User): Promise<void>;
  findByEmail(email: string): Promise<User | null>;
}

// DTO для передачи данных
interface RegisterUserRequest {
  email: string;
  password: string;
}

interface RegisterUserResponse {
  success: boolean;
  userId?: string;
  error?: string;
}

// Use case
class RegisterUserUseCase implements InputBoundary<RegisterUserRequest, RegisterUserResponse> {
  constructor(private userRepository: UserRepository) {}
  
  async execute(request: RegisterUserRequest): Promise<RegisterUserResponse> {
    try {
      // Валидация ввода
      if (!request.email || !request.password) {
        return { success: false, error: "Email and password are required" };
      }
      
      // Проверка существования пользователя
      const existingUser = await this.userRepository.findByEmail(request.email);
      if (existingUser) {
        return { success: false, error: "User already exists" };
      }
      
      // Создание нового пользователя
      const user = new User(request.email, request.password);
      if (!user.validate()) {
        return { success: false, error: "Invalid user data" };
      }
      
      // Сохранение
      await this.userRepository.save(user);
      
      return { success: true, userId: user.id };
    } catch (error) {
      return { success: false, error: (error as Error).message };
    }
  }
}
```

### Интерфейсы адаптеров
```typescript
// Интерфейс адаптера (порта)
interface UserPresenter extends OutputBoundary<RegisterUserResponse> {
  present(response: RegisterUserResponse): void;
}

interface UserGateway {
  register(request: RegisterUserRequest): Promise<RegisterUserResponse>;
}

// Реализация адаптера
class UserControllerAdapter implements UserGateway {
  constructor(
    private useCase: InputBoundary<RegisterUserRequest, RegisterUserResponse>,
    private presenter: OutputBoundary<RegisterUserResponse>
  ) {}
  
  async register(request: RegisterUserRequest): Promise<RegisterUserResponse> {
    const response = await this.useCase.execute(request);
    this.presenter.present(response);
    return response;
  }
}

class UserPresenterAdapter implements UserPresenter {
  present(response: RegisterUserResponse): void {
    if (response.success) {
      console.log(`User registered successfully with ID: ${response.userId}`);
    } else {
      console.error(`Registration failed: ${response.error}`);
    }
  }
}
```

### Внешние адаптеры (фреймворки и драйверы)
```typescript
// Внешняя реализация репозитория (Framework & Drivers)
class DatabaseUserRepository implements UserRepository {
  constructor(private database: any) {} // внешняя зависимость
  
  async save(user: User): Promise<void> {
    // Сохранение в базу данных
    await this.database.users.insert({
      id: user.id,
      email: user.email,
      password: user.password,
      createdAt: user.createdAt
    });
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const userData = await this.database.users.findOne({ email });
    if (!userData) return null;
    
    const user = new User(userData.email, userData.password);
    user.id = userData.id;
    user.createdAt = new Date(userData.createdAt);
    return user;
  }
}

// Использование в главном приложении
class AppCompositionRoot {
  static create(): UserControllerAdapter {
    // Внешние зависимости
    const database = { /* инициализация базы данных */ };
    const userRepository = new DatabaseUserRepository(database);
    
    // Промежуточные зависимости
    const useCase = new RegisterUserUseCase(userRepository);
    const presenter = new UserPresenterAdapter();
    
    // Адаптер (внешний слой)
    return new UserControllerAdapter(useCase, presenter);
  }
}

const controller = AppCompositionRoot.create();
controller.register({ email: "user@example.com", password: "securepassword" });
```

## MVC Pattern

### Классический MVC
```typescript
// Model
interface UserModel {
  email: string;
  password: string;
  authenticated: boolean;
}

class UserMVC {
  private model: UserModel = {
    email: "",
    password: "",
    authenticated: false
  };
  
  setEmail(email: string): void {
    this.model.email = email;
  }
  
  setPassword(password: string): void {
    this.model.password = password;
  }
  
  authenticate(): boolean {
    // Логика аутентификации
    this.model.authenticated = this.model.email.includes('@') && 
                              this.model.password.length >= 8;
    return this.model.authenticated;
  }
  
  getData(): UserModel {
    return { ...this.model }; // возвращаем копию
  }
}

// View
interface View {
  render(data: any): void;
  bindEvents(controller: Controller): void;
}

class UserView implements View {
  private container: HTMLElement;
  
  constructor(containerId: string) {
    this.container = document.getElementById(containerId)!;
  }
  
  render(data: UserModel): void {
    this.container.innerHTML = `
      <div>
        <h3>User: ${data.authenticated ? 'Authenticated' : 'Not authenticated'}</h3>
        <p>Email: ${data.email}</p>
      </div>
    `;
  }
  
  bindEvents(controller: Controller): void {
    const form = this.container.querySelector('form');
    if (form) {
      form.addEventListener('submit', (e) => {
        e.preventDefault();
        const emailInput = form.querySelector('input[name="email"]') as HTMLInputElement;
        const passwordInput = form.querySelector('input[name="password"]') as HTMLInputElement;
        
        controller.handleLogin(emailInput.value, passwordInput.value);
      });
    }
  }
}

// Controller
interface Controller {
  handleLogin(email: string, password: string): void;
}

class UserController implements Controller {
  constructor(
    private model: UserMVC,
    private view: View
  ) {
    this.view.bindEvents(this);
  }
  
  handleLogin(email: string, password: string): void {
    this.model.setEmail(email);
    this.model.setPassword(password);
    
    const authenticated = this.model.authenticate();
    
    if (authenticated) {
      console.log('Login successful');
      this.view.render(this.model.getData());
    } else {
      console.log('Login failed');
      // показать ошибку
    }
  }
}

// Использование
const userMVC = new UserMVC();
const userView = new UserView('user-container');
const userController = new UserController(userMVC, userView);
```

## MVVM Pattern

### Model-View-ViewModel
```typescript
// Model
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// ViewModel
class ProductViewModel {
  // Observable свойства
  private _name: string = '';
  private _price: number = 0;
  private _inStock: boolean = false;
  
  // Команды
  private _saveCommand: () => void;
  private _deleteCommand: () => void;
  
  constructor(private productService: ProductService) {
    this._saveCommand = this.save.bind(this);
    this._deleteCommand = this.delete.bind(this);
  }
  
  // Геттеры и сеттеры с уведомлением об изменениях
  get name(): string { return this._name; }
  set name(value: string) { 
    this._name = value;
    this.onPropertyChanged('name');
  }
  
  get price(): number { return this._price; }
  set price(value: number) { 
    this._price = value;
    this.onPropertyChanged('price');
    this.onPropertyChanged('priceFormatted');
  }
  
  get inStock(): boolean { return this._inStock; }
  set inStock(value: boolean) { 
    this._inStock = value;
    this.onPropertyChanged('inStock');
    this.onPropertyChanged('availabilityText');
  }
  
  get priceFormatted(): string {
    return `$${this._price.toFixed(2)}`;
  }
  
  get availabilityText(): string {
    return this._inStock ? 'In Stock' : 'Out of Stock';
  }
  
  get saveCommand(): () => void { return this._saveCommand; }
  get deleteCommand(): () => void { return this._deleteCommand; }
  
  // Событие изменения свойства
  private onPropertyChanged(property: string): void {
    // Уведомить View об изменении
    this.notifyView(property);
  }
  
  private notifyView(property: string): void {
    // Реализация уведомления View (обычно через биндинг)
    console.log(`Property ${property} changed in ViewModel`);
  }
  
  private async save(): Promise<void> {
    const product: Product = {
      id: 0, // будет установлен на бэкенде
      name: this._name,
      price: this._price,
      inStock: this._inStock
    };
    
    try {
      await this.productService.save(product);
      console.log('Product saved successfully');
    } catch (error) {
      console.error('Error saving product:', error);
    }
  }
  
  private async delete(): Promise<void> {
    if (confirm('Are you sure you want to delete this product?')) {
      try {
        // удаление продукта
        console.log('Product deleted');
      } catch (error) {
        console.error('Error deleting product:', error);
      }
    }
  }
}

// Сервис для работы с моделью
interface ProductService {
  save(product: Product): Promise<void>;
  delete(id: number): Promise<void>;
  getById(id: number): Promise<Product>;
}

class ApiProductService implements ProductService {
  async save(product: Product): Promise<void> {
    // API вызов
    console.log('Saving product to API:', product);
  }
  
  async delete(id: number): Promise<void> {
    // API вызов
    console.log('Deleting product from API:', id);
  }
  
  async getById(id: number): Promise<Product> {
    // API вызов
    return { id, name: 'Sample Product', price: 99.99, inStock: true };
  }
}
```

## Hexagonal Architecture

### Ports and Adapters
```typescript
// Port (интерфейс, определяемый доменом)
interface OrderProcessingService {
  placeOrder(order: Order): Promise<OrderResult>;
  cancelOrder(orderId: string): Promise<boolean>;
  getOrderStatus(orderId: string): Promise<OrderStatus>;
}

// Primary Adapter (Driving Adapter)
class OrderController {
  constructor(private service: OrderProcessingService) {}
  
  async handlePlaceOrder(req: any, res: any): Promise<void> {
    try {
      const order: Order = req.body;
      const result = await this.service.placeOrder(order);
      res.json(result);
    } catch (error) {
      res.status(500).json({ error: (error as Error).message });
    }
  }
  
  async handleCancelOrder(orderId: string, res: any): Promise<void> {
    try {
      const success = await this.service.cancelOrder(orderId);
      res.json({ success });
    } catch (error) {
      res.status(500).json({ error: (error as Error).message });
    }
  }
}

// Domain Model
interface Order {
  id: string;
  items: OrderItem[];
  totalAmount: number;
  status: 'pending' | 'processing' | 'completed' | 'cancelled';
  createdAt: Date;
}

interface OrderItem {
  productId: string;
  quantity: number;
  price: number;
}

interface OrderResult {
  success: boolean;
  orderId?: string;
  error?: string;
}

type OrderStatus = 'pending' | 'processing' | 'completed' | 'cancelled';

// Secondary Adapter (Driven Adapter)
class PaymentAdapter {
  async processPayment(order: Order): Promise<boolean> {
    // Внешний адаптер для платежной системы
    console.log('Processing payment for order:', order.id);
    return true; // упрощенно
  }
}

class InventoryAdapter {
  async reserveItems(items: OrderItem[]): Promise<boolean> {
    // Внешний адаптер для системы инвентаря
    console.log('Reserving inventory for items:', items);
    return true; // упрощенно
  }
  
  async releaseItems(items: OrderItem[]): Promise<void> {
    // Освобождение зарезервированных товаров
    console.log('Releasing reserved items:', items);
  }
}

class NotificationAdapter {
  async sendConfirmation(order: Order): Promise<void> {
    // Внешний адаптер для уведомлений
    console.log('Sending confirmation for order:', order.id);
  }
}

// Use Case Implementation (Domain Service)
class OrderProcessingServiceImpl implements OrderProcessingService {
  constructor(
    private paymentAdapter: PaymentAdapter,
    private inventoryAdapter: InventoryAdapter,
    private notificationAdapter: NotificationAdapter
  ) {}
  
  async placeOrder(order: Order): Promise<OrderResult> {
    // Проверка заказа
    if (order.items.length === 0) {
      return { success: false, error: "Order must have items" };
    }
    
    if (order.totalAmount <= 0) {
      return { success: false, error: "Order must have positive amount" };
    }
    
    try {
      // Резервирование инвентаря
      const inventoryReserved = await this.inventoryAdapter.reserveItems(order.items);
      if (!inventoryReserved) {
        return { success: false, error: "Insufficient inventory" };
      }
      
      // Обработка платежа
      const paymentProcessed = await this.paymentAdapter.processPayment(order);
      if (!paymentProcessed) {
        // Освобождение инвентаря при ошибке платежа
        await this.inventoryAdapter.releaseItems(order.items);
        return { success: false, error: "Payment processing failed" };
      }
      
      // Уведомление пользователя
      await this.notificationAdapter.sendConfirmation(order);
      
      // Обновление статуса заказа
      order.status = 'processing';
      
      return { success: true, orderId: order.id };
    } catch (error) {
      // Освобождение инвентаря при ошибке
      try {
        await this.inventoryAdapter.releaseItems(order.items);
      } catch {
        // Игнорировать ошибку освобождения, если основная ошибка была
      }
      
      return { success: false, error: (error as Error).message };
    }
  }
  
  async cancelOrder(orderId: string): Promise<boolean> {
    // Логика отмены заказа
    console.log(`Cancelling order: ${orderId}`);
    return true; // упрощенно
  }
  
  async getOrderStatus(orderId: string): Promise<OrderStatus> {
    // Логика проверки статуса заказа
    console.log(`Getting status for order: ${orderId}`);
    return 'pending'; // упрощенно
  }
}

// Composition Root
class HexagonalApp {
  static create(): OrderController {
    // Внешние адаптеры (драйверы)
    const paymentAdapter = new PaymentAdapter();
    const inventoryAdapter = new InventoryAdapter();
    const notificationAdapter = new NotificationAdapter();
    
    // Ядро приложения (Use Case)
    const orderService = new OrderProcessingServiceImpl(
      paymentAdapter,
      inventoryAdapter,
      notificationAdapter
    );
    
    // Входной адаптер
    const controller = new OrderController(orderService);
    
    return controller;
  }
}
```

## Микросервисная архитектура

### Пример микросервиса
```typescript
// Domain
interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
}

// API Gateway
interface UserService {
  createUser(userData: { email: string; name: string }): Promise<{ id: string }>;
  getUserById(id: string): Promise<User>;
  updateUser(id: string, updates: Partial<User>): Promise<boolean>;
  deleteUser(id: string): Promise<boolean>;
}

// User Service (микросервис)
class UserMicroservice {
  private users: Map<string, User> = new Map();
  
  async createUser(userData: { email: string; name: string }): Promise<{ id: string }> {
    const id = this.generateId();
    const user: User = {
      id,
      email: userData.email,
      name: userData.name,
      createdAt: new Date()
    };
    
    this.users.set(id, user);
    return { id };
  }
  
  async getUserById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }
  
  async updateUser(id: string, updates: Partial<User>): Promise<boolean> {
    const user = this.users.get(id);
    if (!user) return false;
    
    const updatedUser = { ...user, ...updates };
    this.users.set(id, updatedUser);
    return true;
  }
  
  async deleteUser(id: string): Promise<boolean> {
    return this.users.delete(id);
  }
  
  private generateId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}

// Service Registry (упрощенная имитация)
interface ServiceRegistry {
  register(serviceName: string, endpoint: string): void;
  lookup(serviceName: string): string;
}

class InMemoryRegistry implements ServiceRegistry {
  private services = new Map<string, string>();
  
  register(serviceName: string, endpoint: string): void {
    this.services.set(serviceName, endpoint);
  }
  
  lookup(serviceName: string): string {
    const endpoint = this.services.get(serviceName);
    if (!endpoint) {
      throw new Error(`Service ${serviceName} not found`);
    }
    return endpoint;
  }
}

// Load Balancer
interface LoadBalancer {
  getEndpoint(serviceName: string): string;
}

class RoundRobinBalancer implements LoadBalancer {
  private instances: Map<string, string[]> = new Map();
  
  addInstance(serviceName: string, endpoint: string): void {
    const current = this.instances.get(serviceName) || [];
    this.instances.set(serviceName, [...current, endpoint]);
  }
  
  getEndpoint(serviceName: string): string {
    const endpoints = this.instances.get(serviceName);
    if (!endpoints || endpoints.length === 0) {
      throw new Error(`No instances available for ${serviceName}`);
    }
    
    // Упрощённый round-robin (в реальности нужно отслеживать состояние инстансов)
    return endpoints[0]; // в реальности нужно вращать
  }
}

// Circuit Breaker
enum CircuitState { CLOSED, OPEN, HALF_OPEN }

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failureCount: number = 0;
  private lastFailure: Date | null = null;
  
  constructor(
    private failureThreshold: number = 5,
    private timeout: number = 60000 // 60 секунд
  ) {}
  
  async call<T>(operation: () => Promise<T>): Promise<T> {
    if (this.shouldOpenCircuit()) {
      throw new Error('Circuit breaker is OPEN');
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private shouldOpenCircuit(): boolean {
    return this.state === CircuitState.OPEN;
  }
  
  private onSuccess(): void {
    this.failureCount = 0;
    this.lastFailure = null;
    this.state = CircuitState.CLOSED;
  }
  
  private onFailure(): void {
    this.failureCount++;
    this.lastFailure = new Date();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = CircuitState.OPEN;
    }
  }
  
  private reset(): void {
    this.state = CircuitState.HALF_OPEN;
    // В реальности нужно проверять, готов ли сервис
  }
}

// Client for microservice
class UserServiceClient implements UserService {
  constructor(
    private registry: ServiceRegistry,
    private loadBalancer: LoadBalancer,
    private circuitBreaker: CircuitBreaker
  ) {}
  
  async createUser(userData: { email: string; name: string }): Promise<{ id: string }> {
    const endpoint = this.loadBalancer.getEndpoint('user-service');
    
    return this.circuitBreaker.call(async () => {
      // Фактический вызов внешнего сервиса (например, через HTTP)
      console.log(`Calling ${endpoint} to create user`);
      return { id: Math.random().toString(36).substr(2, 9) }; // имитация
    });
  }
  
  async getUserById(id: string): Promise<User> {
    const endpoint = this.loadBalancer.getEndpoint('user-service');
    
    return this.circuitBreaker.call(async () => {
      console.log(`Calling ${endpoint} to get user ${id}`);
      // имитация вызова HTTP сервиса
      return {
        id,
        email: 'user@example.com',
        name: 'Test User',
        createdAt: new Date()
      };
    });
  }
  
  async updateUser(id: string, updates: Partial<User>): Promise<boolean> {
    const endpoint = this.loadBalancer.getEndpoint('user-service');
    console.log(`Calling ${endpoint} to update user ${id}`);
    return true; // имитация
  }
  
  async deleteUser(id: string): Promise<boolean> {
    const endpoint = this.loadBalancer.getEndpoint('user-service');
    console.log(`Calling ${endpoint} to delete user ${id}`);
    return true; // имитация
  }
}

// Использование в системе
const registry = new InMemoryRegistry();
registry.register('user-service', 'http://user-service:3000');

const balancer = new RoundRobinBalancer();
balancer.addInstance('user-service', 'http://user-service-1:3000');
balancer.addInstance('user-service', 'http://user-service-2:3000');

const circuitBreaker = new CircuitBreaker(3, 30000); // 3 ошибки, 30 секунд таймаут

const userService = new UserServiceClient(registry, balancer, circuitBreaker);
```

## Лучшие практики архитектурных паттернов

### 1. Разделение ответственностей
```typescript
// Хорошо: каждый слой имеет свои обязанности
interface UserRepository {  // Слой данных
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
}

interface UserService {     // Слой бизнес-логики  
  createUser(email: string, password: string): Promise<User>;
  authenticate(email: string, password: string): Promise<boolean>;
}

interface UserController {  // Слой представления/интерфейса
  handleCreateUser(req: any, res: any): Promise<void>;
  handleLogin(req: any, res: any): Promise<void>;
}
```

### 2. Слабая связанность через интерфейсы
```typescript
// Плохо: жесткая зависимость
class UserService {
  private dbAdapter = new PostgreSQLAdapter(); // жесткая зависимость
}

// Хорошо: зависимость через интерфейс
class UserService {
  constructor(private dbAdapter: DatabaseAdapter) {} // слабая связанность
}
```

### 3. Инверсия зависимостей
```typescript
// Внешний слой определяет зависимости для внутреннего
interface DatabaseAdapter {
  query(sql: string): Promise<any[]>;
}

// Внешний адаптер (внешний слой)
class PostgreSQLAdapter implements DatabaseAdapter {
  async query(sql: string): Promise<any[]> {
    // PostgreSQL специфичная реализация
    return [];
  }
}

// Внешний адаптер (внешний слой)
class MongoDBAdapter implements DatabaseAdapter {
  async query(sql: string): Promise<any[]> {
    // MongoDB специфичная реализация
    return [];
  }
}

// Внутренний слой (бизнес-логика) зависит от абстракции
class BusinessService {
  constructor(private db: DatabaseAdapter) {} // зависит от абстракции, не от деталей
}
```

## Современные тенденции

### Composition Root Pattern
```typescript
// Единая точка композиции всех зависимостей
class CompositionRoot {
  static create(): UserService {
    // Внешние зависимости
    const database = new PostgreSQLAdapter();
    const emailService = new SMTPService();
    const logger = new ConsoleLogger();
    
    // Промежуточные сервисы
    const userRepository = new DatabaseUserRepository(database, logger);
    const emailValidator = new StandardEmailValidator();
    
    // Основной сервис
    return new UserService(
      userRepository,
      emailValidator,
      emailService
    );
  }
}

// Использование в точке входа
const app = CompositionRoot.create();
```

### Feature-Sliced Design
```typescript
// Архитектура, ориентированная на фичи
// features/
//   ├── user-profile/
//   │   ├── model/
//   │   │   ├── types.ts
//   │   │   ├── api.ts
//   │   │   └── index.ts
//   │   ├── ui/
//   │   │   ├── UserProfile.tsx
//   │   │   └── index.ts
//   │   ├── lib/
//   │   │   ├── utils.ts
//   │   │   └── constants.ts
//   │   └── index.ts
//   └── auth/
//       ├── model/
//       └── ui/

// Каждая фича содержит свои слои (model, ui, lib)
interface UserFeatureModel {
  // Внутренние типы фичи
  state: { user?: User; loading: boolean };
  
  // Логика фичи
  actions: {
    loadUser(): Promise<void>;
    updateUser(updates: Partial<User>): Promise<void>;
  };
  
  // Сервисы фичи
  services: {
    api: UserService;
    storage: StorageService;
  };
}
```

## Ошибки и подводные камни

### 1. Зависимости от внешних деталей
```typescript
// ПЛОХО: внутренний слой зависит от внешнего
interface UserService {
  // Зависит от HTTP специфики
  getUser(req: HttpRequest): HttpResponse;
}

// ХОРОШО: абстракции независимы от внешних деталей
interface UserService {
  getUser(id: string): Promise<User>;
}
```

### 2. Неправильное разделение слоев
```typescript
// ПЛОХО: данные и представление перемешаны
class UserControllerWithDB {
  async handleGetUser(req: any, res: any) {
    // Прямой SQL запрос в контроллере
    const result = await db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
    res.json(result);
  }
}

// ХОРОШО: каждый слой делает свою работу
class UserController {
  constructor(private userService: UserService) {}
  
  async handleGetUser(req: any, res: any) {
    const user = await this.userService.getUserById(req.params.id);
    res.json(user);
  }
}

class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async getUserById(id: string) {
    return await this.userRepository.findById(id);
  }
}
```

## Заключение

Архитектурные паттерны в TypeScript:

1. **Обеспечивают масштабируемость** - приложение может расти без усложнения
2. **Повышают тестируемость** - слабая связанность упрощает тестирование
3. **Улучшают сопровождаемость** - четкое разделение ответственностей
4. **Обеспечивают независимость** - бизнес-логика не зависит от реализации
5. **Облегчают параллельную разработку** - разработчики могут работать в своих слоях

Выбор архитектурного паттерна зависит от:

- Размера и сложности приложения
- Команды разработчиков
- Требований к производительности и масштабируемости
- Сроков разработки
- Планов по развитию системы

Каждый паттерн имеет свои преимущества и компромиссы, и выбор должен основываться на конкретных требованиях проекта.

## Связь с другими концепциями
- [[../../dependency-injection/Внедрение-зависимостей]] - внедрение зависимостей в архитектуре
- [[../modules/architecture]] - модульная архитектура
- [[../design-patterns/structural-patterns]] - структурные паттерны в архитектуре
- [[architecture/testing/Архитектура Тестирования]] - тестирование архитектурных слоев
- [[../ecosystem/microservices]] - микроархитектура