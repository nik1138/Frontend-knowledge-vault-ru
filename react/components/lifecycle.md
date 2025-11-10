---
aliases: ["React Lifecycle Methods", "Жизненный цикл компонентов React"]
tags: 
  - #react
  - #javascript
  - #lifecycle
  - #components
  - #hooks
---

# Жизненный цикл React-компонентов

## Введение

Жизненный цикл React-компонента — это набор методов, которые вызываются в определённые моменты существования компонента. Понимание этих методов критически важно для создания эффективных и предсказуемых React-приложений.

## Фазы жизненного цикла

React-компоненты проходят через три основные фазы:

1. **Фаза монтирования** — компонент создаётся и вставляется в DOM
2. **Фаза обновления** — компонент перерисовывается при изменении props или state
3. **Фаза размонтирования** — компонент удаляется из DOM

## Методы фазы монтирования

### constructor(props)

Конструктор вызывается перед монтированием компонента. Это первый метод, который вызывается при создании компонента.

```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
```

> [!tip] Совет
> Используйте конструктор только для инициализации состояния и привязки методов. Избегайте вызова API или выполнения побочных эффектов.

### render()

Метод `render()` — единственный обязательный метод в классовом компоненте. Он должен быть чистой функцией, возвращающей JSX.

```javascript
render() {
  return <div>Счётчик: {this.state.count}</div>;
}
```

### componentDidMount()

Вызывается сразу после монтирования компонента (вставки в дерево). Это идеальное место для выполнения побочных эффектов.

```javascript
componentDidMount() {
  this.fetchData();
  this.timer = setInterval(() => this.tick(), 1000);
}
```

## Методы фазы обновления

### shouldComponentUpdate(nextProps, nextState)

Позволяет сообщить React, нужно ли перерисовывать компонент. Возвращает `true` по умолчанию.

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return this.props.value !== nextProps.value;
}
```

### componentDidUpdate(prevProps, prevState, snapshot)

Вызывается сразу после обновления компонента. Полезен для выполнения операций после обновления DOM.

```javascript
componentDidUpdate(prevProps) {
  if (prevProps.postId !== this.props.postId) {
    this.fetchData();
  }
}
```

## Метод фазы размонтирования

### componentWillUnmount()

Вызывается перед удалением компонента из дерева. Используйте для очистки ресурсов.

```javascript
componentWillUnmount() {
  clearInterval(this.timer);
  this.abortController.abort();
}
```

## Методы обработки ошибок

### componentDidCatch(error, info)

Вызывается, когда ошибка в любом дочернем компоненте. Используется для логирования ошибок.

```javascript
componentDidCatch(error, info) {
  logErrorToService(error, info);
}
```

### static getDerivedStateFromError(error)

Статический метод, используемый для рендеринга резервного UI при возникновении ошибки.

```javascript
static getDerivedStateFromError(error) {
  return { hasError: true };
}
```

## Статические методы жизненного цикла

### static getDerivedStateFromProps(props, state)

Вызывается перед вызовом `render()`. Редко используется, позволяет обновить состояние в зависимости от изменений props.

```javascript
static getDerivedStateFromProps(props, state) {
  if (props.value !== state.prevValue) {
    return {
      prevValue: props.value,
      count: 0
    };
  }
  return null;
}
```

## Альтернативы на основе хуков

С появлением хуков в React 16.8, классовые методы жизненного цикла можно заменить комбинацией [[react/hooks/useEffect]] и других хуков.

### useEffect для всех фаз

```javascript
import React, { useState, useEffect } from 'react';

function MyComponent({ value }) {
  const [count, setCount] = useState(0);

  // Аналог componentDidMount
  useEffect(() => {
    console.log('Компонент смонтирован');
  }, []);

  // Аналог componentDidUpdate
  useEffect(() => {
    if (value !== prevValue) {
      fetchData();
    }
  }, [value]);

  // Аналог componentWillUnmount
  useEffect(() => {
    return () => {
      clearInterval(timer);
    };
  }, []);

  return <div>Счётчик: {count}</div>;
}
```

## Миграция с классовых компонентов на хуки

| Классовый метод | Хуковый эквивалент |
|-----------------|-------------------|
| constructor | useState |
| render | тело компонента |
| componentDidMount | useEffect с пустым массивом зависимостей |
| componentDidUpdate | useEffect с зависимостями |
| componentWillUnmount | return функции очистки в useEffect |
| shouldComponentUpdate | React.memo |
| getDerivedStateFromProps | useState с функцией обновления |

## Производительность и оптимизация

### Избегайте ненужных перерисовок

Используйте `shouldComponentUpdate` или `React.PureComponent` для предотвращения ненужных перерисовок:

```javascript
class OptimizedComponent extends React.PureComponent {
  render() {
    return <div>{this.props.value}</div>;
  }
}
```

### Оптимизация с помощью useMemo и useCallback

```javascript
const memoizedValue = useMemo(() => expensiveCalculation(a, b), [a, b]);
const memoizedCallback = useCallback(() => doSomething(a, b), [a, b]);
```

## Отладка методов жизненного цикла

Для отладки можно использовать DevTools React или добавлять логирование:

```javascript
componentDidMount() {
  console.log('Компонент смонтирован');
  this.fetchData();
}

componentDidUpdate(prevProps) {
  console.log('Компонент обновлён', { prevProps, currentProps: this.props });
}
```

## Распространённые паттерны

### Паттерн "Загрузка данных"

```javascript
class DataComponent extends React.Component {
  state = { data: null, loading: true };

  async componentDidMount() {
    try {
      const data = await api.fetchData();
      this.setState({ data, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  }

  render() {
    if (this.state.loading) return <div>Загрузка...</div>;
    if (this.state.error) return <div>Ошибка: {this.state.error.message}</div>;
    return <div>{this.state.data}</div>;
  }
}
```

## Антипаттерны

### Вызов setState в render

```javascript
// НЕПРАВИЛЬНО
render() {
  this.setState({ count: this.state.count + 1 }); // Бесконечный цикл!
  return <div>{this.state.count}</div>;
}
```

### Выполнение побочных эффектов в render

Избегайте выполнения побочных эффектов в методе `render()`, так как он должен быть чистой функцией.

## Связанные темы

- [[react/components/class-components]] - Классовые компоненты
- [[react/components/function-components]] - Функциональные компоненты
- [[react/hooks/useEffect]] - Хук useEffect
- [[react/state-management]] - Управление состоянием
- [[react/performance]] - Оптимизация производительности
- [[react/error-boundaries]] - Обработка ошибок

## Заключение

Понимание жизненного цикла компонентов помогает создавать более предсказуемые и эффективные React-приложения. С появлением хуков подход к управлению побочными эффектами и состоянием стал более гибким, но знание классовых методов жизненного цикла остаётся важным для поддержки существующего кода.