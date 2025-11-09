# Оптимизация алгоритмов в JavaScript

## Введение

Алгоритмы - это основа эффективного программирования. Правильный выбор и реализация алгоритмов может значительно повлиять на производительность приложения. В этом разделе мы рассмотрим различные техники оптимизации алгоритмов в JavaScript.

## Анализ сложности алгоритмов

Понимание временной и пространственной сложности алгоритмов:

```javascript
// Примеры временной сложности
class TimeComplexityExamples {
  // O(1) - константная сложность
  static getFirstElement(arr) {
    return arr[0]; // Всегда выполняется за одно и то же время
  }
  
  // O(log n) - логарифмическая сложность
  static binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    
    while (left <= right) {
      const mid = Math.floor((left + right) / 2);
      if (arr[mid] === target) {
        return mid;
      } else if (arr[mid] < target) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    
    return -1;
  }
  
  // O(n) - линейная сложность
  static linearSearch(arr, target) {
    for (let i = 0; i < arr.length; i++) {
      if (arr[i] === target) {
        return i;
      }
    }
    return -1;
  }
  
  // O(n log n) - линейно-логарифмическая сложность
  static mergeSort(arr) {
    if (arr.length <= 1) {
      return arr;
    }
    
    const mid = Math.floor(arr.length / 2);
    const left = this.mergeSort(arr.slice(0, mid));
    const right = this.mergeSort(arr.slice(mid));
    
    return this.merge(left, right);
  }
  
  static merge(left, right) {
    const result = [];
    let i = 0;
    let j = 0;
    
    while (i < left.length && j < right.length) {
      if (left[i] < right[j]) {
        result.push(left[i]);
        i++;
      } else {
        result.push(right[j]);
        j++;
      }
    }
    
    return result.concat(left.slice(i)).concat(right.slice(j));
  }
  
  // O(n²) - квадратичная сложность
  static bubbleSort(arr) {
    const result = [...arr];
    
    for (let i = 0; i < result.length; i++) {
      for (let j = 0; j < result.length - i - 1; j++) {
        if (result[j] > result[j + 1]) {
          [result[j], result[j + 1]] = [result[j + 1], result[j]];
        }
      }
    }
    
    return result;
  }
  
  // O(2^n) - экспоненциальная сложность
  static fibonacci(n) {
    if (n <= 1) {
      return n;
    }
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}
```

## Оптимизация поиска

Эффективные алгоритмы поиска:

```javascript
// Оптимизация поиска
class SearchOptimization {
  // Бинарный поиск (O(log n))
  static binarySearch(arr, target) {
    let left = 0;
    let right = arr.length - 1;
    
    while (left <= right) {
      const mid = Math.floor((left + right) / 2);
      
      if (arr[mid] === target) {
        return mid;
      } else if (arr[mid] < target) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
    
    return -1;
  }
  
  // Поиск с использованием Map (O(1) в среднем)
  static createSearchMap(arr) {
    const map = new Map();
    arr.forEach((item, index) => {
      map.set(item, index);
    });
    return map;
  }
  
  static searchWithMap(map, target) {
    return map.get(target);
  }
  
  // Поиск с кэшированием результатов
  static createCachedSearch(arr) {
    const cache = new Map();
    
    return function(target) {
      if (cache.has(target)) {
        return cache.get(target);
      }
      
      const result = arr.indexOf(target);
      cache.set(target, result);
      return result;
    };
  }
  
  // Поиск в больших наборах данных
  static chunkedSearch(largeArray, target, chunkSize = 1000) {
    return new Promise((resolve) => {
      let currentIndex = 0;
      
      function searchChunk() {
        const endIndex = Math.min(currentIndex + chunkSize, largeArray.length);
        const chunk = largeArray.slice(currentIndex, endIndex);
        const indexInChunk = chunk.indexOf(target);
        
        if (indexInChunk !== -1) {
          resolve(currentIndex + indexInChunk);
          return;
        }
        
        currentIndex = endIndex;
        
        if (currentIndex < largeArray.length) {
          // Даем возможность выполнить другие задачи
          setTimeout(searchChunk, 0);
        } else {
          resolve(-1);
        }
      }
      
      searchChunk();
    });
  }
}
```

## Оптимизация сортировки

Эффективные алгоритмы сортировки:

```javascript
// Оптимизация сортировки
class SortOptimization {
  // Быстрая сортировка (QuickSort) - O(n log n) в среднем
  static quickSort(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
      const pivotIndex = this.partition(arr, low, high);
      this.quickSort(arr, low, pivotIndex - 1);
      this.quickSort(arr, pivotIndex + 1, high);
    }
    return arr;
  }
  
  static partition(arr, low, high) {
    const pivot = arr[high];
    let i = low - 1;
    
    for (let j = low; j < high; j++) {
      if (arr[j] <= pivot) {
        i++;
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
    }
    
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;
  }
  
  // Сортировка слиянием (MergeSort) - O(n log n)
  static mergeSort(arr) {
    if (arr.length <= 1) {
      return arr;
    }
    
    const mid = Math.floor(arr.length / 2);
    const left = this.mergeSort(arr.slice(0, mid));
    const right = this.mergeSort(arr.slice(mid));
    
    return this.merge(left, right);
  }
  
  static merge(left, right) {
    const result = [];
    let i = 0;
    let j = 0;
    
    while (i < left.length && j < right.length) {
      if (left[i] <= right[j]) {
        result.push(left[i]);
        i++;
      } else {
        result.push(right[j]);
        j++;
      }
    }
    
    return result.concat(left.slice(i)).concat(right.slice(j));
  }
  
  // Сортировка подсчетом для ограниченного диапазона значений - O(n + k)
  static countingSort(arr, maxVal) {
    const counts = new Array(maxVal + 1).fill(0);
    
    // Подсчитываем количество каждого элемента
    for (const num of arr) {
      counts[num]++;
    }
    
    // Восстанавливаем отсортированный массив
    const result = [];
    for (let i = 0; i < counts.length; i++) {
      for (let j = 0; j < counts[i]; j++) {
        result.push(i);
      }
    }
    
    return result;
  }
  
  // Гибридная сортировка (Timsort подобная)
  static hybridSort(arr) {
    const RUN = 32;
    
    // Сортировка небольших подмассивов вставками
    for (let i = 0; i < arr.length; i += RUN) {
      this.insertionSort(arr, i, Math.min(i + RUN - 1, arr.length - 1));
    }
    
    // Слияние отсортированных подмассивов
    for (let size = RUN; size < arr.length; size *= 2) {
      for (let left = 0; left < arr.length; left += 2 * size) {
        const mid = left + size - 1;
        const right = Math.min(left + 2 * size - 1, arr.length - 1);
        
        if (mid < right) {
          this.mergeInPlace(arr, left, mid, right);
        }
      }
    }
    
    return arr;
  }
  
  static insertionSort(arr, left, right) {
    for (let i = left + 1; i <= right; i++) {
      const key = arr[i];
      let j = i - 1;
      
      while (j >= left && arr[j] > key) {
        arr[j + 1] = arr[j];
        j--;
      }
      
      arr[j + 1] = key;
    }
  }
  
  static mergeInPlace(arr, left, mid, right) {
    const leftArr = arr.slice(left, mid + 1);
    const rightArr = arr.slice(mid + 1, right + 1);
    
    let i = 0, j = 0, k = left;
    
    while (i < leftArr.length && j < rightArr.length) {
      if (leftArr[i] <= rightArr[j]) {
        arr[k] = leftArr[i];
        i++;
      } else {
        arr[k] = rightArr[j];
        j++;
      }
      k++;
    }
    
    while (i < leftArr.length) {
      arr[k] = leftArr[i];
      i++;
      k++;
    }
    
    while (j < rightArr.length) {
      arr[k] = rightArr[j];
      j++;
      k++;
    }
  }
}
```

## Оптимизация структур данных

Эффективные структуры данных для оптимизации:

```javascript
// Оптимизация структур данных
class DataStructureOptimization {
  // Оптимизированная очередь
  static createOptimizedQueue() {
    return {
      items: [],
      offset: 0,
      
      enqueue(item) {
        this.items.push(item);
      },
      
      dequeue() {
        if (this.items.length === 0) {
          return undefined;
        }
        
        const item = this.items[this.offset];
        this.offset++;
        
        // Очищаем массив, если большая часть элементов уже обработана
        if (this.offset * 2 >= this.items.length) {
          this.items = this.items.slice(this.offset);
          this.offset = 0;
        }
        
        return item;
      },
      
      size() {
        return this.items.length - this.offset;
      },
      
      isEmpty() {
        return this.size() === 0;
      }
    };
  }
  
  // Оптимизированная хэш-таблица
  static createOptimizedHashTable() {
    return {
      buckets: new Array(16),
      size: 0,
      loadFactor: 0.75,
      
      hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
          hash = (hash << 5) - hash + key.charCodeAt(i);
          hash |= 0; // Преобразуем в 32-битное целое
        }
        return Math.abs(hash) % this.buckets.length;
      },
      
      set(key, value) {
        const index = this.hash(key);
        
        if (!this.buckets[index]) {
          this.buckets[index] = [];
        }
        
        const bucket = this.buckets[index];
        const existing = bucket.find(item => item.key === key);
        
        if (existing) {
          existing.value = value;
        } else {
          bucket.push({ key, value });
          this.size++;
          
          // Проверяем необходимость ресайза
          if (this.size > this.buckets.length * this.loadFactor) {
            this.resize();
          }
        }
      },
      
      get(key) {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        if (!bucket) {
          return undefined;
        }
        
        const item = bucket.find(item => item.key === key);
        return item ? item.value : undefined;
      },
      
      delete(key) {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        if (!bucket) {
          return false;
        }
        
        const itemIndex = bucket.findIndex(item => item.key === key);
        if (itemIndex !== -1) {
          bucket.splice(itemIndex, 1);
          this.size--;
          return true;
        }
        
        return false;
      },
      
      resize() {
        const oldBuckets = this.buckets;
        this.buckets = new Array(oldBuckets.length * 2);
        this.size = 0;
        
        for (const bucket of oldBuckets) {
          if (bucket) {
            for (const item of bucket) {
              this.set(item.key, item.value);
            }
          }
        }
      }
    };
  }
  
  // Оптимизированное дерево
  static createOptimizedTree() {
    return {
      root: null,
      
      insert(value) {
        this.root = this.insertNode(this.root, value);
      },
      
      insertNode(node, value) {
        if (!node) {
          return { value, left: null, right: null };
        }
        
        if (value < node.value) {
          node.left = this.insertNode(node.left, value);
        } else if (value > node.value) {
          node.right = this.insertNode(node.right, value);
        }
        
        return node;
      },
      
      search(value) {
        return this.searchNode(this.root, value);
      },
      
      searchNode(node, value) {
        if (!node) {
          return false;
        }
        
        if (value === node.value) {
          return true;
        }
        
        if (value < node.value) {
          return this.searchNode(node.left, value);
        } else {
          return this.searchNode(node.right, value);
        }
      },
      
      // Итеративный поиск для избежания переполнения стека
      searchIterative(value) {
        let current = this.root;
        
        while (current) {
          if (value === current.value) {
            return true;
          }
          
          if (value < current.value) {
            current = current.left;
          } else {
            current = current.right;
          }
        }
        
        return false;
      }
    };
  }
}
```

