# Тестирование React: Комплексное Руководство

## Введение

Тестирование является критически важной частью разработки React-приложений. Это помогает обеспечить стабильность кода, предотвращает регрессии и улучшает качество программного обеспечения. В этом руководстве мы рассмотрим все аспекты тестирования React-приложений.

## Пирамида тестирования в React

Пирамида тестирования в React состоит из трех уровней:

- **Модульное тестирование** (~70%): Тестирование отдельных компонентов и функций
- **Интеграционное тестирование** (~20%): Тестирование взаимодействия между компонентами
- **Сквозное тестирование** (~10%): Тестирование полных пользовательских сценариев

```javascript
// Пример структуры тестов в пирамиде
// unit/ - модульные тесты
// integration/ - интеграционные тесты  
// e2e/ - сквозные тесты
```

## Модульное тестирование с Jest и React Testing Library

Jest - это популярный фреймворк для тестирования JavaScript, а React Testing Library предоставляет удобные утилиты для тестирования React-компонентов.

### Установка зависимостей

```bash
npm install --save-dev jest @testing-library/react @testing-library/jest-dom
```

### Базовый пример теста

```jsx
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import Button from './Button';

test('отображает текст кнопки', () => {
  render(<Button>Нажми меня</Button>);
  
  const buttonElement = screen.getByText(/нажми меня/i);
  expect(buttonElement).toBeInTheDocument();
});
```

## Тестирование компонентов

Тестирование компонентов включает проверку рендеринга, обработки событий и состояния.

```jsx
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('увеличивает счетчик при нажатии', () => {
  render(<Counter />);
  
  const button = screen.getByRole('button', { name: /увеличить/i });
  const countElement = screen.getByText('Счет: 0');
  
  fireEvent.click(button);
  
  expect(countElement).toHaveTextContent('Счет: 1');
});
```

## Интеграционное тестирование

Интеграционное тестирование проверяет взаимодействие между несколькими компонентами.

```jsx
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from './UserProfile';
import { fetchUser } from '../api';

// Мокаем API
jest.mock('../api');

test('отображает профиль пользователя после загрузки', async () => {
  fetchUser.mockResolvedValue({ name: 'Иван', email: 'ivan@example.com' });
  
  render(<UserProfile userId="123" />);
  
  expect(screen.getByText(/загрузка/i)).toBeInTheDocument();
  
  await waitFor(() => {
    expect(screen.getByText('Иван')).toBeInTheDocument();
    expect(screen.getByText('ivan@example.com')).toBeInTheDocument();
  });
});
```

## Сквозное тестирование с Cypress

Cypress используется для тестирования полных пользовательских сценариев в реальном браузере.

```javascript
// cypress/integration/user-flow.spec.js
describe('Пользовательский поток', () => {
  it('регистрация нового пользователя', () => {
    cy.visit('/register');
    cy.get('[data-cy=name-input]').type('Иван Иванов');
    cy.get('[data-cy=email-input]').type('ivan@example.com');
    cy.get('[data-cy=password-input]').type('password123');
    cy.get('[data-cy=submit-btn]').click();
    
    cy.url().should('include', '/dashboard');
    cy.contains('Добро пожаловать, Иван Иванов!');
  });
});
```

## Snapshot-тестирование

Snapshot-тестирование позволяет проверить, что UI не изменился неожиданно.

```jsx
import { render } from '@testing-library/react';
import Form from './Form';

test('снимок формы не изменился', () => {
  const { asFragment } = render(<Form />);
  expect(asFragment()).toMatchSnapshot();
});
```

## Тестирование хуков

Для тестирования пользовательских хуков используйте `renderHook`.

```jsx
import { renderHook, act } from '@testing-library/react';
import useCounter from './useCounter';

test('хук useCounter работает корректно', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current.count).toBe(0);
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

## Тестирование состояния и пропсов

```jsx
import { render, screen } from '@testing-library/react';
import UserCard from './UserCard';

