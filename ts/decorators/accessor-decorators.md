# Декораторы аксессоров

Декораторы аксессоров применяются к геттерам и сеттерам свойств и могут использоваться для наблюдения, изменения или замены определения аксессора.

## Базовый синтаксис

Декоратор аксессора объявляется justo перед объявлением геттера или сеттера:

```typescript
class MyClass {
  private _value: string;
  
  @decorator
  get value() {
    return this._value;
  }
  
  @decorator
  set value(val: string) {
    this._value = val;
  }
}
```

Функция декоратора аксессора принимает три параметра:
1. `target` - прототип класса для статических аксессоров или конструктор для аксессоров экземпляра
2. `propertyKey` - имя свойства
3. `descriptor` - дескриптор свойства аксессора

```typescript
function accessorDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // Логика декоратора
}
```

## Простые примеры

### 1. Базовый декоратор аксессора

```typescript
function logAccessor(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  console.log(`Accessor ${propertyKey} is declared in class ${target.constructor.name}`);
}

class User {
  private _name: string;
  
  constructor(name: string) {
    this._name = name;
  }
  
  @logAccessor
  get name(): string {
    return this._name;
  }
  
  @logAccessor
  set name(value: string) {
    this._name = value;
  }
}

// Выведет:
// "Accessor name is declared in class User"
// "Accessor name is declared in class User"

const user = new User("John");
console.log(user.name); // "John"
user.name = "Jane";
console.log(user.name); // "Jane"
```

### 2. Декоратор для валидации аксессоров

```typescript
function validateStringLength(min: number, max: number) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalSetter = descriptor.set;
    
    if (originalSetter) {
      descriptor.set = function(value: string) {
        if (value.length < min || value.length > max) {
          throw new Error(`Value for ${propertyKey} must be between ${min} and ${max} characters`);
        }
        originalSetter.call(this, value);
      };
    }
  };
}

class Product {
  private _title: string = "";
  
  @validateStringLength(1, 100)
  set title(value: string) {
    this._title = value;
  }
  
  get title(): string {
    return this._title;
  }
}

const product = new Product();
product.title = "Valid Product Title"; // ✅ Валидация пройдена
console.log(product.title); // "Valid Product Title"

// product.title = ""; // ❌ Ошибка: Value for title must be between 1 and 100 characters
// product.title = "A".repeat(101); // ❌ Ошибка: Value for title must be between 1 and 100 characters
```

## Продвинутые примеры

### 1. Декоратор для кэширования геттеров

```typescript
function cached(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalGetter = descriptor.get;
  const cachedKey = `__${propertyKey}_cached_value`;
  const cachedTimestampKey = `__${propertyKey}_cached_timestamp`;
  
  if (originalGetter) {
    descriptor.get = function() {
      const now = Date.now();
      const cachedTimestamp = (this as any)[cachedTimestampKey];
      
      // Кэшируем на 5 секунд
      if (cachedTimestamp && now - cachedTimestamp < 5000) {
        return (this as any)[cachedKey];
      }
      
      const value = originalGetter.call(this);
      (this as any)[cachedKey] = value;
      (this as any)[cachedTimestampKey] = now;
      
      return value;
    };
  }
}

class DataProvider {
  private _dataCalls = 0;
  
  @cached
  get expensiveData(): string {
    this._dataCalls++;
    console.log(`Fetching data... Call #${this._dataCalls}`);
    return `Expensive data fetched at ${new Date().toISOString()}`;
  }
}

const provider = new DataProvider();
console.log(provider.expensiveData); // "Fetching data... Call #1" затем данные
console.log(provider.expensiveData); // Данные из кэша (без повторного вызова)

