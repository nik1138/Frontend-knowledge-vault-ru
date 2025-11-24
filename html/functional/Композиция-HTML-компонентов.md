---
aliases: ["Композиция компонентов HTML", "Компоновка HTML", "Композиционный подход"]
tags: [html, functional-programming, components, composition, frontend, best-practices]
---

# Композиция HTML-компонентов

## Обзор

Композиция HTML-компонентов - это подход к созданию пользовательского интерфейса, при котором сложные элементы создаются путем объединения более простых компонентов. В контексте функционального программирования композиция позволяет создавать гибкие, переиспользуемые и легко тестируемые компоненты, которые могут быть объединены различными способами для создания сложных интерфейсов.

В 2025 году композиция компонентов стала основополагающим принципом современной веб-разработки, особенно в экосистемах React, Vue, Svelte и других фреймворков, а также в подходах к чистому JavaScript и веб-компонентам.

## Основные принципы композиции

### 1. Принцип "одна ответственность"

Каждый компонент должен выполнять одну конкретную задачу и выполнять ее хорошо:

```javascript
// Простой компонент кнопки
function Button({ text, onClick, variant = 'primary' }) {
  return `<button class="btn btn-${variant}" onclick="${onClick}">${text}</button>`;
}

// Простой компонент заголовка
function Title({ text, level = 1 }) {
  const tag = `h${level}`;
  return `<${tag} class="title">${text}</${tag}>`;
}

// Компонент карточки, использующий другие компоненты
function Card({ title, content, actions }) {
  return `
    <div class="card">
      <header class="card-header">
        ${Title({ text: title, level: 3 })}
      </header>
      <main class="card-content">
        ${content}
      </main>
      <footer class="card-actions">
        ${actions.map(action => Button(action)).join('')}
      </footer>
    </div>
  `;
}
```

### 2. Повторное использование компонентов

Композиция позволяет использовать одни и те же компоненты в разных контекстах:

```javascript
// Компонент навигации
function Navigation({ items }) {
  return `
    <nav class="navigation">
      <ul class="nav-list">
        ${items.map(item => NavigationItem(item)).join('')}
      </ul>
    </nav>
  `;
}

function NavigationItem({ href, text, isActive = false }) {
  return `
    <li class="nav-item ${isActive ? 'active' : ''}">
      <a href="${href}" class="nav-link">${text}</a>
    </li>
  `;
}

// Компонент боковой панели, использующий тот же NavigationItem
function Sidebar({ sections }) {
  return `
    <aside class="sidebar">
      ${sections.map(section => `
        <div class="sidebar-section">
          <h3 class="section-title">${section.title}</h3>
          <ul class="section-items">
            ${section.items.map(item => NavigationItem(item)).join('')}
          </ul>
        </div>
      `).join('')}
    </aside>
  `;
}
```

### 3. Конфигурируемость через свойства

Компоненты должны быть гибкими и настраиваемыми через свойства:

```javascript
// Гибкий компонент профиля пользователя
function UserProfile({ 
  user, 
  showAvatar = true, 
  showContact = true, 
  onEdit 
}) {
  const avatar = showAvatar ? Avatar({ src: user.avatar, alt: user.name }) : '';
  const contact = showContact ? ContactInfo({ email: user.email, phone: user.phone }) : '';
  
  return `
    <div class="user-profile">
      ${avatar}
      <div class="user-info">
        <h2 class="user-name">${user.name}</h2>
        ${contact}
      </div>
      ${onEdit ? Button({ text: 'Редактировать', onClick: onEdit }) : ''}
    </div>
  `;
}

function Avatar({ src, alt }) {
  return `<img src="${src}" alt="${alt}" class="avatar">`;
}

function ContactInfo({ email, phone }) {
  return `
    <div class="contact-info">
      ${email ? `<div class="email">${email}</div>` : ''}
      ${phone ? `<div class="phone">${phone}</div>` : ''}
    </div>
  `;
}
```

