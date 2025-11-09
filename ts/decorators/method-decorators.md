# Декораторы методов

Декораторы методов применяются к методам класса и могут использоваться для наблюдения, изменения или замены определения метода.

## Базовый синтаксис

Декоратор метода объявляется justo перед объявлением метода:

```typescript
class MyClass {
  @decorator
  method() {
    // ...
  }
}
```

Функция декоратора метода принимает три параметра:
1. `target` - прототип класса для статических методов или конструктор для методов экземпляра
2. `propertyKey` - имя метода
3. `descriptor` - дескриптор свойства метода

```typescript
function methodDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // Логика декоратора
}
```

## Простые примеры

### 1. Базовый декоратор метода

```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with arguments:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Method ${propertyKey} returned:`, result);
    return result;
  };
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
  
  @log
  multiply(a: number, b: number): number {
    return a * b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Выведет:
// "Calling add with arguments: [2, 3]"
// "Method add returned: 5"

calc.multiply(4, 5);
// Выведет:
// "Calling multiply with arguments: [4, 5]"
// "Method multiply returned: 20"
```

### 2. Декоратор для измерения времени выполнения

```typescript
function timing(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const start = performance.now();
    const result = originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} executed in ${end - start} milliseconds`);
    return result;
  };
}

class DataProcessor {
  @timing
  processData(data: number[]): number[] {
    // Имитация сложной обработки
    return data.map(x => x * 2).filter(x => x > 10);
  }
}

const processor = new DataProcessor();
const result = processor.processData([1, 5, 10, 15, 20]);
// Выведет: "processData executed in X.XXX milliseconds"
```

## Продвинутые примеры

### 1. Фабрика декораторов методов

```typescript
function catchError(errorMessage: string = "An error occurred") {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      try {
        return originalMethod.apply(this, args);
      } catch (error) {
        console.error(`${errorMessage}:`, error);
        return null;
      }
    };
  };
}

class ApiService {
  @catchError("Failed to fetch user data")
  getUser(id: string): { id: string; name: string } {
    if (id === "invalid") {
      throw new Error("Invalid user ID");
    }
    return { id, name: `User ${id}` };
  }
}

const api = new ApiService();
const user = api.getUser("123"); // ✅ Работает нормально
const invalidUser = api.getUser("invalid"); // ❌ Ошибка перехвачена
// Выведет: "Failed to fetch user data: Error: Invalid user ID"
```

### 2. Декоратор для кэширования

```typescript
function cache(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const cacheMap = new Map<string, any>();
  
  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cacheMap.has(key)) {
      console.log(`Cache hit for ${propertyKey} with args:`, args);
      return cacheMap.get(key);
    }
    
    console.log(`Cache miss for ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    cacheMap.set(key, result);
    return result;
  };
}

class MathService {
  @cache
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
  
  @cache
  complexCalculation(a: number, b: number): number {
    // Имитация сложных вычислений
    return Math.pow(a, b) + Math.sqrt(a * b);
  }
}

const math = new MathService();
console.log(math.fibonacci(10)); // Вычисление с кэшированием
console.log(math.fibonacci(10)); // Получение из кэша
```

### 3. Декоратор для проверки прав доступа

```typescript
function requireRole(role: string) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      // Предположим, что у нас есть метод getCurrentUser()
      const currentUser = (this as any).getCurrentUser();
      
      if (!currentUser || !currentUser.roles.includes(role)) {
        throw new Error(`Access denied: ${role} role required`);
      }
      
      return originalMethod.apply(this, args);
    };
  };
}

class UserService {
  private currentUser = { roles: ["user", "admin"] };
  
  getCurrentUser() {
    return this.currentUser;
  }
  
  @requireRole("admin")
  deleteUser(id: string): boolean {
    console.log(`User ${id} deleted`);
    return true;
  }
  
  @requireRole("user")
  updateUser(id: string, data: any): boolean {
    console.log(`User ${id} updated with:`, data);
    return true;
  }
}

const userService = new UserService();
userService.updateUser("123", { name: "John" }); // ✅ Доступ есть
userService.deleteUser("123"); // ✅ Доступ есть

