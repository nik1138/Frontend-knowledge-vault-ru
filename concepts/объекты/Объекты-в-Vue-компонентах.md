---
aliases: [Vue-Objects, Objects-in-Vue, Объекты-Vue]
tags: [vue, javascript, frontend, objects]
---

# Объекты в Vue-компонентах

Vue.js активно использует объекты для определения компонентов, управления состоянием, передачи данных и реализации различных паттернов. Понимание работы с объектами в Vue критически важно для создания эффективных и поддерживаемых приложений.

## Определение компонента через объект

### Options API

```javascript
// Простой компонент с объектом опций
export default {
  name: 'UserProfile',
  props: {
    user: {
      type: Object,
      required: true,
      default: () => ({})
    },
    showEmail: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      isEditing: false,
      localUser: { ...this.user } // создаем локальную копию
    };
  },
  computed: {
    displayName() {
      return this.localUser.firstName + ' ' + this.localUser.lastName;
    },
    userInitials() {
      const first = this.localUser.firstName?.charAt(0) || '';
      const last = this.localUser.lastName?.charAt(0) || '';
      return (first + last).toUpperCase();
    }
  },
  methods: {
    startEditing() {
      this.isEditing = true;
      this.localUser = { ...this.user }; // сбрасываем изменения
    },
    saveChanges() {
      // emit события с объектом данных
      this.$emit('update-user', { ...this.localUser });
      this.isEditing = false;
    },
    cancelEditing() {
      this.localUser = { ...this.user }; // отменяем изменения
      this.isEditing = false;
    }
  },
  mounted() {
    console.log('Компонент UserProfile смонтирован');
  }
};
```

### Composition API с объектами

```javascript
import { ref, reactive, computed, toRefs } from 'vue';

export default {
  name: 'UserProfile',
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  emits: ['update-user'],
  setup(props, { emit }) {
    // Реактивный объект для локального состояния
    const state = reactive({
      isEditing: false,
      localUser: { ...props.user }
    });

    // Вычисляемые свойства
    const displayName = computed(() => 
      state.localUser.firstName + ' ' + state.localUser.lastName
    );

    const userInitials = computed(() => {
      const first = state.localUser.firstName?.charAt(0) || '';
      const last = state.localUser.lastName?.charAt(0) || '';
      return (first + last).toUpperCase();
    });

    // Методы
    const startEditing = () => {
      state.isEditing = true;
      state.localUser = { ...props.user };
    };

    const saveChanges = () => {
      emit('update-user', { ...state.localUser });
      state.isEditing = false;
    };

    const cancelEditing = () => {
      state.localUser = { ...props.user };
      state.isEditing = false;
    };

    // Возвращаем объект с тем, что нужно в шаблоне
    return {
      ...toRefs(state),
      displayName,
      userInitials,
      startEditing,
      saveChanges,
      cancelEditing
    };
  }
};
```

## Работа с пропсами объектов

### Валидация объектов в пропсах

```javascript
export default {
  name: 'ProductCard',
  props: {
    // Простая валидация
    product: {
      type: Object,
      required: true
    },
    // Сложная валидация с объектом
    config: {
      type: Object,
      default: () => ({
        showPrice: true,
        showRating: true,
        currency: 'RUB'
      }),
      validator(value) {
        // Проверяем, что объект имеет нужные свойства
        return typeof value.showPrice === 'boolean' &&
               typeof value.showRating === 'boolean' &&
               typeof value.currency === 'string';
      }
    }
  },
  computed: {
    formattedPrice() {
      return new Intl.NumberFormat('ru-RU', {
        style: 'currency',
        currency: this.config.currency
      }).format(this.product.price);
    }
  }
};
```

### Объекты с динамическими свойствами

```javascript
export default {
  name: 'DynamicForm',
  props: {
    fields: {
      type: Array,
      required: true
    },
    initialValues: {
      type: Object,
      default: () => ({})
    }
  },
  data() {
    return {
      formData: {}
    };
  },
  created() {
    // Инициализация formData на основе полей
    this.formData = this.fields.reduce((acc, field) => {
      acc[field.name] = this.initialValues[field.name] || field.defaultValue || '';
      return acc;
    }, {});
  },
  methods: {
    updateField(fieldName, value) {
      this.$set(this.formData, fieldName, value);
      // или в Vue 3: this.formData[fieldName] = value;
    },
    submitForm() {
      this.$emit('submit', { ...this.formData });
    }
  }
};
```

## Реактивные объекты

### Использование reactive()

```javascript
import { reactive, ref, computed } from 'vue';

export default {
  setup() {
    // Реактивный объект
    const userProfile = reactive({
      personal: {
        firstName: 'Иван',
        lastName: 'Иванов',
        email: 'ivan@example.com'
      },
      preferences: {
        theme: 'light',
        notifications: true,
        language: 'ru'
      },
      profile: {
        bio: '',
        avatar: null,
        social: {
          twitter: '',
          github: ''
        }
      }
    });

    // Реактивное вычисляемое свойство
    const fullName = computed(() => 
      `${userProfile.personal.firstName} ${userProfile.personal.lastName}`
    );

    // Методы для обновления
    const updatePersonalInfo = (info) => {
      Object.assign(userProfile.personal, info);
    };

    const updatePreferences = (prefs) => {
      Object.assign(userProfile.preferences, prefs);
    };

    return {
      userProfile,
      fullName,
      updatePersonalInfo,
      updatePreferences
    };
  }
};
```

