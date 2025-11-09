---
tags: 
  - programming
  - architecture
  - design-patterns
  - dependency-injection
  - implementation
---

# Паттерны и реализация внедрения зависимостей

## Введение

В этой статье мы рассмотрим различные паттерны внедрения зависимостей и их реализации в различных языках программирования. Мы также изучим продвинутые концепции, такие как жизненный цикл зависимостей, разрешение зависимостей и контейнеры DI.

## Продвинутые паттерны внедрения зависимостей

### 1. Factory Pattern с DI

Иногда зависимости нужно создавать динамически. В таких случаях используется фабрика:

```javascript
class UserRepositoryFactory {
  constructor(databaseConfig) {
    this.config = databaseConfig;
  }
  
  create(type) {
    switch(type) {
      case 'mysql':
        return new MySQLUserRepository(this.config.mysql);
      case 'postgres':
        return new PostgreSQLUserRepository(this.config.postgres);
      default:
        throw new Error(`Unknown repository type: ${type}`);
    }
  }
}

class UserService {
  constructor(userRepositoryFactory) {
    this.repositoryFactory = userRepositoryFactory;
  }
  
  async getUser(userId, dbType) {
    const repository = this.repositoryFactory.create(dbType);
    return await repository.findById(userId);
  }
}
```

### 2. Provider Pattern

Поставщики позволяют откладывать создание зависимостей до момента их фактического использования:

```javascript
class LazyDatabaseProvider {
  constructor(connectionConfig) {
    this.config = connectionConfig;
    this._instance = null;
  }
  
  get() {
    if (!this._instance) {
      this._instance = new DatabaseConnection(this.config);
    }
    return this._instance;
  }
}

class UserService {
  constructor(databaseProvider) {
    this.dbProvider = databaseProvider;
  }
  
  async getUser(id) {
    // Подключение создается только при необходимости
    const db = this.dbProvider.get();
    return await db.findById(id);
  }
}
```

## Жизненный цикл зависимостей

В контейнерах DI обычно поддерживаются следующие жизненные циклы:

### Transient (Временные)

Новый экземпляр создается каждый раз при запросе:

```javascript
container.register('logger', () => new Logger(), false); // Не синглтон
```

### Singleton (Одиночки)

Один и тот же экземпляр используется для всех запросов:

```javascript
container.register('database', () => new DatabaseConnection(config), true); // Синглтон
```

### Scoped (Область видимости)

Один экземпляр используется в пределах определенной области (например, HTTP-запрос):

```javascript
class ScopedContainer {
  constructor(parentContainer) {
    this.parentContainer = parentContainer;
    this.scopedInstances = new Map();
  }
  
  resolve(name) {
    if (!this.scopedInstances.has(name)) {
      const factory = this.parentContainer.getFactory(name);
      this.scopedInstances.set(name, factory(this));
    }
    return this.scopedInstances.get(name);
  }
}
```

## Продвинутая реализация контейнера DI

```javascript
class AdvancedDIContainer {
  constructor() {
    this.registrations = new Map();
    this.instances = new Map();
    this.scopes = [];
  }
  
  // Регистрация с различными опциями
  register(name, factory, options = {}) {
    const registration = {
      factory,
      lifecycle: options.lifecycle || 'transient', // transient, singleton, scoped
      dependencies: options.dependencies || [],
      setup: options.setup || null
    };
    
    this.registrations.set(name, registration);
  }
  
  // Регистрация интерфейса к реализации
  registerInterface(interfaceName, implementationName, options = {}) {
    this.register(interfaceName, (container) => {
      return container.resolve(implementationName);
    }, options);
  }
  
  resolve(name) {
    const registration = this.registrations.get(name);
    if (!registration) {
      throw new Error(`Registration for ${name} not found`);
    }
    
    // Для синглтонов проверяем кэш
    if (registration.lifecycle === 'singleton') {
      if (!this.instances.has(name)) {
        const instance = this._createInstance(registration);
        this.instances.set(name, instance);
      }
      return this.instances.get(name);
    }
    
    // Для scoped проверяем текущую область
    if (registration.lifecycle === 'scoped') {
      const currentScope = this.scopes[this.scopes.length - 1];
      if (currentScope && currentScope.has(name)) {
        return currentScope.get(name);
      }
      
      const instance = this._createInstance(registration);
      if (currentScope) {
        currentScope.set(name, instance);
      }
      return instance;
    }
    
    // Для transient создаем каждый раз
    return this._createInstance(registration);
  }
  
  _createInstance(registration) {
    // Рекурсивное разрешение зависимостей
    const dependencies = registration.dependencies.map(dep => this.resolve(dep));
    const instance = registration.factory(this, ...dependencies);
    
    // Выполнение дополнительной настройки
    if (registration.setup) {
      registration.setup(instance);
    }
    
    return instance;
  }
  
  createScope() {
    const scope = new Map();
    this.scopes.push(scope);
    return {
      resolve: (name) => this.resolve(name),
      dispose: () => {
        const disposedScope = this.scopes.pop();
        if (disposedScope) {
          // Очистка ресурсов при необходимости
          for (const [name, instance] of disposedScope) {
            if (instance && typeof instance.dispose === 'function') {
              instance.dispose();
            }
          }
        }
      }
    };
  }
}

// Пример использования
const container = new AdvancedDIContainer();

// Регистрация базовых сервисов
container.register('config', () => loadConfig());
container.register('logger', (container) => new Logger(container.resolve('config')), { 
  lifecycle: 'singleton' 
});

// Регистрация репозитория
container.register('userRepository', (container) => 
  new MySQLUserRepository(container.resolve('config')), {
    lifecycle: 'singleton',
    dependencies: ['config']
  });

// Регистрация сервиса
container.register('userService', (container) => 
  new UserService(container.resolve('userRepository'), container.resolve('logger')), {
    dependencies: ['userRepository', 'logger']
  });

// Регистрация интерфейса
container.registerInterface('IUserRepository', 'userRepository');
```

