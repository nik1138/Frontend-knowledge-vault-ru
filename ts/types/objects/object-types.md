# Object Types

Объектные типы в TypeScript описывают структуру объектов, функций и массивов. Они определяют набор свойств и их типов, которые должен иметь объект.

## Объектные литералы

### Простые объектные типы
```typescript
// Анонимный объектный тип
let user: { name: string; age: number } = { name: "Alice", age: 30 };

// Типизация вложенных объектов
let employee: {
  id: number;
  personal: {
    name: string;
    age: number;
  };
  work: {
    position: string;
    department: string;
  };
} = {
  id: 1,
  personal: { name: "Alice", age: 30 },
  work: { position: "Developer", department: "IT" }
};

// Необязательные свойства
let config: { host: string; port?: number; ssl?: boolean } = { 
  host: "localhost" 
  // port и ssl не обязательны
};
```

### Сокращенная запись объектов
```typescript
// Сокращенная запись при создании объектов
function createUser(name: string, age: number) {
  // При создании объекта не обязательно указывать тип явно
  return { name, age }; // TypeScript выводит тип { name: string; age: number }
}

// Деструктуризация с типизацией
function processUser({ name, age }: { name: string; age: number }) {
  console.log(`Processing ${name}, age ${age}`);
}
```

## Интерфейсы для объектов

### Определение форм через интерфейсы
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

let user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  isActive: true
};

// Интерфейсы могут быть расширены
interface PremiumUser extends User {
  level: "gold" | "silver" | "bronze";
  benefits: string[];
}

let premiumUser: PremiumUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com", 
  isActive: true,
  level: "gold",
  benefits: ["priority", "discount"]
};
```

### Индексные сигнатуры
```typescript
// Строковые индексные сигнатуры
interface StringDictionary {
  [key: string]: string;
}

let dictionary: StringDictionary = {
  hello: "world",
  foo: "bar"
};

// Смешанные сигнатуры
interface MixedDictionary {
  [key: string]: string | number; // Разные типы для значений
  length: number; // Конкретное свойство
  name: string;   // Конкретное свойство
}

// Числовые индексные сигнатуры (для массивов)
interface NumberDictionary {
  [index: number]: string;
  length: number; // OK: число совместимо с индексной сигнатурой
  // name: string; // Ошибка: строка несовместима с числовыми индексами
}
```

## Типы функций как объекты

### Типизация функций
```typescript
// Тип функции
let greet: (name: string) => string;
greet = function(name: string): string {
  return `Hello, ${name}`;
};

// Или с интерфейсом
interface GreetingFunction {
  (name: string): string;
}

let myGreet: GreetingFunction = (name) => `Hi, ${name}`;

// Функции как свойства объектов
interface UserService {
  getUser(id: number): User;
  createUser(userData: { name: string; email: string }): User;
  updateUser(id: number, updates: Partial<User>): User;
}

// Функция с this контекстом
interface ObjectWithValue {
  value: string;
  getFormattedValue(): string;
}

function getFormattedValue(this: ObjectWithValue): string {
  return `Value: ${this.value}`;
}

let obj: ObjectWithValue = {
  value: "test",
  getFormattedValue
};
```

## Массивы как объекты

### Типизация массивов
```typescript
// Типизация массивов
let numbers: number[] = [1, 2, 3, 4];
let names: Array<string> = ["Alice", "Bob", "Charlie"];

// Массивы как объекты с числовыми индексами
interface NumberArray {
  [index: number]: number;
  length: number;
  push(item: number): number;
  pop(): number | undefined;
}

// Многомерные массивы
let matrix: number[][] = [[1, 2], [3, 4]];
let cube: number[][][] = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]];

// readonly массивы
let readonlyNumbers: readonly number[] = [1, 2, 3];
// readonlyNumbers.push(4); // Ошибка: push не доступен для readonly
```

## Продвинутые объектные типы

### Смешанные типы
```typescript
// Объекты, которые могут быть вызваны как функции
interface CallableObject {
  (input: string): number;
  property: string;
  method(): void;
}

function createCallable(): CallableObject {
  const obj = ((input: string) => input.length) as CallableObject;
  obj.property = "test";
  obj.method = () => console.log("method called");
  return obj;
}

const callable = createCallable();
callable("hello"); // 5
callable.property; // "test"
callable.method(); // "method called"
```

### Геттеры и сеттеры
```typescript
interface PersonWithValidation {
  name: string;
  age: number;
  readonly birthYear: number;
}

class Person implements PersonWithValidation {
  private _name: string;
  private _age: number;
  
  constructor(name: string, age: number) {
    this._name = name;
    this._age = age;
  }
  
  get name(): string {
    return this._name;
  }
  
  set name(value: string) {
    if (value.length < 2) {
      throw new Error("Name too short");
    }
    this._name = value;
  }
  
  get age(): number {
    return this._age;
  }
  
  set age(value: number) {
    if (value < 0 || value > 150) {
      throw new Error("Invalid age");
    }
    this._age = value;
  }
  
