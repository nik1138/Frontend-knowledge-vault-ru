# Упражнения по операторам

## Упражнение 5: Математические операции
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