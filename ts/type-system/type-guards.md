# Type Guards

Type Guards (стражи типов) - это выражения, которые проверяют тип значения на этапе выполнения и обеспечивают сужение типов в соответствующих ветках кода. Это ключевая концепция для безопасной работы с объединениями типов в TypeScript.

## Основные понятия

### Что такое Type Guard
Type Guard - это выражение, которое возвращает `true` или `false` и сообщает TypeScript, что в определенной ветке кода переменная имеет конкретный тип. Это позволяет TypeScript безопасно сужать типы в условных блоках.

```typescript
// Простой пример с typeof
function processValue(value: string | number) {
  if (typeof value === "string") {
    // В этой ветке TypeScript знает, что value - string
    return value.toUpperCase(); // безопасный вызов строкового метода
  }
  // В этой ветке TypeScript знает, что value - number
  return value.toFixed(2); // безопасный вызов числового метода
}
```

## Встроенные Type Guards

### typeof оператор
```typescript
function processValue(value: string | number | boolean) {
  if (typeof value === "string") {
    // value: string
    return value.toUpperCase();
  } else if (typeof value === "number") {
    // value: number
    return value.toFixed(2);
  } else {
    // value: boolean
    return value ? "true" : "false";
  }
}

// typeof также работает с функциями
function processCallback(callback: (() => void) | string) {
  if (typeof callback === "function") {
    // callback: (() => void)
    callback();
  } else {
    // callback: string
    console.log(callback);
  }
}
```

### instanceof оператор
```typescript
class Dog {
  constructor(public name: string) {}
  woof() { return `${this.name} говорит гав!`; }
}

class Cat {
  constructor(public name: string) {}
  meow() { return `${this.name} говорит мяу!`; }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    // animal: Dog в этой ветке
    return animal.woof();
  } else {
    // animal: Cat в этой ветке
    return animal.meow();
  }
}

// instanceof с встроенными типами
function processDate(date: Date | string) {
  if (date instanceof Date) {
    // date: Date
    return date.toISOString();
  } else {
    // date: string
    return new Date(date).toISOString();
  }
}
```

### in оператор
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
  if ("fly" in pet) {
    // pet: Bird
    pet.fly();
  } else {
    // pet: Fish
    pet.swim();
  }
}

// Проверка свойств в объектах
interface User {
  name: string;
  email: string;
}

interface Admin {
  name: string;
  email: string;
  role: "admin";
  permissions: string[];
}

function processAccount(account: User | Admin) {
  console.log(account.name); // доступно в обоих типах
  
  if ("role" in account) {
    // account: Admin
    console.log(account.permissions.join(", "));
  } else {
    // account: User
    console.log("Regular user account");
  }
}
```

## Пользовательские Type Guards

### Type Predicate Functions
```typescript
// Функция-предикат типа: (val: any) => val is SomeType
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isArray(value: unknown): value is unknown[] {
  return Array.isArray(value);
}

// Использование
function processUnknown(value: unknown) {
  if (isString(value)) {
    // value: string в этой ветке
    return value.toUpperCase();
  } else if (isNumber(value)) {
    // value: number в этой ветке
    return value.toFixed(2);
  } else if (isArray(value)) {
    // value: unknown[] в этой ветке
    return value.length;
  }
  return "Unknown type";
}

// Примеры более специфичных type guards
function isHTMLElement(element: unknown): element is HTMLElement {
  return element instanceof HTMLElement;
}

function isMouseEvent(event: unknown): event is MouseEvent {
  return event instanceof MouseEvent;
}

function isKeyboardEvent(event: unknown): event is KeyboardEvent {
  return event instanceof KeyboardEvent;
}
```

### Сложные пользовательские guards
```typescript
// Проверка на конкретный интерфейс
interface Dog {
  breed: string;
  bark(): void;
}

interface Cat {
  breed: string;
  meow(): void;
}

function isDog(animal: any): animal is Dog {
  return (
    animal &&
    typeof animal.breed === "string" &&
    typeof animal.bark === "function"
  );
}

function isCat(animal: any): animal is Cat {
  return (
    animal &&
    typeof animal.breed === "string" &&
    typeof animal.meow === "function"
  );
}

