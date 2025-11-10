# React Context API

## Введение

React Context API - это встроенный механизм для передачи данных через дерево компонентов без необходимости передавать пропсы на каждом уровне. Это позволяет избежать **[[prop drilling]]** и упрощает управление состоянием в приложении.

## Когда использовать Context

Используйте Context API когда:
- Данные должны быть доступны нескольким компонентам на разных уровнях дерева
- Избегается передача пропсов через множество промежуточных компонентов
- Управление аутентификацией, темами, настройками приложения

Не используйте для:
- Простой передачи данных между двумя компонентами
- Часто изменяющихся данных (лучше использовать [[react/state-management]] или [[react/hooks]])

## Создание контекста

Функция `createContext` создает объект контекста:

```javascript
import React, { createContext, useContext } from 'react';

const ThemeContext = createContext();
```

Можно указать начальное значение:

```javascript
const ThemeContext = createContext('light'); // значение по умолчанию
```

## Провайдер контекста

Провайдер (`Provider`) делает значение контекста доступным для дочерних компонентов:

```javascript
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  );
}
```

## Использование контекста

Хук `useContext` позволяет получить значение контекста:

```javascript
function Header() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <header className={theme}>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Переключить тему
      </button>
    </header>
  );
}
```

## Вложенные провайдеры

Контекст можно переопределять на разных уровнях:

```javascript
function App() {
  return (
    <ThemeContext.Provider value="light">
      <ChildComponent />
    </ThemeContext.Provider>
  );
}

function ChildComponent() {
  return (
    <ThemeContext.Provider value="dark">
      <GrandChildComponent /> {/* будет использовать "dark" */}
    </ThemeContext.Provider>
  );
}
```

## Оптимизация контекста

Для избежания ненужных перерендеров оборачивайте значения в `useMemo` или `useCallback`:

```javascript
function App() {
  const [state, setState] = useState(initialState);
  
  const contextValue = useMemo(() => ({
    state,
    updateState: useCallback((newState) => setState(newState), [])
  }), [state]);
  
  return (
    <StateContext.Provider value={contextValue}>
      {/* дочерние компоненты */}
    </StateContext.Provider>
  );
}
```

## Performance considerations

- Контекст вызывает перерендер всех потребителей при изменении значения
- Используйте мемоизацию значений контекста
- Разделяйте контексты по функциональности для минимизации перерендеров

## Паттерн Context + Reducer

Сочетание `Context` и `useReducer` создает мощную систему управления состоянием:

```javascript
const AppReducer = (state, action) => {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'LOGOUT':
      return { ...state, user: null };
    default:
      return state;
  }
};

const AppStateContext = createContext();
const AppDispatchContext = createContext();

export function AppProvider({ children }) {
  const [state, dispatch] = useReducer(AppReducer, initialState);
  
  return (
    <AppStateContext.Provider value={state}>
      <AppDispatchContext.Provider value={dispatch}>
        {children}
      </AppDispatchContext.Provider>
    </AppStateContext.Provider>
  );
}

export const useAppState = () => useContext(AppStateContext);
export const useAppDispatch = () => useContext(AppDispatchContext);
```

## Избегание проп дриллинга

Context API решает проблему передачи пропсов через несколько уровней компонентов:

```javascript
// Без контекста - проп дриллинг
function App() {
  return <ComponentA user={user} />;
}
function ComponentA({ user }) {
  return <ComponentB user={user} />;
}
function ComponentB({ user }) {
  return <ComponentC user={user} />;
}

// С контекстом - прямой доступ
function App() {
  return (
    <UserContext.Provider value={user}>
      <ComponentA />
    </UserContext.Provider>
  );
}
function ComponentC() {
  const user = useContext(UserContext);
  // прямой доступ к пользователю
}
```

## Управление сложным состоянием

Для сложных состояний используйте комбинацию контекста с `useReducer`:

```javascript
const UserContext = createContext();

function UserProvider({ children }) {
  const [users, setUsers] = useState([]);
  const [currentUser, setCurrentUser] = useState(null);
  
  const updateUser = (id, data) => {
    setUsers(prev => prev.map(u => u.id === id ? { ...u, ...data } : u));
  };
  
  const value = {
    users,
    currentUser,
    setCurrentUser,
    updateUser
  };
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

## Комбинация с кастомными хуками

Создавайте кастомные хуки для работы с контекстом:

```javascript
const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

## Ограничения Context API

- Перерендер всего дерева потребителей при изменении значения
- Неэффективен для часто изменяющихся данных
- Отладка может быть сложной в больших приложениях
- Меньше инструментов для тестирования по сравнению с внешними решениями

## Сравнение с другими решениями

| Решение | Когда использовать |
|--------|------------------|
| Context API | Простое управление состоянием, темы, аутентификация |
| Redux | Сложные приложения, нужна отладка, middleware |
| Zustand | Простота, производительность, минимальная настройка |
| MobX | Реактивность, сложные зависимости данных |

## Лучшие практики

- Разделяйте контексты по функциональности
- Используйте мемоизацию для значений
- Создавайте кастомные хуки для работы с контекстом
- Оборачивайте провайдеры в отдельные компоненты
- Проверяйте наличие контекста в кастомных хуках

## Безопасность

- Не храните чувствительные данные в контексте без шифрования
- Используйте проверки аутентификации в провайдерах
- Очищайте данные при выходе из системы

## Связи с другими файлами

- [[react/hooks]] - для понимания работы хуков
- [[react/state-management]] - альтернативы контексту
- [[react/components]] - для понимания передачи данных
- [[react/performance]] - оптимизация рендеринга

## Теги

#react #context-api #state-management #frontend #javascript #best-practices #performance