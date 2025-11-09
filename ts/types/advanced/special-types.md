# Special Types

Специальные типы в TypeScript - это уникальные типы системы, которые не вписываются в стандартную категоризацию примитивов или объектов. Они выполняют специфические функции в системе типов и имеют особое поведение.

## Any Type

### Использование any
```typescript
// any отключает проверку типов
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // OK, definitely a boolean

// any в массивах
let list: any[] = [1, true, "free"];
list[1] = 100;

// any для неизвестных значений
function processValue(value: any) {
  // TypeScript не проверяет типы внутри функции
  return value.something.method();
}

// any часто возникает при импорте из JavaScript
declare function externalLib(): any;
```

### Когда использовать any и альтернативы
```typescript
// ПЛОХО: чрезмерное использование any
function badExample(data: any): any {
  return data.process();
}

// ЛУЧШЕ: использование unknown и утверждений типов
function betterExample(data: unknown): string {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
  return "";
}

// ИЛИ: обобщения
function genericExample<T>(data: T): T {
  return data;
}
```

## Unknown Type

### Использование unknown
```typescript
// unknown - безопасная альтернатива any
let value: unknown;

value = "hello";
value = 42;
value = true;
value = { name: "Alice" };

// НЕЛЬЗЯ использовать unknown напрямую
// console.log(value.toUpperCase()); // Ошибка!
// value(); // Ошибка!
// value.prop; // Ошибка!

// Для использования нужно уточнить тип
if (typeof value === "string") {
  console.log(value.toUpperCase()); // OK
}

// Или с приведением типов
const str = value as string;
console.log(str.toUpperCase()); // OK, но небезопасно
```

### Примеры безопасного использования unknown
```typescript
// Функция для безопасной обработки данных
function safeJsonParse(text: string): unknown {
  try {
    return JSON.parse(text);
  } catch {
    return undefined;
  }
}

function processJson(json: string) {
  const data = safeJsonParse(json);
  
  if (data && typeof data === "object" && data !== null) {
    if ("name" in data && typeof data.name === "string") {
      console.log(data.name);
    }
  }
}

// Тип-предикат для уточнения unknown
function isValidUser(obj: unknown): obj is { name: string; age: number } {
  return (
    typeof obj === "object" &&
    obj !== null &&
    "name" in obj &&
    typeof obj.name === "string" &&
    "age" in obj &&
    typeof obj.age === "number"
  );
}

const userData: unknown = { name: "Alice", age: 30 };
if (isValidUser(userData)) {
  // userData теперь имеет тип { name: string; age: number }
  console.log(userData.name);
}
```

## Never Type

### Использование never
```typescript
// never для функций, которые никогда не возвращаются
function error(message: string): never {
  throw new Error(message);
}

// never для функций, которые работают вечно
function infiniteLoop(): never {
  while (true) {}
}

// never в неполных switch
function assertNever(x: never): never {
  throw new Error("Unexpected object: " + x);
}

// Полезно для проверки исчерпания в union типах
type Color = "red" | "green" | "blue";

function getColorName(color: Color) {
  switch (color) {
    case "red": return "Red";
    case "green": return "Green";
    case "blue": return "Blue";
    default:
      // Если добавить новый цвет в Color без обновления switch,
      // будет ошибка: string не совместим с never
      return assertNever(color);
  }
}
```

### never в сужениях типов
```typescript
// never как результат исключения
function processValue(value: string | number | boolean) {
  if (typeof value === "string") {
    console.log("String:", value);
  } else if (typeof value === "number") {
    console.log("Number:", value);
  } else if (typeof value === "boolean") {
    console.log("Boolean:", value);
  } else {
    // В этой точке TypeScript знает, что value имеет тип never
    // потому что все возможные типы уже обработаны
    const exhaustiveCheck: never = value;
    throw new Error(`Unexpected value: ${exhaustiveCheck}`);
  }
}
```

## Void Type

### Использование void
```typescript
// void для функций без возвращаемого значения
function warnUser(): void {
  console.log("This is my warning message");
}

// void для переменных (редко используется)
let unusable: void;
unusable = undefined; // OK
// unusable = null; // OK только если не strictNullChecks

// void в коллбэках
function executeCallback(callback: () => void) {
  callback();
  return "done";
}

executeCallback(() => console.log("Callback executed"));
```

### Разница между void и undefined
```typescript
// void - отсутствие возвращаемого значения
function returnsVoid(): void {
  console.log("Hello");
  // Неявно возвращает undefined
}

// undefined - конкретное значение
function returnsUndefined(): undefined {
  console.log("Hello");
  return undefined;
}

// Объявление переменных
let voidVar: void;
let undefVar: undefined;

voidVar = undefined; // OK
// voidVar = null; // OK если не strictNullChecks, иначе ошибка
// undefVar = null; // Ошибка: null не может быть присвоено undefined
```

