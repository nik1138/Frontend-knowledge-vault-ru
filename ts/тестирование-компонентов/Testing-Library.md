---
aliases: [Testing Library, React Testing Library, DOM Testing Library]
tags: [testing, javascript, typescript, frontend, react]
---

# Testing Library - основы тестирования компонентов

Testing Library - это набор библиотек и утилит, которые помогают тестировать компоненты с точки зрения пользователя. Основная идея Testing Library заключается в том, чтобы тестировать компоненты так, как они будут использоваться реальными пользователями, а не тестировать внутреннюю реализацию.

## Обзор Testing Library

Testing Library предоставляет простой и эффективный способ тестирования компонентов, сосредоточившись на поведении, а не на деталях реализации. Это делает тесты более надежными и устойчивыми к рефакторингу.

### Основные принципы Testing Library

- **Тестирование как пользователь**: Тесты должны воспроизводить поведение пользователя, а не внутреннюю реализацию компонента
- **Сосредоточение на поведении**: Тесты должны проверять, что компонент делает, а не как он это делает
- **Избегание тестирования деталей реализации**: Не тестировать внутренние состояния, переменные или методы компонента

## Установка Testing Library

Для использования Testing Library в проекте на React с TypeScript, установите необходимые зависимости:

```bash
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

## Основные функции Testing Library

### render()

Функция `render()` используется для рендеринга компонента в тестовой среде. Она возвращает объект с методами для взаимодействия с компонентом.

```typescript
import { render } from '@testing-library/react';
import MyComponent from './MyComponent';

test('renders component correctly', () => {
  const { getByText } = render(<MyComponent />);
  const element = getByText(/hello world/i);
  expect(element).toBeInTheDocument();
});
```

### getBy*, queryBy*, findBy*

Testing Library предоставляет различные методы для поиска элементов:

- `getBy*` - возвращает элемент, если он существует, иначе выбрасывает ошибку
- `queryBy*` - возвращает элемент, если он существует, иначе возвращает null
- `findBy*` - асинхронно возвращает элемент (полезно для элементов, которые появляются после асинхронных операций)

### Селекторы

Testing Library предоставляет различные селекторы для поиска элементов:

- `getByRole` - поиск по ARIA ролям
- `getByLabelText` - поиск по связанной метке
- `getByPlaceholderText` - поиск по placeholder тексту
- `getByText` - поиск по содержимому текста
- `getByDisplayValue` - поиск по значению в input полях

## Практический пример

Рассмотрим пример тестирования компонента кнопки:

```typescript
// Button.tsx
import React from 'react';

interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ onClick, children, disabled = false }) => (
  <button onClick={onClick} disabled={disabled}>
    {children}
  </button>
);

export default Button;
```

```typescript
// Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

test('renders button with correct text', () => {
  render(<Button>Click me</Button>);
  const buttonElement = screen.getByText(/click me/i);
  expect(buttonElement).toBeInTheDocument();
});

test('calls onClick handler when clicked', () => {
  const mockOnClick = jest.fn();
  render(<Button onClick={mockOnClick}>Click me</Button>);
  
  const buttonElement = screen.getByRole('button', { name: /click me/i });
  fireEvent.click(buttonElement);
  
  expect(mockOnClick).toHaveBeenCalledTimes(1);
});

test('is disabled when disabled prop is true', () => {
  render(<Button disabled>Click me</Button>);
  const buttonElement = screen.getByRole('button', { name: /click me/i });
  expect(buttonElement).toBeDisabled();
});
```

## User Event

Для более реалистичного тестирования пользовательских взаимодействий используйте `@testing-library/user-event`:

```typescript
import userEvent from '@testing-library/user-event';

test('handles form submission', async () => {
  const user = userEvent.setup();
  const mockSubmit = jest.fn();
  
  render(
    <form onSubmit={mockSubmit}>
      <input type="text" data-testid="input" />
      <button type="submit">Submit</button>
    </form>
  );
  
  const input = screen.getByTestId('input');
  const button = screen.getByRole('button', { name: /submit/i });
  
  await user.type(input, 'Hello World');
  await user.click(button);
  
  expect(mockSubmit).toHaveBeenCalledTimes(1);
});
```

## Асинхронные тесты

Testing Library предоставляет удобные методы для работы с асинхронными операциями:

```typescript
// ComponentWithAsyncData.tsx
import React, { useState, useEffect } from 'react';

const ComponentWithAsyncData: React.FC = () => {
  const [data, setData] = useState<string | null>(null);
  
  useEffect(() => {
    setTimeout(() => {
      setData('Async data loaded');
    }, 1000);
  }, []);
  
  return <div>{data || 'Loading...'}</div>;
};

export default ComponentWithAsyncData;
```

```typescript
// ComponentWithAsyncData.test.tsx
import { render, screen } from '@testing-library/react';
import ComponentWithAsyncData from './ComponentWithAsyncData';

test('displays async data after loading', async () => {
  render(<ComponentWithAsyncData />);
  
  // Проверяем, что отображается "Loading..."
  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  
  // Ждем, пока асинхронные данные загрузятся
  expect(await screen.findByText(/async data loaded/i)).toBeInTheDocument();
});
```

## Настройка тестовой среды

Для правильной работы Testing Library в TypeScript проектах рекомендуется настроить тестовую среду:

```typescript
// setupTests.ts
import '@testing-library/jest-dom';
```

## Заключение

Testing Library - мощный инструмент для тестирования компонентов, который помогает создавать надежные и устойчивые к изменениям тесты. Его фокус на поведении пользователя делает тесты более значимыми и полезными для обеспечения качества приложения.

См. также:
- [[Jest-с-компонентами]]
- [[Cypress-для-компонентов]]
- [[Snapshot-тестирование]]
- [[E2E-тестирование-компонентов]]