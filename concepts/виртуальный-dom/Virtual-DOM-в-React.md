---
aliases: ["Virtual DOM в React", "React Virtual DOM", "Виртуальный DOM React"]
tags: [frontend, react, virtual-dom, javascript, performance]
---

# Virtual DOM в React

## Введение

Virtual DOM (виртуальный DOM) в React - это концепция, лежащая в основе архитектуры фреймворка, обеспечивающая высокую производительность при обновлениях пользовательского интерфейса. React создает легковесное представление реального DOM в памяти и использует его для определения минимального набора изменений, необходимых для обновления браузерного DOM.

## Архитектура React Fiber

React 16 представил новую архитектуру под названием Fiber, которая заменила предыдущую рекурсивную реализацию Virtual DOM. Fiber позволяет:

- Прерывать и возобновлять работу по рендерингу
- Приоритизировать обновления
- Использовать время браузера более эффективно

### Основные компоненты Fiber

```javascript
// Структура Fiber узла в упрощенном виде
const FiberNode = {
  tag: 1, // Тип узла (HostComponent, FunctionComponent и т.д.)
  key: null,
  elementType: null,
  type: null,
  stateNode: null, // Реальный DOM-элемент или экземпляр компонента
  return: null, // Родительский Fiber
  child: null, // Первый дочерний Fiber
  sibling: null, // Следующий сестринский Fiber
  alternate: null, // Предыдущая версия Fiber для сравнения
  pendingProps: null,
  memoizedProps: null,
  memoizedState: null,
  effectTag: 0, // Типы эффектов (Placement, Update, Deletion)
  nextEffect: null
};
```

## Рендеринг в React

Процесс рендеринга в React включает несколько этапов:

### 1. Рендер-фаза (Render Phase)

Во время этой фазы React:
- Вычисляет новое дерево Fiber
- Сравнивает его с предыдущим деревом (reconciliation)
- Определяет минимальный набор изменений

```javascript
// Упрощенная реализация процесса рендеринга
function reconcileChildren(currentFiber, newChildren) {
  let oldChild = currentFiber.child;
  let newChild = newChildren[0]; // Упрощение
  
  if (!oldChild && newChild) {
    // Добавление нового узла
    const newFiber = createFiber(newChild);
    currentFiber.child = newFiber;
    newFiber.return = currentFiber;
  } else if (oldChild && !newChild) {
    // Удаление узла
    currentFiber.effectTag = Deletion;
    currentFiber.deletions = [oldChild];
  } else if (oldChild && newChild) {
    // Обновление узла
    if (oldChild.type === newChild.type) {
      oldChild.pendingProps = newChild.props;
      currentFiber.child = oldChild; // Используем существующий Fiber
    } else {
      // Тип узла изменился - замена
      const newFiber = createFiber(newChild);
      currentFiber.effectTag = Placement;
      currentFiber.child = newFiber;
      newFiber.return = currentFiber;
    }
  }
}
```

### 2. Фаза коммита (Commit Phase)

Во время этой фазы React:
- Применяет вычисленные изменения к реальному DOM
- Выполняет эффекты (useEffect, componentDidMount и т.д.)

```javascript
// Упрощенная реализация коммит-фазы
function commitWork(fiber) {
  if (!fiber) return;
  
  const effectTag = fiber.effectTag;
  
  if (effectTag & Placement) {
    // Добавление узла
    commitPlacement(fiber);
  } else if (effectTag & Update) {
    // Обновление узла
    commitUpdate(fiber);
  } else if (effectTag & Deletion) {
    // Удаление узла
    commitDeletion(fiber);
  }
  
  commitWork(fiber.child);
  commitWork(fiber.sibling);
}
```

## Reconciliation (согласование)

React использует алгоритм согласования для определения изменений между старым и новым деревьями:

### Стратегии согласования

1. **Сравнение по типу элемента**
2. **Использование ключей (keys)**
3. **Рекурсивное сравнение дочерних элементов**

