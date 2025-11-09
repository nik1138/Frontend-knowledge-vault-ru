# Управляющие структуры в JavaScript: Условия и циклы

## Что такое управляющие структуры

Управляющие структуры в JavaScript позволяют контролировать поток выполнения программы. Они определяют, какие части кода будут выполнены и сколько раз, в зависимости от условий и других факторов.

## Условные операторы

### if...else

Оператор `if` выполняет код, если указанное условие истинно. Оператор `else` выполняет код, если условие ложно:

```javascript
let age = 20;

if (age >= 18) {
  console.log("Совершеннолетний");
} else {
  console.log("Несовершеннолетний");
}

// Множественные условия
let score = 85;

if (score >= 90) {
  console.log("Отлично");
} else if (score >= 70) {
  console.log("Хорошо");
} else if (score >= 50) {
  console.log("Удовлетворительно");
} else {
  console.log("Неудовлетворительно");
}
```

### Вложенные условия

```javascript
let temperature = 25;
let isRaining = false;

if (temperature > 20) {
  if (!isRaining) {
    console.log("Отличная погода для прогулки!");
  } else {
    console.log("Тепло, но идет дождь");
  }
} else {
  console.log("Слишком холодно для прогулки");
}
```

### switch

Оператор `switch` используется для выбора одного из нескольких вариантов выполнения кода:

```javascript
let day = 3;
let dayName;

switch (day) {
  case 1:
    dayName = "Понедельник";
    break;
  case 2:
    dayName = "Вторник";
    break;
  case 3:
    dayName = "Среда";
    break;
  case 4:
    dayName = "Четверг";
    break;
  case 5:
    dayName = "Пятница";
    break;
  case 6:
    dayName = "Суббота";
    break;
  case 7:
    dayName = "Воскресенье";
    break;
  default:
    dayName = "Неверный день";
}

console.log(dayName); // "Среда"

// Switch без break (сквозное выполнение)
let grade = "B";
let message;

switch (grade) {
  case "A":
  case "B":
  case "C":
    message = "Прошел";
    break;
  case "D":
  case "F":
    message = "Не прошел";
    break;
  default:
    message = "Неверная оценка";
}

console.log(message); // "Прошел"
```

## Тернарный оператор

Тернарный оператор - это сокращенная форма записи условного выражения:

```javascript
let age = 20;
let status = age >= 18 ? "Совершеннолетний" : "Несовершеннолетний";
console.log(status); // "Совершеннолетний"

// Вложенный тернарный оператор
let access = age >= 18 ? 
  (age >= 65 ? "Пенсионер" : "Взрослый") : 
  "Ребенок";
console.log(access); // "Взрослый"
```

## Циклы

### for

Цикл `for` используется, когда известно количество итераций:

```javascript
// Базовый for цикл
for (let i = 0; i < 5; i++) {
  console.log(`Итерация ${i}`);
}

// Вывод: Итерация 0, Итерация 1, Итерация 2, Итерация 3, Итерация 4

// Цикл с несколькими переменными
for (let i = 0, j = 10; i < 5; i++, j--) {
  console.log(`i = ${i}, j = ${j}`);
}

// Цикл без инициализации
let counter = 0;
for (; counter < 3; counter++) {
  console.log(`Счетчик: ${counter}`);
}

// Бесконечный цикл с прерыванием
for (let i = 0; ; i++) {
  if (i > 3) break;
  console.log(`Значение: ${i}`);
}
```

### for...in

Цикл `for...in` используется для перебора перечисляемых свойств объекта:

```javascript
let person = {
  name: "Иван",
  age: 30,
  city: "Москва"
};

for (let key in person) {
  console.log(`${key}: ${person[key]}`);
}

// Вывод:
// name: Иван
// age: 30
// city: Москва

// С массивами (не рекомендуется)
let colors = ["красный", "зеленый", "синий"];
for (let index in colors) {
  console.log(`${index}: ${colors[index]}`);
}
```

### for...of

Цикл `for...of` используется для перебора значений итерируемых объектов (массивов, строк и т.д.):

```javascript
// С массивами
let fruits = ["яблоко", "банан", "апельсин"];
for (let fruit of fruits) {
  console.log(fruit);
}

// Вывод: яблоко, банан, апельсин

// С строками
let message = "Привет";
for (let char of message) {
  console.log(char);
}

// Вывод: П, р, и, в, е, т

// С объектами (с использованием Object.values, Object.keys, Object.entries)
let user = {
  name: "Мария",
  age: 25,
  role: "developer"
};

for (let value of Object.values(user)) {
  console.log(value);
}

for (let key of Object.keys(user)) {
  console.log(key);
}

for (let [key, value] of Object.entries(user)) {
  console.log(`${key}: ${value}`);
}
```

### while

Цикл `while` выполняется, пока условие истинно:

```javascript
let count = 0;

while (count < 5) {
  console.log(`Счетчик: ${count}`);
  count++;
}

// Вывод: Счетчик: 0, Счетчик: 1, Счетчик: 2, Счетчик: 3, Счетчик: 4

// Цикл с постусловием
let number = 1;
while (number < 100) {
  number *= 2;
}
console.log(number); // 128
```

### do...while

Цикл `do...while` похож на `while`, но гарантирует выполнение тела цикла хотя бы один раз:

