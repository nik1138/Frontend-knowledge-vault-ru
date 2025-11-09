# CSS фреймворки: Tailwind CSS, Bootstrap, Bulma и другие

## Введение

CSS фреймворки предоставляют готовые компоненты, сетки и утилиты для быстрой разработки веб-интерфейсов. Они значительно ускоряют процесс разработки, обеспечивая согласованность и адаптивность. В этой статье мы рассмотрим популярные CSS фреймворки, их особенности, преимущества и недостатки.

## Популярные CSS фреймворки

### 1. Tailwind CSS

Tailwind CSS - это utility-first CSS фреймворк, который предоставляет низкоуровневые утилиты для создания пользовательских дизайнов без написания пользовательского CSS.

#### Основные особенности Tailwind

```html
<!-- Пример использования Tailwind CSS -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tailwind Example</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="max-w-md w-full bg-white rounded-lg shadow-lg p-6">
        <h1 class="text-2xl font-bold text-gray-800 mb-4">Привет, Tailwind!</h1>
        <p class="text-gray-600 mb-6">Это пример компонента с использованием Tailwind CSS.</p>
        
        <div class="flex space-x-4">
            <button class="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded transition duration-300">
                Primary
            </button>
            <button class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-medium py-2 px-4 rounded transition duration-300">
                Secondary
            </button>
        </div>
    </div>
</body>
</html>
```

#### Продвинутые возможности Tailwind

```html
<!-- Пример сложного компонента -->
<div class="relative overflow-hidden rounded-2xl bg-gradient-to-r from-blue-500 to-purple-600 p-0.5 shadow-xl">
    <div class="rounded-[calc(1.5rem-1px)] bg-white p-6">
        <div class="flex items-center justify-between">
            <div>
                <h3 class="text-lg font-semibold text-gray-900">Заголовок</h3>
                <p class="mt-1 text-sm text-gray-500">Описание</p>
            </div>
            <div class="flex-shrink-0">
                <span class="inline-flex items-center rounded-full bg-green-100 px-3 py-1 text-xs font-medium text-green-800">
                    Active
                </span>
            </div>
        </div>
        
        <div class="mt-6">
            <div class="flex items-center">
                <div class="h-2 flex-1 rounded-full bg-gray-200">
                    <div class="h-2 rounded-full bg-blue-500" style="width: 75%"></div>
                </div>
                <span class="ml-3 text-sm font-medium text-gray-900">75%</span>
            </div>
        </div>
        
        <div class="mt-6 flex justify-between">
            <button class="rounded-md bg-blue-600 px-4 py-2 text-sm font-semibold text-white shadow-sm hover:bg-blue-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">
                Действие
            </button>
            <button class="rounded-md bg-white px-4 py-2 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50">
                Отмена
            </button>
        </div>
    </div>
</div>
```

#### Конфигурация Tailwind

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
    './app/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
      animation: {
        'fade-in': 'fadeIn 0.5s ease-in-out',
        'slide-up': 'slideUp 0.3s ease-out',
      },
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        slideUp: {
          '0%': { transform: 'translateY(20px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      }
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
    require('@tailwindcss/aspect-ratio'),
  ],
}
```

#### Плагины и расширения Tailwind

```javascript
// custom-plugin.js
const plugin = require('tailwindcss/plugin');

