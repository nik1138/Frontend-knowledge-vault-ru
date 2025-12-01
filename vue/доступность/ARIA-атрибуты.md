---
aliases: ["ARIA", "Роль и состояние", "Доступные интерфейсы"]
tags: ["vue", "accessibility", "aria", "attributes"]
---

# ARIA-атрибуты в Vue-приложениях

## Введение

ARIA (Accessible Rich Internet Applications) - это набор атрибутов, которые помогают создавать доступные веб-приложения для пользователей с ограниченными возможностями. ARIA-атрибуты дополняют семантическую разметку, предоставляя дополнительную информацию вспомогательным технологиям.

## Основные категории ARIA-атрибутов

### Роли (Roles)

Роли описывают назначение элемента:

- `role="button"` - элемент ведет себя как кнопка
- `role="navigation"` - навигационная область
- `role="alert"` - важное сообщение
- `role="dialog"` - модальное окно

```vue
<template>
  <div class="custom-button" 
       role="button" 
       tabindex="0" 
       @click="handleClick"
       @keydown.enter="handleClick"
       @keydown.space="handleClick">
    {{ buttonText }}
  </div>
</template>

<script>
export default {
  name: 'AccessibleButton',
  props: {
    buttonText: {
      type: String,
      default: 'Кнопка'
    }
  },
  methods: {
    handleClick() {
      // Обработка клика
      this.$emit('click');
    }
  }
}
</script>
```

### Свойства (Properties)

Свойства описывают характеристики элемента:

- `aria-label` - текстовая метка элемента
- `aria-labelledby` - ссылка на элемент, который служит меткой
- `aria-describedby` - ссылка на элемент с описанием
- `aria-haspopup` - указывает, что элемент открывает всплывающее окно

```vue
<template>
  <div class="search-container">
    <input 
      type="text" 
      id="search-input"
      :aria-label="$t('search_placeholder')"
      :aria-describedby="searchHint ? 'search-hint' : null"
      v-model="searchQuery"
    >
    
    <div v-if="searchHint" id="search-hint" class="search-hint">
      {{ searchHint }}
    </div>
    
    <button 
      :aria-label="$t('search_button')"
      @click="performSearch">
      🔍
    </button>
  </div>
</template>

<script>
export default {
  name: 'AccessibleSearch',
  data() {
    return {
      searchQuery: '',
      searchHint: 'Введите минимум 3 символа для поиска'
    }
  },
  methods: {
    performSearch() {
      // Логика поиска
    }
  }
}
</script>
```

### Состояния (States)

Состояния описывают текущее состояние элемента:

- `aria-disabled` - элемент недоступен
- `aria-expanded` - элемент развернут/свернут
- `aria-checked` - состояние чекбокса/радио-кнопки
- `aria-selected` - элемент выбран

```vue
<template>
  <div class="accordion-item">
    <button 
      class="accordion-header"
      :aria-expanded="isOpen"
      :aria-controls="`panel-${id}`"
      @click="toggleAccordion">
      {{ title }}
    </button>
    
    <div 
      :id="`panel-${id}`"
      class="accordion-panel"
      :hidden="!isOpen"
      role="region"
      :aria-labelledby="`header-${id}`">
      <slot></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'AccessibleAccordion',
  props: {
    title: String,
    id: String
  },
  data() {
    return {
      isOpen: false
    }
  },
  methods: {
    toggleAccordion() {
      this.isOpen = !this.isOpen;
    }
  }
}
</script>
```

## Практические примеры использования

### 1. Динамические ARIA-атрибуты

```vue
<template>
  <div class="form-group">
    <label :for="inputId">{{ label }}</label>
    <input
      :id="inputId"
      :type="type"
      :value="modelValue"
      :aria-invalid="hasError"
      :aria-describedby="hasError ? `${inputId}-error` : null"
      @input="$emit('update:modelValue', $event.target.value)"
    >
    
    <div v-if="hasError" 
         :id="`${inputId}-error`" 
         class="error-message"
         role="alert">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'AccessibleInput',
  props: {
    label: String,
    modelValue: String,
    type: {
      type: String,
      default: 'text'
    },
    hasError: Boolean,
    errorMessage: String
  },
  computed: {
    inputId() {
      return `input-${this._uid}`;
    }
  }
}
</script>
```

