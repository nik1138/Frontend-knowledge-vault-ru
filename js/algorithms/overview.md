# Algorithms API

## Введение

Алгоритмы и структуры данных являются фундаментом эффективного программирования. В этой главе мы рассмотрим основные алгоритмы, структуры данных и их реализацию в JavaScript: сортировка, поиск, графы, деревья, динамическое программирование и другие важные алгоритмические концепции.

## Содержание

- [[Структуры данных]]
- [[Алгоритмы сортировки]]
- [[Алгоритмы поиска]]
- [[Графовые алгоритмы]]
- [[Деревья и алгоритмы на деревьях]]
- [[Динамическое программирование]]
- [[Жадные алгоритмы]]
- [[Алгоритмы на строках]]
- [[Алгоритмы на массивах]]

## Структуры данных

### Массивы и списки

```javascript
// Связный список
class ListNode {
    constructor(val, next = null) {
        this.val = val;
        this.next = next;
    }
}

class LinkedList {
    constructor() {
        this.head = null;
        this.size = 0;
    }
    
    // Добавление элемента в начало
    prepend(val) {
        const newNode = new ListNode(val);
        newNode.next = this.head;
        this.head = newNode;
        this.size++;
        return this;
    }
    
    // Добавление элемента в конец
    append(val) {
        const newNode = new ListNode(val);
        
        if (!this.head) {
            this.head = newNode;
        } else {
            let current = this.head;
            while (current.next) {
                current = current.next;
            }
            current.next = newNode;
        }
        
        this.size++;
        return this;
    }
    
    // Вставка по индексу
    insertAt(index, val) {
        if (index < 0 || index > this.size) {
            throw new Error('Index out of bounds');
        }
        
        if (index === 0) {
            return this.prepend(val);
        }
        
        const newNode = new ListNode(val);
        let current = this.head;
        
        for (let i = 0; i < index - 1; i++) {
            current = current.next;
        }
        
        newNode.next = current.next;
        current.next = newNode;
        this.size++;
        return this;
    }
    
    // Удаление по значению
    remove(val) {
        if (!this.head) return null;
        
        if (this.head.val === val) {
            this.head = this.head.next;
            this.size--;
            return val;
        }
        
        let current = this.head;
        while (current.next) {
            if (current.next.val === val) {
                current.next = current.next.next;
                this.size--;
                return val;
            }
            current = current.next;
        }
        
        return null;
    }
    
    // Поиск элемента
    find(val) {
        let current = this.head;
        let index = 0;
        
        while (current) {
            if (current.val === val) {
                return { node: current, index };
            }
            current = current.next;
            index++;
        }
        
        return null;
    }
    
    // Получение элемента по индексу
    get(index) {
        if (index < 0 || index >= this.size) {
            throw new Error('Index out of bounds');
        }
        
        let current = this.head;
        for (let i = 0; i < index; i++) {
            current = current.next;
        }
        
        return current.val;
    }
    
    // Преобразование в массив
    toArray() {
        const result = [];
        let current = this.head;
        
        while (current) {
            result.push(current.val);
            current = current.next;
        }
        
        return result;
    }
    
    // Разворот списка
    reverse() {
        let prev = null;
        let current = this.head;
        let next = null;
        
        while (current) {
            next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        
        this.head = prev;
        return this;
    }
    
    // Длина списка
    length() {
        return this.size;
    }
    
    // Проверка на пустоту
    isEmpty() {
        return this.size === 0;
    }
}

// Пример использования
const list = new LinkedList();
list.append(1).append(2).append(3).prepend(0);
console.log(list.toArray()); // [0, 1, 2, 3]
console.log(list.get(2)); // 2
list.remove(2);
console.log(list.toArray()); // [0, 1, 3]
```

### Стек и очередь

```javascript
// Стек (LIFO - Last In, First Out)
class Stack {
    constructor() {
        this.items = [];
    }
    
    push(element) {
        this.items.push(element);
    }
    
    pop() {
        if (this.isEmpty()) {
            throw new Error('Stack is empty');
        }
        return this.items.pop();
    }
    
    peek() {
        if (this.isEmpty()) {
            return null;
        }
        return this.items[this.items.length - 1];
    }
    
    isEmpty() {
        return this.items.length === 0;
    }
    
    size() {
        return this.items.length;
    }
    
    clear() {
        this.items = [];
    }
    
    toArray() {
        return [...this.items];
    }
}

// Очередь (FIFO - First In, First Out)
class Queue {
    constructor() {
        this.items = [];
    }
    
    enqueue(element) {
        this.items.push(element);
    }
    
    dequeue() {
        if (this.isEmpty()) {
            throw new Error('Queue is empty');
        }
        return this.items.shift();
    }
    
    front() {
        if (this.isEmpty()) {
            return null;
        }
        return this.items[0];
    }
    
    rear() {
        if (this.isEmpty()) {
            return null;
        }
        return this.items[this.items.length - 1];
    }
    
    isEmpty() {
        return this.items.length === 0;
    }
    
    size() {
        return this.items.length;
    }
    
    clear() {
        this.items = [];
    }
    
    toArray() {
        return [...this.items];
    }
}

// Двусторонняя очередь (Deque)
class Deque {
    constructor() {
        this.items = [];
    }
    
    // Добавление в начало
    addFront(element) {
        this.items.unshift(element);
    }
    
    // Добавление в конец
    addRear(element) {
        this.items.push(element);
    }
    
    // Удаление из начала
    removeFront() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items.shift();
    }
    
    // Удаление из конца
    removeRear() {
        if (this.isEmpty()) {
            throw new Error('Deque is empty');
        }
        return this.items.pop();
    }
    
    front() {
        if (this.isEmpty()) return null;
        return this.items[0];
    }
    
    rear() {
        if (this.isEmpty()) return null;
        return this.items[this.items.length - 1];
    }
    
    isEmpty() {
        return this.items.length === 0;
    }
    
    size() {
        return this.items.length;
    }
    
    toArray() {
        return [...this.items];
    }
}

// Примеры использования
const stack = new Stack();
stack.push(1);
stack.push(2);
stack.push(3);
console.log(stack.pop()); // 3
console.log(stack.peek()); // 2

const queue = new Queue();
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
console.log(queue.dequeue()); // 1
console.log(queue.front()); // 2

const deque = new Deque();
deque.addRear(1);
deque.addRear(2);
deque.addFront(0);
console.log(deque.toArray()); // [0, 1, 2]
console.log(deque.removeRear()); // 2
console.log(deque.removeFront()); // 0
```

### Хэш-таблица

