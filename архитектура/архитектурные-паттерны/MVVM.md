---
aliases: ["Model-View-ViewModel", "Модель-Представление-Модель-представления"]
tags: [architecture, frontend, pattern, mvvm]
---

# MVVM (Model-View-ViewModel)

## Обзор

MVVM (Model-View-ViewModel) - это архитектурный паттерн, разработанный Microsoft для упрощения разработки пользовательского интерфейса с использованием привязки данных. Паттерн особенно популярен в разработке на WPF, Silverlight и современных фреймворках JavaScript/TypeScript.

## Структура паттерна

MVVM состоит из трех основных компонентов:

### Model (Модель)
- **Отвечает за**: бизнес-логику, данные и правила валидации
- **Не зависит от**: View или ViewModel
- **Содержит**: данные приложения и бизнес-правила

```javascript
// Пример модели в MVVM
class UserModel {
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
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
}
```

### View (Представление)
- **Отвечает за**: отображение данных и пользовательский интерфейс
- **Связывается с**: ViewModel через привязку данных
- **Содержит минимальную логику**: только для отображения

```html
<!-- Пример представления с привязкой данных (используя шаблонизатор) -->
<div id="user-view">
  <h2>Данные пользователя</h2>
  <p>Имя: <span data-bind="text: name"></span></p>
  <p>Email: <span data-bind="text: email"></span></p>
  
  <form data-bind="submit: updateUser">
    <input type="text" 
           data-bind="value: nameInput" 
           placeholder="Имя">
    <input type="email" 
           data-bind="value: emailInput" 
           placeholder="Email">
    <button type="submit">Обновить</button>
  </form>
  
  <div data-bind="text: errorMessage, visible: hasError" style="color: red;"></div>
</div>
```

### ViewModel (Модель представления)
- **Отвечает за**: подготовку данных для View и обработку пользовательских действий
- **Содержит**: данные, доступные для View, и команды
- **Связывается с**: Model и View через привязку данных

```javascript
// Пример ViewModel с реактивными данными
class UserViewModel {
  constructor(model) {
    this.model = model;
    
    // Реактивные свойства (упрощенная реализация)
    this._name = model.name;
    this._email = model.email;
    this._nameInput = '';
    this._emailInput = '';
    this._errorMessage = '';
    this._hasError = false;
    
    // Создаем геттеры/сеттеры для реактивности
    this.createReactiveProperties();
  }

  createReactiveProperties() {
    // Упрощенная реализация реактивности
    Object.defineProperties(this, {
      name: {
        get: () => this._name,
        set: (value) => {
          this._name = value;
          this.updateView();
        }
      },
      email: {
        get: () => this._email,
        set: (value) => {
          this._email = value;
          this.updateView();
        }
      },
      nameInput: {
        get: () => this._nameInput,
        set: (value) => {
          this._nameInput = value;
        }
      },
      emailInput: {
        get: () => this._emailInput,
        set: (value) => {
          this._emailInput = value;
        }
      },
      errorMessage: {
        get: () => this._errorMessage,
        set: (value) => {
          this._errorMessage = value;
          this.hasError = !!value;
        }
      },
      hasError: {
        get: () => this._hasError,
        set: (value) => {
          this._hasError = value;
        }
      }
    });
  }

  updateUser() {
    let hasChanges = false;
    
    if (this.nameInput) {
      const success = this.model.updateName(this.nameInput);
      if (!success) {
        this.errorMessage = 'Имя не может быть пустым';
        return;
      }
      this.name = this.model.name;
      hasChanges = true;
    }
    
    if (this.emailInput) {
      const success = this.model.updateEmail(this.emailInput);
      if (!success) {
        this.errorMessage = 'Некорректный формат email';
        return;
      }
      this.email = this.model.email;
      hasChanges = true;
    }
    
    if (hasChanges) {
      this.nameInput = '';
      this.emailInput = '';
      this.errorMessage = '';
    }
  }

  updateView() {
    // В реальном приложении это будет автоматически обновляться через привязку данных
    const nameElement = document.querySelector('[data-bind="text: name"]');
    const emailElement = document.querySelector('[data-bind="text: email"]');
    const nameInputElement = document.querySelector('input[data-bind="value: nameInput"]');
    const emailInputElement = document.querySelector('input[data-bind="value: emailInput"]');
    const errorElement = document.querySelector('[data-bind="text: errorMessage"]');
    
    if (nameElement) nameElement.textContent = this.name;
    if (emailElement) emailElement.textContent = this.email;
    if (errorElement) errorElement.textContent = this.errorMessage;
  }
}

// Пример с использованием современного фреймворка (Vue.js)
const UserVueComponent = {
  template: `
    <div id="user-view">
      <h2>Данные пользователя</h2>
      <p>Имя: {{ name }}</p>
      <p>Email: {{ email }}</p>
      
      <form @submit.prevent="updateUser">
        <input v-model="nameInput" type="text" placeholder="Имя">
        <input v-model="emailInput" type="email" placeholder="Email">
        <button type="submit">Обновить</button>
      </form>
      
      <div v-if="hasError" style="color: red;">{{ errorMessage }}</div>
    </div>
  `,
  data() {
    return {
      name: 'Иван Петров',
      email: 'ivan@example.com',
      nameInput: '',
      emailInput: '',
      errorMessage: '',
      hasError: false
    }
  },
  methods: {
    updateUser() {
      // Логика обновления пользователя
      if (this.nameInput) {
        if (this.nameInput.trim() === '') {
          this.errorMessage = 'Имя не может быть пустым';
          this.hasError = true;
          return;
        }
        this.name = this.nameInput;
        this.nameInput = '';
      }
      
      if (this.emailInput) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(this.emailInput)) {
          this.errorMessage = 'Некорректный формат email';
          this.hasError = true;
          return;
        }
        this.email = this.emailInput;
        this.emailInput = '';
      }
      
      this.errorMessage = '';
      this.hasError = false;
    }
  }
}
```