```javascript
// Алгоритм согласования для дочерних элементов
function reconcileChildFibers(returnFiber, currentFirstChild, newChildren) {
  if (typeof newChildren === 'object' && newChildren !== null) {
    if (Array.isArray(newChildren)) {
      return reconcileChildrenArray(returnFiber, currentFirstChild, newChildren);
    } else {
      return reconcileSingleChild(returnFiber, currentFirstChild, newChildren);
    }
  }
  
  // Обработка других типов (null, строк и т.д.)
  deleteRemainingChildren(returnFiber, currentFirstChild);
  return null;
}

function reconcileChildrenArray(returnFiber, currentFirstChild, newChildren) {
  let resultingFirstChild = null;
  let previousNewFiber = null;
  
  let oldFiber = currentFirstChild;
  let newIdx = 0;
  let nextOldFiber = null;
  
  // Первый проход: сравнение по индексу
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    if (oldFiber.index > newIdx) {
      nextOldFiber = oldFiber;
      oldFiber = null;
    } else {
      nextOldFiber = oldFiber.sibling;
    }
    
    const newFiber = updateSlot(returnFiber, oldFiber, newChildren[newIdx], newIdx);
    
    if (newFiber === null) {
      if (oldFiber === null) {
        oldFiber = nextOldFiber;
      }
      break;
    }
    
    if (shouldTrackSideEffects) {
      if (oldFiber && newFiber.alternate === null) {
        // Удаление старого узла
        deleteChild(returnFiber, oldFiber);
      }
    }
    
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    
    previousNewFiber = newFiber;
    oldFiber = nextOldFiber;
  }
  
  // Добавление новых элементов или удаление лишних
  if (newIdx === newChildren.length) {
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }
  
  if (oldFiber === null) {
    for (; newIdx < newChildren.length; newIdx++) {
      const newFiber = createChild(returnFiber, newChildren[newIdx], newIdx);
      if (newFiber !== null) {
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
    }
    return resultingFirstChild;
  }
  
  // Создание маппинга для поиска по ключам
  const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
  
  for (; newIdx < newChildren.length; newIdx++) {
    const newFiber = updateFromMap(existingChildren, returnFiber, newIdx, newChildren[newIdx]);
    
    if (newFiber !== null) {
      if (shouldTrackSideEffects) {
        if (newFiber.alternate !== null) {
          existingChildren.delete(newFiber.key === null ? newIdx : newFiber.key);
        }
      }
      
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      
      previousNewFiber = newFiber;
    }
  }
  
  if (shouldTrackSideEffects) {
    existingChildren.forEach(child => deleteChild(returnFiber, child));
  }
  
  return resultingFirstChild;
}
```

## Ключи (Keys) в React

Ключи играют важную роль в эффективности Virtual DOM в React:

```jsx
// Правильное использование ключей
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}

// Неправильное использование ключей (не рекомендуется)
function BadTodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

## Оптимизации производительности

React предоставляет несколько способов оптимизации работы с Virtual DOM:

### React.memo

```jsx
// Мемоизация функциональных компонентов
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  // Рендер компонента
  return <div>{data.value}</div>;
});

// С пользовательской функцией сравнения
const CustomMemoComponent = React.memo(function CustomMemoComponent({ data }) {
  return <div>{data.value}</div>;
}, (prevProps, nextProps) => {
  // Возвращаем true, если пропсы равны
  return prevProps.data.id === nextProps.data.id;
});
```

### useMemo и useCallback

```jsx
import { useMemo, useCallback } from 'react';

function ExpensiveComponent({ items, multiplier }) {
  // Мемоизация вычислений
  const expensiveValue = useMemo(() => {
    return items.map(item => item * multiplier);
  }, [items, multiplier]);
  
  // Мемоизация функций
  const handleClick = useCallback((id) => {
    console.log(`Clicked item ${id}`);
  }, []);
  
  return (
    <div>
      {expensiveValue.map((value, index) => (
        <button key={index} onClick={() => handleClick(index)}>
          {value}
        </button>
      ))}
    </div>
  );
}
```

### Стратегии разделения кода

React поддерживает lazy loading компонентов:

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Загрузка...</div>}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}
```

## Практические примеры

### Пример 1: Работа со списками

```jsx
import React, { useState } from 'react';

function ListExample() {
  const [items, setItems] = useState([
    { id: 1, text: 'Элемент 1' },
    { id: 2, text: 'Элемент 2' },
    { id: 3, text: 'Элемент 3' }
  ]);
  
  const addItem = () => {
    const newItem = {
      id: Date.now(),
      text: `Элемент ${items.length + 1}`
    };
    setItems([...items, newItem]);
  };
  
  const removeItem = (id) => {
    setItems(items.filter(item => item.id !== id));
  };
  
  return (
    <div>
      <button onClick={addItem}>Добавить элемент</button>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.text}
            <button onClick={() => removeItem(item.id)}>Удалить</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Пример 2: Условный рендеринг

```jsx
import React, { useState } from 'react';

function ConditionalRender() {
  const [showDetails, setShowDetails] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowDetails(!showDetails)}>
        {showDetails ? 'Скрыть' : 'Показать'} детали
      </button>
      
      {showDetails && (
        <div className="details">
          <h3>Детальная информация</h3>
          <p>Это дополнительный контент, который появляется при нажатии</p>
        </div>
      )}
    </div>
  );
}
```

## Заключение

Virtual DOM в React - это мощный механизм, обеспечивающий высокую производительность при обновлениях UI. Архитектура Fiber позволяет React эффективно управлять процессом рендеринга, а правильное использование ключей и оптимизаций позволяет создавать быстрые и отзывчивые приложения.

См. также:
- [[Реализация-Virtual-DOM]]
- [[Diffing-алгоритмы]]
- [[Virtual-DOM-в-Vue]]
- [[Виртуальный-DOM]]