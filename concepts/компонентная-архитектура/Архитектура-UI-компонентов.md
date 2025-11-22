---
aliases: [Архитектура UI компонентов, UI архитектура]
tags: [frontend, architecture, components, ui]
---

# Архитектура UI-компонентов

Архитектура UI-компонентов — это систематический подход к проектированию, организации и взаимодействию пользовательских интерфейсов в современных веб-приложениях. Она определяет, как компоненты должны быть структурированы, как они взаимодействуют друг с другом и как управляется состояние интерфейса.

## Основные принципы архитектуры UI-компонентов

### 1. Модульность
Каждый компонент должен быть автономной единицей с четко определенными границами и ответственностью.

### 2. Повторное использование
Компоненты должны быть спроектированы так, чтобы их можно было использовать в различных контекстах.

### 3. Изолированность
Компоненты должны быть максимально изолированы от внешнего контекста и зависеть только от переданных свойств (props).

### 4. Предсказуемость
Компонент должен предсказуемо реагировать на изменения входных данных.

## Структура UI-компонентов

### Уровни компонентов

#### 1. Презентационные компоненты (Dumb Components)
- Отвечают только за отображение данных
- Не содержат логики бизнес-процессов
- Получают данные через props
- Часто являются чистыми функциями

```jsx
// Презентационный компонент
const UserCard = ({ user, onEdit, onDelete }) => (
  <div className="user-card">
    <h3>{user.name}</h3>
    <p>{user.email}</p>
    <div className="user-actions">
      <button onClick={() => onEdit(user.id)}>Редактировать</button>
      <button onClick={() => onDelete(user.id)}>Удалить</button>
    </div>
  </div>
);
```

#### 2. Контейнерные компоненты (Smart Components)
- Управляют состоянием и логикой
- Выполняют запросы к API
- Обрабатывают события и обновляют данные

```jsx
// Контейнерный компонент
class UserListContainer extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
      loading: true,
      error: null
    };
  }

  componentDidMount() {
    this.fetchUsers();
  }

  fetchUsers = async () => {
    try {
      const users = await api.getUsers();
      this.setState({ users, loading: false });
    } catch (error) {
      this.setState({ error, loading: false });
    }
  };

  handleDelete = async (userId) => {
    try {
      await api.deleteUser(userId);
      this.setState(prevState => ({
        users: prevState.users.filter(user => user.id !== userId)
      }));
    } catch (error) {
      console.error('Ошибка удаления пользователя:', error);
    }
  };

  render() {
    const { users, loading, error } = this.state;
    
    if (loading) return <div>Загрузка...</div>;
    if (error) return <div>Ошибка: {error.message}</div>;
    
    return (
      <div className="user-list">
        {users.map(user => (
          <UserCard
            key={user.id}
            user={user}
            onEdit={this.handleEdit}
            onDelete={this.handleDelete}
          />
        ))}
      </div>
    );
  }
}
```

#### 3. Служебные компоненты (Utility Components)
- Обеспечивают вспомогательную функциональность
- Например: модальные окна, уведомления, лоадеры

```jsx
// Служебный компонент
const Modal = ({ isOpen, onClose, title, children }) => {
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
```

## Паттерны управления состоянием

### 1. Локальное состояние компонента
Используется для управления состоянием, которое используется только внутри одного компонента.

```jsx
const SearchComponent = () => {
  const [searchTerm, setSearchTerm] = React.useState('');
  const [results, setResults] = React.useState([]);

  const handleSearch = async (term) => {
    const searchResults = await searchAPI.search(term);
    setResults(searchResults);
  };

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && handleSearch(searchTerm)}
      />
      <SearchResults results={results} />
    </div>
  );
};
```

### 2. Подъем состояния (Lifting State Up)
Когда несколько компонентов нуждаются в доступе к одному и тому же состоянию, оно поднимается к ближайшему общему предку.

```jsx
const FilterableProductTable = () => {
  const [filterText, setFilterText] = React.useState('');
  const [inStockOnly, setInStockOnly] = React.useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterChange={setFilterText}
        onStockChange={setInStockOnly}
      />
      <ProductTable
        products={PRODUCTS}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
};
```

### 3. Контекст (Context)
Для передачи данных через дерево компонентов без необходимости передавать props на каждом уровне.

```jsx
// Создание контекста
const ThemeContext = React.createContext();

// Провайдер контекста
const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = React.useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Использование контекста
const Header = () => {
  const { theme, setTheme } = React.useContext(ThemeContext);

  return (
    <header className={`header ${theme}`}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Переключить тему
      </button>
    </header>
  );
};
```

### 4. Внешние библиотеки управления состоянием
Для сложных приложений используются библиотеки вроде Redux, Zustand или MobX.

```javascript
// Пример с Zustand
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
}));

const Counter = () => {
  const { count, increment, decrement } = useStore();

  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
};
```

## Архитектурные шаблоны

### 1. Atomic Design
Подход к проектированию интерфейсов, основанный на построении компонентов от простого к сложному:

- **Атомы** (Atoms): Базовые элементы (кнопки, поля ввода)
- **Молекулы** (Molecules): Комбинации атомов (формы поиска)
- **Организмы** (Organisms): Комбинации молекул (шапка сайта)
- **Шаблоны** (Templates): Структура страниц
- **Страницы** (Pages): Конкретные реализации шаблонов

