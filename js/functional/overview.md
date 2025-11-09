# Functional Programming API

## Введение

Функциональное программирование - это парадигма программирования, в которой вычисления описываются как вычисление математических функций без изменения состояния и без данных с побочными эффектами. В этой главе мы рассмотрим основные концепции функционального программирования в JavaScript: чистые функции, неизменяемость, каррирование, композиция и другие важные паттерны.

## Содержание

- [[Чистые функции]]
- [[Неизменяемость]]
- [[Каррирование]]
- [[Композиция функций]]
- [[Функции высшего порядка]]
- [[Функторы и монады]]
- [[Работа с массивами функционально]]
- [[Обработка ошибок функционально]]
- [[Рекурсия]]

## Чистые функции

Чистые функции - это функции, которые при одинаковых входных данных всегда возвращают одинаковый результат и не имеют побочных эффектов.

### Основы чистых функций

```javascript
// Чистая функция
function add(a, b) {
    return a + b;
}

// Чистая функция с более сложной логикой
function calculateDiscount(price, discountPercent) {
    if (discountPercent < 0 || discountPercent > 100) {
        throw new Error('Неверный процент скидки');
    }
    return price * (1 - discountPercent / 100);
}

// Нечистая функция (имеет побочный эффект)
function logAdd(a, b) {
    console.log('Сложение:', a, b); // Побочный эффект
    return a + b;
}

// Нечистая функция (зависит от внешнего состояния)
let taxRate = 0.1;
function calculateWithTax(amount) {
    return amount * (1 + taxRate); // Зависит от внешней переменной
}

// Чистая версия
function calculateWithTaxPure(amount, taxRate) {
    return amount * (1 + taxRate);
}

// Пример чистой функции для обработки данных
function processUserData(users, options = {}) {
    const { 
        sortBy = 'name', 
        filterBy = null, 
        limit = null 
    } = options;
    
    let result = [...users]; // Создаем копию, чтобы не мутировать оригинал
    
    if (filterBy) {
        result = result.filter(user => user.active === filterBy.active);
    }
    
    result.sort((a, b) => {
        if (a[sortBy] < b[sortBy]) return -1;
        if (a[sortBy] > b[sortBy]) return 1;
        return 0;
    });
    
    if (limit) {
        result = result.slice(0, limit);
    }
    
    return result;
}
```

### Преимущества чистых функций

```javascript
// Демонстрация преимуществ чистых функций

// 1. Легкость тестирования
function multiply(a, b) {
    return a * b;
}

// Простой тест
console.assert(multiply(2, 3) === 6, 'multiply(2, 3) should equal 6');
console.assert(multiply(-1, 5) === -5, 'multiply(-1, 5) should equal -5');

// 2. Кэширование результатов (мемоизация)
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Взято из кэша');
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        console.log('Вычислено и закэшировано');
        return result;
    };
}

// Мемоизированная функция
const memoizedExpensiveFunction = memoize((n) => {
    // Симуляция тяжелой вычислительной операции
    let result = 0;
    for (let i = 0; i < n * 1000000; i++) {
        result += Math.sqrt(i);
    }
    return result;
});

// Первый вызов - вычисление
console.log(memoizedExpensiveFunction(100));
// Повторный вызов с теми же аргументами - из кэша
console.log(memoizedExpensiveFunction(100));

// 3. Легкость отладки
function calculateTax(amount, rate) {
    // Чистая функция легко отлаживается
    const tax = amount * rate;
    const total = amount + tax;
    return { amount, tax, total };
}

const taxResult = calculateTax(100, 0.1);
console.log('Результат расчета налога:', taxResult);
```

## Неизменяемость

Неизменяемость означает, что данные не могут быть изменены после создания. Вместо изменения объекта создается новый объект с измененными свойствами.

### Работа с неизменяемыми данными

