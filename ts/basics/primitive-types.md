# Примитивные типы в TypeScript

## Введение

Примитивные типы - это основные типы данных, встроенные в JavaScript и поддерживаемые TypeScript. Они являются основой для всех других типов и определяют базовые значения, с которыми работает язык.

## Основные примитивные типы

### string

Тип `string` представляет собой последовательность символов, используемых для представления текстовых данных.

```typescript
// Объявление строковых переменных
let firstName: string = "Иван";
let lastName: string = 'Петров';
let fullName: string = `Иван Петров`; // шаблонная строка

// Примеры использования
const greeting: string = `Привет, ${firstName} ${lastName}!`;
const message: string = "Добро пожаловать в TypeScript";
```

### number

Тип `number` представляет как целые, так и дробные числа. В TypeScript все числа являются 64-битными значениями с плавающей точкой.

```typescript
// Объявление числовых переменных
let age: number = 30;
let price: number = 99.99;
let count: number = 0b1010; // двоичный литерал
let hex: number = 0xf00d;   // шестнадцатеричный литерал
let binary: number = 0b1111; // двоичный литерал
let octal: number = 0o744;   // восьмеричный литерал
```

### boolean

Тип `boolean` представляет логические значения, которые могут быть либо `true`, либо `false`.

```typescript
// Объявление булевых переменных
let isCompleted: boolean = true;
let isLoading: boolean = false;
let hasError: boolean = false;

// Примеры использования
const isAuthenticated: boolean = checkUserAuth();
const isValid: boolean = validateInput(inputValue);
```

### null и undefined

Типы `null` и `undefined` имеют собственные типы, каждый из которых имеет только одно значение - `null` и `undefined` соответственно.

```typescript
// Объявление переменных с null и undefined
let emptyValue: null = null;
let notAssigned: undefined = undefined;

// Эти типы могут быть присвоены другим типам (если не включен strict mode)
let nullableString: string | null = null;
let possiblyUndefined: number | undefined = undefined;
```

## Дополнительные примитивные типы

### symbol

Тип `symbol` представляет уникальные и неизменяемые значения, которые могут использоваться в качестве идентификаторов для свойств объектов.

```typescript
// Создание символа
const sym1: symbol = Symbol("description");
const sym2: symbol = Symbol("another description");

// Символы уникальны
console.log(sym1 === sym2); // false

// Использование символа как ключа свойства
const id = Symbol("id");
const obj = {
  [id]: 123,
  name: "Object"
};
```

### bigint

Тип `bigint` представляет целые числа произвольной длины. Доступен в современных версиях JavaScript.

```typescript
// Создание bigint
const bigNumber: bigint = 1234567890123456789012345678901234567890n;
const anotherBig: bigint = BigInt(123456);

// Использование в арифметических операциях
const result: bigint = bigNumber + anotherBig;
```

## Практические примеры

### Использование примитивных типов в функциях

```typescript
// Функция, возвращающая строку
function getUserInfo(name: string, age: number, isActive: boolean): string {
  return `${name} is ${age} years old and is ${isActive ? 'active' : 'inactive'}`;
}

// Пример вызова
const userInfo: string = getUserInfo("Алексей", 25, true);
console.log(userInfo); // "Алексей is 25 years old and is active"
```

### Комбинирование примитивных типов

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
  lastLogin: null | string; // может быть null или строкой
}

const user: User = {
  id: 1,
  name: "Мария",
  email: "maria@example.com",
  isActive: true,
  lastLogin: null
};
```

## Связь с другими концепциями

- [[basics/type-annotations|Аннотации типов]] - как указываются типы переменных
- [[type-system/type-inference|Вывод типов]] - как TypeScript автоматически определяет типы
- [[types/type-aliases-comprehensive|Псевдонимы типов]] - для создания более сложных типов на основе примитивов

## Заключение

Примитивные типы являются основой системы типов TypeScript и JavaScript. Понимание этих типов критически важно для правильной типизации приложений и обеспечения безопасности типов в коде.