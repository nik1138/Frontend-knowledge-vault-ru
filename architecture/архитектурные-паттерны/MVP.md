---
aliases: ["Model-View-Presenter", "Архитектурный паттерн MVP"]
tags: ["#architecture", "#frontend", "#mvp", "#pattern"]
---

# MVP (Model-View-Presenter)

## Описание

MVP (Model-View-Presenter) - это архитектурный паттерн, который является модификацией паттерна MVC. В MVP Presenter выступает в роли посредника между View и Model, обеспечивая более строгое разделение ответственности и упрощая тестирование за счет изоляции View от Model.

## Компоненты

### Model (Модель)
- Отвечает за данные и бизнес-логику
- Не зависит от View и Presenter
- Предоставляет данные для Presenter

### View (Представление)
- Отображает данные
- Передает пользовательские действия Presenter
- Имеет ссылку на Presenter
- В MVP View может быть пассивным (Passive View) или супервизором (Supervising Controller)

### Presenter (Презентер)
- Получает данные от Model
- Подготавливает данные для отображения во View
- Обрабатывает пользовательские действия из View
- Обновляет Model при необходимости

## Типы MVP

### Passive View (Пассивное представление)
- View не знает о Model
- Вся логика обновления View находится в Presenter
- View предоставляет интерфейс для обновления (сеттеры, методы)

### Supervising Controller (Супервизор-контроллер)
- Некоторые элементы View связаны с Model напрямую (например, через привязку данных)
- Presenter управляет более сложной логикой и событиями

## Преимущества

- **Лучшая тестируемость**: View изолирован от Model, Presenter легко тестировать
- **Четкое разделение ответственности**: Каждый компонент имеет свою роль
- **Упрощение View**: View становится проще, особенно в пассивной реализации
- **Независимость от UI-фреймворка**: Логика в Presenter не зависит от конкретного UI

## Недостатки

- **Сложность Presenter**: Может стать "толстым", особенно в больших приложениях
- **Увеличение количества кода**: Требуется больше интерфейсов и классов
- **Сложность для начинающих**: Требует понимания архитектурных концепций

## Пример реализации на JavaScript

```javascript
// Model
class UserModel {
  constructor() {
    this.users = [];
  }

  addUser(user) {
    this.users.push(user);
  }

  getUsers() {
    return this.users;
  }
}

// View Interface
class UserViewInterface {
  renderUsers(users) {}
  showAddUserForm() {}
  hideAddUserForm() {}
  setPresenter(presenter) {}
}

// View Implementation
class UserView extends UserViewInterface {
  constructor() {
    super();
    this.presenter = null;
    this.container = document.getElementById('users');
    this.addUserBtn = document.getElementById('addUserBtn');
    this.userNameInput = document.getElementById('userName');
    
    this.addUserBtn.addEventListener('click', () => {
      this.presenter.handleAddUser(this.userNameInput.value);
    });
  }

  setPresenter(presenter) {
    this.presenter = presenter;
  }

  renderUsers(users) {
    this.container.innerHTML = '';
    users.forEach(user => {
      const div = document.createElement('div');
      div.textContent = user.name;
      this.container.appendChild(div);
    });
  }
}

// Presenter
class UserPresenter {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    this.view.setPresenter(this);
  }

  loadUsers() {
    const users = this.model.getUsers();
    this.view.renderUsers(users);
  }

  handleAddUser(userName) {
    if (userName.trim() !== '') {
      this.model.addUser({ name: userName });
      this.loadUsers(); // Обновляем представление
    }
  }
}

// Использование
const userModel = new UserModel();
const userView = new UserView();
const userPresenter = new UserPresenter(userModel, userView);

// Загрузка начальных данных
userPresenter.loadUsers();
```

## Применение в современных фреймворках

- **Angular**: Часто используется в тестировании компонентов
- **React**: Может быть реализован с помощью хуков и контекста
- **Vue.js**: Возможна реализация через компоненты и store

## Сравнение с другими паттернами

- [[MVC]] - более тесная связь между View и Controller
- [[MVVM]] - автоматическая привязка данных между View и ViewModel
- [[Flux]] - односторонний поток данных
- [[Clean-Architecture]] - более глубокое разделение на слои

## Лучшие практики

1. **Создавать интерфейсы для View** - упрощает тестирование
2. **Изолировать бизнес-логику в Model** - Presenter должен быть тонким
3. **Избегать прямого доступа к DOM в Presenter** - вся работа с DOM во View
4. **Использовать Dependency Injection** - упрощает тестирование и поддержку

## Рекомендации по использованию

MVP особенно эффективен в приложениях с:
- Высокими требованиями к тестированию
- Сложной бизнес-логикой
- Необходимостью отделить UI от бизнес-логики
- Командной разработкой, где разные разработчики работают над разными слоями

## См. также

- [[MVC]]
- [[MVVM]]
- [[Flux]]
- [[Clean-Architecture]]
- [[State-менеджмент]]
