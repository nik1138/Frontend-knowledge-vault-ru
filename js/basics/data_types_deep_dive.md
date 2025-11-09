# Глубокое погружение в типы данных и преобразования типов JavaScript

## Обзор

JavaScript - это язык с динамической типизацией, где типы данных определяются во время выполнения. Понимание глубоких концепций типов данных, их внутреннего представления и механизмов преобразования критически важно для написания надежного и эффективного кода. В этой статье мы рассмотрим как примитивные, так и объектные типы, а также сложные механизмы преобразования типов.

## Примитивные типы данных и их особенности

### Числа и их внутреннее представление

JavaScript использует формат IEEE 754 для представления чисел с плавающей точкой двойной точности (64-битные числа):

```javascript
// Проблемы с точностью чисел
console.log(0.1 + 0.2 === 0.3); // false
console.log(0.1 + 0.2); // 0.30000000000000004

// Решение проблемы точности
function isEqual(a, b, epsilon = Number.EPSILON) {
    return Math.abs(a - b) < epsilon;
}

console.log(isEqual(0.1 + 0.2, 0.3)); // true

// Специальные числовые значения
console.log(1 / 0); // Infinity
console.log(-1 / 0); // -Infinity
console.log(0 / 0); // NaN (Not-a-Number)

// Проверка на NaN (Number.isNaN предпочтительнее)
console.log(isNaN(NaN)); // true
console.log(Number.isNaN(NaN)); // true (лучше использовать этот метод)
console.log(isNaN("hello")); // true (ложное срабатывание)
console.log(Number.isNaN("hello")); // false (корректная проверка)

// Проверка на конечность
console.log(isFinite(123)); // true
console.log(isFinite(Infinity)); // false
console.log(Number.isFinite(123)); // true

// Максимальное и минимальное безопасное целое
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991
console.log(Number.MIN_SAFE_INTEGER); // -9007199254740991

// Проверка на безопасное целое
console.log(Number.isSafeInteger(9007199254740991)); // true
console.log(Number.isSafeInteger(9007199254740992)); // false

// Создание чисел различными способами
const num1 = 42; // литерал
const num2 = Number("42"); // конструктор
const num3 = +"42"; // унарный плюс
const num4 = parseInt("42", 10); // парсинг
const num5 = parseFloat("42.5"); // парсинг с плавающей точкой
```

### BigInt и работа с большими числами

BigInt позволяет работать с целыми числами произвольной длины:

```javascript
// Создание BigInt
const bigNumber = 1234567890123456789012345678901234567890n;
const anotherBig = BigInt("123456789012345678901234567890");
const fromNumber = BigInt(123456); // из числа

// Операции с BigInt
const result = bigNumber + 10n; // нужно использовать суффикс n для чисел
console.log(result);

// BigInt нельзя сравнивать с обычными числами без преобразования
console.log(bigNumber > Number.MAX_SAFE_INTEGER); // true
// console.log(bigNumber > 12345678901234567890); // может вызвать потерю точности при преобразовании

// Преобразование BigInt к числу (с осторожностью!)
try {
    const normalNumber = Number(bigNumber);
    console.log(Number.isSafeInteger(normalNumber)); // false (может быть потеря точности)
} catch (e) {
    console.error("Ошибка преобразования BigInt к Number:", e.message);
}

// Арифметические операции с BigInt
const a = 10n;
const b = 5n;

console.log(a + b); // 15n
console.log(a - b); // 5n
console.log(a * b); // 50n
console.log(a / b); // 2n (деление нацело)
console.log(a % b); // 0n
console.log(a ** 2n); // 100n (возведение в степень)
```

### Строки: неизменяемость и внутренние оптимизации

