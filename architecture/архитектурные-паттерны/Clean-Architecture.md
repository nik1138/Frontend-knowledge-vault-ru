---
aliases: ["Чистая архитектура", "Архитектурный паттерн Clean Architecture"]
tags: ["#architecture", "#frontend", "#clean-architecture", "#pattern", "#backend"]
---

# Clean Architecture (Чистая архитектура)

## Описание

Clean Architecture (Чистая архитектура) - это архитектурный паттерн, предложенный Робертом Мартином (Uncle Bob), который акцентирует внимание на отделении бизнес-логики от деталей реализации. Этот паттерн создает архитектуру, которая является:
- Тестируемой
- Независимой от UI
- Независимой от базы данных
- Независимой от любых агентов внешнего воздействия

## Основные принципы

### Соблюдение Dependency Rule (Правило зависимости)
- Внутренние слои не должны зависеть от внешних слоев
- Зависимости всегда направлены внутрь
- Внешние слои могут зависеть от внутренних, но не наоборот

### Независимость от фреймворков
- Архитектура не зависит от каких-либо фреймворков
- Приложение может быть построено без использования фреймворков
- Фреймворки рассматриваются как инструменты, а не как основа архитектуры

### Тестируемость
- Бизнес-правила могут быть протестированы без UI, базы данных, веб-сервера или других внешних элементов

### Независимость от UI
- UI может быть изменен без изменения внутренних слоев
- Логика приложения не зависит от представления

### Независимость от базы данных
- Бизнес-правила не привязаны к конкретной базе данных
- Можно легко заменить одну СУБД на другую

## Слои Clean Architecture

### Entities (Сущности)
- Наиболее внутренний слой
- Содержит бизнес-объекты и бизнес-логику
- Представляют наиболее общие правила, которые важны независимо от приложения

### Use Cases (Сценарии использования)
- Содержат прикладную бизнес-логику
- Координируют поток выполнения в приложении
- Не зависят от внешних слоев (UI, базы данных, внешних сервисов)

### Interface Adapters (Адаптеры интерфейсов)
- Преобразуют данные из формата, удобного для Use Cases и Entities, в формат, удобный для внешних слоев
- Включают контроллеры, представления, фасады сервисов и т.д.

### Frameworks and Drivers (Фреймворки и драйверы)
- Наиболее внешний слой
- Включает базы данных, веб-фреймворки, UI и т.д.
- Все остальные слои не зависят от этого слоя

## Структура для фронтенд-приложений

Хотя Clean Architecture изначально была разработана для бэкенда, её можно адаптировать для фронтенд-разработки:

### Domain Layer (Слой домена)
- Содержит бизнес-сущности и интерфейсы репозиториев
- Определяет бизнес-правила и логику
- Не зависит от фреймворков и внешних библиотек

### Data Layer (Слой данных)
- Реализует интерфейсы репозиториев
- Содержит логику работы с API, базой данных, кэшем
- Преобразует внешние данные в доменные сущности

### Presentation Layer (Слой презентации)
- Содержит UI-логику (контроллеры, презентеры, ViewModels)
- Не знает о деталях реализации других слоев
- Взаимодействует с Use Cases через интерфейсы

### UI Layer (Слой пользовательского интерфейса)
- Компоненты представления (React, Vue, Angular и т.д.)
- Отвечает за отображение данных и обработку пользовательских событий

## Преимущества

- **Тестируемость**: Бизнес-логика легко тестируется изолированно
- **Независимость от фреймворков**: Легко заменить один фреймворк на другой
- **Независимость от UI**: Можно легко изменить интерфейс без изменения логики
- **Независимость от базы данных**: Можно легко переключиться между разными СУБД
- **Поддерживаемость**: Четкое разделение ответственности облегчает сопровождение
- **Масштабируемость**: Архитектура позволяет легко добавлять новые функции

## Недостатки

- **Сложность для простых приложений**: Может быть избыточной для небольших проектов
- **Большое количество кода**: Требует написания большого количества интерфейсов и абстракций
- **Кривая обучения**: Требует времени для понимания концепции
- **Производительность**: Дополнительные уровни абстракции могут влиять на производительность

## Пример реализации на JavaScript

```javascript
// Domain Layer - Entities
class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }
}

// Domain Layer - Repository Interface
class UserRepositoryInterface {
  async getUser(id) {
    throw new Error('Method not implemented');
  }

  async getAllUsers() {
    throw new Error('Method not implemented');
  }

  async saveUser(user) {
    throw new Error('Method not implemented');
  }

  async deleteUser(id) {
    throw new Error('Method not implemented');
  }
}

// Domain Layer - Use Cases
class GetUserUseCase {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async execute(userId) {
    if (!userId) {
      throw new Error('User ID is required');
    }
    
    const user = await this.userRepository.getUser(userId);
    if (!user) {
      throw new Error('User not found');
    }
    
    return user;
  }
}

class GetAllUsersUseCase {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async execute() {
    return await this.userRepository.getAllUsers();
  }
}

class CreateUserUseCase {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async execute(userData) {
    if (!userData.name || !userData.email) {
      throw new Error('Name and email are required');
    }

    const user = new User(Date.now(), userData.name, userData.email);
    return await this.userRepository.saveUser(user);
  }
}

// Data Layer - Repository Implementation
class ApiUserRepository extends UserRepositoryInterface {
  constructor(apiClient) {
    super();
    this.apiClient = apiClient;
  }

  async getUser(id) {
    try {
      const response = await this.apiClient.get(`/users/${id}`);
      return new User(response.data.id, response.data.name, response.data.email);
    } catch (error) {
      console.error('Error fetching user:', error);
      return null;
    }
  }

  async getAllUsers() {
    try {
      const response = await this.apiClient.get('/users');
      return response.data.map(user => 
        new User(user.id, user.name, user.email)
      );
    } catch (error) {
      console.error('Error fetching users:', error);
      return [];
    }
  }

  async saveUser(user) {
    try {
      const response = await this.apiClient.post('/users', {
        name: user.name,
        email: user.email
      });
      return new User(response.data.id, response.data.name, response.data.email);
    } catch (error) {
      console.error('Error saving user:', error);
      throw error;
    }
  }

  async deleteUser(id) {
    try {
      await this.apiClient.delete(`/users/${id}`);
      return true;
    } catch (error) {
      console.error('Error deleting user:', error);
      return false;
    }
  }
}

// Data Layer - API Client
class HttpClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async get(endpoint) {
    const response = await fetch(`${this.baseURL}${endpoint}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return { data: await response.json() };
  }

  async post(endpoint, data) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(data),
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return { data: await response.json() };
  }

  async delete(endpoint) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      method: 'DELETE',
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return { data: {} };
  }
}

