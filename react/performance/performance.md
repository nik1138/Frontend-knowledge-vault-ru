---
aliases: ["React Оптимизация Производительности", "React Performance"]
tags: [react, performance, optimization, memoization, rendering]
---

# Оптимизация производительности React

## Введение

React - мощная библиотека для создания пользовательских интерфейсов, но без надлежащей оптимизации приложения могут работать медленно. Эта статья охватывает ключевые стратегии оптимизации производительности в React-приложениях.

## Механизм рендеринга React

React использует виртуальный DOM для эффективного обновления реального DOM. При изменении состояния или пропсов React:

1. Создает новое дерево виртуального DOM
2. Сравнивает его с предыдущим деревом (reconciliation)
3. Вычисляет минимальный набор изменений
4. Применяет изменения к реальному DOM

> [!warning] Важно
> Понимание этого процесса критично для оптимизации производительности, так как позволяет избежать ненужных перерисовок.

## Стратегии оптимизации рендеринга

### Избегайте ненужных перерисовок

Компоненты перерисовываются при изменении состояния родителя или при изменении пропсов. Избегайте ненужных перерисовок:

```jsx
// Плохо - создает новую функцию при каждом рендере
function BadComponent() {
  return <ChildComponent onClick={() => console.log('click')} />;
}

// Хорошо - использует useCallback
function GoodComponent() {
  const handleClick = useCallback(() => console.log('click'), []);
  return <ChildComponent onClick={handleClick} />;
}
```

### PureComponents и React.memo

React.memo - это HOC, который предотвращает перерисовку компонента, если пропсы не изменились:

```jsx
const MyComponent = React.memo(function MyComponent({ name }) {
  return <div>{name}</div>;
});
```

[[react/components/components]] содержит больше информации о компонентах React.

## Хуки оптимизации

### useMemo

Используйте `useMemo` для мемоизации вычислений:

```jsx
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

### useCallback

`useCallback` мемоизирует функции:

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

[[react/hooks/hooks]] содержит подробную информацию о хуках.

## Разделение кода

### Lazy Loading и Suspense

Разделение кода позволяет загружать компоненты по требованию:

```jsx
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}
```

## Оптимизация больших списков

### Виртуальный скроллинг

Для больших списков используйте библиотеки виртуального скроллинга, такие как `react-window`:

```jsx
import { FixedSizeList as List } from 'react-window';

function Row({ index, style }) {
  return <div style={style}>Item {index}</div>;
}

function VirtualizedList({ items }) {
  return (
    <List
      height={400}
      itemCount={items.length}
      itemSize={35}
    >
      {Row}
    </List>
  );
}
```

## Оптимизация Context

Context может вызывать ненужные перерисовки. Оптимизируйте его использование:

```jsx
// Разделяйте контексты по функциональности
const ThemeContext = createContext();
const UserContext = createContext();
```

[[react/state/context]] содержит подробную информацию об использовании Context API.

## Оптимизация состояния

Используйте подходящие стратегии управления состоянием:

- Для локального состояния - `useState`, `useReducer`
- Для глобального состояния - Redux, Zustand или Context API
- Для асинхронных данных - React Query, SWR

## Инструменты профилирования

### Profiler API

React предоставляет компонент Profiler для измерения производительности:

```jsx
<Profiler id="ListItem" onRender={onRenderCallback}>
  <ListItem />
</Profiler>
```

### Инструменты разработчика React

- React DevTools Profiler
- Chrome DevTools Performance
- Web Vitals

## Оптимизация сборки

### Tree Shaking

Tree Shaking удаляет неиспользуемый код из финальной сборки. Используйте ES6-импорты:

```jsx
// Плохо
import _ from 'lodash';
const result = _.pick(object, ['a', 'b']);

// Хорошо
import { pick } from 'lodash';
const result = pick(object, ['a', 'b']);
```

## Тестирование производительности

### Профилирование в продакшене

Используйте Web Vitals для измерения ключевых метрик:

- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)

## Распространенные ошибки

1. Создание анонимных функций в рендере
2. Неправильное использование ключей (keys)
3. Чрезмерная мемоизация
4. Неоптимизированные циклы рендеринга
5. Неправильное разделение кода

## Заключение

Оптимизация производительности React - это итеративный процесс. Начинайте с измерения, определяйте узкие места, а затем применяйте соответствующие стратегии. Помните, что преждевременная оптимизация может усложнить код без реальной пользы.

## Связанные темы

- [[react/components/components]]
- [[react/hooks/hooks]]
- [[react/state/context]]
- [[react/state/state-management]]
- [[react/performance/bundling]]

## Теги

#react #performance #optimization #memoization #rendering #javascript #frontend