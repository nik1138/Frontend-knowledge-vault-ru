---
aliases: ["Flux-архитектура", "Flux-паттерн"]
tags: [architecture, frontend, pattern, flux, state-management]
---

# Flux (Архитектурный паттерн)

## Обзор

Flux - это архитектурный паттерн, разработанный Facebook (ныне Meta) для построения клиентских веб-приложений. В отличие от традиционных MVC-паттернов, Flux использует односторонний поток данных, что делает состояние приложения более предсказуемым и легким для отладки. Паттерн стал основой для Redux и других библиотек управления состоянием.

## Структура паттерна

Flux состоит из четырех основных компонентов:

### Actions (Действия)
- **Отвечают за**: передачу данных в систему
- **Содержат**: тип действия и полезную нагрузку
- **Инициируются**: пользовательскими событиями или другими частями приложения

```javascript
// Пример actions
const ActionTypes = {
  USER_UPDATE_NAME: 'USER_UPDATE_NAME',
  USER_UPDATE_EMAIL: 'USER_UPDATE_EMAIL',
  USER_LOAD_REQUEST: 'USER_LOAD_REQUEST',
  USER_LOAD_SUCCESS: 'USER_LOAD_SUCCESS',
  USER_LOAD_ERROR: 'USER_LOAD_ERROR'
};

// Action creators
function updateUserEmail(email) {
  return {
    type: ActionTypes.USER_UPDATE_EMAIL,
    payload: { email }
  };
}

function updateUserName(name) {
  return {
    type: ActionTypes.USER_UPDATE_NAME,
    payload: { name }
  };
}

function loadUserRequest() {
  return {
    type: ActionTypes.USER_LOAD_REQUEST
  };
}

function loadUserSuccess(user) {
  return {
    type: ActionTypes.USER_LOAD_SUCCESS,
    payload: { user }
  };
}

function loadUserError(error) {
  return {
    type: ActionTypes.USER_LOAD_ERROR,
    payload: { error }
  };
}
```

### Dispatcher (Диспетчер)
- **Отвечает за**: централизованную обработку действий
- **Обеспечивает**: односторонний поток данных
- **Управляет**: порядком обработки действий

```javascript
// Простая реализация Dispatcher
class Dispatcher {
  constructor() {
    this.callbacks = [];
    this.isDispatching = false;
    this.pendingPayload = null;
  }

  register(callback) {
    this.callbacks.push(callback);
    return this.callbacks.length - 1; // Возвращаем ID регистрации
  }

  unregister(id) {
    delete this.callbacks[id];
  }

  dispatch(payload) {
    if (this.isDispatching) {
      throw new Error('Cannot dispatch while dispatching');
    }

    this.isDispatching = true;
    this.pendingPayload = payload;

    try {
      this.callbacks.forEach(callback => {
        if (callback) {
          callback(payload);
        }
      });
    } finally {
      this.isDispatching = false;
      this.pendingPayload = null;
    }
  }

  waitFor(ids) {
    ids.forEach(id => {
      if (this.callbacks[id]) {
        this.callbacks[id](this.pendingPayload);
      }
    });
  }
}

const AppDispatcher = new Dispatcher();
```

### Stores (Хранилища)
- **Отвечают за**: хранение и управление состоянием
- **Содержат**: бизнес-логику, связанную с определенным аспектом приложения
- **Уведомляют**: View об изменениях состояния

```javascript
// Пример Store
class UserStore {
  constructor(dispatcher) {
    this.dispatcher = dispatcher;
    this.user = {
      name: 'Иван Петров',
      email: 'ivan@example.com',
      loading: false,
      error: null
    };
    
    // Регистрируем callback в диспетчере
    this.dispatchId = dispatcher.register(this.handleAction.bind(this));
  }

  handleAction(action) {
    switch (action.type) {
      case ActionTypes.USER_UPDATE_NAME:
        this.user.name = action.payload.name;
        this.emitChange();
        break;
      case ActionTypes.USER_UPDATE_EMAIL:
        this.user.email = action.payload.email;
        this.emitChange();
        break;
      case ActionTypes.USER_LOAD_REQUEST:
        this.user.loading = true;
        this.user.error = null;
        this.emitChange();
        break;
      case ActionTypes.USER_LOAD_SUCCESS:
        this.user = {
          ...this.user,
          ...action.payload.user,
          loading: false
        };
        this.emitChange();
        break;
      case ActionTypes.USER_LOAD_ERROR:
        this.user.loading = false;
        this.user.error = action.payload.error;
        this.emitChange();
        break;
      default:
        // Ничего не делаем
    }
  }

  getUser() {
    return this.user;
  }

  emitChange() {
    // В реальном приложении это будет событие, которое View слушают
    if (this.changeListener) {
      this.changeListener();
    }
  }

  addChangeListener(callback) {
    this.changeListener = callback;
  }

  removeChangeListener(callback) {
    this.changeListener = null;
  }
}

const userStore = new UserStore(AppDispatcher);
```

