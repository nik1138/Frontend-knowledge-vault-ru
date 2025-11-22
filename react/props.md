---
aliases: ["React Props", "Свойства React", "Передача данных в React"]
tags: [react, props, component-architecture, best-practices]
---

# React Props - Подробное Руководство

Props (свойства) - это механизм передачи данных от родительского компонента к дочернему в React. Это основной способ создания переиспользуемых компонентов с изменяемым поведением.

## Что такое Props в React

Props - это аргументы, передаваемые компоненту при его вызове. Они позволяют передавать данные, функции и другие значения от родительского компонента к дочернему.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

function App() {
  return <Welcome name="Алиса" />;
}
```

Props неизменяемы (immutable) - компонент не может изменить свои props.

## Props vs State

| Props | State |
|-------|-------|
| Передаются извне | Управляется внутри компонента |
| Неизменяемы | Изменяемые с помощью setState/useState |
| Используются для передачи данных | Используется для хранения данных компонента |
| Определяются родительским компонентом | Определяются внутри компонента |

Подробнее о состоянии см. [[react/state]].

## Передача Props в Компоненты

Props передаются как атрибуты JSX-элемента:

```jsx
function UserCard({ name, email, avatar }) {
  return (
    <div className="user-card">
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}

function App() {
  return (
    <UserCard 
      name="Иван Петров" 
      email="ivan@example.com" 
      avatar="/images/ivan.jpg" 
    />
  );
}
```

## Деструктуризация Props

Деструктуризация позволяет извлекать значения props напрямую:

```jsx
// Без деструктуризации
function Button(props) {
  return <button className={props.className}>{props.children}</button>;
}

// С деструктуризацией
function Button({ className, children, onClick }) {
  return <button className={className} onClick={onClick}>
    {children}
  </button>;
}
```

## Значения Props по Умолчанию

### Для функциональных компонентов:
```jsx
function Greeting({ name = "Гость", greeting = "Привет" }) {
  return <div>{greeting}, {name}!</div>;
}

// Или через объект параметров:
function Greeting({ name, greeting }) {
  const defaultName = name || "Гость";
  const defaultGreeting = greeting || "Привет";
  return <div>{defaultGreeting}, {defaultName}!</div>;
}
```

### Для классовых компонентов:
```jsx
class Welcome extends React.Component {
  static defaultProps = {
    name: "Гость",
    greeting: "Привет"
  };

  render() {
    return <h1>{this.props.greeting}, {this.props.name}!</h1>;
  }
}
```

## Валидация Props

Используйте `PropTypes` для проверки типов props:

```jsx
import PropTypes from 'prop-types';

function ProductCard({ title, price, rating }) {
  return (
    <div>
      <h3>{title}</h3>
      <p>Цена: {price} руб.</p>
      <p>Рейтинг: {rating}/5</p>
    </div>
  );
}

ProductCard.propTypes = {
  title: PropTypes.string.isRequired,
  price: PropTypes.number.isRequired,
  rating: PropTypes.number
};

ProductCard.defaultProps = {
  rating: 0
};
```

С версии React 18.2+ рекомендуется использовать TypeScript для статической проверки типов, подробнее в [[react/typescript]].

## Children Props

Props `children` позволяет передавать контент в компонент:

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

function App() {
  return (
    <Card title="Информация">
      <p>Это дочерний контент</p>
      <button>Кнопка</button>
    </Card>
  );
}
```

## Паттерн Render Props

Render props позволяет передавать функцию в качестве значения props:

```jsx
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };

  render() {
    return (
      <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
        {this.props.render(this.state)}
      </div>
    );
  }
}

function App() {
  return (
    <MouseTracker render={(mouse) => (
      <h1>Мышь на позиции: {mouse.x}, {mouse.y}</h1>
    )} />
  );
}
```

## Условные Props

Передача props на основе условий:

```jsx
function Button({ isActive, ...props }) {
  return (
    <button
      className={`btn ${isActive ? 'btn-active' : 'btn-inactive'}`}
      disabled={!isActive}
      {...props}
    >
      {props.children}
    </button>
  );
}

// Условная передача props:
function App() {
  const [isEnabled, setIsEnabled] = useState(true);
  
  return (
    <Button 
      variant="primary"
      {...(isEnabled && { onClick: handleClick })}
    >
      Кнопка
    </Button>
  );
}
```

## Распространение Props (Props Spreading)

Распространение позволяет передавать все props в дочерний компонент:

```jsx
function CustomInput({ label, ...props }) {
  return (
    <div>
      <label>{label}</label>
      <input {...props} />
    </div>
  );
}

// Использование:
<CustomInput 
  label="Имя" 
  type="text" 
  placeholder="Введите имя" 
  defaultValue="Алексей" 
/>
```

## Компоновка Компонентов с Props

Props позволяют создавать гибкие и переиспользуемые компоненты:

```jsx
function Layout({ header, sidebar, content, footer }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{content}</main>
      <footer>{footer}</footer>
    </div>
  );
}

function App() {
  const header = <Header title="Мой сайт" />;
  const sidebar = <Navigation />;
  const content = <MainContent />;
  const footer = <Footer />;
  
  return <Layout 
    header={header} 
    sidebar={sidebar} 
    content={content} 
    footer={footer} 
  />;
}
```

## Обработка Props в Функциональных и Классовых Компонентах

### Функциональный компонент:
```jsx
function FunctionalComponent({ title, description }) {
  return (
    <div>
      <h1>{title}</h1>
      <p>{description}</p>
    </div>
  );
}
```

### Классовый компонент:
```jsx
class ClassComponent extends React.Component {
  render() {
    return (
      <div>
        <h1>{this.props.title}</h1>
        <p>{this.props.description}</p>
      </div>
    );
  }
}
```

## Рассмотрение Производительности с Props

- Избегайте создания новых объектов/функций в props при каждом рендере
- Используйте `React.memo()` для предотвращения ненужных рендеров
- Используйте `useCallback` и `useMemo` для оптимизации функций и значений

```jsx
const OptimizedComponent = React.memo(({ items, onItemClick }) => {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

// В родительском компоненте:
function Parent({ items }) {
  const handleItemClick = useCallback((item) => {
    // Обработчик
  }, []);

  return <OptimizedComponent items={items} onItemClick={handleItemClick} />;
}
```

## Лучшие Практики для Props

1. **Используйте описательные имена props**
2. **Предоставляйте значения по умолчанию**
3. **Валидируйте props с помощью PropTypes или TypeScript**
4. **Избегайте передачи лишних данных**
5. **Используйте деструктуризацию для улучшения читаемости**
6. **Используйте `React.memo()` для компонентов, чувствительных к производительности**

## Связь с другими файлами React

- [[react/state]] - Управление состоянием компонента
- [[react/lifecycle]] - Жизненный цикл компонентов
- [[Типизация хуков React в TypeScript]] - Хуки для функциональных компонентов
- [[react/component-architecture]] - Архитектура компонентов
- [[react/events]] - Обработка событий
- [[react/typescript]] - Использование TypeScript с React

Props и state работают вместе для управления данными в приложении. Props передают данные сверху вниз, а state управляет внутренними данными компонента.

## Ключевые Выводы

- Props - это неизменяемые данные, передаваемые от родителя к потомку
- Используйте деструктуризацию для удобного доступа к props
- Валидируйте props для повышения надежности приложения
- Оптимизируйте передачу props для лучшей производительности
- Props - основа для создания переиспользуемых и гибких компонентов