```javascript
// Простая реализация хэш-таблицы
class HashTable {
    constructor(size = 16) {
        this.buckets = new Array(size);
        this.size = size;
        this.count = 0;
        this.loadFactorThreshold = 0.75;
    }
    
    // Простая хэш-функция
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = (hash * 31 + key.charCodeAt(i)) % this.size;
        }
        return hash;
    }
    
    // Увеличение размера таблицы при превышении порога
    resize() {
        const oldBuckets = this.buckets;
        this.size *= 2;
        this.count = 0;
        this.buckets = new Array(this.size);
        
        for (const bucket of oldBuckets) {
            if (bucket) {
                for (const [key, value] of bucket) {
                    this.set(key, value);
                }
            }
        }
    }
    
    set(key, value) {
        // Проверка необходимости увеличения размера
        if (this.count >= this.size * this.loadFactorThreshold) {
            this.resize();
        }
        
        const index = this.hash(key);
        
        if (!this.buckets[index]) {
            this.buckets[index] = [];
        }
        
        const bucket = this.buckets[index];
        const existingPairIndex = bucket.findIndex(pair => pair[0] === key);
        
        if (existingPairIndex !== -1) {
            // Обновление существующего значения
            bucket[existingPairIndex][1] = value;
        } else {
            // Добавление новой пары
            bucket.push([key, value]);
            this.count++;
        }
    }
    
    get(key) {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        if (!bucket) return undefined;
        
        const pair = bucket.find(pair => pair[0] === key);
        return pair ? pair[1] : undefined;
    }
    
    has(key) {
        return this.get(key) !== undefined;
    }
    
    delete(key) {
        const index = this.hash(key);
        const bucket = this.buckets[index];
        
        if (!bucket) return false;
        
        const pairIndex = bucket.findIndex(pair => pair[0] === key);
        if (pairIndex === -1) return false;
        
        bucket.splice(pairIndex, 1);
        this.count--;
        
        // Очистка пустой корзины
        if (bucket.length === 0) {
            this.buckets[index] = undefined;
        }
        
        return true;
    }
    
    keys() {
        const keys = [];
        for (const bucket of this.buckets) {
            if (bucket) {
                for (const [key] of bucket) {
                    keys.push(key);
                }
            }
        }
        return keys;
    }
    
    values() {
        const values = [];
        for (const bucket of this.buckets) {
            if (bucket) {
                for (const [, value] of bucket) {
                    values.push(value);
                }
            }
        }
        return values;
    }
    
    entries() {
        const entries = [];
        for (const bucket of this.buckets) {
            if (bucket) {
                for (const [key, value] of bucket) {
                    entries.push([key, value]);
                }
            }
        }
        return entries;
    }
    
    clear() {
        this.buckets = new Array(this.size);
        this.count = 0;
    }
    
    loadFactor() {
        return this.count / this.size;
    }
}

// Пример использования
const hashTable = new HashTable();
hashTable.set('name', 'John');
hashTable.set('age', 30);
hashTable.set('city', 'New York');

console.log(hashTable.get('name')); // 'John'
console.log(hashTable.has('age')); // true
console.log(hashTable.keys()); // ['name', 'age', 'city']
console.log(hashTable.loadFactor()); // 0.125 (3/16)
```

## Алгоритмы сортировки

### Сортировка пузырьком

```javascript
// Сортировка пузырьком
function bubbleSort(arr) {
    const result = [...arr]; // Создаем копию массива
    const n = result.length;
    
    for (let i = 0; i < n - 1; i++) {
        let swapped = false;
        
        for (let j = 0; j < n - i - 1; j++) {
            if (result[j] > result[j + 1]) {
                // Меняем местами элементы
                [result[j], result[j + 1]] = [result[j + 1], result[j]];
                swapped = true;
            }
        }
        
        // Если не было обменов, массив уже отсортирован
        if (!swapped) break;
    }
    
    return result;
}

// Оптимизированная версия с подсчетом проходов
function optimizedBubbleSort(arr) {
    const result = [...arr];
    let n = result.length;
    
    while (n > 1) {
        let newN = 0;
        
        for (let i = 1; i < n; i++) {
            if (result[i - 1] > result[i]) {
                [result[i - 1], result[i]] = [result[i], result[i - 1]];
                newN = i;
            }
        }
        
        n = newN;
    }
    
    return result;
}
```

### Сортировка вставками

```javascript
// Сортировка вставками
function insertionSort(arr) {
    const result = [...arr];
    
    for (let i = 1; i < result.length; i++) {
        const current = result[i];
        let j = i - 1;
        
        // Сдвигаем элементы, которые больше текущего
        while (j >= 0 && result[j] > current) {
            result[j + 1] = result[j];
            j--;
        }
        
        // Вставляем текущий элемент на правильное место
        result[j + 1] = current;
    }
    
    return result;
}

// Бинарная сортировка вставками (оптимизированная версия)
function binaryInsertionSort(arr) {
    const result = [...arr];
    
    for (let i = 1; i < result.length; i++) {
        const current = result[i];
        
        // Найти позицию для вставки с помощью бинарного поиска
        let left = 0;
        let right = i;
        
        while (left < right) {
            const mid = Math.floor((left + right) / 2);
            if (result[mid] > current) {
                right = mid;
            } else {
                left = mid + 1;
            }
        }
        
        // Сдвинуть элементы и вставить текущий
        for (let k = i; k > left; k--) {
            result[k] = result[k - 1];
        }
        
        result[left] = current;
    }
    
    return result;
}
```

### Сортировка выбором

```javascript
// Сортировка выбором
function selectionSort(arr) {
    const result = [...arr];
    const n = result.length;
    
    for (let i = 0; i < n - 1; i++) {
        let minIndex = i;
        
        // Найти минимальный элемент в оставшейся части массива
        for (let j = i + 1; j < n; j++) {
            if (result[j] < result[minIndex]) {
                minIndex = j;
            }
        }
        
        // Поменять местами текущий элемент с минимальным
        if (minIndex !== i) {
            [result[i], result[minIndex]] = [result[minIndex], result[i]];
        }
    }
    
    return result;
}
```

### Быстрая сортировка

```javascript
// Быстрая сортировка (QuickSort)
function quickSort(arr) {
    if (arr.length <= 1) {
        return arr;
    }
    
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = [];
    const right = [];
    const equal = [];
    
    for (const element of arr) {
        if (element < pivot) {
            left.push(element);
        } else if (element > pivot) {
            right.push(element);
        } else {
            equal.push(element);
        }
    }
    
    return [
        ...quickSort(left),
        ...equal,
        ...quickSort(right)
    ];
}

// Быстрая сортировка с разделением (in-place)
function quickSortInPlace(arr, left = 0, right = arr.length - 1) {
    if (left < right) {
        const pivotIndex = partition(arr, left, right);
        quickSortInPlace(arr, left, pivotIndex - 1);
        quickSortInPlace(arr, pivotIndex + 1, right);
    }
    return arr;
}

function partition(arr, left, right) {
    const pivot = arr[right];
    let i = left - 1;
    
    for (let j = left; j < right; j++) {
        if (arr[j] <= pivot) {
            i++;
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
    }
    
    [arr[i + 1], arr[right]] = [arr[right], arr[i + 1]];
    return i + 1;
}

// Рандомизированная быстрая сортировка
function randomizedQuickSort(arr) {
    if (arr.length <= 1) {
        return arr;
    }
    
    // Выбрать случайный элемент как опорный
    const randomIndex = Math.floor(Math.random() * arr.length);
    const pivot = arr[randomIndex];
    
    const left = [];
    const right = [];
    const equal = [];
    
    for (const element of arr) {
        if (element < pivot) {
            left.push(element);
        } else if (element > pivot) {
            right.push(element);
        } else {
            equal.push(element);
        }
    }
    
    return [
        ...randomizedQuickSort(left),
        ...equal,
        ...randomizedQuickSort(right)
    ];
}
```

