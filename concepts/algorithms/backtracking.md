---
aliases: ["Алгоритмы перебора с возвратом", "Рекурсивный перебор"]
tags: 
  - #алгоритмы 
  - #рекурсия 
  - #backtracking 
  - #оптимизация
  - #алгоритмы-поиска
---

# Алгоритмы перебора с возвратом (Backtracking)

**Backtracking** — это алгоритмическая парадигма, которая рекурсивно строит решение пошагово, отбрасывая решения, которые не удовлетворяют ограничениям задачи. Это улучшенная версия [[brute-force]] подхода, где на каждом шаге мы проверяем, приведет ли текущая конфигурация к допустимому решению.

## Основные понятия

Backtracking работает по принципу "попробуй и вернись", если текущий путь не приводит к решению. Он строит дерево решений, где каждая ветвь представляет собой возможный путь к решению. Если на каком-то этапе становится ясно, что текущий путь не ведет к решению, алгоритм возвращается назад и пробует другой путь.

> [!note] Примечание
> Backtracking особенно эффективен для задач, где решения строятся пошагово, и каждое частичное решение может быть проверено на допустимость.

## Backtracking vs Brute Force

| Критерий | Backtracking | Brute Force |
|----------|--------------|-------------|
| Подход | Умный перебор с отсечением | Полный перебор всех возможностей |
| Эффективность | Высокая за счет отсечения недопустимых решений | Низкая — проверяет все комбинации |
| Временная сложность | Может быть значительно улучшена | Обычно экспоненциальная |
| Применение | Задачи с ограничениями, оптимизационные | Простые задачи без ограничений |

## Дерево решений

В backtracking задача представляется в виде дерева, где:
- **Корень** — начальное состояние
- **Узлы** — частичные решения
- **Листья** — полные решения или тупиковые пути
- **Ветви** — возможные выборы на каждом шаге

```javascript
function backtracking(путь, возможные_выборы) {
    // Базовый случай: если путь является решением
    if (является_решением(путь)) {
        сохранить_решение(путь);
        return;
    }
    
    // Перебор всех возможных выборов
    for (выбор in возможные_выборы) {
        // Проверка ограничений
        if (!ограничение_соблюдено(путь, выбор)) continue;
        
        // Сделать выбор
        путь.добавить(выбор);
        
        // Рекурсивный вызов
        backtracking(путь, возможные_выборы);
        
        // Отменить выбор (возврат)
        путь.удалить_последний();
    }
}
```

## Общие паттерны backtracking

### 1. Subset Pattern
```javascript
function findSubsets(nums) {
    const result = [];
    
    function backtrack(start, current) {
        result.push([...current]);
        
        for (let i = start; i < nums.length; i++) {
            current.push(nums[i]);
            backtrack(i + 1, current);
            current.pop();
        }
    }
    
    backtrack(0, []);
    return result;
}
```
**Временная сложность:** O(2^n)  
**Пространственная сложность:** O(n)

### 2. Permutation Pattern
```javascript
function permute(nums) {
    const result = [];
    const used = new Array(nums.length).fill(false);
    
    function backtrack(current) {
        if (current.length === nums.length) {
            result.push([...current]);
            return;
        }
        
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            
            current.push(nums[i]);
            used[i] = true;
            backtrack(current);
            current.pop();
            used[i] = false;
        }
    }
    
    backtrack([]);
    return result;
}
```
**Временная сложность:** O(n!)  
**Пространственная сложность:** O(n)

## Классические задачи

### Задача N-ферзей
Размещение N ферзей на шахматной доске N×N так, чтобы они не били друг друга.

```javascript
function solveNQueens(n) {
    const result = [];
    const board = Array(n).fill().map(() => Array(n).fill('.'));
    
    function isValid(row, col) {
        // Проверка столбца
        for (let i = 0; i < row; i++) {
            if (board[i][col] === 'Q') return false;
        }
        
        // Проверка диагонали влево вверх
        for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
            if (board[i][j] === 'Q') return false;
        }
        
        // Проверка диагонали вправо вверх
        for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
            if (board[i][j] === 'Q') return false;
        }
        
        return true;
    }
    
    function backtrack(row) {
        if (row === n) {
            result.push(board.map(row => row.join('')));
            return;
        }
        
        for (let col = 0; col < n; col++) {
            if (isValid(row, col)) {
                board[row][col] = 'Q';
                backtrack(row + 1);
                board[row][col] = '.';
            }
        }
    }
    
    backtrack(0);
    return result;
}
```
**Временная сложность:** O(N!)  
**Пространственная сложность:** O(N²)

### Sudoku Solver
Решение судоку с использованием backtracking.

