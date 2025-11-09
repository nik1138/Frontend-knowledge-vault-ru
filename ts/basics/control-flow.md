# Поток управления и сужение типов в TypeScript

## Введение

Поток управления (control flow) в TypeScript - это анализ того, как значение может изменяться в зависимости от выполнения различных участков кода. Сужение типов (type narrowing) - это процесс, при котором TypeScript уточняет тип переменной на основе проверок, выполненных в коде. Эти концепции позволяют TypeScript более точно определять типы переменных в различных участках кода, обеспечивая лучшую безопасность типов.

## Основные концепции потока управления

### Проверки типа (Type Guards)

Проверки типа позволяют TypeScript определить конкретный тип переменной в определенной ветке кода:

```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    // В этой ветке TypeScript знает, что padding - это number
    return " ".repeat(padding) + value;
  }
  // В этой ветке TypeScript знает, что padding - это string
  return padding + value;
}

// Пример с объектными типами
interface Bird {
  type: "bird";
  flyingSpeed: number;
}

interface Horse {
  type: "horse";
  runningSpeed: number;
}

function moveAnimal(animal: Bird | Horse) {
  // Дискриминированный объединение - проверка свойства type
  if (animal.type === "bird") {
    // Здесь TypeScript знает, что animal - это Bird
    console.log(`Птица летит со скоростью ${animal.flyingSpeed}`);
  } else {
    // Здесь TypeScript знает, что animal - это Horse
    console.log(`Лошадь бежит со скоростью ${animal.runningSpeed}`);
  }
}
```

### Проверки на null и undefined

```typescript
function liveDangerously(x?: number | null) {
  // Без проверки TypeScript не знает, является ли x null или undefined
  // console.log(x.toFixed()); // ОШИБКА: x может быть null или undefined

  if (x == null) {
    // Здесь x может быть null или undefined
    return 0;
  }

  // Здесь TypeScript знает, что x - это number
  return x.toFixed(); // OK
}

// Использование проверки на null/undefined
function processValue(value: string | null) {
  if (value) {
    // Здесь TypeScript знает, что value не null и не undefined
    return value.toUpperCase();
  }
  return "default";
}
```

## Сужение типов (Type Narrowing)

### Условные операторы

```typescript
function example(value: string | number) {
  if (typeof value === "string") {
    // Здесь value - это string
    return value.toUpperCase();
  }
  // Здесь TypeScript знает, что value - это number
  return value.toFixed(2);
}

// Сложные проверки
function getLength(x: unknown) {
  if (typeof x === "string" || Array.isArray(x)) {
    // Здесь x может быть string или массивом
    return x.length;
  }
  // В других случаях возвращаем 0
  return 0;
}
```

### Проверка на принадлежность классу

```typescript
class Car {
  drive() {
    console.log("Машина едет");
  }
}

class Truck {
  loadCargo(cargo: string) {
    console.log(`Грузим ${cargo}`);
  }
}

type Vehicle = Car | Truck;

function useVehicle(vehicle: Vehicle) {
  if (vehicle instanceof Car) {
    // Здесь TypeScript знает, что vehicle - это Car
    vehicle.drive();
  } else {
    // Здесь TypeScript знает, что vehicle - это Truck
    vehicle.loadCargo("коробки");
  }
}
```

### Проверка на принадлежность свойству

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

function printEmployeeInformation(emp: UnknownEmployee) {
  console.log("Имя: " + emp.name);

  if ("privileges" in emp) {
    // Здесь TypeScript знает, что emp - это Admin
    console.log("Привилегии: " + emp.privileges);
  }

  if ("startDate" in emp) {
    // Здесь TypeScript знает, что emp - это Employee
    console.log("Дата начала работы: " + emp.startDate);
  }
}
```

## Продвинутые паттерны сужения типов

### Определяемые пользователем проверки типов

```typescript
// Функция-проверка типа (type predicate)
function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function isString(value: unknown): value is string {
  return typeof value === "string";
}

function processValue(value: unknown) {
  if (isNumber(value)) {
    // Здесь TypeScript знает, что value - это number
    return value.toFixed(2);
  }

  if (isString(value)) {
    // Здесь TypeScript знает, что value - это string
    return value.toUpperCase();
  }

  return "Неизвестный тип";
}

// Проверка на существование свойства
interface Bird {
  fly(): void;
}

interface Dog {
  walk(): void;
}

