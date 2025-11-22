---
aliases: [Медиатор JavaScript, Реализация медиатора, Паттерн медиатор JS]
tags: [javascript, паттерны-проектирования, медиатор, фронтенд]
---

# Реализация паттерна Медиатор в JavaScript

## Обзор

Реализация паттерна Медиатор в JavaScript позволяет создать гибкую архитектуру, в которой объекты взаимодействуют через центральный объект-посредник. Это особенно полезно во фронтенд-разработке для управления взаимодействием между компонентами.

## Базовая реализация

### Простой пример

```javascript
// Медиатор
class ChatRoomMediator {
  constructor() {
    this.users = [];
  }

  addUser(user) {
    this.users.push(user);
    user.setMediator(this);
  }

  sendMessage(message, sender) {
    this.users.forEach(user => {
      if (user !== sender) {
        user.receive(message, sender);
      }
    });
  }
}

// Коллега
class User {
  constructor(name) {
    this.name = name;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
  }

  send(message) {
    console.log(`${this.name} sends: ${message}`);
    this.mediator.sendMessage(message, this);
  }

  receive(message, sender) {
    console.log(`${this.name} received message from ${sender.name}: ${message}`);
  }
}

// Использование
const chatRoom = new ChatRoomMediator();
const user1 = new User('Alice');
const user2 = new User('Bob');
const user3 = new User('Charlie');

chatRoom.addUser(user1);
chatRoom.addUser(user2);
chatRoom.addUser(user3);

user1.send('Hello everyone!');
```

## Продвинутая реализация с событиями

### Использование события для управления взаимодействием

```javascript
// Медиатор с поддержкой событий
class EventMediator {
  constructor() {
    this.subscribers = {};
  }

  subscribe(event, callback) {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
  }

  unsubscribe(event, callback) {
    if (this.subscribers[event]) {
      this.subscribers[event] = this.subscribers[event].filter(cb => cb !== callback);
    }
  }

  publish(event, data) {
    if (this.subscribers[event]) {
      this.subscribers[event].forEach(callback => callback(data));
    }
  }
}

// Пример использования
const mediator = new EventMediator();

// Компоненты
const button = {
  click() {
    console.log('Button clicked');
    mediator.publish('button:click', { source: 'button' });
  }
};

const modal = {
  init() {
    mediator.subscribe('button:click', (data) => {
      console.log(`Modal opened due to ${data.source} event`);
      // Открытие модального окна
    });
  }
};

const notification = {
  init() {
    mediator.subscribe('button:click', (data) => {
      console.log(`Notification shown due to ${data.source} event`);
      // Показ уведомления
    });
  }
};

// Инициализация
modal.init();
notification.init();

// Симуляция клика
button.click();
```

## Практические примеры

### Управление формой с помощью медиатора

```javascript
class FormMediator {
  constructor() {
    this.components = {};
  }

  register(name, component) {
    this.components[name] = component;
    component.setMediator(this);
  }

  notify(sender, event, data) {
    switch (event) {
      case 'field:change':
        this.handleFieldChange(sender, data);
        break;
      case 'form:submit':
        this.handleFormSubmit(sender, data);
        break;
      case 'validation:complete':
        this.handleValidationComplete(sender, data);
        break;
    }
  }

  handleFieldChange(sender, data) {
    // Валидация поля
    const isValid = this.validateField(data.fieldName, data.value);
    
    // Обновление других компонентов
    this.components.status.updateFieldStatus(data.fieldName, isValid);
    
    // Проверка готовности формы
    if (this.isFormValid()) {
      this.components.submitButton.enable();
    } else {
      this.components.submitButton.disable();
    }
  }

  handleFormSubmit(sender, data) {
    // Отправка данных формы
    console.log('Form submitted with data:', data);
  }

  validateField(fieldName, value) {
    // Простая валидация
    return value && value.trim().length > 0;
  }

  isFormValid() {
    return Object.keys(this.components).every(key => {
      const component = this.components[key];
      return !component.requiresValidation || component.isValid();
    });
  }

  handleValidationComplete(sender, data) {
    // Обработка завершения валидации
    console.log('Validation complete for:', data);
  }
}

// Компонент поля ввода
class InputField {
  constructor(name, element) {
    this.name = name;
    this.element = element;
    this.mediator = null;
  }

  setMediator(mediator) {
    this.mediator = mediator;
    this.element.addEventListener('input', (e) => {
      this.mediator.notify(this, 'field:change', {
        fieldName: this.name,
        value: e.target.value
      });
    });
  }

  getValue() {
    return this.element.value;
  }

  setValue(value) {
    this.element.value = value;
  }
}

// Компонент кнопки отправки
class SubmitButton {
  constructor(element) {
    this.element = element;
    this.isEnabled = false;
  }

  enable() {
    this.element.disabled = false;
    this.isEnabled = true;
  }

  disable() {
    this.element.disabled = true;
    this.isEnabled = false;
  }

  setMediator(mediator) {
    this.mediator = mediator;
    this.element.addEventListener('click', () => {
      if (this.isEnabled) {
        this.mediator.notify(this, 'form:submit', this.getFormData());
      }
    });
  }

  getFormData() {
    // Получение данных формы через медиатор
    const formData = {};
    Object.keys(this.mediator.components).forEach(key => {
      const component = this.mediator.components[key];
      if (component.getValue) {
        formData[key] = component.getValue();
      }
    });
    return formData;
  }
}

// Использование
const formMediator = new FormMediator();

const nameInput = new InputField('name', document.getElementById('name'));
const emailInput = new InputField('email', document.getElementById('email'));
const submitButton = new SubmitButton(document.getElementById('submit'));

formMediator.register('name', nameInput);
formMediator.register('email', emailInput);
formMediator.register('submitButton', submitButton);
```

## Преимущества использования медиатора в JavaScript

- **Снижение связанности**: Компоненты не знают друг о друге напрямую.
- **Гибкость**: Легко добавлять новые компоненты или изменять логику взаимодействия.
- **Тестируемость**: Компоненты легче тестировать изолированно.

## Возможные проблемы

- **Сложность медиатора**: При большом количестве компонентов медиатор может стать сложным.
- **Производительность**: Дополнительный уровень абстракции может повлиять на производительность.

## Заключение

Паттерн Медиатор в JavaScript позволяет создать гибкую и легко поддерживаемую архитектуру приложения. Он особенно полезен в сложных интерфейсах, где множество компонентов должны взаимодействовать друг с другом.

Для изучения реализации в других фреймворках см.:
- [[Реализация-в-React]]
- [[Реализация-в-Vue]]

Для практических примеров см. [[Примеры-использования]].
