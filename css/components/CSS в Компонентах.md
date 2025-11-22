# CSS в Компонентах

Стилизация компонентов - важная часть разработки пользовательского интерфейса. Существует несколько подходов к применению CSS в компонентах, каждый из которых имеет свои преимущества и недостатки.

## Встроенные стили (Inline Styles)

Встроенные стили позволяют применять CSS непосредственно к элементам через атрибут `style`:

```jsx
const MyComponent = () => {
  const styles = {
    backgroundColor: '#f0f0f0',
    padding: '20px',
    borderRadius: '8px'
  };

  return <div style={styles}>Содержимое компонента</div>;
};
```

### Преимущества:
- Инкапсуляция стилей в компоненте
- Возможность динамической стилизации через JavaScript
- Нет конфликта имен

### Недостатки:
- Ограниченные возможности (нет псевдоклассов, медиа-запросов)
- Сложнее использовать префиксы для браузеров

## CSS-модули

CSS-модули позволяют использовать локальные CSS-классы в компонентах:

```css
/* Button.module.css */
.button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
}

.primary {
  background-color: #28a745;
}
```

```jsx
import styles from './Button.module.css';

const Button = ({ variant = 'default', children }) => {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
};
```

## Styled Components

Библиотека styled-components позволяет создавать компоненты с привязанными стилями:

```jsx
import styled from 'styled-components';

const StyledButton = styled.button`
  background-color: ${props => props.primary ? '#28a745' : '#007bff'};
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    opacity: 0.8;
  }

  @media (max-width: 768px) {
    width: 100%;
  }
`;

const Button = ({ primary, children }) => (
  <StyledButton primary={primary}>{children}</StyledButton>
);
```

## Emotion

Emotion - еще одна популярная библиотека для CSS-in-JS:

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

const buttonStyle = css`
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background-color: #0056b3;
  }
`;

const Button = ({ children }) => (
  <button css={buttonStyle}>{children}</button>
);
```

## Tailwind CSS

Tailwind CSS предоставляет utility-first подход к стилизации:

```jsx
const Card = ({ title, children }) => (
  <div className="bg-white rounded-lg shadow-md p-6 max-w-sm">
    <h3 className="text-xl font-semibold mb-2 text-gray-800">{title}</h3>
    <div className="text-gray-600">{children}</div>
    <button className="mt-4 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
      Действие
    </button>
  </div>
);
```

## Стратегии стилизации

### 1. Инкапсуляция стилей
Каждый компонент должен инкапсулировать свои стили, чтобы избежать конфликтов с другими компонентами.

### 2. Повторное использование
Стили должны быть структурированы так, чтобы их можно было легко переиспользовать в разных компонентах.

### 3. Темизация
Компоненты должны поддерживать темы, позволяя изменять внешний вид приложения без изменения кода компонентов.

### 4. Адаптивность
Стили компонентов должны адаптироваться к различным размерам экранов и устройствам.

## Лучшие практики

1. **Используйте семантические имена классов** - они должны описывать назначение элемента, а не его визуальное представление.

2. **Избегайте глубокой вложенности** - это упрощает поддержку и предотвращает проблемы с приоритетом стилей.

3. **Разделяйте стили и логику** - не смешивайте сложные вычисления стилей с рендерингом компонента.

4. **Документируйте стили компонентов** - особенно если компонент предполагает кастомизацию через CSS-переменные или классы.

## Связанные Концепции

- [[CSS Архитектура]]
- [[Темизация]]
- [[Responsive Design]]
- [[Адаптивная Верстка]]

## Теги

#css #styling #components #frontend #design