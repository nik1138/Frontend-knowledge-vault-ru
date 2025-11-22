---
aliases: [Vue Hooks, Хуки Vue, Вью Хуки]
tags: [vue, hooks, frontend, javascript]
---

# Хуки в Vue

## Введение

Vue.js не использует термин "хуки" в том же смысле, что и React, но имеет похожие концепции для управления состоянием и жизненным циклом компонентов. В Vue 3 с появлением Composition API были введены функции, аналогичные хукам React, которые позволяют более гибко управлять логикой компонентов.

## Composition API

Composition API - это набор API, который позволяет определять логику компонента с использованием импортируемых функций, а не объявлять свойства и методы в объекте `data`, `methods`, и т.д.

### setup()

Функция `setup()` - это точка входа для Composition API. Она вызывается перед созданием компонента и позволяет определить реактивное состояние и методы.

```vue
<template>
  <div>
    <p>Счетчик: {{ count }}</p>
    <button @click="increment">Увеличить</button>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const count = ref(0);

    const increment = () => {
      count.value++;
    };

    // Возвращаем значения, которые будут доступны в шаблоне
    return {
      count,
      increment
    };
  }
};
</script>
```

### ref и reactive

`ref` и `reactive` - основные функции для создания реактивного состояния.

```javascript
import { ref, reactive } from 'vue';

// ref - для примитивных значений
const count = ref(0);

// reactive - для объектов
const state = reactive({
  count: 0,
  name: 'Vue'
});

// Для доступа к значению ref используем .value
console.log(count.value); // 0
count.value++;

// reactive объекты доступны напрямую
console.log(state.count); // 0
state.count++;
```

## Жизненный цикл компонента

В Composition API хуки жизненного цикла регистрируются как функции с префиксом `on`.

```javascript
import { 
  onMounted, 
  onUpdated, 
  onUnmounted, 
  ref 
} from 'vue';

export default {
  setup() {
    const element = ref(null);

    onMounted(() => {
      console.log('Компонент смонтирован');
      // Доступ к DOM элементу
      console.log(element.value);
    });

    onUpdated(() => {
      console.log('Компонент обновлен');
    });

    onUnmounted(() => {
      console.log('Компонент будет размонтирован');
    });

    return {
      element
    };
  }
};
```

## computed и watch

### computed

`computed` позволяет создавать вычисляемые значения, которые автоматически обновляются при изменении зависимостей.

```javascript
import { ref, computed } from 'vue';

export default {
  setup() {
    const firstName = ref('John');
    const lastName = ref('Doe');

    const fullName = computed(() => `${firstName.value} ${lastName.value}`);

    return {
      firstName,
      lastName,
      fullName
    };
  }
};
```

### watch и watchEffect

`watch` позволяет отслеживать изменения реактивных значений, а `watchEffect` автоматически отслеживает зависимости.

```javascript
import { ref, watch, watchEffect } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const double = ref(0);

    // watch - отслеживает конкретные значения
    watch(count, (newVal, oldVal) => {
      console.log(`count изменился с ${oldVal} на ${newVal}`);
      double.value = newVal * 2;
    });

    // watchEffect - автоматически отслеживает зависимости
    watchEffect(() => {
      console.log(`Значение count: ${count.value}`);
    });

    return {
      count,
      double
    };
  }
};
```

## Пользовательские композиции (Custom Composables)

В Vue функции, использующие Composition API, часто называют "composables" или "composables functions". Это аналог пользовательских хуков в React.

```javascript
// composables/useCounter.js
import { ref } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);

  const increment = () => {
    count.value++;
  };

  const decrement = () => {
    count.value--;
  };

  const reset = () => {
    count.value = initialValue;
  };

  return {
    count,
    increment,
    decrement,
    reset
  };
}

// Использование в компоненте
import { useCounter } from '@/composables/useCounter';

export default {
  setup() {
    const { count, increment, decrement, reset } = useCounter(10);

    return {
      count,
      increment,
      decrement,
      reset
    };
  }
};
```

## provide и inject

`provide` и `inject` позволяют передавать данные между компонентами без пропсов.

```javascript
// Родительский компонент
import { provide, ref } from 'vue';

export default {
  setup() {
    const theme = ref('dark');

    provide('theme', theme);
    provide('changeTheme', (newTheme) => {
      theme.value = newTheme;
    });
  }
};

// Дочерний компонент
import { inject } from 'vue';

export default {
  setup() {
    const theme = inject('theme');
    const changeTheme = inject('changeTheme');

    return {
      theme,
      changeTheme
    };
  }
};
```

## Особенности и лучшие практики

1. **Организация логики**: Composition API позволяет группировать связанную логику в одном месте, что улучшает читаемость кода.

2. **Переиспользование логики**: Composables позволяют легко делить и переиспользовать состояние и логику между компонентами.

3. **Типизация**: Composition API хорошо работает с TypeScript, позволяя создавать типизированные композиции.

4. **Производительность**: Composition API может улучшить производительность за счет более эффективного отслеживания зависимостей.

## Заключение

Composition API в Vue 3 предоставляет мощный способ управления логикой компонентов, аналогичный хукам в React. Он позволяет создавать более читаемый, переиспользуемый и масштабируемый код.

См. также:
- [[Пользовательские-хуки]]
- [[Лучшие-практики-хуков]]
- [[Хуки-в-React]]
- [[Хуки-в-Svelte]]