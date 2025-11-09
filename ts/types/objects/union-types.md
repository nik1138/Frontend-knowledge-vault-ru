# Union Types

Объединения типов (Union Types) в TypeScript позволяют указать, что значение может быть одним из нескольких типов. Это обозначается с помощью оператора `|` и является одной из фундаментальных возможностей системы типов.

## Основы объединений

### Простые объединения
```typescript
// Объединение примитивных типов
let unionValue: string | number;
unionValue = "hello";    // OK
unionValue = 42;         // OK
// unionValue = true;     // Ошибка: boolean не входит в объединение

// Функция, принимающая объединение
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase();
  } else {
    return id.toString();
  }
}

// Использование
console.log(formatId("abc")); // "ABC"
console.log(formatId(123));   // "123"
```

### Объединения объектных типов
```typescript
interface User {
  type: "user";
  name: string;
  email: string;
}

interface Admin {
  type: "admin";
  name: string;
  privileges: string[];
}

// Объединение объектных типов
type UserRole = User | Admin;

function printUserRole(role: UserRole) {
  console.log(`Name: ${role.name}`);
  console.log(`Type: ${role.type}`);
  
  // TypeScript знает, что type существует у обоих вариантов
  
  // Но доступ к специфичным свойствам требует проверки
  if (role.type === "admin") {
    // Теперь TypeScript знает, что это Admin
    console.log(`Privileges: ${role.privileges.join(", ")}`);
  } else {
    // Это User (так как type === "user")
    console.log(`Email: ${role.email}`);
  }
}
```

## Сужение типов (Type Narrowing)

### Проверка типов
```typescript
// Использование typeof для сужения
function processValue(value: string | number | boolean) {
  if (typeof value === "string") {
    // В этой ветке value имеет тип string
    return value.toUpperCase();
  } else if (typeof value === "number") {
    // В этой ветке value имеет тип number
    return value.toFixed(2);
  } else {
    // В этой ветке value имеет тип boolean
    return value ? "true" : "false";
  }
}

// Использование in оператора
interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

function move(animal: Fish | Bird) {
  if ('swim' in animal) {
    // animal имеет тип Fish
    animal.swim();
  } else {
    // animal имеет тип Bird
    animal.fly();
  }
}
```

### Утверждения типов и пользовательские проверки
```typescript
// Пользовательская функция-проверка типа
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function getPetMove(pet: Fish | Bird) {
  if (isFish(pet)) {
    // pet теперь имеет тип Fish
    pet.swim();
  } else {
    // pet теперь имеет тип Bird
    pet.fly();
  }
}
```

## Продвинутые объединения

### Объединения с null и undefined
```typescript
// Объединения часто включают null/undefined
function printUser(user: { name: string } | null | undefined) {
  if (user) {
    // user имеет тип { name: string }
    console.log(user.name);
  } else {
    console.log("No user provided");
  }
}

// Или с Optional Chaining
function printUserSafe(user: { name: string; age?: number } | null) {
  console.log(user?.name?.toUpperCase());
  console.log(user?.age?.toFixed(2));
}
```

### Объединения массивов
```typescript
// Массив значений разных типов
let values: (string | number)[] = ["hello", 42, "world", 123];

// Каждый элемент может быть строкой или числом
values.push("new string");
values.push(999);

// Обработка массива объединений
function processValues(items: (string | number)[]) {
  items.forEach(item => {
    if (typeof item === "string") {
      console.log("String:", item.toUpperCase());
    } else {
      console.log("Number:", item.toFixed(2));
    }
  });
}
```

## Дизайны с объединениями

### Тегированные объединения (Discriminated Unions)
```typescript
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

type RequestState = LoadingState | SuccessState | ErrorState;

function handleRequestState(state: RequestState) {
  // Сужение типа через дискриминант
  switch (state.status) {
    case "loading":
      // state имеет тип LoadingState
      if (state.progress) {
        console.log(`Загрузка: ${state.progress}%`);
      } else {
        console.log("Загрузка...");
      }
      break;
      
    case "success":
      // state имеет тип SuccessState
      console.log(`Успех: ${state.data}`);
      break;
      
    case "error":
      // state имеет тип ErrorState
      console.log(`Ошибка: ${state.error}`);
      break;
  }
}
```

### Объединения с обобщениями
```typescript
// Обобщенные объединения
function processInput<T>(input: T | T[]): T[] {
  if (Array.isArray(input)) {
    return input;
  } else {
    return [input];
  }
}

// Использование
const arr1 = processInput("hello");     // string[]
const arr2 = processInput([1, 2, 3]);   // number[]
const arr3 = processInput(42);          // number[]
```

