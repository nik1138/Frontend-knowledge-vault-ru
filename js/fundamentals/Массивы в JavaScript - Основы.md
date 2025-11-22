# Массивы в JavaScript: Основы

## Что такое массивы

Массивы в JavaScript - это упорядоченные коллекции значений, которые могут содержать элементы любого типа. Они являются специальным типом объектов и предоставляют множество методов для работы с данными.

## Создание массивов

### Литерал массива

```javascript
// Пустой массив
let emptyArray = [];

// Массив с элементами
let fruits = ["яблоко", "банан", "апельсин"];
let numbers = [1, 2, 3, 4, 5];
let mixed = [1, "строка", true, null, { name: "Иван" }];

console.log(fruits); // ["яблоко", "банан", "апельсин"]
console.log(numbers); // [1, 2, 3, 4, 5]
```

### Конструктор Array

```javascript
// Пустой массив
let arr1 = new Array();

// Массив с заданной длиной
let arr2 = new Array(5); // [empty × 5]

// Массив с элементами
let arr3 = new Array(1, 2, 3); // [1, 2, 3]

// Внимание: если передать одно число, создастся массив с заданной длиной
let arr4 = new Array(5); // [empty × 5]
let arr5 = new Array("5"); // ["5"] (строка)

console.log(arr2); // [empty × 5]
console.log(arr3); // [1, 2, 3]
```

### Array.of() и Array.from()

```javascript
// Array.of() - создает массив из переданных аргументов
let arr1 = Array.of(1); // [1]
let arr2 = Array.of(1, 2, 3); // [1, 2, 3]

// Array.from() - создает массив из массивоподобного объекта или итерируемого объекта
let arr3 = Array.from("hello"); // ["h", "e", "l", "l", "o"]
let arr4 = Array.from([1, 2, 3], x => x * 2); // [2, 4, 6]

// Создание массива из NodeList
let divs = document.querySelectorAll("div");
let divArray = Array.from(divs);
```

## Доступ к элементам массива

### Индексация

```javascript
let colors = ["красный", "зеленый", "синий"];

// Доступ к элементам по индексу (начинается с 0)
console.log(colors[0]); // "красный"
console.log(colors[1]); // "зеленый"
console.log(colors[2]); // "синий"

// Последний элемент
console.log(colors[colors.length - 1]); // "синий"

// Изменение элементов
colors[1] = "желтый";
console.log(colors); // ["красный", "желтый", "синий"]

// Добавление элементов по индексу
colors[3] = "фиолетовый";
console.log(colors); // ["красный", "желтый", "синий", "фиолетовый"]

// Доступ к несуществующему индексу
console.log(colors[10]); // undefined
```

## Свойства массивов

### length

```javascript
let numbers = [1, 2, 3, 4, 5];

console.log(numbers.length); // 5

// Изменение длины массива
numbers.length = 3;
console.log(numbers); // [1, 2, 3]

// Увеличение длины массива
numbers.length = 5;
console.log(numbers); // [1, 2, 3, empty × 2]

// Создание массива с заданной длиной
let emptyArr = new Array(10);
console.log(emptyArr.length); // 10
```

## Добавление и удаление элементов

### push() и unshift()

```javascript
let fruits = ["яблоко", "банан"];

// Добавление в конец массива
let newLength = fruits.push("апельсин");
console.log(fruits); // ["яблоко", "банан", "апельсин"]
console.log(newLength); // 3

// Добавление в начало массива
newLength = fruits.unshift("киви");
console.log(fruits); // ["киви", "яблоко", "банан", "апельсин"]
console.log(newLength); // 4

// Добавление нескольких элементов
fruits.push("груша", "виноград");
fruits.unshift("манго", "ананас");
console.log(fruits); // ["манго", "ананас", "киви", "яблоко", "банан", "апельсин", "груша", "виноград"]
```

### pop() и shift()

```javascript
let colors = ["красный", "зеленый", "синий"];

// Удаление последнего элемента
let lastColor = colors.pop();
console.log(lastColor); // "синий"
console.log(colors); // ["красный", "зеленый"]

// Удаление первого элемента
let firstColor = colors.shift();
console.log(firstColor); // "красный"
console.log(colors); // ["зеленый"]

// Возвращаемые значения
let emptyArray = [];
console.log(emptyArray.pop()); // undefined
console.log(emptyArray.shift()); // undefined
```

### splice()

```javascript
let numbers = [1, 2, 3, 4, 5];

// Удаление элементов: splice(начальный_индекс, количество_элементов)
let removed = numbers.splice(2, 2);
console.log(removed); // [3, 4]
console.log(numbers); // [1, 2, 5]

// Вставка элементов: splice(индекс, 0, элементы_для_вставки)
numbers.splice(2, 0, "новый", "элемент");
console.log(numbers); // [1, 2, "новый", "элемент", 5]

// Замена элементов: splice(индекс, количество_для_удаления, новые_элементы)
numbers.splice(1, 2, "заменено");
console.log(numbers); // [1, "заменено", "элемент", 5]
```

## Объединение и разделение массивов

