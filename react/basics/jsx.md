# JSX: Полное руководство

JSX (JavaScript XML) — это синтаксическое расширение JavaScript, которое позволяет писать код, похожий на HTML, внутри JavaScript. JSX не понимается браузерами напрямую и требует транспиляции в обычный JavaScript перед выполнением.

## Что такое JSX

JSX — это синтаксис, вдохновлённый XML, который позволяет писать выражения разметки в стиле HTML внутри JavaScript кода. Он используется в React для описания внешнего вида пользовательского интерфейса.

```jsx
const element = <h1>Привет, мир!</h1>;
```

> [!note] Примечание
> JSX не является обязательным для использования React, но он делает создание компонентов более интуитивным и похожим на создание HTML.

## JSX трансформация

Когда JSX код компилируется, он преобразуется в вызовы `React.createElement()`. Например:

```jsx
// До трансформации
const element = <h1 className="greeting">Привет!</h1>;

// После трансформации
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Привет!'
);
```

Это преобразование обычно выполняется с помощью Babel.

## Элементы против компонентов

**React элемент** — это простой объект, описывающий, что нужно отрендерить. Это нечто вроде "описания" DOM узла:

```jsx
const element = <div>Привет</div>;
```

**React компонент** — это функция или класс, который возвращает элемент React:

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}</h1>;
}
```

## Выражения в JSX

В JSX можно вставлять любые JavaScript выражения, заключая их в фигурные скобки `{}`:

```jsx
const name = 'Алиса';
const element = <h1>Привет, {name}!</h1>;
```

Также можно вставлять вычисления:

```jsx
const user = { firstName: 'Иван', lastName: 'Иванов' };
const element = <h2>Привет, {user.firstName + ' ' + user.lastName}</h2>;
```

## Атрибуты в JSX

Атрибуты в JSX записываются как в HTML, но с некоторыми отличиями:

```jsx
// Строковые атрибуты
const element = <div tabIndex="0">...</div>;

// Переменные в атрибутах
const className = 'container';
const element = <div className={className}>...</div>;

// Атрибуты с вычислениями
const isActive = true;
const element = <button disabled={!isActive}>Кнопка</button>;
```

> [!warning] Важно
> В JSX атрибут `class` заменяется на `className`, `for` — на `htmlFor`, так как `class` и `for` являются зарезервированными словами в JavaScript.

## Дочерние элементы в JSX

JSX позволяет вкладывать элементы друг в друга:

```jsx
const element = (
  <div>
    <h1>Заголовок</h1>
    <p>Параграф</p>
  </div>
);
```

Когда тег не имеет дочерних элементов, его можно закрыть сразу:

```jsx
const element = <img src="image.jpg" alt="Картинка" />;
```

## Условный рендеринг в JSX

В JSX можно использовать условные выражения:

```jsx
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  
  if (isLoggedIn) {
    return <h1>Добро пожаловать!</h1>;
  }
  return <h1>Пожалуйста, войдите.</h1>;
}

// Или с помощью тернарного оператора
function Greeting(props) {
  return (
    <div>
      {props.isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
    </div>
  );
}

// Или с помощью логического И
function Mailbox(props) {
  return (
    <div>
      <h1>Привет!</h1>
      {props.messages.length > 0 && <h2>У вас {props.messages.length} сообщений.</h2>}
    </div>
  );
}
```

## Списки и ключи в JSX

При рендеринге списков в JSX необходимо использовать атрибут `key`:

```jsx
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

Ключи помогают React определять, какие элементы изменились, были добавлены или удалены.

## JSX с пропсами

Компоненты могут принимать пропсы (свойства) через JSX:

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}</h1>;
}

// Использование компонента с пропсами
<Welcome name="Алиса" />
```

## Композиция компонентов с JSX

Компоненты могут использоваться внутри других компонентов:

```jsx
function App() {
  return (
    <div>
      <Welcome name="Алиса" />
      <Welcome name="Боб" />
    </div>
  );
}
```

## Spread атрибуты в JSX

Можно использовать оператор spread для передачи пропсов:

```jsx
const props = { firstName: 'Иван', lastName: 'Иванов' };
const element = <Component {...props} />;
```

## JSX фрагменты

Фрагменты позволяют группировать дочерние элементы без добавления лишних DOM-узлов:

```jsx
import React, { Fragment } from 'react';

// С использованием тега Fragment
function App() {
  return (
    <Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </Fragment>
  );
}

// Или с сокращённым синтаксисом
function App() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC />
    </>
  );
}
```

## Ограничения JSX

- JSX выражения должны быть валидными (всегда возвращать один корневой элемент или фрагмент)
- Имена компонентов должны начинаться с заглавной буквы
- JSX чувствителен к регистру

## Соображения безопасности в JSX (предотвращение XSS)

React по умолчанию экранирует все значения, вставляемые в JSX, что помогает предотвратить XSS атаки:

```jsx
// Это безопасно
const content = '<script>alert("XSS")</script>';
const element = <div>{content}</div>; // Будет отображено как текст, а не выполнено
```

> [!caution] Осторожно
> Использование `dangerouslySetInnerHTML` отключает защиту от XSS и должно использоваться с осторожностью.

## Производительность в JSX

- Используйте `React.memo()` для предотвращения ненужных повторных рендеров
- Избегайте создания новых объектов и функций в JSX
- Оптимизируйте рендеринг списков с помощью `key`

## Лучшие практики JSX

- Используйте осмысленные имена компонентов
- Разбивайте сложные компоненты на более мелкие
- Используйте деструктуризацию для пропсов
- Организуйте атрибуты на нескольких строках при большом количестве

```jsx
function UserProfile({
  name,
  email,
  avatar,
  isActive
}) {
  return (
    <div className="user-profile">
      <img src={avatar} alt={name} />
      <h2>{name}</h2>
      <p>{email}</p>
      {isActive && <span className="status">Активен</span>}
    </div>
  );
}
```

## Отличия от HTML

- Атрибут `class` → `className`
- Атрибут `for` → `htmlFor`
- События именуются в camelCase: `onClick` вместо `onclick`
- Условия выражаются через JavaScript выражения, а не через специальные атрибуты

## Спецификации JSX компилятора

JSX спецификация определяет синтаксис, но не реализацию. Каждый компилятор может реализовать JSX по-своему, но большинство следуют общим принципам трансформации в вызовы `React.createElement()`.

## Babel трансформация

Babel с плагином `@babel/preset-react` преобразует JSX в вызовы `React.createElement()`:

```jsx
// До
const element = <h1 className="title">Привет</h1>;

// После
var element = React.createElement(
  "h1",
  { className: "title" },
  "Привет"
);
```

## Связь с другими файлами React

- [[react/basics/components]] - компоненты и их жизненный цикл
- [[react/basics/props-and-state]] - пропсы и состояние
- [[react/basics/events]] - обработка событий
- [[react/advanced/hooks]] - хуки для управления состоянием
- [[react/advanced/context]] - контекст для передачи данных

#react #jsx #javascript #programming #frontend