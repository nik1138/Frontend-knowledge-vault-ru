---
aliases: [MVC, MVVM, Model-View-Controller, Model-View-ViewModel]
tags: [javascript, архитектура, паттерны, MVC, MVVM]
---

# MVC и MVVM: Архитектурные паттерны в JavaScript

## Обзор

MVC (Model-View-Controller) и MVVM (Model-View-ViewModel) - это архитектурные паттерны, которые помогают организовать код приложения, разделяя ответственность между различными компонентами. Эти паттерны особенно важны в JavaScript-приложениях для поддержания чистоты кода и упрощения сопровождения.

## MVC (Model-View-Controller)

### Основные компоненты

- **Model** - отвечает за данные и бизнес-логику
- **View** - отвечает за отображение данных пользователю
- **Controller** - обрабатывает пользовательский ввод и координирует Model и View

### Реализация MVC в JavaScript

```javascript
// Model
class UserModel {
  constructor() {
    this.users = [];
  }
  
  getAllUsers() {
    return this.users;
  }
  
  addUser(user) {
    this.users.push(user);
    this.notify();
  }
  
  notify() {
    // Уведомляем контроллер о изменениях
    if (this.onChange) {
      this.onChange();
    }
  }
}

// View
class UserView {
  constructor() {
    this.template = document.getElementById('user-template');
  }
  
  displayUsers(users) {
    const container = document.getElementById('users-container');
    container.innerHTML = '';
    
    users.forEach(user => {
      const userElement = this.template.content.cloneNode(true);
      userElement.querySelector('.user-name').textContent = user.name;
      userElement.querySelector('.user-email').textContent = user.email;
      container.appendChild(userElement);
    });
  }
}

// Controller
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    this.model.onChange = () => this.updateView();
    
    // Инициализация
    this.updateView();
  }
  
  addNewUser(userData) {
    this.model.addUser(userData);
  }
  
  updateView() {
    const users = this.model.getAllUsers();
    this.view.displayUsers(users);
  }
}

// Использование
const userModel = new UserModel();
const userView = new UserView();
const userController = new UserController(userModel, userView);

// Добавление пользователей
userController.addNewUser({ name: 'Иван Иванов', email: 'ivan@example.com' });
userController.addNewUser({ name: 'Мария Смирнова', email: 'maria@example.com' });
```

## MVVM (Model-View-ViewModel)

### Основные компоненты

- **Model** - представляет данные и бизнес-логику
- **View** - пользовательский интерфейс
- **ViewModel** - абстракция View, которая содержит представление данных и команды для взаимодействия с Model

### Реализация MVVM в JavaScript

```javascript
// Model
class ProductModel {
  constructor() {
    this.products = [
      { id: 1, name: 'Ноутбук', price: 50000 },
      { id: 2, name: 'Смартфон', price: 30000 }
    ];
  }
  
  getProducts() {
    return this.products;
  }
  
  addProduct(product) {
    this.products.push(product);
  }
}

// ViewModel
class ProductViewModel {
  constructor(model) {
    this.model = model;
    this.products = this.model.getProducts();
    this.filteredProducts = [...this.products];
    this.searchTerm = '';
    
    // Привязка данных
    this.bindings = {};
  }
  
  setSearchTerm(term) {
    this.searchTerm = term;
    this.filterProducts();
    this.notify('filteredProducts');
  }
  
  filterProducts() {
    if (!this.searchTerm) {
      this.filteredProducts = [...this.products];
    } else {
      this.filteredProducts = this.products.filter(
        product => product.name.toLowerCase().includes(this.searchTerm.toLowerCase())
      );
    }
  }
  
  addProduct(product) {
    this.model.addProduct(product);
    this.products = this.model.getProducts();
    this.filterProducts();
    this.notify('products');
    this.notify('filteredProducts');
  }
  
  bind(property, callback) {
    if (!this.bindings[property]) {
      this.bindings[property] = [];
    }
    this.bindings[property].push(callback);
  }
  
  notify(property) {
    if (this.bindings[property]) {
      this.bindings[property].forEach(callback => callback(this[property]));
    }
  }
}

// View
class ProductView {
  constructor(viewModel) {
    this.viewModel = viewModel;
    this.init();
  }
  
  init() {
    // Привязка поиска
    const searchInput = document.getElementById('search-input');
    searchInput.addEventListener('input', (e) => {
      this.viewModel.setSearchTerm(e.target.value);
    });
    
    // Привязка списка продуктов
    this.viewModel.bind('filteredProducts', (products) => {
      this.renderProducts(products);
    });
    
    // Инициализация отображения
    this.renderProducts(this.viewModel.filteredProducts);
  }
  
  renderProducts(products) {
    const container = document.getElementById('product-list');
    container.innerHTML = '';
    
    products.forEach(product => {
      const productElement = document.createElement('div');
      productElement.className = 'product';
      productElement.innerHTML = `
        <h3>${product.name}</h3>
        <p>Цена: ${product.price} руб.</p>
      `;
      container.appendChild(productElement);
    });
  }
}

// Использование
const productModel = new ProductModel();
const productViewModel = new ProductViewModel(productModel);
const productView = new ProductView(productViewModel);
```

## Сравнение MVC и MVVM

| Характеристика | MVC | MVVM |
|----------------|-----|------|
| Связь компонентов | Controller управляет Model и View | ViewModel связывает Model и View |
| Тестирование | Простое тестирование Controller | Простое тестирование ViewModel |
| Сложность | Более простой для понимания | Более сложный, но гибкий |
| Использование в JS | Часто используется в серверных приложениях | Популярен в клиентских SPA |

## Практические рекомендации для российских разработчиков

### Выбор паттерна

В 2025 году российские разработчики чаще выбирают:

- **MVC** для традиционных веб-приложений с серверным рендерингом
- **MVVM** для одностраничных приложений (SPA) с богатым пользовательским интерфейсом

### Современные фреймворки

- **Vue.js** использует концепции MVVM
- **Angular** имеет встроенные возможности для реализации MVVM
- **React** позволяет реализовать оба паттерна через подходящие архитектурные решения

## Примеры использования в российской экосистеме

В российских компаниях, таких как Яндекс, Mail.ru Group и Сбер, MVC и MVVM используются для:

- Создания масштабируемых веб-приложений
- Поддержания читаемости кода в больших командах
- Упрощения тестирования и сопровождения

## Ссылки

- [[Паттерны-проектирования]] - для понимания других архитектурных паттернов
- [[Компонентная-архитектура]] - для понимания современных подходов к структурированию UI
- [[Архитектура-приложений]] - для комплексного понимания архитектуры
- [[Модульность]] - для понимания организации кода

## Ключевые теги

#javascript #mvc #mvvm #архитектура #паттерны #ru2025