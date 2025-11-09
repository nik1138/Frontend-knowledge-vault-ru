# Типы данных в JavaScript: Основы

## Обзор типов данных

JavaScript имеет несколько встроенных типов данных, которые можно разделить на две категории:
1. Примитивные типы данных
2. Сложные (ссылочные) типы данных

## Примитивные типы данных

### Number (Число)

Числовой тип данных в JavaScript используется для представления как целых чисел, так и чисел с плавающей точкой:

```javascript
let integer = 42;
let float = 3.14159;
let negative = -10;
let zero = 0;
let infinity = Infinity;
let notANumber = NaN; // "Not a Number"

console.log(typeof integer); // "number"
console.log(typeof float); // "number"
```

### String (Строка)

Строковый тип данных используется для представления текста:

```javascript
let singleQuote = 'Строка в одинарных кавычках';
let doubleQuote = "Строка в двойных кавычках";
let backtick = `Строка в обратных кавычках`;

// Конкатенация строк
let firstName = "Иван";
let lastName = "Иванов";
let fullName = firstName + " " + lastName; // "Иван Иванов"

// Интерполяция строк (шаблонные литералы)
let greeting = `Привет, ${firstName}!`; // "Привет, Иван!"
```

### Boolean (Булево значение)

Булев тип данных может принимать только два значения: `true` или `false`:

```javascript
let isTrue = true;
let isFalse = false;

// Булевы значения часто используются в условных выражениях
let age = 25;
let isAdult = age >= 18; // true

console.log(typeof isTrue); // "boolean"
```

### Undefined

Значение `undefined` означает, что переменная была объявлена, но ей не было присвоено значение:

```javascript
let emptyVariable;
console.log(emptyVariable); // undefined
console.log(typeof emptyVariable); // "undefined"

// Функции без явного возврата возвращают undefined
function doNothing() {
  // Ничего не возвращает
}

console.log(doNothing()); // undefined
```

### Null

Значение `null` представляет собой "ничего" или "пустое значение". Это специальное значение, которое означает "ничего":

```javascript
let emptyValue = null;
console.log(emptyValue); // null
console.log(typeof null); // "object" (это ошибка в JavaScript, но она осталась для совместимости)
```

### Symbol (Символ)

Symbol - это уникальный и неизменяемый тип данных, который может использоваться как ключ для свойств объекта:

```javascript
let sym1 = Symbol("описание");
let sym2 = Symbol("описание");

console.log(sym1 === sym2); // false (каждый символ уникален)

// Символы часто используются для создания уникальных ключей
let id = Symbol("id");
let user = {
  name: "Иван",
  [id]: 123 // Свойство с ключом-символом
};
```

### BigInt

BigInt - это числовой тип данных, который может представлять целые числа произвольной точности:

```javascript
let bigNumber = 1234567890123456789012345678901234567890n;
let anotherBigNumber = BigInt("1234567890123456789012345678901234567890");

console.log(typeof bigNumber); // "bigint"
```

## Сложные (ссылочные) типы данных

### Object (Объект)

Объекты используются для хранения коллекций данных и более сложных сущностей:

```javascript
let person = {
  name: "Иван",
  age: 30,
  isStudent: false,
  greet: function() {
    return `Привет, меня зовут ${this.name}`;
  }
};

console.log(person.name); // "Иван"
console.log(person["age"]); // 30
console.log(person.greet()); // "Привет, меня зовут Иван"
console.log(typeof person); // "object"
```

### Array (Массив)

Массивы - это специальный тип объектов, используемый для хранения упорядоченных коллекций значений:

```javascript
let fruits = ["яблоко", "банан", "апельсин"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "строка", true, null, { name: "Иван" }];

console.log(fruits[0]); // "яблоко"
console.log(numbers.length); // 5
console.log(typeof fruits); // "object" (массивы - это объекты)
console.log(Array.isArray(fruits)); // true (проверка, является ли значение массивом)
```

### Function (Функция)

Функции - это объекты, которые можно вызывать:

