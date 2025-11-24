---
aliases: [Snapshot тестирование, Снапшот тестирование, Jest Snapshot]
tags: [testing, javascript, typescript, frontend, react, jest]
---

# Snapshot тестирование компонентов - проверка визуальных изменений

Snapshot тестирование - это метод тестирования, при котором сохраняется "снимок" результата рендеринга компонента, и в последующих тестах сравнивается с текущим результатом. Это позволяет быстро выявлять непреднамеренные визуальные изменения в компонентах.

## Обзор Snapshot тестирования

Snapshot тестирование было популяризировано фреймворком Jest и стало стандартным подходом для проверки визуальных изменений в компонентах. При первом запуске теста создается снимок (snapshot) рендеринга компонента, который сохраняется в файле. При последующих запусках теста текущий результат сравнивается с сохраненным снимком, и тест проходит успешно, только если они совпадают.

### Преимущества Snapshot тестирования

- **Быстрая проверка визуальных изменений**: Легко выявлять непреднамеренные изменения в UI
- **Минимальные усилия на написание тестов**: Не нужно вручную писать сложные проверки
- **Полное покрытие структуры**: Проверяется вся структура компонента, включая атрибуты
- **Документирование ожидаемого поведения**: Снапшоты служат документацией ожидаемого результата

### Недостатки Snapshot тестирования

- **Ложные срабатывания**: Тесты могут падать при незначительных, но намеренных изменениях
- **Недостаточная семантическая проверка**: Не проверяет функциональность, только структуру
- **Зависимость от данных**: Если компонент зависит от внешних данных, снапшоты могут быть хрупкими

## Основы Snapshot тестирования

Для использования snapshot тестирования в TypeScript проекте с Jest:

```typescript
// UserProfile.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import UserProfile from './UserProfile';

test('matches snapshot for user profile', () => {
  const { asFragment } = render(
    <UserProfile 
      user={{ 
        id: '123', 
        name: 'John Doe', 
        email: 'john@example.com',
        avatar: 'https://example.com/avatar.jpg'
      }} 
    />
  );
  
  expect(asFragment()).toMatchSnapshot();
});
```

При первом запуске теста Jest создаст файл с расширением `.snap` в папке `__snapshots__`, содержащий сериализованный снимок компонента:

```typescript
// UserProfile.test.tsx.snap
exports[`matches snapshot for user profile 1`] = `
<div>
  <img
    alt="Avatar"
    src="https://example.com/avatar.jpg"
  />
  <h1>
    John Doe
  </h1>
  <p>
    john@example.com
  </p>
</div>
`;
```

## Практические примеры

### Простой компонент

```typescript
// Button.tsx
import React from 'react';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary', 
  disabled = false, 
  children, 
  onClick 
}) => {
  const baseClass = 'btn';
  const variantClass = `btn-${variant}`;
  const disabledClass = disabled ? 'btn-disabled' : '';
  
  return (
    <button 
      className={`${baseClass} ${variantClass} ${disabledClass}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

export default Button;
```

```typescript
// Button.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import Button from './Button';

test('matches snapshot for primary button', () => {
  const { asFragment } = render(<Button>Click me</Button>);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for secondary button', () => {
  const { asFragment } = render(<Button variant="secondary">Click me</Button>);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for disabled button', () => {
  const { asFragment } = render(<Button disabled>Click me</Button>);
  expect(asFragment()).toMatchSnapshot();
});
```

### Компонент с состоянием

```typescript
// Counter.tsx
import React, { useState } from 'react';

interface CounterProps {
  initialValue?: number;
}

const Counter: React.FC<CounterProps> = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue);

  return (
    <div className="counter">
      <span>Count: {count}</span>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
    </div>
  );
};

export default Counter;
```

```typescript
// Counter.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import Counter from './Counter';

test('matches snapshot for counter with initial value', () => {
  const { asFragment } = render(<Counter initialValue={5} />);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for counter with default value', () => {
  const { asFragment } = render(<Counter />);
  expect(asFragment()).toMatchSnapshot();
});
```

## Мокирование динамических данных

Один из основных вызовов при snapshot тестировании - работа с динамическими данными, такими как даты или случайные ID:

```typescript
// UserCard.tsx
import React from 'react';

