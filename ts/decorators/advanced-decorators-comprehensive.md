# Декораторы в TypeScript

Декораторы - это экспериментальная возможность TypeScript, позволяющая добавлять аннотации и метапрограммирование к объявлениям классов, методов, свойств и параметров. Они позволяют изменять поведение или добавлять метаданные к существующему коду без его прямого изменения.

## Основные концепции декораторов

### 1. Включение декораторов

Для использования декораторов нужно включить соответствующую опцию в `tsconfig.json`:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true  // для метаданных рефлексии
  }
}
```

### 2. Типы декораторов

TypeScript поддерживает следующие типы декораторов:
- Декораторы классов
- Декораторы методов
- Декораторы аксессоров (геттеров/сеттеров)
- Декораторы свойств
- Декораторы параметров

## Простые примеры декораторов

### 1. Декоратор класса

```typescript
// Простой декоратор класса
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Person {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  greet() {
    return `Hello, I'm ${this.name}`;
  }
}
```

### 2. Декоратор метода

```typescript
// Декоратор метода для логирования
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with arguments:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };
  
  return descriptor;
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
calc.add(2, 3); // Логирует вызов и возвращаемое значение
```

## Продвинутые паттерны декораторов

### 1. Декораторы с параметрами

```typescript
// Декоратор с параметрами
function validateType(expectedType: string) {
  return function (target: any, propertyKey: string) {
    // Здесь можно добавить логику проверки типа
    console.log(`Validating property ${propertyKey} to be of type ${expectedType}`);
  };
}

class User {
  @validateType("string")
  name: string;
  