```javascript
// Объявление функции
function greet(name) {
  return `Привет, ${name}!`;
}

// Функциональное выражение
let sayHello = function(name) {
  return `Привет, ${name}!`;
};

// Стрелочная функция
let sayHi = (name) => `Привет, ${name}!`;

console.log(typeof greet); // "function"
console.log(greet("Иван")); // "Привет, Иван!"
```

## Проверка типов данных

### Оператор typeof

Оператор `typeof` возвращает строку, указывающую тип операнда:

```javascript
console.log(typeof 42); // "number"
console.log(typeof "строка"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object" (особенность JavaScript)
console.log(typeof Symbol("id")); // "symbol"
console.log(typeof BigInt(123)); // "bigint"
console.log(typeof {}); // "object"
console.log(typeof []); // "object"
console.log(typeof function(){}); // "function"
```

### Другие способы проверки типов

```javascript
// Проверка массива
let arr = [1, 2, 3];
console.log(Array.isArray(arr)); // true

// Проверка NaN
let notANumber = NaN;
console.log(Number.isNaN(notANumber)); // true

// Проверка конечности числа
let finite = 42;
let infinite = Infinity;
console.log(Number.isFinite(finite)); // true
console.log(Number.isFinite(infinite)); // false
```

## Преобразование типов

### Явное преобразование

```javascript
// Преобразование в строку
let num = 123;
let str = String(num); // "123"

// Преобразование в число
let strNum = "123";
let numFromString = Number(strNum); // 123
let parsedInt = parseInt("123.45"); // 123
let parsedFloat = parseFloat("123.45"); // 123.45

// Преобразование в булево значение
let truthy = Boolean("строка"); // true
let falsy = Boolean(""); // false
```

### Неявное преобразование

```javascript
// Преобразование в строку при конкатенации
let result = "Число: " + 42; // "Число: 42"

// Преобразование в число при арифметических операциях
let sum = "5" - 3; // 2 (строка преобразуется в число)
let product = "5" * "3"; // 15

// Логические преобразования
if ("строка") {
  console.log("Непустая строка - true"); // Выполнится
}

if (0) {
  console.log("0 - false"); // Не выполнится
}
```

## Особенности типов данных в JavaScript

### Динамическая типизация

JavaScript использует динамическую типизацию, что означает, что переменная может хранить значения разных типов:

```javascript
let dynamicVar = 42; // number
dynamicVar = "строка"; // string
dynamicVar = true; // boolean
dynamicVar = { x: 1 }; // object
```

### Сравнение значений

```javascript
// Строгое сравнение (===)
console.log(5 === "5"); // false (разные типы)
console.log(5 === 5); // true

// Нестрогое сравнение (==)
console.log(5 == "5"); // true (происходит преобразование типов)
console.log(null == undefined); // true
console.log(0 == false); // true
```

## Практические примеры

### Работа с различными типами данных

```javascript
// Создание объекта с разными типами данных
const userProfile = {
  id: 123, // number
  username: "ivan_ivanov", // string
  isActive: true, // boolean
  lastLogin: null, // null
  preferences: { // object
    theme: "dark",
    language: "ru"
  },
  tags: ["frontend", "javascript", "developer"], // array
  createdAt: new Date(), // object (Date)
  getFullInfo: function() { // function
    return `${this.username} (${this.id})`;
  }
};

console.log(userProfile.getFullInfo()); // "ivan_ivanov (123)"
```

### Проверка и обработка типов

```javascript
function processData(data) {
  if (typeof data === "string") {
    return data.toUpperCase();
  } else if (typeof data === "number") {
    return data * 2;
  } else if (Array.isArray(data)) {
    return data.length;
  } else if (data === null || data === undefined) {
    return "Нет данных";
  } else {
    return "Неизвестный тип";
  }
}

console.log(processData("привет")); // "ПРИВЕТ"
console.log(processData(5)); // 10
console.log(processData([1, 2, 3])); // 3
console.log(processData(null)); // "Нет данных"
```

## Заключение

Понимание типов данных в JavaScript критически важно для написания надежного кода. Знание различий между примитивными и ссылочными типами, умение проверять типы данных и правильно преобразовывать их поможет избежать многих распространенных ошибок.

#javascript #programming #data-types #fundamentals #web-development