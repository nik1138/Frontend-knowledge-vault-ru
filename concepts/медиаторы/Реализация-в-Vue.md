---
aliases: [Медиатор Vue, Паттерн медиатор в Vue, Вью медиатор]
tags: [vue, паттерны-проектирования, медиатор, фронтенд]
---

# Реализация паттерна Медиатор в Vue

## Обзор

В Vue.js паттерн Медиатор можно реализовать несколькими способами: через глобальный событийный шину (Event Bus), Vuex/Pinia для управления состоянием, или через централизованный сервис. Это позволяет компонентам взаимодействовать друг с другом без прямой зависимости, что особенно полезно в сложных приложениях с множеством взаимодействующих компонентов.

## Реализация через глобальную событийную шину

### Простой медиатор с использованием EventBus

```javascript
// Создание глобальной событийной шины
import { createApp } from 'vue';

// Создаем экземпляр для событийной шины
const EventBus = {
  install(app) {
    app.config.globalProperties.$eventBus = this;
    this.app = app;
    this.events = {};
  },

  $on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  },

  $off(event, callback) {
    if (this.events[event]) {
      if (callback) {
        this.events[event] = this.events[event].filter(cb => cb !== callback);
      } else {
        delete this.events[event];
      }
    }
  },

  $emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
};

// Использование в приложении
const app = createApp({});
app.use(EventBus);

// Компонент, отправляющий события
export default {
  name: 'ButtonComponent',
  methods: {
    handleClick() {
      this.$eventBus.$emit('button:click', {
        source: 'ButtonComponent',
        timestamp: Date.now()
      });
    }
  },
  template: `
    <button @click="handleClick">Кнопка</button>
  `
}

// Компонент, слушающий события
export default {
  name: 'ListenerComponent',
  mounted() {
    this.$eventBus.$on('button:click', this.handleEvent);
  },
  beforeUnmount() {
    this.$eventBus.$off('button:click', this.handleEvent);
  },
  methods: {
    handleEvent(data) {
      console.log('Получено событие:', data);
    }
  },
  template: `
    <div>Компонент слушает события</div>
  `
}
```

## Реализация через Pinia

### Медиатор как центральный хранилище

```javascript
// stores/mediator.js
import { defineStore } from 'pinia';

export const useMediatorStore = defineStore('mediator', {
  state: () => ({
    user: null,
    isAuthenticated: false,
    modals: {},
    notifications: []
  }),

  actions: {
    login(userData) {
      this.user = userData;
      this.isAuthenticated = true;
      this.$eventBus.$emit('user:login', userData);
    },

    logout() {
      this.user = null;
      this.isAuthenticated = false;
      this.$eventBus.$emit('user:logout');
    },

    showModal(modalData) {
      this.modals[modalData.id] = modalData;
      this.$eventBus.$emit('modal:show', modalData);
    },

    hideModal(modalId) {
      this.modals[modalId] = null;
      delete this.modals[modalId];
      this.$eventBus.$emit('modal:hide', { id: modalId });
    },

    addNotification(notificationData) {
      this.notifications.push({
        ...notificationData,
        id: Date.now()
      });
      this.$eventBus.$emit('notification:add', notificationData);
    },

    removeNotification(notificationId) {
      this.notifications = this.notifications.filter(n => n.id !== notificationId);
    }
  },

  getters: {
    activeModals: (state) => {
      return Object.values(state.modals).filter(modal => modal !== null);
    },
    
    latestNotification: (state) => {
      return state.notifications.length > 0 
        ? state.notifications[state.notifications.length - 1] 
        : null;
    }
  }
});

// main.js
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import App from './App.vue';

const app = createApp(App);
const pinia = createPinia();

app.use(pinia);
app.mount('#app');
```

```vue
<!-- components/LoginForm.vue -->
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="username" placeholder="Имя пользователя" required />
    <input v-model="email" placeholder="Email" required />
    <button type="submit">Войти</button>
  </form>
</template>

<script>
import { useMediatorStore } from '@/stores/mediator';

export default {
  name: 'LoginForm',
  data() {
    return {
      username: '',
      email: ''
    };
  },
  setup() {
    const mediator = useMediatorStore();
    return { mediator };
  },
  methods: {
    handleSubmit() {
      const userData = {
        username: this.username,
        email: this.email
      };
      
      this.mediator.login(userData);
      this.mediator.addNotification({
        type: 'success',
        message: `Добро пожаловать, ${this.username}!`
      });
      
      // Сброс формы
      this.username = '';
      this.email = '';
    }
  }
};
</script>
```

