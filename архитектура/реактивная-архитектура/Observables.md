---
aliases: [Observable, Наблюдаемые, Наблюдаемые последовательности]
tags: [javascript, rxjs, reactive, observables, streams, patterns]
---

# Observables (Наблюдаемые последовательности)

**Observable** — это основная концепция реактивного программирования, представляющая собой поток данных, который может изменяться со временем. Observable позволяет подписаться на поток событий и реагировать на них.

## Определение Observable

Observable — это объект, который:
- Может испускать (emit) значения
- Может испускать ошибки
- Может сигнализировать о завершении потока
- Поддерживает механизм подписки (subscription)

## Структура Observable

Observable состоит из трех основных компонентов:

### Испускание значений (Emission)
- `next(value)` — испускает следующее значение
- Происходит в определенный момент времени
- Может происходить как синхронно, так и асинхронно

### Обработка ошибок (Error)
- `error(exception)` — сигнализирует об ошибке
- Завершает Observable
- Вызывает соответствующий колбэк в observer

### Завершение (Completion)
- `complete()` — сигнализирует о завершении потока
- Не испускает значений
- Также завершает Observable

## Создание Observables

### Простое создание

```javascript
import { Observable } from 'rxjs';

const observable$ = new Observable(observer => {
  observer.next('Первое значение');
  observer.next('Второе значение');
  observer.complete(); // или observer.error(new Error('Ошибка'));
});
```

### Использование creation operators

```javascript
import { of, from, interval, timer } from 'rxjs';

// of — создает Observable из переданных значений
const numbers$ = of(1, 2, 3, 4, 5);

// from — создает Observable из массива, строки, Promise и т.д.
const array$ = from([1, 2, 3, 4, 5]);

// interval — испускает значения через определенные интервалы
const timer$ = interval(1000); // испускает 0, 1, 2... каждую секунду

// timer — испускает значение через определенное время
const delayed$ = timer(2000); // испускает 0 через 2 секунды
```

## Подписка на Observable

```javascript
const subscription = observable$.subscribe({
  next: value => console.log('Получено значение:', value),
  error: error => console.error('Произошла ошибка:', error),
  complete: () => console.log('Поток завершен')
});

// Отписка
subscription.unsubscribe();
```

## Hot vs Cold Observables

### Cold Observables
- Создают новый источник данных для каждого подписчика
- Начинают испускать данные только при подписке
- Пример: `of`, `from`, `interval`

### Hot Observables
- Испускают данные независимо от наличия подписчиков
- Все подписчики получают одни и те же данные
- Пример: `Subject`, `BehaviorSubject`

```javascript
import { Subject, BehaviorSubject } from 'rxjs';

// Hot Observable - Subject
const subject$ = new Subject();
subject$.subscribe(value => console.log('Подписчик 1:', value));
subject$.next('Значение 1'); // Подписчик 1 получит это
subject$.subscribe(value => console.log('Подписчик 2:', value)); // Подписчик 2 не получит предыдущие значения
subject$.next('Значение 2'); // Оба подписчика получат это

// Hot Observable - BehaviorSubject
const behaviorSubject$ = new BehaviorSubject('Начальное значение');
behaviorSubject$.subscribe(value => console.log('Подписчик 1:', value)); // Получит 'Начальное значение'
behaviorSubject$.next('Новое значение');
behaviorSubject$.subscribe(value => console.log('Подписчик 2:', value)); // Также получит 'Новое значение'
```

## Практические примеры в российских реалиях 2025 года

### HTTP-запросы

```javascript
import { Observable, throwError } from 'rxjs';
import { map, catchError, retry } from 'rxjs/operators';
import { ajax } from 'rxjs/ajax';

// Создание Observable для HTTP-запроса
const getUsers$ = (): Observable<User[]> => {
  return ajax.getJSON('/api/users').pipe(
    map(response => response.data as User[]),
    retry(3),
    catchError(error => {
      console.error('Ошибка получения пользователей:', error);
      return throwError(() => error);
    })
  );
};
```

### Обработка пользовательского ввода

```javascript
import { fromEvent, merge } from 'rxjs';
import { map, debounceTime, distinctUntilChanged, filter } from 'rxjs/operators';

// Создание Observable из событий ввода
const inputElement = document.getElementById('search-input');
const input$ = fromEvent(inputElement, 'input').pipe(
  map(event => (event.target as HTMLInputElement).value),
  debounceTime(300),
  distinctUntilChanged(),
  filter(value => value.length > 2)
);

input$.subscribe(query => performSearch(query));
```

