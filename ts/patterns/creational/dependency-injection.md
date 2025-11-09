# Dependency Injection in TypeScript

> [!note] Смотрите также
> Для общего понимания концепции внедрения зависимостей, см. [[../../../concepts/dependency-injection/Внедрение-зависимостей|основную концепцию внедрения зависимостей]].

Внедрение зависимостей (Dependency Injection, DI) - это паттерн проектирования, при котором объекты получают свои зависимости извне, а не создают их самостоятельно. Это позволяет достичь слабой связанности, улучшить тестируемость и обеспечить гибкость архитектуры.

## Основные концепции

### Принцип инверсии зависимости
```typescript
// НАРУШЕНИЕ принципа инверсии зависимостей
class EmailService {
  send(email: string): void {
    console.log(`Sending email: ${email}`);
  }
}

class UserService {
  private emailService = new EmailService(); // Жесткая зависимость
  
  registerUser(email: string): void {
    // Регистрация пользователя
    this.emailService.send(`Welcome ${email}`);
  }
}

// ПРАВИЛЬНО: через интерфейс
interface EmailServiceInterface {
  send(email: string): void;
}

class ConcreteEmailService implements EmailServiceInterface {
  send(email: string): void {
    console.log(`Sending email: ${email}`);
  }
}

class UserServiceDI {
  constructor(private emailService: EmailServiceInterface) {} // Зависимость через конструктор
  
  registerUser(email: string): void {
    // Регистрация пользователя
    this.emailService.send(`Welcome ${email}`);
  }
}

// Использование
const emailService = new ConcreteEmailService();
const userService = new UserServiceDI(emailService);
```

### Типы внедрения зависимостей

#### 1. Через конструктор
```typescript
class OrderService {
  constructor(
    private paymentService: PaymentService,
    private inventoryService: InventoryService,
    private notificationService: NotificationService
  ) {}
  
  async processOrder(orderData: Order): Promise<OrderResult> {
    // Используем внедренные зависимости
    const paymentResult = await this.paymentService.processPayment(orderData.payment);
    if (!paymentResult.success) {
      return { success: false, error: paymentResult.error };
    }
    
    const inventoryResult = await this.inventoryService.reserveItems(orderData.items);
    if (!inventoryResult.success) {
      await this.paymentService.refund(paymentResult.transactionId);
      return { success: false, error: inventoryResult.error };
    }
    
    const notificationResult = await this.notificationService.sendConfirmation(orderData);
    return { success: true, orderId: orderData.id };
  }
}
```

#### 2. Через свойства
```typescript
class ReportService {
  private dataSource!: DataSource;
  private formatter!: Formatter;
  
  // Сеттеры для внедрения зависимостей
  setDataSource(ds: DataSource): void {
    this.dataSource = ds;
  }
  
  setFormatter(formatter: Formatter): void {
    this.formatter = formatter;
  }
  
  async generateReport(): Promise<string> {
    if (!this.dataSource || !this.formatter) {
      throw new Error("Dependencies not injected");
    }
    
    const data = await this.dataSource.getData();
    return this.formatter.format(data);
  }
}
```

#### 3. Через методы
```typescript
interface Configurable {
  configure(logger: Logger, cache: Cache): void;
}

class DataProcessor implements Configurable {
  private logger: Logger | null = null;
  private cache: Cache | null = null;
  
  configure(logger: Logger, cache: Cache): void {
    this.logger = logger;
    this.cache = cache;
  }
  
  process(data: any): any {
    if (this.logger) {
      this.logger.log("Processing data");
    }
    
    if (this.cache) {
      const cached = this.cache.get(data.key);
      if (cached) return cached;
    }
    
    const result = this.transform(data);
    
    if (this.cache) {
      this.cache.set(data.key, result);
    }
    
    return result;
  }
  
  private transform(data: any): any {
    // логика трансформации
    return data;
  }
}
```

## Реализация DI Container