function makeSound(animal: unknown) {
  if (isDog(animal)) {
    // animal: Dog
    animal.bark();
  } else if (isCat(animal)) {
    // animal: Cat
    animal.meow();
  } else {
    // animal: unknown
    console.log("Unknown animal");
  }
}

// Проверка на объект с конкретной структурой
interface ConfigObject {
  apiUrl: string;
  timeout: number;
  debug?: boolean;
}

function isConfigObject(obj: any): obj is ConfigObject {
  return (
    obj &&
    typeof obj === "object" &&
    typeof obj.apiUrl === "string" &&
    typeof obj.timeout === "number"
  );
}
```

## Literal Type Guards

### Проверка конкретных значений
```typescript
type Status = "loading" | "success" | "error";

function handleStatus(status: Status) {
  if (status === "loading") {
    // status: "loading"
    console.log("Загрузка...");
  } else if (status === "success") {
    // status: "success"
    console.log("Успех!");
  } else {
    // status: "error" (поскольку остался только один вариант)
    console.log("Ошибка!");
  }
}

// Сравнение с литералами в объектах
interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState {
  status: "success";
  data: string;
}

interface ErrorState {
  status: "error";
  error: string;
}

type ResponseState = LoadingState | SuccessState | ErrorState;

function handleResponse(response: ResponseState) {
  if (response.status === "loading") {
    // response: LoadingState
    console.log(`Загрузка: ${response.progress ?? 0}%`);
  } else if (response.status === "success") {
    // response: SuccessState
    console.log(`Результат: ${response.data}`);
  } else {
    // response: ErrorState
    console.log(`Ошибка: ${response.error}`);
  }
}
```

## Discriminated Unions

### Использование discriminant properties
```typescript
// Дискриминированные объединения используют литеральные type guards
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Square | Triangle;

function calculateArea(shape: Shape): number {
  switch (shape.kind) {  // `kind` - это discriminator
    case "circle":
      // shape: Circle
      return Math.PI * shape.radius ** 2;
    case "square":
      // shape: Square
      return shape.side ** 2;
    case "triangle":
      // shape: Triangle
      return (shape.base * shape.height) / 2;
  }
}

// Комплексный пример с несколькими дискриминантами
type RequestStatus = "idle" | "loading" | "success" | "error";

interface IdleState {
  status: "idle";
}

interface LoadingState {
  status: "loading";
  startTime: number;
}

interface SuccessState<T> {
  status: "success";
  data: T;
  duration: number;
}

interface ErrorState {
  status: "error";
  error: Error;
  duration: number;
}

type RequestState<T> = IdleState | LoadingState | SuccessState<T> | ErrorState;

function getRequestInfo<T>(state: RequestState<T>) {
  switch (state.status) {
    case "idle":
      // state: IdleState
      return "Request idle";
    case "loading":
      // state: LoadingState
      return `Loading for ${(Date.now() - state.startTime) / 1000}s`;
    case "success":
      // state: SuccessState<T>
      return `Success after ${state.duration}ms with ${JSON.stringify(state.data)}`;
    case "error":
      // state: ErrorState
      return `Error after ${state.duration}ms: ${state.error.message}`;
  }
}
```

## Assertion Functions

### Custom assertion functions
```typescript
// Assertion functions не возвращаются при false
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Expected value to be a string");
  }
}

function assertNumber(value: unknown): asserts value is number {
  if (typeof value !== "number") {
    throw new Error("Expected value to be a number");
  }
}

