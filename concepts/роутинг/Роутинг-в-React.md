---
aliases: ["React Router", "React Navigation"]
tags: [routing, react, frontend, javascript]
---

# Роутинг в React

Роутинг в React - это механизм навигации между различными компонентами приложения без перезагрузки страницы. Для реализации роутинга в React-приложениях обычно используется библиотека React Router.

## Основные понятия

React Router - это стандартная библиотека для реализации клиентского роутинга в React-приложениях. Она позволяет:
- Определять маршруты с помощью компонентов
- Передавать параметры между маршрутами
- Управлять историей браузера
- Обрабатывать переходы между страницами

### Основные компоненты React Router

- `<BrowserRouter>` - основной компонент для работы с HTML5 History API
- `<Routes>` - контейнер для маршрутов (в React Router v6+)
- `<Route>` - определяет маршрут и соответствующий компонент
- `<Link>` - создает ссылки для навигации
- `useNavigate` - хук для программной навигации
- `useParams` - хук для получения параметров маршрута
- `useLocation` - хук для получения информации о текущем местоположении

## Установка React Router

```bash
npm install react-router-dom
```

## Базовая реализация

### Настройка маршрутов

```jsx
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Home from './components/Home';
import About from './components/About';
import Contact from './components/Contact';
import NotFound from './components/NotFound';

function App() {
    return (
        <Router>
            <div>
                <nav>
                    <ul>
                        <li><Link to="/">Главная</Link></li>
                        <li><Link to="/about">О нас</Link></li>
                        <li><Link to="/contact">Контакты</Link></li>
                    </ul>
                </nav>

                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="/about" element={<About />} />
                    <Route path="/contact" element={<Contact />} />
                    <Route path="*" element={<NotFound />} />
                </Routes>
            </div>
        </Router>
    );
}

export default App;
```

### Компоненты страниц

```jsx
// Home.jsx
import React from 'react';

function Home() {
    return (
        <div>
            <h1>Главная страница</h1>
            <p>Добро пожаловать на главную страницу!</p>
        </div>
    );
}

export default Home;

// About.jsx
import React from 'react';

function About() {
    return (
        <div>
            <h1>О нас</h1>
            <p>Информация о компании</p>
        </div>
    );
}

export default About;
```

## Продвинутые возможности

### Динамические маршруты

```jsx
import { Routes, Route, useParams } from 'react-router-dom';
import UserDetail from './components/UserDetail';

// В App.jsx
<Route path="/users/:userId" element={<UserDetail />} />

// UserDetail.jsx
function UserDetail() {
    const { userId } = useParams();
    
    return (
        <div>
            <h1>Детали пользователя</h1>
            <p>ID пользователя: {userId}</p>
        </div>
    );
}
```

### Вложенные маршруты

```jsx
import { Routes, Route, Outlet } from 'react-router-dom';

function Layout() {
    return (
        <div>
            <header>Шапка приложения</header>
            <main>
                <Outlet /> {/* Здесь будут отображаться дочерние маршруты */}
            </main>
            <footer>Футер приложения</footer>
        </div>
    );
}

function App() {
    return (
        <Router>
            <Routes>
                <Route path="/" element={<Layout />}>
                    <Route index element={<Home />} />
                    <Route path="about" element={<About />} />
                    <Route path="contact" element={<Contact />} />
                    <Route path="*" element={<NotFound />} />
                </Route>
            </Routes>
        </Router>
    );
}
```

### Защищенные маршруты

```jsx
import { Navigate, Outlet } from 'react-router-dom';

// Компонент для проверки аутентификации
function ProtectedRoute({ isAllowed, redirectTo = '/login' }) {
    return isAllowed ? <Outlet /> : <Navigate to={redirectTo} />;
}

// Использование в маршрутах
<Route 
    path="/dashboard" 
    element={
        <ProtectedRoute isAllowed={isAuthenticated} redirectTo="/login" />
    }
>
    <Route path="" element={<Dashboard />} />
</Route>
```

## Хуки React Router

### useNavigate

```jsx
import { useNavigate } from 'react-router-dom';

function MyComponent() {
    const navigate = useNavigate();
    
    const handleClick = () => {
        navigate('/new-page'); // Программная навигация
    };
    
    const goBack = () => {
        navigate(-1); // Навигация назад
    };
    
    return (
        <div>
            <button onClick={handleClick}>Перейти на новую страницу</button>
            <button onClick={goBack}>Назад</button>
        </div>
    );
}
```

### useLocation

```jsx
import { useLocation } from 'react-router-dom';

function LocationDisplay() {
    const location = useLocation();
    
    return (
        <div>
            <p>Текущий путь: {location.pathname}</p>
            <p>Поиск: {location.search}</p>
        </div>
    );
}
```

### useSearchParams