```javascript
// Строки неизменяемы в JavaScript
let str = "hello";
str[0] = "H"; // не изменит строку
console.log(str); // "hello", а не "Hello"

// Оптимизация строк в движке V8
let s1 = "hello";
let s2 = "hello";
console.log(s1 === s2); // true (строки интернированы)

// Интернирование строк (внутренняя оптимизация)
// Движок может интернировать короткие строки для экономии памяти
const short = "abc";
const alsoShort = "abc";
console.log(Object.is(short, alsoShort)); // true в большинстве случаев

// Шаблонные строки и производительность
const name = "Иван";
const age = 30;

// Менее эффективно для множественной конкатенации
const greeting1 = "Привет, меня зовут " + name + ", мне " + age + " лет";

// Более эффективно
const greeting2 = `Привет, меня зовут ${name}, мне ${age} лет`;

// Для конкатенации большого количества строк лучше использовать массив
function inefficientStringConcat(items) {
    let result = "";
    for (let i = 0; i < items.length; i++) {
        result += items[i]; // создает новую строку на каждой итерации
    }
    return result;
}

function efficientStringConcat(items) {
    const parts = [];
    for (let i = 0; i < items.length; i++) {
        parts.push(items[i]);
    }
    return parts.join("");
}

// Методы строк и их особенности
const text = "  Hello, World!  ";
console.log(text.trim()); // "Hello, World!"
console.log(text.toUpperCase()); // "  HELLO, WORLD!  "
console.log(text.substring(2, 7)); // "Hello"
console.log(text.slice(-7, -1)); // "World!"

// Регулярные выражения со строками
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
console.log(emailRegex.test("user@example.com")); // true
console.log("hello world".replace(/\s/, "_")); // "hello_world"
```

### Символы и их применение

Символы позволяют создавать уникальные идентификаторы:

```javascript
// Символы для создания уникальных идентификаторов
const id1 = Symbol("id");
const id2 = Symbol("id");
console.log(id1 === id2); // false (символы уникальны)

// Символы как ключи свойств
const user = {
    name: "Иван"
};

const symbolKey = Symbol("id");
user[symbolKey] = 123;

console.log(user[symbolKey]); // 123
console.log(Object.keys(user)); // ["name"] (символьные ключи не перечисляются)
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id)]
console.log(Reflect.ownKeys(user)); // ["name", Symbol(id)] (все ключи)

// Общие символы (shared symbols)
const sharedId = Symbol.for("sharedId");
const anotherSharedId = Symbol.for("sharedId");
console.log(sharedId === anotherSharedId); // true

// Получение имени общего символа
console.log(Symbol.keyFor(sharedId)); // "sharedId"

// Известные символы (well-known symbols)
const objWithIterator = {
    items: [1, 2, 3],
    [Symbol.iterator]: function* () {
        for (const item of this.items) {
            yield item * 2;
        }
    }
};

console.log([...objWithIterator]); // [2, 4, 6]

// Символы для скрытых свойств
const hiddenProperty = Symbol('hidden');
class MyClass {
    constructor(value) {
        this[hiddenProperty] = value;
    }
    
    getValue() {
        return this[hiddenProperty];
    }
}

const instance = new MyClass('secret');
console.log(instance.getValue()); // 'secret'
console.log(Object.keys(instance)); // [] - символ не виден
```

## Объектные типы данных и их особенности

### Объекты и прототипы

```javascript
// Создание объектов различными способами
const obj1 = {}; // литерал
const obj2 = new Object(); // конструктор
const obj3 = Object.create(null); // без прототипа
const obj4 = Object.create(Object.prototype); // с прототипом Object

// Доступ к свойствам
const user = {
    name: "Иван",
    age: 30,
    "full name": "Иван Иванов", // свойство с пробелом
    42: "числовой ключ" // числовой ключ становится строкой
};

console.log(user.name); // "Иван"
console.log(user["full name"]); // "Иван Иванов"
console.log(user[42]); // "числовой ключ"
console.log(user["42"]); // "числовой ключ"

// Вычисляемые имена свойств
const propertyName = "dynamicProperty";
const obj = {
    [propertyName]: "значение",
    [`${propertyName}_computed`]: "вычисленное значение",
    [Symbol("sym")]: "символьное свойство"
};

// Деструктуризация объектов
const { name, age } = user;
const { name: userName, age: userAge } = user;

// Распространение (spread)
const extendedUser = { ...user, email: "ivan@example.com", active: true };

// Опциональная деструктуризация
const { address = "не указан", phone: phoneNumber = "не указан" } = user;

// Прототипы и наследование
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    console.log(`${this.name} издает звук`);
};

function Dog(name, breed) {
    Animal.call(this, name); // вызов родительского конструктора
    this.breed = breed;
}

// Установка прототипа
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function() {
    console.log(`${this.name} лает`);
};

const dog = new Dog("Бобик", "Лабрадор");
dog.speak(); // "Бобик лает"
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
```

