# Декораторы параметров

Декораторы параметров применяются к параметрам методов и конструкторов класса и могут использоваться для наблюдения за параметрами или извлечения информации о них.

## Базовый синтаксис

Декоратор параметра объявляется перед параметром метода или конструктора:

```typescript
class MyClass {
  method(@decorator param: string) {
    // ...
  }
}
```

Функция декоратора параметра принимает три параметра:
1. `target` - прототип класса для статических методов или конструктор для методов экземпляра
2. `propertyKey` - имя метода (или undefined для конструктора)
3. `parameterIndex` - индекс параметра в списке параметров

```typescript
function parameterDecorator(
  target: any,
  propertyKey: string | undefined,
  parameterIndex: number
) {
  // Логика декоратора
}
```

## Простые примеры

### 1. Базовый декоратор параметра

```typescript
function logParameter(target: any, propertyKey: string | undefined, parameterIndex: number) {
  console.log(`Parameter at index ${parameterIndex} of method ${propertyKey || 'constructor'} is decorated`);
}

class User {
  constructor(@logParameter name: string, @logParameter age: number) {
    console.log(`Creating user: ${name}, age: ${age}`);
  }
  
  greet(@logParameter message: string) {
    console.log(`Greeting: ${message}`);
  }
}

// Выведет:
// "Parameter at index 0 of method constructor is decorated"
// "Parameter at index 1 of method constructor is decorated"
// "Parameter at index 0 of method greet is decorated"

const user = new User("John", 30);
user.greet("Hello");
```

### 2. Декоратор для валидации обязательных параметров

```typescript
const requiredParameters: Map<any, Map<string | undefined, number[]>> = new Map();

function required(target: any, propertyKey: string | undefined, parameterIndex: number) {
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
        if (args[index] === undefined || args[index] === null) {
          throw new Error(`Parameter at index ${index} of method ${propertyKey} is required`);
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

class UserService {
  @validate
  createUser(@required name: string, @required email: string, age?: number) {
    console.log(`Creating user: ${name}, email: ${email}, age: ${age}`);
  }
  
  @validate
  updateUser(@required id: string, @required data: any) {
    console.log(`Updating user ${id} with:`, data);
  }
}

const service = new UserService();
service.createUser("John", "john@example.com", 30); // ✅ Валидация пройдена
service.createUser("Jane", "jane@example.com"); // ✅ Валидация пройдена (age опциональный)

// service.createUser("John", undefined); // ❌ Ошибка: Parameter at index 1 of method createUser is required
// service.updateUser(undefined, { name: "John" }); // ❌ Ошибка: Parameter at index 0 of method updateUser is required
```

## Продвинутые примеры

### 1. Декоратор для типизации параметров

```typescript
const parameterTypes: Map<any, Map<string | undefined, Map<number, string>>> = new Map();

function type(expectedType: string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    if (!parameterTypes.has(constructor)) {
      parameterTypes.set(constructor, new Map());
    }
    
    const methodMap = parameterTypes.get(constructor)!;
    
    if (!methodMap.has(propertyKey)) {
      methodMap.set(propertyKey, new Map());
    }
    
    methodMap.get(propertyKey)!.set(parameterIndex, expectedType);
  };
}

function validateTypes(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const constructor = target.constructor;
    const methodMap = parameterTypes.get(constructor);
    
    if (methodMap && methodMap.has(propertyKey)) {
      const typeMap = methodMap.get(propertyKey)!;
      
      for (const [index, expectedType] of typeMap) {
        if (index < args.length) {
          const actualType = typeof args[index];
          if (actualType !== expectedType) {
            throw new Error(`Parameter at index ${index} of method ${propertyKey} expected type ${expectedType}, got ${actualType}`);
          }
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

class Calculator {
  @validateTypes
  add(@type("number") a: number, @type("number") b: number): number {
    return a + b;
  }
  
  @validateTypes
  greet(@type("string") name: string, @type("number") age: number): string {
    return `Hello, ${name}! You are ${age} years old.`;
  }
}

const calc = new Calculator();
console.log(calc.add(2, 3)); // 5
console.log(calc.greet("John", 30)); // "Hello, John! You are 30 years old."

// calc.add("2", 3); // ❌ Ошибка: Parameter at index 0 of method add expected type number, got string
// calc.greet(123, 30); // ❌ Ошибка: Parameter at index 0 of method greet expected type string, got number
```

### 2. Декоратор для логирования параметров

```typescript
function logParameters(target: any, propertyKey: string | undefined, parameterIndex: number) {
  // Этот декоратор просто помечает параметры для логирования
  // Фактическая логика будет в декораторе метода
}

function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling method ${propertyKey} with parameters:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Method ${propertyKey} returned:`, result);
    return result;
  };
}

