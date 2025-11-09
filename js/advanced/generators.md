# Генераторы и итераторы в JavaScript

## Что такое генераторы

Генераторы - это специальный тип функций в JavaScript, которые могут приостанавливать свое выполнение и возобновлять его позже. Они позволяют создавать итерируемые объекты и реализовывать сложные асинхронные паттерны.

## Основы генераторов

### Создание генераторов

```javascript
// Функция-генератор
function* simpleGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

// Создание итератора
const iterator = simpleGenerator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

### Передача значений в генератор

```javascript
function* generatorWithInput() {
  const input1 = yield "Первое значение";
  console.log("Получено:", input1);
  
  const input2 = yield "Второе значение";
  console.log("Получено:", input2);
  
  return "Конец";
}

const gen = generatorWithInput();

console.log(gen.next()); // { value: "Первое значение", done: false }
console.log(gen.next("Данные 1")); // { value: "Второе значение", done: false }
console.log(gen.next("Данные 2")); // { value: "Конец", done: true }
```

### Обработка ошибок в генераторах

```javascript
function* errorHandlingGenerator() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (error) {
    console.log("Ошибка в генераторе:", error.message);
    yield "Ошибка обработана";
  }
}

const gen = errorHandlingGenerator();

console.log(gen.next()); // { value: 1, done: false }
console.log(gen.throw(new Error("Тестовая ошибка"))); // { value: "Ошибка обработана", done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

## Итераторы

### Создание итераторов

```javascript
// Пользовательский итератор
class NumberIterator {
  constructor(max) {
    this.max = max;
  }
  
  [Symbol.iterator]() {
    let current = 0;
    const max = this.max;
    
    return {
      next() {
        if (current <= max) {
          return { value: current++, done: false };
        } else {
          return { done: true };
        }
      }
    };
  }
}

const numbers = new NumberIterator(3);
for (const num of numbers) {
  console.log(num); // 0, 1, 2, 3
}
```

### Итераторы в генераторах

```javascript
// Генератор как итератор
function* range(start, end, step = 1) {
  for (let i = start; i <= end; i += step) {
    yield i;
  }
}

for (const num of range(1, 10, 2)) {
  console.log(num); // 1, 3, 5, 7, 9
}

// Преобразование массива в генератор
function* arrayGenerator(arr) {
  for (const item of arr) {
    yield item;
  }
}

const items = [1, 2, 3, 4, 5];
for (const item of arrayGenerator(items)) {
  console.log(item); // 1, 2, 3, 4, 5
}
```

## Асинхронные генераторы

### Асинхронные генераторы

```javascript
// Асинхронный генератор
async function* asyncNumberGenerator(start, end, delay = 1000) {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, delay));
    yield i;
  }
}

// Использование асинхронного генератора
async function useAsyncGenerator() {
  console.log("Начало генерации...");
  
  for await (const number of asyncNumberGenerator(1, 5, 500)) {
    console.log("Сгенерировано:", number);
  }
  
  console.log("Генерация завершена");
}

useAsyncGenerator();
```

### Асинхронные итераторы

```javascript
// Асинхронный итератор для обработки потоковых данных
class AsyncDataIterator {
  constructor(urls) {
    this.urls = urls;
    this.index = 0;
  }
  
  async next() {
    if (this.index >= this.urls.length) {
      return { done: true };
    }
    
    try {
      const response = await fetch(this.urls[this.index++]);
      const data = await response.json();
      return { value: data, done: false };
    } catch (error) {
      return { value: { error: error.message }, done: false };
    }
  }
  
  [Symbol.asyncIterator]() {
    return this;
  }
}

// Использование
async function processData() {
  const urls = [
    'https://jsonplaceholder.typicode.com/posts/1',
    'https://jsonplaceholder.typicode.com/posts/2',
    'https://jsonplaceholder.typicode.com/posts/3'
  ];
  
  const iterator = new AsyncDataIterator(urls);
  
  for await (const data of iterator) {
    console.log('Получены данные:', data);
  }
}

processData();
```

## Практические примеры

### Бесконечные последовательности

