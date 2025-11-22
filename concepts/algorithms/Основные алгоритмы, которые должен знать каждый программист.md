---
aliases: ["Алгоритмы для программистов", "Основные алгоритмы"]
tags: 
  - #algorithms
  - #programming
  - #data-structures
  - #complexity
---

# Основные алгоритмы, которые должен знать каждый программист

## Введение

Алгоритмы - это фундамент программирования. Знание основных алгоритмов позволяет эффективно решать задачи и оптимизировать приложения. В этом руководстве представлены наиболее важные алгоритмы с описанием, примерами кода и практическим применением.

## 1. Бинарный поиск (Binary Search)

### Описание
Алгоритм поиска значения в отсортированном массиве, который делит массив пополам на каждом шаге.

### Сложность
- **Время**: O(log n)
- **Память**: O(1) для итеративной, O(log n) для рекурсивной реализации

### Когда использовать
- Поиск элемента в отсортированном массиве
- Поиск позиции вставки
- Решение задач на поиск границ

### Пример кода
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```

### Практическое применение
- Поиск в базах данных
- Работа с отсортированными данными
- Алгоритмы оптимизации

См. также: [[concepts/data-structures/arrays]], [[concepts/search-algorithms]]

## 2. Быстрая сортировка (Quick Sort)

### Описание
Разделяй и властвуй алгоритм сортировки, который выбирает опорный элемент и разделяет массив на части.

### Сложность
- **Время**: O(n log n) средний случай, O(n²) худший случай
- **Память**: O(log n) для рекурсивного стека

### Когда использовать
- Когда нужна эффективная сортировка в среднем случае
- Когда не требуется стабильная сортировка
- Для больших наборов данных

### Пример кода
```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    
    return quick_sort(left) + middle + quick_sort(right)
```

### Практическое применение
- Сортировка больших массивов
- Внутренние реализации в языках программирования
- Алгоритмы поиска k-го по величине элемента

См. также: [[concepts/sorting-algorithms]], [[concepts/divide-and-conquer]]

## 3. Сортировка слиянием (Merge Sort)

### Описание
Стабильный алгоритм сортировки, использующий принцип "разделяй и властвуй".

### Сложность
- **Время**: O(n log n) во всех случаях
- **Память**: O(n)

### Когда использовать
- Когда нужна гарантия O(n log n)
- Для стабильной сортировки
- При работе с связными списками

### Пример кода
```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

### Практическое применение
- Сортировка внешних файлов
- Стабильная сортировка
- В алгоритмах подсчета инверсий

См. также: [[concepts/sorting-algorithms]], [[concepts/divide-and-conquer]]

## 4. Поиск в ширину (BFS)

### Описание
Алгоритм обхода графа, исследующий узлы по уровням от начальной вершины.

### Сложность
- **Время**: O(V + E), где V - вершины, E - рёбра
- **Память**: O(V)

### Когда использовать
- Поиск кратчайшего пути в невзвешенном графе
- Обход графа по уровням
- Поиск компонент связности

### Пример кода
```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue = deque([start])
    result = []
    
    while queue:
        node = queue.popleft()
        if node not in visited:
            visited.add(node)
            result.append(node)
            queue.extend(neighbor for neighbor in graph[node] if neighbor not in visited)
    
    return result
```

### Практическое применение
- Поиск в социальных сетях
- Алгоритмы маршрутизации
- Поиск ближайших соседей

См. также: [[concepts/graphs]], [[concepts/tree-traversal]]

## 5. Поиск в глубину (DFS)

### Описание
Алгоритм обхода графа, исследующий узлы настолько глубоко, насколько возможно.

### Сложность
- **Время**: O(V + E)
- **Память**: O(V) для рекурсивного стека

### Когда использовать
- Поиск пути в лабиринте
- Топологическая сортировка
- Обнаружение циклов в графе

### Пример кода
```python
def dfs_recursive(graph, node, visited=None):
    if visited is None:
        visited = set()
    
    visited.add(node)
    result = [node]
    
    for neighbor in graph[node]:
        if neighbor not in visited:
            result.extend(dfs_recursive(graph, neighbor, visited))
    
    return result

def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    result = []
    
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            result.append(node)
            stack.extend(neighbor for neighbor in graph[node] if neighbor not in visited)
    
    return result
```

### Практическое применение
- Игровые алгоритмы (AI)
- Поиск решений головоломок
- Системы рекомендаций

См. также: [[concepts/graphs]], [[concepts/tree-traversal]]

## 6. Алгоритм Дейкстры (Dijkstra's Algorithm)

### Описание
Алгоритм нахождения кратчайших путей от одной вершины ко всем другим в взвешенном графе с неотрицательными весами.

### Сложность
- **Время**: O((V + E) log V) с кучей
- **Память**: O(V)

### Когда использовать
- Нахождение кратчайшего пути в картах
- Сетевые маршруты
- Когда веса неотрицательные