### Глубокая и поверхностная реактивность

```javascript
import { reactive, shallowReactive, readonly, shallowReadonly } from 'vue';

export default {
  setup() {
    // Глубокая реактивность (по умолчанию)
    const deepReactive = reactive({
      level1: {
        level2: {
          value: 'reactive'
        }
      }
    });

    // Поверхностная реактивность - только первый уровень
    const shallowReactiveObj = shallowReactive({
      level1: {
        level2: {
          value: 'not reactive'
        }
      }
    });

    // Только для чтения
    const readOnly = readonly({
      config: {
        apiUrl: 'https://api.example.com',
        timeout: 5000
      }
    });

    // Поверхностное "только для чтения"
    const shallowReadOnly = shallowReadonly({
      settings: {
        theme: 'dark',
        language: 'ru'
      }
    });

    return {
      deepReactive,
      shallowReactiveObj,
      readOnly,
      shallowReadOnly
    };
  }
};
```

## Объекты в хранилище (Vuex/Pinia)

### Vuex с объектами

```javascript
// store/modules/user.js
export default {
  namespaced: true,
  state: {
    profile: {
      id: null,
      name: '',
      email: '',
      permissions: [],
      preferences: {
        theme: 'light',
        language: 'ru'
      }
    },
    loading: false,
    error: null
  },
  mutations: {
    SET_PROFILE(state, profile) {
      // Полная замена профиля
      state.profile = { ...state.profile, ...profile };
    },
    UPDATE_PROFILE_FIELD(state, { field, value }) {
      // Обновление конкретного поля
      if (field.includes('.')) {
        // Обработка вложенных полей
        const [parent, child] = field.split('.');
        state.profile[parent][child] = value;
      } else {
        state.profile[field] = value;
      }
    },
    SET_LOADING(state, loading) {
      state.loading = loading;
    },
    SET_ERROR(state, error) {
      state.error = error;
    }
  },
  actions: {
    async fetchProfile({ commit }, userId) {
      commit('SET_LOADING', true);
      try {
        const response = await api.getUserProfile(userId);
        commit('SET_PROFILE', response.data);
      } catch (error) {
        commit('SET_ERROR', error.message);
      } finally {
        commit('SET_LOADING', false);
      }
    },
    updateProfileField({ commit }, payload) {
      commit('UPDATE_PROFILE_FIELD', payload);
    }
  },
  getters: {
    profile: state => state.profile,
    fullName: state => `${state.profile.name}`,
    hasPermission: state => permission => 
      state.profile.permissions.includes(permission)
  }
};
```

### Pinia с объектами

```javascript
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
  state: () => ({
    profile: {
      id: null,
      name: '',
      email: '',
      permissions: [],
      preferences: {
        theme: 'light',
        language: 'ru'
      }
    },
    session: {
      token: null,
      expiresAt: null
    }
  }),

  getters: {
    isAuthenticated: (state) => !!state.session.token,
    fullName: (state) => `${state.profile.name}`,
    hasPermission: (state) => {
      return (permission) => state.profile.permissions.includes(permission);
    }
  },

  actions: {
    setProfile(profileData) {
      // Обновление профиля
      Object.assign(this.profile, profileData);
    },

    updateProfileField(fieldPath, value) {
      // Обновление вложенного поля
      const keys = fieldPath.split('.');
      let target = this.profile;
      
      for (let i = 0; i < keys.length - 1; i++) {
        target = target[keys[i]];
      }
      
      target[keys[keys.length - 1]] = value;
    },

    async login(credentials) {
      try {
        const response = await api.login(credentials);
        this.session.token = response.data.token;
        this.session.expiresAt = response.data.expiresAt;
        
        // Загрузка профиля
        await this.fetchProfile();
        
        return { success: true };
      } catch (error) {
        return { success: false, error: error.message };
      }
    },

    logout() {
      this.session.token = null;
      this.session.expiresAt = null;
      this.profile = {
        id: null,
        name: '',
        email: '',
        permissions: [],
        preferences: {
          theme: 'light',
          language: 'ru'
        }
      };
    }
  }
});
```

## Объекты в шаблонах

### Итерация по объектам