## Мемоизация и кэширование

Оптимизация с помощью мемоизации:

```javascript
// Мемоизация и кэширование
class MemoizationOptimization {
  // Простая мемоизация
  static memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn.apply(this, args);
      cache.set(key, result);
      return result;
    };
  }
  
  // Мемоизация с ограничением размера кэша
  static memoizeWithLimit(fn, limit = 100) {
    const cache = new Map();
    
    return function(...args) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        // Перемещаем элемент в конец (LRU)
        const value = cache.get(key);
        cache.delete(key);
        cache.set(key, value);
        return value;
      }
      
      const result = fn.apply(this, args);
      
      // Ограничиваем размер кэша
      if (cache.size >= limit) {
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
      }
      
      cache.set(key, result);
      return result;
    };
  }
  
  // Мемоизация для рекурсивных функций
  static memoizeRecursive(fn) {
    const cache = new Map();
    
    function memoized(...args) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn.apply(fn, args);
      cache.set(key, result);
      return result;
    }
    
    return memoized;
  }
  
  // Пример использования мемоизации
  static fibonacci = this.memoizeRecursive(function(n) {
    if (n <= 1) {
      return n;
    }
    return this(n - 1) + this(n - 2);
  });
  
  // Кэширование HTTP запросов
  static createCachedAPI() {
    const cache = new Map();
    const cacheTimeout = 5 * 60 * 1000; // 5 минут
    
    return {
      async fetch(url, options = {}) {
        const cacheKey = `${url}_${JSON.stringify(options)}`;
        const cached = cache.get(cacheKey);
        
        if (cached && Date.now() - cached.timestamp < cacheTimeout) {
          return cached.data;
        }
        
        try {
          const response = await fetch(url, options);
          const data = await response.json();
          
          cache.set(cacheKey, {
            data,
            timestamp: Date.now()
          });
          
          return data;
        } catch (error) {
          // В случае ошибки возвращаем кэшированные данные, если есть
          if (cached) {
            console.warn('Using cached data due to network error:', error);
            return cached.data;
          }
          throw error;
        }
      },
      
      clearCache() {
        cache.clear();
      },
      
      getCacheSize() {
        return cache.size;
      }
    };
  }
}
```

## Практические рекомендации

1. **Анализируйте сложность алгоритмов** - выбирайте оптимальные алгоритмы для задач
2. **Используйте подходящие структуры данных** - Map, Set, WeakMap, WeakSet
3. **Применяйте мемоизацию** - для функций с повторяющимися вызовами
4. **Оптимизируйте поиск** - используйте бинарный поиск или хэш-таблицы
5. **Выбирайте правильные алгоритмы сортировки** - в зависимости от данных
6. **Избегайте вложенных циклов** - когда это возможно
7. **Используйте итеративные подходы** - вместо рекурсивных для больших данных
8. **Кэшируйте результаты** - для повторяющихся вычислений

Оптимизация алгоритмов - ключ к созданию быстрых и эффективных приложений. Понимание сложности и выбор правильных алгоритмов помогут значительно улучшить производительность кода.

#javascript #algorithms #optimization #performance #sorting #searching #memoization