// Ждем 6 секунд
setTimeout(() => {
  console.log(provider.expensiveData); // "Fetching data... Call #2" затем новые данные
}, 6000);
```

### 2. Декоратор для отслеживания изменений через сеттеры

```typescript
function trackChanges(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalSetter = descriptor.set;
  const historyKey = `__${propertyKey}_history`;
  
  // Инициализируем историю
  Object.defineProperty(target, historyKey, {
    value: [],
    writable: true,
    enumerable: false,
    configurable: true
  });
  
  if (originalSetter) {
    descriptor.set = function(value: any) {
      const oldValue = (this as any)[propertyKey];
      originalSetter.call(this, value);
      
      const history = (this as any)[historyKey];
      history.push({
        oldValue: oldValue,
        newValue: value,
        timestamp: new Date()
      });
    };
  }
  
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

class User {
  private _name: string = "";
  private _email: string = "";
  
  @trackChanges
  set name(value: string) {
    this._name = value;
  }
  
  get name(): string {
    return this._name;
  }
  
  @trackChanges
  set email(value: string) {
    this._email = value;
  }
  
  get email(): string {
    return this._email;
  }
}

const user = new User();
user.name = "John";
user.name = "Jane";
user.email = "john@example.com";

console.log(user.getNameHistory());
// [
//   { oldValue: "", newValue: "John", timestamp: 2023-01-01T00:00:00.000Z },
//   { oldValue: "John", newValue: "Jane", timestamp: 2023-01-01T00:00:01.000Z }
// ]

console.log(user.getEmailHistory());
// [
//   { oldValue: "", newValue: "john@example.com", timestamp: 2023-01-01T00:00:02.000Z }
// ]
```

### 3. Декоратор для создания readonly свойств

```typescript
function readonly(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.set = function() {
    throw new Error(`Cannot set readonly property ${propertyKey}`);
  };
}

class Config {
  private _apiUrl: string = "";
  
  constructor(apiUrl: string) {
    this._apiUrl = apiUrl;
  }
  
  @readonly
  get apiUrl(): string {
    return this._apiUrl;
  }
  
  // Сеттер будет перезаписан декоратором
  set apiUrl(value: string) {
    this._apiUrl = value;
  }
}

const config = new Config("https://api.example.com");
console.log(config.apiUrl); // "https://api.example.com"

// config.apiUrl = "https://new-api.example.com"; // ❌ Ошибка: Cannot set readonly property apiUrl
```

## Практические примеры

### 1. Декоратор для автоматической сериализации аксессоров

```typescript
const serializableAccessors: Map<any, string[]> = new Map();

function serializable(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const constructor = target.constructor;
  
  if (!serializableAccessors.has(constructor)) {
    serializableAccessors.set(constructor, []);
  }
  
  serializableAccessors.get(constructor)!.push(propertyKey);
  
  // Убеждаемся, что геттер перечисляем
  if (descriptor.get) {
    descriptor.enumerable = true;
  }
}

function serialize(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function() {
    const constructor = this.constructor;
    const props = serializableAccessors.get(constructor) || [];
    
    const serialized: any = {};
    for (const prop of props) {
      serialized[prop] = this[prop];
    }
    
    return JSON.stringify(serialized);
  };
}

class User {
  private _firstName: string;
  private _lastName: string;
  
  constructor(firstName: string, lastName: string) {
    this._firstName = firstName;
    this._lastName = lastName;
  }
  
  @serializable
  get firstName(): string {
    return this._firstName;
  }
  
  @serializable
  get lastName(): string {
    return this._lastName;
  }
  
  @serializable
  get fullName(): string {
    return `${this._firstName} ${this._lastName}`;
  }
  
  @serialize
  toJSON() {
    // Реализация будет заменена декоратором
  }
}

const user = new User("John", "Doe");
console.log(user.toJSON()); // {"firstName":"John","lastName":"Doe","fullName":"John Doe"}
```

### 2. Декоратор для валидации и форматирования данных

```typescript
function formatEmail(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalSetter = descriptor.set;
  
  if (originalSetter) {
    descriptor.set = function(value: string) {
      // Форматируем email в нижний регистр
      const formattedValue = value.toLowerCase().trim();
      
      // Проверяем валидность
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(formattedValue)) {
        throw new Error(`Invalid email format: ${value}`);
      }
      
      originalSetter.call(this, formattedValue);
    };
  }
}

function formatName(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalSetter = descriptor.set;
  
  if (originalSetter) {
    descriptor.set = function(value: string) {
      // Форматируем имя: первая буква заглавная, остальные строчные
      const formattedValue = value.trim().toLowerCase().replace(/\b\w/g, l => l.toUpperCase());
      originalSetter.call(this, formattedValue);
    };
  }
}

class Contact {
  private _email: string = "";
  private _name: string = "";
  
  @formatEmail
  set email(value: string) {
    this._email = value;
  }
  
  get email(): string {
    return this._email;
  }
  
  @formatName
  set name(value: string) {
    this._name = value;
  }
  
  get name(): string {
    return this._name;
  }
}

const contact = new Contact();
contact.email = "  JOHN@EXAMPLE.COM  "; // Будет отформатирован в "john@example.com"
contact.name = "  jOhN dOe  "; // Будет отформатирован в "John Doe"

