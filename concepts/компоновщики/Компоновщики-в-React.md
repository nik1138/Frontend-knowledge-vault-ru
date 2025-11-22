---
aliases: ["Компоновщики в React", "Composite Pattern React", "React Composite"]
tags: ["#react", "#javascript", "#design-patterns", "#frontend", "#component-architecture"]
---

# Компоновщики в React

Паттерн **Компоновщик** в React является фундаментальной частью архитектуры фреймворка. React построен на концепции компонентов, которые могут быть как простыми (атомарными), так и составными (содержащими другие компоненты), образуя древовидную структуру пользовательского интерфейса.

## Основы компоновщика в React

В React каждый компонент может содержать другие компоненты, создавая иерархию, аналогичную паттерну Компоновщик. Это позволяет:

- Строить сложные интерфейсы из простых компонентов
- Повторно использовать компоненты
- Управлять состоянием на разных уровнях иерархии
- Создавать гибкие и масштабируемые интерфейсы

## Типы компонентов в React

### Функциональные компоненты

```jsx
// Листовой компонент
function Button({ text, onClick }) {
  return <button onClick={onClick}>{text}</button>;
}

// Составной компонент
function ButtonGroup({ buttons }) {
  return (
    <div className="button-group">
      {buttons.map((btn, index) => (
        <Button key={index} text={btn.text} onClick={btn.onClick} />
      ))}
    </div>
  );
}
```

### Классовые компоненты

```jsx
class Card extends React.Component {
  render() {
    const { title, children } = this.props;
    return (
      <div className="card">
        <h3>{title}</h3>
        <div className="card-content">
          {children} {/* Слот для вложенных компонентов */}
        </div>
      </div>
    );
  }
}

class Dashboard extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      widgets: []
    };
  }

  render() {
    return (
      <div className="dashboard">
        {this.state.widgets.map((widget, index) => (
          <Card key={index} title={widget.title}>
            {widget.content}
          </Card>
        ))}
      </div>
    );
  }
}
```

## Использование children

Одной из ключевых особенностей React, поддерживающей паттерн Компоновщик, является использование `props.children`:

```jsx
function Layout({ children }) {
  return (
    <div className="layout">
      <Header />
      <main className="content">
        {children} {/* Здесь будут вложенные компоненты */}
      </main>
      <Footer />
    </div>
  );
}

function App() {
  return (
    <Layout>
      <div>Содержимое приложения</div>
      <Button text="Нажми меня" />
    </Layout>
  );
}
```

## Композиция компонентов

React поощряет композицию компонентов вместо наследования:

```jsx
// Вместо наследования
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <button className="close-button" onClick={onClose}>×</button>
        <div className="modal-content">
          {children}
        </div>
      </div>
    </div>
  );
}

function ConfirmModal({ isOpen, onConfirm, onCancel, message }) {
  return (
    <Modal isOpen={isOpen} onClose={onCancel}>
      <p>{message}</p>
      <div className="modal-actions">
        <Button text="Подтвердить" onClick={onConfirm} />
        <Button text="Отмена" onClick={onCancel} />
      </div>
    </Modal>
  );
}
```

## Пример сложной композиции

```jsx
// Базовый компонент списка
function List({ items, renderItem }) {
  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={index} className="list-item">
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Компонент элемента списка
function ListItem({ item }) {
  return (
    <div className="list-item-content">
      <h4>{item.title}</h4>
      <p>{item.description}</p>
    </div>
  );
}

// Компонент для отображения списка с фильтрацией
function FilterableList({ items, filter }) {
  const filteredItems = items.filter(item => 
    item.title.toLowerCase().includes(filter.toLowerCase())
  );
  
  return (
    <div className="filterable-list">
      <input 
        type="text" 
        placeholder="Фильтр..." 
        onChange={(e) => /* обработка изменения фильтра */}
      />
      <List 
        items={filteredItems} 
        renderItem={(item) => <ListItem item={item} />}
      />
    </div>
  );
}
```

## Паттерны композиции в React

### Container/Presenter паттерн

```jsx
// Контейнерный компонент (содержит логику)
class UserProfileContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      loading: true
    };
  }

  componentDidMount() {
    this.fetchUser();
  }

  fetchUser = async () => {
    try {
      const user = await api.getUser(this.props.userId);
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ loading: false, error });
    }
  };

  render() {
    return (
      <UserProfilePresenter 
        user={this.state.user} 
        loading={this.state.loading}
        error={this.state.error}
      />
    );
  }
}

// Презентационный компонент (только отображение)
function UserProfilePresenter({ user, loading, error }) {
  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  if (!user) return <div>Пользователь не найден</div>;

  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### Higher-Order Components (HOC)

```jsx
// HOC для добавления логики загрузки
function withLoading(WrappedComponent) {
  return function WithLoadingComponent(props) {
    if (props.loading) {
      return <div>Загрузка...</div>;
    }
    return <WrappedComponent {...props} />;
  };
}

// Использование HOC
const UserCardWithLoading = withLoading(UserCard);
```

## Hooks и композиция

С появлением хуков в React появилась новая парадигма для композиции логики:

```jsx
// Кастомный хук для управления формой
function useForm(initialValues) {
  const [values, setValues] = React.useState(initialValues);
  
  return {
    values,
    handleChange: (name, value) => {
      setValues(prev => ({ ...prev, [name]: value }));
    },
    reset: () => setValues(initialValues)
  };
}

// Компонент формы, использующий кастомный хук
function ContactForm() {
  const { values, handleChange, reset } = useForm({
    name: '',
    email: '',
    message: ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    // Отправка формы
    reset();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={values.name}
        onChange={(e) => handleChange('name', e.target.value)}
        placeholder="Имя"
      />
      <input
        type="email"
        name="email"
        value={values.email}
        onChange={(e) => handleChange('email', e.target.value)}
        placeholder="Email"
      />
      <textarea
        name="message"
        value={values.message}
        onChange={(e) => handleChange('message', e.target.value)}
        placeholder="Сообщение"
      />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

## Преимущества использования паттерна Компоновщик в React

- **Повторное использование кода**: компоненты можно использовать в разных частях приложения
- **Легкость тестирования**: отдельные компоненты легче тестировать
- **Модульность**: каждая часть интерфейса изолирована
- **Гибкость**: легко изменять структуру интерфейса

## Практические рекомендации

1. **Используйте семантическую композицию** - компоненты должны иметь понятное назначение
2. **Ограничьте глубину вложенности** - слишком глубокие иерархии сложны в поддержке
3. **Используйте TypeScript** для лучшей типизации компонентов
4. **Разделяйте ответственность** - контейнерные и презентационные компоненты

## Связь с другими концепциями

- [[Компонентная-архитектура]] - архитектурный подход
- [[Принципы-программирования]] - основы проектирования
- [[Жизненный-цикл-компонента]] - управление состоянием
- [[Хуки]] - современный способ управления логикой
- [[Паттерн-компоновщик]] - теоретическая основа

## Заключение

Паттерн Компоновщик в React реализуется естественным образом через композицию компонентов. Это делает React мощным инструментом для создания сложных и гибких пользовательских интерфейсов, где каждый компонент может быть как простым элементом, так и контейнером для других компонентов.

> [!tip]
> Используйте композицию компонентов для создания гибких и повторно используемых интерфейсов в React.

> [!warning]
> Избегайте слишком глубоких иерархий компонентов, которые могут ухудшить производительность и читаемость кода.
