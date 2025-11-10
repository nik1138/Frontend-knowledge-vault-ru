---
aliases: ["React Props", "Свойства React", "Компоненты React"]
tags: [react, props, components, javascript, frontend, best-practices]
---

# Подробное руководство по Props в React

Props (свойства) - это механизм передачи данных от родительского компонента к дочернему в React. Это основной способ настройки компонентов и передачи им информации.

## Что такое Props

Props - это объект, содержащий данные, передаваемые от родительского компонента к дочернему. Props являются **неизменяемыми** (immutable) и передаются только сверху вниз по иерархии компонентов.

```jsx
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

function App() {
  return <Welcome name="Алексей" />;
}
```

## Props vs State

| Props | State |
|-------|-------|
| Передаются от родителя | Управляется компонентом |
| Неизменяемы | Изменяемое |
| Статичны для одного рендеринга | Может изменяться во времени |
| Используются для настройки компонента | Используются для управления данными внутри компонента |

Подробнее о различиях см. [[react/state/state-basics]].

## Props Validation

Валидация props позволяет проверять типы и обязательные поля, предотвращая ошибки в работе компонентов:

```jsx
import PropTypes from 'prop-types';

function UserCard({ name, age, isActive }) {
  return (
    <div className={`user-card ${isActive ? 'active' : ''}`}>
      <h2>{name}</h2>
      <p>Возраст: {age}</p>
    </div>
  );
}

UserCard.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isActive: PropTypes.bool
};

UserCard.defaultProps = {
  age: 0,
  isActive: false
};
```

## DefaultProps

DefaultProps позволяют задавать значения по умолчанию для props:

```jsx
function Button({ text, variant = 'primary', disabled = false }) {
  return (
    <button 
      className={`btn btn-${variant}`} 
      disabled={disabled}
    >
      {text}
    </button>
  );
}

// Или с использованием defaultProps:
Button.defaultProps = {
  variant: 'primary',
  disabled: false
};
```

## Обработка Props в функциональных и классовых компонентах

### Функциональные компоненты

```jsx
function Greeting({ name, greeting = 'Привет' }) {
  return <div>{greeting}, {name}!</div>;
}
```

### Классовые компоненты

```jsx
class Greeting extends React.Component {
  render() {
    const { name, greeting = 'Привет' } = this.props;
    return <div>{greeting}, {name}!</div>;
  }
}
```

## Props Spreading

Оператор spread позволяет передавать все свойства объекта как props:

```jsx
const user = { name: 'Мария', age: 25, city: 'Москва' };

function UserProfile(props) {
  return (
    <div>
      <p>Имя: {props.name}</p>
      <p>Возраст: {props.age}</p>
      <p>Город: {props.city}</p>
    </div>
  );
}

// Использование:
<UserProfile {...user} />
```

## Children Props

Children - специальный prop, содержащий дочерние элементы компонента:

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

// Использование:
<Card title="Заголовок">
  <p>Это дочерний контент</p>
  <button>Кнопка</button>
</Card>
```

## Render Props

Паттерн Render Props позволяет делиться кодом между компонентами с помощью функции prop:

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (event) => {
      setPosition({ x: event.clientX, y: event.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Использование:
<MouseTracker render={(position) => (
  <div>Позиция мыши: {position.x}, {position.y}</div>
)} />
```

## Условные Props

Условные props позволяют динамически передавать свойства:

```jsx
function DynamicButton({ isLoading, ...props }) {
  return (
    <button
      {...props}
      disabled={isLoading || props.disabled}
      className={`btn ${isLoading ? 'loading' : ''} ${props.className || ''}`}
    >
      {isLoading ? 'Загрузка...' : props.children}
    </button>
  );
}
```

## Трансформация Props

Иногда нужно преобразовать props перед передачей:

```jsx
function UserList({ users, formatName = (name) => name }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {formatName(user.name)} - {user.email}
        </li>
      ))}
    </ul>
  );
}

// Использование:
<UserList 
  users={users} 
  formatName={(name) => name.toUpperCase()} 
/>
```

## Решения проблемы Prop Drilling

Prop Drilling - это передача props через несколько уровней компонентов. Решения:

1. **React Context** - [[react/context/context-overview]]
2. **Redux/MobX** - [[react/state/redux-integration]]
3. **Custom hooks** - [[react/hooks/custom-hooks]]

## Соглашения об именовании Props

- Используйте camelCase для имен props
- Предпочтительно использовать существительные
- Для boolean props используйте префиксы: `is`, `has`, `can`, `should`
- Для функций используйте префиксы: `on`, `handle`

```jsx
function Modal({ 
  isOpen,           // boolean
  hasCloseButton,   // boolean
  canClose,         // boolean
  shouldAnimate,    // boolean
  onClose,          // function
  onConfirm         // function
}) {
  // ...
}
```

## Рассмотрение производительности с Props

- Избегайте создания новых объектов/функций в props при каждом рендеринге
- Используйте `React.memo` для оптимизации компонентов
- Используйте `useCallback` и `useMemo` для стабильных функций и значений

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // Плохо - новая функция при каждом рендеринге
  // return <Child onClick={() => console.log('click')} />;
  
  // Хорошо - стабильная функция
  const handleClick = useCallback(() => {
    console.log('click');
  }, []);
  
  return <Child count={count} onClick={handleClick} />;
}
```

## Библиотека PropTypes

PropTypes помогает находить ошибки типов в props:

```jsx
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,
  requiredFunc: PropTypes.func.isRequired,
  requiredAny: PropTypes.any.isRequired
};
```

## TypeScript с Props

TypeScript обеспечивает строгую типизацию props:

```tsx
interface UserCardProps {
  name: string;
  age?: number;
  isActive: boolean;
  onEdit: (id: string) => void;
}

const UserCard: React.FC<UserCardProps> = ({ name, age = 0, isActive, onEdit }) => {
  return (
    <div className={isActive ? 'active' : ''}>
      <h2>{name}</h2>
      {age > 0 && <p>Возраст: {age}</p>}
    </div>
  );
};
```

## Тестирование компонентов с Props

При тестировании компонентов важно проверить различные комбинации props:

```jsx
// Пример теста с Jest и React Testing Library
import { render, screen } from '@testing-library/react';

test('отображает имя пользователя', () => {
  render(<UserCard name="Анна" />);
  expect(screen.getByText(/Анна/i)).toBeInTheDocument();
});

test('отображает статус активности', () => {
  render(<UserCard name="Анна" isActive={true} />);
  expect(screen.getByRole('status')).toHaveClass('active');
});
```

## Связи с другими файлами React

- [[react/components/component-patterns]] - шаблоны компонентов
- [[react/components/controlled-vs-uncontrolled]] - управляемые и неуправляемые компоненты
- [[react/hooks/using-props-with-hooks]] - использование props с хуками
- [[react/fundamentals/react-lifecycle]] - жизненный цикл компонентов
- [[react/typescript/typing-components]] - типизация компонентов в TypeScript
- [[react/context/context-patterns]] - использование контекста как альтернативы props drilling

Props являются основой взаимодействия между компонентами в React и понимание их правильного использования критично для создания эффективных и поддерживаемых приложений.