```javascript
// Утилиты для работы с неизменяемыми данными
const ImmutableUtils = {
    // Неизменяемое обновление объекта
    updateObject(obj, updates) {
        return { ...obj, ...updates };
    },
    
    // Неизменяемое обновление вложенного свойства
    updateNestedProperty(obj, path, value) {
        const keys = path.split('.');
        const lastKey = keys.pop();
        const lastObj = keys.reduce((acc, key) => ({ ...acc, [key]: acc[key] }), obj);
        
        return {
            ...obj,
            ...this.setNestedProperty({}, keys, { ...lastObj, [lastKey]: value })
        };
    },
    
    setNestedProperty(obj, keys, value) {
        if (keys.length === 0) return value;
        
        const [head, ...tail] = keys;
        return { [head]: this.setNestedProperty(obj[head] || {}, tail, value) };
    },
    
    // Неизменяемое обновление массива
    updateArrayItem(array, index, updater) {
        return array.map((item, i) => 
            i === index ? updater(item) : item
        );
    },
    
    // Неизменяемое добавление в массив
    addToArray(array, item) {
        return [...array, item];
    },
    
    // Неизменяемое удаление из массива
    removeFromArray(array, index) {
        return array.filter((_, i) => i !== index);
    },
    
    // Неизменяемое удаление элемента по значению
    removeItem(array, item) {
        return array.filter(x => x !== item);
    }
};

// Пример использования
const user = {
    name: 'John',
    age: 30,
    address: {
        street: '123 Main St',
        city: 'New York',
        country: 'USA'
    },
    hobbies: ['reading', 'swimming']
};

// Обновление объекта без мутации
const updatedUser = ImmutableUtils.updateObject(user, { age: 31 });
console.log('Оригинал:', user.age); // 30
console.log('Обновленный:', updatedUser.age); // 31

// Обновление вложенного свойства
const userWithNewCity = ImmutableUtils.updateNestedProperty(
    user, 
    'address.city', 
    'Boston'
);
console.log('Город в оригинале:', user.address.city); // New York
console.log('Город в обновленном:', userWithNewCity.address.city); // Boston

// Работа с массивами
const updatedHobbies = ImmutableUtils.addToArray(user.hobbies, 'cooking');
console.log('Оригинальные хобби:', user.hobbies); // ['reading', 'swimming']
console.log('Новые хобби:', updatedHobbies); // ['reading', 'swimming', 'cooking']
```

### Библиотека неизменяемых структур данных

```javascript
// Простая реализация неизменяемой структуры данных
class ImmutableList {
    constructor(items = []) {
        this.items = [...items];
        Object.freeze(this); // Запрещаем изменение самого объекта
    }
    
    push(item) {
        return new ImmutableList([...this.items, item]);
    }
    
    pop() {
        if (this.items.length === 0) return this;
        return new ImmutableList(this.items.slice(0, -1));
    }
    
    shift() {
        if (this.items.length === 0) return this;
        return new ImmutableList(this.items.slice(1));
    }
    
    unshift(item) {
        return new ImmutableList([item, ...this.items]);
    }
    
    concat(otherList) {
        return new ImmutableList([...this.items, ...otherList.items]);
    }
    
    slice(start, end) {
        return new ImmutableList(this.items.slice(start, end));
    }
    
    map(fn) {
        return new ImmutableList(this.items.map(fn));
    }
    
    filter(fn) {
        return new ImmutableList(this.items.filter(fn));
    }
    
    reduce(fn, initialValue) {
        return this.items.reduce(fn, initialValue);
    }
    
    forEach(fn) {
        this.items.forEach(fn);
    }
    
    find(fn) {
        return this.items.find(fn);
    }
    
    findIndex(fn) {
        return this.items.findIndex(fn);
    }
    
    indexOf(item) {
        return this.items.indexOf(item);
    }
    
    includes(item) {
        return this.items.includes(item);
    }
    
    get length() {
        return this.items.length;
    }
    
    get(index) {
        return this.items[index];
    }
    
    set(index, value) {
        const newItems = [...this.items];
        newItems[index] = value;
        return new ImmutableList(newItems);
    }
    
    toArray() {
        return [...this.items]; // Возвращаем копию
    }
    
    toString() {
        return `ImmutableList(${this.items.join(', ')})`;
    }
    
    toJSON() {
        return this.items;
    }
}

// Использование ImmutableList
const list = new ImmutableList([1, 2, 3]);
const newList = list.push(4).push(5);
console.log('Оригинальный список:', list.toArray()); // [1, 2, 3]
console.log('Новый список:', newList.toArray()); // [1, 2, 3, 4, 5]

// Цепочка операций
const result = new ImmutableList([1, 2, 3, 4, 5])
    .map(x => x * 2)
    .filter(x => x > 4)
    .toArray();

console.log('Результат цепочки:', result); // [6, 8, 10]
```

