---
aliases: ["Хитрости работы с числами в JavaScript", "Математические функции JS"]
tags: [js, numbers, math, tricks, frontend]
---

# Числа и математика: особенности и полезные функции

JavaScript предоставляет множество возможностей для работы с числами и математическими операциями. Ниже приведены важные особенности и полезные функции для эффективной работы с числами.

## 1. Проверка на число и тип числа

```javascript
// Проверка на конечное число
console.log(Number.isFinite(42)); // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isFinite('42')); // false

// Проверка на целое число
console.log(Number.isInteger(42)); // true
console.log(Number.isInteger(42.5)); // false
console.log(Number.isInteger('42')); // false

// Проверка на NaN (не число)
console.log(Number.isNaN(NaN)); // true
console.log(Number.isNaN('hello')); // false
console.log(isNaN('hello')); // true (в отличие от Number.isNaN)

// Проверка на безопасное целое число (без потери точности)
console.log(Number.isSafeInteger(2**53 - 1)); // true
console.log(Number.isSafeInteger(2**53)); // false
```

## 2. Округление чисел

```javascript
const num = 4.678;

// Округление до ближайшего целого
console.log(Math.round(num)); // 5

// Округление вниз
console.log(Math.floor(num)); // 4

// Округление вверх
console.log(Math.ceil(num)); // 5

// Обрезание дробной части
console.log(Math.trunc(num)); // 4

// Округление до определенного количества знаков после запятой
function roundToDecimal(num, decimals) {
  const factor = Math.pow(10, decimals);
  return Math.round(num * factor) / factor;
}

console.log(roundToDecimal(4.678, 2)); // 4.68
console.log(roundToDecimal(4.671, 2)); // 4.67

// Альтернатива через toFixed (возвращает строку)
console.log(parseFloat(4.678.toFixed(2))); // 4.68
```

## 3. Форматирование чисел

```javascript
const number = 1234567.89;

// Форматирование с разделителями тысяч
console.log(number.toLocaleString()); // '1,234,567.89' (в зависимости от локали)

// Форматирование с указанием локали
console.log(number.toLocaleString('ru-RU')); // '1 234 567,89'
console.log(number.toLocaleString('de-DE')); // '1.234.567,89'

// Форматирование с фиксированным количеством знаков после запятой
console.log(number.toFixed(2)); // '1234567.89'

// Форматирование как процент
const ratio = 0.85;
console.log(ratio.toLocaleString('ru-RU', { style: 'percent' })); // '85%'

// Форматирование валюты
console.log(number.toLocaleString('ru-RU', { 
  style: 'currency', 
  currency: 'RUB' 
})); // '1 234 567,89 ₽'

console.log(number.toLocaleString('en-US', { 
  style: 'currency', 
  currency: 'USD' 
})); // '$1,234,567.89'
```

## 4. Работа с плавающей точкой и точностью

```javascript
// Проблема точности при работе с дробными числами
console.log(0.1 + 0.2); // 0.30000000000000004

// Решение проблемы точности
function preciseAdd(a, b, decimals = 10) {
  return Math.round((a + b) * Math.pow(10, decimals)) / Math.pow(10, decimals);
}

console.log(preciseAdd(0.1, 0.2)); // 0.3

// Функция для сравнения чисел с учетом точности
function isEqual(a, b, epsilon = Number.EPSILON) {
  return Math.abs(a - b) < epsilon;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true

// Округление до ближайшего кратного
function roundToMultiple(num, multiple) {
  return Math.round(num / multiple) * multiple;
}

console.log(roundToMultiple(12.34, 0.5)); // 12.5
console.log(roundToMultiple(12.34, 5)); // 10
```

## 5. Генерация случайных чисел

```javascript
// Случайное число от 0 до 1
console.log(Math.random()); // например, 0.84423456789

// Случайное целое число в диапазоне [min, max]
function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

console.log(randomInt(1, 10)); // случайное число от 1 до 10

// Случайное число с плавающей точкой в диапазоне [min, max)
function randomFloat(min, max) {
  return Math.random() * (max - min) + min;
}

console.log(randomFloat(1.5, 5.7)); // например, 3.245

// Случайный элемент из массива
function randomItem(arr) {
  return arr[Math.floor(Math.random() * arr.length)];
}

console.log(randomItem(['red', 'green', 'blue'])); // например, 'green'

// Случайная строка из заданных символов
function randomString(length, chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789') {
  let result = '';
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return result;
}

console.log(randomString(8)); // например, 'aB3xK9mP'
```

## 6. Преобразование систем счисления

```javascript
// Преобразование в двоичную систему
console.log((42).toString(2)); // '101010'

// Преобразование в восьмеричную систему
console.log((42).toString(8)); // '52'

// Преобразование в шестнадцатеричную систему
console.log((255).toString(16)); // 'ff'

// Преобразование из других систем счисления
console.log(parseInt('101010', 2)); // 42 (двоичная)
console.log(parseInt('52', 8)); // 42 (восьмеричная)
console.log(parseInt('ff', 16)); // 255 (шестнадцатеричная)

// Альтернативные способы
console.log(+'0b101010'); // 42 (двоичная)
console.log(+'0o52'); // 42 (восьмеричная)
console.log(+'0xff'); // 255 (шестнадцатеричная)
```

