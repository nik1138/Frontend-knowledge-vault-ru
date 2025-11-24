---
aliases: [Ленивая загрузка, Lazy Loading, Отложенная загрузка]
tags: [frontend, performance, optimization, loading]
---

# lazy-loading

## Введение

Lazy loading (ленивая загрузка) - это техника оптимизации производительности, при которой ресурсы загружаются только тогда, когда они действительно необходимы. Это особенно важно для веб-страниц с большим количеством изображений, видео или других тяжелых ресурсов.

## Принцип работы

Ленивая загрузка работает по следующему принципу:
1. Сначала загружаются только ресурсы, видимые пользователю (above the fold)
2. Ресурсы, находящиеся ниже по странице, загружаются только при приближении к ним
3. Это уменьшает начальное время загрузки и потребление трафика

## Ленивая загрузка изображений

### С использованием атрибута loading

Современные браузеры поддерживают встроенный атрибут `loading`:

```html
<!-- Ленивая загрузка изображений -->
<img src="image.jpg" alt="Описание" loading="lazy" />

<!-- Нормальная загрузка для важных изображений -->
<img src="hero-image.jpg" alt="Главное изображение" loading="eager" />
```

### Реализация с Intersection Observer API

```javascript
// Ленивая загрузка изображений с использованием Intersection Observer
const imageObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const img = entry.target;
            
            // Замена placeholder на реальное изображение
            img.src = img.dataset.src;
            img.classList.remove('lazy');
            
            // Удаление обсервера после загрузки
            observer.unobserve(img);
        }
    });
});

// Применение к изображениям с классом lazy
document.querySelectorAll('img[data-src]').forEach(img => {
    imageObserver.observe(img);
});
```

```html
<!-- HTML-разметка для ленивой загрузки -->
<img 
    class="lazy" 
    data-src="real-image.jpg" 
    src="placeholder.jpg" 
    alt="Описание изображения" 
/>
```

### Стили для плавного перехода

```css
img.lazy {
    opacity: 0;
    transition: opacity 0.3s;
}

img:not(.lazy) {
    opacity: 1;
}

/* Заглушка для улучшения UX */
.lazy-placeholder {
    background-color: #f0f0f0;
    min-height: 200px;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

## Ленивая загрузка компонентов в React

### React.lazy и Suspense

```jsx
import { lazy, Suspense } from 'react';

// Ленивая загрузка компонента
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
    return (
        <div>
            <header>Важные компоненты</header>
            <Suspense fallback={<div>Загрузка...</div>}>
                <HeavyComponent />
            </Suspense>
        </div>
    );
}
```

### Условная загрузка с React Router

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ContactPage = lazy(() => import('./pages/ContactPage'));

function AppRoutes() {
    return (
        <Suspense fallback={<div>Загрузка страницы...</div>}>
            <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/about" element={<AboutPage />} />
                <Route path="/contact" element={<ContactPage />} />
            </Routes>
        </Suspense>
    );
}
```

## Ленивая загрузка видео

### HTML5 видео с lazy loading

```html
<!-- Ленивая загрузка видео -->
<video 
    preload="none" 
    controls 
    poster="preview.jpg"
    data-src="video.mp4">
    <source data-src="video.mp4" type="video/mp4">
</video>
```

```javascript
// Скрипт для ленивой загрузки видео
const videoObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const video = entry.target;
            const source = video.querySelector('source');
            
            // Загрузка видео только при видимости
            source.src = source.dataset.src;
            video.load();
            
            observer.unobserve(video);
        }
    });
});

document.querySelectorAll('video[data-src]').forEach(video => {
    videoObserver.observe(video);
});
```

## Ленивая загрузка данных

### С использованием React Query

```jsx
import { useQuery } from 'react-query';

function UserProfile({ userId }) {
    // Запрос данных только при необходимости
    const { data, isLoading, isError } = useQuery(
        ['user', userId],
        () => fetchUser(userId),
        {
            enabled: !!userId, // Выполнять запрос только при наличии userId
            staleTime: 5 * 60 * 1000, // Данные считаются свежими 5 минут
        }
    );

    if (isLoading) return <div>Загрузка профиля...</div>;
    if (isError) return <div>Ошибка загрузки</div>;

    return <div>{data.name}</div>;
}
```

## Продвинутые техники

### Предзагрузка с приоритетами

```javascript
// Умная предзагрузка с учетом скорости соединения
function preloadBasedOnConnection() {
    const connection = navigator.connection || navigator.mozConnection || navigator.webkitConnection;
    
    if (connection) {
        const effectiveType = connection.effectiveType;
        
        if (effectiveType === 'slow-2g' || effectiveType === '2g') {
            // Загружать только критические ресурсы
            return 'critical';
        } else if (effectiveType === '3g') {
            // Загружать ресурсы с низким приоритетом
            return 'low-priority';
        } else {
            // Полная загрузка
            return 'full';
        }
    }
    
    return 'full'; // По умолчанию
}
```

### Ленивая загрузка шрифтов

```css
/* Определение шрифтов с ленивой загрузкой */
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
    font-display: swap; /* Показывать fallback шрифт до загрузки */
}
```

## Лучшие практики

1. **Используйте Intersection Observer** вместо обработчиков прокрутки для лучшей производительности
2. **Добавляйте placeholder'ы** для улучшения пользовательского опыта
3. **Предзагружайте ресурсы** с высоким приоритетом
4. **Тестируйте на слабых устройствах** и медленных соединениях
5. **Измеряйте влияние** lazy loading на UX и производительность

## Потенциальные проблемы

- **SEO влияние** - поисковые роботы могут не увидеть лениво загружаемый контент
- **Долгая загрузка** - при неправильной реализации может замедлить отображение
- **Сложность отладки** - усложняет процесс тестирования и отладки

## См. также

- [[Оптимизация-рендеринга]]
- [[Кэширование]]
- [[Intersection Observer API]]
- [[React Performance Patterns]]