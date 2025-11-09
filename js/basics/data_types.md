# Типы данных в JavaScript: Продвинутые концепции

## Обзор

JavaScript имеет несколько встроенных типов данных, которые делятся на примитивные и объектные типы. Понимание особенностей каждого типа и их поведения в движке JavaScript критически важно для написания надежного кода.

## Примитивные типы данных

- `number` - числа (целые и дробные)
- `string` - строки
- `boolean` - логический тип (true/false)
- `undefined` - значение не определено
- `null` - "ничего", пустое значение
- `symbol` - уникальные и неизменяемые значения (ES6+)
- `bigint` - большие целые числа (ES2020+)

## Объектные типы данных

- `Object` - объекты
- `Array` - массивы
- `Function` - функции
- `Date` - даты
- `RegExp` - регулярные выражения
- и другие

## Примеры

```javascript
// Число
let age = 25;

// Строка
let name = "Анна";

// Булевый тип
let isActive = true;

// Undefined
let emptyIsUndefined;

// Null
let emptyIsNull = null;

// Symbol
let id = Symbol("id");

// BigInt
let bigNumber = 123456789012345678901234567890n;

// Объект
let user = { name: "Петр", age: 35 };

// Массив
let fruits = ["яблоко", "банан", "апельсин"];
```

## Проверка типа данных

```javascript
typeof "hello"     // "string"
typeof 42          // "number"
typeof true        // "boolean"
typeof undefined   // "undefined"
typeof null        // "object" (это ошибка в JavaScript)
```

## Продвинутые концепции типов данных

### Подробнее о Number

JavaScript использует формат IEEE 754 для представления чисел с плавающей точкой двойной точности (64-битные числа). Это приводит к интересным особенностям:

```javascript
// Проблемы с точностью
0.1 + 0.2 === 0.3; // false
console.log(0.1 + 0.2); // 0.30000000000000004

// Решение проблемы точности
function isEqual(a, b, epsilon = Number.EPSILON) {
  return Math.abs(a - b) < epsilon;
}

isEqual(0.1 + 0.2, 0.3); // true

// Специальные значения
console.log(1 / 0); // Infinity
console.log(-1 / 0); // -Infinity
console.log(0 / 0); // NaN (Not-a-Number)

// Проверка на NaN
isNaN(NaN); // true
Number.isNaN(NaN); // true (лучше использовать этот метод)
isNaN("hello"); // true (ложное срабатывание)
Number.isNaN("hello"); // false (корректная проверка)

// Проверка на конечность
isFinite(123); // true
isFinite(Infinity); // false
Number.isFinite(123); // true
```

### Продвинутые возможности BigInt

```javascript
// BigInt для работы с очень большими числами
const bigNumber = 1234567890123456789012345678901234567890n;
const anotherBig = BigInt("123456789012345678901234567890");

// Операции с BigInt
const result = bigNumber + 10n; // нужно использовать суффикс n для чисел
console.log(result);

// BigInt нельзя сравнивать с обычными числами без преобразования
// bigNumber > 12345678901234567890; // может вызвать ошибку

// Преобразование BigInt к числу
const normalNumber = Number(bigNumber);
```

### Строки: неизменяемость и оптимизация

```javascript
// Строки в JavaScript неизменяемы
let str = "hello";
str[0] = "H"; // не изменит строку
console.log(str); // "hello", а не "Hello"

// Оптимизация строк в движке V8
let s1 = "hello";
let s2 = "hello";
console.log(s1 === s2); // true (строки интернированы)

// Интернирование строк
// Движок может интернировать короткие строки для экономии памяти
const short = "abc";
const alsoShort = "abc";
console.log(Object.is(short, alsoShort)); // true в большинстве случаев

// Шаблонные строки и производительность
const name = "Иван";
const age = 30;

// Менее эффективно
const greeting1 = "Привет, меня зовут " + name + ", мне " + age + " лет";

// Более эффективно
const greeting2 = `Привет, меня зовут ${name}, мне ${age} лет`;
```

### Символы и их применение

```javascript
// Символы для создания уникальных идентификаторов
const id1 = Symbol("id");
const id2 = Symbol("id");
console.log(id1 === id2); // false (символы уникальны)

// Символы как ключи свойств
const user = {
  name: "Иван"
};

const symbolKey = Symbol("id");
user[symbolKey] = 123;

console.log(user[symbolKey]); // 123
console.log(Object.keys(user)); // ["name"] (символьные ключи не перечисляются)

// Общие символы (shared symbols)
const sharedId = Symbol.for("sharedId");
const anotherSharedId = Symbol.for("sharedId");
console.log(sharedId === anotherSharedId); // true

// Получение имени общего символа
console.log(Symbol.keyFor(sharedId)); // "sharedId"
```

## Типизация в JavaScript

### Динамическая типизация и автоматическое преобразование

