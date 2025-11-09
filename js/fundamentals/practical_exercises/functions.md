# Упражнения по функциям

## Упражнение 8: Создание и использование функций
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