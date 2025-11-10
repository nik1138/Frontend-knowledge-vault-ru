---
aliases: ["React Context", "Context API", "Контекст в React"]
tags: 
  - react
  - state-management
  - context-api
  - hooks
  - best-practices
---

# React Context API: Полное руководство

React Context API — это мощный инструмент для управления состоянием на уровне всего приложения, позволяющий передавать данные через дерево компонентов без необходимости передавать пропсы на каждом уровне.

## Что такое Context API

Context API позволяет создавать глобальное состояние, которое может быть доступно любому компоненту в дереве без передачи через пропсы. Это особенно полезно для таких данных, как темы, пользовательские данные, настройки и т.д.

```jsx
import React, { createContext, useContext, useReducer } from 'react';

const AppContext = createContext();
```

## Создание контекста с createContext

Функция `createContext` создает контекстный объект с двумя свойствами: `Provider` и `Consumer`.

```jsx
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
});
```

## Provider компонент

Компонент `Provider` позволяет дочерним компонентам использовать контекст. Он принимает проп `value`, который передается всем потребителям.

```jsx
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <MainComponent />
    </ThemeContext.Provider>
  );
}
```

## Хук useContext

Хук `useContext` позволяет использовать значение контекста в функциональных компонентах.

```jsx
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

## Вложенные контексты

Контексты можно вкладывать друг в друга для управления различными аспектами состояния.

```jsx
function App() {
  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={theme}>
        <MainComponent />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

## Оптимизация производительности контекста

> [!tip] Подсказка
> Контекст может вызывать повторный рендеринг всех потребителей при изменении значения. Используйте `useMemo` и `useCallback` для оптимизации.

```jsx
function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  // Оптимизированное значение контекста
  const userContextValue = useMemo(() => ({
    user,
    setUser
  }), [user]);

  const themeContextValue = useMemo(() => ({
    theme,
    setTheme
  }), [theme]);

  return (
    <UserContext.Provider value={userContextValue}>
      <ThemeContext.Provider value={themeContextValue}>
        <MainComponent />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

## Когда использовать контекст

### Используйте контекст для:

- Глобальных данных (тема, аутентификация, языки)
- Данных, которые часто используются несколькими компонентами
- Упрощения передачи пропсов через многоуровневое дерево

### Не используйте контекст для:

- Локального состояния
- Часто изменяющихся данных (может вызвать ненужные ререндеры)
- Простых случаев передачи пропсов

## Альтернативы контексту

- [[react/state-management/redux]] - полноценное решение для управления состоянием
- [[react/hooks/use-reducer]] - для сложной логики управления состоянием
- [[react/state-management/zustand]] - легковесное решение для управления состоянием
- [[react/hooks/use-state]] - для локального состояния

## Практические примеры использования

### Пример: Аутентификация пользователя

```jsx
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => setUser(userData);
  const logout = () => setUser(null);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

## Анти-паттерны

> [!warning] Предупреждение
> Избегайте создания слишком большого количества контекстов или передачи слишком большого объекта в контекст.

## Стратегии оптимизации контекста

1. Разделение контекстов по функциональности
2. Использование `React.memo` для компонентов
3. Использование `useMemo` и `useCallback` для значений
4. Комбинация с `useReducer` для сложной логики

## Комбинирование контекста с редьюсером

```jsx
const initialState = { count: 0 };

function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

const CounterContext = createContext();

export function CounterProvider({ children }) {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  );
}
```

## Сравнение с другими решениями управления состоянием

| Решение | Преимущества | Недостатки |
|--------|-------------|------------|
| Context API | Встроен в React, простой | Может вызывать лишние ререндеры |
| Redux | Мощный, предсказуемый | Более сложный, требует дополнительных зависимостей |
| Zustand | Простой, минимальный бойлерплейт | Меньше экосистемы, чем Redux |

## Лучшие практики

1. **Разделяйте контексты** по функциональности
2. **Используйте кастомные хуки** для доступа к контексту
3. **Оптимизируйте значения контекста** с помощью `useMemo`
4. **Используйте провайдеры в отдельных компонентах**
5. **Тестируйте контексты отдельно**

## Распространенные ошибки

- Создание слишком большого контекста
- Передача функций без `useCallback` (вызывает ререндеры)
- Использование контекста вместо пропсов для локальных данных
- Отсутствие провайдера при использовании контекста

## Тестирование компонентов с контекстом

```jsx
import { render, screen } from '@testing-library/react';
import { ThemeContext } from './ThemeContext';

const renderWithProvider = (component) => {
  return render(
    <ThemeContext.Provider value={{ theme: 'dark', toggleTheme: jest.fn() }}>
      {component}
    </ThemeContext.Provider>
  );
};

test('renders with dark theme', () => {
  renderWithProvider(<Header />);
  expect(screen.getByRole('button')).toBeInTheDocument();
});
```

## Связь с другими файлами

- [[react/hooks/use-state]] - для понимания локального состояния
- [[react/hooks/use-reducer]] - для сложной логики управления состоянием
- [[react/component-lifecycle]] - для понимания жизненного цикла компонентов
- [[react/performance/optimization]] - для оптимизации производительности
- [[react/testing/context-testing]] - для тестирования контекста

## Заключение

Context API — это мощный инструмент для управления глобальным состоянием в React-приложениях. Правильное использование позволяет избежать "prop drilling" и упрощает передачу данных по дереву компонентов. Однако важно помнить о производительности и использовать контекст осознанно.

Для более сложных сценариев управления состоянием рекомендуется рассмотреть [[react/state-management/redux]] или [[react/state-management/zustand]].