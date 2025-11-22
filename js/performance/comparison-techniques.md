---
aliases: ["Сравнение производительности разных подходов", "Бенчмарки JavaScript"]
tags: [js, performance, benchmarking, frontend]
---

# Сравнение производительности разных подходов

Практическое сравнение различных подходов к решению одних и тех же задач в JavaScript, с акцентом на производительность и эффективность использования ресурсов.

## 1. Сравнение методов работы с массивами

### Поиск элементов в массиве

```javascript
// Подготовка данных для тестов
const largeArray = Array.from({ length: 100000 }, (_, i) => i);
const target = 99999;
const targetMissing = 200000;

// Тест: Array.prototype.indexOf vs Set.has
console.group('Поиск элемента в массиве');

console.time('indexOf - существующий элемент');
const indexResult = largeArray.indexOf(target);
console.timeEnd('indexOf - существующий элемент');

console.time('Set.has - существующий элемент');
const set = new Set(largeArray);
const setResult = set.has(target);
console.timeEnd('Set.has - существующий элемент');

console.time('indexOf - отсутствующий элемент');
const indexMissing = largeArray.indexOf(targetMissing);
console.timeEnd('indexOf - отсутствующий элемент');

console.time('Set.has - отсутствующий элемент');
const setMissing = set.has(targetMissing);
console.timeEnd('Set.has - отсутствующий элемент');

console.log(`indexOf результат: ${indexResult}, Set.has результат: ${setResult}`);
console.log(`indexOf (отсутствует): ${indexMissing}, Set.has (отсутствует): ${setMissing}`);
console.groupEnd();

// Тест: Поиск с условием
const objectArray = Array.from({ length: 100000 }, (_, i) => ({ id: i, value: `item-${i}`, active: i % 2 === 0 }));

console.time('find - объект с условием');
const findResult = objectArray.find(item => item.id === 50000 && item.active);
console.timeEnd('find - объект с условием');

console.time('filter + [0] - объект с условием');
const filterResult = objectArray.filter(item => item.id === 50000 && item.active)[0];
console.timeEnd('filter + [0] - объект с условием');

console.time('for loop - объект с условием');
let loopResult;
for (let i = 0; i < objectArray.length; i++) {
    if (objectArray[i].id === 50000 && objectArray[i].active) {
        loopResult = objectArray[i];
        break;
    }
}
console.timeEnd('for loop - объект с условием');

console.log('Результаты поиска:', { find: !!findResult, filter: !!filterResult, loop: !!loopResult });
```

### Фильтрация и преобразование массивов

```javascript
// Подготовка данных
const numbers = Array.from({ length: 1000000 }, (_, i) => i);

console.group('Фильтрация и преобразование');

// Тест: Цепочка методов vs одиночный цикл
console.time('map + filter цепочка');
const chainResult = numbers
    .map(n => n * 2)
    .filter(n => n % 4 === 0)
    .slice(0, 1000);
console.timeEnd('map + filter цепочка');

console.time('Одиночный цикл с условиями');
const loopResult = [];
for (let i = 0; i < numbers.length && loopResult.length < 1000; i++) {
    const doubled = numbers[i] * 2;
    if (doubled % 4 === 0) {
        loopResult.push(doubled);
    }
}
console.timeEnd('Одиночный цикл с условиями');

// Тест: Использование for...of vs forEach
const smallArray = Array.from({ length: 10000 }, (_, i) => i);

console.time('forEach');
let forEachSum = 0;
smallArray.forEach(num => {
    forEachSum += num;
});
console.timeEnd('forEach');

console.time('for...of');
let forOfSum = 0;
for (const num of smallArray) {
    forOfSum += num;
}
console.timeEnd('for...of');

console.time('for loop');
let forLoopSum = 0;
for (let i = 0; i < smallArray.length; i++) {
    forLoopSum += smallArray[i];
}
console.timeEnd('for loop');

console.log(`forEach: ${forEachSum}, for...of: ${forOfSum}, for: ${forLoopSum}`);
console.groupEnd();
```

