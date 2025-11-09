# Защитники типов в TypeScript

Защитники типов (type guards) - это выражения, которые проверяют тип переменной в момент выполнения и позволяют TypeScript сужать (narrow) тип переменной в определенной ветке кода. Они обеспечивают безопасность типов при работе с объединениями типов и позволяют избежать ошибок во время выполнения.

## Основная концепция

Защитник типа - это функция, которая возвращает `value is SomeType`, где `value` - параметр функции, а `SomeType` - тип, который будет присвоен параметру, если функция возвращает `true`.

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function processValue(value: string | number) {
  if (isString(value)) {
    // В этой ветке TypeScript знает, что value - это string
    console.log(value.toUpperCase()); // OK
  } else {
    // В этой ветке TypeScript знает, что value - это number
    console.log(value.toFixed(2)); // OK
  }
}
```

## Встроенные защитники типов

### 1. Проверка typeof

```typescript
function processValue(value: string | number) {
  if (typeof value === "string") {
    // Здесь value сужается до string
    console.log(value.toUpperCase());
  } else {
    // Здесь value сужается до number
    console.log(value.toFixed(2));
  }
}
```

### 2. Проверка instanceof

```typescript
class Dog {
  name: string;
  constructor(name: string) { this.name = name; }
  bark() { console.log("Гав!"); }
}

class Cat {
  name: string;
  constructor(name: string) { this.name = name; }
  meow() { console.log("Мяу!"); }
}

function processAnimal(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    // Здесь animal сужается до Dog
    animal.bark();
  } else {
    // Здесь animal сужается до Cat
    animal.meow();
  }
}
```

### 3. Проверка in

```typescript
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(pet: Bird | Fish) {
  if ("swim" in pet) {
    // pet сужается до Fish
    pet.swim();
  } else {
    // pet сужается до Bird
    pet.fly();
  }
}
```

## Пользовательские защитники типов

### 1. Базовые защитники

```typescript
interface Admin {
  name: string;
  privileges: string[];
}

interface Employee {
  name: string;
  startDate: Date;
}

type UnknownEmployee = Employee | Admin;

function isAdmin(emp: UnknownEmployee): emp is Admin {
  return "privileges" in emp;
}

function isEmployee(emp: UnknownEmployee): emp is Employee {
  return "startDate" in emp;
}

function printEmployeeInformation(emp: UnknownEmployee) {
  console.log("Имя: " + emp.name);

  if (isAdmin(emp)) {
    // Здесь emp сужается до Admin
    console.log("Привилегии: " + emp.privileges.join(", "));
  } else {
    // Здесь emp сужается до Employee
    console.log("Дата начала: " + emp.startDate);
  }
}
```

### 2. Защитники для литеральных типов

```typescript
type Result = 
  | { success: true; value: number }
  | { success: false; error: string };

function isSuccess(result: Result): result is { success: true; value: number } {
  return result.success;
}

function processResult(result: Result) {
  if (isSuccess(result)) {
    // Здесь result сужается до { success: true; value: number }
    console.log("Значение:", result.value);
  } else {
    // Здесь result сужается до { success: false; error: string }
    console.error("Ошибка:", result.error);
  }
}
```

## Продвинутые паттерны

### 1. Защитники для сложных типов

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

type Shape = Circle | Square;

// Защитник для конкретного типа в дискриминированном объединении
function isCircle(shape: Shape): shape is Circle {
  return shape.kind === "circle";
}

function isSquare(shape: Shape): shape is Square {
  return shape.kind === "square";
}

function getArea(shape: Shape): number {
  if (isCircle(shape)) {
    // shape сужается до Circle
    return Math.PI * shape.radius ** 2;
  } else {
    // shape сужается до Square
    return shape.side ** 2;
  }
}
```

### 2. Комбинированные защитники

```typescript
function isNotNull<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function processInput(input: string | number | null | undefined) {
  if (isNotNull(input) && isNumber(input)) {
    // input сужается до number
    return input * 2;
  }
  
  return 0;
}
```

### 3. Защитники для массивов

```typescript
function isStringArray(value: unknown): value is string[] {
  return Array.isArray(value) && value.every(item => typeof item === "string");
}

function processArray(arr: unknown) {
  if (isStringArray(arr)) {
    // arr сужается до string[]
    return arr.map(s => s.toUpperCase());
  }
  
  return [];
}
```

## Практические применения

### 1. Работа с API ответами

```typescript
interface ApiResponse<T> {
  data?: T;
  error?: string;
  status: number;
}

function isSuccessfulResponse<T>(response: ApiResponse<T>): response is ApiResponse<T> & { data: T } {
  return response.status >= 200 && response.status < 300 && 'data' in response;
}

function isErrorResponse<T>(response: ApiResponse<T>): response is ApiResponse<T> & { error: string } {
  return response.status >= 400 || 'error' in response;
}

function handleResponse<T>(response: ApiResponse<T>) {
  if (isSuccessfulResponse(response)) {
    // response имеет тип с обязательным полем data
    console.log("Данные:", response.data);
  } else if (isErrorResponse(response)) {
    // response имеет тип с обязательным полем error
    console.error("Ошибка:", response.error);
  } else {
    console.log("Неопределенный статус:", response.status);
  }
}
```