## Паттерны композиции

### 1. Паттерн "Контейнер-компонент"

Разделение логики получения данных и отображения:

```javascript
// Контейнерный компонент (управляет состоянием)
class UserListContainer {
  constructor(apiService) {
    this.apiService = apiService;
    this.users = [];
  }
  
  async loadUsers() {
    this.users = await this.apiService.fetchUsers();
    this.render();
  }
  
  render() {
    // Представительный компонент (чистый)
    return UserList({ users: this.users });
  }
}

// Представительный компонент (чистая функция)
function UserList({ users }) {
  return `
    <div class="user-list">
      ${users.map(user => UserCard({ user })).join('')}
    </div>
  `;
}

function UserCard({ user }) {
  return `
    <div class="user-card">
      <h3>${user.name}</h3>
      <p>${user.email}</p>
    </div>
  `;
}
```

### 2. Паттерн "Композиция через дочерние элементы"

Передача компонентов как дочерних элементов:

```javascript
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return '';
  
  return `
    <div class="modal-overlay" onclick="${onClose}">
      <div class="modal" onclick="event.stopPropagation()">
        <button class="modal-close" onclick="${onClose}">×</button>
        <div class="modal-content">
          ${typeof children === 'function' ? children() : children}
        </div>
      </div>
    </div>
  `;
}

// Использование
function UserProfileModal({ user, isOpen, onClose }) {
  return Modal({ 
    isOpen, 
    onClose,
    children: () => `
      <div>
        <h2>Профиль пользователя</h2>
        ${UserProfile({ user, showContact: true })}
      </div>
    `
  });
}
```

### 3. Паттерн "Компонент высшего порядка"

Создание компонентов, которые оборачивают другие компоненты:

```javascript
function withLoadingState(WrappedComponent) {
  return function(props) {
    if (props.isLoading) {
      return '<div class="loading">Загрузка...</div>';
    }
    if (props.error) {
      return `<div class="error">${props.error}</div>`;
    }
    return WrappedComponent(props);
  };
}

// Использование
const UserListWithLoading = withLoadingState(UserList);
const PostListWithLoading = withLoadingState(PostList);
```

### 4. Паттерн "Провайдер данных"

Компонент, который предоставляет данные дочерним компонентам:

```javascript
function DataProvider({ data, render }) {
  return render(data);
}

// Использование
function App() {
  const appData = { 
    user: { name: 'Иван', role: 'admin' },
    settings: { theme: 'dark', language: 'ru' }
  };
  
  return DataProvider({
    data: appData,
    render: (data) => `
      <div class="app">
        ${Header({ user: data.user })}
        ${MainContent({ settings: data.settings })}
        ${Footer()}
      </div>
    `
  });
}
```

## Продвинутые техники композиции

### 1. Условная композиция

Компоновка компонентов на основе условий:

```javascript
function Layout({ user, children }) {
  const sidebar = user ? Sidebar({ user }) : '';
  const authHeader = user ? AuthHeader({ user }) : GuestHeader();
  
  return `
    <div class="layout">
      <header class="header">
        ${authHeader}
      </header>
      <div class="main-container">
        ${sidebar}
        <main class="main-content">
          ${children}
        </main>
      </div>
      <footer class="footer">
        ${Footer()}
      </footer>
    </div>
  `;
}
```

### 2. Композиция с состоянием

Создание компонентов с внутренним состоянием:

```javascript
class CollapsiblePanel {
  constructor(title, content) {
    this.title = title;
    this.content = content;
    this.isExpanded = false;
  }
  
  toggle() {
    this.isExpanded = !this.isExpanded;
    this.render();
  }
  
  render() {
    return `
      <div class="collapsible-panel">
        <div class="panel-header" onclick="collapsiblePanels[${this.id}].toggle()">
          <h3>${this.title}</h3>
          <span class="toggle-icon">${this.isExpanded ? '▼' : '▶'}</span>
        </div>
        <div class="panel-content" style="display: ${this.isExpanded ? 'block' : 'none'}">
          ${this.content}
        </div>
      </div>
    `;
  }
}
```

