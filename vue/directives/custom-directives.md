# Custom Directives

## Определение
Пользовательские директивы позволяют добавлять собственное поведение к DOM элементам. Vue предоставляет API для создания собственных директив, которые могут выполнять различные манипуляции с DOM в ответ на изменения данных.

## Типы директив

### Элементные директивы
Привязываются к конкретному DOM элементу и могут манипулировать им.

### Компонентные директивы
Применяются к компонентам, но имеют доступ к корневому элементу компонента.

## Создание директив

### Простая директива
```javascript
// focus.js
const focusDirective = {
  mounted(el) {
    el.focus()
  }
}

// Регистрация
app.directive('focus', focusDirective)

// Использование
<template>
  <input v-focus />
</template>
```

### Директива с параметрами
```javascript
// color.js
const colorDirective = {
  mounted(el, binding) {
    el.style.color = binding.value
  },
  updated(el, binding) {
    el.style.color = binding.value
  }
}

// Регистрация
app.directive('color', colorDirective)

// Использование
<template>
  <p v-color="currentColor">Текст с динамическим цветом</p>
</template>
```

## Жизненный цикл директив

### Хуки жизненного цикла
- `created` - вызывается до связывания элемента с узлом
- `beforeMount` - вызывается перед монтированием директивы
- `mounted` - вызывается после монтирования элемента
- `beforeUpdate` - вызывается перед обновлением содержащего элемента
- `updated` - вызывается после обновления содержащего элемента
- `beforeUnmount` - вызывается перед размонтированием элемента
- `unmounted` - вызывается после размонтирования элемента

### Пример с полным жизненным циклом
```javascript
// tooltip.js
const tooltipDirective = {
  created(el, binding) {
    // Создание вспомогательных элементов/обработчиков
  },
  mounted(el, binding) {
    // Установка начального состояния
    if (binding.value) {
      createTooltip(el, binding.value)
    }
  },
  updated(el, binding) {
    // Обновление при изменении значения
    if (binding.value !== binding.oldValue) {
      updateTooltip(el, binding.value)
    }
  },
  beforeUnmount(el) {
    // Очистка
    removeTooltip(el)
  }
}

function createTooltip(el, text) {
  // Логика создания тултипа
}

function updateTooltip(el, text) {
  // Логика обновления тултипа
}

function removeTooltip(el) {
  // Логика удаления тултипа
}
```

## Параметры директивы

### Binding объект
`binding` объект содержит следующие свойства:
- `value` - значение переданное директиве
- `oldValue` - предыдущее значение (только в beforeUpdate и updated)
- `arg` - аргумент директивы
- `modifiers` - модификаторы директивы
- `instance` - экземпляр компонента
- `dir` - определение директивы

### Использование параметров
```vue
<template>
  <!-- v-demo:foo.bar.baz="value" -->
  <div v-demo:my-argument="{ color: 'white', text: 'Привет!' }">Привет!</div>
</template>

<script>
const demoDirective = {
  mounted(el, binding) {
    console.log(binding.arg) // "my-argument"
    console.log(binding.value) // { color: 'white', text: 'Привет!' }
    console.log(binding.modifiers) // { bar: true, baz: true }
    
    el.style.color = binding.value.color
    el.textContent = binding.value.text
  }
}
</script>
```

## Глобальная регистрация

### В приложении
```javascript
// main.js
import { createApp } from 'vue'
import { focusDirective } from './directives/focus'

const app = createApp(App)

app.directive('focus', focusDirective)
app.directive('tooltip', tooltipDirective)
```

## Локальная регистрация

### В компоненте
```vue
<script setup>
import { ref } from 'vue'

// Локальная директива
const vFocus = {
  mounted(el) {
    el.focus()
  }
}

// Использование
const inputRef = ref(null)

const focusInput = () => {
  inputRef.value.focus() // Альтернативный способ
}
</script>

<template>
  <input v-focus ref="inputRef" />
</template>
```

## Практические примеры

### Директива для обработки кликов вне элемента
```javascript
// click-outside.js
const clickOutsideDirective = {
  beforeMount(el, binding) {
    el.clickOutsideEvent = (event) => {
      if (!(el === event.target || el.contains(event.target))) {
        binding.value(event)
      }
    }
    document.addEventListener('click', el.clickOutsideEvent)
  },
  unmounted(el) {
    document.removeEventListener('click', el.clickOutsideEvent)
  }
}

// Использование
<template>
  <div v-click-outside="handleClickOutside" class="dropdown">
    <!-- содержимое dropdown -->
  </div>
</template>
```

### Директива для lazy loading изображений
```javascript
// lazy-load.js
const lazyLoadDirective = {
  mounted(el, binding) {
    const img = new Image()
    img.onload = () => {
      el.src = binding.value
      el.classList.add('loaded')
    }
    img.src = binding.value
  }
}

// Использование
<template>
  <img v-lazy="imageSrc" alt="описание" />
</template>
```

### Директива для управления прокруткой
```javascript
// scroll-lock.js
const scrollLockDirective = {
  mounted(el, binding) {
    if (binding.value) {
      document.body.style.overflow = 'hidden'
    }
  },
  updated(el, binding) {
    if (binding.value) {
      document.body.style.overflow = 'hidden'
    } else {
      document.body.style.overflow = ''
    }
  },
  unmounted() {
    document.body.style.overflow = ''
  }
}
```

## Composition API и директивы

### Использование в setup()
```vue
<script setup>
import { onMounted, getCurrentInstance } from 'vue'

// Регистрация директивы в компоненте
const instance = getCurrentInstance()
instance.appContext.app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})
</script>
```

### Директивы как композаблы
```javascript
// useDirective.js
export function useClickOutside(callback) {
  const directive = {
    mounted(el) {
      el.clickOutsideEvent = (event) => {
        if (!(el === event.target || el.contains(event.target))) {
          callback(event)
        }
      }
      document.addEventListener('click', el.clickOutsideEvent)
    },
    unmounted(el) {
      document.removeEventListener('click', el.clickOutsideEvent)
    }
  }
  
  return directive
}
```

## Лучшие практики

### Назначение директив
- Используйте директивы для манипуляций с DOM
- Для логики лучше использовать композаблы
- Избегайте сложной логики в директивах
- Обязательно очищайте ресурсы в `unmounted`

### Производительность
```javascript
// Хорошо: минимальные операции в директиве
const resizeDirective = {
  mounted(el, binding) {
    const handler = () => binding.value()
    window.addEventListener('resize', handler)
    
    // Сохранение обработчика для очистки
    el.__resizeHandler__ = handler
  },
  unmounted(el) {
    window.removeEventListener('resize', el.__resizeHandler__)
  }
}
```

## Тестирование директив
```javascript
import { mount } from '@vue/test-utils'
import FocusDirective from '@/directives/focus'

test('фокусирует элемент при монтировании', () => {
  const wrapper = mount({
    template: '<input v-focus />',
    directives: {
      focus: FocusDirective
    }
  })
  
  expect(wrapper.element).toBe(document.activeElement)
})
```

## Сравнение с другими подходами

| Подход | Когда использовать |
|--------|-------------------|
| Директивы | Манипуляции с DOM |
| Композаблы | Логика и состояние |
| Компоненты | Пользовательский интерфейс |
| Props/Events | Передача данных |

## Связь с другими концепциями
- [[directives/v-model]] - директивы для форм
- [[components/Components]] - взаимодействие с компонентами
- [[reactivity/Reactivity System]] - реактивность в директивах
- [[performance/Performance]] - производительность директив