---
aliases: ["Реактивное программирование", "RxJS"]
tags: 
  - "#javascript"
  - "#observables"
  - "#reactive-programming"
  - "#rxjs"
  - "#async"
---

# Основы JavaScript Observable

## Что такое Observable

Observable - это наблюдаемый объект, который представляет собой поток асинхронных событий, к которым можно подписаться и обрабатывать их по мере поступления. Это ключевая концепция реактивного программирования в JavaScript.

В отличие от традиционных событийных моделей, Observable предоставляет богатый набор методов для преобразования, фильтрации и комбинации потоков данных.

## Концепции реактивного программирования

Реактивное программирование - это парадигма программирования, ориентированная на потоки данных и распространение изменений. В этой парадигме приложение реагирует на изменения в данных, а не управляется последовательными командами.

Ключевые принципы:
- **Потоки данных** - данные рассматриваются как потоки, которые можно преобразовывать
- **Наблюдение** - компоненты могут наблюдать за изменениями в потоках данных
- **Декларативность** - вы описываете, что хотите получить, а не как это получить

## Создание Observable

Observable можно создать несколькими способами:

```javascript
// 1. Используя конструктор
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.complete();
});

// 2. Используя статические методы RxJS
const numbers$ = of(1, 2, 3, 4, 5);
const interval$ = interval(1000);
const click$ = fromEvent(document, 'click');
```

## Подписка на Observable

Чтобы получать значения из Observable, нужно подписаться на него:

```javascript
const subscription = observable.subscribe({
  next: value => console.log(value),
  error: err => console.error(err),
  complete: () => console.log('Completed')
});

// Отмена подписки
subscription.unsubscribe();
```

## Операторы Observable

Операторы позволяют преобразовывать и комбинировать потоки данных:

### Map
```javascript
// Преобразование значений
from([1, 2, 3]).pipe(
  map(x => x * 2)
).subscribe(console.log); // 2, 4, 6
```

### Filter
```javascript
// Фильтрация значений
from([1, 2, 3, 4, 5]).pipe(
  filter(x => x % 2 === 0)
).subscribe(console.log); // 2, 4
```

### Merge
```javascript
// Объединение нескольких потоков
merge(
  interval(1000).pipe(map(x => `A${x}`)),
  interval(1500).pipe(map(x => `B${x}`))
).subscribe(console.log);
```

### CombineLatest
```javascript
// Комбинация последних значений из нескольких потоков
combineLatest([
  of(1, 2, 3),
  of('a', 'b', 'c')
]).subscribe(console.log);
// [1, 'a'], [1, 'b'], [1, 'c'], [2, 'c'], [3, 'c']
```

## Обработка ошибок

Observable предоставляет несколько способов обработки ошибок:

```javascript
// catchError - перехват и обработка ошибки
source$.pipe(
  catchError(error => {
    console.error('Error caught:', error);
    return of('default value');
  })
).subscribe();

// retry - повторная попытка при ошибке
source$.pipe(
  retry(3) // попробовать 3 раза
).subscribe();
```

## Сравнение с Promise

| Observable | Promise |
|------------|---------|
| Множество значений | Одно значение |
| Отмена подписки | Нет отмены |
| Ленивое выполнение | Выполняется сразу |
| Операторы преобразования | Ограниченные возможности |

## Async Iterators vs Observable

Async iterators подходят для последовательного получения значений:

```javascript
// Async iterator
async function* asyncGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

// Observable
const observable = from([1, 2, 3]);
```

Observable более мощный для сложных преобразований и комбинаций потоков данных.

## Практические примеры

### Пример: Поиск с задержкой
```javascript
const searchInput = document.getElementById('search');
const search$ = fromEvent(searchInput, 'input').pipe(
  map(event => event.target.value),
  filter(query => query.length > 2),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => 
    fetch(`/api/search?q=${query}`).then(r => r.json())
  )
);

search$.subscribe(results => {
  // Отобразить результаты
});
```

### Пример: Обработка кликов
```javascript
const button = document.getElementById('button');
const clicks$ = fromEvent(button, 'click');

clicks$.pipe(
  scan(count => count + 1, 0),
  map(count => `Кликнуто ${count} раз`)
).subscribe(text => {
  document.getElementById('counter').textContent = text;
});
```

## Основы RxJS

RxJS (Reactive Extensions for JavaScript) - это библиотека для реактивного программирования с использованием Observables.

Основные компоненты:
- [[js/observables/Observable]] - поток данных
- [[js/observables/Observer]] - объект, который наблюдает за Observable
- [[js/observables/Subscription]] - объект управления подпиской
- [[js/observables/Operators]] - функции преобразования потоков
- [[js/observables/Subject]] - одновременно Observable и Observer

## Марбл-диаграммы

Марбл-диаграммы - визуальное представление работы операторов Observable:

```
source$: --1--2--3--4--5--|
map(x => x*2): --2--4--6--8--10-|
```

Эти диаграммы помогают понять, как операторы преобразуют потоки данных во времени.

## Обработка обратного давления (Backpressure)

При высокой частоте поступления данных может возникнуть проблема обратного давления:

```javascript
// buffer - буферизация значений
source$.pipe(
  bufferTime(1000) // группировка значений по 1 секунде
).subscribe();

// throttle - пропуск значений
source$.pipe(
  throttleTime(100) // пропускать значения в течение 100мс после получения
).subscribe();

// sample - выборка значений
source$.pipe(
  sample(interval(1000)) // выборка значений каждую секунду
).subscribe();
```

## Связь с другими JS файлами

- [[js/async/async-await]] - сравнение с традиционным асинхронным подходом
- [[js/promises/promises]] - основы работы с промисами
- [[js/events/event-emitters]] - события и обработчики
- [[js/rxjs/rxjs-operators]] - подробное описание операторов RxJS
- [[js/reactive/reactive-patterns]] - паттерны реактивного программирования

## Заключение

Observable - мощная концепция для работы с асинхронными потоками данных в JavaScript. Она позволяет создавать гибкие, декларативные решения для обработки событий, запросов и других асинхронных операций.

Понимание основ Observable открывает возможности для создания более предсказуемых и легко тестируемых приложений.