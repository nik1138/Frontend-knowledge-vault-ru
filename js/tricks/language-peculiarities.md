---
aliases: ["Особенности JavaScript", "JS Peculiarities", "JavaScript Quirks"]
tags: [js, quirks, gotchas, advanced]
---

# Неочевидные особенности языка

## Введение

JavaScript полон неожиданных особенностей и подводных камней, которые могут удивить даже опытных разработчиков. Понимание этих особенностей помогает писать более надежный и предсказуемый код.

## Особенности типизации и сравнения

### 1. Ложные значения (falsy values)

```javascript
// Все ложные значения в JavaScript
const falsyValues = [
  false,      // Boolean
  0,          // Number
  -0,         // Number (отрицательный ноль)
  0n,         // BigInt
  '',         // String
  null,       // Null
  undefined,  // Undefined
  NaN         // Not a Number
];

// Проверка ложных значений
falsyValues.forEach(value => {
  console.log(`${value} is falsy: ${!value}`);
});

// Особенность: пустой массив и объект - это truthy значения!
console.log(Boolean([]));    // true
console.log(Boolean({}));    // true
console.log(Boolean(new Date(''))); // false (неправильная дата)
```

### 2. Сравнение с == и ===

```javascript
// Странное поведение == (лояльное сравнение)
console.log(0 == false);        // true
console.log(1 == true);         // true
console.log('0' == false);      // true
console.log('1' == true);       // true
console.log(null == undefined); // true
console.log(' \t\r\n ' == 0);  // true (строка с пробелами)

// Сравнение строк
console.log('0' == 0);          // true
console.log('0' == false);      // true
console.log(false == 'false');  // false (!)
console.log(' \n\t ' == 0);     // true

// Более странные примеры
console.log([] == false);       // true
console.log([] == ![]);         // true (!)
console.log([0] == false);      // true
console.log([1] == true);       // true

// Почему [1] == true, но [2] != true?
console.log([1] == 1);          // true (массив преобразуется к строке "1", потом к числу 1)
console.log([2] == 2);          // true (работает и для [2])
console.log([1,2] == '1,2');    // true
console.log([1,2] == '12');     // false

// Всегда используйте === для предсказуемого поведения
console.log(0 === false);       // false
console.log('0' === 0);         // false
console.log(null === undefined); // false
```

### 3. Особенности сравнения объектов

```javascript
// Сравнение объектов происходит по ссылке, а не по значению
const obj1 = { a: 1 };
const obj2 = { a: 1 };
const obj3 = obj1;

console.log(obj1 == obj2);  // false (разные объекты в памяти)
console.log(obj1 === obj2); // false (разные объекты в памяти)
console.log(obj1 === obj3); // true (один и тот же объект)

// Даже одинаковые массивы не равны
console.log([1, 2, 3] == [1, 2, 3]);  // false
console.log([1, 2, 3] === [1, 2, 3]); // false

// Но массив равен самому себе
let arr = [1, 2, 3];
console.log(arr === arr); // true

// Особенность: пустые массивы при сравнении с 0
console.log([] == 0);      // true
console.log([] == false);  // true
console.log([].length == 0); // true
console.log(Boolean([]));    // true (!)
```

## Особенности работы с числами

### 4. Проблемы с точностью чисел с плавающей запятой

```javascript
// Классическая проблема точности
console.log(0.1 + 0.2);           // 0.30000000000000004 (!)
console.log(0.1 + 0.2 === 0.3);   // false (!)
console.log(0.1 + 0.2 == 0.3);    // false (!)

// Решения проблемы
function isEqual(a, b, epsilon = Number.EPSILON) {
  return Math.abs(a - b) < epsilon;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true

// Или округление
function roundTo(num, decimals) {
  return Math.round(num * Math.pow(10, decimals)) / Math.pow(10, decimals);
}

console.log(roundTo(0.1 + 0.2, 1)); // 0.3

// Другие странные числовые особенности
console.log(0.1 + 0.2 + 0.3);     // 0.6000000000000001
console.log(0.3 + 0.6);           // 0.8999999999999999
```

