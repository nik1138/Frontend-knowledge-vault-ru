---
aliases: [Reactive Extensions, RxJS библиотека]
tags: [javascript, library, rxjs, reactive, observables, operators]
---

# RxJS (Reactive Extensions for JavaScript)

**RxJS** — это библиотека для реактивного программирования с использованием наблюдаемых последовательностей (Observables). Она предоставляет мощные инструменты для работы с асинхронными потоками данных и событиями в JavaScript.

## Основные концепции RxJS

### Observable
Представляет поток данных, который может быть как синхронным, так и асинхронным. [[Observables]]

### Observer
Объект, который определяет, как обрабатывать уведомления от Observable:
- `next` — обработка следующего значения
- `error` — обработка ошибки
- `complete` — обработка завершения потока

### Subscription
Объект, который представляет собой активную подписку на Observable и позволяет отписаться от него.

### Operators
Функции, которые позволяют трансформировать, фильтровать и комбинировать потоки данных:
- **Pipeable operators**: `map`, `filter`, `switchMap`, `mergeMap`, `catchError` и др.
- **Creation operators**: `of`, `from`, `interval`, `timer`, `ajax` и др.

## Установка и использование

```bash
npm install rxjs
```

### Базовое использование

```javascript
import { Observable, of } from 'rxjs';
import { map, filter } from 'rxjs/operators';

// Создание Observable
const numbers$ = of(1, 2, 3, 4, 5);

// Трансформация с помощью операторов
const evenSquares$ = numbers$.pipe(
  filter(n => n % 2 === 0),
  map(n => n * n)
);

// Подписка на результат
evenSquares$.subscribe({
  next: value => console.log(value), // Вывод: 4, 16
  complete: () => console.log('Завершено')
});
```

## Основные операторы RxJS

### Transformation Operators
- `map` — трансформирует каждое значение
- `mapTo` — заменяет все значения на одно значение
- `pluck` — извлекает свойство из объекта
- `scan` — аккумулирует значения (аналог reduce, но с промежуточными результатами)

### Filtering Operators
- `filter` — фильтрует значения по условию
- `take` — берет первые N значений
- `takeUntil` — берет значения до тех пор, пока другой Observable не испустит значение
- `distinct` — исключает дубликаты
- `debounceTime` — задерживает испускание значений на определенное время

### Combination Operators
- `merge` — объединяет несколько Observable
- `concat` — последовательно объединяет Observable
- `combineLatest` — объединяет последние значения из нескольких Observable
- `forkJoin` — ждет завершения всех Observable, как Promise.all

### Flattening Operators
- `switchMap` — отменяет предыдущий Observable при получении нового
- `mergeMap` — параллельно обрабатывает несколько Observable
- `concatMap` — последовательно обрабатывает Observable
- `exhaustMap` — игнорирует новые Observable, пока предыдущий не завершен

## Практические примеры использования в российских реалиях 2025 года

### HTTP-запросы
```javascript
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

// Запрос данных с обработкой ошибок
const apiCall$ = ajax.getJSON('/api/users').pipe(
  map(response => response.data),
  catchError(error => {
    console.error('Ошибка запроса:', error);
    return throwError(() => error);
  })
);

apiCall$.subscribe(data => console.log(data));
```

### Обработка пользовательского ввода
```javascript
import { fromEvent } from 'rxjs';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';

// Поиск с задержкой и отменой предыдущих запросов
const searchInput = document.getElementById('search');
const search$ = fromEvent(searchInput, 'input').pipe(
  map(event => event.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(query => {
    if (query.length > 2) {
      return ajax.getJSON(`/api/search?q=${query}`);
    }
    return of([]);
  })
);

search$.subscribe(results => displayResults(results));
```

### Управление состоянием
```javascript
import { BehaviorSubject, combineLatest } from 'rxjs';
import { map, distinctUntilChanged } from 'rxjs/operators';

// Управление состоянием с помощью BehaviorSubject
const userState$ = new BehaviorSubject({ name: '', role: '' });
const filterState$ = new BehaviorSubject({ active: true });

const combinedState$ = combineLatest([userState$, filterState$]).pipe(
  map(([user, filter]) => ({ ...user, ...filter })),
  distinctUntilChanged((prev, curr) => JSON.stringify(prev) === JSON.stringify(curr))
);

combinedState$.subscribe(state => updateUI(state));
```

## Best Practices в российской разработке

### Управление ресурсами
```javascript
import { Subscription } from 'rxjs';

// Правильное управление подписками
const subscription = new Subscription();

subscription.add(
  observable1$.subscribe(data => console.log(data))
);

subscription.add(
  observable2$.subscribe(data => console.log(data))
);

// Отписка при очистке
return () => subscription.unsubscribe();
```

### Обработка ошибок
```javascript
import { catchError, retry, of } from 'rxjs';

// Повтор запроса при ошибке с fallback
apiCall$.pipe(
  retry(3),
  catchError(error => {
    // Логирование ошибки
    console.error('API ошибка:', error);
    // Возврат значения по умолчанию
    return of([]);
  })
).subscribe(data => handleData(data));
```

### Оптимизация производительности
- Использовать `shareReplay` для кеширования результатов
- Избегать создания новых Observable без необходимости
- Использовать `async` pipe в Angular для автоматического управления подписками

## Интеграция с фреймворками

### Angular
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Observable, Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

@Component({
  selector: 'app-example',
  template: `<div>{{ data$ | async }}</div>`
})
export class ExampleComponent implements OnInit, OnDestroy {
  data$: Observable<any>;
  private destroy$ = new Subject<void>();

  ngOnInit() {
    this.data$ = someObservable$.pipe(
      takeUntil(this.destroy$)
    );
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### React
```javascript
import { useEffect, useState } from 'react';
import { useObservable } from 'observable-hooks';

// Пользовательский хук для работы с Observable
function useObservableState(observable$) {
  const [state, setState] = useState(null);
  
  useEffect(() => {
    const subscription = observable$.subscribe(setState);
    return () => subscription.unsubscribe();
  }, [observable$]);
  
  return state;
}
```

## Тестирование RxJS кода

[[Тестирование]] — отдельный раздел, посвященный тестированию реактивного кода.

## Альтернативы RxJS

- **Most.js** — функциональная библиотека для работы с потоками данных
- **XStream** — библиотека для реактивного программирования с акцентом на производительность
- **Bacon.js** — библиотека для функционального реактивного программирования

## Заключение

RxJS остается одной из самых мощных и гибких библиотек для реактивного программирования в JavaScript экосистеме. В российских реалиях 2025 года она особенно востребована в корпоративных приложениях, где требуется сложная логика обработки данных и высокая отзывчивость интерфейса.

> [!tip]
> Начинайте с простых операторов и постепенно расширяйте знания. RxJS имеет крутую кривую обучения, но мощные возможности для решения сложных задач.

> [!warning]
> Избегайте избыточного использования RxJS в простых сценариях. Иногда обычный async/await или Promise будут проще и понятнее.