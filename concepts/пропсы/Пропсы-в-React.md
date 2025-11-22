---
aliases: ["Props в React", "Свойства в React", "React Props"]
tags: [frontend, react, props, javascript, component-architecture, jsx]
---

# Пропсы в React

**Пропсы в React** (React props) — это механизм передачи данных от родительского компонента к дочернему. Props (сокращение от properties) представляют собой неизменяемые (immutable) значения, которые компонент получает извне и использует для настройки своего поведения или отображения.

## Основные концепции пропсов в React

Пропсы являются основным способом передачи данных в React-компонентах. Они обеспечивают одностороннюю передачу данных от родителя к потомку, что соответствует философии React о предсказуемом потоке данных.

### Простой пример передачи пропсов

```jsx
// Родительский компонент
function App() {
  return (
    <div>
      <Welcome name="Иван" age={25} />
      <Welcome name="Мария" age={30} />
    </div>
  );
}

// Дочерний компонент
function Welcome(props) {
  return <h1>Привет, {props.name}! Тебе {props.age} лет.</h1>;
}
```

## Виды пропсов в React

### 1. Примитивные пропсы

```jsx
function Greeting({ name, age, isActive }) {
  return (
    <div>
      <p>Имя: {name}</p>
      <p>Возраст: {age}</p>
      <p>Статус: {isActive ? 'Активен' : 'Неактивен'}</p>
    </div>
  );
}

// Использование
<Greeting name="Алексей" age={35} isActive={true} />
```

### 2. Объектные пропсы

```jsx
function UserProfile({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
      <p>Возраст: {user.age}</p>
    </div>
  );
}

// Использование
<UserProfile 
  user={{ 
    name: 'Елена', 
    email: 'elena@example.com', 
    age: 28 
  }} 
/>
```

### 3. Функциональные пропсы (колбэки)

```jsx
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

function Parent() {
  const handleClick = () => {
    console.log('Кнопка нажата!');
  };

  return <Button onClick={handleClick}>Нажми меня</Button>;
}
```

### 4. Пропсы-дети (children)

Специальный пропс `children` позволяет передавать содержимое между открывающим и закрывающим тегами компонента:

```jsx
function Card({ children, title }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Использование
<Card title="Информация">
  <p>Это содержимое будет передано через пропс children</p>
  <ul>
    <li>Элемент 1</li>
    <li>Элемент 2</li>
  </ul>
</Card>
```

## Валидация пропсов в React

### PropTypes

React предоставляет вспомогательную библиотеку PropTypes для проверки типов пропсов:

```jsx
import PropTypes from 'prop-types';

function ProductCard({ name, price, category, isAvailable, onAddToCart }) {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>Цена: {price}$</p>
      <p>Категория: {category}</p>
      <p>Доступен: {isAvailable ? 'Да' : 'Нет'}</p>
      <button onClick={onAddToCart}>Добавить в корзину</button>
    </div>
  );
}

ProductCard.propTypes = {
  name: PropTypes.string.isRequired,
  price: PropTypes.number.isRequired,
  category: PropTypes.oneOf(['electronics', 'clothing', 'books']).isRequired,
  isAvailable: PropTypes.bool,
  onAddToCart: PropTypes.func
};

ProductCard.defaultProps = {
  isAvailable: true,
  onAddToCart: () => {}
};
```

### TypeScript

Современный подход к валидации пропсов в React — использование TypeScript:

```tsx
interface ProductCardProps {
  name: string;
  price: number;
  category: 'electronics' | 'clothing' | 'books';
  isAvailable?: boolean;
  onAddToCart?: () => void;
}

const ProductCard: React.FC<ProductCardProps> = ({ 
  name, 
  price, 
  category, 
  isAvailable = true, 
  onAddToCart = () => {} 
}) => {
  return (
    <div className="product-card">
      <h3>{name}</h3>
      <p>Цена: {price}$</p>
      <p>Категория: {category}</p>
      <p>Доступен: {isAvailable ? 'Да' : 'Нет'}</p>
      <button onClick={onAddToCart}>Добавить в корзину</button>
    </div>
  );
};
```

## Распространенные паттерны использования пропсов

### 1. Компоненты высшего порядка (HOC)

```jsx
function withLoading(WrappedComponent) {
  return function EnhancedComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Загрузка...</div>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Использование
const UserCardWithLoading = withLoading(UserCard);
<UserCardWithLoading user={userData} isLoading={loading} />;
```

### 2. Render Props

```jsx
function DataProvider({ render, url }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(setData)
      .finally(() => setLoading(false));
  }, [url]);

  return render({ data, loading });
}

// Использование
<DataProvider 
  url="/api/users" 
  render={({ data, loading }) => 
    loading ? <div>Загрузка...</div> : <UserList users={data} />
  } 
/>
```

### 3. Compound Components

