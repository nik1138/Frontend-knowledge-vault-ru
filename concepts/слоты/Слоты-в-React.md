---
aliases: [React Slots, Слоты в React, React Component Composition]
tags: [react, frontend, components, composition]
---

# Слоты в React

React не имеет встроенной концепции слотов в том виде, в котором они реализованы в Vue.js или Svelte. Однако, в React существует несколько паттернов для достижения аналогичного функционала: использование `children`, `Render Props` и `Compound Components`. Эти паттерны позволяют создавать гибкие и переиспользуемые компоненты.

## Основы композиции компонентов в React

В React композиция компонентов достигается через проп `children`, который позволяет передавать содержимое из родительского компонента в дочерний.

### Использование children

```jsx
// ChildComponent.jsx
import React from 'react';

const ChildComponent = ({ children }) => {
  return (
    <div className="card">
      <h3>Заголовок карточки</h3>
      <div className="content">
        {children}
      </div>
    </div>
  );
};

export default ChildComponent;
```

```jsx
// ParentComponent.jsx
import React from 'react';
import ChildComponent from './ChildComponent';

const ParentComponent = () => {
  return (
    <ChildComponent>
      <p>Это содержимое будет отображено внутри слота</p>
    </ChildComponent>
  );
};

export default ParentComponent;
```

## Передача нескольких "слотов" через пропсы

Для реализации нескольких областей контента можно использовать именованные пропсы:

```jsx
// LayoutComponent.jsx
import React from 'react';

const LayoutComponent = ({ header, children, footer }) => {
  return (
    <div className="layout">
      <header>
        {header}
      </header>
      <main>
        {children}
      </main>
      <footer>
        {footer}
      </footer>
    </div>
  );
};

export default LayoutComponent;
```

```jsx
// ParentComponent.jsx
import React from 'react';
import LayoutComponent from './LayoutComponent';

const ParentComponent = () => {
  return (
    <LayoutComponent
      header={<h1>Заголовок страницы</h1>}
      footer={<p>&copy; 2023 Все права защищены</p>}
    >
      <p>Основной контент страницы</p>
    </LayoutComponent>
  );
};

export default ParentComponent;
```

## Render Props

Render Props - это паттерн, который позволяет передавать функцию в компонент, которая возвращает React-элемент. Это аналог скоуп-слотов в Vue.js:

```jsx
// UserCard.jsx
import React, { Component } from 'react';

class UserCard extends Component {
  state = {
    userData: {
      name: 'Иван',
      email: 'ivan@example.com',
      role: 'admin'
    }
  };

  checkAuth = () => {
    return true; // условная проверка
  };

  render() {
    const { userData } = this.state;
    const { render } = this.props;

    return (
      <div className="user-card">
        {render({
          user: userData,
          isLoggedIn: this.checkAuth()
        })}
      </div>
    );
  }
}

export default UserCard;
```

```jsx
// ParentComponent.jsx
import React from 'react';
import UserCard from './UserCard';

const ParentComponent = () => {
  return (
    <UserCard render={({ user, isLoggedIn }) => (
      <div>
        {isLoggedIn ? (
          <div>
            <h2>Привет, {user.name}!</h2>
            <p>Ваш email: {user.email}</p>
          </div>
        ) : (
          <p>Пожалуйста, войдите в систему</p>
        )}
      </div>
    )} />
  );
};

export default ParentComponent;
```

## Compound Components

Compound Components - это паттерн, при котором компоненты работают вместе как единое целое. Это позволяет создавать более сложные компоненты с внутренним состоянием:

```jsx
// Tabs.jsx
import React, { useState } from 'react';

const Tabs = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);

  const contextValue = {
    activeTab,
    setActiveTab
  };

  return (
    <div className="tabs-container">
      <div className="tabs-header">
        {React.Children.map(children, (child, index) => {
          if (child.type.displayName === 'TabList') {
            return React.cloneElement(child, { activeTab, setActiveTab });
          }
          return null;
        })}
      </div>
      <div className="tabs-content">
        {React.Children.map(children, (child, index) => {
          if (child.type.displayName === 'TabPanels') {
            return React.cloneElement(child, { activeTab });
          }
          return null;
        })}
      </div>
    </div>
  );
};

const TabList = ({ children, activeTab, setActiveTab }) => {
  return (
    <div className="tab-list">
      {React.Children.map(children, (child, index) => {
        return React.cloneElement(child, { 
          index, 
          isActive: activeTab === index,
          onClick: () => setActiveTab(index)
        });
      })}
    </div>
  );
};

TabList.displayName = 'TabList';

const Tab = ({ children, index, isActive, onClick }) => {
  return (
    <button 
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

const TabPanels = ({ children, activeTab }) => {
  return (
    <div className="tab-panels">
      {React.Children.map(children, (child, index) => {
        if (index === activeTab) {
          return child;
        }
        return null;
      })}
    </div>
  );
};

TabPanels.displayName = 'TabPanels';

Tabs.TabList = TabList;
Tabs.Tab = Tab;
Tabs.TabPanels = TabPanels;

export default Tabs;
```