### Сортировка слиянием

```javascript
// Сортировка слиянием (MergeSort)
function mergeSort(arr) {
    if (arr.length <= 1) {
        return arr;
    }
    
    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));
    
    return merge(left, right);
}

function merge(left, right) {
    const result = [];
    let leftIndex = 0;
    let rightIndex = 0;
    
    while (leftIndex < left.length && rightIndex < right.length) {
        if (left[leftIndex] < right[rightIndex]) {
            result.push(left[leftIndex]);
            leftIndex++;
        } else {
            result.push(right[rightIndex]);
            rightIndex++;
        }
    }
    
    // Добавить оставшиеся элементы
    return result
        .concat(left.slice(leftIndex))
        .concat(right.slice(rightIndex));
}

// Сортировка слиянием с пользовательской функцией сравнения
function mergeSortWithComparator(arr, compareFunction) {
    if (arr.length <= 1) {
        return arr;
    }
    
    const mid = Math.floor(arr.length / 2);
    const left = mergeSortWithComparator(arr.slice(0, mid), compareFunction);
    const right = mergeSortWithComparator(arr.slice(mid), compareFunction);
    
    return mergeWithComparator(left, right, compareFunction);
}

function mergeWithComparator(left, right, compareFunction) {
    const result = [];
    let leftIndex = 0;
    let rightIndex = 0;
    
    while (leftIndex < left.length && rightIndex < right.length) {
        if (compareFunction(left[leftIndex], right[rightIndex]) <= 0) {
            result.push(left[leftIndex]);
            leftIndex++;
        } else {
            result.push(right[rightIndex]);
            rightIndex++;
        }
    }
    
    return result
        .concat(left.slice(leftIndex))
        .concat(right.slice(rightIndex));
}

// Пример сортировки объектов
const users = [
    { name: 'Alice', age: 30 },
    { name: 'Bob', age: 25 },
    { name: 'Charlie', age: 35 }
];

const sortedByAge = mergeSortWithComparator(users, (a, b) => a.age - b.age);
console.log(sortedByAge);
```

### Пирамидальная сортировка

```javascript
// Пирамидальная сортировка (HeapSort)
function heapSort(arr) {
    const result = [...arr];
    const n = result.length;
    
    // Построение кучи (перегруппировка массива)
    for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
        heapify(result, n, i);
    }
    
    // Один за другим извлекаем элементы из кучи
    for (let i = n - 1; i > 0; i--) {
        // Перемещаем текущий корень в конец
        [result[0], result[i]] = [result[i], result[0]];
        
        // Вызываем heapify на уменьшенной куче
        heapify(result, i, 0);
    }
    
    return result;
}

function heapify(arr, n, i) {
    let largest = i; // Инициализируем наибольший элемент как корень
    const left = 2 * i + 1;
    const right = 2 * i + 2;
    
    // Если левый дочерний элемент больше корня
    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }
    
    // Если правый дочерний элемент больше, чем самый большой элемент на данный момент
    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }
    
    // Если самый большой элемент не корень
    if (largest !== i) {
        [arr[i], arr[largest]] = [arr[largest], arr[i]];
        
        // Рекурсивно преобразуем в двоичную кучу затронутое поддерево
        heapify(arr, n, largest);
    }
}
```

## Алгоритмы поиска

### Линейный поиск

```javascript
// Линейный поиск
function linearSearch(arr, target) {
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === target) {
            return i; // Возвращаем индекс найденного элемента
        }
    }
    return -1; // Элемент не найден
}

// Линейный поиск с пользовательской функцией сравнения
function linearSearchWithPredicate(arr, predicate) {
    for (let i = 0; i < arr.length; i++) {
        if (predicate(arr[i], i, arr)) {
            return i;
        }
    }
    return -1;
}

// Пример использования
const numbers = [64, 34, 25, 12, 22, 11, 90];
console.log(linearSearch(numbers, 25)); // 2
console.log(linearSearchWithPredicate(numbers, x => x > 50)); // 0 (индекс 64)
```

### Бинарный поиск

```javascript
// Бинарный поиск (работает только с отсортированным массивом)
function binarySearch(arr, target) {
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
    
    return -1; // Элемент не найден
}

// Рекурсивная версия бинарного поиска
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
    if (left > right) {
        return -1;
    }
    
    const mid = Math.floor((left + right) / 2);
    
    if (arr[mid] === target) {
        return mid;
    } else if (arr[mid] < target) {
        return binarySearchRecursive(arr, target, mid + 1, right);
    } else {
        return binarySearchRecursive(arr, target, left, mid - 1);
    }
}

// Бинарный поиск для нахождения вставки
function binarySearchInsertPosition(arr, target) {
    let left = 0;
    let right = arr.length;
    
    while (left < right) {
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    
    return left; // Позиция, куда нужно вставить элемент
}

// Пример использования
const sortedNumbers = [11, 12, 22, 25, 34, 64, 90];
console.log(binarySearch(sortedNumbers, 25)); // 3
console.log(binarySearchInsertPosition(sortedNumbers, 27)); // 4 (позиция между 25 и 34)
```

### Поиск в глубину (DFS)

```javascript
// Граф представлен в виде списка смежности
class Graph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(vertex1, vertex2) {
        // Добавляем обе вершины, если их нет
        this.addVertex(vertex1);
        this.addVertex(vertex2);
        
        // Добавляем ребра (для неориентированного графа)
        this.adjacencyList.get(vertex1).push(vertex2);
        this.adjacencyList.get(vertex2).push(vertex1);
    }
    
    // Поиск в глубину (рекурсивный)
    dfsRecursive(startVertex, visited = new Set(), result = []) {
        visited.add(startVertex);
        result.push(startVertex);
        
        const neighbors = this.adjacencyList.get(startVertex) || [];
        
        for (const neighbor of neighbors) {
            if (!visited.has(neighbor)) {
                this.dfsRecursive(neighbor, visited, result);
            }
        }
        
        return result;
    }
    
    // Поиск в глубину (итеративный с использованием стека)
    dfsIterative(startVertex) {
        const stack = [startVertex];
        const visited = new Set();
        const result = [];
        
        visited.add(startVertex);
        
        while (stack.length > 0) {
            const currentVertex = stack.pop();
            result.push(currentVertex);
            
            const neighbors = this.adjacencyList.get(currentVertex) || [];
            
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    stack.push(neighbor);
                }
            }
        }
        
        return result;
    }
    
    // Поиск пути между двумя вершинами
    findPath(start, end) {
        const stack = [[start, [start]]]; // [вершина, путь до нее]
        const visited = new Set([start]);
        
        while (stack.length > 0) {
            const [currentVertex, path] = stack.pop();
            
            if (currentVertex === end) {
                return path;
            }
            
            const neighbors = this.adjacencyList.get(currentVertex) || [];
            
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    stack.push([neighbor, [...path, neighbor]]);
                }
            }
        }
        
        return null; // Путь не найден
    }
}

// Пример использования
const graph = new Graph();
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('C', 'E');
graph.addEdge('D', 'E');

console.log('DFS рекурсивный:', graph.dfsRecursive('A'));
console.log('DFS итеративный:', graph.dfsIterative('A'));
console.log('Путь от A до E:', graph.findPath('A', 'E'));
```