### Работа с WebSocket

```javascript
import { Observable, webSocket } from 'rxjs';
import { filter, map, takeUntil } from 'rxjs/operators';

// Создание Observable из WebSocket соединения
const ws$ = webSocket('ws://localhost:8080');

// Отправка сообщения
ws$.next({ type: 'JOIN_ROOM', payload: { roomId: '123' } });

// Получение сообщений
const messages$ = ws$.pipe(
  filter(data => data.type === 'MESSAGE'),
  map(data => data.payload)
);

messages$.subscribe(message => displayMessage(message));
```

## Преимущества Observables

1. **Единая модель для асинхронности** — Observable унифицирует работу с Promise, событиями DOM, HTTP-запросами и другими асинхронными операциями
2. **Отмена операций** — возможность отписаться и отменить выполнение асинхронной операции
3. **Композиция** — возможность комбинировать разные Observable с помощью операторов
4. **Управление ошибками** — встроенные механизмы обработки ошибок
5. **Ленивость** — Observable начинает испускать данные только при наличии подписчика

## Проблемы и решения

### Утечки памяти
```javascript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

class Component {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    someObservable$
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => this.handleData(data));
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Дублирование запросов
```javascript
import { shareReplay } from 'rxjs/operators';

// Кеширование результатов
const cachedRequest$ = expensiveApiCall$.pipe(
  shareReplay(1) // Кеширует последнее значение и делится им между подписчиками
);
```

## Сравнение с другими подходами

### Observable vs Promise
| Характеристика | Promise | Observable |
|---|---|---|
| Количество значений | Одно значение | Множество значений |
| Отмена | Нет | Да (через Subscription) |
|Ленивость| Нет (выполняется сразу) | Да (при подписке) |
| Композиция | Ограниченная | Богатая (операторы) |
| Обработка ошибок | .catch() | catchError operator |

### Observable vs Event Emitters
| Характеристика | Event Emitters | Observable |
|---|---|---|
| Композиция | Ограниченная | Богатая |
| Операторы | Нет | Да |
| Управление подписками | Вручную | Автоматически |
| Обработка ошибок | Вручную | Операторы |

## Продвинутые концепции

### Subjects
```javascript
import { Subject, BehaviorSubject, ReplaySubject, AsyncSubject } from 'rxjs';

// Subject - базовый Hot Observable
const subject = new Subject();

// BehaviorSubject - хранит последнее значение
const behaviorSubject = new BehaviorSubject('initial');

// ReplaySubject - хранит несколько последних значений
const replaySubject = new ReplaySubject(3); // хранит 3 последних значения

// AsyncSubject - испускает только последнее значение при завершении
const asyncSubject = new AsyncSubject();
```

### Multicasting
```javascript
import { multicast, refCount, publish } from 'rxjs/operators';

// Совместное использование одного источника между несколькими подписчиками
const shared$ = source$.pipe(
  multicast(() => new Subject()),
  refCount()
);

// Или с помощью publish (альтернатива multicast + refCount)
const published$ = source$.pipe(publish(), refCount());
```

## Практические рекомендации

### При создании Observables
1. Всегда завершайте Observable (complete или error)
2. Используйте соответствующие операторы для обработки ошибок
3. Учитывайте Hot vs Cold при проектировании

### При подписке
1. Обязательно отписывайтесь при уничтожении компонентов
2. Используйте takeUntil для автоматического управления подписками
3. Обрабатывайте ошибки на уровне подписки

## Связь с другими концепциями

- [[Реактивное-программирование]] — общая концепция реактивности
- [[RxJS]] — основная библиотека для работы с Observables
- [[Потоки-данных]] — более общее понятие потоков данных
- [[State-менеджмент]] — управление состоянием с помощью Observables

## Заключение

Observables являются фундаментальной концепцией реактивного программирования в JavaScript. В российской разработке 2025 года они особенно важны для построения сложных интерактивных приложений, где требуется обработка множества асинхронных событий и потоков данных.

> [!tip]
> Используйте Observable для унификации обработки асинхронных операций в приложении — это упрощает код и делает его более предсказуемым.

> [!warning]
> Избегайте создания "горячих" Observables без необходимости — это может привести к неожиданному поведению и усложнить отладку.