interface UserCardProps {
  user: {
    id: string;
    name: string;
    email: string;
    createdAt: Date;
  };
}

const UserCard: React.FC<UserCardProps> = ({ user }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>Email: {user.email}</p>
      <p>Member since: {user.createdAt.toLocaleDateString()}</p>
    </div>
  );
};

export default UserCard;
```

```typescript
// UserCard.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import UserCard from './UserCard';

// Мокаем дату для предсказуемости
const mockDate = new Date('2023-01-01T00:00:00.000Z');
global.Date = jest.fn(() => mockDate) as any;

test('matches snapshot for user card with mocked date', () => {
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    createdAt: new Date('2023-01-01T00:00:00.000Z')
  };

  const { asFragment } = render(<UserCard user={user} />);
  expect(asFragment()).toMatchSnapshot();
});
```

Альтернативно, можно использовать функцию для замены динамических значений:

```typescript
// UserCard.test.tsx (альтернативный подход)
import React from 'react';
import { render } from '@testing-library/react';
import UserCard from './UserCard';

test('matches snapshot for user card with date normalization', () => {
  const user = {
    id: '123',
    name: 'John Doe',
    email: 'john@example.com',
    createdAt: new Date('2023-01-01T00:00:00.000Z')
  };

  const { asFragment } = render(<UserCard user={user} />);
  
  // Нормализуем дату перед созданием снапшота
  expect(asFragment()).toMatchSnapshot();
});

// Для замены динамических значений можно использовать:
expect(asFragment()).toMatchSnapshot({
  createdAt: expect.any(Date)
});
```

## Пользовательские снапшотные расширения

Можно создавать пользовательские расширения для Jest, чтобы улучшить процесс snapshot тестирования:

```typescript
// serializers/date-serializer.ts
import { DefaultSerializer } from 'jest-snapshot';

export default {
  test: (value: any) => value instanceof Date,
  print: (value: Date) => `Date(${value.toISOString()})`,
};
```

```javascript
// jest.config.js
module.exports = {
  // ... другие настройки
  snapshotSerializers: [
    '<rootDir>/src/serializers/date-serializer.ts',
  ],
};
```

## Тестирование компонентов с пропсами

```typescript
// ProductCard.tsx
import React from 'react';

interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  inStock: boolean;
}

interface ProductCardProps {
  product: Product;
  onAddToCart?: (productId: string) => void;
}

const ProductCard: React.FC<ProductCardProps> = ({ product, onAddToCart }) => {
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>{product.description}</p>
      <div className="price">${product.price.toFixed(2)}</div>
      <div className={product.inStock ? 'in-stock' : 'out-of-stock'}>
        {product.inStock ? 'In Stock' : 'Out of Stock'}
      </div>
      <button 
        onClick={() => onAddToCart && onAddToCart(product.id)}
        disabled={!product.inStock}
      >
        Add to Cart
      </button>
    </div>
  );
};

export default ProductCard;
```

```typescript
// ProductCard.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import ProductCard from './ProductCard';

test('matches snapshot for in-stock product', () => {
  const product = {
    id: '1',
    name: 'Laptop',
    price: 999.99,
    description: 'High-performance laptop',
    inStock: true
  };

  const { asFragment } = render(
    <ProductCard 
      product={product} 
      onAddToCart={jest.fn()} 
    />
  );
  
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for out-of-stock product', () => {
  const product = {
    id: '2',
    name: 'Mouse',
    price: 29.99,
    description: 'Wireless mouse',
    inStock: false
  };

  const { asFragment } = render(
    <ProductCard 
      product={product} 
      onAddToCart={jest.fn()} 
    />
  );
  
  expect(asFragment()).toMatchSnapshot();
});
```

## Расширенные возможности

### Снапшоты с различными состояниями

```typescript
// TodoItem.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import TodoItem from './TodoItem';

const todo = {
  id: '1',
  text: 'Buy groceries',
  completed: false
};