module.exports = plugin(function({ addUtilities, theme, e }) {
  addUtilities({
    '.text-shadow': {
      textShadow: '0 2px 4px rgba(0, 0, 0, 0.1)',
    },
    '.text-shadow-lg': {
      textShadow: '0 4px 8px rgba(0, 0, 0, 0.12), 0 2px 4px rgba(0, 0, 0, 0.08)',
    },
    '.grayscale': {
      filter: 'grayscale(100%)',
    },
  });
});
```

### 2. Bootstrap

Bootstrap - один из самых популярных CSS фреймворков, предоставляет готовые компоненты и адаптивную сетку.

#### Основные особенности Bootstrap

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bootstrap Example</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">
                        <h5 class="card-title">Заголовок карточки</h5>
                    </div>
                    <div class="card-body">
                        <p class="card-text">Содержимое карточки...</p>
                        <a href="#" class="btn btn-primary">Действие</a>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="list-group">
                    <a href="#" class="list-group-item list-group-item-action active">Элемент 1</a>
                    <a href="#" class="list-group-item list-group-item-action">Элемент 2</a>
                    <a href="#" class="list-group-item list-group-item-action">Элемент 3</a>
                </div>
            </div>
        </div>
    </div>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

#### Продвинутые компоненты Bootstrap

```html
<!-- Пример сложного компонента -->
<div class="container">
    <div class="row">
        <div class="col-lg-12">
            <nav aria-label="breadcrumb">
                <ol class="breadcrumb">
                    <li class="breadcrumb-item"><a href="#">Главная</a></li>
                    <li class="breadcrumb-item"><a href="#">Библиотека</a></li>
                    <li class="breadcrumb-item active" aria-current="page">Данные</li>
                </ol>
            </nav>
        </div>
    </div>

    <div class="row">
        <div class="col-lg-8">
            <div class="card mb-4">
                <div class="card-header">
                    <ul class="nav nav-tabs card-header-tabs" id="myTab" role="tablist">
                        <li class="nav-item" role="presentation">
                            <button class="nav-link active" id="home-tab" data-bs-toggle="tab" data-bs-target="#home" type="button" role="tab">Главная</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="profile-tab" data-bs-toggle="tab" data-bs-target="#profile" type="button" role="tab">Профиль</button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" id="contact-tab" data-bs-toggle="tab" data-bs-target="#contact" type="button" role="tab">Контакты</button>
                        </li>
                    </ul>
                </div>
                <div class="card-body">
                    <div class="tab-content" id="myTabContent">
                        <div class="tab-pane fade show active" id="home" role="tabpanel">
                            <h5>Главная вкладка</h5>
                            <p>Содержимое главной вкладки...</p>
                        </div>
                        <div class="tab-pane fade" id="profile" role="tabpanel">
                            <h5>Профиль</h5>
                            <p>Содержимое профиля...</p>
                        </div>
                        <div class="tab-pane fade" id="contact" role="tabpanel">
                            <h5>Контакты</h5>
                            <p>Содержимое контактов...</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="col-lg-4">
            <div class="card">
                <div class="card-header">
                    <h6 class="mb-0">Прогресс</h6>
                </div>
                <div class="card-body">
                    <div class="progress mb-3">
                        <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" style="width: 75%"></div>
                    </div>
                    
                    <div class="d-grid gap-2">
                        <button class="btn btn-primary" type="button">Primary</button>
                        <button class="btn btn-outline-secondary" type="button">Secondary</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

#### Кастомизация Bootstrap

```scss
// custom-bootstrap.scss
// Импортируем переменные Bootstrap
@import "bootstrap/scss/functions";
@import "bootstrap/scss/variables";
@import "bootstrap/scss/mixins";

// Переопределяем переменные
$primary: #ff6b6b;
$secondary: #4ecdc4;
$success: #51cf66;
$info: #339af0;
$warning: #fcc419;
$danger: #ff6b6b;

$font-family-sans-serif: 'Inter', 'Segoe UI', sans-serif;
$border-radius: 8px;
$border-radius-lg: 12px;
$border-radius-sm: 4px;

// Импортируем остальные компоненты
@import "bootstrap/scss/bootstrap";
```

### 3. Bulma

Bulma - это современный CSS фреймворк, основанный на Flexbox.

#### Основные особенности Bulma

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bulma Example</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
</head>
<body>
    <section class="section">
        <div class="container">
            <div class="columns">
                <div class="column is-8">
                    <div class="card">
                        <div class="card-image">
                            <figure class="image is-4by3">
                                <img src="https://placehold.co/400x300" alt="Placeholder image">
                            </figure>
                        </div>
                        <div class="card-content">
                            <div class="media">
                                <div class="media-content">
                                    <p class="title is-4">Заголовок</p>
                                    <p class="subtitle is-6">@username</p>
                                </div>
                            </div>
                            
                            <div class="content">
                                <p>Содержимое карточки...</p>
                                <a>@someone</a> mentioned <a>@you</a> in a comment.
                            </div>
                        </div>
                    </div>
                </div>
                
                <div class="column is-4">
                    <div class="box">
                        <h3 class="title is-5">Боковая панель</h3>
                        <div class="content">
                            <ul>
                                <li>Элемент 1</li>
                                <li>Элемент 2</li>
                                <li>Элемент 3</li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</body>
</html>
```

#### Продвинутые возможности Bulma

```html
<!-- Пример сложного макета -->
<section class="hero is-primary">
    <div class="hero-body">
        <div class="container">
            <h1 class="title">Заголовок героя</h1>
            <p class="subtitle">Подзаголовок героя</p>
        </div>
    </div>
</section>