### 2. Валидация данных

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function isValidUser(obj: any): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    typeof obj.id === "number" &&
    typeof obj.name === "string" &&
    typeof obj.email === "string" &&
    obj.email.includes("@")
  );
}

function processUserData(data: unknown) {
  if (isValidUser(data)) {
    // data сужается до User
    console.log(`Пользователь: ${data.name}, email: ${data.email}`);
  } else {
    console.error("Неверный формат данных пользователя");
  }
}
```

### 3. Работа с различными типами данных

```typescript
type Primitive = string | number | boolean;

function isPrimitive(value: unknown): value is Primitive {
  return ["string", "number", "boolean"].includes(typeof value);
}

function deepClone<T>(obj: T): T {
  if (!isPrimitive(obj) && obj !== null && typeof obj === "object") {
    if (Array.isArray(obj)) {
      return obj.map(item => deepClone(item)) as any;
    } else {
      const cloned: any = {};
      for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
          cloned[key] = deepClone(obj[key]);
        }
      }
      return cloned;
    }
  }
  return obj;
}
```

## Защитники типов в функциях высшего порядка

```typescript
function filterByType<T, U extends T>(
  items: T[],
  predicate: (item: T) => item is U
): U[] {
  return items.filter(predicate);
}

interface Dog { type: "dog"; name: string; breed: string; }
interface Cat { type: "cat"; name: string; age: number; }

type Animal = Dog | Cat;

const animals: Animal[] = [
  { type: "dog", name: "Buddy", breed: "Golden Retriever" },
  { type: "cat", name: "Whiskers", age: 3 }
];

const dogs = filterByType(animals, (animal): animal is Dog => animal.type === "dog");
// dogs имеет тип Dog[]
```

## Ограничения и особенности

### 1. Ограничения на возврат

Защитник типа должен возвращать `value is SomeType`, а не просто булевое значение:

```typescript
// Правильно
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Неправильно - не работает как защитник типа
function checkIsString(value: unknown): boolean {
  return typeof value === "string";
}
```

### 2. Ограничения области действия

Защитники типов действуют только в области, где они проверяются:

```typescript
function example(input: string | number) {
  if (isString(input)) {
    // Здесь input - string
    console.log(input.toUpperCase());
  }
  // Здесь input снова string | number
  // console.log(input.toUpperCase()); // Ошибка!
}
```

### 3. Ассертивные функции

Иногда используются ассертивные функции вместо защитников:

```typescript
function assert(condition: boolean, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

function assertIsString(value: unknown): asserts value is string {
  assert(typeof value === "string", "Значение должно быть строкой");
}

function processValue(value: unknown) {
  assertIsString(value);
  // После этой строки TypeScript знает, что value - string
  console.log(value.toUpperCase());
}
```

## Практические примеры из реальной разработки

### 1. Защитники для работы с DOM

```typescript
function isInputElement(element: HTMLElement): element is HTMLInputElement {
  return element.tagName === "INPUT";
}

function isButtonElement(element: HTMLElement): element is HTMLButtonElement {
  return element.tagName === "BUTTON";
}

function handleElement(element: HTMLElement) {
  if (isInputElement(element)) {
    // element сужается до HTMLInputElement
    console.log("Значение инпута:", element.value);
  } else if (isButtonElement(element)) {
    // element сужается до HTMLButtonElement
    console.log("Текст кнопки:", element.textContent);
  }
}
```

### 2. Защитники для работы с ошибками

```typescript
interface CustomError {
  name: string;
  message: string;
  code: number;
}

function isCustomError(error: unknown): error is CustomError {
  return (
    typeof error === "object" &&
    error !== null &&
    "name" in error &&
    "message" in error &&
    "code" in error &&
    typeof (error as any).code === "number"
  );
}

function handleError(error: unknown) {
  if (isCustomError(error)) {
    // error сужается до CustomError
    console.error(`Код ошибки ${error.code}: ${error.message}`);
  } else if (error instanceof Error) {
    // error сужается до Error
    console.error(`Ошибка: ${error.message}`);
  } else {
    console.error("Неизвестная ошибка:", error);
  }
}
```

## Лучшие практики

1. **Используйте понятные имена для защитников** - `isString`, `isValidUser` и т.д.
2. **Документируйте защитники** - особенно сложные или неочевидные
3. **Избегайте сложных логических выражений** - разбивайте сложные проверки
4. **Тестируйте защитники** - убедитесь, что они работают корректно
5. **Используйте ассертивные функции** - когда нужно прервать выполнение при ошибке
6. **Создавайте переиспользуемые защитники** - для часто используемых проверок

## Связи с другими концепциями

- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - часто используются вместе с защитниками
- [[ts/basics/control-flow|Анализ потока управления]] - основа для сужения типов
- [[ts/type-system/type-compatibility|Совместимость типов]] - как сужение влияет на совместимость
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использовать infer в защитниках
- [[js/error-handling]] - обработка ошибок в JavaScript/TypeScript