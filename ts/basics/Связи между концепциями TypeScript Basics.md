# Связи между концепциями TypeScript Basics

## Архитектурная схема Basics

```
                    ┌─────────────────────────────────────────────────────────┐
                    │               TypeScript Basics                         │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────┐
        ▼                                     ▼                                                   ▼
  ┌─────────────────┐               ┌──────────────────────────┐                            ┌─────────────────┐
  │ Type Annotations│               │      Variables &         │                            │  Control Flow   │
  │                 │               │      Constants           │                            │                 │
  │ - Primitive     │◄──────────────┤ - var, let, const        │───────────────────────────►│ - Conditional   │
  │   Types         │               │ - Type Inference         │                            │   Statements    │
  │ - Object Types  │               │ - Scope Rules            │                            │ - Loops         │
  │ - Function      │               │ - const vs readonly      │                            │ - Type Guards   │
  │   Types         │               │ - any vs unknown         │                            │ - Assertions    │
  └─────────────────┘               │ - Literal Types          │                            └─────────────────┘
        │                           └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │            Functions              │
        │                           │                                 │
        │                           │ - Parameters & Return Types     │
        │                           │ - Arrow Functions               │
        │                           │ - Function Types                │
        │                           │ - Generics in Functions         │
        │                           │ - Callbacks                     │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │           Arrays                │
        │                           │                                 │
        │                           │ - Array Types                   │
        │                           │ - Methods (map, filter, etc.)   │
        │                           │ - Multidimensional Arrays       │
        │                           │ - Generics with Arrays          │
        └───────────────────────────┤ - Type Narrowing in Arrays      │
                                    └─────────────────────────────────┘
```

## Подробные связи между концепциями

### 1. Type Annotations ↔ Variables
```typescript
// Аннотации типов определяют тип переменных
let name: string = "Alice";        // string annotation
let age: number = 30;              // number annotation
const isActive: boolean = true;    // boolean annotation

// Вывод типов часто работает без аннотаций
let inferredName = "Bob";        // string (выведен)
let inferredAge = 25;            // number (выведен)

// но аннотации необходимы для сложных типов
let user: { name: string; age: number } = { name: "Carol", age: 35 };

// и для ограничений
type Status = "active" | "inactive";
let status: Status = "active";   // литеральный тип
```

### 2. Variables ↔ Functions
```typescript
// Переменные могут содержать функции
let greet: (name: string) => string;
greet = function(name: string): string {
  return `Hello, ${name}!`;
};

// Функции как параметры
function executeCallback(callback: () => void) {
  callback();
}

// Переменные с функциональными типами
type MathOperation = (a: number, b: number) => number;
let add: MathOperation = (a, b) => a + b;
let multiply: MathOperation = (a, b) => a * b;
```

### 3. Control Flow ↔ Type Annotations
```typescript
// Условия сужают типы на основе аннотаций
function processValue(value: string | number) {
  if (typeof value === "string") {
    // value имеет тип string в этой ветке
    return value.toUpperCase();  // безопасный вызов строкового метода
  } else {
    // value имеет тип number в этой ветке
    return value.toFixed(2);     // безопасный вызов числового метода
  }
}

// Проверки с литеральными типами
type Direction = "up" | "down" | "left" | "right";
function move(direction: Direction) {
  switch (direction) {
    case "up":
      // direction имеет тип "up" в этой ветке
      return { y: -1, x: 0 };
    case "down":
      // direction имеет тип "down" в этой ветке
      return { y: 1, x: 0 };
    default:
      // direction имеет тип "left" | "right" в этой ветке
      const remaining: "left" | "right" = direction;
      return remaining === "left" ? { x: -1, y: 0 } : { x: 1, y: 0 };
  }
}
```

### 4. Arrays ↔ Functions
```typescript
// Функции работают с массивами и определяют их типы
function processStringArray(items: string[]): string[] {
  return items.map(item => item.toUpperCase());
}

// Массивы функций
type Validator = (value: string) => boolean;
let validators: Validator[] = [
  (s) => s.length > 0,           // не пустая строка
  (s) => s.length <= 255,        // не длиннее 255 символов
  (s) => /^[a-zA-Z0-9]+$/.test(s) // только буквы и цифры
];

// Функции, возвращающие массивы
function createRange(start: number, end: number): number[] {
  return Array.from({ length: end - start + 1 }, (_, i) => start + i);
}
```

### 5. Control Flow ↔ Arrays
```typescript
// Поток управления в методах массивов
let mixed: (string | number)[] = ["hello", 42, "world", 123];

// filter + type guard
let strings = mixed.filter((item): item is string => typeof item === "string");
// strings теперь имеет тип string[]

// map с сужением типов
let processed = mixed.map(item => {
  if (typeof item === "string") {
    // item имеет тип string
    return item.toUpperCase();
  } else {
    // item имеет тип number
    return item.toFixed(2);
  }
});
// processed имеет тип (string | number)[]

// find с конкретным результатом
let firstString = mixed.find((item): item is string => typeof item === "string");
// firstString имеет тип string | undefined
```

## Интеграционные паттерны