## 2. Сравнение методов работы с объектами

### Доступ к свойствам

```javascript
// Подготовка тестовых объектов
const deepObject = {
    level1: {
        level2: {
            level3: {
                level4: {
                    value: 'deep value',
                    array: Array.from({ length: 1000 }, (_, i) => ({ id: i, data: `item-${i}` }))
                }
            }
        }
    }
};

console.group('Доступ к свойствам объекта');

// Тест: Прямой доступ vs безопасный доступ
console.time('Прямой доступ');
const directAccess = deepObject.level1.level2.level3.level4.value;
console.timeEnd('Прямой доступ');

console.time('Безопасный доступ (вручную)');
const safeAccess = deepObject.level1 && 
                  deepObject.level1.level2 && 
                  deepObject.level1.level2.level3 && 
                  deepObject.level1.level2.level3.level4 && 
                  deepObject.level1.level2.level3.level4.value;
console.timeEnd('Безопасный доступ (вручную)');

// Вспомогательная функция для безопасного доступа
function safeGet(obj, path) {
    return path.split('.').reduce((current, key) => current && current[key], obj);
}

console.time('Безопасный доступ (функция)');
const functionAccess = safeGet(deepObject, 'level1.level2.level3.level4.value');
console.timeEnd('Безопасный доступ (функция)');

console.log(`Прямой доступ: ${directAccess}`);
console.log(`Безопасный доступ: ${safeAccess}`);
console.log(`Функция доступа: ${functionAccess}`);
console.groupEnd();
```

### Создание и объединение объектов

```javascript
// Подготовка тестовых данных
const obj1 = { a: 1, b: 2, c: 3 };
const obj2 = { d: 4, e: 5, f: 6 };
const obj3 = { g: 7, h: 8, i: 9 };

console.group('Создание и объединение объектов');

// Тест: Object.assign vs spread оператор vs ручное объединение
console.time('Object.assign');
const assignResult = Object.assign({}, obj1, obj2, obj3);
console.timeEnd('Object.assign');

console.time('Spread оператор');
const spreadResult = { ...obj1, ...obj2, ...obj3 };
console.timeEnd('Spread оператор');

console.time('Ручное объединение');
const manualResult = {};
Object.keys(obj1).forEach(key => manualResult[key] = obj1[key]);
Object.keys(obj2).forEach(key => manualResult[key] = obj2[key]);
Object.keys(obj3).forEach(key => manualResult[key] = obj3[key]);
console.timeEnd('Ручное объединение');

// Тест: Создание объекта с вычисляемыми ключами
const keys = ['a', 'b', 'c', 'd', 'e'];
const values = [1, 2, 3, 4, 5];

console.time('Object.fromEntries');
const fromEntriesResult = Object.fromEntries(keys.map((key, i) => [key, values[i]]));
console.timeEnd('Object.fromEntries');

console.time('Цикл с присваиванием');
const loopResult = {};
for (let i = 0; i < keys.length; i++) {
    loopResult[keys[i]] = values[i];
}
console.timeEnd('Цикл с присваиванием');

console.log('Результаты объединения совпадают:', 
    JSON.stringify(assignResult) === JSON.stringify(spreadResult) &&
    JSON.stringify(spreadResult) === JSON.stringify(manualResult));
console.groupEnd();
```

## 3. Сравнение строковых операций

### Конкатенация строк