## Каррирование

Каррирование - это техника преобразования функции с множеством аргументов в последовательность функций, каждая из которых принимает один аргумент.

### Основы каррирования

```javascript
// Функция для каррирования
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...moreArgs) {
                return curried.apply(this, args.concat(moreArgs));
            };
        }
    };
}

// Примеры каррированных функций
const multiply = (a, b, c) => a * b * c;
const curriedMultiply = curry(multiply);

// Использование
console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24

// Более сложный пример
const formatUserMessage = (greeting, name, message) => `${greeting}, ${name}! ${message}`;
const curriedFormat = curry(formatUserMessage);

const morningGreeting = curriedFormat('Доброе утро');
const johnGreeting = morningGreeting('Джон');
const finalMessage = johnGreeting('Как твои дела?');

console.log(finalMessage); // "Доброе утро, Джон! Как твои дела?"

// Или в одну цепочку
console.log(curriedFormat('Привет')('Анна')('Рад тебя видеть!'));
```

### Продвинутое каррирование

```javascript
// Каррирование с поддержкой placeholder
function advancedCurry(fn) {
    const placeholder = advancedCurry.placeholder || Symbol('placeholder');
    
    function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        
        return function(...moreArgs) {
            let argIndex = 0;
            const newArgs = args.map(arg => 
                arg === placeholder ? moreArgs[argIndex++] : arg
            );
            
            // Добавляем оставшиеся аргументы
            newArgs.push(...moreArgs.slice(argIndex));
            
            return curried.apply(this, newArgs);
        };
    }
    
    curried.placeholder = placeholder;
    return curried;
}

// Использование с placeholder
const subtract = (a, b, c) => a - b - c;
const curriedSubtract = advancedCurry(subtract);

// Использование placeholder
const subtract10 = curriedSubtract(10, _, _); // _ - placeholder
const result = subtract10(3, 2); // 10 - 3 - 2 = 5

// Или частичное применение
const partialSubtract = curriedSubtract(_, 5, _);
const result2 = partialSubtract(20, 3); // 20 - 5 - 3 = 12

// Каррирование для работы с массивами
const map = curry((fn, array) => array.map(fn));
const filter = curry((predicate, array) => array.filter(predicate));
const reduce = curry((reducer, initialValue, array) => array.reduce(reducer, initialValue));

// Пример использования
const double = x => x * 2;
const isEven = x => x % 2 === 0;
const sum = (acc, val) => acc + val;

const numbers = [1, 2, 3, 4, 5, 6];
const result3 = reduce(sum)(0)(map(double)(filter(isEven)(numbers)));
console.log(result3); // (2*2) + (4*2) + (6*2) = 4 + 8 + 12 = 24
```

## Композиция функций

Композиция функций - это процесс объединения двух или более функций для создания новой функции.

### Основы композиции

```javascript
// Композиция функций (справа налево)
function compose(...fns) {
    return (value) => fns.reduceRight((acc, fn) => fn(acc), value);
}

// Композиция функций (слева направо)
function pipe(...fns) {
    return (value) => fns.reduce((acc, fn) => fn(acc), value);
}

// Примеры функций для композиции
const toUpperCase = str => str.toUpperCase();
const addExclamation = str => str + '!';
const repeat = times => str => str.repeat(times);

// Использование композиции
const processText = compose(
    repeat(2),
    addExclamation,
    toUpperCase
);

console.log(processText('hello')); // "HELLO!HELLO!"

// Использование pipe (цепочка слева направо)
const processTextPipe = pipe(
    toUpperCase,
    addExclamation,
    repeat(3)
);

console.log(processTextPipe('world')); // "WORLD!WORLD!WORLD!"

// Более сложный пример
const getLength = str => str.length;
const double = num => num * 2;
const add10 = num => num + 10;

// Композиция: удвоить длину строки и добавить 10
const processString = pipe(
    getLength,
    double,
    add10
);

console.log(processString('hello')); // 5 * 2 + 10 = 20
```

