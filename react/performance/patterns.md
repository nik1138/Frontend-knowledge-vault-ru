---
aliases: ["Оптимизация производительности React", "React производительность"]
tags: 
  - "#react"
  - "#performance"
  - "#optimization"
  - "#frontend"
---

# Паттерны и техники оптимизации производительности в React

## Введение

Оптимизация производительности в React - это критический аспект разработки масштабируемых приложений. Эта статья охватывает ключевые стратегии и паттерны, которые помогут улучшить производительность ваших React-приложений.

## Стратегии оптимизации производительности

### Оптимизация рендеринга

Ключевая стратегия - избегать ненужных рендеров компонентов. Это можно достичь с помощью:

- Использования `React.memo` для компонентов
- Применения `useMemo` и `useCallback`
- Оптимизации обновлений состояния

> [!tip] Совет
> Всегда начинайте с профилирования приложения с помощью [[React DevTools]] перед оптимизацией.

## Компонентная мемоизация

### React.memo подробное использование

`React.memo` - это функция высшего порядка, которая мемоизирует компонент, предотвращая повторный рендеринг при неизменных пропсах:

```jsx
const MyComponent = React.memo(function MyComponent({ name, items }) {
  return (
    <div>
      <h1>{name}</h1>
      {items.map(item => <Item key={item.id} data={item} />)}
    </div>
  );
});
```

> [!warning] Важно
> Используйте `React.memo` только при наличии тяжелых вычислений или большого количества дочерних компонентов.

## Паттерны useMemo и useCallback

### useMemo для мемоизации значений

```jsx
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

### useCallback для мемоизации функций

```jsx
const handleClick = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

## shouldComponentUpdate оптимизация

Для классовых компонентов можно использовать `shouldComponentUpdate` для предотвращения ненужных рендеров:

```jsx
class MyComponent extends React.PureComponent {
  shouldComponentUpdate(nextProps, nextState) {
    return nextProps.value !== this.props.value;
  }
  
  render() {
    return <div>{this.props.value}</div>;
  }
}
```

## Виртуализация

Для списков с большим количеством элементов используйте виртуализацию:

```jsx
import { FixedSizeList as List } from 'react-window';

const VirtualizedList = ({ items }) => (
  <List
    height={400}
    itemCount={items.length}
    itemSize={50}
  >
    {({ index, style }) => (
      <div style={style}>{items[index].name}</div>
    )}
  </List>
);
```

## Паттерны код-сплиттинга

### Lazy Loading с Suspense

```jsx
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Загрузка...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

## Оптимизация бандла

### Tree Shaking

Убедитесь, что используете ES6-модули для эффективного tree shaking:

```jsx
// Хорошо - позволяет tree shaking
import { debounce } from 'lodash-es';

// Плохо - импортирует весь lodash
import _ from 'lodash';
```

## Оптимизация изображений

### Lazy Loading изображений

```jsx
<img 
  src={imageSrc} 
  loading="lazy" 
  alt="Описание" 
/>
```

Или с использованием сторонних библиотек:

```jsx
import { LazyLoadImage } from 'react-lazy-load-image-component';

<LazyLoadImage
  src={imageSrc}
  effect="blur"
  alt="Описание"
/>
```

## CSS оптимизации

### CSS-in-JS оптимизации

Используйте библиотеки с поддержкой оптимизации стилей:

```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  padding: 1rem;
`;
```

## Загрузка шрифтов

### Оптимизация загрузки шрифтов

```jsx
// В HTML
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
```

## Оптимизация рендеринга

### Эффективные обновления состояния

Используйте функциональные обновления для предотвращения race condition:

```jsx
// Плохо
setCount(count + 1);

// Хорошо
setCount(prevCount => prevCount + 1);
```

## Предотвращение утечек памяти

### Очистка эффектов

```jsx
useEffect(() => {
  const subscription = subscribeToData();
  
  return () => {
    subscription.unsubscribe(); // Очистка
  };
}, []);
```

## Оптимизация согласования (Reconciliation)

React использует алгоритм согласования для определения изменений в DOM. Для оптимизации:

- Используйте стабильные `key` для списков
- Сохраняйте структуру компонентов последовательной
- Избегайте перемещения компонентов в дереве

## Профилирование с React DevTools

Используйте [[React DevTools Profiler]] для анализа производительности:

- Компонентная трассировка
- Время рендеринга
- Частота рендеров

## Тестирование производительности

### Автоматизированное тестирование

```jsx
// Пример теста производительности
import { render } from '@testing-library/react';

test('компонент рендерится быстро', () => {
  const start = performance.now();
  render(<MyComponent />);
  const end = performance.now();
  
  expect(end - start).toBeLessThan(100); // <100ms
});
```

## Мониторинг производительности в продакшене

### Инструменты мониторинга

- Web Vitals
- Error tracking
- Performance budgets

## Связанные темы

- [[React Hooks]]
- [[React State Management]]
- [[React Context]]
- [[React Styling]]
- [[React Testing]]

## Заключение

Эффективная оптимизация производительности требует системного подхода и понимания внутреннего устройства React. Важно начинать с профилирования, затем применять соответствующие паттерны оптимизации и измерять результаты.

#react #performance #optimization #frontend