# Продвинутые функции в JavaScript

## Введение

Функции в JavaScript являются объектами первого класса, что означает, что они могут быть переданы как аргументы, возвращены из других функций и присвоены переменным. Это уникальная особенность JavaScript, которая открывает множество возможностей для написания гибкого и мощного кода.

## Функции высшего порядка

Функции высшего порядка - это функции, которые принимают другие функции в качестве аргументов или возвращают функции в качестве результата.

```javascript
// Функция высшего порядка, принимающая функцию как аргумент
function calculate(operation, a, b) {
  return operation(a, b);
}

// Функции для передачи в calculate
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

// Использование
console.log(calculate(add, 5, 3));      // 8
console.log(calculate(multiply, 5, 3));  // 15

// Функция высшего порядка, возвращающая функцию
function createMultiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

## Замыкания

Замыкание - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Замыкания позволяют функциям запоминать и получать доступ к переменным из внешней области видимости даже после завершения выполнения внешней функции.

```javascript
// Простое замыкание
function outerFunction(x) {
  return function innerFunction(y) {
    return x + y;
  };
}

const add5 = outerFunction(5);
console.log(add5(3));  // 8

// Практическое использование замыканий
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
console.log(counter.increment());  // 1
console.log(counter.increment());  // 2
console.log(counter.decrement());  // 1
console.log(counter.getCount());   // 1

// Приватные переменные с помощью замыканий
function createUser(name) {
  let _name = name;
  
  return {
    getName: function() {
      return _name;
    },
    setName: function(newName) {
      if (newName && typeof newName === 'string') {
        _name = newName;
      }
    }
  };
}

const user = createUser("Иван");
console.log(user.getName());  // "Иван"
user.setName("Мария");
console.log(user.getName());  // "Мария"
// user._name не доступен напрямую
```

## Каррирование

Каррирование - это техника преобразования функции с несколькими аргументами в последовательность функций с одним аргументом.

```javascript
// Функция без каррирования
function multiply(a, b, c) {
  return a * b * c;
}

console.log(multiply(2, 3, 4));  // 24

// Каррированная функция
function curryMultiply(a) {
  return function(b) {
    return function(c) {
      return a * b * c;
    };
  };
}

console.log(curryMultiply(2)(3)(4));  // 24

// Более компактная запись с помощью стрелочных функций
const curryMultiplyArrow = a => b => c => a * b * c;
console.log(curryMultiplyArrow(2)(3)(4));  // 24

// Универсальная функция каррирования
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...nextArgs) {
        return curried(...args, ...nextArgs);
      };
    }
  };
}

// Пример использования
function sum(a, b, c) {
  return a + b + c;
}

const curriedSum = curry(sum);
console.log(curriedSum(1)(2)(3));    // 6
console.log(curriedSum(1, 2)(3));    // 6
console.log(curriedSum(1)(2, 3));    // 6
console.log(curriedSum(1, 2, 3));    // 6
```

## Композиция функций

Композиция функций - это процесс объединения двух или более функций для создания новой функции.

```javascript
// Функция композиции
function compose(...functions) {
  return function(value) {
    return functions.reduceRight((acc, fn) => fn(acc), value);
  };
}

// Функция пайплайна (слева направо)
function pipe(...functions) {
  return function(value) {
    return functions.reduce((acc, fn) => fn(acc), value);
  };
}

// Вспомогательные функции
const add = x => y => x + y;
const multiply = x => y => x * y;
const square = x => x * x;

// Композиция справа налево: square(multiply(2)(add(1)(5)))
const composed = compose(square, multiply(2), add(1));
console.log(composed(5));  // 144 ((5 + 1) * 2)² = 12² = 144

// Пайплайн слева направо: square(multiply(2)(add(1)(5)))
const piped = pipe(add(1), multiply(2), square);
console.log(piped(5));  // 144 ((5 + 1) * 2)² = 12² = 144

// Практический пример: обработка данных
const users = [
  { name: "Иван", age: 25, active: true },
  { name: "Мария", age: 30, active: false },
  { name: "Алексей", age: 35, active: true }
];