// Если бы у пользователя не было роли "admin":
// userService.deleteUser("123"); // ❌ Ошибка: Access denied: admin role required
```

## Практические примеры

### 1. Декоратор для валидации параметров

```typescript
function validateParams(...validators: ((arg: any) => boolean)[]) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      for (let i = 0; i < validators.length && i < args.length; i++) {
        if (!validators[i](args[i])) {
          throw new Error(`Parameter ${i} failed validation for method ${propertyKey}`);
        }
      }
      
      return originalMethod.apply(this, args);
    };
  };
}

function isString(arg: any): boolean {
  return typeof arg === "string" && arg.length > 0;
}

function isPositiveNumber(arg: any): boolean {
  return typeof arg === "number" && arg > 0;
}

class OrderService {
  @validateParams(isString, isPositiveNumber)
  createOrder(productId: string, quantity: number): { id: string; total: number } {
    return {
      id: `order_${Date.now()}`,
      total: quantity * 10 // Цена за единицу
    };
  }
}

const orderService = new OrderService();
const order = orderService.createOrder("product123", 5); // ✅ Валидация пройдена
// orderService.createOrder("", 5); // ❌ Ошибка валидации
// orderService.createOrder("product123", -1); // ❌ Ошибка валидации
```

### 2. Декоратор для повторных попыток

```typescript
function retry(maxAttempts: number = 3, delay: number = 1000) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args: any[]) {
      let lastError: Error | null = null;
      
      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error as Error;
          console.log(`Attempt ${attempt} failed:`, error);
          
          if (attempt < maxAttempts) {
            console.log(`Retrying in ${delay}ms...`);
            await new Promise(resolve => setTimeout(resolve, delay));
          }
        }
      }
      
      throw new Error(`Method ${propertyKey} failed after ${maxAttempts} attempts: ${lastError?.message}`);
    };
  };
}

class NetworkService {
  @retry(3, 1000)
  async fetchData(url: string): Promise<any> {
    // Имитация сетевого запроса с возможными ошибками
    if (Math.random() < 0.7) { // 70% шанс ошибки
      throw new Error("Network error");
    }
    
    return { data: "Success!" };
  }
}

const networkService = new NetworkService();
networkService.fetchData("https://api.example.com/data")
  .then(result => console.log("Success:", result))
  .catch(error => console.error("Final error:", error));
```

### 3. Декоратор для отслеживания изменений

```typescript
function trackChanges(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const prevState = JSON.stringify(this);
    const result = originalMethod.apply(this, args);
    const newState = JSON.stringify(this);
    
    if (prevState !== newState) {
      console.log(`State changed in method ${propertyKey}:`, {
        before: JSON.parse(prevState),
        after: JSON.parse(newState)
      });
    }
    
    return result;
  };
}

class BankAccount {
  balance: number = 0;
  transactions: string[] = [];
  
  @trackChanges
  deposit(amount: number): void {
    this.balance += amount;
    this.transactions.push(`Deposit: +${amount}`);
  }
  
  @trackChanges
  withdraw(amount: number): void {
    if (this.balance >= amount) {
      this.balance -= amount;
      this.transactions.push(`Withdraw: -${amount}`);
    } else {
      throw new Error("Insufficient funds");
    }
  }
}

const account = new BankAccount();
account.deposit(100); 
// Выведет: "State changed in method deposit: { before: { balance: 0, transactions: [] }, after: { balance: 100, transactions: ["Deposit: +100"] } }"

account.withdraw(50);
// Выведет: "State changed in method withdraw: { before: { balance: 100, transactions: ["Deposit: +100"] }, after: { balance: 50, transactions: ["Deposit: +100", "Withdraw: -50"] } }"
```

## Композиция декораторов методов

Несколько декораторов могут применяться к одному методу:

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
  @first()
  @second()
  method() {}
}

const example = new Example();
example.method();
// Выведет:
// "first(): factory evaluated"
// "second(): factory evaluated"
// "second(): called"
// "first(): called"
```

## Ограничения и особенности

1. Декораторы методов применяются во время определения класса
2. Декораторы могут изменять дескриптор свойства метода
3. При асинхронных методах декораторы должны корректно обрабатывать Promise
4. Декораторы выполняются в порядке определения, но применяются в обратном порядке

## Связанные концепции

- [[ts/decorators/class-decorators|Декораторы классов]]
- [[ts/decorators/property-decorators|Декораторы свойств]]
- [[ts/decorators/parameter-decorators|Декораторы параметров]]
- [[ts/decorators/accessor-decorators|Декораторы аксессоров]]