// Presentation Layer - View Model
class UserViewModel {
  constructor(getUserUseCase, getAllUsersUseCase, createUserUseCase) {
    this.getUserUseCase = getUserUseCase;
    this.getAllUsersUseCase = getAllUsersUseCase;
    this.createUserUseCase = createUserUseCase;
    this.users = [];
    this.currentUser = null;
    this.loading = false;
    this.error = null;
  }

  async loadUsers() {
    this.loading = true;
    this.error = null;
    
    try {
      this.users = await this.getAllUsersUseCase.execute();
    } catch (error) {
      this.error = error.message;
    } finally {
      this.loading = false;
    }
  }

  async loadUser(id) {
    this.loading = true;
    this.error = null;
    
    try {
      this.currentUser = await this.getUserUseCase.execute(id);
    } catch (error) {
      this.error = error.message;
    } finally {
      this.loading = false;
    }
  }

  async createUser(userData) {
    this.loading = true;
    this.error = null;
    
    try {
      const newUser = await this.createUserUseCase.execute(userData);
      this.users = [...this.users, newUser];
      return newUser;
    } catch (error) {
      this.error = error.message;
      throw error;
    } finally {
      this.loading = false;
    }
  }
}

// UI Layer - Component (React-style pseudocode)
class UserComponent {
  constructor(viewModel) {
    this.viewModel = viewModel;
    this.container = document.getElementById('user-container');
  }

  async initialize() {
    await this.viewModel.loadUsers();
    this.render();
    this.bindEvents();
  }

  render() {
    if (this.viewModel.loading) {
      this.container.innerHTML = '<div>Загрузка...</div>';
      return;
    }

    if (this.viewModel.error) {
      this.container.innerHTML = `<div class="error">Ошибка: ${this.viewModel.error}</div>`;
      return;
    }

    this.container.innerHTML = `
      <h2>Пользователи</h2>
      <ul>
        ${this.viewModel.users.map(user => 
          `<li>${user.name} (${user.email})</li>`
        ).join('')}
      </ul>
      <button id="addUserBtn">Добавить пользователя</button>
    `;
  }

  bindEvents() {
    document.getElementById('addUserBtn').addEventListener('click', async () => {
      const name = prompt('Имя пользователя:');
      const email = prompt('Email пользователя:');
      
      if (name && email) {
        try {
          await this.viewModel.createUser({ name, email });
          this.render();
        } catch (error) {
          alert('Ошибка при создании пользователя: ' + error.message);
        }
      }
    });
  }
}

// Composition Root - Dependency Injection
function createApp() {
  const httpClient = new HttpClient('https://api.example.com');
  const userRepository = new ApiUserRepository(httpClient);
  
  const getUserUseCase = new GetUserUseCase(userRepository);
  const getAllUsersUseCase = new GetAllUsersUseCase(userRepository);
  const createUserUseCase = new CreateUserUseCase(userRepository);
  
  const viewModel = new UserViewModel(
    getUserUseCase,
    getAllUsersUseCase,
    createUserUseCase
  );
  
  const component = new UserComponent(viewModel);
  
  return component;
}

// Использование
const app = createApp();
app.initialize();
```

## Применение в современных фреймворках

### React
- Использование хуков для управления состоянием представления
- Создание сервисов и репозиториев вне компонентов
- Использование контекста для внедрения зависимостей

### Angular
- Встроенные возможности для внедрения зависимостей
- Сервисы для бизнес-логики
- Компоненты только для презентации

### Vue.js
- Vuex/Pinia для управления состоянием
- Композиция для разделения логики
- Компоненты для представления

## Сравнение с другими паттернами

- [[MVC]] - более простая структура, но сложнее масштабировать
- [[MVP]] - более строгое разделение, но не такое гибкое
- [[MVVM]] - автоматическая привязка данных, но сложнее для сложных архитектур
- [[Flux]] - односторонний поток данных, но не такое глубокое разделение слоев

## Лучшие практики

1. **Создавать интерфейсы для внешних зависимостей** - упрощает тестирование и замену
2. **Использовать Dependency Injection** - облегчает тестирование и поддержку
3. **Следить за направлением зависимостей** - всегда внутрь
4. **Изолировать бизнес-логику в доменном слое** - не смешивать с UI или данными
5. **Использовать DTO (Data Transfer Objects)** для передачи данных между слоями

## Рекомендации по использованию

Clean Architecture особенно эффективна в приложениях с:
- Сложной бизнес-логикой
- Долгосрочной перспективой развития
- Высокими требованиями к тестируемости
- Необходимостью частой смены UI или внешних сервисов
- Командной разработкой с несколькими разработчиками

## См. также

- [[MVC]]
- [[MVP]]
- [[MVVM]]
- [[Flux]]
- [[State-менеджмент]]
- [[Dependency-Injection]]
