---
aliases: ["Полиморфизм в React", "React Полиморфизм"]
tags: ["#react", "#polymorphism", "#frontend", "#components"]
---

# Полиморфизм-в-React

## Введение

Полиморфизм в React — это способность компонентов принимать разные формы или типы данных, сохраняя при этом одинаковый интерфейс взаимодействия. В контексте React полиморфизм проявляется в различных формах: через компоненты высшего порядка (HOC), рендер-пропсы, хуки и дженерики (в TypeScript).

Полиморфизм позволяет создавать более гибкие и переиспользуемые компоненты, которые могут работать с различными типами данных или отображать разное содержимое в зависимости от переданных пропсов.

## Типы полиморфизма в React

### 1. Компонентный полиморфизм

Один из наиболее распространенных способов реализации полиморфизма в React — это создание компонентов, которые могут принимать разные типы элементов или компонентов в качестве пропсов.

```jsx
// Пример полиморфного компонента Button
const Button = ({ as: Component = 'button', children, variant = 'primary', ...props }) => {
  const baseClasses = 'btn';
  const variantClass = `btn-${variant}`;
  
  return (
    <Component className={`${baseClasses} ${variantClass}`} {...props}>
      {children}
    </Component>
  );
};

// Использование с разными элементами
const App = () => (
  <div>
    <Button as="button" variant="primary">Обычная кнопка</Button>
    <Button as="a" href="/link" variant="secondary">Ссылка-кнопка</Button>
    <Button as={Link} to="/react-link" variant="success">React Router Link</Button>
  </div>
);
```

### 2. Рендер-пропсы (Render Props)

Рендер-пропсы позволяют передавать функцию, которая возвращает JSX, что обеспечивает полиморфное поведение компонентов.

```jsx
// Компонент с рендер-пропсом для управления состоянием
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isToggleOn: false };
  }

  handleClick = () => {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        {this.props.render(this.state.isToggleOn)}
      </div>
    );
  }
}

// Использование с разными визуальными представлениями
const App = () => (
  <div>
    <Toggle render={(isOn) => (
      <div>{isOn ? 'ВКЛ' : 'ВЫКЛ'}</div>
    )} />
    
    <Toggle render={(isOn) => (
      <button>{isOn ? 'Свет горит' : 'Свет выключен'}</button>
    )} />
  </div>
);
```

### 3. Компоненты высшего порядка (HOC)

HOC позволяют расширять функциональность компонентов, создавая полиморфные обертки.

```jsx
// HOC для добавления логики загрузки данных
const withData = (WrappedComponent, fetchData) => {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = { data: null, loading: true };
    }

    async componentDidMount() {
      const data = await fetchData();
      this.setState({ data, loading: false });
    }

    render() {
      return (
        <WrappedComponent
          {...this.props}
          data={this.state.data}
          loading={this.state.loading}
        />
      );
    }
  };
};

// Разные компоненты могут использовать один HOC
const UserList = ({ data, loading }) => (
  <div>
    {loading ? 'Загрузка...' : data.map(user => <div key={user.id}>{user.name}</div>)}
  </div>
);

const PostList = ({ data, loading }) => (
  <div>
    {loading ? 'Загрузка...' : data.map(post => <div key={post.id}>{post.title}</div>)}
  </div>
);

const UserListWithData = withData(UserList, () => fetch('/api/users').then(r => r.json()));
const PostListWithData = withData(PostList, () => fetch('/api/posts').then(r => r.json()));
```

### 4. Хуки и полиморфизм

Хуки также могут демонстрировать полиморфизм, особенно когда они принимают разные типы данных или функции.

```jsx
// Хук для управления списком с разными типами элементов
const useList = (initialItems = []) => {
  const [items, setItems] = useState(initialItems);

  const addItem = (item) => {
    setItems(prev => [...prev, item]);
  };

  const removeItem = (index) => {
    setItems(prev => prev.filter((_, i) => i !== index));
  };

  const updateItem = (index, newItem) => {
    setItems(prev => prev.map((item, i) => i === index ? newItem : item));
  };

  return { items, addItem, removeItem, updateItem };
};

// Использование хука с разными типами данных
const TodoApp = () => {
  const todoList = useList([]);
  const userList = useList([]);

  return (
    <div>
      <h2>Список задач</h2>
      {/* Работает с задачами */}
      {todoList.items.map((todo, i) => <div key={i}>{todo}</div>)}
      
      <h2>Список пользователей</h2>
      {/* Работает с пользователями */}
      {userList.items.map((user, i) => <div key={i}>{user.name}</div>)}
    </div>
  );
};
```

## Полиморфизм с TypeScript

TypeScript позволяет более строго определить полиморфные компоненты с помощью дженериков.

