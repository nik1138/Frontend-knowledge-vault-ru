---
aliases: [Vue Effects, Vue Composition API, Эффекты в Вью]
tags: [vue, frontend, effects, composition-api]
---

# Эффекты-в-Vue

## Введение

В Vue.js эффекты - это реактивные вычисления, которые автоматически запускаются при изменении зависимостей. В отличие от React, Vue предоставляет встроенную реактивную систему, что делает работу с эффектами более интуитивной.

## Composition API и эффекты

С приходом Composition API Vue предоставляет несколько способов работы с эффектами:

### onMounted, onUpdated, onUnmounted

Эти функции являются эквивалентом методов жизненного цикла из Options API:

```javascript
import { ref, onMounted, onUpdated, onUnmounted } from 'vue';

export default {
  setup() {
    const count = ref(0);

    onMounted(() => {
      console.log('Компонент смонтирован');
    });

    onUpdated(() => {
      console.log('Компонент обновлен');
    });

    onUnmounted(() => {
      console.log('Компонент будет размонтирован');
    });

    return { count };
  }
};
```

### watch и watchEffect

`watch` позволяет отслеживать конкретные реактивные значения:

```javascript
import { ref, watch, watchEffect } from 'vue';

export default {
  setup() {
    const question = ref('');
    const answer = ref('Вопросы обычно содержат "?"');

    // watch - отслеживает конкретное значение
    watch(question, async (newQuestion, oldQuestion) => {
      if (newQuestion.indexOf('?') > -1) {
        answer.value = 'Думаю...';
        try {
          const res = await fetch('https://yesno.wtf/api');
          answer.value = (await res.json()).answer;
        } catch (error) {
          answer.value = 'Ошибка! Не могу получить ответ =(';
        }
      }
    });

    // watchEffect - отслеживает все реактивные зависимости автоматически
    watchEffect(async () => {
      const response = await fetch(`https://api.dictionary.com/meaning/${question.value}`);
      // Автоматически перезапускается при изменении question.value
    });

    return {
      question,
      answer
    };
  }
};
```

## Подробнее о watchEffect

`watchEffect` особенно полезен, когда вы не знаете, какие зависимости будут использоваться:

```javascript
import { ref, reactive, watchEffect } from 'vue';

export default {
  setup() {
    const searchQuery = ref('');
    const results = ref([]);

    watchEffect(async () => {
      if (searchQuery.value) {
        results.value = await searchAPI(searchQuery.value);
      }
    });

    return {
      searchQuery,
      results
    };
  }
};
```

## Очистка эффектов

В `watch` и `watchEffect` можно возвращать функцию очистки:

```javascript
import { watchEffect } from 'vue';

export default {
  setup() {
    const subscription = ref(null);

    watchEffect((onInvalidate) => {
      const currentSubscription = subscribeToSource();
      subscription.value = currentSubscription;

      // Функция очистки
      onInvalidate(() => {
        currentSubscription.unsubscribe();
      });
    });

    return { subscription };
  }
};
```

## Практические примеры

### Работа с API

```javascript
import { ref, watch, onMounted } from 'vue';

export default {
  setup() {
    const userId = ref(null);
    const user = ref(null);
    const loading = ref(false);

    const fetchUser = async (id) => {
      loading.value = true;
      try {
        const response = await fetch(`/api/users/${id}`);
        user.value = await response.json();
      } catch (error) {
        console.error('Ошибка загрузки пользователя:', error);
      } finally {
        loading.value = false;
      }
    };

    // Отслеживаем изменение userId и загружаем пользователя
    watch(userId, (newId) => {
      if (newId) {
        fetchUser(newId);
      }
    });

    onMounted(() => {
      if (userId.value) {
        fetchUser(userId.value);
      }
    });

    return {
      userId,
      user,
      loading
    };
  }
};
```

### Подписка на события DOM

```javascript
import { ref, onMounted, onUnmounted } from 'vue';

export default {
  setup() {
    const windowWidth = ref(window.innerWidth);

    const handleResize = () => {
      windowWidth.value = window.innerWidth;
    };

    onMounted(() => {
      window.addEventListener('resize', handleResize);
    });

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize);
    });

    return { windowWidth };
  }
};
```

## watch vs watchEffect

| Характеристика | watch | watchEffect |
|----------------|-------|-------------|
| Отслеживание зависимостей | Явное | Автоматическое |
| Доступ к старому/новому значению | Да | Нет (только через замыкание) |
| Ленивое выполнение | Да | Да |
| Подходит для | Сложных вычислений | Простых сайд-эффектов |

## Оптимизация эффектов

### Опции watch

```javascript
import { watch } from 'vue';

watch(
  source, // источник
  callback,
  {
    immediate: true,    // запускать сразу
    deep: true,         // глубокое наблюдение
    flush: 'post',      // когда запускать ('pre', 'post', 'sync')
  }
);
```

### flush опции

- `pre` - перед обновлением DOM (по умолчанию)
- `post` - после обновления DOM
- `sync` - синхронно (редко используется)

```javascript
import { watch } from 'vue';

watch(
  () => componentData.value,
  (newData) => {
    // выполнится после обновления DOM
  },
  { flush: 'post' }
);
```

## Распространенные ошибки

### 1. Бесконечные циклы

```javascript
// ПЛОХО - может вызвать бесконечный цикл
watch(count, (newVal) => {
  count.value = newVal + 1; // изменение зависимости внутри watch
});

// ХОРОШО - проверка условия
watch(count, (newVal, oldVal) => {
  if (newVal !== oldVal + 1) {
    count.value = newVal + 1;
  }
});
```

### 2. Утечки памяти

```javascript
// ПЛОХО - не очищаем подписки
onMounted(() => {
  window.addEventListener('resize', handleResize);
  // Никогда не удаляем событие!
});

// ХОРОШО - очищаем ресурсы
onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});
```

## Сравнение с React

| React | Vue |
|-------|-----|
| useEffect | watch, watchEffect, onMounted и др. |
| Один хук для всех эффектов | Разные функции для разных случаев |
| Явные зависимости | Автоматическое или явное отслеживание |

## Заключение

Эффекты в Vue обеспечивают мощную и гибкую систему для работы с побочными эффектами. Composition API предоставляет более явные и контролируемые способы управления эффектами по сравнению с Options API.

См. также:
- [[Побочные-эффекты]]
- [[Управление-эффектами]]
- [[Эффекты-в-React]]
- [[Эффекты-в-Svelte]]