### Массивы и их особенности

```javascript
// Создание массивов
const arr1 = [1, 2, 3];
const arr2 = new Array(3); // [empty × 3] - разреженный массив
const arr3 = Array.from({ length: 3 }, (_, i) => i); // [0, 1, 2]
const arr4 = Array.of(1, 2, 3); // [1, 2, 3] (в отличие от new Array(1, 2, 3))

// Разреженные массивы
const sparse = [1, , 3]; // [1, empty, 3]
console.log(sparse.length); // 3
console.log(sparse[1]); // undefined
console.log(1 in sparse); // false

// Плотные массивы
const dense = [1, undefined, 3];
console.log(1 in dense); // true

// Методы массивов и производительность
const numbers = [1, 2, 3, 4, 5];

// map возвращает новый массив
const doubled = numbers.map(n => n * 2);

// filter возвращает новый массив
const evens = numbers.filter(n => n % 2 === 0);

// reduce может быть более эффективным для сложных операций
const sum = numbers.reduce((acc, n) => acc + n, 0);

// forEach не возвращает значение
numbers.forEach(n => console.log(n));

// Проверка на массив
console.log(Array.isArray([])); // true
console.log(Array.isArray(new Array())); // true
console.log(Array.isArray({})); // false

// TypedArray для числовых данных
const intArray = new Int32Array([1, 2, 3, 4, 5]);
const floatArray = new Float64Array(10); // 10 элементов с 0

// ArrayBuffer для бинарных данных
const buffer = new ArrayBuffer(16);
const view = new DataView(buffer);
view.setInt32(0, 123456);
console.log(view.getInt32(0)); // 123456
```

## Механизмы преобразования типов

### Неявное преобразование (type coercion)

```javascript
// Правила неявного преобразования
console.log("" + 1 + 0); // "10" (строка + число = строка, затем конкатенация)
console.log("" - 1 + 0); // -1 (пустая строка становится 0)
console.log(true + false); // 1 (true=1, false=0)
console.log(6 / "3"); // 2 (строка преобразуется в число)
console.log("2" * "3"); // 6 (обе строки преобразуются в числа)
console.log(4 + 5 + "px"); // "9px" (4+5=9, затем 9+"px"="9px")
console.log("$" + 4 + 5); // "$45" (конкатенация слева направо)
console.log("4" - 2); // 2 (строка преобразуется в число)
console.log("4px" - 2); // NaN (нельзя преобразовать "4px" в число)
console.log("  -9  " + 5); // "  -9  5" (конкатенация)
console.log("  -9  " - 5); // -14 (преобразование в число)
console.log(null + 1); // 1 (null становится 0)
console.log(undefined + 1); // NaN (undefined становится NaN)

// Логические преобразования (falsy значения)
const falsyValues = [false, 0, -0, "", null, undefined, NaN];
falsyValues.forEach(val => console.log(`${val} -> Boolean: ${Boolean(val)}`));

// truthy значения
const truthyValues = [true, 1, "0", "false", [], {}, function() {}, new Date()];
truthyValues.forEach(val => console.log(`${val} -> Boolean: ${Boolean(val)}`));

// Преобразование объектов
const obj = {
    valueOf: () => 42,
    toString: () => "object"
};

console.log(obj + 1); // 43 (использует valueOf для численных операций)
console.log(String(obj)); // "object" (использует toString для строковых операций)
console.log(obj == "object"); // false (использует valueOf для ==)
console.log(obj == 42); // true (использует valueOf для ==)

// Пользовательские методы преобразования
const customObj = {
    [Symbol.toPrimitive](hint) {
        if (hint === "number") {
            return 100;
        } else if (hint === "string") {
            return "custom";
        } else {
            return true; // по умолчанию
        }
    }
};

console.log(Number(customObj)); // 100
console.log(String(customObj)); // "custom"
console.log(Boolean(customObj)); // true
```

### Явное преобразование типов

