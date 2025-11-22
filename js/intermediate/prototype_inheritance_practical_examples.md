---
tags: [javascript, frontend, intermediate, prototypes, oop]
aliases: [прототипное наследование в js для фронтенда, практические примеры прототипов]
---

# Прототипное наследование в JavaScript - Практические примеры для Frontend разработки

## Введение

Прототипное наследование - это фундаментальная концепция JavaScript, которая лежит в основе объектно-ориентированного программирования в языке. В этой статье рассмотрим практические примеры использования прототипного наследования с акцентом на задачи фронтенд разработки.

## Что такое прототипное наследование?

В JavaScript каждый объект имеет прототип, который сам является объектом. Объект наследует свойства и методы своего прототипа. Это позволяет создавать цепочки наследования и делиться функциональностью между объектами.

## Практический пример: Создание компонентной системы

```javascript
// Базовый класс компонента
function BaseComponent(element) {
  this.element = element;
  this.state = {};
  this.listeners = [];
}

BaseComponent.prototype.setState = function(newState) {
  Object.assign(this.state, newState);
  this.render();
};

BaseComponent.prototype.render = function() {
  // Базовая реализация, будет переопределена в наследниках
  console.log('Рендер компонента');
};

BaseComponent.prototype.addEventListener = function(event, handler) {
  this.element.addEventListener(event, handler);
  this.listeners.push({ event, handler });
};

BaseComponent.prototype.destroy = function() {
  // Удаление всех слушателей событий
  this.listeners.forEach(({ event, handler }) => {
    this.element.removeEventListener(event, handler);
  });
  this.listeners = [];
};

// Наследование: компонент кнопки
function ButtonComponent(element, text) {
  BaseComponent.call(this, element);
  this.text = text;
}

// Установка прототипного наследования
ButtonComponent.prototype = Object.create(BaseComponent.prototype);
ButtonComponent.prototype.constructor = ButtonComponent;

// Переопределение метода render
ButtonComponent.prototype.render = function() {
  this.element.textContent = this.text;
  this.element.classList.add('btn');
  
  if (this.state.disabled) {
    this.element.disabled = true;
    this.element.classList.add('btn-disabled');
  } else {
    this.element.disabled = false;
    this.element.classList.remove('btn-disabled');
  }
};

// Добавление специфичного метода
ButtonComponent.prototype.setText = function(newText) {
  this.text = newText;
  this.render();
};

// Использование компонента
const buttonElement = document.getElementById('my-button');
const button = new ButtonComponent(buttonElement, 'Нажми меня');

button.addEventListener('click', () => {
  button.setState({ disabled: true });
  setTimeout(() => {
    button.setState({ disabled: false });
  }, 2000);
});
```

## Практический пример: Система валидации форм

```javascript
// Базовый валидатор
function BaseValidator() {
  this.errors = [];
}

BaseValidator.prototype.validate = function(value) {
  // Базовая валидация
  this.errors = [];
  return true;
};

BaseValidator.prototype.addError = function(message) {
  this.errors.push(message);
};

BaseValidator.prototype.getErrors = function() {
  return this.errors;
};

BaseValidator.prototype.isValid = function() {
  return this.errors.length === 0;
};

// Валидатор email
function EmailValidator() {
  BaseValidator.call(this);
}

EmailValidator.prototype = Object.create(BaseValidator.prototype);
EmailValidator.prototype.constructor = EmailValidator;

EmailValidator.prototype.validate = function(email) {
  BaseValidator.prototype.validate.call(this);
  
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  
  if (!emailRegex.test(email)) {
    this.addError('Некорректный email адрес');
    return false;
  }
  
  return true;
};

// Валидатор пароля
function PasswordValidator(minLength = 8) {
  BaseValidator.call(this);
  this.minLength = minLength;
}

PasswordValidator.prototype = Object.create(BaseValidator.prototype);
PasswordValidator.prototype.constructor = PasswordValidator;

PasswordValidator.prototype.validate = function(password) {
  BaseValidator.prototype.validate.call(this);
  
  if (password.length < this.minLength) {
    this.addError(`Пароль должен содержать не менее ${this.minLength} символов`);
    return false;
  }
  
  if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password)) {
    this.addError('Пароль должен содержать заглавную букву, строчную букву и цифру');
    return false;
  }
  
  return true;
};

// Композитный валидатор
function CompositeValidator() {
  BaseValidator.call(this);
  this.validators = [];
}

CompositeValidator.prototype = Object.create(BaseValidator.prototype);
CompositeValidator.prototype.constructor = CompositeValidator;

CompositeValidator.prototype.addValidator = function(validator) {
  this.validators.push(validator);
};

CompositeValidator.prototype.validate = function(value) {
  BaseValidator.prototype.validate.call(this);
  
  let isValid = true;
  
  this.validators.forEach(validator => {
    if (!validator.validate(value)) {
      isValid = false;
      this.errors.push(...validator.getErrors());
    }
  });
  
  return isValid;
};

// Использование системы валидации
const emailValidator = new EmailValidator();
const passwordValidator = new PasswordValidator(8);
const formValidator = new CompositeValidator();

formValidator.addValidator(emailValidator);
formValidator.addValidator(passwordValidator);

// Проверка данных формы
const formData = {
  email: 'user@example.com',
  password: 'MySecurePassword123'
};

const isEmailValid = emailValidator.validate(formData.email);
const isPasswordValid = passwordValidator.validate(formData.password);
const isFormValid = formValidator.validate(formData);

console.log('Email валидный:', isEmailValid);
console.log('Пароль валидный:', isPasswordValid);
console.log('Форма валидная:', isFormValid);
```

