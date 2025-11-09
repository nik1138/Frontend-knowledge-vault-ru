---
aliases: ["Комбинаторный поиск", "Генерация комбинаций", "Генерация перестановок"]
tags: ["#algorithms", "#combinatorics", "#search", "#recursion", "#backtracking"]
---

# Комбинаторный поиск

## Введение

**Комбинаторный поиск** — это область алгоритмов, занимающаяся систематическим перебором всех возможных комбинаторных объектов, удовлетворяющих заданным условиям. Он используется для решения задач, где требуется найти все возможные комбинации, перестановки, подмножества или другие структуры в конечном множестве.

## Основные понятия

Комбинаторный поиск включает в себя генерацию различных типов **комбинаторных объектов**:

- [[concepts/combinatorics/combinations|Комбинации]] — выбор $k$ элементов из $n$ без учета порядка
- [[concepts/combinatorics/permutations|Перестановки]] — упорядоченные последовательности элементов
- [[concepts/combinatorics/subsets|Подмножества]] — все возможные подмножества множества
- [[concepts/combinatorics/partitions|Разбиения]] — разбиения числа или множества на части

## Типы комбинаторных объектов

### Комбинации

Комбинации — это выбор $k$ элементов из $n$ элементов без учета порядка. Количество комбинаций вычисляется по формуле:

$$ C(n, k) = \frac{n!}{k!(n-k)!} $$

### Перестановки

Перестановки — это упорядоченные последовательности всех или части элементов. Количество перестановок $n$ элементов:

$$ P(n) = n! $$

### Подмножества

Подмножество — это любое подмножество элементов из заданного множества. Количество подмножеств $n$-элементного множества:

$$ 2^n $$

### Разбиения

Разбиение — это способ разделения множества на непересекающиеся подмножества. Разбиения могут быть разными в зависимости от критериев.

## Алгоритмы генерации

### Генерация комбинаций

```python
def combinations(arr, k):
    """
    Генерация всех комбинаций длины k из массива arr
    """
    def backtrack(start, current):
        if len(current) == k:
            result.append(current[:])
            return
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current)
            current.pop()
    
    result = []
    backtrack(0, [])
    return result

# Пример
arr = [1, 2, 3, 4]
k = 2
print(combinations(arr, k))  # [[1, 2], [1, 3], [1, 4], [2, 3], [2, 4], [3, 4]]
```

### Генерация перестановок

```python
def permutations(arr):
    """
    Генерация всех перестановок массива arr
    """
    def backtrack(current, used):
        if len(current) == len(arr):
            result.append(current[:])
            return
        for i in range(len(arr)):
            if not used[i]:
                current.append(arr[i])
                used[i] = True
                backtrack(current, used)
                current.pop()
                used[i] = False
    
    result = []
    used = [False] * len(arr)
    backtrack([], used)
    return result

# Пример
arr = [1, 2, 3]
print(permutations(arr))  # [[1, 2, 3], [1, 3, 2], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
```

### Генерация подмножеств

```python
def subsets(arr):
    """
    Генерация всех подмножеств массива arr
    """
    def backtrack(start, current):
        result.append(current[:])
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current)
            current.pop()
    
    result = []
    backtrack(0, [])
    return result

# Пример
arr = [1, 2, 3]
print(subsets(arr))  # [[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
```

## Оптимизация и техники

### Отсечение (Pruning)

При поиске можно отсекать ветви, которые заведомо не приведут к решению:

```python
def combinations_with_pruning(arr, k, target_sum):
    """
    Генерация комбинаций с отсечением по сумме
    """
    def backtrack(start, current, current_sum):
        if len(current) == k:
            if current_sum == target_sum:
                result.append(current[:])
            return
        # Отсечение: если текущая сумма уже превышает целевую
        if current_sum > target_sum:
            return
        for i in range(start, len(arr)):
            current.append(arr[i])
            backtrack(i + 1, current, current_sum + arr[i])
            current.pop()
    
    result = []
    backtrack(0, [], 0)
    return result
```

### Использование битовых масок

Для генерации подмножеств можно использовать битовые маски:

```python
def subsets_with_bitmask(arr):
    """
    Генерация подмножеств с использованием битовых масок
    """
    n = len(arr)
    result = []
    for mask in range(1 << n):  # 2^n возможных масок
        subset = []
        for i in range(n):
            if mask & (1 << i):  # Если i-й бит установлен
                subset.append(arr[i])
        result.append(subset)
    return result
```

## Связь с другими алгоритмами

Комбинаторный поиск тесно связан с другими алгоритмами:

- [[concepts/algorithms/backtracking|Возврат по следу (backtracking)]] — основной подход для реализации комбинаторного поиска
- [[concepts/algorithms/dynamic-programming|Динамическое программирование]] — для подсчета количества комбинаторных объектов
- [[concepts/algorithms/graph-theory|Теория графов]] — для представления комбинаторных структур как графов
- [[concepts/algorithms/branch-and-bound|Метод ветвей и границ]] — для оптимизации перебора

## Применение

Комбинаторный поиск применяется в:

- [[concepts/algorithms/cryptography|Криптографии]] — для перебора ключей
- [[concepts/algorithms/game-theory|Теории игр]] — для поиска оптимальных стратегий
- [[concepts/algorithms/scheduling|Планировании]] — для поиска оптимальных расписаний
- [[concepts/algorithms/genetic-algorithms|Генетических алгоритмах]] — для генерации популяций

## Заключение

Комбинаторный поиск — мощный инструмент для решения задач, требующих систематического перебора комбинаторных объектов. Понимание базовых концепций и алгоритмов позволяет эффективно решать широкий класс задач в области алгоритмов и дискретной математики.

> [!tip] 
> Для оптимизации комбинаторного поиска всегда старайтесь использовать отсечение (pruning), чтобы избежать генерации ненужных комбинаций.

> [!warning] 
> Избегайте генерации всех комбинаций, если задача требует только подсчета их количества — используйте [[concepts/algorithms/dynamic-programming|динамическое программирование]] вместо этого.