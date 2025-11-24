---
aliases: [RxJS Operators, Операторы в RxJS]
tags: [reactive-programming, rxjs, operators, transformation, filtering, combination]
---

# Операторы RxJS

## Общее понятие операторов

Операторы в RxJS - это функции, которые принимают один Observable и возвращают новый Observable. Они позволяют трансформировать, фильтровать, комбинировать и манипулировать потоками данных. Операторы являются ключевым элементом реактивного программирования, позволяющим создавать сложные цепочки обработки данных.

## Категории операторов

### Операторы преобразования
Преобразуют значения в потоке данных.

#### map
Преобразует каждое значение в потоке с помощью предоставленной функции.

```javascript
import { of } from 'rxjs';
import { map } from 'rxjs/operators';

const source$ = of(1, 2, 3, 4, 5);
const result$ = source$.pipe(
  map(value => value * 2)
);

result$.subscribe(value => console.log(value)); // 2, 4, 6, 8, 10
```

#### mapTo
Возвращает одно и то же значение для каждого элемента в потоке.

```javascript
import { interval } from 'rxjs';
import { mapTo } from 'rxjs/operators';

const result$ = interval(1000).pipe(
  mapTo('Привет!')
);

result$.subscribe(value => console.log(value)); // 'Привет!' каждую секунду
```

#### pluck
Извлекает определенное свойство из объекта.

```javascript
import { of } from 'rxjs';
import { pluck } from 'rxjs/operators';

const users$ = of(
  { name: 'Алексей', age: 25 },
  { name: 'Мария', age: 30 },
  { name: 'Иван', age: 35 }
);

const names$ = users$.pipe(
  pluck('name')
);

names$.subscribe(name => console.log(name)); // 'Алексей', 'Мария', 'Иван'
```

### Операторы фильтрации
Фильтруют значения в потоке по определенным критериям.

#### filter
Фильтрует значения по условию.

```javascript
import { of } from 'rxjs';
import { filter } from 'rxjs/operators';

const numbers$ = of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
const evenNumbers$ = numbers$.pipe(
  filter(value => value % 2 === 0)
);

evenNumbers$.subscribe(value => console.log(value)); // 2, 4, 6, 8, 10
```

#### distinct
Фильтрует дубликаты.

```javascript
import { of } from 'rxjs';
import { distinct } from 'rxjs/operators';

const source$ = of(1, 2, 2, 3, 3, 4, 5, 5);
const distinct$ = source$.pipe(
  distinct()
);

distinct$.subscribe(value => console.log(value)); // 1, 2, 3, 4, 5
```

#### take
Возвращает указанное количество значений из начала потока.

```javascript
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

const limited$ = interval(1000).pipe(
  take(5)
);

limited$.subscribe(value => console.log(value)); // 0, 1, 2, 3, 4
```

### Операторы комбинации
Объединяют несколько потоков данных.

#### merge
Объединяет несколько потоков, сохраняя порядок поступления значений.

```javascript
import { interval, timer } from 'rxjs';
import { merge, map } from 'rxjs/operators';

const first$ = interval(1000).pipe(map(value => `Первый: ${value}`));
const second$ = timer(500, 1000).pipe(map(value => `Второй: ${value}`));

const merged$ = first$.pipe(
  merge(second$)
);

merged$.subscribe(value => console.log(value));
```

#### combineLatest
Объединяет последние значения из нескольких потоков.

```javascript
import { combineLatest, interval } from 'rxjs';
import { map } from 'rxjs/operators';

const first$ = interval(1000);
const second$ = interval(1500);

const combined$ = combineLatest([first$, second$]).pipe(
  map(([first, second]) => `Первый: ${first}, Второй: ${second}`)
);

combined$.subscribe(value => console.log(value));
```

#### concat
Объединяет потоки последовательно.

```javascript
import { of, concat } from 'rxjs';
import { delay } from 'rxjs/operators';

const first$ = of(1, 2, 3).pipe(delay(2000));
const second$ = of(4, 5, 6);

const concatenated$ = concat(first$, second$);

concatenated$.subscribe(value => console.log(value)); // 1, 2, 3, 4, 5, 6
```

### Операторы агрегации
Собирают значения из потока в одно значение.

#### reduce
Собирает все значения потока в одно значение.

```javascript
import { of } from 'rxjs';
import { reduce } from 'rxjs/operators';

const sum$ = of(1, 2, 3, 4, 5).pipe(
  reduce((acc, value) => acc + value, 0)
);

sum$.subscribe(result => console.log(result)); // 15
```

#### scan
Аккумулирует значения по мере их поступления.

```javascript
import { of } from 'rxjs';
import { scan } from 'rxjs/operators';

const accumulated$ = of(1, 2, 3, 4, 5).pipe(
  scan((acc, value) => acc + value, 0)
);

accumulated$.subscribe(value => console.log(value)); // 1, 3, 6, 10, 15
```

### Операторы управления временем

#### debounceTime
Ожидает указанное время и выпускает последнее значение.

```javascript
import { fromEvent } from 'rxjs';
import { debounceTime, map } from 'rxjs/operators';

const input = document.getElementById('search-input');
const search$ = fromEvent(input, 'input').pipe(
  debounceTime(300),
  map(event => event.target.value)
);

search$.subscribe(value => console.log('Поиск:', value));
```

#### throttleTime
Ограничивает частоту значений в потоке.

```javascript
import { fromEvent } from 'rxjs';
import { throttleTime, map } from 'rxjs/operators';

const button = document.getElementById('my-button');
const clicks$ = fromEvent(button, 'click').pipe(
  throttleTime(1000),
  map(() => 'Клик')
);

clicks$.subscribe(value => console.log(value)); // Только один клик в секунду
```

## Практические рекомендания по использованию

1. **Используйте pipe()** - начиная с RxJS 5.5, используйте операторы через pipe() для лучшей читаемости и tree-shaking
2. **Понимайте разницу между операторами** - например, merge vs concat vs combineLatest
3. **Учитывайте производительность** - избегайте использования ресурсоемких операторов в частых потоках

> [!tip] Совет
> Используйте marble diagrams для визуализации работы операторов. Это помогает понять, как операторы изменяют потоки данных во времени.

## Связанные концепции

- [[Observable]]
- [[Потоки-данных]]
- [[Обработка-событий]]
- [[Реактивные-паттерны]]

## Заключение

Операторы RxJS предоставляют мощный инструментарий для манипуляции потоками данных. Правильное использование операторов позволяет создавать сложные, но понятные цепочки обработки данных, которые легко тестировать и поддерживать.