```javascript
// Генератор чисел Фибоначчи
function* fibonacci() {
  let a = 0, b = 1;
  
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// Получение первых 10 чисел Фибоначчи
const fib = fibonacci();
for (let i = 0; i < 10; i++) {
  console.log(fib.next().value);
}

// Генератор простых чисел
function* primes() {
  function isPrime(num) {
    if (num < 2) return false;
    for (let i = 2; i <= Math.sqrt(num); i++) {
      if (num % i === 0) return false;
    }
    return true;
  }
  
  let num = 2;
  while (true) {
    if (isPrime(num)) {
      yield num;
    }
    num++;
  }
}

// Получение первых 10 простых чисел
const primeGen = primes();
for (let i = 0; i < 10; i++) {
  console.log(primeGen.next().value);
}
```

### Обработка потоковых данных

```javascript
// Генератор для чтения файла построчно
function* lineReader(text) {
  const lines = text.split('\n');
  for (const line of lines) {
    yield line;
  }
}

// Обработка большого текста
const largeText = `Строка 1
Строка 2
Строка 3
Строка 4
Строка 5`;

for (const line of lineReader(largeText)) {
  console.log('Обработана строка:', line);
}

// Генератор для обработки данных с пагинацией
async function* paginatedDataFetcher(apiUrl, maxPages = 10) {
  let page = 1;
  let hasNext = true;
  
  while (hasNext && page <= maxPages) {
    try {
      const response = await fetch(`${apiUrl}?page=${page}`);
      const data = await response.json();
      
      yield { page, data };
      
      // Проверка наличия следующей страницы
      hasNext = data.hasNextPage || false;
      page++;
    } catch (error) {
      yield { page, error: error.message };
      break;
    }
  }
}

// Использование
async function processAllPages() {
  const apiUrl = 'https://api.example.com/data';
  
  for await (const { page, data, error } of paginatedDataFetcher(apiUrl)) {
    if (error) {
      console.error(`Ошибка на странице ${page}:`, error);
      continue;
    }
    
    console.log(`Обработана страница ${page}:`, data);
  }
}
```

### Утилиты для работы с генераторами

```javascript
// Утилита для преобразования генератора в массив
function generatorToArray(generator) {
  const result = [];
  for (const value of generator) {
    result.push(value);
  }
  return result;
}

// Утилита для фильтрации значений генератора
function* filterGenerator(generator, predicate) {
  for (const value of generator) {
    if (predicate(value)) {
      yield value;
    }
  }
}

// Утилита для преобразования значений генератора
function* mapGenerator(generator, mapper) {
  for (const value of generator) {
    yield mapper(value);
  }
}

// Утилита для ограничения количества значений
function* takeGenerator(generator, count) {
  let taken = 0;
  for (const value of generator) {
    if (taken >= count) break;
    yield value;
    taken++;
  }
}

// Пример использования утилит
function* numbers() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

// Получение первых 10 четных чисел
const evenNumbers = takeGenerator(
  filterGenerator(numbers(), n => n % 2 === 0),
  10
);

console.log(generatorToArray(evenNumbers)); // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### Генераторы для управления состоянием

```javascript
// Генератор для конечного автомата
function* stateMachine() {
  let state = 'idle';
  
  while (true) {
    const action = yield state;
    
    switch (state) {
      case 'idle':
        if (action === 'start') state = 'running';
        break;
      case 'running':
        if (action === 'pause') state = 'paused';
        else if (action === 'stop') state = 'idle';
        break;
      case 'paused':
        if (action === 'resume') state = 'running';
        else if (action === 'stop') state = 'idle';
        break;
    }
  }
}

// Использование конечного автомата
const machine = stateMachine();
console.log(machine.next().value); // 'idle'
console.log(machine.next('start').value); // 'running'
console.log(machine.next('pause').value); // 'paused'
console.log(machine.next('resume').value); // 'running'
```

## Асинхронные паттерны с генераторами

### Асинхронный контроль потока

```javascript
// Утилита для выполнения асинхронных операций с генератором
function asyncRunner(generatorFunction) {
  return function(...args) {
    const generator = generatorFunction(...args);
    
    function run(result) {
      if (result.done) {
        return Promise.resolve(result.value);
      }
      
      return Promise.resolve(result.value)
        .then(value => run(generator.next(value)))
        .catch(error => run(generator.throw(error)));
    }
    
    return run(generator.next());
  };
}

// Асинхронный генератор с контролем потока
function* asyncFlow() {
  try {
    const user = yield fetch('/api/user').then(res => res.json());
    console.log('Пользователь:', user);
    
    const posts = yield fetch(`/api/users/${user.id}/posts`).then(res => res.json());
    console.log('Посты:', posts);
    
    return { user, posts };
  } catch (error) {
    console.error('Ошибка:', error);
    throw error;
  }
}

