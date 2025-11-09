# Область видимости и замыкания в JavaScript

## Область видимости (Scope)

Область видимости определяет доступность переменных в различных частях кода. В JavaScript существуют следующие типы областей видимости:

### Глобальная область видимости
Переменные, объявленные вне любых функций или блоков, находятся в глобальной области видимости:

```javascript
var globalVar = "Я глобальная переменная";
let globalLet = "Я тоже глобальная";
const globalConst = "И я глобальная";

function test() {
    console.log(globalVar); // Доступна
}
```

**Важно:** Избегайте глобальных переменных, так как они могут вызвать конфликты и трудности в отладке.

### Функциональная область видимости
Переменные, объявленные внутри функции, доступны только внутри этой функции:

```javascript
function outerFunction() {
    var functionScoped = "Я в функциональной области";
    
    function innerFunction() {
        console.log(functionScoped); // Доступна
    }
    
    innerFunction();
}

console.log(functionScoped); // Ошибка: переменная не определена
```

### Блочная область видимости (ES6+)
Переменные, объявленные через `let` и `const`, имеют блочную область видимости:

```javascript
if (true) {
    let blockScoped = "Я в блочной области";
    const anotherBlock = "Я тоже в блочной области";
    var functionScoped = "Я в функциональной области";
}

console.log(blockScoped); // Ошибка: переменная не определена
console.log(anotherBlock); // Ошибка: переменная не определена
console.log(functionScoped); // Доступна
```

## Hoisting (поднятие)

JavaScript "поднимает" объявления переменных и функций в начало их области видимости:

```javascript
// Переменные var поднимаются с undefined
console.log(hoistedVar); // undefined (не ошибка)
var hoistedVar = 5;
console.log(hoistedVar); // 5

// Функции поднимаются полностью
console.log(hoistedFunction()); // "Я поднятая функция"

function hoistedFunction() {
    return "Я поднятая функция";
}

// Функциональные выражения НЕ поднимаются
// console.log(notHoistedFunction); // Ошибка: Cannot access before initialization
const notHoistedFunction = function() {
    return "Я не поднятая функция";
};
```

### Различия между var, let и const

```javascript
// var - поднимается с undefined
function testVar() {
    console.log(a); // undefined
    var a = 5;
    console.log(a); // 5
}

// let/const - поднимаются, но в "временной мертвой зоне"
function testLet() {
    console.log(b); // ReferenceError
    let b = 5;
}

function testConst() {
    console.log(c); // ReferenceError
    const c = 5;
}
```

## Замыкания (Closures)

Замыкание — это комбинация функции и лексического окружения, в котором эта функция была объявлена. Функция может получить доступ к переменным из внешней функции даже после возврата внешней функции.

### Базовый пример замыкания

```javascript
function outerFunction(x) {
    return function innerFunction(y) {
        return x + y; // innerFunction имеет доступ к x из внешней области
    };
}

const addFive = outerFunction(5);
console.log(addFive(3)); // 8
console.log(addFive(10)); // 15
```

### Практическое применение замыканий

#### 1. Инкапсуляция данных

```javascript
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
        }
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
// count недоступен напрямую: counter.count вернет undefined
```

#### 2. Каррирование

```javascript
function multiply(a) {
    return function(b) {
        return a * b;
    };
}

const multiplyByTwo = multiply(2);
console.log(multiplyByTwo(5)); // 10

// Стрелочные функции для каррирования
const multiplyArrow = a => b => a * b;
const double = multiplyArrow(2);
console.log(double(8)); // 16
```

#### 3. Создание приватных методов

```javascript
const bankAccount = (function() {
    let balance = 0; // Приватная переменная
    
    return {
        deposit: function(amount) {
            if (amount > 0) {
                balance += amount;
                return `Внесено: ${amount}, баланс: ${balance}`;
            }
        },
        withdraw: function(amount) {
            if (amount > 0 && amount <= balance) {
                balance -= amount;
                return `Снято: ${amount}, баланс: ${balance}`;
            }
            return "Недостаточно средств";
        },
        getBalance: function() {
            return balance;
        }
    };
})();

console.log(bankAccount.deposit(100)); // "Внесено: 100, баланс: 100"
console.log(bankAccount.withdraw(30)); // "Снято: 30, баланс: 70"
console.log(bankAccount.getBalance()); // 70
```