```vue
<template>
  <div>
    <!-- Итерация по свойствам объекта -->
    <div v-for="(value, key) in user.profile" :key="key" class="profile-field">
      <label>{{ key }}:</label>
      <input 
        :value="value" 
        @input="updateField(key, $event.target.value)"
        :type="getFieldType(key)"
      />
    </div>

    <!-- Использование объекта конфигурации -->
    <button 
      v-for="action in actions" 
      :key="action.name"
      :class="action.class"
      @click="action.handler"
      :disabled="action.disabled"
    >
      {{ action.label }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'ProfileEditor',
  data() {
    return {
      user: {
        profile: {
          firstName: 'Иван',
          lastName: 'Иванов',
          email: 'ivan@example.com'
        }
      },
      actions: [
        {
          name: 'save',
          label: 'Сохранить',
          class: 'btn btn-primary',
          handler: this.saveProfile,
          disabled: false
        },
        {
          name: 'cancel',
          label: 'Отмена',
          class: 'btn btn-secondary',
          handler: this.cancelEdit,
          disabled: false
        }
      ]
    };
  },
  methods: {
    updateField(key, value) {
      this.$set(this.user.profile, key, value);
      // или в Vue 3: this.user.profile[key] = value;
    },
    getFieldType(key) {
      if (key.includes('email')) return 'email';
      if (key.includes('password')) return 'password';
      return 'text';
    },
    saveProfile() {
      // логика сохранения
    },
    cancelEdit() {
      // логика отмены
    }
  }
};
</script>
```

## Объекты и производительность

### Использование toRefs для избежания ненужных рендеров

```javascript
import { reactive, toRefs, computed } from 'vue';

export default {
  setup() {
    const state = reactive({
      user: {
        id: 1,
        name: 'Иван',
        email: 'ivan@example.com'
      },
      settings: {
        theme: 'light',
        notifications: true
      }
    });

    // Используем toRefs для сохранения реактивности
    const refs = toRefs(state);

    // Вычисляемые свойства
    const userProfile = computed(() => ({
      ...state.user,
      displayName: `${state.user.name}`
    }));

    return {
      ...refs,
      userProfile
    };
  }
};
```

### Объекты с кэшированием

```javascript
import { ref, computed, watch } from 'vue';

export default {
  setup() {
    const rawData = ref([]);
    const filterConfig = ref({
      searchTerm: '',
      category: 'all',
      sortBy: 'name'
    });

    // Кэшированный отфильтрованный объект
    const filteredData = computed(() => {
      let result = [...rawData.value];

      if (filterConfig.value.searchTerm) {
        result = result.filter(item =>
          item.name.toLowerCase().includes(filterConfig.value.searchTerm.toLowerCase())
        );
      }

      if (filterConfig.value.category !== 'all') {
        result = result.filter(item => item.category === filterConfig.value.category);
      }

      result.sort((a, b) => {
        if (filterConfig.value.sortBy === 'name') {
          return a.name.localeCompare(b.name);
        }
        return a[filterConfig.value.sortBy] - b[filterConfig.value.sortBy];
      });

      return result;
    });

    const stats = computed(() => ({
      total: rawData.value.length,
      filtered: filteredData.value.length,
      categories: [...new Set(rawData.value.map(item => item.category))]
    }));

    return {
      rawData,
      filterConfig,
      filteredData,
      stats
    };
  }
};
```

## Практические рекомендации

### Объекты для конфигурации компонентов

```javascript
// Конфигурация таблицы данных
const tableConfig = {
  columns: [
    { key: 'id', label: 'ID', sortable: true },
    { key: 'name', label: 'Имя', sortable: true },
    { key: 'email', label: 'Email', sortable: false }
  ],
  pagination: {
    pageSize: 10,
    currentPage: 1
  },
  sorting: {
    field: 'name',
    direction: 'asc'
  },
  filters: {
    active: true,
    searchTerm: ''
  }
};

export default {
  name: 'DataTable',
  props: {
    config: {
      type: Object,
      default: () => ({ ...tableConfig })
    },
    data: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      localConfig: { ...this.config }
    };
  }
};
```

### Объекты для управления формами

```javascript
// Универсальный хук для форм
function useForm(initialValues, validationRules = {}) {
  const formState = reactive({
    values: { ...initialValues },
    errors: {},
    touched: {},
    isValid: true,
    isSubmitting: false
  });

  const validateField = (name, value) => {
    if (validationRules[name]) {
      const rule = validationRules[name];
      if (rule.required && !value) {
        return 'Поле обязательно для заполнения';
      }
      if (rule.pattern && !rule.pattern.test(value)) {
        return rule.message || 'Неверный формат';
      }
    }
    return null;
  };

  const updateField = (name, value) => {
    formState.values[name] = value;
    
    // Помечаем поле как "прикосновение"
    formState.touched[name] = true;
    
    // Валидируем поле
    const error = validateField(name, value);
    if (error) {
      formState.errors[name] = error;
    } else {
      delete formState.errors[name];
    }
    
    // Проверяем общую валидность
    formState.isValid = Object.keys(formState.errors).length === 0;
  };

  return {
    ...toRefs(formState),
    updateField
  };
}
```

## Заключение

Объекты в Vue-компонентах играют центральную роль в организации данных, состояния и логики. Правильное использование реактивных объектов, понимание различий между глубокой и поверхностной реактивностью, а также эффективное управление сложными состояниями критически важно для создания высокопроизводительных Vue-приложений.

См. также:
- [[Объекты-в-JavaScript]]
- [[Объекты-в-TypeScript]]
- [[Vue-компоненты]]
- [[Vue-Composition-API]]
- [[Объекты-и-их-свойства]]