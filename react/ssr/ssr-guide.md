---
aliases: ["React SSR", "Серверный рендеринг React", "Next.js SSR"]
tags: 
  - #react
  - #ssr
  - #nextjs
  - #seo
  - #performance
---

# Серверный рендеринг React (SSR)

## Что такое SSR

Серверный рендеринг (SSR) — это техника, при которой React-компоненты рендерятся на сервере и отправляются клиенту как готовый HTML. Это позволяет улучшить время загрузки страницы и SEO по сравнению с традиционным клиентским рендерингом (CSR).

```jsx
// Пример серверного рендеринга с ReactDOMServer
import { renderToString } from 'react-dom/server';
import App from './App';

const html = renderToString(<App />);
```

## Преимущества и недостатки SSR

### Преимущества
- **Улучшенное SEO** — поисковые роботы могут индексировать контент
- **Быстрая первоначальная загрузка** — пользователь видит контент быстрее
- **Лучшая производительность на слабых устройствах** — меньше работы для браузера

### Недостатки
- **Увеличенная сложность** — больше кода и конфигурации
- **Более высокая нагрузка на сервер** — каждый запрос требует рендеринга
- **Потенциальные проблемы с кэшированием** — динамический контент сложнее кэшировать

## Когда использовать SSR vs CSR

**Использовать SSR когда:**
- Критически важен SEO (например, блоги, интернет-магазины)
- Контент должен быть виден сразу при загрузке
- Есть пользователи с медленными устройствами/сетью

**Использовать CSR когда:**
- Приложение похоже на десктопное (например, панель управления)
- SEO не важен
- Интерактивность важнее скорости первой загрузки

## Функции SSR в Next.js

### getServerSideProps

Выполняется на сервере при каждом запросе:

```jsx
export async function getServerSideProps(context) {
  const res = await fetch(`https://api.example.com/data`);
  const data = await res.json();

  return {
    props: {
      data
    }
  };
}
```

### getStaticProps

Генерирует страницу во время сборки:

```jsx
export async function getStaticProps(context) {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();

  return {
    props: {
      data
    },
    revalidate: 3600 // перегенерация каждые час
  };
}
```

### getStaticPaths

Используется для динамических маршрутов при статической генерации:

```jsx
export async function getStaticPaths() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  const paths = posts.map((post) => ({
    params: { id: post.id.toString() }
  }));

  return {
    paths,
    fallback: false
  };
}
```

## Традиционная настройка SSR с ReactDOMServer

```jsx
// server.js
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';
import App from './App';

const app = express();

app.get('*', (req, res) => {
  const html = renderToString(<App />);
  res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        <title>My App</title>
      </head>
      <body>
        <div id="root">${html}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `);
});

app.listen(3000);
```

## Получение данных в SSR

Данные должны быть получены до рендеринга компонентов:

```jsx
function Post({ post }) {
  return <div>{post.title}</div>;
}

export async function getServerSideProps({ params }) {
  const res = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await res.json();

  return {
    props: {
      post
    }
  };
}

export default Post;
```

## Стилизация в SSR приложениях

При использовании CSS-in-JS библиотек необходимо экстрагировать стили на сервере:

```jsx
// Emotion SSR
import { extractCritical } from '@emotion/server';

const { html, ids, css } = extractCritical(renderToString(<App />));

// Включить стили в HTML
```

## Роутинг с SSR

В Next.js роутинг интегрирован, но в традиционных SSR приложениях:

```jsx
import { StaticRouter } from 'react-router-dom/server';

// На сервере
<StaticRouter location={req.url}>
  <App />
</StaticRouter>

// На клиенте
<BrowserRouter>
  <App />
</BrowserRouter>
```

## Context API в SSR

Для передачи данных через контекст в SSR:

```jsx
// server.js
const initialData = { /* данные */ };
const contextValue = { data: initialData };

const appHtml = renderToString(
  <Context.Provider value={contextValue}>
    <App />
  </Context.Provider>
);

// Передать в клиент
```

## Обработка кода специфичного для браузера

Избегайте использования window, document и других браузерных API на сервере:

```jsx
useEffect(() => {
  // Этот код выполнится только в браузере
  if (typeof window !== 'undefined') {
    // код для браузера
  }
}, []);

// Или с помощью условной проверки
const isBrowser = typeof window !== 'undefined';
```

## Процесс гидратации

Клиент берет на себя управление после SSR:

```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

## Соображения производительности при SSR

- Минимизировать время выполнения серверного рендеринга
- Использовать кэширование результатов
- Оптимизировать размер бандла
- Использовать код-сплиттинг

## SEO преимущества SSR

- Поисковые роботы видят полный HTML
- Контент индексируется сразу
- Метатеги могут быть динамически сгенерированы

## Размер бандла в SSR

- Серверный бандл должен быть легким
- Клиентский бандл оптимизируется как обычно
- Использовать tree-shaking и код-сплиттинг

## Стратегии кэширования

- Кэширование на уровне страницы
- Кэширование результатов API
- Использование CDN для статических ресурсов

## Соображения при деплое

- Требуется серверное окружение (Node.js)
- Обработка ошибок сервера
- Логирование и мониторинг

## Безопасность в SSR

- Санитизация ввода пользователя
- Защита от XSS атак
- Правильная обработка заголовков

## Обработка ошибок в SSR

```jsx
export async function getServerSideProps() {
  try {
    const res = await fetch('...');
    if (!res.ok) throw new Error('API error');
    const data = await res.json();
    return { props: { data } };
  } catch (error) {
    return {
      props: { error: error.message }
    };
  }
}
```

## Отладка SSR приложений

- Использовать логирование на сервере
- Проверять различия между серверным и клиентским рендерингом
- Использовать инструменты разработчика браузера

## Тестирование SSR

```jsx
// Тестирование серверного рендеринга
import { renderToString } from 'react-dom/server';

test('renders component on server', () => {
  const html = renderToString(<App />);
  expect(html).toContain('expected content');
});
```

## Связи с другими файлами React

- [[react/react-components]] - для понимания компонентного подхода
- [[react/react-hooks]] - для работы с состоянием в SSR
- [[react/react-performance]] - для оптимизации SSR приложений
- [[react/react-context]] - для передачи данных в SSR
- [[react/react-routing]] - для роутинга в SSR приложениях
- [[nextjs/nextjs-ssg]] - для понимания статической генерации
- [[seo/seo-optimization]] - для SEO аспектов SSR