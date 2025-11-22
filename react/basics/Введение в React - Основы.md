---
aliases: ["React Основы", "Введение в React"]
tags: 
  - #react
  - #javascript
  - #frontend
  - #programming
  - #web-development
---

# Введение в React: Основы

## Что такое React?

React - это библиотека JavaScript для создания пользовательских интерфейсов, разработанная Facebook. Это декларативная, эффективная и гибкая библиотека, которая позволяет создавать сложные интерактивные интерфейсы из небольших повторно используемых компонентов.

React позволяет разбивать UI на независимые части и работать с ними по отдельности. Он управляет состоянием приложения и автоматически обновляет DOM при изменении данных.

> [!info] Информация
> React был представлен в 2013 году и стал одной из самых популярных библиотек для разработки интерфейсов благодаря своей производительности и подходу к созданию компонентов.

## Установка и настройка

Для начала работы с React можно использовать Create React App - официальный инструмент для создания React-приложений:

```bash
npx create-react-app my-app
cd my-app
npm start
```

Также можно подключить React через CDN в HTML-файле, но Create React App предпочтительнее для серьезных проектов.

## Первый компонент

Простой функциональный компонент в React:

```jsx
import React from 'react';

function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

export default Welcome;
```

## JSX: Основы

JSX - это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобный код внутри JavaScript. JSX преобразуется в обычные вызовы React.createElement().

```jsx
const element = <h1>Привет, мир!</h1>;

// Эквивалентно:
const element = React.createElement('h1', null, 'Привет, мир!');
```

### Особенности JSX:

- Атрибуты называются в формате camelCase (className вместо class)
- Выражения вставляются в фигурных скобках: `{expression}`
- JSX-элементы должны быть закрыты

## Элементы vs Компоненты

**Элементы** - это простейшие строительные блоки React. Они описывают, что мы хотим видеть на экране. Элементы являются неизменяемыми.

```jsx
const element = <h1>Привет, мир</h1>;
```

**Компоненты** - это функции или классы, которые возвращают элементы. Компоненты позволяют разделить UI на независимые части, которые можно повторно использовать.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}</h1>;
}
```

## Рендеринг элементов

Для рендеринга React-элемента в DOM, мы передаем корневой DOM-элемент и вызываем `ReactDOM.render()`:

```jsx
const rootElement = document.getElementById('root');
ReactDOM.render(<h1>Привет, мир!</h1>, rootElement);
```

## Компоненты и props

Props (свойства) - это входные данные для компонентов. Они передаются от родительского компонента к дочернему и являются неизменяемыми.

```jsx
function Avatar(props) {
  return (
    <img className="Avatar"
         src={props.user.avatarUrl}
         alt={props.user.name} />
  );
}

function Welcome(props) {
  return (
    <div>
      <h1>Привет, {props.name}!</h1>
      <Avatar user={props.user} />
    </div>
  );
}
```

## Состояние (state) и setState

State позволяет компоненту изменяться во времени. В функциональных компонентах используется хук `useState`, а в классовых компонентах - свойство `this.state`.

### Функциональный компонент со state:

```jsx
import React, { useState } from 'react';

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

### Классовый компонент со state:

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  render() {
    return (
      <div>
        <p>Счетчик: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Увеличить
        </button>
      </div>
    );
  }
}
```

## Обработка событий

Обработка событий в React похожа на обработку событий в DOM, но с некоторыми синтаксическими различиями:

- События именуются в формате camelCase
- Передается функция, а не строка
- Используется SyntheticEvent - кросс-браузерная обертка над нативными событиями

```jsx
function Button() {
  const handleClick = () => {
    console.log('Кнопка нажата!');
  };

  return <button onClick={handleClick}>Нажми меня</button>;
}
```

## Условный рендеринг

В React можно создавать различные компоненты для инкапсуляции поведения и затем отображать только те, которые необходимы.

```jsx
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  
  if (isLoggedIn) {
    return <h1>Добро пожаловать!</h1>;
  }
  return <h1>Пожалуйста, войдите.</h1>;
}
```

Также можно использовать логические операторы && или тернарный оператор:

```jsx
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  
  return (
    <div>
      <h1>Привет!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          У вас {unreadMessages.length} непрочитанных сообщений.
        </h2>
      }
    </div>
  );
}
```

## Списки и ключи

При отображении списков в React необходимо использовать ключи (keys) для идентификации элементов.

```jsx
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