// Использование
const runAsyncFlow = asyncRunner(asyncFlow);
runAsyncFlow()
  .then(result => console.log('Результат:', result))
  .catch(error => console.error('Ошибка:', error));
```

### Retry паттерн с генераторами

```javascript
// Генератор для повторных попыток
function* retryGenerator(operation, maxRetries = 3, delay = 1000) {
  let attempt = 0;
  
  while (attempt < maxRetries) {
    try {
      const result = yield operation();
      return result;
    } catch (error) {
      attempt++;
      if (attempt >= maxRetries) {
        throw error;
      }
      
      console.log(`Попытка ${attempt} не удалась, повтор через ${delay}мс`);
      yield new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // Экспоненциальное увеличение задержки
    }
  }
}

// Утилита для запуска retry генератора
async function runRetry(generator) {
  let result = generator.next();
  
  while (!result.done) {
    try {
      const value = await result.value;
      result = generator.next(value);
    } catch (error) {
      result = generator.throw(error);
    }
  }
  
  return result.value;
}

// Пример использования
async function unreliableOperation() {
  if (Math.random() < 0.7) {
    throw new Error("Случайная ошибка");
  }
  return "Успех!";
}

const retryGen = retryGenerator(unreliableOperation, 5, 500);
runRetry(retryGen)
  .then(result => console.log("Результат:", result))
  .catch(error => console.error("Все попытки не удались:", error.message));
```

## Лучшие практики

### 1. Используйте генераторы для больших наборов данных

```javascript
// Плохо: создание большого массива в памяти
function getLargeArray() {
  const result = [];
  for (let i = 0; i < 1000000; i++) {
    result.push(i);
  }
  return result;
}

// Хорошо: генератор для экономии памяти
function* largeSequence() {
  for (let i = 0; i < 1000000; i++) {
    yield i;
  }
}

// Использование только нужных значений
const sequence = largeSequence();
for (let i = 0; i < 10; i++) {
  console.log(sequence.next().value);
}
```

### 2. Обрабатывайте ошибки правильно

```javascript
function* safeGenerator() {
  try {
    yield 1;
    yield 2;
    // Потенциально опасная операция
    yield someRiskyOperation();
    yield 3;
  } catch (error) {
    console.error('Ошибка в генераторе:', error.message);
    yield 'error-state';
  } finally {
    console.log('Генератор завершен');
  }
}
```

### 3. Используйте асинхронные генераторы для потоковых данных

```javascript
// Хорошо: асинхронный генератор для обработки потоковых данных
async function* processStream(stream) {
  for await (const chunk of stream) {
    // Обработка чанка данных
    const processed = await processChunk(chunk);
    yield processed;
  }
}
```

## Современные возможности

### for-await-of цикл

```javascript
// Современный способ работы с асинхронными итераторами
async function processAsyncData() {
  const asyncIterable = {
    [Symbol.asyncIterator]() {
      return {
        current: 0,
        async next() {
          if (this.current < 3) {
            await new Promise(resolve => setTimeout(resolve, 1000));
            return { value: this.current++, done: false };
          } else {
            return { done: true };
          }
        }
      };
    }
  };
  
  // Использование for-await-of
  for await (const value of asyncIterable) {
    console.log('Получено значение:', value);
  }
}

processAsyncData();
```

### Генераторы и Promise

```javascript
// Генератор, возвращающий Promise
function* asyncGenerator() {
  yield fetch('/api/data1').then(res => res.json());
  yield fetch('/api/data2').then(res => res.json());
  yield fetch('/api/data3').then(res => res.json());
}

// Обработка Promise из генератора
async function processAsyncGenerator() {
  for (const promise of asyncGenerator()) {
    try {
      const data = await promise;
      console.log('Получены данные:', data);
    } catch (error) {
      console.error('Ошибка:', error);
    }
  }
}
```

## Заключение

Генераторы и итераторы являются мощными инструментами в JavaScript, которые позволяют создавать эффективные и гибкие решения для обработки данных. Они особенно полезны для работы с большими наборами данных, реализации асинхронных паттернов и создания итерируемых объектов. Понимание этих концепций открывает возможности для написания более эффективного и читаемого кода.

Асинхронные генераторы и итераторы расширяют возможности для работы с потоковыми данными и асинхронными операциями, делая код более декларативным и легким для понимания. Правильное использование этих механизмов помогает создавать масштабируемые и поддерживаемые приложения.

#javascript #programming #generators #iterators #async #web-development