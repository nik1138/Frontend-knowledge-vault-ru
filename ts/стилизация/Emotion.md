---
aliases: ["Emotion", "Emotion CSS", "CSS-in-JS с Emotion"]
tags: [programming, css-in-js, typescript, frontend, styling]
---

# Emotion

Emotion - это мощная библиотека для стилизации компонентов в React-приложениях, использующая подход CSS-in-JS. Она предоставляет гибкие возможности для динамической стилизации, темизации и оптимизации рендеринга стилей.

## Обзор

Emotion - это библиотека для стилизации, которая поддерживает несколько подходов: через шаблонные строки (css-prop и styled API) и через объекты стилей. Она предоставляет высокую производительность и гибкость при создании стилизованных компонентов.

## Установка и настройка

Для начала работы с Emotion в TypeScript-проекте:

```bash
npm install @emotion/react @emotion/styled
npm install --save-dev @types/styled-system__css
```

Для использования css-prop также установите:

```bash
npm install @emotion/babel-plugin
```

### Настройка Babel (опционально)

```json
// .babelrc или babel.config.js
{
  "plugins": ["@emotion"]
}
```

### Базовое использование

```typescript
import { css } from '@emotion/react';

const MyComponent = () => (
  <div 
    css={css`
      padding: 20px;
      background-color: #f0f0f0;
      border-radius: 8px;
    `}
  >
    Стилизованный компонент с Emotion
  </div>
);
```

## styled API

Emotion предоставляет API, похожий на styled-components:

```typescript
import styled from '@emotion/styled';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
}

const Button = styled.button<ButtonProps>`
  padding: ${props => {
    switch (props.size) {
      case 'small': return '5px 10px';
      case 'large': return '15px 30px';
      default: return '10px 20px';
    }
  }};
  
  background-color: ${props => 
    props.variant === 'secondary' ? '#6c757d' : '#007bff'
  };
  
  color: white;
  border: none;
  border-radius: 4px;
  cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};
  opacity: ${props => props.disabled ? 0.6 : 1};
  
  &:hover:not([disabled]) {
    background-color: ${props => 
      props.variant === 'secondary' ? '#545b62' : '#0056b3'
    };
  }
`;

// Использование
const App = () => (
  <div>
    <Button variant="primary" size="medium">
      Основная кнопка
    </Button>
    <Button variant="secondary" size="small" disabled>
      Вторичная кнопка
    </Button>
  </div>
);
```

## css-prop API

Emotion позволяет использовать css-prop для стилизации компонентов:

```typescript
import { css } from '@emotion/react';
import { useState } from 'react';

interface CardProps {
  isActive?: boolean;
}

const Card = ({ isActive, children }: { isActive?: boolean; children: React.ReactNode }) => (
  <div 
    css={css`
      padding: 20px;
      border-radius: 8px;
      background-color: ${isActive ? '#e3f2fd' : '#f5f5f5'};
      border: 2px solid ${isActive ? '#2196f3' : '#ddd'};
      transition: all 0.3s ease;
      cursor: pointer;
      
      &:hover {
        box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      }
    `}
  >
    {children}
  </div>
);

// Использование
const App = () => {
  const [activeCard, setActiveCard] = useState(false);
  
  return (
    <Card isActive={activeCard} onClick={() => setActiveCard(!activeCard)}>
      Нажмите для переключения состояния
    </Card>
  );
};
```

## Темизация

Emotion предоставляет систему темизации через ThemeProvider:

```typescript
import { ThemeProvider, useTheme, css } from '@emotion/react';

interface Theme {
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
  breakpoints: {
    mobile: string;
    tablet: string;
    desktop: string;
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
  },
  breakpoints: {
    mobile: '768px',
    tablet: '1024px',
    desktop: '1200px',
  }
};

// Использование темы в компоненте
const StyledCard = ({ children }: { children: React.ReactNode }) => {
  const theme = useTheme<Theme>();
  
  return (
    <div 
      css={css`
        background-color: ${theme.colors.background};
        color: ${theme.colors.text};
        padding: ${theme.spacing.medium};
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        
        @media (max-width: ${theme.breakpoints.mobile}) {
          padding: ${theme.spacing.small};
        }
      `}
    >
      {children}
    </div>
  );
};

// Использование в приложении
const App = () => (
  <ThemeProvider theme={theme}>
    <StyledCard>
      <h1>Карточка с темой</h1>
    </StyledCard>
  </ThemeProvider>
);
```

## Адаптивный дизайн

Emotion отлично поддерживает адаптивный дизайн:

```typescript
import styled from '@emotion/styled';
import { useTheme } from '@emotion/react';

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

// Использование хелперов для медиа-запросов
const mediaQueries = {
  small: '@media (max-width: 768px)',
  medium: '@media (min-width: 769px) and (max-width: 1024px)',
  large: '@media (min-width: 1025px)'
};

const ResponsiveButton = styled.button<{ $isActive: boolean }>`
  padding: 12px 24px;
  background-color: ${props => props.$isActive ? '#28a745' : '#007bff'};
  color: white;
  border: none;
  border-radius: 4px;
  
  ${mediaQueries.small} {
    width: 100%;
    font-size: 18px;
  }
  
  ${mediaQueries.large} {
    font-size: 16px;
  }
