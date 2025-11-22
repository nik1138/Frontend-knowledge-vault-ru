# Функции в JavaScript: Основы

## Что такое функции

Функции в JavaScript - это блоки кода, предназначенные для выполнения определенной задачи. Они позволяют структурировать код, избегать дублирования и создавать переиспользуемые компоненты.

## Объявление функций

### Обычное объявление функции

```javascript
// Объявление функции
function greet(name) {
  return `Привет, ${name}!`;
}

// Вызов функции
console.log(greet("Иван")); // "Привет, Иван!"
```

### Функциональное выражение

```javascript
// Функциональное выражение
const sayHello = function(name) {
  return `Привет, ${name}!`;
};

console.log(sayHello("Мария")); // "Привет, Мария!"

// Именованное функциональное выражение
const factorial = function fac(n) {
  return n < 2 ? 1 : n * fac(n - 1);
};

console.log(factorial(5)); // 120
```

### Стрелочные функции

```javascript
// Стрелочная функция с одним параметром
const square = x => x * x;
console.log(square(5)); // 25

// Стрелочная функция с несколькими параметрами
const add = (a, b) => a + b;
console.log(add(3, 4)); // 7

// Стрелочная функция без параметров
const greet = () => "Привет!";
console.log(greet()); // "Привет!"

// Стрелочная функция с телом в фигурных скобках
const multiply = (a, b) => {
  const result = a * b;
  return result;
};
console.log(multiply(3, 4)); // 12
```

## Параметры функций

### Параметры по умолчанию

```javascript
// Параметры по умолчанию (ES6)
function greet(name = "Гость", greeting = "Привет") {
  return `${greeting}, ${name}!`;
}

console.log(greet()); // "Привет, Гость!"
console.log(greet("Иван")); // "Привет, Иван!"
console.log(greet("Мария", "Здравствуйте")); // "Здравствуйте, Мария!"
```

### Оставшиеся параметры (Rest parameters)

```javascript
// Оставшиеся параметры
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// Комбинирование с обычными параметрами
function multiply(multiplier, ...numbers) {
  return numbers.map(num => num * multiplier);
}

console.log(multiply(2, 1, 2, 3)); // [2, 4, 6]
```

### Деструктуризация параметров

```javascript
// Деструктуризация объекта в параметрах
function createUser({ name, age, email }) {
  return {
    name: name,
    age: age,
    email: email,
    createdAt: new Date()
  };
}

const user = createUser({
  name: "Иван",
  age: 30,
  email: "ivan@example.com"
});

console.log(user);

// Деструктуризация массива в параметрах
function processCoordinates([x, y]) {
  return {
    x: x,
    y: y,
    distance: Math.sqrt(x * x + y * y)
  };
}

console.log(processCoordinates([3, 4])); // { x: 3, y: 4, distance: 5 }
```

## Возврат значений

### Возврат одного значения

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(2, 3)); // 5
```

### Возврат объектов и массивов

```javascript
// Возврат объекта
function createUser(name, age) {
  return {
    name: name,
    age: age,
    id: Math.random()
  };
}

const user = createUser("Иван", 30);
console.log(user);

// Возврат массива
function getMinMax(numbers) {
  return [Math.min(...numbers), Math.max(...numbers)];
}

const [min, max] = getMinMax([1, 5, 3, 9, 2]);
console.log(`Мин: ${min}, Макс: ${max}`); // "Мин: 1, Макс: 9"
```

### Функции без возврата значения

```javascript
// Функции без явного возврата возвращают undefined
function logMessage(message) {
  console.log(message);
}

const result = logMessage("Привет");
console.log(result); // undefined
```

## Область видимости и замыкания

### Локальные и глобальные переменные

```javascript
let globalVar = "Глобальная переменная";

function example() {
  let localVar = "Локальная переменная";
  
  console.log(globalVar); // Доступна внутри функции
  console.log(localVar); // Доступна внутри функции
}

example();
// console.log(localVar); // Ошибка! Переменная не доступна вне функции
```

### Замыкания

```javascript
// Замыкание
function outer(x) {
  return function inner(y) {
    return x + y;
  };
}

const add5 = outer(5);
console.log(add5(3)); // 8

// Практический пример замыкания
function createCounter() {
  let count = 0;
  
  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1
```

## Рекурсивные функции

```javascript
// Рекурсивная функция для вычисления факториала
function factorial(n) {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n - 1);
}

console.log(factorial(5)); // 120

