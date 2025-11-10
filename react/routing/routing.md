---
tags: [react, routing, javascript, frontend]
aliases: [react-router, react navigation, spa routing]
---

# Маршрутизация в React

## Введение

Маршрутизация в React позволяет создавать одностраничные приложения (SPA) с навигацией между различными представлениями без перезагрузки страницы. Это ключевой аспект современных веб-приложений.

## Клиентская vs серверная маршрутизация

### Клиентская маршрутизация

Клиентская маршрутизация обрабатывает навигацию на стороне клиента, используя JavaScript для отображения разных компонентов в зависимости от URL. Это позволяет избежать полной перезагрузки страницы и создает более плавный пользовательский опыт.

```jsx
// Пример клиентской маршрутизации
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './components/Home';
import About from './components/About';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### Серверная маршрутизация

Серверная маршрутизация требует запроса к серверу для получения нового HTML при каждом переходе. Хотя это обеспечивает лучшую SEO поддержку, пользовательский опыт может быть менее плавным.

## React Router DOM

React Router DOM - это наиболее популярная библиотека для маршрутизации в React-приложениях. Она предоставляет декларативный способ навигации по приложению.

### Основные компоненты

- `BrowserRouter` - основной контейнер для маршрутов
- `Routes` - компонент, содержащий все маршруты
- `Route` - определяет путь и соответствующий компонент
- `Link` - создает ссылки для навигации

## Основы маршрутизации

### Установка

```bash
npm install react-router-dom
```

### Базовый пример

```jsx
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <Router>
      <nav>
        <Link to="/">Главная</Link>
        <Link to="/about">О нас</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Router>
  );
}
```

## Вложенные маршруты

Вложенные маршруты позволяют создавать иерархическую структуру навигации:

```jsx
<Routes>
  <Route path="/dashboard" element={<Dashboard />}>
    <Route path="profile" element={<Profile />} />
    <Route path="settings" element={<Settings />} />
    <Route index element={<DashboardHome />} />
  </Route>
</Routes>
```

В компоненте `Dashboard` нужно использовать `Outlet` для отображения дочерних маршрутов:

```jsx
import { Outlet } from 'react-router-dom';

function Dashboard() {
  return (
    <div>
      <h1>Панель управления</h1>
      <nav>
        <Link to="profile">Профиль</Link>
        <Link to="settings">Настройки</Link>
      </nav>
      <Outlet />
    </div>
  );
}
```

## Параметры маршрутов

Параметры маршрутов позволяют передавать данные через URL:

```jsx
<Route path="/user/:id" element={<UserProfile />} />
```

Получение параметров в компоненте:

```jsx
import { useParams } from 'react-router-dom';

function UserProfile() {
  const { id } = useParams();
  return <h1>Профиль пользователя {id}</h1>;
}
```

## Программная навигация

Для программной навигации используется хук `useNavigate`:

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();

  const handleClick = () => {
    navigate('/new-path');
  };

  return <button onClick={handleClick}>Перейти</button>;
}
```

## Защита маршрутов

Защита маршрутов позволяет ограничить доступ к определенным страницам:

```jsx
import { Navigate } from 'react-router-dom';

function ProtectedRoute({ children }) {
  const isAuthenticated = checkAuth(); // ваша функция проверки аутентификации

  return isAuthenticated ? children : <Navigate to="/login" />;
}

// Использование
<Route 
  path="/admin" 
  element={
    <ProtectedRoute>
      <AdminPanel />
    </ProtectedRoute>
  } 
/>
```

## Плавные переходы между маршрутами

Для анимации переходов между маршрутами можно использовать библиотеки, такие как Framer Motion:

```jsx
import { motion } from 'framer-motion';

function AnimatedRoute({ children }) {
  return (
    <motion.div
      initial={{ opacity: 0, x: -100 }}
      animate={{ opacity: 1, x: 0 }}
      transition={{ duration: 0.3 }}
    >
      {children}
    </motion.div>
  );
}
```

## Ленивая загрузка маршрутов

Ленивая загрузка позволяет загружать компоненты только при необходимости:

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./components/Home'));
const About = lazy(() => import('./components/About'));

function App() {
  return (
    <Suspense fallback={<div>Загрузка...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  );
}
```

## SEO при использовании SPA маршрутизации

При использовании клиентской маршрутизации важно учитывать SEO. Для улучшения SEO рекомендуется:

- Использовать Server-Side Rendering (SSR) с Next.js
- Добавлять динамические метатеги
- Обеспечивать правильную работу скелетонов для поисковых роботов

## Обработка 404 ошибок

Для обработки несуществующих маршрутов:

```jsx
<Route path="*" element={<NotFound />} />
```

## Паттерны URL

React Router поддерживает различные паттерны URL:

- `/users/:id` - параметр
- `/users/:userId/posts/:postId` - несколько параметров
- `/docs/*` - подстановочный маршрут
- `/search?query=react` - строки запроса

## Оптимизация маршрутов

- Использовать ленивую загрузку для больших компонентов
- Минимизировать количество маршрутов
- Кэшировать часто используемые компоненты
- Оптимизировать структуру навигации

## Альтернативы React Router

- Next.js Router - для приложений Next.js
- Wouter - легковесная альтернатива
- Reach Router - альтернатива от создателей React Router

## Тестирование маршрутов

Для тестирования маршрутов рекомендуется использовать:

```jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';

test('рендерит главную страницу', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText(/Главная/i)).toBeInTheDocument();
});
```

## Лучшие практики

- Структурировать маршруты логически
- Использовать централизованный файл маршрутов
- Защищать конфиденциальные маршруты
- Обрабатывать ошибки навигации
- Использовать именованные маршруты для сложных приложений

## Связи с другими React файлами

- [[react/components]] - компоненты, используемые в маршрутах
- [[react/hooks]] - хуки для работы с маршрутизацией
- [[react/state-management]] - управление состоянием в маршрутах
- [[react/performance]] - оптимизация производительности маршрутов
- [[react/testing]] - тестирование компонентов маршрутов

## Заключение

Эффективная маршрутизация - ключ к созданию удобных и интуитивно понятных React-приложений. Понимание основных концепций и следование лучшим практикам поможет создавать качественные SPA с плавной навигацией.
