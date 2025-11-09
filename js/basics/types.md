# Типы данных в JavaScript: глубокое понимание

## Введение

В JavaScript существует 8 основных типов данных (7 примитивных + объекты), каждый из которых имеет свои особенности и нюансы. Понимание типов данных критически важно для написания надежного кода и избежания неожиданного поведения.

## Примитивные типы данных

### Number
JavaScript использует один тип данных для всех чисел: 64-битный формат с плавающей запятой (IEEE 754). Это включает целые числа, дроби и специальные значения:

```javascript
// Обычные числа
let integer = 42;
let float = 3.14159;
let negative = -10;

// Специальные числовые значения
let infinity = Infinity;
let negInfinity = -Infinity;
let notANumber = NaN; // "Not a Number" - результат недопустимой операции

console.log(1 / 0); // Infinity
console.log("не число" / 2); // NaN
console.log(NaN === NaN); // false - NaN уникален, не равен самому себе
```

**Важные особенности:**
- Точность вычислений может быть потеряна при работе с дробями:
```javascript
console.log(0.1 + 0.2); // 0.30000000000000004
console.log(0.1 + 0.2 == 0.3); // false
```
- Решение: использовать округление при сравнении:
```javascript
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON); // true
```

### BigInt
Появился в ES2020 для работы с числами произвольной длины:

```javascript
const bigNumber = 1234567890123456789012345678901234567890n;
const anotherBig = BigInt("1234567890123456789012345678901234567890");

// Операции с BigInt
console.log(bigNumber + 1n); // 1234567890123456789012345678901234567891n
```

### String
Строки в JavaScript неизменяемы. Существует несколько способов создания строк:

```javascript
// Одинарные кавычки
let single = 'Привет';

// Двойные кавычки
let double = "Привет";

// Обратные кавычки (шаблонные строки)
let template = `Привет, ${name}!`;

// Многострочные строки
let multiline = `
    Это
    многострочная
    строка
`;

// Строки как объекты
let str = "Привет";
console.log(str.length); // 6
console.log(str[0]); // 'П'
console.log(str.toUpperCase()); // 'ПРИВЕТ'
```

**Важно:** строки индексируются с 0, и вы не можете изменить символ по индексу:
```javascript
str[0] = 'п'; // Ошибка в строгом режиме, игнорируется в нестрогом
console.log(str); // Все равно "Привет"
```

### Boolean
Два значения: `true` и `false`:

```javascript
let isGreater = 5 > 3; // true
let isLess = 5 < 3; // false

// Ложные значения (falsy values)
// false, 0, "", null, undefined, NaN
console.log(Boolean(0)); // false
console.log(Boolean("0")); // true - строка, даже с "0", является true
console.log(Boolean("")); // false
console.log(Boolean(" ")); // true - строка с пробелом
```

### null и undefined
- `null` - "ничего", "пусто", "известно что пусто"
- `undefined` - "значение не присвоено", "неизвестно"

```javascript
let age;
console.log(age); // undefined

let user = null;
console.log(user); // null

// Проверка на null/undefined
console.log(typeof null); // "object" - это известная ошибка в JavaScript
console.log(typeof undefined); // "undefined"
console.log(null === undefined); // false
console.log(null == undefined); // true
```

### Symbol
Уникальные и неизменяемые значения, используемые как идентификаторы для свойств объектов:

```javascript
let id1 = Symbol("id");
let id2 = Symbol("id");

console.log(id1 == id2); // false - даже с одинаковым описанием
console.log(id1.description); // "id"

// Символы не преобразуются к строке автоматически
// console.log("Мой символ: " + id1); // Ошибка!
console.log("Мой символ: " + id1.toString()); // "Мой символ: Symbol(id)"
```

### Object
Коллекция свойств, которые могут быть любыми типами данных:

```javascript
let user = {
    name: "Иван",
    age: 30,
    isAdmin: true
};

// Доступ к свойствам
console.log(user.name); // "Иван"
console.log(user["name"]); // "Иван"
```

## Определение типа данных

