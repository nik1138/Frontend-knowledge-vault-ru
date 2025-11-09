# Итераторы и генераторы в JavaScript

## Введение

Итераторы и генераторы - это мощные возможности ES6+, которые позволяют создавать и работать с последовательностями данных более эффективно и гибко. Они являются основой для многих современных JavaScript API и паттернов программирования.

## Итераторы

Итератор - это объект, который предоставляет доступ к элементам коллекции по одному за раз, сохраняя при этом позицию в последовательности.

### Протокол итератора

Объект является итератором, если он реализует метод `next()`, который возвращает объект с двумя свойствами:
- `value` - текущее значение
- `done` - булево значение, указывающее, завершена ли последовательность

```javascript
// Простой итератор
function createIterator(array) {
  let index = 0;
  
  return {
    next: function() {
      if (index < array.length) {
        return {
          value: array[index++],
          done: false
        };
      } else {
        return {
          done: true
        };
      }
    }
  };
}

const iterator = createIterator(['a', 'b', 'c']);

console.log(iterator.next());  // { value: 'a', done: false }
console.log(iterator.next());  // { value: 'b', done: false }
console.log(iterator.next());  // { value: 'c', done: false }
console.log(iterator.next());  // { done: true }
```

### Итерируемые объекты

Итерируемый объект - это объект, который реализует метод `@@iterator` (символ `Symbol.iterator`). Этот метод возвращает итератор.

```javascript
// Создание итерируемого объекта
const iterableObject = {
  data: [1, 2, 3, 4, 5],
  
  [Symbol.iterator]() {
    let index = 0;
    const data = this.data;
    
    return {
      next() {
        if (index < data.length) {
          return {
            value: data[index++],
            done: false
          };
        } else {
          return {
            done: true
          };
        }
      }
    };
  }
};

// Использование итерируемого объекта
for (const value of iterableObject) {
  console.log(value);  // 1, 2, 3, 4, 5
}

// Преобразование в массив
console.log([...iterableObject]);  // [1, 2, 3, 4, 5]
```

### Встроенные итерируемые объекты

Многие встроенные объекты JavaScript являются итерируемыми:
- `Array`
- `String`
- `Map`
- `Set`
- `arguments`
- `NodeList`

```javascript
// Примеры итерируемых объектов
const array = [1, 2, 3];
const string = "hello";
const map = new Map([['a', 1], ['b', 2]]);
const set = new Set([1, 2, 3]);

// Все они могут использоваться с for...of
for (const item of array) console.log(item);
for (const char of string) console.log(char);
for (const [key, value] of map) console.log(key, value);
for (const value of set) console.log(value);

// И с spread оператором
console.log([...array]);  // [1, 2, 3]
console.log([...string]); // ['h', 'e', 'l', 'l', 'o']
```

## Генераторы

Генераторы - это специальный тип функций, которые могут приостанавливать и возобновлять выполнение. Они возвращают объект Generator, который является итератором.

### Основы генераторов

```javascript
// Объявление генератора
function* generatorFunction() {
  yield 1;
  yield 2;
  yield 3;
}

const generator = generatorFunction();

console.log(generator.next());  // { value: 1, done: false }
console.log(generator.next());  // { value: 2, done: false }
console.log(generator.next());  // { value: 3, done: false }
console.log(generator.next());  // { value: undefined, done: true }

// Использование с for...of
for (const value of generatorFunction()) {
  console.log(value);  // 1, 2, 3
}
```

### Передача значений в генератор

```javascript
function* generatorWithInput() {
  const input1 = yield 'Первое значение';
  console.log('Получено:', input1);
  
  const input2 = yield 'Второе значение';
  console.log('Получено:', input2);
  
  return 'Финальное значение';
}

const gen = generatorWithInput();

console.log(gen.next());           // { value: 'Первое значение', done: false }
console.log(gen.next('Ввод 1'));   // Получено: Ввод 1
                                   // { value: 'Второе значение', done: false }
console.log(gen.next('Ввод 2'));   // Получено: Ввод 2
                                   // { value: 'Финальное значение', done: true }
```

### Обработка ошибок в генераторах

```javascript
function* errorHandlingGenerator() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (error) {
    console.log('Ошибка в генераторе:', error.message);
    yield 'Ошибка обработана';
  }
}

const gen = errorHandlingGenerator();

console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }

// Передача ошибки в генератор
console.log(gen.throw(new Error('Произошла ошибка')));  
// Ошибка в генераторе: Произошла ошибка
// { value: 'Ошибка обработана', done: false }

console.log(gen.next());  // { value: undefined, done: true }
```