### Простой DI контейнер
```typescript
interface Token {
  name: string;
}

// Класс для создания токенов (чтобы избежать коллизий строк)
class InjectionToken<T> implements Token {
  constructor(public name: string) {}
}

type Provider<T = any> = {
  token: Token;
  useFactory?: (...deps: any[]) => T;
  useClass?: new (...args: any[]) => T;
  useValue?: T;
  deps?: Token[];
  singleton?: boolean;
};

class DIContainer {
  private providers = new Map<Token, Provider>();
  private instances = new Map<Token, any>();
  
  register<T>(provider: Provider<T>): void {
    this.providers.set(provider.token, provider);
  }
  
  resolve<T>(token: Token): T {
    // Проверяем, есть ли уже инстанс для синглтона
    if (this.instances.has(token)) {
      return this.instances.get(token);
    }
    
    const provider = this.providers.get(token);
    if (!provider) {
      throw new Error(`No provider found for token: ${token.name}`);
    }
    
    let instance: T;
    
    if (provider.useValue !== undefined) {
      instance = provider.useValue;
    } else if (provider.useClass) {
      const dependencies = provider.deps?.map(dep => this.resolve(dep)) || [];
      instance = new provider.useClass(...dependencies);
    } else if (provider.useFactory) {
      const dependencies = provider.deps?.map(dep => this.resolve(dep)) || [];
      instance = provider.useFactory(...dependencies);
    } else {
      throw new Error(`Invalid provider for token: ${token.name}`);
    }
    
    // Сохраняем инстанс, если это синглтон
    if (provider.singleton) {
      this.instances.set(token, instance);
    }
    
    return instance;
  }
  
  // Проверка наличия провайдера
  has(token: Token): boolean {
    return this.providers.has(token);
  }
  
  // Очистка инстансов
  clear(): void {
    this.instances.clear();
  }
}

// Использование
const EMAIL_SERVICE = new InjectionToken<EmailServiceInterface>('EMAIL_SERVICE');
const USER_REPOSITORY = new InjectionToken<UserRepository>('USER_REPOSITORY');

const container = new DIContainer();

container.register({
  token: EMAIL_SERVICE,
  useClass: ConcreteEmailService,
  singleton: true
});

container.register({
  token: USER_REPOSITORY,
  useClass: DatabaseUserRepository,
  singleton: true
});

container.register({
  token: 'USER_SERVICE',
  useFactory: (emailService, userRepository) => 
    new UserServiceDI(emailService, userRepository),
  deps: [EMAIL_SERVICE, USER_REPOSITORY]
});

const userService = container.resolve<UserService>('USER_SERVICE');
```

### Продвинутый DI контейнер с декораторами
```typescript
// Декораторы для DI (требуется experimentalDecorators)
const Injectable = (): ClassDecorator => (target) => {
  // Маркер класса как инжектируемого
};

const Inject = (token: Token): ParameterDecorator => {
  return (target, propertyKey, parameterIndex) => {
    // Сохраняем токен в метаданных
    const existingTokens = Reflect.getMetadata('injection_tokens', target) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata('injection_tokens', existingTokens, target);
  };
};

@Injectable()
class AdvancedUserService {
  constructor(
    @Inject(EMAIL_SERVICE) private emailService: EmailServiceInterface,
    @Inject(USER_REPOSITORY) private userRepository: UserRepository
  ) {}
  
  async registerUser(userData: User): Promise<User> {
    const user = await this.userRepository.create(userData);
    await this.emailService.send(`Welcome ${user.email}`);
    return user;
  }
}
```

## Встроенные DI в фреймворках

### DI в Angular
```typescript
// Angular использует встроенный DI контейнер
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // синглтон на уровне приложения
})
export class AngularUserService {
  constructor(
    private http: HttpClient, // автоматическое внедрение
    private authService: AuthService
  ) {}
  
  async getUser(id: number): Promise<User> {
    const response = await this.http.get(`/api/users/${id}`);
    return response as User;
  }
}

// Регистрация провайдеров
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
    { provide: 'API_URL', useValue: 'https://api.example.com' }
  ]
})
export class AppModule {}
```

### DI в NestJS
```typescript
import { Injectable, Module } from '@nestjs/common';

@Injectable()
export class NestUserService {
  constructor(
    @Inject('DATABASE_CONNECTION') private connection: Connection,
    private emailService: EmailService
  ) {}
  
  findAll(): Promise<User[]> {
    return this.connection.query<User[]>('SELECT * FROM users');
  }
}

@Module({
  providers: [
    UserService,
    EmailService,
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: () => createConnection(),
    },
  ],
  exports: [UserService],
})
export class UserModule {}
```

## Функциональный подход к DI

### Service Locator паттерн
```typescript
// Альтернатива DI - Service Locator
interface ServiceLocator {
  get<T>(token: string): T;
  register<T>(token: string, factory: () => T): void;
}

class SimpleServiceLocator implements ServiceLocator {
  private services = new Map<string, () => any>();
  private instances = new Map<string, any>();
  
  register<T>(token: string, factory: () => T): void {
    this.services.set(token, factory);
  }
  
  get<T>(token: string): T {
    if (this.instances.has(token)) {
      return this.instances.get(token);
    }
    
    const factory = this.services.get(token);
    if (!factory) {
      throw new Error(`Service not registered: ${token}`);
    }
    
    const instance = factory();
    if (this.services.get(token) !== this.instances.get(token)) {
      // Сохраняем синглтон
      this.instances.set(token, instance);
    }
    
    return instance as T;
  }
}

// Использование
const locator = new SimpleServiceLocator();
locator.register('emailService', () => new ConcreteEmailService());
locator.register('userService', () => 
  new UserServiceDI(locator.get('emailService'))
);

const userService = locator.get<UserService>('userService');
```

