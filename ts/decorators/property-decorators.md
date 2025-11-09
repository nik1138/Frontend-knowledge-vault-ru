# Декораторы свойств

Декораторы свойств применяются к свойствам класса и могут использоваться для наблюдения за объявлением свойства или изменения его определения.

## Базовый синтаксис

Декоратор свойства объявляется justo перед объявлением свойства:

```typescript
class MyClass {
  @decorator
  property: string;
}
```

Функция декоратора свойства принимает два параметра:
1. `target` - прототип класса для статических свойств или конструктор для свойств экземпляра
2. `propertyKey` - имя свойства

```typescript
function propertyDecorator(target: any, propertyKey: string) {
  // Логика декоратора
}
```

## Простые примеры

### 1. Базовый декоратор свойства

```typescript
function logProperty(target: any, propertyKey: string) {
  console.log(`Property ${propertyKey} is declared in class ${target.constructor.name}`);
}

class User {
  @logProperty
  name: string;
  
  @logProperty
  age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

// Выведет:
// "Property name is declared in class User"
// "Property age is declared in class User"

const user = new User("John", 30);
```

### 2. Декоратор для установки значения по умолчанию

```typescript
function defaultValue(value: any) {
  return function(target: any, propertyKey: string) {
    Object.defineProperty(target, propertyKey, {
      value: value,
      writable: true,
      enumerable: true,
      configurable: true
    });
  };
}

class Product {
  @defaultValue("Untitled Product")
  title: string;
  
  @defaultValue(0)
  price: number;
  
  @defaultValue(true)
  inStock: boolean;
}

const product = new Product();
console.log(product.title);   // "Untitled Product"
console.log(product.price);   // 0
console.log(product.inStock); // true
```

## Продвинутые примеры

### 1. Декоратор для валидации свойств

```typescript
function required(target: any, propertyKey: string) {
  let value: any;
  
  const descriptor: PropertyDescriptor = {
    get() {
      return value;
    },
    set(newValue: any) {
      if (newValue === undefined || newValue === null) {
        throw new Error(`Property ${propertyKey} is required`);
      }
      value = newValue;
    },
    enumerable: true,
    configurable: true
  };
  
  Object.defineProperty(target, propertyKey, descriptor);
}

function minLength(min: number) {
  return function(target: any, propertyKey: string) {
    let value: string;
    
    const descriptor: PropertyDescriptor = {
      get() {
        return value;
      },
      set(newValue: string) {
        if (newValue && newValue.length < min) {
          throw new Error(`Property ${propertyKey} must be at least ${min} characters long`);
        }
        value = newValue;
      },
      enumerable: true,
      configurable: true
    };
    
    Object.defineProperty(target, propertyKey, descriptor);
  };
}

function range(min: number, max: number) {
  return function(target: any, propertyKey: string) {
    let value: number;
    
    const descriptor: PropertyDescriptor = {
      get() {
        return value;
      },
      set(newValue: number) {
        if (newValue < min || newValue > max) {
          throw new Error(`Property ${propertyKey} must be between ${min} and ${max}`);
        }
        value = newValue;
      },
      enumerable: true,
      configurable: true
    };
    
    Object.defineProperty(target, propertyKey, descriptor);
  };
}

class User {
  @required
  @minLength(2)
  name: string;
  
  @required
  @range(0, 150)
  age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

const user = new User("John", 30); // ✅ Валидация пройдена
console.log(user.name); // "John"
console.log(user.age);  // 30

// const invalidUser = new User("", 30); // ❌ Ошибка: Property name must be at least 2 characters long
// const invalidUser2 = new User("John", -5); // ❌ Ошибка: Property age must be between 0 and 150
```

### 2. Декоратор для отслеживания изменений свойств

```typescript
function trackChanges(target: any, propertyKey: string) {
  let value: any;
  
  const descriptor: PropertyDescriptor = {
    get() {
      console.log(`Getting value of ${propertyKey}:`, value);
      return value;
    },
    set(newValue: any) {
      console.log(`Setting ${propertyKey} from ${value} to ${newValue}`);
      value = newValue;
    },
    enumerable: true,
    configurable: true
  };
  
  Object.defineProperty(target, propertyKey, descriptor);
}

class BankAccount {
  @trackChanges
  balance: number = 0;
  
  @trackChanges
  accountHolder: string;
  
  constructor(accountHolder: string) {
    this.accountHolder = accountHolder;
  }
  
  deposit(amount: number) {
    this.balance += amount;
  }
  
  withdraw(amount: number) {
    this.balance -= amount;
  }
}

const account = new BankAccount("John Doe");
// Выведет: "Setting accountHolder from undefined to John Doe"

account.deposit(100);
// Выведет: "Getting value of balance: 0"
// Выведет: "Setting balance from 0 to 100"

console.log(account.balance);
// Выведет: "Getting value of balance: 100"
// Выведет: 100
```

### 3. Декоратор для создания вычисляемых свойств

```typescript
function computed(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const getter = descriptor.get;
  const cachedKey = `__${propertyKey}__cache`;
  
  if (getter) {
    descriptor.get = function() {
      if (!(cachedKey in this)) {
        Object.defineProperty(this, cachedKey, {
          value: getter.call(this),
          enumerable: false,
          writable: false,
          configurable: false
        });
      }
      return this[cachedKey];
    };
  }
  
  return descriptor;
}

class Rectangle {
  width: number;
  height: number;
  
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
  
  @computed
  get area() {
    console.log("Calculating area...");
    return this.width * this.height;
  }
  
  @computed
  get perimeter() {
    console.log("Calculating perimeter...");
    return 2 * (this.width + this.height);
  }
}

const rect = new Rectangle(5, 3);
console.log(rect.area); // "Calculating area..." затем 15
console.log(rect.area); // 15 (из кэша, без повторного вычисления)

console.log(rect.perimeter); // "Calculating perimeter..." затем 16
console.log(rect.perimeter); // 16 (из кэша, без повторного вычисления)
```

