# Система типов TypeScript

Система типов TypeScript - это одна из самых мощных и гибких систем типов среди всех языков программирования. Она обеспечивает статическую типизацию с возможностью вывода типов, структурную типизацию и множество продвинутых возможностей.

## Содержание раздела

1. [[ts/type-system/type-inference|Вывод типов]] - Как TypeScript автоматически определяет типы
2. [[ts/type-system/type-compatibility|Совместимость типов]] - Когда один тип может быть использован вместо другого
3. [[ts/type-system/keyof-operator|Оператор keyof]] - Получение ключей объекта на уровне типов
4. [[ts/type-system/indexed-access-types|Индексные типы]] - Получение типов значений по ключам
5. [[ts/type-system/type-guards|Защитники типов]] - Уточнение типов в runtime
6. [[ts/type-system/structural-typing|Структурная типизация]] - Основа системы типов TypeScript
7. [[ts/type-system/duck-typing|Утиная типизация]] - Практическое применение структурной типизации

## Введение

Система типов TypeScript предоставляет разработчикам мощные инструменты для создания надежного и поддерживаемого кода. В отличие от многих других языков, TypeScript использует структурную (или утиную) типизацию, где совместимость типов определяется их формой, а не именами.

## Основные концепции

### 1. Вывод типов

TypeScript может автоматически выводить типы переменных, параметров функций и возвращаемых значений:

```typescript
let x = 3;           // TypeScript выводит тип number
let y = "hello";     // TypeScript выводит тип string

function add(a: number, b: number) {
  return a + b;      // TypeScript выводит возвращаемый тип number
}
```

### 2. Совместимость типов

Совместимость типов в TypeScript основана на структурной типизации:

```typescript
interface Point {
  x: number;
  y: number;
}

class Point2D {
  x: number;
  y: number;
  
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

// Совместимость: структура совпадает, несмотря на разные имена
let p: Point = new Point2D(1, 2);  // OK
```

### 3. Оператор keyof

Оператор `keyof` позволяет получать объединение строковых литералов всех публичных свойств типа:

```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonKeys = keyof Person; // "name" | "age" | "email"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## Практическое применение

### 1. Создание типобезопасных библиотек

Система типов TypeScript позволяет создавать библиотеки с высокой степенью типобезопасности:

```typescript
// Универсальная функция получения значения по ключу
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

### 2. Работа с API

Система типов помогает создавать типобезопасные обертки для API:

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Защитник типа для проверки успешного ответа
function isSuccessResponse<T>(response: ApiResponse<T>): response is ApiResponse<T> & { status: 200 } {
  return response.status === 200;
}
```

### 3. Событийная система

С помощью системы типов можно создать строго типизированную событийную систему:

```typescript
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
}

class TypedEventEmitter {
  private listeners: {
    [K in keyof EventMap]?: ((data: EventMap[K]) => void)[];
  } = {};
  
  on<K extends keyof EventMap>(event: K, listener: (data: EventMap[K]) => void): void {
    // ...
  }
  
  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void {
    // ...
  }
}
```

## Расширенные возможности

### 1. Условные типы с системой типов

```typescript
// Условные типы позволяют создавать сложные преобразования типов
type NonNullable<T> = T extends null | undefined ? never : T;

type Str = NonNullable<string | null>; // string
```

### 2. Сопоставляющие типы

```typescript
// Сопоставляющие типы позволяют преобразовывать существующие типы
type Partial<T> = {
  [P in keyof T]?: T[P];
};
```

### 3. Вывод типов в условных типах

```typescript
// Использование infer для вывода типов
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type Str = ReturnType<() => string>; // string
```

## Связи с другими концепциями

Система типов тесно связана с другими аспектами TypeScript:

- [[ts/generics/TS Generics|Дженерики]] - Параметризация типов
- [[ts/advanced-types/conditional-types|Условные типы]] - Типы, зависящие от условий
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - Преобразование существующих типов
- [[ts/utility-types/index|Утилиты типов]] - Встроенные и пользовательские типы-помощники
- [[ts/advanced-types/template-literal-types|Типы-шаблонные литералы]] - Строковые типы на основе шаблонов

## Рекомендации по изучению

1. Начните с основ вывода типов и совместимости
2. Освойте работу с оператором keyof и индексными типами
3. Практикуйтесь в создании защитников типов
4. Изучите структурную типизацию и ее практическое применение
5. Освойте продвинутые возможности с условными и сопоставляющими типами

## Следующие шаги

- [[ts/utility-types/index|Утилиты типов]] - Встроенные и пользовательские типы-помощники
- [[ts/advanced-types/index|Расширенные типы]] - Продвинутые возможности системы типов
- [[ts/generics/TS Generics|Дженерики]] - Параметрический полиморфизм в TypeScript