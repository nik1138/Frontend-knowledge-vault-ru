---
aliases: ["Жизненный цикл компонентов React", "Методы жизненного цикла"]
tags: 
  - #react
  - #lifecycle
  - #components
  - #javascript
  - #frontend
---

# Жизненный цикл компонентов React

Жизненный цикл компонентов React описывает последовательность вызовов методов от создания компонента до его удаления из DOM. Понимание этих методов критически важно для эффективной разработки приложений на React.

## Фазы жизненного цикла

Компоненты React проходят через три основные фазы:

- **Фаза монтирования** — компонент создается и вставляется в DOM
- **Фаза обновления** — компонент перерисовывается при изменении пропсов или состояния
- **Фаза размонтирования** — компонент удаляется из DOM

## Методы фазы монтирования

### constructor(props)

Конструктор вызывается при создании экземпляра компонента. Используется для инициализации состояния и привязки методов.

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }
}
```

> [!note] Важно
> Всегда вызывайте `super(props)` в начале конструктора, иначе `this.props` будет неопределенным.

### render()

Метод `render()` — единственный обязательный метод в классовом компоненте. Возвращает React-элемент, который описывает, что должно быть отображено.

```jsx
render() {
  return <div>Счетчик: {this.state.count}</div>;
}
```

### componentDidMount()

Вызывается сразу после монтирования компонента. Используется для:

- Выполнения сетевых запросов
- Установки подписок
- Инициализации сторонних библиотек

```jsx
componentDidMount() {
  fetch('/api/data')
    .then(response => response.json())
    .then(data => this.setState({ data }));
}
```

## Методы фазы обновления

### getDerivedStateFromProps(props, state)

Статический метод, вызываемый перед `render()`. Возвращает обновленное состояние на основе пропсов.

```jsx
static getDerivedStateFromProps(props, state) {
  if (props.value !== state.prevValue) {
    return {
      prevValue: props.value,
      count: props.value
    };
  }
  return null;
}
```

### shouldComponentUpdate(nextProps, nextState)

Позволяет оптимизировать производительность, предотвращая ненужные перерисовки.

```jsx
shouldComponentUpdate(nextProps, nextState) {
  return nextProps.value !== this.props.value;
}
```

### componentDidUpdate(prevProps, prevState, snapshot)

Вызывается после обновления компонента. Используется для выполнения побочных эффектов после обновления.

```jsx
componentDidUpdate(prevProps, prevState) {
  if (prevProps.value !== this.props.value) {
    // Выполнить действия при изменении пропсов
  }
}
```

## Метод фазы размонтирования

### componentWillUnmount()

Вызывается перед удалением компонента из DOM. Используется для очистки:

- Подписок
- Таймеров
- Асинхронных запросов

```jsx
componentWillUnmount() {
  window.removeEventListener('resize', this.handleResize);
  this.timer && clearInterval(this.timer);
}
```

## Методы обработки ошибок

### static getDerivedStateFromError(error)

Вызывается при ошибке в дочернем компоненте. Позволяет обновить состояние для отображения резервного UI.

```jsx
static getDerivedStateFromError(error) {
  return { hasError: true };
}
```

### componentDidCatch(error, info)

Вызывается при ошибке в дочернем компоненте. Используется для логирования ошибок.

```jsx
componentDidCatch(error, info) {
  console.error('Ошибка компонента:', error);
  console.log(info.componentStack);
}
```

## Статические методы жизненного цикла

Кроме `getDerivedStateFromProps` и `getDerivedStateFromError`, статические методы не являются частью стандартного жизненного цикла, но могут использоваться в классовых компонентах для определенных целей.

## Переход от классовых жизненных циклов к хукам

С появлением хуков в React 16.8, необходимость в классовых компонентах значительно сократилась.

### Эквиваленты хуков для методов жизненного цикла

| Метод жизненного цикла | Эквивалентный хук |
|------------------------|-------------------|
| constructor | `useState` для инициализации состояния |
| render | Тело функционального компонента |
| componentDidMount | `useEffect(() => { /* код */ }, [])` |
| componentDidUpdate | `useEffect(() => { /* код */ })` или `useEffect(() => { /* код */ }, [dependencies])` |
| componentWillUnmount | Возврат функции из `useEffect` |
| getDerivedStateFromProps | `useMemo` или `useCallback` |
| shouldComponentUpdate | `React.memo` |
| componentDidCatch, getDerivedStateFromError | Error Boundary (требует классовый компонент) |

### Пример миграции

```jsx
// Классовый компонент
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }

  componentDidMount() {
    this.interval = setInterval(() => {
      this.setState(prevState => ({
        seconds: prevState.seconds + 1
      }));
    }, 1000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return <div>Секунд: {this.state.seconds}</div>;
  }
}

// Функциональный компонент с хуками
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    return () => clearInterval(interval); // componentWillUnmount эквивалент
  }, []);

  return <div>Секунд: {seconds}</div>;
}
```

## Рассмотрения производительности

- Избегайте выполнения тяжелых вычислений в `render()`
- Используйте `shouldComponentUpdate` или `React.memo` для предотвращения ненужных перерисовок
- Не создавайте новые объекты/функции в `render()` без необходимости

## Отладка методов жизненного цикла

- Используйте `console.log` внутри методов для отслеживания их вызова
- React DevTools помогает визуализировать жизненный цикл компонентов
- Плагины браузера могут показывать производительность и обновления

## Распространенные паттерны

- Использование `componentDidMount` для загрузки данных
- Очистка ресурсов в `componentWillUnmount`
- Использование `getDerivedStateFromProps` для синхронизации состояния с пропсами

## Антипаттерны

- Выполнение побочных эффектов в `render()`
- Неочистка подписок в `componentWillUnmount`
- Вызов `setState` в `getDerivedStateFromProps` без необходимости

## Современные подходы к эквивалентам жизненного цикла

Функциональные компоненты с хуками становятся стандартом для новых приложений. Для обработки ошибок все еще требуются классовые компоненты в виде Error Boundaries.

## Связанные темы

- [[react/hooks/useEffect]] - подробнее о хуке, эквивалентном многим методам жизненного цикла
- [[react/components/class-components]] - основы классовых компонентов
- [[react/components/function-components]] - функциональные компоненты
- [[react/error-boundaries]] - обработка ошибок в React
- [[react/performance/optimization]] - оптимизация производительности компонентов
