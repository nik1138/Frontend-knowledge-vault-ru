# Замыкания в JavaScript: Функциональное программирование и глубокие паттерны

## Обзор

Замыкание (closure) - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Замыкания позволяют функции получать доступ к переменным из внешней (обрамляющей) функции даже после того, как внешняя функция завершила выполнение. Замыкания являются фундаментальной концепцией в функциональном программировании и позволяют реализовать множество мощных паттернов.

## Как работают замыкания

```javascript
function outerFunction(x) {
    // Внешняя функция объявляет переменную
    let outerVariable = x;

    // Внутренняя функция имеет доступ к переменной внешней функции
    function innerFunction(y) {
        return outerVariable + y;
    }

    // Возвращаем внутреннюю функцию
    return innerFunction;
}

const closureExample = outerFunction(10);
console.log(closureExample(5)); // 15
```

В этом примере `innerFunction` сохраняет доступ к `outerVariable` даже после того, как `outerFunction` завершила выполнение.

## Практические примеры замыканий

### 1. Приватные переменные и методы

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
console.log(counter.decrement()); // 1
console.log(counter.getCount());  // 1
// count недоступна напрямую: counter.count возвращает undefined
```

### 2. Функции-фабрики

```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### 3. Каррирование

```javascript
function add(a) {
    return function(b) {
        return a + b;
    };
}

const addFive = add(5);
console.log(addFive(3)); // 8

// Или в виде стрелочной функции
const multiply = a => b => a * b;
const multiplyByTen = multiply(10);
console.log(multiplyByTen(7)); // 70
```

### 4. Обработчики событий

```javascript
function setupButtons() {
    for (let i = 0; i < 3; i++) {
        // Каждая итерация создает замыкание, захватывающее текущее значение i
        setTimeout(() => {
            console.log(`Кнопка ${i} нажата`);
        }, 1000);
    }
}

setupButtons(); // Выведет: "Кнопка 0 нажата", "Кнопка 1 нажата", "Кнопка 2 нажата"
```

## Распространенные ошибки

### Неправильное использование замыканий в циклах с var

```javascript
// ПЛОХО: использование var в цикле
function badExample() {
    for (var i = 0; i < 3; i++) {
        setTimeout(function() {
            console.log(i); // Выведет 3, 3, 3
        }, 100);
    }
}

// ХОРОШО: использование let или замыкания
function goodExample1() {
    for (let i = 0; i < 3; i++) {
        setTimeout(function() {
            console.log(i); // Выведет 0, 1, 2
        }, 100);
    }
}

function goodExample2() {
    for (var i = 0; i < 3; i++) {
        (function(index) {
            setTimeout(function() {
                console.log(index); // Выведет 0, 1, 2
            }, 100);
        })(i);
    }
}
```

## Преимущества замыканий

1. **Инкапсуляция**: Позволяют создавать приватные переменные и методы
2. **Сохранение состояния**: Функции могут сохранять состояние между вызовами
3. **Модульность**: Помогают создавать более модульный и повторно используемый код

## Возможные проблемы

1. **Утечки памяти**: Неправильное использование замыканий может привести к утечкам памяти
2. **Производительность**: Замыкания могут потреблять больше памяти, чем обычные функции
3. **Сложность отладки**: Сложные цепочки замыканий могут быть трудны для понимания и отладки

## Продвинутые паттерны с замыканиями

### Функции высшего порядка и замыкания

```javascript
// Функция, возвращающая функцию-валидатор
const createValidator = (validationFn) => (errorMessage) => (value) => {
    if (!validationFn(value)) {
        throw new Error(errorMessage);
    }
    return value;
};

// Создание специфичных валидаторов
const isString = createValidator(val => typeof val === 'string')('Значение должно быть строкой');
const isPositiveNumber = createValidator(val => typeof val === 'number' && val > 0)('Число должно быть положительным');

// Использование валидаторов
try {
    console.log(isString("hello")); // "hello"
    console.log(isPositiveNumber(5)); // 5
    // console.log(isPositiveNumber(-1)); // Ошибка
} catch (error) {
    console.error(error.message);
}
```

### Композиция функций с замыканиями