function isBird(animal: Bird | Dog): animal is Bird {
  return "fly" in animal;
}

function move(animal: Bird | Dog) {
  if (isBird(animal)) {
    animal.fly(); // TypeScript знает, что это Bird
  } else {
    animal.walk(); // TypeScript знает, что это Dog
  }
}
```

### Сужение типов в циклах

```typescript
function processItems(items: (string | number)[]) {
  for (const item of items) {
    // В каждой итерации происходит сужение типа
    if (typeof item === "string") {
      console.log(item.toUpperCase());
    } else {
      console.log(item.toFixed(2));
    }
  }
}

// Сужение типов в массиве с разными типами
function handleMixedArray(arr: (string | number | boolean)[]) {
  const strings: string[] = [];
  const numbers: number[] = [];
  const booleans: boolean[] = [];

  for (const element of arr) {
    if (typeof element === "string") {
      strings.push(element);
    } else if (typeof element === "number") {
      numbers.push(element);
    } else {
      booleans.push(element);
    }
  }

  return { strings, numbers, booleans };
}
```

## Практические примеры

### Обработка API-ответов

```typescript
interface SuccessResponse {
  success: true;
  data: any;
}

interface ErrorResponse {
  success: false;
  error: string;
  code: number;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  if (response.success) {
    // Здесь TypeScript знает, что response - это SuccessResponse
    console.log("Данные:", response.data);
  } else {
    // Здесь TypeScript знает, что response - это ErrorResponse
    console.log(`Ошибка ${response.code}: ${response.error}`);
  }
}

// Более сложный пример с кастомной проверкой
function isSuccess(response: ApiResponse): response is SuccessResponse {
  return response.success;
}

function processResponse(response: ApiResponse) {
  if (isSuccess(response)) {
    // Здесь TypeScript знает, что response - это SuccessResponse
    return response.data;
  }
  // Здесь TypeScript знает, что response - это ErrorResponse
  throw new Error(response.error);
}
```

### Работа с объединениями объектных типов

```typescript
interface LoadingState {
  status: "loading";
}

interface ErrorState {
  status: "error";
  error: string;
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

type State<T> = LoadingState | ErrorState | SuccessState<T>;

function renderState<T>(state: State<T>) {
  switch (state.status) {
    case "loading":
      // TypeScript знает, что это LoadingState
      return "Загрузка...";
    case "error":
      // TypeScript знает, что это ErrorState
      return `Ошибка: ${state.error}`;
    case "success":
      // TypeScript знает, что это SuccessState<T>
      return `Данные: ${JSON.stringify(state.data)}`;
  }
}

// Обработка разных типов событий
interface ClickEvent {
  type: "click";
  elementId: string;
  x: number;
  y: number;
}

interface KeyboardEvent {
  type: "keyboard";
  key: string;
  code: string;
}

type AppEvent = ClickEvent | KeyboardEvent;

function handleAppEvent(event: AppEvent) {
  switch (event.type) {
    case "click":
      // TypeScript знает, что это ClickEvent
      console.log(`Клик по элементу ${event.elementId} на позиции (${event.x}, ${event.y})`);
      break;
    case "keyboard":
      // TypeScript знает, что это KeyboardEvent
      console.log(`Нажата клавиша ${event.key} (${event.code})`);
      break;
  }
}
```

## Связь с другими концепциями

- [[type-system/type-guards|Тип-гарды]] - более подробное рассмотрение проверок типов
- [[types/union-intersection-types-comprehensive|Объединения и пересечения типов]] - основа для сужения типов
- [[type-system/type-assertions-narrowing|Утверждения и сужение типов]] - явное указание типов
- [[advanced-types/discriminated-unions-comprehensive|Дискриминированные объединения]] - продвинутый паттерн сужения

## Лучшие практики

1. **Используйте дискриминированные объединения** - добавляйте поле-дискриминатор для лучшего сужения типов
2. **Создавайте пользовательские проверки типов** - для сложных сценариев сужения
3. **Избегайте избыточных проверок** - TypeScript автоматически сужает типы в большинстве случаев
4. **Используйте исчерпывающие проверки** - для обеспечения обработки всех возможных типов

## Заключение

Поток управления и сужение типов - ключевые концепции TypeScript, которые обеспечивают безопасность типов в условиях неопределенности. Понимание этих концепций позволяет писать более надежный и выразительный код, используя мощь системы типов TypeScript для предотвращения ошибок во время выполнения.