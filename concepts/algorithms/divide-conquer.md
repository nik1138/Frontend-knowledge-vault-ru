---
aliases: ["Разделяй и властвуй", "Алгоритмы разбиения"]
tags: 
  - #algorithms
  - #divide-and-conquer
  - #sorting
  - #searching
---

# Алгоритмы "Разделяй и властвуй" (Divide and Conquer)

Алгоритмы "разделяй и властвуй" — это парадигма проектирования алгоритмов, при которой проблема рекурсивно разбивается на две или более подпроблемы того же или связанного типа, решаются эти подпроблемы, а затем их решения объединяются для получения решения исходной проблемы.

## Основные этапы

1. **Разделение (Divide)** — разбиение задачи на подзадачи
2. **Покорение (Conquer)** — решение подзадач рекурсивно
3. **Объединение (Combine)** — объединение решений подзадач

## Когда использовать

- Задача может быть разбита на независимые подзадачи
- Подзадачи аналогичны исходной задаче
- Объединение решений подзадач эффективно
- Рекурсивная структура задачи очевидна

## Преимущества и недостатки

> [!tip] Преимущества
> - Часто приводит к эффективным алгоритмам
> - Удобен для параллелизации
> - Четкая рекурсивная структура

> [!warning] Недостатки
> - Может использовать дополнительную память (стек рекурсии)
> - Не всегда оптимален для всех задач
> - Может быть сложнее реализовать, чем итеративные подходы

## Паттерны распознавания задач

- Массивы/списки, которые можно разделить
- Поиск/сортировка задач
- Задачи на максимум/минимум
- Геометрические задачи
- Рекурсивные структуры данных

## Реализации алгоритмов

### Сортировка слиянием (Merge Sort)

```javascript
function mergeSort(arr) {
    if (arr.length <= 1) return arr;
    
    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));
    
    return merge(left, right);
}

function merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < left.length && j < right.length) {
        if (left[i] <= right[j]) {
            result.push(left[i++]);
        } else {
            result.push(right[j++]);
        }
    }
    
    return result.concat(left.slice(i), right.slice(j));
}
```

**Сложность**: O(n log n) / O(n log n) / O(n log n)  
**Память**: O(n)

### Быстрая сортировка (Quick Sort)

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
    if (low < high) {
        const pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
    return arr;
}

function partition(arr, low, high) {
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
```

**Сложность**: O(n log n) / O(n²) / O(n log n)  
**Память**: O(log n)

### Бинарный поиск (Binary Search)

```javascript
function binarySearch(arr, target) {
    let left = 0, right = arr.length - 1;
    
    while (left <= right) {
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

**Сложность**: O(log n)  
**Память**: O(1)

### Максимальный подмассив (Kadane's Algorithm)

> [!note] Примечание
> Хотя классический алгоритм Кадане не является "разделяй и властвуй", существует версия с этим подходом

```javascript
function maxSubArrayDC(nums, left = 0, right = nums.length - 1) {
    if (left === right) return nums[left];
    
    const mid = Math.floor((left + right) / 2);
    
    const leftSum = maxSubArrayDC(nums, left, mid);
    const rightSum = maxSubArrayDC(nums, mid + 1, right);
    const crossSum = maxCrossingSum(nums, left, mid, right);
    
    return Math.max(leftSum, rightSum, crossSum);
}

function maxCrossingSum(nums, left, mid, right) {
    let leftSum = -Infinity, sum = 0;
    for (let i = mid; i >= left; i--) {
        sum += nums[i];
        leftSum = Math.max(leftSum, sum);
    }
    
    let rightSum = -Infinity;
    sum = 0;
    for (let i = mid + 1; i <= right; i++) {
        sum += nums[i];
        rightSum = Math.max(rightSum, sum);
    }
    
    return leftSum + rightSum;
}
```

**Сложность**: O(n log n)  
**Память**: O(log n)

### Ближайшая пара точек

```javascript
function closestPair(points) {
    points.sort((a, b) => a.x - b.x);
    return closestPairRec(points, 0, points.length - 1);
}

function closestPairRec(points, low, high) {
    if (high - low <= 2) {
        // Базовый случай: <= 3 точки
        return bruteForce(points, low, high);
    }
    
    const mid = Math.floor((low + high) / 2);
    const midPoint = points[mid];
    
    const dl = closestPairRec(points, low, mid);
    const dr = closestPairRec(points, mid + 1, high);
    
    let d = Math.min(dl, dr);
    
    // Проверить точки в полосе
    const strip = [];
    for (const point of points) {
        if (Math.abs(point.x - midPoint.x) < d) {
            strip.push(point);
        }
    }
    
    return Math.min(d, stripClosest(strip, d));
}

function bruteForce(points, low, high) {
    let min = Infinity;
    for (let i = low; i <= high; i++) {
        for (let j = i + 1; j <= high; j++) {
            min = Math.min(min, dist(points[i], points[j]));
        }
    }
    return min;
}

function dist(p1, p2) {
    return Math.sqrt(Math.pow(p1.x - p2.x, 2) + Math.pow(p1.y - p2.y, 2));
}
```

**Сложность**: O(n log n)  
**Память**: O(n)

### Быстрое возведение в степень

```javascript
function fastPower(base, exp) {
    if (exp === 0) return 1;
    if (exp % 2 === 0) {
        const half = fastPower(base, exp / 2);
        return half * half;
    } else {
        return base * fastPower(base, exp - 1);
    }
}
```

**Сложность**: O(log n)  
**Память**: O(log n)

### Умножение матриц Страссена

> [!note] Примечание
> Алгоритм Страссена умножает матрицы за O(n^log₂7) ≈ O(n^2.807)

## Рекуррентные соотношения

Общая форма: T(n) = aT(n/b) + f(n)

- a — количество подзадач
- n/b — размер подзадач
- f(n) — стоимость деления/объединения

**Мастер-теорема**:
- Если f(n) = O(n^log_b(a) - ε), то T(n) = Θ(n^log_b(a))
- Если f(n) = Θ(n^log_b(a)), то T(n) = Θ(n^log_b(a) * log n)
- Если f(n) = Ω(n^log_b(a) + ε), то T(n) = Θ(f(n))

## Связи с другими концепциями

- [[dynamic-programming]] — обе парадигмы решают задачи, разбивая их на подзадачи
- [[recursion]] — основа рекурсивной структуры
- [[sorting-algorithms]] — сортировка слиянием и быстрая сортировка
- [[searching-algorithms]] — бинарный поиск
- [[time-complexity]] — анализ сложности алгоритмов
- [[matrix-operations]] — умножение матриц Страссена

## Заключение

Алгоритмы "разделяй и властвуй" являются мощной парадигмой для решения многих вычислительных задач. Они обеспечивают эффективные решения для сортировки, поиска, вычислительной геометрии и других областей. Понимание этой парадигмы критично для разработки эффективных алгоритмов.