```javascript
// Утилита для композиции функций
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

// Утилита для последовательного выполнения (pipe)
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

// Функции, использующие замыкания для создания частично примененных функций
const multiplyBy = factor => value => value * factor;
const add = num => value => value + num;
const power = exp => value => Math.pow(value, exp);

// Пример композиции
const complexOperation = pipe(
    add(10),          // 5 + 10 = 15
    multiplyBy(2),    // 15 * 2 = 30
    power(2)          // 30^2 = 900
);

console.log(complexOperation(5)); // 900
```

### Мемоизация с замыканиями

```javascript
// Простая мемоизация
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

// Функция с вычислительно сложной операцией
const fibonacci = memoize((n) => {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // 55
console.log(fibonacci(10)); // 55 (из кэша)
```

### Продвинутая мемоизация с ограничениями

```javascript
// Мемоизация с ограничением размера кэша (LRU)
const memoizeWithLimit = (fn, limit = 100) => {
    const cache = new Map();
    
    return (...args) => {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            // Перемещаем элемент в конец (как недавно использованный)
            const value = cache.get(key);
            cache.delete(key);
            cache.set(key, value);
            return value;
        }
        
        const result = fn(...args);
        
        if (cache.size >= limit) {
            // Удаляем первый элемент (самый старый)
            const firstKey = cache.keys().next().value;
            cache.delete(firstKey);
        }
        
        cache.set(key, result);
        return result;
    };
};

// Пример использования
const expensiveOperation = memoizeWithLimit(x => {
    console.log(`Вычисление для ${x}...`);
    return x * x;
}, 3);

console.log(expensiveOperation(1)); // Вычисление для 1... 1
console.log(expensiveOperation(2)); // Вычисление для 2... 4
console.log(expensiveOperation(3)); // Вычисление для 3... 9
console.log(expensiveOperation(1)); // 1 (из кэша)
console.log(expensiveOperation(4)); // Вычисление для 4... 16 (удаляет 2 из кэша)
console.log(expensiveOperation(2)); // Вычисление для 2... 4 (2 был удален)
```

### Замыкания для управления состоянием

```javascript
// Паттерн "состояние" с замыканиями
const createStateManager = (initialState = {}) => {
    let state = { ...initialState };
    const listeners = [];
    
    return {
        getState: () => ({ ...state }),
        setState: (newState) => {
            const prevState = { ...state };
            state = { ...state, ...newState };
            
            // Уведомляем слушателей об изменении состояния
            listeners.forEach(listener => listener(state, prevState));
        },
        subscribe: (listener) => {
            listeners.push(listener);
            // Возвращаем функцию отписки
            return () => {
                const index = listeners.indexOf(listener);
                if (index > -1) {
                    listeners.splice(index, 1);
                }
            };
        }
    };
};

// Использование
const store = createStateManager({ count: 0, name: 'Тест' });

const unsubscribe = store.subscribe((newState, prevState) => {
    console.log('Состояние изменилось:', prevState, '->', newState);
});

store.setState({ count: 5 });
store.setState({ name: 'Новое имя' });

console.log(store.getState()); // { count: 5, name: 'Новое имя' }
unsubscribe(); // Отписываемся от изменений
```

### Промисы и замыкания

```javascript
// Замыкания для управления асинхронными операциями
const createAsyncCache = () => {
    const cache = new Map();
    
    return {
        async get(key, fetcher) {
            if (cache.has(key)) {
                return cache.get(key);
            }
            
            const promise = fetcher(key);
            cache.set(key, promise);
            
            try {
                const result = await promise;
                return result;
            } catch (error) {
                cache.delete(key);
                throw error;
            }
        },
        
        clear() {
            cache.clear();
        }
    };
};

// Использование
const asyncCache = createAsyncCache();

const fetchData = async (id) => {
    // Симуляция асинхронной операции
    await new Promise(resolve => setTimeout(resolve, 1000));
    return `Данные для ${id}`;
};

// Первый вызов - ожидание 1 секунды
asyncCache.get('user1', fetchData).then(data => console.log(data));

// Второй вызов - немедленный результат из кэша
asyncCache.get('user1', fetchData).then(data => console.log(data));
```

### Функции-предикаты и замыкания