```javascript
// Явное преобразование типов
const strNum = "123";
const num = Number(strNum); // 123
const bool = Boolean(strNum); // true

// Неявное преобразование (приведение типов)
console.log("5" - 3); // 2 (строка преобразуется в число)
console.log("5" + 3); // "53" (число преобразуется в строку)
console.log("5" == 5); // true (с приведением типов)
console.log("5" === 5); // false (без приведения типов)

// Правила приведения типов
"" + 1 + 0; // "10"
"" - 1 + 0; // -1
true + false; // 1
6 / "3"; // 2
"2" * "3"; // 6
4 + 5 + "px"; // "9px"
"$" + 4 + 5; // "$45"
"4" - 2; // 2
"4px" - 2; // NaN
"  -9  " + 5; // "  -9  5"
"  -9  " - 5; // -14
null + 1; // 1
undefined + 1; // NaN
```

### Псевдо-статическая типизация с JSDoc

```javascript
/**
 * @typedef {Object} User
 * @property {number} id - Уникальный идентификатор пользователя
 * @property {string} name - Имя пользователя
 * @property {number} [age] - Возраст пользователя (необязательно)
 * @property {boolean} isActive - Статус активности
 */

/**
 * @param {User} user - Объект пользователя
 * @param {string} property - Имя свойства для получения
 * @returns {any} Значение свойства
 */
function getUserProperty(user, property) {
  return user[property];
}

/**
 * @param {Array<number>} numbers - Массив чисел
 * @returns {number} Сумма чисел
 */
function sum(numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}

// Использование типов
/** @type {User} */
const user = {
  id: 1,
  name: "Иван",
  age: 30,
  isActive: true
};

const total = sum([1, 2, 3, 4, 5]); // 15
```

### Совместимость с TypeScript

```typescript
// Примитивные типы
let isDone: boolean = false;
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
let name: string = "Иван";
let list: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3];

// Tuple
let x: [string, number];
x = ["hello", 10]; // OK
// x = [10, "hello"]; // Ошибка

// Enum
enum Color { Red = 1, Green, Blue }
let c: Color = Color.Green; // 2

// Any и Unknown
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // OK, definitely a boolean

let unsure: unknown = 4;
// unsure.slice(1) // Ошибка - нужно проверить тип
if (typeof unsure === "string") {
    unsure.slice(1); // OK
}

// Void, Null, Undefined
function warnUser(): void {
    console.log("This is my warning message");
}

let u: undefined = undefined;
let n: null = null;

// Never
function error(message: string): never {
    throw new Error(message);
}

function infiniteLoop(): never {
    while (true) {}
}
```

## Особенности движка JavaScript

### Представление типов в движке

```javascript
// Внутренние представления типов в V8
// Small Integer (Smi): числа от -2^31 до 2^31-1 (или -2^32 до 2^32-1 на 64-битных системах)
let smallNum = 123; // хранится как Smi

// Heap Number: числа, которые не помещаются в Smi
let bigNum = 123.456; // хранится как Heap Number

// Преобразования при операциях могут изменить внутреннее представление
let x = 10; // Smi
x = x + 0.1; // теперь Heap Number
```

### Оптимизация объектов

```javascript
// Движок использует Hidden Classes для оптимизации объектов
function Point(x, y) {
  this.x = x;
  this.y = y;
}

const p1 = new Point(1, 2); // Создает Hidden Class
const p2 = new Point(3, 4); // Использует тот же Hidden Class

// Добавление свойств в другом порядке может привести к созданию нового Hidden Class
const p3 = { y: 2, x: 1 }; // Другой Hidden Class - менее эффективно

// Избегайте изменения формы объекта после создания
function badExample() {
  let obj = { a: 1 };
  obj.b = 2; // Изменение формы
  obj.c = 3; // Еще одно изменение формы
  return obj;
}

function goodExample() {
  return { a: 1, b: 2, c: 3 }; // Форма определена сразу
}
```

### Преобразование типов и производительность

```javascript
// Избегайте частых преобразований типов в циклах
function inefficient(arr) {
  let result = 0;
  for (let i = 0; i < arr.length; i++) {
    // Неявное преобразование типов в каждой итерации
    result += arr[i] + ""; // преобразование в строку
  }
  return result;
}

function efficient(arr) {
  // Явное преобразование перед циклом
  const stringArray = arr.map(item => String(item));
  let result = "";
  for (let i = 0; i < stringArray.length; i++) {
    result += stringArray[i];
  }
  return result;
}
```

## Практические рекомендации

### Проверка типов

```javascript
// Проверка типов для примитивов
function isString(value) {
  return typeof value === 'string' || value instanceof String;
}

function isNumber(value) {
  return typeof value === 'number' && isFinite(value);
}

function isBoolean(value) {
  return typeof value === 'boolean';
}

function isNull(value) {
  return value === null;
}

function isUndefined(value) {
  return value === undefined;
}

function isNullOrUndefined(value) {
  return value == null; // проверяет и null, и undefined
}

// Проверка типов для объектов
function isObject(value) {
  return value !== null && typeof value === 'object' && !Array.isArray(value);
}

function isArray(value) {
  return Array.isArray(value);
}

function isFunction(value) {
  return typeof value === 'function';
}

// Проверка на пустоту
function isEmpty(value) {
  if (value === null || value === undefined) return true;
  if (typeof value === 'string') return value.trim().length === 0;
  if (Array.isArray(value)) return value.length === 0;
  if (typeof value === 'object') return Object.keys(value).length === 0;
  return false;
}
```

