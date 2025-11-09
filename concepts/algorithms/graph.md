# Графовые алгоритмы

## Введение

Графовые алгоритмы — это набор методов для решения задач, связанных с графами. Графы представляют собой структуры данных, состоящие из вершин (узлов) и рёбер (соединений между узлами). Они широко применяются в компьютерных науках, математике, социальных сетях, транспортных системах и многих других областях.

## Основные понятия

- **Граф** — совокупность вершин и рёбер
- **Ориентированный граф** — рёбра имеют направление
- **Неориентированный граф** — рёбра не имеют направления
- **Взвешенный граф** — рёбра имеют числовые значения (веса)
- **Маршрут** — последовательность рёбер между вершинами
- **Цикл** — маршрут, начинающийся и заканчивающийся в одной вершине

## DFS (Depth-First Search) — Поиск в глубину

### Описание
DFS — это алгоритм обхода графа, который начинает с корневой вершины и исследует как можно дальше каждую ветвь перед возвратом.

### Реализация
```javascript
function dfs(graph, start, visited = new Set()) {
    visited.add(start);
    console.log(start); // Посещаем вершину
    
    for (const neighbor of graph[start]) {
        if (!visited.has(neighbor)) {
            dfs(graph, neighbor, visited);
        }
    }
}
```

### Сложности
- **Время**: O(V + E), где V — количество вершин, E — количество рёбер
- **Память**: O(V) для хранения посещённых вершин

### Применение
- Обнаружение циклов
- Топологическая сортировка
- Проверка связности компонентов

### Преимущества и недостатки
- **Преимущества**: Простота реализации, эффективен для поиска пути
- **Недостатки**: Может застрять в глубокой ветви, не гарантирует кратчайший путь

## BFS (Breadth-First Search) — Поиск в ширину

### Описание
BFS — это алгоритм обхода графа, который начинает с корневой вершины и исследует соседей текущего уровня перед переходом к следующему уровню.

### Реализация
```javascript
function bfs(graph, start) {
    const queue = [start];
    const visited = new Set([start]);
    
    while (queue.length > 0) {
        const current = queue.shift();
        console.log(current); // Посещаем вершину
        
        for (const neighbor of graph[current]) {
            if (!visited.has(neighbor)) {
                visited.add(neighbor);
                queue.push(neighbor);
            }
        }
    }
}
```

### Сложности
- **Время**: O(V + E)
- **Память**: O(V)

### Применение
- Нахождение кратчайшего пути в невзвешенном графе
- Проверка двудольности графа
- Поиск компонентов связности

### Преимущества и недостатки
- **Преимущества**: Гарантирует нахождение кратчайшего пути в невзвешенном графе
- **Недостатки**: Требует больше памяти по сравнению с DFS

## Алгоритм Дейкстры

### Описание
Алгоритм Дейкстры находит кратчайшие пути от одной вершины ко всем другим вершинам в взвешенном графе с неотрицательными весами.

### Реализация
```javascript
function dijkstra(graph, start) {
    const distances = {};
    const previous = {};
    const unvisited = new Set();
    
    // Инициализация
    for (const vertex in graph) {
        distances[vertex] = Infinity;
        unvisited.add(vertex);
    }
    distances[start] = 0;
    
    while (unvisited.size > 0) {
        // Находим вершину с минимальным расстоянием
        let current = null;
        let minDistance = Infinity;
        
        for (const vertex of unvisited) {
            if (distances[vertex] < minDistance) {
                minDistance = distances[vertex];
                current = vertex;
            }
        }
        
        if (distances[current] === Infinity) break; // Недостижимые вершины
        
        unvisited.delete(current);
        
        // Обновляем расстояния до соседей
        for (const [neighbor, weight] of Object.entries(graph[current])) {
            const distance = distances[current] + weight;
            if (distance < distances[neighbor]) {
                distances[neighbor] = distance;
                previous[neighbor] = current;
            }
        }
    }
    
    return { distances, previous };
}
```

### Сложности
- **Время**: O((V + E) log V) с использованием приоритетной очереди
- **Память**: O(V)

### Применение
- Маршрутизация в сетях
- Поиск кратчайших путей в картах
- Оптимизация логистики

### Преимущества и недостатки
- **Преимущества**: Гарантирует кратчайший путь в графе с неотрицательными весами
- **Недостатки**: Не работает с отрицательными весами

## Алгоритм Беллмана-Форда

### Описание
Алгоритм Беллмана-Форда находит кратчайшие пути от одной вершины ко всем другим вершинам в взвешенном графе, включая графы с отрицательными весами.

