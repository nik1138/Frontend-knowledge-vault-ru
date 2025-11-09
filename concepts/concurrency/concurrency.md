---
aliases: ["Параллелизм", "Асинхронное программирование", "Многопоточность"]
tags: 
  - #programming/concepts
  - #concurrency
  - #parallelism
  - #threads
  - #async
  - #performance
---

# Конкурентность и Параллелизм

## Определение Конкурентности и Параллелизма

**Конкурентность** (concurrency) - это способность системы обрабатывать несколько задач одновременно, но не обязательно параллельно. Это свойство системы управлять несколькими вычислениями, которые могут перекрываться во времени. Конкурентность позволяет программе эффективно использовать ресурсы, переключаясь между задачами, когда одна из них ожидает.

**Параллелизм** (parallelism) - это выполнение нескольких вычислений одновременно, в буквальном смысле параллельно, обычно на нескольких ядрах процессора. Цель параллелизма - ускорить выполнение задач за счет распределения вычислений.

> [!note] Важно
> Конкурентность - это способ организации кода, а параллелизм - это способ выполнения кода.

## Различия между Конкурентностью и Параллелизмом

| Характеристика | Конкурентность | Параллелизм |
|----------------|----------------|-------------|
| Цель | Управление несколькими задачами | Выполнение нескольких задач одновременно |
| Ресурсы | Может работать на одном ядре | Требует несколько ядер/процессоров |
| Преимущества | Лучшее использование ресурсов, отклик | Повышение производительности |
| Примеры | Веб-серверы, GUI приложения | Научные вычисления, обработка изображений |

## Потоковая Конкурентность (Thread-based Concurrency)

Потоки - это наименее абстрактный способ достижения конкурентности. Каждый поток может выполнять задачу независимо от других потоков в рамках одного процесса.

```javascript
// Пример многопоточности в Java (псевдокод)
class WorkerThread extends Thread {
    public void run() {
        System.out.println("Поток " + Thread.currentThread().getId() + " работает");
    }
}

// Создание и запуск потоков
for (int i = 0; i < 5; i++) {
    WorkerThread worker = new WorkerThread();
    worker.start();
}
```

```python
# Пример многопоточности в Python
import threading
import time

def worker(name):
    print(f"Поток {name} начал работу")
    time.sleep(2)
    print(f"Поток {name} завершил работу")

# Создание и запуск потоков
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"Worker-{i}",))
    threads.append(t)
    t.start()

# Ожидание завершения всех потоков
for t in threads:
    t.join()
```

## Асинхронное Программирование (Async Programming)

Асинхронное программирование позволяет выполнять задачи без блокировки основного потока выполнения. Вместо ожидания завершения операции, программа может продолжать выполнение других задач.

```javascript
// Пример асинхронного кода в JavaScript
async function fetchData() {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error('Ошибка при получении данных:', error);
    }
}

// Использование Promise
function delayedPromise(value, delay) {
    return new Promise((resolve) => {
        setTimeout(() => resolve(value), delay);
    });
}

// Параллельное выполнение нескольких асинхронных операций
async function runConcurrentTasks() {
    const [result1, result2] = await Promise.all([
        delayedPromise('Результат 1', 1000),
        delayedPromise('Результат 2', 1500)
    ]);
    
    console.log(result1, result2);
}
```

```python
# Пример асинхронного кода в Python
import asyncio

async def fetch_data(url):
    print(f"Начало получения данных из {url}")
    await asyncio.sleep(1)  # Имитация сетевого запроса
    print(f"Данные из {url} получены")
    return f"Данные из {url}"

async def main():
    # Параллельное выполнение асинхронных задач
    tasks = [
        fetch_data("https://api1.example.com"),
        fetch_data("https://api2.example.com"),
        fetch_data("https://api3.example.com")
    ]
    
    results = await asyncio.gather(*tasks)
    for result in results:
        print(result)

# Запуск асинхронной функции
asyncio.run(main())
```

## Гонки Данных (Race Conditions)

Гонка данных возникает, когда несколько потоков или процессов одновременно обращаются к общему ресурсу и хотя бы один из них изменяет этот ресурс. Результат зависит от того, какой поток выполнится первым, что делает программу непредсказуемой.

```python
import threading

# Пример гонки данных
counter = 0

def increment():
    global counter
    for _ in range(100000):
        counter += 1  # Это не атомарная операция!

# Создание и запуск потоков
threads = []
for i in range(2):
    t = threading.Thread(target=increment)
    threads.append(t)
    t.start()

for t in threads:
    t.join()

print(f"Результат: {counter}")  # Может быть не 200000 из-за гонки данных
```