```javascript
// Создание предикатов с замыканиями
const createPredicate = (comparator, value) => (input) => comparator(input, value);

// Основные компараторы
const equals = (a, b) => a === b;
const greaterThan = (a, b) => a > b;
const lessThan = (a, b) => a < b;
const contains = (a, b) => a.includes && a.includes(b);

// Создание предикатов
const isFive = createPredicate(equals, 5);
const isGreaterThanTen = createPredicate(greaterThan, 10);
const containsHello = createPredicate(contains, 'hello');

console.log(isFive(5)); // true
console.log(isGreaterThanTen(15)); // true
console.log(containsHello('hello world')); // true

// Комбинирование предикатов
const and = (...predicates) => (value) => predicates.every(p => p(value));
const or = (...predicates) => (value) => predicates.some(p => p(value));

const isPositiveEven = and(
    createPredicate(greaterThan, 0),
    n => n % 2 === 0
);

console.log(isPositiveEven(4)); // true
console.log(isPositiveEven(3)); // false
```

## Функциональное программирование с замыканиями

### Чистые функции и замыкания

```javascript
// Чистая функция, использующая замыкание для сохранения конфигурации
const createFormatter = (config) => {
    const { prefix = '', suffix = '', transform = x => x } = config;
    
    // Возвращаемая функция - чистая, если transform - чистая функция
    return (value) => `${prefix}${transform(value)}${suffix}`;
};

const dollarFormatter = createFormatter({ 
    prefix: '$', 
    transform: value => value.toFixed(2) 
});

const upperCaseFormatter = createFormatter({ 
    transform: value => value.toUpperCase() 
});

console.log(dollarFormatter(123.4)); // $123.40
console.log(upperCaseFormatter('hello')); // HELLO
```

### Каррирование и частичное применение

```javascript
// Универсальное каррирование
const curry = (fn) => {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function (...moreArgs) {
                return curried.apply(this, args.concat(moreArgs));
            };
        }
    };
};

// Пример функции для каррирования
const multiply = (a, b, c) => a * b * c;

// Каррированная версия
const curriedMultiply = curry(multiply);

// Частичное применение
const multiplyBy10 = curriedMultiply(10);
const multiplyBy10And5 = multiplyBy10(5);

console.log(multiplyBy10And5(2)); // 100 (10 * 5 * 2)

// Или в один шаг
console.log(curriedMultiply(2)(3)(4)); // 24
```

### Функции для работы с коллекциями

```javascript
// Функции высшего порядка для работы с коллекциями
const map = fn => arr => arr.map(fn);
const filter = predicate => arr => arr.filter(predicate);
const reduce = (reducer, initial) => arr => arr.reduce(reducer, initial);

// Создание специфичных обработчиков с замыканиями
const createFieldExtractor = fieldName => obj => obj[fieldName];
const createFieldFilter = (fieldName, value) => obj => obj[fieldName] === value;

// Пример использования
const users = [
    { id: 1, name: 'Иван', age: 25, active: true },
    { id: 2, name: 'Мария', age: 30, active: false },
    { id: 3, name: 'Петр', age: 35, active: true }
];

const activeUsers = pipe(
    filter(createFieldFilter('active', true)),
    map(createFieldExtractor('name'))
)(users);

console.log(activeUsers); // ['Иван', 'Петр']
```

## Практические примеры из промышленной разработки

### Middleware паттерн

```javascript
// Паттерн middleware с использованием замыканий
const applyMiddleware = (...middlewares) => {
    return (handler) => {
        return middlewares.reverse().reduce((acc, middleware) => {
            return middleware(acc);
        }, handler);
    };
};

// Пример middleware функций
const logger = (handler) => (...args) => {
    console.log('Вызов функции с аргументами:', args);
    const result = handler(...args);
    console.log('Результат:', result);
    return result;
};

const validator = (handler) => (...args) => {
    if (args.some(arg => arg === null || arg === undefined)) {
        throw new Error('Аргументы не должны быть null или undefined');
    }
    return handler(...args);
};

const timer = (handler) => (...args) => {
    const start = Date.now();
    const result = handler(...args);
    const end = Date.now();
    console.log(`Выполнение заняло ${end - start} мс`);
    return result;
};

// Создание обработчика с middleware
const enhancedHandler = applyMiddleware(logger, validator, timer)((a, b) => a + b);

console.log(enhancedHandler(5, 3)); // 8
```