```javascript
// Явное преобразование с использованием конструкторов
const strNum = "123";
const num = Number(strNum); // 123
const bool = Boolean(strNum); // true (непустая строка)
const str = String(num); // "123"

// Преобразование в число
console.log(Number("123")); // 123
console.log(+"123"); // 123 (унарный плюс)
console.log(parseInt("123", 10)); // 123
console.log(parseFloat("123.45")); // 123.45

// Преобразование в логическое
console.log(Boolean("")); // false
console.log(Boolean("0")); // true (непустая строка!)
console.log(Boolean(0)); // false
console.log(Boolean({})); // true (объект существует)
console.log(Boolean([])); // true (массив существует)

// Преобразование в строку
console.log(String(123)); // "123"
console.log((123).toString()); // "123"
console.log(123 + ""); // "123"

// Преобразование объектов в массивы
const objToArray = { a: 1, b: 2, c: 3 };
console.log(Object.keys(objToArray)); // ["a", "b", "c"]
console.log(Object.values(objToArray)); // [1, 2, 3]
console.log(Object.entries(objToArray)); // [["a", 1], ["b", 2], ["c", 3]]

// Преобразование с проверками
function safeNumberConversion(value) {
    const num = Number(value);
    if (isNaN(num)) {
        throw new Error(`Невозможно преобразовать в число: ${value}`);
    }
    return num;
}

try {
    console.log(safeNumberConversion("123")); // 123
    console.log(safeNumberConversion("abc")); // бросит ошибку
} catch (e) {
    console.error(e.message);
}
```

## Глубокое преобразование и валидация типов

### Продвинутые методы проверки типов

```javascript
// Проверка типов для примитивов
function getType(value) {
    return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType(42)); // "Number"
console.log(getType("str")); // "String"
console.log(getType([])); // "Array"
console.log(getType({})); // "Object"
console.log(getType(null)); // "Null"
console.log(getType(undefined)); // "Undefined"

// Продвинутые проверки типов
function isString(value) {
    return typeof value === 'string' || value instanceof String;
}

function isNumber(value) {
    return typeof value === 'number' && isFinite(value);
}

function isInteger(value) {
    return typeof value === 'number' && isFinite(value) && Math.floor(value) === value;
}

function isFloat(value) {
    return typeof value === 'number' && isFinite(value) && Math.floor(value) !== value;
}

function isBoolean(value) {
    return typeof value === 'boolean';
}

function isNull(value) {
    return value === null;
}

function isUndefined(value) {
    return value === undefined;
}

function isNullOrUndefined(value) {
    return value == null; // проверяет и null, и undefined
}

// Проверка типов для объектов
function isObject(value) {
    return value !== null && typeof value === 'object' && !Array.isArray(value);
}

function isArray(value) {
    return Array.isArray(value);
}

function isFunction(value) {
    return typeof value === 'function';
}

function isPlainObject(value) {
    if (typeof value !== 'object' || value === null) return false;
    return Object.getPrototypeOf(value) === Object.prototype || Object.getPrototypeOf(value) === null;
}

function isEmpty(value) {
    if (value === null || value === undefined) return true;
    if (typeof value === 'string') return value.trim().length === 0;
    if (Array.isArray(value)) return value.length === 0;
    if (typeof value === 'object') return Object.keys(value).length === 0;
    return false;
}

// Примеры использования проверок
console.log(isInteger(42)); // true
console.log(isFloat(42.5)); // true
console.log(isPlainObject({})); // true
console.log(isPlainObject([])); // false
```

### Валидация сложных структур данных

