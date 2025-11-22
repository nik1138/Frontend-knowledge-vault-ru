---
aliases: ["Хитрости работы с массивами в JavaScript", "Методы массивов JS"]
tags: [js, arrays, tricks, frontend]
---

# Хитрости работы с массивами в JavaScript

В JavaScript массивы - это мощная структура данных, которая предоставляет множество методов для манипуляции данными. Ниже приведены полезные хитрости и методы для эффективной работы с массивами.

## 1. Удаление дубликатов из массива

```javascript
// Использование Set
const arrayWithDuplicates = [1, 2, 2, 3, 4, 4, 5];
const uniqueArray = [...new Set(arrayWithDuplicates)];
console.log(uniqueArray); // [1, 2, 3, 4, 5]

// Использование filter и indexOf
const uniqueArray2 = arrayWithDuplicates.filter((item, index) => 
  arrayWithDuplicates.indexOf(item) === index
);
```

## 2. Проверка наличия элемента в массиве

```javascript
const fruits = ['apple', 'banana', 'orange'];

// Использование includes
console.log(fruits.includes('banana')); // true

// Использование some для сложных условий
const numbers = [1, 2, 3, 4, 5];
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true
```

## 3. Группировка элементов по свойству

```javascript
const people = [
  { name: 'Alice', age: 25, city: 'New York' },
  { name: 'Bob', age: 30, city: 'London' },
  { name: 'Charlie', age: 25, city: 'New York' },
  { name: 'David', age: 30, city: 'London' }
];

const groupedByCity = people.reduce((acc, person) => {
  const city = person.city;
  if (!acc[city]) {
    acc[city] = [];
  }
  acc[city].push(person);
  return acc;
}, {});

console.log(groupedByCity);
// {
//   'New York': [{ name: 'Alice', age: 25, city: 'New York' }, { name: 'Charlie', age: 25, city: 'New York' }],
//   'London': [{ name: 'Bob', age: 30, city: 'London' }, { name: 'David', age: 30, city: 'London' }]
// }
```

## 4. Создание массива с определенным размером и значением

```javascript
// Создание массива с определенным значением
const filledArray = new Array(5).fill(0); // [0, 0, 0, 0, 0]

// Создание массива с индексами
const indexArray = Array.from({ length: 5 }, (_, i) => i); // [0, 1, 2, 3, 4]

// Создание массива с вычисляемыми значениями
const squaredArray = Array.from({ length: 5 }, (_, i) => i * i); // [0, 1, 4, 9, 16]
```

## 5. Плоская распаковка вложенных массивов

```javascript
const nestedArray = [1, [2, 3], [4, [5, 6]], 7];
const flatArray = nestedArray.flat(); // [1, 2, 3, 4, [5, 6], 7]
const flatDeepArray = nestedArray.flat(Infinity); // [1, 2, 3, 4, 5, 6, 7]

// Использование flatMap для одновременного маппинга и распаковки
const words = ['hello world', 'foo bar', 'test'];
const allWords = words.flatMap(str => str.split(' ')); // ['hello', 'world', 'foo', 'bar', 'test']
```

## 6. Слияние и объединение массивов

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// Использование spread-оператора
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Использование concat
const concatenated = arr1.concat(arr2); // [1, 2, 3, 4, 5, 6]

// Условное добавление элементов
const condition = true;
const result = [...arr1, ...(condition ? arr2 : [])]; // [1, 2, 3, 4, 5, 6]
```

## 7. Поиск элементов с условиями

```javascript
const users = [
  { id: 1, name: 'Alice', active: true },
  { id: 2, name: 'Bob', active: false },
  { id: 3, name: 'Charlie', active: true }
];

// Поиск первого элемента, удовлетворяющего условию
const activeUser = users.find(user => user.active); // { id: 1, name: 'Alice', active: true }

// Поиск индекса элемента
const inactiveIndex = users.findIndex(user => !user.active); // 1

// Фильтрация по условию
const activeUsers = users.filter(user => user.active); // [{ id: 1, name: 'Alice', active: true }, { id: 3, name: 'Charlie', active: true }]
```

## 8. Преобразование массива в объект

```javascript
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
];

