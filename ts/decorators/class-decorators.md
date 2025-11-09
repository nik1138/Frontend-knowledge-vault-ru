# Декораторы классов

Декораторы классов применяются к конструктору класса и могут использоваться для наблюдения, изменения или замены определения класса.

## Базовый синтаксис

Декоратор класса объявляется justo перед объявлением класса:

```typescript
@decorator
class MyClass {
  // ...
}
```

Функция декоратора класса принимает один параметр - конструктор декорируемого класса:

```typescript
function classDecorator(constructor: Function) {
  // Логика декоратора
}
```

## Простые примеры

### 1. Базовый декоратор класса

```typescript
function logClass(constructor: Function) {
  console.log(`Class ${constructor.name} is being created`);
}

@logClass
class User {
  constructor(public name: string, public age: number) {}
}

// При создании экземпляра класса будет выведено:
// "Class User is being created"
const user = new User("John", 30);
```

### 2. Декоратор, добавляющий статические свойства

```typescript
function addMetadata(constructor: Function) {
  constructor.prototype.metadata = {
    createdAt: new Date(),
    version: "1.0.0"
  };
}

@addMetadata
class Product {
  constructor(public name: string, public price: number) {}
}

const product = new Product("Laptop", 999);
console.log((product as any).metadata); 
// { createdAt: 2023-01-01T00:00:00.000Z, version: "1.0.0" }
```

## Продвинутые примеры

### 1. Фабрика декораторов классов

```typescript
function configurable(isConfigurable: boolean) {
  return function (constructor: Function) {
    Object.defineProperty(constructor, 'configurable', {
      value: isConfigurable,
      writable: false,
      enumerable: false,
      configurable: false
    });
  };
}

@configurable(true)
class Service {
  constructor(public name: string) {}
}

console.log((Service as any).configurable); // true
```

### 2. Декоратор для реализации паттерна Singleton

```typescript
function singleton<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    static instance: InstanceType<T>;
    
    constructor(...args: any[]) {
      if (!(this.constructor as any).instance) {
        super(...args);
        (this.constructor as any).instance = this;
      }
      return (this.constructor as any).instance;
    }
  } as T;
}

@singleton
class DatabaseConnection {
  private connectionString: string;
  
  constructor(connectionString: string) {
    this.connectionString = connectionString;
    console.log(`Connecting to ${connectionString}`);
  }
  
  query(sql: string) {
    console.log(`Executing query: ${sql}`);
  }
}

const conn1 = new DatabaseConnection("localhost:5432");
const conn2 = new DatabaseConnection("localhost:5432");

console.log(conn1 === conn2); // true
```

### 3. Декоратор для добавления методов

```typescript
function addMethods(constructor: Function) {
  constructor.prototype.serialize = function() {
    return JSON.stringify(this);
  };
  
  constructor.prototype.clone = function() {
    const cloned = new (this.constructor)();
    Object.assign(cloned, this);
    return cloned;
  };
}

@addMethods
class User {
  constructor(public name: string, public age: number) {}
}

const user = new User("John", 30);
console.log(user.serialize()); // {"name":"John","age":30}

const clonedUser = user.clone();
console.log(clonedUser.name); // "John"
console.log(clonedUser === user); // false
```

## Практические примеры

### 1. Декоратор для валидации

```typescript
interface ValidationMetadata {
  requiredFields: string[];
}

function validate(requiredFields: string[]) {
  return function (constructor: Function) {
    const metadata: ValidationMetadata = {
      requiredFields
    };
    
    constructor.prototype.validationMetadata = metadata;
    
    // Добавляем метод валидации
    constructor.prototype.validate = function() {
      for (const field of requiredFields) {
        if (this[field] === undefined || this[field] === null) {
          throw new Error(`Field ${field} is required`);
        }
      }
      return true;
    };
  };
}

@validate(["name", "email"])
class User {
  constructor(public name: string, public email: string, public age?: number) {}
}

const user = new User("John", "john@example.com");
user.validate(); // ✅ Проходит валидацию

const invalidUser = new User("", "john@example.com");
// invalidUser.validate(); // ❌ Ошибка: Field name is required
```

### 2. Декоратор для логирования

```typescript
function logCreation(logLevel: "info" | "warn" | "error" = "info") {
  return function (constructor: Function) {
    const originalConstructor = constructor;
    
    // Заменяем конструктор на новый
    const newConstructor: any = function(...args: any[]) {
      console[logLevel](`Creating instance of ${originalConstructor.name}`);
      const instance = new originalConstructor(...args);
      console[logLevel](`Instance created:`, instance);
      return instance;
    };
    
    // Копируем прототип
    newConstructor.prototype = originalConstructor.prototype;
    
    return newConstructor;
  };
}

@logCreation("info")
class Service {
  constructor(public name: string) {}
}

const service = new Service("PaymentService");
// Выведет:
// "Creating instance of Service"
// "Instance created:" Service { name: "PaymentService" }
```

### 3. Декоратор для реализации паттерна Observer

```typescript
type ObserverCallback = (data: any) => void;

class Observable {
  private observers: ObserverCallback[] = [];
  
  subscribe(callback: ObserverCallback) {
    this.observers.push(callback);
  }
  
  notify(data: any) {
    this.observers.forEach(callback => callback(data));
  }
}

function observable<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    private observable = new Observable();
    
    subscribe(callback: ObserverCallback) {
      this.observable.subscribe(callback);
    }
    
    protected notifyObservers(data: any) {
      this.observable.notify(data);
    }
  } as T & { new(...args: any[]): { subscribe(callback: ObserverCallback): void } };
}

@observable
class UserService {
  private users: string[] = [];
  
  addUser(name: string) {
    this.users.push(name);
    (this as any).notifyObservers({ type: 'userAdded', user: name });
  }
}

const userService = new UserService();
userService.subscribe((data) => {
  console.log('Observer notified:', data);
});

userService.addUser("John");
// Выведет: "Observer notified: { type: 'userAdded', user: 'John' }"
```

## Композиция декораторов

Несколько декораторов могут применяться к одному классу:

```typescript
function first() {
  console.log("first(): factory evaluated");
  return function (constructor: Function) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (constructor: Function) {
    console.log("second(): called");
  };
}

@first()
@second()
class Example {
  constructor() {}
}

// Выведет:
// "first(): factory evaluated"
// "second(): factory evaluated"
// "second(): called"
// "first(): called"
```

## Ограничения и особенности

1. Декораторы классов применяются во время определения класса, а не во время создания экземпляра
2. Декораторы могут изменять конструктор класса или его прототип
3. При изменении конструктора важно сохранить прототип оригинального конструктора
4. Декораторы выполняются в порядке определения, но применяются в обратном порядке

## Связанные концепции

- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/property-decorators|Декораторы свойств]]
- [[ts/decorators/parameter-decorators|Декораторы параметров]]
- [[ts/decorators/accessor-decorators|Декораторы аксессоров]]