---
aliases: [E2E тестирование компонентов, Сквозное тестирование, Интеграционное тестирование компонентов]
tags: [testing, javascript, typescript, frontend, react, cypress, playwright]
---

# E2E тестирование компонентов - комплексная проверка функциональности

E2E (End-to-End) тестирование компонентов - это подход к тестированию, при котором компоненты тестируются в комплексе, как часть более крупной системы, с эмуляцией реальных пользовательских сценариев. Это позволяет проверить не только отдельные компоненты, но и их взаимодействие друг с другом и с внешними сервисами.

## Обзор E2E тестирования компонентов

E2E тестирование компонентов отличается от традиционного E2E тестирования тем, что фокусируется на тестировании взаимодействия между компонентами в изолированной среде, а не на полной эмуляции пользовательского пути через всё приложение. Это позволяет находить ошибки интеграции на более ранней стадии разработки.

### Преимущества E2E тестирования компонентов

- **Проверка взаимодействия компонентов**: Тестирование того, как компоненты работают вместе
- **Реалистичная среда**: Тестирование в условиях, близких к реальным
- **Выявление проблем интеграции**: Обнаружение ошибок, возникающих при взаимодействии нескольких компонентов
- **Проверка пользовательских сценариев**: Тестирование реальных пользовательских путей в рамках компонентов

### Инструменты для E2E тестирования компонентов

- **Cypress Component Testing**: Тестирование компонентов в реальном браузере
- **Playwright Component Testing**: Альтернатива Cypress с поддержкой нескольких браузеров
- **Testing Library с Puppeteer**: Комбинация Testing Library и headless браузера

## Основы E2E тестирования компонентов

### Настройка Cypress для E2E тестирования компонентов

```bash
npm install --save-dev cypress @cypress/react
```

Файл конфигурации `cypress.config.ts`:

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  component: {
    devServer: {
      framework: 'react',
      bundler: 'vite', // или 'webpack'
    },
  },
  
  e2e: {
    setupNodeEvents(on, config) {
      // настройка событий для E2E тестов
    },
    baseUrl: 'http://localhost:3000',
  },
});
```

## Практические примеры E2E тестирования компонентов

### Простой сценарий взаимодействия компонентов

```typescript
// UserForm.tsx
import React, { useState } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserFormProps {
  onSubmit: (user: Omit<User, 'id'>) => void;
  onCancel: () => void;
}

const UserForm: React.FC<UserFormProps> = ({ onSubmit, onCancel }) => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit({ name, email });
  };

  return (
    <form onSubmit={handleSubmit} data-cy="user-form">
      <div>
        <label htmlFor="name">Name:</label>
        <input
          id="name"
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          data-cy="name-input"
        />
      </div>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          data-cy="email-input"
        />
      </div>
      <button type="submit" data-cy="submit-btn">Submit</button>
      <button type="button" onClick={onCancel} data-cy="cancel-btn">Cancel</button>
    </form>
  );
};

export default UserForm;
```

```typescript
// UserList.tsx
import React from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserListProps {
  users: User[];
  onEdit: (user: User) => void;
  onDelete: (id: string) => void;
}

const UserList: React.FC<UserListProps> = ({ users, onEdit, onDelete }) => {
  return (
    <div data-cy="user-list">
      {users.map((user) => (
        <div key={user.id} className="user-item" data-cy={`user-item-${user.id}`}>
          <span>{user.name} ({user.email})</span>
          <button onClick={() => onEdit(user)} data-cy={`edit-btn-${user.id}`}>Edit</button>
          <button onClick={() => onDelete(user.id)} data-cy={`delete-btn-${user.id}`}>Delete</button>
        </div>
      ))}
    </div>
  );
};

export default UserList;
```

```typescript
// UserManagement.tsx
import React, { useState } from 'react';
import UserForm from './UserForm';
import UserList from './UserList';

interface User {
  id: string;
  name: string;
  email: string;
}