test('matches snapshot for incomplete todo', () => {
  const { asFragment } = render(<TodoItem todo={todo} />);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for completed todo', () => {
  const completedTodo = { ...todo, completed: true };
  const { asFragment } = render(<TodoItem todo={completedTodo} />);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot with different priorities', () => {
  const highPriorityTodo = { ...todo, priority: 'high' };
  const mediumPriorityTodo = { ...todo, priority: 'medium' };
  const lowPriorityTodo = { ...todo, priority: 'low' };
  
  expect(render(<TodoItem todo={highPriorityTodo} />).asFragment()).toMatchSnapshot('high-priority');
  expect(render(<TodoItem todo={mediumPriorityTodo} />).asFragment()).toMatchSnapshot('medium-priority');
  expect(render(<TodoItem todo={lowPriorityTodo} />).asFragment()).toMatchSnapshot('low-priority');
});
```

### Снапшоты для разных размеров экрана

```typescript
// ResponsiveComponent.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import ResponsiveComponent from './ResponsiveComponent';

test('matches snapshot for mobile view', () => {
  Object.defineProperty(window, 'innerWidth', {
    writable: true,
    configurable: true,
    value: 375,
  });
  
  const { asFragment } = render(<ResponsiveComponent />);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for tablet view', () => {
  Object.defineProperty(window, 'innerWidth', {
    writable: true,
    configurable: true,
    value: 768,
  });
  
  const { asFragment } = render(<ResponsiveComponent />);
  expect(asFragment()).toMatchSnapshot();
});

test('matches snapshot for desktop view', () => {
  Object.defineProperty(window, 'innerWidth', {
    writable: true,
    configurable: true,
    value: 1200,
  });
  
  const { asFragment } = render(<ResponsiveComponent />);
  expect(asFragment()).toMatchSnapshot();
});
```

## Обновление снапшотов

При преднамеренных изменениях в компонентах нужно обновить снапшоты:

```bash
# Обновить все снапшоты
npm test -- -u

# Или в интерактивном режиме
npm test -- --watch
# Затем нажать 'u' для обновления устаревших снапшотов
```

## Лучшие практики

### 1. Используйте снапшоты как дополнение, а не основу тестирования

```typescript
// Плохо: только снапшот
test('renders user profile', () => {
  const { asFragment } = render(<UserProfile user={mockUser} />);
  expect(asFragment()).toMatchSnapshot();
});

// Хорошо: снапшот + семантические проверки
test('renders user profile with correct information', () => {
  const { asFragment, getByText } = render(<UserProfile user={mockUser} />);
  
  // Семантические проверки
  expect(getByText(mockUser.name)).toBeInTheDocument();
  expect(getByText(mockUser.email)).toBeInTheDocument();
  
  // Снапшот для проверки структуры
  expect(asFragment()).toMatchSnapshot();
});
```

### 2. Избегайте снапшотов для компонентов с динамическими данными

```typescript
// Плохо: снапшот компонента с текущей датой
test('matches snapshot with current date', () => {
  const { asFragment } = render(<CurrentDateDisplay />);
  expect(asFragment()).toMatchSnapshot(); // Всегда будет отличаться
});

// Хорошо: мокаем дату или не используем снапшот
test('displays current date', () => {
  const mockDate = new Date('2023-01-01');
  jest
    .spyOn(global.Date, 'now')
    .mockImplementation(() => mockDate.getTime());
    
  const { getByText } = render(<CurrentDateDisplay />);
  expect(getByText('01/01/2023')).toBeInTheDocument();
  
  jest.clearAllMocks();
});
```

### 3. Используйте описательные имена для снапшотов

```typescript
test('renders primary button with icon', () => {
  const { asFragment } = render(
    <Button variant="primary" icon="home">
      Home
    </Button>
  );
  expect(asFragment()).toMatchSnapshot('primary-button-with-icon');
});
```

## Заключение

Snapshot тестирование является мощным инструментом для проверки визуальных изменений в компонентах. При правильном использовании оно помогает быстро выявлять непреднамеренные изменения и служит формой документации ожидаемого поведения компонентов. Однако важно использовать его как дополнение к другим видам тестирования, а не как основу тестовой стратегии.

См. также:
- [[Testing-Library]]
- [[Jest-с-компонентами]]
- [[Cypress-для-компонентов]]
- [[E2E-тестирование-компонентов]]