`;
```

## Анимации

Emotion поддерживает CSS-анимации:

```typescript
import { css, keyframes } from '@emotion/react';

const bounce = keyframes`
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-30px);
  }
  60% {
    transform: translateY(-15px);
  }
`;

const AnimatedButton = ({ children }: { children: React.ReactNode }) => (
  <button
    css={css`
      padding: 10px 20px;
      background-color: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      animation: ${bounce} 2s infinite;
    `}
  >
    {children}
  </button>
);
```

## Объекты стилей

Emotion также поддерживает стили в формате объектов:

```typescript
import { css } from '@emotion/react';

const buttonStyle = {
  backgroundColor: '#007bff',
  color: 'white',
  border: 'none',
  padding: '10px 20px',
  borderRadius: '4px',
  cursor: 'pointer',
  '&:hover': {
    backgroundColor: '#0056b3'
  }
};

const Button = () => (
  <button 
    css={css(buttonStyle)}
  >
    Кнопка со стилями в объекте
  </button>
);

// Динамические стили в объектах
interface DynamicButtonProps {
  variant: 'primary' | 'secondary';
}

const DynamicButton = ({ variant }: DynamicButtonProps) => {
  const style = {
    backgroundColor: variant === 'primary' ? '#007bff' : '#6c757d',
    color: 'white',
    border: 'none',
    padding: '10px 20px',
    borderRadius: '4px',
    cursor: 'pointer' as const,
    '&:hover': {
      backgroundColor: variant === 'primary' ? '#0056b3' : '#545b62'
    }
  };
  
  return (
    <button css={css(style)}>
      Динамическая кнопка
    </button>
  );
};
```

## Продвинутые возможности

### Стилизация через утилиты

```typescript
import { css } from '@emotion/react';

// Утилита для создания адаптивных отступов
const spacing = (size: number) => `${size * 8}px`;

const SpacingContainer = ({ 
  children, 
  padding = 2, 
  margin = 0 
}: { 
  children: React.ReactNode; 
  padding?: number; 
  margin?: number; 
}) => (
  <div 
    css={css`
      padding: ${spacing(padding)};
      margin: ${spacing(margin)};
    `}
  >
    {children}
  </div>
);
```

### Условные стили

```typescript
import { css } from '@emotion/react';

interface StatusIndicatorProps {
  status: 'success' | 'error' | 'warning' | 'info';
}

const StatusIndicator = ({ status }: StatusIndicatorProps) => {
  const statusColors = {
    success: '#28a745',
    error: '#dc3545',
    warning: '#ffc107',
    info: '#17a2b8'
  };
  
  return (
    <div 
      css={css`
        width: 20px;
        height: 20px;
        border-radius: 50%;
        background-color: ${statusColors[status]};
        display: inline-block;
      `}
    />
  );
};
```

### Стилизация дочерних элементов

```typescript
import styled from '@emotion/styled';

const CardList = styled.div`
  display: flex;
  flex-direction: column;
  gap: 16px;
  
  & > * {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
  }
  
  & > *:hover {
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  }
`;
```

## Преимущества Emotion

- **Гибкость**: Поддержка нескольких подходов к стилизации
- **Производительность**: Оптимизированная вставка стилей
- **Темизация**: Полная поддержка темизации через ThemeProvider
- **Типобезопасность**: Полная типизация через TypeScript
- **SSR**: Хорошая поддержка серверного рендеринга
- **Кэширование**: Эффективное кэширование стилей

## Недостатки и ограничения

- **Сложность настройки**: Требует больше настройки по сравнению с CSS-модулями
- **Размер бандла**: Может увеличить размер бандла из-за runtime-обработки
- **Обучение**: Требует изучения нового подхода к стилизации
- **Отладка**: Иногда сложнее отлаживать стили в браузере

## Лучшие практики

1. **Используйте консистентный подход**:
   ```typescript
   // Выберите один подход и придерживайтесь его в проекте
   // styled API или css-prop
   ```

2. **Используйте темы для консистентности**:
   ```typescript
   const StyledComponent = ({ children }: { children: React.ReactNode }) => {
     const theme = useTheme<Theme>();
     
     return (
       <div 
         css={css`
           color: ${theme.colors.text};
           background: ${theme.colors.background};
         `}
       >
         {children}
       </div>
     );
   };
   ```

3. **Избегайте дублирования стилей**:
   ```typescript
   // Создавайте переиспользуемые стили
   const commonButtonStyle = css`
     padding: 10px 20px;
     border: none;
     border-radius: 4px;
     cursor: pointer;
   `;
   ```

## Альтернативы

- [[Styled-components]]
- [[CSS-модули]]
- [[Tailwind CSS]]
- [[Стилизация-компонентов]]

## Заключение

Emotion предоставляет мощный и гибкий способ стилизации компонентов в React-приложениях. Благодаря поддержке нескольких подходов к стилизации и полной интеграции с TypeScript, это отличный выбор для сложных приложений, где важна производительность и гибкость стилизации.

> [!tip]
> Используйте Emotion в сочетании с TypeScript для максимальной типобезопасности и автодополнения в IDE.

> [!warning]
> При использовании Emotion в SSR-приложениях обязательно настройте изоморфное кэширование стилей для корректной работы.