### Dependency Injection через функции высшего порядка
```typescript
// Функциональная реализация DI
type ServiceFactory<T> = (services: ServiceLocator) => T;

const createUserService: ServiceFactory<UserService> = (services) => {
  const emailService = services.get<EmailService>('emailService');
  const userRepository = services.get<UserRepository>('userRepository');
  
  return new UserServiceDI(emailService, userRepository);
};

const createOrderService: ServiceFactory<OrderService> = (services) => {
  const paymentService = services.get<PaymentService>('paymentService');
  const inventoryService = services.get<InventoryService>('inventoryService');
  const notificationService = services.get<NotificationService>('notificationService');
  
  return new OrderService(paymentService, inventoryService, notificationService);
};

// Композиция зависимостей
const createServices = (locator: ServiceLocator) => ({
  userService: createUserService(locator),
  orderService: createOrderService(locator)
});

// Использование
const locator = new SimpleServiceLocator();
// регистрируем зависимости...
const services = createServices(locator);
```

## Практические примеры

### DI в системе логирования
```typescript
interface Logger {
  log(level: string, message: string, meta?: any): void;
}

class ConsoleLogger implements Logger {
  log(level: string, message: string, meta?: any): void {
    console[level](message, meta);
  }
}

class FileLogger implements Logger {
  constructor(private filePath: string) {}
  
  log(level: string, message: string, meta?: any): void {
    // запись в файл
    const logEntry = `[${new Date().toISOString()}] ${level}: ${message}`;
    require('fs').appendFileSync(this.filePath, logEntry + '\n');
  }
}

class CompositeLogger implements Logger {
  constructor(private loggers: Logger[]) {}
  
  log(level: string, message: string, meta?: any): void {
    this.loggers.forEach(logger => logger.log(level, message, meta));
  }
}

// Фабрика логгеров
class LoggerFactory {
  static createLogger(environment: string): Logger {
    switch (environment) {
      case 'development':
        return new ConsoleLogger();
      case 'production':
        return new CompositeLogger([
          new ConsoleLogger(),
          new FileLogger('./logs/app.log')
        ]);
      default:
        return new ConsoleLogger();
    }
  }
}

// Использование в сервисе
class APIService {
  constructor(@Inject(LOGGER_TOKEN) private logger: Logger) {}
  
  async getData(): Promise<any> {
    this.logger.log('info', 'Fetching data from API');
    
    try {
      const response = await fetch('/api/data');
      this.logger.log('info', 'Data fetched successfully');
      return response.json();
    } catch (error) {
      this.logger.log('error', 'Failed to fetch data', error);
      throw error;
    }
  }
}
```

### DI для работы с конфигурацией
```typescript
interface Config {
  get<T>(key: string, defaultValue?: T): T;
  set<T>(key: string, value: T): void;
}

class EnvironmentConfig implements Config {
  get<T>(key: string, defaultValue?: T): T {
    const value = process.env[key];
    return value !== undefined ? JSON.parse(value) : defaultValue;
  }
  
  set<T>(key: string, value: T): void {
    process.env[key] = JSON.stringify(value);
  }
}

class FileConfig implements Config {
  constructor(private configPath: string) {}
  
  get<T>(key: string, defaultValue?: T): T {
    const config = JSON.parse(require('fs').readFileSync(this.configPath, 'utf8'));
    return config[key] !== undefined ? config[key] : defaultValue;
  }
  
  set<T>(key: string, value: T): void {
    const config = JSON.parse(require('fs').readFileSync(this.configPath, 'utf8'));
    config[key] = value;
    require('fs').writeFileSync(this.configPath, JSON.stringify(config, null, 2));
  }
}

class ConfigAdapter implements Config {
  private configService: Config;
  
  constructor(environment: string) {
    this.configService = environment === 'docker' 
      ? new FileConfig('/config/config.json') 
      : new EnvironmentConfig();
  }
  
  get<T>(key: string, defaultValue?: T): T {
    return this.configService.get(key, defaultValue);
  }
  
  set<T>(key: string, value: T): void {
    this.configService.set(key, value);
  }
}

// Регистрация в контейнере
const CONFIG_TOKEN = new InjectionToken<Config>('CONFIG');

container.register({
  token: CONFIG_TOKEN,
  useFactory: () => new ConfigAdapter(process.env.NODE_ENV || 'development'),
  singleton: true
});
```