### 5. Особенности parseInt и числовых преобразований

```javascript
// Проблемы с parseInt без указания основания
console.log(parseInt('10'));      // 10
console.log(parseInt('10.5'));    // 10
console.log(parseInt('10px'));    // 10
console.log(parseInt('010'));     // 8 (!) - восьмеричная система
console.log(parseInt('0x10'));    // 16 - шестнадцатеричная система

// Правильное использование
console.log(parseInt('010', 10)); // 10 - указываем основание

// Странное поведение parseInt с массивами
console.log(parseInt('10', 2));   // 2 (10 в двоичной системе)
console.log(parseInt('10', 8));   // 8 (10 в восьмеричной системе)
console.log(parseInt('10', 16));  // 16 (10 в шестнадцатеричной системе)

// parseInt с массивом (редкий случай)
console.log(['10', '10', '10'].map(parseInt)); // [10, NaN, 2] (!)

// Почему так происходит:
// ['10', '10', '10'].map(parseInt) эквивалентно:
// parseInt('10', 0) -> 10 (основание 10)
// parseInt('10', 1) -> NaN (основание 1 недопустимо)
// parseInt('10', 2) -> 2 (10 в двоичной системе)

// Правильный способ:
console.log(['10', '10', '10'].map(num => parseInt(num, 10))); // [10, 10, 10]
```

### 6. Особенности Number.MAX_SAFE_INTEGER

```javascript
// Максимальное безопасное целое число
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER); // -9007199254740991

// Что происходит за пределами безопасного диапазона
const maxSafe = Number.MAX_SAFE_INTEGER;
console.log(maxSafe);               // 9007199254740991
console.log(maxSafe + 1);           // 9007199254740992
console.log(maxSafe + 2);           // 9007199254740992 (!)
console.log(maxSafe + 1 === maxSafe + 2); // true (!)

// BigInt для работы с большими числами
const bigNumber = 9007199254740991n;
console.log(bigNumber + 1n);        // 9007199254740992n
console.log(bigNumber + 2n);        // 9007199254740993n
console.log(bigNumber + 1n === bigNumber + 2n); // false

// Смешивание BigInt и Number
// console.log(bigNumber + 1); // TypeError: Cannot mix BigInt and other types
console.log(Number(bigNumber) + 1); // 9007199254740992 (небезопасно!)
```

## Особенности работы с массивами

### 7. Пустые ячейки в массивах

```javascript
// Создание массивов с пустыми ячейками
const arr1 = new Array(5);           // [empty × 5]
const arr2 = [1, , , 4];             // [1, empty, empty, 4]
const arr3 = [1, undefined, undefined, 4]; // [1, undefined, undefined, 4]

console.log(arr1.length);            // 5
console.log(arr2.length);            // 4
console.log(arr3.length);            // 4

// Различия между empty и undefined
console.log(0 in arr1);              // false (ячейка не существует)
console.log(0 in arr2);              // true (ячейка существует, но пустая)
console.log(0 in arr3);              // true (ячейка существует с undefined)

// Итерация по массивам с пустыми ячейками
arr2.forEach((item, index) => {
  console.log(`${index}: ${item}`);  // 0: 1, 3: 4 (пропускает пустые)
});

// for...in также пропускает пустые ячейки в arr1, но обрабатывает arr2
for (let i in arr1) console.log(i);  // ничего не выводит
for (let i in arr2) console.log(i);  // 0, 3

// for...of пропускает пустые ячейки
for (const item of arr2) {
  console.log(item);                 // 1, 4
}

// map также пропускает пустые ячейки
console.log(arr2.map(x => x * 2));   // [2, empty, empty, 8]
```

### 8. Особенности методов массивов