### Поиск в ширину (BFS)

```javascript
// Поиск в ширину
class GraphBFS {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(vertex1, vertex2) {
        this.addVertex(vertex1);
        this.addVertex(vertex2);
        
        this.adjacencyList.get(vertex1).push(vertex2);
        this.adjacencyList.get(vertex2).push(vertex1);
    }
    
    // Поиск в ширину
    bfs(startVertex) {
        const queue = [startVertex];
        const visited = new Set([startVertex]);
        const result = [];
        
        while (queue.length > 0) {
            const currentVertex = queue.shift();
            result.push(currentVertex);
            
            const neighbors = this.adjacencyList.get(currentVertex) || [];
            
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    queue.push(neighbor);
                }
            }
        }
        
        return result;
    }
    
    // Найти кратчайший путь (в терминах количества ребер)
    shortestPath(start, end) {
        if (start === end) return [start];
        
        const queue = [[start, [start]]]; // [вершина, путь до нее]
        const visited = new Set([start]);
        
        while (queue.length > 0) {
            const [currentVertex, path] = queue.shift();
            
            const neighbors = this.adjacencyList.get(currentVertex) || [];
            
            for (const neighbor of neighbors) {
                if (neighbor === end) {
                    return [...path, neighbor];
                }
                
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    queue.push([neighbor, [...path, neighbor]]);
                }
            }
        }
        
        return null; // Путь не найден
    }
    
    // Найти расстояние от стартовой вершины до всех остальных
    bfsDistance(start) {
        const queue = [start];
        const distances = new Map([[start, 0]]);
        const visited = new Set([start]);
        
        while (queue.length > 0) {
            const currentVertex = queue.shift();
            const currentDistance = distances.get(currentVertex);
            
            const neighbors = this.adjacencyList.get(currentVertex) || [];
            
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    distances.set(neighbor, currentDistance + 1);
                    queue.push(neighbor);
                }
            }
        }
        
        return distances;
    }
}

// Пример использования
const bfsGraph = new GraphBFS();
bfsGraph.addEdge('A', 'B');
bfsGraph.addEdge('A', 'C');
bfsGraph.addEdge('B', 'D');
bfsGraph.addEdge('C', 'E');
bfsGraph.addEdge('D', 'E');
bfsGraph.addEdge('E', 'F');

console.log('BFS:', bfsGraph.bfs('A'));
console.log('Кратчайший путь от A до F:', bfsGraph.shortestPath('A', 'F'));
console.log('Расстояния от A:', Object.fromEntries(bfsGraph.bfsDistance('A')));
```

## Графовые алгоритмы

### Алгоритм Дейкстры

```javascript
// Взвешенный граф
class WeightedGraph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(vertex1, vertex2, weight) {
        this.addVertex(vertex1);
        this.addVertex(vertex2);
        
        this.adjacencyList.get(vertex1).push({ node: vertex2, weight });
        this.adjacencyList.get(vertex2).push({ node: vertex1, weight }); // Для неориентированного графа
    }
    
    // Алгоритм Дейкстры для нахождения кратчайших путей
    dijkstra(start, end = null) {
        const distances = new Map();
        const previous = new Map();
        const visited = new Set();
        const nodes = new MinPriorityQueue();
        
        // Инициализация расстояний
        for (const [vertex] of this.adjacencyList) {
            if (vertex === start) {
                distances.set(vertex, 0);
                nodes.enqueue(vertex, 0);
            } else {
                distances.set(vertex, Infinity);
                nodes.enqueue(vertex, Infinity);
            }
            previous.set(vertex, null);
        }
        
        while (!nodes.isEmpty()) {
            const smallest = nodes.dequeue();
            
            if (smallest === end) {
                break; // Найден кратчайший путь до конечной вершины
            }
            
            if (distances.get(smallest) === Infinity) {
                continue; // Все оставшиеся вершины недостижимы
            }
            
            visited.add(smallest);
            
            const neighbors = this.adjacencyList.get(smallest) || [];
            
            for (const neighbor of neighbors) {
                if (!visited.has(neighbor.node)) {
                    const candidate = distances.get(smallest) + neighbor.weight;
                    const currentDistance = distances.get(neighbor.node);
                    
                    if (candidate < currentDistance) {
                        distances.set(neighbor.node, candidate);
                        previous.set(neighbor.node, smallest);
                        nodes.enqueue(neighbor.node, candidate);
                    }
                }
            }
        }
        
        // Восстановление пути до конечной вершины
        if (end) {
            const path = [];
            let current = end;
            
            while (current !== null) {
                path.unshift(current);
                current = previous.get(current);
            }
            
            return {
                distance: distances.get(end),
                path: path.length > 1 ? path : null
            };
        }
        
        // Возврат всех расстояний
        return distances;
    }
}

// Минимальная приоритетная очередь для алгоритма Дейкстры
class MinPriorityQueue {
    constructor() {
        this.values = [];
    }
    
    enqueue(val, priority) {
        this.values.push({ val, priority });
        this.sort();
    }
    
    dequeue() {
        return this.values.shift()?.val;
    }
    
    sort() {
        this.values.sort((a, b) => a.priority - b.priority);
    }
    
    isEmpty() {
        return this.values.length === 0;
    }
}

// Пример использования
const weightedGraph = new WeightedGraph();
weightedGraph.addEdge('A', 'B', 4);
weightedGraph.addEdge('A', 'C', 2);
weightedGraph.addEdge('B', 'C', 1);
weightedGraph.addEdge('B', 'D', 5);
weightedGraph.addEdge('C', 'D', 8);
weightedGraph.addEdge('C', 'E', 10);
weightedGraph.addEdge('D', 'E', 2);

const dijkstraResult = weightedGraph.dijkstra('A', 'E');
console.log('Кратчайший путь от A до E:', dijkstraResult);
```

### Алгоритм поиска компонентов связности

```javascript
// Найти все компоненты связности в графе
class ConnectedComponents {
    constructor() {
        this.graph = new Map();
    }
    
    addEdge(vertex1, vertex2) {
        if (!this.graph.has(vertex1)) this.graph.set(vertex1, []);
        if (!this.graph.has(vertex2)) this.graph.set(vertex2, []);
        
        this.graph.get(vertex1).push(vertex2);
        this.graph.get(vertex2).push(vertex1);
    }
    
    findComponents() {
        const visited = new Set();
        const components = [];
        
        for (const vertex of this.graph.keys()) {
            if (!visited.has(vertex)) {
                const component = [];
                this.dfs(vertex, visited, component);
                components.push(component);
            }
        }
        
        return components;
    }
    
    dfs(vertex, visited, component) {
        visited.add(vertex);
        component.push(vertex);
        
        const neighbors = this.graph.get(vertex) || [];
        
        for (const neighbor of neighbors) {
            if (!visited.has(neighbor)) {
                this.dfs(neighbor, visited, component);
            }
        }
    }
    
    // Проверить, связаны ли две вершины
    areConnected(vertex1, vertex2) {
        const visited = new Set();
        return this.dfsCheck(vertex1, vertex2, visited);
    }
    
    dfsCheck(current, target, visited) {
        if (current === target) return true;
        
        visited.add(current);
        
        const neighbors = this.graph.get(current) || [];
        
        for (const neighbor of neighbors) {
            if (!visited.has(neighbor)) {
                if (this.dfsCheck(neighbor, target, visited)) {
                    return true;
                }
            }
        }
        
        return false;
    }
}

// Пример использования
const cc = new ConnectedComponents();
cc.addEdge('A', 'B');
cc.addEdge('B', 'C');
cc.addEdge('D', 'E');
cc.addEdge('F', 'G');
cc.addEdge('G', 'H');

console.log('Компоненты связности:', cc.findComponents());
console.log('A и C связаны:', cc.areConnected('A', 'C'));
console.log('A и D связаны:', cc.areConnected('A', 'D'));
```

