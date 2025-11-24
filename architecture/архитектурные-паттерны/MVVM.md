---
aliases: ["Model-View-ViewModel", "Архитектурный паттерн MVVM"]
tags: ["#architecture", "#frontend", "#mvvm", "#pattern"]
---

# MVVM (Model-View-ViewModel)

## Описание

MVVM (Model-View-ViewModel) - это архитектурный паттерн, разработанный Microsoft для упрощения разработки пользовательских интерфейсов с использованием платформы WPF и Silverlight. Паттерн позволяет отделить разработку графического интерфейса от бизнес-логики приложения, обеспечивая двустороннюю привязку данных между View и ViewModel.

## Компоненты

### Model (Модель)
- Отвечает за данные и бизнес-логику
- Не зависит от View и ViewModel
- Может уведомлять ViewModel об изменениях

### View (Представление)
- Отображает данные из ViewModel
- Использует привязку данных для синхронизации с ViewModel
- Обычно не содержит сложной логики
- Отвечает за визуальное представление и пользовательские интерфейсы

### ViewModel (Модель представления)
- Прокладка между View и Model
- Предоставляет данные для View в удобном для отображения формате
- Реализует команды для обработки пользовательских действий
- Обеспечивает уведомления об изменениях (INotifyPropertyChanged в .NET, Observable в других реализациях)

## Основные концепции

### Привязка данных (Data Binding)
- Автоматическая синхронизация между View и ViewModel
- Двусторонняя привязка: изменения в View автоматически отражаются в ViewModel и наоборот
- Уменьшает количество кода для обновления интерфейса

### Команды (Commands)
- Механизм для обработки пользовательских действий
- Изолирует логику обработки событий от View
- Позволяет легко тестировать пользовательские действия

### Уведомления об изменениях (Change Notifications)
- ViewModel уведомляет View об изменениях данных
- Обеспечивает актуальность отображения
- Реализуется через механизмы Observable

## Преимущества

- **Упрощение разработки интерфейса**: Разделение UI и логики
- **Улучшенная тестируемость**: ViewModel легко тестировать изолированно
- **Повторное использование кода**: ViewModel может использоваться разными View
- **Автоматическая синхронизация**: Привязка данных уменьшает boilerplate кода
- **Поддержка нескольких представлений**: Одна ViewModel может обслуживать несколько View

## Недостатки

- **Сложность для начинающих**: Требует понимания концепции привязки данных
- **Отладка сложных связей**: Сложнее отлаживать двустороннюю привязку
- **Производительность**: Частые уведомления об изменениях могут повлиять на производительность
- **Сложность в асинхронных сценариях**: Требует дополнительных подходов для обработки асинхронности

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

// Observable Helper
class Observable {
  constructor(value) {
    this.value = value;
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
    observer(this.value); // Уведомляем сразу
  }

  set(newValue) {
    this.value = newValue;
    this.notify();
  }

  get() {
    return this.value;
  }

  notify() {
    this.observers.forEach(observer => observer(this.value));
  }
}

// Command Helper
class Command {
  constructor(execute, canExecute = () => true) {
    this.execute = execute;
    this.canExecute = canExecute;
  }

  executeCommand(...args) {
    if (this.canExecute()) {
      this.execute(...args);
    }
  }
}

// ViewModel
class UserViewModel {
  constructor(model) {
    this.model = model;
    
    // Observable свойства
    this.users = new Observable([]);
    this.newUserName = new Observable('');
    
    // Commands
    this.addUserCommand = new Command(() => {
      const name = this.newUserName.get();
      if (name.trim() !== '') {
        this.model.addUser({ name: name });
        this.updateUsers();
        this.newUserName.set(''); // Очищаем поле ввода
      }
    }, () => this.newUserName.get().trim() !== '');
    
    // Инициализация
    this.updateUsers();
  }
  
  updateUsers() {
    this.users.set(this.model.getUsers());
  }
}

// View
class UserView {
  constructor(viewModel) {
    this.viewModel = viewModel;
    this.container = document.getElementById('users');
    this.input = document.getElementById('userName');
    this.addButton = document.getElementById('addUserBtn');
    
    // Привязка данных к ViewModel
    this.bindViewModel();
  }
  
  bindViewModel() {
    // Привязка списка пользователей
    this.viewModel.users.subscribe(users => {
      this.renderUsers(users);
    });
    
    // Привязка поля ввода
    this.viewModel.newUserName.subscribe(value => {
      this.input.value = value;
    });
    
    // Привязка события изменения поля ввода
    this.input.addEventListener('input', (e) => {
      this.viewModel.newUserName.set(e.target.value);
    });
    
    // Привязка кнопки
    this.addButton.addEventListener('click', () => {
      this.viewModel.addUserCommand.executeCommand();
    });
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

// Использование
const userModel = new UserModel();
const userViewModel = new UserViewModel(userModel);
const userView = new UserView(userViewModel);
```

## Применение в современных фреймворках

- **Angular**: Использует концепции MVVM через привязку данных и компоненты
- **Vue.js**: Встроенная поддержка MVVM с реактивными данными
- **React**: Может быть реализован с помощью библиотек (MobX, React Hooks)

## Сравнение с другими паттернами

- [[MVC]] - более тесная связь между компонентами
- [[MVP]] - явное управление Presenter, без двусторонней привязки
- [[Flux]] - односторонний поток данных
- [[Clean-Architecture]] - более глубокое разделение на слои

## Лучшие практики

1. **Использовать реактивные библиотеки** для упрощения привязки данных (RxJS, MobX, Vue реактивность)
2. **Изолировать бизнес-логику в Model** - ViewModel должна быть тонкой
3. **Использовать валидацию на уровне ViewModel** для удобства отображения ошибок
4. **Разделять сложные ViewModel** на более мелкие для лучшей поддержки

## Рекомендации по использованию

MVVM особенно эффективен в приложениях с:
- Сложными формами и пользовательскими интерфейсами
- Требованиями к частому обновлению отображения
- Необходимостью двусторонней привязки данных
- Высокими требованиями к тестируемости UI-логики

## См. также

- [[MVC]]
- [[MVP]]
- [[Flux]]
- [[Clean-Architecture]]
- [[State-менеджмент]]