### 1. Паттерн "Конфигурация + Валидация"
```typescript
// Связь: Type Annotations + Control Flow + Arrays
interface ConfigItem {
  key: string;
  value: string | number | boolean;
  required: boolean;
}

let config: ConfigItem[] = [
  { key: "apiUrl", value: "https://api.example.com", required: true },
  { key: "timeout", value: 5000, required: false },
  { key: "debug", value: true, required: false }
];

function validateConfig(items: ConfigItem[]): boolean {
  return items
    .filter(item => item.required)
    .every(item => {
      if (typeof item.value === "string") {
        return item.value.length > 0;
      } else if (typeof item.value === "number") {
        return item.value > 0;
      } else {
        return typeof item.value === "boolean";
      }
    });
}

const isValid = validateConfig(config);
```

### 2. Паттерн "Цепочка обработки данных"
```typescript
// Связь: Variables + Functions + Arrays + Control Flow
type DataProcessor<T, U> = (data: T) => U;

let processors: DataProcessor<any, any>[] = [
  (data: string) => data.trim(),           // удалить пробелы
  (data: string) => data.toLowerCase(),    // в нижний регистр
  (data: string) => data.replace(/\s+/g, ' ')  // заменить множ. пробелы
];

function processString(input: string, processors: DataProcessor<string, string>[]): string {
  return processors.reduce((acc, processor) => {
    return processor(acc);
  }, input);
}

const result = processString("  HELLO    WORLD  ", processors);
// "hello world"
```

### 3. Паттерн "Типобезопасное API"
```typescript
// Связь: все концепции basics
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type HttpResponse<T> = { success: true; data: T } | { success: false; error: string };

interface ApiRequest {
  method: HttpMethod;
  endpoint: string;
  headers?: { [key: string]: string };
}

class TypedApi {
  private requests: ApiRequest[] = [];
  
  addRequest<T>(request: Omit<ApiRequest, 'method'> & { method: "GET" }): Promise<HttpResponse<T>>;
  addRequest<T>(request: Omit<ApiRequest, 'method'> & { method: "POST" }): Promise<HttpResponse<T>>;
  addRequest<T>(request: Omit<ApiRequest, 'method'> & { method: HttpMethod }): Promise<HttpResponse<T>> {
    const fullRequest: ApiRequest = {
      headers: {},
      ...request
    };
    
    this.requests.push(fullRequest);
    
    // симуляция запроса
    return Promise.resolve({ success: true, data: {} as T });
  }
  
  getRequests(): ApiRequest[] {
    return [...this.requests]; // безопасная копия
  }
}
```

## Практическое применение связей

### 1. Система событий
```typescript
// Интеграция всех концепций basics
type EventName = `user:${"login" | "logout" | "update"}` | `system:${"start" | "stop"}`;

interface Event<T = any> {
  name: EventName;
  data: T;
  timestamp: number;
}

type EventHandler<T = any> = (event: Event<T>) => void;

class EventBus {
  private handlers: { [K in EventName]?: EventHandler[] } = {};
  private eventQueue: Event[] = [];
  
  on<T>(name: EventName, handler: EventHandler<T>): void {
    if (!this.handlers[name]) {
      this.handlers[name] = [];
    }
    this.handlers[name]!.push(handler as EventHandler);
  }
  
  emit<T>(name: EventName, data: T): void {
    this.eventQueue.push({
      name,
      data,
      timestamp: Date.now()
    });
    
    const handlers = this.handlers[name];
    if (handlers) {
      for (const handler of handlers) {
        handler({ name, data, timestamp: Date.now() });
      }
    }
  }
  
  getEventHistory(): Event[] {
    return [...this.eventQueue];
  }
}

// Использование
const bus = new EventBus();
bus.on("user:login", (event) => {
  console.log(`User logged in at ${event.timestamp}`);
});
```

## Лучшие практики интеграции

### 1. Порядок объявления
1. **Сначала типы** - определите структуры данных
2. **Потом переменные** - создайте экземпляры с типами
3. **Затем функции** - обработчики и утилиты
4. **Потом поток управления** - условия и циклы
5. **Финальная композиция** - объединение всех частей

### 2. Оптимизация типов
```typescript
// Избегайте дублирования в цепочках
const processedData = rawData
  .filter((item): item is ValidItem => isValid(item))  // 1. сужение типов
  .map(transformItem)                                  // 2. преобразование
  .filter(item => item.active)                         // 3. дополнительная фильтрация
  .map(item => ({ ...item, processed: true }));        // 4. финальное преобразование
```

### 3. Ошибки и безопасность
```typescript
// Безопасная обработка с проверками
function safeProcess<T>(items: T[], processor: (item: T) => T): T[] {
  if (!Array.isArray(items)) {
    throw new Error("Items must be an array");
  }
  
  return items.map(item => {
    try {
      return processor(item);
    } catch (error) {
      console.warn("Processing error:", error);
      return item; // возврат оригинального значения при ошибке
    }
  });
}
```

## Связь с остальными концепциями TypeScript

- **[[../type-system/type-inference]]** - как система выводит типы из связанных концепций
- **[[../type-system/type-compatibility]]** - совместимость между типами из разных концепций
- **[[TS Generics]]** - обобщения для универсальных связей
- **[[TS Interfaces and Classes]]** - объектно-ориентированные паттерны
- **[[TypeScript Advanced Types]]** - продвинутые паттерны интеграции