# Утверждения типов и сужение типов в TypeScript

Утверждения типов (type assertions) и сужение типов (type narrowing) - это важные механизмы в TypeScript, которые позволяют разработчику сообщать компилятору о конкретном типе значения, когда система типов не может определить его автоматически.

## Утверждения типов (Type Assertions)

Утверждения типов позволяют указать TypeScript, что вы знаете больше о типе значения, чем может определить система типов.

### Синтаксис утверждений типов

```typescript
// Синтаксис с угловыми скобками (не рекомендуется в JSX)
let someValue: any = "hello world";
let strLength1: number = (<string>someValue).length;

// Синтаксис с as (рекомендуемый)
let strLength2: number = (someValue as string).length;

// Утверждение типа переменной напрямую
let strLength3: number = (someValue as string).length;
```

## Простые примеры утверждений типов

### 1. Работа с any типом

```typescript
function getLength(value: any): number {
  // value может быть строкой, массивом или чем-то еще
  if (typeof value === "string" || Array.isArray(value)) {
    return value.length; // Ошибка: TypeScript не знает, что у value есть свойство length
  }
  return 0;
}

// С утверждением типа:
function getLengthFixed(value: any): number {
  if (typeof value === "string" || Array.isArray(value)) {
    return (value as string | any[]).length; // OK
  }
  return 0;
}
```

### 2. Работа с DOM элементами

```typescript
// Получение элемента по ID
const inputElement = document.getElementById("inputId");
// inputElement имеет тип HTMLElement | null

// Утверждение типа для получения доступа к специфичным свойствам
const input = inputElement as HTMLInputElement;
// Теперь можно обращаться к свойствам input.value, input.checked и т.д.

// Более безопасный способ - двойное утверждение
const specificInput = document.getElementById("inputId") as unknown as HTMLInputElement;
```

### 3. Двойные утверждения

```typescript
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

// Иногда нужно использовать двойное утверждение для обхода ограничений системы типов
const animal: Cat = { meow() {} };

// Это может быть нужно, когда утверждение несовместимо
const dog = animal as unknown as Dog;
// dog.bark(); // Ошибка выполнения, но TypeScript не ругается
```

## Сужение типов (Type Narrowing)

Сужение типов - это процесс, при котором TypeScript уточняет тип переменной в определенной области кода на основе проверок.

### 1. Проверка typeof

```typescript
function processValue(value: string | number) {
  if (typeof value === "string") {
    // В этой ветке TypeScript знает, что value - это string
    console.log(value.toUpperCase()); // OK
    console.log(value.length); // OK
  } else {
    // В этой ветке TypeScript знает, что value - это number
    console.log(value.toFixed(2)); // OK
    console.log(value * 2); // OK
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
    // animal сужается до Dog
    animal.bark();
    console.log(animal.name);
  } else {
    // animal сужается до Cat
    animal.meow();
    console.log(animal.name);
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

## Защитники типов (Type Guards)

Защитники типов - это функции, которые возвращают `value is SomeType`, позволяя TypeScript сужать тип за пределами функции.

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return "swim" in pet;
}

function getFood(pet: Fish | Bird) {
  if (isFish(pet)) {
    // pet сужается до Fish
    return "водоросли";
  } else {
    // pet сужается до Bird
    return "черви";
  }
}
```

## Продвинутые паттерны сужения типов

### 1. Дискриминированные объединения

```typescript
interface Loading {
  state: "loading";
}

interface Success {
  state: "success";
  data: string;
}

interface Error {
  state: "error";
  message: string;
}

type Status = Loading | Success | Error;

function handleStatus(status: Status) {
  if (status.state === "loading") {
    // status сужается до Loading
    console.log("Загрузка...");
  } else if (status.state === "success") {
    // status сужается до Success
    console.log(`Данные: ${status.data}`);
  } else {
    // status сужается до Error
    console.log(`Ошибка: ${status.message}`);
  }
}
```

### 2. Ассертивные функции

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

### 2. Валидация пользовательского ввода

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

### 3. Работа с не полностью типизированными библиотеками

```typescript
// Предположим, что внешняя библиотека возвращает any
declare function externalLibraryFunction(): any;

interface ExpectedResult {
  id: number;
  name: string;
  isValid: boolean;
}

function getTypedResult(): ExpectedResult {
  const result = externalLibraryFunction();
  
  // Утверждение типа с последующей валидацией
  const typedResult = result as ExpectedResult;
  
  // Дополнительная проверка структуры
  if (!isValidExpectedResult(typedResult)) {
    throw new Error("Неверная структура результата");
  }
  
  return typedResult;
}

function isValidExpectedResult(obj: any): obj is ExpectedResult {
  return (
    typeof obj === "object" &&
    obj !== null &&
    typeof obj.id === "number" &&
    typeof obj.name === "string" &&
    typeof obj.isValid === "boolean"
  );
}
```

## Опасности и лучшие практики

### 1. Избегайте небезопасных утверждений

```typescript
const someValue: unknown = { name: "Alice", age: 30 };

// Небезопасно:
const person1 = someValue as { name: string };
// Нет проверки, что someValue действительно имеет поле name

// Безопаснее:
if (typeof someValue === "object" && someValue !== null && "name" in someValue) {
  const person2 = someValue as { name: string };
  console.log(person2.name);
}
```

### 2. Используйте защитники типов вместо утверждений