### Продвинутая композиция

```javascript
// Улучшенная система композиции с обработкой ошибок
class FunctionComposer {
    constructor() {
        this.functions = [];
    }
    
    // Добавление функции в цепочку
    add(fn) {
        this.functions.push(fn);
        return this;
    }
    
    // Добавление асинхронной функции
    addAsync(asyncFn) {
        this.functions.push(async (value) => {
            const result = await asyncFn(value);
            return result;
        });
        return this;
    }
    
    // Условное добавление функции
    addIf(condition, fn) {
        if (condition) {
            this.add(fn);
        }
        return this;
    }
    
    // Компиляция в исполняемую функцию
    compile() {
        return (value) => {
            let result = value;
            for (const fn of this.functions) {
                result = fn(result);
            }
            return result;
        };
    }
    
    // Асинхронная компиляция
    async compileAsync() {
        return async (value) => {
            let result = value;
            for (const fn of this.functions) {
                result = await Promise.resolve(fn(result));
            }
            return result;
        };
    }
    
    // Сброс цепочки
    reset() {
        this.functions = [];
        return this;
    }
    
    // Получение количества функций
    get length() {
        return this.functions.length;
    }
}

// Использование FunctionComposer
const composer = new FunctionComposer();

const result = composer
    .add(str => str.trim())
    .add(str => str.toLowerCase())
    .add(str => str.replace(/\s+/g, '-'))
    .add(str => `processed-${str}`)
    .compile()('  Hello World  ');

console.log(result); // "processed-hello-world"

// Асинхронная цепочка
const asyncComposer = new FunctionComposer();

asyncComposer
    .add(str => str.trim())
    .addAsync(async str => {
        // Симуляция асинхронной операции
        await new Promise(resolve => setTimeout(resolve, 100));
        return str.toUpperCase();
    })
    .add(str => `ASYNC: ${str}`);

const asyncProcess = await asyncComposer.compileAsync();
const asyncResult = await asyncProcess('  hello  ');
console.log(asyncResult); // "ASYNC: HELLO"
```

## Функции высшего порядка

Функции высшего порядка - это функции, которые принимают другие функции в качестве аргументов или возвращают функции.

### Основы функций высшего порядка

```javascript
// Функция, возвращающая функцию
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(4)); // 12

// Функция, принимающая функцию
function applyOperation(numbers, operation) {
    return numbers.map(operation);
}

const numbers = [1, 2, 3, 4, 5];
const doubled = applyOperation(numbers, x => x * 2);
const squared = applyOperation(numbers, x => x * x);

console.log(doubled); // [2, 4, 6, 8, 10]
console.log(squared); // [1, 4, 9, 16, 25]

// Более сложные функции высшего порядка
function memoize(fn) {
    const cache = new Map();
    return function(...args) {
        const key = JSON.stringify(args);
        if (cache.has(key)) {
            return cache.get(key);
        }
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

function debounce(fn, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => fn.apply(this, args), delay);
    };
}

function throttle(fn, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            fn.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}
```

### Продвинутые функции высшего порядка

