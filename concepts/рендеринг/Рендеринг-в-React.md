---
aliases: [React Rendering, Рендеринг в React, React Reconciliation]
tags: [frontend, react, dom, virtual-dom, performance]
---

# Рендеринг в React

## Определение

**Рендеринг в React** - это процесс преобразования компонентов React в элементы реального DOM для отображения пользовательского интерфейса. React использует виртуальный DOM и алгоритм согласования (reconciliation) для эффективного обновления интерфейса.

## Архитектура рендеринга в React

React использует многоуровневую архитектуру для управления процессом рендеринга:

1. **React Element** - Декларативное описание пользовательского интерфейса
2. **Fiber** - Внутреннее представление работы, которое позволяет React приостанавливать, возобновлять и отменять рендеринг
3. **Reconciler** - Алгоритм, который определяет, какие части UI нужно обновить
4. **Renderer** - Конкретная реализация (React DOM, React Native и т.д.)

## Основные концепции

### 1. JSX и элементы React
```jsx
// JSX преобразуется в вызовы React.createElement()
const element = <h1 className="greeting">Привет, мир!</h1>;

// Это эквивалентно:
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Привет, мир!'
);
```

### 2. Виртуальный DOM
React создает виртуальное представление DOM в памяти и синхронизирует его с реальным DOM.

```jsx
// React создает виртуальные узлы для каждого элемента
const virtualElement = {
  type: 'div',
  props: {
    className: 'container',
    children: [
      { type: 'h1', props: { children: 'Заголовок' } },
      { type: 'p', props: { children: 'Текст' } }
    ]
  }
};
```

## Фазы рендеринга

### 1. Фаза Render (Фаза работы)
- React определяет, какие компоненты нужно обновить
- Выполняется функция компонента или метод `render()`
- Создается новое виртуальное дерево
- Этот этап может быть приостановлен и отложен (в Concurrent Mode)

### 2. Фаза Commit (Фаза фиксации)
- Применяются изменения к реальному DOM
- Выполняются эффекты (useEffect, componentDidMount и т.д.)
- Этот этап синхронный и не может быть приостановлен

## Алгоритм согласования (Reconciliation)

React использует эвристический алгоритм O(n) для определения минимального количества изменений:

### 1. Сравнение типов элементов
```jsx
// Если тип элемента меняется, React полностью заменяет поддерево
// Было:
<div>
  <Counter />
</div>

// Стало:
<span>
  <Counter />
</span>

// React уничтожит и пересоздаст всё поддерево
```

### 2. Использование ключей (keys)
```jsx
// Правильное использование ключей для оптимизации перемещений
const TodoList = ({ todos }) => (
  <ul>
    {todos.map(todo => (
      <li key={todo.id}>{todo.text}</li> // Хорошо - уникальный ID
    ))}
  </ul>
);

// Избегать использования индексов как ключей при перемещениях
const BadList = ({ items }) => (
  <ul>
    {items.map((item, index) => (
      <li key={index}>{item.name}</li> // Плохо - при перемещениях будет некорректно
    ))}
  </ul>
);
```

## Оптимизации рендеринга

### 1. React.memo()
```jsx
// Поверхностное сравнение пропсов для функциональных компонентов
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  console.log('Рендеринг ExpensiveComponent');
  return <div>{data.value}</div>;
});

// Кастомная функция сравнения
const CustomMemoComponent = React.memo(
  ({ a, b }) => <div>{a} - {b}</div>,
  (prevProps, nextProps) => {
    // Возвращает true если пропсы равны (пропускает рендер)
    return prevProps.a === nextProps.a && prevProps.b === nextProps.b;
  }
);
```

### 2. useMemo и useCallback
```jsx
import { useMemo, useCallback } from 'react';

const ParentComponent = ({ items, filter }) => {
  // Мемоизация вычислений
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);

  // Мемоизация функций
  const handleUpdate = useCallback((id) => {
    console.log('Обновление элемента:', id);
  }, []);

  return (
    <div>
      {filteredItems.map(item => (
        <ChildComponent
          key={item.id}
          item={item}
          onUpdate={handleUpdate}
        />
      ))}
    </div>
  );
};
```