### Возврат значений из генераторов

```javascript
function* returnGenerator() {
  yield 1;
  yield 2;
  return 'Финальное значение';
  yield 3;  // Это значение не будет доступно
}

const gen = returnGenerator();

console.log(gen.next());  // { value: 1, done: false }
console.log(gen.next());  // { value: 2, done: false }
console.log(gen.next());  // { value: 'Финальное значение', done: true }
console.log(gen.next());  // { value: undefined, done: true }

// При использовании spread оператора return значение игнорируется
console.log([...returnGenerator()]);  // [1, 2]
```

## Продвинутое использование генераторов

### Делегирование генераторов

```javascript
function* generator1() {
  yield 1;
  yield 2;
}

function* generator2() {
  yield 3;
  yield 4;
}

function* combinedGenerator() {
  yield* generator1();  // Делегирование generator1
  yield* generator2();  // Делегирование generator2
  yield* [5, 6];        // Делегирование массиву
  yield* 'аб';          // Делегирование строке
}

for (const value of combinedGenerator()) {
  console.log(value);  // 1, 2, 3, 4, 5, 6, 'а', 'б'
}
```

### Асинхронные генераторы

```javascript
// Асинхронный генератор
async function* asyncGenerator() {
  yield await Promise.resolve(1);
  yield await Promise.resolve(2);
  yield await Promise.resolve(3);
}

// Использование асинхронного генератора
async function useAsyncGenerator() {
  for await (const value of asyncGenerator()) {
    console.log(value);  // 1, 2, 3
  }
}

useAsyncGenerator();

// Асинхронный итератор
const asyncIterable = {
  async *[Symbol.asyncIterator]() {
    yield await Promise.resolve('Первое значение');
    yield await Promise.resolve('Второе значение');
    yield await Promise.resolve('Третье значение');
  }
};

async function useAsyncIterable() {
  for await (const value of asyncIterable) {
    console.log(value);
  }
}

useAsyncIterable();
```

## Практические примеры

### Бесконечные последовательности

```javascript
// Генератор бесконечной последовательности чисел Фибоначчи
function* fibonacci() {
  let a = 0, b = 1;
  
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Получение первых 10 чисел Фибоначчи
const fib = fibonacci();
const firstTen = [];

for (let i = 0; i < 10; i++) {
  firstTen.push(fib.next().value);
}

console.log(firstTen);  // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

// Генератор случайных чисел
function* randomNumbers(min, max) {
  while (true) {
    yield Math.floor(Math.random() * (max - min + 1)) + min;
  }
}

const randomGen = randomNumbers(1, 100);
console.log(randomGen.next().value);  // Случайное число от 1 до 100
console.log(randomGen.next().value);  // Случайное число от 1 до 100
```

### Обработка данных с помощью генераторов

```javascript
// Генератор для обработки больших наборов данных
function* processData(data) {
  for (const item of data) {
    // Имитация обработки
    const processed = item.toUpperCase();
    yield processed;
  }
}

const data = ['привет', 'мир', 'javascript'];
const processor = processData(data);

for (const result of processor) {
  console.log(result);  // ПРИВЕТ, МИР, JAVASCRIPT
}

// Генератор для фильтрации данных
function* filterData(data, predicate) {
  for (const item of data) {
    if (predicate(item)) {
      yield item;
    }
  }
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const evenNumbers = filterData(numbers, x => x % 2 === 0);

console.log([...evenNumbers]);  // [2, 4, 6, 8, 10]
```

### Создание пользовательских итераторов

```javascript
// Итератор для диапазона чисел
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }
  
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i += this.step) {
      yield i;
    }
  }
}

const range = new Range(1, 10, 2);
console.log([...range]);  // [1, 3, 5, 7, 9]

for (const num of new Range(5, 15, 3)) {
  console.log(num);  // 5, 8, 11, 14
}

// Итератор для дерева
class TreeNode {
  constructor(value) {
    this.value = value;
    this.children = [];
  }
  
  addChild(child) {
    this.children.push(child);
  }
  
  // Итератор для обхода в глубину
  *[Symbol.iterator]() {
    yield this.value;
    
    for (const child of this.children) {
      yield* child;
    }
  }
}

// Создание дерева
const root = new TreeNode('root');
const child1 = new TreeNode('child1');
const child2 = new TreeNode('child2');
const grandchild1 = new TreeNode('grandchild1');

root.addChild(child1);
root.addChild(child2);
child1.addChild(grandchild1);

// Обход дерева
for (const value of root) {
  console.log(value);  // root, child1, grandchild1, child2
}
```

