# Каррирование в JavaScript: Функциональное программирование и глубокие паттерны

## Обзор

Каррирование (currying) - это техника преобразования функции с несколькими аргументами в последовательность функций, каждая из которых принимает один аргумент. Это позволяет частично применять функции и создавать новые функции с предустановленными аргументами. Каррирование является важным паттерном в функциональном программировании, позволяющим создавать гибкие и переиспользуемые компоненты.

## Основы каррирования

```javascript
// Некаррированная функция
function add(a, b, c) {
    return a + b + c;
}

// Каррированная версия той же функции
function addCurried(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

// Использование
console.log(add(1, 2, 3));           // 6
console.log(addCurried(1)(2)(3));    // 6
```

## Каррирование с использованием стрелочных функций

```javascript
// Каррированная функция с использованием стрелочных функций
const multiply = a => b => c => a * b * c;

console.log(multiply(2)(3)(4)); // 24

// Частичное применение
const multiplyByTwo = multiply(2);
const multiplyByTwoAndThree = multiplyByTwo(3);

console.log(multiplyByTwoAndThree(5)); // 30
console.log(multiply(2)(3)(5));        // 30
```

## Универсальная функция каррирования

```javascript
function curry(fn) {
    return function curried(...args) {
        // Если передано достаточно аргументов, выполнить функцию
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            // Иначе вернуть функцию, которая будет ждать остальные аргументы
            return function(...nextArgs) {
                return curried.apply(this, args.concat(nextArgs));
            };
        }
    };
}

// Пример использования универсальной функции каррирования
function sum(a, b, c, d) {
    return a + b + c + d;
}

const curriedSum = curry(sum);

console.log(curriedSum(1)(2)(3)(4));           // 10
console.log(curriedSum(1, 2)(3, 4));           // 10
console.log(curriedSum(1, 2, 3, 4));           // 10
console.log(curriedSum(1)(2, 3, 4));           // 10
```

## Практические примеры использования каррирования

### 1. Функции валидации

```javascript
const check = validator => value => validator(value);

const isType = type => value => typeof value === type;
const isPositive = num => num > 0;
const isLongerThan = length => str => str.length > length;

const isString = check(isType('string'));
const isNumber = check(isType('number'));
const isPositiveNumber = value => isNumber(value) && isPositive(value);
const isLongString = check(isLongerThan(10));

console.log(isString("hello"));        // true
console.log(isPositiveNumber(5));      // true
console.log(isPositiveNumber(-3));     // false
console.log(isLongString("short"));    // false
console.log(isLongString("very long string")); // true
```

### 2. Функции форматирования

```javascript
const format = prefix => suffix => value => `${prefix}${value}${suffix}`;

const bold = format('<b>')('</b>');
const italic = format('<i>')('</i>');
const highlight = format('<mark>')('</mark>');

console.log(bold('JavaScript'));      // <b>JavaScript</b>
console.log(italic('is awesome'));    // <i>is awesome</i>
console.log(highlight('Functional Programming')); // <mark>Functional Programming</mark>
```

### 3. Функции фильтрации

```javascript
const filter = predicate => array => array.filter(predicate);

const greaterThan = num => value => value > num;
const lessThan = num => value => value < num;
const equalTo = num => value => value === num;

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const getGreaterThan5 = filter(greaterThan(5));
const getLessThan8 = filter(lessThan(8));
const getEqualTo3 = filter(equalTo(3));

console.log(getGreaterThan5(numbers)); // [6, 7, 8, 9, 10]
console.log(getLessThan8(numbers));    // [1, 2, 3, 4, 5, 6, 7]
console.log(getEqualTo3([1, 2, 3, 4, 3, 5])); // [3, 3]
```

## Продвинутые примеры

### Композиция каррированных функций

```javascript
const compose = (...fns) => value => fns.reduceRight((acc, fn) => fn(acc), value);

const add10 = x => x + 10;
const multiplyBy2 = x => x * 2;
const subtract5 = x => x - 5;

const complexOperation = compose(subtract5, multiplyBy2, add10);

console.log(complexOperation(5)); // ((5 + 10) * 2) - 5 = 25
```