### Реализация
```javascript
function bellmanFord(graph, start) {
    const distances = {};
    const previous = {};
    
    // Инициализация
    for (const vertex in graph) {
        distances[vertex] = Infinity;
    }
    distances[start] = 0;
    
    // Релаксация рёбер |V| - 1 раз
    const vertices = Object.keys(graph);
    for (let i = 0; i < vertices.length - 1; i++) {
        for (const u of vertices) {
            for (const [v, weight] of Object.entries(graph[u])) {
                if (distances[u] + weight < distances[v]) {
                    distances[v] = distances[u] + weight;
                    previous[v] = u;
                }
            }
        }
    }
    
    // Проверка на отрицательный цикл
    for (const u of vertices) {
        for (const [v, weight] of Object.entries(graph[u])) {
            if (distances[u] + weight < distances[v]) {
                throw new Error("Обнаружен отрицательный цикл");
            }
        }
    }
    
    return { distances, previous };
}
```

### Сложности
- **Время**: O(VE)
- **Память**: O(V)

### Применение
- Обнаружение отрицательных циклов
- Финансовые расчёты с отрицательными весами
- Сравнение валютных курсов

### Преимущества и недостатки
- **Преимущества**: Работает с отрицательными весами, обнаруживает отрицательные циклы
- **Недостатки**: Медленнее, чем Дейкстра

## Алгоритм Флойда-Уоршелла

### Описание
Алгоритм Флойда-Уоршелла находит кратчайшие пути между всеми парами вершин в взвешенном графе.

### Реализация
```javascript
function floydWarshall(graph) {
    const vertices = Object.keys(graph);
    const n = vertices.length;
    const dist = {};
    
    // Инициализация матрицы расстояний
    for (const u of vertices) {
        dist[u] = {};
        for (const v of vertices) {
            if (u === v) {
                dist[u][v] = 0;
            } else if (graph[u] && graph[u][v] !== undefined) {
                dist[u][v] = graph[u][v];
            } else {
                dist[u][v] = Infinity;
            }
        }
    }
    
    // Основной цикл алгоритма
    for (const k of vertices) {
        for (const i of vertices) {
            for (const j of vertices) {
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    
    return dist;
}
```

### Сложности
- **Время**: O(V³)
- **Память**: O(V²)

### Применение
- Поиск всех кратчайших путей
- Анализ социальных сетей
- Оптимизация сетевых соединений

### Преимущества и недостатки
- **Преимущества**: Находит все кратчайшие пути за один запуск
- **Недостатки**: Высокая временная сложность

## Алгоритм Прима

### Описание
Алгоритм Прима находит минимальное остовное дерево (MST) для связного взвешенного неориентированного графа.

### Реализация
```javascript
function prim(graph) {
    const vertices = Object.keys(graph);
    const visited = new Set();
    const mst = [];
    const edges = [];
    
    // Начинаем с произвольной вершины
    const startVertex = vertices[0];
    visited.add(startVertex);
    
    // Добавляем рёбра из начальной вершины в очередь
    for (const [neighbor, weight] of Object.entries(graph[startVertex])) {
        edges.push({ from: startVertex, to: neighbor, weight });
    }
    
    // Сортируем рёбра по весу (в реальной реализации используйте приоритетную очередь)
    edges.sort((a, b) => a.weight - b.weight);
    
    while (visited.size < vertices.length && edges.length > 0) {
        const minEdge = edges.shift();
        
        if (visited.has(minEdge.to)) continue;
        
        visited.add(minEdge.to);
        mst.push(minEdge);
        
        // Добавляем рёбра из новой вершины
        for (const [neighbor, weight] of Object.entries(graph[minEdge.to])) {
            if (!visited.has(neighbor)) {
                edges.push({ from: minEdge.to, to: neighbor, weight });
            }
        }
        
        // Пересортировываем
        edges.sort((a, b) => a.weight - b.weight);
    }
    
    return mst;
}
```

### Сложности
- **Время**: O(E log V) с приоритетной очередью
- **Память**: O(V)

### Применение
- Создание эффективных сетей (электросети, дороги)
- Кластеризация данных
- Приближённые решения TSP

### Преимущества и недостатки
- **Преимущества**: Гарантирует минимальное остовное дерево
- **Недостатки**: Работает только с неориентированными графами

## Алгоритм Крускала

### Описание
Алгоритм Крускала также находит минимальное остовное дерево, но использует другой подход — сортировку рёбер по весу.