```javascript
console.group('Конкатенация строк');

// Подготовка данных
const parts = Array.from({ length: 1000 }, (_, i) => `part-${i}`);

// Тест: Различные методы конкатенации
console.time('Template literals (один)');
const templateResult = parts.map(part => `${part}`).join(' ');
console.timeEnd('Template literals (один)');

console.time('Array.join');
const joinResult = parts.join(' ');
console.timeEnd('Array.join');

console.time('String.concat');
let concatResult = '';
for (const part of parts) {
    concatResult = concatResult.concat(' ', part);
}
concatResult = concatResult.substring(1); // убираем первый пробел
console.timeEnd('String.concat');

console.time('Простая конкатенация +=');
let plusEqualsResult = '';
for (let i = 0; i < parts.length; i++) {
    plusEqualsResult += (i > 0 ? ' ' : '') + parts[i];
}
console.timeEnd('Простая конкатенация +=');

// Тест: Добавление к строке в цикле
console.time('+= в большом цикле');
let largeConcat = '';
for (let i = 0; i < 10000; i++) {
    largeConcat += `item-${i} `;
}
console.timeEnd('+= в большом цикле');

console.time('Array.join в большом цикле');
const largeParts = [];
for (let i = 0; i < 10000; i++) {
    largeParts.push(`item-${i}`);
}
const largeJoinResult = largeParts.join(' ');
console.timeEnd('Array.join в большом цикле');

console.groupEnd();
```

### Поиск и замена в строках

```javascript
// Подготовка строки для тестов
const largeString = Array.from({ length: 10000 }, (_, i) => `word-${i}`).join(' ');

console.group('Поиск и замена в строках');

// Тест: Поиск подстроки
console.time('String.includes');
const includesResult = largeString.includes('word-5000');
console.timeEnd('String.includes');

console.time('String.indexOf');
const indexOfResult = largeString.indexOf('word-5000') !== -1;
console.timeEnd('String.indexOf');

console.time('RegExp.test');
const regexResult = /word-5000/.test(largeString);
console.timeEnd('RegExp.test');

// Тест: Замена подстроки
const testString = 'The quick brown fox jumps over the lazy dog. The fox is quick.';
const longTestString = testString.repeat(1000);

console.time('String.replace (одно вхождение)');
const replaceResult = testString.replace('fox', 'cat');
console.timeEnd('String.replace (одно вхождение)');

console.time('String.replaceAll (несколько вхождений)');
const replaceAllResult = testString.replaceAll('the', 'a');
console.timeEnd('String.replaceAll (несколько вхождений)');

console.time('RegExp replace (глобальный)');
const regexReplaceResult = testString.replace(/the/gi, 'a');
console.timeEnd('RegExp replace (глобальный)');

console.time('Split + join');
const splitJoinResult = testString.split('the').join('a');
console.timeEnd('Split + join');

console.log(`includes: ${includesResult}, indexOf: ${indexOfResult}, regex: ${regexResult}`);
console.groupEnd();
```

## 4. Сравнение DOM-операций

### Создание и добавление элементов

```javascript
// Подготовка контейнера для тестов
const container = document.createElement('div');
document.body.appendChild(container);

console.group('DOM операции');

// Тест: Создание элементов разными способами
console.time('createElement');
const elements1 = [];
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    elements1.push(div);
}
console.timeEnd('createElement');

console.time('innerHTML');
let innerHTMLStr = '';
for (let i = 0; i < 1000; i++) {
    innerHTMLStr += `<div>Item ${i}</div>`;
}
console.timeEnd('innerHTML');

console.time('createElement с добавлением');
const fragment1 = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    fragment1.appendChild(div);
}
console.timeEnd('createElement с добавлением');

console.time('innerHTML с добавлением');
const container2 = document.createElement('div');
container2.innerHTML = innerHTMLStr;
console.timeEnd('innerHTML с добавлением');

// Тест: Поиск элементов
const searchContainer = document.createElement('div');
document.body.appendChild(searchContainer);

// Создаем тестовые элементы
for (let i = 0; i < 10000; i++) {
    const div = document.createElement('div');
    div.className = `item item-${i % 100}`;
    div.id = `item-${i}`;
    div.textContent = `Item ${i}`;
    searchContainer.appendChild(div);
}

console.time('getElementById');
const byId = document.getElementById('item-5000');
console.timeEnd('getElementById');

console.time('getElementsByClassName');
const byClass = document.getElementsByClassName('item-1');
console.timeEnd('getElementsByClassName');

console.time('querySelector');
const byQuery = document.querySelector('.item-1');
console.timeEnd('querySelector');

console.time('querySelectorAll');
const byQueryAll = document.querySelectorAll('.item-1');
console.timeEnd('querySelectorAll');

console.log(`Найдено элементов: ${byClass.length}, querySelectorAll: ${byQueryAll.length}`);
console.groupEnd();

// Убираем созданные элементы
document.body.removeChild(container);
document.body.removeChild(searchContainer);
```

