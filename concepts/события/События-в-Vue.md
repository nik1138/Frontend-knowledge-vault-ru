---
aliases: [События в Vue, Vue Events, Обработка событий в Vue]
tags: [vue, javascript, events, frontend, programming]
---

# События-в-Vue

## Введение

Vue.js предоставляет мощную и интуитивно понятную систему обработки событий, которая позволяет легко реагировать на действия пользователя и другие события. Обработка событий в Vue сочетает в себе простоту использования с гибкостью и возможностями масштабирования для сложных приложений.

## Базовая обработка событий

### Директива v-on

В Vue для обработки событий используется директива `v-on` или её сокращённая форма `@`:

```vue
<template>
  <div>
    <!-- Полная форма -->
    <button v-on:click="handleClick">Кликни меня</button>
    
    <!-- Сокращённая форма -->
    <button @click="handleClick">Кликни меня</button>
    
    <!-- С inline-выражением -->
    <button @click="count++">Счётчик: {{ count }}</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    handleClick() {
      console.log('Кнопка нажата!');
    }
  }
};
</script>
```

## Передача аргументов в обработчики

### Передача событий

```vue
<template>
  <div>
    <!-- Передача нативного события -->
    <button @click="handleClick">Кликни меня</button>
    
    <!-- Передача дополнительных аргументов -->
    <button @click="handleClickWithArgs('Vue', $event)">
      Клик с аргументами
    </button>
  </div>
</template>

<script>
export default {
  methods: {
    handleClick(event) {
      console.log('Клик!', event.target);
    },
    handleClickWithArgs(framework, event) {
      console.log(`Клик по ${framework}`, event);
    }
  }
};
</script>
```

## Модификаторы событий

Vue предоставляет удобные модификаторы для обработки распространённых задач:

```vue
<template>
  <div>
    <!-- .stop - останавливает всплытие события -->
    <div @click="outerClick">
      Внешний div
      <button @click.stop="innerClick">Кнопка</button>
    </div>
    
    <!-- .prevent - предотвращает действие по умолчанию -->
    <form @submit.prevent="handleSubmit">
      <input type="text" name="username">
      <button type="submit">Отправить</button>
    </form>
    
    <!-- .capture - использует фазу захвата -->
    <div @click.capture="handleCapture">
      <button @click="handleClick">Кнопка</button>
    </div>
    
    <!-- .self - обработчик сработает только если событие произошло на этом элементе -->
    <div @click.self="handleSelf">
      <button @click="handleChild">Кнопка</button>
    </div>
    
    <!-- .once - обработчик сработает только один раз -->
    <button @click.once="handleOnce">Один раз</button>
    
    <!-- .passive - указывает, что обработчик не вызывает preventDefault -->
    <div @scroll.passive="handleScroll" style="height: 100px; overflow: auto;">
      Длинный контент...
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    outerClick() {
      console.log('Внешний клик');
    },
    innerClick() {
      console.log('Внутренний клик');
      // Всплытие остановлено, внешний клик не сработает
    },
    handleSubmit() {
      console.log('Форма отправлена (но не на сервер)');
      // preventDefault уже вызван
    },
    handleCapture() {
      console.log('Захват события');
    },
    handleClick() {
      console.log('Обычный клик');
    },
    handleSelf() {
      console.log('Клик на самом элементе');
    },
    handleChild() {
      console.log('Клик на дочернем элементе');
    },
    handleOnce() {
      console.log('Это сообщение появится только один раз');
    },
    handleScroll() {
      console.log('Прокрутка');
    }
  }
};
</script>
```

## Модификаторы клавиатуры

Для обработки клавиатурных событий Vue предоставляет модификаторы клавиш:

```vue
<template>
  <div>
    <!-- Обычные клавиши -->
    <input @keyup.enter="submitForm" placeholder="Нажмите Enter">
    <input @keyup.space="handleSpace" placeholder="Нажмите пробел">
    
    <!-- Системные клавиши -->
    <input @keydown.ctrl.c="copy" placeholder="Ctrl+C">
    <input @keydown.alt.enter="specialAction" placeholder="Alt+Enter">
    
    <!-- Комбинации клавиш -->
    <input @keydown.ctrl.shift.t="toggleTheme" placeholder="Ctrl+Shift+T">
    
    <!-- Специальные клавиши -->
    <input @keydown.esc="closeModal" placeholder="Нажмите Esc">
    <input @keydown.tab="handleTab" placeholder="Нажмите Tab">
    
    <!-- Кастомные коды клавиш -->
    <input @keydown.13="handleKey13" placeholder="Клавиша с кодом 13">
    
    <!-- .exact модификатор - только указанная клавиша без модификаторов -->
    <div @click.ctrl.exact="handleCtrlClick">
      Клик с Ctrl (без других модификаторов)
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    submitForm() {
      console.log('Форма отправлена');
    },
    handleSpace() {
      console.log('Пробел нажат');
    },
    copy() {
      console.log('Копирование');
    },
    specialAction() {
      console.log('Специальное действие');
    },
    toggleTheme() {
      console.log('Переключение темы');
    },
    closeModal() {
      console.log('Закрытие модального окна');
    },
    handleTab() {
      console.log('Tab нажат');
    },
    handleKey13() {
      console.log('Клавиша 13 нажата');
    },
    handleCtrlClick() {
      console.log('Ctrl+Click без других модификаторов');
    }
  }
};
</script>
```

## Работа с формами

### Обработка событий формы

```vue
<template>
  <form @submit.prevent="handleSubmit">
    <div>
      <label>Имя:</label>
      <input 
        v-model="formData.name" 
        @blur="validateName"
        @input="clearNameError"
      >
      <span v-if="errors.name" class="error">{{ errors.name }}</span>
    </div>
    
    <div>
      <label>Email:</label>
      <input 
        v-model="formData.email" 
        @blur="validateEmail"
        @input="clearEmailError"
      >
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <label>Сообщение:</label>
      <textarea 
        v-model="formData.message" 
        @input="handleMessageChange"
      ></textarea>
    </div>
    
    <button 
      type="submit" 
      :disabled="!isFormValid"
    >
      Отправить
    </button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      formData: {
        name: '',
        email: '',
        message: ''
      },
      errors: {
        name: '',
        email: ''
      }
    };
  },
  computed: {
    isFormValid() {
      return this.formData.name && 
             this.isValidEmail(this.formData.email) && 
             this.formData.message;
    }
  },
  methods: {
    handleSubmit() {
      if (this.isFormValid) {
        console.log('Данные формы:', this.formData);
        // Отправка данных
      } else {
        console.log('Форма содержит ошибки');
      }
    },
    validateName() {
      if (!this.formData.name.trim()) {
        this.errors.name = 'Имя обязательно';
      }
    },
    clearNameError() {
      this.errors.name = '';
    },
    validateEmail() {
      if (!this.isValidEmail(this.formData.email)) {
        this.errors.email = 'Некорректный email';
      }
    },
    clearEmailError() {
      this.errors.email = '';
    },
    isValidEmail(email) {
      const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return re.test(email);
    },
    handleMessageChange() {
      // Логика при изменении сообщения
      console.log('Сообщение изменено');
    }
  }
};
</script>
```

## Компонентные события

### Создание и вызов пользовательских событий

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <button @click="notifyParent">Уведомить родителя</button>
    <button @click="sendData">Отправить данные</button>
  </div>
</template>

<script>
export default {
  name: 'ChildComponent',
  methods: {
    notifyParent() {
      // Вызов пользовательского события
      this.$emit('child-clicked');
    },
    sendData() {
      // Отправка данных с событием
      const data = {
        message: 'Привет от ребенка',
        timestamp: new Date()
      };
      this.$emit('data-from-child', data);
    }
  }
};
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <div>
    <h3>Родительский компонент</h3>
    <ChildComponent 
      @child-clicked="handleChildClick"
      @data-from-child="handleChildData"
    />
    <p>Сообщения от ребенка: {{ messages }}</p>
  </div>
</template>

<script>
import ChildComponent from './ChildComponent.vue';

