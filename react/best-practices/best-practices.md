---
aliases: ["React лучшие практики", "React паттерны", "React структура компонентов"]
tags: 
  - #react
  - #best-practices
  - #frontend
  - #javascript
  - #component-design
---

# Лучшие практики React разработки

## Введение

React — мощная библиотека для создания пользовательских интерфейсов. Эта документация охватывает лучшие практики разработки с React, включая структуру компонентов, оптимизацию производительности, паттерны и рекомендации по организации кода.

## Структура компонентов

### Основные принципы

- **Компоненты должны быть предсказуемыми**: каждый компонент должен возвращать одинаковый результат при одинаковых входных данных
- **Компоненты должны быть чистыми**: избегайте побочных эффектов внутри тела компонента
- **Компоненты должны быть переиспользуемыми**: проектируйте компоненты так, чтобы их можно было использовать в разных частях приложения

```jsx
// Хорошо: чистый компонент с пропсами
function UserCard({ user, onClick }) {
  return (
    <div className="user-card" onClick={onClick}>
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### Типы компонентов

- **Представления (Presentational)**: отвечают за визуальное отображение
- **Контейнеры (Container)**: управляют данными и логикой
- **Компоненты-обертки (Wrapper)**: оборачивают другие компоненты, добавляя функциональность

## Организация кода

### Архитектура файлов

```
src/
├── components/           # Переиспользуемые компоненты
│   ├── ui/              # Базовые UI компоненты
│   ├── common/          # Общие компоненты
│   └── features/        # Компоненты, связанные с фичами
├── pages/               # Страницы приложения
├── hooks/               # Пользовательские хуки
├── context/             # Контексты React
├── services/            # API сервисы
├── utils/               # Утилиты
├── types/               # TypeScript типы
└── assets/              # Статические ресурсы
```

### Именование файлов

- Используйте PascalCase для компонентов: `UserProfile.jsx`
- Используйте camelCase для хуков: `useAuth.js`
- Добавляйте суффиксы для специфичных файлов: `Button.styles.js`, `api.service.js`

## Паттерны React

### Хуки (Hooks)

- **useState**: для управления локальным состоянием
- **useEffect**: для побочных эффектов
- **useContext**: для доступа к контексту
- **useReducer**: для сложного состояния
- **useMemo/useCallback**: для оптимизации производительности

```jsx
import { useState, useEffect, useCallback, useMemo } from 'react';

function TodoList({ userId }) {
  const [todos, setTodos] = useState([]);
  
  // Оптимизированный коллбэк
  const handleAddTodo = useCallback((text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, userId }]);
  }, [userId]);
  
  // Оптимизированное вычисление
  const completedCount = useMemo(() => 
    todos.filter(todo => todo.completed).length, 
    [todos]
  );
  
  useEffect(() => {
    fetchTodos(userId).then(setTodos);
  }, [userId]);
  
  return (
    <div>
      <TodoForm onAdd={handleAddTodo} />
      <TodoItems items={todos} />
      <p>Завершено: {completedCount}</p>
    </div>
  );
}
```

### Высокоуровневые компоненты (HOC)

```jsx
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const { isAuthenticated } = useAuth();
    
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    
    return <WrappedComponent {...props} />;
  };
}

const ProtectedProfile = withAuth(Profile);
```

### Render Props

```jsx
class DataFetcher extends React.Component {
  state = { data: null, loading: true };
  
  componentDidMount() {
    this.props.fetchData().then(data => {
      this.setState({ data, loading: false });
    });
  }
  
  render() {
    return this.props.render(this.state);
  }
}

// Использование
<DataFetcher
  fetchData={() => fetch('/api/users')}
  render={({ data, loading }) => 
    loading ? <LoadingSpinner /> : <UserList users={data} />
  }
/>
```

## Оптимизация производительности

### React.memo

```jsx
// Оптимизация компонентов, предотвращающая ненужные перерендеры
const ExpensiveComponent = React.memo(({ data, theme }) => {
  return (
    <div className={`component ${theme}`}>
      {data.map(item => <Item key={item.id} item={item} />)}
    </div>
  );
});
```

### useCallback и useMemo

```jsx
function ParentComponent({ items }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // Памоизируем коллбэк, чтобы избежать пересоздания функции
  const handleSelect = useCallback((id) => {
    setSelectedId(id);
  }, [setSelectedId]);
  
  // Мемоизируем вычисления
  const expensiveValue = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return (
    <div>
      <ChildComponent 
        items={items} 
        onSelect={handleSelect}
        total={expensiveValue}
      />
    </div>
  );
}
```

### Code Splitting

```jsx
import { lazy, Suspense } from 'react';
import LoadingSpinner from './LoadingSpinner';

const LazyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <div>
      <Header />
      <Suspense fallback={<LoadingSpinner />}>
        <LazyComponent />
      </Suspense>
    </div>
  );
}
```

## Управление состоянием

### Локальное состояние

- Используйте `useState` для простого состояния
- Используйте `useReducer` для сложного состояния с логикой

### Глобальное состояние

- Context API для простых случаев
- Redux Toolkit для сложных приложений
- Zustand для легковесного глобального состояния

```jsx
// Пример с Context API
const AppContext = createContext();

export function AppProvider({ children }) {
  const [state, dispatch] = useReducer(appReducer, initialState);
  
  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
}

// Использование в компонентах
function Component() {
  const { state, dispatch } = useContext(AppContext);
  // ...
}
```

## Безопасность

- Экранируйте пользовательский ввод
- Не используйте `dangerouslySetInnerHTML` без необходимости
- Валидируйте и очищайте данные перед отображением

```jsx
// Неправильно
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// Правильно - используйте библиотеки для очистки HTML
import DOMPurify from 'dompurify';
const cleanHTML = DOMPurify.sanitize(userInput);
<div dangerouslySetInnerHTML={{ __html: cleanHTML }} />
```

## Доступность (Accessibility)

- Используйте семантические теги
- Добавляйте `aria-*` атрибуты
- Обеспечьте навигацию с клавиатуры
- Поддержка скринридеров

```jsx
function AccessibleButton({ onClick, children, isActive }) {
  return (
    <button
      role="button"
      tabIndex="0"
      aria-pressed={isActive}
      aria-label={isActive ? "Активный элемент" : "Неактивный элемент"}
      onClick={onClick}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          onClick();
        }
      }}
    >
      {children}
    </button>
  );
}
```

## Формы

### С использованием библиотек

- React Hook Form
- Formik
- Final Form

### Без библиотек

```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Очистка ошибки при изменении поля
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    if (!formData.name.trim()) newErrors.name = 'Имя обязательно';
    if (!formData.email.trim()) {
      newErrors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email некорректен';
    }
    return newErrors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Отправка формы
    submitForm(formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
          aria-invalid={!!errors.name}
          aria-describedby={errors.name ? "name-error" : undefined}
        />
        {errors.name && (
          <span id="name-error" role="alert">{errors.name}</span>
        )}
      </div>
      {/* Другие поля формы */}
      <button type="submit">Отправить</button>
    </form>
  );
}
```

## Ошибки и отладка

### Error Boundaries

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Ошибка в компоненте:', error, errorInfo);
    // Отправка отчета об ошибке в сервис отслеживания
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}

// Использование
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

## Современные возможности React

- React 18: автоматическое пакетирование, новые хуки (useId, useSyncExternalStore)
- Concurrent Features: Suspense, Transitions
- Strict Mode: обнаружение потенциальных проблем

```jsx
// Использование автоматического пакетирования
import { flushSync } from 'react-dom';

function handleClick() {
  setCount(c => c + 1);
  // Это вызовет перерендер сразу
  flushSync(() => {
    setFlag(f => !f);
  });
  // DOM обновлен
}
```

## Линтинг и форматирование

- ESLint с плагином react
- Prettier для форматирования
- Husky + lint-staged для пре-коммит хуков

```json
// .eslintrc.json
{
  "extends": [
    "react-app",
    "react-app/jest"
  ],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react/jsx-props-no-spreading": "warn"
  }
}
```

## Документирование

- JSDoc для функций и компонентов
- Storybook для документации компонентов
- Комментарии для сложной логики

```jsx
/**
 * Компонент профиля пользователя
 * @param {Object} user - Объект пользователя
 * @param {string} user.name - Имя пользователя
 * @param {string} user.email - Email пользователя
 * @param {Function} onEdit - Коллбэк для редактирования профиля
 */
function UserProfile({ user, onEdit }) {
  // реализация
}
```

## Рефакторинг

- Извлечение компонентов
- Упрощение условных выражений
- Разделение ответственности
- Использование паттернов проектирования

## Работа в команде

- Единый стиль кодирования
- Code Review процессы
- Использование шаблонов компонентов
- Документирование архитектурных решений

## Связанные файлы

- [[react/hooks/hooks]] - Подробное руководство по React хукам
- [[react/context/context]] - Управление состоянием через Context API
- [[react/performance/performance]] - Глубокое погружение в оптимизацию производительности
- [[react/testing/testing]] - Тестирование React компонентов
- [[react/forms/forms]] - Работа с формами в React
- [[react/security/security]] - Безопасность в React приложениях
- [[react/accessibility/accessibility]] - Доступность в React
- [[component-architecture/component-architecture]] - Архитектура компонентов
- [[js/es6-features/es6-features]] - Современные возможности JavaScript
- [[ts/react-typescript/react-typescript]] - Использование TypeScript с React

## Заключение

Следование этим лучшим практикам поможет создавать более надежные, масштабируемые и поддерживаемые React-приложения. Помните, что практики могут изменяться со временем, и важно оставаться в курсе новых возможностей и рекомендаций сообщества.