```javascript
// Система middleware функций высшего порядка
class MiddlewarePipeline {
    constructor() {
        this.middlewares = [];
    }
    
    use(middleware) {
        this.middlewares.push(middleware);
        return this;
    }
    
    async run(context) {
        let index = 0;
        
        const dispatch = async (i) => {
            if (i === this.middlewares.length) return Promise.resolve();
            
            const middleware = this.middlewares[i];
            if (index >= i) return Promise.reject(new Error('next() called multiple times'));
            
            index = i;
            return Promise.resolve(middleware(context, dispatch.bind(null, i + 1)));
        };
        
        return dispatch(0);
    }
}

// Пример использования middleware
const pipeline = new MiddlewarePipeline();

pipeline
    .use(async (ctx, next) => {
        console.log('1. Начало обработки');
        await next();
        console.log('1. Конец обработки');
    })
    .use(async (ctx, next) => {
        console.log('2. Проверка аутентификации');
        ctx.authenticated = true;
        await next();
    })
    .use(async (ctx, next) => {
        console.log('3. Логирование');
        ctx.timestamp = new Date();
        await next();
    });

const context = { request: { url: '/api/data' } };
pipeline.run(context).then(() => {
    console.log('Контекст после обработки:', context);
});

// Функция высшего порядка для создания валидаторов
function createValidator(rules) {
    return function(data) {
        const errors = [];
        
        for (const [field, rule] of Object.entries(rules)) {
            const value = getNestedValue(data, field);
            
            if (rule.required && (value === undefined || value === null || value === '')) {
                errors.push(`${field} is required`);
                continue;
            }
            
            if (rule.type && typeof value !== rule.type) {
                errors.push(`${field} must be of type ${rule.type}`);
            }
            
            if (rule.min && typeof value === 'number' && value < rule.min) {
                errors.push(`${field} must be at least ${rule.min}`);
            }
            
            if (rule.pattern && typeof value === 'string' && !rule.pattern.test(value)) {
                errors.push(`${field} format is invalid`);
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    };
}

function getNestedValue(obj, path) {
    return path.split('.').reduce((current, key) => current?.[key], obj);
}

// Использование валидатора
const userValidator = createValidator({
    'user.name': { required: true, type: 'string' },
    'user.email': { required: true, type: 'string', pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ },
    'user.age': { type: 'number', min: 0 }
});

const result = userValidator({
    user: { name: 'John', email: 'john@example.com', age: 25 }
});

console.log('Результат валидации:', result);
```

## Функторы и монады

Функторы и монады - это понятия из функционального программирования, которые помогают работать с контекстами и обрабатывать значения в определенных структурах данных.

### Функторы

```javascript
// Базовый интерфейс функтора
class Functor {
    map(fn) {
        throw new Error('Метод map должен быть реализован');
    }
}

// Maybe функтор для обработки null/undefined
class Maybe extends Functor {
    constructor(value) {
        super();
        this.value = value;
    }
    
    static of(value) {
        return new Maybe(value);
    }
    
    static nothing() {
        return new Maybe(null);
    }
    
    static fromNullable(value) {
        return value != null ? Maybe.of(value) : Maybe.nothing();
    }
    
    map(fn) {
        return this.value != null ? Maybe.of(fn(this.value)) : Maybe.nothing();
    }
    
    flatMap(fn) {
        return this.value != null ? fn(this.value) : Maybe.nothing();
    }
    
    getOrElse(defaultValue) {
        return this.value != null ? this.value : defaultValue;
    }
    
    filter(predicate) {
        return this.value != null && predicate(this.value) ? this : Maybe.nothing();
    }
    
    isNothing() {
        return this.value == null;
    }
    
    toString() {
        return this.value != null ? `Maybe(${this.value})` : 'Maybe(Nothing)';
    }
}

// Использование Maybe
const result1 = Maybe.fromNullable('Hello')
    .map(str => str.toUpperCase())
    .map(str => str + '!')
    .getOrElse('Default');

console.log(result1); // "HELLO!"

const result2 = Maybe.fromNullable(null)
    .map(str => str.toUpperCase())
    .getOrElse('Default');

console.log(result2); // "Default"

// Either монада для обработки ошибок
class Either extends Functor {
    constructor(value) {
        super();
        this.value = value;
    }
    
    static of(value) {
        return new Right(value);
    }
    
    static left(value) {
        return new Left(value);
    }
    
    static right(value) {
        return new Right(value);
    }
}

class Right extends Either {
    map(fn) {
        return Either.of(fn(this.value));
    }
    
    flatMap(fn) {
        return fn(this.value);
    }
    
    getOrElse() {
        return this.value;
    }
    
    toString() {
        return `Right(${this.value})`;
    }
}

class Left extends Either {
    map(fn) {
        return this;
    }
    
    flatMap(fn) {
        return this;
    }
    
    getOrElse(defaultValue) {
        return defaultValue;
    }
    
    toString() {
        return `Left(${this.value})`;
    }
}

// Использование Either
const divide = (a, b) => {
    return b === 0 ? Either.left('Division by zero') : Either.right(a / b);
};

const result3 = divide(10, 2)
    .map(x => x * 2)
    .map(x => x + 1)
    .getOrElse(0);

console.log(result3); // 11

const result4 = divide(10, 0)
    .map(x => x * 2)
    .getOrElse('Error occurred');

console.log(result4); // "Error occurred"
```