## Практический пример: Система UI виджетов

```javascript
// Базовый класс виджета
function BaseWidget(options = {}) {
  this.options = {
    container: document.body,
    visible: true,
    ...options
  };
  
  this.element = null;
  this.isVisible = this.options.visible;
  
  this.init();
}

BaseWidget.prototype.init = function() {
  this.createElement();
  this.render();
  this.bindEvents();
};

BaseWidget.prototype.createElement = function() {
  this.element = document.createElement('div');
  this.element.className = 'widget';
};

BaseWidget.prototype.render = function() {
  if (this.element) {
    this.element.style.display = this.isVisible ? 'block' : 'none';
  }
};

BaseWidget.prototype.show = function() {
  this.isVisible = true;
  this.render();
};

BaseWidget.prototype.hide = function() {
  this.isVisible = false;
  this.render();
};

BaseWidget.prototype.destroy = function() {
  if (this.element && this.element.parentNode) {
    this.element.parentNode.removeChild(this.element);
  }
};

BaseWidget.prototype.bindEvents = function() {
  // Базовая привязка событий
};

// Виджет прогресс-бара
function ProgressBarWidget(container, options = {}) {
  BaseWidget.call(this, { container, ...options });
}

ProgressBarWidget.prototype = Object.create(BaseWidget.prototype);
ProgressBarWidget.prototype.constructor = ProgressBarWidget;

ProgressBarWidget.prototype.createElement = function() {
  this.element = document.createElement('div');
  this.element.className = 'progress-bar';
  
  this.bar = document.createElement('div');
  this.bar.className = 'progress-bar-fill';
  
  this.label = document.createElement('span');
  this.label.className = 'progress-bar-label';
  
  this.element.appendChild(this.bar);
  this.element.appendChild(this.label);
};

ProgressBarWidget.prototype.render = function() {
  BaseWidget.prototype.render.call(this);
  
  const percent = this.options.percent || 0;
  this.bar.style.width = `${percent}%`;
  this.label.textContent = `${percent}%`;
};

ProgressBarWidget.prototype.setProgress = function(percent) {
  this.options.percent = Math.min(100, Math.max(0, percent));
  this.render();
};

// Виджет уведомлений
function NotificationWidget(container, options = {}) {
  BaseWidget.call(this, { container, ...options });
}

NotificationWidget.prototype = Object.create(BaseWidget.prototype);
NotificationWidget.prototype.constructor = NotificationWidget;

NotificationWidget.prototype.createElement = function() {
  this.element = document.createElement('div');
  this.element.className = 'notification';
};

NotificationWidget.prototype.showNotification = function(message, type = 'info', duration = 3000) {
  this.element.textContent = message;
  this.element.className = `notification notification-${type}`;
  this.show();
  
  if (duration > 0) {
    setTimeout(() => {
      this.hide();
    }, duration);
  }
};

// Использование виджетов
const progressBar = new ProgressBarWidget(document.getElementById('progress-container'));
const notification = new NotificationWidget();

progressBar.setProgress(50);

// Показать уведомление
notification.showNotification('Операция завершена успешно!', 'success');
```

## Практический пример: Система управления данными