const UserManagement: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [editingUser, setEditingUser] = useState<User | null>(null);
  const [showForm, setShowForm] = useState(false);

  const handleSubmit = (userData: Omit<User, 'id'>) => {
    if (editingUser) {
      // Редактирование существующего пользователя
      setUsers(users.map(user => 
        user.id === editingUser.id ? { ...userData, id: editingUser.id } : user
      ));
    } else {
      // Добавление нового пользователя
      const newUser: User = {
        ...userData,
        id: Date.now().toString(),
      };
      setUsers([...users, newUser]);
    }
    resetForm();
  };

  const handleEdit = (user: User) => {
    setEditingUser(user);
    setShowForm(true);
  };

  const handleDelete = (id: string) => {
    setUsers(users.filter(user => user.id !== id));
  };

  const resetForm = () => {
    setEditingUser(null);
    setShowForm(false);
  };

  return (
    <div className="user-management">
      <h2>User Management</h2>
      
      {!showForm ? (
        <button 
          onClick={() => setShowForm(true)} 
          data-cy="add-user-btn"
        >
          Add User
        </button>
      ) : (
        <UserForm 
          onSubmit={handleSubmit} 
          onCancel={resetForm}
        />
      )}
      
      <UserList 
        users={users} 
        onEdit={handleEdit} 
        onDelete={handleDelete} 
      />
    </div>
  );
};

export default UserManagement;
```

```typescript
// cypress/component/UserManagement.cy.tsx
import React from 'react';
import UserManagement from '../../src/components/UserManagement';

describe('User Management Component E2E', () => {
  it('allows adding a new user and displays it in the list', () => {
    cy.mount(<UserManagement />);
    
    // Проверяем, что список пользователей пуст
    cy.get('[data-cy=user-list]').should('be.empty');
    
    // Нажимаем кнопку "Add User"
    cy.get('[data-cy=add-user-btn]').click();
    
    // Заполняем форму
    cy.get('[data-cy=name-input]').type('John Doe');
    cy.get('[data-cy=email-input]').type('john@example.com');
    
    // Отправляем форму
    cy.get('[data-cy=submit-btn]').click();
    
    // Проверяем, что пользователь появился в списке
    cy.get('[data-cy=user-item-*]').should('have.length', 1);
    cy.get('[data-cy=user-item-*]').should('contain.text', 'John Doe');
    cy.get('[data-cy=user-item-*]').should('contain.text', 'john@example.com');
  });

  it('allows editing an existing user', () => {
    cy.mount(<UserManagement />);
    
    // Добавляем первого пользователя
    cy.get('[data-cy=add-user-btn]').click();
    cy.get('[data-cy=name-input]').type('John Doe');
    cy.get('[data-cy=email-input]').type('john@example.com');
    cy.get('[data-cy=submit-btn]').click();
    
    // Редактируем пользователя
    cy.get('[data-cy=edit-btn-*]').click();
    
    // Проверяем, что форма заполнена текущими данными
    cy.get('[data-cy=name-input]').should('have.value', 'John Doe');
    cy.get('[data-cy=email-input]').should('have.value', 'john@example.com');
    
    // Изменяем данные
    cy.get('[data-cy=name-input]').clear().type('Jane Doe');
    cy.get('[data-cy=email-input]').clear().type('jane@example.com');
    
    // Сохраняем изменения
    cy.get('[data-cy=submit-btn]').click();
    
    // Проверяем, что изменения отображаются в списке
    cy.get('[data-cy=user-item-*]').should('contain.text', 'Jane Doe');
    cy.get('[data-cy=user-item-*]').should('contain.text', 'jane@example.com');
  });

  it('allows deleting a user', () => {
    cy.mount(<UserManagement />);
    
    // Добавляем пользователя
    cy.get('[data-cy=add-user-btn]').click();
    cy.get('[data-cy=name-input]').type('John Doe');
    cy.get('[data-cy=email-input]').type('john@example.com');
    cy.get('[data-cy=submit-btn]').click();
    
    // Проверяем, что пользователь добавлен
    cy.get('[data-cy=user-item-*]').should('have.length', 1);
    
    // Удаляем пользователя
    cy.get('[data-cy=delete-btn-*]').click();
    
    // Проверяем, что пользователь удален
    cy.get('[data-cy=user-list]').should('be.empty');
  });

  it('cancels form properly', () => {
    cy.mount(<UserManagement />);
    
    // Открываем форму
    cy.get('[data-cy=add-user-btn]').click();
    
    // Заполняем поля
    cy.get('[data-cy=name-input]').type('Test User');
    cy.get('[data-cy=email-input]').type('test@example.com');
    
    // Нажимаем отмену
    cy.get('[data-cy=cancel-btn]').click();
    
    // Проверяем, что форма закрыта и пользователь не добавлен
    cy.get('[data-cy=name-input]').should('not.exist');
    cy.get('[data-cy=user-list]').should('be.empty');
  });
});
```

## Тестирование с асинхронными операциями

```typescript
// AsyncUserManagement.tsx
import React, { useState, useEffect } from 'react';
import UserForm from './UserForm';
import UserList from './UserList';

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserService {
  getUsers: () => Promise<User[]>;
  addUser: (user: Omit<User, 'id'>) => Promise<User>;
  updateUser: (id: string, user: Omit<User, 'id'>) => Promise<User>;
  deleteUser: (id: string) => Promise<void>;
}

