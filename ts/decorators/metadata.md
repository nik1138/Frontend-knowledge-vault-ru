# Метаданные и рефлексия

Метаданные и рефлексия в TypeScript позволяют извлекать информацию о типах и структуре кода во время выполнения. Это мощный механизм, который используется в декораторах для создания сложных фреймворков и библиотек.

## Введение в рефлексию

Рефлексия - это возможность программы исследовать и изменять свою структуру и поведение во время выполнения. В TypeScript рефлексия реализуется через API `reflect-metadata`.

## Установка и настройка

Для работы с метаданными необходимо установить пакет:

```bash
npm install reflect-metadata
```

И добавить в начало вашего файла:

```typescript
import "reflect-metadata";
```

## Базовые примеры

### 1. Получение типов параметров

```typescript
import "reflect-metadata";

class UserService {
  getUser(id: string, includeDetails: boolean): any {
    return { id, name: `User ${id}`, ...(includeDetails && { email: `${id}@example.com` }) };
  }
}

// Получение типов параметров метода
const paramTypes = Reflect.getMetadata("design:paramtypes", UserService.prototype, "getUser");
console.log(paramTypes); // [String, Boolean]

// Получение типа возвращаемого значения
const returnType = Reflect.getMetadata("design:returntype", UserService.prototype, "getUser");
console.log(returnType); // Object

// Получение типа объекта
const type = Reflect.getMetadata("design:type", UserService);
console.log(type); // Function
```

### 2. Декораторы с автоматическим извлечением типов

```typescript
import "reflect-metadata";

function validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const paramTypes = Reflect.getMetadata("design:paramtypes", target, propertyKey);
  
  descriptor.value = function(...args: any[]) {
    if (paramTypes) {
      for (let i = 0; i < args.length; i++) {
        const arg = args[i];
        const expectedType = paramTypes[i];
        
        if (expectedType) {
          // Для примитивных типов
          if (expectedType === String && typeof arg !== "string") {
            throw new Error(`Parameter ${i} of method ${propertyKey} expected string, got ${typeof arg}`);
          }
          
          if (expectedType === Number && typeof arg !== "number") {
            throw new Error(`Parameter ${i} of method ${propertyKey} expected number, got ${typeof arg}`);
          }
          
          if (expectedType === Boolean && typeof arg !== "boolean") {
            throw new Error(`Parameter ${i} of method ${propertyKey} expected boolean, got ${typeof arg}`);
          }
          
          // Для классов
          if (typeof expectedType === "function" && expectedType !== String && expectedType !== Number && expectedType !== Boolean) {
            if (!(arg instanceof expectedType)) {
              throw new Error(`Parameter ${i} of method ${propertyKey} expected instance of ${expectedType.name}`);
            }
          }
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

class User {
  constructor(public name: string, public age: number) {}
}

class Order {
  constructor(public userId: string, public amount: number) {}
}

class Service {
  @validate
  processUser(user: User) {
    console.log(`Processing user: ${user.name}, age: ${user.age}`);
  }
  
  @validate
  processOrder(order: Order) {
    console.log(`Processing order: ${order.userId}, amount: ${order.amount}`);
  }
  
  @validate
  calculateTotal(price: number, tax: number): number {
    return price * (1 + tax);
  }
}

const service = new Service();
service.processUser(new User("John", 30)); // ✅ Валидация пройдена
service.processOrder(new Order("user123", 100)); // ✅ Валидация пройдена
console.log(service.calculateTotal(100, 0.1)); // ✅ Валидация пройдена

// service.processUser({ name: "John", age: 30 }); // ❌ Ошибка: Parameter 0 expected instance of User
// service.calculateTotal("100", 0.1); // ❌ Ошибка: Parameter 0 expected number, got string
```

## Продвинутые примеры

### 1. Создание ORM с использованием метаданных