console.log(contact.email); // "john@example.com"
console.log(contact.name);  // "John Doe"

// contact.email = "invalid-email"; // ❌ Ошибка: Invalid email format: invalid-email
```

### 3. Декоратор для создания computed свойств с зависимостями

```typescript
const computedDependencies: Map<any, Map<string, string[]>> = new Map();
const computedCache: Map<any, Map<string, { value: any; timestamp: number }>> = new Map();

function dependsOn(...dependencies: string[]) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const constructor = target.constructor;
    
    if (!computedDependencies.has(constructor)) {
      computedDependencies.set(constructor, new Map());
    }
    
    computedDependencies.get(constructor)!.set(propertyKey, dependencies);
    
    const originalGetter = descriptor.get;
    
    if (originalGetter) {
      descriptor.get = function() {
        const instance = this;
        
        // Инициализируем кэш для экземпляра
        if (!computedCache.has(instance)) {
          computedCache.set(instance, new Map());
        }
        
        const instanceCache = computedCache.get(instance)!;
        const now = Date.now();
        
        // Проверяем кэш
        if (instanceCache.has(propertyKey)) {
          const cached = instanceCache.get(propertyKey)!;
          // Кэш действует 1 секунду
          if (now - cached.timestamp < 1000) {
            return cached.value;
          }
        }
        
        // Вычисляем значение
        const value = originalGetter.call(this);
        
        // Сохраняем в кэш
        instanceCache.set(propertyKey, {
          value: value,
          timestamp: now
        });
        
        return value;
      };
    }
  };
}

function invalidateCache(target: any, propertyKey: string) {
  const originalSetter = Object.getOwnPropertyDescriptor(target, propertyKey)?.set;
  
  if (originalSetter) {
    Object.defineProperty(target, propertyKey, {
      set: function(value: any) {
        originalSetter.call(this, value);
        
        // Инвалидируем кэш computed свойств, зависящих от этого свойства
        const constructor = this.constructor;
        const depsMap = computedDependencies.get(constructor);
        
        if (depsMap) {
          for (const [computedProp, dependencies] of depsMap) {
            if (dependencies.includes(propertyKey)) {
              const instanceCache = computedCache.get(this);
              if (instanceCache) {
                instanceCache.delete(computedProp);
              }
            }
          }
        }
      },
      get: Object.getOwnPropertyDescriptor(target, propertyKey)?.get,
      enumerable: true,
      configurable: true
    });
  }
}

class Rectangle {
  private _width: number = 0;
  private _height: number = 0;
  
  @invalidateCache
  set width(value: number) {
    this._width = value;
  }
  
  get width(): number {
    return this._width;
  }
  
  @invalidateCache
  set height(value: number) {
    this._height = value;
  }
  
  get height(): number {
    return this._height;
  }
  
  @dependsOn("width", "height")
  get area(): number {
    console.log("Calculating area...");
    return this._width * this._height;
  }
  
  @dependsOn("width", "height")
  get perimeter(): number {
    console.log("Calculating perimeter...");
    return 2 * (this._width + this._height);
  }
}

const rect = new Rectangle();
rect.width = 5;
rect.height = 3;

console.log(rect.area); // "Calculating area..." затем 15
console.log(rect.area); // 15 (из кэша)

rect.width = 10; // Инвалидирует кэш area и perimeter
console.log(rect.area); // "Calculating area..." затем 30 (новое значение)
```

## Композиция декораторов аксессоров

Несколько декораторов могут применяться к одному аксессору:

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

class Example {
  private _value: string = "";
  
  @first()
  @second()
  get value(): string {
    return this._value;
  }
  
  set value(val: string) {
    this._value = val;
  }
}

const example = new Example();
console.log(example.value);
// Выведет:
// "first(): factory evaluated"
// "second(): factory evaluated"
// "second(): called"
// "first(): called"
```

## Ограничения и особенности

1. Декораторы аксессоров применяются к геттерам и сеттерам отдельно
2. Для свойства с геттером и сеттером декораторы применяются к обоим
3. Декораторы аксессоров получают тот же дескриптор, что и декораторы методов
4. При использовании декораторов аксессоров важно сохранять оригинальные геттеры и сеттеры

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/property-decorators|Декораторы свойств]]
- [[ts/decorators/parameter-decorators|Декораторы параметров]]