## Практические примеры

### Обработка API ответов
```typescript
interface ApiSuccess<T> {
  success: true;
  data: T;
}

interface ApiError {
  success: false;
  error: string;
  statusCode: number;
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

async function fetchUserData(id: number): Promise<ApiResponse<{ name: string; email: string }>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (response.ok) {
      const data = await response.json();
      return { success: true, data };
    } else {
      return { 
        success: false, 
        error: "User not found", 
        statusCode: response.status 
      };
    }
  } catch (error) {
    return { 
      success: false, 
      error: (error as Error).message, 
      statusCode: 500 
    };
  }
}

// Использование
async function displayUser(id: number) {
  const response = await fetchUserData(id);
  
  if (response.success) {
    // response имеет тип ApiSuccess
    console.log(`User: ${response.data.name}`);
  } else {
    // response имеет тип ApiError
    console.log(`Error ${response.statusCode}: ${response.error}`);
  }
}
```

### Опциональные параметры
```typescript
// Использование объединений для опциональных параметров
function createElement(
  tag: "div" | "span" | "p" | "a",
  options?: { 
    text?: string; 
    classes?: string[]; 
    attrs?: { [key: string]: string } 
  }
) {
  const element = document.createElement(tag);
  
  if (options?.text) {
    element.textContent = options.text;
  }
  
  if (options?.classes) {
    element.classList.add(...options.classes);
  }
  
  if (options?.attrs) {
    Object.entries(options.attrs).forEach(([key, value]) => {
      element.setAttribute(key, value);
    });
  }
  
  return element;
}

// Использование
const div = createElement("div", { text: "Hello", classes: ["container", "active"] });
```

## Распределительные объединения

### Как объединения работают с обобщениями
```typescript
// Объединения распределяются в обобщенных типах
type ToArray<T> = T extends any ? T[] : never;

// При использовании с объединением:
type T1 = ToArray<string | number>;  // string[] | number[]
// Это эквивалентно:
// (string extends any ? string[] : never) | (number extends any ? number[] : never)

// Пример использования
function wrapInArray<T>(value: T): ToArray<T> {
  return [value] as ToArray<T>;
}

const result = wrapInArray("hello" as string | number);
// result имеет тип: string[] | number[]
```

## Лучшие практики

### Сужение типов
```typescript
// Хорошо: явное сужение типов
function handleValue(value: string | number | boolean) {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  } else {
    return value.toString();
  }
}

// Использование предикатов типа для сложных проверок
interface Circle { kind: "circle"; radius: number; }
interface Square { kind: "square"; side: number; }
type Shape = Circle | Square;

function isCircle(shape: Shape): shape is Circle {
  return shape.kind === "circle";
}

function getArea(shape: Shape): number {
  if (isCircle(shape)) {
    // TypeScript знает, что это Circle
    return Math.PI * shape.radius ** 2;
  } else {
    // TypeScript знает, что это Square
    return shape.side ** 2;
  }
}
```

### Избегайте слишком сложных объединений
```typescript
// ПЛОХО: слишком сложное объединение
type ComplexUnion = 
  | { type: "a"; propA: string }
  | { type: "b"; propB: number } 
  | { type: "c"; propC: boolean }
  | { type: "d"; propD: string[] }
  | { type: "e"; propE: { nested: number } };

// ЛУЧШЕ: разбить на логические группы
type SimpleShape = 
  | { type: "circle"; radius: number }
  | { type: "rectangle"; width: number; height: number };

type ComplexShape = 
  | { type: "composite"; shapes: SimpleShape[] }
  | { type: "transformed"; shape: SimpleShape; transform: string };
```

## Ошибки и ловушки

### Частые ошибки с объединениями
```typescript
// Ошибка: доступ к свойству, которого нет в каждом типе объединения
interface A { type: "a"; value: string; }
interface B { type: "b"; count: number; }

function badExample(union: A | B) {
  // ОШИБКА: не все типы в объединении имеют свойство 'value'
  // return union.value; // Ошибка компиляции
}

// Правильно: сначала сузить тип
function goodExample(union: A | B) {
  if (union.type === "a") {
    // Теперь union имеет тип A
    return union.value.toUpperCase();
  } else {
    // union имеет тип B
    return union.count.toString();
  }
}
```

## Связь с другими концепциями
- [[discriminated-unions]] - специальный случай объединений с дискриминантами
- [[type-system/type-guards]] - уточнение типов в объединениях
- [[intersection-types]] - противоположность объединений
- [[advanced-types/conditional-types]] - условные типы часто работают с объединениями