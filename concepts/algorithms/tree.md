---
aliases: ["Деревья", "Алгоритмы деревьев", "Структуры данных деревья"]
tags: 
  - #algorithms
  - #data-structures
  - #trees
  - #programming
  - #computer-science
---

# Алгоритмы деревьев

## Введение

Деревья - это иерархическая структура данных, состоящая из узлов, где каждый узел может иметь дочерние узлы. В отличие от линейных структур данных, таких как массивы или списки, деревья позволяют эффективно представлять иерархические отношения.

## Типы деревьев

### Бинарное дерево
Каждый узел имеет не более двух дочерних узлов: левого и правого.

```javascript
class TreeNode {
    constructor(val, left = null, right = null) {
        this.val = val;
        this.left = left;
        this.right = right;
    }
}
```

### Полное бинарное дерево
Все уровни, кроме, возможно, последнего, полностью заполнены, а все узлы на последнем уровне максимально левее.

### Сбалансированное дерево
Высота левого и правого поддеревьев любого узла отличается не более чем на 1.

### B-дерево
Многоуровневое дерево, часто используемое в базах данных и файловых системах.

## Обход деревьев

### Обход в глубину (DFS)

#### Прямой обход (Pre-order)
Сначала посещаем текущий узел, затем левое поддерево, затем правое поддерево.

```javascript
function preOrder(root) {
    if (!root) return;
    console.log(root.val); // Посещение узла
    preOrder(root.left);
    preOrder(root.right);
}
```

#### Центрированный обход (In-order)
Сначала левое поддерево, затем текущий узел, затем правое поддерево.

```javascript
function inOrder(root) {
    if (!root) return;
    inOrder(root.left);
    console.log(root.val); // Посещение узла
    inOrder(root.right);
}
```

#### Обратный обход (Post-order)
Сначала левое поддерево, затем правое поддерево, затем текущий узел.

```javascript
function postOrder(root) {
    if (!root) return;
    postOrder(root.left);
    postOrder(root.right);
    console.log(root.val); // Посещение узла
}
```

### Обход в ширину (BFS)

```javascript
function bfs(root) {
    if (!root) return [];
    
    const queue = [root];
    const result = [];
    
    while (queue.length > 0) {
        const node = queue.shift();
        result.push(node.val);
        
        if (node.left) queue.push(node.left);
        if (node.right) queue.push(node.right);
    }
    
    return result;
}
```

## Бинарное дерево поиска (BST)

BST - это бинарное дерево, в котором для каждого узла все значения в левом поддереве меньше значения узла, а все значения в правом поддереве больше.

```javascript
class BST {
    constructor() {
        this.root = null;
    }
    
    insert(val) {
        this.root = this._insertNode(this.root, val);
    }
    
    _insertNode(node, val) {
        if (!node) return new TreeNode(val);
        
        if (val < node.val) {
            node.left = this._insertNode(node.left, val);
        } else if (val > node.val) {
            node.right = this._insertNode(node.right, val);
        }
        
        return node;
    }
    
    search(val) {
        return this._searchNode(this.root, val);
    }
    
    _searchNode(node, val) {
        if (!node || node.val === val) return node;
        
        if (val < node.val) {
            return this._searchNode(node.left, val);
        } else {
            return this._searchNode(node.right, val);
        }
    }
}
```

## AVL-деревья

AVL-деревья - это самобалансирующиеся бинарные деревья поиска, в которых высота двух поддеревьев любого узла отличается не более чем на 1.