```javascript
// splice может возвращать пустой массив
const arr = [1, 2, 3];
const removed = arr.splice(5, 2);    // [] (ничего не удалено)
console.log(removed);                // []
console.log(arr);                    // [1, 2, 3]

// slice с отрицательными индексами
const letters = ['a', 'b', 'c', 'd', 'e'];
console.log(letters.slice(-2));      // ['d', 'e']
console.log(letters.slice(1, -1));   // ['b', 'c', 'd']

// fill с объектами (все элементы ссылаются на один объект!)
const matrix = new Array(3).fill([]); // [[], [], []]
matrix[0].push(1);
console.log(matrix);                 // [[1], [1], [1]] (!)

// Правильный способ:
const properMatrix = Array.from({ length: 3 }, () => []);

// flat с глубиной
const nested = [1, [2, [3, [4]]]];
console.log(nested.flat());          // [1, 2, [3, [4]]]
console.log(nested.flat(2));         // [1, 2, 3, [4]]
console.log(nested.flat(Infinity));  // [1, 2, 3, 4]
```

## Особенности работы с функциями

### 9. arguments и стрелочные функции

```javascript
// arguments не доступен в стрелочных функциях
function regularFunction() {
  console.log(arguments);            // Arguments object
  return Array.from(arguments);
}

const arrowFunction = () => {
  // console.log(arguments);         // ReferenceError: arguments is not defined
  return Array.from(arguments);      // ReferenceError
};

// arguments - это Array-like, но не массив
function example() {
  console.log(Array.isArray(arguments)); // false
  console.log(arguments instanceof Array); // false
  console.log(arguments.length);     // длина
  console.log(arguments[0]);         // первый элемент
  
  // Преобразование к массиву
  const argsArray = Array.from(arguments);
  const argsArray2 = [...arguments];
  const argsArray3 = Array.prototype.slice.call(arguments);
}

// Правильное использование параметров
const properFunction = (...args) => {
  console.log(Array.isArray(args));  // true
  return args;
};
```

### 10. Особенности привязки this

```javascript
const obj = {
  name: 'Объект',
  regularFunction: function() {
    console.log(this.name);          // 'Объект'
  },
  arrowFunction: () => {
    console.log(this.name);          // undefined (в браузере this = Window)
  },
  nested: {
    name: 'Вложенный',
    method: function() {
      console.log(this.name);        // 'Вложенный'
      
      const arrow = () => console.log(this.name); // 'Вложенный' (лексическое this)
      const regular = function() { console.log(this.name); }; // undefined (если вызвать напрямую)
      
      arrow();
      regular();
    }
  }
};

// Потеря контекста
const method = obj.regularFunction;
method();                            // undefined (!)

// Привязка контекста
const boundMethod = obj.regularFunction.bind(obj);
boundMethod();                       // 'Объект'

// Особенность call/apply с примитивами
function example() {
  console.log(this);
}

example.call('hello');              // String {'hello'} (обертка в объект)
example.call(42);                   // Number {42} (обертка в объект)
example.call(true);                 // Boolean {true} (обертка в объект)
```

## Особенности работы с замыканиями

### 11. Замыкания в циклах

```javascript
// Классическая проблема
const funcs = [];
for (var i = 0; i < 3; i++) {
  funcs.push(function() { return i; });
}

console.log(funcs[0]());             // 3 (!)
console.log(funcs[1]());             // 3 (!)
console.log(funcs[2]());             // 3 (!)

// Решение 1: IIFE (Immediately Invoked Function Expression)
const funcs2 = [];
for (var i = 0; i < 3; i++) {
  funcs2.push((function(index) {
    return function() { return index; };
  })(i));
}

console.log(funcs2[0]());            // 0
console.log(funcs2[1]());            // 1
console.log(funcs2[2]());            // 2

// Решение 2: let вместо var
const funcs3 = [];
for (let i = 0; i < 3; i++) {
  funcs3.push(function() { return i; });
}

console.log(funcs3[0]());            // 0
console.log(funcs3[1]());            // 1
console.log(funcs3[2]());            // 2

// Решение 3: const в цикле
const funcs4 = [];
for (let i = 0; i < 3; i++) {
  const index = i;
  funcs4.push(() => index);
}

console.log(funcs4[0]());            // 0
console.log(funcs4[1]());            // 1
console.log(funcs4[2]());            // 2
```

