---
aliases: ["Singleton в Vue", "Синглтон Vue", "Vue Singleton"]
tags: [vue, frontend, design-patterns]
---

# Использование паттерна Синглтон в Vue.js

## Обзор

Паттерн Синглтон в Vue.js может быть эффективно использован для управления глобальным состоянием, кэширования данных, управления API клиентами и других задач, где необходим уникальный экземпляр. Vue предоставляет несколько способов интеграции синглтонов с его системой реактивности.

## Основные области применения

### 1. Управление глобальным состоянием

```javascript
// store.js
import { reactive, toRaw } from 'vue';

class GlobalStore {
    constructor() {
        if (GlobalStore.instance) {
            return GlobalStore.instance;
        }
        
        this.state = reactive({
            user: null,
            theme: 'light',
            language: 'ru',
            notifications: []
        });
        
        GlobalStore.instance = this;
        return this;
    }
    
    setState(newState) {
        Object.assign(this.state, newState);
    }
    
    getState() {
        // Возвращаем копию состояния, чтобы избежать мутаций вне синглтона
        return { ...toRaw(this.state) };
    }
    
    subscribe(callback) {
        // В Vue реактивность работает через наблюдение за свойствами
        return () => {
            // Нет необходимости в явной отписке при использовании computed/watch
        };
    }
}

export default new GlobalStore();
```

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <h1>Пользователь: {{ store.state.user?.name || 'Гость' }}</h1>
    <button @click="updateUser">Обновить пользователя</button>
    <ThemeSwitcher />
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import globalStore from './store';

export default {
  name: 'App',
  setup() {
    const store = ref(globalStore);
    
    const updateUser = () => {
      store.value.setState({
        user: { name: 'John', id: 1 }
      });
    };
    
    return {
      store,
      updateUser
    };
  }
};
</script>
```

### 2. Кэширование данных

```javascript
// cacheService.js
import { reactive } from 'vue';

class CacheService {
    constructor() {
        if (CacheService.instance) {
            return CacheService.instance;
        }
        
        this.cache = reactive(new Map());
        this.timeouts = new Map();
        
        CacheService.instance = this;
        return this;
    }
    
    set(key, value, ttl = 300000) { // 5 минут по умолчанию
        this.cache.set(key, {
            value,
            timestamp: Date.now(),
            ttl
        });
        
        // Автоматическое удаление по истечении времени
        if (this.timeouts.has(key)) {
            clearTimeout(this.timeouts.get(key));
        }
        
        const timeout = setTimeout(() => {
            this.cache.delete(key);
            this.timeouts.delete(key);
        }, ttl);
        
        this.timeouts.set(key, timeout);
    }
    
    get(key) {
        const item = this.cache.get(key);
        if (!item) return null;
        
        // Проверка срока действия
        if (Date.now() - item.timestamp > item.ttl) {
            this.cache.delete(key);
            return null;
        }
        
        return item.value;
    }
    
    has(key) {
        return this.cache.has(key) && this.get(key) !== null;
    }
    
    clear() {
        this.cache.clear();
        this.timeouts.forEach(timeout => clearTimeout(timeout));
        this.timeouts.clear();
    }
    
    // Получение реактивного значения из кэша
    getReactive(key) {
        return () => this.cache.get(key)?.value;
    }
}

export default new CacheService();
```

```vue
<!-- UserProfile.vue -->
<template>
  <div v-if="loading">Загрузка...</div>
  <div v-else-if="user">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <button @click="refreshUser">Обновить</button>
  </div>
  <div v-else>Пользователь не найден</div>
</template>

<script>
import { ref, onMounted } from 'vue';
import cacheService from './cacheService';

export default {
  name: 'UserProfile',
  props: {
    userId: {
      type: Number,
      required: true
    }
  },
  setup(props) {
    const user = ref(null);
    const loading = ref(true);
    
    const fetchUser = async () => {
      loading.value = true;
      
      try {
        // Проверка кэша
        const cachedUser = cacheService.get(`user_${props.userId}`);
        
        if (cachedUser) {
          user.value = cachedUser;
          loading.value = false;
          return;
        }
        
        // Загрузка данных
        const response = await fetch(`/api/users/${props.userId}`);
        const userData = await response.json();
        
        // Кэширование
        cacheService.set(`user_${props.userId}`, userData);
        user.value = userData;
      } catch (error) {
        console.error('Ошибка загрузки пользователя:', error);
      } finally {
        loading.value = false;
      }
    };
    
    const refreshUser = async () => {
      // Удаление из кэша для принудительного обновления
      cacheService.cache.delete(`user_${props.userId}`);
      await fetchUser();
    };
    
    onMounted(fetchUser);
    
    return {
      user,
      loading,
      refreshUser
    };
  }
};
</script>
```

### 3. Управление API клиентом

```javascript
// apiClient.js
import { reactive } from 'vue';