### Работа с неопределенными значениями

```javascript
// Использование оператора нулевого слияния (ES2020)
const user = {
  name: "Иван",
  age: null
};

// Получение значения с fallback
const displayName = user.name ?? "Гость"; // "Иван"
const displayAge = user.age ?? 0; // 0 (а не null)

// Оператор цепочки вызовов (optional chaining)
const address = user.address?.street ?? "Адрес не указан";

// Сравнение с логическим ИЛИ
const oldWay = user.name || "Гость"; // работает, но возвращает "Гость" для пустой строки
const newWay = user.name ?? "Гость"; // возвращает "Гость" только для null/undefined
```

### Оптимизация памяти

```javascript
// Использование WeakMap и WeakSet для избежания утечек памяти
const weakMap = new WeakMap();
const obj = {};

// WeakMap не препятствует сборке мусора
weakMap.set(obj, "данные");
obj = null; // объект может быть собран сборщиком мусора

// Обычный Map препятствует сборке мусора
const map = new Map();
const obj2 = {};
map.set(obj2, "данные");
obj2 = null; // объект НЕ будет собран сборщиком мусора, пока находится в Map
```

## Современные возможности

### BigInt и большие числа

```javascript
// Работа с криптографией и большими числами
const maxSafeInteger = Number.MAX_SAFE_INTEGER; // 9007199254740991
console.log(maxSafeInteger + 1 === maxSafeInteger + 2); // true (!)

// Использование BigInt для точных вычислений
const bigInt1 = BigInt(Number.MAX_SAFE_INTEGER) + 1n;
const bigInt2 = BigInt(Number.MAX_SAFE_INTEGER) + 2n;
console.log(bigInt1 === bigInt2); // false
```

### Типизированные массивы

```javascript
// Для работы с бинарными данными
const buffer = new ArrayBuffer(16); // 16 байт
const int32View = new Int32Array(buffer); // 4 элемента по 4 байта
const float64View = new Float64Array(buffer); // 2 элемента по 8 байт

int32View[0] = 42;
int32View[1] = 123;

console.log(int32View[0]); // 42
console.log(int32View[1]); // 123

// Используется для работы с изображениями, аудио, видео и другими бинарными данными
```

### Специфичные проверки типов

```javascript
// Проверка на целое число
function isInteger(value) {
  return typeof value === 'number' && isFinite(value) && Math.floor(value) === value;
}

// Проверка на число с плавающей точкой
function isFloat(value) {
  return typeof value === 'number' && isFinite(value) && Math.floor(value) !== value;
}

// Проверка на "настоящий" объект (не массив, не null, не функция)
function isPlainObject(value) {
  if (typeof value !== 'object' || value === null) return false;
  return Object.getPrototypeOf(value) === Object.prototype || Object.getPrototypeOf(value) === null;
}
```

## Лучшие практики и советы по производительности

### Использование правильных типов для задач

```javascript
// Для числовых вычислений используйте числа, а не строки
function calculateSum(numbers) {
  // Плохо: преобразование строк в числа
  return numbers.map(n => +n).reduce((sum, n) => sum + n, 0);
}

function calculateSumOptimized(numbers) {
  // Хорошо: работа с числами напрямую
  return numbers.reduce((sum, n) => sum + n, 0);
}

// Для проверки на истинность используйте явные преобразования
function isValid(value) {
  // Явное преобразование к булевому типу
  return Boolean(value);
}

// Для проверки на равенство используйте строгое сравнение
function checkEquality(a, b) {
  return a === b; // вместо a == b
}
```

### Оптимизация строковых операций

```javascript
// Для конкатенации большого количества строк используйте массив
function inefficientStringConcat(items) {
  let result = "";
  for (let i = 0; i < items.length; i++) {
    result += items[i]; // создает новую строку на каждой итерации
  }
  return result;
}

function efficientStringConcat(items) {
  const parts = [];
  for (let i = 0; i < items.length; i++) {
    parts.push(items[i]);
  }
  return parts.join("");
}

// Или используйте template literals для нескольких строк
const message = `
  Имя: ${name}
  Возраст: ${age}
  Город: ${city}
`;
```

## Заключение

Понимание продвинутых концепций типов данных в JavaScript позволяет писать более эффективный, надежный и производительный код. Знание особенностей каждого типа, способов автоматического и явного преобразования, а также оптимизаций движка JavaScript критически важно для профессиональной разработки.

#programming #javascript #data-types #typing #engine #performance #web-development #type-coercion #v8