## 7. Математические утилиты

```javascript
// Проверка диапазона
function isInRange(value, min, max) {
  return value >= min && value <= max;
}

// Ограничение значения диапазоном (clamp)
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

console.log(clamp(15, 5, 10)); // 10
console.log(clamp(3, 5, 10)); // 5
console.log(clamp(7, 5, 10)); // 7

// Линейная интерполяция
function lerp(start, end, factor) {
  return start + (end - start) * factor;
}

console.log(lerp(0, 100, 0.5)); // 50

// Отображение значения из одного диапазона в другой
function mapValue(value, fromMin, fromMax, toMin, toMax) {
  return toMin + (value - fromMin) * (toMax - toMin) / (fromMax - fromMin);
}

console.log(mapValue(50, 0, 100, 0, 10)); // 5

// Проверка четности/нечетности
function isEven(num) {
  return num % 2 === 0;
}

function isOdd(num) {
  return num % 2 !== 0;
}

console.log(isEven(4)); // true
console.log(isOdd(5)); // true
```

## 8. Работа с большими числами

```javascript
// BigInt для работы с очень большими целыми числами
const bigNumber = 1234567890123456789012345678901234567890n;
console.log(bigNumber); // 1234567890123456789012345678901234567890n

// Преобразование в BigInt
const regularNumber = 42;
const bigNumber2 = BigInt(regularNumber);
console.log(bigNumber2); // 42n

// Операции с BigInt
console.log(bigNumber + 100n); // 1234567890123456789012345678901234567990n

// ВНИМАНИЕ: BigInt нельзя использовать с обычными числами в одной операции
// console.log(bigNumber + 42); // Ошибка!
console.log(bigNumber + 42n); // Правильно
```

## 9. Формулы и вычисления

```javascript
// Вычисление расстояния между двумя точками
function distance(x1, y1, x2, y2) {
  return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
}

console.log(distance(0, 0, 3, 4)); // 5

// Вычисление факториала
function factorial(n) {
  if (n < 0) return NaN;
  if (n === 0 || n === 1) return 1;
  return n * factorial(n - 1);
}

// Альтернатива с циклом (для больших чисел)
function factorialIterative(n) {
  if (n < 0) return NaN;
  if (n === 0 || n === 1) return 1;
  
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

console.log(factorial(5)); // 120

// Проверка на простое число
function isPrime(num) {
  if (num <= 1) return false;
  if (num <= 3) return true;
  if (num % 2 === 0 || num % 3 === 0) return false;
  
  for (let i = 5; i * i <= num; i += 6) {
    if (num % i === 0 || num % (i + 2) === 0) return false;
  }
  return true;
}

console.log(isPrime(17)); // true
console.log(isPrime(18)); // false

// Генерация случайного цвета в HEX
function randomHexColor() {
  return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0');
}

console.log(randomHexColor()); // например, '#a3f2c1'
```

## 10. Работа с процентами

```javascript
// Вычисление процента от числа
function percentage(part, whole) {
  return (part / whole) * 100;
}

// Вычисление значения по проценту
function percentageValue(percent, whole) {
  return (percent / 100) * whole;
}

// Увеличение числа на процент
function increaseByPercent(value, percent) {
  return value * (1 + percent / 100);
}

// Уменьшение числа на процент
function decreaseByPercent(value, percent) {
  return value * (1 - percent / 100);
}

console.log(percentage(25, 100)); // 25
console.log(percentageValue(25, 200)); // 50
console.log(increaseByPercent(100, 20)); // 120
console.log(decreaseByPercent(100, 20)); // 80
```

## 11. Константы и специальные значения

```javascript
// Математические константы
console.log(Math.PI); // 3.141592653589793
console.log(Math.E); // 2.718281828459045
console.log(Math.LN2); // 0.6931471805599453
console.log(Math.LN10); // 2.302585092994046

// Специальные значения
console.log(Infinity); // Infinity
console.log(-Infinity); // -Infinity
console.log(NaN); // NaN

// Проверки специальных значений
console.log(isFinite(42)); // true
console.log(isFinite(Infinity)); // false
console.log(isNaN(NaN)); // true
```

## 12. Работа с углами и тригонометрией

```javascript
// Преобразование градусов в радианы
function degreesToRadians(degrees) {
  return degrees * (Math.PI / 180);
}

// Преобразование радианов в градусы
function radiansToDegrees(radians) {
  return radians * (180 / Math.PI);
}

console.log(degreesToRadians(180)); // 3.141592653589793 (PI)
console.log(radiansToDegrees(Math.PI)); // 180

// Пример использования тригонометрических функций
function getCoordinates(angle, radius) {
  const x = radius * Math.cos(degreesToRadians(angle));
  const y = radius * Math.sin(degreesToRadians(angle));
  return { x, y };
}

console.log(getCoordinates(45, 10)); // { x: 7.0710678118654755, y: 7.071067811865475 }
```

Эти математические функции и особенности работы с числами помогут вам создавать более точные и эффективные вычисления в ваших JavaScript приложениях.

См. также:
- [[js/tricks/arrays-tricks]] - Работа с массивами
- [[js/tricks/strings-tricks]] - Работа со строками
- [[js/tricks/objects-tricks]] - Работа с объектами