class ApiClient {
    constructor() {
        if (ApiClient.instance) {
            return ApiClient.instance;
        }
        
        this.baseURL = process.env.VUE_APP_API_URL || 'http://localhost:3000';
        this.token = null;
        
        // Состояние для отслеживания активных запросов
        this.activeRequests = reactive(new Map());
        
        ApiClient.instance = this;
        return this;
    }
    
    setToken(token) {
        this.token = token;
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const requestId = `${options.method || 'GET'}_${url}`;
        
        // Отслеживание активного запроса
        this.activeRequests.set(requestId, true);
        
        const config = {
            headers: {
                'Content-Type': 'application/json',
                ...(this.token && { 'Authorization': `Bearer ${this.token}` }),
                ...options.headers
            },
            ...options
        };
        
        try {
            const response = await fetch(url, config);
            
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            
            return await response.json();
        } catch (error) {
            console.error('API ошибка:', error);
            throw error;
        } finally {
            // Удаление из активных запросов
            this.activeRequests.delete(requestId);
        }
    }
    
    get(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'GET' });
    }
    
    post(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    put(endpoint, data, options = {}) {
        return this.request(endpoint, {
            ...options,
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    delete(endpoint, options = {}) {
        return this.request(endpoint, { ...options, method: 'DELETE' });
    }
    
    // Проверка, активен ли запрос
    isActive(requestId) {
        return this.activeRequests.has(requestId);
    }
}

export default new ApiClient();
```

## Использование с Vue 3 Composition API

```javascript
// composables/useSingleton.js
import { readonly, toRaw } from 'vue';
import globalStore from '../store';

export function useSingletonStore() {
    // Возвращаем реактивное состояние
    const state = globalStore.state;
    
    const setState = (newState) => {
        globalStore.setState(newState);
    };
    
    const getState = () => {
        return readonly({ ...toRaw(state) });
    };
    
    return {
        state,
        setState,
        getState
    };
}
```

```vue
<!-- ComponentUsingComposable.vue -->
<template>
  <div>
    <h2>Состояние из синглтона</h2>
    <p>Пользователь: {{ state.user?.name || 'Не авторизован' }}</p>
    <p>Тема: {{ state.theme }}</p>
    <button @click="updateUser">Обновить пользователя</button>
  </div>
</template>

<script>
import { useSingletonStore } from '@/composables/useSingleton';

export default {
  name: 'ComponentUsingComposable',
  setup() {
    const { state, setState } = useSingletonStore();
    
    const updateUser = () => {
      setState({
        user: { 
          name: 'Jane', 
          id: 2,
          lastLogin: new Date().toISOString()
        }
      });
    };
    
    return {
      state,
      updateUser
    };
  }
};
</script>
```

## Использование с Vue 2 Options API

```javascript
// mixins/singletonMixin.js
import globalStore from '../store';

export default {
    data() {
        return {
            singletonState: globalStore.state
        };
    },
    methods: {
        updateSingletonState(newState) {
            globalStore.setState(newState);
        },
        getSingletonState() {
            return globalStore.getState();
        }
    }
};
```

```vue
<!-- ComponentUsingMixin.vue -->
<template>
  <div>
    <h2>Состояние: {{ singletonState.user?.name || 'Гость' }}</h2>
    <button @click="updateUser">Обновить пользователя</button>
  </div>
</template>

<script>
import singletonMixin from '@/mixins/singletonMixin';

export default {
  name: 'ComponentUsingMixin',
  mixins: [singletonMixin],
  methods: {
    updateUser() {
      this.updateSingletonState({
        user: { name: 'Alice', id: 3 }
      });
    }
  }
};
</script>
```

## Практические рекомендации

### 1. Интеграция с Vue DevTools

Для лучшей отладки синглтонов в Vue DevTools можно добавить специальные методы:

```javascript
// debuggableSingleton.js
import { reactive } from 'vue';

class DebuggableSingleton {
    constructor() {
        if (DebuggableSingleton.instance) {
            return DebuggableSingleton.instance;
        }
        
        this.state = reactive({
            data: {},
            history: []
        });
        
        DebuggableSingleton.instance = this;
        return this;
    }
    
    setState(newState, action = 'UPDATE') {
        const prevState = { ...this.state.data };
        Object.assign(this.state.data, newState);
        
        // Сохранение истории изменений для отладки
        this.state.history.push({
            action,
            prevState: { ...prevState },
            newState: { ...newState },
            timestamp: Date.now()
        });
    }
    
    // Метод для очистки истории (для отладки)
    clearHistory() {
        this.state.history = [];
    }
    
    // Метод для отката изменений (для отладки)
    undo() {
        if (this.state.history.length > 0) {
            const lastChange = this.state.history.pop();
            this.state.data = { ...lastChange.prevState };
        }
    }
}

export default new DebuggableSingleton();
```

### 2. SSR безопасность

При использовании синглтонов в SSR (например, с Nuxt.js) важно учитывать, что синглтон будет общим для всех запросов на сервере:

```javascript
// ssrSafeSingleton.js
import { reactive } from 'vue';

let instance = null;

class SsrSafeSingleton {
    constructor() {
        // В браузере используем синглтон
        if (typeof window !== 'undefined') {
            if (instance) {
                return instance;
            }
            instance = this;
        }
        
        // На сервере создаем новый экземпляр для каждого запроса
        this.data = reactive({});
    }
    
    set(key, value) {
        this.data[key] = value;
    }
    
    get(key) {
        return this.data[key];
    }
}

export default SsrSafeSingleton;
```

### 3. Тестирование синглтонов в Vue

Для тестирования компонентов, использующих синглтоны, рекомендуется использовать моки:

```javascript
// UserProfile.test.js
import { mount } from '@vue/test-utils';
import UserProfile from '@/components/UserProfile';
import cacheService from '@/services/cacheService';

describe('UserProfile', () => {
    beforeEach(() => {
        // Очистка кэша перед каждым тестом
        cacheService.clear();
    });
    
    it('отображает данные пользователя из кэша', async () => {
        // Заполнение кэша тестовыми данными
        cacheService.set('user_1', { id: 1, name: 'Test User', email: 'test@example.com' });
        
        const wrapper = mount(UserProfile, {
            props: {
                userId: 1
            }
        });
        
        // Ожидание рендеринга
        await wrapper.vm.$nextTick();
        
        expect(wrapper.text()).toContain('Test User');
    });
    
    it('загружает данные, если нет в кэше', async () => {
        // Мок fetch API
        global.fetch = jest.fn(() =>
            Promise.resolve({
                ok: true,
                json: () => Promise.resolve({ id: 2, name: 'API User', email: 'api@example.com' })
            })
        );
        
        const wrapper = mount(UserProfile, {
            props: {
                userId: 2
            }
        });
        
        // Ожидание асинхронной операции
        await new Promise(resolve => setTimeout(resolve, 0));
        
        expect(wrapper.text()).toContain('API User');
        expect(global.fetch).toHaveBeenCalledWith('/api/users/2');
    });
});
```

## Альтернативы синглтону в Vue

### 1. Vuex

Vuex - официальная библиотека управления состоянием для Vue:

```javascript
// store/index.js
import { createStore } from 'vuex';

export default createStore({
    state: {
        user: null,
        theme: 'light'
    },
    mutations: {
        SET_USER(state, user) {
            state.user = user;
        },
        SET_THEME(state, theme) {
            state.theme = theme;
        }
    },
    actions: {
        updateUser({ commit }, user) {
            commit('SET_USER', user);
        }
    }
});
```

### 2. Pinia

Pinia - современная альтернатива Vuex:

```javascript
// stores/user.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
    state: () => ({
        user: null,
        theme: 'light'
    }),
    actions: {
        updateUser(user) {
            this.user = user;
        },
        setTheme(theme) {
            this.theme = theme;
        }
    }
});
```

## Заключение

Паттерн Синглтон в Vue.js может быть эффективным инструментом для управления глобальным состоянием и общими ресурсами. Vue предоставляет отличную интеграцию с реактивной системой, позволяя легко использовать синглтоны в компонентах.

Однако в современных Vue-приложениях часто используются специализированные решения, такие как Pinia или Vuex, которые лучше интегрированы с экосистемой Vue и обеспечивают лучшую поддержку и тестируемость.

## См. также

- [[Паттерн-синглтон]]
- [[Реализация-в-JavaScript]]
- [[Использование-в-React]]
- [[Проблемы-и-альтернативы]]
- [[Vuex]]
- [[Pinia]]
- [[Vue Composition API]]