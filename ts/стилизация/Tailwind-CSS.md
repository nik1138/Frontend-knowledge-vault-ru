---
aliases: ["Tailwind CSS", "Tailwind", "Тейлвинд CSS"]
tags: [programming, css, utility-first, frontend, styling]
---

# Tailwind CSS

Tailwind CSS - это мощный CSS-фреймворк с утилитарным подходом к стилизации, который предоставляет набор предварительно определенных классов для быстрого создания пользовательских интерфейсов без написания пользовательского CSS.

## Обзор

Tailwind CSS использует утилитарный подход к стилизации, предоставляя небольшие, однозначные классы, которые можно комбинировать для создания сложных компонентов. В отличие от традиционных CSS-фреймворков, Tailwind не предоставляет готовые компоненты, а дает вам инструменты для создания уникального дизайна.

## Установка и настройка

Для начала работы с Tailwind CSS в TypeScript-проекте:

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Настройка конфигурации

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/**/*.{js,jsx,ts,tsx}',
    './pages/**/*.{js,jsx,ts,tsx}',
    './components/**/*.{js,jsx,ts,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: '#007bff',
        secondary: '#6c757d',
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      }
    },
  },
  plugins: [],
}
```

### Подключение стилей

```css
/* src/index.css или src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Основы использования

### Базовые классы

```tsx
// Button.tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  onClick?: () => void;
}

const Button: React.FC<ButtonProps> = ({ 
  children, 
  variant = 'primary', 
  size = 'md',
  disabled = false,
  onClick 
}) => {
  // Определение классов на основе пропсов
  const baseClasses = 'inline-flex items-center justify-center font-medium rounded-md border transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500 border-transparent',
    secondary: 'bg-gray-600 text-white hover:bg-gray-700 focus:ring-gray-500 border-transparent',
    outline: 'bg-transparent text-blue-600 hover:bg-blue-50 focus:ring-blue-500 border-blue-600',
  };
  
  const sizeClasses = {
    sm: 'text-xs py-1.5 px-3',
    md: 'text-sm py-2 px-4',
    lg: 'text-base py-2.5 px-5',
  };
  
  const disabledClasses = disabled ? 'opacity-50 cursor-not-allowed' : '';
  
  const classes = [
    baseClasses,
    variantClasses[variant],
    sizeClasses[size],
    disabledClasses
  ].join(' ');

  return (
    <button 
      className={classes}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;
```

## Адаптивный дизайн

Tailwind CSS предоставляет встроенные медиа-запросы через префиксы:

```tsx
// ResponsiveComponent.tsx
const ResponsiveComponent = () => (
  <div className="container mx-auto px-4">
    {/* Столбцы по-умолчанию: 100% на мобильных */}
    {/* md: - 768px+, lg: - 1024px+, xl: - 1280px+ */}
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Карточка 1</h3>
        <p className="text-gray-600">Содержимое карточки</p>
      </div>
      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Карточка 2</h3>
        <p className="text-gray-600">Содержимое карточки</p>
      </div>
      <div className="bg-white p-6 rounded-lg shadow-md">
        <h3 className="text-lg font-semibold mb-2">Карточка 3</h3>
        <p className="text-gray-600">Содержимое карточки</p>
      </div>
    </div>
  </div>
);
```

## Темизация и кастомизация

### Расширение темы

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      maxWidth: {
        '8xl': '88rem',
      }
    },
  },
}
```

### Пользовательские компоненты

```css
/* src/index.css */
@layer components {
  .btn-primary {
    @apply bg-blue-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-blue-700 transition-colors;
  }
  
  .btn-secondary {
    @apply bg-gray-600 text-white font-semibold py-2 px-4 rounded-lg hover:bg-gray-700 transition-colors;
  }
  
  .card {
    @apply bg-white rounded-xl shadow-md overflow-hidden;
  }
  
  .input-field {
    @apply w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent;
  }
}
```

## TypeScript интеграция

### Типизация для утилит

```tsx
// utils/tailwind.ts
export type ColorVariant = 'primary' | 'secondary' | 'success' | 'danger' | 'warning' | 'info';

export const getColorClasses = (variant: ColorVariant): string => {
  const colorMap: Record<ColorVariant, string> = {
    primary: 'bg-blue-500 text-white',
    secondary: 'bg-gray-500 text-white',
    success: 'bg-green-500 text-white',
    danger: 'bg-red-500 text-white',
    warning: 'bg-yellow-500 text-gray-900',
    info: 'bg-cyan-500 text-white',
  };
  
  return colorMap[variant];
};

// Использование в компоненте
interface StatusBadgeProps {
  status: ColorVariant;
  text: string;
}

const StatusBadge: React.FC<StatusBadgeProps> = ({ status, text }) => (
  <span className={`inline-flex items-center px-3 py-1 rounded-full text-sm font-medium ${getColorClasses(status)}`}>
    {text}
  </span>
);
```

### Утилиты для условных классов

```tsx
// utils/classNameUtils.ts
export const cn = (...classes: (string | boolean | undefined)[]): string => {
  return classes.filter(Boolean).join(' ');
};

// Использование
interface CardProps {
  isActive?: boolean;
  isError?: boolean;
  children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ isActive, isError, children }) => (
  <div className={cn(
    'p-6 rounded-lg border',
    isActive && 'border-blue-500 bg-blue-50',
    isError && 'border-red-500 bg-red-50',
    !isActive && !isError && 'border-gray-200'
  )}>
    {children}
  </div>
);
```

## Продвинутые возможности

### Dark mode

```tsx
// Включите dark mode в конфигурации
// tailwind.config.js
module.exports = {
  darkMode: 'class', // или 'media'
  // ...
}
```

