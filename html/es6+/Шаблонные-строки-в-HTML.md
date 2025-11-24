---
aliases: ["Шаблонные строки в HTML", "Template literals в HTML", "Шаблоны в HTML"]
tags: [html, es6, template-literals, javascript, фронтенд]
---

# Шаблонные строки в HTML

## Обзор

Шаблонные строки (template literals) - это мощная возможность ES6, которая позволяет создавать строки с встроенными выражениями и многострочным содержимым. В контексте HTML шаблонные строки значительно упрощают создание динамического содержимого и позволяют более эффективно управлять HTML-шаблонами.

## Синтаксис шаблонных строк

Шаблонные строки заключаются в обратные апострофы (`` ` ``) и позволяют встраивать выражения с помощью синтаксиса `${expression}`:

```javascript
const name = 'Иван';
const age = 30;
const message = `Привет, ${name}! Тебе ${age} лет.`;
console.log(message); // "Привет, Иван! Тебе 30 лет."
```

## Применение в HTML

### 1. Генерация HTML-разметки

Шаблонные строки особенно полезны для генерации HTML-разметки на основе данных:

```javascript
function createUserCard(user) {
  return `
    <div class="user-card">
      <img src="${user.avatar}" alt="${user.name}" class="avatar">
      <h3 class="username">${user.name}</h3>
      <p class="email">${user.email}</p>
      <p class="bio">${user.bio || ''}</p>
    </div>
  `;
}

// Использование
const user = {
  name: 'Анна Петрова',
  email: 'anna@example.com',
  avatar: '/images/avatar.jpg',
  bio: 'Разработчик с 5-летним стажем'
};

document.getElementById('container').innerHTML = createUserCard(user);
```

### 2. Создание сложных HTML-структур

Шаблонные строки позволяют создавать сложные HTML-структуры с минимальными усилиями:

```javascript
function createProductList(products) {
  return `
    <div class="product-list">
      ${products.map(product => `
        <div class="product-card" data-id="${product.id}">
          <img src="${product.image}" alt="${product.name}" class="product-image">
          <div class="product-info">
            <h3 class="product-name">${product.name}</h3>
            <p class="product-description">${product.description}</p>
            <div class="product-price">${formatPrice(product.price)}</div>
            <button class="add-to-cart" data-product-id="${product.id}">
              Добавить в корзину
            </button>
          </div>
        </div>
      `).join('')}
    </div>
  `;
}

function formatPrice(price) {
  // Форматирование цены с учетом российских стандартов
  return new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency: 'RUB'
  }).format(price);
}
```

## Практические примеры

### 1. Управление DOM с шаблонными строками

```javascript
class TemplateManager {
  static createUserRow(user) {
    return `
      <tr data-user-id="${user.id}">
        <td>${this.escapeHtml(user.name)}</td>
        <td>${this.escapeHtml(user.email)}</td>
        <td>${this.formatDate(user.created)}</td>
        <td>
          <button class="edit-btn" data-action="edit" data-id="${user.id}">Редактировать</button>
          <button class="delete-btn" data-action="delete" data-id="${user.id}">Удалить</button>
        </td>
      </tr>
    `;
  }
  
  static escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
  }
  
  static formatDate(dateString) {
    const date = new Date(dateString);
    return date.toLocaleDateString('ru-RU');
  }
}