```vue
<!-- components/UserProfile.vue -->
<template>
  <div v-if="mediator.isAuthenticated">
    <h2>Профиль пользователя</h2>
    <p>Имя: {{ mediator.user.username }}</p>
    <p>Email: {{ mediator.user.email }}</p>
    <button @click="handleLogout">Выйти</button>
  </div>
  <div v-else>
    <p>Пожалуйста, войдите в систему</p>
  </div>
</template>

<script>
import { useMediatorStore } from '@/stores/mediator';

export default {
  name: 'UserProfile',
  setup() {
    const mediator = useMediatorStore();
    return { mediator };
  },
  methods: {
    handleLogout() {
      this.mediator.logout();
      this.mediator.addNotification({
        type: 'info',
        message: 'Вы успешно вышли из системы'
      });
    }
  }
};
</script>
```

## Реализация через централизованный сервис

### Медиатор как отдельный сервис

```javascript
// services/MediatorService.js
class MediatorService {
  constructor() {
    this.subscribers = {};
    this.state = {};
  }

  // Подписка на события
  subscribe(event, callback) {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
    
    // Возвращаем функцию для отписки
    return () => {
      this.unsubscribe(event, callback);
    };
  }

  // Отписка от событий
  unsubscribe(event, callback) {
    if (this.subscribers[event]) {
      this.subscribers[event] = this.subscribers[event].filter(cb => cb !== callback);
    }
  }

  // Публикация события
  publish(event, data) {
    if (this.subscribers[event]) {
      this.subscribers[event].forEach(callback => callback(data));
    }
  }

  // Управление состоянием
  setState(key, value) {
    this.state[key] = value;
    this.publish(`state:${key}:change`, value);
  }

  getState(key) {
    return this.state[key];
  }

  // Методы для специфических сценариев
  handleFormSubmission(formData) {
    // Логика обработки формы
    this.publish('form:submit', formData);
    
    // Валидация
    const isValid = this.validateForm(formData);
    if (isValid) {
      this.publish('form:valid', formData);
    } else {
      this.publish('form:invalid', formData);
    }
  }

  validateForm(formData) {
    // Простая валидация
    return Object.values(formData).every(value => value && value.trim() !== '');
  }
}

// Создаем глобальный экземпляр
export const mediatorService = new MediatorService();

// Регистрируем в Vue приложении
export const MediatorPlugin = {
  install(app) {
    app.config.globalProperties.$mediator = mediatorService;
    app.provide('mediator', mediatorService);
  }
};
```

```javascript
// main.js
import { createApp } from 'vue';
import { MediatorPlugin } from './services/MediatorService';
import App from './App.vue';

const app = createApp(App);

app.use(MediatorPlugin);
app.mount('#app');
```

```vue
<!-- components/FormComponent.vue -->
<template>
  <form @submit.prevent="handleSubmit">
    <input v-model="formData.name" placeholder="Имя" required />
    <input v-model="formData.email" placeholder="Email" required />
    <input v-model="formData.phone" placeholder="Телефон" />
    <button type="submit" :disabled="!isFormValid">Отправить</button>
  </form>
</template>

<script>
export default {
  name: 'FormComponent',
  data() {
    return {
      formData: {
        name: '',
        email: '',
        phone: ''
      },
      isFormValid: false
    };
  },
  mounted() {
    // Подписываемся на события валидации
    this.unsubscribeValidation = this.$mediator.subscribe('form:valid', () => {
      this.isFormValid = true;
    });
    
    this.unsubscribeInvalid = this.$mediator.subscribe('form:invalid', () => {
      this.isFormValid = false;
    });
  },
  beforeUnmount() {
    // Отписываемся от событий
    if (this.unsubscribeValidation) this.unsubscribeValidation();
    if (this.unsubscribeInvalid) this.unsubscribeInvalid();
  },
  methods: {
    handleSubmit() {
      this.$mediator.handleFormSubmission(this.formData);
    }
  }
};
</script>
```