test('отображает имя пользователя из пропсов', () => {
  render(<UserCard name="Анна" />);
  
  expect(screen.getByText('Анна')).toBeInTheDocument();
});

test('отображает аватар при наличии', () => {
  render(<UserCard name="Анна" avatar="/avatar.jpg" />);
  
  const avatar = screen.getByRole('img');
  expect(avatar).toHaveAttribute('src', '/avatar.jpg');
});
```

## Тестирование жизненных циклов

Для компонентов на основе классов:

```jsx
import { render } from '@testing-library/react';
import UserProfile from './UserProfile';

test('компонент загружает данные при монтировании', () => {
  const mockFetchUser = jest.fn();
  UserProfile.prototype.fetchUser = mockFetchUser;
  
  render(<UserProfile userId="123" />);
  
  expect(mockFetchUser).toHaveBeenCalledWith('123');
});
```

Для компонентов на основе хуков:

```jsx
import { render, waitFor } from '@testing-library/react';
import UserList from './UserList';

test('загружает список пользователей при монтировании', async () => {
  render(<UserList />);
  
  await waitFor(() => {
    expect(screen.getByText('Пользователь 1')).toBeInTheDocument();
  });
});
```

## Лучшие практики тестирования React-компонентов

- Пишите тесты, которые отражают поведение пользователя
- Используйте семантические методы для поиска элементов
- Избегайте тестирования внутренней реализации
- Поддерживайте тесты в актуальном состоянии
- Организуйте тесты по принципу AAA (Arrange, Act, Assert)

## Мокирование и заглушки в тестах React

```jsx
// Мокирование модуля
jest.mock('../api', () => ({
  fetchUserData: jest.fn(() => Promise.resolve({ name: 'Иван' }))
}));

// Мокирование функции
const mockFn = jest.fn();
fireEvent.click(screen.getByRole('button'));
expect(mockFn).toHaveBeenCalled();

// Мокирование импорта
jest.mock('react-i18next', () => ({
  useTranslation: () => ({ t: (key) => key })
}));
```

## Разработка через тестирование (TDD) в React

TDD - это подход, при котором сначала пишутся тесты, а затем код для прохождения этих тестов.

```jsx
// 1. Сначала пишем тест
test('отображает заголовок', () => {
  render(<Header />);
  expect(screen.getByRole('heading')).toBeInTheDocument();
});

// 2. Затем реализуем компонент
const Header = () => <h1>Мой заголовок</h1>;

// 3. Запускаем тест и убеждаемся, что он проходит
```

## Тестирование производительности

```jsx
import { render } from '@testing-library/react';

test('компонент рендерится быстро', () => {
  const start = performance.now();
  render(<ComplexComponent />);
  const end = performance.now();
  
  expect(end - start).toBeLessThan(100); // менее 100мс
});
```

## Тестирование доступности

```jsx
import { render } from '@testing-library/react';
import '@testing-library/jest-dom';

test('кнопка имеет корректную семантику', () => {
  const { container } = render(<Button>Нажми меня</Button>);
  
  expect(container.firstChild).toHaveAccessibleName('Нажми меня');
  expect(container.firstChild).toHaveAttribute('role', 'button');
});
```

## Отладка тестов

Для отладки тестов используйте:

- `debug()` для вывода DOM-структуры
- `console.log` в тестах
- режим отладки Jest: `npm test -- --debug`

```jsx
import { render, screen } from '@testing-library/react';

test('отладка компонента', () => {
  render(<MyComponent />);
  screen.debug(); // выводит DOM в консоль
});
```

## Связь с другими файлами React

- [[react/components/components]] - компоненты React
- [[react/hooks/hooks]] - хуки React
- [[react/state-management/state-management]] - управление состоянием
- [[react/performance/performance]] - производительность
- [[react/accessibility/accessibility]] - доступность

## Заключение

Эффективное тестирование React-приложений требует комбинации различных подходов и инструментов. Следование лучшим практикам и использование правильных инструментов помогает создавать надежные и поддерживаемые приложения.

#react #testing #jest #react-testing-library #cypress #best-practices #frontend