```javascript
// Валидация сложных объектов
function validateUser(user) {
    const errors = [];
    
    if (typeof user !== 'object' || user === null) {
        errors.push('Пользователь должен быть объектом');
        return { valid: false, errors };
    }
    
    if (typeof user.name !== 'string' || user.name.trim().length === 0) {
        errors.push('Имя пользователя обязательно и должно быть строкой');
    }
    
    if (typeof user.age !== 'number' || user.age < 0 || user.age > 150) {
        errors.push('Возраст должен быть числом от 0 до 150');
    }
    
    if (user.email && typeof user.email !== 'string') {
        errors.push('Email должен быть строкой');
    }
    
    if (user.preferences && typeof user.preferences !== 'object') {
        errors.push('Предпочтения должны быть объектом');
    }
    
    return {
        valid: errors.length === 0,
        errors
    };
}

// Использование валидации
const testUser = {
    name: "Иван",
    age: 30,
    email: "ivan@example.com",
    preferences: { theme: "dark" }
};

const validationResult = validateUser(testUser);
console.log('Валидация прошла:', validationResult.valid);
if (!validationResult.valid) {
    console.log('Ошибки:', validationResult.errors);
}

// Валидация с использованием схемы
function createValidator(schema) {
    return function validate(data) {
        const errors = [];
        
        for (const [field, rules] of Object.entries(schema)) {
            const value = data[field];
            
            for (const rule of rules) {
                const result = rule(value, data);
                if (result !== true) {
                    errors.push(`${field}: ${result}`);
                }
            }
        }
        
        return {
            valid: errors.length === 0,
            errors
        };
    };
}

// Определение правил валидации
const required = (value) => value != null && value !== '' || 'обязательное поле';
const minLength = (min) => (value) => !value || value.length >= min || `минимум ${min} символов`;
const isEmail = (value) => !value || /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) || 'некорректный email';
const isNumberInRange = (min, max) => (value) => typeof value === 'number' && value >= min && value <= max || `должно быть между ${min} и ${max}`;

// Создание валидатора
const userValidator = createValidator({
    name: [required, minLength(2)],
    email: [required, isEmail],
    age: [required, (value) => typeof value === 'number' || 'должно быть числом', isNumberInRange(0, 120)]
});

// Использование валидатора
const userData = { name: "И", email: "invalid-email", age: 150 };
const result = userValidator(userData);
console.log('Результат валидации:', result);
```

## Внутренние механизмы преобразования типов

### Алгоритмы преобразования

```javascript
// Алгоритм преобразования объекта в примитив
function toPrimitive(input, preferredType) {
    // preferredType может быть 'string', 'number' или 'default'
    const hint = preferredType || (input instanceof Date ? 'string' : 'number');
    
    if (hint === 'string') {
        // Попробовать toString, затем valueOf
        if (typeof input.toString === 'function') {
            const str = input.toString();
            if (typeof str !== 'object') return str;
        }
        if (typeof input.valueOf === 'function') {
            const val = input.valueOf();
            if (typeof val !== 'object') return val;
        }
    } else {
        // Попробовать valueOf, затем toString
        if (typeof input.valueOf === 'function') {
            const val = input.valueOf();
            if (typeof val !== 'object') return val;
        }
        if (typeof input.toString === 'function') {
            const str = input.toString();
            if (typeof str !== 'object') return str;
        }
    }
    
    throw new TypeError('Cannot convert object to primitive value');
}

// Использование Symbol.toPrimitive
const objWithToPrimitive = {
    value: 42,
    [Symbol.toPrimitive](hint) {
        console.log(`Преобразование с hint: ${hint}`);
        if (hint === 'number') return this.value;
        if (hint === 'string') return `obj(${this.value})`;
        return this.value; // default
    }
};

console.log(Number(objWithToPrimitive)); // 42
console.log(String(objWithToPrimitive)); // "obj(42)"
console.log(objWithToPrimitive + ""); // "42" (default hint)
```

### Сравнение с преобразованием типов

```javascript
// Сравнение == и === с преобразованием типов
console.log(1 == "1"); // true (преобразование типов)
console.log(1 === "1"); // false (без преобразования)

// Подробности алгоритма абстрактного равенства
// 1. Если типы одинаковые -> строгое равенство
// 2. null и undefined равны друг другу
console.log(null == undefined); // true
console.log(null === undefined); // false

// 3. Число и строка -> строка преобразуется в число
console.log(42 == "42"); // true
console.log(0 == false); // true
console.log("" == false); // true

// 4. Объект и примитив -> объект преобразуется в примитив
const objForComparison = {
    valueOf: () => 42
};
console.log(objForComparison == 42); // true

// Алгоритм для === (строгого равенства)
// 1. Если типы разные -> false
// 2. null === null, undefined === undefined
// 3. Числа: NaN !== NaN, +0 === -0
// 4. Остальные: идентичны если имеют одинаковое значение

console.log(NaN === NaN); // false
console.log(+0 === -0); // true
console.log(0 === -0); // true
```

## Современные возможности и оптимизации

### Операторы нулевого слияния и опциональной цепочки