## Деревья и алгоритмы на деревьях

### Бинарное дерево поиска

```javascript
// Узел бинарного дерева
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}

// Бинарное дерево поиска
class BinarySearchTree {
    constructor() {
        this.root = null;
    }
    
    // Вставка значения
    insert(val) {
        this.root = this.insertNode(this.root, val);
        return this;
    }
    
    insertNode(node, val) {
        if (!node) {
            return new TreeNode(val);
        }
        
        if (val < node.val) {
            node.left = this.insertNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.insertNode(node.right, val);
        }
        // Если val === node.val, не вставляем дубликат
        
        return node;
    }
    
    // Поиск значения
    search(val) {
        return this.searchNode(this.root, val);
    }
    
    searchNode(node, val) {
        if (!node || node.val === val) {
            return node;
        }
        
        if (val < node.val) {
            return this.searchNode(node.left, val);
        } else {
            return this.searchNode(node.right, val);
        }
    }
    
    // Удаление значения
    remove(val) {
        this.root = this.removeNode(this.root, val);
        return this;
    }
    
    removeNode(node, val) {
        if (!node) {
            return null;
        }
        
        if (val < node.val) {
            node.left = this.removeNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.removeNode(node.right, val);
        } else {
            // Найдена вершина для удаления
            
            // Случай 1: узел без детей
            if (!node.left && !node.right) {
                return null;
            }
            
            // Случай 2: узел с одним ребенком
            if (!node.left) {
                return node.right;
            }
            if (!node.right) {
                return node.left;
            }
            
            // Случай 3: узел с двумя детьми
            const minRight = this.findMin(node.right);
            node.val = minRight.val;
            node.right = this.removeNode(node.right, minRight.val);
        }
        
        return node;
    }
    
    findMin(node) {
        while (node.left) {
            node = node.left;
        }
        return node;
    }
    
    findMax(node = this.root) {
        if (!node) return null;
        while (node.right) {
            node = node.right;
        }
        return node;
    }
    
    // Обходы дерева
    inOrderTraversal() {
        const result = [];
        this.inOrder(this.root, result);
        return result;
    }
    
    inOrder(node, result) {
        if (node) {
            this.inOrder(node.left, result);
            result.push(node.val);
            this.inOrder(node.right, result);
        }
    }
    
    preOrderTraversal() {
        const result = [];
        this.preOrder(this.root, result);
        return result;
    }
    
    preOrder(node, result) {
        if (node) {
            result.push(node.val);
            this.preOrder(node.left, result);
            this.preOrder(node.right, result);
        }
    }
    
    postOrderTraversal() {
        const result = [];
        this.postOrder(this.root, result);
        return result;
    }
    
    postOrder(node, result) {
        if (node) {
            this.postOrder(node.left, result);
            this.postOrder(node.right, result);
            result.push(node.val);
        }
    }
    
    // Обход в ширину (BFS)
    levelOrderTraversal() {
        if (!this.root) return [];
        
        const result = [];
        const queue = [this.root];
        
        while (queue.length > 0) {
            const node = queue.shift();
            result.push(node.val);
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        
        return result;
    }
    
    // Проверка, является ли дерево BST
    isValidBST() {
        return this.validateBST(this.root, null, null);
    }
    
    validateBST(node, min, max) {
        if (!node) return true;
        
        if ((min !== null && node.val <= min) || (max !== null && node.val >= max)) {
            return false;
        }
        
        return this.validateBST(node.left, min, node.val) && 
               this.validateBST(node.right, node.val, max);
    }
    
    // Поиск следующего элемента (in-order successor)
    findSuccessor(val) {
        let current = this.root;
        let successor = null;
        
        while (current) {
            if (val < current.val) {
                successor = current;
                current = current.left;
            } else {
                current = current.right;
            }
        }
        
        return successor;
    }
    
    // Поиск предыдущего элемента (in-order predecessor)
    findPredecessor(val) {
        let current = this.root;
        let predecessor = null;
        
        while (current) {
            if (val > current.val) {
                predecessor = current;
                current = current.right;
            } else {
                current = current.left;
            }
        }
        
        return predecessor;
    }
}

// Пример использования
const bst = new BinarySearchTree();
bst.insert(50).insert(30).insert(70).insert(20).insert(40).insert(60).insert(80);

console.log('In-order traversal:', bst.inOrderTraversal()); // [20, 30, 40, 50, 60, 70, 80]
console.log('Pre-order traversal:', bst.preOrderTraversal()); // [50, 30, 20, 40, 70, 60, 80]
console.log('Level-order traversal:', bst.levelOrderTraversal()); // [50, 30, 70, 20, 40, 60, 80]
console.log('Is valid BST:', bst.isValidBST()); // true
console.log('Search 40:', bst.search(40) ? 'Found' : 'Not found'); // Found
console.log('Min value:', bst.findMin(bst.root).val); // 20
console.log('Max value:', bst.findMax().val); // 80
```

### AVL дерево (самобалансирующееся дерево)

