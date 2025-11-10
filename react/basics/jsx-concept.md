---
tags: [react, jsx, javascript, frontend]
aliases: [JSX, React JSX, JSX спецификация]
---

# JSX в React: Полное руководство

## Что такое JSX

JSX (JavaScript XML) — это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобный синтаксис внутри JavaScript-кода. Это не обязательная часть React, но делает создание компонентов более интуитивным и визуально понятным.

```jsx
const element = <h1>Привет, мир!</h1>;
```

JSX сочетает в себе силу JavaScript и выразительность HTML, позволяя создавать компоненты с привычной разметкой, но с возможностью использовать все возможности JavaScript.

## Спецификация JSX

JSX определяется [спецификацией JSX](https://facebook.github.io/jsx/), которая описывает синтаксические правила:
- Элементы должны быть закрыты (или использовать самозакрывающийся тег)
- Только один корневевой элемент разрешен
- Имена тегов чувствительны к регистру (начинаются с заглавной буквы для компонентов)
- Атрибуты используют camelCase (например, `className` вместо `class`)

## Процесс трансформации

JSX не может быть выполнено браузером напрямую. Он должен быть преобразован в обычный JavaScript с помощью транспайлера, обычно Babel. Трансформация происходит следующим образом:

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

Функция `React.createElement` создает объект, описывающий, что должно быть отображено на экране.

## Создание элементов с JSX

JSX позволяет создавать React-элементы с использованием HTML-подобного синтаксиса:

```jsx
// Простой элемент
const heading = <h1>Заголовок</h1>;

// Элемент с дочерними элементами
const container = (
  <div>
    <h1>Заголовок</h1>
    <p>Параграф</p>
  </div>
);

// Вызов компонента
const app = <MyComponent name="Пример" />;
```

## Выражения в JSX

В JSX можно вставлять JavaScript-выражения, заключая их в фигурные скобки `{}`:

```jsx
const name = 'Алексей';
const element = <h1>Привет, {name}!</h1>;

// Выражения могут быть сложными
const element2 = <p>Результат: {2 + 2}</p>;
const element3 = <div>{user.isLoggedIn ? <span>Вошел</span> : <span>Не вошел</span>}</div>;
```

## Атрибуты в JSX

Атрибуты в JSX следуют правилам JavaScript, а не HTML:
- Используется camelCase: `className`, `htmlFor`, `tabIndex`
- Значения могут быть строками или выражениями в фигурных скобках
- Некоторые атрибуты отличаются от HTML: `class` → `className`, `for` → `htmlFor`

```jsx
// Атрибут как строка
<img src="avatar.png" alt="Аватар" />

// Атрибут из переменной
const imageUrl = 'profile.jpg';
<img src={imageUrl} alt="Профиль" />

// Условный атрибут
<div className={isActive ? 'active' : 'inactive'}>
  Контент
</div>
```

## Дочерние элементы в JSX

JSX поддерживает вложенные элементы, которые становятся дочерними для родительского компонента:

```jsx
// Простые дочерние элементы
const card = (
  <div>
    <h2>Заголовок</h2>
    <p>Описание</p>
  </div>
);

// Дочерние элементы из компонентов
const app = (
  <div>
    <Header />
    <MainContent />
    <Footer />
  </div>
);
```

## Условный рендеринг с JSX

JSX поддерживает различные способы условного рендеринга:

```jsx
// С использованием логического И
{isLoggedIn && <UserGreeting />}

// С использованием тернарного оператора
{isLoggedIn ? <UserGreeting /> : <GuestGreeting />}

// С использованием функций
function renderContent() {
  if (isLoading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  return <div>Данные загружены</div>;
}

const element = <div>{renderContent()}</div>;
```

## Списки и ключи в JSX

При отображении списков важно использовать атрибут `key` для эффективного обновления:

```jsx
const items = ['яблоко', 'банан', 'апельсин'];

const listItems = items.map((item, index) => 
  <li key={index}>{item}</li>
);

// Лучше использовать уникальный идентификатор
const todoItems = todos.map(todo =>
  <TodoItem key={todo.id} todo={todo} />
);
```

## Ограничения JSX

JSX имеет несколько важных ограничений:
- Только один корневевой элемент разрешен (используйте [[react/patterns/fragments|фрагменты]] для обхода)
- Некоторые слова являются зарезервированными (например, `class`, `for`)
- Невозможно использовать JavaScript-комментарии внутри JSX (используйте `{/* комментарий */}`)

## Рассмотрения безопасности с JSX

React автоматически экранирует значения, вставленные в JSX, предотвращая XSS-атаки:

```jsx
// Безопасно, даже если userComment содержит HTML
const element = <div>{userComment}</div>;
```

Однако при использовании `dangerouslySetInnerHTML` нужно быть осторожным, так как это позволяет вставлять "сырой" HTML.

## Рассмотрения производительности

- Избегайте создания новых функций/объектов внутри JSX в циклах
- Используйте `React.memo` для предотвращения ненужных перерендеров
- Правильно используйте ключи для списков
- Избегайте лишних перерендеров с помощью `useCallback` и `useMemo`

## JSX против Hyperscript

JSX и Hyperscript оба позволяют создавать элементы с помощью JavaScript, но имеют различия:

```jsx
// JSX
const element = <div className="container">Привет</div>;

// Hyperscript
const element = h('div.container', 'Привет');
```

JSX более похож на HTML и широко используется в экосистеме React, в то время как Hyperscript легче и может использоваться без транспиляции.

## Подробности трансформации Babel

Babel преобразует JSX в вызовы `React.createElement`:

```jsx
// До
const element = <MyComponent className="test">Привет</MyComponent>;

// После
const element = React.createElement(
  MyComponent,
  { className: 'test' },
  'Привет'
);
```

Современные версии Babel позволяют использовать новые функции JSX трансформации, которые не требуют импорта React.

## Распространенные шаблоны JSX

### Условные классы
```jsx
<div className={`button ${isActive ? 'active' : ''}`}>Кнопка</div>
```

### Условный рендеринг
```jsx
{items.length > 0 && <List items={items} />}
```

### Спред-атрибуты
```jsx
const props = { className: 'button', onClick: handleClick };
<button {...props}>Кнопка</button>
```

## Спред-атрибуты

JSX поддерживает спред-оператор для передачи атрибутов:

```jsx
const buttonProps = {
  className: 'btn',
  type: 'submit',
  disabled: false
};

<button {...buttonProps}>Отправить</button>;
```

Это особенно полезно при создании компонентов-оберток.

## Фрагменты

Фрагменты позволяют группировать элементы без добавления лишних DOM-узлов:

```jsx
// Синтаксис фрагмента
const Component = () => (
  <>
    <h1>Заголовок</h1>
    <p>Параграф</p>
  </>
);

// Или с явным тегом
const Component = () => (
  <React.Fragment>
    <h1>Заголовок</h1>
    <p>Параграф</p>
  </React.Fragment>
);
```

## Связь с другими файлами React

- [[react/basics/components|Компоненты]] - JSX используется для создания компонентов
- [[react/basics/elements-and-instances|Элементы и экземпляры]] - JSX преобразуется в элементы React
- [[react/patterns/conditional-rendering|Условный рендеринг]] - продвинутые техники условного рендеринга
- [[react/patterns/fragments|Фрагменты]] - альтернатива при необходимости возвращать несколько элементов
- [[react/basics/state-and-props|Состояние и пропсы]] - JSX позволяет передавать данные между компонентами
- [[react/basics/events|События]] - JSX позволяет обрабатывать события