### typeof оператор
```javascript
typeof undefined // "undefined"
typeof 0 // "number"
typeof true // "boolean"
typeof "foo" // "string"
typeof Symbol("id") // "symbol"
typeof Math // "object" (встроенный объект)
typeof null // "object" - это ошибка в языке
typeof alert // "function" (внутренне специальный подтип объекта)
```

### Проверка на null
```javascript
// Правильная проверка на null
console.log(obj === null); // true если obj - null

// Проверка на null или undefined
if (value == null) {
    // выполнится если value равен null или undefined
}

// Проверка на объект (но не функцию или null)
console.log(typeof obj === "object" && obj !== null); // true для объектов
```

## Преобразование типов

JavaScript автоматически преобразует типы в определенных ситуациях:

```javascript
// Строковое преобразование
String(123) // "123"
"" + 123 // "123"

// Численное преобразование
Number("123") // 123
+"123" // 123
parseInt("123.45") // 123
parseFloat("123.45") // 123.45

// Логическое преобразование
Boolean("0") // true (!)
Boolean("") // false
!!"0" // true
```

## Современные возможности

### Optional chaining (?.)
Безопасный доступ к вложенным свойствам:

```javascript
let user = {};
console.log(user.address?.street); // undefined, без ошибки
console.log(user.address?.street?.name); // undefined, без ошибки

// Методы
user.sayHi?.(); // Вызов метода, если он существует
```

### Nullish coalescing operator (??)
Возвращает первый операнд, если он не null/undefined:

```javascript
let user = null;
let defaultName = "Иван";

console.log(user ?? defaultName); // "Иван"

// Разница между || и ??
console.log(0 || "default"); // "default"
console.log(0 ?? "default"); // 0
```

## Лучшие практики

1. **Используйте строгое равенство (===)**
```javascript
// Плохо
if (value == null) { }

// Лучше для null/undefined
if (value == null) { } // работает для null и undefined

// Или явно
if (value === null || value === undefined) { }
```

2. **Проверяйте типы при необходимости**
```javascript
// Проверка на массив
Array.isArray(arr)

// Проверка на объект (но не null)
typeof obj === 'object' && obj !== null && !Array.isArray(obj)

// Проверка на функцию
typeof fn === 'function'
```

3. **Используйте Number.isNaN() вместо глобального isNaN()**
```javascript
// Плохо
isNaN("строка"); // true - но это не NaN

// Хорошо
Number.isNaN("строка"); // false
Number.isNaN(NaN); // true
```

## Примеры из промышленной разработки

### Валидация входных данных
```javascript
function validateInput(data) {
    if (typeof data !== 'object' || data === null) {
        throw new Error('Ожидается объект');
    }
    
    const { name, age } = data;
    
    if (typeof name !== 'string' || name.trim() === '') {
        throw new Error('Имя должно быть непустой строкой');
    }
    
    if (typeof age !== 'number' || age <= 0 || !Number.isInteger(age)) {
        throw new Error('Возраст должен быть положительным целым числом');
    }
    
    return true;
}
```

### Утилита для безопасной работы с типами
```javascript
const TypeChecker = {
    isString: (value) => typeof value === 'string',
    isNumber: (value) => typeof value === 'number' && !isNaN(value),
    isBoolean: (value) => typeof value === 'boolean',
    isFunction: (value) => typeof value === 'function',
    isArray: (value) => Array.isArray(value),
    isObject: (value) => typeof value === 'object' && value !== null && !Array.isArray(value),
    isNull: (value) => value === null,
    isUndefined: (value) => value === undefined,
    isNullOrUndefined: (value) => value === null || value === undefined,
    isEmptyString: (value) => TypeChecker.isString(value) && value.trim() === ''
};

// Использование
if (TypeChecker.isNullOrUndefined(user)) {
    // обработка отсутствия пользователя
}
```

## Ссылки на связанные темы
- [[js/basics/scope]] - Область видимости и замыкания
- [[js/functions/closures]] - Замыкания
- [[js/es6+/destructuring]] - Деструктуризация
- [[js/testing/type-checking]] - Типизация в тестах

#programming #javascript #types #best-practices