```jsx
// Атом - кнопка
const Button = ({ children, onClick, variant = 'primary' }) => (
  <button className={`btn btn-${variant}`} onClick={onClick}>
    {children}
  </button>
);

// Молекула - форма поиска
const SearchForm = ({ onSearch }) => {
  const [query, setQuery] = React.useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Поиск..."
      />
      <Button type="submit">Найти</Button>
    </form>
  );
};

// Организм - шапка сайта
const Header = ({ onSearch }) => (
  <header className="header">
    <h1>Мой сайт</h1>
    <SearchForm onSearch={onSearch} />
  </header>
);
```

### 2. Паттерн "Контейнеры и презентационные компоненты"

Уже рассмотренный выше паттерн, который разделяет компоненты на:
- Презентационные (отвечают за внешний вид)
- Контейнерные (управляют логикой и состоянием)

### 3. Паттерн "Состояние машины" (State Machine)

Используется для управления сложными состояниями с предсказуемыми переходами:

```javascript
import { createMachine, useMachine } from '@xstate/react';

const toggleMachine = createMachine({
  id: 'toggle',
  initial: 'inactive',
  states: {
    inactive: {
      on: { TOGGLE: 'active' }
    },
    active: {
      on: { TOGGLE: 'inactive' }
    }
  }
});

const Toggle = () => {
  const [current, send] = useMachine(toggleMachine);

  return (
    <button
      onClick={() => send('TOGGLE')}
      className={current.matches('active') ? 'active' : ''}
    >
      {current.matches('active') ? 'ВКЛ' : 'ВЫКЛ'}
    </button>
  );
};
```

## Лучшие практики архитектуры UI-компонентов

### 1. Именование компонентов
- Используйте понятные и описательные имена
- Следуйте соглашениям о наименовании (PascalCase для компонентов)
- Используйте префиксы для групп компонентов (например, `ButtonPrimary`, `ButtonSecondary`)

### 2. Структура файлов
```
components/
├── atoms/
│   ├── Button/
│   ├── Input/
│   └── Icon/
├── molecules/
│   ├── SearchForm/
│   ├── LoginForm/
│   └── Card/
├── organisms/
│   ├── Header/
│   ├── Navigation/
│   └── ProductGrid/
├── templates/
│   ├── PageLayout/
│   └── DashboardLayout/
└── pages/
    ├── HomePage/
    └── ProductPage/
```

### 3. Типизация
Используйте TypeScript или PropTypes для определения типов props:

```typescript
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
  };
  onEdit: (id: number) => void;
  onDelete: (id: number) => void;
}

const UserCard: React.FC<UserCardProps> = ({ user, onEdit, onDelete }) => {
  // Реализация компонента
};
```

### 4. Тестирование
- Покрывайте компоненты юнит-тестами
- Используйте интеграционные тесты для проверки взаимодействия
- Проводите тестирование доступности

```javascript
// Тестирование с React Testing Library
import { render, fireEvent, screen } from '@testing-library/react';
import UserCard from './UserCard';

test('вызывает onEdit при клике на кнопку редактирования', () => {
  const onEditMock = jest.fn();
  const user = { id: 1, name: 'John', email: 'john@example.com' };
  
  render(<UserCard user={user} onEdit={onEditMock} onDelete={jest.fn()} />);
  
  fireEvent.click(screen.getByText('Редактировать'));
  expect(onEditMock).toHaveBeenCalledWith(1);
});
```

## Архитектурные ограничения и решения

### 1. Проблема "prop drilling"
Когда props передаются через несколько уровней компонентов.

**Решения:**
- Использование Context API
- Разделение компонентов на более мелкие
- Использование внешнего состояния (Redux, Zustand)

### 2. Управление сложным состоянием
Для сложных приложений с множественными источниками истины.

**Решения:**
- Централизованное управление состоянием
- Использование библиотек управления состоянием
- Архитектурные паттерны (Flux, Redux)

### 3. Производительность
Неэффективное обновление компонентов может привести к снижению производительности.

**Решения:**
- Memoization (React.memo, useMemo, useCallback)
- Оптимизация рендеринга
- Использование виртуального скроллинга для больших списков

```jsx
// Оптимизация с React.memo
const ExpensiveComponent = React.memo(({ data }) => {
  return (
    <div>
      {/* Рендеринг сложного содержимого */}
      {data.map(item => <Item key={item.id} item={item} />)}
    </div>
  );
});

// Оптимизация коллбэков
const ParentComponent = () => {
  const [items, setItems] = React.useState([]);

  const handleItemChange = React.useCallback((id, value) => {
    setItems(prev => prev.map(item => 
      item.id === id ? { ...item, value } : item
    ));
  }, [setItems]);

  return (
    <div>
      {items.map(item => (
        <ChildComponent 
          key={item.id} 
          item={item} 
          onChange={handleItemChange} 
        />
      ))}
    </div>
  );
};
```

## Связанные концепции

- [[Архитектурные-паттерны-компонентов]]
- [[Микрофронтенды]]
- [[Компонентные-библиотеки]]
- [[Дизайн-системы-и-архитектура]]

## Теги

#frontend #architecture #components #ui #react #state-management #atomic-design