```typescript
interface Admin {
  name: string;
  privileges: string[];
}

interface User {
  name: string;
  email: string;
}

type Person = Admin | User;

// Не рекомендуется:
function isAdmin(person: Person): boolean {
  return "privileges" in person;
}

function processPerson(person: Person) {
  if (isAdmin(person)) {
    // TypeScript не знает, что person - Admin, потому что isAdmin не является защитником типа
    // (person as Admin).privileges; // Лучше избежать такого
  }
}

// Рекомендуется:
function isAdminTyped(person: Person): person is Admin {
  return "privileges" in person;
}

function processPersonTyped(person: Person) {
  if (isAdminTyped(person)) {
    // Теперь TypeScript знает, что person - Admin
    console.log(person.privileges);
  }
}
```

### 3. Используйте as const для сохранения литеральных типов

```typescript
// Без as const TypeScript расширяет типы
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000
};
// тип: { apiUrl: string; timeout: number; }

// С as const TypeScript сохраняет литеральные типы
const configLiteral = {
  apiUrl: "https://api.example.com",
  timeout: 5000
} as const;
// тип: { readonly apiUrl: "https://api.example.com"; readonly timeout: 5000; }
```

## Практические примеры из реальной разработки

### 1. Работа с DOM и типизацией элементов

```typescript
function handleFormSubmit(event: Event) {
  // Утверждение типа для получения доступа к специфичным свойствам формы
  const form = event.target as HTMLFormElement;
  
  // Дальнейшее сужение типов для конкретных элементов формы
  const emailInput = form.elements.namedItem("email") as HTMLInputElement;
  const passwordInput = form.elements.namedItem("password") as HTMLInputElement;
  
  // Валидация данных
  if (!emailInput.value.includes("@")) {
    alert("Неверный формат email");
    return;
  }
  
  // Отправка данных
  submitLogin(emailInput.value, passwordInput.value);
}

// Более безопасный подход с проверками
function handleFormSubmitSafe(event: Event) {
  if (!(event.target instanceof HTMLFormElement)) {
    return; // Не форма
  }
  
  const form = event.target;
  const emailElement = form.elements.namedItem("email");
  const passwordElement = form.elements.namedItem("password");
  
  if (!(emailElement instanceof HTMLInputElement) || 
      !(passwordElement instanceof HTMLInputElement)) {
    return; // Элементы не являются input
  }
  
  // Теперь TypeScript знает типы элементов
  if (!emailElement.value.includes("@")) {
    alert("Неверный формат email");
    return;
  }
  
  submitLogin(emailElement.value, passwordElement.value);
}
```

### 2. Работа с асинхронными операциями

```typescript
interface ApiSuccess<T> {
  success: true;
  data: T;
}

interface ApiError {
  success: false;
  error: string;
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

async function fetchData<T>(url: string): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return { success: false, error: `HTTP ${response.status}` };
    }
    
    const data = await response.json() as T; // Утверждение типа для данных
    return { success: true, data };
  } catch (error) {
    return { success: false, error: (error as Error).message };
  }
}

// Использование с сужением типов
async function processUserData(userId: number) {
  const response = await fetchData<User>(`/api/users/${userId}`);
  
  if (response.success) {
    // response сужается до ApiSuccess<User>
    console.log("Пользователь:", response.data.name);
  } else {
    // response сужается до ApiError
    console.error("Ошибка:", response.error);
  }
}
```

### 3. Создание типобезопасных утилит

```typescript
// Утилита для безопасного доступа к вложенным свойствам
function safeGet<T, K extends keyof T>(obj: T, key: K): T[K] | undefined {
  try {
    return obj[key];
  } catch {
    return undefined;
  }
}

// Утилита для проверки, является ли значение определенного типа
function isOfType<T>(value: any, validator: (val: any) => val is T): value is T {
  return validator(value);
}

// Пример валидатора
function isValidUserType(obj: any): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    typeof obj.id === "number" &&
    typeof obj.name === "string" &&
    typeof obj.email === "string"
  );
}

// Использование
const data: unknown = { id: 1, name: "John", email: "john@example.com" };
if (isOfType(data, isValidUserType)) {
  // data теперь типизирован как User
  console.log(`Пользователь: ${data.name}`);
}
```

## Ограничения и особенности

### 1. Утверждения не проверяются во время выполнения

```typescript
const value: any = "hello";
const num = value as number; // Компилируется, но небезопасно
// console.log(num.toFixed(2)); // Ошибка выполнения: num.toFixed is not a function
```

### 2. Ограничения на утверждения между несовместимыми типами

```typescript
interface Source {
  name: string;
}

interface Target {
  age: number;
}

const source: Source = { name: "Test" };
// const target = source as Target; // Может сработать, если структуры не пересекаются
// TypeScript позволяет это, потому что в JS любой объект может быть преобразован в любой
```

## Лучшие практики

1. **Предпочитайте сужение типов утверждениям** - используйте проверки typeof, instanceof, in и защитники типов
2. **Используйте as const для литеральных значений** - для сохранения точных типов
3. **Избегайте утверждений any** - они отключают проверку типов
4. **Создавайте защитники типов для сложных проверок** - для переиспользуемой логики сужения
5. **Используйте ассертивные функции для проверок выполнения** - когда нужно прервать выполнение при ошибке
6. **Документируйте использование утверждений типов** - особенно когда они необходимы по объективным причинам

## Связи с другими концепциями

- [[ts/type-system/type-guards|Защитники типов]] - расширенное понимание сужения типов
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - мощный паттерн сужения
- [[ts/basics/control-flow|Анализ потока управления]] - основа для сужения типов
- [[ts/type-system/type-compatibility|Совместимость типов]] - как сужение влияет на совместимость
- [[js/type-coercion]] - различия между JS и TS подходами к типизации