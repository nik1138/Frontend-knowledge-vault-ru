---
aliases: ["Типы данных JavaScript", "Проверка типов JavaScript", "JavaScript типизация"]
tags: [javascript, types, type-checking, programming]
---

# Типы данных и проверка типов в JavaScript

JavaScript - это язык с динамической типизацией, где типы значений определяются во время выполнения. В этой статье рассмотрим основные концепции типизации в JavaScript, от примитивных типов до современных подходов с TypeScript.

## Примитивные типы против объектов

JavaScript имеет два основных типа значений: **примитивные типы** и **объекты**.

### Примитивные типы

Примитивные типы являются неизменяемыми (immutable) значениями:

- `undefined` - значение по умолчанию для неинициализированных переменных
- `null` - намеренно отсутствующее значение
- `boolean` - логическое значение (`true` или `false`)
- `number` - числовое значение (включая `NaN` и `Infinity`)
- `string` - строковое значение
- `symbol` - уникальный и неизменяемый идентификатор
- `bigint` - целое число произвольной длины

```javascript
let age = 25; // number
let name = "Иван"; // string
let isActive = true; // boolean
let id = Symbol('id'); // symbol
let bigNumber = 123456789012345678901234567890n; // bigint
```

### Объекты

Объекты - это коллекции пар ключ-значение и включают:

- Объекты (`Object`)
- Массивы (`Array`)
- Функции (`Function`)
- Даты (`Date`)
- Регулярные выражения (`RegExp`)
- И другие специализированные объекты

```javascript
let person = { name: "Анна", age: 30 }; // Object
let numbers = [1, 2, 3]; // Array
let greet = function() { return "Привет"; }; // Function
```

## Оператор typeof

Оператор `typeof` позволяет определить тип значения:

```javascript
console.log(typeof 42); // "number"
console.log(typeof "hello"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof {}); // "object"
console.log(typeof []); // "object" (важное исключение!)
console.log(typeof null); // "object" (историческая ошибка)
console.log(typeof function() {}); // "function"
```

## Методы проверки типов

### Проверка с помощью instanceof

Оператор `instanceof` проверяет, был ли объект создан с помощью определенного конструктора:

```javascript
let arr = [1, 2, 3];
console.log(arr instanceof Array); // true
console.log(arr instanceof Object); // true (массивы также являются объектами)
```

### Проверка с помощью Object.prototype.toString

Метод `Object.prototype.toString.call()` предоставляет надежный способ определения типа:

```javascript
console.log(Object.prototype.toString.call([])); // "[object Array]"
console.log(Object.prototype.toString.call({})); // "[object Object]"
console.log(Object.prototype.toString.call(null)); // "[object Null]"
console.log(Object.prototype.toString.call(undefined)); // "[object Undefined]"
```

### Проверка на конкретный тип

```javascript
function isArray(value) {
    return Array.isArray ? Array.isArray(value) : Object.prototype.toString.call(value) === '[object Array]';
}

function isPlainObject(value) {
    return value !== null && 
           typeof value === 'object' && 
           value.constructor === Object;
}
```

## Преобразование типов

JavaScript автоматически преобразует типы в определенных ситуациях, что называется **приведением типов**.

### Явное преобразование

```javascript
let str = "123";
let num = Number(str); // 123
let bool = Boolean(str); // true
let strNum = String(num); // "123"
```

### Неявное преобразование (приведение типов)

```javascript
console.log(5 + "5"); // "55" (число преобразуется в строку)
console.log(5 * "5"); // 25 (строка преобразуется в число)
console.log(true + 1); // 2 (true преобразуется в 1)
```

## Truthy и Falsy значения

В JavaScript каждое значение может быть оценено как `true` или `false` в логическом контексте.

### Falsy значения:

- `false`
- `0`
- `-0`
- `0n` (BigInt)
- `""` (пустая строка)
- `null`
- `undefined`
- `NaN`

```javascript
if ("") console.log("Это не выполнится"); // пустая строка - falsy
if (0) console.log("Это не выполнится"); // 0 - falsy
if ("0") console.log("Это выполнится"); // ненулевая строка - truthy
```

## Сравнения

JavaScript имеет два типа сравнений: строгое и нестрогое.

