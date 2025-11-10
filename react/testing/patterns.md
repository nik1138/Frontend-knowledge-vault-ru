---
aliases: ["Тестирование React паттерны", "React тестирование"]
tags: 
  - "#react"
  - "#testing"
  - "#best-practices"
  - "#frontend"
---

# Паттерны тестирования React

## Обзор

Тестирование в React - это критическая часть разработки надежных приложений. Правильная стратегия тестирования обеспечивает стабильность компонентов и быстрое выявление ошибок.

## Типы тестирования

### Модульное тестирование

Модульное тестирование проверяет отдельные компоненты изолированно. Используйте [[React Testing Library]] для рендеринга компонентов и проверки их поведения.

```jsx
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('отображает текст кнопки', () => {
  render(<Button>Нажми меня</Button>);
  expect(screen.getByText('Нажми меня')).toBeInTheDocument();
});
```

### Интеграционное тестирование

Интеграционные тесты проверяют взаимодействие между компонентами. Проверяйте, как компоненты работают вместе в более крупных сегментах UI.

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import TodoList from './TodoList';

test('добавляет новую задачу при отправке формы', () => {
  render(<TodoList />);
  const input = screen.getByPlaceholderText('Добавить задачу');
  const button = screen.getByText('Добавить');

  fireEvent.change(input, { target: { value: 'Новая задача' } });
  fireEvent.click(button);

  expect(screen.getByText('Новая задача')).toBeInTheDocument();
});
```

### E2E тестирование

End-to-end тесты проверяют приложение как единое целое, имитируя действия пользователя. Используйте [[Cypress]] или [[Playwright]] для автоматизации браузерных сценариев.

## Библиотеки тестирования

### Jest

Jest - это популярный фреймворк для тестирования JavaScript. Он включает в себя встроенные средства для мокирования, снапшотов и покрытия кода.

### React Testing Library

React Testing Library позволяет тестировать компоненты так, как пользователь видит и взаимодействует с ними. Сосредоточьтесь на поведении, а не на реализации.

### Enzyme

Enzyme был популярным инструментом для тестирования React, но теперь рекомендуется использовать React Testing Library из-за его фокуса на поведении пользователя.

## Основные паттерны тестирования

### Тестирование рендеринга компонентов

Проверьте, что компонент правильно отображает ожидаемый контент:

```jsx
test('рендерит заголовок', () => {
  render(<Header title="Мое приложение" />);
  expect(screen.getByText('Мое приложение')).toBeInTheDocument();
});
```

### Тестирование взаимодействия

Имитируйте действия пользователя и проверьте реакцию компонента:

```jsx
test('вызывает колбэк при клике', () => {
  const handleClick = jest.fn();
  render(<Button onClick={handleClick}>Кликни</Button>);
  
  fireEvent.click(screen.getByText('Кликни'));
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

### Тестирование состояния

Проверьте, как компонент обрабатывает изменения состояния:

```jsx
test('обновляет счетчик при клике', () => {
  render(<Counter />);
  const button = screen.getByText('Счетчик: 0');
  
  fireEvent.click(button);
  expect(screen.getByText('Счетчик: 1')).toBeInTheDocument();
});
```

### Тестирование пропсов

Убедитесь, что компонент корректно использует переданные пропсы:

```jsx
test('отображает имя пользователя', () => {
  render(<UserProfile name="Иван" />);
  expect(screen.getByText('Имя: Иван')).toBeInTheDocument();
});
```

## Тестирование хуков

Тестируйте пользовательские хуки с помощью специальной обертки:

```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

test('инициализирует счетчик с 0', () => {
  const { result } = renderHook(() => useCounter());
  expect(result.current.count).toBe(0);
});

test('увеличивает счетчик', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Тестирование форм

Проверьте обработку ввода и валидацию:

```jsx
test('проверяет валидацию формы', async () => {
  render(<LoginForm />);
  const button = screen.getByRole('button');
  
  fireEvent.click(button);
  expect(await screen.findByText('Email обязателен')).toBeInTheDocument();
});
```

## Тестирование маршрутизации

Используйте [[React Router]] тест утилиты для проверки навигации:

```jsx
import { render, screen } from '@testing-library/react';
import { MemoryRouter } from 'react-router-dom';
import App from './App';

test('рендерит главную страницу', () => {
  render(
    <MemoryRouter initialEntries={['/']}>
      <App />
    </MemoryRouter>
  );
  
  expect(screen.getByText('Главная')).toBeInTheDocument();
});
```

## Асинхронное тестирование

Работайте с асинхронными операциями с помощью `async/await`:

```jsx
test('загружает данные', async () => {
  render(<DataComponent />);
  
  expect(screen.getByText('Загрузка...')).toBeInTheDocument();
  expect(await screen.findByText('Данные загружены')).toBeInTheDocument();
});
```

## Тестирование доступности

Проверьте компоненты на соответствие стандартам доступности:

```jsx
test('имеет корректную разметку', () => {
  const { container } = render(<Button>Кнопка</Button>);
  expect(container.firstChild).toHaveAccessibleName('Кнопка');
});
```

## Визуальное регрессионное тестирование

Используйте инструменты вроде [[Percy]] или [[Chromatic]] для обнаружения визуальных изменений.

## Снапшот тестирование

Сохраняйте снимки компонентов и проверяйте их изменения:

```jsx
import renderer from 'react-test-renderer';

test('соответствует снапшоту', () => {
  const tree = renderer.create(<Button>Кнопка</Button>).toJSON();
  expect(tree).toMatchSnapshot();
});
```

## Стратегии мокирования

Используйте моки для изоляции компонентов:

```jsx
jest.mock('./api', () => ({
  fetchData: jest.fn(() => Promise.resolve({ data: 'test' }))
}));
```

## Лучшие практики

- Пишите тесты, которые проверяют поведение, а не реализацию
- Используйте семантичные селекторы DOM
- Избегайте тестирования внутренних деталей реализации
- Поддерживайте баланс между типами тестов
- Пишите понятные описания тестов

## TDD в React

Разработка через тестирование (TDD) начинается с написания теста до реализации:

1. Напишите тест для нового функционала
2. Убедитесь, что тест падает
3. Напишите минимальный код для прохождения теста
4. Рефакторите при необходимости

## Рабочий процесс тестирования компонентов

1. Определите поведение компонента
2. Напишите тесты для ожидаемого поведения
3. Реализуйте компонент
4. Запустите тесты
5. Повторите при необходимости

## Тестирование управления состоянием

Проверьте интеграцию с Redux или Context API:

```jsx
import { render, screen } from '@testing-library/react';
import { Provider } from 'react-redux';
import configureStore from 'redux-mock-store';
import TodoList from './TodoList';

test('отображает задачи из состояния', () => {
  const mockStore = configureStore();
  const store = mockStore({
    todos: [{ id: 1, text: 'Задача 1', completed: false }]
  });
  
  render(
    <Provider store={store}>
      <TodoList />
    </Provider>
  );
  
  expect(screen.getByText('Задача 1')).toBeInTheDocument();
});
```

## Тестирование обработчиков ошибок

Проверьте, как компоненты обрабатывают ошибки:

```jsx
import { render, screen } from '@testing-library/react';
import ErrorBoundary from './ErrorBoundary';

test('отображает сообщение об ошибке', () => {
  const BrokenComponent = () => {
    throw new Error('Ошибка!');
  };
  
  render(
    <ErrorBoundary>
      <BrokenComponent />
    </ErrorBoundary>
  );
  
  expect(screen.getByText('Что-то пошло не так')).toBeInTheDocument();
});
```

## Инструменты для тестирования производительности

Используйте [[Lighthouse]] или [[Web Vitals]] для измерения производительности в тестах.

## Антипаттерны тестирования

- Тестирование слишком большого количества внутренних деталей
- Создание хрупких тестов, которые часто ломаются
- Отсутствие тестов для критических путей пользователя
- Игнорирование асинхронных операций

## Покрытие кода

Стремитесь к разумному покрытию (обычно 80-90%), но не гонитесь за 100%:

- Покрывайте критические пути
- Тестируйте граничные условия
- Проверяйте обработку ошибок

## Связанные темы

- [[React best practices]]
- [[Component architecture]]
- [[State management patterns]]
- [[Testing tools comparison]]