function assert(condition: boolean, message: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

// Использование в коде
function processValue(value: unknown) {
  assertString(value);
  // После вызова assertString, TypeScript знает, что value - string
  return value.toUpperCase();
}

function processData(data: { name: unknown; age: unknown }) {
  assertString(data.name);
  assertNumber(data.age);
  
  // Теперь TypeScript знает типы
  return {
    name: data.name.toUpperCase(), // data.name: string
    age: data.age.toFixed(2)       // data.age: number
  };
}
```

## Advanced Guard Patterns

### Combining guards
```typescript
// Комбинирование нескольких guards
function isNonNull<T>(value: T | null | undefined): value is T {
  return value !== null && value !== undefined;
}

function isNonEmptyString(value: unknown): value is string {
  return typeof value === "string" && value.length > 0;
}

function processUserInput(input: string | null | undefined) {
  if (isNonNull(input) && isNonEmptyString(input)) {
    // input: string (после обеих проверок)
    return input.trim().toUpperCase();
  }
  return null;
}

// Сложные проверки
function isValidEmail(value: unknown): value is string {
  return typeof value === "string" && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}

function validateUser(data: any): data is { name: string; email: string } {
  return (
    data &&
    typeof data === "object" &&
    isNonEmptyString(data.name) &&
    isValidEmail(data.email)
  );
}

function handleUserData(unknownData: unknown) {
  if (validateUser(unknownData)) {
    // unknownData: { name: string; email: string }
    console.log(`User: ${unknownData.name}, Email: ${unknownData.email}`);
  } else {
    console.log("Invalid user data");
  }
}
```

### Guard factories
```typescript
// Функция для создания guards
function createGuard<T>(predicate: (value: any) => boolean) {
  return (value: any): value is T => predicate(value);
}

// Создание конкретных guards
const isValidDate = createGuard<Date>((value: any): value is Date => {
  return value instanceof Date && !isNaN(value.getTime());
});

const isArrayOfStrings = createGuard<string[]>((value: any): value is string[] => {
  return Array.isArray(value) && value.every(item => typeof item === "string");
});

// Использование
function processDate(date: unknown) {
  if (isValidDate(date)) {
    // date: Date
    return date.toISOString();
  }
  return null;
}

function processStringArray(arr: unknown) {
  if (isArrayOfStrings(arr)) {
    // arr: string[]
    return arr.map(s => s.toUpperCase());
  }
  return [];
}
```

## Practical Examples

### API Response Handling
```typescript
interface SuccessResponse<T> {
  success: true;
  data: T;
}

interface ErrorResponse {
  success: false;
  error: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccessful<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.success === true;
}

function handleApiResponse<T>(response: ApiResponse<T>) {
  if (isSuccessful(response)) {
    // response: SuccessResponse<T>
    console.log("Data received:", response.data);
    return response.data;
  } else {
    // response: ErrorResponse
    console.error("Error:", response.error);
    throw new Error(response.error);
  }
}

// Использование
const successResponse: ApiResponse<string> = {
  success: true,
  data: "Operation completed"
};

const errorResponse: ApiResponse<string> = {
  success: false,
  error: "Something went wrong"
};

const result = handleApiResponse(successResponse); // string
```

### Event Handling
```typescript
// Type guards для событий
type EventType = "mouse" | "keyboard" | "touch";

interface BaseEvent {
  type: EventType;
  timestamp: number;
}

interface MouseEvent extends BaseEvent {
  type: "mouse";
  x: number;
  y: number;
  button: number;
}

interface KeyboardEvent extends BaseEvent {
  type: "keyboard";
  key: string;
  code: string;
  ctrlKey: boolean;
  shiftKey: boolean;
}

interface TouchEvent extends BaseEvent {
  type: "touch";
  touches: { x: number; y: number }[];
}

type GeneralEvent = MouseEvent | KeyboardEvent | TouchEvent;

function isMouseEvent(event: GeneralEvent): event is MouseEvent {
  return event.type === "mouse";
}

function isKeyboardEvent(event: GeneralEvent): event is KeyboardEvent {
  return event.type === "keyboard";
}

function isTouchEvent(event: GeneralEvent): event is TouchEvent {
  return event.type === "touch";
}

function handleEvent(event: GeneralEvent) {
  if (isMouseEvent(event)) {
    // event: MouseEvent
    console.log(`Mouse click at (${event.x}, ${event.y})`);
  } else if (isKeyboardEvent(event)) {
    // event: KeyboardEvent
    console.log(`Key pressed: ${event.key}, Ctrl: ${event.ctrlKey}`);
  } else if (isTouchEvent(event)) {
    // event: TouchEvent
    console.log(`Touch points: ${event.touches.length}`);
  }
}
```

## Best Practices

### Exhaustiveness checking
```typescript
// Использование never для проверки исчерпания
type Action = 
  | { type: "ADD"; payload: { id: number; name: string } }
  | { type: "REMOVE"; payload: { id: number } }
  | { type: "UPDATE"; payload: { id: number; changes: Partial<{ name: string }> } };

function handleAction(action: Action) {
  switch (action.type) {
    case "ADD":
      console.log("Adding", action.payload);
      break;
    case "REMOVE":
      console.log("Removing", action.payload);
      break;
    case "UPDATE":
      console.log("Updating", action.payload);
      break;
    default:
      // Если добавить новый тип в Action без обновления switch,
      // будет ошибка: string не совместим с never
      const exhaustive: never = action;
      throw new Error(`Unhandled action type: ${exhaustive}`);
  }
}
```

### Reusable guards
```typescript
// Гарды должны быть переиспользуемыми
const isNotNull = <T>(value: T | null | undefined): value is T => 
  value !== null && value !== undefined;

const isString = (value: unknown): value is string => 
  typeof value === "string";

const isNumber = (value: unknown): value is number => 
  typeof value === "number";

const isBoolean = (value: unknown): value is boolean => 
  typeof value === "boolean";

// Использование в функциях
function safeParseInt(value: unknown): number | null {
  if (isString(value) && isNotNull(value)) {
    const parsed = parseInt(value, 10);
    return isNaN(parsed) ? null : parsed;
  }
  return null;
}
```

## Common Mistakes

### Incorrect guard implementations
```typescript
// ПЛОХО: функция не является type guard (должна использовать value is Type)
function isStringBad(value: unknown): boolean {
  return typeof value === "string";
}

// Это НЕ будет работать как type guard:
// if (isStringBad(someValue)) {
//   someValue.toUpperCase(); // ОШИБКА: TypeScript не знает тип
// }

// ПРАВИЛЬНО: использование type predicate
function isStringGood(value: unknown): value is string {
  return typeof value === "string";
}

// Теперь работает:
// if (isStringGood(someValue)) {
//   someValue.toUpperCase(); // OK: TypeScript знает, что это строка
// }
```

### Overly broad guards
```typescript
// ПЛОХО: слишком общий guard
function isObject(value: unknown): value is object {
  return typeof value === "object" && value !== null;
}

// Это не очень полезно, так как object включает в себя Array, Date, RegExp, и т.д.

// ЛУЧШЕ: более конкретный guard
function isPlainObject(value: unknown): value is Record<string, any> {
  return typeof value === "object" && value !== null && !Array.isArray(value) && !(value instanceof Date);
}
```

## Advanced Topics

### Control flow analysis with guards
```typescript
// TypeScript анализирует поток выполнения с guards
function processInput(input: string | number | null) {
  if (typeof input === "string") {
    // input: string
    return input.toUpperCase();
  }
  
  // На этом этапе input: number | null
  if (input === null) {
    // input: null
    return "NULL_VALUE";
  }
  
  // На этом этапе input: number (null исключен)
  return input.toFixed(2);
}

// Guards и циклы
function findString(arr: (string | number)[]): string | undefined {
  for (const item of arr) {
    if (typeof item === "string") {
      // item: string в этом месте
      return item;
    }
  }
  return undefined;
}
```

## Integration with other TypeScript features

### With generics
```typescript
// Generic type guards
function isType<T>(value: unknown, typeName: string): value is T {
  return typeof value === typeName;
}

// Или более сложный generic guard
function hasProperty<O, K extends keyof O>(
  obj: O,
  key: K
): obj is O & Record<K, NonNullable<O[K]>> {
  return obj[key] !== null && obj[key] !== undefined;
}
```

### With utility types
```typescript
// Combinig with utility types
type RequiredKeys<T> = {
  [K in keyof T]-?: T[K] extends undefined ? never : K;
}[keyof T];

function hasRequiredKeys<T, K extends RequiredKeys<T>>(
  obj: T,
  key: K
): obj is T & { [P in K]: NonNullable<T[P]> } {
  return obj[key] !== undefined;
}
```

## Conclusion

Type Guards являются важнейшим инструментом для безопасной работы с объединениями типов в TypeScript. Они позволяют:

1. **Сужать типы** в условных выражениях
2. **Обеспечивать безопасность типов** при работе с динамическими данными
3. **Улучшать автодополнение** и проверки в редакторе
4. **Делать код более читаемым** и самодокументированным

Правильное использование type guards значительно повышает надежность TypeScript приложений.