```javascript
// AVL дерево - самобалансирующееся бинарное дерево поиска
class AVLNode {
    constructor(val) {
        this.val = val;
        this.left = null;
        this.right = null;
        this.height = 1;
    }
}

class AVLTree {
    constructor() {
        this.root = null;
    }
    
    getHeight(node) {
        return node ? node.height : 0;
    }
    
    getBalance(node) {
        return node ? this.getHeight(node.left) - this.getHeight(node.right) : 0;
    }
    
    updateHeight(node) {
        if (node) {
            node.height = 1 + Math.max(this.getHeight(node.left), this.getHeight(node.right));
        }
    }
    
    // Правый поворот
    rotateRight(y) {
        const x = y.left;
        const T2 = x.right;
        
        // Выполнить поворот
        x.right = y;
        y.left = T2;
        
        // Обновить высоты
        this.updateHeight(y);
        this.updateHeight(x);
        
        // Возвратить новый корень
        return x;
    }
    
    // Левый поворот
    rotateLeft(x) {
        const y = x.right;
        const T2 = y.left;
        
        // Выполнить поворот
        y.left = x;
        x.right = T2;
        
        // Обновить высоты
        this.updateHeight(x);
        this.updateHeight(y);
        
        // Возвратить новый корень
        return y;
    }
    
    insert(val) {
        this.root = this.insertNode(this.root, val);
        return this;
    }
    
    insertNode(node, val) {
        // Выполнить стандартную вставку BST
        if (!node) {
            return new AVLNode(val);
        }
        
        if (val < node.val) {
            node.left = this.insertNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.insertNode(node.right, val);
        } else {
            // Равные значения не допускаются в BST
            return node;
        }
        
        // Обновить высоту текущего узла
        this.updateHeight(node);
        
        // Получить фактор баланса для проверки необходимости балансировки
        const balance = this.getBalance(node);
        
        // Если узел стал несбалансированным, то есть 4 случая
        
        // Левый левый случай
        if (balance > 1 && val < node.left.val) {
            return this.rotateRight(node);
        }
        
        // Правый правый случай
        if (balance < -1 && val > node.right.val) {
            return this.rotateLeft(node);
        }
        
        // Левый правый случай
        if (balance > 1 && val > node.left.val) {
            node.left = this.rotateLeft(node.left);
            return this.rotateRight(node);
        }
        
        // Правый левый случай
        if (balance < -1 && val < node.right.val) {
            node.right = this.rotateRight(node.right);
            return this.rotateLeft(node);
        }
        
        // Возвратить (неизмененный) указатель узла
        return node;
    }
    
    // Вспомогательная функция для удаления узла
    remove(val) {
        this.root = this.removeNode(this.root, val);
        return this;
    }
    
    removeNode(node, val) {
        // Выполнить стандартную операцию BST
        if (!node) {
            return node;
        }
        
        if (val < node.val) {
            node.left = this.removeNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.removeNode(node.right, val);
        } else {
            // Узел для удаления найден
            if (!node.left || !node.right) {
                const temp = node.left || node.right;
                
                if (!temp) {
                    // Нет детей
                    node = null;
                } else {
                    // Один ребенок
                    node = temp;
                }
            } else {
                // Узел с двумя детьми: получить inorder successor
                const temp = this.getMinValueNode(node.right);
                
                // Скопировать данные из inorder successor
                node.val = temp.val;
                
                // Удалить inorder successor
                node.right = this.removeNode(node.right, temp.val);
            }
        }
        
        if (!node) {
            return node;
        }
        
        // Обновить высоту текущего узла
        this.updateHeight(node);
        
        // Получить фактор баланса
        const balance = this.getBalance(node);
        
        // Если узел стал несбалансированным, то есть 4 случая
        
        // Левый левый случай
        if (balance > 1 && this.getBalance(node.left) >= 0) {
            return this.rotateRight(node);
        }
        
        // Левый правый случай
        if (balance > 1 && this.getBalance(node.left) < 0) {
            node.left = this.rotateLeft(node.left);
            return this.rotateRight(node);
        }
        
        // Правый правый случай
        if (balance < -1 && this.getBalance(node.right) <= 0) {
            return this.rotateLeft(node);
        }
        
        // Правый левый случай
        if (balance < -1 && this.getBalance(node.right) > 0) {
            node.right = this.rotateRight(node.right);
            return this.rotateLeft(node);
        }
        
        return node;
    }
    
    getMinValueNode(node) {
        let current = node;
        while (current.left) {
            current = current.left;
        }
        return current;
    }
    
    search(val) {
        return this.searchNode(this.root, val);
    }
    
    searchNode(node, val) {
        if (!node || node.val === val) {
            return node;
        }
        
        if (val < node.val) {
            return this.searchNode(node.left, val);
        } else {
            return this.searchNode(node.right, val);
        }
    }
    
    // Обход дерева
    inOrderTraversal() {
        const result = [];
        this.inOrder(this.root, result);
        return result;
    }
    
    inOrder(node, result) {
        if (node) {
            this.inOrder(node.left, result);
            result.push({ val: node.val, height: node.height });
            this.inOrder(node.right, result);
        }
    }
}

// Пример использования AVL дерева
const avl = new AVLTree();
avl.insert(10).insert(20).insert(30).insert(40).insert(50).insert(25);

console.log('AVL tree in-order:', avl.inOrderTraversal());
```

## Динамическое программирование

### Фибоначчи с мемоизацией

```javascript
// Класс для демонстрации динамического программирования
class DynamicProgramming {
    constructor() {
        this.memo = new Map();
    }
    
    // Числа Фибоначчи с мемоизацией
    fibonacci(n) {
        if (n <= 1) return n;
        
        if (this.memo.has(n)) {
            return this.memo.get(n);
        }
        
        const result = this.fibonacci(n - 1) + this.fibonacci(n - 2);
        this.memo.set(n, result);
        return result;
    }
    
    // Числа Фибоначчи с табуляцией (итеративный подход)
    fibonacciTabulation(n) {
        if (n <= 1) return n;
        
        const dp = new Array(n + 1);
        dp[0] = 0;
        dp[1] = 1;
        
        for (let i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // Задача о рюкзаке (0-1)
    knapsack(weights, values, capacity) {
        const n = weights.length;
        const dp = Array(n + 1).fill().map(() => Array(capacity + 1).fill(0));
        
        for (let i = 1; i <= n; i++) {
            for (let w = 1; w <= capacity; w++) {
                // Не брать текущий предмет
                dp[i][w] = dp[i - 1][w];
                
                // Попробовать взять текущий предмет
                if (weights[i - 1] <= w) {
                    dp[i][w] = Math.max(
                        dp[i][w],
                        dp[i - 1][w - weights[i - 1]] + values[i - 1]
                    );
                }
            }
        }
        
        return dp[n][capacity];
    }
    
    // Наибольшая общая подпоследовательность (LCS)
    longestCommonSubsequence(str1, str2) {
        const m = str1.length;
        const n = str2.length;
        
        // Создаем таблицу для хранения результатов
        const dp = Array(m + 1).fill().map(() => Array(n + 1).fill(0));
        
        // Заполняем таблицу
        for (let i = 1; i <= m; i++) {
            for (let j = 1; j <= n; j++) {
                if (str1[i - 1] === str2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        
        // Восстановление самой подпоследовательности
        let lcs = '';
        let i = m, j = n;
        
        while (i > 0 && j > 0) {
            if (str1[i - 1] === str2[j - 1]) {
                lcs = str1[i - 1] + lcs;
                i--;
                j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                i--;
            } else {
                j--;
            }
        }
        
        return {
            length: dp[m][n],
            sequence: lcs
        };
    }
    
    // Наибольшая возрастающая подпоследовательность (LIS)
    longestIncreasingSubsequence(arr) {
        if (arr.length === 0) return 0;
        
        const dp = new Array(arr.length).fill(1);
        
        for (let i = 1; i < arr.length; i++) {
            for (let j = 0; j < i; j++) {
                if (arr[j] < arr[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
        }
        
        return Math.max(...dp);
    }
    
    // Оптимальное бинарное дерево поиска
    optimalBST(keys, freq) {
        const n = keys.length;
        
        // cost[i][j] будет хранить стоимость оптимального BST для keys[i..j]
        const cost = Array(n).fill().map(() => Array(n).fill(0));
        
        // Инициализация диагональных значений
        for (let i = 0; i < n; i++) {
            cost[i][i] = freq[i];
        }
        
        // Рассчитать стоимость для подмножеств разного размера
        for (let L = 2; L <= n; L++) {
            for (let i = 0; i <= n - L; i++) {
                const j = i + L - 1;
                cost[i][j] = Number.MAX_SAFE_INTEGER;
                
                // Попробовать сделать все ключи в диапазоне [i..j] корнем
                let sum = this.sum(freq, i, j);
                
                for (let r = i; r <= j; r++) {
                    const c = (r > i ? cost[i][r - 1] : 0) +
                             (r < j ? cost[r + 1][j] : 0) +
                             sum;
                    if (c < cost[i][j]) {
                        cost[i][j] = c;
                    }
                }
            }
        }
        
        return cost[0][n - 1];
    }
    
    sum(freq, i, j) {
        let s = 0;
        for (let k = i; k <= j; k++) {
            s += freq[k];
        }
        return s;
    }
}

// Примеры использования
const dp = new DynamicProgramming();

console.log('Fibonacci(10):', dp.fibonacci(10)); // 55
console.log('Fibonacci tabulation(10):', dp.fibonacciTabulation(10)); // 55

const weights = [10, 20, 30];
const values = [60, 100, 120];
const capacity = 50;
console.log('Knapsack result:', dp.knapsack(weights, values, capacity)); // 220

const lcs = dp.longestCommonSubsequence('ABCDGH', 'AEDFHR');
console.log('LCS:', lcs); // { length: 3, sequence: 'ADH' }

const lis = dp.longestIncreasingSubsequence([10, 22, 9, 33, 21, 50, 41, 60]);
console.log('LIS length:', lis); // 5

const keys = [10, 12, 20];
const freq = [34, 8, 50];
console.log('Optimal BST cost:', dp.optimalBST(keys, freq));
```