#### 4. Обработчики событий с замыканиями

```javascript
function attachListeners() {
    const buttons = document.querySelectorAll('.btn');
    
    for (let i = 0; i < buttons.length; i++) {
        // Замыкание сохраняет значение i для каждого обработчика
        buttons[i].addEventListener('click', function() {
            console.log(`Кнопка ${i} нажата`); // i захвачено замыканием
        });
    }
}

// Альтернатива с использованием forEach
function attachListenersWithForEach() {
    document.querySelectorAll('.btn').forEach((button, index) => {
        button.addEventListener('click', function() {
            console.log(`Кнопка ${index} нажата`);
        });
    });
}
```

## Современные возможности и лучшие практики

### Использование модулей ES6

```javascript
// counter.js
let count = 0;

export function increment() {
    return ++count;
}

export function decrement() {
    return --count;
}

export function getCount() {
    return count;
}

// Использование
// import { increment, decrement, getCount } from './counter.js';
```

### IIFE (Immediately Invoked Function Expression)

```javascript
// Защита от загрязнения глобального пространства
const MyModule = (function() {
    let privateVariable = "Я приватная переменная";
    
    function privateMethod() {
        return "Я приватный метод";
    }
    
    return {
        publicMethod: function() {
            return `Публичный доступ к: ${privateVariable}`;
        },
        
        anotherPublicMethod: function() {
            return privateMethod();
        }
    };
})();

console.log(MyModule.publicMethod()); // "Публичный доступ к: Я приватная переменная"
```

### Лексическое окружение

Каждая функция во время выполнения имеет ссылку на внешнее лексическое окружение:

```javascript
function createMultiplier(multiplier) {
    // Внешнее лексическое окружение для innerFunction
    return function innerFunction(number) {
        // innerFunction имеет доступ к multiplier из внешнего окружения
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

## Проблемы и решения

### Частая ошибка с циклами и замыканиями

```javascript
// Плохой пример
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Выведет 3, 3, 3
    }, 100);
}

// Решение 1: IIFE
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(function() {
            console.log(index); // Выведет 0, 1, 2
        }, 100);
    })(i);
}

// Решение 2: Использование let вместо var
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Выведет 0, 1, 2
    }, 100);
}

// Решение 3: Использование bind
for (var i = 0; i < 3; i++) {
    setTimeout(function(index) {
        console.log(index);
    }.bind(null, i), 100);
}
```

## Примеры из промышленной разработки

### Создание кеширующей функции

```javascript
function createCachedFunction(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Взято из кеша');
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Пример использования
const expensiveCalculation = createCachedFunction((x, y) => {
    console.log('Выполняется сложное вычисление...');
    return x * y + Math.random();
});

console.log(expensiveCalculation(5, 10)); // Выполняется сложное вычисление..., результат
console.log(expensiveCalculation(5, 10)); // Взято из кеша, результат
```

### Фабрика функций валидации

```javascript
function createValidator(rules) {
    return function(data) {
        const errors = [];
        
        for (const [field, rule] of Object.entries(rules)) {
            if (!rule(data[field])) {
                errors.push(`Поле ${field} не прошло валидацию`);
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    };
}

// Создание валидатора
const userValidator = createValidator({
    name: value => typeof value === 'string' && value.length > 0,
    age: value => typeof value === 'number' && value >= 0 && value <= 120,
    email: value => typeof value === 'string' && value.includes('@')
});

// Использование
const result = userValidator({ name: "Иван", age: 25, email: "ivan@example.com" });
console.log(result.isValid); // true
```

## Лучшие практики

1. **Используйте `const` и `let` вместо `var`**
2. **Минимизируйте использование глобальных переменных**
3. **Используйте замыкания для инкапсуляции данных**
4. **Будьте осторожны с замыканиями в циклах**
5. **Используйте модули ES6 для организации кода**

## Ссылки на связанные темы
- [[js/basics/types]] - Типы данных
- [[js/functions/hoisting]] - Поднятие переменных
- [[js/es6+/modules]] - Модули ES6
- [[js/testing/mocking]] - Мокирование в тестах

#programming #javascript #scope #closures #best-practices