### Каррирование для функций высшего порядка

```javascript
const map = fn => array => array.map(fn);
const pipe = (...fns) => value => fns.reduce((acc, fn) => fn(acc), value);

const double = x => x * 2;
const increment = x => x + 1;
const isEven = n => n % 2 === 0;

const numbers = [1, 2, 3, 4, 5];

// Удвоить, увеличить на 1, отфильтровать четные
const processNumbers = pipe(
    map(double),
    map(increment),
    arr => arr.filter(isEven)
);

console.log(processNumbers(numbers)); // [4, 6, 8, 10]
```

## Преимущества каррирования

1. **Повторное использование**: Можно создавать новые функции из существующих
2. **Частичное применение**: Можно фиксировать некоторые аргументы
3. **Композиция**: Легко комбинировать функции
4. **Читаемость**: Код становится более декларативным

## Возможные проблемы

1. **Производительность**: Каррирование может добавлять накладные расходы
2. **Сложность понимания**: Может быть сложнее для понимания новичками
3. **Отладка**: Сложнее отлаживать цепочки каррированных функций

## Продвинутые паттерны каррирования

### Улучшенная функция каррирования с поддержкой функций с переменным числом аргументов

```javascript
// Улучшенная функция каррирования с поддержкой rest параметров
const curryAdvanced = (fn) => {
    const curried = (...args) => {
        // Если передано достаточно аргументов, выполнить функцию
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            // Иначе вернуть функцию, которая будет ждать остальные аргументы
            return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
        }
    };
    return curried;
};

// Каррирование функций с rest параметрами
const sumAll = (...numbers) => numbers.reduce((sum, num) => sum + num, 0);

// Для работы с rest параметрами нужно использовать специальный подход
const curryRest = (fn, arity = fn.length) => {
    return function curried(...args) {
        if (args.length >= arity) {
            return fn.apply(this, args);
        } else {
            return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
        }
    };
};

// Пример использования
const multiplyAll = (factor, ...numbers) => numbers.map(n => n * factor);
const curriedMultiply = curryRest(multiplyAll, 3); // Указываем, что ожидаем 3 аргумента

console.log(curriedMultiply(2, 1, 2, 3)); // [2, 4, 6]
console.log(curriedMultiply(2)(1, 2, 3)); // [2, 4, 6]
```

### Каррирование с обработкой ошибок

```javascript
// Каррирование с обработкой ошибок
const safeCurry = (fn) => {
    return function curried(...args) {
        try {
            if (args.length >= fn.length) {
                return fn.apply(this, args);
            } else {
                return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
            }
        } catch (error) {
            console.error(`Ошибка при вызове функции: ${error.message}`);
            throw error;
        }
    };
};

// Пример функции с возможной ошибкой
const divide = (a, b) => {
    if (b === 0) throw new Error("Деление на ноль");
    return a / b;
};

const safeDivide = safeCurry(divide);

try {
    console.log(safeDivide(10)(2)); // 5
    console.log(safeDivide(10)(0)); // Ошибка: Деление на ноль
} catch (error) {
    console.log("Обработанная ошибка:", error.message);
}
```

### Каррирование для работы с асинхронными функциями

```javascript
// Каррирование асинхронных функций
const curryAsync = (fn) => {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
        }
    };
};

// Пример асинхронной функции
const fetchUserData = async (baseUrl, userId) => {
    // Симуляция асинхронного запроса
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id: userId, name: `User ${userId}`, url: `${baseUrl}/users/${userId}` };
};

const curriedFetchUser = curryAsync(fetchUserData);
const fetchFromApi = curriedFetchUser('https://api.example.com');

// Использование
fetchFromApi(123)
    .then(user => console.log('Пользователь:', user))
    .catch(error => console.error('Ошибка:', error));

// Частичное применение для создания специфичных функций
const fetchGithubUser = curriedFetchUser('https://api.github.com');
fetchGithubUser(456)
    .then(user => console.log('GitHub пользователь:', user));
```