// Функции для обработки
const isActive = user => user.active;
const getName = user => user.name;
const toUpperCase = str => str.toUpperCase();
const addGreeting = str => `Привет, ${str}!`;

// Композиция для получения приветствий активных пользователей
const getGreetings = pipe(
  users => users.filter(isActive),
  users => users.map(getName),
  names => names.map(toUpperCase),
  greetings => greetings.map(addGreeting)
);

console.log(getGreetings(users));  
// ["Привет, ИВАН!", "Привет, АЛЕКСЕЙ!"]
```

## Мемоизация

Мемоизация - это техника оптимизации, которая заключается в кэшировании результатов выполнения функции для предотвращения повторных вычислений.

```javascript
// Простая мемоизация
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log("Возвращаем кэшированный результат");
      return cache.get(key);
    }
    
    console.log("Выполняем вычисление");
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Пример: мемоизированная функция факториала
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

const memoizedFactorial = memoize(factorial);

console.log(memoizedFactorial(5));  // Выполняем вычисление: 120
console.log(memoizedFactorial(5));  // Возвращаем кэшированный результат: 120
console.log(memoizedFactorial(6));  // Выполняем вычисление: 720

// Мемоизация для функций с несколькими аргументами
function add(a, b) {
  return a + b;
}

const memoizedAdd = memoize(add);

console.log(memoizedAdd(2, 3));  // Выполняем вычисление: 5
console.log(memoizedAdd(2, 3));  // Возвращаем кэшированный результат: 5
console.log(memoizedAdd(3, 2));  // Выполняем вычисление: 5 (разный порядок аргументов)

// Продвинутая мемоизация с ограничением размера кэша
function memoizeWithLimit(fn, limit = 100) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      // Перемещаем элемент в конец для реализации LRU
      const value = cache.get(key);
      cache.delete(key);
      cache.set(key, value);
      return value;
    }
    
    const result = fn.apply(this, args);
    
    // Ограничиваем размер кэша
    if (cache.size >= limit) {
      const firstKey = cache.keys().next().value;
      cache.delete(firstKey);
    }
    
    cache.set(key, result);
    return result;
  };
}
```

## Частичное применение

Частичное применение - это техника создания новой функции путем фиксации некоторых аргументов исходной функции.

```javascript
// Функция для частичного применения
function partial(fn, ...fixedArgs) {
  return function(...remainingArgs) {
    // Заменяем undefined в fixedArgs на значения из remainingArgs
    const args = fixedArgs.map(arg => arg === undefined ? remainingArgs.shift() : arg);
    // Добавляем оставшиеся аргументы
    return fn.apply(this, [...args, ...remainingArgs]);
  };
}

// Пример функции
function greet(greeting, punctuation, name) {
  return `${greeting}, ${name}${punctuation}`;
}

// Частичное применение
const sayHello = partial(greet, "Привет", "!");
const askQuestion = partial(greet, "Как дела", "?");

console.log(sayHello("Иван"));      // "Привет, Иван!"
console.log(askQuestion("Мария"));   // "Как дела, Мария?"

// Частичное применение с пропуском аргументов
const greetWithName = partial(greet, undefined, "!", undefined);
console.log(greetWithName("Здравствуйте", "Алексей"));  // "Здравствуйте, Алексей!"