## Механизмы Синхронизации

Для предотвращения гонок данных используются различные механизмы синхронизации:

### Блокировки (Locks)
```python
import threading

lock = threading.Lock()
counter = 0

def safe_increment():
    global counter
    for _ in range(100000):
        with lock:  # Атомарная операция
            counter += 1
```

### Семафоры
```python
import threading
import time

# Ограничение числа одновременных операций
semaphore = threading.Semaphore(2)  # Только 2 потока могут выполнить операцию одновременно

def limited_task(task_id):
    with semaphore:
        print(f"Задача {task_id} выполняется")
        time.sleep(2)
        print(f"Задача {task_id} завершена")
```

### Условные переменные
```python
import threading

condition = threading.Condition()
items = []

def consumer():
    with condition:
        while len(items) == 0:
            condition.wait()  # Ждать, пока не появятся элементы
        item = items.pop(0)
        print(f"Потреблен элемент: {item}")

def producer():
    global items
    with condition:
        items.append("новый элемент")
        condition.notify()  # Уведомить ожидающий поток
```

## Паттерны Параллельных Вычислений

### Паттерн "Разделяй и Властвуй"
```python
import concurrent.futures

def process_chunk(chunk):
    # Обработка части данных
    return sum(x * x for x in chunk)

def parallel_map(data, chunk_size=1000):
    chunks = [data[i:i+chunk_size] for i in range(0, len(data), chunk_size)]
    
    with concurrent.futures.ThreadPoolExecutor() as executor:
        results = list(executor.map(process_chunk, chunks))
    
    return sum(results)
```

### Паттерн "Воркер Пула"
```python
from multiprocessing import Pool
import os

def worker_function(n):
    # Вычисления, которые можно распределить
    return n * n

def parallel_processing(numbers):
    # Использование пула процессов
    with Pool(processes=os.cpu_count()) as pool:
        results = pool.map(worker_function, numbers)
    return results
```

## Цикл Событий (Event Loop)

Цикл событий - это конструкция программного обеспечения, которая ждет и отправляет события или сообщения в программу. Это основа асинхронного программирования в однопоточных средах.

```javascript
// Пример цикла событий в JavaScript
console.log('1. Начало выполнения');

setTimeout(() => {
    console.log('3. Выполнено через 0 мс');
}, 0);

Promise.resolve().then(() => {
    console.log('2. Выполнено как микрозадача');
});

console.log('4. Конец выполнения');

// Вывод: 1, 4, 2, 3
```

## Практические Примеры

### Пример 1: Веб-сервер с конкурентной обработкой запросов
```python
import asyncio
from aiohttp import web

async def handle_request(request):
    # Имитация обработки запроса
    await asyncio.sleep(1)
    return web.Response(text=f"Обработан запрос: {request.path}")

app = web.Application()
app.router.add_get('/', handle_request)
app.router.add_get('/api/data', handle_request)

# Запуск сервера с конкурентной обработкой
if __name__ == '__main__':
    web.run_app(app, host='127.0.0.1', port=8080)
```

### Пример 2: Параллельная обработка изображений
```python
from multiprocessing import Pool
from PIL import Image
import os

def resize_image(filename):
    with Image.open(filename) as img:
        resized = img.resize((800, 600))
        resized.save(f"resized_{filename}")
    return f"Обработано: {filename}"

def batch_resize_image(image_files):
    with Pool(processes=os.cpu_count()) as pool:
        results = pool.map(resize_image, image_files)
    return results
```

## Связи с Другими Концепциями

- [[algorithms]] - Алгоритмы могут быть адаптированы для параллельного выполнения
- [[performance-optimization]] - Конкурентность и параллелизм являются ключевыми аспектами оптимизации производительности
- [[memory-management]] - Конкурентный доступ к памяти требует специального управления
- [[thread-safety]] - Безопасность при работе с потоками
- [[event-driven-architecture]] - Архитектурный подход, основанный на обработке событий
- [[functional-programming]] - Функциональное программирование упрощает конкурентное программирование за счет отсутствия изменяемого состояния

## Заключение

Конкурентность и параллелизм - ключевые концепции современного программирования, позволяющие эффективно использовать вычислительные ресурсы. Понимание различий между этими понятиями и знание механизмов их реализации позволяет создавать более производительные и масштабируемые приложения.

Выбор подхода зависит от задачи: для улучшения отзывчивости приложений подходит асинхронное программирование, а для вычислительно интенсивных задач - параллельные вычисления.