```javascript
// Оператор нулевого слияния (ES2020)
const user = {
    name: "Иван",
    age: null,
    address: {
        city: "Москва"
    }
};

// Получение значения с fallback
const displayName = user.name ?? "Гость"; // "Иван"
const displayAge = user.age ?? 0; // 0 (а не null)
const fallbackAge = user.nonExistentProperty ?? -1; // -1

// Сравнение с логическим ИЛИ
const oldWay = user.name || "Гость"; // работает, но возвращает "Гость" для пустой строки
const newWay = user.name ?? "Гость"; // возвращает "Гость" только для null/undefined

// Оператор цепочки вызовов (optional chaining)
const city = user.address?.city ?? "Город не указан"; // "Москва"
const street = user.address?.street ?? "Улица не указана"; // "Улица не указана"
const nonExistent = user.nonExistent?.property; // undefined (без ошибки)

// Глубокий доступ к вложенным свойствам
const config = {
    app: {
        settings: {
            theme: "dark",
            language: "ru"
        }
    }
};

// Безопасный глубокий доступ
const theme = config?.app?.settings?.theme ?? "light";
const nonExistentDeep = config?.app?.nonExistent?.property; // undefined

// Опциональная цепочка с вызовом методов
const objWithMethod = {
    method: function() {
        return "результат";
    }
};

const result = objWithMethod?.method?.() ?? "метод не существует";
console.log(result); // "результат"

const nullObj = null;
const nullResult = nullObj?.method?.() ?? "объект равен null";
console.log(nullResult); // "объект равен null"
```

### Типизированные массивы и буферы

```javascript
// Работа с бинарными данными
const buffer = new ArrayBuffer(1024); // 1 КБ
const view = new DataView(buffer);

// Установка и получение значений разных типов
view.setInt32(0, 123456); // по смещению 0
view.setFloat64(8, 3.14159); // по смещению 8
view.setUint8(16, 255); // по смещению 16

console.log(view.getInt32(0)); // 123456
console.log(view.getFloat64(8)); // 3.14159
console.log(view.getUint8(16)); // 255

// Типизированные массивы
const int8Array = new Int8Array(buffer, 0, 4); // 4 элемента типа Int8
const uint16Array = new Uint16Array(buffer, 4, 2); // 2 элемента типа Uint16
const float32Array = new Float32Array(buffer, 8, 1); // 1 элемент типа Float32

// Использование в веб-приложениях (например, для работы с изображениями)
function processImageData(imageData) {
    const uint8ClampedArray = imageData.data; // Uint8ClampedArray
    const length = uint8ClampedArray.length;

    // Обработка RGBA значений (4 байта на пиксель)
    for (let i = 0; i < length; i += 4) {
        const r = uint8ClampedArray[i];
        const g = uint8ClampedArray[i + 1];
        const b = uint8ClampedArray[i + 2];
        const a = uint8ClampedArray[i + 3];

        // Применение фильтра (например, сепия)
        const gray = 0.299 * r + 0.587 * g + 0.114 * b;
        uint8ClampedArray[i] = Math.min(255, gray + 80);     // R
        uint8ClampedArray[i + 1] = Math.min(255, gray + 40); // G
        uint8ClampedArray[i + 2] = Math.max(0, gray - 60);   // B
        // A остается без изменений
    }
}

// Обмен данными между типизированными массивами
function arrayConversion() {
    const sourceArray = new Float32Array([1.1, 2.2, 3.3]);
    const targetArray = new Int32Array(sourceArray.length);
    
    // Преобразование при копировании
    for (let i = 0; i < sourceArray.length; i++) {
        targetArray[i] = sourceArray[i]; // 1.1 -> 1, 2.2 -> 2, 3.3 -> 3
    }
    
    console.log([...targetArray]); // [1, 2, 3]
}
```

## Практические примеры из промышленной разработки

### Система типизации для приложения