```javascript
let userInput;

do {
  userInput = prompt("Введите 'выход' для завершения:");
  console.log(`Вы ввели: ${userInput}`);
} while (userInput !== "выход");

// Этот цикл выполнится хотя бы один раз, даже если условие сразу ложно
let x = 5;
do {
  console.log("Это выполнится один раз");
  x++;
} while (x < 5);
```

## Управление потоком выполнения

### break

Оператор `break` используется для досрочного выхода из цикла или switch:

```javascript
// Выход из цикла
for (let i = 0; i < 10; i++) {
  if (i === 5) {
    break; // Выход из цикла при i = 5
  }
  console.log(i);
}

// Вывод: 0, 1, 2, 3, 4

// Выход из вложенного цикла с меткой
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break outer; // Выход из внешнего цикла
    }
    console.log(`i = ${i}, j = ${j}`);
  }
}
```

### continue

Оператор `continue` используется для пропуска оставшейся части текущей итерации цикла:

```javascript
// Пропуск четных чисел
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    continue; // Пропустить четные числа
  }
  console.log(i);
}

// Вывод: 1, 3, 5, 7, 9

// В while цикле
let num = 0;
while (num < 10) {
  num++;
  if (num % 2 === 0) {
    continue;
  }
  console.log(num);
}
```

## Практические примеры

### Поиск простых чисел

```javascript
function findPrimes(limit) {
  let primes = [];
  
  for (let num = 2; num <= limit; num++) {
    let isPrime = true;
    
    for (let i = 2; i <= Math.sqrt(num); i++) {
      if (num % i === 0) {
        isPrime = false;
        break;
      }
    }
    
    if (isPrime) {
      primes.push(num);
    }
  }
  
  return primes;
}

console.log(findPrimes(30)); // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

### Работа с массивами

```javascript
// Фильтрация массива с использованием цикла
function filterEvenNumbers(numbers) {
  let result = [];
  
  for (let number of numbers) {
    if (number % 2 === 0) {
      result.push(number);
    }
  }
  
  return result;
}

console.log(filterEvenNumbers([1, 2, 3, 4, 5, 6])); // [2, 4, 6]

// Поиск максимального значения
function findMax(arr) {
  if (arr.length === 0) return undefined;
  
  let max = arr[0];
  
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
      max = arr[i];
    }
  }
  
  return max;
}

console.log(findMax([3, 7, 2, 9, 1])); // 9
```

### Валидация данных

```javascript
function validateUserInput(input) {
  // Проверка на пустое значение
  if (!input) {
    return "Поле не может быть пустым";
  }
  
  // Проверка длины
  if (input.length < 3) {
    return "Значение должно содержать минимум 3 символа";
  }
  
  // Проверка на допустимые символы
  if (!/^[a-zA-Zа-яА-Я0-9_]+$/.test(input)) {
    return "Значение может содержать только буквы, цифры и подчеркивание";
  }
  
  return "Валидация пройдена";
}

console.log(validateUserInput("")); // "Поле не может быть пустым"
console.log(validateUserInput("ab")); // "Значение должно содержать минимум 3 символа"
console.log(validateUserInput("user@name")); // "Значение может содержать только буквы, цифры и подчеркивание"
console.log(validateUserInput("username")); // "Валидация пройдена"
```

### Меню с выбором действий

```javascript
function showMenu() {
  let choice;
  
  do {
    choice = prompt(`Выберите действие:
1. Сложение
2. Вычитание
3. Умножение
4. Деление
5. Выход`);

    switch (choice) {
      case "1":
        let a = parseFloat(prompt("Введите первое число:"));
        let b = parseFloat(prompt("Введите второе число:"));
        alert(`Результат: ${a + b}`);
        break;
      case "2":
        a = parseFloat(prompt("Введите первое число:"));
        b = parseFloat(prompt("Введите второе число:"));
        alert(`Результат: ${a - b}`);
        break;
      case "3":
        a = parseFloat(prompt("Введите первое число:"));
        b = parseFloat(prompt("Введите второе число:"));
        alert(`Результат: ${a * b}`);
        break;
      case "4":
        a = parseFloat(prompt("Введите первое число:"));
        b = parseFloat(prompt("Введите второе число:"));
        if (b !== 0) {
          alert(`Результат: ${a / b}`);
        } else {
          alert("Деление на ноль невозможно");
        }
        break;
      case "5":
        alert("До свидания!");
        break;
      default:
        alert("Неверный выбор");
    }
  } while (choice !== "5");
}
```

## Рекомендации по использованию

1. Используйте `for...of` для перебора значений массивов и других итерируемых объектов
2. Используйте `for...in` только для перебора свойств объектов
3. Используйте `for` цикл, когда известно количество итераций
4. Используйте `while` цикл, когда количество итераций неизвестно
5. Используйте `switch` вместо множественных `if...else`, когда проверяете одно значение на несколько условий
6. Используйте `break` и `continue` для управления потоком выполнения циклов

## Заключение

Управляющие структуры являются основой программирования на JavaScript. Правильное использование условных операторов и циклов позволяет создавать сложные программы с различными сценариями выполнения. Понимание этих конструкций критически важно для написания эффективного и читаемого кода.

#javascript #programming #control-structures #conditions #loops #fundamentals #web-development