### Работа с массивами функционально

```javascript
// Функциональные утилиты для работы с массивами
const ArrayUtils = {
    // fmap для массивов (тоже самое что map)
    fmap: curry((fn, arr) => arr.map(fn)),
    
    // filter
    filter: curry((predicate, arr) => arr.filter(predicate)),
    
    // reduce
    reduce: curry((reducer, initialValue, arr) => arr.reduce(reducer, initialValue)),
    
    // flatMap (bind для массивов)
    flatMap: curry((fn, arr) => arr.flatMap(fn)),
    
    // concat
    concat: curry((arr1, arr2) => arr1.concat(arr2)),
    
    // take
    take: curry((n, arr) => arr.slice(0, n)),
    
    // drop
    drop: curry((n, arr) => arr.slice(n)),
    
    // find
    find: curry((predicate, arr) => arr.find(predicate)),
    
    // some
    some: curry((predicate, arr) => arr.some(predicate)),
    
    // every
    every: curry((predicate, arr) => arr.every(predicate)),
    
    // uniq - удаление дубликатов
    uniq: arr => [...new Set(arr)],
    
    // uniqBy - удаление дубликатов по критерию
    uniqBy: curry((keyFn, arr) => {
        const seen = new Set();
        return arr.filter(item => {
            const key = keyFn(item);
            if (seen.has(key)) {
                return false;
            }
            seen.add(key);
            return true;
        });
    }),
    
    // groupBy - группировка по критерию
    groupBy: curry((keyFn, arr) => {
        return arr.reduce((groups, item) => {
            const key = keyFn(item);
            if (!groups[key]) {
                groups[key] = [];
            }
            groups[key].push(item);
            return groups;
        }, {});
    }),
    
    // partition - разделение на два массива
    partition: curry((predicate, arr) => {
        return arr.reduce((acc, item) => {
            const target = predicate(item) ? acc[0] : acc[1];
            target.push(item);
            return acc;
        }, [[], []]);
    })
};

// Примеры использования
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Цепочка функциональных операций
const result = pipe(
    ArrayUtils.filter(x => x % 2 === 0), // четные числа
    ArrayUtils.fmap(x => x * x),         // в квадрат
    ArrayUtils.reduce((sum, x) => sum + x, 0) // сумма
)(numbers);

console.log(result); // 2² + 4² + 6² + 8² + 10² = 4 + 16 + 36 + 64 + 100 = 220

// Работа с объектами
const users = [
    { id: 1, name: 'Alice', age: 25, active: true },
    { id: 2, name: 'Bob', age: 30, active: false },
    { id: 3, name: 'Charlie', age: 35, active: true },
    { id: 4, name: 'Alice', age: 28, active: true }
];

// Группировка пользователей по имени
const groupedByName = ArrayUtils.groupBy(user => user.name, users);
console.log('Группировка по имени:', groupedByName);

// Уникальные пользователи по ID
const uniqueUsers = ArrayUtils.uniqBy(user => user.id, users);
console.log('Уникальные пользователи:', uniqueUsers);

// Разделение на активных и неактивных
const [activeUsers, inactiveUsers] = ArrayUtils.partition(user => user.active, users);
console.log('Активные:', activeUsers);
console.log('Неактивные:', inactiveUsers);
```