### Фабрики компонентов

```javascript
// Фабрика компонентов с замыканиями
const createComponent = (tagName, defaultProps = {}) => {
    return (props = {}, children = '') => {
        const finalProps = { ...defaultProps, ...props };
        const propsString = Object.entries(finalProps)
            .map(([key, value]) => `${key}="${value}"`)
            .join(' ');
        
        return `<${tagName} ${propsString}>${children}</${tagName}>`;
    };
};

// Создание специфичных компонентов
const createButton = createComponent('button', { type: 'button' });
const createInput = createComponent('input', { type: 'text' });

console.log(createButton({ class: 'btn-primary' }, 'Нажми меня'));
// <button type="button" class="btn-primary">Нажми меня</button>

console.log(createInput({ placeholder: 'Введите текст', id: 'input-field' }));
// <input type="text" placeholder="Введите текст" id="input-field">
```

### Управление асинхронными операциями

```javascript
// Паттерн для управления асинхронными операциями с замыканиями
const createAsyncController = () => {
    let abortController = null;
    
    return {
        async execute(asyncFn, ...args) {
            // Отменяем предыдущую операцию, если она есть
            if (abortController) {
                abortController.abort();
            }
            
            // Создаем новый контроллер
            abortController = new AbortController();
            
            try {
                // Передаем сигнал отмены в асинхронную функцию
                return await asyncFn(abortController.signal, ...args);
            } catch (error) {
                if (error.name === 'AbortError') {
                    console.log('Операция была отменена');
                }
                throw error;
            }
        },
        
        cancel() {
            if (abortController) {
                abortController.abort();
            }
        }
    };
};

// Пример асинхронной функции с поддержкой отмены
const asyncOperation = async (signal, delay) => {
    return new Promise((resolve, reject) => {
        const timeout = setTimeout(() => {
            resolve(`Операция завершена через ${delay}мс`);
        }, delay);
        
        signal.addEventListener('abort', () => {
            clearTimeout(timeout);
            reject(new Error('AbortError'));
        });
    });
};

// Использование
const controller = createAsyncController();

controller.execute(asyncOperation, 2000)
    .then(result => console.log(result))
    .catch(error => console.error(error.message));

// Отмена операции через 1 секунду
setTimeout(() => controller.cancel(), 1000);
```

## Лучшие практики и советы по производительности

### Оптимизация замыканий

```javascript
// Избегайте создания замыканий в циклах без необходимости
const handlers = [];

// Плохо: создание замыкания в каждом шаге цикла
for (let i = 0; i < 1000; i++) {
    handlers.push(() => console.log(i)); // Каждый handler захватывает свою i
}

// Лучше: создание внешней функции
const createHandler = (index) => () => console.log(index);
const betterHandlers = [];
for (let i = 0; i < 1000; i++) {
    betterHandlers.push(createHandler(i));
}

// Использование bind для избежания замыканий
const data = [1, 2, 3, 4, 5];
const multipliers = [2, 3, 4];

// Плохо: замыкание для каждой операции
const badResults = data.map(x => multipliers.map(m => x * m));

// Лучше: вынесение функции
const multiplyBy = (multiplier) => (value) => value * multiplier;
const goodResults = data.map(x => multipliers.map(multiplyBy(x)));
```

### Управление памятью

```javascript
// Правильное управление памятью с замыканиями
const createMemoryEfficientProcessor = () => {
    let largeData = new Array(1000000).fill(0); // Большие данные
    
    return {
        process: () => {
            // Обработка только небольшой части данных
            const smallResult = largeData.slice(0, 10).map(x => x + 1);
            return smallResult;
        },
        
        cleanup: () => {
            // Очистка больших данных, которые больше не нужны
            largeData = null;
        }
    };
};

const processor = createMemoryEfficientProcessor();
const result = processor.process();
processor.cleanup(); // Освобождение памяти
```

### Использование современных возможностей

