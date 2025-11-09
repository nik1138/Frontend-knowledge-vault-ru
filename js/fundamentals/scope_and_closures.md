# Область видимости и замыкания в JavaScript: Основы

## Что такое область видимости

Область видимости (scope) в JavaScript определяет доступность переменных, функций и объектов в определенной части кода во время выполнения. Это ключевая концепция, которая определяет, где в коде можно использовать различные переменные и функции.

## Типы областей видимости

### Глобальная область видимости

Переменные, объявленные вне любых функций, имеют глобальную область видимости и доступны из любого места в коде:

```javascript
// Глобальная переменная
let globalVar = "Я глобальная переменная";

function example() {
  // Доступ к глобальной переменной из функции
  console.log(globalVar); // "Я глобальная переменная"
}

example();
console.log(globalVar); // "Я глобальная переменная" (доступна и здесь)

// Глобальные переменные доступны в любом месте
if (true) {
  console.log(globalVar); // "Я глобальная переменная"
}
```

### Локальная область видимости

Переменные, объявленные внутри функции, имеют локальную область видимости и доступны только внутри этой функции:

```javascript
function outer() {
  let localVar = "Я локальная переменная";
  
  console.log(localVar); // "Я локальная переменная" (доступна внутри функции)
  
  function inner() {
    // Вложенная функция также имеет доступ к переменным внешней функции
    console.log(localVar); // "Я локальная переменная"
  }
  
  inner();
}

outer();
// console.log(localVar); // Ошибка! Переменная не доступна вне функции
```

### Блочная область видимости

С введением `let` и `const` в ES6 появилась блочная область видимости:

```javascript
function example() {
  if (true) {
    var functionScoped = "Функциональная область";
    let blockScoped = "Блочная область";
    const constant = "Константа";
    
    console.log(functionScoped); // "Функциональная область"
    console.log(blockScoped); // "Блочная область"
    console.log(constant); // "Константа"
  }
  
  console.log(functionScoped); // "Функциональная область" (доступна)
  // console.log(blockScoped); // ReferenceError! Не доступна вне блока
  // console.log(constant); // ReferenceError! Не доступна вне блока
}

example();
```

## Лексическая область видимости

JavaScript использует лексическую (статическую) область видимости, что означает, что область видимости переменных определяется местом их объявления в исходном коде:

```javascript
let globalVar = "Глобальная";

function outer() {
  let outerVar = "Внешняя";
  
  function inner() {
    let innerVar = "Внутренняя";
    
    // inner имеет доступ к своим переменным
    console.log(innerVar); // "Внутренняя"
    
    // inner имеет доступ к переменным outer
    console.log(outerVar); // "Внешняя"
    
    // inner имеет доступ к глобальным переменным
    console.log(globalVar); // "Глобальная"
  }
  
  inner();
  
  // outer имеет доступ к своим переменным
  console.log(outerVar); // "Внешняя"
  
  // outer имеет доступ к глобальным переменным
  console.log(globalVar); // "Глобальная"
  
  // outer НЕ имеет доступа к переменным inner
  // console.log(innerVar); // ReferenceError!
}

outer();
```

## Поднятие (Hoisting)

Поднятие - это механизм JavaScript, при котором объявления переменных и функций перемещаются в начало своей области видимости:

### Поднятие переменных

```javascript
function hoistingExample() {
  console.log(a); // undefined (не ошибка!)
  console.log(b); // ReferenceError: Cannot access 'b' before initialization
  console.log(c); // ReferenceError: Cannot access 'c' before initialization
  
  var a = 1;
  let b = 2;
  const c = 3;
  
  console.log(a); // 1
  console.log(b); // 2
  console.log(c); // 3
}

hoistingExample();
```

### Поднятие функций

```javascript
function functionHoisting() {
  // Функции-объявления поднимаются полностью
  console.log(declaredFunction()); // "Я функция-объявление"
  
  // Функциональные выражения поднимаются как переменные
  // console.log(expressionFunction()); // TypeError: expressionFunction is not a function
  
  // Стрелочные функции поднимаются как переменные
  // console.log(arrowFunction()); // TypeError: arrowFunction is not a function
  
  // Объявление функции
  function declaredFunction() {
    return "Я функция-объявление";
  }
  
  // Функциональное выражение
  var expressionFunction = function() {
    return "Я функциональное выражение";
  };
  
  // Стрелочная функция
  var arrowFunction = () => {
    return "Я стрелочная функция";
  };
  
  console.log(expressionFunction()); // "Я функциональное выражение"
  console.log(arrowFunction()); // "Я стрелочная функция"
}

functionHoisting();
```

## Временная мертвая зона (Temporal Dead Zone - TDZ)

Переменные, объявленные с помощью `let` и `const`, находятся во временной мертвой зоне от начала блока до момента их объявления:

```javascript
function tdzExample() {
  // console.log(a); // ReferenceError: Cannot access 'a' before initialization
  // console.log(b); // ReferenceError: Cannot access 'b' before initialization
  
  let a = 1;
  const b = 2;
  
  console.log(a); // 1
  console.log(b); // 2
}

tdzExample();
```

## Замыкания

Замыкание - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Замыкания позволяют функции иметь доступ к переменным внешней функции даже после завершения выполнения внешней функции:

### Базовое замыкание

