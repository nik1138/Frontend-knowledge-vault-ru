---
aliases: ["Тестирование React", "Стратегии тестирования", "React тесты"]
tags: 
  - #react
  - #testing
  - #javascript
  - #frontend
---

# Стратегии тестирования React приложений

## Введение

Тестирование React приложений - критический аспект разработки, обеспечивающий надежность, стабильность и качество кода. В этой статье рассмотрим основные стратегии тестирования компонентов React, начиная от юнит-тестов и заканчивая тестами сквозного типа.

## Тестовый пирамиды в React

Тестовая пирамида в React приложениях состоит из трех уровней:

- **Юнит-тесты (70%)** - тестирование отдельных компонентов и хуков
- **Интеграционные тесты (20%)** - тестирование взаимодействия между компонентами
- **Сквозные тесты (10%)** - тестирование пользовательских сценариев

```javascript
// Пример структуры тестов
├── src/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.jsx
│   │   │   └── Button.test.jsx
│   │   └── Form/
│   │       ├── Form.jsx
│   │       └── Form.test.jsx
│   └── hooks/
│       ├── useCounter/
│       │   ├── useCounter.js
│       │   └── useCounter.test.js
```

## Основы Jest

[Jest](../tools/jest.md) - популярный фреймворк для тестирования JavaScript приложений, включая React. Основные возможности:

- Автоматическое обнаружение тестов
- Встроенные моки и спайы
- Покрытие кода
- Изоляция тестов

```javascript
describe('Button component', () => {
  test('renders correctly', () => {
    expect(1).toBe(1);
  });
});
```

## React Testing Library

React Testing Library (RTL) - рекомендуемый способ тестирования React компонентов. Он фокусируется на тестировании того, как пользователь взаимодействует с компонентом.

```javascript
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button with correct text', () => {
  render(<Button>Click me</Button>);
  const button = screen.getByText(/click me/i);
  expect(button).toBeInTheDocument();
});
```

## Сравнение Enzyme и React Testing Library

| Критерий | Enzyme | React Testing Library |
|----------|--------|----------------------|
| Подход | Тестирование реализации | Тестирование поведения |
| DOM | Внутреннее состояние | Пользовательское взаимодействие |
| Поддержка | Неактивно развивается | Активно развивается |
| Рекомендация | Для legacy проектов | Для новых проектов |

## Юнит-тестирование компонентов

Юнит-тестирование проверяет отдельные компоненты в изоляции:

```javascript
import { render, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('counter increments correctly', () => {
  const { getByText } = render(<Counter />);
  const button = getByText(/increment/i);
  
  fireEvent.click(button);
  expect(getByText(/count: 1/i)).toBeInTheDocument();
});
```

## Интеграционное тестирование

Интеграционные тесты проверяют взаимодействие между компонентами:

```javascript
import { render, screen } from '@testing-library/react';
import TodoApp from './TodoApp';

test('adds todo item', () => {
  render(<TodoApp />);
  // Тестирование взаимодействия между компонентами
});
```

## Тестирование хуков

Для тестирования кастомных хуков используем `renderHook`:

```javascript
import { renderHook, act } from '@testing-library/react-hooks';
import useCounter from './useCounter';

test('hook manages state correctly', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Тестирование форм

Тестирование форм включает проверку валидации, отправки и управления состоянием:

```javascript
import userEvent from '@testing-library/user-event';

test('form submission with validation', async () => {
  const user = userEvent.setup();
  const { getByLabelText, getByText } = render(<Form />);
  
  await user.type(getByLabelText(/name/i), 'John');
  await user.click(getByText(/submit/i));
  
  expect(getByText(/success/i)).toBeInTheDocument();
});
```

## Асинхронное тестирование

Для тестирования асинхронных операций используем `async/await` и `waitFor`:

```javascript
test('fetches data correctly', async () => {
  server.use(
    rest.get('/api/data', (req, res, ctx) => {
      return res(ctx.json({ data: 'test' }));
    })
  );
  
  render(<DataComponent />);
  
  expect(await screen.findByText(/test/i)).toBeInTheDocument();
});
```

## Тестирование с Context

Тестирование компонентов, использующих React Context:

```javascript
const wrapper = ({ children }) => (
  <ThemeProvider value={{ theme: 'dark' }}>
    {children}
  </ThemeProvider>
);

test('uses correct theme', () => {
  render(<Component />, { wrapper });
  // Проверка поведения компонента
});
```

## Мокирование стратегий

Мокирование внешних зависимостей и API:

```javascript
// Мокирование модуля
jest.mock('../api', () => ({
  fetchData: jest.fn(() => Promise.resolve({ data: 'mocked' }))
}));

// Мокирование функции
const mockFn = jest.fn();
```

## Организация тестов

Рекомендуемая структура организации тестов:

```javascript
// [ComponentName].test.jsx
describe('[ComponentName]', () => {
  describe('rendering', () => {
    test('renders correctly with default props', () => {
      // тест
    });
  });
  
  describe('interaction', () => {
    test('handles click event', () => {
      // тест
    });
  });
});
```

## Покрытие кода

Для анализа покрытия используем встроенные возможности Jest:

```bash
npm test -- --coverage
```

## Тестирование производительности

Тестирование производительности компонентов:

```javascript
test('component renders within expected time', () => {
  const start = performance.now();
  render(<HeavyComponent />);
  const end = performance.now();
  
  expect(end - start).toBeLessThan(100);
});
```

## Доступность (Accessibility) тестирование

Проверка доступности компонентов:

```javascript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('has no accessibility violations', async () => {
  const { container } = render(<Component />);
  const results = await axe(container);
  
  expect(results).toHaveNoViolations();
});
```

## CI/CD интеграция

Интеграция тестов в CI/CD пайплайн:

```yaml
# .github/workflows/test.yml
- name: Run tests
  run: npm test -- --coverage
- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v1
```

## Лучшие практики

- Пишите тесты, которые проверяют поведение, а не реализацию
- Используйте семантические селекторы из RTL
- Изолируйте тесты друг от друга
- Тестируйте пользовательские сценарии
- Поддерживайте баланс между различными уровнями тестирования

## См. также

- [[react/components/testing]] - Тестирование компонентов
- [[react/hooks/testing]] - Тестирование хуков
- [[react/tools/jest]] - Jest инструмент
- [[react/tools/cypress]] - E2E тестирование
- [[react/testing/accessibility]] - Тестирование доступности
- [[react/testing/performance]] - Тестирование производительности

## Заключение

Эффективные стратегии тестирования React приложений обеспечивают стабильность, надежность и высокое качество кода. Используйте правильный баланс между различными уровнями тестирования и следуйте лучшим практикам для достижения наилучших результатов.