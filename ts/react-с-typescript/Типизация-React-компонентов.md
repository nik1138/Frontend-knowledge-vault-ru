---
aliases: [React Components Typing, Типизированные компоненты React]
tags: [typescript, react, components, typing]
---

# Типизация React-компонентов

## Обзор

Типизация React-компонентов с использованием TypeScript позволяет разработчикам создавать более надежные и поддерживаемые приложения. Это помогает предотвратить ошибки на этапе разработки, обеспечивает автодополнение в редакторах кода и делает код более понятным для других разработчиков.

## Основы типизации компонентов

### Функциональные компоненты

В React с TypeScript функциональные компоненты могут быть типизированы несколькими способами:

#### 1. Простой функциональный компонент

```typescript
import React from 'react';

const Welcome: React.FC = () => {
  return <h1>Добро пожаловать!</h1>;
};

export default Welcome;
```

#### 2. Функциональный компонент с пропсами

```typescript
import React from 'react';

interface UserProps {
  name: string;
  age: number;
  email?: string;
}

const UserProfile: React.FC<UserProps> = ({ name, age, email }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Возраст: {age}</p>
      {email && <p>Email: {email}</p>}
    </div>
  );
};

export default UserProfile;
```

#### 3. Современный подход без React.FC

Согласно современным практикам, предпочтительно использовать явные типы возвращаемого значения:

```typescript
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
}

const Button = ({ children, onClick, disabled = false, variant = 'primary' }: ButtonProps): JSX.Element => {
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  );
};

export default Button;
```

### Компоненты класса

```typescript
import React, { Component } from 'react';

interface CounterState {
  count: number;
}

interface CounterProps {
  initialValue?: number;
  step?: number;
}

class Counter extends Component<CounterProps, CounterState> {
  constructor(props: CounterProps) {
    super(props);
    this.state = {
      count: props.initialValue || 0,
    };
  }

  increment = () => {
    const { step = 1 } = this.props;
    this.setState(prevState => ({
      count: prevState.count + step
    }));
  };

  render() {
    return (
      <div>
        <p>Текущее значение: {this.state.count}</p>
        <button onClick={this.increment}>Увеличить</button>
      </div>
    );
  }
}

export default Counter;
```

## Типизация пропсов

### Обязательные и необязательные пропсы

```typescript
interface ProductCardProps {
  // Обязательные пропсы
  title: string;
  price: number;
  
  // Необязательные пропсы
  description?: string;
  discount?: number;
  inStock?: boolean;
}

const ProductCard: React.FC<ProductCardProps> = ({ 
  title, 
  price, 
  description, 
  discount = 0, 
  inStock = true 
}) => {
  return (
    <div className="product-card">
      <h3>{title}</h3>
      {description && <p>{description}</p>}
      <div className="price-info">
        {discount > 0 ? (
          <>
            <span className="original-price">${price}</span>
            <span className="discounted-price">${price * (1 - discount)}</span>
          </>
        ) : (
          <span>${price}</span>
        )}
      </div>
      <div className={`stock-status ${inStock ? 'in-stock' : 'out-of-stock'}`}>
        {inStock ? 'В наличии' : 'Нет в наличии'}
      </div>
    </div>
  );
};
```

### Пропсы с ограниченными значениями (union types)

```typescript
interface AlertProps {
  message: string;
  type: 'success' | 'error' | 'warning' | 'info';
  closable?: boolean;
}

const Alert: React.FC<AlertProps> = ({ message, type, closable = false }) => {
  return (
    <div className={`alert alert-${type}`}>
      {message}
      {closable && <button className="close-btn">X</button>}
    </div>
  );
};
```

## Работа с детьми (children)

### React.ReactNode

```typescript
interface CardProps {
  children: React.ReactNode;
  title?: string;
  className?: string;
}

const Card: React.FC<CardProps> = ({ children, title, className = '' }) => {
  return (
    <div className={`card ${className}`}>
      {title && <h2 className="card-title">{title}</h2>}
      <div className="card-content">
        {children}
      </div>
    </div>
  );
};
```

### ReactElement vs ReactNode

```typescript
interface ModalProps {
  isOpen: boolean;
  // ReactNode включает в себя строки, числа, ReactElement и их массивы
  children: React.ReactNode;
  onClose: () => void;
}

const Modal: React.FC<ModalProps> = ({ isOpen, children, onClose }) => {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button onClick={onClose} className="close-button">X</button>
        {children}
      </div>
    </div>
  );
};
```

## Типизация событий

```typescript
interface FormProps {
  onSubmit: (data: { email: string; password: string }) => void;
}

const LoginForm: React.FC<FormProps> = ({ onSubmit }) => {
  const [formData, setFormData] = React.useState({
    email: '',
    password: ''
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Пароль"
      />
      <button type="submit">Войти</button>
    </form>
  );
};
```

## Дженерики в компонентах

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const UserList: React.FC<{ users: User[] }> = ({ users }) => {
  return (
    <List
      items={users}
      keyExtractor={(user) => user.id}
      renderItem={(user) => (
        <div>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      )}
    />
  );
};
```

## Условная типизация и полиморфизм

```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary';
  size: 'small' | 'medium' | 'large';
  children: React.ReactNode;
  as?: React.ElementType; // Позволяет использовать компонент как другой HTML элемент
}

const PolymorphicButton = React.forwardRef<
  HTMLButtonElement | HTMLAnchorElement,
  ButtonProps & React.ComponentPropsWithoutRef<'button' | 'a'>
>(({ variant, size, children, as: Component = 'button', ...props }, ref) => {
  return (
    <Component
      ref={ref as React.Ref<any>}
      className={`btn btn-${variant} btn-${size}`}
      {...props}
    >
      {children}
    </Component>
  );
});

PolymorphicButton.displayName = 'Button';
```

## Лучшие практики

1. **Используйте интерфейсы для определения пропсов**:
   ```typescript
   interface ComponentProps {
     // определение пропсов
   }
   ```

2. **Избегайте чрезмерного использования React.FC**:
   Вместо `React.FC<Props>` лучше использовать явное указание типа возвращаемого значения.

3. **Используйте деструктуризацию пропсов**:
   ```typescript
   const Component = ({ prop1, prop2 }: ComponentProps) => {
     // код компонента
   };
   ```

4. **Типизируйте все возможные значения**:
   Используйте union types для ограниченного набора значений.

5. **Используйте Partial и Omit для производных типов**:
   ```typescript
   interface FullUser {
     id: number;
     name: string;
     email: string;
     password: string;
   }

   type UserRegistration = Omit<FullUser, 'id'>; // Все поля кроме id
   type UserProfile = Pick<FullUser, 'id' | 'name' | 'email'>; // Только указанные поля
   ```

## Связанные темы

- [[Типизация-React-хуков]]
- [[Типизация-React-контекста]]
- [[HOC-с-типами]]
- [[Рендер-пропсы-с-типами]]
- [[React-паттерны-типов]]

## Заключение

Типизация React-компонентов с использованием TypeScript значительно повышает качество кода и уменьшает количество ошибок. Понимание основных паттернов типизации позволяет создавать более надежные и поддерживаемые приложения.