```javascript
// Система проверки типов для приложения
class TypeChecker {
    static isType(value, type) {
        switch (type) {
            case 'string':
                return typeof value === 'string';
            case 'number':
                return typeof value === 'number' && isFinite(value);
            case 'boolean':
                return typeof value === 'boolean';
            case 'array':
                return Array.isArray(value);
            case 'object':
                return value !== null && typeof value === 'object' && !Array.isArray(value);
            case 'function':
                return typeof value === 'function';
            case 'null':
                return value === null;
            case 'undefined':
                return value === undefined;
            default:
                return false;
        }
    }
    
    static validateStructure(obj, schema) {
        const errors = [];
        
        for (const [key, expectedType] of Object.entries(schema)) {
            if (!(key in obj)) {
                errors.push(`Отсутствует обязательное поле: ${key}`);
                continue;
            }
            
            if (!this.isType(obj[key], expectedType)) {
                errors.push(`Поле ${key} должно быть типа ${expectedType}, получено: ${typeof obj[key]}`);
            }
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
    
    static deepClone(obj) {
        if (obj === null || typeof obj !== 'object') return obj;
        if (obj instanceof Date) return new Date(obj);
        if (obj instanceof Array) return obj.map(item => this.deepClone(item));
        if (obj instanceof RegExp) return new RegExp(obj);
        if (typeof obj === 'object') {
            const clonedObj = Array.isArray(obj) ? [] : {};
            for (let key in obj) {
                if (obj.hasOwnProperty(key)) {
                    clonedObj[key] = this.deepClone(obj[key]);
                }
            }
            return clonedObj;
        }
    }
}

// Использование системы типизации
const userSchema = {
    id: 'number',
    name: 'string',
    email: 'string',
    isActive: 'boolean',
    preferences: 'object'
};

const userData = {
    id: 1,
    name: 'Иван',
    email: 'ivan@example.com',
    isActive: true,
    preferences: { theme: 'dark' }
};

const validation = TypeChecker.validateStructure(userData, userSchema);
console.log('Валидация структуры:', validation);
```

### Преобразование данных в API

```javascript
// Утилиты для преобразования данных API
class DataTransformer {
    // Преобразование строк в числа в объекте
    static stringsToNumbers(obj) {
        const result = {};
        for (const [key, value] of Object.entries(obj)) {
            if (typeof value === 'string' && !isNaN(value) && !isNaN(parseFloat(value))) {
                result[key] = parseFloat(value);
            } else if (typeof value === 'object' && value !== null) {
                result[key] = this.stringsToNumbers(value);
            } else {
                result[key] = value;
            }
        }
        return result;
    }
    
    // Преобразование дат из строк
    static stringsToDates(obj, dateFields = []) {
        const result = {};
        for (const [key, value] of Object.entries(obj)) {
            if (dateFields.includes(key) && typeof value === 'string') {
                result[key] = new Date(value);
            } else if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
                result[key] = this.stringsToDates(value, dateFields);
            } else {
                result[key] = value;
            }
        }
        return result;
    }
    
    // Очистка данных от null/undefined
    static clean(obj) {
        const result = {};
        for (const [key, value] of Object.entries(obj)) {
            if (value !== null && value !== undefined) {
                if (typeof value === 'object' && !Array.isArray(value)) {
                    const cleanedValue = this.clean(value);
                    if (Object.keys(cleanedValue).length > 0) {
                        result[key] = cleanedValue;
                    }
                } else {
                    result[key] = value;
                }
            }
        }
        return result;
    }
    
    // Преобразование для отправки на сервер
    static forServer(obj) {
        return JSON.parse(JSON.stringify(obj, (key, value) => {
            // Преобразование Date в ISO строку
            if (value instanceof Date) {
                return value.toISOString();
            }
            // Преобразование BigInt к строке
            if (typeof value === 'bigint') {
                return value.toString();
            }
            return value;
        }));
    }
}

// Пример использования
const rawData = {
    id: "123",
    name: "Иван",
    age: "30",
    salary: "50000.50",
    birthDate: "1990-01-01",
    active: "true",
    metadata: {
        created: "2023-01-01T00:00:00Z",
        lastLogin: null
    }
};

const transformed = DataTransformer.stringsToNumbers(rawData);
console.log('Преобразованные числа:', transformed);
// { id: 123, name: "Иван", age: 30, salary: 50000.5, birthDate: "1990-01-01", active: "true", ... }

const withDates = DataTransformer.stringsToDates(transformed, ['birthDate', 'created', 'lastLogin']);
console.log('С преобразованными датами:', withDates);

const cleaned = DataTransformer.clean(withDates);
console.log('Очищенные данные:', cleaned);
```