### 3. Композиция с событиями

Компоненты, которые могут взаимодействовать друг с другом:

```javascript
class TodoApp {
  constructor() {
    this.todos = [];
    this.filter = 'all';
  }
  
  addTodo(text) {
    this.todos = [...this.todos, {
      id: Date.now(),
      text,
      completed: false
    }];
    this.render();
  }
  
  toggleTodo(id) {
    this.todos = this.todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
    this.render();
  }
  
  render() {
    return `
      <div class="todo-app">
        ${TodoInput({ onAdd: (text) => this.addTodo(text) })}
        ${TodoList({ 
          todos: this.todos.filter(todo => {
            if (this.filter === 'active') return !todo.completed;
            if (this.filter === 'completed') return todo.completed;
            return true;
          }),
          onToggle: (id) => this.toggleTodo(id)
        })}
        ${TodoFilters({ 
          currentFilter: this.filter,
          onChange: (filter) => { this.filter = filter; this.render(); }
        })}
      </div>
    `;
  }
}

function TodoInput({ onAdd }) {
  return `
    <div class="todo-input">
      <input type="text" id="new-todo" placeholder="Добавить задачу...">
      <button onclick="todoApp.addTodo(document.getElementById('new-todo').value); document.getElementById('new-todo').value=''">
        Добавить
      </button>
    </div>
  `;
}

function TodoList({ todos, onToggle }) {
  return `
    <ul class="todo-list">
      ${todos.map(todo => `
        <li class="todo-item ${todo.completed ? 'completed' : ''}">
          <input 
            type="checkbox" 
            ${todo.completed ? 'checked' : ''} 
            onchange="todoApp.toggleTodo(${todo.id})"
          >
          <span>${todo.text}</span>
        </li>
      `).join('')}
    </ul>
  `;
}

function TodoFilters({ currentFilter, onChange }) {
  return `
    <div class="todo-filters">
      <button class="${currentFilter === 'all' ? 'active' : ''}" onclick="todoApp.filter='all'">Все</button>
      <button class="${currentFilter === 'active' ? 'active' : ''}" onclick="todoApp.filter='active'">Активные</button>
      <button class="${currentFilter === 'completed' ? 'active' : ''}" onclick="todoApp.filter='completed'">Завершенные</button>
    </div>
  `;
}
```

## Практические применения в российском IT

### В крупных компаниях

В 2025 году российские IT-гиганты, такие как Яндекс, Сбер, Тинькофф и Avito, активно используют композицию компонентов для:

1. **Создания библиотек компонентов** - унифицированные UI-библиотеки, используемые во всех продуктах компании
2. **Масштабирования команд разработки** - независимая работа разных команд над компонентами
3. **Унификации пользовательского опыта** - консистентный интерфейс во всех продуктах
4. **Быстрого прототипирования** - быстрая сборка новых интерфейсов из готовых компонентов

### В государственных проектах

Государственные цифровые платформы, такие как Госуслуги, используют композицию компонентов для:

1. **Обеспечения доступности** - стандартизированные компоненты соответствуют требованиям WCAG
2. **Упрощения сопровождения** - централизованное обновление компонентов
3. **Гарантии качества** - тщательно протестированные компоненты повторно используются в разных системах
4. **Сокращения времени вывода на рынок** - быстрая сборка новых сервисов из готовых компонентов

## Лучшие практики композиции

### 1. Иерархическая организация компонентов

Создание логической иерархии компонентов:

```
App
├── Header
│   ├── Navigation
│   └── UserMenu
├── Main
│   ├── Sidebar
│   └── Content
│       ├── ArticleList
│       │   └── ArticleCard
│       └── Pagination
└── Footer
```

### 2. Именование компонентов

Использование понятных имен для компонентов:

```javascript
// Хорошо: понятные имена
function UserProfileCard() { ... }
function SearchResultsList() { ... }
function ProductGallery() { ... }

// Менее понятно
function Comp1() { ... }
function MyComponent() { ... }
function Thing() { ... }
```

### 3. Документирование компонентов

Каждый компонент должен быть должным образом документирован:

```javascript
/**
 * Компонент карточки товара
 * Отображает основную информацию о товаре и кнопку добавления в корзину
 * 
 * @param {Object} props - Свойства компонента
 * @param {Object} props.product - Объект товара с полями id, name, price, image
 * @param {Function} props.onAddToCart - Функция, вызываемая при добавлении в корзину
 * @param {boolean} props.showDescription - Показывать ли описание товара
 * 
 * @returns {string} HTML-разметка карточки товара
 */
function ProductCard({ product, onAddToCart, showDescription = false }) {
  return `
    <div class="product-card" data-product-id="${product.id}">
      <img src="${product.image}" alt="${product.name}" class="product-image">
      <h3 class="product-name">${product.name}</h3>
      <div class="product-price">${product.price} руб.</div>
      ${showDescription ? `<p class="product-description">${product.description}</p>` : ''}
      <button class="add-to-cart-btn" onclick="onAddToCart(${product.id})">
        Добавить в корзину
      </button>
    </div>
  `;
}
```

### 4. Тестирование компонентов

Каждый компонент должен быть легко тестируемым:

```javascript
// Простой тест для компонента
function testProductCard() {
  const product = {
    id: 1,
    name: 'Тестовый товар',
    price: 1000,
    image: '/test-image.jpg'
  };
  
  const html = ProductCard({ product, onAddToCart: () => {} });
  
  // Проверка, что HTML содержит ожидаемые элементы
  console.assert(html.includes(product.name), 'HTML должен содержать название товара');
  console.assert(html.includes(product.price.toString()), 'HTML должен содержать цену товара');
  console.assert(html.includes('add-to-cart-btn'), 'HTML должен содержать кнопку добавления в корзину');
}
```

### 5. Оптимизация производительности

При композиции компонентов важно учитывать производительность:

```javascript
// Использование мемоизации для сложных компонентов
const memoizedComponents = new Map();

function memoizeComponent(componentFn) {
  return function(props) {
    const key = JSON.stringify(props);
    if (memoizedComponents.has(key)) {
      return memoizedComponents.get(key);
    }
    
    const result = componentFn(props);
    memoizedComponents.set(key, result);
    return result;
  };
}

const ExpensiveList = memoizeComponent(function({ items }) {
  return `
    <ul class="expensive-list">
      ${items.map(item => `<li>${item.name}</li>`).join('')}
    </ul>
  `;
});
```

## Заключение

Композиция HTML-компонентов является ключевым принципом современной фронтенд-разработки, позволяющим создавать гибкие, масштабируемые и легко поддерживаемые пользовательские интерфейсы. В условиях российского IT-ландшафта 2025 года, где важна скорость разработки и качество продуктов, композиция компонентов становится не просто хорошей практикой, а необходимостью.

Преимущества композиции особенно важны в крупных проектах, где множество разработчиков работают с общим кодом, и где требуется высокая степень повторного использования кода и унификации пользовательского опыта.

> [!tip] Совет
> Начинайте с простых компонентов и постепенно создавайте более сложные композиции. Следуйте принципу "одна ответственность" - каждый компонент должен делать одну вещь и делать ее хорошо.

> [!warning] Важно
> Избегайте создания компонентов, которые слишком тесно связаны друг с другом. Хороший признак слабой связанности - возможность использовать компонент в разных контекстах без изменений.

## См. также

- [[Декларативная-разметка]]
- [[Функции-высшего-порядка-в-HTML]]
- [[Чистые-компоненты]]
- [[Неизменяемость-в-HTML]]
- [[Функциональное программирование в JavaScript]]
- [[Веб-компоненты]]