```javascript
// Базовый класс хранилища данных
function BaseStore(initialData = {}) {
  this.data = { ...initialData };
  this.listeners = [];
}

BaseStore.prototype.get = function(key) {
  return this.data[key];
};

BaseStore.prototype.set = function(key, value) {
  this.data[key] = value;
  this.notifyListeners(key, value);
};

BaseStore.prototype.getAll = function() {
  return { ...this.data };
};

BaseStore.prototype.subscribe = function(callback) {
  this.listeners.push(callback);
};

BaseStore.prototype.unsubscribe = function(callback) {
  const index = this.listeners.indexOf(callback);
  if (index > -1) {
    this.listeners.splice(index, 1);
  }
};

BaseStore.prototype.notifyListeners = function(key, value) {
  this.listeners.forEach(callback => callback(key, value, this.data));
};

// Хранилище пользовательских данных
function UserStore(initialData = {}) {
  BaseStore.call(this, {
    user: null,
    isLoggedIn: false,
    preferences: {},
    ...initialData
  });
}

UserStore.prototype = Object.create(BaseStore.prototype);
UserStore.prototype.constructor = UserStore;

UserStore.prototype.login = async function(credentials) {
  try {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });
    
    if (response.ok) {
      const userData = await response.json();
      this.set('user', userData);
      this.set('isLoggedIn', true);
      return true;
    }
    
    return false;
  } catch (error) {
    console.error('Ошибка входа:', error);
    return false;
  }
};

UserStore.prototype.logout = function() {
  this.set('user', null);
  this.set('isLoggedIn', false);
};

UserStore.prototype.updatePreferences = function(newPreferences) {
  const currentPrefs = this.get('preferences') || {};
  const updatedPrefs = { ...currentPrefs, ...newPreferences };
  this.set('preferences', updatedPrefs);
};

// Использование хранилища
const userStore = new UserStore();

// Подписка на изменения
userStore.subscribe((key, value, allData) => {
  console.log(`Данные изменились: ${key} =`, value);
  
  if (key === 'isLoggedIn') {
    const loginStatusElement = document.getElementById('login-status');
    if (loginStatusElement) {
      loginStatusElement.textContent = value ? 'Вход выполнен' : 'Необходим вход';
    }
  }
});

// Пример использования
// userStore.login({ email: 'user@example.com', password: 'password' });
```

## Современный подход с ES6 классами

Хотя в примерах выше используется функциональный подход с прототипами, современный JavaScript предоставляет синтаксис классов, который также использует прототипное наследование под капотом:

```javascript
// Базовый компонент с использованием ES6 классов
class BaseComponent {
  constructor(element) {
    this.element = element;
    this.state = {};
    this.listeners = [];
  }
  
  setState(newState) {
    Object.assign(this.state, newState);
    this.render();
  }
  
  render() {
    // Базовая реализация
  }
  
  addEventListener(event, handler) {
    this.element.addEventListener(event, handler);
    this.listeners.push({ event, handler });
  }
  
  destroy() {
    this.listeners.forEach(({ event, handler }) => {
      this.element.removeEventListener(event, handler);
    });
    this.listeners = [];
  }
}

// Наследование с использованием ES6 классов
class ModalComponent extends BaseComponent {
  constructor(element, options = {}) {
    super(element);
    this.options = {
      closable: true,
      backdrop: true,
      ...options
    };
  }
  
  render() {
    this.element.classList.add('modal');
    
    if (this.state.isOpen) {
      this.element.style.display = 'block';
      if (this.options.backdrop) {
        document.body.style.overflow = 'hidden';
      }
    } else {
      this.element.style.display = 'none';
      document.body.style.overflow = '';
    }
  }
  
  open() {
    this.setState({ isOpen: true });
  }
  
  close() {
    this.setState({ isOpen: false });
  }
  
  bindEvents() {
    if (this.options.closable) {
      const closeBtn = this.element.querySelector('.modal-close');
      if (closeBtn) {
        this.addEventListener('click', (e) => {
          if (e.target === closeBtn || e.target.classList.contains('modal-backdrop')) {
            this.close();
          }
        });
      }
    }
  }
}

// Использование
const modalElement = document.getElementById('my-modal');
const modal = new ModalComponent(modalElement);
```

## Лучшие практики при использовании прототипного наследования в фронтенд разработке

### 1. Используйте наследование для расширения функциональности

```javascript
// Плохо - дублирование кода
function Button1() { /* ... */ }
function Button2() { /* ... */ }
function Button3() { /* ... */ }

// Хорошо - наследование
function BaseButton() { /* общая функциональность */ }
function PrimaryButton() { BaseButton.call(this); }
PrimaryButton.prototype = Object.create(BaseButton.prototype);

function SecondaryButton() { BaseButton.call(this); }
SecondaryButton.prototype = Object.create(BaseButton.prototype);
```

### 2. Избегайте глубоких иерархий наследования

```javascript
// Лучше использовать композицию
function ComponentWithState() {
  this.stateManager = new StateManager();
  this.eventManager = new EventManager();
}
```

## Заключение

Прототипное наследование в JavaScript фронтенд разработке позволяет создавать гибкие, расширяемые и переиспользуемые компоненты. Понимание этой концепции помогает писать более эффективный и поддерживаемый код, особенно при создании сложных интерфейсов и архитектурных решений.

## См. также

- [[closure_practical_examples.md]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
- [[Объектно-ориентированное программирование в JavaScript]]