### Стили и классы

```javascript
// Подготовка элементов для тестов стилей
const testElements = [];
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.className = 'test-item';
    testElements.push(div);
}

console.group('Операции со стилями и классами');

// Тест: Добавление классов
console.time('className (+=)');
testElements.forEach(el => {
    el.className += ' new-class';
});
console.timeEnd('className (+=)');

// Сбрасываем для следующего теста
testElements.forEach(el => el.className = 'test-item');

console.time('classList.add');
testElements.forEach(el => {
    el.classList.add('new-class');
});
console.timeEnd('classList.add');

// Сбрасываем для следующего теста
testElements.forEach(el => el.className = 'test-item');

console.time('setAttribute');
testElements.forEach(el => {
    const classes = el.getAttribute('class') || '';
    el.setAttribute('class', classes + ' new-class');
});
console.timeEnd('setAttribute');

console.log('Сравнение методов добавления классов');
console.groupEnd();
```

## 5. Сравнение асинхронных операций

### Promise vs async/await

```javascript
console.group('Асинхронные операции');

// Подготовка асинхронных функций для тестов
function asyncOperation(value) {
    return new Promise(resolve => {
        setTimeout(() => resolve(value * 2), 1);
    });
}

// Тест: Promise.chain vs async/await
console.time('Promise.chain');
Promise.resolve(1)
    .then(value => asyncOperation(value))
    .then(value => asyncOperation(value))
    .then(value => asyncOperation(value))
    .then(result => {
        // console.log('Promise.chain result:', result);
    });
console.timeEnd('Promise.chain');

async function asyncTest() {
    let value = 1;
    value = await asyncOperation(value);
    value = await asyncOperation(value);
    value = await asyncOperation(value);
    // console.log('async/await result:', value);
}

console.time('async/await');
asyncTest();
console.timeEnd('async/await');

// Тест: Параллельное выполнение
const tasks = Array.from({ length: 100 }, (_, i) => () => asyncOperation(i));

console.time('Promise.all');
Promise.all(tasks.map(task => task()));
console.timeEnd('Promise.all');

console.time('for await');
async function forAwaitTest() {
    for await (const task of tasks) {
        await task();
    }
}
forAwaitTest();
console.timeEnd('for await');

console.groupEnd();
```

### Оптимизация сетевых запросов

```javascript
// Имитация сетевых запросов
function mockFetch(url) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ url, data: `Response from ${url}`, timestamp: Date.now() });
        }, Math.random() * 100);
    });
}

const urls = Array.from({ length: 50 }, (_, i) => `https://api.example.com/data/${i}`);

console.group('Сетевые запросы');

// Тест: Все запросы параллельно
console.time('Promise.all - все параллельно');
Promise.all(urls.map(url => mockFetch(url)))
    .then(results => {
        // console.log(`Получено ${results.length} ответов`);
    });
console.timeEnd('Promise.all - все параллельно');

// Тест: Ограниченное количество параллельных запросов
async function fetchWithLimit(urls, limit = 5) {
    const results = [];
    const executing = [];
    
    for (const [index, url] of urls.entries()) {
        const promise = mockFetch(url).then(result => {
            results[index] = result;
        });
        
        executing.push(promise);
        
        if (executing.length >= limit) {
            await Promise.race(executing);
            executing.splice(executing.findIndex(p => p === promise), 1);
        }
    }
    
    await Promise.all(executing);
    return results;
}