### Пример кода
```python
import heapq

def dijkstra(graph, start):
    distances = {node: float('infinity') for node in graph}
    distances[start] = 0
    pq = [(0, start)]
    previous = {}
    
    while pq:
        current_distance, current_node = heapq.heappop(pq)
        
        if current_distance > distances[current_node]:
            continue
        
        for neighbor, weight in graph[current_node].items():
            distance = current_distance + weight
            
            if distance < distances[neighbor]:
                distances[neighbor] = distance
                previous[neighbor] = current_node
                heapq.heappush(pq, (distance, neighbor))
    
    return distances, previous
```

### Практическое применение
- GPS-навигация
- Сетевые протоколы
- Планирование маршрутов

См. также: [[concepts/graphs]], [[concepts/shortest-path]]

## 7. Динамическое программирование (Dynamic Programming)

### Описание
Метод решения задач, при котором сложная задача разбивается на более простые подзадачи.

### Сложность
- Зависит от конкретной задачи
- Обычно улучшает экспоненциальную сложность до полиномиальной

### Когда использовать
- Когда задача имеет оптимальную подструктуру
- При наличии перекрывающихся подзадач
- Для оптимизации рекурсивных решений

### Пример кода
```python
def fibonacci_dp(n):
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    
    return dp[n]

def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0 for _ in range(capacity + 1)] for _ in range(n + 1)]
    
    for i in range(1, n + 1):
        for w in range(1, capacity + 1):
            if weights[i - 1] <= w:
                dp[i][w] = max(
                    values[i - 1] + dp[i - 1][w - weights[i - 1]],
                    dp[i - 1][w]
                )
            else:
                dp[i][w] = dp[i - 1][w]
    
    return dp[n][capacity]
```

### Практическое применение
- Финансовое планирование
- Оптимизация ресурсов
- Биоинформатика

См. также: [[concepts/recursion]], [[concepts/optimization]]

## 8. Операции с хеш-таблицами (Hash Tables)

### Описание
Структура данных, реализующая ассоциативный массив с доступом за O(1) в среднем случае.

### Сложность
- **Поиск/Вставка/Удаление**: O(1) средний случай, O(n) худший случай
- **Память**: O(n)

### Когда использовать
- Быстрый поиск
- Удаление дубликатов
- Подсчет частоты элементов

### Пример кода
```python
# Встроенные хеш-таблицы в Python (словари)
hash_table = {}
hash_table['key'] = 'value'
value = hash_table.get('key', 'default')

# Кастомная реализация (упрощённая)
class HashTable:
    def __init__(self, size=10):
        self.size = size
        self.table = [[] for _ in range(self.size)]
    
    def _hash(self, key):
        return hash(key) % self.size
    
    def put(self, key, value):
        index = self._hash(key)
        for pair in self.table[index]:
            if pair[0] == key:
                pair[1] = value
                return
        self.table[index].append([key, value])
    
    def get(self, key):
        index = self._hash(key)
        for pair in self.table[index]:
            if pair[0] == key:
                return pair[1]
        return None
```

### Практическое применение
- Кеширование данных
- Реализация баз данных
- Компиляторы (таблицы символов)

См. также: [[concepts/data-structures/hash-tables]], [[concepts/collections]]

## 9. Обходы деревьев (Tree Traversals)

### Описание
Алгоритмы посещения всех узлов дерева в определенном порядке.

### Сложность
- **Время**: O(n), где n - количество узлов
- **Память**: O(h), где h - высота дерева

### Типы обходов
1. **Прямой (Pre-order)**: корень → левое поддерево → правое поддерево
2. **Симметричный (In-order)**: левое поддерево → корень → правое поддерево
3. **Обратный (Post-order)**: левое поддерево → правое поддерево → корень

### Пример кода
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def preorder_traversal(root):
    if not root:
        return []
    return [root.val] + preorder_traversal(root.left) + preorder_traversal(root.right)

def inorder_traversal(root):
    if not root:
        return []
    return inorder_traversal(root.left) + [root.val] + inorder_traversal(root.right)

def postorder_traversal(root):
    if not root:
        return []
    return postorder_traversal(root.left) + postorder_traversal(root.right) + [root.val]

# Итеративный обход
def inorder_iterative(root):
    result, stack = [], []
    current = root
    
    while stack or current:
        while current:
            stack.append(current)
            current = current.left
        current = stack.pop()
        result.append(current.val)
        current = current.right
    
    return result
```

### Практическое применение
- Синтаксические деревья
- Файловые системы
- Иерархические структуры

См. также: [[concepts/data-structures/trees]], [[concepts/recursion]]

## Заключение

Эти алгоритмы составляют основу эффективного программирования. Понимание их работы, сложности и применения позволяет:
- Писать более эффективный код
- Лучше решать алгоритмические задачи
- Готовиться к техническим интервью
- Делать осознанный выбор структур данных и алгоритмов

Для глубокого понимания важно не только знать реализации, но и понимать, когда и почему использовать каждый алгоритм.

> [!tip] Совет
> Практика - ключ к освоению алгоритмов. Решайте задачи на платформах вроде LeetCode, HackerRank или Codeforces, чтобы закрепить знания.

> [!warning] Важно
> Не пытайтесь запомнить все алгоритмы сразу. Начните с базовых и постепенно расширяйте знания, практикуясь на реальных задачах.

См. также: [[concepts/computational-complexity]], [[concepts/data-structures/overview]]