### Асинхронные зависимости
```typescript
// Асинхронная инициализация зависимостей
interface AsyncInitializer {
  initialize(): Promise<void>;
}

class DatabaseService implements AsyncInitializer {
  private connection: any;
  
  async initialize(): Promise<void> {
    this.connection = await createDatabaseConnection();
    await this.runMigrations();
  }
  
  async runMigrations(): Promise<void> {
    // выполнение миграций
  }
  
  query(sql: string): Promise<any> {
    if (!this.connection) {
      throw new Error("Database not initialized");
    }
    return this.connection.query(sql);
  }
}

class ServiceWithAsyncDeps {
  private isInitialized = false;
  
  constructor(private database: DatabaseService) {}
  
  async ensureInitialized(): Promise<void> {
    if (!this.isInitialized) {
      await this.database.initialize();
      this.isInitialized = true;
    }
  }
  
  async getUser(id: number): Promise<any> {
    await this.ensureInitialized();
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}
```

## Продвинутые паттерны DI

### Conditional Dependencies
```typescript
// Условные зависимости
interface FeatureFlagService {
  isEnabled(flag: string): boolean;
}

class FeatureService {
  private featureImpl: FeatureImplementation;
  
  constructor(
    private flagService: FeatureFlagService,
    private newFeatureService: NewFeatureService,
    private legacyFeatureService: LegacyFeatureService
  ) {
    // Выбор реализации на основе флагов
    this.featureImpl = flagService.isEnabled('new_feature') 
      ? newFeatureService 
      : legacyFeatureService;
  }
  
  execute(): void {
    this.featureImpl.execute();
  }
}
```

### Factory Pattern + DI
```typescript
// Фабрика с внедрением зависимостей
class ServiceFactory {
  constructor(
    private database: DatabaseService,
    private logger: Logger,
    private config: Config
  ) {}
  
  createUserService(): UserService {
    return new UserService(
      this.database,
      this.logger,
      this.config.get<number>('user_timeout', 5000)
    );
  }
  
  createOrderService(): OrderService {
    return new OrderService(
      this.database,
      this.logger,
      this.config.get<number>('order_timeout', 10000)
    );
  }
}

// Регистрация фабрики в контейнере
container.register({
  token: SERVICE_FACTORY,
  useFactory: (database, logger, config) => 
    new ServiceFactory(database, logger, config),
  deps: [DATABASE_SERVICE, LOGGER, CONFIG]
});
```

### Decorator-based Registration
```typescript
// Регистрация через декораторы
type Constructor<T = {}> = new (...args: any[]) => T;

function Injectable<T extends Constructor>(Base: T) {
  Reflect.defineMetadata('injectable', true, Base);
  return Base;
}

function Inject(token: Token) {
  return function(target: any, propertyKey: string | symbol, parameterIndex: number) {
    const existingTokens = Reflect.getMetadata('param_tokens', target, propertyKey) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata('param_tokens', existingTokens, target, propertyKey);
  };
}

@Injectable
class DecoratedService {
  constructor(
    @Inject(HTTP_CLIENT) private http: HttpClient,
    @Inject(CONFIG) private config: Config
  ) {}
  
  async fetchData(): Promise<any> {
    const url = this.config.get<string>('api_url');
    return this.http.get(url);
  }
}
```

## Тестирование с DI

### Mocking зависимости
```typescript
// Тестирование с mocked зависимостями
class MockEmailService implements EmailServiceInterface {
  emailsSent: string[] = [];
  
  async send(email: string): Promise<void> {
    this.emailsSent.push(email);
  }
}

class MockUserRepository implements UserRepository {
  private users: User[] = [];
  
  async create(userData: Partial<User>): Promise<User> {
    const user = { id: this.users.length + 1, ...userData } as User;
    this.users.push(user);
    return user;
  }
  
  async findById(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
}

// Тестирование
describe('UserService', () => {
  let userService: UserService;
  let mockEmailService: MockEmailService;
  let mockUserRepository: MockUserRepository;
  
  beforeEach(() => {
    mockEmailService = new MockEmailService();
    mockUserRepository = new MockUserRepository();
    userService = new UserServiceDI(mockEmailService, mockUserRepository);
  });
  
  it('should register user and send welcome email', async () => {
    const newUser = { email: 'test@example.com', name: 'Test User' };
    const result = await userService.register(newUser);
    
    expect(result.email).toBe('test@example.com');
    expect(mockEmailService.emailsSent).toContain('Welcome test@example.com');
  });
});
```