### concat()

```javascript
let arr1 = [1, 2, 3];
let arr2 = [4, 5, 6];
let arr3 = [7, 8, 9];

// Объединение массивов
let combined = arr1.concat(arr2);
console.log(combined); // [1, 2, 3, 4, 5, 6]

// Объединение нескольких массивов
let all = arr1.concat(arr2, arr3);
console.log(all); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Объединение с отдельными элементами
let withElements = arr1.concat(10, 11, arr2);
console.log(withElements); // [1, 2, 3, 10, 11, 4, 5, 6]
```

### join()

```javascript
let fruits = ["яблоко", "банан", "апельсин"];

// Преобразование массива в строку
let str1 = fruits.join();
console.log(str1); // "яблоко,банан,апельсин"

// С пользовательским разделителем
let str2 = fruits.join(" - ");
console.log(str2); // "яблоко - банан - апельсин"

let str3 = fruits.join("");
console.log(str3); // "яблокобананапельсин"

// Пустой массив
let empty = [];
console.log(empty.join("-")); // ""
```

## Поиск элементов

### indexOf() и lastIndexOf()

```javascript
let numbers = [1, 2, 3, 2, 4, 2];

// Поиск первого вхождения
console.log(numbers.indexOf(2)); // 1
console.log(numbers.indexOf(5)); // -1 (не найдено)

// Поиск с заданной позиции
console.log(numbers.indexOf(2, 2)); // 3 (ищем 2 начиная с индекса 2)

// Поиск последнего вхождения
console.log(numbers.lastIndexOf(2)); // 5
console.log(numbers.lastIndexOf(2, 4)); // 3 (ищем 2 с конца до индекса 4)
```

### includes()

```javascript
let fruits = ["яблоко", "банан", "апельсин"];

console.log(fruits.includes("банан")); // true
console.log(fruits.includes("груша")); // false

// Поиск с заданной позиции
console.log(fruits.includes("банан", 2)); // false (ищем "банан" начиная с индекса 2)
```

### find() и findIndex()

```javascript
let users = [
  { id: 1, name: "Иван", age: 25 },
  { id: 2, name: "Мария", age: 30 },
  { id: 3, name: "Алексей", age: 20 }
];

// Поиск элемента по условию
let user = users.find(u => u.age > 25);
console.log(user); // { id: 2, name: "Мария", age: 30 }

// Поиск индекса элемента по условию
let index = users.findIndex(u => u.name === "Алексей");
console.log(index); // 2

// Если элемент не найден
let notFound = users.find(u => u.age > 50);
console.log(notFound); // undefined

let notFoundIndex = users.findIndex(u => u.name === "Неизвестный");
console.log(notFoundIndex); // -1
```

## Преобразование массивов

### slice()

```javascript
let numbers = [1, 2, 3, 4, 5];

// Создание копии массива
let copy = numbers.slice();
console.log(copy); // [1, 2, 3, 4, 5]

// Получение части массива
let part = numbers.slice(1, 4); // с индекса 1 по 4 (не включая 4)
console.log(part); // [2, 3, 4]

// Отрицательные индексы
let lastTwo = numbers.slice(-2);
console.log(lastTwo); // [4, 5]

let middle = numbers.slice(1, -1);
console.log(middle); // [2, 3, 4]
```

### reverse()

```javascript
let numbers = [1, 2, 3, 4, 5];

// Изменяет исходный массив
numbers.reverse();
console.log(numbers); // [5, 4, 3, 2, 1]

// Для создания копии без изменения оригинала
let original = [1, 2, 3, 4, 5];
let reversed = [...original].reverse();
console.log(original); // [1, 2, 3, 4, 5] (не изменился)
console.log(reversed); // [5, 4, 3, 2, 1]
```

### sort()

```javascript
let fruits = ["банан", "апельсин", "яблоко", "манго"];

// Сортировка строк (по умолчанию)
fruits.sort();
console.log(fruits); // ["апельсин", "банан", "манго", "яблоко"]

// Сортировка чисел (требует функции сравнения)
let numbers = [10, 5, 40, 25, 1000, 1];
numbers.sort();
console.log(numbers); // [1, 10, 1000, 25, 40, 5] (лексикографическая сортировка)

// Правильная сортировка чисел
numbers.sort((a, b) => a - b); // по возрастанию
console.log(numbers); // [1, 5, 10, 25, 40, 1000]

numbers.sort((a, b) => b - a); // по убыванию
console.log(numbers); // [1000, 40, 25, 10, 5, 1]

// Сортировка объектов
let people = [
  { name: "Иван", age: 30 },
  { name: "Мария", age: 25 },
  { name: "Алексей", age: 35 }
];

people.sort((a, b) => a.age - b.age);
console.log(people); // отсортировано по возрасту
```

## Практические примеры

### Работа с корзиной покупок