### Функции для работы с объектами

```javascript
// Каррированные функции для работы с объектами
const prop = key => obj => obj[key];
const propEq = (key, value) => obj => obj[key] === value;
const whereEq = (spec) => obj => Object.keys(spec).every(key => obj[key] === spec[key]);
const mapObj = fn => obj => Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, fn(v)]));

// Пример использования
const users = [
    { id: 1, name: 'Иван', age: 25, active: true },
    { id: 2, name: 'Мария', age: 30, active: false },
    { id: 3, name: 'Петр', age: 35, active: true }
];

const activeUsers = users.filter(propEq('active', true));
const userNames = users.map(prop('name'));
const matureUsers = users.filter(propEq('age', 30));

console.log('Активные пользователи:', activeUsers);
console.log('Имена пользователей:', userNames);
console.log('Пользователи 30 лет:', matureUsers);

// Комбинирование условий
const isMatureActive = whereEq({ age: 35, active: true });
const matureActiveUsers = users.filter(isMatureActive);
console.log('Зрелые активные пользователи:', matureActiveUsers);
```

### Функции для работы с массивами

```javascript
// Каррированные функции для работы с массивами
const map = fn => arr => arr.map(fn);
const filter = predicate => arr => arr.filter(predicate);
const reduce = (reducer, initial) => arr => arr.reduce(reducer, initial);
const take = n => arr => arr.slice(0, n);
const drop = n => arr => arr.slice(n);
const find = predicate => arr => arr.find(predicate);

// Композиция операций над массивами
const processUsers = pipe(
    filter(propEq('active', true)),  // Фильтруем активных пользователей
    map(prop('name')),               // Извлекаем имена
    take(2)                          // Берем первых двух
);

const result = processUsers(users);
console.log('Первые 2 активных пользователя:', result); // ['Иван', 'Петр']
```

## Функциональное программирование с каррированием

### Чистые функции и каррирование

```javascript
// Чистые функции с каррированием
const createCalculator = (operation) => (a) => (b) => {
    switch(operation) {
        case 'add': return a + b;
        case 'subtract': return a - b;
        case 'multiply': return a * b;
        case 'divide': return b !== 0 ? a / b : NaN;
        default: throw new Error('Неизвестная операция');
    }
};

const add = createCalculator('add');
const multiply = createCalculator('multiply');
const divide = createCalculator('divide');

console.log(add(5)(3));      // 8
console.log(multiply(4)(7)); // 28
console.log(divide(10)(2));  // 5
```

### Функторы и каррирование

```javascript
// Функтор для работы с Maybe (обработка null/undefined)
const Maybe = (value) => ({
    value,
    map: fn => value != null ? Maybe(fn(value)) : Maybe(null),
    chain: fn => value != null ? fn(value) : Maybe(null),
    getOrElse: defaultValue => value != null ? value : defaultValue
});

// Каррированная функция для безопасной работы с Maybe
const safeProp = key => obj => obj != null ? obj[key] : null;
const safeHead = arr => arr && arr.length > 0 ? arr[0] : null;

// Пример использования
const user = { name: 'Иван', address: { city: 'Москва' } };
const missingUser = null;

const getCity = pipe(
    safeProp('address'),
    addr => addr ? safeProp('city')(addr) : null
);

console.log(getCity(user));       // 'Москва'
console.log(getCity(missingUser)); // undefined

// С использованием Maybe
const maybeUserCity = Maybe(user)
    .map(safeProp('address'))
    .map(safeProp('city'))
    .getOrElse('Город не указан');

console.log(maybeUserCity); // 'Москва'
```

### Монады и каррирование

