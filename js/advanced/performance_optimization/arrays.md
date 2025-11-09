# Оптимизация работы с массивами в JavaScript

## Введение

Массивы - одна из самых часто используемых структур данных в JavaScript. Правильная оптимизация операций с массивами может значительно улучшить производительность приложений, особенно при работе с большими объемами данных.

## Эффективные методы массивов

Выбор правильных методов массивов для конкретных задач:

```javascript
// Оптимизация операций с массивами
class ArrayOptimization {
  // Неэффективная фильтрация и преобразование
  static inefficientChain(arr) {
    return arr
      .filter(x => x > 0)
      .map(x => x * 2)
      .filter(x => x < 100)
      .map(x => x + 1);
  }
  
  // Оптимизированная версия в один проход
  static optimizedSinglePass(arr) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
      const val = arr[i];
      if (val > 0) {
        const transformed = val * 2;
        if (transformed < 100) {
          result.push(transformed + 1);
        }
      }
    }
    return result;
  }
  
  // Использование reduce для комбинирования операций
  static reduceApproach(arr) {
    return arr.reduce((acc, val) => {
      if (val > 0) {
        const transformed = val * 2;
        if (transformed < 100) {
          acc.push(transformed + 1);
        }
      }
      return acc;
    }, []);
  }
  
  // Тестирование производительности
  static benchmark() {
    const data = new Array(100000).fill(0).map(() => Math.random() * 200 - 100);
    const profiler = new PerformanceProfiler();
    
    profiler.measureFunction(() => this.inefficientChain(data), 'Неэффективная цепочка');
    profiler.measureFunction(() => this.optimizedSinglePass(data), 'Оптимизированный проход');
    profiler.measureFunction(() => this.reduceApproach(data), 'Reduce подход');
  }
}
```

## Оптимизация поиска в массивах

Разные методы поиска имеют разную сложность:

```javascript
// Оптимизация поиска в массивах
class ArraySearchOptimization {
  // Линейный поиск - O(n)
  static linearSearch(arr, target) {
    for (let i = 0; i < arr.length; i++) {
      if (arr[i] === target) {
        return i;
      }
    }
    return -1;
  }
  
  // Использование indexOf - O(n)
  static indexOfSearch(arr, target) {
    return arr.indexOf(target);
  }
  
  // Использование includes - O(n)
  static includesSearch(arr, target) {
    return arr.includes(target);
  }
  
  // Поиск в отсортированном массиве - O(log n)
  static binarySearch(sortedArr, target) {
    let left = 0;
    let right = sortedArr.length - 1;
    
    while (left <= right) {
      const mid = Math.floor((left + right) / 2);
      if (sortedArr[mid] === target) {
        return mid;
      } else if (sortedArr[mid] < target) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    
    return -1;
  }
  
  // Использование Set для быстрого поиска - O(1)
  static setBasedSearch(arr, targets) {
    const set = new Set(arr);
    return targets.map(target => set.has(target));
  }
}
```

## Оптимизация сортировки массивов

Выбор правильных алгоритмов сортировки:

```javascript
// Оптимизация сортировки массивов
class ArraySortOptimization {
  // Встроенная сортировка (Timsort) - O(n log n)
  static nativeSort(arr) {
    return [...arr].sort((a, b) => a - b);
  }
  
  // Сортировка подсчетом для ограниченного диапазона - O(n + k)
  static countingSort(arr, max) {
    const counts = new Array(max + 1).fill(0);
    
    // Подсчитываем количество каждого элемента
    for (const num of arr) {
      counts[num]++;
    }
    
    // Восстанавливаем отсортированный массив
    const sorted = [];
    for (let i = 0; i < counts.length; i++) {
      for (let j = 0; j < counts[i]; j++) {
        sorted.push(i);
      }
    }
    
    return sorted;
  }
  
  // Сортировка по ключу для объектов
  static sortByKey(arr, key) {
    return [...arr].sort((a, b) => {
      if (a[key] < b[key]) return -1;
      if (a[key] > b[key]) return 1;
      return 0;
    });
  }
  
  // Стабильная сортировка
  static stableSort(arr, compareFn) {
    // Добавляем индекс для сохранения стабильности
    const indexed = arr.map((item, index) => ({ item, index }));
    
    return indexed
      .sort((a, b) => {
        const result = compareFn(a.item, b.item);
        return result !== 0 ? result : a.index - b.index;
      })
      .map(({ item }) => item);
  }
}
```

## Оптимизация вставки и удаления

Эффективные методы добавления и удаления элементов:

