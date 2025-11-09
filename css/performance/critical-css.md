# Производительность - Критический CSS

Критический CSS (Critical CSS) - это подмножество CSS-правил, необходимых для отображения выше склада (above-the-fold) контента страницы. Правильное извлечение и встраивание критического CSS позволяет ускорить начальную загрузку страницы и улучшить пользовательский опыт.

## Что такое критический CSS

Критический CSS включает в себя только те стили, которые необходимы для отображения видимой части страницы при первой загрузке. Все остальные стили могут быть загружены позже, что улучшает время отображения контента.

### Пример критического и некритического CSS

```css
/* КРИТИЧЕСКИЙ CSS - необходим для отображения основного контента */
.header { height: 60px; background: white; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.main-content { min-height: 400px; padding: 20px; }
.cta-button { padding: 12px 24px; background: #007bff; color: white; border: none; }

/* НЕКРИТИЧЕСКИЙ CSS - используется для элементов, не видимых при загрузке */
.testimonials { margin-top: 100px; } /* Ниже видимой области */
.footer { margin-top: 50px; } /* Ниже видимой области */
```

## Методы извлечения критического CSS

### 1. Ручное извлечение

```html
<!DOCTYPE html>
<html>
<head>
  <title>Пример с критическим CSS</title>
  <!-- Встроенный критический CSS -->
  <style>
    /* Критические стили */
    * { box-sizing: border-box; }
    body { margin: 0; font-family: Arial, sans-serif; }
    .header { height: 60px; background: white; display: flex; align-items: center; }
    .logo { font-size: 1.5rem; font-weight: bold; color: #007bff; }
    .main-content { min-height: 400px; padding: 20px; }
    .hero { text-align: center; padding: 40px 20px; }
    .hero-title { font-size: 2.5rem; margin-bottom: 20px; }
    .cta-button { padding: 15px 30px; background: #007bff; color: white; 
                  border: none; border-radius: 4px; font-size: 1.1rem; }
  </style>
  
  <!-- Отложенная загрузка остальных стилей -->
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
<body>
  <header class="header">
    <div class="logo">Мой Сайт</div>
  </header>
  
  <main class="main-content">
    <section class="hero">
      <h1 class="hero-title">Добро пожаловать!</h1>
      <button class="cta-button">Начать</button>
    </section>
  </main>
  
  <!-- Контент, который появляется ниже видимой области -->
  <section class="testimonials">
    <h2>Отзывы клиентов</h2>
    <!-- Много контента -->
  </section>
  
  <footer class="footer">
    <p>&copy; 2023 Все права защищены</p>
  </footer>
</body>
</html>
```

### 2. Автоматическое извлечение с инструментами

#### Penthouse

```javascript
// penthouse-config.js
const penthouse = require('penthouse');

penthouse({
  url: 'http://localhost:3000',
  width: 1300,
  height: 900,
  cssString: 'path/to/your/stylesheet.css',
  forceInclude: [
    '.keep-me',
    '.even-if-not-visible'
  ]
}).then(criticalCss => {
  // Сохраняем критический CSS в файл
  require('fs').writeFileSync('critical.css', criticalCss);
});
```

#### Critical (инструмент командной строки)

```bash
# Установка
npm install -g critical

# Извлечение критического CSS
critical index.html > critical.css

# С указанием размеров
critical index.html --width 1300 --height 900 > critical.css

# С указанием CSS файла
critical index.html --css styles.css --width 1300 --height 900 > critical.css
```

## Практические примеры

