---
aliases: ["Styled Components", "Styled-Components", "Стилизация через Styled Components"]
tags: [programming, css-in-js, typescript, frontend, styling]
---

# Styled-components

Styled-components - это библиотека для стилизации компонентов в React-приложениях, использующая подход CSS-in-JS. Она позволяет писать настоящий CSS-код в JavaScript/TypeScript, обеспечивая изоляцию стилей и динамическую стилизацию через пропсы.

## Обзор

Styled-components позволяет создавать компоненты с встроенными стилями, используя шаблонные строки JavaScript. Это дает возможность использовать полную мощь CSS с динамическими возможностями JavaScript.

## Установка и настройка

Для начала работы с styled-components в TypeScript-проекте:

```bash
npm install styled-components
npm install --save-dev @types/styled-components
```

### Базовое использование

```typescript
import styled from 'styled-components';

// Создание базового компонента с стилями
const Button = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  background-color: #007bff;
  color: white;
  
  &:hover {
    background-color: #0056b3;
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;

// Использование компонента
const MyComponent = () => (
  <Button>Нажми меня</Button>
);
```

## Передача пропсов

Styled-components позволяет передавать пропсы в стили:

```typescript
import styled from 'styled-components';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

const StyledButton = styled.button<ButtonProps>`
  padding: ${({ size }) => {
    switch (size) {
      case 'small': return '5px 10px';
      case 'large': return '15px 30px';
      default: return '10px 20px';
    }
  }};
  
  background-color: ${({ variant }) => 
    variant === 'secondary' ? '#6c757d' : '#007bff'
  };
  
  color: white;
  border: none;
  border-radius: 4px;
  cursor: ${({ disabled }) => disabled ? 'not-allowed' : 'pointer'};
  opacity: ${({ disabled }) => disabled ? 0.6 : 1};
  
  &:hover:not(:disabled) {
    background-color: ${({ variant }) => 
      variant === 'secondary' ? '#545b62' : '#0056b3'
    };
  }
`;

// Использование
const App = () => (
  <div>
    <StyledButton variant="primary" size="medium">
      Основная кнопка
    </StyledButton>
    <StyledButton variant="secondary" size="small" disabled>
      Вторичная кнопка
    </StyledButton>
  </div>
);
```

## Темизация

Styled-components предоставляет мощную систему темизации через ThemeProvider:

```typescript
import styled, { ThemeProvider, DefaultTheme } from 'styled-components';

// Определение типа темы
interface Theme extends DefaultTheme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
  };
  spacing: {
    small: string;
    medium: string;
    large: string;
  };
}

const theme: Theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff',
    text: '#333333',
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px',
  }
};

// Компонент с использованием темы
const Card = styled.div`
  background-color: ${props => props.theme.colors.background};
  color: ${props => props.theme.colors.text};
  padding: ${props => props.theme.spacing.medium};
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
`;

// Использование в приложении
const App = () => (
  <ThemeProvider theme={theme}>
    <Card>
      <h1>Карточка с темой</h1>
    </Card>
  </ThemeProvider>
);
```

## Адаптивный дизайн

Styled-components отлично поддерживает медиа-запросы:

```typescript
import styled from 'styled-components';

const Container = styled.div`
  width: 100%;
  padding: 16px;
  
  @media (min-width: 768px) {
    max-width: 750px;
    margin: 0 auto;
    padding: 24px;
  }
  
  @media (min-width: 1024px) {
    max-width: 1200px;
    padding: 32px;
  }
`;

// Использование CSS-переменных
const ResponsiveButton = styled.button<{ $isActive: boolean }>`
  padding: 12px 24px;
  background-color: ${props => props.$isActive ? '#28a745' : '#007bff'};
  color: white;
  border: none;
  border-radius: 4px;
  transition: background-color 0.3s ease;
  
  @media (max-width: 768px) {
    width: 100%;
    font-size: 18px;
  }
`;
```

## Расширение существующих компонентов

Styled-components позволяет расширять уже существующие компоненты:

```typescript
import styled from 'styled-components';
import { Button } from 'react-bootstrap'; // или любой другой компонент

const StyledBootstrapButton = styled(Button)`
  && {  // Двойной амперсанд для повышения специфичности
    background-color: #ff6b6b !important;
    border-color: #ff6b6b !important;
    
    &:hover {
      background-color: #ff5252 !important;
      border-color: #ff5252 !important;
    }
  }