// Использование
const users = [
  { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', created: '2025-01-15' },
  { id: 2, name: 'Мария Смирнова', email: 'maria@example.com', created: '2025-01-16' }
];

const tableBody = document.querySelector('#users-table tbody');
users.forEach(user => {
  tableBody.insertAdjacentHTML('beforeend', TemplateManager.createUserRow(user));
});
```

### 2. Условная отрисовка с шаблонными строками

```javascript
function renderUserProfile(user) {
  return `
    <div class="profile-card">
      <img src="${user.avatar}" alt="${user.name}" class="profile-avatar">
      <div class="profile-info">
        <h2>${user.name}</h2>
        <p class="profile-email">${user.email}</p>
        ${user.verified ? '<span class="verified-badge">✓ Подтверждён</span>' : ''}
        ${user.bio ? `<p class="profile-bio">${user.bio}</p>` : ''}
        <div class="social-links">
          ${user.twitter ? `<a href="${user.twitter}" class="social-link">Twitter</a>` : ''}
          ${user.linkedin ? `<a href="${user.linkedin}" class="social-link">LinkedIn</a>` : ''}
        </div>
      </div>
    </div>
  `;
}
```

### 3. Работа с атрибутами и классами

```javascript
function createButton(text, options = {}) {
  const {
    type = 'button',
    classes = '',
    disabled = false,
    onClick = '',
    dataAttributes = {}
  } = options;
  
  const dataAttrs = Object.entries(dataAttributes)
    .map(([key, value]) => `data-${key}="${value}"`)
    .join(' ');
  
  return `
    <button 
      type="${type}"
      class="btn ${classes}"
      ${disabled ? 'disabled' : ''}
      ${onClick ? `onclick="${onClick}"` : ''}
      ${dataAttrs}>
      ${text}
    </button>
  `;
}

// Использование
const buttonHtml = createButton('Отправить', {
  type: 'submit',
  classes: 'btn-primary btn-large',
  disabled: false,
  dataAttributes: {
    action: 'submit-form',
    formId: 'registration-form'
  }
});
```

## Безопасность при использовании шаблонных строк

При работе с пользовательским вводом в шаблонных строках важно учитывать безопасность:

```javascript
class SafeTemplateRenderer {
  static escapeHtml(unsafe) {
    if (typeof unsafe !== 'string') return unsafe;
    
    return unsafe
      .replace(/&/g, "&amp;")
      .replace(/</g, "&lt;")
      .replace(/>/g, "&gt;")
      .replace(/"/g, "&quot;")
      .replace(/'/g, "&#039;");
  }
  
  static createSafeUserCard(user) {
    return `
      <div class="user-card">
        <h3>${this.escapeHtml(user.name)}</h3>
        <p>${this.escapeHtml(user.comment || '')}</p>
      </div>
    `;
  }
  
  static createAttributeSafe(attributes) {
    return Object.entries(attributes)
      .map(([key, value]) => `${key}="${this.escapeHtml(value)}"`)
      .join(' ');
  }
}
```

## Продвинутые паттерны

### 1. Шаблонные строки с функциями

```javascript
const template = (strings, ...values) => {
  // Кастомная обработка шаблонных строк
  let result = strings[0];
  
  values.forEach((value, i) => {
    const processedValue = typeof value === 'string' ? 
      SafeTemplateRenderer.escapeHtml(value) : 
      value;
    result += processedValue + strings[i + 1];
  });
  
  return result;
};

// Использование
const user = { name: '<script>alert("XSS")</script>Иван', email: 'ivan@example.com' };
const safeHtml = template`
  <div class="user">
    <h3>${user.name}</h3>
    <p>${user.email}</p>
  </div>
`;
```

### 2. Шаблоны с локализацией

```javascript
class LocalizedTemplate {
  constructor(locale = 'ru-RU') {
    this.locale = locale;
    this.translations = {
      ru: {
        welcome: 'Добро пожаловать',
        logout: 'Выйти',
        profile: 'Профиль'
      },
      en: {
        welcome: 'Welcome',
        logout: 'Logout',
        profile: 'Profile'
      }
    };
  }
  
  t(key) {
    return this.translations[this.locale]?.[key] || key;
  }
  
  createHeader(user) {
    return `
      <header class="app-header">
        <h1>${this.t('welcome')}, ${user.name}!</h1>
        <nav>
          <a href="/profile">${this.t('profile')}</a>
          <a href="/logout">${this.t('logout')}</a>
        </nav>
      </header>
    `;
  }
}
```

## Совместимость и поддержка

Шаблонные строки поддерживаются всеми современными браузерами, включая:

- Chrome 41+
- Firefox 34+
- Safari 9+
- Edge 13+
- Opera 28+

Для поддержки старых браузеров рекомендуется использовать транспиляцию через Babel или другие инструменты.

## Заключение

Шаблонные строки в HTML позволяют создавать динамический контент более читаемым и поддерживаемым способом. В российских проектах 2025 года они особенно полезны для генерации интернационализированного контента и обеспечения безопасности при работе с пользовательским вводом.

> [!tip] Совет
> При использовании шаблонных строк для генерации HTML всегда экранируйте пользовательский ввод, чтобы предотвратить XSS-атаки.

> [!warning] Важно
> В России в 2025 году особое внимание уделяется требованиям безопасности. При использовании шаблонных строк всегда учитывайте правила экранирования и валидации данных.

## Связанные темы

- [[Модульность-в-HTML]]
- [[Деструктуризация-в-HTML]]
- [[Новые-атрибуты-и-элементы]]
- [[Современные-подходы-к-валидации]]
- [[ES6 в HTML]]
- [[Шаблоны веб-приложений]]