# Операторы в JavaScript: Основы

## Что такое операторы

Операторы в JavaScript - это специальные символы или ключевые слова, которые выполняют операции над операндами (переменными и значениями). Они позволяют выполнять математические вычисления, сравнения, логические операции и другие действия.

## Арифметические операторы

Арифметические операторы используются для выполнения математических операций:

```javascript
let a = 10;
let b = 3;

// Сложение
console.log(a + b); // 13

// Вычитание
console.log(a - b); // 7

// Умножение
console.log(a * b); // 30

// Деление
console.log(a / b); // 3.3333333333333335

// Остаток от деления
console.log(a % b); // 1

// Возведение в степень (ES2016)
console.log(a ** b); // 1000

// Инкремент (увеличение на 1)
let counter = 1;
console.log(++counter); // 2 (префиксный инкремент)
console.log(counter++); // 2 (постфиксный инкремент)
console.log(counter); // 3

// Декремент (уменьшение на 1)
let count = 5;
console.log(--count); // 4 (префиксный декремент)
console.log(count--); // 4 (постфиксный декремент)
console.log(count); // 3
```

## Операторы присваивания

Операторы присваивания используются для присвоения значений переменным:

```javascript
let x = 10;

// Простое присваивание
x = 5; // x теперь равно 5

// Присваивание с сложением
x += 3; // эквивалентно x = x + 3; x теперь равно 8

// Присваивание с вычитанием
x -= 2; // эквивалентно x = x - 2; x теперь равно 6

// Присваивание с умножением
x *= 4; // эквивалентно x = x * 4; x теперь равно 24

// Присваивание с делением
x /= 3; // эквивалентно x = x / 3; x теперь равно 8

// Присваивание с остатком
x %= 5; // эквивалентно x = x % 5; x теперь равно 3

// Присваивание со степенью
x **= 2; // эквивалентно x = x ** 2; x теперь равно 9
```

## Операторы сравнения

Операторы сравнения используются для сравнения двух значений и возвращают булево значение (true или false):

```javascript
let a = 5;
let b = 10;

// Равно (не строгое сравнение)
console.log(a == b); // false
console.log(a == "5"); // true (происходит преобразование типов)

// Строгое равно
console.log(a === b); // false
console.log(a === "5"); // false (разные типы)
console.log(a === 5); // true

// Не равно (не строгое сравнение)
console.log(a != b); // true
console.log(a != "5"); // false

// Строгое не равно
console.log(a !== b); // true
console.log(a !== "5"); // true
console.log(a !== 5); // false

// Больше
console.log(a > b); // false
console.log(b > a); // true

// Меньше
console.log(a < b); // true
console.log(b < a); // false

// Больше или равно
console.log(a >= b); // false
console.log(a >= 5); // true

// Меньше или равно
console.log(a <= b); // true
console.log(a <= 5); // true
```

## Логические операторы

Логические операторы используются для выполнения логических операций:

```javascript
let isTrue = true;
let isFalse = false;

// Логическое И (&&)
console.log(isTrue && isFalse); // false
console.log(isTrue && true); // true

// Логическое ИЛИ (||)
console.log(isTrue || isFalse); // true
console.log(false || false); // false

// Логическое НЕ (!)
console.log(!isTrue); // false
console.log(!isFalse); // true

// Примеры использования в условных выражениях
let age = 25;
let hasLicense = true;

if (age >= 18 && hasLicense) {
  console.log("Можно водить машину");
}

let isWeekend = true;
let isHoliday = false;

if (isWeekend || isHoliday) {
  console.log("Можно отдыхать");
}
```

## Операторы строк

Операторы строк используются для работы со строками:

```javascript
let firstName = "Иван";
let lastName = "Иванов";

// Конкатенация строк
let fullName = firstName + " " + lastName; // "Иван Иванов"

// Конкатенация с присваиванием
let greeting = "Привет, ";
greeting += firstName; // "Привет, Иван"

// Сравнение строк
console.log("a" < "b"); // true
console.log("Я" > "А"); // true
```

## Условный (тернарный) оператор

Тернарный оператор - это сокращенная форма записи условного выражения:

```javascript
let age = 20;

// Полная форма if-else
let message;
if (age >= 18) {
  message = "Совершеннолетний";
} else {
  message = "Несовершеннолетний";
}

// Тернарный оператор
let message2 = age >= 18 ? "Совершеннолетний" : "Несовершеннолетний";

console.log(message); // "Совершеннолетний"
console.log(message2); // "Совершеннолетний"

// Вложенный тернарный оператор
let access = age >= 18 ? (age >= 65 ? "Пенсионер" : "Взрослый") : "Ребенок";
console.log(access); // "Взрослый"
```

## Операторы типа

Операторы типа используются для определения типа данных:

```javascript
// typeof - возвращает строку с типом данных
console.log(typeof 42); // "number"
console.log(typeof "строка"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof {a: 1}); // "object"
console.log(typeof function(){}); // "function"

// instanceof - проверяет, принадлежит ли объект к определенному классу
let arr = [1, 2, 3];
let obj = {a: 1};

console.log(arr instanceof Array); // true
console.log(obj instanceof Object); // true
console.log(arr instanceof Object); // true (массивы - это объекты)
```

## Побитовые операторы

Побитовые операторы работают с 32-битными целыми числами на уровне битов:

```javascript
let a = 5; // 101 в двоичной системе
let b = 3; // 011 в двоичной системе

// Побитовое И (&)
console.log(a & b); // 1 (001 в двоичной системе)

// Побитовое ИЛИ (|)
console.log(a | b); // 7 (111 в двоичной системе)

// Побитовое исключающее ИЛИ (^)
console.log(a ^ b); // 6 (110 в двоичной системе)

// Побитовое НЕ (~)
console.log(~a); // -6

// Побитовый сдвиг влево (<<)
console.log(a << 1); // 10 (1010 в двоичной системе)

// Побитовый сдвиг вправо (>>)
console.log(a >> 1); // 2 (10 в двоичной системе)

// Побитовый сдвиг вправо без знака (>>>)
console.log(a >>> 1); // 2
```

## Операторы объединения с null и опциональной цепочки

Эти операторы были добавлены в более новых версиях JavaScript:

```javascript
// Оператор нулевого слияния (??) (ES2020)
let userName = null;
let defaultName = "Гость";
let displayName = userName ?? defaultName;
console.log(displayName); // "Гость"

let userAge = 0;
let displayAge = userAge ?? 25;
console.log(displayAge); // 0 (0 не является null или undefined)

// Оператор нулевого слияния отличается от || тем, что он рассматривает только null и undefined
let count = 0;
let result1 = count || 10; // 10 (0 считается falsy)
let result2 = count ?? 10; // 0 (0 не null и не undefined)

// Опциональная цепочка (?.) (ES2020)
let user = {
  name: "Иван",
  address: {
    street: "Ленина 10"
  }
};

console.log(user?.name); // "Иван"
console.log(user?.address?.street); // "Ленина 10"
console.log(user?.phone?.number); // undefined (не ошибка)

// Опциональная цепочка с методами
let obj = {
  method: function() {
    return "результат";
  }
};

console.log(obj.method?.()); // "результат"
console.log(obj.nonExistentMethod?.()); // undefined
```

## Приоритет операторов

Операторы в JavaScript имеют разный приоритет выполнения:

```javascript
// Умножение выполняется раньше сложения
let result1 = 2 + 3 * 4; // 14, а не 20

// Использование скобок для изменения приоритета
let result2 = (2 + 3) * 4; // 20

// Приоритет логических операторов
let a = true;
let b = false;
let c = true;

let result3 = a || b && c; // true (&& имеет более высокий приоритет, чем ||)
let result4 = (a || b) && c; // true
```

## Практические примеры

### Комбинирование операторов

```javascript
// Проверка возраста и прав доступа
function checkAccess(age, hasPermission) {
  return age >= 18 && hasPermission ? "Доступ разрешен" : "Доступ запрещен";
}

console.log(checkAccess(20, true)); // "Доступ разрешен"
console.log(checkAccess(16, true)); // "Доступ запрещен"

// Расчет скидки
function calculateDiscount(price, isMember, isHoliday) {
  let discount = 0;
  
  if (isMember) {
    discount += 0.1; // 10% скидка для участников
  }
  
  if (isHoliday) {
    discount += 0.05; // 5% дополнительная скидка в праздники
  }
  
  // Максимальная скидка 15%
  discount = discount > 0.15 ? 0.15 : discount;
  
  return price * (1 - discount);
}

console.log(calculateDiscount(100, true, true)); // 85 (15% скидка)
```

### Работа с побитовыми операторами

```javascript
// Использование побитовых операторов для работы с флагами
const READ_PERMISSION = 1;    // 001
const WRITE_PERMISSION = 2;   // 010
const EXECUTE_PERMISSION = 4; // 100

let userPermissions = READ_PERMISSION | WRITE_PERMISSION; // 011 (чтение и запись)

// Проверка разрешений
function hasPermission(permissions, permission) {
  return (permissions & permission) === permission;
}

console.log(hasPermission(userPermissions, READ_PERMISSION)); // true
console.log(hasPermission(userPermissions, EXECUTE_PERMISSION)); // false

// Добавление разрешения
userPermissions |= EXECUTE_PERMISSION; // 111 (все разрешения)

// Удаление разрешения
userPermissions &= ~WRITE_PERMISSION; // 101 (чтение и выполнение)
```

## Заключение

Операторы являются фундаментальной частью JavaScript, позволяя выполнять различные операции с данными. Понимание их работы, приоритета и особенностей поможет писать более эффективный и читаемый код.

#javascript #programming #operators #fundamentals #web-development