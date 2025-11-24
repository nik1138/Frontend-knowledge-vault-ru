---
aliases: [MVC, MVVM, Model-View-Controller, Model-View-ViewModel]
tags: [javascript, architecture, mvc, mvvm, frontend, patterns]
---

# MVC и MVVM в JavaScript

## Введение

MVC (Model-View-Controller) и MVVM (Model-View-ViewModel) - это архитектурные паттерны, которые помогают организовать код в приложениях, разделяя ответственность между различными компонентами. Эти паттерны особенно важны в JavaScript-приложениях, где управление состоянием и взаимодействие с пользовательским интерфейсом требуют четкой структуры.

В российском контексте 2025 года, эти паттерны становятся особенно актуальными при разработке корпоративных решений и веб-приложений, соответствующих требованиям локализации и безопасности.

## MVC (Model-View-Controller)

### Основные компоненты

- **Model** - отвечает за данные и бизнес-логику
- **View** - представляет пользовательский интерфейс
- **Controller** - обрабатывает пользовательский ввод и координирует взаимодействие между Model и View

### Пример реализации MVC на JavaScript

```javascript
// Model
class UserModel {
  constructor() {
    this.users = [];
  }

  addUser(userData) {
    const user = {
      id: Date.now(),
      ...userData,
      createdAt: new Date()
    };
    this.users.push(user);
    return user;
  }

  getUser(id) {
    return this.users.find(user => user.id === id);
  }

  getAllUsers() {
    return this.users;
  }
}

// View
class UserView {
  constructor() {
    this.container = document.getElementById('users-container');
  }

  displayUsers(users) {
    this.container.innerHTML = '';
    users.forEach(user => {
      const userElement = document.createElement('div');
      userElement.className = 'user-card';
      userElement.innerHTML = `
        <h3>${user.name}</h3>
        <p>ИД: ${user.id}</p>
        <p>Электронная почта: ${user.email}</p>
        <p>Дата регистрации: ${user.createdAt.toLocaleDateString('ru-RU')}</p>
      `;
      this.container.appendChild(userElement);
    });
  }

  displayUser(user) {
    this.container.innerHTML = `
      <div class="user-detail">
        <h2>Детали пользователя</h2>
        <p>Имя: ${user.name}</p>
        <p>ИД: ${user.id}</p>
        <p>Email: ${user.email}</p>
        <p>Дата регистрации: ${user.createdAt.toLocaleDateString('ru-RU')}</p>
      </div>
    `;
  }
}

// Controller
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
  }

  addNewUser(userData) {
    const user = this.model.addUser(userData);
    this.view.displayUsers(this.model.getAllUsers());
    return user;
  }

  showAllUsers() {
    const users = this.model.getAllUsers();
    this.view.displayUsers(users);
  }

  showUser(id) {
    const user = this.model.getUser(id);
    if (user) {
      this.view.displayUser(user);
    } else {
      this.view.container.innerHTML = '<p>Пользователь не найден</p>';
    }
  }
}

// Инициализация MVC архитектуры
const userModel = new UserModel();
const userView = new UserView();
const userController = new UserController(userModel, userView);

// Пример использования
userController.addNewUser({ name: 'Иван Петров', email: 'ivan@example.com' });
userController.addNewUser({ name: 'Мария Сидорова', email: 'maria@example.com' });
userController.showAllUsers();
```

### Преимущества MVC

- **Разделение ответственности** - каждый компонент имеет четко определенную роль
- **Повторное использование кода** - Model может использоваться несколькими View
- **Простота тестирования** - каждый компонент можно тестировать отдельно
- **Легкость поддержки** - изменения в одном компоненте не влияют на другие

### Недостатки MVC

- **Сложность для маленьких приложений** - может быть избыточным
- **Связность Controller и View** - могут быть тесно связаны
- **Проблемы с производительностью** - при частых обновлениях интерфейса

## MVVM (Model-View-ViewModel)

### Основные компоненты

- **Model** - отвечает за данные и бизнес-логику
- **View** - представляет пользовательский интерфейс
- **ViewModel** - обеспечивает связь между Model и View, содержит представление данных и команды

### Пример реализации MVVM на JavaScript

