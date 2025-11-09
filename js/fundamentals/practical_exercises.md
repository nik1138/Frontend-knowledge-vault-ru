# Практические упражнения по основам JavaScript

## Введение

Практические упражнения - важная часть изучения JavaScript. Они помогают закрепить теоретические знания и развить навыки программирования. В этом руководстве представлены упражнения по всем основным темам JavaScript.

## Упражнения по переменным

### Упражнение 1: Калькулятор переменных
Создайте программу, которая выполняет базовые операции с переменными:

```javascript
// Задание: Создайте переменные для хранения имени, возраста и города
// Выполните следующие операции:
// 1. Создайте переменные с вашими данными
// 2. Выведите их в консоль
// 3. Измените значения переменных
// 4. Создайте константу для хранения года рождения

let name = "Иван";
let age = 25;
let city = "Москва";
const birthYear = 1998;

console.log(`Меня зовут ${name}, мне ${age} лет, я живу в ${city}`);
console.log(`Я родился в ${birthYear} году`);
```

### Упражнение 2: Преобразование типов
Практикуйте преобразование типов данных:

```javascript
// Задание: Преобразуйте различные типы данных
// 1. Преобразуйте строку в число
// 2. Преобразуйте число в строку
// 3. Преобразуйте булево значение в число
// 4. Преобразуйте значение в булево

let str = "123";
let num = 456;
let bool = true;
let empty = null;

// Преобразование строк в числа
console.log(Number(str)); // 123
console.log(+str); // 123
console.log(parseInt("123.45")); // 123
console.log(parseFloat("123.45")); // 123.45

// Преобразование чисел в строки
console.log(String(num)); // "456"
console.log(num.toString()); // "456"
console.log("" + num); // "456"

// Преобразование в булевы значения
console.log(Boolean(str)); // true
console.log(Boolean(empty)); // false
console.log(!!str); // true
```

## Упражнения по типам данных

### Упражнение 3: Работа с массивами
Практикуйте операции с массивами:

```javascript
// Задание: Создайте массив и выполните различные операции
// 1. Создайте массив с элементами разных типов
// 2. Добавьте элементы в начало и конец массива
// 3. Удалите элементы из массива
// 4. Найдите элементы в массиве

let mixedArray = [1, "строка", true, null, {name: "Иван"}];

// Добавление элементов
mixedArray.push("новый элемент в конце");
mixedArray.unshift("новый элемент в начале");

// Удаление элементов
let lastElement = mixedArray.pop(); // Удаляет последний элемент
let firstElement = mixedArray.shift(); // Удаляет первый элемент

// Поиск элементов
let index = mixedArray.indexOf("строка");
let found = mixedArray.find(element => typeof element === "object");

console.log(mixedArray);
console.log("Индекс 'строка':", index);
console.log("Найденный объект:", found);
```

### Упражнение 4: Работа с объектами
Практикуйте операции с объектами:

```javascript
// Задание: Создайте объект пользователя и выполните операции
// 1. Создайте объект с данными пользователя
// 2. Добавьте и удалите свойства
// 3. Переберите свойства объекта
// 4. Проверьте наличие свойств

let user = {
  name: "Иван",
  age: 30,
  email: "ivan@example.com",
  isActive: true
};

// Добавление свойств
user.city = "Москва";
user["phone"] = "+7 (123) 456-78-90";

// Удаление свойств
delete user.isActive;

// Перебор свойств
for (let key in user) {
  console.log(`${key}: ${user[key]}`);
}

// Проверка наличия свойств
console.log("name" in user); // true
console.log("isActive" in user); // false
console.log(user.hasOwnProperty("age")); // true

// Деструктуризация объекта
const {name, age, ...otherProps} = user;
console.log(name, age, otherProps);
```

## Упражнения по операторам

### Упражнение 5: Математические операции
Практикуйте использование математических операторов:

```javascript
// Задание: Создайте калькулятор с различными операциями
// 1. Выполните базовые арифметические операции
// 2. Используйте операторы присваивания
// 3. Примените операторы сравнения
// 4. Используйте логические операторы

function calculator(a, b, operation) {
  switch (operation) {
    case '+':
      return a + b;
    case '-':
      return a - b;
    case '*':
      return a * b;
    case '/':
      return b !== 0 ? a / b : "Деление на ноль!";
    case '%':
      return a % b;
    case '**':
      return a ** b;
    default:
      return "Неизвестная операция";
  }
}

// Тестирование калькулятора
console.log(calculator(10, 5, '+')); // 15
console.log(calculator(10, 5, '-')); // 5
console.log(calculator(10, 5, '*')); // 50
console.log(calculator(10, 5, '/')); // 2
console.log(calculator(10, 3, '%')); // 1
console.log(calculator(2, 3, '**')); // 8

// Операторы присваивания
let x = 10;
x += 5; // x = x + 5
x -= 3; // x = x - 3
x *= 2; // x = x * 2
console.log(x); // 24

// Операторы сравнения
console.log(5 > 3); // true
console.log(5 === "5"); // false
console.log(5 == "5"); // true
console.log(null == undefined); // true
console.log(null === undefined); // false
```

## Упражнения по управляющим структурам

### Упражнение 6: Условные операторы
Практикуйте использование условных операторов:

```javascript
// Задание: Создайте программу для определения категории возраста
// 1. Используйте if/else
// 2. Используйте тернарный оператор
// 3. Используйте switch

function getAgeCategory(age) {
  // if/else
  if (age < 13) {
    return "ребенок";
  } else if (age < 20) {
    return "подросток";
  } else if (age < 65) {
    return "взрослый";
  } else {
    return "пенсионер";
  }
}

function getAgeCategoryTernary(age) {
  // Тернарный оператор
  return age < 13 ? "ребенок" : 
         age < 20 ? "подросток" : 
         age < 65 ? "взрослый" : "пенсионер";
}

function getAgeCategorySwitch(age) {
  // Switch с диапазонами
  const category = age < 13 ? 1 : 
                   age < 20 ? 2 : 
                   age < 65 ? 3 : 4;
  
  switch (category) {
    case 1:
      return "ребенок";
    case 2:
      return "подросток";
    case 3:
      return "взрослый";
    case 4:
      return "пенсионер";
    default:
      return "неизвестно";
  }
}

// Тестирование
console.log(getAgeCategory(8)); // ребенок
console.log(getAgeCategory(16)); // подросток
console.log(getAgeCategory(35)); // взрослый
console.log(getAgeCategory(70)); // пенсионер
```

### Упражнение 7: Циклы
Практикуйте использование различных типов циклов:

```javascript
// Задание: Создайте программы с использованием разных типов циклов
// 1. Используйте for
// 2. Используйте while
// 3. Используйте do/while
// 4. Используйте for...of и for...in

// Цикл for
function sumNumbers(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}

// Цикл while
function countdown(n) {
  let result = [];
  while (n > 0) {
    result.push(n);
    n--;
  }
  return result;
}

// Цикл do/while
function getRandomEven() {
  let num;
  do {
    num = Math.floor(Math.random() * 100);
  } while (num % 2 !== 0);
  return num;
}

// Цикл for...of
function processArray(arr) {
  let processed = [];
  for (let item of arr) {
    if (typeof item === 'number') {
      processed.push(item * 2);
    } else {
      processed.push(item);
    }
  }
  return processed;
}

// Цикл for...in
function getObjectKeys(obj) {
  let keys = [];
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      keys.push(key);
    }
  }
  return keys;
}

// Тестирование
console.log(sumNumbers(10)); // 55
console.log(countdown(5)); // [5, 4, 3, 2, 1]
console.log(getRandomEven()); // четное число
console.log(processArray([1, "hello", 3, "world"])); // [2, "hello", 6, "world"]
console.log(getObjectKeys({a: 1, b: 2, c: 3})); // ["a", "b", "c"]
```

## Упражнения по функциям

### Упражнение 8: Создание и использование функций
Практикуйте различные типы функций:

```javascript
// Задание: Создайте различные типы функций
// 1. Обычные функции
// 2. Функции-стрелки
// 3. Функции с параметрами по умолчанию
// 4. Рекурсивные функции

// Обычная функция
function greet(name) {
  return `Привет, ${name}!`;
}

// Функция-стрелка
const multiply = (a, b) => a * b;

// Функция с параметрами по умолчанию
function createUser(name, age = 18, city = "Не указан") {
  return {
    name: name,
    age: age,
    city: city,
    createdAt: new Date()
  };
}

// Рекурсивная функция
function factorial(n) {
  if (n <= 1) {
    return 1;
  }
  return n * factorial(n - 1);
}

// Функция высшего порядка
function calculatorFactory(operation) {
  switch (operation) {
    case 'add':
      return (a, b) => a + b;
    case 'subtract':
      return (a, b) => a - b;
    case 'multiply':
      return (a, b) => a * b;
    case 'divide':
      return (a, b) => b !== 0 ? a / b : "Деление на ноль!";
    default:
      return () => "Неизвестная операция";
  }
}

// Тестирование
console.log(greet("Иван")); // Привет, Иван!
console.log(multiply(5, 3)); // 15
console.log(createUser("Мария", 25)); // объект пользователя
console.log(factorial(5)); // 120

const add = calculatorFactory('add');
const divide = calculatorFactory('divide');
console.log(add(10, 5)); // 15
console.log(divide(10, 2)); // 5
```

## Упражнения по области видимости и замыканиям

### Упражнение 9: Область видимости
Практикуйте понимание области видимости:

```javascript
// Задание: Исследуйте различные области видимости
// 1. Глобальная область видимости
// 2. Локальная область видимости
// 3. Блочная область видимости
// 4. Замыкания

// Глобальная переменная
let globalVar = "Я глобальная переменная";

function scopeExample() {
  // Локальная переменная
  let localVar = "Я локальная переменная";
  
  if (true) {
    // Блочная переменная
    let blockVar = "Я блочная переменная";
    var functionVar = "Я функциональная переменная";
    
    console.log(globalVar); // Доступна
    console.log(localVar);  // Доступна
    console.log(blockVar);  // Доступна
    console.log(functionVar); // Доступна
  }
  
  console.log(globalVar); // Доступна
  console.log(localVar);  // Доступна
  // console.log(blockVar);  // Ошибка: не определена
  console.log(functionVar); // Доступна (var всплывает)
}

// Замыкание
function createCounter() {
  let count = 0;
  
  return function() {
    count++;
    return count;
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (независимый счетчик)
console.log(counter1()); // 3
```

### Упражнение 10: Практическое замыкание
Создайте практический пример использования замыкания:

```javascript
// Задание: Создайте модуль для управления списком задач
function createTaskManager() {
  let tasks = [];
  
  return {
    addTask: function(task) {
      tasks.push({
        id: Date.now(),
        text: task,
        completed: false,
        createdAt: new Date()
      });
    },
    
    completeTask: function(taskId) {
      const task = tasks.find(t => t.id === taskId);
      if (task) {
        task.completed = true;
      }
    },
    
    removeTask: function(taskId) {
      tasks = tasks.filter(t => t.id !== taskId);
    },
    
    getTasks: function() {
      return [...tasks]; // Возвращаем копию массива
    },
    
    getCompletedTasks: function() {
      return tasks.filter(t => t.completed);
    },
    
    getPendingTasks: function() {
      return tasks.filter(t => !t.completed);
    }
  };
}

// Использование
const taskManager = createTaskManager();

taskManager.addTask("Изучить JavaScript");
taskManager.addTask("Сделать упражнения");
taskManager.addTask("Написать проект");

console.log("Все задачи:", taskManager.getTasks());

taskManager.completeTask(taskManager.getTasks()[0].id);
console.log("Завершенные задачи:", taskManager.getCompletedTasks());
console.log("Ожидающие задачи:", taskManager.getPendingTasks());
```

## Комплексные упражнения

### Упражнение 11: Создание мини-приложения
Создайте небольшое приложение, объединяющее все изученные концепции:

```javascript
// Задание: Создайте приложение для управления контактами
class ContactManager {
  constructor() {
    this.contacts = [];
    this.nextId = 1;
  }
  
  // Добавление контакта
  addContact(name, phone, email) {
    if (!name || !phone) {
      throw new Error("Имя и телефон обязательны");
    }
    
    const contact = {
      id: this.nextId++,
      name: name.trim(),
      phone: phone.trim(),
      email: email ? email.trim() : "",
      createdAt: new Date()
    };
    
    this.contacts.push(contact);
    return contact;
  }
  
  // Поиск контактов
  findContacts(searchTerm) {
    if (!searchTerm) {
      return [...this.contacts];
    }
    
    const term = searchTerm.toLowerCase();
    return this.contacts.filter(contact => 
      contact.name.toLowerCase().includes(term) ||
      contact.phone.includes(term) ||
      contact.email.toLowerCase().includes(term)
    );
  }
  
  // Обновление контакта
  updateContact(id, updates) {
    const contact = this.contacts.find(c => c.id === id);
    if (!contact) {
      throw new Error("Контакт не найден");
    }
    
    Object.assign(contact, updates);
    return contact;
  }
  
  // Удаление контакта
  removeContact(id) {
    const index = this.contacts.findIndex(c => c.id === id);
    if (index === -1) {
      throw new Error("Контакт не найден");
    }
    
    return this.contacts.splice(index, 1)[0];
  }
  
  // Получение всех контактов
  getAllContacts() {
    return [...this.contacts];
  }
  
  // Статистика
  getStats() {
    return {
      total: this.contacts.length,
      withEmail: this.contacts.filter(c => c.email).length,
      createdToday: this.contacts.filter(c => 
        new Date(c.createdAt).toDateString() === new Date().toDateString()
      ).length
    };
  }
}

// Использование
const contacts = new ContactManager();

try {
  contacts.addContact("Иван Иванов", "+7 (123) 456-78-90", "ivan@example.com");
  contacts.addContact("Мария Петрова", "+7 (123) 456-78-91");
  contacts.addContact("Алексей Сидоров", "+7 (123) 456-78-92", "alex@example.com");
  
  console.log("Все контакты:", contacts.getAllContacts());
  console.log("Поиск 'Иван':", contacts.findContacts("Иван"));
  console.log("Статистика:", contacts.getStats());
  
  // Обновление контакта
  const firstContact = contacts.getAllContacts()[0];
  contacts.updateContact(firstContact.id, { email: "ivan.ivanov@example.com" });
  
  console.log("Обновленный контакт:", contacts.findContacts("Иван")[0]);
} catch (error) {
  console.error("Ошибка:", error.message);
}
```

## Рекомендации по выполнению упражнений

### 1. Последовательность выполнения
1. Начните с простых упражнений по каждой теме
2. Постепенно переходите к более сложным
3. Пишите код самостоятельно, не копируя
4. Тестируйте каждую функцию отдельно

### 2. Практические советы
```javascript
// Используйте console.log для отладки
function debugExample() {
  let x = 5;
  console.log("Значение x:", x);
  
  x *= 2;
  console.log("После умножения:", x);
  
  return x;
}

// Используйте try/catch для обработки ошибок
function safeOperation() {
  try {
    // Операция, которая может вызвать ошибку
    let result = riskyOperation();
    return result;
  } catch (error) {
    console.error("Произошла ошибка:", error.message);
    return null;
  }
}

// Используйте комментарии для объяснения кода
function wellDocumentedFunction(param1, param2) {
  // Проверка входных параметров
  if (!param1 || !param2) {
    throw new Error("Параметры обязательны");
  }
  
  // Основная логика функции
  const result = param1 + param2;
  
  // Возвращаем результат
  return result;
}
```

### 3. Самопроверка
- Код работает без ошибок?
- Результат соответствует ожиданиям?
- Код читаем и понятен?
- Использованы лучшие практики?

## Заключение

Практические упражнения - ключ к освоению JavaScript. Регулярная практика помогает:
- Закрепить теоретические знания
- Развить алгоритмическое мышление
- Научиться решать реальные задачи
- Повысить уверенность в своих навыках

Продолжайте практиковаться, экспериментируйте с кодом и не бойтесь ошибок - они являются частью процесса обучения.

#javascript #practice #exercises #learning #programming