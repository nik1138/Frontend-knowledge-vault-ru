---
title: React Props - Подробное Руководство
date: 2025-11-10
tags: [react, props, components, javascript, frontend]
---

# React Props - Подробное Руководство

Props (сокращение от properties) - это основной механизм передачи данных между компонентами в React. Это неизменяемые (immutable) значения, которые передаются от родительского компонента к дочернему.

## Что такое Props

Props - это объект, содержащий данные, передаваемые компоненту при его вызове. Это основной способ передачи информации от родительского компонента к дочернему. Props позволяют создавать переиспользуемые и гибкие компоненты.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Использование компонента с передачей props
<Welcome name="Алиса" />
```

## Props vs State

| Props | State |
|-------|-------|
| Передаются от родителя к потомку | Управляется внутри компонента |
| Неизменяемы (immutable) | Изменяемое состояние компонента |
| Определяются при вызове компонента | Управляется с помощью `useState` или `this.setState` |
| Позволяют настраивать компонент | Позволяет компоненту изменяться во времени |

```jsx
function UserProfile({ name }) { // name - это prop
  const [age, setAge] = useState(25); // age - это state
  
  return (
    <div>
      <p>Имя: {name}</p> {/* Используем prop */}
      <p>Возраст: {age}</p> {/* Используем state */}
      <button onClick={() => setAge(age + 1)}>Старше</button>
    </div>
  );
}
```

## Передача Props

Props передаются компоненту как атрибуты при его вызове:

```jsx
function Card({ title, content, isHighlighted }) {
  return (
    <div className={isHighlighted ? 'highlighted' : ''}>
      <h2>{title}</h2>
      <p>{content}</p>
    </div>
  );
}

// Передача различных типов props
<Card 
  title="Заголовок карточки"
  content="Содержимое карточки"
  isHighlighted={true}
  count={5}
/>
```

## Default Props

Значения по умолчанию для props можно задать несколькими способами:

```jsx
// Способ 1: Деструктуризация с значениями по умолчанию
function Button({ text = "Кнопка", type = "button", onClick = () => {} }) {
  return <button type={type} onClick={onClick}>{text}</button>;
}

// Способ 2: Свойство defaultProps (для функциональных компонентов)
function Input({ placeholder, value }) {
  return <input placeholder={placeholder} value={value} />;
}

Input.defaultProps = {
  placeholder: "Введите значение",
  value: ""
};
```

## Props Validation

Валидация props помогает избежать ошибок при передаче неправильных данных:

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, email }) {
  return (
    <div>
      <p>Имя: {name}</p>
      <p>Возраст: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string
};

// Для TypeScript можно использовать интерфейсы:
interface UserCardProps {
  name: string;
  age?: number;
  email: string;
}
```

## Children Props

Специальный prop `children` позволяет передавать дочерние элементы компоненту:

```jsx
function Panel({ children, title }) {
  return (
    <div className="panel">
      <h2>{title}</h2>
      <div className="content">
        {children} {/* Рендерим дочерние элементы */}
      </div>
    </div>
  );
}

// Использование:
<Panel title="Моя панель">
  <p>Это дочерний элемент</p>
  <button>Кнопка</button>
</Panel>
```

## Spreading Props

Оператор расширения (...) позволяет передавать объект как набор props:

```jsx
const user = {
  name: "Иван",
  age: 30,
  email: "ivan@example.com"
};

// Вместо передачи каждого prop по отдельности:
<UserCard name={user.name} age={user.age} email={user.email} />

// Можно использовать spread-оператор:
<UserCard {...user} />
```

## Custom Props

Кроме стандартных данных, можно передавать функции, объекты и другие компоненты:

```jsx
function List({ items, renderItem, onItemClick }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index} onClick={() => onItemClick(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Использование с кастомными props
<List 
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  onItemClick={(user) => console.log('Выбран пользователь:', user)}
/>
```

## Conditional Rendering with Props

Props можно использовать для условного рендеринга элементов:

```jsx
function Notification({ isVisible, message, type }) {
  if (!isVisible) return null;
  
  return (
    <div className={`notification ${type}`}>
      {message}
    </div>
  );
}

// Использование:
<Notification 
  isVisible={showNotification} 
  message="Успешно сохранено!" 
  type="success" 
/>
```