## Лучшие практики и советы по производительности

### Оптимизация преобразований типов

```javascript
// Избегайте частых преобразований типов в циклах
function inefficientTypeConversion(arr) {
    let result = 0;
    for (let i = 0; i < arr.length; i++) {
        // Неявное преобразование типов в каждой итерации
        result += arr[i] + ""; // преобразование в строку
    }
    return result;
}

function efficientTypeConversion(arr) {
    // Явное преобразование перед циклом
    const stringArray = arr.map(item => String(item));
    let result = "";
    for (let i = 0; i < stringArray.length; i++) {
        result += stringArray[i];
    }
    return result;
}

// Использование типизированных массивов для числовых данных
function processNumbersEfficiently(numbers) {
    // Создаем типизированный массив для лучшей производительности
    const typedArray = new Float64Array(numbers);
    let sum = 0;

    for (let i = 0; i < typedArray.length; i++) {
        sum += typedArray[i];
    }

    return sum;
}

// Кеширование преобразованных значений
const typeConversionCache = new Map();

function cachedStringConversion(value) {
    if (typeConversionCache.has(value)) {
        return typeConversionCache.get(value);
    }
    
    const result = String(value);
    typeConversionCache.set(value, result);
    return result;
}

// Проверка типов с кешированием
function cachedTypeCheck(value, expectedType) {
    const cacheKey = `${typeof value}:${expectedType}`;
    
    if (typeConversionCache.has(cacheKey)) {
        return typeConversionCache.get(cacheKey);
    }
    
    const result = TypeChecker.isType(value, expectedType);
    typeConversionCache.set(cacheKey, result);
    return result;
}

// Использование WeakMap для приватных данных без утечек памяти
const privateData = new WeakMap();

class SafeDataProcessor {
    constructor(data) {
        // Приватные данные, которые не будут препятствовать сборке мусора
        privateData.set(this, { processed: false, data });
    }

    process() {
        const internalData = privateData.get(this);
        if (!internalData.processed) {
            // Обработка данных
            internalData.result = this.performProcessing(internalData.data);
            internalData.processed = true;
        }
        return internalData.result;
    }
    
    performProcessing(data) {
        // Реальная обработка данных
        return Array.isArray(data) ? data.map(item => String(item)) : String(data);
    }
}
```

### Безопасные преобразования

```javascript
// Безопасные преобразования с обработкой ошибок
class SafeConverter {
    static toNumber(value, defaultValue = 0) {
        const num = Number(value);
        return isNaN(num) ? defaultValue : num;
    }
    
    static toInteger(value, defaultValue = 0) {
        const num = Number(value);
        return Number.isSafeInteger(num) ? num : defaultValue;
    }
    
    static toBoolean(value, defaultValue = false) {
        if (typeof value === 'boolean') return value;
        if (typeof value === 'string') return value.toLowerCase() === 'true';
        if (typeof value === 'number') return value !== 0;
        return defaultValue;
    }
    
    static toString(value, defaultValue = '') {
        if (value === null || value === undefined) return defaultValue;
        return String(value);
    }
    
    static toArray(value, defaultValue = []) {
        if (Array.isArray(value)) return value;
        if (value === null || value === undefined) return defaultValue;
        if (typeof value === 'string') return [value];
        if (typeof value === 'object') return Object.values(value);
        return [value];
    }
}

// Использование безопасных преобразований
console.log(SafeConverter.toNumber("123")); // 123
console.log(SafeConverter.toNumber("abc", -1)); // -1
console.log(SafeConverter.toBoolean("true")); // true
console.log(SafeConverter.toBoolean("false")); // false
console.log(SafeConverter.toString(null, "default")); // "default"
console.log(SafeConverter.toArray({a: 1, b: 2})); // [1, 2]
```

## Заключение

Понимание глубоких концепций типов данных и преобразований типов в JavaScript позволяет писать более надежный, производительный и предсказуемый код. Знание внутренних механизмов преобразования, особенностей каждого типа данных и современных возможностей языка критически важно для профессиональной разработки. Важно использовать подходящие методы преобразования в зависимости от задачи и всегда учитывать возможные ошибки и крайние случаи.

#programming #javascript #data-types #type-conversion #coercion #v8 #optimization #validation #web-development #advanced-concepts #type-checking