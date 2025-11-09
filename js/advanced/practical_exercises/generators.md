# Упражнения по генераторам и итераторам

## Упражнение 4: Создание итераторов и генераторов
Создайте различные итераторы и генераторы для работы с данными:

```javascript
// iterable-utils.js - утилиты для работы с итерируемыми объектами
class IterableUtils {
  // Создание итератора для диапазона чисел
  static *range(start, end, step = 1) {
    if (step === 0) {
      throw new Error('Шаг не может быть равен нулю');
    }
    
    if (step > 0) {
      for (let i = start; i <= end; i += step) {
        yield i;
      }
    } else {
      for (let i = start; i >= end; i += step) {
        yield i;
      }
    }
  }
  
  // Создание итератора для бесконечной последовательности
  static *infiniteSequence(start = 0, step = 1) {
    let current = start;
    while (true) {
      yield current;
      current += step;
    }
  }
  
  // Создание итератора для фибоначчи
  static *fibonacci() {
    let a = 0, b = 1;
    while (true) {
      yield a;
      [a, b] = [b, a + b];
    }
  }
  
  // Создание итератора для случайных чисел
  static *randomNumbers(min = 0, max = 1, count = Infinity) {
    let generated = 0;
    while (generated < count) {
      yield Math.random() * (max - min) + min;
      generated++;
    }
  }
  
  // Создание итератора для перебора объекта
  static *objectEntries(obj) {
    for (const key in obj) {
      if (obj.hasOwnProperty(key)) {
        yield [key, obj[key]];
      }
    }
  }
  
  // Создание итератора для "плоского" массива
  static *flatten(array) {
    for (const item of array) {
      if (Array.isArray(item)) {
        yield* this.flatten(item); // Рекурсивное разворачивание
      } else {
        yield item;
      }
    }
  }
}

// data-processor.js - процессор данных с использованием генераторов
class DataProcessor {
  // Обработка данных с помощью генераторов
  static *processData(data, processors) {
    for (const item of data) {
      let processedItem = item;
      for (const processor of processors) {
        processedItem = processor(processedItem);
        if (processedItem === null || processedItem === undefined) {
          break; // Пропускаем элемент, если он был отфильтрован
        }
      }
      if (processedItem !== null && processedItem !== undefined) {
        yield processedItem;
      }
    }
  }
  
  // Пагинация данных
  static *paginate(data, pageSize) {
    for (let i = 0; i < data.length; i += pageSize) {
      yield data.slice(i, i + pageSize);
    }
  }
  
  // Группировка данных
  static *groupBy(data, keyFunction) {
    const groups = new Map();
    
    // Группируем данные
    for (const item of data) {
      const key = keyFunction(item);
      if (!groups.has(key)) {
        groups.set(key, []);
      }
      groups.get(key).push(item);
    }
    
    // Возвращаем группы
    for (const [key, items] of groups) {
      yield [key, items];
    }
  }
  
  // Слияние нескольких итерируемых объектов
  static *merge(...iterables) {
    const iterators = iterables.map(it => it[Symbol.iterator]());
    let active = iterators.length;
    
    while (active > 0) {
      for (let i = 0; i < iterators.length; i++) {
        const iterator = iterators[i];
        if (iterator) {
          const { value, done } = iterator.next();
          if (!done) {
            yield value;
          } else {
            iterators[i] = null;
            active--;
          }
        }
      }
    }
  }
}

// async-iterable-utils.js - асинхронные итерируемые объекты
class AsyncIterableUtils {
  // Асинхронный генератор для чтения данных из API
  static async *fetchDataPages(url, pageSize = 10) {
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
      try {
        const response = await fetch(`${url}?page=${page}&limit=${pageSize}`);
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        
        const data = await response.json();
        
        // Проверяем, есть ли еще данные
        if (Array.isArray(data) && data.length > 0) {
          yield data;
          hasMore = data.length === pageSize; // Если получили полную страницу, могут быть еще данные
        } else {
          hasMore = false;
        }
        
        page++;
      } catch (error) {
        console.error(`Ошибка при получении страницы ${page}:`, error.message);
        hasMore = false;
      }
    }
  }
  
  // Асинхронный генератор для обработки потоковых данных
  static async *processStream(stream, processor) {
    for await (const chunk of stream) {
      const processed = await processor(chunk);
      if (processed !== null && processed !== undefined) {
        yield processed;
      }
    }
  }
  
  // Асинхронный генератор для таймера
  static async *timer(interval, count = Infinity) {
    let ticks = 0;
    while (ticks < count) {
      await new Promise(resolve => setTimeout(resolve, interval));
      yield ticks++;
    }
  }
}

// Тестирование итераторов и генераторов
function testIteratorsAndGenerators() {
  console.log('=== Тестирование итераторов и генераторов ===');
  
  // Тестирование range
  console.log('Диапазон от 1 до 10 с шагом 2:');
  for (const num of IterableUtils.range(1, 10, 2)) {
    process.stdout.write(num + ' ');
  }
  console.log();
  
  // Тестирование fibonacci
  console.log('Первые 10 чисел Фибоначчи:');
  let fibCount = 0;
  for (const num of IterableUtils.fibonacci()) {
    if (fibCount >= 10) break;
    process.stdout.write(num + ' ');
    fibCount++;
  }
  console.log();
  
  // Тестирование flatten
  console.log('Плоский массив:');
  const nestedArray = [1, [2, 3], [4, [5, 6]], 7];
  for (const item of IterableUtils.flatten(nestedArray)) {
    process.stdout.write(item + ' ');
  }
  console.log();
  
  // Тестирование processData
  console.log('Обработка данных:');
  const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  const processors = [
    x => x * 2,           // Умножаем на 2
    x => x % 2 === 0 ? x : null, // Фильтруем четные
    x => x > 5 ? x : null  // Фильтруем больше 5
  ];
  
  for (const item of DataProcessor.processData(numbers, processors)) {
    process.stdout.write(item + ' ');
  }
  console.log();
  
  // Тестирование paginate
  console.log('Пагинация:');
  const largeArray = Array.from({length: 25}, (_, i) => i + 1);
  let pageCount = 0;
  for (const page of DataProcessor.paginate(largeArray, 7)) {
    if (pageCount >= 3) break; // Показываем только первые 3 страницы
    console.log(`Страница ${pageCount + 1}:`, page);
    pageCount++;
  }
  
  // Тестирование groupBy
  console.log('Группировка:');
  const people = [
    { name: 'Иван', age: 25, city: 'Москва' },
    { name: 'Мария', age: 30, city: 'Москва' },
    { name: 'Алексей', age: 25, city: 'Санкт-Петербург' },
    { name: 'Елена', age: 30, city: 'Санкт-Петербург' }
  ];
  
  for (const [city, residents] of DataProcessor.groupBy(people, p => p.city)) {
    console.log(`${city}: ${residents.map(p => p.name).join(', ')}`);
  }
}

// Раскомментируйте для запуска тестов
// testIteratorsAndGenerators();
```

#javascript #generators #iterators #advanced #data-processing