```typescript
import "reflect-metadata";

// Декораторы для ORM
function Entity(tableName?: string) {
  return function(constructor: Function) {
    Reflect.defineMetadata("entity:table", tableName || constructor.name.toLowerCase(), constructor);
  };
}

function Column(columnName?: string, options: { type?: string; nullable?: boolean } = {}) {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    // Получаем существующие колонки или создаем новый массив
    const columns = Reflect.getMetadata("entity:columns", constructor) || [];
    
    columns.push({
      property: propertyKey,
      column: columnName || propertyKey,
      type: options.type || Reflect.getMetadata("design:type", target, propertyKey)?.name.toLowerCase(),
      nullable: options.nullable !== false
    });
    
    Reflect.defineMetadata("entity:columns", columns, constructor);
  };
}

function PrimaryKey() {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    Reflect.defineMetadata("entity:primarykey", propertyKey, constructor);
  };
}

function OneToMany(targetEntity: string) {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const relations = Reflect.getMetadata("entity:relations", constructor) || [];
    relations.push({
      property: propertyKey,
      targetEntity: targetEntity,
      type: "oneToMany"
    });
    
    Reflect.defineMetadata("entity:relations", relations, constructor);
  };
}

// Модели
@Entity("users")
class User {
  @PrimaryKey()
  @Column("id", { type: "string" })
  id: string;
  
  @Column("name", { type: "string" })
  name: string;
  
  @Column("email", { type: "string" })
  email: string;
  
  @Column("age", { type: "number", nullable: true })
  age?: number;
  
  @OneToMany("Order")
  orders: Order[];
}

@Entity("orders")
class Order {
  @PrimaryKey()
  @Column("id", { type: "string" })
  id: string;
  
  @Column("user_id", { type: "string" })
  userId: string;
  
  @Column("amount", { type: "number" })
  amount: number;
  
  @Column("created_at", { type: "date" })
  createdAt: Date;
}

// ORM сервис
class ORMService {
  static getEntityInfo(constructor: Function): any {
    return {
      table: Reflect.getMetadata("entity:table", constructor),
      columns: Reflect.getMetadata("entity:columns", constructor) || [],
      primaryKey: Reflect.getMetadata("entity:primarykey", constructor),
      relations: Reflect.getMetadata("entity:relations", constructor) || []
    };
  }
  
  static generateSchema(constructor: Function): string {
    const info = this.getEntityInfo(constructor);
    let schema = `CREATE TABLE ${info.table} (\n`;
    
    const columns = info.columns.map((col: any) => {
      let columnDef = `  ${col.column} ${col.type.toUpperCase()}`;
      if (!col.nullable) {
        columnDef += " NOT NULL";
      }
      if (col.property === info.primaryKey) {
        columnDef += " PRIMARY KEY";
      }
      return columnDef;
    });
    
    schema += columns.join(",\n");
    schema += "\n);";
    
    return schema;
  }
}

// Использование
console.log(ORMService.getEntityInfo(User));
// {
//   table: "users",
//   columns: [
//     { property: "id", column: "id", type: "string", nullable: true },
//     { property: "name", column: "name", type: "string", nullable: true },
//     { property: "email", column: "email", type: "string", nullable: true },
//     { property: "age", column: "age", type: "number", nullable: true }
//   ],
//   primaryKey: "id",
//   relations: [
//     { property: "orders", targetEntity: "Order", type: "oneToMany" }
//   ]
// }

console.log(ORMService.generateSchema(User));
// CREATE TABLE users (
//   id STRING NOT NULL PRIMARY KEY,
//   name STRING NOT NULL,
//   email STRING NOT NULL,
//   age NUMBER
// );

console.log(ORMService.generateSchema(Order));
// CREATE TABLE orders (
//   id STRING NOT NULL PRIMARY KEY,
//   user_id STRING NOT NULL,
//   amount NUMBER NOT NULL,
//   created_at DATE NOT NULL
// );
```

### 2. Создание DI контейнера с рефлексией

```typescript
import "reflect-metadata";

// Декораторы для DI
function Injectable() {
  return function(constructor: Function) {
    // Помечаем класс как инъектируемый
    Reflect.defineMetadata("di:injectable", true, constructor);
  };
}

function Inject(token?: string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    const tokens = Reflect.getMetadata("di:inject", constructor) || [];
    tokens[parameterIndex] = token;
    
    Reflect.defineMetadata("di:inject", tokens, constructor);
  };
}

// DI контейнер
class DIContainer {
  private static instances = new Map<string, any>();
  private static bindings = new Map<string, any>();
  
  static bind(token: string, implementation: any) {
    this.bindings.set(token, implementation);
  }
  
  static get<T>(token: string): T {
    if (this.instances.has(token)) {
      return this.instances.get(token);
    }
    
    const implementation = this.bindings.get(token);
    if (!implementation) {
      throw new Error(`No binding found for token: ${token}`);
    }
    
    const instance = this.resolve(implementation);
    this.instances.set(token, instance);
    return instance;
  }
  
  static resolve<T>(constructor: new (...args: any[]) => T): T {
    // Получаем токены для инъекции
    const tokens = Reflect.getMetadata("di:inject", constructor) || [];
    const paramTypes = Reflect.getMetadata("design:paramtypes", constructor) || [];
    
    // Разрешаем зависимости
    const injections = paramTypes.map((paramType: any, index: number) => {
      const token = tokens[index] || paramType.name;
      return this.get(token);
    });
    
    return new constructor(...injections);
  }
}

// Сервисы
@Injectable()
class Logger {
  log(message: string) {
    console.log(`[LOG] ${message}`);
  }
}

@Injectable()
class Database {
  constructor(@Inject("Logger") private logger: Logger) {}
  
  query(sql: string) {
    this.logger.log(`Executing query: ${sql}`);
    return [{ id: 1, name: "John" }];
  }
}

@Injectable()
class UserService {
  constructor(
    @Inject("Logger") private logger: Logger,
    @Inject("Database") private database: Database
  ) {
    this.logger.log("UserService created");
  }
  
  getUsers() {
    this.logger.log("Fetching users");
    return this.database.query("SELECT * FROM users");
  }
}

// Настройка DI
DIContainer.bind("Logger", Logger);
DIContainer.bind("Database", Database);
DIContainer.bind("UserService", UserService);

// Использование
const userService = DIContainer.get<UserService>("UserService");
console.log(userService.getUsers());
// [LOG] UserService created
// [LOG] Fetching users
// [LOG] Executing query: SELECT * FROM users
// [{ id: 1, name: "John" }]
```

### 3. Создание валидатора с метаданными

```typescript
import "reflect-metadata";

// Декораторы для валидации
function Required() {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const validations = Reflect.getMetadata("validation:rules", constructor) || {};
    if (!validations[propertyKey]) {
      validations[propertyKey] = [];
    }
    
    validations[propertyKey].push({ rule: "required" });
    Reflect.defineMetadata("validation:rules", validations, constructor);
  };
}

function MinLength(length: number) {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const validations = Reflect.getMetadata("validation:rules", constructor) || {};
    if (!validations[propertyKey]) {
      validations[propertyKey] = [];
    }
    
    validations[propertyKey].push({ rule: "minLength", value: length });
    Reflect.defineMetadata("validation:rules", validations, constructor);
  };
}

function MaxLength(length: number) {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const validations = Reflect.getMetadata("validation:rules", constructor) || {};
    if (!validations[propertyKey]) {
      validations[propertyKey] = [];
    }
    
    validations[propertyKey].push({ rule: "maxLength", value: length });
    Reflect.defineMetadata("validation:rules", validations, constructor);
  };
}

function Email() {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const validations = Reflect.getMetadata("validation:rules", constructor) || {};
    if (!validations[propertyKey]) {
      validations[propertyKey] = [];
    }
    
    validations[propertyKey].push({ rule: "email" });
    Reflect.defineMetadata("validation:rules", validations, constructor);
  };
}

// Валидатор
class Validator {
  static validate<T extends object>(instance: T): { isValid: boolean; errors: string[] } {
    const constructor = instance.constructor;
    const validations = Reflect.getMetadata("validation:rules", constructor) || {};
    const errors: string[] = [];
    
    for (const propertyKey in validations) {
      const rules = validations[propertyKey];
      const value = (instance as any)[propertyKey];
      
      for (const rule of rules) {
        switch (rule.rule) {
          case "required":
            if (value === undefined || value === null || value === "") {
              errors.push(`Property ${propertyKey} is required`);
            }
            break;
            
          case "minLength":
            if (typeof value === "string" && value.length < rule.value) {
              errors.push(`Property ${propertyKey} must be at least ${rule.value} characters long`);
            }
            break;
            
          case "maxLength":
            if (typeof value === "string" && value.length > rule.value) {
              errors.push(`Property ${propertyKey} must be no more than ${rule.value} characters long`);
            }
            break;
            
          case "email":
            if (typeof value === "string") {
              const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
              if (!emailRegex.test(value)) {
                errors.push(`Property ${propertyKey} must be a valid email address`);
              }
            }
            break;
        }
      }
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
}

// Модель с валидацией
class User {
  @Required()
  @MinLength(2)
  @MaxLength(50)
  name: string;
  
  @Required()
  @Email()
  email: string;
  
  @MinLength(6)
  password: string;
  
  constructor(name: string, email: string, password: string) {
    this.name = name;
    this.email = email;
    this.password = password;
  }
}

// Использование валидатора
const validUser = new User("John Doe", "john@example.com", "password123");
const invalidUser = new User("", "invalid-email", "123");

console.log(Validator.validate(validUser));
// { isValid: true, errors: [] }

console.log(Validator.validate(invalidUser));
// { 
//   isValid: false, 
//   errors: [
//     "Property name is required",
//     "Property email must be a valid email address",
//     "Property password must be at least 6 characters long"
//   ] 
// }
```

## Практические примеры

### 1. Создание сериализатора с метаданными

```typescript
import "reflect-metadata";

// Декораторы для сериализации
function Serializable(name?: string) {
  return function(constructor: Function) {
    Reflect.defineMetadata("serializable:name", name || constructor.name, constructor);
  };
}

function SerializeAs(name: string) {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const mappings = Reflect.getMetadata("serializable:mappings", constructor) || {};
    mappings[propertyKey] = name;
    Reflect.defineMetadata("serializable:mappings", mappings, constructor);
  };
}

function Ignore() {
  return function(target: any, propertyKey: string) {
    const constructor = target.constructor;
    
    const ignored = Reflect.getMetadata("serializable:ignored", constructor) || [];
    ignored.push(propertyKey);
    Reflect.defineMetadata("serializable:ignored", ignored, constructor);
  };
}

// Сериализатор
class Serializer {
  static serialize<T extends object>(instance: T): string {
    const constructor = instance.constructor;
    const mappings = Reflect.getMetadata("serializable:mappings", constructor) || {};
    const ignored = Reflect.getMetadata("serializable:ignored", constructor) || [];
    
    const serialized: any = {};
    
    for (const propertyKey in instance) {
      if (ignored.includes(propertyKey)) {
        continue;
      }
      
      const serializedKey = mappings[propertyKey] || propertyKey;
      serialized[serializedKey] = (instance as any)[propertyKey];
    }
    
    return JSON.stringify(serialized);
  }
  
  static deserialize<T>(json: string, constructor: new () => T): T {
    const data = JSON.parse(json);
    const instance = new constructor();
    const mappings = Reflect.getMetadata("serializable:mappings", constructor) || {};
    
    // Создаем обратное отображение
    const reverseMappings: Record<string, string> = {};
    for (const key in mappings) {
      reverseMappings[mappings[key]] = key;
    }
    
    for (const key in data) {
      const propertyKey = reverseMappings[key] || key;
      (instance as any)[propertyKey] = data[key];
    }
    
    return instance;
  }
}

// Модель с сериализацией
@Serializable("user")
class User {
  @SerializeAs("user_name")
  name: string;
  
  @SerializeAs("user_email")
  email: string;
  
  @Ignore()
  password: string;
  
  createdAt: Date;
  
  constructor(name: string, email: string, password: string) {
    this.name = name;
    this.email = email;
    this.password = password;
    this.createdAt = new Date();
  }
}

// Использование сериализатора
const user = new User("John Doe", "john@example.com", "secret123");
const serialized = Serializer.serialize(user);
console.log(serialized);
// {"user_name":"John Doe","user_email":"john@example.com","createdAt":"2023-01-01T00:00:00.000Z"}

const deserialized = Serializer.deserialize(serialized, User);
console.log(deserialized);
// User { name: "John Doe", email: "john@example.com", password: undefined, createdAt: Date }
```

### 2. Создание маршрутизатора с метаданными

```typescript
import "reflect-metadata";

// Декораторы для маршрутизации
function Controller(basePath: string) {
  return function(constructor: Function) {
    Reflect.defineMetadata("controller:basepath", basePath, constructor);
  };
}

function Route(method: string, path: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const constructor = target.constructor;
    
    const routes = Reflect.getMetadata("controller:routes", constructor) || [];
    routes.push({
      method: method.toUpperCase(),
      path,
      handler: propertyKey
    });
    
    Reflect.defineMetadata("controller:routes", routes, constructor);
  };
}

function Param(name: string) {
  return function(target: any, propertyKey: string, parameterIndex: number) {
    const constructor = target.constructor;
    
    const params = Reflect.getMetadata("controller:params", constructor) || {};
    if (!params[propertyKey]) {
      params[propertyKey] = {};
    }
    
    params[propertyKey][parameterIndex] = name;
    Reflect.defineMetadata("controller:params", params, constructor);
  };
}

// Маршрутизатор
class Router {
  private routes: Array<{
    method: string;
    path: string;
    handler: Function;
    controller: any;
    params: Record<number, string>;
  }> = [];
  
  registerController(constructor: new () => any) {
    const instance = new constructor();
    const basePath = Reflect.getMetadata("controller:basepath", constructor) || "";
    const routes = Reflect.getMetadata("controller:routes", constructor) || [];
    const params = Reflect.getMetadata("controller:params", constructor) || {};
    
    for (const route of routes) {
      this.routes.push({
        method: route.method,
        path: basePath + route.path,
        handler: (instance as any)[route.handler],
        controller: instance,
        params: params[route.handler] || {}
      });
    }
  }
  
  handleRequest(method: string, path: string, params: Record<string, any> = {}) {
    const route = this.routes.find(r => 
      r.method === method.toUpperCase() && r.path === path
    );
    
    if (route) {
      // Преобразуем параметры в правильный порядок
      const orderedParams: any[] = [];
      for (const index in route.params) {
        const paramName = route.params[index];
        orderedParams[parseInt(index)] = params[paramName];
      }
      
      return route.handler.apply(route.controller, orderedParams);
    }
    
    return { error: "Route not found" };
  }
  
  getRoutes() {
    return this.routes.map(r => ({
      method: r.method,
      path: r.path,
      handler: r.handler.name
    }));
  }
}

// Контроллеры
@Controller("/api/users")
class UserController {
  @Route("GET", "/")
  getAllUsers() {
    return [{ id: "1", name: "John" }, { id: "2", name: "Jane" }];
  }
  
  @Route("GET", "/:id")
  getUserById(@Param("id") id: string) {
    return { id, name: `User ${id}` };
  }
  
  @Route("POST", "/")
  createUser(@Param("name") name: string, @Param("email") email: string) {
    return { id: "123", name, email };
  }
}

@Controller("/api/orders")
class OrderController {
  @Route("GET", "/")
  getAllOrders() {
    return [{ id: "1", userId: "1", amount: 100 }];
  }
  
  @Route("POST", "/")
  createOrder(@Param("userId") userId: string, @Param("amount") amount: number) {
    return { id: "123", userId, amount };
  }
}

// Использование маршрутизатора
const router = new Router();
router.registerController(UserController);
router.registerController(OrderController);

console.log("Registered routes:");
console.log(router.getRoutes());

console.log("\nHandling requests:");
console.log(router.handleRequest("GET", "/api/users/"));
console.log(router.handleRequest("GET", "/api/users/:id", { id: "123" }));
console.log(router.handleRequest("POST", "/api/users/", { name: "John", email: "john@example.com" }));
console.log(router.handleRequest("GET", "/api/orders/"));
console.log(router.handleRequest("POST", "/api/orders/", { userId: "123", amount: 100 }));
```

## Ограничения и особенности

1. Для работы с метаданными необходимо включить `emitDecoratorMetadata` в `tsconfig.json`:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

2. Метаданные доступны только для типов, которые могут быть представлены в runtime
3. Сложные generic типы могут не сохраняться в метаданных
4. Метаданные увеличивают размер бандла и могут замедлять выполнение

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/factories|Фабрики декораторов]]
- [[ts/decorators/composition|Композиция декораторов]]