## Примеры из промышленной разработки

### Функциональная система обработки данных

```javascript
// Функциональная система обработки данных
class FunctionalDataProcessor {
    constructor() {
        this.transformers = [];
        this.validators = [];
        this.errorHandlers = [];
    }
    
    // Добавление трансформера
    addTransformer(transformer) {
        this.transformers.push(transformer);
        return this;
    }
    
    // Добавление валидатора
    addValidator(validator) {
        this.validators.push(validator);
        return this;
    }
    
    // Добавление обработчика ошибок
    addErrorHandler(handler) {
        this.errorHandlers.push(handler);
        return this;
    }
    
    // Обработка данных
    process(data) {
        try {
            // Валидация
            for (const validator of this.validators) {
                const validation = validator(data);
                if (!validation.isValid) {
                    throw new Error(`Validation failed: ${validation.errors.join(', ')}`);
                }
            }
            
            // Трансформация
            let result = data;
            for (const transformer of this.transformers) {
                result = transformer(result);
            }
            
            return Maybe.of(result);
        } catch (error) {
            // Обработка ошибок
            for (const handler of this.errorHandlers) {
                handler(error, data);
            }
            return Maybe.nothing();
        }
    }
    
    // Асинхронная обработка
    async processAsync(data) {
        try {
            // Асинхронная валидация
            for (const validator of this.validators) {
                if (validator.async) {
                    const validation = await validator(data);
                    if (!validation.isValid) {
                        throw new Error(`Async validation failed: ${validation.errors.join(', ')}`);
                    }
                }
            }
            
            // Асинхронная трансформация
            let result = data;
            for (const transformer of this.transformers) {
                if (transformer.async) {
                    result = await transformer(result);
                } else {
                    result = transformer(result);
                }
            }
            
            return Either.right(result);
        } catch (error) {
            return Either.left(error);
        }
    }
}

// Создание процессора
const processor = new FunctionalDataProcessor();

// Добавление трансформеров
processor
    .addTransformer(data => ({ ...data, processedAt: new Date().toISOString() }))
    .addTransformer(data => ({ ...data, id: data.id || Math.random().toString(36).substr(2, 9) }))
    .addTransformer(data => ({ ...data, status: 'processed' }));

// Добавление валидаторов
processor
    .addValidator(data => {
        const errors = [];
        if (!data.name) errors.push('Name is required');
        if (!data.email) errors.push('Email is required');
        return { isValid: errors.length === 0, errors };
    });

// Добавление обработчиков ошибок
processor
    .addErrorHandler((error, data) => {
        console.error('Processing error:', error.message, 'for data:', data);
    });

// Использование
const result = processor.process({
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
});

console.log('Результат обработки:', result.getOrElse('Processing failed'));
```

### Функциональная система маршрутизации