### Нестрогое сравнение (==)

Оператор `==` выполняет приведение типов перед сравнением:

```javascript
console.log(5 == "5"); // true
console.log(true == 1); // true
console.log(false == 0); // true
console.log(null == undefined); // true
```

### Строгое сравнение (===)

Оператор `===` не выполняет приведение типов:

```javascript
console.log(5 === "5"); // false
console.log(true === 1); // false
console.log(null === undefined); // false
```

## Duck Typing

JavaScript использует концепцию "утиной типизации" - "Если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка". Объект считается подходящим, если он имеет нужные свойства и методы:

```javascript
// Проверка наличия необходимых методов
function hasQuack(obj) {
    return typeof obj.quack === 'function';
}

let duck = {
    quack: function() { return "Кря-кря"; }
};

console.log(hasQuack(duck)); // true
```

## Runtime vs Static Typing

JavaScript - язык с динамической (runtime) типизацией, что означает, что типы проверяются во время выполнения программы, в отличие от статической типизации, где типы проверяются во время компиляции.

### Runtime Typing

```javascript
let value = 42;
console.log(typeof value); // "number"
value = "строка";
console.log(typeof value); // "string"
```

## JSDoc для типов

JSDoc позволяет документировать типы в JavaScript коде:

```javascript
/**
 * @param {number} a - Первое число
 * @param {number} b - Второе число
 * @returns {number} Сумма двух чисел
 */
function add(a, b) {
    return a + b;
}

/**
 * @typedef {Object} User
 * @property {string} name - Имя пользователя
 * @property {number} age - Возраст пользователя
 */

/**
 * @param {User} user - Объект пользователя
 */
function greetUser(user) {
    console.log(`Привет, ${user.name}!`);
}
```

## Основы TypeScript

TypeScript добавляет статическую типизацию к JavaScript:

```typescript
let name: string = "Алексей";
let age: number = 30;
let isActive: boolean = true;

function greet(person: string): string {
    return `Привет, ${person}!`;
}

interface User {
    name: string;
    age: number;
}

function processUser(user: User) {
    console.log(user.name);
}
```

## Типы Symbol и BigInt

### Symbol

`Symbol` - это примитивный тип, представляющий уникальные идентификаторы:

```javascript
let id1 = Symbol('id');
let id2 = Symbol('id');
console.log(id1 === id2); // false (символы всегда уникальны)

// Глобальные символы
let globalId = Symbol.for('globalId');
let sameGlobalId = Symbol.for('globalId');
console.log(globalId === sameGlobalId); // true
```

### BigInt

`BigInt` позволяет работать с целыми числами произвольной длины:

```javascript
let bigNumber = 123456789012345678901234567890n;
let anotherBig = BigInt("123456789012345678901234567890");

console.log(bigNumber + 1n); // Ошибка - нельзя смешивать BigInt и Number
console.log(bigNumber + 2n); // 123456789012345678901234567892n
```

## Type Guards

Type guards позволяют проверять типы в runtime:

```javascript
function isString(value) {
    return typeof value === 'string';
}

function processValue(value) {
    if (isString(value)) {
        // Здесь TypeScript знает, что value - строка
        return value.toUpperCase();
    }
    return value;
}

// Проверка массива
function isArray(value) {
    return Array.isArray(value);
}

// Проверка на null/undefined
function isValid(value) {
    return value != null; // проверяет и null, и undefined
}
```

## Связь с другими файлами

- [[js/functions]] - функции и их типы
- [[js/objects]] - объекты и прототипы
- [[js/arrays]] - массивы и их методы
- [[js/es6-features]] - современные возможности ES6+
- [[ts/basics]] - основы TypeScript

## Заключение

Понимание систем типизации в JavaScript критически важно для написания надежного кода. От примитивных типов до современных систем статической типизации, каждая концепция играет важную роль в разработке JavaScript приложений.

## Ключевые выводы

- JavaScript использует динамическую типизацию
- Существует 7 примитивных типов и объекты
- Оператор `typeof` помогает определять типы
- Следует использовать строгое сравнение `===` для избежания неожиданного приведения типов
- JSDoc и TypeScript помогают документировать и проверять типы
- Duck typing позволяет гибко работать с объектами