```tsx
// Компонент с поддержкой темной темы
const ThemedComponent = () => (
  <div className="bg-white dark:bg-gray-800 text-gray-900 dark:text-white p-6 rounded-lg">
    <h2 className="text-xl font-bold mb-4">Заголовок</h2>
    <p className="mb-4">Текст в светлой и темной теме</p>
    <button className="bg-blue-500 hover:bg-blue-700 text-white py-2 px-4 rounded dark:bg-blue-700 dark:hover:bg-blue-900">
      Кнопка
    </button>
  </div>
);
```

### Псевдоклассы и состояния

```tsx
const InteractiveButton = () => (
  <button className="
    bg-blue-500 
    hover:bg-blue-700 
    focus:outline-none 
    focus:ring-2 
    focus:ring-blue-500 
    focus:ring-opacity-50
    active:bg-blue-800
    disabled:opacity-50
    disabled:cursor-not-allowed
    transition
    duration-150
    ease-in-out
  ">
    Интерактивная кнопка
  </button>
);
```

### Пользовательские свойства и переменные

```tsx
// Использование CSS-переменных с Tailwind
const CustomStyledComponent = () => (
  <div 
    className="p-6 rounded-lg"
    style={{
      '--custom-color': '#3b82f6',
      '--custom-size': '20px'
    } as React.CSSProperties}
  >
    <div className="text-[color:var(--custom-color)] text-[length:var(--custom-size)]">
      Текст с пользовательскими переменными
    </div>
  </div>
);
```

## Практические примеры

### Карточка продукта

```tsx
interface ProductCardProps {
  title: string;
  description: string;
  price: number;
  imageUrl: string;
  onAddToCart: () => void;
}

const ProductCard: React.FC<ProductCardProps> = ({ 
  title, 
  description, 
  price, 
  imageUrl,
  onAddToCart 
}) => (
  <div className="max-w-sm rounded overflow-hidden shadow-lg bg-white">
    <img 
      className="w-full h-48 object-cover" 
      src={imageUrl} 
      alt={title}
    />
    <div className="px-6 py-4">
      <div className="font-bold text-xl mb-2 text-gray-900">{title}</div>
      <p className="text-gray-700 text-base mb-4">{description}</p>
      <div className="flex justify-between items-center">
        <span className="text-gray-900 font-semibold">${price.toFixed(2)}</span>
        <button 
          onClick={onAddToCart}
          className="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline transition-colors"
        >
          В корзину
        </button>
      </div>
    </div>
  </div>
);
```

### Форма с валидацией

```tsx
interface FormFieldProps {
  label: string;
  type: string;
  error?: string;
  touched?: boolean;
  required?: boolean;
}

const FormField: React.FC<FormFieldProps> = ({ 
  label, 
  type, 
  error, 
  touched, 
  required = false 
}) => (
  <div className="mb-4">
    <label className="block text-gray-700 text-sm font-bold mb-2">
      {label} {required && <span className="text-red-500">*</span>}
    </label>
    <input
      type={type}
      className={`
        shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 
        leading-tight focus:outline-none focus:shadow-outline
        ${error && touched ? 'border-red-500' : ''}
      `}
    />
    {error && touched && (
      <p className="text-red-500 text-xs italic mt-1">{error}</p>
    )}
  </div>
);
```

## Преимущества Tailwind CSS

- **Быстрая разработка**: Быстрое создание интерфейсов без написания CSS
- **Консистентность**: Согласованные отступы, цвета и размеры через систему тем
- **Адаптивность**: Встроенные адаптивные утилиты
- **Производительность**: Утилиты объединяются в оптимизированный CSS
- **Кастомизация**: Полная настраиваемость через конфигурацию

## Недостатки и ограничения

- **Объемный HTML**: Классы могут сделать HTML-разметку громоздкой
- **Кривая обучения**: Требует привыкания к утилитарному подходу
- **Ограниченная гибкость**: Некоторые сложные стили требуют пользовательского CSS
- **Проблемы с переиспользованием**: Сложнее создавать переиспользуемые компоненты

## Лучшие практики

1. **Используйте семантические имена переменных**:
   ```tsx
   // Хорошо
   const buttonSize = size === 'sm' ? 'py-1 px-2' : 'py-2 px-4';
   
   // Избегайте
   const small = 'py-1 px-2';
   ```

2. **Создавайте переиспользуемые компоненты**:
   ```tsx
   // components/Button.tsx
   const Button = ({ children, variant = 'primary', ...props }) => (
     <button 
       className={`
         inline-flex items-center justify-center rounded-md 
         font-medium transition-colors
         ${variant === 'primary' 
           ? 'bg-blue-600 text-white hover:bg-blue-700' 
           : 'bg-gray-600 text-white hover:bg-gray-700'}
       `}
       {...props}
     >
       {children}
     </button>
   );
   ```

3. **Используйте @apply для сложных компонентов**:
   ```css
   /* В файле стилей */
   .btn {
     @apply inline-flex items-center justify-center rounded-md font-medium transition-colors;
   }
   ```

## Альтернативы

- [[Styled-components]]
- [[Emotion]]
- [[CSS-модули]]
- [[Стилизация-компонентов]]

## Заключение

Tailwind CSS представляет собой мощный инструмент для быстрой разработки интерфейсов с предсказуемыми и согласованными стилями. Его утилитарный подход особенно полезен для прототипирования и создания уникальных дизайнов без необходимости писать пользовательский CSS.

> [!tip]
> Используйте плагин Tailwind CSS IntelliSense для VS Code для автодополнения классов и улучшения разработки.

> [!warning]
> Избегайте чрезмерного использования утилит, которые делают HTML-разметку трудночитаемой. Для сложных компонентов лучше использовать @apply или создавать переиспользуемые компоненты.