```javascript
// Model
class OrderModel {
  constructor() {
    this.orders = [];
  }

  addOrder(orderData) {
    const order = {
      id: Date.now(),
      ...orderData,
      status: 'новый',
      createdAt: new Date()
    };
    this.orders.push(order);
    return order;
  }

  updateOrderStatus(id, status) {
    const order = this.orders.find(order => order.id === id);
    if (order) {
      order.status = status;
      order.updatedAt = new Date();
    }
    return order;
  }

  getAllOrders() {
    return this.orders;
  }
}

// ViewModel
class OrderViewModel {
  constructor(model) {
    this.model = model;
    this.orders = this.model.getAllOrders();
    this.filteredOrders = [];
    this.filter = 'all';
  }

  addOrder(orderData) {
    const order = this.model.addOrder(orderData);
    this.orders.push(order);
    this.applyFilter();
    return order;
  }

  updateOrderStatus(id, status) {
    const order = this.model.updateOrderStatus(id, status);
    this.applyFilter();
    return order;
  }

  setFilter(filter) {
    this.filter = filter;
    this.applyFilter();
  }

  applyFilter() {
    switch(this.filter) {
      case 'new':
        this.filteredOrders = this.orders.filter(order => order.status === 'новый');
        break;
      case 'in_progress':
        this.filteredOrders = this.orders.filter(order => order.status === 'в_работе');
        break;
      case 'completed':
        this.filteredOrders = this.orders.filter(order => order.status === 'завершен');
        break;
      default:
        this.filteredOrders = [...this.orders];
    }
  }

  getOrdersForDisplay() {
    return this.filteredOrders;
  }

  // Команда для оформления заказа
  placeOrder(customerInfo) {
    return this.addOrder({
      customer: customerInfo,
      items: [],
      total: 0
    });
  }

  // Команда для расчета налогов для российского рынка
  calculateTax(amount) {
    return amount * 0.13; // НДФЛ в России
  }
}

// View
class OrderView {
  constructor(viewModel) {
    this.viewModel = viewModel;
    this.container = document.getElementById('orders-container');
    this.bindEvents();
  }

  bindEvents() {
    // Привязка событий к элементам управления
    document.getElementById('filter-new').addEventListener('click', () => {
      this.viewModel.setFilter('new');
      this.render();
    });

    document.getElementById('filter-progress').addEventListener('click', () => {
      this.viewModel.setFilter('in_progress');
      this.render();
    });

    document.getElementById('filter-completed').addEventListener('click', () => {
      this.viewModel.setFilter('completed');
      this.render();
    });
  }

  render() {
    const orders = this.viewModel.getOrdersForDisplay();
    this.container.innerHTML = '';
    
    orders.forEach(order => {
      const orderElement = document.createElement('div');
      orderElement.className = 'order-card';
      orderElement.innerHTML = `
        <div class="order-header">
          <span class="order-id">Заказ #${order.id}</span>
          <span class="order-status status-${order.status}">${this.getStatusText(order.status)}</span>
        </div>
        <div class="order-details">
          <p>Клиент: ${order.customer?.name || 'Не указан'}</p>
          <p>Дата: ${order.createdAt.toLocaleDateString('ru-RU')}</p>
          <p>Статус: ${this.getStatusText(order.status)}</p>
        </div>
        <button class="update-status-btn" data-id="${order.id}" data-status="в_работе">В работу</button>
        <button class="update-status-btn" data-id="${order.id}" data-status="завершен">Завершить</button>
      `;
      
      this.container.appendChild(orderElement);
    });

    // Добавляем обработчики для кнопок обновления статуса
    document.querySelectorAll('.update-status-btn').forEach(button => {
      button.addEventListener('click', (e) => {
        const orderId = parseInt(e.target.dataset.id);
        const newStatus = e.target.dataset.status;
        this.viewModel.updateOrderStatus(orderId, newStatus);
        this.render();
      });
    });
  }

  getStatusText(status) {
    const statusMap = {
      'новый': 'Новый',
      'в_работе': 'В работе',
      'завершен': 'Завершен'
    };
    return statusMap[status] || status;
  }
}

// Инициализация MVVM архитектуры
const orderModel = new OrderModel();
const orderViewModel = new OrderViewModel(orderModel);
const orderView = new OrderView(orderViewModel);

// Пример использования
orderViewModel.addOrder({
  customer: { name: 'ООО Ромашка', inn: '7701234567' },
  items: [{ name: 'Товар 1', quantity: 2, price: 1000 }],
  total: 2000
});

orderView.render();
```

### Преимущества MVVM

- **Двустороннее связывание данных** - автоматическое обновление View при изменении ViewModel
- **Четкое разделение логики** - View не содержит бизнес-логики
- **Удобство тестирования** - ViewModel можно тестировать независимо от View
- **Поддержка сложных интерфейсов** - особенно полезен для приложений с богатым UI

### Недостатки MVVM

- **Сложность реализации двустороннего связывания** - требует дополнительной инфраструктуры
- **Проблемы с производительностью** - при большом количестве связанных данных
- **Сложность отладки** - из-за автоматического связывания данных