console.time('Ограниченный параллелизм (5)');
fetchWithLimit(urls, 5);
console.timeEnd('Ограниченный параллелизм (5)');

// Тест: Последовательные запросы
console.time('Последовательные запросы');
async function sequentialFetch() {
    const results = [];
    for (const url of urls) {
        const result = await mockFetch(url);
        results.push(result);
    }
    return results;
}
sequentialFetch();
console.timeEnd('Последовательные запросы');

console.groupEnd();
```

## 6. Сравнение алгоритмов сортировки

### Встроенные vs пользовательские сортировки

```javascript
console.group('Алгоритмы сортировки');

// Подготовка данных для сортировки
const unsortedNumbers = Array.from({ length: 100000 }, () => Math.floor(Math.random() * 100000));
const unsortedObjects = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    value: Math.floor(Math.random() * 100000),
    name: `item-${i}`
}));

// Тест: Встроенная сортировка чисел
console.time('Array.sort() - числа');
const sortedNumbers1 = [...unsortedNumbers].sort((a, b) => a - b);
console.timeEnd('Array.sort() - числа');

// Тест: Пользовательская сортировка подсчетом (для ограниченного диапазона)
function countingSort(arr, max) {
    const count = new Array(max + 1).fill(0);
    const output = [];
    
    for (const num of arr) {
        count[num]++;
    }
    
    for (let i = 0; i <= max; i++) {
        while (count[i] > 0) {
            output.push(i);
            count[i]--;
        }
    }
    
    return output;
}

// Только для небольшого диапазона значений
const smallRangeNumbers = Array.from({ length: 10000 }, () => Math.floor(Math.random() * 1000));
console.time('Сортировка подсчетом');
const countingSorted = countingSort(smallRangeNumbers, 1000);
console.timeEnd('Сортировка подсчетом');

// Тест: Сортировка объектов
console.time('Array.sort() - объекты по числовому полю');
const sortedObjects1 = [...unsortedObjects].sort((a, b) => a.value - b.value);
console.timeEnd('Array.sort() - объекты по числовому полю');

console.time('Array.sort() - объекты по строковому полю');
const sortedObjects2 = [...unsortedObjects].sort((a, b) => a.name.localeCompare(b.name));
console.timeEnd('Array.sort() - объекты по строковому полю');

// Тест: Сортировка с кэшированием вычисляемого значения
console.time('Сортировка с кэшированием');
const decorated = unsortedObjects.map(obj => ({
    original: obj,
    sortKey: obj.name.toLowerCase()
}));
decorated.sort((a, b) => a.sortKey.localeCompare(b.sortKey));
const sortedWithCache = decorated.map(item => item.original);
console.timeEnd('Сортировка с кэшированием');

console.groupEnd();
```

## 7. Сравнение методов кэширования

### Map vs Object для кэширования

```javascript
console.group('Кэширование');

// Подготовка данных для кэширования
const keys = Array.from({ length: 100000 }, (_, i) => i.toString());
const values = Array.from({ length: 100000 }, (_, i) => `value-${i}`);

// Тест: Object как кэш
console.time('Object как кэш - запись');
const objectCache = {};
for (let i = 0; i < keys.length; i++) {
    objectCache[keys[i]] = values[i];
}
console.timeEnd('Object как кэш - запись');

console.time('Object как кэш - чтение');
for (let i = 0; i < 10000; i++) {
    const key = keys[Math.floor(Math.random() * keys.length)];
    const value = objectCache[key];
}
console.timeEnd('Object как кэш - чтение');

// Тест: Map как кэш
console.time('Map как кэш - запись');
const mapCache = new Map();
for (let i = 0; i < keys.length; i++) {
    mapCache.set(keys[i], values[i]);
}
console.timeEnd('Map как кэш - запись');