### 3. React.lazy() и Suspense
```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

const App = () => (
  <div>
    <Suspense fallback={<div>Загрузка...</div>}>
      <LazyComponent />
    </Suspense>
  </div>
);
```

## Concurrent Features

### 1. React 18 и автоматический батчинг
```jsx
// В React 18 обновления автоматически батчатся
function Component() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    // Оба обновления будут батчены, даже в асинхронных колбэках
    setCount(c => c + 1);
    setFlag(f => !f);
  }

  return <button onClick={handleClick}>Нажми меня</button>;
}
```

### 2. startTransition
```jsx
import { startTransition, useState } from 'react';

function Component() {
  const [input, setInput] = useState('');
  const [list, setList] = useState([]);

  function handleInputChange(e) {
    setInput(e.target.value);
    
    // Не срочные обновления - не блокируют UI
    startTransition(() => {
      setList(expensiveCalculation(e.target.value));
    });
  }

  return (
    <div>
      <input value={input} onChange={handleInputChange} />
      <ul>{list.map(item => <li key={item.id}>{item.name}</li>)}</ul>
    </div>
  );
}
```

## Практические рекомендации

### 1. Оптимизация списков
```jsx
// Использование Windowing для больших списков
import { FixedSizeList as List } from 'react-window';

const ItemList = ({ items }) => (
  <List
    height={600}
    itemCount={items.length}
    itemSize={50}
    overscanCount={5}
  >
    {({ index, style }) => (
      <div style={style} key={items[index].id}>
        {items[index].name}
      </div>
    )}
  </List>
);
```

### 2. Правильное управление состоянием
```jsx
// Использование нормализованных структур данных
const initialState = {
  entities: {
    1: { id: 1, name: 'Item 1', category: 'A' },
    2: { id: 2, name: 'Item 2', category: 'B' },
  },
  ids: [1, 2]
};

// Вместо вложенных массивов, которые сложно обновлять
const badState = [
  { id: 1, name: 'Item 1', category: 'A' },
  { id: 2, name: 'Item 2', category: 'B' },
];
```

### 3. Использование DevTools
React DevTools помогают анализировать производительность рендеринга:
- Profiler для измерения времени рендеринга
- Highlight updates для визуализации перерисовок
- Component inspector для анализа дерева компонентов

## Частые ошибки и антипаттерны

### 1. Неправильное использование ключей
```jsx
// Плохо - использование индекса при сортировке/фильтрации
{items.map((item, index) => (
  <div key={index}>{item.name}</div>
))}

// Хорошо - использование уникального ID
{items.map(item => (
  <div key={item.id}>{item.name}</div>
))}
```

### 2. Создание функций в рендере
```jsx
// Плохо - создание новой функции при каждом рендере
function BadComponent({ items }) {
  return items.map(item => (
    <ChildComponent
      key={item.id}
      onClick={() => handleClick(item.id)} // Новая функция каждый раз!
    />
  ));
}

// Хорошо - использование useCallback или передача ID
function GoodComponent({ items, onItemClick }) {
  return items.map(item => (
    <ChildComponent
      key={item.id}
      onClick={() => onItemClick(item.id)}
    />
  ));
}
```

### 3. Ненужные перерисовки
```jsx
// Плохо - объект создается каждый раз
function BadComponent() {
  return <ExpensiveChild data={{ value: 42 }} />;
}

// Хорошо - мемоизация или вынос за пределы рендера
function GoodComponent() {
  const memoizedData = useMemo(() => ({ value: 42 }), []);
  return <ExpensiveChild data={memoizedData} />;
}
```

## Заключение

Рендеринг в React - это сложная, но эффективная система, которая позволяет создавать отзывчивые пользовательские интерфейсы. Понимание внутренних механизмов позволяет разработчикам создавать более производительные приложения и избегать распространенных ошибок.

## См. также

- [[Виртуальный DOM]]
- [[Реальный DOM]]
- [[Рендеринг в Vue]]
- [[Рендеринг в Svelte]]
- [[Оптимизация производительности]]
- [[React Fiber]]