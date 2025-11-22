---
aliases: [Не повторяйся, Принцип DRY, Избегание дублирования кода]
tags: [programming, principles, code-quality, best-practices]
---

# DRY - Don't Repeat Yourself

DRY (Don't Repeat Yourself) — это принцип программирования, призывающий избегать дублирования кода. Всякая логика должна быть представлена в системе только один раз. Если у вас есть дублирующийся код, это может привести к ошибкам и затруднить поддержку.

## Суть принципа

> Каждая часть знаний должна иметь единственное, несомненное, авторитетное представление в системе.

Это означает, что любая логика или данные в приложении должны быть определены только один раз, чтобы избежать несоответствий и облегчить поддержку.

## Примеры дублирования кода

### Плохой пример:

```javascript
// Много повторяющегося кода
function calculateTotalPrice(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  return total;
}

function calculateTotalWeight(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].weight * items[i].quantity;
  }
  return total;
}

function calculateTotalTax(items) {
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    total += items[i].tax * items[i].quantity;
  }
  return total;
}
```

### Хороший пример:

```javascript
// Используем обобщенную функцию
function calculateTotal(items, property) {
  return items.reduce((total, item) => total + item[property] * item.quantity, 0);
}

// Теперь можно использовать для разных свойств
const totalPrice = calculateTotal(items, 'price');
const totalWeight = calculateTotal(items, 'weight');
const totalTax = calculateTotal(items, 'tax');
```

## Практические способы применения DRY

### 1. Создание общих функций

```javascript
// Вместо:
function formatUser(user) {
  return `${user.firstName} ${user.lastName}`;
}

function formatCustomer(customer) {
  return `${customer.firstName} ${customer.lastName}`;
}

// Используем общую функцию:
function formatPerson(person) {
  return `${person.firstName} ${person.lastName}`;
}

// Применение:
const userName = formatPerson(user);
const customerName = formatPerson(customer);
```

### 2. Использование конфигурационных объектов

```javascript
// Вместо:
const apiUrl = 'https://api.example.com/v1/users';
const userHeaders = { 'Content-Type': 'application/json', 'Authorization': 'Bearer token' };

const productUrl = 'https://api.example.com/v1/products';
const productHeaders = { 'Content-Type': 'application/json', 'Authorization': 'Bearer token' };

// Используем конфигурацию:
const config = {
  baseUrl: 'https://api.example.com/v1',
  headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer token' }
};

const apiUrl = `${config.baseUrl}/users`;
const productUrl = `${config.baseUrl}/products`;
```

### 3. Создание общих компонентов в React

```jsx
// Вместо дублирования:
function UserProfile({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function CustomerProfile({ customer }) {
  return (
    <div className="customer-card">
      <img src={customer.avatar} alt={customer.name} />
      <h2>{customer.name}</h2>
      <p>{customer.email}</p>
    </div>
  );
}

// Общий компонент:
function ProfileCard({ person, className }) {
  return (
    <div className={`profile-card ${className}`}>
      <img src={person.avatar} alt={person.name} />
      <h2>{person.name}</h2>
      <p>{person.email}</p>
    </div>
  );
}

// Использование:
<UserProfile person={user} className="user" />
<CustomerProfile person={customer} className="customer" />
```

### 4. Использование шаблонов и наследования

```javascript
// Общий класс для API сервисов
class ApiService {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
  }

  async get(endpoint) {
    const response = await fetch(`${this.baseUrl}/${endpoint}`);
    return response.json();
  }

  async post(endpoint, data) {
    const response = await fetch(`${this.baseUrl}/${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }
}

// Специфические сервисы наследуют общую логику
class UserService extends ApiService {
  constructor() {
    super('https://api.example.com/users');
  }

  async getCurrentUser() {
    return this.get('current');
  }
}

class ProductService extends ApiService {
  constructor() {
    super('https://api.example.com/products');
  }

  async getFeaturedProducts() {
    return this.get('featured');
  }
}
```

## Инструменты для обнаружения дубликатов

### ESLint правило для обнаружения дублирующихся блоков кода:

```javascript
// .eslintrc.js
module.exports = {
  plugins: ['sonarjs'],
  rules: {
    'sonarjs/no-duplicate-string': 'error',
    'sonarjs/no-identical-functions': 'error'
  }
};
```

## Потенциальные ловушки

1. **Переусердствование с DRY**: Иногда небольшое дублирование кода может быть более понятным, чем сложная абстракция
2. **Создание избыточных абстракций**: Не создавайте абстракции "на будущее", только если действительно видите дублирование
3. **Сложные обобщенные функции**: Иногда лучше иметь немного дублированного, но простого кода, чем одну сложную обобщенную функцию

## Пример из реальной жизни: CSS

### Плохой пример:

```css
/* Много повторяющихся стилей */
.header {
  font-family: Arial, sans-serif;
  font-size: 18px;
  color: #333;
  margin: 10px;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.sidebar {
  font-family: Arial, sans-serif;
  font-size: 16px;
  color: #333;
  margin: 10px;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.footer {
  font-family: Arial, sans-serif;
  font-size: 14px;
  color: #333;
  margin: 10px;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
```

### Хороший пример:

```css
/* Общие стили */
.card {
  font-family: Arial, sans-serif;
  color: #333;
  margin: 10px;
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Специфичные стили */
.header {
  font-size: 18px;
}

.sidebar {
  font-size: 16px;
}

.footer {
  font-size: 14px;
}
```

## Заключение

Принцип DRY помогает создавать более поддерживаемый и читаемый код. Однако важно применять его с умом, не создавая излишних абстракций. Цель — упростить код, а не усложнить его.

> [[SOLID]] | [[KISS]] | [[YAGNI]] | [[Композиция-вместо-наследования]]