console.time('Map как кэш - чтение');
for (let i = 0; i < 10000; i++) {
    const key = keys[Math.floor(Math.random() * keys.length)];
    const value = mapCache.get(key);
}
console.timeEnd('Map как кэш - чтение');

// Тест: WeakMap для кэширования с автоматической очисткой
console.time('WeakMap - запись');
const weakMapCache = new WeakMap();
const objectsForWeakMap = keys.slice(0, 1000).map(key => ({}));
objectsForWeakMap.forEach((obj, i) => {
    weakMapCache.set(obj, values[i]);
});
console.timeEnd('WeakMap - запись');

console.time('WeakMap - чтение');
objectsForWeakMap.forEach(obj => {
    const value = weakMapCache.get(obj);
});
console.timeEnd('WeakMap - чтение');

console.log(`Object кэш размер: ${Object.keys(objectCache).length}`);
console.log(`Map кэш размер: ${mapCache.size}`);
console.groupEnd();
```

## 8. Практические рекомендации

### Выбор подходящего метода

```javascript
// Утилита для сравнения производительности
class PerformanceComparator {
    static async compare(name, implementations, iterations = 1000) {
        console.group(`Сравнение: ${name}`);
        
        const results = {};
        
        for (const [implName, implementation] of Object.entries(implementations)) {
            const times = [];
            
            for (let i = 0; i < iterations; i++) {
                const start = performance.now();
                await Promise.resolve(implementation());
                const end = performance.now();
                times.push(end - start);
            }
            
            const avg = times.reduce((a, b) => a + b, 0) / times.length;
            const min = Math.min(...times);
            const max = Math.max(...times);
            
            results[implName] = { avg, min, max };
            console.log(`${implName}: avg=${avg.toFixed(4)}ms, min=${min.toFixed(4)}ms, max=${max.toFixed(4)}ms`);
        }
        
        console.groupEnd();
        return results;
    }
}

// Пример использования утилиты
PerformanceComparator.compare('Поиск в массиве', {
    'indexOf': () => {
        const arr = [1, 2, 3, 4, 5];
        return arr.indexOf(3);
    },
    'find': () => {
        const arr = [1, 2, 3, 4, 5];
        return arr.find(x => x === 3);
    },
    'for loop': () => {
        const arr = [1, 2, 3, 4, 5];
        for (let i = 0; i < arr.length; i++) {
            if (arr[i] === 3) return i;
        }
        return -1;
    }
}, 10000);

// Общие рекомендации по выбору методов:

console.group('Рекомендации по выбору методов');

console.log(`
1. Для поиска элементов:
   - Используйте Set.has() для O(1) поиска при частых проверках
   - Используйте Array.find() для поиска с условием
   - Используйте Array.indexOf() для поиска по значению
   - Используйте бинарный поиск для отсортированных массивов

2. Для работы с DOM:
   - Используйте DocumentFragment для множественных вставок
   - Используйте делегирование событий для динамических элементов
   - Используйте classList.add/remove вместо манипуляций строкой

3. Для асинхронных операций:
   - Используйте Promise.all для независимых операций
   - Используйте ограниченный параллелизм для сетевых запросов
   - Используйте мемоизацию для дорогостоящих вычислений

4. Для кэширования:
   - Используйте Map для произвольных ключей
   - Используйте WeakMap для автоматической очистки
   - Используйте Object для простых строковых ключей
`);

console.groupEnd();
```

Эти сравнения помогут вам выбрать наиболее эффективные методы для конкретных задач, основываясь на реальных измерениях производительности.

См. также:
- [[js/performance/additional-techniques]] - Дополнительные техники оптимизации
- [[js/performance/profiling-tips]] - Практические рекомендации по профилированию
- [[js/tricks/arrays-tricks]] - Хитрости работы с массивами
- [[js/snippets/number-formatting-snippets]] - Сниппеты для форматирования чисел