# Итерации по массивам в JavaScript

## Обзор

JavaScript предоставляет несколько способов итерации по массивам, каждый из которых подходит для разных сценариев. Выбор метода зависит от задачи: нужно ли изменить элементы, выполнить побочные эффекты или получить результат.

## Циклы для итерации

### for loop

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];

for (let i = 0; i < fruits.length; i++) {
    console.log(fruits[i]);
}
// Вывод: яблоко, банан, апельсин
```

### for...of

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];

for (const fruit of fruits) {
    console.log(fruit);
}
// Вывод: яблоко, банан, апельсин

// Получение индекса с помощью entries()
for (const [index, fruit] of fruits.entries()) {
    console.log(`${index}: ${fruit}`);
}
// Вывод: 0: яблоко, 1: банан, 2: апельсин
```

### for...in

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];

for (const index in fruits) {
    console.log(`${index}: ${fruits[index]}`);
}
// Вывод: 0: яблоко, 1: банан, 2: апельсин

// ВНИМАНИЕ: for...in перебирает все перечисляемые свойства, включая унаследованные
// Обычно for...of предпочтительнее для массивов
```

## Методы массивов для итерации

### forEach()

```javascript
const fruits = ['яблоко', 'банан', 'апельсин'];

fruits.forEach(fruit => {
    console.log(fruit);
});
// Вывод: яблоко, банан, апельсин

// С индексом
fruits.forEach((fruit, index) => {
    console.log(`${index}: ${fruit}`);
});
// Вывод: 0: яблоко, 1: банан, 2: апельсин

// forEach не возвращает значение, всегда undefined
const result = fruits.forEach(fruit => fruit.toUpperCase());
console.log(result); // undefined
```

### map()

```javascript
const numbers = [1, 2, 3, 4, 5];

// Создание нового массива с преобразованными элементами
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
console.log(numbers); // [1, 2, 3, 4, 5] (оригинал не изменен)

// Сложное преобразование
const users = [
    { firstName: 'Иван', lastName: 'Иванов' },
    { firstName: 'Мария', lastName: 'Петрова' }
];

const fullNames = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ['Иван Иванов', 'Мария Петрова']
```

### filter()

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Фильтрация четных чисел
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// Фильтрация строк по длине
const words = ['JavaScript', 'JS', 'Programming', 'Code'];
const longWords = words.filter(word => word.length > 6);
console.log(longWords); // ['JavaScript', 'Programming']
```

### reduce()

```javascript
const numbers = [1, 2, 3, 4, 5];

// Суммирование
const sum = numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0);
console.log(sum); // 15

// Поиск максимального значения
const max = numbers.reduce((acc, curr) => curr > acc ? curr : acc, 0);
console.log(max); // 5

// Преобразование массива объектов в один объект
const products = [
    { id: 1, name: 'Телефон', price: 500 },
    { id: 2, name: 'Планшет', price: 800 },
    { id: 3, name: 'Ноутбук', price: 1200 }
];

const productsById = products.reduce((acc, product) => {
    acc[product.id] = product;
    return acc;
}, {});
console.log(productsById);
// {
//   1: { id: 1, name: 'Телефон', price: 500 },
//   2: { id: 2, name: 'Планшет', price: 800 },
//   3: { id: 3, name: 'Ноутбук', price: 1200 }
// }
```

## Дополнительные методы итерации

### some() и every()

```javascript
const numbers = [2, 4, 6, 8, 10];

// Проверка, есть ли хотя бы один нечетный элемент
const hasOdd = numbers.some(num => num % 2 !== 0);
console.log(hasOdd); // false

// Проверка, все ли элементы четные
const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // true

// Практический пример: проверка валидности пользователей
const users = [
    { name: 'Иван', age: 25, isActive: true },
    { name: 'Мария', age: 30, isActive: true },
    { name: 'Алексей', age: 17, isActive: false }
];

const allAdults = users.every(user => user.age >= 18);
console.log(allAdults); // false

const hasActiveUsers = users.some(user => user.isActive);
console.log(hasActiveUsers); // true
```

### find() и findIndex()

```javascript
const users = [
    { id: 1, name: 'Иван', role: 'admin' },
    { id: 2, name: 'Мария', role: 'user' },
    { id: 3, name: 'Алексей', role: 'moderator' }
];

// Поиск первого пользователя с ролью 'admin'
const admin = users.find(user => user.role === 'admin');
console.log(admin); // { id: 1, name: 'Иван', role: 'admin' }

// Поиск индекса пользователя
const index = users.findIndex(user => user.name === 'Мария');
console.log(index); // 1
```

## Совместное использование методов

```javascript
const orders = [
    { id: 1, customer: 'Иван', amount: 150, status: 'completed' },
    { id: 2, customer: 'Мария', amount: 75, status: 'pending' },
    { id: 3, customer: 'Алексей', amount: 200, status: 'completed' },
    { id: 4, customer: 'Ольга', amount: 50, status: 'cancelled' }
];

// Найти завершенные заказы на сумму больше 100 и получить имена клиентов
const highValueCompletedCustomers = orders
    .filter(order => order.status === 'completed' && order.amount > 100)
    .map(order => order.customer);

console.log(highValueCompletedCustomers); // ['Иван', 'Алексей']

// Альтернативный способ с использованием reduce
const expensiveOrders = orders.reduce((result, order) => {
    if (order.status === 'completed' && order.amount > 100) {
        result.push(order.customer);
    }
    return result;
}, []);

console.log(expensiveOrders); // ['Иван', 'Алексей']
```

## Асинхронная итерация

```javascript
// Использование async/await с массивами
async function processArrayAsync(arr) {
    const results = [];
    for (const item of arr) {
        const result = await someAsyncOperation(item);
        results.push(result);
    }
    return results;
}

// Использование Promise.all() для параллельной обработки
async function processArrayParallel(arr) {
    const promises = arr.map(item => someAsyncOperation(item));
    return await Promise.all(promises);
}

// Заглушка для асинхронной операции
function someAsyncOperation(item) {
    return new Promise(resolve => {
        setTimeout(() => resolve(`Обработано: ${item}`), 100);
    });
}

// Пример использования
async function example() {
    const items = ['A', 'B', 'C'];
    const result = await processArrayAsync(items);
    console.log(result);
}
```

## Производительность и выбор метода

- `for` и `for...of` - самые быстрые для простых итераций
- `forEach()` - хорош для выполнения побочных эффектов
- `map()` - когда нужно создать новый массив на основе старого
- `filter()` - когда нужно получить подмножество элементов
- `reduce()` - для агрегации данных или сложных преобразований
- `find()` - для поиска первого подходящего элемента