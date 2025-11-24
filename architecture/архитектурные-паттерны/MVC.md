---
aliases: ["Model-View-Controller", "Архитектурный паттерн MVC"]
tags: ["#architecture", "#frontend", "#mvc", "#pattern"]
---

# MVC (Model-View-Controller)

## Описание

MVC (Model-View-Controller) - это архитектурный паттерн, который разделяет приложение на три основные компоненты: Model (Модель), View (Представление) и Controller (Контроллер). Этот паттерн помогает организовать код, улучшает тестируемость и позволяет легче вносить изменения в приложение.

## Компоненты

### Model (Модель)
- Отвечает за данные и бизнес-логику
- Не зависит от View и Controller
- Уведомляет View об изменениях (в некоторых реализациях)

### View (Представление)
- Отображает данные из Model
- Обычно не содержит сложной логики
- Может взаимодействовать с Controller

### Controller (Контроллер)
- Обрабатывает пользовательский ввод
- Координирует взаимодействие между Model и View
- Принимает решения о том, какие данные отображать

## Преимущества

- **Разделение ответственности**: Каждый компонент имеет четко определенную роль
- **Повторное использование кода**: Model может использоваться разными View
- **Упрощение тестирования**: Логика изолирована в Controller и Model
- **Поддержка нескольких интерфейсов**: Одна Model может обслуживать несколько View

## Недостатки

- **Сложность для небольших приложений**: Может быть избыточным для простых задач
- **Тесная связь между View и Controller**: В некоторых реализациях сложно изменить один без другого
- **Сложность в асинхронных приложениях**: Требует дополнительных паттернов для обработки асинхронности

## Пример реализации на JavaScript

```javascript
// Model
class UserModel {
  constructor() {
    this.users = [];
    this.observers = [];
  }

  addUser(user) {
    this.users.push(user);
    this.notifyObservers();
  }

  getUsers() {
    return this.users;
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  notifyObservers() {
    this.observers.forEach(observer => observer.update());
  }
}

// View
class UserView {
  constructor() {
    this.container = document.getElementById('users');
  }

  render(users) {
    this.container.innerHTML = '';
    users.forEach(user => {
      const div = document.createElement('div');
      div.textContent = user.name;
      this.container.appendChild(div);
    });
  }
}

// Controller
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    
    this.model.addObserver(this);
    this.initEventListeners();
  }

  initEventListeners() {
    document.getElementById('addUserBtn').addEventListener('click', () => {
      const name = document.getElementById('userName').value;
      this.model.addUser({ name: name });
    });
  }

  update() {
    const users = this.model.getUsers();
    this.view.render(users);
  }
}

// Использование
const userModel = new UserModel();
const userView = new UserView();
const userController = new UserController(userModel, userView);
```

## Применение в современных фреймворках

- **Angular**: Использует концепции MVC, хотя и с некоторыми отличиями
- **React**: Компонентный подход частично реализует MVC-принципы
- **Vue.js**: Также поддерживает MVC-подобную архитектуру

## Сравнение с другими паттернами

- [[MVP]] - более строгое разделение между View и Model
- [[MVVM]] - автоматическая синхронизация между View и Model
- [[Flux]] - односторонний поток данных
- [[Clean-Architecture]] - более глубокое разделение на слои

## Лучшие практики

1. **Изолировать бизнес-логику в Model**
2. **Не допускать прямого взаимодействия между View и Model**
3. **Контроллер должен быть тонким** - основная логика в Model
4. **Использовать Observer Pattern для связи Model и View**

## Рекомендации по использованию

MVC особенно эффективен в приложениях с:
- Четко определенной бизнес-логикой
- Необходимостью поддержки нескольких интерфейсов
- Командной разработкой, где разные разработчики работают над разными компонентами

## См. также

- [[MVP]]
- [[MVVM]]
- [[Flux]]
- [[Clean-Architecture]]
- [[State-менеджмент]]

</content>