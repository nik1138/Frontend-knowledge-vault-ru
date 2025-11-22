---
aliases: ["Композиция в React", "React композиция компонентов", "Композиция в React приложениях"]
tags: ["#frontend", "#react", "#composition", "#components", "#architecture", "#javascript"]
---

# Композиция в React

## Введение

**Композиция в React** — это фундаментальный паттерн архитектуры, который позволяет создавать сложные пользовательские интерфейсы путем объединения более простых компонентов. React поощряет использование композиции вместо наследования для построения компонентов, что делает код более гибким, переиспользуемым и предсказуемым.

## Принципы композиции в React

### 1. Принцип "Композиция лучше наследования"
React рекомендует создавать компоненты, которые могут быть объединены друг с другом, вместо создания сложных иерархий наследования. Это делает компоненты более предсказуемыми и легкими для понимания.

### 2. Использование props.children
Один из ключевых способов композиции в React — использование `props.children`, который позволяет передавать дочерние элементы в компонент:

```jsx
function FancyBorder(props) {
  return (
    <div className={`border-${props.color}`}>
      {props.children}
    </div>
  );
}

function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="dialog-title">Добро пожаловать</h1>
      <p className="dialog-message">Спасибо, что посетили наш космический корабль!</p>
    </FancyBorder>
  );
}
```

### 3. Передача компонентов в качестве пропсов
React позволяет передавать компоненты как пропсы, что обеспечивает высокую гибкость:

```jsx
function Dialog(props) {
  return (
    <div className="dialog">
      <div className="dialog-title">{props.title}</div>
      <div className="dialog-content">{props.content}</div>
    </div>
  );
}

function App() {
  return (
    <Dialog
      title={<h2>Привет!</h2>}
      content={<p>Содержимое диалога</p>}
    />
  );
}
```

## Практические примеры композиции в React

### Пример 1: Композиция через props.children

```jsx
// Card.jsx
import React from 'react';

const Card = ({ children, className = '', header, footer }) => {
  return (
    <div className={`card ${className}`}>
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
};

// Использование
const UserProfileCard = () => {
  return (
    <Card 
      header={<h2>Профиль пользователя</h2>}
      footer={<button>Редактировать</button>}
    >
      <div>
        <p>Имя: Иван Иванов</p>
        <p>Email: ivan@example.com</p>
      </div>
    </Card>
  );
};
```

### Пример 2: Компоненты с "отверстиями" (slots)

```jsx
// Layout.jsx
import React from 'react';

const Layout = ({ header, sidebar, main, footer }) => {
  return (
    <div className="layout">
      <header>{header}</header>
      <div className="content-wrapper">
        {sidebar && <aside className="sidebar">{sidebar}</aside>}
        <main className="main-content">{main}</main>
      </div>
      <footer>{footer}</footer>
    </div>
  );
};

// Использование
const AppLayout = () => {
  return (
    <Layout
      header={<Navigation />}
      sidebar={<SidebarMenu />}
      main={<MainContent />}
      footer={<Footer />}
    />
  );
};
```

### Пример 3: Условная композиция

```jsx
// ConditionalWrapper.jsx
import React from 'react';

const ConditionalWrapper = ({ condition, wrapper, children }) => {
  return condition ? wrapper(children) : children;
};

// Использование
const MyComponent = ({ isAdmin, children }) => {
  return (
    <ConditionalWrapper
      condition={isAdmin}
      wrapper={(children) => (
        <div className="admin-panel">
          <AdminControls />
          {children}
        </div>
      )}
    >
      <div className="regular-content">{children}</div>
    </ConditionalWrapper>
  );
};
```

### Пример 4: Композиция с хуками

```jsx
// useModal.jsx
import { useState, useCallback } from 'react';

const useModal = () => {
  const [isOpen, setIsOpen] = useState(false);

  const openModal = useCallback(() => setIsOpen(true), []);
  const closeModal = useCallback(() => setIsOpen(false), []);

  return { isOpen, openModal, closeModal };
};

// Modal.jsx
import React from 'react';

const Modal = ({ isOpen, onClose, children, title }) => {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        <div className="modal-header">
          <h3>{title}</h3>
          <button onClick={onClose}>×</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
};

// Композиция компонентов
const UserManagement = () => {
  const { isOpen, openModal, closeModal } = useModal();

  return (
    <div>
      <button onClick={openModal}>Управление пользователями</button>
      <Modal isOpen={isOpen} onClose={closeModal} title="Управление пользователями">
        <UserList />
      </Modal>
    </div>
  );
};
```

## Паттерны композиции в React

### 1. Container/Presentational Pattern
Разделение компонентов на контейнерные (управляющие данными) и презентационные (отвечающие за отображение):

```jsx
// Presentational Component
const UserList = ({ users, onUserClick }) => {
  return (
    <ul className="user-list">
      {users.map(user => (
        <li key={user.id} onClick={() => onUserClick(user)}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  );
};

// Container Component
const UserListContainer = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUsers().then(setUsers).finally(() => setLoading(false));
  }, []);

  const handleUserClick = (user) => {
    console.log('Пользователь выбран:', user);
  };

  if (loading) return <div>Загрузка...</div>;

  return <UserList users={users} onUserClick={handleUserClick} />;
};
```

