# Композиция декораторов

Композиция декораторов - это процесс применения нескольких декораторов к одному объявлению, что позволяет комбинировать различные аспекты поведения в одном месте.

## Порядок применения декораторов

В TypeScript декораторы применяются в определенном порядке:

1. **Для параметров**: Слева направо
2. **Для методов, аксессоров и свойств**: Сверху вниз
3. **Для классов**: Снизу вверх

### Пример порядка применения

```typescript
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

function third() {
  console.log("third(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("third(): called");
  };
}

class Example {
  @first()
  @second()
  @third()
  method() {}
}

// Выведет:
// "first(): factory evaluated"
// "second(): factory evaluated"
// "third(): factory evaluated"
// "third(): called"
// "second(): called"
// "first(): called"
```

## Комбинирование разных типов декораторов

### 1. Комбинирование декораторов класса и метода

```typescript
function logClass(constructor: Function) {
  console.log(`Class ${constructor.name} is being created`);
}

function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling method ${propertyKey}`);
    return originalMethod.apply(this, args);
  };
}

@logClass
class UserService {
  @logMethod
  getUser(id: string) {
    return { id, name: `User ${id}` };
  }
}

// Выведет: "Class UserService is being created"

const service = new UserService();
service.getUser("123");
// Выведет: "Calling method getUser"
```

### 2. Комбинирование декораторов свойств и аксессоров

```typescript
function validate(required: boolean) {
  return function(target: any, propertyKey: string) {
    let value: any;
    
    const descriptor: PropertyDescriptor = {
      get() {
        if (required && (value === undefined || value === null)) {
          throw new Error(`Property ${propertyKey} is required`);
        }
        return value;
      },
      set(newValue: any) {
        value = newValue;
      },
      enumerable: true,
      configurable: true
    };
    
    Object.defineProperty(target, propertyKey, descriptor);
  };
}

function trackChanges(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalSetter = descriptor.set;
  
  if (originalSetter) {
    descriptor.set = function(value: any) {
      console.log(`Setting ${propertyKey} to:`, value);
      originalSetter.call(this, value);
    };
  }
}

class User {
  private _name: string;
  
  @validate(true)
  @trackChanges
  set name(value: string) {
    this._name = value;
  }
  
  get name(): string {
    return this._name;
  }
}

const user = new User();
user.name = "John"; // Выведет: "Setting name to: John"
console.log(user.name); // "John"

// user.name = undefined; // При попытке получить значение будет ошибка: Property name is required
```

### 3. Комбинирование декораторов параметров и методов

```typescript
const requiredParameters: Map<any, Map<string, number[]>> = new Map();

function required(target: any, propertyKey: string, parameterIndex: number) {
  const constructor = target.constructor;
  
  if (!requiredParameters.has(constructor)) {
    requiredParameters.set(constructor, new Map());
  }
  
  const methodMap = requiredParameters.get(constructor)!;
  
  if (!methodMap.has(propertyKey)) {
    methodMap.set(propertyKey, []);
  }
  
  methodMap.get(propertyKey)!.push(parameterIndex);
}

function validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const constructor = target.constructor;
    const methodMap = requiredParameters.get(constructor);
    
    if (methodMap && methodMap.has(propertyKey)) {
      const requiredIndices = methodMap.get(propertyKey)!;
      
      for (const index of requiredIndices) {
        if (index < args.length && (args[index] === undefined || args[index] === null)) {
          throw new Error(`Parameter at index ${index} of method ${propertyKey} is required`);
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Method ${propertyKey} returned:`, result);
    return result;
  };
}

class OrderService {
  @validate
  @log
  createOrder(@required productId: string, @required quantity: number, customerId?: string) {
    return {
      id: `order_${Date.now()}`,
      productId,
      quantity,
      customerId: customerId || "anonymous"
    };
  }
}

const service = new OrderService();
service.createOrder("product123", 5);
// Выведет: "Calling createOrder with args: ['product123', 5, undefined]"
// Выведет: "Method createOrder returned: { id: 'order_1234567890', productId: 'product123', quantity: 5, customerId: 'anonymous' }"

// service.createOrder(undefined, 5); // ❌ Ошибка: Parameter at index 0 of method createOrder is required
```

## Продвинутые примеры композиции

### 1. Создание фреймворка с композицией декораторов

```typescript
// Декораторы для контроллеров
function controller(basePath: string) {
  return function(constructor: Function) {
    constructor.prototype.basePath = basePath;
    console.log(`Controller ${constructor.name} registered with base path: ${basePath}`);
  };
}

// Декораторы для маршрутов
function route(method: string, path: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    if (!target.routes) {
      target.routes = [];
    }
    
    target.routes.push({
      method,
      path,
      handler: propertyKey
    });
    
    console.log(`Route ${method} ${path} registered for method ${propertyKey}`);
  };
}

// Декораторы для параметров маршрутов
function param(name: string) {
  return function(target: any, propertyKey: string, parameterIndex: number) {
    if (!target.routeParams) {
      target.routeParams = {};
    }
    
    if (!target.routeParams[propertyKey]) {
      target.routeParams[propertyKey] = [];
    }
    
    target.routeParams[propertyKey][parameterIndex] = name;
  };
}

// Декораторы для middleware
function middleware(...middlewares: Function[]) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    if (!target.middlewares) {
      target.middlewares = {};
    }
    
    target.middlewares[propertyKey] = middlewares;
  };
}

// Декораторы для валидации
function validateBody(schema: any) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      // Здесь была бы реализация валидации
      console.log(`Validating body against schema for method ${propertyKey}`);
      return originalMethod.apply(this, args);
    };
  };
}

// Использование
@controller("/api/users")
class UserController {
  @route("GET", "/")
  @middleware(authMiddleware, loggingMiddleware)
  getAllUsers() {
    return [{ id: "1", name: "John" }];
  }
  
  @route("GET", "/:id")
  @middleware(authMiddleware)
  getUserById(@param("id") id: string) {
    return { id, name: `User ${id}` };
  }
  
  @route("POST", "/")
  @validateBody({ name: "string", email: "string" })
  @middleware(authMiddleware, validationMiddleware)
  createUser(@param("body") body: any) {
    return { id: "123", ...body };
  }
}

function authMiddleware(req: any, res: any, next: Function) {
  console.log("Auth middleware");
  next();
}

function loggingMiddleware(req: any, res: any, next: Function) {
  console.log("Logging middleware");
  next();
}

function validationMiddleware(req: any, res: any, next: Function) {
  console.log("Validation middleware");
  next();
}

// Выведет:
// "Controller UserController registered with base path: /api/users"
// "Route GET / registered for method getAllUsers"
// "Route GET /:id registered for method getUserById"
// "Route POST / registered for method createUser"
```

### 2. Композиция декораторов для создания ORM

```typescript
// Декораторы для модели
function entity(tableName: string) {
  return function(constructor: Function) {
    constructor.prototype.tableName = tableName;
    console.log(`Entity ${constructor.name} mapped to table: ${tableName}`);
  };
}

// Декораторы для колонок
function column(columnName: string, options?: { primaryKey?: boolean; nullable?: boolean }) {
  return function(target: any, propertyKey: string) {
    if (!target.columns) {
      target.columns = {};
    }
    
    target.columns[propertyKey] = {
      name: columnName,
      ...options
    };
  };
}

// Декораторы для отношений
function oneToMany(targetEntity: string, foreignKey: string) {
  return function(target: any, propertyKey: string) {
    if (!target.relations) {
      target.relations = {};
    }
    
    target.relations[propertyKey] = {
      type: "oneToMany",
      targetEntity,
      foreignKey
    };
  };
}

// Декораторы для валидации
function required() {
  return function(target: any, propertyKey: string) {
    if (!target.validations) {
      target.validations = {};
    }
    
    if (!target.validations[propertyKey]) {
      target.validations[propertyKey] = [];
    }
    
    target.validations[propertyKey].push("required");
  };
}

function minLength(length: number) {
  return function(target: any, propertyKey: string) {
    if (!target.validations) {
      target.validations = {};
    }
    
    if (!target.validations[propertyKey]) {
      target.validations[propertyKey] = [];
    }
    
    target.validations[propertyKey].push({ minLength: length });
  };
}

// Декораторы для индексов
function index(name?: string) {
  return function(target: any, propertyKey: string) {
    if (!target.indexes) {
      target.indexes = [];
    }
    
    target.indexes.push({
      name: name || `${target.constructor.name}_${propertyKey}_idx`,
      column: propertyKey
    });
  };
}

// Использование
@entity("users")
class User {
  @column("id", { primaryKey: true })
  @required()
  id: string;
  
  @column("name")
  @required()
  @minLength(2)
  @index("users_name_idx")
  name: string;
  
  @column("email")
  @required()
  email: string;
  
  @oneToMany("Order", "userId")
  orders: Order[];
}

@entity("orders")
class Order {
  @column("id", { primaryKey: true })
  @required()
  id: string;
  
  @column("user_id")
  @required()
  userId: string;
  
  @column("total")
  @required()
  total: number;
}

// Выведет:
// "Entity User mapped to table: users"
// "Entity Order mapped to table: orders"
```

### 3. Композиция декораторов для создания DI контейнера

```typescript
// Реестр сервисов
const serviceRegistry: Map<string, any> = new Map();
const dependencyMetadata: Map<any, Map<string, string[]>> = new Map();

// Декораторы для регистрации сервисов
function service(token?: string) {
  return function(constructor: Function) {
    const serviceName = token || constructor.name;
    serviceRegistry.set(serviceName, constructor);
    console.log(`Service ${constructor.name} registered as ${serviceName}`);
  };
}

// Декораторы для инъекции зависимостей
function inject(token: string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    if (!dependencyMetadata.has(constructor)) {
      dependencyMetadata.set(constructor, new Map());
    }
    
    const methodMap = dependencyMetadata.get(constructor)!;
    const methodName = propertyKey || "constructor";
    
    if (!methodMap.has(methodName)) {
      methodMap.set(methodName, []);
    }
    
    const dependencies = methodMap.get(methodName)!;
    dependencies[parameterIndex] = token;
  };
}

// Декораторы для singleton
function singleton(target: any) {
  const originalConstructor = target;
  let instance: any;
  
  const newConstructor: any = function(...args: any[]) {
    if (!instance) {
      instance = new originalConstructor(...args);
    }
    return instance;
  };
  
  newConstructor.prototype = originalConstructor.prototype;
  return newConstructor;
}

// Декораторы для scoped lifetime
function scoped(target: any) {
  // Реализация scoped lifetime
  console.log(`Service ${target.name} registered as scoped`);
  return target;
}

// Декораторы для transient lifetime
function transient(target: any) {
  // Реализация transient lifetime
  console.log(`Service ${target.name} registered as transient`);
  return target;
}

// Использование
@service("logger")
@singleton
class Logger {
  log(message: string) {
    console.log(`[LOG] ${message}`);
  }
}

@service("database")
@singleton
class Database {
  query(sql: string) {
    console.log(`Executing: ${sql}`);
    return [];
  }
}

@service("userService")
@singleton
class UserService {
  constructor(
    @inject("logger") private logger: Logger,
    @inject("database") private database: Database
  ) {}
  
  getUsers() {
    this.logger.log("Fetching users");
    return this.database.query("SELECT * FROM users");
  }
}

@service("orderService")
@scoped
class OrderService {
  constructor(
    @inject("logger") private logger: Logger,
    @inject("database") private database: Database
  ) {}
  
  getOrders() {
    this.logger.log("Fetching orders");
    return this.database.query("SELECT * FROM orders");
  }
}

// Выведет:
// "Service Logger registered as logger"
// "Service Logger registered as singleton"
// "Service Database registered as database"
// "Service Database registered as singleton"
// "Service UserService registered as userService"
// "Service UserService registered as singleton"
// "Service OrderService registered as orderService"
// "Service OrderService registered as scoped"
```

## Практические рекомендации

### 1. Порядок применения декораторов

```typescript
// Рекомендуемый порядок для методов:
class Example {
  @validate  // 1. Валидация (внешние проверки)
  @authorize // 2. Авторизация (проверка прав доступа)
  @log       // 3. Логирование (запись входных данных)
  @cache     // 4. Кэширование (проверка кэша)
  @timing    // 5. Измерение времени (начало замера)
  method(@required @validateParam param: string) {
    // 6. Основная логика метода
  }
  // 7. Измерение времени (окончание замера)
  // 8. Кэширование (сохранение результата)
  // 9. Логирование (запись результата)
}
```

### 2. Избегайте конфликтов между декораторами

```typescript
// Плохо: декораторы конфликтуют между собой
class BadExample {
  @readonly  // Делает свойство только для чтения
  @writable  // Делает свойство изменяемым
  property: string;
}

// Хорошо: декораторы дополняют друг друга
class GoodExample {
  @validate(required())
  @format(trim(), lowercase())
  @trackChanges
  property: string;
}
```

## Ограничения и особенности

1. Порядок применения декораторов важен для правильной работы
2. Декораторы одного типа применяются в определенном порядке
3. При конфликте декораторов поведение может быть непредсказуемым
4. Сложная композиция может затруднить отладку

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/property-decorators|Декораторы свойств]]
- [[ts/decorators/accessor-decorators|Декораторы аксессоров]]
- [[ts/decorators/parameter-decorators|Декораторы параметров]]