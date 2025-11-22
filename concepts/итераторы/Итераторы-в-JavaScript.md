---
aliases: ["Итераторы в JavaScript", "JavaScript Iterators", "Работа с итераторами в JS"]
tags: [javascript, frontend-development, iterators, es6]
---

# Итераторы в JavaScript

## Обзор

JavaScript предоставляет мощную и гибкую систему итераторов, которая была внедрена в ES6. Эта система позволяет эффективно перебирать элементы коллекций данных, таких как массивы, строки, Map, Set и пользовательские объекты. Итераторы в JavaScript обеспечивают унифицированный способ доступа к элементам различных типов коллекций, что делает код более читаемым и поддерживаемым.

## Протокол итератора

JavaScript определяет два протокола итератора:
- **Протокол итерируемого объекта** — позволяет определить способ итерации объекта
- **Протокол итератора** — позволяет получить значение из последовательности

### Протокол итератора

Объект является итератором, если он реализует метод `next()`, который возвращает объект с двумя свойствами:
- `value` — значение следующего элемента
- `done` — логическое значение, указывающее, достигнут ли конец последовательности

```javascript
const simpleIterator = {
  items: ['один', 'два', 'три'],
  index: 0,
  
  next() {
    if (this.index < this.items.length) {
      return {
        value: this.items[this.index++],
        done: false
      };
    } else {
      return {
        done: true
      };
    }
  }
};

console.log(simpleIterator.next()); // { value: 'один', done: false }
console.log(simpleIterator.next()); // { value: 'два', done: false }
console.log(simpleIterator.next()); // { value: 'три', done: false }
console.log(simpleIterator.next()); // { done: true }
```

### Протокол итерируемого объекта

Объект является итерируемым, если он имеет метод с ключом `Symbol.iterator`, который возвращает итератор. Множество встроенных типов JavaScript являются итерируемыми:

- Массивы
- Строки
- Map
- Set
- arguments
- NodeList

```javascript
const myArray = [1, 2, 3];
const iterator = myArray[Symbol.iterator]();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

## Цикл for...of

Цикл `for...of` был добавлен в ES6 для итерации по итерируемым объектам. Он автоматически использует протокол итератора:

```javascript
const iterable = [10, 20, 30];

for (const value of iterable) {
  console.log(value);
}
// Вывод: 10, 20, 30
```

Цикл `for...of` работает с любыми итерируемыми объектами:

```javascript
// Итерация по строке
for (const char of 'hello') {
  console.log(char);
}
// Вывод: h, e, l, l, o

// Итерация по Set
const mySet = new Set([1, 2, 3]);
for (const item of mySet) {
  console.log(item);
}
// Вывод: 1, 2, 3

// Итерация по Map
const myMap = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of myMap) {
  console.log(key, value);
}
// Вывод: a 1, b 2
```

## Методы массивов и итераторы

Многие методы массивов, такие как `map`, `filter`, `reduce`, `forEach`, используют итераторы внутренне:

```javascript
const numbers = [1, 2, 3, 4, 5];

// map - создает новый массив с результатами вызова функции для каждого элемента
const doubled = numbers.map(x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter - создает новый массив с элементами, прошедшими проверку
const evens = numbers.filter(x => x % 2 === 0);
console.log(evens); // [2, 4]

// reduce - применяет функцию к аккумулятору и каждому элементу
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 15
```

## Генераторы

Генераторы — это специальный тип функций, которые могут приостанавливать своё выполнение и возвращать значения по запросу. Они являются итераторами и создаются с помощью ключевого слова `function*`:

```javascript
function* simpleGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = simpleGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Использование генератора в цикле for...of
for (const value of simpleGenerator()) {
  console.log(value);
}
// Вывод: 1, 2, 3
```

### Генераторы как итераторы для пользовательских объектов

```javascript
const collection = {
  items: [1, 2, 3, 4, 5],
  
  *[Symbol.iterator]() {
    for (const item of this.items) {
      yield item;
    }
  }
};

for (const item of collection) {
  console.log(item);
}
// Вывод: 1, 2, 3, 4, 5
```

## Практические примеры

### Создание итератора для диапазона чисел

```javascript
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  *[Symbol.iterator]() {
    for (let i = this.start; i < this.end; i += this.step) {
      yield i;
    }
  }
}

const range = new Range(1, 10, 2);
for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}
```

### Итератор для дерева

```javascript
class TreeNode {
  constructor(value, children = []) {
    this.value = value;
    this.children = children;
  }

  *[Symbol.iterator]() {
    yield this.value;
    for (const child of this.children) {
      yield* child; // Рекурсивная итерация через yield*
    }
  }
}

const tree = new TreeNode('root', [
  new TreeNode('child1', [
    new TreeNode('grandchild1'),
    new TreeNode('grandchild2')
  ]),
  new TreeNode('child2')
]);

for (const node of tree) {
  console.log(node);
}
// Вывод: root, child1, grandchild1, grandchild2, child2
```

### Итератор для асинхронных данных

```javascript
async function* asyncIterator() {
  const urls = ['https://api1.com', 'https://api2.com', 'https://api3.com'];
  
  for (const url of urls) {
    yield await fetch(url).then(response => response.json());
  }
}

// Использование асинхронного итератора
async function processData() {
  for await (const data of asyncIterator()) {
    console.log(data);
  }
}
```

## Итераторы и производительность

Итераторы могут быть эффективны с точки зрения производительности, особенно при использовании ленивых вычислений:

```javascript
// Ленивый итератор для бесконечной последовательности
function* fibonacci() {
  let a = 0, b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fib = fibonacci();
for (let i = 0; i < 10; i++) {
  console.log(fib.next().value); // 0, 1, 1, 2, 3, 5, 8, 13, 21, 34
}
```

## Практические рекомендации

- Используйте `for...of` для итерации по итерируемым объектам
- Реализуйте `Symbol.iterator` для пользовательских коллекций
- Используйте генераторы для создания сложных итераторов
- Используйте `yield*` для делегирования итерации другим итераторам
- Для асинхронных данных используйте `for await...of`
- Обратите внимание на производительность при работе с большими коллекциями

## Связь с другими концепциями

- [[Паттерн-итератор]] — базовый паттерн, на котором основаны итераторы в JavaScript
- [[Асинхронное-программирование]] — асинхронные итераторы и генераторы
- [[Функциональное-программирование]] — методы массивов как функции высшего порядка
- [[Генераторы]] — специальный тип функций для создания итераторов

## Заключение

Итераторы в JavaScript предоставляют мощный и гибкий способ работы с коллекциями данных. Они интегрированы в язык и его встроенные типы, что делает их удобными для использования. Понимание и эффективное использование итераторов позволяет писать более чистый, читаемый и поддерживаемый код.

## См. также

- [[Итераторы-в-React]]
- [[Итераторы-в-Vue]]
- [[Итераторы-и-производительность]]
- [[Паттерн-итератор]]
- [[Асинхронное-программирование]]
- [[Функциональное-программирование]]