class ApiClient {
  @logMethod
  fetchUser(@logParameters id: string, @logParameters includeDetails: boolean = false) {
    return {
      id,
      name: `User ${id}`,
      ...(includeDetails && { email: `${id}@example.com` })
    };
  }
  
  @logMethod
  createUser(@logParameters userData: { name: string; email: string }) {
    return {
      id: `user_${Date.now()}`,
      ...userData
    };
  }
}

const client = new ApiClient();
client.fetchUser("123", true);
// Выведет:
// "Calling method fetchUser with parameters: ['123', true]"
// "Method fetchUser returned: { id: '123', name: 'User 123', email: '123@example.com' }"

client.createUser({ name: "John", email: "john@example.com" });
// Выведет:
// "Calling method createUser with parameters: [{ name: 'John', email: 'john@example.com' }]"
// "Method createUser returned: { id: 'user_1234567890', name: 'John', email: 'john@example.com' }"
```

### 3. Декоратор для инъекции зависимостей

```typescript
const dependencies: Map<string, any> = new Map();

function registerService(name: string, service: any) {
  dependencies.set(name, service);
}

function inject(token: string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    // Этот декоратор помечает параметр для инъекции зависимости
    // Фактическая инъекция будет в декораторе класса
  };
}

function injectable(target: any) {
  const originalConstructor = target;
  
  const newConstructor: any = function(...args: any[]) {
    // Получаем параметры конструктора, которые нужно инъектить
    // В реальной реализации это делается через рефлексию или метаданные
    const injectedArgs = [...args];
    
    // Здесь упрощенная реализация инъекции
    // В реальности нужно использовать рефлексию для получения типов параметров
    return new originalConstructor(...injectedArgs);
  };
  
  newConstructor.prototype = originalConstructor.prototype;
  return newConstructor;
}

// Пример сервисов
class Logger {
  log(message: string) {
    console.log(`[LOG] ${message}`);
  }
}

class Database {
  query(sql: string) {
    console.log(`Executing query: ${sql}`);
    return [{ id: 1, name: "John" }];
  }
}

// Регистрируем сервисы
registerService("logger", new Logger());
registerService("database", new Database());

@injectable
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

const userService = new UserService(new Logger(), new Database());
console.log(userService.getUsers());
// Выведет:
// "[LOG] Fetching users"
// "Executing query: SELECT * FROM users"
// [{ id: 1, name: "John" }]
```

## Практические примеры

### 1. Декоратор для валидации диапазонов параметров

```typescript
const parameterRanges: Map<any, Map<string | undefined, Map<number, { min: number; max: number }>>> = new Map();

function range(min: number, max: number) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    if (!parameterRanges.has(constructor)) {
      parameterRanges.set(constructor, new Map());
    }
    
    const methodMap = parameterRanges.get(constructor)!;
    
    if (!methodMap.has(propertyKey)) {
      methodMap.set(propertyKey, new Map());
    }
    
    methodMap.get(propertyKey)!.set(parameterIndex, { min, max });
  };
}