```javascript
// Использование WeakMap для избежания утечек памяти
const privateData = new WeakMap();

class MyClass {
    constructor(value) {
        // Приватные данные, которые не будут препятствовать сборке мусора
        privateData.set(this, { value, processed: false });
    }
    
    getValue() {
        return privateData.get(this).value;
    }
    
    process() {
        const data = privateData.get(this);
        data.processed = true;
        data.value = data.value * 2;
    }
}

// Использование замыканий с современными возможностями
const createConfigurableFunction = (options = {}) => {
    const {
        multiplier = 1,
        addend = 0,
        transformer = x => x
    } = options;
    
    // Замыкание захватывает только необходимые переменные
    return (value) => transformer((value * multiplier) + addend);
};

const specialFunction = createConfigurableFunction({
    multiplier: 2,
    addend: 10,
    transformer: x => Math.round(x)
});

console.log(specialFunction(5)); // Math.round((5 * 2) + 10) = 20
```

## Заключение

Замыкания являются мощным инструментом в JavaScript, особенно в контексте функционального программирования. Они позволяют создавать гибкие, модульные и повторно используемые компоненты, обеспечивать инкапсуляцию данных и управлять состоянием приложений. Понимание продвинутых паттернов с замыканиями критически важно для профессиональной разработки на JavaScript.

#programming #javascript #closures #functional-programming #higher-order-functions #currying #memoization #web-development

## Обзор

Замыкание (closure) - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Замыкания позволяют функции получать доступ к переменным из внешней (обрамляющей) функции даже после того, как внешняя функция завершила выполнение.

## Как работают замыкания

```javascript
function outerFunction(x) {
    // Внешняя функция объявляет переменную
    let outerVariable = x;
    
    // Внутренняя функция имеет доступ к переменной внешней функции
    function innerFunction(y) {
        return outerVariable + y;
    }
    
    // Возвращаем внутреннюю функцию
    return innerFunction;
}

const closureExample = outerFunction(10);
console.log(closureExample(5)); // 15
```

В этом примере `innerFunction` сохраняет доступ к `outerVariable` даже после того, как `outerFunction` завершила выполнение.

## Практические примеры замыканий

### 1. Приватные переменные и методы

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
console.log(counter.decrement()); // 1
console.log(counter.getCount());  // 1
// count недоступна напрямую: counter.count возвращает undefined
```

### 2. Функции-фабрики

```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### 3. Каррирование

```javascript
function add(a) {
    return function(b) {
        return a + b;
    };
}

const addFive = add(5);
console.log(addFive(3)); // 8

// Или в виде стрелочной функции
const multiply = a => b => a * b;
const multiplyByTen = multiply(10);
console.log(multiplyByTen(7)); // 70
```

### 4. Обработчики событий

```javascript
function setupButtons() {
    for (let i = 0; i < 3; i++) {
        // Каждая итерация создает замыкание, захватывающее текущее значение i
        setTimeout(() => {
            console.log(`Кнопка ${i} нажата`);
        }, 1000);
    }
}

setupButtons(); // Выведет: "Кнопка 0 нажата", "Кнопка 1 нажата", "Кнопка 2 нажата"
```

## Распространенные ошибки

### Неправильное использование замыканий в циклах с var

```javascript
// ПЛОХО: использование var в цикле
function badExample() {
    for (var i = 0; i < 3; i++) {
        setTimeout(function() {
            console.log(i); // Выведет 3, 3, 3
        }, 100);
    }
}

// ХОРОШО: использование let или замыкания
function goodExample1() {
    for (let i = 0; i < 3; i++) {
        setTimeout(function() {
            console.log(i); // Выведет 0, 1, 2
        }, 100);
    }
}

function goodExample2() {
    for (var i = 0; i < 3; i++) {
        (function(index) {
            setTimeout(function() {
                console.log(index); // Выведет 0, 1, 2
            }, 100);
        })(i);
    }
}
```

## Преимущества замыканий

1. **Инкапсуляция**: Позволяют создавать приватные переменные и методы
2. **Сохранение состояния**: Функции могут сохранять состояние между вызовами
3. **Модульность**: Помогают создавать более модульный и повторно используемый код

## Возможные проблемы

1. **Утечки памяти**: Неправильное использование замыканий может привести к утечкам памяти
2. **Производительность**: Замыкания могут потреблять больше памяти, чем обычные функции
3. **Сложность отладки**: Сложные цепочки замыканий могут быть трудны для понимания и отладки