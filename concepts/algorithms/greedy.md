---
aliases: ["Жадные алгоритмы", "Алгоритмы оптимизации"]
tags: 
  - #algorithms 
  - #optimization 
  - #programming 
  - #data-structures
---

# Жадные алгоритмы (Greedy Algorithms)

## Введение

Жадные алгоритмы — это класс алгоритмов оптимизации, которые делают локально оптимальный выбор на каждом шаге в надежде получить глобально оптимальное решение. Они строят решение пошагово, выбирая на каждом шаге то, что кажется лучшим в данный момент.

> [!warning] Важно
> Жадные алгоритмы не всегда приводят к глобально оптимальному решению. Их применимость зависит от свойств задачи.

## Основные принципы

- **Локальная оптимальность**: На каждом шаге выбирается локально оптимальное решение
- **Отсутствие возврата**: Принятое решение не пересматривается
- **Простота реализации**: Часто проще, чем динамическое программирование

## Когда использовать жадный подход

- Задача обладает **свойством жадного выбора** (локальный оптимум ведет к глобальному)
- Задача имеет **свойство оптимальной подструктуры**
- Подходит для задач оптимизации, где локальные решения ведут к глобальному

## Примеры задач и алгоритмов

### 1. Задача выбора активностей (Activity Selection Problem)

**Описание**: Выбрать максимальное количество непересекающихся по времени активностей.

**Алгоритм**:
1. Отсортировать активности по времени окончания
2. Выбрать первую активность
3. Для каждой следующей активности: если ее время начала >= времени окончания последней выбранной, добавить ее

```python
def activity_selection(activities):
    # activities = [(start, end), ...]
    activities.sort(key=lambda x: x[1])  # Сортировка по времени окончания
    
    selected = [activities[0]]
    last_end = activities[0][1]
    
    for i in range(1, len(activities)):
        if activities[i][0] >= last_end:
            selected.append(activities[i])
            last_end = activities[i][1]
    
    return selected

# Время: O(n log n), Пространство: O(1)
```

### 2. Дробный рюкзак (Fractional Knapsack Problem)

**Описание**: В отличие от 0/1 рюкзака, можно брать доли предметов.

**Алгоритм**:
1. Отсортировать предметы по удельной стоимости (cost/weight)
2. Брать предметы в порядке убывания удельной стоимости до заполнения рюкзака

```python
def fractional_knapsack(capacity, items):
    # items = [(weight, value), ...]
    items.sort(key=lambda x: x[1]/x[0], reverse=True)  # По удельной стоимости
    
    total_value = 0
    for weight, value in items:
        if capacity >= weight:
            total_value += value
            capacity -= weight
        else:
            total_value += value * (capacity / weight)
            break
    
    return total_value

# Время: O(n log n), Пространство: O(1)
```

### 3. Кодирование Хаффмана (Huffman Coding)

**Описание**: Алгоритм сжатия данных, использующий дерево Хаффмана.

**Алгоритм**:
1. Создать листья для каждого символа с их частотами
2. Повторно объединять два узла с наименьшими частотами
3. Результат — бинарное дерево кодирования

```python
import heapq
from collections import Counter

class Node:
    def __init__(self, char, freq, left=None, right=None):
        self.char = char
        self.freq = freq
        self.left = left
        self.right = right

    def __lt__(self, other):
        return self.freq < other.freq

def huffman_coding(text):
    freq = Counter(text)
    heap = [Node(char, freq) for char, freq in freq.items()]
    heapq.heapify(heap)
    
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        merged = Node(None, left.freq + right.freq, left, right)
        heapq.heappush(heap, merged)
    
    root = heap[0]
    
    def generate_codes(node, prefix="", codebook={}):
        if node.char is not None:
            codebook[node.char] = prefix
        else:
            generate_codes(node.left, prefix + "0", codebook)
            generate_codes(node.right, prefix + "1", codebook)
        return codebook
    
    return generate_codes(root)

# Время: O(n log n), Пространство: O(n)
```

### 4. Минимальное остовное дерево

#### Алгоритм Прима

**Описание**: Находит минимальное остовное дерево взвешенного неориентированного графа.

