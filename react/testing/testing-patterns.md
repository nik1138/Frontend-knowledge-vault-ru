---
aliases: ["Тестирование React компонентов", "Паттерны тестирования React"]
tags: ["#react", "#testing", "#javascript", "#frontend", "#best-practices"]
---

# Паттерны тестирования React

## Обзор

Тестирование React приложений требует понимания различных уровней тестирования и подходов. В этом руководстве рассмотрены ключевые паттерны для эффективного тестирования React компонентов и приложений.

## Unit тестирование React компонентов

Unit тестирование направлено на проверку отдельных компонентов в изоляции.

```jsx
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button with correct text', () => {
  render(<Button text="Click me" />);
  const buttonElement = screen.getByText(/click me/i);
  expect(buttonElement).toBeInTheDocument();
});
```

Ключевые паттерны:
- Тестирование рендеринга компонентов
- Проверка свойств (props)
- Тестирование обработчиков событий
- Использование [[react/testing/react-testing-library|React Testing Library]]

## Интеграционное тестирование в React

Интеграционные тесты проверяют взаимодействие между компонентами.

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import TodoList from './TodoList';
import TodoItem from './TodoItem';

test('completes todo item', () => {
  render(<TodoList />);
  const todoItem = screen.getByText(/learn react testing/i);
  fireEvent.click(todoItem);
  expect(todoItem).toHaveClass('completed');
});
```

## E2E тестирование подходы

Для E2E тестирования рекомендуется использовать:
- [[cypress]] для комплексного тестирования пользовательских сценариев
- Puppeteer для автоматизации браузера
- Playwright для современных E2E тестов

## Паттерны React Testing Library

React Testing Library предоставляет инструменты для тестирования компонентов так, как это делает пользователь.

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('increases counter on button click', () => {
  render(<Counter />);
  const button = screen.getByRole('button', { name: /increase/i });
  fireEvent.click(button);
  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});
```

## Jest с React

Jest интегрируется с React через `@testing-library/react` для unit тестирования.

```jsx
import renderer from 'react-test-renderer';
import Header from './Header';

test('matches snapshot', () => {
  const tree = renderer.create(<Header title="Test App" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```

## Снэпшот тестирование

Снэпшоты полезны для отслеживания изменений в разметке компонентов:

> [!warning] Внимание
> Не используйте снэпшоты для компонентов с часто изменяющимися данными

```jsx
test('header renders correctly', () => {
  const { asFragment } = render(<Header />);
  expect(asFragment()).toMatchSnapshot();
});
```

## Стратегии мокирования

Мокирование позволяет изолировать компоненты:

```jsx
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';

jest.mock('./api', () => ({
  fetchUser: jest.fn(() => Promise.resolve({ name: 'John' }))
}));

test('displays user name', async () => {
  render(<UserProfile userId={1} />);
  expect(await screen.findByText(/john/i)).toBeInTheDocument();
});
```

## Паттерны тестирования хуков

Для тестирования хуков используйте `renderHook`:

```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

test('initializes with zero', () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0);
});
```

## Тестирование провайдеров и потребителей контекста

```jsx
import { render, screen } from '@testing-library/react';
import { ThemeProvider, ThemeContext } from './ThemeContext';

const TestComponent = () => {
  const theme = React.useContext(ThemeContext);
  return <div data-testid="theme">{theme}</div>;
};

test('provides theme to child components', () => {
  render(
    <ThemeProvider>
      <TestComponent />
    </ThemeProvider>
  );
  expect(screen.getByTestId('theme')).toHaveTextContent('light');
});
```

## Тестирование форм и пользовательских взаимодействий

```jsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import LoginForm from './LoginForm';

test('submits form with valid data', async () => {
  render(<LoginForm onSubmit={jest.fn()} />);
  
  fireEvent.change(screen.getByLabelText(/email/i), {
    target: { value: 'test@example.com' }
  });
  
  fireEvent.click(screen.getByRole('button', { name: /submit/i }));
  
  await waitFor(() => {
    expect(screen.getByText(/success/i)).toBeInTheDocument();
  });
});
```

## Асинхронные тесты с React

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import AsyncComponent from './AsyncComponent';

test('shows data after loading', async () => {
  render(<AsyncComponent />);
  expect(screen.getByText(/loading/i)).toBeInTheDocument();
  
  await waitFor(() => screen.getByText(/loaded data/i));
  expect(screen.getByText(/loaded data/i)).toBeInTheDocument();
});
```

## Тестирование границ ошибок

```jsx
import { render, screen } from '@testing-library/react';
import ErrorBoundary from './ErrorBoundary';

const BrokenComponent = () => {
  throw new Error('Broken!');
};

test('shows error message when child throws', () => {
  render(
    <ErrorBoundary>
      <BrokenComponent />
    </ErrorBoundary>
  );
  expect(screen.getByText(/something went wrong/i)).toBeInTheDocument();
});
```

## Тестирование производительности

Для тестирования производительности используйте:
- `console.time()` для измерения времени рендеринга
- React DevTools Profiler
- Web Vitals для измерения производительности пользовательского опыта

## Тестирование доступности

```jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import App from './App';

expect.extend(toHaveNoViolations);

test('has no accessibility violations', async () => {
  const { container } = render(<App />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

## Отладка тестов в React

Для отладки тестов:
- Используйте `debug()` для просмотра DOM
- Запускайте тесты в режиме watch: `npm test -- --watch`
- Используйте `screen.debug()` для логирования текущего DOM

## Организация и структура тестов

Рекомендуемая структура:
- `Component.test.jsx` рядом с компонентом
- `hooks/` для тестов хуков
- `utils/` для тестов утилит
- `integration/` для интеграционных тестов

## Связанные темы

- [[react/react-best-practices]]
- [[react/react-architecture]]
- [[react/testing/react-testing-library]]
- [[react/testing/jest-setup]]
- [[react/testing/cypress-integration]]
- [[react/hooks/testing-hooks]]

## Заключение

Эффективное тестирование React приложений требует сочетания unit, интеграционного и E2E тестирования. Используйте React Testing Library для тестирования пользовательских взаимодействий, Jest для unit тестирования и соответствующие инструменты для E2E тестирования. Следуйте паттернам, описанным в этом руководстве, чтобы создавать надежные и поддерживаемые тесты.