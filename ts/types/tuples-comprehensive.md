# Кортежи в TypeScript

Кортежи (tuples) - это тип, представляющий массив с фиксированным количеством элементов, где каждый элемент может иметь определенный тип. В отличие от массивов, где все элементы обычно одного типа, кортежи позволяют указать конкретный тип для каждой позиции в массиве.

## Основной синтаксис

Кортежи объявляются с помощью квадратных скобок с перечислением типов элементов:

```typescript
// Простой кортеж из двух элементов
type NameAge = [string, number];
const person: NameAge = ["Alice", 30];  // OK
// const invalid: NameAge = ["Bob"];  // Ошибка: ожидается 2 элемента

// Доступ к элементам по индексу
const name: string = person[0];
const age: number = person[1];
```

## Простые примеры

### 1. Базовые кортежи

```typescript
// Географические координаты
type Coordinates = [number, number];  // [широта, долгота]
const moscow: Coordinates = [55.7558, 37.6173];

// HTTP ответ
type HttpResponse = [number, string];  // [status, message]
const response: HttpResponse = [200, "OK"];

// Цвет в формате RGB
type RGB = [number, number, number];  // [red, green, blue]
const red: RGB = [255, 0, 0];
```

### 2. Кортежи с разными типами

```typescript
type ComplexTuple = [string, number, boolean, Date];
const record: ComplexTuple = ["John", 25, true, new Date()];

const name: string = record[0];      // string
const age: number = record[1];       // number
const active: boolean = record[2];   // boolean
const date: Date = record[3];        // Date
```

## Расширенные возможности

### 1. Опциональные элементы в кортежах

```typescript
// Кортеж с опциональным последним элементом
type OptionalTuple = [string, number, boolean?];
const tuple1: OptionalTuple = ["hello", 42];           // OK
const tuple2: OptionalTuple = ["hello", 42, true];     // OK
// const tuple3: OptionalTuple = ["hello"];           // Ошибка: ожидается минимум 2 элемента

// Опциональные элементы могут быть только в конце
type WithOptional = [string, number?, boolean?];  // OK
// type Invalid = [string, number?, boolean];  // Ошибка: опциональные элементы должны быть в конце
```

### 2. Кортежи с остаточными элементами (rest elements)

```typescript
// Кортеж с фиксированными первыми элементами и произвольным количеством остальных
type FlexibleTuple = [string, ...number[]];
const scores1: FlexibleTuple = ["Alice", 95, 87, 92];     // OK
const scores2: FlexibleTuple = ["Bob", 78];               // OK
// const scores3: FlexibleTuple = [95, "Alice"];         // Ошибка: первый элемент должен быть string

// Кортеж с несколькими фиксированными элементами и остатком
type MixedTuple = [string, string, ...number[]];
const data: MixedTuple = ["John", "Doe", 25, 30, 35];  // [firstName, lastName, ages...]
```

### 3. Именованные кортежи

```typescript
// Именованные элементы кортежа (для лучшей читаемости)
type NamedTuple = [name: string, age: number, active: boolean];
const user: NamedTuple = ["Alice", 30, true];

// Имена не влияют на тип, но улучшают автодополнение и документацию
```

## Практические применения

### 1. Возвращаемые значения функций

```typescript
// Функция, возвращающая результат и флаг успеха
function divide(a: number, b: number): [result: number, success: boolean] {
  if (b === 0) {
    return [0, false];
  }
  return [a / b, true];
}

const [result, success] = divide(10, 2);
if (success) {
  console.log(`Результат: ${result}`);
} else {
  console.log("Деление на ноль!");
}

// Функция, возвращающая значение и возможную ошибку
function tryParseInt(str: string): [value: number, error?: string] {
  const num = parseInt(str, 10);
  if (isNaN(num)) {
    return [0, "Неверный формат числа"];
  }
  return [num];
}

const [value, error] = tryParseInt("42");
if (error) {
  console.error(error);
} else {
  console.log(`Парсинг успешен: ${value}`);
}
```

### 2. Работа с параметрами функций

```typescript
// Использование кортежей для распределения параметров
type Point = [x: number, y: number];

function calculateDistance([x1, y1]: Point, [x2, y2]: Point): number {
  return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
}

const pointA: Point = [0, 0];
const pointB: Point = [3, 4];
const distance = calculateDistance(pointA, pointB);  // 5
```

### 3. Состояния и переходы

```typescript
// Определение переходов в конечном автомате
type StateTransition = [from: string, to: string, event: string];
type StateMachine = StateTransition[];

const loginFlow: StateMachine = [
  ["unauthenticated", "loading", "login_request"],
  ["loading", "authenticated", "login_success"],
  ["loading", "unauthenticated", "login_failure"],
  ["authenticated", "unauthenticated", "logout"]
];
```

## Продвинутые паттерны

### 1. Кортежи с условными типами

```typescript
// Утилита для получения типа элемента кортежа по индексу
type ElementAt<T extends readonly unknown[], I extends number> = T[I];

type MyTuple = [string, number, boolean];
type FirstType = ElementAt<MyTuple, 0>;   // string
type SecondType = ElementAt<MyTuple, 1>;  // number
type ThirdType = ElementAt<MyTuple, 2>;   // boolean

// Утилита для получения длины кортежа
type Length<T extends readonly unknown[]> = T["length"];

type TupleLength = Length<MyTuple>;  // 3
```

### 2. Кортежи в обобщениях

```typescript
// Функция, меняющая местами элементы кортежа
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}

const original: [string, number] = ["hello", 42];
const swapped = swap(original);  // [number, string]
console.log(swapped); // [42, "hello"]
```