## Null и Undefined Types

### Специфика null и undefined
```typescript
// Типы null и undefined
let u: undefined = undefined;
let n: null = null;

// По умолчанию в не strict режиме являются подтипами всех других типов
let num: number = undefined; // OK в не strict режиме
let str: string = null;      // OK в не strict режиме

// В strict режиме можно использовать объединения
let nullableString: string | null = null;
let optionalNumber: number | undefined = undefined;

// Проверка на null/undefined
function doSomething(callback?: () => void) {
  if (callback) { // проверяет на null и undefined
    callback();
  }
}
```

### Утверждения с null/undefined
```typescript
// Необязательные свойства
interface User {
  name: string;
  email?: string; // тоже что и email: string | undefined
}

// Проверка необязательных свойств
function processUser(user: User) {
  if (user.email) { // проверяет на null и undefined
    console.log(user.email.toUpperCase());
  }
  
  // ИЛИ явная проверка
  if (typeof user.email !== "undefined") {
    console.log(user.email);
  }
}

// Операторы опциональной цепочки
interface Response {
  data?: {
    user?: {
      name?: string;
    };
  };
}

function getUserName(response: Response): string {
  return response.data?.user?.name || "Anonymous";
}
```

## Global Types

### globalThis и Window
```typescript
// TypeScript различает глобальные объекты в разных средах
if (typeof window !== "undefined") {
  // Клиентский код
  console.log("Browser environment");
} else if (typeof global !== "undefined") {
  // Node.js код
  console.log("Node.js environment");
} else if (typeof self !== "undefined") {
  // Web worker
  console.log("Web worker environment");
}

// Использование globalThis для универсального доступа
function getGlobalObject() {
  if (typeof globalThis !== "undefined") return globalThis;
  if (typeof window !== "undefined") return window;
  if (typeof global !== "undefined") return global;
  if (typeof self !== "undefined") return self;
  
  throw new Error("Unable to locate global object");
}
```

## Сравнение специальных типов

### Иерархия специальных типов
```typescript
// В системе типов TypeScript:
// never - нижний тип (подтип всех типов)
//     ↓
// Типы значений (string, number, boolean, etc.)
//     ↓  
// unknown - верхний тип (супертип всех типов)

// any - исключение: совместим с любыми типами в обе стороны
// и отключает проверку типов

// Демонстрация иерархии
function typeHierarchyExample() {
  let neverVal: never = (function(): never { throw new Error(); })();
  let stringVal: string = "hello";
  let unknownVal: unknown = "world";
  
  // never может быть присвоено любому типу
  let str: string = neverVal; // OK
  let num: number = neverVal; // OK
  
  // любое значение может быть присвоено unknown
  unknownVal = stringVal;     // OK
  unknownVal = neverVal;      // OK
  
  // но unknown не может быть присвоено другим типам без проверки
  // stringVal = unknownVal;  // Ошибка!
}
```

## Практические применения

### Защита типов в API
```typescript
// Использование never для безопасности типов
interface Success {
  type: "success";
  data: string;
}

interface Error {
  type: "error";
  message: string;
}

type Result = Success | Error;

function handleResult(result: Result): string {
  switch (result.type) {
    case "success":
      return `Success: ${result.data}`;
    case "error":
      return `Error: ${result.message}`;
    default:
      // Если добавить новый тип в Result без обновления switch,
      // будет ошибка, потому что default получит never
      const exhaustive: never = result;
      throw new Error(`Unhandled case: ${exhaustive}`);
  }
}
```

### Обработка динамических данных
```typescript
// Безопасная обработка неизвестных данных
function safeDataProcessor(unknownData: unknown) {
  if (typeof unknownData === "object" && unknownData !== null) {
    if ("name" in unknownData && typeof unknownData.name === "string") {
      return { processed: true, name: unknownData.name };
    }
  }
  return { processed: false };
}

// Использование any для совместимости с JS
function integrateWithJsLibrary(data: any) {
  // Внутри функции можно отключить проверку типов
  // Но нужно возвращать типизированные значения
  const result = (data && data.process && data.process()) || null;
  return result as string | null;
}
```

## Лучшие практики

### Обработка специальных типов
1. **Используйте `unknown` вместо `any`** для безопасной обработки данных
2. **Используйте `never`** для проверки исчерпания в объединениях
3. **Избегайте `any`** если возможно
4. **Используйте объединения с `null`/`undefined`** в strict режиме
5. **Используйте `void`** для функций без возвращаемого значения

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - как специальные типы совместимы
- [[type-system/type-guards]] - уточнение unknown и any
- [[union-types]] - объединения с null/undefined
- [[discriminated-unions]] - использование never для проверки исчерпания