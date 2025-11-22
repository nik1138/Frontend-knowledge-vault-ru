---
aliases: [React Context, Контекст в React]
tags: [react, context, state-management, frontend]
---

# Контекст-в-React

## Обзор

Контекст в React - это встроенная функция, которая позволяет передавать данные через дерево компонентов без необходимости передавать пропсы на каждом уровне. Это особенно полезно для данных, которые могут быть использованы многими компонентами на разных уровнях дерева, таких как текущий пользователь, тема или языковые настройки.

## Зачем использовать контекст?

Традиционный способ передачи данных в React - через пропсы. Однако при глубокой вложенности компонентов передача пропсов через несколько уровней может привести к "prop drilling" (пробуривание пропсов), что усложняет код и делает его менее поддерживаемым. Контекст решает эту проблему, позволяя компонентам получать данные напрямую из глобального хранилища без необходимости передавать их через промежуточные компоненты.

## Основные концепции

### Context API

React предоставляет Context API, который включает в себя:
- `React.createContext()` - создает объект контекста
- `<Context.Provider>` - компонент, который позволяет дочерним компонентам использовать данные контекста
- `<Context.Consumer>` - компонент для получения данных контекста (альтернатива хуку `useContext`)
- `useContext()` - хук для получения данных контекста в функциональных компонентах

## Практическое применение

### Создание контекста

```jsx
import React, { createContext, useContext, useState } from 'react';

// Создаем контекст
const ThemeContext = createContext();

// Создаем провайдер контекста
export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Компонент, использующий контекст
export const ThemedComponent = () => {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <div className={`theme-${theme}`}>
      <p>Текущая тема: {theme}</p>
      <button onClick={toggleTheme}>Переключить тему</button>
    </div>
  );
};
```

### Использование нескольких контекстов

```jsx
import React, { createContext, useContext, useState } from 'react';

// Создаем несколько контекстов
const UserContext = createContext();
const ThemeContext = createContext();

// Комбинированный провайдер
export const AppProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        {children}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
};

// Компонент, использующий несколько контекстов
export const UserProfile = () => {
  const { user } = useContext(UserContext);
  const { theme } = useContext(ThemeContext);

  return (
    <div className={`profile theme-${theme}`}>
      <h2>Профиль пользователя: {user?.name || 'Гость'}</h2>
    </div>
  );
};
```

## Примеры использования

### 1. Управление темой приложения

```jsx
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const value = {
    theme,
    setTheme,
    toggleTheme: () => setTheme(prev => prev === 'light' ? 'dark' : 'light')
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
};
```

### 2. Управление аутентификацией

```jsx
import React, { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Проверяем аутентификацию при монтировании
    checkAuthStatus();
  }, []);

  const checkAuthStatus = async () => {
    // Логика проверки аутентификации
    const token = localStorage.getItem('token');
    if (token) {
      // Восстанавливаем данные пользователя
      try {
        const userData = await fetchUserData(token);
        setUser(userData);
      } catch (error) {
        localStorage.removeItem('token');
      }
    }
    setLoading(false);
  };

  const login = async (credentials) => {
    const response = await authenticate(credentials);
    if (response.token) {
      localStorage.setItem('token', response.token);
      setUser(response.user);
      return true;
    }
    return false;
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};
```

## Лучшие практики

### 1. Разделяйте контексты по функциональности

Не создавайте один "глобальный" контекст для всего приложения. Вместо этого создавайте отдельные контексты для разных аспектов состояния (например, тема, аутентификация, локализация).

### 2. Используйте кастомные хуки

Создавайте кастомные хуки для работы с контекстом, чтобы скрыть сложность и обеспечить согласованное поведение:

```jsx
// hooks/useTheme.js
import { useContext } from 'react';
import { ThemeContext } from '../contexts/ThemeContext';

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### 3. Оборачивайте провайдеры в отдельные компоненты

Создавайте отдельные компоненты-обертки для провайдеров, чтобы облегчить их использование и тестирование:

```jsx
// providers/AppProviders.js
import React from 'react';
import { ThemeProvider } from './ThemeProvider';
import { AuthProvider } from './AuthProvider';
import { LocalizationProvider } from './LocalizationProvider';

export const AppProviders = ({ children }) => (
  <AuthProvider>
    <ThemeProvider>
      <LocalizationProvider>
        {children}
      </LocalizationProvider>
    </ThemeProvider>
  </AuthProvider>
);
```

## Потенциальные проблемы и решения

### 1. Частые перерендеры

Контекст может вызывать перерендеры дочерних компонентов при каждом изменении значения. Для решения этой проблемы:

- Используйте `React.memo()` для компонентов, которые не должны перерендериваться при изменении контекста
- Разделяйте контексты по типам данных (например, отдельно для темы и отдельно для пользователя)
- Используйте `useMemo()` для стабилизации значений контекста

### 2. Сложность тестирования

При тестировании компонентов, использующих контекст, необходимо оборачивать их в соответствующие провайдеры:

```jsx
// test/ThemeContext.test.js
import React from 'react';
import { render, screen } from '@testing-library/react';
import { ThemeProvider } from '../contexts/ThemeContext';
import { ThemedComponent } from '../components/ThemedComponent';

test('renders with correct theme', () => {
  render(
    <ThemeProvider>
      <ThemedComponent />
    </ThemeProvider>
  );
  
  expect(screen.getByText(/текущая тема/i)).toBeInTheDocument();
});
```

## Сравнение с другими решениями

### Context API vs Redux

| Характеристика | Context API | Redux |
|---|---|---|
| Сложность настройки | Низкая | Средняя |
| Объем кода | Минимальный | Больше |
| Встроенный в React | Да | Нет (требует библиотеку) |
| Инструменты разработчика | Ограниченные | Мощные |
| Производительность | Может быть проблемой при частых изменениях | Оптимизирована для сложных сценариев |

Context API подходит для простых случаев управления состоянием, в то время как Redux лучше подходит для сложных приложений с частыми изменениями состояния.

## Ссылки и ресурсы

- [[Провайдеры-контекста]]
- [[Потребители-контекста]]
- [[Глобальное-состояние]]
- [[Хуки]]
- [[Компонентная-архитектура]]

## Заключение

Context API в React предоставляет мощный и простой способ управления глобальным состоянием без необходимости использования внешних библиотек. Правильное использование контекста может значительно упростить архитектуру приложения и улучшить его поддерживаемость. Однако важно понимать ограничения контекста и использовать его там, где это действительно необходимо.