```vue
<!-- components/NotificationComponent.vue -->
<template>
  <div class="notifications">
    <div 
      v-for="notification in notifications" 
      :key="notification.id" 
      :class="['notification', notification.type]"
      @click="removeNotification(notification.id)"
    >
      {{ notification.message }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'NotificationComponent',
  data() {
    return {
      notifications: []
    };
  },
  mounted() {
    this.unsubscribe = this.$mediator.subscribe('notification:add', (data) => {
      this.notifications.push({
        ...data,
        id: Date.now()
      });
      
      // Автоматическое удаление уведомления через 5 секунд
      setTimeout(() => {
        this.removeNotification(data.id);
      }, 5000);
    });
  },
  beforeUnmount() {
    if (this.unsubscribe) this.unsubscribe();
  },
  methods: {
    removeNotification(id) {
      this.notifications = this.notifications.filter(n => n.id !== id);
    }
  }
};
</script>

<style scoped>
.notification {
  padding: 10px;
  margin: 5px 0;
  border-radius: 4px;
  cursor: pointer;
}

.notification.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.notification.error {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

.notification.info {
  background-color: #d1ecf1;
  color: #0c5460;
  border: 1px solid #bee5eb;
}
</style>
```

## Пример сложного взаимодействия компонентов

### Управление корзиной покупок через медиатор

```javascript
// services/CartMediator.js
import { reactive } from 'vue';

class CartMediator {
  constructor() {
    this.state = reactive({
      items: [],
      total: 0,
      itemCount: 0
    });
    this.subscribers = {};
  }

  subscribe(event, callback) {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
    
    return () => {
      this.unsubscribe(event, callback);
    };
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

  addItem(item) {
    const existingItem = this.state.items.find(i => i.id === item.id);
    
    if (existingItem) {
      existingItem.quantity += item.quantity || 1;
    } else {
      this.state.items.push({
        ...item,
        quantity: item.quantity || 1
      });
    }
    
    this.calculateTotal();
    this.publish('cart:item:add', item);
  }

  removeItem(itemId) {
    this.state.items = this.state.items.filter(item => item.id !== itemId);
    this.calculateTotal();
    this.publish('cart:item:remove', { id: itemId });
  }

  updateQuantity(itemId, quantity) {
    const item = this.state.items.find(i => i.id === itemId);
    if (item) {
      item.quantity = Math.max(1, quantity);
      this.calculateTotal();
      this.publish('cart:item:update', { id: itemId, quantity });
    }
  }

  clearCart() {
    this.state.items = [];
    this.calculateTotal();
    this.publish('cart:clear');
  }

  calculateTotal() {
    this.state.total = this.state.items.reduce(
      (sum, item) => sum + (item.price * item.quantity), 
      0
    );
    this.state.itemCount = this.state.items.reduce(
      (count, item) => count + item.quantity, 
      0
    );
  }

  getCartState() {
    return this.state;
  }
}

export const cartMediator = new CartMediator();
```

```vue
<!-- components/CartSummary.vue -->
<template>
  <div class="cart-summary">
    <h3>Корзина</h3>
    <p>Товаров: {{ cartState.itemCount }}</p>
    <p>Общая сумма: {{ cartState.total }} руб.</p>
    <button @click="handleCheckout" :disabled="cartState.itemCount === 0">
      Оформить заказ
    </button>
  </div>
</template>

<script>
import { cartMediator } from '@/services/CartMediator';

export default {
  name: 'CartSummary',
  data() {
    return {
      cartState: cartMediator.getCartState()
    };
  },
  methods: {
    handleCheckout() {
      // Логика оформления заказа
      console.log('Оформление заказа на сумму:', this.cartState.total);
      cartMediator.publish('cart:checkout', {
        items: this.cartState.items,
        total: this.cartState.total
      });
    }
  }
};
</script>
```

## Преимущества использования медиатора в Vue

- **Снижение связанности**: Компоненты не зависят напрямую друг от друга.
- **Централизованное управление состоянием**: Логика взаимодействия сосредоточена в одном месте.
- **Упрощение тестирования**: Компоненты легче тестировать изолированно.

## Возможные проблемы

- **Сложность отладки**: Событийная архитектура может усложнить отслеживание потока данных.
- **Производительность**: Большое количество подписчиков может повлиять на производительность.

## Заключение

Паттерн Медиатор в Vue позволяет создать гибкую архитектуру с централизованным управлением взаимодействиями между компонентами. Он особенно полезен в сложных приложениях с множеством взаимодействующих компонентов.

Для изучения реализации в других технологиях см.:
- [[Реализация-в-JavaScript]]
- [[Реализация-в-React]]

Для практических примеров см. [[Примеры-использования]].