## Преимущества MVVM

1. **Привязка данных**: автоматическое обновление интерфейса при изменении данных
2. **Высокая тестируемость**: ViewModel можно тестировать без UI
3. **Четкое разделение ответственности**: каждый компонент имеет конкретную роль
4. **Упрощение разработки интерфейса**: разработчики могут работать независимо

## Недостатки MVVM

1. **Сложность понимания**: требует понимания привязки данных
2. **Проблемы с производительностью**: при большом количестве привязок
3. **Сложность отладки**: сложнее отследить поток данных

## Применение в российских реалиях 2025

MVVM особенно популярен в российской разработке:

- **Frontend-фреймворках**: Vue.js, Angular, Knockout.js
- **Desktop-приложениях**: WPF с C#
- **Mobile-приложениях**: Xamarin, особенно в корпоративной среде
- **Современных веб-приложениях**: где важна реактивность и связывание данных

В условиях импортозамещения и перехода на отечественные решения MVVM остается важным паттерном для разработки пользовательских интерфейсов.

## Сравнение с другими паттернами

- [[MVC]]: MVVM имеет более тесную интеграцию между View и ViewModel
- [[MVP]]: MVVM использует привязку данных вместо явного управления View
- [[Flux]]: предлагает другой подход к управлению состоянием
- [[Чистая-архитектура]]: более сложная, но гибкая архитектура

## Практические рекомендации

1. **Используйте готовые фреймворки**: для привязки данных (Vue, Angular, React с MobX)
2. **Избегайте сложной логики в ViewModel**: бизнес-логика должна быть в Model
3. **Разделяйте ViewModel по функциональности**: не создавайте слишком большие классы
4. **Используйте типизацию**: особенно в TypeScript для лучшей поддержки

## Заключение

MVVM - мощный паттерн для разработки интерфейсов с привязкой данных. Особенно эффективен в современных фреймворках, где привязка данных реализована на уровне платформы.

## См. также

- [[MVC]]
- [[MVP]]
- [[Flux]]
- [[Чистая-архитектура]]
- [[Архитектурные-паттерны]]