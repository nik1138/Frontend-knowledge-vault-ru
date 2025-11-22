---
aliases: ["CSS-in-JS библиотеки", "Styled Components", "Emotion", "Stitches", "CSS в JavaScript"]
tags: ["css", "css-in-js", "styled-components", "emotion", "stitches", "libraries"]
---

# CSS-in-JS библиотеки: styled-components, emotion, stitches

CSS-in-JS — это подход к стилизации компонентов, при котором CSS-правила определяются непосредственно в JavaScript/TypeScript коде. Рассмотрим три популярные библиотеки, реализующие этот подход: styled-components, Emotion и Stitches.

## Styled Components

Styled Components — одна из самых популярных библиотек для CSS-in-JS, позволяющая создавать компоненты с привязанными стилями.

### Установка и базовое использование

```bash
npm install styled-components
```

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background-color: #007bff;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
  
  &:hover {
    background-color: #0056b3;
  }
  
  ${props => props.primary && `
    background-color: #28a745;
  `}
`;

// Использование компонента
const App = () => (
  <div>
    <Button>Обычная кнопка</Button>
    <Button primary>Главная кнопка</Button>
  </div>
);
```

### Особенности Styled Components

1. **Автоматическая изоляция CSS**: Каждый компонент получает уникальный CSS-класс
2. **Поддержка пропсов**: Возможность динамически изменять стили на основе пропсов
3. **Темизация**: Встроенная поддержка тем через ThemeProvider
4. **SSR**: Поддержка серверного рендеринга

```jsx
import { ThemeProvider } from 'styled-components';

const theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    success: '#28a745'
  }
};

const StyledButton = styled.button`
  background-color: ${props => props.theme.colors[props.color] || props.theme.colors.primary};
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
`;

const App = () => (
  <ThemeProvider theme={theme}>
    <StyledButton color="primary">Primary</StyledButton>
    <StyledButton color="success">Success</StyledButton>
  </ThemeProvider>
);
```

## Emotion

Emotion — это мощная библиотека для стилизации с поддержкой как объектного, так и строкового синтаксиса.

### Установка и использование

```bash
npm install @emotion/react @emotion/styled
```

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';
import styled from '@emotion/styled';

// Стилизация с помощью css пропа
const Button = ({ children, primary }) => (
  <button
    css={css`
      background-color: ${primary ? '#28a745' : '#007bff'};
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      border-radius: 0.25rem;
      cursor: pointer;
      
      &:hover {
        opacity: 0.8;
      }
    `}
  >
    {children}
  </button>
);

// Стилизация с помощью styled компонентов
const StyledButton = styled.button`
  background-color: ${props => props.primary ? '#28a745' : '#007bff'};
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 0.25rem;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
`;
```

### Особенности Emotion

1. **Два подхода к стилизации**: `css` проп и `styled` компоненты
2. **Высокая производительность**: Встроенный оптимизатор
3. **Поддержка TypeScript**: Отличная типизация
4. **Серверный рендеринг**: Полная поддержка SSR
5. **Плагины**: Возможность расширения функциональности

## Stitches

Stitches — это высокопроизводительная библиотека от команды Modulz, ориентированная на создание дизайн-систем.

### Установка и использование

```bash
npm install @stitches/react
```

```jsx
import { createStitches } from '@stitches/react';

const { styled, css, theme } = createStitches({
  theme: {
    colors: {
      primary: '#007bff',
      secondary: '#6c757d',
      success: '#28a745',
      gray100: '#f8f9fa',
      gray900: '#212529',
    },
    space: {
      1: '0.25rem',
      2: '0.5rem',
      3: '1rem',
      4: '2rem',
    },
    sizes: {
      1: '0.25rem',
      2: '0.5rem',
      3: '1rem',
      4: '2rem',
    },
  },
  utils: {
    marginX: (value) => ({ marginLeft: value, marginRight: value }),
    marginY: (value) => ({ marginTop: value, marginBottom: value }),
  },
});

const Button = styled('button', {
  // Стили по умолчанию
  backgroundColor: '$primary',
  color: 'white',
  border: 'none',
  padding: '$2 $3',
  borderRadius: '$1',
  cursor: 'pointer',
  
  // Варианты
  variants: {
    variant: {
      primary: {
        backgroundColor: '$primary',
      },
      success: {
        backgroundColor: '$success',
      },
    },
    size: {
      small: {
        fontSize: '0.75rem',
        padding: '$1 $2',
      },
      large: {
        fontSize: '1.25rem',
        padding: '$3 $4',
      },
    },
  },
  
  // Состояния
  compoundVariants: [
    {
      variant: 'primary',
      size: 'large',
      css: {
        fontWeight: 'bold',
      },
    },
  ],
  
  defaultVariants: {
    variant: 'primary',
    size: 'small',
  },
});

// Использование
const App = () => (
  <div>
    <Button>Обычная кнопка</Button>
    <Button variant="success">Успешная кнопка</Button>
    <Button size="large">Большая кнопка</Button>
    <Button variant="success" size="large">Большая успешная кнопка</Button>
  </div>
);
```

### Особенности Stitches

1. **Типобезопасность**: Полная поддержка TypeScript
2. **Встроенные варианты**: Удобное создание разных вариантов компонентов
3. **Темизация**: Встроенная система темизации
4. **Утилиты**: Возможность создания пользовательских утилит
5. **Производительность**: Оптимизация во время сборки
6. **Предсказуемые имена классов**: Улучшенная отладка

## Сравнение библиотек

| Характеристика | Styled Components | Emotion | Stitches |
|----------------|-------------------|---------|----------|
| Размер бандла | ~13KB | ~6KB | ~10KB |
| TypeScript | Отличная поддержка | Отличная поддержка | Отличная поддержка |
| Темизация | Через ThemeProvider | Через ThemeProvider | Встроенная система |
| Варианты | Нет встроенного | Пользовательская реализация | Встроенные варианты |
| Производительность | Хорошая | Отличная | Отличная |
| Изоляция CSS | Автоматическая | Автоматическая | Автоматическая |
| Объектный синтаксис | Нет | Есть | Есть |

## Практические рекомендации

1. **Styled Components**: Хорош выбор для проектов, где важна простота и знакомство с библиотекой
2. **Emotion**: Отличный выбор для проектов, требующих гибкости и высокой производительности
3. **Stitches**: Идеален для создания дизайн-систем и проектов, где важна типобезопасность

> [!tip] Совет
> При выборе библиотеки учитывайте размер проекта, команду разработчиков и требования к производительности.

> [!warning] Важно
> Все CSS-in-JS библиотеки требуют дополнительного времени на выполнение в браузере по сравнению с обычным CSS.

[[Преимущества и недостатки CSS-in-JS подхода]]
[[Интеграция CSS-in-JS с различными фреймворками]]
[[CSS-архитектура]]