### Реализация
```javascript
class UnionFind {
    constructor(vertices) {
        this.parent = {};
        this.rank = {};
        
        for (const vertex of vertices) {
            this.parent[vertex] = vertex;
            this.rank[vertex] = 0;
        }
    }
    
    find(vertex) {
        if (this.parent[vertex] !== vertex) {
            this.parent[vertex] = this.find(this.parent[vertex]); // Путь сжатия
        }
        return this.parent[vertex];
    }
    
    union(v1, v2) {
        const root1 = this.find(v1);
        const root2 = this.find(v2);
        
        if (root1 === root2) return false;
        
        if (this.rank[root1] < this.rank[root2]) {
            this.parent[root1] = root2;
        } else if (this.rank[root1] > this.rank[root2]) {
            this.parent[root2] = root1;
        } else {
            this.parent[root2] = root1;
            this.rank[root1]++;
        }
        
        return true;
    }
}

function kruskal(graph) {
    // Получаем все рёбра
    const edges = [];
    const vertices = Object.keys(graph);
    
    for (const u of vertices) {
        for (const [v, weight] of Object.entries(graph[u])) {
            if (u < v) { // Избегаем дубликатов
                edges.push({ from: u, to: v, weight });
            }
        }
    }
    
    // Сортируем рёбра по весу
    edges.sort((a, b) => a.weight - b.weight);
    
    const uf = new UnionFind(vertices);
    const mst = [];
    
    for (const edge of edges) {
        if (uf.union(edge.from, edge.to)) {
            mst.push(edge);
            if (mst.length === vertices.length - 1) break; // MST готов
        }
    }
    
    return mst;
}
```

### Сложности
- **Время**: O(E log E)
- **Память**: O(V)

### Применение
- То же, что и у Прима
- Более эффективен для разреженных графов

### Преимущества и недостатки
- **Преимущества**: Простота реализации, эффективен для разреженных графов
- **Недостатки**: Требует структуру данных Union-Find

## Топологическая сортировка

### Описание
Топологическая сортировка упорядочивает вершины ориентированного ациклического графа так, что для каждого ребра (u, v), вершина u появляется перед v в упорядочении.

### Реализация
```javascript
function topologicalSort(graph) {
    const visited = new Set();
    const stack = [];
    
    function dfs(vertex) {
        visited.add(vertex);
        
        for (const neighbor of graph[vertex] || []) {
            if (!visited.has(neighbor)) {
                dfs(neighbor);
            }
        }
        
        stack.unshift(vertex); // Добавляем в начало стека
    }
    
    for (const vertex in graph) {
        if (!visited.has(vertex)) {
            dfs(vertex);
        }
    }
    
    return stack;
}
```

### Сложности
- **Время**: O(V + E)
- **Память**: O(V)

### Применение
- Планирование задач с зависимостями
- Компиляция программ
- Определение порядка выполнения курсов

### Преимущества и недостатки
- **Преимущества**: Полезна для задач с зависимостями
- **Недостатки**: Работает только для ациклических графов

## Сравнение алгоритмов

| Алгоритм | Время | Память | Отрицательные веса | Ориентированный | Применение |
|----------|-------|--------|-------------------|------------------|------------|
| DFS | O(V+E) | O(V) | Да | Да | Обход, циклы |
| BFS | O(V+E) | O(V) | Да | Да | Кратчайший путь (невзвешенный) |
| Дейкстра | O((V+E)logV) | O(V) | Нет | Да | Кратчайший путь (неотрицательный) |
| Беллман-Форд | O(VE) | O(V) | Да | Да | Кратчайший путь (с отрицательным) |
| Флойд-Уоршелл | O(V³) | O(V²) | Да | Да | Все кратчайшие пути |
| Прим | O(ElogV) | O(V) | Нет | Нет | MST |
| Крускал | O(ElogE) | O(V) | Нет | Нет | MST |
| Топологическая | O(V+E) | O(V) | Нет | Да | Зависимости |

## Практические применения

- **DFS**: Используется в решениях головоломок с одним решением (например, лабиринты)
- **BFS**: Применяется в системах GPS для поиска ближайших мест
- **Дейкстра**: Используется в протоколах маршрутизации OSPF
- **Беллман-Форд**: Применяется в протоколах маршрутизации, таких как BGP
- **Флойд-Уоршелл**: Используется в анализе социальных сетей
- **Прим/Крускал**: Применяются при проектировании сетей и кластеризации
- **Топологическая сортировка**: Используется в системах планирования и компиляции

## Связанные темы

- [[tree]] - деревья как частный случай графов
- [[sorting]] - сортировки, используемые в графовых алгоритмах
- [[data-structures]] - структуры данных для представления графов
- [[dynamic-programming]] - динамическое программирование, используемое в некоторых алгоритмах
- [[algorithm-complexity]] - анализ сложности алгоритмов

## Заключение

Графовые алгоритмы являются важной частью компьютерных наук и находят применение в самых разных областях. Выбор правильного алгоритма зависит от специфики задачи, структуры графа и требований к производительности. Понимание различий между алгоритмами позволяет эффективно решать задачи, связанные с графами.

#tags: #programming #algorithms #graph-theory #data-structures #complexity #computer-science