### 3. Кортежи и деструктуризация

```typescript
// Использование кортежей с деструктуризацией
type UserTuple = [id: number, name: string, email: string];

function processUser([id, name, email]: UserTuple): void {
  console.log(`Обработка пользователя: ${name} (ID: ${id})`);
  console.log(`Email: ${email}`);
}

const user: UserTuple = [1, "Alice", "alice@example.com"];
processUser(user);
```

## Практические примеры из реальной разработки

### 1. Работа с API и парами данных

```typescript
// Пара ключ-значение для параметров запроса
type QueryParam = [key: string, value: string];
type QueryParams = QueryParam[];

function buildQueryString(params: QueryParams): string {
  return params.map(([key, value]) => `${encodeURIComponent(key)}=${encodeURIComponent(value)}`).join('&');
}

const params: QueryParams = [
  ["name", "John"],
  ["age", "30"],
  ["city", "New York"]
];

const queryString = buildQueryString(params);  // "name=John&age=30&city=New%20York"
```

### 2. Создание типобезопасных маппингов

```typescript
// Типобезопасный маппинг строк к типам
type TypeMapping = [
  ["string", string],
  ["number", number],
  ["boolean", boolean],
  ["date", Date]
][number];  // Это создает объединение: ["string", string] | ["number", number] | ...

function convertValue(type: TypeMapping[0], value: string): TypeMapping[1] {
  switch (type) {
    case "string":
      return value as TypeMapping[1];
    case "number":
      return Number(value) as TypeMapping[1];
    case "boolean":
      return (value === "true") as TypeMapping[1];
    case "date":
      return new Date(value) as TypeMapping[1];
    default:
      throw new Error(`Неизвестный тип: ${type}`);
  }
}
```

### 3. Работа с конфигурацией и настройками

```typescript
// Кортеж для определения настроек с типами
type SettingDefinition = [name: string, type: "string" | "number" | "boolean", defaultValue: unknown];

const settingsConfig: SettingDefinition[] = [
  ["apiUrl", "string", "https://api.example.com"],
  ["timeout", "number", 5000],
  ["debug", "boolean", false],
  ["retries", "number", 3]
];

type SettingValue<T extends SettingDefinition> = 
  T[1] extends "string" ? string :
  T[1] extends "number" ? number :
  T[1] extends "boolean" ? boolean :
  unknown;

// Тип для объекта настроек
type SettingsObject = {
  [K in SettingDefinition as K[0]]: SettingValue<Extract<SettingDefinition, [K, any, any]>>
};
```

### 4. Создание типобезопасных событий

```typescript
// Определение событий с их типами данных
type EventDefinition = [
  name: string,
  payload: Record<string, unknown>
];

type UserLoginEvent = ["user_login", { userId: number; timestamp: Date }];
type UserLogoutEvent = ["user_logout", { userId: number; duration: number }];
type ErrorEvent = ["error", { message: string; code: number }];

type AppEvent = UserLoginEvent | UserLogoutEvent | ErrorEvent;

function emitEvent<E extends AppEvent[0]>(
  eventName: E,
  payload: Extract<AppEvent, [E, any]>[1]
): void {
  // Отправка события
  console.log(`Событие: ${eventName}`, payload);
}

// Использование
emitEvent("user_login", { userId: 123, timestamp: new Date() });  // OK
// emitEvent("user_login", { wrongField: "value" });  // Ошибка: неправильная структура payload
```

## Ограничения и особенности

### 1. Ограничения на изменение

```typescript
const tuple: [string, number] = ["hello", 42];

// Можно изменить существующие элементы
tuple[0] = "world";  // OK
tuple[1] = 100;      // OK

// Но нельзя добавлять или удалять элементы
// tuple[2] = true;   // Ошибка: индекс 2 не существует
// tuple.length = 5;  // Ошибка: свойство length только для чтения у кортежей
```

### 2. Совместимость с массивами

```typescript
const tuple: [string, number] = ["hello", 42];
const array: (string | number)[] = tuple;  // OK: кортеж совместим с массивом объединения

// Но не наоборот
const arr: (string | number)[] = ["hello", 42, true];  // (string | number | boolean)[]
// const invalidTuple: [string, number] = arr;  // Ошибка: arr может содержать больше элементов или другие типы
```

### 3. Длина кортежа в системе типов

```typescript
// TypeScript отслеживает длину кортежа как числовой литеральный тип
type TwoElements = [string, number];
type Length = TwoElements["length"];  // 2 (не просто number)

// Это позволяет создавать условные типы на основе длины
type IsPair<T extends readonly unknown[]> = T["length"] extends 2 ? true : false;
type Result = IsPair<[string, number]>;  // true
type Result2 = IsPair<[string]>;         // false
```

## Лучшие практики

1. **Используйте именованные элементы для улучшения читаемости** - особенно для кортежей с несколькими элементами
2. **Документируйте сложные кортежи** - особенно когда они используются в API
3. **Предпочитайте объекты для сложных структур данных** - кортежи лучше использовать для простых пар/триплетов
4. **Используйте кортежи для возвращаемых значений функций** - когда нужно вернуть несколько значений разных типов
5. **Будьте осторожны с изменением кортежей** - типы элементов фиксированы, но значения можно изменять
6. **Используйте кортежи с остаточными элементами для гибких структур** - когда нужна фиксированная часть и переменная часть

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - часто используются для работы с кортежами
- [[ts/generics/TS Generics|Обобщения]] - кортежи часто параметризованы
- [[ts/types/type-aliases|Псевдонимы типов]] - кортежи часто используют псевдонимы
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - могут применяться к кортежам
- [[js/arrays]] - кортежи как типизированные массивы