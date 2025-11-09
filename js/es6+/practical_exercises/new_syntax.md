# Практические упражнения: Новые синтаксические возможности ES6+

## Упражнение 1: let/const вместо var

### Задача
Перепишите следующий код, заменив var на let/const где это необходимо:

```javascript
var userName = "John";
var userAge = 25;

for (var i = 0; i < 3; i++) {
  var message = "Попытка " + (i + 1);
  console.log(message);
}

function getUserInfo() {
  var userName = "Jane";
  var userAge = 30;
  return userName + " (" + userAge + ")";
}

console.log(getUserInfo());
console.log(userName);
```

### Требования
1. Используйте const для неизменяемых переменных
2. Используйте let для переменных, которые изменяются
3. Обеспечьте правильную область видимости переменных

### Решение
```javascript
const userName = "John";
const userAge = 25;

for (let i = 0; i < 3; i++) {
  const message = "Попытка " + (i + 1);
  console.log(message);
}

function getUserInfo() {
  const userName = "Jane";
  const userAge = 30;
  return userName + " (" + userAge + ")";
}

console.log(getUserInfo());
console.log(userName);
```

## Упражнение 2: Деструктуризация объектов

### Задача
Используя деструктуризацию, извлеките свойства name, age и city из объекта пользователя и создайте переменные с именами userName, userAge, userCity:

```javascript
const user = {
  name: "Alice",
  age: 28,
  email: "alice@example.com",
  city: "New York",
  country: "USA"
};
```

### Требования
1. Используйте деструктуризацию с переименованием переменных
2. Установите значение по умолчанию для свойства country ("Unknown")
3. Извлеките только указанные свойства

### Решение
```javascript
const { name: userName, age: userAge, city: userCity, country = "Unknown" } = user;

console.log(userName); // Alice
console.log(userAge);  // 28
console.log(userCity); // New York
console.log(country);  // USA
```

## Упражнение 3: Деструктуризация массивов

### Задача
Используя деструктуризацию массивов, извлеките первые три элемента массива и остальные элементы в отдельную переменную:

```javascript
const colors = ["red", "green", "blue", "yellow", "purple", "orange"];
```

### Требования
1. Извлеките первые три элемента в переменные first, second, third
2. Остальные элементы соберите в массив rest
3. Пропустите второй элемент при деструктуризации

### Решение
```javascript
const [first, , second, third, ...rest] = colors;

console.log(first);  // red
console.log(second); // blue
console.log(third);  // yellow
console.log(rest);   // ["purple", "orange"]
```

## Упражнение 4: Шаблонные строки

### Задача
Перепишите следующий код, используя шаблонные строки вместо конкатенации:

```javascript
const product = "Laptop";
const price = 999;
const discount = 15;
const tax = 8.5;

const message = "Product: " + product + "\n" +
                "Price: $" + price + "\n" +
                "Discount: " + discount + "%\n" +
                "Total: $" + (price * (1 - discount/100) * (1 + tax/100)).toFixed(2);
```

### Требования
1. Используйте шаблонные строки с многострочным форматированием
2. Вставьте выражения непосредственно в строку
3. Сохраните форматирование вывода

### Решение
```javascript
const product = "Laptop";
const price = 999;
const discount = 15;
const tax = 8.5;

const message = `Product: ${product}
Price: $${price}
Discount: ${discount}%
Total: $${(price * (1 - discount/100) * (1 + tax/100)).toFixed(2)}`;

console.log(message);
```

## Упражнение 5: Spread/Rest операторы

### Задача
Создайте функцию mergeArrays, которая принимает любое количество массивов и возвращает объединенный массив без дубликатов:

### Требования
1. Используйте rest параметры для приема массивов
2. Используйте spread оператор для объединения массивов
3. Удалите дубликаты из результата

### Решение
```javascript
function mergeArrays(...arrays) {
  const merged = [].concat(...arrays);
  return [...new Set(merged)];
}

const arr1 = [1, 2, 3];
const arr2 = [3, 4, 5];
const arr3 = [5, 6, 7];

console.log(mergeArrays(arr1, arr2, arr3)); // [1, 2, 3, 4, 5, 6, 7]
```

## Упражнение 6: Стрелочные функции

### Задача
Перепишите следующие функции в виде стрелочных функций:

```javascript
function calculateArea(width, height) {
  return width * height;
}

function greetUser(name) {
  return "Hello, " + name + "!";
}

function isEven(number) {
  if (number % 2 === 0) {
    return true;
  } else {
    return false;
  }
}

const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(function(num) {
  return num * 2;
});
```

### Требования
1. Используйте краткий синтаксис для простых функций
2. Используйте блочный синтаксис для функций с несколькими операторами
3. Примените стрелочные функции в методе массива

### Решение
```javascript
const calculateArea = (width, height) => width * height;

const greetUser = name => `Hello, ${name}!`;

const isEven = number => number % 2 === 0;

const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);

console.log(calculateArea(5, 3)); // 15
console.log(greetUser("Alice"));  // Hello, Alice!
console.log(isEven(4));           // true
console.log(doubled);             // [2, 4, 6, 8, 10]
```

## Упражнение 7: Классы

### Задача
Создайте класс BankAccount с методами deposit, withdraw и getBalance. Добавьте наследование для класса SavingsAccount с процентной ставкой:

### Требования
1. Базовый класс BankAccount с приватным балансом
2. Методы для пополнения, снятия и проверки баланса
3. Класс SavingsAccount, наследующий от BankAccount
4. Метод для начисления процентов в SavingsAccount

### Решение
```javascript
class BankAccount {
  #balance = 0;
  
  constructor(initialBalance = 0) {
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      return this.#balance;
    }
    throw new Error("Сумма должна быть положительной");
  }
  
  withdraw(amount) {
    if (amount > 0 && amount <= this.#balance) {
      this.#balance -= amount;
      return this.#balance;
    }
    throw new Error("Недостаточно средств или неверная сумма");
  }
  
  getBalance() {
    return this.#balance;
  }
}

class SavingsAccount extends BankAccount {
  constructor(initialBalance, interestRate) {
    super(initialBalance);
    this.interestRate = interestRate;
  }
  
  addInterest() {
    const interest = this.getBalance() * this.interestRate / 100;
    this.deposit(interest);
    return interest;
  }
}

// Пример использования
const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500

const savings = new SavingsAccount(1000, 5);
savings.addInterest();
console.log(savings.getBalance()); // 1050
```

## Упражнение 8: Модули

### Задача
Создайте модуль calculator.js с функциями сложения, вычитания, умножения и деления, и импортируйте их в main.js:

### Требования
1. Создайте модуль calculator.js с именованными экспортами
2. Создайте файл main.js с импортом функций
3. Продемонстрируйте использование импортированных функций

### Решение

calculator.js:
```javascript
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => {
  if (b === 0) throw new Error("Деление на ноль");
  return a / b;
};

export default function calculate(operation, a, b) {
  switch(operation) {
    case 'add': return add(a, b);
    case 'subtract': return subtract(a, b);
    case 'multiply': return multiply(a, b);
    case 'divide': return divide(a, b);
    default: throw new Error("Неизвестная операция");
  }
}
```

main.js:
```javascript
import calculate, { add, subtract, multiply, divide } from './calculator.js';

console.log(add(5, 3));        // 8
console.log(subtract(10, 4));  // 6
console.log(multiply(3, 7));   // 21
console.log(divide(15, 3));    // 5
console.log(calculate('add', 2, 3)); // 5