### Пример 1: Веб-сайт с критическим CSS

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Оптимизированный сайт</title>
  
  <!-- Встроенный критический CSS -->
  <style>
    /* Сброс стилей */
    * { box-sizing: border-box; }
    body { margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
    
    /* Структура страницы */
    .container { max-width: 1200px; margin: 0 auto; padding: 0 15px; }
    
    /* Шапка */
    .header { 
      height: 70px; 
      background: white; 
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
      display: flex;
      align-items: center;
    }
    
    .logo { 
      font-size: 1.5rem; 
      font-weight: 700; 
      color: #007bff; 
      text-decoration: none;
    }
    
    /* Герой-секция */
    .hero { 
      min-height: 80vh; 
      display: flex; 
      align-items: center; 
      justify-content: center; 
      text-align: center;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
    }
    
    .hero-title { 
      font-size: 3rem; 
      margin-bottom: 1rem; 
      font-weight: 700;
    }
    
    .hero-subtitle { 
      font-size: 1.2rem; 
      margin-bottom: 2rem; 
      opacity: 0.9;
    }
    
    .cta-button { 
      padding: 15px 30px; 
      background: white; 
      color: #667eea; 
      border: none; 
      border-radius: 50px; 
      font-size: 1.1rem; 
      font-weight: 600; 
      cursor: pointer;
      transition: transform 0.2s ease, box-shadow 0.2s ease;
    }
    
    .cta-button:hover { 
      transform: translateY(-2px); 
      box-shadow: 0 5px 15px rgba(0,0,0,0.2);
    }
    
    /* Навигация */
    .nav { 
      margin-left: auto; 
      display: flex; 
      gap: 2rem;
    }
    
    .nav-link { 
      text-decoration: none; 
      color: #333; 
      font-weight: 500;
    }
  </style>
  
  <!-- Отложенная загрузка остальных стилей -->
  <link rel="preload" href="styles.css" as="style" onload="this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
<body>
  <header class="header">
    <div class="container">
      <a href="/" class="logo">MySite</a>
      <nav class="nav">
        <a href="/about" class="nav-link">О нас</a>
        <a href="/services" class="nav-link">Услуги</a>
        <a href="/contact" class="nav-link">Контакты</a>
      </nav>
    </div>
  </header>
  
  <main>
    <section class="hero">
      <div class="container">
        <h1 class="hero-title">Создаем цифровые решения</h1>
        <p class="hero-subtitle">Профессиональные веб-решения для вашего бизнеса</p>
        <button class="cta-button">Начать проект</button>
      </div>
    </section>
    
    <!-- Этот контент ниже видимой области и будет в остальных стилях -->
    <section class="features">
      <div class="container">
        <h2>Наши услуги</h2>
        <!-- Много контента -->
      </div>
    </section>
  </main>
</body>
</html>
```

### Пример 2: Критический CSS для электронной коммерции

```html
<!DOCTYPE html>
<html>
<head>
  <title>Магазин - Главная</title>
  
  <!-- Встроенный критический CSS -->
  <style>
    /* Базовые стили */
    * { box-sizing: border-box; }
    body { margin: 0; font-family: system-ui; }
    
    /* Шапка магазина */
    .header { 
      height: 60px; 
      background: white; 
      border-bottom: 1px solid #e0e0e0;
      position: sticky;
      top: 0;
      z-index: 100;
    }
    
    .header-container { 
      max-width: 1200px; 
      margin: 0 auto; 
      padding: 0 15px; 
      display: flex;
      align-items: center;
    }
    
    .logo { 
      font-size: 1.5rem; 
      font-weight: bold; 
      color: #e74c3c; 
      text-decoration: none;
      margin-right: 2rem;
    }
    
    /* Поиск */
    .search-form { 
      flex: 1; 
      max-width: 500px;
    }
    
    .search-input { 
      width: 100%; 
      padding: 10px 15px; 
      border: 1px solid #ddd; 
      border-radius: 4px;
      font-size: 1rem;
    }
    
    /* Корзина */
    .cart-icon { 
      width: 40px; 
      height: 40px; 
      background: #e74c3c; 
      color: white; 
      border-radius: 50%; 
      display: flex; 
      align-items: center; 
      justify-content: center; 
      text-decoration: none;
      margin-left: 1rem;
      position: relative;
    }
    
    .cart-count { 
      position: absolute; 
      top: -5px; 
      right: -5px; 
      background: yellow; 
      color: black; 
      border-radius: 50%; 
      width: 18px; 
      height: 18px; 
      font-size: 0.7rem; 
      display: flex; 
      align-items: center; 
      justify-content: center;
    }
    
    /* Герой-баннер */
    .hero-banner { 
      height: 400px; 
      background: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('banner-bg.jpg');
      background-size: cover;
      background-position: center;
      display: flex;
      align-items: center;
      justify-content: center;
      color: white;
      text-align: center;
    }
    
    .banner-content { 
      max-width: 600px; 
      padding: 20px;
    }
    
    .banner-title { 
      font-size: 2.5rem; 
      margin-bottom: 1rem;
    }
    
    .banner-subtitle { 
      font-size: 1.2rem; 
      margin-bottom: 2rem;
    }
    
    /* Рекомендуемые товары */
    .featured-products { 
      padding: 40px 0; 
    }
    
    .section-title { 
      text-align: center; 
      margin-bottom: 2rem; 
      font-size: 1.8rem;
    }
    
    .product-grid { 
      max-width: 1200px; 
      margin: 0 auto; 
      padding: 0 15px; 
      display: grid; 
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); 
      gap: 20px;
    }
    
    .product-card { 
      border: 1px solid #eee; 
      border-radius: 8px; 
      overflow: hidden; 
      text-align: center;
      transition: transform 0.2s ease, box-shadow 0.2s ease;
    }
    
    .product-card:hover { 
      transform: translateY(-5px); 
      box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    }
    
    .product-image { 
      width: 100%; 
      height: 200px; 
      object-fit: cover;
    }
    
    .product-title { 
      padding: 10px; 
      margin: 0; 
      font-weight: 500;
    }
    
    .product-price { 
      padding: 0 10px 15px; 
      font-weight: bold; 
      color: #e74c3c;
    }
  </style>
  
  <!-- Отложенная загрузка остальных стилей -->
  <link rel="preload" href="ecommerce.css" as="style" onload="this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="ecommerce.css"></noscript>