## Transforming Props

Иногда нужно трансформировать props перед использованием:

```jsx
function FormattedDate({ date, format = 'long' }) {
  const formatDate = (dateString) => {
    const date = new Date(dateString);
    return format === 'short' 
      ? date.toLocaleDateString() 
      : date.toLocaleString();
  };
  
  return <span>{formatDate(date)}</span>;
}
```

## Props Drilling

Props drilling - это ситуация, когда props передаются через несколько уровней компонентов, даже если промежуточные компоненты не используют эти данные:

```jsx
// Проблема: user передается через несколько уровней
function App() {
  const user = { name: "Алекс", id: 1 };
  return <Header user={user} />;
}

function Header({ user }) {
  return <Navigation user={user} />; // Header не использует user
}

function Navigation({ user }) {
  return <UserProfile user={user} />; // Navigation не использует user
}

function UserProfile({ user }) {
  return <span>{user.name}</span>; // Только здесь используется user
}
```

## Alternatives to Props Drilling

### Context API

```jsx
import { createContext, useContext } from 'react';

const UserContext = createContext();

function App() {
  const user = { name: "Алекс", id: 1 };
  return (
    <UserContext.Provider value={user}>
      <Header />
    </UserContext.Provider>
  );
}

function UserProfile() {
  const user = useContext(UserContext);
  return <span>{user.name}</span>;
}
```

### State Management Libraries

Для сложных приложений можно использовать библиотеки управления состоянием:
- [[react/ecosystem/redux]]
- [[react/ecosystem/zustand]]
- [[react/ecosystem/mobx]]

## Functional vs Class Components Props

### Functional Components
```jsx
function MyComponent({ name, age }) {
  return <div>Привет, {name}, возраст: {age}</div>;
}
```

### Class Components
```jsx
class MyComponent extends React.Component {
  render() {
    const { name, age } = this.props;
    return <div>Привет, {name}, возраст: {age}</div>;
  }
}
```

## Prop Types

Для проверки типов props можно использовать PropTypes или TypeScript:

```jsx
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isActive: PropTypes.bool,
  tags: PropTypes.arrayOf(PropTypes.string),
  onClick: PropTypes.func,
  config: PropTypes.shape({
    theme: PropTypes.string,
    size: PropTypes.oneOf(['small', 'medium', 'large'])
  })
};
```

## Performance Considerations

### 1. Избегайте создания новых объектов/функций в рендере
```jsx
// Плохо - создает новый объект при каждом рендере
<UserCard user={{ name: "Иван", age: 30 }} />

// Хорошо - объект создается один раз
const user = { name: "Иван", age: 30 };
<UserCard user={user} />
```

### 2. Memoization
```jsx
import { memo } from 'react';

const ExpensiveComponent = memo(({ data }) => {
  return <div>{data.value}</div>;
});
```

### 3. Избегайте ненужных изменений props
```jsx
// Плохо - каждый рендер создает новую функцию
<MyComponent onClick={() => console.log('click')} />

// Хорошо - используем useCallback
const handleClick = useCallback(() => {
  console.log('click');
}, []);

<MyComponent onClick={handleClick} />
```

## Связь с другими файлами

- [[react/basics/components]] - Базовые компоненты React
- [[react/basics/state]] - Управление состоянием компонентов
- [[react/context/context-api]] - Context API для избежания props drilling
- [[react/basics/jsx]] - Синтаксис JSX
- [[react/typescript/typing-props]] - Типизация props в TypeScript
- [[react/performance/optimization]] - Оптимизация производительности

## Ключевые выводы

1. Props - это неизменяемые данные, передаваемые от родителя к потомку
2. Используйте деструктуризацию для удобной работы с props
3. Всегда валидируйте props для предотвращения ошибок
4. Используйте Context API или state management библиотеки для избежания props drilling
5. Обращайте внимание на производительность при передаче функций и объектов
6. Помните о различии между props и state

Props являются фундаментальной частью архитектуры React и понимание их правильного использования критически важно для разработки эффективных и поддерживаемых приложений.