```python
def prim_mst(graph, start):
    # graph = {node: [(neighbor, weight), ...]}
    mst = []
    visited = set([start])
    edges = [(weight, start, neighbor) 
             for neighbor, weight in graph[start]]
    heapq.heapify(edges)
    
    while edges:
        weight, u, v = heapq.heappop(edges)
        if v not in visited:
            visited.add(v)
            mst.append((u, v, weight))
            
            for neighbor, w in graph[v]:
                if neighbor not in visited:
                    heapq.heappush(edges, (w, v, neighbor))
    
    return mst

# Время: O(E log V), Пространство: O(V)
```

#### Алгоритм Крускала

**Описание**: Также находит минимальное остовное дерево.

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True

def kruskal_mst(n, edges):
    # edges = [(weight, u, v), ...]
    edges.sort()
    uf = UnionFind(n)
    mst = []
    
    for weight, u, v in edges:
        if uf.union(u, v):
            mst.append((u, v, weight))
            if len(mst) == n - 1:
                break
    
    return mst

# Время: O(E log E), Пространство: O(V)
```

### 5. Алгоритм Дейкстры

**Описание**: Находит кратчайшие пути от одной вершины ко всем другим в графе с неотрицательными весами.

```python
def dijkstra(graph, start):
    # graph = {node: [(neighbor, weight), ...]}
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    pq = [(0, start)]
    
    while pq:
        current_dist, u = heapq.heappop(pq)
        
        if current_dist > distances[u]:
            continue
            
        for neighbor, weight in graph[u]:
            new_dist = distances[u] + weight
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                heapq.heappush(pq, (new_dist, neighbor))
    
    return distances

# Время: O((V + E) log V), Пространство: O(V)
```

> [!tip] Заметка
> Алгоритм Дейкстры формально является жадным алгоритмом, так как на каждом шаге выбирает ближайшую вершину.

### 6. Задача о размене монет (Coin Change)

**Описание**: Найти минимальное количество монет для суммы (работает не всегда).

```python
def greedy_coin_change(coins, amount):
    coins.sort(reverse=True)
    result = []
    
    for coin in coins:
        while amount >= coin:
            result.append(coin)
            amount -= coin
    
    return result if amount == 0 else None

# Время: O(amount), Пространство: O(1) (не всегда оптимально)
```

### 7. Планирование задач (Job Scheduling)

**Описание**: Задача с приоритетами/доходами и временем выполнения.

```python
def job_scheduling(jobs):
    # jobs = [(deadline, profit), ...]
    jobs.sort(key=lambda x: x[1], reverse=True)  # По убыванию прибыли
    result = []
    time_slots = set(range(len(jobs), 0, -1))  # Доступные временные слоты
    
    for deadline, profit in jobs:
        # Найти ближайший доступный слот до дедлайна
        slot = max([t for t in time_slots if t <= deadline], default=0)
        if slot > 0:
            result.append((deadline, profit))
            time_slots.remove(slot)
    
    return result

# Время: O(n^2), Пространство: O(n)
```

## Доказательства корректности

Для доказательства корректности жадных алгоритмов обычно используются:
- **Метод обмена**: Показать, что любое оптимальное решение можно модифицировать, чтобы включить жадный выбор
- **Метод сохранения свойства**: Показать, что жадный выбор сохраняет свойство оптимальной подструктуры

## Преимущества и недостатки

> [!success] Преимущества
> - Простота реализации
> - Хорошая производительность
> - Интуитивно понятны

> [!danger] Недостатки
> - Не гарантируют глобального оптимума
> - Требуют доказательства корректности
> - Ограниченная применимость

## Паттерны распознавания задач

- **Оптимизация с локальными выборами**
- **Задачи на упорядочивание/сортировку**
- **Задачи на покрытие/выбор подмножества**
- **Оптимизация с ограничениями**

## Связь с другими концепциями

- [[Dynamic Programming]] - Альтернатива, когда жадный подход не работает
- [[Graph Theory]] - Многие жадные алгоритмы работают с графами
- [[Data Structures]] - Часто используют кучи, множества и другие структуры

## Заключение

Жадные алгоритмы — мощный инструмент в арсенале программиста, особенно когда задача обладает свойствами, позволяющими использовать локальные оптимальные решения для получения глобального оптимума. Важно уметь распознавать такие задачи и доказывать корректность решений.