```javascript
class AVLNode {
    constructor(val) {
        this.val = val;
        this.left = null;
        this.right = null;
        this.height = 1;
    }
}

class AVLTree {
    getHeight(node) {
        return node ? node.height : 0;
    }
    
    getBalance(node) {
        return node ? this.getHeight(node.left) - this.getHeight(node.right) : 0;
    }
    
    rotateRight(y) {
        const x = y.left;
        const T2 = x.right;
        
        x.right = y;
        y.left = T2;
        
        y.height = Math.max(this.getHeight(y.left), this.getHeight(y.right)) + 1;
        x.height = Math.max(this.getHeight(x.left), this.getHeight(x.right)) + 1;
        
        return x;
    }
    
    rotateLeft(x) {
        const y = x.right;
        const T2 = y.left;
        
        y.left = x;
        x.right = T2;
        
        x.height = Math.max(this.getHeight(x.left), this.getHeight(x.right)) + 1;
        y.height = Math.max(this.getHeight(y.left), this.getHeight(y.right)) + 1;
        
        return y;
    }
    
    insert(val) {
        this.root = this._insert(this.root, val);
    }
    
    _insert(node, val) {
        // Стандартная вставка BST
        if (!node) return new AVLNode(val);
        
        if (val < node.val) {
            node.left = this._insert(node.left, val);
        } else if (val > node.val) {
            node.right = this._insert(node.right, val);
        } else {
            return node; // Дубликаты не допускаются
        }
        
        // Обновление высоты
        node.height = 1 + Math.max(this.getHeight(node.left), this.getHeight(node.right));
        
        // Получение фактора баланса
        const balance = this.getBalance(node);
        
        // Левый-левый случай
        if (balance > 1 && val < node.left.val) {
            return this.rotateRight(node);
        }
        
        // Правый-правый случай
        if (balance < -1 && val > node.right.val) {
            return this.rotateLeft(node);
        }
        
        // Левый-правый случай
        if (balance > 1 && val > node.left.val) {
            node.left = this.rotateLeft(node.left);
            return this.rotateRight(node);
        }
        
        // Правый-левый случай
        if (balance < -1 && val < node.right.val) {
            node.right = this.rotateRight(node.right);
            return this.rotateLeft(node);
        }
        
        return node;
    }
}
```

## Красно-черные деревья

Красно-черные деревья - это самобалансирующиеся бинарные деревья поиска с дополнительными свойствами:

1. Каждый узел либо красный, либо черный
2. Корень всегда черный
3. Все листья (NULL) черные
4. Красные узлы не могут быть дочерними для красных узлов
5. Любой путь от узла до его потомков-листов содержит одинаковое количество черных узлов

## Часто решаемые задачи на деревьях

### Максимальная глубина дерева

```javascript
function maxDepth(root) {
    if (!root) return 0;
    return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
}
```

### Проверка симметричности дерева

```javascript
function isSymmetric(root) {
    if (!root) return true;
    
    function isMirror(left, right) {
        if (!left && !right) return true;
        if (!left || !right) return false;
        
        return left.val === right.val &&
               isMirror(left.left, right.right) &&
               isMirror(left.right, right.left);
    }
    
    return isMirror(root.left, root.right);
}
```

### Сумма узлов на каждом уровне

```javascript
function levelSums(root) {
    if (!root) return [];
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
        const levelSize = queue.length;
        let levelSum = 0;
        
        for (let i = 0; i < levelSize; i++) {
            const node = queue.shift();
            levelSum += node.val;
            
            if (node.left) queue.push(node.left);
            if (node.right) queue.push(node.right);
        }
        
        result.push(levelSum);
    }
    
    return result;
}
```

## Сложность алгоритмов

| Операция | BST (средний) | BST (худший) | AVL/Red-Black |
|----------|---------------|--------------|---------------|
| Поиск    | O(log n)      | O(n)         | O(log n)      |
| Вставка  | O(log n)      | O(n)         | O(log n)      |
| Удаление | O(log n)      | O(n)         | O(log n)      |

## Связь с другими структурами данных

- [[Graphs]] - Деревья являются специальным случаем графов без циклов
- [[Recursion]] - Обход деревьев часто реализуется рекурсивно
- [[Stack]] - Используется для итеративного обхода деревьев
- [[Queue]] - Используется для обхода в ширину (BFS)

## Заключение

Деревья являются фундаментальной структурой данных в программировании. Они позволяют эффективно хранить иерархические данные и обеспечивают быстрый доступ к информации. Понимание различных типов деревьев и алгоритмов их обхода критически важно для решения сложных задач программирования.

> [!tip] 
> Практикуйтесь в реализации основных операций с деревьями, так как они часто встречаются на собеседованиях по программированию.

> [!warning]
> При работе с деревьями всегда учитывайте случаи, когда дерево может быть пустым или содержать только один узел.