`;
```

## Анимации

Styled-components поддерживает CSS-анимации:

```typescript
import styled, { keyframes } from 'styled-components';

const pulse = keyframes`
  0% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(0, 123, 255, 0.7);
  }
  70% {
    transform: scale(1.05);
    box-shadow: 0 0 0 10px rgba(0, 123, 255, 0);
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(0, 123, 255, 0);
  }
`;

const PulsingButton = styled.button`
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  animation: ${pulse} 2s infinite;
`;
```

## Типизация пропсов

Важно правильно типизировать пропсы для styled-компонентов:

```typescript
import styled from 'styled-components';

interface StyledDivProps {
  $backgroundColor?: string;
  $textColor?: string;
  $padding?: string;
}

const StyledDiv = styled.div<StyledDivProps>`
  background-color: ${props => props.$backgroundColor || '#ffffff'};
  color: ${props => props.$textColor || '#000000'};
  padding: ${props => props.$padding || '16px'};
  border-radius: 4px;
`;

// Использование
const Example = () => (
  <StyledDiv 
    $backgroundColor="#f8f9fa" 
    $textColor="#495057"
    $padding="20px"
  >
    Стилизованный компонент с типизацией
  </StyledDiv>
);
```

## Продвинутые возможности

### Глобальные стили

```typescript
import { createGlobalStyle } from 'styled-components';

export const GlobalStyle = createGlobalStyle`
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  body {
    font-family: 'Arial', sans-serif;
    background-color: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
  }
`;
```

### Абстракции и утилиты

```typescript
import styled from 'styled-components';

// Утилита для создания адаптивных отступов
const getSpacing = (size: 'small' | 'medium' | 'large') => {
  switch (size) {
    case 'small': return '8px';
    case 'large': return '24px';
    default: return '16px';
  }
};

interface SpacingProps {
  $padding?: 'small' | 'medium' | 'large';
  $margin?: 'small' | 'medium' | 'large';
}

export const SpacingContainer = styled.div<SpacingProps>`
  padding: ${props => props.$padding ? getSpacing(props.$padding) : '0'};
  margin: ${props => props.$margin ? getSpacing(props.$margin) : '0'};
`;
```

## Преимущества styled-components

- **Изоляция стилей**: Автоматическая изоляция стилей через уникальные классы
- **Динамическая стилизация**: Возможность изменения стилей на основе пропсов
- **Темизация**: Поддержка тем через ThemeProvider
- **Типобезопасность**: Полная типизация через TypeScript
- **Уменьшение конфликтов**: Уникальные имена классов предотвращают конфликты

## Недостатки и ограничения

- **Размер бандла**: Добавляет размер к бандлу из-за runtime-обработки
- **SEO и SSR**: Требует дополнительной настройки для серверного рендеринга
- **Обучение**: Требует изучения нового подхода к стилизации
- **Отладка**: Иногда сложнее отлаживать стили в браузере

## Лучшие практики

1. **Используйте именованные экспорты для сложных стилей**:
   ```typescript
   export const ButtonContainer = styled.div`
     display: flex;
     gap: 8px;
   `;
   ```

2. **Избегайте слишком глубокой вложенности**:
   ```typescript
   // Лучше избегать:
   const Nested = styled.div`
     > div > span {
       color: red;
     }
   `;
   ```

3. **Используйте темы для консистентности**:
   ```typescript
   const StyledComponent = styled.div`
     color: ${props => props.theme.colors.text};
     background: ${props => props.theme.colors.background};
   `;
   ```

## Альтернативы

- [[Emotion]]
- [[CSS-модули]]
- [[Tailwind CSS]]
- [[Стилизация-компонентов]]

## Заключение

Styled-components предоставляет мощный и гибкий способ стилизации компонентов в React-приложениях. Благодаря полной интеграции с TypeScript и поддержке темизации, это отличный выбор для сложных приложений, где важна динамическая стилизация и типобезопасность.

> [!tip]
> Используйте именованные пропсы с префиксом $ (например, `$isActive`) для передачи стилизационных пропсов, чтобы отличать их от обычных пропсов компонента.

> [!warning]
> Избегайте чрезмерного использования styled-components в простых проектах, где CSS-модули или Tailwind CSS могут быть более подходящими решениями.