```javascript
// Управление корзиной покупок
function createShoppingCart() {
  let items = [];
  
  return {
    // Добавление товара
    addItem: function(product, quantity = 1) {
      const existingItem = items.find(item => item.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        items.push({
          ...product,
          quantity: quantity
        });
      }
    },
    
    // Удаление товара
    removeItem: function(productId) {
      items = items.filter(item => item.id !== productId);
    },
    
    // Обновление количества
    updateQuantity: function(productId, quantity) {
      const item = items.find(item => item.id === productId);
      if (item) {
        if (quantity <= 0) {
          this.removeItem(productId);
        } else {
          item.quantity = quantity;
        }
      }
    },
    
    // Получение всех товаров
    getItems: function() {
      return [...items]; // возвращаем копию
    },
    
    // Получение общего количества товаров
    getTotalItems: function() {
      return items.reduce((total, item) => total + item.quantity, 0);
    },
    
    // Получение общей стоимости
    getTotalPrice: function() {
      return items.reduce((total, item) => total + (item.price * item.quantity), 0);
    },
    
    // Очистка корзины
    clear: function() {
      items = [];
    }
  };
}

// Использование
const cart = createShoppingCart();

const products = [
  { id: 1, name: "Ноутбук", price: 50000 },
  { id: 2, name: "Мышь", price: 1000 },
  { id: 3, name: "Клавиатура", price: 2000 }
];

cart.addItem(products[0], 1);
cart.addItem(products[1], 2);
cart.addItem(products[2], 1);

console.log("Товаров:", cart.getTotalItems()); // 4
console.log("Стоимость:", cart.getTotalPrice()); // 54000
```

### Фильтрация и сортировка данных

```javascript
// Работа с коллекцией пользователей
const users = [
  { id: 1, name: "Иван", age: 25, department: "IT", salary: 50000 },
  { id: 2, name: "Мария", age: 30, department: "HR", salary: 45000 },
  { id: 3, name: "Алексей", age: 35, department: "IT", salary: 70000 },
  { id: 4, name: "Ольга", age: 28, department: "Finance", salary: 55000 },
  { id: 5, name: "Дмитрий", age: 32, department: "IT", salary: 65000 }
];

// Фильтрация пользователей по отделу
const itUsers = users.filter(user => user.department === "IT");
console.log("Пользователи IT:", itUsers);

// Сортировка по возрасту
const sortedByAge = [...users].sort((a, b) => a.age - b.age);
console.log("По возрасту:", sortedByAge);

// Сортировка по зарплате (по убыванию)
const sortedBySalary = [...users].sort((a, b) => b.salary - a.salary);
console.log("По зарплате:", sortedBySalary);

// Поиск пользователей с зарплатой выше средней
const averageSalary = users.reduce((sum, user) => sum + user.salary, 0) / users.length;
const highEarners = users.filter(user => user.salary > averageSalary);
console.log("Высокие зарплаты:", highEarners);

// Группировка по отделам
const groupedByDepartment = users.reduce((groups, user) => {
  groups[user.department] = groups[user.department] || [];
  groups[user.department].push(user);
  return groups;
}, {});

console.log("По отделам:", groupedByDepartment);
```

### Работа с матрицами

```javascript
// Создание и работа с двумерными массивами (матрицами)
function createMatrix(rows, cols, initialValue = 0) {
  return Array(rows).fill().map(() => Array(cols).fill(initialValue));
}

// Создание матрицы 3x3
let matrix = createMatrix(3, 3, 0);
console.log(matrix);
// [[0, 0, 0], [0, 0, 0], [0, 0, 0]]

// Заполнение матрицы
for (let i = 0; i < matrix.length; i++) {
  for (let j = 0; j < matrix[i].length; j++) {
    matrix[i][j] = i * matrix[i].length + j + 1;
  }
}

console.log(matrix);
// [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// Транспонирование матрицы
function transpose(matrix) {
  return matrix[0].map((_, colIndex) => matrix.map(row => row[colIndex]));
}

let transposed = transpose(matrix);
console.log("Транспонированная:", transposed);
// [[1, 4, 7], [2, 5, 8], [3, 6, 9]]

// Сумма элементов матрицы
function matrixSum(matrix) {
  return matrix.reduce((total, row) => 
    total + row.reduce((rowSum, cell) => rowSum + cell, 0), 0);
}

console.log("Сумма:", matrixSum(matrix)); // 45
```

## Рекомендации по использованию

1. Используйте литерал массива `[]` для создания массивов
2. Используйте `push()` и `pop()` для работы с концом массива (более эффективно)
3. Используйте `unshift()` и `shift()` для работы с началом массива (менее эффективно)
4. Используйте `splice()` для вставки/удаления элементов в середине массива
5. Используйте `slice()` для создания копий массивов
6. Используйте `concat()` для объединения массивов без изменения оригиналов
7. Используйте `map()`, `filter()`, `reduce()` для функциональной работы с массивами
8. Используйте spread оператор `[...array]` для создания поверхностных копий

## Заключение

Массивы являются одной из самых важных структур данных в JavaScript. Они предоставляют мощные методы для хранения, поиска, сортировки и преобразования данных. Понимание работы с массивами критически важно для эффективного программирования на JavaScript.

#javascript #programming #arrays #fundamentals #web-development