### 2. Higher-Order Components (HOC)
Функции, которые принимают компонент и возвращают новый компонент с дополнительной функциональностью:

```jsx
// HOC для загрузки данных
const withData = (WrappedComponent, dataFetcher) => {
  return (props) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
      dataFetcher().then(setData).finally(() => setLoading(false));
    }, []);

    if (loading) return <div>Загрузка...</div>;

    return <WrappedComponent {...props} data={data} />;
  };
};

// Использование
const EnhancedUserList = withData(UserList, fetchUsers);
```

### 3. Render Props
Паттерн, при котором компонент принимает функцию, которая определяет, что отображать:

```jsx
// DataProvider.jsx
class DataProvider extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: null, loading: true };
  }

  componentDidMount() {
    this.props.dataFetcher().then(data => {
      this.setState({ data, loading: false });
    });
  }

  render() {
    return this.props.children(this.state);
  }
}

// Использование
const App = () => (
  <DataProvider dataFetcher={fetchUsers}>
    {({ data, loading }) => 
      loading ? <div>Загрузка...</div> : <UserList users={data} />
    }
  </DataProvider>
);
```

### 4. Compound Components
Компоненты, которые работают вместе как единое целое:

```jsx
// Tabs.jsx
import React, { useState, createContext, useContext } from 'react';

const TabsContext = createContext();

export const Tabs = ({ children, defaultActiveTab }) => {
  const [activeTab, setActiveTab] = useState(defaultActiveTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};

export const TabList = ({ children }) => {
  return <div className="tab-list">{children}</div>;
};

export const Tab = ({ id, children }) => {
  const { activeTab, setActiveTab } = useContext(TabsContext);
  const isActive = activeTab === id;

  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
};

export const TabPanels = ({ children }) => {
  return <div className="tab-panels">{children}</div>;
};

export const TabPanel = ({ id, children }) => {
  const { activeTab } = useContext(TabsContext);
  const isVisible = activeTab === id;

  return isVisible ? <div className="tab-panel">{children}</div> : null;
};

// Использование
const MyTabs = () => (
  <Tabs defaultActiveTab="profile">
    <TabList>
      <Tab id="profile">Профиль</Tab>
      <Tab id="settings">Настройки</Tab>
      <Tab id="help">Помощь</Tab>
    </TabList>
    <TabPanels>
      <TabPanel id="profile">
        <ProfileContent />
      </TabPanel>
      <TabPanel id="settings">
        <SettingsContent />
      </TabPanel>
      <TabPanel id="help">
        <HelpContent />
      </TabPanel>
    </TabPanels>
  </Tabs>
);
```

## Рекомендации по композиции в React

### 1. Используйте композицию для создания универсальных компонентов
Создавайте компоненты, которые могут быть использованы в различных контекстах:

```jsx
// Layout.jsx
const Layout = ({ children, sidebar, header, footer }) => (
  <div className="layout">
    {header && <header>{header}</header>}
    <div className="content">
      {sidebar && <aside>{sidebar}</aside>}
      <main>{children}</main>
    </div>
    {footer && <footer>{footer}</footer>}
  </div>
);

// Можно использовать в разных частях приложения
const HomePage = () => (
  <Layout header={<Header />} sidebar={<Sidebar />}>
    <HomeContent />
  </Layout>
);

const ProfilePage = () => (
  <Layout header={<Header />} sidebar={<ProfileSidebar />}>
    <ProfileContent />
  </Layout>
);
```

### 2. Избегайте глубокой вложенности
Слишком глубокая композиция может усложнить отладку и понимание компонентов:

```jsx
// Плохо: слишком глубокая вложенность
const BadExample = () => (
  <Container>
    <Section>
      <Block>
        <Card>
          <Item>
            <Element>Содержимое</Element>
          </Item>
        </Card>
      </Block>
    </Section>
  </Container>
);

// Лучше: разбить на более логичные компоненты
const GoodExample = () => (
  <PageLayout>
    <UserProfileSection>
      <UserInfoCard />
    </UserProfileSection>
  </PageLayout>
);
```

### 3. Используйте TypeScript для улучшения композиции
TypeScript помогает создавать более надежные композиции:

```tsx
interface CardProps {
  children: React.ReactNode;
  title?: string;
  actions?: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ children, title, actions }) => {
  return (
    <div className="card">
      {title && <h3>{title}</h3>}
      <div className="card-content">{children}</div>
      {actions && <div className="card-actions">{actions}</div>}
    </div>
  );
};
```

## Контрольные вопросы

1. В чем разница между композицией и наследованием в React?
2. Как использовать props.children для композиции компонентов?
3. Какие паттерны композиции вы знаете и когда их использовать?
4. Как обеспечить типобезопасность в композиции компонентов?
5. Какие преимущества дает композиция перед другими подходами?

## См. также

- [[Композиция компонентов]]
- [[Агрегация компонентов]]
- [[Принципы объектно-ориентированного программирования]]
- [[React компоненты]]
- [[Хуки в React]]

## Внешние ресурсы

- [React Documentation - Composition vs Inheritance](https://reactjs.org/docs/composition-vs-inheritance.html)
- [React Patterns - Component Composition](https://reactpatterns.com/#component-composition)
- [Advanced React Patterns](https://github.com/kentcdodds/advanced-react-patterns)