## Сравнение MVC и MVVM

| Критерий | MVC | MVVM |
|----------|-----|------|
| Разделение ответственности | Высокое | Высокое |
| Сложность реализации | Средняя | Высокая |
| Тестирование | Простое | Простое |
| Двустороннее связывание | Нет | Да |
| Подходящие сценарии | Веб-приложения, REST API | Богатые клиентские приложения |
| Производительность | Высокая | Может снижаться при большом объеме данных |

## Применение в российском контексте 2025 года

В условиях российского рынка 2025 года, MVC и MVVM применяются с учетом следующих факторов:

### Соответствие требованиям

```javascript
// Пример ViewModel с учетом российского законодательства
class RussianBusinessViewModel {
  constructor(model) {
    this.model = model;
  }

  // Метод для проверки ИНН
  validateINN(inn) {
    // Логика проверки ИНН по российским правилам
    if (inn.length === 10 || inn.length === 12) {
      // Проверка контрольных цифр
      return this.calculateINNControlSum(inn);
    }
    return false;
  }

  calculateINNControlSum(inn) {
    // Реализация проверки контрольных сумм ИНН
    // В зависимости от длины ИНН (10 или 12 цифр)
    if (inn.length === 10) {
      const coefficients = [2, 4, 10, 3, 5, 9, 4, 6, 8];
      let sum = 0;
      for (let i = 0; i < 9; i++) {
        sum += parseInt(inn[i]) * coefficients[i];
      }
      const controlDigit = sum % 11 % 10;
      return controlDigit === parseInt(inn[9]);
    } else if (inn.length === 12) {
      const coefficients1 = [7, 2, 4, 10, 3, 5, 9, 4, 6, 8];
      const coefficients2 = [3, 7, 2, 4, 10, 3, 5, 9, 4, 6, 8];
      let sum1 = 0, sum2 = 0;
      
      for (let i = 0; i < 10; i++) {
        sum1 += parseInt(inn[i]) * coefficients1[i];
        sum2 += parseInt(inn[i]) * coefficients2[i];
      }
      
      const controlDigit1 = sum1 % 11 % 10;
      const controlDigit2 = sum2 % 11 % 10;
      
      return controlDigit1 === parseInt(inn[10]) && controlDigit2 === parseInt(inn[11]);
    }
    return false;
  }

  // Метод для расчета налогов по российским ставкам
  calculateTaxes(income) {
    const personalIncomeTax = income * 0.13; // НДФЛ 13%
    const pensionContribution = income * 0.22; // Пенсионные взносы
    const medicalContribution = income * 0.051; // Медицинские взносы
    
    return {
      personalIncomeTax,
      pensionContribution,
      medicalContribution,
      totalDeductions: personalIncomeTax + pensionContribution + medicalContribution
    };
  }
}
```

### Локализация

```javascript
// Пример View с поддержкой русского языка и российских форматов
class RussianLocaleView {
  formatDate(date) {
    return date.toLocaleDateString('ru-RU', {
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      weekday: 'long'
    });
  }

  formatCurrency(amount) {
    return new Intl.NumberFormat('ru-RU', {
      style: 'currency',
      currency: 'RUB',
      minimumFractionDigits: 2
    }).format(amount);
  }

  formatPhoneNumber(phone) {
    // Форматирование российского номера телефона
    return phone.replace(/(\d{1})(\d{3})(\d{3})(\d{2})(\d{2})/, '+$1 ($2) $3-$4-$5');
  }
}
```

## Современные фреймворки и MVC/MVVM

Современные JavaScript-фреймворки реализуют концепции MVC и MVVM по-своему:

- **Angular** - использует компонентную архитектуру, близкую к MVVM с двусторонним связыванием
- **Vue.js** - предоставляет двустороннее связывание данных, подобно MVVM
- **React** - ближе к MVC с контроллерами в виде хуков и функций

## Заключение

MVC и MVVM остаются важными архитектурными паттернами в JavaScript-разработке. Выбор между ними зависит от конкретных требований проекта, сложности интерфейса и предпочтений команды разработчиков.

В российском контексте 2025 года, важно учитывать требования локализации, безопасности и соответствия законодательству при реализации этих паттернов.

> [!tip] Совет
> Используйте MVC для более простых приложений с четким разделением логики, и MVVM для сложных интерфейсов с активным взаимодействием пользователя и автоматическим обновлением данных.

См. также: [[Паттерны-проектирования]], [[Компонентная-архитектура]], [[Архитектура-приложений]]