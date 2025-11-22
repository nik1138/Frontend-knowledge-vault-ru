---
aliases: [Цикл событий, Event Loop, Событийный цикл]
tags: [javascript, async-programming, event-loop, frontend]
---

# Event Loop (Цикл событий)

## Описание

Event Loop (цикл событий) - это механизм, который позволяет JavaScript выполнять неблокирующие асинхронные операции, несмотря на то, что он однопоточный. Он координирует выполнение кода, обработку событий и выполнение callback-функций.

## Архитектура JavaScript Runtime

JavaScript Runtime состоит из нескольких компонентов:
- **Call Stack (Стек вызовов)** - место, где выполняется синхронный код
- **Callback Queue (Очередь колбэков)** - очередь асинхронных callback-функций
- **Event Loop (Цикл событий)** - механизм, который перемещает callback-функции из очереди в стек
- **Web APIs** - браузерные API, такие как setTimeout, DOM, fetch и т.д.

## Принцип работы

1. **Выполнение синхронного кода** - код выполняется построчно в стеке вызовов
2. **Отправка асинхронных операций** - Web APIs обрабатывают асинхронные операции (таймеры, сетевые запросы и т.д.)
3. **Добавление callback-функций в очередь** - когда асинхронная операция завершена, callback добавляется в очередь
4. **Перемещение callback-функций в стек** - Event Loop проверяет, пуст ли стек, и если да, перемещает callback из очереди в стек

### Пример работы

```javascript
console.log('1'); // 1. Выполняется сразу

setTimeout(() => {
    console.log('2'); // 4. Выполняется после всех синхронных операций
}, 0);

console.log('3'); // 2. Выполняется сразу

// Вывод в консоль: 1, 3, 2
```

### Различные очереди callback-ов

JavaScript имеет несколько очередей для различных типов задач:

1. **Callback Queue (Macrotask Queue)** - содержит таймеры, события, I/O операции
2. **Microtask Queue** - содержит Promise callbacks, queueMicrotask

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0); // Macrotask

Promise.resolve().then(() => console.log('3')); // Microtask
Promise.resolve().then(() => console.log('4')); // Microtask

console.log('5');

// Вывод: 1, 5, 3, 4, 2
// Сначала выполняются все синхронные операции (1, 5)
// Затем все microtasks (3, 4)
// Затем одна macrotask (2)
```

## Microtasks vs Macrotasks

- **Macrotasks** - setTimeout, setInterval, setImmediate, I/O, UI rendering
- **Microtasks** - Promise callbacks, queueMicrotask, Object.observe

Microtasks выполняются до следующей macrotask и до рендеринга страницы.

```javascript
// Пример приоритета microtasks над macrotasks
setTimeout(() => console.log('setTimeout'), 0); // Macrotask

Promise.resolve()
    .then(() => console.log('Promise 1'))
    .then(() => console.log('Promise 2'));

queueMicrotask(() => console.log('Microtask 1'));
queueMicrotask(() => console.log('Microtask 2'));

// Вывод: Promise 1, Promise 2, Microtask 1, Microtask 2, setTimeout
```

## Практические применения

### Оптимизация производительности

```javascript
// Разбиение тяжелой задачи на части с помощью queueMicrotask
function heavyTask(items) {
    let index = 0;
    
    function processChunk() {
        const chunkSize = 10; // Обрабатываем по 10 элементов
        const endIndex = Math.min(index + chunkSize, items.length);
        
        for (; index < endIndex; index++) {
            // Выполняем обработку элемента
            processItem(items[index]);
        }
        
        if (index < items.length) {
            // Если есть еще элементы, планируем следующую часть
            queueMicrotask(processChunk);
        }
    }
    
    processChunk();
}
```

### Обеспечение порядка выполнения

```javascript
// Использование Promise для обеспечения правильного порядка
function ensureOrder() {
    return new Promise(resolve => {
        console.log('Начало асинхронной операции');
        
        setTimeout(() => {
            console.log('Асинхронная операция завершена');
            resolve('Результат');
        }, 100);
    });
}

ensureOrder().then(result => {
    console.log('Обработка результата:', result);
});
```

## Практические советы

1. **Понимайте приоритеты** - microtasks выполняются перед macrotasks
2. **Используйте Promise для асинхронных операций** - они попадают в microtask queue
3. **Избегайте блокировки стека** - тяжелые синхронные операции мешают обработке событий
4. **Используйте setTimeout с 0 для отложенного выполнения** - позволяет освободить стек
5. **Разделяйте тяжелые задачи** - используйте queueMicrotask или другие методы для разбиения

## Связанные темы

- [[Callbacks]] - как колбэки взаимодействуют с Event Loop
- [[Promises]] - как промисы используют microtask queue
- [[Async-Await]] - как async/await работает поверх промисов
- [[Обработка-ошибок-в-асинхронном-коде]] - особенности обработки ошибок в асинхронном коде