## Жадные алгоритмы

### Задача о выборе активностей

```javascript
// Жадные алгоритмы
class GreedyAlgorithms {
    // Задача о выборе активностей
    // Выбираем максимальное количество непересекающихся активностей
    activitySelection(activities) {
        // Сортируем активности по времени окончания
        const sorted = [...activities].sort((a, b) => a.end - b.end);
        const selected = [];
        
        if (sorted.length === 0) return selected;
        
        selected.push(sorted[0]);
        let lastEnd = sorted[0].end;
        
        for (let i = 1; i < sorted.length; i++) {
            if (sorted[i].start >= lastEnd) {
                selected.push(sorted[i]);
                lastEnd = sorted[i].end;
            }
        }
        
        return selected;
    }
    
    // Задача о минимальном количестве монет
    minCoins(coins, amount) {
        coins.sort((a, b) => b - a); // Сортируем в порядке убывания
        
        const result = [];
        let remaining = amount;
        
        for (const coin of coins) {
            while (remaining >= coin) {
                result.push(coin);
                remaining -= coin;
            }
        }
        
        return remaining === 0 ? result : null; // null если невозможно
    }
    
    // Кодирование Хаффмана (упрощенная версия)
    huffmanEncoding(frequencies) {
        const nodes = Object.entries(frequencies).map(([char, freq]) => ({
            char,
            freq,
            left: null,
            right: null
        }));
        
        // Используем простую сортировку вместо приоритетной очереди
        while (nodes.length > 1) {
            nodes.sort((a, b) => a.freq - b.freq);
            
            const left = nodes.shift();
            const right = nodes.shift();
            
            const merged = {
                char: null,
                freq: left.freq + right.freq,
                left,
                right
            };
            
            nodes.push(merged);
        }
        
        // Генерация кодов
        const codes = {};
        if (nodes.length === 1) {
            this.generateCodes(nodes[0], '', codes);
        }
        
        return codes;
    }
    
    generateCodes(node, code, codes) {
        if (node.char !== null) {
            codes[node.char] = code || '0'; // Для одного символа
        } else {
            if (node.left) this.generateCodes(node.left, code + '0', codes);
            if (node.right) this.generateCodes(node.right, code + '1', codes);
        }
    }
    
    // Жадный алгоритм для задачи коммивояжера (приближенное решение)
    travelingSalesman(graph, start) {
        const unvisited = new Set(Object.keys(graph));
        const path = [start];
        unvisited.delete(start);
        let current = start;
        
        while (unvisited.size > 0) {
            let nearest = null;
            let minDistance = Infinity;
            
            for (const neighbor of unvisited) {
                const distance = graph[current][neighbor];
                if (distance < minDistance) {
                    minDistance = distance;
                    nearest = neighbor;
                }
            }
            
            path.push(nearest);
            unvisited.delete(nearest);
            current = nearest;
        }
        
        // Вернуться в начальную точку
        path.push(start);
        return path;
    }
}

// Примеры использования
const greedy = new GreedyAlgorithms();

// Задача о выборе активностей
const activities = [
    { start: 1, end: 4 },
    { start: 3, end: 5 },
    { start: 0, end: 6 },
    { start: 5, end: 7 },
    { start: 8, end: 9 },
    { start: 5, end: 9 }
];

console.log('Selected activities:', greedy.activitySelection(activities));

// Задача о минимальном количестве монет
const coins = [1, 5, 10, 25];
console.log('Min coins for 30:', greedy.minCoins(coins, 30)); // [25, 5]

// Кодирование Хаффмана
const frequencies = { a: 45, b: 13, c: 12, d: 16, e: 9, f: 5 };
console.log('Huffman codes:', greedy.huffmanEncoding(frequencies));

// Задача коммивояжера
const graph = {
    A: { B: 10, C: 15, D: 20 },
    B: { A: 10, C: 35, D: 25 },
    C: { A: 15, B: 35, D: 30 },
    D: { A: 20, B: 25, C: 30 }
};

console.log('TSP path:', greedy.travelingSalesman(graph, 'A'));
```

## Примеры из промышленной разработки

### Алгоритм поиска в тексте (Кнут-Моррис-Пратт)

