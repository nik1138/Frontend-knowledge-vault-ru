---
aliases: [Cypress для компонентов, Cypress Component Testing, Тестирование компонентов Cypress]
tags: [testing, javascript, typescript, frontend, react, cypress]
---

# Cypress для компонентов - сквозное тестирование UI

Cypress - это мощный фреймворк для тестирования, который традиционно использовался для E2E тестирования, но теперь также предоставляет возможности для тестирования компонентов изолированно. Cypress Component Testing позволяет тестировать компоненты в реалистичной среде браузера с возможностью манипуляции DOM и взаимодействия с пользовательскими событиями.

## Обзор Cypress Component Testing

Cypress Component Testing был представлен как альтернатива другим инструментам тестирования компонентов, таким как Jest и Testing Library. В отличие от них, Cypress запускает тесты в реальном браузере, что позволяет тестировать компоненты с учетом всех аспектов веб-рендеринга, включая CSS, анимации и взаимодействие с DOM.

### Преимущества Cypress Component Testing

- **Реалистичная среда**: Тесты запускаются в реальном браузере, что обеспечивает более точное воспроизведение поведения компонента
- **Мощные инструменты отладки**: Встроенный отладчик, возможность записи видео и скриншотов
- **Интеграция с CI/CD**: Хорошо интегрируется с системами непрерывной интеграции
- **Надежные селекторы**: Использование селекторов, которые имитируют поведение пользователя

## Установка Cypress Component Testing

Для использования Cypress Component Testing в проекте с TypeScript и React:

```bash
npm install --save-dev cypress @cypress/react
```

После установки запустите инициализацию Cypress:

```bash
npx cypress open
```

Это создаст необходимые конфигурационные файлы. Затем настройте Cypress для тестирования компонентов, изменив `cypress.config.ts`:

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite', // или 'webpack' в зависимости от вашего сборщика
    },
  },
});
```

## Базовое тестирование компонентов

### Простой пример тестирования компонента

```typescript
// Button.tsx
import React from 'react';

interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
  disabled?: boolean;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ 
  onClick, 
  children, 
  disabled = false, 
  variant = 'primary' 
}) => {
  const baseClasses = 'btn';
  const variantClass = `btn-${variant}`;
  const disabledClass = disabled ? 'btn-disabled' : '';
  
  return (
    <button 
      className={`${baseClasses} ${variantClass} ${disabledClass}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;
```

```typescript
// cypress/component/Button.cy.tsx
import React from 'react';
import Button from '../../src/components/Button';

describe('Button Component', () => {
  it('renders correctly with children', () => {
    cy.mount(<Button>Click me</Button>);
    cy.get('button').should('contain.text', 'Click me');
  });

  it('calls onClick handler when clicked', () => {
    const onClickSpy = cy.spy().as('onClickSpy');
    cy.mount(<Button onClick={onClickSpy}>Click me</Button>);
    
    cy.get('button').click();
    cy.get('@onClickSpy').should('have.been.calledOnce');
  });

  it('is disabled when disabled prop is true', () => {
    cy.mount(<Button disabled>Click me</Button>);
    cy.get('button').should('be.disabled');
  });

  it('applies correct CSS classes based on variant', () => {
    cy.mount(<Button variant="secondary">Click me</Button>);
    cy.get('button')
      .should('have.class', 'btn')
      .and('have.class', 'btn-secondary');
  });
});
```

## Расширенные возможности Cypress для компонентов

### Тестирование состояния компонента

```typescript
// Counter.tsx
import React, { useState } from 'react';

interface CounterProps {
  initialValue?: number;
}

const Counter: React.FC<CounterProps> = ({ initialValue = 0 }) => {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(count + 1);
  const decrement = () => setCount(count - 1);

  return (
    <div className="counter">
      <span data-cy="count">Count: {count}</span>
      <button data-cy="increment-btn" onClick={increment}>+</button>
      <button data-cy="decrement-btn" onClick={decrement}>-</button>
    </div>
  );
};

export default Counter;
```

```typescript
// cypress/component/Counter.cy.tsx
import React from 'react';
import Counter from '../../src/components/Counter';

describe('Counter Component', () => {
  it('starts with initial value', () => {
    cy.mount(<Counter initialValue={5} />);
    cy.get('[data-cy=count]').should('have.text', 'Count: 5');
  });

  it('increments count when increment button is clicked', () => {
    cy.mount(<Counter />);
    cy.get('[data-cy=count]').should('have.text', 'Count: 0');
    
    cy.get('[data-cy=increment-btn]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 1');
    
    cy.get('[data-cy=increment-btn]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 2');
  });

  it('decrements count when decrement button is clicked', () => {
    cy.mount(<Counter initialValue={5} />);
    cy.get('[data-cy=count]').should('have.text', 'Count: 5');
    
    cy.get('[data-cy=decrement-btn]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 4');
  });

  it('handles multiple clicks efficiently', () => {
    cy.mount(<Counter />);
    
    // Кликнем 10 раз
    for (let i = 0; i < 10; i++) {
      cy.get('[data-cy=increment-btn]').click();
    }
    
    cy.get('[data-cy=count]').should('have.text', 'Count: 10');
  });
});
```

### Тестирование асинхронных операций

```typescript
// AsyncComponent.tsx
import React, { useState, useEffect } from 'react';

interface AsyncComponentProps {
  fetchData: () => Promise<string>;
}

const AsyncComponent: React.FC<AsyncComponentProps> = ({ fetchData }) => {
  const [data, setData] = useState<string | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadData = async () => {
      try {
        const result = await fetchData();
        setData(result);
      } catch (err) {
        setError((err as Error).message);
      } finally {
        setLoading(false);
      }
    };

    loadData();
  }, [fetchData]);

  if (loading) return <div data-cy="loading">Loading...</div>;
  if (error) return <div data-cy="error">Error: {error}</div>;
  if (!data) return <div data-cy="no-data">No data</div>;

  return <div data-cy="data">{data}</div>;
};

export default AsyncComponent;
```

```typescript
// cypress/component/AsyncComponent.cy.tsx
import React from 'react';
import AsyncComponent from '../../src/components/AsyncComponent';

describe('AsyncComponent', () => {
  it('displays loading state initially', () => {
    const fetchData = cy.stub().resolves('Test Data');
    cy.mount(<AsyncComponent fetchData={fetchData} />);
    
    cy.get('[data-cy=loading]').should('be.visible');
    cy.get('[data-cy=data]').should('not.exist');
  });

  it('displays data when fetch is successful', () => {
    const fetchData = cy.stub().resolves('Test Data');
    cy.mount(<AsyncComponent fetchData={fetchData} />);
    
    cy.get('[data-cy=loading]').should('not.exist');
    cy.get('[data-cy=data]').should('have.text', 'Test Data');
  });

  it('displays error when fetch fails', () => {
    const fetchData = cy.stub().rejects(new Error('API Error'));
    cy.mount(<AsyncComponent fetchData={fetchData} />);
    
    cy.get('[data-cy=loading]').should('not.exist');
    cy.get('[data-cy=error]').should('have.text', 'Error: API Error');
  });

  it('handles multiple requests correctly', () => {
    const fetchData = cy.stub();
    fetchData.onFirstCall().resolves('First Data');
    fetchData.onSecondCall().resolves('Second Data');
    
    cy.mount(<AsyncComponent fetchData={fetchData} />);
    
    cy.get('[data-cy=data]').should('have.text', 'First Data');
    
    // Повторный вызов компонента с новой функцией
    cy.mount(<AsyncComponent fetchData={fetchData} />);
    cy.get('[data-cy=data]').should('have.text', 'Second Data');
  });
});
```

## Мокирование сервисов и API

Cypress предоставляет мощные возможности для мокирования API и сервисов:

```typescript
// UserProfile.tsx
import React, { useState, useEffect } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserProfileProps {
  userId: string;
  userService: {
    getUser: (id: string) => Promise<User>;
  };
}

const UserProfile: React.FC<UserProfileProps> = ({ userId, userService }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadUser = async () => {
      try {
        const userData = await userService.getUser(userId);
        setUser(userData);
      } catch (err) {
        setError((err as Error).message);
      } finally {
        setLoading(false);
      }
    };

    loadUser();
  }, [userId, userService]);

  if (loading) return <div data-cy="loading">Loading...</div>;
  if (error) return <div data-cy="error">Error: {error}</div>;
  if (!user) return <div data-cy="no-user">No user found</div>;

  return (
    <div data-cy="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

export default UserProfile;
```

```typescript
// cypress/component/UserProfile.cy.tsx
import React from 'react';
import UserProfile from '../../src/components/UserProfile';

describe('UserProfile Component', () => {
  it('displays user data when API call succeeds', () => {
    const mockUserService = {
      getUser: cy.stub().resolves({
        id: '123',
        name: 'John Doe',
        email: 'john@example.com'
      })
    };

    cy.mount(
      <UserProfile 
        userId="123" 
        userService={mockUserService} 
      />
    );

    cy.get('[data-cy=user-profile]').within(() => {
      cy.get('h2').should('have.text', 'John Doe');
      cy.get('p').should('have.text', 'john@example.com');
    });

    cy.wrap(mockUserService.getUser).should('have.been.calledOnceWith', '123');
  });

  it('displays error when API call fails', () => {
    const mockUserService = {
      getUser: cy.stub().rejects(new Error('User not found'))
    };

    cy.mount(
      <UserProfile 
        userId="999" 
        userService={mockUserService} 
      />
    );

    cy.get('[data-cy=error]').should('have.text', 'Error: User not found');
  });

  it('handles different user data correctly', () => {
    const mockUserService = {
      getUser: cy.stub()
        .onFirstCall().resolves({ id: '1', name: 'Alice', email: 'alice@example.com' })
        .onSecondCall().resolves({ id: '2', name: 'Bob', email: 'bob@example.com' })
    };

    // Тестируем первого пользователя
    cy.mount(
      <UserProfile 
        userId="1" 
        userService={mockUserService} 
      />
    );

    cy.get('[data-cy=user-profile]').within(() => {
      cy.get('h2').should('have.text', 'Alice');
      cy.get('p').should('have.text', 'alice@example.com');
    });

    // Тестируем второго пользователя
    cy.mount(
      <UserProfile 
        userId="2" 
        userService={mockUserService} 
      />
    );

    cy.get('[data-cy=user-profile]').within(() => {
      cy.get('h2').should('have.text', 'Bob');
      cy.get('p').should('have.text', 'bob@example.com');
    });
  });
});
```

## Тестирование пользовательских хуков с Cypress

```typescript
// useCounter.ts
import { useState, useCallback } from 'react';

export interface UseCounterOptions {
  initialValue?: number;
  step?: number;
}

export const useCounter = (options: UseCounterOptions = {}) => {
  const { initialValue = 0, step = 1 } = options;
  const [count, setCount] = useState(initialValue);

  const increment = useCallback(() => setCount(prev => prev + step), [step]);
  const decrement = useCallback(() => setCount(prev => prev - step), [step]);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);

  return { count, increment, decrement, reset };
};
```

```typescript
// CounterWithHook.tsx
import React from 'react';
import { useCounter } from './useCounter';

interface CounterWithHookProps {
  initialValue?: number;
  step?: number;
}

const CounterWithHook: React.FC<CounterWithHookProps> = ({ 
  initialValue = 0, 
  step = 1 
}) => {
  const { count, increment, decrement, reset } = useCounter({ 
    initialValue, 
    step 
  });

  return (
    <div>
      <span data-cy="count">Count: {count}</span>
      <button data-cy="increment" onClick={increment}>+</button>
      <button data-cy="decrement" onClick={decrement}>-</button>
      <button data-cy="reset" onClick={reset}>Reset</button>
    </div>
  );
};

export default CounterWithHook;
```

```typescript
// cypress/component/CounterWithHook.cy.tsx
import React from 'react';
import CounterWithHook from '../../src/components/CounterWithHook';

describe('CounterWithHook Component', () => {
  it('works with default values', () => {
    cy.mount(<CounterWithHook />);
    
    cy.get('[data-cy=count]').should('have.text', 'Count: 0');
    
    cy.get('[data-cy=increment]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 1');
    
    cy.get('[data-cy=decrement]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 0');
  });

  it('works with custom initial value', () => {
    cy.mount(<CounterWithHook initialValue={5} />);
    
    cy.get('[data-cy=count]').should('have.text', 'Count: 5');
    
    cy.get('[data-cy=increment]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 6');
  });

  it('works with custom step value', () => {
    cy.mount(<CounterWithHook step={5} />);
    
    cy.get('[data-cy=count]').should('have.text', 'Count: 0');
    
    cy.get('[data-cy=increment]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 5');
    
    cy.get('[data-cy=decrement]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 0');
  });

  it('resets to initial value', () => {
    cy.mount(<CounterWithHook initialValue={10} />);
    
    cy.get('[data-cy=increment]').click();
    cy.get('[data-cy=increment]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 12');
    
    cy.get('[data-cy=reset]').click();
    cy.get('[data-cy=count]').should('have.text', 'Count: 10');
  });
});
```

## Работа с событиями и формами

```typescript
// LoginForm.tsx
import React, { useState } from 'react';

interface LoginFormProps {
  onSubmit: (credentials: { username: string; password: string }) => void;
}

const LoginForm: React.FC<LoginFormProps> = ({ onSubmit }) => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ username, password });
  };

  return (
    <form onSubmit={handleSubmit} data-cy="login-form">
      <div>
        <label htmlFor="username">Username:</label>
        <input
          id="username"
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          data-cy="username-input"
        />
      </div>
      <div>
        <label htmlFor="password">Password:</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          data-cy="password-input"
        />
      </div>
      <button type="submit" data-cy="submit-btn">Login</button>
    </form>
  );
};

export default LoginForm;
```

```typescript
// cypress/component/LoginForm.cy.tsx
import React from 'react';
import LoginForm from '../../src/components/LoginForm';

describe('LoginForm Component', () => {
  it('submits form with correct credentials', () => {
    const onSubmitSpy = cy.spy().as('onSubmitSpy');
    cy.mount(<LoginForm onSubmit={onSubmitSpy} />);
    
    cy.get('[data-cy=username-input]').type('testuser');
    cy.get('[data-cy=password-input]').type('password123');
    cy.get('[data-cy=submit-btn]').click();
    
    cy.get('@onSubmitSpy').should('have.been.calledOnceWith', {
      username: 'testuser',
      password: 'password123'
    });
  });

  it('allows typing in username field', () => {
    cy.mount(<LoginForm onSubmit={cy.stub()} />);
    
    cy.get('[data-cy=username-input]').type('testuser');
    cy.get('[data-cy=username-input]').should('have.value', 'testuser');
  });

  it('allows typing in password field', () => {
    cy.mount(<LoginForm onSubmit={cy.stub()} />);
    
    cy.get('[data-cy=password-input]').type('password123');
    cy.get('[data-cy=password-input]').should('have.value', 'password123');
  });

  it('prevents default form submission', () => {
    const preventDefaultSpy = cy.spy().as('preventDefaultSpy');
    cy.mount(<LoginForm onSubmit={cy.stub()} />);
    
    cy.get('form').then($form => {
      $form.on('submit', preventDefaultSpy);
    });
    
    cy.get('[data-cy=submit-btn]').click();
    cy.get('@preventDefaultSpy').should('have.been.calledOnce');
  });
});
```

## Визуальное тестирование и скриншоты

Cypress также поддерживает визуальное тестирование компонентов:

```typescript
// cypress/component/VisualTests.cy.tsx
import React from 'react';
import Button from '../../src/components/Button';

describe('Visual Tests', () => {
  it('matches snapshot of primary button', () => {
    cy.mount(<Button variant="primary">Primary Button</Button>);
    cy.get('button').matchImageSnapshot('primary-button');
  });

  it('matches snapshot of secondary button', () => {
    cy.mount(<Button variant="secondary">Secondary Button</Button>);
    cy.get('button').matchImageSnapshot('secondary-button');
  });

  it('matches snapshot of disabled button', () => {
    cy.mount(<Button disabled>Disabled Button</Button>);
    cy.get('button').matchImageSnapshot('disabled-button');
  });
});
```

## Заключение

Cypress Component Testing предоставляет мощную альтернативу традиционным инструментам тестирования компонентов, обеспечивая реалистичную среду браузера для тестирования. Его возможности по отладке, мокированию и визуальному тестированию делают его отличным выбором для комплексного тестирования компонентов.

См. также:
- [[Testing-Library]]
- [[Jest-с-компонентами]]
- [[Snapshot-тестирование]]
- [[E2E-тестирование-компонентов]]