```jsx
import { useSearchParams } from 'react-router-dom';

function SearchComponent() {
    const [searchParams, setSearchParams] = useSearchParams();
    const query = searchParams.get('q') || '';
    
    const updateQuery = (newQuery) => {
        setSearchParams({ q: newQuery });
    };
    
    return (
        <div>
            <input 
                value={query}
                onChange={(e) => updateQuery(e.target.value)}
                placeholder="Поиск..."
            />
        </div>
    );
}
```

## Ленивая загрузка (Lazy Loading)

```jsx
import React, { Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Home = React.lazy(() => import('./components/Home'));
const About = React.lazy(() => import('./components/About'));
const Contact = React.lazy(() => import('./components/Contact'));

function App() {
    return (
        <Router>
            <Suspense fallback={<div>Загрузка...</div>}>
                <Routes>
                    <Route path="/" element={<Home />} />
                    <Route path="/about" element={<About />} />
                    <Route path="/contact" element={<Contact />} />
                </Routes>
            </Suspense>
        </Router>
    );
}
```

## Роутинг с параметрами

### Параметры маршрута

```jsx
// Маршрут: /users/:userId/posts/:postId
<Route path="/users/:userId/posts/:postId" element={<PostDetail />} />

// PostDetail.jsx
import { useParams } from 'react-router-dom';

function PostDetail() {
    const { userId, postId } = useParams();
    
    return (
        <div>
            <h1>Пост {postId} пользователя {userId}</h1>
        </div>
    );
}
```

### Query параметры

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
    const [searchParams] = useSearchParams();
    const category = searchParams.get('category') || 'all';
    const page = parseInt(searchParams.get('page')) || 1;
    
    return (
        <div>
            <h2>Категория: {category}</h2>
            <h3>Страница: {page}</h3>
        </div>
    );
}
```

## Обработка ошибок

```jsx
import { useRouteError } from 'react-router-dom';

function ErrorBoundary() {
    const error = useRouteError();
    console.error(error);
    
    return (
        <div>
            <h1>Произошла ошибка</h1>
            <p>{error.message}</p>
        </div>
    );
}

// В маршрутах
<Route path="/problematic-route" element={<SomeComponent />} errorElement={<ErrorBoundary />} />
```

## Практические рекомендации

### 1. Структура файлов

```
src/
├── components/
│   ├── Home.jsx
│   ├── About.jsx
│   ├── Contact.jsx
│   └── NotFound.jsx
├── routes/
│   └── index.jsx
└── App.jsx
```

### 2. Централизованное определение маршрутов

```jsx
// routes/index.jsx
export const routes = [
    { path: '/', component: Home, name: 'Главная' },
    { path: '/about', component: About, name: 'О нас' },
    { path: '/contact', component: Contact, name: 'Контакты' },
    { path: '*', component: NotFound, name: '404' }
];
```

### 3. Типизация (с TypeScript)

```tsx
import { RouteObject } from 'react-router-dom';

interface AppRoute {
    path: string;
    component: React.ComponentType;
    name: string;
    protected?: boolean;
}

const routes: AppRoute[] = [
    { path: '/', component: Home, name: 'Главная' },
    { path: '/about', component: About, name: 'О нас' },
    { path: '/dashboard', component: Dashboard, name: 'Панель', protected: true }
];
```

### 4. Анимации при переходе

```jsx
import { motion } from 'framer-motion';

function AnimatedRoutes() {
    return (
        <AnimatePresence mode="wait">
            <Routes>
                <Route 
                    path="/" 
                    element={
                        <motion.div
                            initial={{ opacity: 0, x: -100 }}
                            animate={{ opacity: 1, x: 0 }}
                            exit={{ opacity: 0, x: 100 }}
                            transition={{ duration: 0.3 }}
                        >
                            <Home />
                        </motion.div>
                    } 
                />
            </Routes>
        </AnimatePresence>
    );
}
```

## Сравнение с другими подходами

| Характеристика | React Router | [[Клиентский-роутинг]] | [[Серверный-роутинг]] |
|----------------|--------------|------------------------|----------------------|
| Реализация | Библиотека | Встроенный механизм | Серверный код |
| Сложность | Средняя | Высокая | Низкая |
| SEO | Требует SSR | Требует дополнительной обработки | Лучше по умолчанию |

## Заключение

React Router предоставляет мощный и гибкий способ реализации навигации в React-приложениях. Его возможности позволяют создавать сложные маршруты с параметрами, вложенными компонентами и защитой доступа. Понимание принципов работы роутинга в React важно для создания качественных одностраничных приложений.

Для получения дополнительной информации о роутинге смотрите:
- [[Клиентский-роутинг]]
- [[Серверный-роутинг]]
- [[Роутинг-в-Vue]]
- [[Роутинг-в-Svelte]]

## Ключевые теги

#routing #react #react-router #frontend #javascript #spa #web-development