</head>
<body>
  <header class="header">
    <div class="header-container">
      <a href="/" class="logo">ShopName</a>
      <form class="search-form">
        <input type="text" class="search-input" placeholder="Поиск товаров...">
      </form>
      <a href="/cart" class="cart-icon">
        🛒
        <span class="cart-count">0</span>
      </a>
    </div>
  </header>
  
  <main>
    <section class="hero-banner">
      <div class="banner-content">
        <h1 class="banner-title">Сезонные скидки до 50%</h1>
        <p class="banner-subtitle">Лучшие товары по отличным ценам</p>
        <button class="cta-button">Посмотреть акции</button>
      </div>
    </section>
    
    <section class="featured-products">
      <h2 class="section-title">Рекомендуемые товары</h2>
      <div class="product-grid">
        <div class="product-card">
          <img src="product1.jpg" alt="Товар 1" class="product-image">
          <h3 class="product-title">Товар 1</h3>
          <div class="product-price">$29.99</div>
        </div>
        <div class="product-card">
          <img src="product2.jpg" alt="Товар 2" class="product-image">
          <h3 class="product-title">Товар 2</h3>
          <div class="product-price">$39.99</div>
        </div>
        <div class="product-card">
          <img src="product3.jpg" alt="Товар 3" class="product-image">
          <h3 class="product-title">Товар 3</h3>
          <div class="product-price">$49.99</div>
        </div>
      </div>
    </section>
  </main>
</body>
</html>
```

## Современные подходы

### 1. Критический CSS с Webpack

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const Critters = require('critters/webpack');

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css',
    }),
    new Critters({
      // Опции для извлечения критического CSS
      preload: 'media',
      inlineFonts: true,
      preloadFonts: true,
    }),
  ],
};
```