```javascript
// Алгоритм Кнута-Морриса-Пратта для поиска подстроки
class KMPMatcher {
    // Вычисление префикс-функции
    computeLPS(pattern) {
        const lps = new Array(pattern.length).fill(0);
        let len = 0;
        let i = 1;
        
        while (i < pattern.length) {
            if (pattern[i] === pattern[len]) {
                len++;
                lps[i] = len;
                i++;
            } else {
                if (len !== 0) {
                    len = lps[len - 1];
                } else {
                    lps[i] = 0;
                    i++;
                }
            }
        }
        
        return lps;
    }
    
    // Поиск подстроки в строке
    search(text, pattern) {
        if (!pattern) return 0;
        if (pattern.length > text.length) return -1;
        
        const lps = this.computeLPS(pattern);
        const results = [];
        
        let i = 0; // индекс для text
        let j = 0; // индекс для pattern
        
        while (i < text.length) {
            if (pattern[j] === text[i]) {
                i++;
                j++;
            }
            
            if (j === pattern.length) {
                results.push(i - j);
                j = lps[j - 1];
            } else if (i < text.length && pattern[j] !== text[i]) {
                if (j !== 0) {
                    j = lps[j - 1];
                } else {
                    i++;
                }
            }
        }
        
        return results.length > 0 ? results : [-1];
    }
}

// Алгоритм Рабина-Карпа
class RabinKarpMatcher {
    constructor(prime = 101) {
        this.prime = prime;
    }
    
    search(text, pattern) {
        const results = [];
        const textLen = text.length;
        const patternLen = pattern.length;
        
        if (patternLen > textLen) return [-1];
        
        const patternHash = this.calculateHash(pattern, patternLen);
        let textHash = this.calculateHash(text, patternLen);
        const h = this.calculateH(patternLen);
        
        for (let i = 0; i <= textLen - patternLen; i++) {
            if (patternHash === textHash) {
                // Проверка на совпадение
                let match = true;
                for (let j = 0; j < patternLen; j++) {
                    if (text[i + j] !== pattern[j]) {
                        match = false;
                        break;
                    }
                }
                if (match) {
                    results.push(i);
                }
            }
            
            // Рассчитываем хэш для следующего окна
            if (i < textLen - patternLen) {
                textHash = this.recalculateHash(text, i, i + patternLen, textHash, h);
            }
        }
        
        return results.length > 0 ? results : [-1];
    }
    
    calculateHash(str, len) {
        let hash = 0;
        for (let i = 0; i < len; i++) {
            hash = (hash * 256 + str.charCodeAt(i)) % this.prime;
        }
        return hash;
    }
    
    recalculateHash(str, oldIndex, newIndex, oldHash, h) {
        let newHash = oldHash - str.charCodeAt(oldIndex) * h;
        newHash = (newHash * 256 + str.charCodeAt(newIndex)) % this.prime;
        
        if (newHash < 0) {
            newHash += this.prime;
        }
        
        return newHash;
    }
    
    calculateH(len) {
        let h = 1;
        for (let i = 0; i < len - 1; i++) {
            h = (h * 256) % this.prime;
        }
        return h;
    }
}

// Пример использования
const kmp = new KMPMatcher();
const rk = new RabinKarpMatcher();

const text = "ABABDABACDABABCABCABCABCABC";
const pattern = "ABABCABCABCABC";

console.log('KMP search results:', kmp.search(text, pattern));
console.log('Rabin-Karp search results:', rk.search(text, pattern));
```

### Система кэширования с LRU

```javascript
// LRU (Least Recently Used) кэш
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map();
        
        // Двусвязный список для отслеживания порядка использования
        this.head = {};
        this.tail = {};
        this.head.next = this.tail;
        this.tail.prev = this.head;
    }
    
    get(key) {
        if (this.cache.has(key)) {
            const node = this.cache.get(key);
            
            // Перемещаем узел в начало (наиболее недавно использованный)
            this.moveToHead(node);
            
            return node.value;
        }
        
        return -1;
    }
    
    put(key, value) {
        if (this.cache.has(key)) {
            // Обновляем значение и перемещаем в начало
            const node = this.cache.get(key);
            node.value = value;
            this.moveToHead(node);
        } else {
            // Создаем новый узел
            const newNode = { key, value };
            
            if (this.cache.size >= this.capacity) {
                // Удаляем наименее недавно использованный элемент
                const tail = this.removeTail();
                this.cache.delete(tail.key);
            }
            
            this.cache.set(key, newNode);
            this.addNode(newNode);
        }
    }
    
    // Добавить узел в начало списка
    addNode(node) {
        node.prev = this.head;
        node.next = this.head.next;
        
        this.head.next.prev = node;
        this.head.next = node;
    }
    
    // Удалить узел из списка
    removeNode(node) {
        const prev = node.prev;
        const newNext = node.next;
        
        prev.next = newNext;
        newNext.prev = prev;
    }
    
    // Переместить узел в начало
    moveToHead(node) {
        this.removeNode(node);
        this.addNode(node);
    }
    
    // Удалить последний узел
    removeTail() {
        const realTail = this.tail.prev;
        this.removeNode(realTail);
        return realTail;
    }
    
    // Получить все ключи в порядке использования (наиболее недавние первыми)
    getKeysInOrder() {
        const keys = [];
        let current = this.head.next;
        
        while (current !== this.tail) {
            keys.push(current.key);
            current = current.next;
        }
        
        return keys;
    }
}

// Пример использования
const lruCache = new LRUCache(2);
lruCache.put(1, 1);
lruCache.put(2, 2);
console.log(lruCache.get(1)); // 1
lruCache.put(3, 3); // Удаляет ключ 2
console.log(lruCache.get(2)); // -1 (не найден)
lruCache.put(4, 4); // Удаляет ключ 1
console.log(lruCache.get(1)); // -1 (не найден)
console.log(lruCache.get(3)); // 3
console.log(lruCache.get(4)); // 4
```

## Лучшие практики

### 1. Выбор правильного алгоритма

```javascript
// Временная сложность различных алгоритмов сортировки:
// - Bubble Sort: O(n²) - использовать только для очень маленьких массивов
// - Insertion Sort: O(n²) - эффективен для маленьких или почти отсортированных массивов
// - Quick Sort: O(n log n) средний случай - универсальный выбор
// - Merge Sort: O(n log n) - стабильная сортировка с предсказуемой производительностью
// - Heap Sort: O(n log n) - полезен, когда нужна гарантия O(n log n)

// Выбор алгоритма поиска:
// - Linear Search: O(n) - для неотсортированных данных или маленьких массивов
// - Binary Search: O(log n) - для отсортированных данных
```

### 2. Оптимизация использования памяти

```javascript
// Использование итеративных подходов вместо рекурсивных для больших данных
function factorialIterative(n) {
    let result = 1;
    for (let i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}

// Рекурсивный подход может вызвать переполнение стека для больших n
function factorialRecursive(n) {
    if (n <= 1) return 1;
    return n * factorialRecursive(n - 1);
}
```

## Безопасность

При реализации алгоритмов важно учитывать безопасность:

```javascript
// Безопасная реализация алгоритмов с проверкой ввода
class SecureAlgorithms {
    // Безопасная сортировка с проверкой входных данных
    static secureSort(arr, compareFn) {
        // Проверка на null/undefined
        if (!arr || !Array.isArray(arr)) {
            throw new Error('Input must be an array');
        }
        
        // Проверка на слишком большой массив
        if (arr.length > 1000000) {
            throw new Error('Array too large for sorting');
        }
        
        // Создание копии массива для предотвращения мутации
        const safeArr = [...arr];
        
        // Проверка и валидация функции сравнения
        if (compareFn && typeof compareFn !== 'function') {
            throw new Error('Compare function must be a function');
        }
        
        // Использование безопасного алгоритма сортировки
        return safeArr.sort(compareFn);
    }
    
    // Безопасный бинарный поиск
    static secureBinarySearch(sortedArr, target) {
        if (!sortedArr || !Array.isArray(sortedArr)) {
            throw new Error('Input must be an array');
        }
        
        let left = 0;
        let right = sortedArr.length - 1;
        
        // Дополнительная проверка, что массив отсортирован
        for (let i = 1; i < sortedArr.length; i++) {
            if (sortedArr[i] < sortedArr[i - 1]) {
                throw new Error('Array must be sorted');
            }
        }
        
        while (left <= right) {
            // Защита от переполнения при вычислении mid
            const mid = left + Math.floor((right - left) / 2);
            
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
}

// Использование
try {
    const sortedArray = [1, 3, 5, 7, 9, 11, 13];
    const index = SecureAlgorithms.secureBinarySearch(sortedArray, 7);
    console.log('Found at index:', index);
} catch (error) {
    console.error('Security error:', error.message);
}
```

## Теги

#javascript #algorithms #data-structures #sorting #searching #graphs #trees #dynamic-programming #greedy #strings