### 12. Особенности модулей и области видимости

```javascript
// Особенности var, let, const
function example() {
  console.log(a);                    // undefined (не ReferenceError!)
  console.log(b);                    // ReferenceError: Cannot access before initialization
  console.log(c);                    // ReferenceError: Cannot access before initialization
  
  var a = 1;
  let b = 2;
  const c = 3;
}

// Временная мертвая зона (Temporal Dead Zone)
function tdzExample() {
  typeof undeclaredVariable;         // 'undefined' (не ошибка)
  // typeof uninitializedLet;       // ReferenceError в TDZ
  let uninitializedLet;
}

// Особенности с объектами и const
const obj = { name: 'Имя' };
obj.name = 'Новое имя';             // Работает! const защищает ссылку, не содержимое
// obj = { name: 'Другой' };        // TypeError: Assignment to constant variable

// Глобальный объект
var globalVar = 'var';
let globalLet = 'let';
const globalConst = 'const';

console.log(window.globalVar);       // 'var'
console.log(window.globalLet);       // undefined
console.log(window.globalConst);     // undefined
```

## Особенности работы с прототипами

### 13. Изменение прототипов встроенных объектов

```javascript
// Изменение прототипа Array
Array.prototype.sum = function() {
  return this.reduce((acc, val) => acc + val, 0);
};

console.log([1, 2, 3].sum());       // 6

// Проблема: for...in перебирает добавленные методы
const arr = [1, 2, 3];
for (let key in arr) {
  console.log(key);                  // 0, 1, 2, 'sum' (!)
}

// Решение: использовать hasOwnProperty
for (let key in arr) {
  if (arr.hasOwnProperty(key)) {
    console.log(key);                // 0, 1, 2
  }
}

// Или использовать Object.getOwnPropertyNames
console.log(Object.getOwnPropertyNames(arr)); // ['0', '1', '2']

// Лучше использовать Object.defineProperty для неперечисляемых свойств
Object.defineProperty(Array.prototype, 'sum2', {
  value: function() { return this.reduce((a, b) => a + b, 0); },
  enumerable: false,                 // не будет в for...in
  writable: true,
  configurable: true
});

// Теперь sum2 не появляется в for...in
for (let key in [1, 2, 3]) {
  console.log(key);                  // 0, 1, 2
}
```

### 14. Особенности наследования

```javascript
// Прототипная цепочка
function Parent() {
  this.parentProp = 'parent';
}

Parent.prototype.parentMethod = function() {
  return 'parent method';
};

function Child() {
  this.childProp = 'child';
}

// Неправильное наследование
Child.prototype = Parent.prototype;  // Проблема: изменения в Child affect Parent

// Правильное наследование
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

Child.prototype.childMethod = function() {
  return 'child method';
};

const child = new Child();
console.log(child.parentMethod());   // 'parent method'
console.log(child.childMethod());    // 'child method'

// Проверка instanceof
console.log(child instanceof Child); // true
console.log(child instanceof Parent); // true
console.log(child instanceof Object); // true

// Особенность: конструктор может быть изменен
function AnotherChild() {}
AnotherChild.prototype = Object.create(Parent.prototype);
// Забыли установить constructor!

const anotherChild = new AnotherChild();
console.log(anotherChild.constructor === Parent); // true (!)
console.log(anotherChild.constructor === AnotherChild); // false (!)
```

## Особенности работы с асинхронностью

### 15. Особенности Promise