### 2. Критический CSS с PostCSS

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-critical-split')({
      // Настройки для разделения критического и некритического CSS
      height: 500,
      width: 1200,
      extract: true,
    }),
  ],
};
```

### 3. Критический CSS в React-приложениях

```jsx
// components/CriticalCSS.js
import { Helmet } from 'react-helmet';

const CriticalCSS = () => (
  <Helmet>
    <style>{`
      * { box-sizing: border-box; }
      body { margin: 0; font-family: -apple-system, BlinkMacSystemFont, sans-serif; }
      .header { height: 60px; background: white; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
      .main-content { min-height: 400px; }
      .cta-button { padding: 12px 24px; background: #007bff; color: white; }
    `}</style>
    <link rel="preload" href="/styles.css" as="style" onLoad="this.onload=null;this.rel='stylesheet'" />
  </Helmet>
);

export default CriticalCSS;
```

## Инструменты для работы с критическим CSS

### 1. Critical (Node.js)

```javascript
const critical = require('critical');

critical.generate({
  base: 'path/to/base',
  src: 'index.html',
  dest: 'index-critical.html',
  inline: true,
  width: 1337,
  height: 800,
  // Другие опции...
});
```

### 2. Penthouse

```javascript
const penthouse = require('penthouse');

penthouse({
  url: 'http://localhost:8080',
  width: 1300,
  height: 900,
  cssString: 'body{margin:0;}', // или путь к CSS файлу
  timeout: 30000,
  strict: false,
  maxEmbeddedBase64Length: 1000,
  renderWaitTime: 2000,
  blockJSRequests: true,
}).then(criticalCss => {
  console.log(criticalCss);
});
```

### 3. Inline Critical

```bash
# Установка
npm install -g inline-critical

# Использование
inline-critical index.html styles.css --width 1300 --height 900 --strategy inline --extract
```

## Лучшие практики

### 1. Определение критического размера

```css
/* Рекомендуемый размер критического CSS: до 14 КБ */
/* Это пример оптимизированного критического CSS */

/* Базовые стили */
* { box-sizing: border-box; }
body { margin: 0; font-family: system-ui; }

/* Шапка */
.header { height: 60px; background: white; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }

/* Основной контент */
.main { min-height: 400px; padding: 20px; }

/* Кнопки */
.btn { padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; }

/* Типографика */
h1 { font-size: 2rem; margin: 0.67em 0; }
p { margin: 1em 0; }

/* Сетка */
.container { max-width: 1200px; margin: 0 auto; padding: 0 15px; }
```

### 2. Тестирование и мониторинг

```javascript
// Пример тестирования производительности
function measureFCP() {
  new PerformanceObserver((entryList) => {
    for (const entry of entryList.getEntries()) {
      console.log('First Contentful Paint:', entry.startTime);
      // Отправка метрики в аналитику
    }
  }).observe({ entryTypes: ['paint'] });
}

// Вызов при загрузке
document.addEventListener('DOMContentLoaded', measureFCP);
```

### 3. Управление обновлениями

```javascript
// Скрипт для обновления критического CSS при изменениях
const fs = require('fs');
const path = require('path');
const penthouse = require('penthouse');

async function updateCriticalCSS() {
  const criticalCss = await penthouse({
    url: 'http://localhost:3000',
    width: 1300,
    height: 900,
    cssString: fs.readFileSync('styles.css', 'utf8'),
  });
  
  fs.writeFileSync('critical.css', criticalCss);
  console.log('Критический CSS обновлен');
}

// Запуск при сборке
updateCriticalCSS();
```

## Заключение

Критический CSS - важный элемент оптимизации производительности веб-сайтов. Правильное извлечение и встраивание критических стилей может значительно улучшить время отображения контента и пользовательский опыт. Важно использовать подходящие инструменты и регулярно тестировать эффективность оптимизации. Комбинация ручной и автоматической оптимизации дает наилучшие результаты.

#programming #css #critical-css #performance #web-development #frontend