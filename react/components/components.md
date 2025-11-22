# Компоненты React: Полное руководство

## Введение

Компоненты являются основой React-приложений. Они позволяют разбивать интерфейс на независимые части, которые можно повторно использовать и управлять ими по отдельности. В этом руководстве мы рассмотрим все аспекты компонентов React, от базовых типов до продвинутых паттернов.

## Типы компонентов

### Функциональные компоненты

Функциональные компоненты - это JavaScript-функции, которые принимают `props` и возвращают JSX. Они являются предпочтительным способом создания компонентов в современном React:

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Или с использованием стрелочных функций
const Welcome = (props) => {
  return <h1>Привет, {props.name}!</h1>;
};
```

Функциональные компоненты проще для понимания и тестирования. С введением хуков они получили возможность управления состоянием и побочными эффектами.

### Классовые компоненты

Классовые компоненты определяются как ES6-классы, расширяющие `React.Component`:

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}
```

Классовые компоненты имеют доступ к методам жизненного цикла и состоянию через `this.state`. Хотя они все еще поддерживаются, функциональные компоненты с хуками считаются более предпочтительным подходом.

Сравнение функциональных и классовых компонентов можно найти в [[react/components/functional-vs-class-components]].

## Жизненный цикл компонентов

### Методы жизненного цикла (для классовых компонентов)

Классовые компоненты имеют методы жизненного цикла, которые вызываются на разных этапах существования компонента:

- **Mounting (Монтирование)**:
  - `constructor()`: Инициализация состояния и привязка методов
  - `render()`: Создание DOM-элементов
  - `componentDidMount()`: Выполняется после монтирования

- **Updating (Обновление)**:
  - `componentDidUpdate()`: Выполняется после обновления

- **Unmounting (Размонтирование)**:
  - `componentWillUnmount()`: Выполняется перед удалением

### Хуки жизненного цикла (для функциональных компонентов)

Функциональные компоненты используют хуки для реализации логики жизненного цикла:

- `useEffect`: Объединяет `componentDidMount`, `componentDidUpdate` и `componentWillUnmount`
- `useState`: Управление локальным состоянием
- `useContext`: Доступ к контексту

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `Вы нажали ${count} раз`;
  });

  return (
    <div>
      <p>Вы нажали {count} раз</p>
      <button onClick={() => setCount(count + 1)}>
        Нажми меня
      </button>
    </div>
  );
}
```

Для более подробной информации о хуках см. [[Типизация хуков React в TypeScript]].

## Паттерны компоновки

### Композиция через props и children

Компоненты могут использовать `props.children` для вставки дочерних элементов:

```jsx
function FancyBorder(props) {
  return (
    <div className={'border-' + props.color}>
      {props.children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Добро пожаловать
      </h1>
      <p className="Dialog-message">
        Спасибо, что посетили наш космический корабль!
      </p>
    </FancyBorder>
  );
}
```

### Паттерн Render Props

Паттерн Render Props позволяет компонентам использовать функцию для определения того, что рендерить:

```jsx
class DataFetcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: null };
  }

  componentDidMount() {
    // Логика получения данных
  }

  render() {
    return this.props.render(this.state.data);
  }
}

// Использование
<DataFetcher render={(data) => (
  <div>{data ? data.content : 'Загрузка...'}</div>
)} />
```

### Составные компоненты

Составные компоненты объединяют несколько связанных компонентов в единый интерфейс:

```jsx
const Menu = ({ children }) => (
  <div className="menu">{children}</div>
);

const MenuItem = ({ children, onClick }) => (
  <div className="menu-item" onClick={onClick}>
    {children}
  </div>
);

// Использование
<Menu>
  <MenuItem onClick={() => console.log('Item 1')}>
    Пункт 1
  </MenuItem>
  <MenuItem onClick={() => console.log('Item 2')}>
    Пункт 2
  </MenuItem>
</Menu>
```

## Управление состоянием компонента

### Локальное состояние

Функциональные компоненты используют `useState` для управления локальным состоянием:

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
};
```

### Внешнее состояние

Для сложных приложений может использоваться внешнее управление состоянием через библиотеки вроде Redux или Context API:

```jsx
const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

## Props и Children

Props (свойства) - это способ передачи данных от родительского компонента к дочернему:

```jsx
function Avatar(props) {
  return (
    <img 
      className={props.className}
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}

// Использование
<Avatar 
  className="profile-img" 
  user={currentUser} 
/>
```

`children` - это специальный prop, содержащий дочерние элементы, переданные компоненту:

```jsx
function Card(props) {
  return (
    <div className="card">
      {props.children}
    </div>
  );
}

// Использование
<Card>
  <h2>Заголовок</h2>
  <p>Содержимое карточки</p>
</Card>
```

## Паттерн Forward Ref

`forwardRef` позволяет передавать ref от родительского компонента к дочернему:

```jsx
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="fancy-button">
    {props.children}
  </button>
));

// Использование
const ref = useRef();
<FancyButton ref={ref}>Нажми меня</FancyButton>;
```

## Fragments

Fragments позволяют группировать элементы без добавления лишних DOM-узлов:

```jsx
function ItemList() {
  return (
    <>
      <li>Элемент 1</li>
      <li>Элемент 2</li>
      <li>Элемент 3</li>
    </>
  );
}
```

## Пользовательские хуки как компоненты

Пользовательские хуки позволяют извлекать логику компонентов в повторно используемые функции:

```jsx
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}

// Использование в компоненте
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Загрузка...';
  }
  return isOnline ? 'Онлайн' : 'Офлайн';
}
```

## Оптимизация компонентов

### React.memo

`React.memo` - это функция высшего порядка, которая предотвращает повторный рендеринг компонента, если его props не изменились:

```jsx
const MyComponent = React.memo(function MyComponent({ name, age }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>{age} лет</p>
    </div>
  );
});
```

### PureComponent

Для классовых компонентов можно использовать `PureComponent`, который реализует поверхностное сравнение props и state:

```jsx
class MyComponent extends React.PureComponent {
  render() {
    return <div>{this.props.value}</div>;
  }
}
```

## Лучшие практики

1. **Компоненты должны быть чистыми**: Не мутируйте props, возвращайте одинаковые результаты для одних и тех же входных данных
2. **Одна задача**: Компонент должен решать одну задачу, делиться логикой через композицию
3. **Именование**: Используйте понятные имена компонентов (например, `UserProfile`, а не `Component1`)
4. **Организация**: Разделяйте компоненты по функциональности в разных файлах
5. **Тестирование**: Пишите юнит-тесты для компонентов

## Связи с другими файлами

- [[react/state-management]] - Управление состоянием в React
- [[Типизация хуков React в TypeScript]] - Пользовательские и встроенные хуки
- [[Типизация контекста в React с TypeScript]] - Контекстная передача данных
- [[react/styling]] - Стилизация компонентов
- [[react/testing]] - Тестирование компонентов
- [[react/performance]] - Производительность и оптимизация

## Заключение

Компоненты - это фундаментальная концепция React, позволяющая создавать масштабируемые и поддерживаемые пользовательские интерфейсы. Понимание различных типов компонентов, паттернов композиции и методов оптимизации позволяет создавать эффективные и надежные приложения.

---

**Теги:** #react #components #javascript #frontend #programming #best-practices #web-development