## Практические примеры

### 1. Декоратор для автоматической сериализации

```typescript
const serializableProperties: Map<any, string[]> = new Map();

function serializable(target: any, propertyKey: string) {
  const constructor = target.constructor;
  
  if (!serializableProperties.has(constructor)) {
    serializableProperties.set(constructor, []);
  }
  
  serializableProperties.get(constructor)!.push(propertyKey);
}

function serialize(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function() {
    const constructor = this.constructor;
    const props = serializableProperties.get(constructor) || [];
    
    const serialized: any = {};
    for (const prop of props) {
      serialized[prop] = this[prop];
    }
    
    return JSON.stringify(serialized);
  };
}

class User {
  @serializable
  name: string;
  
  @serializable
  email: string;
  
  // Это свойство не будет сериализовано
  password: string;
  
  constructor(name: string, email: string, password: string) {
    this.name = name;
    this.email = email;
    this.password = password;
  }
  
  @serialize
  toJSON() {
    // Реализация будет заменена декоратором
  }
}

const user = new User("John", "john@example.com", "secret123");
console.log(user.toJSON()); // {"name":"John","email":"john@example.com"}
```

### 2. Декоратор для создания свойств с ограничениями

```typescript
function positive(target: any, propertyKey: string) {
  let value: number;
  
  const descriptor: PropertyDescriptor = {
    get() {
      return value;
    },
    set(newValue: number) {
      if (newValue <= 0) {
        throw new Error(`Property ${propertyKey} must be positive`);
      }
      value = newValue;
    },
    enumerable: true,
    configurable: true
  };
  
  Object.defineProperty(target, propertyKey, descriptor);
}

function email(target: any, propertyKey: string) {
  let value: string;
  
  const descriptor: PropertyDescriptor = {
    get() {
      return value;
    },
    set(newValue: string) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (newValue && !emailRegex.test(newValue)) {
        throw new Error(`Property ${propertyKey} must be a valid email address`);
      }
      value = newValue;
    },
    enumerable: true,
    configurable: true
  };
  
  Object.defineProperty(target, propertyKey, descriptor);
}

class Order {
  @positive
  quantity: number;
  
  @positive
  price: number;
  
  @email
  customerEmail: string;
  
  constructor(quantity: number, price: number, customerEmail: string) {
    this.quantity = quantity;
    this.price = price;
    this.customerEmail = customerEmail;
  }
  
  get total() {
    return this.quantity * this.price;
  }
}

const order = new Order(5, 100, "customer@example.com"); // ✅ Валидация пройдена
console.log(order.total); // 500

// const invalidOrder = new Order(-1, 100, "customer@example.com"); // ❌ Ошибка: Property quantity must be positive
// const invalidOrder2 = new Order(5, 100, "invalid-email"); // ❌ Ошибка: Property customerEmail must be a valid email address
```

### 3. Декоратор для создания свойств с историей изменений

```typescript
function history(target: any, propertyKey: string) {
  const historyKey = `__${propertyKey}_history`;
  let value: any;
  
  // Инициализируем историю
  Object.defineProperty(target, historyKey, {
    value: [],
    writable: true,
    enumerable: false,
    configurable: true
  });
  
  const descriptor: PropertyDescriptor = {
    get() {
      return value;
    },
    set(newValue: any) {
      const history = (this as any)[historyKey];
      history.push({
        value: value,
        timestamp: new Date()
      });
      value = newValue;
    },
    enumerable: true,
    configurable: true
  };
  
  Object.defineProperty(target, propertyKey, descriptor);
  
  // Добавляем метод для получения истории
  Object.defineProperty(target, `get${propertyKey.charAt(0).toUpperCase() + propertyKey.slice(1)}History`, {
    value: function() {
      return (this as any)[historyKey];
    },
    enumerable: false,
    configurable: true,
    writable: true
  });
}

class Document {
  @history
  title: string;
  
  @history
  content: string;
  
  constructor(title: string, content: string) {
    this.title = title;
    this.content = content;
  }
}

const doc = new Document("Initial Title", "Initial content");
doc.title = "Updated Title";
doc.content = "Updated content";

console.log(doc.getTitleHistory());
// [
//   { value: undefined, timestamp: 2023-01-01T00:00:00.000Z },
//   { value: "Initial Title", timestamp: 2023-01-01T00:00:01.000Z }
// ]

console.log(doc.getContentHistory());
// [
//   { value: undefined, timestamp: 2023-01-01T00:00:00.000Z },
//   { value: "Initial content", timestamp: 2023-01-01T00:00:01.000Z }
// ]
```

## Ограничения и особенности

1. Декораторы свойств применяются во время определения класса
2. Декораторы свойств не могут напрямую изменять дескриптор свойства (в отличие от декораторов методов)
3. Для изменения поведения свойства необходимо использовать `Object.defineProperty`
4. Декораторы свойств не получают дескриптор свойства в качестве параметра

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/parameter-decorators|Декораторы параметров]]
- [[ts/decorators/accessor-decorators|Декораторы аксессоров]]