// Рекурсивная функция для вычисления чисел Фибоначчи
function fibonacci(n) {
  if (n <= 1) {
    return n;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(7)); // 13
```

## Методы объектов

```javascript
// Методы объектов
const calculator = {
  add: function(a, b) {
    return a + b;
  },
  
  subtract: function(a, b) {
    return a - b;
  },
  
  multiply(a, b) { // Сокращенный синтаксис (ES6)
    return a * b;
  },
  
  divide: (a, b) => { // Стрелочная функция как метод
    if (b === 0) {
      throw new Error("Деление на ноль");
    }
    return a / b;
  }
};

console.log(calculator.add(5, 3)); // 8
console.log(calculator.multiply(4, 2)); // 8
```

## Callback функции

```javascript
// Callback функции
function processArray(arr, callback) {
  const result = [];
  for (let item of arr) {
    result.push(callback(item));
  }
  return result;
}

const numbers = [1, 2, 3, 4, 5];
const doubled = processArray(numbers, x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Асинхронный callback
function delayedGreeting(name, callback) {
  setTimeout(() => {
    callback(`Привет, ${name}!`);
  }, 1000);
}

delayedGreeting("Иван", message => {
  console.log(message); // "Привет, Иван!" (через 1 секунду)
});
```

## Практические примеры

### Валидация данных

```javascript
// Функция валидации email
function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// Функция валидации пароля
function isValidPassword(password) {
  return password.length >= 8 && 
         /[A-Z]/.test(password) && 
         /[a-z]/.test(password) && 
         /\d/.test(password);
}

// Комбинированная функция валидации
function validateUser(userData) {
  const errors = [];
  
  if (!userData.name || userData.name.trim().length === 0) {
    errors.push("Имя обязательно");
  }
  
  if (!isValidEmail(userData.email)) {
    errors.push("Неверный формат email");
  }
  
  if (!isValidPassword(userData.password)) {
    errors.push("Пароль должен содержать минимум 8 символов, включая заглавные и строчные буквы и цифры");
  }
  
  return {
    isValid: errors.length === 0,
    errors: errors
  };
}

const userData = {
  name: "Иван Иванов",
  email: "ivan@example.com",
  password: "Password123"
};

const validationResult = validateUser(userData);
console.log(validationResult);
```

### Работа с массивами

```javascript
// Утилиты для работы с массивами
const arrayUtils = {
  // Фильтрация четных чисел
  filterEven: function(numbers) {
    return numbers.filter(num => num % 2 === 0);
  },
  
  // Поиск максимального значения
  findMax: function(numbers) {
    if (numbers.length === 0) return undefined;
    return Math.max(...numbers);
  },
  
  // Группировка по свойству
  groupBy: function(arr, key) {
    return arr.reduce((groups, item) => {
      const group = item[key];
      groups[group] = groups[group] || [];
      groups[group].push(item);
      return groups;
    }, {});
  }
};

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(arrayUtils.filterEven(numbers)); // [2, 4, 6, 8, 10]
console.log(arrayUtils.findMax(numbers)); // 10

const users = [
  { name: "Иван", department: "IT" },
  { name: "Мария", department: "HR" },
  { name: "Алексей", department: "IT" },
  { name: "Ольга", department: "Finance" }
];

console.log(arrayUtils.groupBy(users, "department"));
```

### Фабрика объектов

```javascript
// Фабричная функция для создания пользователей
function createUser(name, email, role = "user") {
  // Приватный метод (через замыкание)
  const generateId = () => Math.random().toString(36).substr(2, 9);
  
  return {
    id: generateId(),
    name: name,
    email: email,
    role: role,
    createdAt: new Date(),
    
    // Методы
    getInfo: function() {
      return `${this.name} (${this.email}) - ${this.role}`;
    },
    
    isAdmin: function() {
      return this.role === "admin";
    },
    
    updateEmail: function(newEmail) {
      if (newEmail && newEmail.includes("@")) {
        this.email = newEmail;
        return true;
      }
      return false;
    }
  };
}

const user1 = createUser("Иван", "ivan@example.com");
const user2 = createUser("Мария", "maria@example.com", "admin");

console.log(user1.getInfo()); // "Иван (ivan@example.com) - user"
console.log(user2.isAdmin()); // true
```

## Рекомендации по использованию

1. Используйте стрелочные функции для коротких функций и callback'ов
2. Используйте обычные функции для методов объектов, когда нужен `this`
3. Используйте параметры по умолчанию вместо проверок внутри функции
4. Используйте rest параметры для функций с переменным количеством аргументов
5. Разбивайте сложные задачи на несколько небольших функций
6. Делайте функции чистыми, когда это возможно (одинаковый результат для одинаковых входных данных)

## Заключение

Функции являются одной из самых важных концепций в JavaScript. Они позволяют создавать модульный, переиспользуемый и поддерживаемый код. Понимание различных способов объявления функций, работы с параметрами и области видимости критически важно для эффективного программирования на JavaScript.

#javascript #programming #functions #fundamentals #web-development