```javascript
function solveSudoku(board) {
    function isValid(board, row, col, num) {
        // Проверка строки
        for (let j = 0; j < 9; j++) {
            if (board[row][j] == num) return false;
        }
        
        // Проверка столбца
        for (let i = 0; i < 9; i++) {
            if (board[i][col] == num) return false;
        }
        
        // Проверка 3x3 блока
        const boxRow = Math.floor(row / 3) * 3;
        const boxCol = Math.floor(col / 3) * 3;
        
        for (let i = 0; i < 3; i++) {
            for (let j = 0; j < 3; j++) {
                if (board[boxRow + i][boxCol + j] == num) return false;
            }
        }
        
        return true;
    }
    
    function backtrack() {
        for (let i = 0; i < 9; i++) {
            for (let j = 0; j < 9; j++) {
                if (board[i][j] === '.') {
                    for (let num = 1; num <= 9; num++) {
                        if (isValid(board, i, j, num.toString())) {
                            board[i][j] = num.toString();
                            
                            if (backtrack()) return true;
                            
                            board[i][j] = '.'; // Возврат
                        }
                    }
                    return false; // Не удалось разместить число
                }
            }
        }
        return true; // Все ячейки заполнены
    }
    
    backtrack();
    return board;
}
```
**Временная сложность:** O(9^(n²))  
**Пространственная сложность:** O(1)

### Генерация подмножеств
См. пример выше в разделе "Subset Pattern".

### Решение лабиринта
```javascript
function solveMaze(maze) {
    const n = maze.length;
    const solution = Array(n).fill().map(() => Array(n).fill(0));
    
    function isValid(x, y) {
        return x >= 0 && x < n && y >= 0 && y < n && maze[x][y] === 1;
    }
    
    function solve(x, y) {
        if (x === n - 1 && y === n - 1) {
            solution[x][y] = 1;
            return true;
        }
        
        if (isValid(x, y)) {
            solution[x][y] = 1;
            
            if (solve(x + 1, y)) return true; // Вниз
            if (solve(x, y + 1)) return true; // Вправо
            
            solution[x][y] = 0; // Возврат
            return false;
        }
        
        return false;
    }
    
    if (solve(0, 0)) {
        return solution;
    } else {
        return "Нет решения";
    }
}
```
**Временная сложность:** O(2^(n²))  
**Пространственная сложность:** O(n²)

### Раскраска графа
```javascript
function graphColoring(graph, m) {
    const V = graph.length;
    const color = Array(V).fill(0);
    
    function isSafe(v, c) {
        for (let i = 0; i < V; i++) {
            if (graph[v][i] === 1 && color[i] === c) {
                return false;
            }
        }
        return true;
    }
    
    function solve(v) {
        if (v === V) return true;
        
        for (let c = 1; c <= m; c++) {
            if (isSafe(v, c)) {
                color[v] = c;
                
                if (solve(v + 1)) return true;
                
                color[v] = 0; // Возврат
            }
        }
        
        return false;
    }
    
    if (solve(0)) {
        return color;
    } else {
        return "Нет решения";
    }
}
```
**Временная сложность:** O(m^V)  
**Пространственная сложность:** O(V)

## Когда использовать backtracking

Backtracking наиболее эффективен в следующих случаях:
- Когда пространство решений велико, но может быть сокращено с помощью ограничений
- Для задач удовлетворения ограничений (CSP)
- Когда требуется найти все возможные решения
- В задачах оптимизации, где можно отсечь недопустимые решения заранее

## Преимущества и недостатки

### Преимущества
- **Эффективность:** Может быть значительно быстрее brute force за счет отсечения недопустимых решений
- **Гибкость:** Подходит для широкого класса задач
- **Полнота решения:** Находит все возможные решения

### Недостатки
- **Экспоненциальная сложность:** В худшем случае может иметь экспоненциальную временную сложность
- **Сложность реализации:** Требует аккуратного управления состоянием
- **Потенциальная неэффективность:** Для некоторых задач может быть неэффективным

## Техники отсечения (Pruning)

1. **Early termination** — прекращение поиска, когда становится ясно, что текущий путь не ведет к решению
2. **Constraint propagation** — обновление ограничений для уменьшения пространства поиска
3. **Symmetry breaking** — исключение симметричных решений
4. **Heuristic ordering** — упорядочивание выборов по эвристике для более быстрого нахождения решения

## Стратегии оптимизации

- **Мемоизация** — кэширование результатов для избежания повторных вычислений
- **Итеративное углубление** — постепенное увеличение глубины поиска
- **Оптимизация структур данных** — использование эффективных структур для проверки ограничений
- **Параллелизация** — распараллеливание поиска при наличии независимых ветвей

## Связь с другими концепциями

- [[recursion]] — основа backtracking алгоритмов
- [[dynamic-programming]] — альтернатива для некоторых задач
- [[graph-theory]] — используется в задачах на графах
- [[combinatorics]] — область применения комбинаторных задач
- [[optimization]] — область применения оптимизационных задач

## Заключение

Backtracking — мощная парадигма для решения задач, требующих перебора вариантов. Он особенно эффективен для задач удовлетворения ограничений, где можно отсечь недопустимые решения на ранних этапах. Хотя временная сложность может быть высокой в худшем случае, на практике алгоритм часто работает значительно быстрее благодаря отсечению недопустимых путей.