<section class="section">
    <div class="container">
        <div class="columns is-multiline">
            <div class="column is-4-desktop is-6-tablet">
                <div class="card">
                    <div class="card-content">
                        <div class="content">
                            <h3 class="title is-4">Карточка 1</h3>
                            <p>Описание карточки 1</p>
                        </div>
                    </div>
                </div>
            </div>
            
            <div class="column is-4-desktop is-6-tablet">
                <div class="card">
                    <div class="card-content">
                        <div class="content">
                            <h3 class="title is-4">Карточка 2</h3>
                            <p>Описание карточки 2</p>
                        </div>
                    </div>
                </div>
            </div>
            
            <div class="column is-4-desktop is-6-tablet">
                <div class="card">
                    <div class="card-content">
                        <div class="content">
                            <h3 class="title is-4">Карточка 3</h3>
                            <p>Описание карточки 3</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        
        <div class="field is-grouped is-grouped-centered">
            <p class="control">
                <button class="button is-primary">Primary</button>
            </p>
            <p class="control">
                <button class="button is-light">Light</button>
            </p>
        </div>
    </div>
</section>
```

### 4. Foundation

Foundation - это адаптивный CSS фреймворк от Zurb.

#### Основные особенности Foundation

```html
<!DOCTYPE html>
<html class="no-js" lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Foundation Example</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/foundation-sites@6.7.5/dist/css/foundation.min.css">
</head>
<body>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <div class="cell small-12">
                <h1>Foundation Grid</h1>
            </div>
        </div>
        
        <div class="grid-x grid-padding-x">
            <div class="cell medium-8">
                <div class="callout">
                    <h5>Основной контент</h5>
                    <p>Здесь находится основной контент страницы.</p>
                </div>
            </div>
            <div class="cell medium-4">
                <div class="callout">
                    <h5>Боковая панель</h5>
                    <p>Дополнительная информация.</p>
                </div>
            </div>
        </div>
    </div>
    
    <script src="https://cdn.jsdelivr.net/npm/foundation-sites@6.7.5/dist/js/foundation.min.js"></script>
    <script>
        $(document).foundation();
    </script>
</body>
</html>
```

## Сравнение фреймворков

### Таблица сравнения

| Функция | Tailwind CSS | Bootstrap | Bulma | Foundation |
|---------|--------------|-----------|-------|------------|
| Подход | Utility-first | Компонентный | Flexbox-based | Grid-based |
| Размер (gzipped) | ~3KB (после purge) | ~20KB | ~200KB | ~25KB |
| Адаптивность | Excellent | Excellent | Excellent | Excellent |
| Настройка | Высокая | Средняя | Средняя | Средняя |
| Изучение | Среднее | Легкое | Среднее | Среднее |
| Производительность | Высокая | Средняя | Средняя | Средняя |
| Сообщество | Большое | Очень большое | Среднее | Среднее |
| JS компоненты | Минимум | Много | Немного | Немного |

### Производительность и размер

```html
<!-- Сравнение размеров CSS файлов -->
<!-- Tailwind (после purge и minification): ~3-10KB -->
<!-- Bootstrap: ~20KB -->
<!-- Bulma: ~150-200KB -->
<!-- Foundation: ~25KB -->

<!-- Пример минимального Tailwind CSS после purge -->
/*
! tailwindcss v3.3.0 | MIT License | https://tailwindcss.com
*//*.bg-white{--tw-bg-opacity:1;background-color:rgb(255 255 255/var(--tw-bg-opacity))}* ...
*/
```

## Современные тенденции

### 1. Utility-first подход

```html
<!-- Пример utility-first подхода -->
<div class="bg-white rounded-lg shadow-md p-6 max-w-md mx-auto">
    <div class="flex items-center space-x-4 mb-4">
        <img class="w-12 h-12 rounded-full" src="avatar.jpg" alt="Avatar">
        <div>
            <h3 class="font-semibold text-lg">Имя пользователя</h3>
            <p class="text-gray-500 text-sm">@username</p>
        </div>
    </div>
    
    <p class="text-gray-700 mb-4">
        Это пример использования utility-first подхода. Каждый класс решает конкретную задачу.
    </p>
    
    <div class="flex justify-between items-center">
        <span class="text-sm text-gray-500">2 часа назад</span>
        <button class="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded transition duration-300">
            Подробнее
        </button>
    </div>
