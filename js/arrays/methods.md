# Методы массивов в JavaScript

## Обзор

Массивы в JavaScript имеют множество встроенных методов, которые позволяют легко манипулировать данными. Эти методы можно разделить на мутирующие (изменяющие исходный массив) и немутирующие (возвращающие новый массив).

## Мутирующие методы

### push() и pop()

```javascript
const fruits = ['яблоко', 'банан'];
console.log(fruits.push('апельсин')); // 3 (новая длина массива)
console.log(fruits); // ['яблоко', 'банан', 'апельсин']

console.log(fruits.pop()); // 'апельсин' (удаленный элемент)
console.log(fruits); // ['яблоко', 'банан']
```

### shift() и unshift()

```javascript
const numbers = [2, 3, 4];
console.log(numbers.unshift(1)); // 4 (новая длина массива)
console.log(numbers); // [1, 2, 3, 4]

console.log(numbers.shift()); // 1 (удаленный элемент)
console.log(numbers); // [2, 3, 4]
```

### splice()

```javascript
const colors = ['красный', 'зеленый', 'синий', 'желтый'];
// Удаление элементов: начиная с индекса 1, удалить 2 элемента
const removed = colors.splice(1, 2);
console.log(removed); // ['зеленый', 'синий']
console.log(colors); // ['красный', 'желтый']

// Вставка элементов: начиная с индекса 1, удалить 0 элементов, добавить 'оранжевый' и 'фиолетовый'
colors.splice(1, 0, 'оранжевый', 'фиолетовый');
console.log(colors); // ['красный', 'оранжевый', 'фиолетовый', 'желтый']
```

### reverse() и sort()

```javascript
const numbers = [3, 1, 4, 1, 5];
numbers.reverse();
console.log(numbers); // [5, 1, 4, 1, 3]

const letters = ['c', 'a', 'b'];
letters.sort();
console.log(letters); // ['a', 'b', 'c']

// Сортировка чисел (по умолчанию сортирует как строки)
const nums = [10, 5, 40, 25, 1000, 1];
nums.sort((a, b) => a - b); // Сортировка по возрастанию
console.log(nums); // [1, 5, 10, 25, 40, 1000]
```

## Немутирующие методы

### concat()

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = arr1.concat(arr2);
console.log(combined); // [1, 2, 3, 4, 5, 6]
console.log(arr1); // [1, 2, 3] (не изменился)
```

### slice()

```javascript
const original = [1, 2, 3, 4, 5];
const subset = original.slice(1, 4); // с индекса 1 по 4 (не включая 4)
console.log(subset); // [2, 3, 4]
console.log(original); // [1, 2, 3, 4, 5] (не изменился)
```

### map()

```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8]
console.log(numbers); // [1, 2, 3, 4] (не изменился)
```

### filter()

```javascript
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6]
```

### reduce()

```javascript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0);
console.log(sum); // 10

// Более сложный пример: группировка объектов
const students = [
    { name: 'Иван', grade: 85 },
    { name: 'Мария', grade: 92 },
    { name: 'Алексей', grade: 78 }
];

const gradeGroups = students.reduce((groups, student) => {
    const letter = student.grade >= 90 ? 'A' : student.grade >= 80 ? 'B' : 'C';
    groups[letter] = groups[letter] || [];
    groups[letter].push(student);
    return groups;
}, {});

console.log(gradeGroups);
// {
//   B: [{ name: 'Иван', grade: 85 }],
//   A: [{ name: 'Мария', grade: 92 }],
//   C: [{ name: 'Алексей', grade: 78 }]
// }
```

## Методы для проверки условий

### every() и some()

```javascript
const numbers = [2, 4, 6, 8];
console.log(numbers.every(num => num % 2 === 0)); // true (все четные)
console.log(numbers.some(num => num > 5)); // true (есть число > 5)

const mixed = [2, 4, 6, 9];
console.log(mixed.every(num => num % 2 === 0)); // false (не все четные)
```

### find() и findIndex()

```javascript
const users = [
    { id: 1, name: 'Иван' },
    { id: 2, name: 'Мария' },
    { id: 3, name: 'Алексей' }
];

const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Мария' }

const index = users.findIndex(u => u.name === 'Алексей');
console.log(index); // 2
```

## Методы для поиска и включения

### includes()

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];
console.log(fruits.includes('банан')); // true
console.log(fruits.includes('груша')); // false
```

### indexOf() и lastIndexOf()

```javascript
const arr = [1, 2, 3, 2, 4];
console.log(arr.indexOf(2)); // 1 (первое вхождение)
console.log(arr.lastIndexOf(2)); // 3 (последнее вхождение)
console.log(arr.indexOf(5)); // -1 (не найдено)
```

## Дополнительные методы

### flat() и flatMap() (ES2019+)

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];
console.log(nested.flat()); // [1, 2, 3, 4, [5, 6]]
console.log(nested.flat(2)); // [1, 2, 3, 4, 5, 6] (уровень вложенности 2)

const words = ['hello world', 'javascript is awesome'];
const letters = words.flatMap(str => str.split(' '));
console.log(letters); // ['hello', 'world', 'javascript', 'is', 'awesome']
```

### join()

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];
console.log(fruits.join(', ')); // 'яблоко, банан, апельсин'
console.log(fruits.join(' - ')); // 'яблоко - банан - апельсин'
```