export default {
  name: 'ParentComponent',
  components: {
    ChildComponent
  },
  data() {
    return {
      messages: []
    };
  },
  methods: {
    handleChildClick() {
      console.log('Ребёнок был кликнут');
      this.messages.push('Клик в ребёнке');
    },
    handleChildData(data) {
      console.log('Получены данные от ребёнка:', data);
      this.messages.push(data.message);
    }
  }
};
</script>
```

## Работа с событиями в Composition API

### Vue 3 Composition API

```vue
<template>
  <div>
    <p>Счётчик: {{ count }}</p>
    <button @click="increment">Увеличить</button>
    <button @click="decrement">Уменьшить</button>
    
    <input 
      v-model="inputValue" 
      @keyup.enter="addItem"
      placeholder="Добавить элемент"
    >
    
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.text }}
        <button @click="removeItem(item.id)">Удалить</button>
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const inputValue = ref('');
    const items = ref([]);
    let nextId = 1;

    const increment = () => {
      count.value++;
    };

    const decrement = () => {
      count.value--;
    };

    const addItem = () => {
      if (inputValue.value.trim()) {
        items.value.push({
          id: nextId++,
          text: inputValue.value
        });
        inputValue.value = '';
      }
    };

    const removeItem = (id) => {
      items.value = items.value.filter(item => item.id !== id);
    };

    return {
      count,
      inputValue,
      items,
      increment,
      decrement,
      addItem,
      removeItem
    };
  }
};
</script>
```

## Делегирование событий

Vue автоматически использует делегирование событий для эффективности:

```vue
<template>
  <div @click="handleClick">
    <button data-action="save">Сохранить</button>
    <button data-action="delete">Удалить</button>
    <button data-action="edit">Редактировать</button>
  </div>
</template>

<script>
export default {
  methods: {
    handleClick(event) {
      const action = event.target.dataset.action;
      
      switch(action) {
        case 'save':
          this.handleSave();
          break;
        case 'delete':
          this.handleDelete();
          break;
        case 'edit':
          this.handleEdit();
          break;
      }
    },
    handleSave() {
      console.log('Сохранение данных');
    },
    handleDelete() {
      console.log('Удаление данных');
    },
    handleEdit() {
      console.log('Редактирование данных');
    }
  }
};
</script>
```

## Работа с событиями вне компонента

### Использование lifecycle hooks для управления событиями

```vue
<template>
  <div>
    <p>Ширина окна: {{ windowWidth }}</p>
    <p>Высота окна: {{ windowHeight }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight
    };
  },
  mounted() {
    // Добавляем обработчик события resize при монтировании
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    // Удаляем обработчик при размонтировании для предотвращения утечек
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    handleResize() {
      this.windowWidth = window.innerWidth;
      this.windowHeight = window.innerHeight;
    }
  }
};
</script>
```

## Практические примеры

### Пример 1: Кастомный компонент с событиями

```vue
<!-- ToggleSwitch.vue -->
<template>
  <div 
    class="toggle-switch" 
    :class="{ active: modelValue }"
    @click="toggle"
  >
    <div class="toggle-slider" :class="{ on: modelValue }">
      <span class="toggle-text">{{ modelValue ? 'Вкл' : 'Выкл' }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ToggleSwitch',
  props: {
    modelValue: {
      type: Boolean,
      default: false
    }
  },
  emits: ['update:modelValue'], // Объявление генерируемых событий
  methods: {
    toggle() {
      // Использование v-model эквивалентно обновлению через событие
      this.$emit('update:modelValue', !this.modelValue);
    }
  }
};
</script>

<style scoped>
.toggle-switch {
  display: inline-block;
  cursor: pointer;
  padding: 5px;
}

.toggle-slider {
  width: 60px;
  height: 30px;
  background-color: #ccc;
  border-radius: 15px;
  position: relative;
  transition: background-color 0.3s;
}

.toggle-slider.on {
  background-color: #4CAF50;
}

.toggle-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 12px;
  color: white;
}
</style>
```

### Пример 2: Использование toggle switch

```vue
<template>
  <div>
    <h3>Настройки</h3>
    <div>
      <label>Тёмная тема:</label>
      <ToggleSwitch v-model="darkMode" />
    </div>
    <div>
      <label>Уведомления:</label>
      <ToggleSwitch v-model="notifications" />
    </div>
  </div>
</template>

<script>
import ToggleSwitch from './ToggleSwitch.vue';

export default {
  components: {
    ToggleSwitch
  },
  data() {
    return {
      darkMode: false,
      notifications: true
    };
  },
  watch: {
    darkMode(newVal) {
      if (newVal) {
        document.body.classList.add('dark-theme');
      } else {
        document.body.classList.remove('dark-theme');
      }
    }
  }
};
</script>
```

## Заключение

Система обработки событий в Vue предоставляет мощные и гибкие возможности для создания интерактивных приложений. Модификаторы событий, компонентные события и поддержка Composition API делают работу с событиями интуитивно понятной и эффективной.

## Смежные темы

- [[Обработка-событий]]
- [[Всплытие-и-перехват-событий]]
- [[События-в-React]]
- [[События-в-Svelte]]
- [[Компонентная-архитектура]]
- [[Фреймворки]]
- [[Директивы]]