function validateRanges(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const constructor = target.constructor;
    const methodMap = parameterRanges.get(constructor);
    
    if (methodMap && methodMap.has(propertyKey)) {
      const rangeMap = methodMap.get(propertyKey)!;
      
      for (const [index, { min, max }] of rangeMap) {
        if (index < args.length) {
          const value = args[index];
          if (typeof value === "number" && (value < min || value > max)) {
            throw new Error(`Parameter at index ${index} of method ${propertyKey} must be between ${min} and ${max}`);
          }
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

class MathService {
  @validateRanges
  clamp(@range(0, 100) value: number, @range(0, 100) min: number, @range(0, 100) max: number): number {
    return Math.min(Math.max(value, min), max);
  }
  
  @validateRanges
  randomInt(@range(1, 1000) max: number): number {
    return Math.floor(Math.random() * max) + 1;
  }
}

const math = new MathService();
console.log(math.clamp(150, 0, 100)); // 100
console.log(math.randomInt(10)); // Случайное число от 1 до 10

// math.clamp(50, 0, 200); // ❌ Ошибка: Parameter at index 2 of method clamp must be between 0 and 100
// math.randomInt(2000); // ❌ Ошибка: Parameter at index 0 of method randomInt must be between 1 and 1000
```

### 2. Декоратор для форматирования строковых параметров

```typescript
const parameterFormatters: Map<any, Map<string | undefined, Map<number, (value: string) => string>>> = new Map();

function format(formatter: (value: string) => string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    if (!parameterFormatters.has(constructor)) {
      parameterFormatters.set(constructor, new Map());
    }
    
    const methodMap = parameterFormatters.get(constructor)!;
    
    if (!methodMap.has(propertyKey)) {
      methodMap.set(propertyKey, new Map());
    }
    
    methodMap.get(propertyKey)!.set(parameterIndex, formatter);
  };
}

function applyFormatting(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const constructor = target.constructor;
    const methodMap = parameterFormatters.get(constructor);
    
    if (methodMap && methodMap.has(propertyKey)) {
      const formatterMap = methodMap.get(propertyKey)!;
      
      const formattedArgs = [...args];
      for (const [index, formatter] of formatterMap) {
        if (index < formattedArgs.length && typeof formattedArgs[index] === "string") {
          formattedArgs[index] = formatter(formattedArgs[index]);
        }
      }
      
      return originalMethod.apply(this, formattedArgs);
    }
    
    return originalMethod.apply(this, args);
  };
}

class TextService {
  @applyFormatting
  processText(
    @format((s: string) => s.trim().toLowerCase()) text: string,
    @format((s: string) => s.toUpperCase()) format: string
  ): string {
    return `${format}: ${text}`;
  }
  
  @applyFormatting
  createSlug(@format((s: string) => s.trim().toLowerCase().replace(/\s+/g, '-')) title: string): string {
    return title;
  }
}

const textService = new TextService();
console.log(textService.processText("  Hello World  ", "formatted")); // "FORMATTED: hello world"
console.log(textService.createSlug("  My Blog Post Title  ")); // "my-blog-post-title"
```

### 3. Декоратор для отслеживания использования параметров

```typescript
const parameterUsage: Map<any, Map<string | undefined, Map<number, { name: string; count: number }>>> = new Map();

function trackParameter(name: string) {
  return function(target: any, propertyKey: string | undefined, parameterIndex: number) {
    const constructor = target.constructor;
    
    if (!parameterUsage.has(constructor)) {
      parameterUsage.set(constructor, new Map());
    }
    
    const methodMap = parameterUsage.get(constructor)!;
    
    if (!methodMap.has(propertyKey)) {
      methodMap.set(propertyKey, new Map());
    }
    
    methodMap.get(propertyKey)!.set(parameterIndex, { name, count: 0 });
  };
}

function trackUsage(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const constructor = target.constructor;
    const methodMap = parameterUsage.get(constructor);
    
    if (methodMap && methodMap.has(propertyKey)) {
      const usageMap = methodMap.get(propertyKey)!;
      
      for (const [index, usage] of usageMap) {
        if (index < args.length && args[index] !== undefined) {
          usage.count++;
          console.log(`Parameter ${usage.name} (index ${index}) of method ${propertyKey} used. Total usage: ${usage.count}`);
        }
      }
    }
    
    return originalMethod.apply(this, args);
  };
}

class OrderService {
  @trackUsage
  createOrder(
    @trackParameter("productId") productId: string,
    @trackParameter("quantity") quantity: number,
    @trackParameter("customerId") customerId?: string
  ) {
    return {
      id: `order_${Date.now()}`,
      productId,
      quantity,
      customerId: customerId || "anonymous"
    };
  }
  
  @trackUsage
  updateOrder(
    @trackParameter("orderId") orderId: string,
    @trackParameter("status") status: string
  ) {
    console.log(`Updating order ${orderId} status to ${status}`);
  }
}

const orderService = new OrderService();
orderService.createOrder("product123", 5); 
// Выведет: "Parameter productId (index 0) of method createOrder used. Total usage: 1"
// Выведет: "Parameter quantity (index 1) of method createOrder used. Total usage: 1"

orderService.createOrder("product456", 3, "customer789");
// Выведет: "Parameter productId (index 0) of method createOrder used. Total usage: 2"
// Выведет: "Parameter quantity (index 1) of method createOrder used. Total usage: 2"
// Выведет: "Parameter customerId (index 2) of method createOrder used. Total usage: 1"

orderService.updateOrder("order123", "shipped");
// Выведет: "Parameter orderId (index 0) of method updateOrder used. Total usage: 1"
// Выведет: "Parameter status (index 1) of method updateOrder used. Total usage: 1"
```

## Ограничения и особенности

1. Декораторы параметров применяются во время определения класса
2. Декораторы параметров не могут напрямую изменять поведение параметров
3. Для изменения поведения параметров необходимо использовать декораторы методов или классов
4. Декораторы параметров主要用于收集信息，实际的逻辑处理通常在方法或类装饰器中完成

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/method-decorators|Декораторы методов]]
- [[ts/decorators/property-decorators|Декораторы свойств]]
- [[ts/decorators/accessor-decorators|Декораторы аксессоров]]