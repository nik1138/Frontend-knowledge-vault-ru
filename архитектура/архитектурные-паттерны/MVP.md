---
aliases: ["Model-View-Presenter", "Модель-Представление-Презентер"]
tags: [architecture, frontend, pattern, mvp]
---

# MVP (Model-View-Presenter)

## Обзор

MVP (Model-View-Presenter) - это архитектурный паттерн, который является модификацией паттерна MVC. Основная цель MVP - улучшить тестируемость и уменьшить связанность между компонентами, особенно между View и Model. Паттерн был разработан как альтернатива MVC для решения проблем, связанных с тестированием и поддержкой.

## Структура паттерна

MVP также состоит из трех основных компонентов, но с измененной ролью Presenter:

### Model (Модель)
- **Отвечает за**: бизнес-логику, данные и правила валидации
- **Не зависит от**: View или Presenter
- **Содержит**: данные приложения и бизнес-правила

```javascript
// Пример модели в MVP
class UserModel {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  updateName(newName) {
    if (newName && newName.trim() !== '') {
      this.name = newName;
      return true;
    }
    return false;
  }

  updateEmail(newEmail) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (emailRegex.test(newEmail)) {
      this.email = newEmail;
      return true;
    }
    return false;
  }

  validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
}
```

### View (Представление)
- **Отвечает за**: отображение данных и получение пользовательского ввода
- **Содержит минимальную логику**: только для отображения и передачи событий в Presenter
- **Интерфейс**: обычно реализует интерфейс для взаимодействия с Presenter

```html
<!-- Пример представления -->
<div id="user-view">
  <h2>Данные пользователя</h2>
  <p>Имя: <span id="user-name"></span></p>
  <p>Email: <span id="user-email"></span></p>
  
  <form id="user-form">
    <input type="text" id="name-input" placeholder="Имя">
    <input type="email" id="email-input" placeholder="Email">
    <button type="submit">Обновить</button>
  </form>
  
  <div id="error-message" style="color: red;"></div>
</div>
```

```javascript
// Реализация View в JavaScript
class UserView {
  constructor() {
    this.presenter = null;
    this.form = document.getElementById('user-form');
    this.nameInput = document.getElementById('name-input');
    this.emailInput = document.getElementById('email-input');
    this.nameDisplay = document.getElementById('user-name');
    this.emailDisplay = document.getElementById('user-email');
    this.errorMessage = document.getElementById('error-message');
    
    this.initEventListeners();
  }

  setPresenter(presenter) {
    this.presenter = presenter;
  }

  initEventListeners() {
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      const userData = {
        name: this.nameInput.value,
        email: this.emailInput.value
      };
      this.presenter.updateUser(userData);
    });
  }

  showUser(user) {
    this.nameDisplay.textContent = user.name;
    this.emailDisplay.textContent = user.email;
  }

  showError(message) {
    this.errorMessage.textContent = message;
  }

  clearForm() {
    this.nameInput.value = '';
    this.emailInput.value = '';
    this.errorMessage.textContent = '';
  }
}
```

### Presenter (Презентер)
- **Отвечает за**: логику представления и взаимодействие между Model и View
- **Зависит от**: интерфейса View и Model
- **Содержит**: логику обработки пользовательских действий и обновления представления

```javascript
// Пример презентера
class UserPresenter {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    this.view.setPresenter(this);
  }

  updateUser(userData) {
    let hasChanges = false;
    
    if (userData.name) {
      const success = this.model.updateName(userData.name);
      if (!success) {
        this.view.showError('Имя не может быть пустым');
        return;
      }
      hasChanges = true;
    }
    
    if (userData.email) {
      const success = this.model.updateEmail(userData.email);
      if (!success) {
        this.view.showError('Некорректный формат email');
        return;
      }
      hasChanges = true;
    }
    
    if (hasChanges) {
      this.view.showUser(this.model);
      this.view.clearForm();
    }
  }

  loadUser() {
    this.view.showUser(this.model);
  }
}

// Использование
const userModel = new UserModel('Иван Петров', 'ivan@example.com');
const userView = new UserView();
const userPresenter = new UserPresenter(userModel, userView);

// Загрузка начальных данных
userPresenter.loadUser();
```

## Варианты реализации MVP

### Passive View (Пассивное представление)
- View не содержит никакой логики
- Все изменения данных и состояния происходят через Presenter
- Лучше тестируемость, но более многословный код

### Supervising Controller (Контроллер-надзиратель)
- View может автоматически связываться с Model для простых данных
- Presenter управляет сложной логикой и событиями
- Баланс между тестируемостью и простотой

## Преимущества MVP

1. **Лучшая тестируемость**: Presenter можно тестировать без UI
2. **Четкое разделение ответственности**: каждый компонент имеет конкретную роль
3. **Меньшая связанность**: View и Model не зависят друг от друга
4. **Поддержка юнит-тестирования**: легко создавать mock-объекты

## Недостатки MVP

1. **Сложность для начинающих**: требует хорошего понимания архитектуры
2. **Более многословный код**: особенно в сравнении с MVC
3. **Потенциальная перегрузка Presenter**: может стать "толстым"

## Применение в российских реалиях 2025

MVP особенно популярен в российской разработке:

- **Корпоративных приложениях**: где важна тестируемость и поддержка
- **Desktop-приложениях**: особенно на WPF, Windows Forms
- **Android-разработке**: как альтернатива MVVM
- **Проектах с жесткими требованиями к тестированию**: государственные и банковские системы

В условиях импортозамещения MVP остается предпочтительным выбором для систем, где важна стабильность и тестируемость кода.

## Сравнение с другими паттернами

- [[MVC]]: MVP имеет более четкое разделение между View и Model
- [[MVVM]]: MVVM проще в реализации с привязкой данных
- [[Flux]]: предлагает другой подход к управлению состоянием
- [[Чистая-архитектура]]: более сложная, но гибкая архитектура

## Практические рекомендации

1. **Создавайте интерфейсы для View**: это упрощает тестирование
2. **Избегайте прямого доступа к DOM в Presenter**: используйте методы View
3. **Разделяйте Presenter по функциональности**: не создавайте слишком большие классы
4. **Используйте Dependency Injection**: для лучшего управления зависимостями

## Заключение

MVP - отличный выбор для приложений, где важна тестируемость и четкое разделение ответственности. Паттерн особенно полезен в корпоративной среде, где требуется высокое качество кода и поддержка.

## См. также

- [[MVC]]
- [[MVVM]]
- [[Flux]]
- [[Чистая-архитектура]]
- [[Архитектурные-паттерны]]