## Реализация на TypeScript с декораторами

```typescript
// Декораторы для аннотации зависимостей
const INJECTABLE = Symbol('injectable');
const INJECT = Symbol('inject');

// Метаданные для отражения зависимостей
const INJECTION_TOKENS = Symbol('injectionTokens');

function Injectable() {
  return function<T extends { new (...args: any[]): {} }>(constructor: T) {
    Reflect.defineMetadata(INJECTABLE, true, constructor);
    return constructor;
  };
}

function Inject(token: string) {
  return function(target: any, propertyKey: string | symbol, parameterIndex: number) {
    const existingTokens = Reflect.getMetadata(INJECTION_TOKENS, target) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata(INJECTION_TOKENS, existingTokens, target);
  };
}

@Injectable()
class DatabaseService {
  constructor(@Inject('config') private config: any) {}
  
  connect() {
    // Подключение к базе данных
  }
}

@Injectable()
class UserService {
  constructor(
    @Inject('databaseService') private database: DatabaseService,
    @Inject('logger') private logger: Logger
  ) {}
  
  async getUser(id: string) {
    this.logger.log(`Getting user ${id}`);
    return await this.database.findById(id);
  }
}
```

## Асинхронное внедрение зависимостей

В современных приложениях часто необходимо асинхронное создание зависимостей:

```javascript
class AsyncDIContainer {
  constructor() {
    this.registrations = new Map();
    this.instances = new Map();
  }
  
  registerAsync(name, asyncFactory, options = {}) {
    this.registrations.set(name, {
      factory: asyncFactory,
      lifecycle: options.lifecycle || 'transient',
      isAsync: true
    });
  }
  
  async resolveAsync(name) {
    const registration = this.registrations.get(name);
    if (!registration) {
      throw new Error(`Registration for ${name} not found`);
    }
    
    if (registration.lifecycle === 'singleton') {
      if (!this.instances.has(name)) {
        const instance = await registration.factory(this);
        this.instances.set(name, instance);
      }
      return this.instances.get(name);
    }
    
    return await registration.factory(this);
  }
}

// Пример асинхронного создания сервиса
container.registerAsync('databaseService', async (container) => {
  const config = container.resolve('config');
  const db = new DatabaseConnection(config);
  await db.initialize(); // Асинхронная инициализация
  return db;
});
```

## Интеграция с фреймворками

### Внедрение зависимостей в Express.js

```javascript
// Создание middleware для DI
function diMiddleware(container) {
  return (req, res, next) => {
    req.di = container;
    next();
  };
}

// Использование в контроллере
function createUserController(container) {
  return {
    async getUser(req, res) {
      const userService = container.resolve('userService');
      const user = await userService.getUser(req.params.id);
      res.json(user);
    }
  };
}

// Регистрация маршрутов
app.use(diMiddleware(container));
app.get('/users/:id', (req, res, next) => {
  const controller = createUserController(req.di);
  controller.getUser(req, res).catch(next);
});
```

## Лучшие практики

### 1. Избегайте Service Locator

```javascript
// Плохо
class UserService {
  getUser(id) {
    const database = ServiceLocator.get('database');
    return database.findById(id);
  }
}

// Хорошо
class UserService {
  constructor(database) {
    this.database = database;
  }
  
  getUser(id) {
    return this.database.findById(id);
  }
}
```

### 2. Используйте интерфейсы

```javascript
// Определяем абстракцию
interface ILogger {
  log(message: string): void;
  error(message: string): void;
}

// Реализуем конкретный логгер
class ConsoleLogger implements ILogger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }
  
  error(message) {
    console.error(`[ERROR] ${message}`);
  }
}

// Используем в сервисе
class UserService {
  constructor(private logger: ILogger) {}
  
  async registerUser(userData) {
    try {
      // Регистрация пользователя
      this.logger.log('User registered successfully');
    } catch (error) {
      this.logger.error(`Registration failed: ${error.message}`);
      throw error;
    }
  }
}
```

### 3. Правильное управление жизненным циклом

```javascript
// Правильная очистка ресурсов
interface IDisposable {
  dispose(): void;
}

class DatabaseConnection implements IDisposable {
  constructor(config) {
    this.config = config;
    this.connection = null;
  }
  
  async connect() {
    this.connection = await createConnection(this.config);
  }
  
  dispose() {
    if (this.connection) {
      this.connection.close();
    }
  }
}
```

## Заключение

Продвинутые паттерны внедрения зависимостей позволяют создавать гибкие, масштабируемые и легко тестируемые приложения. Правильное использование жизненного цикла зависимостей, фабрик и провайдеров помогает эффективно управлять сложностью приложения.

Следующие статьи в этой серии охватывают:

- [[Testing with Dependency Injection]]
- [[DI in Different Languages]]
- [[DI Frameworks Comparison]]

См. также:
- [[Внедрение-зависимостей]]
- [[Inversion of Control]]
- [[SOLID Principles]]
- [[Clean Architecture]]