```javascript
// Promise.resolve с уже разрешенным Promise
const alreadyResolved = Promise.resolve('уже разрешен');
const wrapped = Promise.resolve(alreadyResolved);
console.log(alreadyResolved === wrapped); // true (Promise.resolve возвращает тот же Promise)

// Promise.reject всегда создает новый Promise
const alreadyRejected = Promise.reject('уже отклонен');
const wrappedRejected = Promise.resolve(alreadyRejected); // Оборачивает rejected Promise
console.log(alreadyRejected === wrappedRejected); // false

// Promise.all с первым отклонением
Promise.all([
  Promise.resolve(1),
  Promise.reject('ошибка'),
  Promise.resolve(3)  // Этот Promise не будет выполнен
]).catch(err => console.log(err));   // 'ошибка'

// Promise.allSettled ждет все Promise
Promise.allSettled([
  Promise.resolve(1),
  Promise.reject('ошибка'),
  Promise.resolve(3)
]).then(results => {
  console.log(results);             // [{status: 'fulfilled', value: 1}, {status: 'rejected', reason: 'ошибка'}, {status: 'fulfilled', value: 3}]
});

// Асинхронность Promise
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');

// Вывод: 1, 4, 3, 2 (Promise имеет более высокий приоритет, чем setTimeout)
```

### 16. Особенности async/await

```javascript
// async функция всегда возвращает Promise
async function asyncFunction() {
  return 'результат';
}

const result = asyncFunction();
console.log(result);                 // Promise { 'результат' }
console.log(result instanceof Promise); // true

// await с примитивами
async function example() {
  const primitive = await 42;
  console.log(primitive);            // 42 (await с примитивом возвращает сам примитив)
  
  const promise = await Promise.resolve('hello');
  console.log(promise);              // 'hello'
}

// await в выражениях
async function complexExample() {
  const url = await Promise.resolve('https://api.example.com');
  const response = await fetch(url);
  const data = await response.json();
  
  // await можно использовать в большинстве выражений
  const arr = [await Promise.resolve(1), await Promise.resolve(2)];
  console.log(arr);                  // [1, 2]
  
  const obj = { value: await Promise.resolve(42) };
  console.log(obj);                  // { value: 42 }
}

// Ошибки в async функциях
async function errorExample() {
  try {
    const result = await Promise.reject(new Error('ошибка'));
  } catch (error) {
    console.log(error.message);      // 'ошибка'
  }
  
  // Если не обернуть в try-catch, ошибка превращается в rejected Promise
  // await Promise.reject(new Error('необработанная ошибка'));
}
```

## Особенности работы с JSON

### 17. Особенности JSON.stringify и JSON.parse

```javascript
// JSON.stringify игнорирует определенные значения
const obj = {
  a: 1,
  b: undefined,
  c: function() {},
  d: Symbol('test'),
  e: null,
  f: NaN,
  g: Infinity
};

console.log(JSON.stringify(obj));    // {"a":1,"e":null,"f":null,"g":null}

// Поведение с циклическими ссылками
const circular = { name: 'circular' };
circular.self = circular;

// JSON.stringify(circular);         // TypeError: Converting circular structure to JSON

// Поведение с Date
const date = new Date();
const dateStr = JSON.stringify({ date: date });
console.log(dateStr);                // {"date":"2023-01-01T00:00:00.000Z"}
console.log(JSON.parse(dateStr));    // { date: "2023-01-01T00:00:00.000Z" } (строка!)

// Решение для Date: реверсирование
const parsedWithDate = JSON.parse(dateStr, (key, value) => {
  if (key === 'date' && typeof value === 'string') {
    return new Date(value);
  }
  return value;
});

// JSON.stringify с replacer
const data = { a: 1, b: 2, secret: 'password' };
const safe = JSON.stringify(data, (key, value) => {
  if (key === 'secret') return undefined;
  return value;
});
console.log(safe);                   // {"a":1,"b":2}

// Поведение с массивами
const sparseArray = [1, , 3];        // [1, empty, 3]
console.log(JSON.stringify(sparseArray)); // [1,null,3] (empty становится null)
```

## Ссылки по теме

- [[js/best-practices]] - Лучшие практики
- [[js/debugging]] - Отладка JavaScript
- [[js/performance]] - Производительность
- [[js/security]] - Безопасность JavaScript