### Views (Представления)
- **Отвечают за**: отображение данных и инициацию действий
- **Получают данные**: из Store через подписку
- **Инициируют действия**: через Action creators

```html
<!-- Пример представления -->
<div id="flux-user-view">
  <h2>Данные пользователя</h2>
  <div v-if="loading">Загрузка...</div>
  <div v-else-if="error" style="color: red;">{{ error }}</div>
  <div v-else>
    <p>Имя: <span id="user-name">{{ user.name }}</span></p>
    <p>Email: <span id="user-email">{{ user.email }}</span></p>
  </div>
  
  <form id="user-form">
    <input type="text" id="name-input" placeholder="Имя">
    <input type="email" id="email-input" placeholder="Email">
    <button type="submit">Обновить</button>
  </form>
</div>
```

```javascript
// Пример View, взаимодействующего с Flux
class FluxUserView {
  constructor(store, dispatcher) {
    this.store = store;
    this.dispatcher = dispatcher;
    this.element = document.getElementById('flux-user-view');
    
    // Подписываемся на изменения Store
    this.store.addChangeListener(this.render.bind(this));
    
    // Инициализируем форму
    this.form = document.getElementById('user-form');
    this.nameInput = document.getElementById('name-input');
    this.emailInput = document.getElementById('email-input');
    
    this.initEventListeners();
    this.render();
  }

  initEventListeners() {
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      
      const name = this.nameInput.value;
      const email = this.emailInput.value;
      
      if (name) {
        this.dispatcher.dispatch(updateUserName(name));
      }
      
      if (email) {
        this.dispatcher.dispatch(updateUserEmail(email));
      }
      
      // Очищаем форму
      this.nameInput.value = '';
      this.emailInput.value = '';
    });
  }

  render() {
    const user = this.store.getUser();
    
    document.getElementById('user-name').textContent = user.name;
    document.getElementById('user-email').textContent = user.email;
    
    // Обновляем состояние загрузки и ошибки
    const loadingElement = this.element.querySelector('div:contains("Загрузка")');
    const errorElement = this.element.querySelector('div[style*="color: red"]');
    
    // В реальном приложении было бы больше логики обновления DOM
    console.log('View обновлен с новыми данными:', user);
  }
}

// Использование
const fluxUserView = new FluxUserView(userStore, AppDispatcher);
```

## Принципы Flux

### Односторонний поток данных
1. **View** инициирует **Action**
2. **Action** отправляется в **Dispatcher**
3. **Dispatcher** передает **Action** всем **Store**
4. **Store** обновляет состояние и уведомляет **View**
5. **View** перерисовывается с новыми данными

### Централизованное управление состоянием
- Все состояние приложения хранится в одном месте (Store)
- Это упрощает отладку и тестирование
- Позволяет легко реализовать функции типа "отменить действие"

## Преимущества Flux

1. **Предсказуемость**: односторонний поток данных делает поведение приложения предсказуемым
2. **Легкость отладки**: можно легко отследить, как данные изменяются
3. **Тестируемость**: компоненты можно тестировать изолированно
4. **Масштабируемость**: подходит для крупных приложений

## Недостатки Flux

1. **Сложность для начинающих**: требует понимания архитектурных концепций
2. **Более многословный код**: по сравнению с простыми подходами
3. **Переизбыток для маленьких приложений**: может быть избыточным

## Применение в российских реалиях 2025

Flux и его производные (в основном Redux) популярны в российской разработке:

- **Крупных веб-приложениях**: где важна предсказуемость состояния
- **Командной разработке**: где важна согласованность архитектуры
- **Проектах с государственными заказчиками**: где важна документируемость
- **Импортозамещающих решениях**: адаптация иностранных паттернов

В условиях санкций и перехода на отечественные решения Flux остается важной архитектурной концепцией, на основе которой создаются собственные решения управления состоянием.

## Сравнение с другими паттернами

- [[MVC]]: Flux использует односторонний поток вместо двунаправленной связи
- [[MVP]]: Flux фокусируется на управлении состоянием, а не на интерфейсе
- [[MVVM]]: Flux не использует привязку данных, а полагается на явные действия
- [[Чистая-архитектура]]: более сложная, но более гибкая архитектура

## Практические рекомендации

1. **Используйте Redux**: для React-приложений вместо чистого Flux
2. **Организуйте Actions логично**: группируйте по функциональности
3. **Избегайте сложных Store**: разделяйте на несколько небольших хранилищ
4. **Используйте DevTools**: для отладки потока данных

## Заключение

Flux - важный архитектурный паттерн, который заложил основу для современных подходов к управлению состоянием в фронтенд-приложениях. Его принципы одностороннего потока данных повлияли на многие современные библиотеки и фреймворки.

## См. также

- [[MVC]]
- [[MVP]]
- [[MVVM]]
- [[Чистая-архитектура]]
- [[Redux]]
- [[Архитектурные-паттерны]]