```javascript
// Монада Either для обработки ошибок
const Right = value => ({
    chain: fn => fn(value),
    map: fn => Right(fn(value)),
    fold: (f, g) => g(value),
    value
});

const Left = value => ({
    chain: fn => Left(value),
    map: fn => Left(value),
    fold: (f, g) => f(value),
    value
});

// Функция валидации с использованием Either
const validateEmail = email => 
    email.includes('@') ? Right(email) : Left('Неверный формат email');

const validateLength = min => str => 
    str.length >= min ? Right(str) : Left(`Длина должна быть не менее ${min} символов`);

// Каррированная функция для валидации
const validateUser = email => 
    validateEmail(email)
        .chain(validateLength(5));

// Пример использования
console.log(validateUser('test@example.com').fold(
    error => `Ошибка: ${error}`,
    success => `Успешно: ${success}`
)); // Успешно: test@example.com

console.log(validateUser('invalid').fold(
    error => `Ошибка: ${error}`,
    success => `Успешно: ${success}`
)); // Ошибка: Неверный формат email
```

## Практические примеры из промышленной разработки

### Middleware с каррированием

```javascript
// Каррированные middleware функции
const logger = (level) => (handler) => (...args) => {
    console.log(`[${level}] Вызов функции с аргументами:`, args);
    const result = handler(...args);
    console.log(`[${level}] Результат:`, result);
    return result;
};

const validator = (schema) => (handler) => (...args) => {
    // Простая проверка соответствия схеме
    if (args.length !== schema.length) {
        throw new Error('Неверное количество аргументов');
    }
    return handler(...args);
};

const timer = (handler) => (...args) => {
    const start = performance.now();
    const result = handler(...args);
    const end = performance.now();
    console.log(`Выполнение заняло ${(end - start).toFixed(2)} мс`);
    return result;
};

// Применение middleware к функции
const processUserData = (name, age) => ({ name, age, processed: true });

const enhancedProcess = pipe(
    timer,
    logger('INFO'),
    validator([String, Number])
)(processUserData);

console.log(enhancedProcess('Иван', 30));
```

### Фабрики компонентов с каррированием

```javascript
// Фабрика компонентов с каррированием
const createComponent = (tagName) => (defaultProps) => (props, children = '') => {
    const allProps = { ...defaultProps, ...props };
    const propsString = Object.entries(allProps)
        .map(([key, value]) => `${key}="${value}"`)
        .join(' ');
    
    return `<${tagName} ${propsString}>${children}</${tagName}>`;
};

// Создание специфичных фабрик компонентов
const createButton = createComponent('button')({ type: 'button', class: 'btn' });
const createInput = createComponent('input')({ type: 'text', class: 'form-control' });

console.log(createButton({ class: 'btn-primary', id: 'submit' }, 'Отправить'));
// <button type="button" class="btn-primary" id="submit">Отправить</button>

console.log(createInput({ placeholder: 'Имя', id: 'name' }));
// <input type="text" class="form-control" placeholder="Имя" id="name">
```

### Конфигурируемые функции

```javascript
// Каррирование для создания конфигурируемых функций
const createApiFetcher = (baseUrl) => (endpoint) => async (params = {}) => {
    const url = new URL(endpoint, baseUrl);
    Object.entries(params).forEach(([key, value]) => {
        url.searchParams.append(key, value);
    });
    
    const response = await fetch(url);
    return response.json();
};

// Создание специфичных API клиентов
const githubApi = createApiFetcher('https://api.github.com');
const usersEndpoint = githubApi('/users');

// Использование
// usersEndpoint({ since: 100, per_page: 5 })
//     .then(data => console.log(data));

// Каррирование для логирования с контекстом
const createLogger = (context) => (level) => (message) => {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] [${context}] [${level}] ${message}`);
};

const userLogger = createLogger('UserModule');
const errorLogger = userLogger('ERROR');
const infoLogger = userLogger('INFO');

errorLogger('Ошибка при создании пользователя');
infoLogger('Пользователь успешно создан');
```

## Лучшие практики и советы по производительности

### Оптимизация каррированных функций

```javascript
// Избегайте создания лишних функций в циклах
const numbers = [1, 2, 3, 4, 5];

// Плохо: новая каррированная функция на каждой итерации
const badResult = numbers.map(n => multiply(2)(n));