```tsx
// Полиморфный компонент с дженериком
interface PolymorphicComponentProps<T extends React.ElementType> {
  as?: T;
  children: React.ReactNode;
}

function PolymorphicDiv<T extends React.ElementType = 'div'>(
  { as, children, ...props }: PolymorphicComponentProps<T> & 
  React.ComponentPropsWithoutRef<T>
) {
  const Component = as || 'div';
  return <Component {...props as any}>{children}</Component>;
}

// Использование с разными типами
const App = () => (
  <>
    <PolymorphicDiv as="h1">Заголовок</PolymorphicDiv>
    <PolymorphicDiv as="p">Параграф</PolymorphicDiv>
    <PolymorphicDiv as={Link} to="/link">Ссылка</PolymorphicDiv>
  </>
);
```

## Практические примеры

### 1. Полиморфный список

```jsx
// Универсальный компонент списка
const List = ({ items, renderItem, emptyMessage = "Нет элементов" }) => {
  if (!items || items.length === 0) {
    return <div>{emptyMessage}</div>;
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
};

// Использование с разными типами данных
const UserList = ({ users }) => (
  <List 
    items={users}
    renderItem={(user) => <span>{user.name} ({user.email})</span>}
    emptyMessage="Нет пользователей"
  />
);

const ProductList = ({ products }) => (
  <List 
    items={products}
    renderItem={(product) => (
      <div>
        <h3>{product.name}</h3>
        <p>Цена: {product.price}</p>
      </div>
    )}
    emptyMessage="Нет товаров"
  />
);
```

### 2. Универсальный модальный компонент

```jsx
const Modal = ({ isOpen, onClose, children, title }) => {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose}>×</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
};

// Использование для разных целей
const App = () => {
  const [showUserModal, setShowUserModal] = useState(false);
  const [showProductModal, setShowProductModal] = useState(false);

  return (
    <div>
      <button onClick={() => setShowUserModal(true)}>Показать пользователя</button>
      <button onClick={() => setShowProductModal(true)}>Показать товар</button>

      <Modal 
        isOpen={showUserModal} 
        onClose={() => setShowUserModal(false)}
        title="Информация о пользователе"
      >
        <UserForm />
      </Modal>

      <Modal 
        isOpen={showProductModal} 
        onClose={() => setShowProductModal(false)}
        title="Информация о товаре"
      >
        <ProductDetails />
      </Modal>
    </div>
  );
};
```

### 3. Полиморфный компонент формы

```jsx
const FormField = ({ 
  type = 'text', 
  label, 
  value, 
  onChange, 
  children,
  ...props 
}) => {
  const inputProps = { type, value, onChange, ...props };

  return (
    <div className="form-field">
      <label>{label}</label>
      {type === 'textarea' ? (
        <textarea {...inputProps} value={value} onChange={onChange} />
      ) : type === 'select' ? (
        <select {...inputProps} value={value} onChange={onChange}>
          {children}
        </select>
      ) : type === 'checkbox' ? (
        <input {...inputProps} checked={value} onChange={onChange} />
      ) : (
        <input {...inputProps} />
      )}
    </div>
  );
};

// Использование для разных типов полей
const ContactForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: '',
    subscribe: false
  });

  const handleChange = (field) => (e) => {
    setFormData(prev => ({
      ...prev,
      [field]: e.target.type === 'checkbox' ? e.target.checked : e.target.value
    }));
  };

  return (
    <form>
      <FormField 
        label="Имя" 
        type="text" 
        value={formData.name} 
        onChange={handleChange('name')} 
      />
      
      <FormField 
        label="Email" 
        type="email" 
        value={formData.email} 
        onChange={handleChange('email')} 
      />
      
      <FormField 
        label="Сообщение" 
        type="textarea" 
        value={formData.message} 
        onChange={handleChange('message')} 
      />
      
      <FormField 
        label="Подписаться" 
        type="checkbox" 
        value={formData.subscribe} 
        onChange={handleChange('subscribe')} 
      />
    </form>
  );
};
```

## Преимущества полиморфизма в React

1. **Повторное использование компонентов** - один компонент может использоваться в разных контекстах
2. **Гибкость** - возможность адаптировать компоненты под разные сценарии использования
3. **Снижение дублирования кода** - уменьшение необходимости создания множества похожих компонентов
4. **Улучшенная тестируемость** - возможность тестировать компоненты с разными типами данных

## Заключение

Полиморфизм в React позволяет создавать более гибкие и переиспользуемые компоненты, которые могут адаптироваться к различным сценариям использования. Это достигается через различные паттерны: компонентный полиморфизм, рендер-пропсы, HOC, хуки и дженерики в TypeScript.

Правильное использование полиморфизма делает код более чистым, понятным и поддерживаемым, позволяя создавать мощные и гибкие UI-библиотеки.

## См. также

- [[Компонентный подход]]
- [[Композиция и агрегация]]
- [[Хуки]]
- [[TypeScript]]
- [[Дизайн-системы]]