```javascript
// Оптимизация вставки и удаления
class ArrayInsertionDeletionOptimization {
  // Неэффективное удаление элементов
  static inefficientRemove(arr, predicate) {
    const result = [];
    for (let i = 0; i < arr.length; i++) {
      if (!predicate(arr[i])) {
        result.push(arr[i]);
      }
    }
    return result;
  }
  
  // Использование filter для удаления
  static filterRemove(arr, predicate) {
    return arr.filter(item => !predicate(item));
  }
  
  // Множественное удаление с индексами
  static multipleRemoveByIndex(arr, indicesToRemove) {
    const sortedIndices = [...indicesToRemove].sort((a, b) => b - a);
    const result = [...arr];
    
    for (const index of sortedIndices) {
      result.splice(index, 1);
    }
    
    return result;
  }
  
  // Вставка в начало массива
  static prependEfficiently(arr, items) {
    // Неэффективно: arr.unshift(...items);
    // Более эффективно:
    return [...items, ...arr];
  }
  
  // Вставка в середину массива
  static insertAt(arr, index, items) {
    // Используем splice для вставки
    const result = [...arr];
    result.splice(index, 0, ...items);
    return result;
  }
}
```

## Работа с TypedArray

Использование TypedArray для числовых данных:

```javascript
// Оптимизация с TypedArray
class TypedArrayOptimization {
  // Создание TypedArray для эффективного хранения чисел
  static createEfficientStorage(size) {
    // TypedArray использует меньше памяти чем обычные массивы
    return new Float64Array(size);
  }
  
  // Преобразование между массивами
  static convertToTyped(arr) {
    return new Float64Array(arr);
  }
  
  static convertFromTyped(typedArr) {
    return Array.from(typedArr);
  }
  
  // Операции с TypedArray
  static processTypedArray(typedArr, operation) {
    // Используем for вместо map для лучшей производительности
    for (let i = 0; i < typedArr.length; i++) {
      typedArr[i] = operation(typedArr[i]);
    }
    return typedArr;
  }
  
  // Сравнение производительности
  static performanceComparison(size) {
    const regularArray = new Array(size).fill(0).map(() => Math.random());
    const typedArray = new Float64Array(size);
    for (let i = 0; i < size; i++) {
      typedArray[i] = Math.random();
    }
    
    console.time('Regular array processing');
    const result1 = regularArray.map(x => x * 2);
    console.timeEnd('Regular array processing');
    
    console.time('TypedArray processing');
    for (let i = 0; i < typedArray.length; i++) {
      typedArray[i] *= 2;
    }
    console.timeEnd('TypedArray processing');
  }
}
```

## Оптимизация многомерных массивов

Работа с многомерными массивами:

```javascript
// Оптимизация многомерных массивов
class MultidimensionalArrayOptimization {
  // Создание многомерного массива
  static create2DArray(rows, cols, initialValue = 0) {
    // Более эффективно:
    const arr = new Array(rows);
    for (let i = 0; i < rows; i++) {
      arr[i] = new Array(cols).fill(initialValue);
    }
    return arr;
    
    // Менее эффективно:
    // return Array.from({ length: rows }, () => 
    //   Array.from({ length: cols }, () => initialValue)
    // );
  }
  
  // Линеаризация многомерного массива
  static flatten2D(arr2D) {
    const result = [];
    for (let i = 0; i < arr2D.length; i++) {
      for (let j = 0; j < arr2D[i].length; j++) {
        result.push(arr2D[i][j]);
      }
    }
    return result;
  }
  
  // Использование одномерного массива для представления 2D
  static createFlat2D(rows, cols, initialValue = 0) {
    return {
      data: new Array(rows * cols).fill(initialValue),
      rows,
      cols,
      get(row, col) {
        return this.data[row * this.cols + col];
      },
      set(row, col, value) {
        this.data[row * this.cols + col] = value;
      }
    };
  }
}
```

## Практические рекомендации

1. **Избегайте цепочек методов** - при работе с большими массивами
2. **Используйте циклы for** - вместо функциональных методов для критических участков
3. **Преобразуйте в Set** - для частых операций поиска
4. **Используйте TypedArray** - для больших массивов числовых данных
5. **Кэшируйте длину массива** - в циклах
6. **Избегайте splice** - в циклах (изменяет индексы)
7. **Используйте flatMap** - для комбинирования flat и map
8. **Применяйте reduce** - для комбинирования нескольких операций

Оптимизация работы с массивами требует понимания как алгоритмов, так и специфики JavaScript. Правильный выбор методов и подходов может значительно улучшить производительность приложений.

#javascript #arrays #optimization #performance #data-structures