### 2. Сложные интерактивные компоненты

```vue
<template>
  <div class="tabs" role="tablist">
    <button
      v-for="(tab, index) in tabs"
      :key="tab.id"
      :id="`tab-${tab.id}`"
      :ref="`tab-${tab.id}`"
      role="tab"
      :aria-selected="activeTab === index"
      :aria-controls="`panel-${tab.id}`"
      :class="{ active: activeTab === index }"
      @click="selectTab(index)"
      @keydown="handleTabKeydown($event, index)">
      {{ tab.title }}
    </button>
    
    <div
      v-for="(tab, index) in tabs"
      :key="`panel-${tab.id}`"
      :id="`panel-${tab.id}`"
      role="tabpanel"
      :aria-labelledby="`tab-${tab.id}`"
      :hidden="activeTab !== index">
      <component :is="tab.component" v-if="activeTab === index" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'AccessibleTabs',
  props: {
    tabs: Array
  },
  data() {
    return {
      activeTab: 0
    }
  },
  methods: {
    selectTab(index) {
      this.activeTab = index;
    },
    handleTabKeydown(event, index) {
      switch (event.key) {
        case 'ArrowRight':
          this.selectTab((index + 1) % this.tabs.length);
          break;
        case 'ArrowLeft':
          this.selectTab((index - 1 + this.tabs.length) % this.tabs.length);
          break;
        case 'Home':
          this.selectTab(0);
          break;
        case 'End':
          this.selectTab(this.tabs.length - 1);
          break;
      }
    }
  }
}
</script>
```

### 3. Уведомления и алерты

```vue
<template>
  <div>
    <div 
      v-if="notification"
      :role="notification.type === 'alert' ? 'alert' : 'status'"
      :aria-live="notification.politeness || 'polite'"
      class="notification"
      :class="notification.type">
      {{ notification.message }}
    </div>
    
    <button @click="showNotification">Показать уведомление</button>
  </div>
</template>

<script>
export default {
  name: 'AccessibleNotifications',
  data() {
    return {
      notification: null
    }
  },
  methods: {
    showNotification() {
      this.notification = {
        type: 'alert',
        message: 'Операция выполнена успешно!',
        politeness: 'assertive'
      };
      
      // Автоматическое скрытие через 5 секунд
      setTimeout(() => {
        this.notification = null;
      }, 5000);
    }
  }
}
</script>
```

## Особенности для российских разработчиков

### Требования законодательства

С 2025 года в России усилен контроль за доступностью веб-сайтов согласно:

- Федеральный закон №419-ФЗ от 25.11.2024
- ГОСТ Р 52872-2022 (новая редакция)
- Указания ФСТЭК и Роскомнадзора

### Локализация ARIA-атрибутов

```vue
<template>
  <button 
    :aria-label="$t('close_modal')"
    @click="closeModal">
    ×
  </button>
</template>

<script>
import { useI18n } from 'vue-i18n';

export default {
  setup() {
    const { t } = useI18n();
    return { t };
  },
  methods: {
    closeModal() {
      this.$emit('close');
    }
  }
}
</script>
```

## Наилучшие практики

1. **Используйте семантические элементы в первую очередь**
   - Только если семантические элементы не подходят, добавляйте ARIA

2. **Не переопределяйте семантику**
   - Не используйте `role="button"` на ссылке, если это не кнопка

3. **Обновляйте ARIA-атрибуты динамически**
   - Используйте Vue-реактивность для обновления состояний

4. **Тестируйте с вспомогательными технологиями**
   - Проверяйте работу с JAWS, NVDA, VoiceOver

## Заключение

ARIA-атрибуты являются мощным инструментом для создания доступных интерфейсов в Vue-приложениях. Правильное использование ARIA позволяет пользователям с ограниченными возможностями эффективно взаимодействовать с веб-приложениями.

Следующие темы для изучения:
- [[Семантическая-разметка]]
- [[Навигация-с-клавиатуры]]
- [[Контрастность]]
- [[Тестирование-доступности]]

## Ключевые теги
#vue #accessibility #aria #frontend #web-development #semantics