```jsx
// Использование Compound Components
import React from 'react';
import Tabs from './Tabs';

const App = () => {
  return (
    <Tabs>
      <Tabs.TabList>
        <Tabs.Tab>Вкладка 1</Tabs.Tab>
        <Tabs.Tab>Вкладка 2</Tabs.Tab>
        <Tabs.Tab>Вкладка 3</Tabs.Tab>
      </Tabs.TabList>
      <Tabs.TabPanels>
        <div>Содержимое вкладки 1</div>
        <div>Содержимое вкладки 2</div>
        <div>Содержимое вкладки 3</div>
      </Tabs.TabPanels>
    </Tabs>
  );
};

export default App;
```

## Использование React Context для передачи данных между слотами

Для передачи данных между компонентами, использующими слоты, можно использовать React Context:

```jsx
// UserContext.jsx
import React, { createContext, useContext } from 'react';

const UserContext = createContext();

export const UserProvider = ({ children, userData }) => {
  return (
    <UserContext.Provider value={userData}>
      {children}
    </UserContext.Provider>
  );
};

export const useUser = () => {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
};
```

```jsx
// CardComponent.jsx
import React from 'react';
import { UserProvider } from './UserContext';

const CardComponent = ({ userData, children }) => {
  return (
    <UserProvider userData={userData}>
      <div className="card">
        <h3>Карточка пользователя</h3>
        {children}
      </div>
    </UserProvider>
  );
};

export default CardComponent;
```

## Практические применения

### 1. Компонент списка с настраиваемым отображением элементов

```jsx
// UserList.jsx
import React from 'react';

const UserList = ({ users, renderItem }) => {
  return (
    <ul className="user-list">
      {users.map((user, index) => (
        <li key={user.id} className="user-item">
          {renderItem ? renderItem({ user, index }) : (
            <span>{user.name}</span>
          )}
        </li>
      ))}
    </ul>
  );
};

export default UserList;
```

```jsx
// Использование
import React from 'react';
import UserList from './UserList';

const App = () => {
  const users = [
    { id: 1, name: 'Иван Иванов', email: 'ivan@example.com' },
    { id: 2, name: 'Мария Смирнова', email: 'maria@example.com' }
  ];

  return (
    <UserList 
      users={users}
      renderItem={({ user, index }) => (
        <div className="user-card">
          <h3>{user.name}</h3>
          <p>{user.email}</p>
          <button onClick={() => selectUser(user)}>Выбрать</button>
        </div>
      )}
    />
  );
};
```

### 2. Модальное окно с настраиваемым содержимым

```jsx
// Modal.jsx
import React, { useEffect } from 'react';

const Modal = ({ isOpen, onClose, children, title }) => {
  useEffect(() => {
    const handleEscape = (e) => {
      if (e.key === 'Escape') onClose();
    };

    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      document.body.style.overflow = 'hidden';
    }

    return () => {
      document.removeEventListener('keydown', handleEscape);
      document.body.style.overflow = 'unset';
    };
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <div className="modal-header">
          <h2>{title}</h2>
          <button className="close-button" onClick={onClose}>×</button>
        </div>
        <div className="modal-body">
          {children}
        </div>
      </div>
    </div>
  );
};

export default Modal;
```

## Сравнение с другими фреймворками

| Паттерн | React | Vue.js | Svelte |
|---------|-------|--------|--------|
| Простые слоты | `children` | `<slot></slot>` | `<slot></slot>` |
| Именованные слоты | Именованные пропсы | `<slot name="...">` | `<slot name="...">` |
| Скоуп-слоты | Render Props | `v-slot` | `let:` |
| Композиция | Compound Components | Слоты + Props | Слоты + Props |

## Лучшие практики

1. **Используйте `children` для простых случаев** - когда нужно передать только один блок содержимого.
2. **Рассмотрите Render Props** для передачи данных из дочернего компонента в родительский.
3. **Используйте Compound Components** для сложных компонентов с общим состоянием.
4. **Документируйте интерфейс компонента** - четко указывайте, какие пропсы принимает компонент и какие данные передаются через Render Props.
5. **Рассмотрите использование TypeScript** для строгой типизации пропсов и данных, передаваемых через Render Props.

## Преимущества паттернов слотов в React

- **Гибкость**: Позволяют создавать компоненты с различными областями для контента
- **Переиспользуемость**: Компоненты могут использоваться в различных контекстах с разным содержимым
- **Контроль над отображением**: Родительский компонент может контролировать, как отображается содержимое

## Связанные темы

- [[Render-Props-в-React]]
- [[Компоненты-высшего-порядка]]
- [[Композиция-компонентов-в-React]]
- [[Скоуп-слоты]]
- [[Именованные-слоты]]

## Теги

#react #frontend #components #composition #render-props #javascript #web-development