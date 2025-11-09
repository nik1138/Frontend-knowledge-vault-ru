# Параллелизм и конкурентность

Функциональное программирование отлично подходит для параллельных и конкурентных вычислений благодаря отсутствию изменяемого состояния. В этой главе мы рассмотрим различные подходы к реализации параллелизма и конкурентности в функциональных приложениях.

## Содержание

1. [Основы параллелизма и конкурентности](#основы-параллелизма-и-конкурентности)
2. [Future и Promise](#future-и-promise)
3. [Параллельные коллекции](#параллельные-коллекции)
4. [Акторы и модели акторов](#акторы-и-модели-акторов)
5. [Software Transactional Memory](#software-transactional-memory)
6. [Пул потоков и воркеры](#пул-потоков-и-воркеры)
7. [Практические примеры](#практические-примеры)

## Основы параллелизма и конкурентности

Параллелизм и конкурентность - это два связанных, но различных понятия в программировании:

- **Параллелизм** - выполнение нескольких задач одновременно на нескольких процессорах или ядрах
- **Конкурентность** - композиция независимо выполняющихся задач, управление доступом к общим ресурсам

В функциональном программировании эти концепции особенно эффективны благодаря неизменяемости данных и отсутствию побочных эффектов.

### Преимущества функционального подхода:

1. **Отсутствие гонок данных** - Неизменяемые данные не могут быть изменены одновременно
2. **Простота композиции** - Чистые функции легко комбинировать
3. **Легкость тестирования** - Нет необходимости в сложных моках состояния
4. **Предсказуемость** - Поведение программы не зависит от порядка выполнения

```typescript
// Простой пример конкурентного выполнения
async function concurrentExample() {
  // Создаем несколько асинхронных задач
  const task1 = async () => {
    console.log("Задача 1 началась");
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log("Задача 1 завершена");
    return "Результат 1";
  };
  
  const task2 = async () => {
    console.log("Задача 2 началась");
    await new Promise(resolve => setTimeout(resolve, 1500));
    console.log("Задача 2 завершена");
    return "Результат 2";
  };
  
  const task3 = async () => {
    console.log("Задача 3 началась");
    await new Promise(resolve => setTimeout(resolve, 800));
    console.log("Задача 3 завершена");
    return "Результат 3";
  };
  
  // Выполняем задачи конкурентно
  console.log("Начало конкурентного выполнения");
  const startTime = Date.now();
  
  const [result1, result2, result3] = await Promise.all([
    task1(),
    task2(),
    task3()
  ]);
  
  const endTime = Date.now();
  console.log(`Все задачи завершены за ${endTime - startTime}ms`);
  console.log("Результаты:", result1, result2, result3);
}

// concurrentExample();
```

### Конкурентные паттерны

```typescript
// Паттерн "Гонка" (Race) - первая завершившаяся задача побеждает
async function raceExample() {
  const fastTask = async () => {
    await new Promise(resolve => setTimeout(resolve, 1000));
    return "Быстрый результат";
  };
  
  const slowTask = async () => {
    await new Promise(resolve => setTimeout(resolve, 3000));
    return "Медленный результат";
  };
  
  try {
    const result = await Promise.race([fastTask(), slowTask()]);
    console.log("Победитель:", result);
  } catch (error) {
    console.error("Ошибка в гонке:", error);
  }
}

// raceExample();

// Паттерн "Все или ничего" (All or Nothing)
async function allOrNothingExample() {
  const tasks = [
    async () => {
      await new Promise(resolve => setTimeout(resolve, 1000));
      return "Задача 1";
    },
    async () => {
      await new Promise(resolve => setTimeout(resolve, 1500));
      return "Задача 2";
    },
    async () => {
      await new Promise(resolve => setTimeout(resolve, 800));
      return "Задача 3";
    }
  ];
  
  try {
    const results = await Promise.all(tasks.map(task => task()));
    console.log("Все задачи завершены:", results);
  } catch (error) {
    console.error("Одна из задач завершилась с ошибкой:", error);
    // Все другие задачи продолжают выполняться, но их результаты игнорируются
  }
}

// allOrNothingExample();

// Паттерн "Попытка с повторами" (Retry)
async function retryExample<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: any;
  
  for (let i = 0; i <= maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      console.log(`Попытка ${i + 1} завершена с ошибкой:`, error);
      
      if (i < maxRetries) {
        console.log(`Ожидание ${delay}ms перед следующей попыткой...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= 2; // Экспоненциальная задержка
      }
    }
  }
  
  throw new Error(`Операция не удалась после ${maxRetries + 1} попыток: ${lastError}`);
}

// Пример использования с нестабильной операцией
async function unstableOperation() {
  if (Math.random() > 0.7) {
    return "Успех!";
  }
  throw new Error("Случайная ошибка");
}

// retryExample(unstableOperation, 5, 500)
//   .then(result => console.log("Результат:", result))
//   .catch(error => console.error("Все попытки провалились:", error));
```

## Future и Promise

Future и Promise - это абстракции для работы с асинхронными вычислениями, которые идеально подходят для функционального программирования.

### Future как функциональная абстракция

```typescript
// Future для асинхронных вычислений
class Future<T> {
  private promise: Promise<T>;
  
  constructor(executor: (resolve: (value: T) => void, reject: (reason: any) => void) => void) {
    this.promise = new Promise(executor);
  }
  
  static of<T>(value: T): Future<T> {
    return new Future(resolve => resolve(value));
  }
  
  static reject<T>(reason: any): Future<T> {
    return new Future((_, reject) => reject(reason));
  }
  
  static fromPromise<T>(promise: Promise<T>): Future<T> {
    return new Future((resolve, reject) => {
      promise.then(resolve, reject);
    });
  }
  
  map<U>(fn: (value: T) => U): Future<U> {
    return new Future((resolve, reject) => {
      this.promise.then(
        value => {
          try {
            resolve(fn(value));
          } catch (error) {
            reject(error);
          }
        },
        reject
      );
    });
  }
  
  flatMap<U>(fn: (value: T) => Future<U>): Future<U> {
    return new Future((resolve, reject) => {
      this.promise.then(
        value => fn(value).promise.then(resolve, reject),
        reject
      );
    });
  }
  
  // Параллельная комбинация двух Future
  zip<U>(other: Future<U>): Future<[T, U]> {
    return new Future((resolve, reject) => {
      let value1: T | undefined;
      let value2: U | undefined;
      let resolved1 = false;
      let resolved2 = false;
      
      this.promise.then(
        value => {
          value1 = value;
          resolved1 = true;
          if (resolved2) {
            resolve([value1!, value2!]);
          }
        },
        reject
      );
      
      other.promise.then(
        value => {
          value2 = value;
          resolved2 = true;
          if (resolved1) {
            resolve([value1!, value2!]);
          }
        },
        reject
      );
    });
  }
  
  // Параллельная обработка массива
  static parallel<T>(futures: Future<T>[]): Future<T[]> {
    return new Future((resolve, reject) => {
      const results: T[] = new Array(futures.length);
      let completed = 0;
      let hasError = false;
      
      futures.forEach((future, index) => {
        future.promise.then(
          value => {
            if (hasError) return;
            results[index] = value;
            completed++;
            if (completed === futures.length) {
              resolve(results);
            }
          },
          error => {
            if (hasError) return;
            hasError = true;
            reject(error);
          }
        );
      });
    });
  }
  
  // Обработка ошибок
  recover(fn: (error: any) => T): Future<T> {
    return new Future((resolve, reject) => {
      this.promise.then(resolve, error => {
        try {
          resolve(fn(error));
        } catch (newError) {
          reject(newError);
        }
      });
    });
  }
  
  // Таймаут
  timeout(ms: number, message: string = "Таймаут"): Future<T> {
    return Future.race([
      this,
      new Future<T>((_, reject) => {
        setTimeout(() => reject(new Error(message)), ms);
      })
    ]);
  }
  
  static race<T>(futures: Future<T>[]): Future<T> {
    return new Future((resolve, reject) => {
      futures.forEach(future => {
        future.promise.then(resolve, reject);
      });
    });
  }
  
  // Преобразование в Promise
  toPromise(): Promise<T> {
    return this.promise;
  }
  
  // Выполнение побочных эффектов
  tap(onSuccess: (value: T) => void, onError?: (error: any) => void): Future<T> {
    return new Future((resolve, reject) => {
      this.promise.then(
        value => {
          try {
            onSuccess(value);
            resolve(value);
          } catch (error) {
            reject(error);
          }
        },
        error => {
          if (onError) {
            try {
              onError(error);
            } catch (tapError) {
              reject(tapError);
              return;
            }
          }
          reject(error);
        }
      );
    });
  }
}

// Пример использования Future
async function futureExample() {
  // Создаем несколько асинхронных операций
  const future1 = new Future<number>((resolve) => {
    setTimeout(() => resolve(10), 1000);
  });
  
  const future2 = new Future<number>((resolve) => {
    setTimeout(() => resolve(20), 1500);
  });
  
  const future3 = new Future<number>((resolve) => {
    setTimeout(() => resolve(30), 800);
  });
  
  // Параллельная комбинация
  const combined = future1.zip(future2);
  const [result1, result2] = await combined.toPromise();
  console.log(`Результаты: ${result1}, ${result2}`);
  
  // Параллельная обработка массива
  const allResults = await Future.parallel([future1, future2, future3]).toPromise();
  console.log(`Все результаты: ${allResults}`);
  
  // Цепочка преобразований
  const transformed = future1
    .map(x => x * 2)
    .map(x => x + 5)
    .recover(error => {
      console.log("Ошибка обработана:", error);
      return 0;
    });
  
  const finalResult = await transformed.toPromise();
  console.log(`Преобразованный результат: ${finalResult}`);
}

// futureExample();
```

### Расширенные возможности Future

```typescript
// Future с поддержкой отмены
class CancellableFuture<T> {
  private promise: Promise<T>;
  private cancelRequested = false;
  private onCancel?: () => void;
  
  constructor(executor: (resolve: (value: T) => void, reject: (reason: any) => void, onCancel: (callback: () => void) => void) => void) {
    this.promise = new Promise((resolve, reject) => {
      const onCancel = (callback: () => void) => {
        this.onCancel = callback;
      };
      
      executor(resolve, reject, onCancel);
    });
  }
  
  static of<T>(value: T): CancellableFuture<T> {
    return new CancellableFuture(resolve => resolve(value));
  }
  
  map<U>(fn: (value: T) => U): CancellableFuture<U> {
    return new CancellableFuture((resolve, reject, onCancel) => {
      const cancel = () => {
        this.cancelRequested = true;
        if (this.onCancel) this.onCancel();
      };
      
      onCancel(cancel);
      
      this.promise.then(
        value => {
          if (!this.cancelRequested) {
            try {
              resolve(fn(value));
            } catch (error) {
              reject(error);
            }
          }
        },
        error => {
          if (!this.cancelRequested) {
            reject(error);
          }
        }
      );
    });
  }
  
  flatMap<U>(fn: (value: T) => CancellableFuture<U>): CancellableFuture<U> {
    return new CancellableFuture((resolve, reject, onCancel) => {
      const cancel = () => {
        this.cancelRequested = true;
        if (this.onCancel) this.onCancel();
      };
      
      onCancel(cancel);
      
      this.promise.then(
        value => {
          if (!this.cancelRequested) {
            fn(value).promise.then(resolve, reject);
          }
        },
        error => {
          if (!this.cancelRequested) {
            reject(error);
          }
        }
      );
    });
  }
  
  cancel(): void {
    this.cancelRequested = true;
    if (this.onCancel) this.onCancel();
  }
  
  toPromise(): Promise<T> {
    return this.promise;
  }
}

// Пример использования отменяемого Future
async function cancellableFutureExample() {
  const cancellableFuture = new CancellableFuture<number>((resolve, reject, onCancel) => {
    const timeoutId = setTimeout(() => {
      resolve(42);
    }, 5000);
    
    onCancel(() => {
      clearTimeout(timeoutId);
      console.log("Операция отменена");
    });
    
    console.log("Операция начата, будет завершена через 5 секунд");
  });
  
  // Отменяем операцию через 2 секунды
  setTimeout(() => {
    cancellableFuture.cancel();
  }, 2000);
  
  try {
    const result = await cancellableFuture.toPromise();
    console.log("Результат:", result);
  } catch (error) {
    console.log("Операция была отменена или завершена с ошибкой:", error);
  }
}

// cancellableFutureExample();
```

## Параллельные коллекции

Параллельные коллекции позволяют выполнять операции над большими наборами данных в несколько потоков.

### Parallel Array

```typescript
// Параллельный массив с поддержкой конкурентных операций
class ParallelArray<T> {
  private readonly data: T[];
  private readonly concurrency: number;
  
  constructor(data: T[], concurrency: number = navigator.hardwareConcurrency || 4) {
    this.data = [...data];
    this.concurrency = concurrency;
  }
  
  // Параллельная операция map
  async parallelMap<U>(fn: (item: T) => U | Promise<U>): Promise<ParallelArray<U>> {
    const chunkSize = Math.ceil(this.data.length / this.concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < this.data.length; i += chunkSize) {
      chunks.push(this.data.slice(i, i + chunkSize));
    }
    
    // Обрабатываем чанки параллельно
    const chunkPromises = chunks.map(chunk => 
      Promise.all(chunk.map(item => Promise.resolve(fn(item))))
    );
    
    const results = await Promise.all(chunkPromises);
    const flattened = results.flat();
    
    return new ParallelArray(flattened, this.concurrency);
  }
  
  // Параллельная операция filter
  async parallelFilter(predicate: (item: T) => boolean | Promise<boolean>): Promise<ParallelArray<T>> {
    const chunkSize = Math.ceil(this.data.length / this.concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < this.data.length; i += chunkSize) {
      chunks.push(this.data.slice(i, i + chunkSize));
    }
    
    // Обрабатываем чанки параллельно
    const chunkPromises = chunks.map(async chunk => {
      const filterPromises = chunk.map(item => Promise.resolve(predicate(item)));
      const filterResults = await Promise.all(filterPromises);
      return chunk.filter((_, index) => filterResults[index]);
    });
    
    const results = await Promise.all(chunkPromises);
    const flattened = results.flat();
    
    return new ParallelArray(flattened, this.concurrency);
  }
  
  // Параллельная операция reduce
  async parallelReduce<U>(
    fn: (accumulator: U, currentValue: T) => U | Promise<U>,
    initialValue: U
  ): Promise<U> {
    const chunkSize = Math.ceil(this.data.length / this.concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < this.data.length; i += chunkSize) {
      chunks.push(this.data.slice(i, i + chunkSize));
    }
    
    // Выполняем reduce для каждого чанка
    const chunkPromises = chunks.map(chunk => {
      return chunk.reduce(async (accPromise, item) => {
        const acc = await accPromise;
        return fn(acc, item);
      }, Promise.resolve(initialValue));
    });
    
    const chunkResults = await Promise.all(chunkPromises);
    
    // Комбинируем результаты чанков
    return chunkResults.reduce((acc, chunkResult) => fn(acc, chunkResult as any), initialValue);
  }
  
  // Параллельная операция forEach
  async parallelForEach(fn: (item: T) => void | Promise<void>): Promise<void> {
    const chunkSize = Math.ceil(this.data.length / this.concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < this.data.length; i += chunkSize) {
      chunks.push(this.data.slice(i, i + chunkSize));
    }
    
    // Обрабатываем чанки параллельно
    const chunkPromises = chunks.map(chunk => 
      Promise.all(chunk.map(item => Promise.resolve(fn(item))))
    );
    
    await Promise.all(chunkPromises);
  }
  
  // Поиск элемента параллельно
  async parallelFind(predicate: (item: T) => boolean | Promise<boolean>): Promise<T | undefined> {
    const chunkSize = Math.ceil(this.data.length / this.concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < this.data.length; i += chunkSize) {
      chunks.push(this.data.slice(i, i + chunkSize));
    }
    
    // Ищем в чанках параллельно
    const chunkPromises = chunks.map(async chunk => {
      for (const item of chunk) {
        if (await Promise.resolve(predicate(item))) {
          return item;
        }
      }
      return undefined;
    });
    
    const results = await Promise.all(chunkPromises);
    return results.find(result => result !== undefined);
  }
  
  toArray(): T[] {
    return [...this.data];
  }
  
  length(): number {
    return this.data.length;
  }
}

// Пример использования Parallel Array
async function parallelArrayExample() {
  // Создаем большой массив данных
  const largeArray = Array.from({ length: 1000000 }, (_, i) => i);
  const parallelArray = new ParallelArray(largeArray, 8);
  
  console.log("Начало параллельных операций...");
  
  // Параллельное преобразование
  console.time("Параллельный map");
  const squared = await parallelArray.parallelMap(x => x * x);
  console.timeEnd("Параллельный map");
  
  // Параллельная фильтрация
  console.time("Параллельный filter");
  const evens = await parallelArray.parallelFilter(x => x % 2 === 0);
  console.timeEnd("Параллельный filter");
  
  // Параллельная агрегация
  console.time("Параллельный reduce");
  const sum = await parallelArray.parallelReduce((acc, val) => acc + val, 0);
  console.timeEnd("Параллельный reduce");
  
  console.log(`Сумма: ${sum}`);
  console.log(`Количество четных чисел: ${evens.length()}`);
  console.log(`Первые 10 квадратов: ${squared.toArray().slice(0, 10)}`);
  
  // Параллельный поиск
  console.time("Параллельный find");
  const found = await parallelArray.parallelFind(x => x === 500000);
  console.timeEnd("Параллельный find");
  
  console.log(`Найден элемент: ${found}`);
}

// parallelArrayExample();
```

### Parallel Stream

```typescript
// Параллельный поток данных
class ParallelStream<T> {
  private readonly source: AsyncIterable<T>;
  private readonly concurrency: number;
  
  constructor(source: AsyncIterable<T>, concurrency: number = navigator.hardwareConcurrency || 4) {
    this.source = source;
    this.concurrency = concurrency;
  }
  
  static fromArray<T>(array: T[], concurrency?: number): ParallelStream<T> {
    return new ParallelStream(
      (async function* () {
        for (const item of array) {
          yield item;
        }
      })(),
      concurrency
    );
  }
  
  // Параллельная трансформация
  async parallelMap<U>(fn: (item: T) => U | Promise<U>): Promise<U[]> {
    const results: U[] = [];
    const workers: Promise<void>[] = [];
    const queue: (() => void)[] = [];
    
    // Создаем рабочие потоки
    for (let i = 0; i < this.concurrency; i++) {
      workers.push(this.createWorker(fn, results, queue));
    }
    
    // Заполняем очередь задачами
    for await (const item of this.source) {
      const task = async () => {
        const result = await Promise.resolve(fn(item));
        results.push(result);
      };
      
      queue.push(task);
    }
    
    // Ждем завершения всех задач
    while (queue.length > 0) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
    
    // Завершаем рабочие потоки
    for (let i = 0; i < this.concurrency; i++) {
      queue.push(() => {}); // Пустая задача для завершения
    }
    
    await Promise.all(workers);
    return results;
  }
  
  private async createWorker<U>(
    fn: (item: T) => U | Promise<U>,
    results: U[],
    queue: (() => Promise<void> | void)[]
  ): Promise<void> {
    while (true) {
      const task = queue.shift();
      if (!task) {
        await new Promise(resolve => setTimeout(resolve, 10));
        continue;
      }
      
      try {
        await task();
      } catch (error) {
        console.error("Ошибка в рабочем потоке:", error);
      }
    }
  }
  
  // Параллельная фильтрация
  async parallelFilter(predicate: (item: T) => boolean | Promise<boolean>): Promise<T[]> {
    const results: T[] = [];
    const workers: Promise<void>[] = [];
    const queue: (() => void)[] = [];
    
    // Создаем рабочие потоки
    for (let i = 0; i < this.concurrency; i++) {
      workers.push(this.createFilterWorker(predicate, results, queue));
    }
    
    // Заполняем очередь задачами
    for await (const item of this.source) {
      const task = async () => {
        const shouldInclude = await Promise.resolve(predicate(item));
        if (shouldInclude) {
          results.push(item);
        }
      };
      
      queue.push(task);
    }
    
    // Ждем завершения всех задач
    while (queue.length > 0) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
    
    // Завершаем рабочие потоки
    for (let i = 0; i < this.concurrency; i++) {
      queue.push(() => {}); // Пустая задача для завершения
    }
    
    await Promise.all(workers);
    return results;
  }
  
  private async createFilterWorker(
    predicate: (item: T) => boolean | Promise<boolean>,
    results: T[],
    queue: (() => Promise<void> | void)[]
  ): Promise<void> {
    while (true) {
      const task = queue.shift();
      if (!task) {
        await new Promise(resolve => setTimeout(resolve, 10));
        continue;
      }
      
      try {
        await task();
      } catch (error) {
        console.error("Ошибка в фильтрующем потоке:", error);
      }
    }
  }
  
  // Параллельная агрегация
  async parallelReduce<U>(
    fn: (accumulator: U, currentValue: T) => U | Promise<U>,
    initialValue: U
  ): Promise<U> {
    let accumulator = initialValue;
    
    // Сначала собираем все элементы
    const items: T[] = [];
    for await (const item of this.source) {
      items.push(item);
    }
    
    // Затем выполняем параллельную агрегацию
    const chunkSize = Math.ceil(items.length / this.concurrency);
    const chunks: T[][] = [];
    
    for (let i = 0; i < items.length; i += chunkSize) {
      chunks.push(items.slice(i, i + chunkSize));
    }
    
    const chunkPromises = chunks.map(chunk => {
      return chunk.reduce(async (accPromise, item) => {
        const acc = await accPromise;
        return fn(acc, item);
      }, Promise.resolve(accumulator));
    });
    
    const chunkResults = await Promise.all(chunkPromises);
    return chunkResults.reduce((acc, chunkResult) => fn(acc, chunkResult as any), accumulator);
  }
}

// Пример использования Parallel Stream
async function parallelStreamExample() {
  // Создаем поток данных
  const dataStream = ParallelStream.fromArray(
    Array.from({ length: 100000 }, (_, i) => i),
    8
  );
  
  console.log("Начало параллельной обработки потока...");
  
  // Параллельное преобразование
  console.time("Параллельный map потока");
  const squared = await dataStream.parallelMap(x => {
    // Симуляция сложных вычислений
    let result = x;
    for (let i = 0; i < 100; i++) {
      result = Math.sqrt(result * result + 1);
    }
    return result;
  });
  console.timeEnd("Параллельный map потока");
  
  console.log(`Обработано ${squared.length} элементов`);
  
  // Параллельная фильтрация
  const filteredStream = ParallelStream.fromArray(
    Array.from({ length: 50000 }, (_, i) => i),
    6
  );
  
  console.time("Параллельный filter потока");
  const filtered = await filteredStream.parallelFilter(x => x % 2 === 0);
  console.timeEnd("Параллельный filter потока");
  
  console.log(`Отфильтровано ${filtered.length} элементов`);
}

// parallelStreamExample();
```

## Акторы и модели акторов

Модель акторов - это концепция конкурентного программирования, где акторы обмениваются сообщениями и не имеют общего состояния.

### Простая реализация акторов

```typescript
// Типы сообщений
type Message = any;

// Интерфейс актора
interface Actor {
  send(message: Message): void;
  stop(): void;
}

// Базовый класс актора
class BaseActor implements Actor {
  private mailbox: Message[] = [];
  private running = true;
  private processing = false;
  
  constructor(private behavior: (message: Message) => void) {}
  
  send(message: Message): void {
    this.mailbox.push(message);
    if (!this.processing) {
      this.processMessages();
    }
  }
  
  private async processMessages(): Promise<void> {
    if (!this.running || this.processing) {
      return;
    }
    
    this.processing = true;
    
    while (this.running && this.mailbox.length > 0) {
      const message = this.mailbox.shift()!;
      try {
        this.behavior(message);
      } catch (error) {
        console.error("Ошибка обработки сообщения:", error);
      }
      
      // Небольшая задержка для предотвращения блокировки
      await new Promise(resolve => setTimeout(resolve, 0));
    }
    
    this.processing = false;
  }
  
  stop(): void {
    this.running = false;
    this.mailbox = [];
  }
}

// Система акторов
class ActorSystem {
  private actors = new Map<string, Actor>();
  
  spawn(name: string, behavior: (message: Message) => void): Actor {
    if (this.actors.has(name)) {
      throw new Error(`Актор с именем ${name} уже существует`);
    }
    
    const actor = new BaseActor(behavior);
    this.actors.set(name, actor);
    return actor;
  }
  
  getActor(name: string): Actor | undefined {
    return this.actors.get(name);
  }
  
  stopActor(name: string): boolean {
    const actor = this.actors.get(name);
    if (actor) {
      actor.stop();
      this.actors.delete(name);
      return true;
    }
    return false;
  }
  
  stopAll(): void {
    for (const actor of this.actors.values()) {
      actor.stop();
    }
    this.actors.clear();
  }
}

// Пример использования акторов
function actorExample() {
  const system = new ActorSystem();
  
  // Создаем актор счетчика
  const counterActor = system.spawn("counter", (message) => {
    if (typeof message === "object" && message.type === "increment") {
      counterActor["count"] = (counterActor["count"] || 0) + 1;
      console.log(`Счетчик: ${counterActor["count"]}`);
    } else if (typeof message === "object" && message.type === "get") {
      console.log(`Текущее значение: ${counterActor["count"] || 0}`);
    }
  });
  
  // Создаем актор логгера
  const loggerActor = system.spawn("logger", (message) => {
    console.log(`[LOG] ${new Date().toISOString()}: ${JSON.stringify(message)}`);
  });
  
  // Отправляем сообщения
  counterActor.send({ type: "increment" });
  counterActor.send({ type: "increment" });
  counterActor.send({ type: "get" });
  
  loggerActor.send("Первое сообщение");
  loggerActor.send("Второе сообщение");
  
  // Останавливаем систему
  setTimeout(() => {
    system.stopAll();
    console.log("Система акторов остановлена");
  }, 1000);
}

// actorExample();
```

### Расширенная модель акторов

```typescript
// Расширенный актор с поддержкой состояния и поведения
class AdvancedActor<T> {
  private state: T;
  private behavior: (state: T, message: Message) => [T, ((message: Message) => void)?];
  private mailbox: Message[] = [];
  private running = true;
  private processing = false;
  
  constructor(
    initialState: T,
    behavior: (state: T, message: Message) => [T, ((message: Message) => void)?]
  ) {
    this.state = initialState;
    this.behavior = behavior;
  }
  
  send(message: Message): void {
    this.mailbox.push(message);
    if (!this.processing) {
      this.processMessages();
    }
  }
  
  private async processMessages(): Promise<void> {
    if (!this.running || this.processing) {
      return;
    }
    
    this.processing = true;
    
    while (this.running && this.mailbox.length > 0) {
      const message = this.mailbox.shift()!;
      try {
        const [newState, sideEffect] = this.behavior(this.state, message);
        this.state = newState;
        
        if (sideEffect) {
          sideEffect(message);
        }
      } catch (error) {
        console.error("Ошибка обработки сообщения:", error);
      }
      
      await new Promise(resolve => setTimeout(resolve, 0));
    }
    
    this.processing = false;
  }
  
  getState(): T {
    return this.state;
  }
  
  stop(): void {
    this.running = false;
    this.mailbox = [];
  }
}

// Пример актора банковского счета
interface BankAccountState {
  balance: number;
  transactions: string[];
}

type BankAccountMessage = 
  | { type: "deposit"; amount: number }
  | { type: "withdraw"; amount: number }
  | { type: "getBalance" }
  | { type: "getTransactions" };

function bankAccountBehavior(
  state: BankAccountState,
  message: BankAccountMessage
): [BankAccountState, ((msg: BankAccountMessage) => void)?] {
  switch (message.type) {
    case "deposit":
      if (message.amount <= 0) {
        console.log("Неверная сумма депозита");
        return [state];
      }
      
      return [{
        balance: state.balance + message.amount,
        transactions: [...state.transactions, `Депозит: +${message.amount}`]
      }];
      
    case "withdraw":
      if (message.amount <= 0) {
        console.log("Неверная сумма снятия");
        return [state];
      }
      
      if (state.balance < message.amount) {
        console.log("Недостаточно средств");
        return [state];
      }
      
      return [{
        balance: state.balance - message.amount,
        transactions: [...state.transactions, `Снятие: -${message.amount}`]
      }];
      
    case "getBalance":
      console.log(`Баланс: ${state.balance}`);
      return [state];
      
    case "getTransactions":
      console.log("Транзакции:", state.transactions);
      return [state];
      
    default:
      console.log("Неизвестное сообщение:", message);
      return [state];
  }
}

// Пример использования банковского актора
function bankAccountExample() {
  const account = new AdvancedActor<BankAccountState>(
    { balance: 0, transactions: [] },
    bankAccountBehavior
  );
  
  // Выполняем операции
  account.send({ type: "deposit", amount: 1000 });
  account.send({ type: "deposit", amount: 500 });
  account.send({ type: "withdraw", amount: 200 });
  account.send({ type: "getBalance" });
  account.send({ type: "getTransactions" });
  
  // Проверяем состояние
  setTimeout(() => {
    console.log("Финальный баланс:", account.getState().balance);
    account.stop();
  }, 100);
}

// bankAccountExample();
```

## Software Transactional Memory

Software Transactional Memory (STM) - это механизм управления конкурентным доступом к разделяемым данным без использования блокировок.

### Простая реализация STM

```typescript
// Транзакционная переменная
class TVar<T> {
  private value: T;
  private version = 0;
  
  constructor(initialValue: T) {
    this.value = initialValue;
  }
  
  get(): T {
    return this.value;
  }
  
  set(newValue: T): void {
    this.value = newValue;
    this.version++;
  }
  
  getVersion(): number {
    return this.version;
  }
}

// Транзакция
class Transaction {
  private reads = new Map<TVar<any>, { value: any; version: number }>();
  private writes = new Map<TVar<any>, any>();
  
  read<T>(tvar: TVar<T>): T {
    if (this.writes.has(tvar)) {
      return this.writes.get(tvar);
    }
    
    const value = tvar.get();
    const version = tvar.getVersion();
    this.reads.set(tvar, { value, version });
    
    return value;
  }
  
  write<T>(tvar: TVar<T>, value: T): void {
    this.writes.set(tvar, value);
  }
  
  validate(): boolean {
    for (const [tvar, { version }] of this.reads) {
      if (!this.writes.has(tvar) && tvar.getVersion() !== version) {
        return false;
      }
    }
    return true;
  }
  
  commit(): void {
    for (const [tvar, value] of this.writes) {
      tvar.set(value);
    }
  }
}

// STM система
class STM {
  static async atomically<T>(operation: (transaction: Transaction) => T): Promise<T> {
    const maxRetries = 10;
    
    for (let attempt = 0; attempt < maxRetries; attempt++) {
      const transaction = new Transaction();
      
      try {
        const result = operation(transaction);
        
        if (transaction.validate()) {
          transaction.commit();
          return result;
        }
      } catch (error) {
        console.log(`Транзакция провалена, попытка ${attempt + 1}:`, error);
      }
      
      // Экспоненциальная задержка перед повторной попыткой
      await new Promise(resolve => setTimeout(resolve, Math.random() * (2 ** attempt) * 10));
    }
    
    throw new Error("Транзакция не удалась после максимального числа попыток");
  }
}

// Пример использования STM
async function stmExample() {
  // Создаем транзакционные переменные
  const balance1 = new TVar(1000);
  const balance2 = new TVar  (500);
  
  console.log("Начальные балансы:", balance1.get(), balance2.get());
  
  // Параллельные транзакции
  const transactions = [
    // Перевод 100 с баланса 1 на баланс 2
    STM.atomically(transaction => {
      const b1 = transaction.read(balance1);
      const b2 = transaction.read(balance2);
      
      if (b1 >= 100) {
        transaction.write(balance1, b1 - 100);
        transaction.write(balance2, b2 + 100);
        console.log("Перевод 100: успешно");
      } else {
        console.log("Перевод 100: недостаточно средств");
      }
    }),
    
    // Перевод 200 с баланса 2 на баланс 1
    STM.atomically(transaction => {
      const b1 = transaction.read(balance1);
      const b2 = transaction.read(balance2);
      
      if (b2 >= 200) {
        transaction.write(balance2, b2 - 200);
        transaction.write(balance1, b1 + 200);
        console.log("Перевод 200: успешно");
      } else {
        console.log("Перевод 200: недостаточно средств");
      }
    }),
    
    // Попытка перевода 2000 (недостаточно средств)
    STM.atomically(transaction => {
      const b1 = transaction.read(balance1);
      const b2 = transaction.read(balance2);
      
      if (b1 >= 2000) {
        transaction.write(balance1, b1 - 2000);
        transaction.write(balance2, b2 + 2000);
        console.log("Перевод 2000: успешно");
      } else {
        console.log("Перевод 2000: недостаточно средств");
      }
    })
  ];
  
  // Выполняем все транзакции параллельно
  await Promise.all(transactions);
  
  console.log("Финальные балансы:", balance1.get(), balance2.get());
}

// stmExample();
```

## Пул потоков и воркеры

Пул потоков позволяет эффективно распределять задачи между ограниченным числом рабочих потоков.

### Worker Pool

```typescript
// Worker Pool для распределения задач
class WorkerPool<T, R> {
  private workers: Worker[] = [];
  private taskQueue: {
    task: T;
    resolve: (result: R) => void;
    reject: (error: any) => void;
    taskId: number;
  }[] = [];
  private idleWorkers: Worker[] = [];
  private nextTaskId = 1;
  
  constructor(
    private workerScript: string,
    private maxWorkers: number = navigator.hardwareConcurrency || 4
  ) {
    this.initializeWorkers();
  }
  
  private initializeWorkers(): void {
    for (let i = 0; i < this.maxWorkers; i++) {
      const worker = new Worker(this.workerScript);
      worker.onmessage = (event) => {
        this.handleWorkerMessage(worker, event);
      };
      worker.onerror = (error) => {
        console.error("Ошибка в воркере:", error);
        this.idleWorkers.push(worker);
        this.processNextTask();
      };
      this.workers.push(worker);
      this.idleWorkers.push(worker);
    }
  }
  
  private handleWorkerMessage(worker: Worker, event: MessageEvent): void {
    const { taskId, result, error } = event.data;
    
    const taskIndex = this.taskQueue.findIndex(t => t.taskId === taskId);
    if (taskIndex >= 0) {
      const task = this.taskQueue[taskIndex];
      this.taskQueue.splice(taskIndex, 1);
      
      if (error) {
        task.reject(new Error(error));
      } else {
        task.resolve(result);
      }
    }
    
    // Возвращаем worker в пул
    this.idleWorkers.push(worker);
    this.processNextTask();
  }
  
  execute(task: T): Promise<R> {
    return new Promise((resolve, reject) => {
      const taskId = this.nextTaskId++;
      this.taskQueue.push({ task, resolve, reject, taskId });
      this.processNextTask();
    });
  }
  
  private processNextTask(): void {
    if (this.taskQueue.length === 0 || this.idleWorkers.length === 0) {
      return;
    }
    
    const task = this.taskQueue.shift()!;
    const worker = this.idleWorkers.pop()!;
    
    worker.postMessage({ taskId: task.taskId, task: task.task });
  }
  
  terminate(): void {
    this.workers.forEach(worker => worker.terminate());
  }
  
  getQueueSize(): number {
    return this.taskQueue.length;
  }
  
  getAvailableWorkers(): number {
    return this.idleWorkers.length;
  }
}

// Пример worker скрипта (worker.js)
// self.onmessage = function(event) {
//   const { taskId, task } = event.data;
//   
//   try {
//     // Симуляция сложных вычислений
//     let result = 0;
//     for (let i = 0; i < task.iterations; i++) {
//       result += Math.sqrt(i * task.multiplier);
//     }
//     
//     self.postMessage({ taskId, result });
//   } catch (error) {
//     self.postMessage({ taskId, error: error.message });
//   }
// };

// Пример использования Worker Pool
async function workerPoolExample() {
  // Создаем worker pool (предполагается, что worker.js существует)
  // const pool = new WorkerPool<{ iterations: number; multiplier: number }, number>(
  //   'worker.js',
  //   4
  // );
  
  // Создаем задачи
  const tasks = [
    { iterations: 1000000, multiplier: 2 },
    { iterations: 1500000, multiplier: 3 },
    { iterations: 800000, multiplier: 1.5 },
    { iterations: 2000000, multiplier: 2.5 },
    { iterations: 1200000, multiplier: 1.8 }
  ];
  
  console.log("Начало выполнения задач в worker pool...");
  console.time("Worker Pool");
  
  // Выполняем задачи параллельно
  // const results = await Promise.all(tasks.map(task => pool.execute(task)));
  
  console.timeEnd("Worker Pool");
  // console.log("Результаты:", results);
  
  // Завершаем worker pool
  // pool.terminate();
}

// workerPoolExample();
```

### Расширенный Worker Pool с мониторингом

```typescript
// Расширенный Worker Pool с мониторингом и статистикой
interface WorkerPoolStats {
  totalTasks: number;
  completedTasks: number;
  failedTasks: number;
  averageExecutionTime: number;
  queueSize: number;
  activeWorkers: number;
  idleWorkers: number;
}

class AdvancedWorkerPool<T, R> {
  private workers: Worker[] = [];
  private taskQueue: {
    task: T;
    resolve: (result: R) => void;
    reject: (error: any) => void;
    taskId: number;
    startTime: number;
  }[] = [];
  private activeTasks = new Map<number, { startTime: number }>();
  private idleWorkers: Worker[] = [];
  private nextTaskId = 1;
  
  // Статистика
  private stats = {
    totalTasks: 0,
    completedTasks: 0,
    failedTasks: 0,
    executionTimes: [] as number[]
  };
  
  constructor(
    private workerScript: string,
    private maxWorkers: number = navigator.hardwareConcurrency || 4
  ) {
    this.initializeWorkers();
  }
  
  private initializeWorkers(): void {
    for (let i = 0; i < this.maxWorkers; i++) {
      const worker = new Worker(this.workerScript);
      worker.onmessage = (event) => {
        this.handleWorkerMessage(worker, event);
      };
      worker.onerror = (error) => {
        this.handleWorkerError(worker, error);
      };
      this.workers.push(worker);
      this.idleWorkers.push(worker);
    }
  }
  
  private handleWorkerMessage(worker: Worker, event: MessageEvent): void {
    const { taskId, result, error } = event.data;
    
    const taskInfo = this.activeTasks.get(taskId);
    if (taskInfo) {
      const executionTime = Date.now() - taskInfo.startTime;
      this.stats.executionTimes.push(executionTime);
      this.activeTasks.delete(taskId);
    }
    
    const taskIndex = this.taskQueue.findIndex(t => t.taskId === taskId);
    if (taskIndex >= 0) {
      const task = this.taskQueue[taskIndex];
      this.taskQueue.splice(taskIndex, 1);
      
      if (error) {
        this.stats.failedTasks++;
        task.reject(new Error(error));
      } else {
        this.stats.completedTasks++;
        task.resolve(result);
      }
    }
    
    // Возвращаем worker в пул
    this.idleWorkers.push(worker);
    this.processNextTask();
  }
  
  private handleWorkerError(worker: Worker, error: ErrorEvent): void {
    console.error("Ошибка в воркере:", error);
    this.stats.failedTasks++;
    
    // Ищем задачу, связанную с этим воркером
    // В упрощенной реализации просто возвращаем воркер в пул
    this.idleWorkers.push(worker);
    this.processNextTask();
  }
  
  execute(task: T): Promise<R> {
    this.stats.totalTasks++;
    const taskId = this.nextTaskId++;
    const startTime = Date.now();
    
    return new Promise((resolve, reject) => {
      this.taskQueue.push({ task, resolve, reject, taskId, startTime });
      this.processNextTask();
    });
  }
  
  private processNextTask(): void {
    if (this.taskQueue.length === 0 || this.idleWorkers.length === 0) {
      return;
    }
    
    const task = this.taskQueue.shift()!;
    const worker = this.idleWorkers.pop()!;
    
    this.activeTasks.set(task.taskId, { startTime: task.startTime });
    worker.postMessage({ taskId: task.taskId, task: task.task });
  }
  
  terminate(): void {
    this.workers.forEach(worker => worker.terminate());
  }
  
  getStats(): WorkerPoolStats {
    const totalExecutionTime = this.stats.executionTimes.reduce((sum, time) => sum + time, 0);
    const averageExecutionTime = this.stats.executionTimes.length > 0 
      ? totalExecutionTime / this.stats.executionTimes.length 
      : 0;
    
    return {
      totalTasks: this.stats.totalTasks,
      completedTasks: this.stats.completedTasks,
      failedTasks: this.stats.failedTasks,
      averageExecutionTime,
      queueSize: this.taskQueue.length,
      activeWorkers: this.workers.length - this.idleWorkers.length,
      idleWorkers: this.idleWorkers.length
    };
  }
  
  resetStats(): void {
    this.stats = {
      totalTasks: 0,
      completedTasks: 0,
      failedTasks: 0,
      executionTimes: []
    };
  }
}

// Пример использования расширенного Worker Pool
async function advancedWorkerPoolExample() {
  console.log("Пример расширенного Worker Pool");
  // В реальной реализации здесь был бы код использования
}
```

## Практические примеры

### Конкурентная обработка данных

```typescript
// Конкурентная обработка больших наборов данных
class ConcurrentDataProcessor {
  // Параллельная обработка с использованием Future
  static async processConcurrent<T, R>(
    data: T[],
    processor: (item: T) => Promise<R>,
    concurrency: number = navigator.hardwareConcurrency || 4
  ): Promise<R[]> {
    const chunkSize = Math.ceil(data.length / concurrency);
    const chunks: T[][] = [];
    
    // Разбиваем данные на чанки
    for (let i = 0; i < data.length; i += chunkSize) {
      chunks.push(data.slice(i, i + chunkSize));
    }
    
    // Обрабатываем чанки параллельно
    const chunkPromises = chunks.map(chunk => {
      return Promise.all(chunk.map(item => processor(item)));
    });
    
    const results = await Promise.all(chunkPromises);
    return results.flat();
  }
  
  // Обработка с ограничением по времени
  static async processWithTimeout<T, R>(
    data: T[],
    processor: (item: T) => Promise<R>,
    timeoutMs: number
  ): Promise<{ results: R[]; timeouts: T[] }> {
    const results: R[] = [];
    const timeouts: T[] = [];
    
    const processItem = async (item: T): Promise<void> => {
      try {
        const result = await Promise.race([
          processor(item),
          new Promise<never>((_, reject) => 
            setTimeout(() => reject(new Error('Таймаут')), timeoutMs)
          )
        ]);
        results.push(result);
      } catch (error) {
        console.log(`Таймаут для элемента:`, item);
        timeouts.push(item);
      }
    };
    
    await Promise.all(data.map(item => processItem(item)));
    return { results, timeouts };
  }
  
  // Обработка с повторными попытками
  static async processWithRetry<T, R>(
    data: T[],
    processor: (item: T) => Promise<R>,
    maxRetries: number = 3
  ): Promise<{ results: R[]; failures: { item: T; error: any }[] }> {
    const results: R[] = [];
    const failures: { item: T; error: any }[] = [];
    
    const processItem = async (item: T): Promise<void> => {
      let lastError: any;
      
      for (let attempt = 0; attempt <= maxRetries; attempt++) {
        try {
          const result = await processor(item);
          results.push(result);
          return;
        } catch (error) {
          lastError = error;
          console.log(`Попытка ${attempt + 1} для элемента провалена:`, error);
          
          if (attempt < maxRetries) {
            await new Promise(resolve => setTimeout(resolve, 1000 * (attempt + 1)));
          }
        }
      }
      
      failures.push({ item, error: lastError });
    };
    
    await Promise.all(data.map(item => processItem(item)));
    return { results, failures };
  }
}

// Пример использования конкурентной обработки
async function concurrentProcessingExample() {
  // Создаем набор данных для обработки
  const data = Array.from({ length: 100 }, (_, i) => ({
    id: i,
    value: Math.random() * 1000
  }));
  
  // Процессор данных
  const processor = async (item: { id: number; value: number }): Promise<number> => {
    // Симуляция сложных вычислений
    await new Promise(resolve => setTimeout(resolve, Math.random() * 100));
    return Math.sqrt(item.value) * item.id;
  };
  
  console.log("Начало конкурентной обработки...");
  console.time("Обработка данных");
  
  // Параллельная обработка
  const results = await ConcurrentDataProcessor.processConcurrent(data, processor, 8);
  
  console.timeEnd("Обработка данных");
  console.log(`Обработано ${results.length} элементов`);
  console.log(`Первые 10 результатов:`, results.slice(0, 10));
  
  // Обработка с таймаутом
  const unstableProcessor = async (item: { id: number; value: number }): Promise<number> => {
    const delay = Math.random() * 2000; // До 2 секунд
    await new Promise(resolve => setTimeout(resolve, delay));
    if (Math.random() > 0.8) {
      throw new Error("Случайная ошибка");
    }
    return item.value * 2;
  };
  
  console.time("Обработка с таймаутом");
  const { results: timeoutResults, timeouts } = 
    await ConcurrentDataProcessor.processWithTimeout(data, unstableProcessor, 1000);
  console.timeEnd("Обработка с таймаутом");
  
  console.log(`Успешно обработано: ${timeoutResults.length}`);
  console.log(`Таймауты: ${timeouts.length}`);
  
  // Обработка с повторными попытками
  console.time("Обработка с повторами");
  const { results: retryResults, failures } = 
    await ConcurrentDataProcessor.processWithRetry(data.slice(0, 20), unstableProcessor, 3);
  console.timeEnd("Обработка с повторами");
  
  console.log(`Успешно обработано с повторами: ${retryResults.length}`);
  console.log(`Провалы: ${failures.length}`);
}

// concurrentProcessingExample();
```

### Реактивные конкурентные системы

```typescript
// Реактивная конкурентная система
class ReactiveSystem<T> {
  private observers: ((data: T) => void)[] = [];
  private dataStream: AsyncIterable<T>;
  
  constructor(dataStream: AsyncIterable<T>) {
    this.dataStream = dataStream;
  }
  
  // Подписка на поток данных
  subscribe(observer: (data: T) => void): () => void {
    this.observers.push(observer);
    
    // Возвращаем функцию для отписки
    return () => {
      const index = this.observers.indexOf(observer);
      if (index >= 0) {
        this.observers.splice(index, 1);
      }
    };
  }
  
  // Параллельная трансформация данных
  map<U>(transform: (data: T) => U | Promise<U>): ReactiveSystem<U> {
    const transformedStream = this.createTransformedStream(transform);
    return new ReactiveSystem(transformedStream);
  }
  
  private async *createTransformedStream<U>(transform: (data: T) => U | Promise<U>): AsyncIterable<U> {
    for await (const data of this.dataStream) {
      const transformed = await Promise.resolve(transform(data));
      yield transformed;
    }
  }
  
  // Параллельная фильтрация данных
  filter(predicate: (data: T) => boolean | Promise<boolean>): ReactiveSystem<T> {
    const filteredStream = this.createFilteredStream(predicate);
    return new ReactiveSystem(filteredStream);
  }
  
  private async *createFilteredStream(predicate: (data: T) => boolean | Promise<boolean>): AsyncIterable<T> {
    for await (const data of this.dataStream) {
      const shouldInclude = await Promise.resolve(predicate(data));
      if (shouldInclude) {
        yield data;
      }
    }
  }
  
  // Запуск обработки потока
  async start(): Promise<void> {
    for await (const data of this.dataStream) {
      // Уведомляем всех наблюдателей параллельно
      await Promise.all(
        this.observers.map(observer => {
          try {
            return Promise.resolve(observer(data));
          } catch (error) {
            console.error("Ошибка в наблюдателе:", error);
            return Promise.resolve();
          }
        })
      );
    }
  }
  
  // Сбор данных в массив
  async toArray(maxItems: number = Infinity): Promise<T[]> {
    const results: T[] = [];
    
    for await (const data of this.dataStream) {
      results.push(data);
      if (results.length >= maxItems) {
        break;
      }
    }
    
    return results;
  }
}

// Пример использования реактивной системы
async function reactiveSystemExample() {
  // Создаем поток данных
  const dataStream = (async function* () {
    for (let i = 0; i < 100; i++) {
      await new Promise(resolve => setTimeout(resolve, 100));
      yield { id: i, value: Math.random() * 100 };
    }
  })();
  
  const system = new ReactiveSystem(dataStream);
  
  // Добавляем трансформации
  const processedSystem = system
    .filter(data => data.value > 50)
    .map(data => ({ ...data, processed: true, sqrt: Math.sqrt(data.value) }));
  
  // Подписываемся на результаты
  const unsubscribe1 = processedSystem.subscribe(data => {
    console.log("Наблюдатель 1:", data);
  });
  
  const unsubscribe2 = processedSystem.subscribe(data => {
    if (data.id % 10 === 0) {
      console.log("Наблюдатель 2 (каждый 10-й элемент):", data);
    }
  });
  
  // Запускаем обработку
  console.log("Запуск реактивной системы...");
  await processedSystem.start();
  
  console.log("Система завершена");
}

// reactiveSystemExample();
```

Параллелизм и конкурентность в функциональном программировании предоставляют мощные инструменты для создания эффективных и масштабируемых приложений. Использование таких концепций, как Future, акторы, STM и пулы воркеров, позволяет разработчикам создавать надежные системы, которые могут эффективно использовать многопроцессорные архитектуры и обрабатывать большие объемы данных параллельно.