# React Контекст

React Контекст - это способ передавать данные через дерево компонентов без необходимости передавать пропсы на каждом уровне. Контекст позволяет создать "глобальную" переменную, к которой могут получить доступ все компоненты в дереве.

## Когда использовать контекст

Контекст предназначен для обмена данными, которые можно считать "глобальными" для дерева компонентов React, например:
- Тема приложения
- Данные пользователя
- Предпочтительный язык

## Создание контекста

```jsx
import React, { createContext, useContext } from 'react';

// Создание контекста
const ThemeContext = createContext();

// Провайдер контекста
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Компонент, использующий контекст
function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <button 
      className={theme}
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
    >
      Кнопка в {theme} теме
    </button>
  );
}
```

## Пример использования контекста

```jsx
import React, { createContext, useContext, useState } from 'react';

// Создание контекста пользователя
const UserContext = createContext();

// Провайдер контекста
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// Компонент, использующий контекст
function UserProfile() {
  const { user } = useContext(UserContext);

  if (!user) {
    return <div>Пользователь не авторизован</div>;
  }

  return <div>Профиль пользователя: {user.name}</div>;
}

// Использование в приложении
function App() {
  return (
    <UserProvider>
      <UserProfile />
    </UserProvider>
  );
}
```

## Контекст и композиция компонентов

Контекст особенно полезен при композиции компонентов, когда необходимо избежать передачи пропсов через много уровней:

```jsx
// Вместо передачи пропсов через несколько компонентов
<Layout theme={theme} user={user} language={language}>
  <Content theme={theme} user={user} language={language}>
    <Article theme={theme} user={user} language={language} />
  </Content>
</Layout>

// Используя контекст
<ThemeContext.Provider value={theme}>
  <UserContext.Provider value={user}>
    <LanguageContext.Provider value={language}>
      <Layout>
        <Content>
          <Article />
        </Content>
      </Layout>
    </LanguageContext.Provider>
  </UserContext.Provider>
</ThemeContext.Provider>
```

## Оптимизация с помощью контекста

Контекст может помочь избежать ненужных ререндеров при правильном использовании:

```jsx
// Создание нескольких контекстов для разных данных
const ThemeContext = createContext();
const UserContext = createContext();
const LanguageContext = createContext();

// Это позволяет обновлять только те компоненты, 
// которые зависят от конкретного контекста
```

## Контекст и производительность

При обновлении значения в контексте, все компоненты, использующие этот контекст, будут перерендерены. Чтобы избежать ненужных ререндеров:
- Оборачивайте значения в `useMemo` или `useCallback`
- Используйте несколько контекстов для разных данных
- Применяйте `React.memo` для компонентов

## Связанные темы

- [[Композиция Компонентов]]
- [[React Хуки]]
- [[Компонентная Архитектура]]
- [[Управление Состоянием]]
- [[React Компоненты]]

## Теги

#react #context #javascript #frontend #components #state-management