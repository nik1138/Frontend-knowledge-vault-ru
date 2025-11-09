---
aliases: ["Функции JavaScript", "JS Функции"]
tags: 
  - #javascript
  - #functions
  - #programming
  - #js-basics
  - #advanced-js
---

# Функции в JavaScript: Полное руководство

## Введение

Функции - это основной строительный блок программ на JavaScript. Они позволяют организовывать код, избегать дублирования и создавать модульные приложения. [[Функции в JavaScript]] являются объектами первого класса, что означает их можно передавать как аргументы, возвращать из других функций и присваивать переменным.

## Типы функций

### Function Declaration

Объявление функции создает функцию с указанным именем.

```javascript
function greet(name) {
    return `Привет, ${name}!`;
}
```

### Function Expression

Выражение функции позволяет создать функцию как часть другого выражения.

```javascript
const greet = function(name) {
    return `Привет, ${name}!`;
};
```

### Arrow Functions

Стрелочные функции - это краткий способ создания функций, особенно полезный для коротких функций.

```javascript
const greet = (name) => `Привет, ${name}!`;
const add = (a, b) => a + b;
```

## Параметры и аргументы

Функции могут принимать параметры, которые становятся аргументами при вызове.

```javascript
function multiply(a, b = 1) { // b имеет значение по умолчанию
    return a * b;
}

// Rest параметры
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}
```

## Higher-Order Functions

Функции высшего порядка принимают другие функции как аргументы или возвращают функции.

```javascript
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
console.log(double(5)); // 10
```

## Pure Functions

Чистые функции всегда возвращают одинаковый результат для одинаковых аргументов и не имеют побочных эффектов.

```javascript
// Чистая функция
function add(a, b) {
    return a + b;
}

// Не чистая функция
let multiplier = 2;
function multiply(a) {
    return a * multiplier; // Зависит от внешней переменной
}
```

## Область видимости и замыкания

Замыкание позволяет функции получать доступ к переменным из внешней области видимости даже после возврата из внешней функции.

```javascript
function createCounter() {
    let count = 0;
    return function() {
        count++;
        return count;
    };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

## Рекурсия

Рекурсивные функции вызывают сами себя до достижения базового условия.

```javascript
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

## Методы функций: call, apply, bind

Эти методы позволяют управлять контекстом выполнения функции.

```javascript
const person = {
    name: 'Иван',
    greet: function(greeting) {
        return `${greeting}, я ${this.name}`;
    }
};

const anotherPerson = { name: 'Мария' };

// call
console.log(person.greet.call(anotherPerson, 'Привет')); // "Привет, я Мария"

// apply
console.log(person.greet.apply(anotherPerson, ['Добрый день'])); // "Добрый день, я Мария"

// bind
const boundGreet = person.greet.bind(anotherPerson, 'Здравствуйте');
console.log(boundGreet()); // "Здравствуйте, я Мария"
```

## Генераторы функций

Генераторы позволяют приостанавливать выполнение функции и возобновлять его позже.

```javascript
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

const gen = numberGenerator();
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
```

## Композиция функций

Композиция позволяет объединять несколько функций в одну.

```javascript
const compose = (f, g) => (x) => f(g(x));

const addOne = (x) => x + 1;
const multiplyByTwo = (x) => x * 2;

const addOneThenMultiplyByTwo = compose(multiplyByTwo, addOne);
console.log(addOneThenMultiplyByTwo(5)); // 12
```

## Callbacks, Promises и Async/Await

### Callbacks

Функции обратного вызова передаются как аргументы и вызываются после завершения операции.

```javascript
function fetchData(callback) {
    setTimeout(() => {
        callback('Данные получены');
    }, 1000);
}

fetchData((data) => console.log(data));
```

### Promises

Promise - это объект, представляющий завершение или сбой асинхронной операции.

```javascript
function fetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            const success = true;
            if (success) {
                resolve('Данные получены успешно');
            } else {
                reject('Ошибка при получении данных');
            }
        }, 1000);
    });
}

fetchData()
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

### Async/Await

Синтаксис async/await упрощает работу с промисами.

```javascript
async function getData() {
    try {
        const result = await fetchData();
        console.log(result);
    } catch (error) {
        console.error(error);
    }
}
```

## Связь с другими файлами JS

Функции могут быть экспортированы и импортированы между файлами:

```javascript
// math.js
export function add(a, b) {
    return a + b;
}

export default function multiply(a, b) {
    return a * b;
}

// main.js
import multiply, { add } from './math.js';
console.log(add(2, 3)); // 5
console.log(multiply(2, 3)); // 6
```

[[Функции в JavaScript]] тесно связаны с другими аспектами языка, такими как [[Объекты в JavaScript]] и [[Массивы в JavaScript]], позволяя создавать мощные и гибкие приложения.

## Заключение

Функции являются основой программирования на JavaScript. Понимание различных типов функций, их особенностей и применения позволяет писать более чистый, модульный и поддерживаемый код. [[Функции в JavaScript]] обеспечивают гибкость, необходимую для создания современных веб-приложений.

Для более глубокого изучения также рекомендуется ознакомиться с [[Замыкания в JavaScript]] и [[Прототипное наследование в JavaScript]].