## Символы (Symbols)

Символы - это уникальные примитивные значения, которые могут использоваться как ключи свойств объектов.

### Основы символов

```javascript
// Создание символов
const symbol1 = Symbol();
const symbol2 = Symbol();

console.log(symbol1 === symbol2);  // false (каждый символ уникален)

// Символы с описанием
const symbolWithDescription = Symbol('описание');
console.log(symbolWithDescription.toString());  // "Symbol(описание)"

// Глобальные символы
const globalSymbol1 = Symbol.for('global');
const globalSymbol2 = Symbol.for('global');

console.log(globalSymbol1 === globalSymbol2);  // true (один и тот же символ)

// Получение ключа глобального символа
console.log(Symbol.keyFor(globalSymbol1));  // "global"
```

### Использование символов как ключей свойств

```javascript
const MY_KEY = Symbol('myKey');

const obj = {
  [MY_KEY]: 'Скрытое значение',
  normalProperty: 'Обычное значение'
};

console.log(obj[MY_KEY]);           // "Скрытое значение"
console.log(obj.normalProperty);    // "Обычное значение"

// Символы не перечисляются в for...in
for (const key in obj) {
  console.log(key);  // только "normalProperty"
}

// Символы не возвращаются Object.keys()
console.log(Object.keys(obj));     // ["normalProperty"]
console.log(Object.getOwnPropertySymbols(obj));  // [Symbol(myKey)]

// Получение всех свойств, включая символьные
console.log(Reflect.ownKeys(obj));  // ["normalProperty", Symbol(myKey)]
```

### Встроенные символы

```javascript
// Symbol.iterator - делает объект итерируемым
const iterableObj = {
  data: [1, 2, 3],
  
  [Symbol.iterator]() {
    let index = 0;
    return {
      next: () => {
        if (index < this.data.length) {
          return { value: this.data[index++], done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
};

// Symbol.toStringTag - настраивает поведение Object.prototype.toString()
class MyClass {
  get [Symbol.toStringTag]() {
    return 'MyCustomClass';
  }
}

const instance = new MyClass();
console.log(Object.prototype.toString.call(instance));  // "[object MyCustomClass]"

// Symbol.toPrimitive - преобразование объекта в примитив
class Temperature {
  constructor(degrees) {
    this.degrees = degrees;
  }
  
  [Symbol.toPrimitive](hint) {
    if (hint === 'string') {
      return `${this.degrees}°C`;
    } else if (hint === 'number') {
      return this.degrees;
    } else {
      // default
      return this.degrees;
    }
  }
}

const temp = new Temperature(25);
console.log(`${temp}`);        // "25°C"
console.log(temp + 5);         // 30
console.log(Number(temp));     // 25
```

## Лучшие практики

### 1. Используйте генераторы для работы с большими наборами данных

```javascript
// Хорошо: генератор для обработки больших файлов
function* processLargeFile(lines) {
  for (const line of lines) {
    // Обработка строки
    const processed = line.trim().toUpperCase();
    if (processed) {
      yield processed;
    }
  }
}

// Плохо: загрузка всего файла в память
function processLargeFileBad(lines) {
  return lines.map(line => line.trim().toUpperCase())
              .filter(line => line.length > 0);
}
```

### 2. Используйте символы для создания приватных свойств

```javascript
const _privateField = Symbol('privateField');

class MyClass {
  constructor(value) {
    this[_privateField] = value;
  }
  
  getPrivateValue() {
    return this[_privateField];
  }
}
```

### 3. Используйте асинхронные генераторы для потоковой обработки данных

```javascript
async function* fetchPaginatedData(url) {
  let nextPage = url;
  
  while (nextPage) {
    const response = await fetch(nextPage);
    const data = await response.json();
    
    yield* data.items;
    
    nextPage = data.nextPage;
  }
}

// Использование
async function processData() {
  for await (const item of fetchPaginatedData('/api/data')) {
    console.log(item);
  }
}
```

## Заключение

Итераторы и генераторы предоставляют мощные инструменты для работы с последовательностями данных в JavaScript. Они позволяют создавать эффективные, ленивые вычисления и улучшают читаемость кода. Символы добавляют дополнительный уровень абстракции и позволяют создавать действительно приватные свойства и специальные методы объектов.

Понимание этих концепций важно для написания современного JavaScript кода и работы с новыми API, которые используют эти возможности.

#javascript #es6 #generators #iterators #symbols #functional-programming