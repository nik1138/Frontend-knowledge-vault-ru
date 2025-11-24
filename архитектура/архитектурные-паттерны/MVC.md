---
aliases: ["Model-View-Controller", "Модель-Представление-Контроллер"]
tags: [architecture, frontend, pattern, mvc]
---

# MVC (Model-View-Controller)

## Обзор

MVC (Model-View-Controller) - это один из самых известных архитектурных паттернов, разработанный для разработки приложений с четким разделением ответственности между бизнес-логикой, пользовательским интерфейсом и управлением пользовательскими действиями. Паттерн был впервые представлен в 1979 году и до сих пор широко используется в современной разработке.

## Структура паттерна

MVC состоит из трех основных компонентов:

### Model (Модель)
- **Отвечает за**: бизнес-логику, данные и правила валидации
- **Хранит**: данные приложения и состояние
- **Уведомляет**: View об изменениях данных (в пассивной модели) или предоставляет данные по запросу (в активной модели)

```javascript
// Пример модели в JavaScript
class UserModel {
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this._observers = [];
  }

  updateName(newName) {
    this.name = newName;
    this.notifyObservers();
  }

  updateEmail(newEmail) {
    this.validateEmail(newEmail);
    this.email = newEmail;
    this.notifyObservers();
  }

  validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      throw new Error('Некорректный формат email');
    }
  }

  subscribe(observer) {
    this._observers.push(observer);
  }

  notifyObservers() {
    this._observers.forEach(observer => observer.update());
  }
}
```

### View (Представление)
- **Отвечает за**: отображение данных модели пользователю
- **Интерпретирует**: команды пользователя (клик, ввод и т.д.)
- **Получает данные**: из Model напрямую или через Controller

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
</div>
```

### Controller (Контроллер)
- **Отвечает за**: обработку пользовательского ввода и взаимодействие между Model и View
- **Преобразует**: команды пользователя в действия модели и обновления представления

```javascript
// Пример контроллера
class UserController {
  constructor(model, view) {
    this.model = model;
    this.view = view;
    this.initEventListeners();
  }

  initEventListeners() {
    const form = document.getElementById('user-form');
    form.addEventListener('submit', (e) => {
      e.preventDefault();
      const nameInput = document.getElementById('name-input');
      const emailInput = document.getElementById('email-input');
      
      try {
        if (nameInput.value) {
          this.model.updateName(nameInput.value);
        }
        if (emailInput.value) {
          this.model.updateEmail(emailInput.value);
        }
        
        // Обновляем представление
        this.updateView();
      } catch (error) {
        console.error('Ошибка обновления пользователя:', error.message);
      }
    });

    // Подписываемся на изменения модели
    this.model.subscribe(this);
  }

  update() {
    this.updateView();
  }

  updateView() {
    document.getElementById('user-name').textContent = this.model.name;
    document.getElementById('user-email').textContent = this.model.email;
    document.getElementById('name-input').value = '';
    document.getElementById('email-input').value = '';
  }
}

// Использование
const userModel = new UserModel('Иван Петров', 'ivan@example.com');
const userView = document.getElementById('user-view');
const userController = new UserController(userModel, userView);

// Инициализация начального состояния
userController.updateView();
```

## Преимущества MVC

1. **Разделение ответственности**: каждый компонент имеет четко определенную роль
2. **Повторное использование кода**: Model может использоваться несколькими View
3. **Легкость тестирования**: компоненты могут тестироваться изолированно
4. **Поддержка параллельной разработки**: разработчики могут работать над разными слоями одновременно

## Недостатки MVC

1. **Сложность для маленьких приложений**: может быть избыточным
2. **Тесная связанность**: View и Controller могут быть тесно связаны
3. **Сложность в реализации**: требует хорошего понимания паттерна

## Применение в российских реалиях 2025

В российской разработке MVC продолжает использоваться в следующих контекстах:

- **Классические веб-приложения**: особенно на .NET, Ruby on Rails, Django
- **Государственные проекты**: где важна структурированность и документируемость
- **Крупные корпоративные системы**: где важна поддержка и масштабируемость

В условиях санкций и импортозамещения MVC остается популярным выбором из-за своей стабильности и поддержки в отечественных фреймворках.

## Сравнение с другими паттернами

- [[MVP]] и [[MVVM]] появились как улучшения MVC для решения проблем тестирования и связанности
- [[Flux]] и [[Чистая-архитектура]] предлагают альтернативные подходы к управлению состоянием

## Практические рекомендации

1. **Избегайте логики в View**: все бизнес-правила должны быть в Model
2. **Минимизируйте зависимость View от Model**: используйте Controller для преобразования данных
3. **Тестируйте компоненты изолированно**: Model должна быть тестируемой без View и Controller

## Заключение

MVC остается важным архитектурным паттерном, особенно для традиционных веб-приложений. Понимание его принципов важно для разработчиков, работающих с классическими фреймворками и системами.

## См. также

- [[MVP]]
- [[MVVM]]
- [[Flux]]
- [[Чистая-архитектура]]
- [[Архитектурные-паттерны]]