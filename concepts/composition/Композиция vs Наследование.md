# Композиция vs Наследование

В объектно-ориентированном программировании и компонентной архитектуре существуют два основных подхода к повторному использованию кода: наследование и композиция. В современном фронтенд-развитии, особенно в React, предпочтение отдается композиции над наследованием.

## Наследование

Наследование - это механизм, при котором один класс (дочерний) наследует свойства и методы другого класса (родительского).

### Пример наследования в JavaScript

```javascript
class Component {
  constructor(props) {
    this.props = props;
  }

  render() {
    throw new Error('Метод render должен быть реализован');
  }
}

class Button extends Component {
  render() {
    return `<button>${this.props.text}</button>`;
  }
}

class IconButton extends Button {
  render() {
    return `<button><img src="${this.props.icon}" /> ${this.props.text}</button>`;
  }
}
```

### Недостатки наследования

1. **Жесткая связь** - дочерний класс жестко связан с родительским
2. **Проблема ромбовидного наследования** - сложности при множественном наследовании
3. **Нарушение инкапсуляции** - внутренние детали родительского класса могут быть доступны
4. **Сложность тестирования** - трудно изолировать компоненты для тестирования
5. **Негибкость** - трудно изменить поведение без изменения иерархии

## Композиция

Композиция - это подход, при котором сложные объекты создаются путем объединения более простых.

### Пример композиции в React

```jsx
// Простой компонент
const Button = ({ children, onClick, type = 'default' }) => (
  <button className={`btn btn-${type}`} onClick={onClick}>
    {children}
  </button>
);

// Композиция компонентов
const IconButton = ({ icon, children, onClick }) => (
  <Button onClick={onClick}>
    <img src={icon} alt="" /> {children}
  </Button>
);

// Использование композиции
const App = () => (
  <div>
    <Button type="primary">Обычная кнопка</Button>
    <IconButton icon="/path/to/icon.png" onClick={() => console.log('click')}>
      Кнопка с иконкой
    </IconButton>
  </div>
);
```

### Преимущества композиции

1. **Гибкость** - компоненты можно легко комбинировать по-разному
2. **Повторное использование** - компоненты можно использовать в разных контекстах
3. **Тестируемость** - проще тестировать отдельные компоненты
4. **Предсказуемость** - легче понять, как компоненты взаимодействуют
5. **Масштабируемость** - проще добавлять новые функции

## Паттерны композиции в React

### 1. Передача дочерних элементов

```jsx
const Card = ({ children, title }) => (
  <div className="card">
    <h2>{title}</h2>
    <div className="card-content">
      {children}
    </div>
  </div>
);

const App = () => (
  <Card title="Моя карточка">
    <p>Содержимое карточки</p>
    <button>Действие</button>
  </Card>
);
```

### 2. Передача компонентов как пропсов

```jsx
const Layout = ({ header, sidebar, content, footer }) => (
  <div className="layout">
    <header>{header}</header>
    <aside>{sidebar}</aside>
    <main>{content}</main>
    <footer>{footer}</footer>
  </div>
);

const App = () => (
  <Layout
    header={<Header />}
    sidebar={<Sidebar />}
    content={<MainContent />}
    footer={<Footer />}
  />
);
```

### 3. Render Props

```jsx
const DataProvider = ({ render, url }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(response => response.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
};

const App = () => (
  <DataProvider
    url="/api/users"
    render={({ data, loading }) =>
      loading ? <div>Загрузка...</div> : <UserList users={data} />
    }
  />
);
```

### 4. Higher-Order Components (HOC)

```jsx
const withLoading = (WrappedComponent) => {
  return (props) => {
    if (props.loading) {
      return <div>Загрузка...</div>;
    }
    return <WrappedComponent {...props} />;
  };
};

const EnhancedUserList = withLoading(UserList);
```

### 5. Compound Components

```jsx
const Tabs = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) => 
        React.cloneElement(child, { 
          isActive: index === activeTab, 
          onClick: () => setActiveTab(index) 
        })
      )}
    </div>
  );
};

const TabList = ({ children }) => <div className="tab-list">{children}</div>;
const Tab = ({ children, isActive, onClick }) => (
  <button 
    className={`tab ${isActive ? 'active' : ''}`}
    onClick={onClick}
  >
    {children}
  </button>
);

// Использование
const App = () => (
  <Tabs>
    <TabList>
      <Tab>Вкладка 1</Tab>
      <Tab>Вкладка 2</Tab>
      <Tab>Вкладка 3</Tab>
    </TabList>
  </Tabs>
);
```

## Когда использовать композицию

1. **Когда нужно создать гибкий интерфейс** - композиция позволяет легко комбинировать компоненты
2. **Для создания библиотек компонентов** - композиция обеспечивает большую гибкость для пользователей библиотеки
3. **При необходимости избежать сложной иерархии классов** - композиция проще в поддержке
4. **Для лучшей тестируемости** - композиция позволяет легче изолировать компоненты

## Когда использовать наследование

1. **В некоторых случаях в ООП** - наследование может быть полезно для общих свойств и методов
2. **При работе с существующими системами** - где наследование уже используется
3. **Для расширения базовых классов** - при разработке фреймворков

## Принципы композиции

### 1. Принцип композиции над наследованием

> "Предпочитайте композицию над наследованием" - один из важных принципов объектно-ориентированного проектирования.

### 2. Принцип единственной ответственности

Каждый компонент должен иметь одну причину для изменения.

### 3. Открытость/закрытость

Компоненты должны быть открыты для расширения, но закрыты для модификации.

## Связанные темы

- [[Композиция Компонентов]]
- [[Компонентная Архитектура]]
- [[Паттерны Проектирования]]
- [[React Компоненты]]
- [[React Хуки]]
- [[Контекст в React]]

## Теги

#composition #inheritance #react #components #design-patterns #frontend #architecture #oop