</div>
```

### 2. JIT (Just-In-Time) режим в Tailwind

```javascript
// tailwind.config.js с JIT режимом
module.exports = {
  mode: 'jit', // Включает JIT компиляцию
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### 3. Кастомные компоненты с фреймворками

```html
<!-- Пример кастомного компонента с Tailwind -->
<div class="custom-card group">
    <div class="bg-white rounded-xl shadow-lg overflow-hidden transition-all duration-300 group-hover:shadow-xl">
        <div class="p-6">
            <div class="flex items-start space-x-4">
                <div class="flex-shrink-0">
                    <div class="bg-gray-200 border-2 border-dashed rounded-xl w-16 h-16" />
                </div>
                <div class="min-w-0 flex-1">
                    <h3 class="text-lg font-semibold text-gray-900 truncate">Заголовок</h3>
                    <p class="text-sm text-gray-500 line-clamp-2">Описание компонента...</p>
                </div>
            </div>
            
            <div class="mt-6 flex items-center justify-between">
                <div class="flex items-center">
                    <div class="flex items-center">
                        <svg class="h-5 w-5 text-yellow-400" fill="currentColor" viewBox="0 0 20 20">
                            <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" />
                        </svg>
                        <span class="ml-1 text-sm font-medium text-gray-900">4.7</span>
                        <span class="ml-1 text-sm font-medium text-gray-500">из 5</span>
                    </div>
                </div>
                
                <button class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                    Действие
                </button>
            </div>
        </div>
    </div>
</div>
```

## Лучшие практики использования фреймворков

### 1. Структура проекта с Tailwind

```javascript
// project-structure
src/
├── components/
│   ├── Button.jsx
│   ├── Card.jsx
│   └── Form.jsx
├── styles/
│   ├── globals.css
│   └── components.css
├── layouts/
│   └── MainLayout.jsx
└── pages/
    ├── Home.jsx
    └── About.jsx
```

### 2. Создание кастомных компонентов

```jsx
// Button.jsx - кастомный компонент с Tailwind
import React from 'react';

const Button = ({ 
  children, 
  variant = 'primary', 
  size = 'md', 
  disabled = false, 
  className = '',
  ...props 
}) => {
  const baseClasses = 'inline-flex items-center justify-center font-medium rounded-md transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500',
    danger: 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500',
    outline: 'border border-gray-300 bg-white text-gray-700 hover:bg-gray-50 focus:ring-blue-500'
  };
  
  const sizes = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-sm',
    lg: 'px-6 py-3 text-base'
  };
  
  const disabledClasses = disabled ? 'opacity-50 cursor-not-allowed' : '';
  
  const classes = `${baseClasses} ${variants[variant]} ${sizes[size]} ${disabledClasses} ${className}`;
  
  return (
    <button 
      className={classes} 
      disabled={disabled}
      {...props}
    >
      {children}
    </button>
  );
};

export default Button;
```

### 3. Темизация

```javascript
// theme.config.js для кастомизации
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      }
    }
  }
}
```

## Примеры из промышленной разработки

### 1. Адаптивная сетка с Tailwind

```html
<!-- Сложная адаптивная сетка -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
    <div class="bg-white rounded-lg shadow-md overflow-hidden">
        <div class="p-6">
            <div class="flex items-center justify-between mb-4">
                <h3 class="text-lg font-semibold text-gray-900">Карточка 1</h3>
                <span class="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-green-100 text-green-800">
                    Новое
                </span>
            </div>
            <p class="text-gray-600 mb-4">Описание карточки 1</p>
            <div class="flex justify-between items-center">
                <span class="text-sm text-gray-500">12.05.2023</span>
                <button class="text-blue-600 hover:text-blue-800 text-sm font-medium">
                    Подробнее →
                </button>
            </div>
        </div>
    </div>
    
    <!-- Повторяющиеся карточки -->
    <div class="bg-white rounded-lg shadow-md overflow-hidden">
        <div class="p-6">
            <div class="flex items-center justify-between mb-4">
                <h3 class="text-lg font-semibold text-gray-900">Карточка 2</h3>
                <span class="inline-flex items-center px-3 py-1 rounded-full text-sm font-medium bg-blue-100 text-blue-800">
                    Популярное
                </span>
            </div>
            <p class="text-gray-600 mb-4">Описание карточки 2</p>
            <div class="flex justify-between items-center">
                <span class="text-sm text-gray-500">15.05.2023</span>
                <button class="text-blue-600 hover:text-blue-800 text-sm font-medium">
                    Подробнее →
                </button>
            </div>
        </div>
    </div>
</div>
```

### 2. Форма с валидацией

```html
<!-- Форма с Tailwind и валидацией -->
<form class="max-w-lg mx-auto">
    <div class="space-y-6">
        <div>
            <label for="email" class="block text-sm font-medium text-gray-700 mb-1">Email</label>
            <div class="relative">
                <input 
                    type="email" 
                    id="email" 
                    class="block w-full pl-10 pr-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                    placeholder="you@example.com">
                <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                    <svg class="h-5 w-5 text-gray-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                        <path d="M2.003 5.884L10 9.882l7.997-3.998A2 2 0 0016 4H4a2 2 0 00-1.997 1.884z" />
                        <path d="M18 8.118l-8 4-8-4V14a2 2 0 002 2h12a2 2 0 002-2V8.118z" />
                    </svg>
                </div>
            </div>
            <p class="mt-1 text-sm text-red-600 hidden">Пожалуйста, введите корректный email.</p>
        </div>
        
        <div>
            <label for="password" class="block text-sm font-medium text-gray-700 mb-1">Пароль</label>
            <input 
                type="password" 
                id="password" 
                class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
                placeholder="••••••••">
        </div>
        
        <div class="flex items-center">
            <input 
                id="remember-me" 
                name="remember-me" 
                type="checkbox" 
                class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded">
            <label for="remember-me" class="ml-2 block text-sm text-gray-900">Запомнить меня</label>
        </div>
        
        <div>
            <button 
                type="submit" 
                class="w-full flex justify-center py-2 px-4 border border-transparent rounded-md shadow-sm text-sm font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition duration-300">
                Войти
            </button>
        </div>
    </div>
</form>
```

### 3. Адаптивная навигация

```html
<!-- Адаптивная навигация с Tailwind -->
<nav class="bg-white shadow-sm">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
            <div class="flex items-center">
                <div class="flex-shrink-0 flex items-center">
                    <span class="text-xl font-bold text-gray-900">Логотип</span>
                </div>
                <div class="hidden md:ml-6 md:flex md:space-x-8">
                    <a href="#" class="border-b-2 border-blue-500 text-gray-900 inline-flex items-center px-1 pt-1 text-sm font-medium">
                        Главная
                    </a>
                    <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 text-sm font-medium">
                        О нас
                    </a>
                    <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 text-sm font-medium">
                        Услуги
                    </a>
                    <a href="#" class="border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700 inline-flex items-center px-1 pt-1 text-sm font-medium">
                        Контакты
                    </a>
                </div>
            </div>
            <div class="flex items-center md:hidden">
                <button type="button" class="inline-flex items-center justify-center p-2 rounded-md text-gray-400 hover:text-gray-500 hover:bg-gray-100 focus:outline-none focus:ring-2 focus:ring-inset focus:ring-blue-500">
                    <span class="sr-only">Открыть главное меню</span>
                    <svg class="block h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
                    </svg>
                </button>
            </div>
            <div class="hidden md:flex items-center">
                <div class="ml-3 relative">
                    <div>
                        <button type="button" class="bg-gray-800 flex text-sm rounded-full focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                            <span class="sr-only">Открыть меню пользователя</span>
                            <img class="h-8 w-8 rounded-full" src="https://placehold.co/32x32" alt="">
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Мобильное меню -->
    <div class="hidden md:hidden">
        <div class="pt-2 pb-3 space-y-1">
            <a href="#" class="bg-blue-50 border-blue-500 text-blue-700 block pl-3 pr-4 py-2 border-l-4 text-base font-medium">
                Главная
            </a>
            <a href="#" class="border-transparent text-gray-500 hover:bg-gray-50 hover:border-gray-300 hover:text-gray-700 block pl-3 pr-4 py-2 border-l-4 text-base font-medium">
                О нас
            </a>
            <a href="#" class="border-transparent text-gray-500 hover:bg-gray-50 hover:border-gray-300 hover:text-gray-700 block pl-3 pr-4 py-2 border-l-4 text-base font-medium">
                Услуги
            </a>
            <a href="#" class="border-transparent text-gray-500 hover:bg-gray-50 hover:border-gray-300 hover:text-gray-700 block pl-3 pr-4 py-2 border-l-4 text-base font-medium">
                Контакты
            </a>
        </div>
    </div>
</nav>
```

## Ссылки на связанные темы
- [[best-practices/best-practices]] - Лучшие практики CSS
- [[methodology/methodology]] - Методологии CSS
- [[performance/performance]] - Производительность CSS
- [[build-tools]] - Инструменты сборки

#programming #css #frameworks #tailwind #bootstrap #bulma #foundation #best-practices