```javascript
// Функциональная система маршрутизации
class FunctionalRouter {
    constructor() {
        this.routes = [];
    }
    
    addRoute(method, path, handler) {
        this.routes.push({ method, path: this.compilePath(path), handler });
        return this;
    }
    
    compilePath(path) {
        // Конвертация строкового пути в регулярное выражение
        const regex = path
            .replace(/:[^\/]+/g, '([^/]+)') // параметры :param
            .replace(/\//g, '\\/'); // экранирование слешей
        
        return {
            pattern: new RegExp(`^${regex}$`),
            paramNames: (path.match(/:[^\/]+/g) || []).map(p => p.slice(1))
        };
    }
    
    match(method, url) {
        for (const route of this.routes) {
            if (route.method === method) {
                const matches = url.match(route.path.pattern);
                if (matches) {
                    const params = {};
                    route.path.paramNames.forEach((name, index) => {
                        params[name] = matches[index + 1];
                    });
                    
                    return {
                        handler: route.handler,
                        params
                    };
                }
            }
        }
        return null;
    }
    
    handleRequest(method, url, request) {
        const match = this.match(method, url);
        if (match) {
            return match.handler({ ...request, params: match.params });
        }
        return Either.left(new Error(`Route not found: ${method} ${url}`));
    }
    
    // Композиция middleware
    applyMiddleware(...middlewares) {
        return (handler) => {
            return middlewares.reverse().reduce((acc, middleware) => {
                return (request) => middleware(acc, request);
            }, handler);
        };
    }
}

// Пример использования
const router = new FunctionalRouter();

// Добавление маршрутов
router
    .addRoute('GET', '/', (req) => ({ message: 'Hello World' }))
    .addRoute('GET', '/users/:id', (req) => ({ 
        message: `User ${req.params.id}`, 
        id: req.params.id 
    }))
    .addRoute('POST', '/users', (req) => ({
        message: 'User created',
        data: req.body
    }));

// Пример middleware
const authMiddleware = (handler, request) => {
    // Проверка аутентификации
    if (request.headers && request.headers.authorization) {
        return handler(request);
    }
    return Either.left(new Error('Unauthorized'));
};

const logMiddleware = (handler, request) => {
    console.log(`${request.method} ${request.url}`);
    return handler(request);
};

// Применение middleware
const enhancedHandler = router.applyMiddleware(logMiddleware, authMiddleware)(
    (req) => router.handleRequest(req.method, req.url, req)
);

// Использование
const request = {
    method: 'GET',
    url: '/users/123',
    headers: { authorization: 'Bearer token' }
};

const response = enhancedHandler(request);
console.log('Response:', response);
```

## Лучшие практики

### 1. Иммутабельность данных

```javascript
// Использование freeze для дополнительной защиты
function deepFreeze(obj) {
    Object.getOwnPropertyNames(obj).forEach(prop => {
        if (obj[prop] !== null && typeof obj[prop] === 'object') {
            deepFreeze(obj[prop]);
        }
    });
    return Object.freeze(obj);
}

const config = deepFreeze({
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    retries: 3
});
```

### 2. Чистые функции

```javascript
// Избегаем мутаций
const addItem = curry((item, array) => [...array, item]);
const removeItem = curry((index, array) => 
    array.filter((_, i) => i !== index)
);
const updateItem = curry((index, updater, array) => 
    array.map((item, i) => i === index ? updater(item) : item)
);
```

### 3. Обработка ошибок

```javascript
// Использование Either для обработки ошибок
const safeDivide = (a, b) => 
    b === 0 ? Either.left('Division by zero') : Either.right(a / b);

const calculate = (x, y, z) => 
    Either.of(x)
        .flatMap(val => safeDivide(val, y))
        .flatMap(val => safeDivide(val, z))
        .getOrElse(0);
```

## Безопасность

При использовании функциональных подходов важно учитывать безопасность:

```javascript
// Безопасная обработка пользовательского ввода функционально
const SecurityUtils = {
    // Функция для очистки пользовательского ввода
    sanitizeInput: curry((input, allowedTags = []) => {
        if (typeof input !== 'string') return input;
        
        // Простая очистка (в продакшене используйте проверенные библиотеки)
        return input
            .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
            .replace(/javascript:/gi, '')
            .replace(/vbscript:/gi, '')
            .replace(/on\w+\s*=/gi, '');
    }),
    
    // Проверка на XSS
    isSafeString: curry((str) => {
        if (typeof str !== 'string') return true;
        
        const dangerousPatterns = [
            /<script/i,
            /javascript:/i,
            /vbscript:/i,
            /on\w+\s*=/i
        ];
        
        return !dangerousPatterns.some(pattern => pattern.test(str));
    }),
    
    // Безопасная валидация
    safeValidate: curry((validator, data) => {
        try {
            return validator(data);
        } catch (error) {
            console.error('Validation error:', error);
            return Either.left(error);
        }
    })
};

// Использование
const userInput = '<script>alert("XSS")</script>Hello World';
const sanitized = SecurityUtils.sanitizeInput(userInput);
console.log('Очищенный ввод:', sanitized); // "Hello World"
```

## Теги

#javascript #functional-programming #pure-functions #immutability #currying #composition #higher-order-functions #functors #monads #fp