// Использование bind для частичного применения
const multiplyBy = (x, y) => x * y;
const double = multiplyBy.bind(null, 2);
const triple = multiplyBy.bind(null, 3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

## Рекурсивные функции

Рекурсивные функции - это функции, которые вызывают сами себя для решения задачи.

```javascript
// Простая рекурсия: факториал
function factorial(n) {
  // Базовый случай
  if (n <= 1) return 1;
  // Рекурсивный случай
  return n * factorial(n - 1);
}

console.log(factorial(5));  // 120

// Рекурсия: числа Фибоначчи
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(10));  // 55

// Оптимизированная версия Фибоначчи с мемоизацией
function memoizedFibonacci() {
  const cache = new Map();
  
  function fib(n) {
    if (n <= 1) return n;
    
    if (cache.has(n)) {
      return cache.get(n);
    }
    
    const result = fib(n - 1) + fib(n - 2);
    cache.set(n, result);
    return result;
  }
  
  return fib;
}

const fib = memoizedFibonacci();
console.log(fib(50));  // 12586269025 (быстро вычисляется)

// Рекурсия для обхода дерева
function traverseTree(node, callback) {
  callback(node);
  
  if (node.children && node.children.length > 0) {
    node.children.forEach(child => traverseTree(child, callback));
  }
}

// Пример использования
const tree = {
  name: "root",
  value: 1,
  children: [
    {
      name: "child1",
      value: 2,
      children: [
        { name: "grandchild1", value: 4, children: [] },
        { name: "grandchild2", value: 5, children: [] }
      ]
    },
    {
      name: "child2",
      value: 3,
      children: [
        { name: "grandchild3", value: 6, children: [] }
      ]
    }
  ]
};

traverseTree(tree, node => console.log(node.name, node.value));
// Вывод: root 1, child1 2, grandchild1 4, grandchild2 5, child2 3, grandchild3 6

// Хвостовая рекурсия
function tailRecursiveFactorial(n, accumulator = 1) {
  if (n <= 1) return accumulator;
  return tailRecursiveFactorial(n - 1, n * accumulator);
}

console.log(tailRecursiveFactorial(5));  // 120
```

## Функции обратного вызова (Callbacks)

Функции обратного вызова - это функции, которые передаются в качестве аргументов другим функциям и вызываются позже.

```javascript
// Простой пример callback
function processUserInput(input, callback) {
  // Имитация обработки
  setTimeout(() => {
    const result = input.toUpperCase();
    callback(null, result);
  }, 1000);
}

processUserInput("привет", (error, result) => {
  if (error) {
    console.error("Ошибка:", error);
  } else {
    console.log("Результат:", result);  // "Результат: ПРИВЕТ"
  }
});

// Callback hell - антипаттерн
function getData(callback) {
  setTimeout(() => {
    callback(null, "данные1", (error, data1, next) => {
      if (!error) {
        setTimeout(() => {
          next(null, "данные2", (error, data2, final) => {
            if (!error) {
              setTimeout(() => {
                final(null, "данные3");
              }, 100);
            }
          });
        }, 100);
      }
    });
  }, 100);
}

// Лучше использовать Promise или async/await вместо callback hell
```

## Лучшие практики

### 1. Используйте чистые функции

```javascript
// Плохо: функция с побочными эффектами
let total = 0;
function addToTotal(value) {
  total += value;  // Изменяет внешнее состояние
  return total;
}

// Хорошо: чистая функция
function add(a, b) {
  return a + b;  // Не изменяет внешнее состояние
}
```

### 2. Избегайте глубокой вложенности

```javascript
// Плохо: глубокая вложенность
function processUserData(users) {
  if (users && users.length > 0) {
    users.forEach(user => {
      if (user.active) {
        if (user.profile) {
          if (user.profile.email) {
            sendEmail(user.profile.email);
          }
        }
      }
    });
  }
}

// Хорошо: ранние возвраты
function processUserData(users) {
  if (!users || users.length === 0) return;
  
  users.forEach(user => {
    if (!user.active) return;
    if (!user.profile) return;
    if (!user.profile.email) return;
    
    sendEmail(user.profile.email);
  });
}
```

### 3. Используйте деструктуризацию для параметров

```javascript
// Плохо: много параметров
function createUser(name, email, age, city, country) {
  // ...
}

// Хорошо: объект с параметрами
function createUser({ name, email, age, city, country }) {
  // ...
}

// Ещё лучше: деструктуризация с значениями по умолчанию
function createUser({ 
  name, 
  email, 
  age = 18, 
  city = "Не указан", 
  country = "Не указана" 
}) {
  // ...
}
```

## Заключение

Продвинутые функции в JavaScript открывают огромные возможности для написания гибкого, переиспользуемого и эффективного кода. Понимание таких концепций, как замыкания, каррирование, композиция и мемоизация, позволяет создавать более сложные и мощные приложения.

Важно помнить, что не всегда нужно использовать все эти техники в каждом проекте. Выбирайте подходящие инструменты для конкретной задачи и следите за читаемостью кода.

#javascript #functions #closures #functional-programming #advanced-concepts