// Преобразование в объект с id в качестве ключей
const userById = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(userById);
// {
//   1: { id: 1, name: 'Alice' },
//   2: { id: 2, name: 'Bob' },
//   3: { id: 3, name: 'Charlie' }
// }

// Использование Object.fromEntries
const pairs = [['a', 1], ['b', 2], ['c', 3]];
const obj = Object.fromEntries(pairs); // { a: 1, b: 2, c: 3 }
```

## 9. Перемешивание массива (алгоритм Фишера-Йетса)

```javascript
function shuffleArray(array) {
  const result = [...array]; // создаем копию массива
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]]; // обмен элементов
  }
  return result;
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];
const shuffled = shuffleArray(numbers);
console.log(shuffled); // например, [3, 7, 1, 9, 2, 5, 8, 4, 6]
```

## 10. Группировка по нескольким критериям

```javascript
const products = [
  { name: 'Laptop', category: 'Electronics', price: 1000 },
  { name: 'Phone', category: 'Electronics', price: 500 },
  { name: 'Book', category: 'Education', price: 20 },
  { name: 'Tablet', category: 'Electronics', price: 300 }
];

// Группировка по категории, затем по цене
const grouped = products.reduce((acc, product) => {
  if (!acc[product.category]) {
    acc[product.category] = [];
  }
  acc[product.category].push(product);
  return acc;
}, {});

console.log(grouped);
```

## 11. Создание подмассивов (chunks)

```javascript
function chunkArray(array, chunkSize) {
  const chunks = [];
  for (let i = 0; i < array.length; i += chunkSize) {
    chunks.push(array.slice(i, i + chunkSize));
  }
  return chunks;
}

const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const chunked = chunkArray(items, 3);
console.log(chunked); // [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
```

## 12. Удаление ложных значений из массива

```javascript
const mixedArray = [0, 1, false, 2, '', 3, null, undefined, 4, 5, NaN];
const truthyValues = mixedArray.filter(Boolean); // [1, 2, 3, 4, 5]

// Или с использованием цикла
const truthyValues2 = mixedArray.filter(item => Boolean(item)); // [1, 2, 3, 4, 5]
```

## 13. Подсчет элементов по критериям

```javascript
const votes = ['yes', 'no', 'yes', 'maybe', 'yes', 'no'];
const voteCount = votes.reduce((acc, vote) => {
  acc[vote] = (acc[vote] || 0) + 1;
  return acc;
}, {});

console.log(voteCount); // { yes: 3, no: 2, maybe: 1 }
```

## 14. Удаление элементов по индексу

```javascript
const arr = [1, 2, 3, 4, 5];

// Удаление одного элемента по индексу
const indexToRemove = 2;
const newArr = [...arr.slice(0, indexToRemove), ...arr.slice(indexToRemove + 1)];
console.log(newArr); // [1, 2, 4, 5]

// Или с использованием splice (мутирует исходный массив)
const arr2 = [1, 2, 3, 4, 5];
arr2.splice(2, 1); // удаляет 1 элемент с индекса 2
console.log(arr2); // [1, 2, 4, 5]
```

## 15. Проверка на равенство массивов

```javascript
function arraysEqual(arr1, arr2) {
  if (arr1.length !== arr2.length) return false;
  
  for (let i = 0; i < arr1.length; i++) {
    if (arr1[i] !== arr2[i]) return false;
  }
  
  return true;
}

console.log(arraysEqual([1, 2, 3], [1, 2, 3])); // true
console.log(arraysEqual([1, 2, 3], [1, 2, 4])); // false
```

Эти хитрости помогут вам более эффективно работать с массивами в JavaScript, улучшая производительность и читаемость кода. Они особенно полезны при разработке frontend приложений, где часто требуется обработка и манипуляция коллекциями данных.

См. также:
- [[js/tricks/strings-tricks]] - Работа со строками
- [[js/tricks/numbers-tricks]] - Числа и математика
- [[js/tricks/objects-tricks]] - Работа с объектами