Ключи помогают React определить, какие элементы изменились, были добавлены или удалены. Идеальный ключ - это строка, которая однозначно идентифицирует элемент среди его соседей.

## Формы и контролируемые компоненты

В HTML элементы формы, такие как `<input>`, `<textarea>` и `<select>`, сами по себе управляют своим состоянием и обновляют его при вводе пользователем. В React изменяемое состояние обычно хранится в свойстве state компонента и обновляется с помощью `setState()`.

Контролируемый компонент - это компонент, значение которого контролируется React:

```jsx
function NameForm() {
  const [value, setValue] = useState('');

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  const handleSubmit = (event) => {
    alert('Отправленное имя: ' + value);
    event.preventDefault();
  };

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Имя:
        <input type="text" value={value} onChange={handleChange} />
      </label>
      <input type="submit" value="Отправить" />
    </form>
  );
}
```

## Поднятие состояния (lifting state up)

Часто несколько компонентов должны отражать одно и то же изменяющееся состояние. В таких случаях рекомендуется поднять общее состояние до ближайшего общего предка.

```jsx
function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  return (
    <fieldset>
      <legend>Введите температуру в {scale}:</legend>
      <input value={temperature}
             onChange={(e) => onTemperatureChange(e.target.value)} />
    </fieldset>
  );
}

function Calculator() {
  const [celsius, setCelsius] = useState('');
  const [fahrenheit, setFahrenheit] = useState('');

  const handleCelsiusChange = (temperature) => {
    setCelsius(temperature);
    setFahrenheit(tryConvert(temperature, celsiusToFahrenheit));
  };

  const handleFahrenheitChange = (temperature) => {
    setFahrenheit(temperature);
    setCelsius(tryConvert(temperature, fahrenheitToCelsius));
  };

  return (
    <div>
      <TemperatureInput
        scale="C"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange} />
      <TemperatureInput
        scale="F"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange} />
    </div>
  );
}
```

## Думай в React (Thinking in React)

При создании React-приложения полезно мысленно разделить интерфейс на компоненты. Вот основные шаги:

1. **Разделите UI на компоненты** - Нарисуйте границы вокруг каждого компонента
2. **Создайте статическую версию** - Реализуйте UI без состояния
3. **Определите минимальное представление состояния** - Что нужно хранить в состоянии?
4. **Определите, где должно находиться состояние** - Какой компонент должен владеть каждым фрагментом состояния?
5. **Добавьте обратные вызовы** - Как родительские компоненты будут общаться с дочерними?

## Связь с другими файлами React

- [[React Hooks]] - для управления состоянием и побочными эффектами в функциональных компонентах
- [[React Lifecycle]] - жизненный цикл компонентов и методы жизненного цикла
- [[React Router]] - навигация и маршрутизация в React-приложениях
- [[React Context]] - передача данных через дерево компонентов без пропсов
- [[React Performance]] - оптимизация производительности React-приложений
- [[React Testing]] - тестирование React-компонентов

## Заключение

React предоставляет мощный инструментарий для создания интерактивных пользовательских интерфейсов. Основные концепции - компоненты, props, state и JSX - формируют основу для понимания React. Понимание этих основ позволяет строить сложные приложения, разбивая их на маленькие, управляемые части.

> [!tip] Совет
> Начинайте с простых компонентов и постепенно усложняйте. Практика - ключ к освоению React!

> [!warning] Важно
> Не забывайте использовать ключи при рендеринге списков и следите за тем, чтобы состояние обновлялось корректно через setState или хуки.
