# Оптимизация использования памяти в JavaScript

## Введение

Оптимизация использования памяти - ключевой аспект разработки производительных JavaScript приложений. Правильное управление памятью помогает уменьшить нагрузку на сборщик мусора, улучшить отзывчивость приложения и предотвратить утечки памяти.

## 1. Использование объектного пула

Объектный пул - это паттерн, который позволяет повторно использовать объекты вместо их постоянного создания и уничтожения.

```javascript
// Пул объектов для повторного использования
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    
    // Предварительное создание объектов
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }
  
  acquire() {
    if (this.pool.length > 0) {
      return this.pool.pop();
    }
    return this.createFn();
  }
  
  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// Пример использования пула для векторов
const vectorPool = new ObjectPool(
  () => ({ x: 0, y: 0 }),  // Создание
  (vec) => { vec.x = 0; vec.y = 0; }  // Сброс
);

function calculateDistance(x1, y1, x2, y2) {
  const vec1 = vectorPool.acquire();
  const vec2 = vectorPool.acquire();
  
  vec1.x = x1; vec1.y = y1;
  vec2.x = x2; vec2.y = y2;
  
  const dx = vec2.x - vec1.x;
  const dy = vec2.y - vec1.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  
  // Возврат объектов в пул
  vectorPool.release(vec1);
  vectorPool.release(vec2);
  
  return distance;
}
```

## 2. Использование массивов фиксированного размера

TypedArray и другие структуры данных фиксированного размера более эффективны по памяти, чем обычные массивы.

```javascript
// Использование TypedArray для эффективного хранения чисел
class EfficientDataStorage {
  constructor(size) {
    // TypedArray использует меньше памяти чем обычные массивы
    this.data = new Float64Array(size);
    this.length = 0;
  }
  
  push(value) {
    if (this.length < this.data.length) {
      this.data[this.length] = value;
      this.length++;
    }
  }
  
  get(index) {
    return this.data[index];
  }
  
  // Преобразование в обычный массив при необходимости
  toArray() {
    return Array.from(this.data.slice(0, this.length));
  }
}

const storage = new EfficientDataStorage(1000000);
for (let i = 0; i < 1000; i++) {
  storage.push(Math.random());
}
```

## 3. Ленивая инициализация

Ленивая инициализация позволяет откладывать создание дорогих объектов до момента их реального использования.

```javascript
// Ленивая инициализация для экономии памяти
class LazyInitializedClass {
  constructor() {
    this._expensiveData = null;
    this._computedValue = null;
  }
  
  get expensiveData() {
    if (this._expensiveData === null) {
      console.log('Инициализация дорогих данных');
      this._expensiveData = new Array(1000000).fill(0).map(() => Math.random());
    }
    return this._expensiveData;
  }
  
  get computedValue() {
    if (this._computedValue === null) {
      console.log('Вычисление значения');
      this._computedValue = this.expensiveData.reduce((sum, val) => sum + val, 0);
    }
    return this._computedValue;
  }
  
  // Метод для освобождения памяти
  clearCache() {
    this._expensiveData = null;
    this._computedValue = null;
  }
}
```

## 4. Использование структур данных оптимального размера

Выбор правильных структур данных может значительно повлиять на использование памяти.

```javascript
// Для небольшого количества элементов Map может быть менее эффективен
const smallObject = {
  key1: 'value1',
  key2: 'value2',
  key3: 'value3'
};

// Для большого количества элементов Map может быть более эффективен
const largeMap = new Map();
for (let i = 0; i < 10000; i++) {
  largeMap.set(`key${i}`, `value${i}`);
}
```

## 5. Минимизация создания временных объектов

Избегайте создания временных объектов в критических участках кода:

```javascript
// Неэффективно
function processArray(arr) {
  return arr.map(item => ({
    id: item.id,
    processed: true,
    timestamp: Date.now()
  }));
}

// Более эффективно
function processArray(arr) {
  const result = [];
  const timestamp = Date.now();
  
  for (let i = 0; i < arr.length; i++) {
    result.push({
      id: arr[i].id,
      processed: true,
      timestamp: timestamp
    });
  }
  
  return result;
}
```

## 6. Использование примитивов вместо объектов

Когда это возможно, используйте примитивы вместо объектов:

```javascript
// Вместо объекта для хранения координат
const pointObj = { x: 10, y: 20 };

// Используйте массив
const pointArr = [10, 20];

// Или отдельные переменные
let x = 10, y = 20;
```

## 7. Оптимизация строк

Строки в JavaScript неизменяемы, поэтому конкатенация создает новые строки:

```javascript
// Неэффективно для большого количества конкатенаций
let result = '';
for (let i = 0; i < 1000; i++) {
  result += `item${i},`;
}

// Более эффективно
const items = [];
for (let i = 0; i < 1000; i++) {
  items.push(`item${i}`);
}
const result = items.join(',');
```

## 8. Использование SharedArrayBuffer для многопоточности

В Web Workers можно использовать SharedArrayBuffer для совместного использования данных без копирования:

```javascript
// В основном потоке
const sharedBuffer = new SharedArrayBuffer(1024);
const sharedArray = new Int32Array(sharedBuffer);

// В Web Worker
// self.onmessage = function(e) {
//   const { sharedBuffer } = e.data;
//   const sharedArray = new Int32Array(sharedBuffer);
//   // Работа с общими данными
// };
```

## Практические рекомендации

1. **Профилируйте использование памяти** - используйте инструменты разработчика для мониторинга
2. **Избегайте глобальных переменных** - они не могут быть собраны сборщиком мусора
3. **Очищайте обработчики событий** - удаляйте их при удалении элементов
4. **Используйте WeakMap и WeakSet** - для хранения метаданных без предотвращения сборки
5. **Применяйте объектные пулы** - для часто создаваемых и уничтожаемых объектов
6. **Используйте TypedArray** - для больших массивов числовых данных
7. **Минимизируйте DOM манипуляции** - они могут создавать временные объекты

Оптимизация использования памяти требует баланса между производительностью и сложностью кода. Всегда измеряйте влияние оптимизаций на реальную производительность приложения.

#javascript #memory-optimization #performance #data-structures