### Test Container
```typescript
// Контейнер для тестирования
class TestContainer extends DIContainer {
  registerMock<T>(token: Token, mock: T): void {
    this.register({
      token,
      useValue: mock
    });
  }
  
  createTestingModule(providers: Provider[]): void {
    providers.forEach(provider => this.register(provider));
  }
}

// Использование в тестах
const testContainer = new TestContainer();
testContainer.registerMock(EMAIL_SERVICE, new MockEmailService());
testContainer.registerMock(USER_REPOSITORY, new MockUserRepository());

const userService = testContainer.resolve<UserService>(USER_SERVICE);
```

## Лучшие практики

### 1. Используйте интерфейсы, а не конкретные классы
```typescript
// ПЛОХО: жесткая зависимость
class BadUserService {
  private emailService = new ConcreteEmailService(); // жесткая зависимость
}

// ХОРОШО: через интерфейс
class GoodUserService {
  constructor(private emailService: EmailServiceInterface) {}
}
```

### 2. Избегайте Service Locator anti-pattern
```typescript
// ПЛОХО: Service Locator делает зависимости скрытыми
class BadService {
  doSomething(): void {
    const emailService = ServiceLocator.get<EmailService>('email');
    emailService.send("message"); // зависимости не очевидны из сигнатуры
  }
}

// ХОРОШО: через конструктор
class GoodService {
  constructor(private emailService: EmailServiceInterface) {}
  
  doSomething(): void {
    this.emailService.send("message"); // зависимости очевидны
  }
}
```

### 3. Не создавайте слишком много зависимостей
```typescript
// ПЛОХО: слишком много зависимостей
class BloatedService {
  constructor(
    private dep1: Dep1,
    private dep2: Dep2,
    private dep3: Dep3,
    private dep4: Dep4,
    private dep5: Dep5,
    private dep6: Dep6,
    private dep7: Dep7
  ) {}
}

// ЛУЧШЕ: объединить связанные зависимости
interface CommonModule {
  emailService: EmailService;
  notificationService: NotificationService;
}

class BetterService {
  constructor(private common: CommonModule) {}
}
```

## Ограничения и подводные камни

### 1. Циклические зависимости
```typescript
// Циклическая зависимость - проблема для DI
class ServiceA {
  constructor(private serviceB: ServiceB) {} // A depends on B
}

class ServiceB {
  constructor(private serviceA: ServiceA) {} // B depends on A
}

// Решения:
// 1. Переработать архитектуру
// 2. Использовать lazy loading
// 3. Использовать shared state
```

### 2. Performance overhead
```typescript
// DI может добавлять накладные расходы
// Оптимизация: использовать синглтоны для тяжелых зависимостей
// Использовать lazy initialization для ресурсоемких сервисов
```

## Современные тенденции

### Constructor injection везде
```typescript
// Современная практика: всегда использовать constructor injection
class ModernService {
  constructor(
    private readonly repository: Repository,
    private readonly logger: Logger,
    private readonly config: Config
  ) {}
  
  // Методы сервиса...
}
```

### Composition Root pattern
```typescript
// Единая точка конфигурации зависимостей
class CompositionRoot {
  static create(): Application {
    const container = new DIContainer();
    
    // Регистрация всех зависимостей
    container.register({
      token: DATABASE_SERVICE,
      useFactory: createDatabaseConnection,
      singleton: true
    });
    
    container.register({
      token: EMAIL_SERVICE,
      useClass: EmailService,
      singleton: true
    });
    
    container.register({
      token: USER_SERVICE,
      useClass: UserService,
      deps: [DATABASE_SERVICE, EMAIL_SERVICE]
    });
    
    return new Application(container);
  }
}

const app = CompositionRoot.create();
```

## Сравнение с другими подходами

| Подход | Преимущества | Недостатки |
|--------|-------------|------------|
| Dependency Injection | Тестируемость, слабая связанность | Сложность конфигурации |
| Service Locator | Простота использования | Скрытые зависимости |
| Constructor Injection | Ясные зависимости | Много бойлерплейта |
| Setter Injection | Гибкость | Возможность использования неполностью сконфигурированного объекта |

## Связь с другими концепциями
- [[../advanced-types/conditional-types]] - условные типы для DI
- [[../generics/di-pattern]] - обобщенные DI паттерны
- [[../modules/container-pattern]] - модульная организация DI
- [[../patterns/structural/adapter-pattern]] - адаптеры для интеграции
- [[../functional-programming/di-alternatives]] - функциональные альтернативы DI
- [[../ecosystem/frameworks/di-systems]] - DI в различных фреймворках