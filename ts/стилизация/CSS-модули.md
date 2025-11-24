---
aliases: ["CSS Modules", "Модульный CSS", "Стилизация через CSS-модули"]
tags: [programming, css, typescript, frontend, styling]
---

# CSS-модули

CSS-модули представляют собой подход к стилизации компонентов в современных веб-приложениях, который позволяет изолировать CSS-классы в пределах компонента, предотвращая конфликты имен и обеспечивая более предсказуемое поведение стилей.

## Обзор

CSS-модули позволяют использовать локальные по умолчанию стили в CSS-файлах. При импорте CSS-модуля в JavaScript/TypeScript компонент, классы автоматически генерируются с уникальными именами, предотвращая влияние стилей на другие компоненты.

## Основы использования

### Настройка

Для использования CSS-модулей в TypeScript-проекте, особенно в React, необходимо:

1. Убедиться, что ваш сборщик (Webpack, Vite и т.д.) настроен на обработку `.module.css` файлов
2. Иметь соответствующие типы для CSS-модулей в TypeScript

### Создание CSS-модуля

Создайте файл с расширением `.module.css`:

```css
/* Button.module.css */
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
```

### Использование в TypeScript-компоненте

```typescript
// Button.tsx
import React from 'react';
import styles from './Button.module.css';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: () => void;
}

const Button: React.FC<ButtonProps> = ({ 
  children, 
  variant = 'primary', 
  disabled = false, 
  onClick 
}) => {
  const buttonClasses = [
    styles.button,
    styles[variant],
    disabled ? styles.disabled : ''
  ].filter(Boolean).join(' ');

  return (
    <button 
      className={buttonClasses}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;
```

## Типизация CSS-модулей

Для корректной типизации CSS-модулей в TypeScript, создайте файл декларации:

```typescript
// types/css-modules.d.ts
declare module '*.module.css' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.scss' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.sass' {
  const classes: { [key: string]: string };
  export default classes;
}

declare module '*.module.less' {
  const classes: { [key: string]: string };
  export default classes;
}
```

## Продвинутые возможности

### Использование переменных CSS

```css
/* variables.module.css */
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --font-size-small: 12px;
  --font-size-medium: 16px;
  --font-size-large: 20px;
}
```

```css
/* Button.module.css */
@import './variables.module.css';

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: var(--font-size-medium);
}

.primary {
  background-color: var(--primary-color);
  color: white;
}
```

### Комбинирование классов

```typescript
import React from 'react';
import styles from './Card.module.css';

const Card: React.FC = ({ children }) => {
  return (
    <div className={`${styles.card} ${styles.elevated}`}>
      {children}
    </div>
  );
};
```

### Глобальные стили

Для случаев, когда нужно использовать глобальные стили, можно использовать специальный синтаксис:

```css
/* Component.module.css */
:global(.global-class) {
  color: red;
}

.localClass {
  color: blue;
}

:global {
  .another-global-class {
    margin: 10px;
  }
}
```

## Преимущества CSS-модулей

- **Изоляция стилей**: Классы изолированы в пределах компонента
- **Предотвращение конфликтов**: Автоматическая генерация уникальных имен классов
- **Легкость тестирования**: Стили компонентов не влияют друг на друга
- **Поддержка IDE**: Возможность переименования классов с сохранением целостности

## Недостатки и ограничения

- **Сложность настройки**: Требует правильной настройки сборки
- **Ограниченная гибкость**: Меньше возможностей для переопределения стилей
- **Меньше поддержки в экосистеме**: Не все библиотеки компонентов используют CSS-модули

## Лучшие практики

1. **Используйте осмысленные имена классов**:
   ```css
   /* Хорошо */
   .userCard { }
   .userCardHeader { }
   .userCardAvatar { }
   
   /* Плохо */
   .a { }
   .b { }
   .c { }
   ```

2. **Организуйте стили по компонентам**:
   ```
   components/
   ├── Button/
   │   ├── Button.tsx
   │   └── Button.module.css
   └── Card/
       ├── Card.tsx
       └── Card.module.css
   ```

3. **Используйте CSS-переменные для темизации**:
   ```css
   .button {
     background-color: var(--primary-color, #007bff);
     color: var(--text-color, white);
   }
   ```

## Совместимость с другими библиотеками

CSS-модули хорошо сочетаются с такими библиотеками как:
- [[React]]
- [[Next.js]]
- [[Webpack]]
- [[Vite]]

## Альтернативы

- [[Styled-components]]
- [[Emotion]]
- [[Tailwind CSS]]
- [[Стилизация-компонентов]]

## Заключение

CSS-модули представляют собой надежный способ организации стилей в компонентном подходе, обеспечивая изоляцию и предотвращая конфликты имен. Они особенно полезны в крупных приложениях, где важна предсказуемость и масштабируемость стилей.

> [!tip] 
> Используйте CSS-модули в сочетании с CSS-переменными для создания гибких и настраиваемых компонентов.

> [!warning]
> Убедитесь, что ваша сборка правильно обрабатывает файлы с расширением `.module.css` для корректной работы уникальной генерации имен классов.