```javascript
function outer(x) {
  // Внешняя функция создает лексическое окружение
  return function inner(y) {
    // Внутренняя функция имеет доступ к переменным внешней функции
    return x + y;
  };
}

const add5 = outer(5); // outer завершает выполнение, но x сохраняется
console.log(add5(3)); // 8 (x = 5, y = 3)

const add10 = outer(10);
console.log(add10(3)); // 13 (x = 10, y = 3)
```

### Практическое использование замыканий

```javascript
// Создание приватных переменных
function createCounter() {
  let count = 0; // Приватная переменная
  
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
    },
    
    reset: function() {
      count = 0;
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1

// Попытка доступа к приватной переменной
// console.log(counter.count); // undefined
```

### Замыкания в циклах

```javascript
// Проблема с var в циклах
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // Выведет 3 три раза
  }, 100);
}

// Решение с помощью замыкания
for (var j = 0; j < 3; j++) {
  (function(index) {
    setTimeout(function() {
      console.log(index); // Выведет 0, 1, 2
    }, 100);
  })(j);
}

// Решение с let (блочная область видимости)
for (let k = 0; k < 3; k++) {
  setTimeout(function() {
    console.log(k); // Выведет 0, 1, 2
  }, 100);
}
```

### Модульный паттерн с замыканиями

```javascript
// Модуль с приватными и публичными методами
const myModule = (function() {
  // Приватные переменные и функции
  let privateVar = "Я приватная переменная";
  
  function privateFunction() {
    return "Я приватная функция";
  }
  
  function privateMethod() {
    return privateVar + " и " + privateFunction();
  }
  
  // Публичный API
  return {
    publicVar: "Я публичная переменная",
    
    publicMethod: function() {
      return "Я публичная функция";
    },
    
    getPrivateData: function() {
      // Используем замыкание для доступа к приватным данным
      return privateMethod();
    },
    
    setPrivateVar: function(value) {
      if (typeof value === "string") {
        privateVar = value;
      }
    }
  };
})();

console.log(myModule.publicVar); // "Я публичная переменная"
console.log(myModule.publicMethod()); // "Я публичная функция"
console.log(myModule.getPrivateData()); // "Я приватная переменная и Я приватная функция"

myModule.setPrivateVar("Новое значение");
console.log(myModule.getPrivateData()); // "Новое значение и Я приватная функция"
```

## Практические примеры

### Фабрика функций

```javascript
// Фабрика функций с использованием замыканий
function createValidator(validationFunction, errorMessage) {
  return function(value) {
    if (!validationFunction(value)) {
      throw new Error(errorMessage);
    }
    return true;
  };
}

// Создание специализированных валидаторов
const emailValidator = createValidator(
  value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  "Неверный формат email"
);

const minLengthValidator = createValidator(
  value => value.length >= 6,
  "Минимальная длина 6 символов"
);

try {
  emailValidator("invalid-email"); // Бросит ошибку
} catch (error) {
  console.log(error.message); // "Неверный формат email"
}

try {
  minLengthValidator("short"); // Бросит ошибку
} catch (error) {
  console.log(error.message); // "Минимальная длина 6 символов"
}
```

### Мемоизация с замыканиями

```javascript
// Мемоизация функций с использованием замыканий
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log("Из кэша");
      return cache.get(key);
    }
    
    console.log("Вычисление");
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Пример использования
const expensiveFunction = memoize(function(n) {
  // Имитация дорогой операции
  let result = 0;
  for (let i = 0; i < n; i++) {
    result += i;
  }
  return result;
});

console.log(expensiveFunction(1000)); // Вычисление: 499500
console.log(expensiveFunction(1000)); // Из кэша: 499500
console.log(expensiveFunction(2000)); // Вычисление: 1999000
```

### Каррирование с замыканиями

```javascript
// Каррирование функций с использованием замыканий
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      return curried.apply(this, args.concat(nextArgs));
    };
  };
}

// Пример использования
function multiply(a, b, c) {
  return a * b * c;
}

const curriedMultiply = curry(multiply);

console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24
console.log(curriedMultiply(2, 3, 4)); // 24
```

## Распространенные ошибки и лучшие практики

### Избегайте глобальных переменных

```javascript
// Плохо: загрязнение глобального пространства имен
var globalCounter = 0;

function increment() {
  globalCounter++;
  return globalCounter;
}

// Хорошо: использование замыканий для инкапсуляции
const counterModule = (function() {
  let counter = 0;
  
  return {
    increment: function() {
      counter++;
      return counter;
    },
    
    decrement: function() {
      counter--;
      return counter;
    },
    
    getCount: function() {
      return counter;
    }
  };
})();
```

### Правильное использование this в замыканиях

```javascript
const obj = {
  name: "Объект",
  
  // Стрелочная функция не создает собственный this
  arrowMethod: function() {
    setTimeout(() => {
      console.log(this.name); // "Объект" (лексический this)
    }, 100);
  },
  
  // Обычная функция создает собственный this
  regularMethod: function() {
    const self = this; // Сохраняем ссылку на this
    setTimeout(function() {
      console.log(self.name); // "Объект"
    }, 100);
  }
};

obj.arrowMethod();
obj.regularMethod();
```

## Заключение

Понимание области видимости и замыканий критически важно для написания эффективного и предсказуемого кода на JavaScript. Эти концепции позволяют создавать приватные переменные, реализовывать модульный паттерн, оптимизировать производительность с помощью мемоизации и решать множество других задач. Правильное использование этих механизмов делает код более безопасным, поддерживаемым и эффективным.

#javascript #programming #scope #closures #fundamentals #web-development