# Базовые типы в TypeScript

## Основные примитивные типы

TypeScript предоставляет богатую систему типов, начиная с базовых примитивных типов:

### String
Тип `string` представляет строковые значения:
```ts
let name: string = "Alice";
```

### Number
Тип `number` представляет как целые, так и дробные числа:
```ts
let age: number = 30;
let price: number = 99.99;
```

### Boolean
Тип `boolean` может принимать значения `true` или `false`:
```ts
let isActive: boolean = true;
```

### Null и Undefined
Типы `null` и `undefined`:
```ts
let u: undefined = undefined;
let n: null = null;
```

### Symbol
Тип `symbol` для уникальных идентификаторов:
```ts
const sym1 = Symbol("key");
const sym2 = Symbol("key");
console.log(sym1 === sym2); // false
```

### BigInt
Тип `bigint` для работы с очень большими целыми числами:
```ts
const bigNumber: bigint = 123456789012345678901234567890n;
```

## Объявление переменных с типами

В TypeScript можно явно указывать типы переменных:

```ts
// Явное указание типа
let userName: string = "John";

// Вывод типа (TypeScript сам определяет тип)
let userAge = 25; // тип number будет выведен автоматически

// Массивы
let numbers: number[] = [1, 2, 3];
let fruits: string[] = ["apple", "banana", "orange"];

// Кортежи
let user: [string, number] = ["John", 25];
```

## Строковые литералы

TypeScript позволяет использовать строковые литералы как типы:

```ts
type Direction = "up" | "down" | "left" | "right";
let move: Direction = "up"; // OK
// let invalidMove: Direction = "diagonal"; // Ошибка компиляции
```

## Практические примеры

```ts
// Определение типа для пользователя
type User = {
  name: string;
  age: number;
  isActive: boolean;
};

const user: User = {
  name: "Alice",
  age: 30,
  isActive: true
};

// Функция с типизацией параметров и возвращаемого значения
function greetUser(user: User): string {
  return `Hello, ${user.name}! You are ${user.age} years old.`;
}

console.log(greetUser(user)); // "Hello, Alice! You are 30 years old."
```

## Ключевые концепции

- **Статическая типизация**: типы проверяются на этапе компиляции
- **Вывод типов**: TypeScript может автоматически определять типы
- **Совместимость типов**: структурная типизация (duck typing)
- **Необязательные типы**: использование `any`, `unknown`, `never` для специальных случаев

## Дальнейшее изучение

- [[../primitive-types|Примитивные типы]]
- [[../literal-types|Литеральные типы]]
- [[../../basics/type-annotations|Аннотации типов]]
- [[../../type-system/type-inference|Вывод типов]]