```jsx
function Toggle({ children, onToggle, defaultActive = false }) {
  const [active, setActive] = useState(defaultActive);

  const toggleContext = {
    active,
    onToggle: () => {
      setActive(!active);
      onToggle && onToggle(!active);
    }
  };

  return (
    <ToggleContext.Provider value={toggleContext}>
      {children}
    </ToggleContext.Provider>
  );
}

function ToggleButton() {
  const { active, onToggle } = useContext(ToggleContext);
  return <button onClick={onToggle}>{active ? 'Выключить' : 'Включить'}</button>;
}

function ToggleContent({ children }) {
  const { active } = useContext(ToggleContext);
  return active ? <div>{children}</div> : null;
}
```

## Пропсы и производительность

### Использование useMemo с пропсами

```jsx
function ExpensiveComponent({ items, filter }) {
  // Избегаем пересчета при каждом рендере
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### Предотвращение ненужных перерендеров

```jsx
const MemoizedChild = React.memo(function Child({ value, onUpdate }) {
  console.log('Рендер Child компонента');
  return <div onClick={() => onUpdate(value + 1)}>{value}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState('');

  return (
    <div>
      <p>Счетчик: {count}</p>
      <p>Другое состояние: {otherState}</p>
      {/* Этот компонент не будет перерендериваться при изменении otherState */}
      <MemoizedChild 
        value={count} 
        onUpdate={setCount} 
      />
      <input 
        value={otherState} 
        onChange={(e) => setOtherState(e.target.value)} 
      />
    </div>
  );
}
```

## Специальные пропсы в React

### spread-оператор для пропсов

```jsx
function App() {
  const buttonProps = {
    className: 'btn btn-primary',
    type: 'button',
    disabled: false
  };

  return <Button {...buttonProps}>Нажми меня</Button>;
}
```

### Пропсы для DOM-элементов

```jsx
function CustomInput({ label, ...inputProps }) {
  return (
    <div>
      <label>{label}</label>
      <input {...inputProps} />
    </div>
  );
}

// Все пропсы, кроме label, будут переданы в input
<CustomInput 
  label="Имя пользователя" 
  type="text" 
  placeholder="Введите имя" 
  defaultValue="Алексей" 
/>
```

## Пропсы и жизненный цикл компонента

Пропсы влияют на поведение компонента на всех этапах его жизненного цикла:

```jsx
function UserComponent({ userId, onUpdate }) {
  const [user, setUser] = useState(null);

  // useEffect с зависимостью от пропсов
  useEffect(() => {
    if (userId) {
      fetchUser(userId).then(setUser);
    }
  }, [userId]); // Зависимость от пропса

  // Изменение пропса вызовет повторный вызов эффекта

  return (
    <div>
      {user ? <UserProfile user={user} /> : <div>Загрузка...</div>}
    </div>
  );
}
```

## Проблемы и решения

### 1. "Prop drilling" (сквозная передача пропсов)

Проблема: передача пропсов через несколько уровней компонентов без использования для этого промежуточных компонентов.

```jsx
// Проблема
function App() {
  return <Layout user={user} theme={theme} />;
}

function Layout({ user, theme }) {
  return <Main user={user} theme={theme} />;
}

function Main({ user, theme }) {
  return <Content user={user} theme={theme} />;
}

function Content({ user, theme }) {
  return <UserProfile user={user} theme={theme} />;
}
```

Решение: использование [[Контекст]] или [[Глобальное состояние]].

### 2. Избыточная передача пропсов

```jsx
// Плохо: передаем весь объект, если нужна только часть
<UserCard user={completeUserData} />

// Лучше: деструктурируем и передаем только нужные данные
const { name, email, avatar } = completeUserData;
<UserCard name={name} email={email} avatar={avatar} />
```

## Лучшие практики

1. **Используйте осмысленные имена пропсов**: Это делает код более читаемым
2. **Минимизируйте количество пропсов**: Слишком много пропсов может усложнить использование компонента
3. **Предоставляйте значения по умолчанию**: Для необязательных пропсов
4. **Валидируйте пропсы**: Используйте PropTypes или TypeScript
5. **Документируйте пропсы**: Комментарии и JSDoc помогают другим разработчикам
6. **Используйте деструктуризацию**: Улучшает читаемость кода
7. **Избегайте передачи лишних данных**: Только то, что действительно нужно компоненту

## Связанные темы

- [[Передача-пропсов]]
- [[Валидация-пропсов]]
- [[Пропсы-по-умолчанию]]
- [[Деструктуризация-пропсов]]
- [[Контекст]]
- [[Глобальное состояние]]
- [[Компонентная архитектура]]
- [[Хуки]]

## Заключение

Пропсы являются фундаментальным механизмом передачи данных в React. Понимание их работы, правильное использование и следование лучшим практикам позволяет создавать гибкие, переиспользуемые и легко поддерживаемые компоненты. Это ключевой элемент архитектуры React-приложений.