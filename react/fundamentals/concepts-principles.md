---
tags: [react, fundamentals, architecture, concepts]
aliases: [React Concepts, React Principles]
---

# Основные концепции и принципы React

React — это библиотека для создания пользовательских интерфейсов, построенная на фундаментальных концепциях, которые обеспечивают предсказуемость, масштабируемость и производительность. В этом руководстве рассмотрим ключевые принципы React и как они работают вместе.

## Декларативное программирование

React использует декларативный подход, при котором разработчик описывает, как UI должен выглядеть в зависимости от состояния данных, а не как он должен изменяться пошагово. Это делает код более предсказуемым и легким для отладки.

```jsx
// Вместо управления DOM напрямую
function renderList(items) {
  const container = document.getElementById('list');
  container.innerHTML = '';
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    container.appendChild(li);
  });
}

// React описывает, как UI должен выглядеть
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

## Компонентная архитектура

React строится на компонентах — независимых, повторно используемых частях интерфейса, которые инкапсулируют логику, состояние и представление. Компоненты могут быть функциональными или классовыми.

```jsx
function Welcome({ name }) {
  return <h1>Привет, {name}!</h1>;
}

// Композиция компонентов
function App() {
  return (
    <div>
      <Welcome name="Алиса" />
      <Welcome name="Боб" />
    </div>
  );
}
```

Для подробного изучения компонентной архитектуры см. [[component-architecture/Component Architecture in React]].

## Односторонний поток данных

В React данные передаются сверху вниз через props. Это делает поток данных предсказуемым и легким для понимания.

```jsx
function Parent({ message }) {
  return <Child message={message} />;
}

function Child({ message }) {
  return <p>{message}</p>;
}
```

## Состояние и пропсы

**Props** — это неизменяемые данные, передаваемые от родителя к потомку. **State** — это изменяемые данные внутри компонента.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}
```

## Композиция против наследования

React поощряет композицию вместо наследования для повторного использования кода. Компоненты могут принимать дочерние элементы через `children` или специальные пропсы.

```jsx
function Dialog({ title, children, onClose }) {
  return (
    <div className="dialog">
      <header>{title}</header>
      <main>{children}</main>
      <footer>
        <button onClick={onClose}>Закрыть</button>
      </footer>
    </div>
  );
}

function App() {
  return (
    <Dialog title="Подтверждение" onClose={() => {}}>
      <p>Вы уверены?</p>
      <button>OK</button>
    </Dialog>
  );
}
```

## Единый источник истины

Состояние приложения должно иметь один источник истины. Это упрощает отладку и предотвращает рассинхронизацию данных.

```jsx
// Плохо: состояние дублируется
function BadExample() {
  const [user, setUser] = useState(null);
  const [userName, setUserName] = useState('');

  useEffect(() => {
    if (user) setUserName(user.name);
  }, [user]);

  // ...
}

// Хорошо: единый источник истины
function GoodExample() {
  const [user, setUser] = useState(null);

  return (
    <div>
      <p>{user?.name}</p>
      <p>{user?.email}</p>
    </div>
  );
}
```

## Принципы неизменяемости

В React состояние и пропсы рассматриваются как неизменяемые. Изменения создаются путем создания новых объектов, а не модификации существующих.

```jsx
// Плохо: мутация состояния
function badUpdate(items) {
  items.push(newItem);
  setState(items);
}

// Хорошо: создание нового состояния
function goodUpdate(items) {
  setState([...items, newItem]);
}
```

## Чистые функции в React

Компоненты в React должны быть чистыми функциями — они должны возвращать одинаковый результат для одних и тех же пропсов и не иметь побочных эффектов.

```jsx
// Чистый компонент
function Greeting({ name }) {
  return <h1>Привет, {name}!</h1>;
}

// Не чистый компонент
function ImpureGreeting() {
  return <h1>Привет, {Math.random()}!</h1>; // Побочный эффект
}
```

## Модель рендеринга React

React перерисовывает компоненты при изменении состояния. Это триггерится вызовами `setState` или хуками `useState`/`useReducer`.

```jsx
function Clock() {
  const [time, setTime] = useState(new Date());

  useEffect(() => {
    const timer = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  return <div>{time.toLocaleTimeString()}</div>;
}
```

## Преимущества виртуального DOM

Виртуальный DOM — это легковесное представление реального DOM. React сравнивает предыдущее и новое состояние виртуального DOM и обновляет только измененные части реального DOM.

> [!tip] 
> Виртуальный DOM позволяет React эффективно обновлять интерфейс, минимизируя дорогостоящие операции с реальным DOM.

## Преимущества JSX

JSX позволяет писать HTML-подобный синтаксис прямо в JavaScript, делая компоненты более читаемыми и понятными.

```jsx
// JSX
const element = <h1>Привет, мир!</h1>;

// То же самое без JSX
const element = React.createElement('h1', null, 'Привет, мир!');
```

## Принципы хуков

Хуки позволяют использовать состояние и другие возможности React в функциональных компонентах. Основные хуки: `useState`, `useEffect`, `useContext`.

```jsx
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Счетчик: ${count}`;
  }, [count]);

  return (
    <button onClick={() => setCount(count + 1)}>
      Нажато {count} раз
    </button>
  );
}
```

Для подробного изучения хуков см. [[react/hooks/React Hooks Guide]].

## Функциональное программирование в React

React активно использует концепции функционального программирования: чистые функции, иммутабельность, композиция.

```jsx
// Композиция функций
const withLogging = (Component) => (props) => {
  console.log('Рендер:', Component.name);
  return <Component {...props} />;
};
```

## Ментальная модель React

React работает по принципу "рендеринга состояния" — UI автоматически обновляется при изменении состояния. Это позволяет разработчику думать о UI в терминах состояния, а не о манипуляциях с DOM.

## Разделение ответственности в React

Каждый компонент отвечает за свою часть интерфейса. Это позволяет изолировать логику и упрощает тестирование и поддержку.

## Предсказуемые обновления состояния

Обновления состояния в React происходят асинхронно и батчируются для оптимизации. Это обеспечивает предсказуемое поведение.

```jsx
// Обновления будут объединены
function increment() {
  setCount(c => c + 1);
  setCount(c => c + 1);
  setCount(c => c + 1); // Счетчик увеличится на 1, а не на 3
}
```

## Связь с другими файлами React

- [[react/state-management/State Management]] — подробнее о управлении состоянием
- [[react/performance/Performance Optimization]] — оптимизация производительности
- [[react/testing/Testing React Components]] — тестирование компонентов
- [[react/context/React Context]] — использование контекста для передачи данных
- [[react/routing/React Router]] — маршрутизация в React-приложениях

## Заключение

Основные принципы React обеспечивают предсказуемую и эффективную разработку интерфейсов. Понимание этих концепций позволяет создавать более масштабируемые и поддерживаемые приложения.

Для практического применения этих концепций см. [[react/patterns/React Patterns]] и [[react/best-practices/React Best Practices]].