interface AsyncUserManagementProps {
  userService: UserService;
}

const AsyncUserManagement: React.FC<AsyncUserManagementProps> = ({ userService }) => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [editingUser, setEditingUser] = useState<User | null>(null);
  const [showForm, setShowForm] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    loadUsers();
  }, []);

  const loadUsers = async () => {
    try {
      setLoading(true);
      const userData = await userService.getUsers();
      setUsers(userData);
    } catch (err) {
      setError((err as Error).message);
    } finally {
      setLoading(false);
    }
  };

  const handleSubmit = async (userData: Omit<User, 'id'>) => {
    try {
      if (editingUser) {
        const updatedUser = await userService.updateUser(editingUser.id, userData);
        setUsers(users.map(user => 
          user.id === editingUser.id ? updatedUser : user
        ));
      } else {
        const newUser = await userService.addUser(userData);
        setUsers([...users, newUser]);
      }
      resetForm();
    } catch (err) {
      setError((err as Error).message);
    }
  };

  const handleEdit = (user: User) => {
    setEditingUser(user);
    setShowForm(true);
  };

  const handleDelete = async (id: string) => {
    try {
      await userService.deleteUser(id);
      setUsers(users.filter(user => user.id !== id));
    } catch (err) {
      setError((err as Error).message);
    }
  };

  const resetForm = () => {
    setEditingUser(null);
    setShowForm(false);
  };

  if (loading) return <div data-cy="loading">Loading...</div>;
  if (error) return <div data-cy="error">Error: {error}</div>;

  return (
    <div className="async-user-management">
      <h2>Async User Management</h2>
      
      {!showForm ? (
        <button 
          onClick={() => setShowForm(true)} 
          data-cy="add-user-btn"
        >
          Add User
        </button>
      ) : (
        <UserForm 
          onSubmit={handleSubmit} 
          onCancel={resetForm}
        />
      )}
      
      <UserList 
        users={users} 
        onEdit={handleEdit} 
        onDelete={handleDelete} 
      />
    </div>
  );
};

export default AsyncUserManagement;
```

```typescript
// cypress/component/AsyncUserManagement.cy.tsx
import React from 'react';
import AsyncUserManagement from '../../src/components/AsyncUserManagement';

