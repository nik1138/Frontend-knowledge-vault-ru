# Упражнения по управляющим структурам

## Упражнение 6: Условные операторы
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

## Упражнение 7: Циклы
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