  @validateType("number")
  age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

### 2. Декоратор свойства

```typescript
// Декоратор для создания отслеживаемого свойства
function observable(target: any, propertyKey: string) {
  let value = target[propertyKey];
  
  const getter = function () {
    console.log(`Getting ${propertyKey}: ${value}`);
    return value;
  };
  
  const setter = function (newVal: any) {
    console.log(`Setting ${propertyKey} from ${value} to ${newVal}`);
    value = newVal;
  };
  
  // Удаление оригинального свойства
  if (delete target[propertyKey]) {
    // Создание нового свойства с геттером и сеттером
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  }
}

class Task {
  @observable
  title: string;
  
  @observable
  completed: boolean;
  
  constructor(title: string, completed: boolean = false) {
    this.title = title;
    this.completed = completed;
  }
}

const task = new Task("Learn decorators");
task.title = "Learn TypeScript decorators"; // Логирует изменение
console.log(task.title); // Логирует получение
```

### 3. Декоратор параметра

```typescript
// Декоратор параметра
function format(formatString: string) {
  return function (target: any, propertyKey: string, parameterIndex: number) {
    // Здесь можно сохранить метаданные о формате
    console.log(`Parameter ${parameterIndex} of ${propertyKey} should be formatted with: ${formatString}`);
  };
}

class Formatter {
  processString(@format("uppercase") input: string): string {
    return input.toUpperCase();
  }
  
  processNumber(@format("currency") amount: number): string {
    return `$${amount.toFixed(2)}`;
  }
}
```

## Практические применения декораторов

### 1. Валидация данных

```typescript
// Декораторы для валидации
const validators: { [key: string]: Function } = {};

function required(target: any, propertyKey: string) {
  validators[propertyKey] = (value: any) => value !== undefined && value !== null && value !== '';
}

function minLength(length: number) {
  return (target: any, propertyKey: string) => {
    validators[propertyKey] = (value: any) => 
      typeof value === 'string' && value.length >= length;
  };
}

function validate<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    constructor(...args: any[]) {
      super(...args);
      
      for (const key in validators) {
        if (validators.hasOwnProperty(key)) {
          const validator = validators[key];
          if (!validator(this[key])) {
            throw new Error(`Validation failed for property: ${key}`);
          }
        }
      }
    }
  };
}

@validate
class UserRegistration {
  @required
  email: string;
  
  @required
  @minLength(3)
  username: string;
  
  @minLength(8)
  password: string;
  
  constructor(email: string, username: string, password: string) {
    this.email = email;
    this.username = username;
    this.password = password;
  }
}

// Использование
try {
  const user = new UserRegistration("test@example.com", "user", "12345678");
  console.log("User created successfully");
} catch (error) {
  console.error(error.message);
}
```

### 2. Управление доступом (авторизация)

```typescript
// Декораторы для авторизации
enum Role {
  Admin = "admin",
  User = "user",
  Guest = "guest"
}

function authorize(roles: Role[]) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function (...args: any[]) {
      // Предполагаем, что у объекта есть свойство userRole
      if (!this.userRole || !roles.includes(this.userRole)) {
        throw new Error(`Access denied. Required roles: ${roles.join(', ')}`);
      }
      return originalMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

class UserService {
  userRole: Role;
  
  constructor(role: Role) {
    this.userRole = role;
  }
  
  @authorize([Role.Admin, Role.User])
  getUserData(userId: number) {
    return `User data for ID: ${userId}`;
  }
  
  @authorize([Role.Admin])
  deleteUser(userId: number) {
    return `User ${userId} deleted`;
  }
}

const adminService = new UserService(Role.Admin);
const userService = new UserService(Role.User);

console.log(adminService.getUserData(1)); // OK
console.log(adminService.deleteUser(1));  // OK

console.log(userService.getUserData(1)); // OK
// console.log(userService.deleteUser(1)); // Ошибка: доступ запрещен
```

### 3. Кэширование результатов

```typescript
// Декоратор для кэширования результатов метода
function cache(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  const cacheMap = new Map<string, any>();
  
  descriptor.value = function (...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cacheMap.has(key)) {
      console.log(`Cache hit for ${propertyKey}`);
      return cacheMap.get(key);
    }
    
    console.log(`Cache miss for ${propertyKey}, computing...`);
    const result = originalMethod.apply(this, args);
    cacheMap.set(key, result);
    
    return result;
  };
  
  return descriptor;
}

class ExpensiveCalculator {
  @cache
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
  
  @cache
  slowOperation(data: string): string {
    // Симуляция медленной операции
    let result = '';
    for (let i = 0; i < 1000000; i++) {
      result += data;
    }
    return `Processed: ${data}`;
  }
}
```

## Рефлексия и метаданные

### 1. Использование reflect-metadata

Для работы с метаданными нужно установить пакет:

```bash
npm install reflect-metadata
```

```typescript
import 'reflect-metadata';

// Сохранение и получение метаданных
function logType(target: any, propertyKey: string) {
  const type = Reflect.getMetadata('design:type', target, propertyKey);
  console.log(`${propertyKey} type: ${type.name}`);
}

function logParamTypes(target: any, propertyKey: string) {
  const types = Reflect.getMetadata('design:paramtypes', target, propertyKey);
  console.log(`${propertyKey} param types: ${types.map(t => t.name).join(', ')}`);
}

function logReturnType(target: any, propertyKey: string) {
  const type = Reflect.getMetadata('design:returntype', target, propertyKey);
  console.log(`${propertyKey} return type: ${type?.name || 'void'}`);
}

class TestClass {
  @logType
  greeting: string;
  
  constructor(@logParamTypes message: string) {
    this.greeting = message;
  }
  
  @logParamTypes
  @logReturnType
  greet(@logParamTypes name: string): string {
    return `${this.greeting}, ${name}!`;
  }
}
```

### 2. Создание собственных метаданных

```typescript
import 'reflect-metadata';

// Кастомный декоратор для добавления метаданных
function route(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    Reflect.defineMetadata('route:path', path, descriptor.value);
    Reflect.defineMetadata('route:method', 'GET', descriptor.value);
  };
}

function controller(prefix: string) {
  return function (constructor: Function) {
    Reflect.defineMetadata('controller:prefix', prefix, constructor);
  };
}

@controller('/api/users')
class UserController {
  @route('/:id')
  getUser() {
    return 'Get user by ID';
  }
  
  @route('/')
  getAllUsers() {
    return 'Get all users';
  }
}

// Использование метаданных
function getRouteInfo(method: Function) {
  const path = Reflect.getMetadata('route:path', method);
  const httpMethod = Reflect.getMetadata('route:method', method);
  return { path, httpMethod };
}

const getUserRoute = getRouteInfo(UserController.prototype.getUser);
console.log(getUserRoute); // { path: '/:id', httpMethod: 'GET' }
```

## Продвинутые паттерны

### 1. Композиция декораторов

```typescript
// Несколько декораторов на одном элементе
function first() {
  console.log("first(): factory evaluation");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluation");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class ExampleClass {
  @first()
  @second()
  method() {
    console.log("method(): called");
  }
}

// Порядок выполнения:
// 1. first(): factory evaluation
// 2. second(): factory evaluation
// 3. second(): called
// 4. first(): called
// 5. method(): called
```

### 2. Декораторы с условной логикой

```typescript
// Декоратор с условной логикой
function conditionalLog(condition: () => boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function (...args: any[]) {
      if (condition()) {
        console.log(`Logging: ${propertyKey} called with`, args);
      }
      return originalMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

// Использование с разными условиями
class DataProcessor {
  @conditionalLog(() => process.env.NODE_ENV === 'development')
  processData(data: any) {
    return data;
  }
  
  @conditionalLog(() => true) // Всегда логирует
  criticalOperation(data: any) {
    return data;
  }
}
```

## Практические примеры из реальной разработки

### 1. Декораторы для DI-контейнера

```typescript
import 'reflect-metadata';

const INJECT_TOKEN = Symbol('inject');
const SINGLETON_TOKEN = Symbol('singleton');

// Декоратор для внедрения зависимости
function Inject(token: string) {
  return function (target: any, propertyKey: string | undefined, parameterIndex: number) {
    const existingTokens = Reflect.getMetadata(INJECT_TOKEN, target) || [];
    existingTokens[parameterIndex] = token;
    Reflect.defineMetadata(INJECT_TOKEN, existingTokens, target);
  };
}

// Декоратор для синглтона
function Singleton() {
  return function (constructor: Function) {
    Reflect.defineMetadata(SINGLETON_TOKEN, true, constructor);
  };
}

// Простой DI-контейнер
class Container {
  private instances = new Map();
  private registrations = new Map();
  
  register(token: string, useClass: any) {
    this.registrations.set(token, useClass);
  }
  
  resolve<T>(token: string): T {
    if (this.instances.has(token)) {
      return this.instances.get(token);
    }
    
    const useClass = this.registrations.get(token);
    if (!useClass) {
      throw new Error(`No registration for token: ${token}`);
    }
    
    // Получение типов параметров конструктора
    const paramTypes = Reflect.getMetadata('design:paramtypes', useClass) || [];
    const injectTokens = Reflect.getMetadata(INJECT_TOKEN, useClass) || [];
    
    const dependencies = paramTypes.map((paramType: any, index: number) => {
      const token = injectTokens[index] || paramType.name;
      return this.resolve(token);
    });
    
    const instance = new useClass(...dependencies);
    
    // Если класс отмечен как Singleton, сохраняем инстанс
    if (Reflect.getMetadata(SINGLETON_TOKEN, useClass)) {
      this.instances.set(token, instance);
    }
    
    return instance;
  }
}
```

### 2. Декораторы для веб-фреймворка

```typescript
import 'reflect-metadata';

// HTTP методы
function Get(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    Reflect.defineMetadata('route:path', path, descriptor.value);
    Reflect.defineMetadata('route:method', 'GET', descriptor.value);
  };
}

function Post(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    Reflect.defineMetadata('route:path', path, descriptor.value);
    Reflect.defineMetadata('route:method', 'POST', descriptor.value);
  };
}

function Body(target: any, propertyKey: string, parameterIndex: number) {
  Reflect.defineMetadata('param:body', parameterIndex, target[propertyKey]);
}

function Query(paramName: string) {
  return function (target: any, propertyKey: string, parameterIndex: number) {
    Reflect.defineMetadata('param:query', { index: parameterIndex, name: paramName }, target[propertyKey]);
  };
}

@controller('/api/users')
class UserController {
  @Get('/')
  getAllUsers() {
    return { users: [] };
  }
  
  @Get('/:id')
  getUser(@Query('includeProfile') includeProfile: boolean) {
    return { id: 1, name: 'John' };
  }
  
  @Post('/')
  createUser(@Body body: { name: string; email: string }) {
    return { id: Date.now(), ...body };
  }
}
```

## Ограничения и особенности

### 1. Совместимость с наследованием

```typescript
// Декораторы работают с наследованием
function logClass(target: Function) {
  console.log(`Class ${target.name} is being defined`);
}

@logClass
class BaseClass {
  method() {
    console.log('Base method');
  }
}

class DerivedClass extends BaseClass {
  // Декоратор базового класса также применяется
  derivedMethod() {
    console.log('Derived method');
  }
}
```

### 2. Ограничения при использовании

- Декораторы - экспериментальная функция, которая может измениться
- Требуют включения специальных флагов компилятора
- Могут усложнить отладку кода
- Не все транспайлеры поддерживают декораторы одинаково

## Лучшие практики

1. **Используйте декораторы осознанно** - не применяйте их ради применения
2. **Документируйте кастомные декораторы** - особенно их поведение и параметры
3. **Тестируйте декораторы отдельно** - как отдельные модули
4. **Избегайте сложной логики в декораторах** - они должны быть простыми и предсказуемыми
5. **Рассмотрите альтернативы** - иногда обычные функции могут быть лучше
6. **Следите за стандартом** - декораторы все еще в стадии стандартизации

## Связи с другими концепциями

- [[ts/classes|Классы]] - декораторы применяются к элементам классов
- [[ts/generics|Обобщения]] - могут использоваться в декораторах
- [[ts/type-system/type-guards|Защитники типов]] - для работы с типами в декораторах
- [[js/object-oriented-programming]] - основы ООП, которые дополняют декораторы
- [[architecture/component-architecture]] - архитектурные паттерны с использованием декораторов