// Лучше: создание частично примененной функции один раз
const multiplyByTwo = multiply(2);
const goodResult = numbers.map(multiplyByTwo);

// Использование memoization для каррированных функций
const memoize = (fn) => {
    const cache = new Map();
    return (...args) => {
        const key = JSON.stringify(args);
        if (cache.has(key)) {
            return cache.get(key);
        }
        const result = fn(...args);
        cache.set(key, result);
        return result;
    };
};

// Каррированная memoized функция
const memoizedAdd = memoize(add);
const curriedMemoizedAdd = curry(memoizedAdd);

// Использование валидации каррированных функций
const createValidatedCurry = (fn, validators) => {
    const curried = (...args) => {
        // Проверяем валидаторы для текущего количества аргументов
        if (args.length <= validators.length) {
            for (let i = 0; i < args.length; i++) {
                if (validators[i] && !validators[i](args[i])) {
                    throw new Error(`Неверный аргумент на позиции ${i}`);
                }
            }
        }
        
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
        }
    };
    return curried;
};

// Пример с валидацией
const validatedDivide = createValidatedCurry(
    (a, b) => a / b,
    [num => typeof num === 'number', num => typeof num === 'number' && num !== 0]
);

try {
    console.log(validatedDivide(10)(2)); // 5
    // console.log(validatedDivide(10)(0)); // Ошибка: Неверный аргумент на позиции 1
} catch (error) {
    console.error(error.message);
}
```

### Типизация каррированных функций

```javascript
// Каррирование с поддержкой типов (для использования с JSDoc или TypeScript)
/**
 * Функция каррирования с типизацией
 * @template T, R
 * @param {function(...any): R} fn - Функция для каррирования
 * @returns {function(...any): (function(...any): R | R)}
 */
const typedCurry = (fn) => {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return (...nextArgs) => curried.apply(this, args.concat(nextArgs));
        }
    };
};

// Пример использования с JSDoc
/**
 * Сложение двух чисел
 * @param {number} a - Первое число
 * @param {number} b - Второе число
 * @returns {number} Сумма чисел
 */
const addNumbers = (a, b) => a + b;

const curriedAdd = typedCurry(addNumbers);
const addFive = curriedAdd(5);
console.log(addFive(3)); // 8
```

## Сравнение с другими паттернами

### Каррирование vs Частичное применение

```javascript
// Каррирование: преобразование функции в последовательность одноаргументных функций
const curriedMultiply = a => b => c => a * b * c;

// Частичное применение: фиксация части аргументов
const partialApply = (fn, ...fixedArgs) => (...args) => fn(...fixedArgs, ...args);

const multiply = (a, b, c) => a * b * c;
const multiplyBy2And3 = partialApply(multiply, 2, 3);

console.log(curriedMultiply(2)(3)(4)); // 24
console.log(multiplyBy2And3(4));      // 24

// Каррирование всегда возвращает функцию, частичное применение может вернуть результат
```

### Интеграция с другими функциональными паттернами

```javascript
// Комбинация каррирования с другими паттернами
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

// Каррированные функции для работы с объектами
const setProp = key => value => obj => ({ ...obj, [key]: value });
const getProp = key => obj => obj[key];

// Пример комплексной обработки данных
const userData = { id: 1, name: 'Иван', active: false };

const processUser = pipe(
    setProp('lastModified')(new Date().toISOString()),
    setProp('processed')(true),
    setProp('name')(name => name.toUpperCase())
);

const result = processUser(userData);
console.log(result);
// { id: 1, name: 'ИВАН', active: false, lastModified: '...', processed: true }
```

## Заключение

Каррирование является мощным инструментом в функциональном программировании, позволяющим создавать гибкие, переиспользуемые и легко комбинируемые функции. Понимание продвинутых паттернов каррирования критически важно для написания чистого, декларативного кода, особенно в контексте функциональных архитектур и библиотек.

#programming #javascript #currying #functional-programming #higher-order-functions #composition #partial-application #web-development