describe('Async User Management Component E2E', () => {
  const mockUserService = {
    getUsers: cy.stub().resolves([
      { id: '1', name: 'John Doe', email: 'john@example.com' },
      { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
    ]),
    addUser: cy.stub().callsFake(async (userData) => ({
      id: Date.now().toString(),
      ...userData
    })),
    updateUser: cy.stub().callsFake(async (id, userData) => ({
      id,
      ...userData
    })),
    deleteUser: cy.stub().resolves()
  };

  beforeEach(() => {
    cy.stub(mockUserService, 'getUsers').resolves([
      { id: '1', name: 'John Doe', email: 'john@example.com' },
      { id: '2', name: 'Jane Smith', email: 'jane@example.com' }
    ]);
  });

  it('loads users on initial render', () => {
    cy.mount(<AsyncUserManagement userService={mockUserService} />);
    
    // Проверяем индикатор загрузки
    cy.get('[data-cy=loading]').should('be.visible');
    
    // После загрузки проверяем список пользователей
    cy.get('[data-cy=user-item-1]').should('contain.text', 'John Doe');
    cy.get('[data-cy=user-item-2]').should('contain.text', 'Jane Smith');
  });

  it('adds a new user via async service', () => {
    cy.mount(<AsyncUserManagement userService={mockUserService} />);
    
    // Ждем загрузки
    cy.get('[data-cy=user-item-1]').should('be.visible');
    
    // Добавляем нового пользователя
    cy.get('[data-cy=add-user-btn]').click();
    cy.get('[data-cy=name-input]').type('New User');
    cy.get('[data-cy=email-input]').type('newuser@example.com');
    cy.get('[data-cy=submit-btn]').click();
    
    // Проверяем, что вызван метод добавления
    cy.wrap(mockUserService.addUser).should('have.been.calledOnce');
    
    // Проверяем, что новый пользователь появился в списке
    cy.get('[data-cy=user-item-*]').should('have.length', 3);
  });

  it('handles service errors gracefully', () => {
    const errorUserService = {
      ...mockUserService,
      getUsers: cy.stub().rejects(new Error('Network error'))
    };
    
    cy.mount(<AsyncUserManagement userService={errorUserService} />);
    
    // Проверяем, что отображается ошибка
    cy.get('[data-cy=error]').should('contain.text', 'Network error');
  });
});
```

## Тестирование сложных пользовательских сценариев

```typescript
// ShoppingCheckout.tsx
import React, { useState } from 'react';

interface Product {
  id: string;
  name: string;
  price: number;
}

interface CartItem {
  product: Product;
  quantity: number;
}

interface ShoppingCartProps {
  products: Product[];
  onCheckout: (items: CartItem[], total: number) => void;
}

const ShoppingCart: React.FC<ShoppingCartProps> = ({ products, onCheckout }) => {
  const [cart, setCart] = useState<CartItem[]>([]);
  const [selectedProduct, setSelectedProduct] = useState<string>('');
  const [quantity, setQuantity] = useState<number>(1);

  const addToCart = () => {
    if (!selectedProduct) return;
    
    const product = products.find(p => p.id === selectedProduct);
    if (!product) return;
    
    setCart(prevCart => {
      const existingItem = prevCart.find(item => item.product.id === selectedProduct);
      if (existingItem) {
        return prevCart.map(item => 
          item.product.id === selectedProduct 
            ? { ...item, quantity: item.quantity + quantity } 
            : item
        );
      } else {
        return [...prevCart, { product, quantity }];
      }
    });
    
    setQuantity(1);
  };

  const removeFromCart = (productId: string) => {
    setCart(prevCart => prevCart.filter(item => item.product.id !== productId));
  };

  const getTotal = () => {
    return cart.reduce((total, item) => total + (item.product.price * item.quantity), 0);
  };

  const handleCheckout = () => {
    onCheckout(cart, getTotal());
  };

  return (
    <div className="shopping-cart">
      <h2>Shopping Cart</h2>
      
      <div className="add-to-cart">
        <select 
          value={selectedProduct} 
          onChange={(e) => setSelectedProduct(e.target.value)}
          data-cy="product-select"
        >
          <option value="">Select a product</option>
          {products.map(product => (
            <option key={product.id} value={product.id}>
              {product.name} - ${product.price.toFixed(2)}
            </option>
          ))}
        </select>
        
        <input
          type="number"
          min="1"
          value={quantity}
          onChange={(e) => setQuantity(parseInt(e.target.value))}
          data-cy="quantity-input"
        />
        
        <button onClick={addToCart} data-cy="add-to-cart-btn">
          Add to Cart
        </button>
      </div>
      
      <div className="cart-items" data-cy="cart-items">
        {cart.map(item => (
          <div key={item.product.id} className="cart-item" data-cy={`cart-item-${item.product.id}`}>
            <span>{item.product.name} x {item.quantity} = ${(item.product.price * item.quantity).toFixed(2)}</span>
            <button 
              onClick={() => removeFromCart(item.product.id)} 
              data-cy={`remove-btn-${item.product.id}`}
            >
              Remove
            </button>
          </div>
        ))}
      </div>
      
      <div className="cart-total" data-cy="cart-total">
        Total: ${getTotal().toFixed(2)}
      </div>
      
      <button 
        onClick={handleCheckout} 
        disabled={cart.length === 0}
        data-cy="checkout-btn"
      >
        Checkout
      </button>
    </div>
  );
};

export default ShoppingCart;
```

```typescript
// cypress/component/ShoppingCart.cy.tsx
import React from 'react';
import ShoppingCart from '../../src/components/ShoppingCart';

describe('Shopping Cart Component E2E', () => {
  const products = [
    { id: '1', name: 'Laptop', price: 999.99 },
    { id: '2', name: 'Mouse', price: 29.99 },
    { id: '3', name: 'Keyboard', price: 79.99 }
  ];

  it('allows adding products to cart and calculating total', () => {
    const onCheckoutSpy = cy.spy().as('onCheckoutSpy');
    cy.mount(<ShoppingCart products={products} onCheckout={onCheckoutSpy} />);
    
    // Выбираем продукт и добавляем в корзину
    cy.get('[data-cy=product-select]').select('1'); // Laptop
    cy.get('[data-cy=quantity-input]').clear().type('2');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    // Проверяем, что продукт добавлен
    cy.get('[data-cy=cart-item-1]').should('contain.text', 'Laptop x 2 = $1999.98');
    
    // Проверяем общую сумму
    cy.get('[data-cy=cart-total]').should('contain.text', 'Total: $1999.98');
    
    // Добавляем еще один продукт
    cy.get('[data-cy=product-select]').select('2'); // Mouse
    cy.get('[data-cy=quantity-input]').clear().type('3');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    // Проверяем обновленную корзину
    cy.get('[data-cy=cart-items]').should('have.length.greaterThan', 1);
    cy.get('[data-cy=cart-total]').should('contain.text', 'Total: $2089.95'); // 1999.98 + (29.99 * 3)
  });

  it('allows removing items from cart', () => {
    const onCheckoutSpy = cy.spy().as('onCheckoutSpy');
    cy.mount(<ShoppingCart products={products} onCheckout={onCheckoutSpy} />);
    
    // Добавляем продукт
    cy.get('[data-cy=product-select]').select('1');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    // Проверяем, что продукт добавлен
    cy.get('[data-cy=cart-item-1]').should('exist');
    cy.get('[data-cy=cart-total]').should('contain.text', 'Total: $999.99');
    
    // Удаляем продукт
    cy.get('[data-cy=remove-btn-1]').click();
    
    // Проверяем, что продукт удален и сумма обновлена
    cy.get('[data-cy=cart-item-1]').should('not.exist');
    cy.get('[data-cy=cart-total]').should('contain.text', 'Total: $0.00');
  });

  it('handles checkout process', () => {
    const onCheckoutSpy = cy.spy().as('onCheckoutSpy');
    cy.mount(<ShoppingCart products={products} onCheckout={onCheckoutSpy} />);
    
    // Добавляем несколько продуктов
    cy.get('[data-cy=product-select]').select('1'); // Laptop
    cy.get('[data-cy=quantity-input]').clear().type('1');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    cy.get('[data-cy=product-select]').select('3'); // Keyboard
    cy.get('[data-cy=quantity-input]').clear().type('2');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    // Проверяем, что кнопка оформления заказа активна
    cy.get('[data-cy=checkout-btn]').should('not.be.disabled');
    
    // Оформляем заказ
    cy.get('[data-cy=checkout-btn]').click();
    
    // Проверяем, что функция оформления заказа была вызвана с правильными параметрами
    cy.get('@onCheckoutSpy').should('have.been.calledOnce');
    
    // Проверяем аргументы вызова
    cy.get('@onCheckoutSpy').then((spy) => {
      const args = spy.firstCall.args;
      const items = args[0];
      const total = args[1];
      
      expect(items).to.have.length(2);
      expect(total).to.equal(1159.97); // 999.99 + (79.99 * 2)
    });
  });

  it('disables checkout button when cart is empty', () => {
    const onCheckoutSpy = cy.spy().as('onCheckoutSpy');
    cy.mount(<ShoppingCart products={products} onCheckout={onCheckoutSpy} />);
    
    // Проверяем, что кнопка оформления заказа отключена
    cy.get('[data-cy=checkout-btn]').should('be.disabled');
    
    // Добавляем продукт
    cy.get('[data-cy=product-select]').select('1');
    cy.get('[data-cy=add-to-cart-btn]').click();
    
    // Проверяем, что кнопка теперь активна
    cy.get('[data-cy=checkout-btn]').should('not.be.disabled');
  });
});
```

## Использование Playwright для E2E тестирования компонентов

```typescript
// playwright/component/ShoppingCart.spec.ts
import { test, expect } from '@playwright/test';
import { mount } from 'playwright-ct-react';
import ShoppingCart from '../../src/components/ShoppingCart';

const products = [
  { id: '1', name: 'Laptop', price: 999.99 },
  { id: '2', name: 'Mouse', price: 29.99 },
];

test('allows adding products to cart', async ({ page }) => {
  const onCheckoutSpy = () => {};
  await mount(<ShoppingCart products={products} onCheckout={onCheckoutSpy} />);
  
  // Выбираем продукт
  await page.locator('select[data-cy="product-select"]').selectOption('1');
  
  // Устанавливаем количество
  await page.locator('input[data-cy="quantity-input"]').fill('2');
  
  // Добавляем в корзину
  await page.locator('button[data-cy="add-to-cart-btn"]').click();
  
  // Проверяем результат
  await expect(page.locator('[data-cy="cart-item-1"]')).toContainText('Laptop x 2 = $1999.98');
});
```

## Лучшие практики E2E тестирования компонентов

### 1. Тестирование пользовательских сценариев, а не отдельных функций

```typescript
// Плохо: тестирование только отдельных действий
test('should add item to cart', () => {
  // Только проверка добавления
});

// Хорошо: тестирование полного пользовательского сценария
test('should allow user to add items to cart and checkout', () => {
  // Полный сценарий: добавление, проверка, оформление заказа
});
```

### 2. Использование описательных имен тестов

```typescript
test('should update cart total when adding multiple quantities of the same product', () => {
  // Тест с понятным описанием сценария
});
```

### 3. Избегайте хрупких селекторов

```typescript
// Плохо: использование CSS классов, которые могут измениться
cy.get('.button-primary').click();

// Хорошо: использование data-атрибутов
cy.get('[data-cy=submit-btn]').click();
```

### 4. Тестирование различных состояний компонентов

```typescript
test('should handle empty cart state', () => {
  // Тестирование состояния, когда корзина пуста
});

test('should handle cart with multiple items', () => {
  // Тестирование состояния с несколькими элементами
});

test('should handle error state', () => {
  // Тестирование состояния ошибки
});
```

## Заключение

E2E тестирование компонентов позволяет проверить взаимодействие между различными компонентами в реалистичной среде, что помогает выявить ошибки интеграции и обеспечить надежность пользовательских сценариев. При правильном подходе E2E тесты компонентов становятся важной частью стратегии тестирования, дополняя юнит-тесты и тесты компонентов.

См. также:
- [[Testing-Library]]
- [[Jest-с-компонентами]]
- [[Cypress-для-компонентов]]
- [[Snapshot-тестирование]]