  get birthYear(): number {
    return new Date().getFullYear() - this._age;
  }
}
```

## Опциональные и readonly свойства

### Необязательные свойства
```typescript
interface SquareConfig {
  color?: string;
  width?: number;
  // свойства с ? являются опциональными
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = { color: "white", area: 100 };
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

// Можно передать только часть свойств
let mySquare = createSquare({ color: "black" });
```

### Только для чтения свойства
```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

let p1: Point = { x: 10, y: 20 };
// p1.x = 5; // Ошибка: не может быть изменено
// p1.y = 15; // Ошибка: не может быть изменено

// readonly массивы
let arr: ReadonlyArray<number> = [1, 2, 3, 4];
// arr[0] = 10; // Ошибка
// arr.push(5); // Ошибка
// arr.length = 0; // Ошибка

let writable = arr as number[]; // Приведение типа для записи
writable[0] = 10; // OK
```

### Readonly с сопоставляющими типами
```typescript
// Создание полностью readonly версии типа
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type ReadonlyTodo = {
  readonly [K in keyof Todo]: Todo[K]
};

let readonlyTodo: ReadonlyTodo = {
  title: "Learn TypeScript",
  description: "Complete the object types chapter",
  completed: false
};

// readonlyTodo.completed = true; // Ошибка
```

## Типизация сложных объектов

### Вложенные объекты и массивы
```typescript
interface Company {
  name: string;
  address: {
    street: string;
    city: string;
    country: string;
    coordinates?: {
      lat: number;
      lng: number;
    };
  };
  employees: Array<{
    id: number;
    name: string;
    position: string;
    contact: {
      email: string;
      phone?: string;
    };
  }>;
  departments: {
    [name: string]: {
      manager: string;
      count: number;
      budget: number;
    };
  };
}

const techCompany: Company = {
  name: "Tech Corp",
  address: {
    street: "123 Tech St",
    city: "Silicon Valley",
    country: "USA",
    coordinates: {
      lat: 37.3861,
      lng: -122.0839
    }
  },
  employees: [
    {
      id: 1,
      name: "Alice",
      position: "Developer",
      contact: {
        email: "alice@tech.com",
        phone: "+1-555-0123"
      }
    }
  ],
  departments: {
    engineering: {
      manager: "Bob",
      count: 15,
      budget: 1000000
    }
  }
};
```

### Типизация конфигурационных объектов
```typescript
interface ApiConfig {
  baseUrl: string;
  timeout: number;
  headers?: {
    [key: string]: string;
  };
  retries?: number;
  auth?: {
    token: string;
    type: "bearer" | "basic";
  };
}

function setupApi(config: ApiConfig): void {
  console.log(`API configured for ${config.baseUrl}`);
}

setupApi({
  baseUrl: "https://api.example.com",
  timeout: 5000,
  headers: {
    "Content-Type": "application/json",
    "Accept": "application/json"
  },
  retries: 3
});
```

## Практические паттерны

### Типизация DTO (Data Transfer Objects)
```typescript
// DTO для входящих данных
interface UserRegistrationDTO {
  readonly timestamp: string;
  readonly userId: string; 
  readonly requestId: string;
  userData: {
    email: string;
    password: string;
    profile: {
      firstName: string;
      lastName: string;
      dateOfBirth?: string;
    };
  };
}

// DTO для исходящих данных
interface UserResponseDTO {
  id: number;
  email: string;
  profile: {
    firstName: string;
    lastName: string;
    fullName: string;
    age?: number;
  };
  createdAt: string;
  lastLoginAt?: string;
}
```

### Типизация событий
```typescript
interface EventMap {
  userLogin: {
    userId: number;
    timestamp: Date;
    ipAddress: string;
  };
  userLogout: {
    userId: number;
    duration: number;
    reason?: string;
  };
  dataUpdate: {
    updatedFields: string[];
    oldValue: any;
    newValue: any;
  };
}

// Универсальный обработчик событий
function handleEvent<T extends keyof EventMap>(
  eventType: T,
  data: EventMap[T]
): void {
  console.log(`Event ${eventType} with data:`, data);
}

// Использование
handleEvent('userLogin', {
  userId: 123,
  timestamp: new Date(),
  ipAddress: "192.168.1.1"
});
```

## Лучшие практики

1. **Используйте интерфейсы для публичных API** - они лучше документируют контракты
2. **Предпочитайте конкретные типы общим** - `status: "active" | "inactive"` лучше `status: string`
3. **Используйте readonly для неизменяемых данных** - повышает безопасность
4. **Документируйте сложные объектные структуры** - особенно вложенные
5. **Избегайте слишком глубокой вложенности** - разбивайте на более мелкие типы

## Связь с другими концепциями
- [[../interfaces-classes/interfaces]] - подробнее о типизации интерфейсами
- [[literal-types]] - литеральные типы в объектах
- [[intersection-types]] - комбинирование объектных типов
- [[../advanced-types/mapped-types]] - преобразование объектных типов