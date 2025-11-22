---
aliases: [Vue директивы, Vue Directives, Директивы Vue]
tags: [vue, frontend, directives, javascript]
---

# Директивы в Vue

## Обзор

Директивы в Vue.js - это специальные атрибуты с префиксом `v-`, которые предоставляют декларативный способ применения изменений DOM или поведения к элементам DOM при изменении состояния приложения. Они являются одной из ключевых особенностей Vue, позволяя легко управлять отображением данных, обработкой событий и другими аспектами пользовательского интерфейса.

## Встроенные директивы

### v-text
Устанавливает текстовое содержимое элемента. Это эквивалентно привязке `textContent`.

```vue
<template>
  <p v-text="message"></p>
</template>

<script>
export default {
  data() {
    return {
      message: 'Привет, Vue!'
    }
  }
}
</script>
```

### v-html
Устанавливает HTML содержимое элемента. Будьте осторожны при использовании этой директивы с непроверенными данными, так как это может привести к XSS-атакам.

```vue
<template>
  <div v-html="rawHtml"></div>
</template>

<script>
export default {
  data() {
    return {
      rawHtml: '<span style="color: red">Красный текст</span>'
    }
  }
}
</script>
```

### v-show
Условное отображение - переключает CSS свойство `display` элемента.

```vue
<template>
  <div v-show="isVisible">Этот элемент может быть скрыт/показан</div>
  <button @click="toggleVisibility">Переключить видимость</button>
</template>

<script>
export default {
  data() {
    return {
      isVisible: true
    }
  },
  methods: {
    toggleVisibility() {
      this.isVisible = !this.isVisible
    }
  }
}
</script>
```

### v-if, v-else, v-else-if
Условная отрисовка - полностью добавляет или удаляет элемент из DOM.

```vue
<template>
  <div v-if="type === 'A'">A</div>
  <div v-else-if="type === 'B'">B</div>
  <div v-else-if="type === 'C'">C</div>
  <div v-else>Не A/B/C</div>
</template>

<script>
export default {
  data() {
    return {
      type: 'A'
    }
  }
}
</script>
```

### v-for
Отрисовка списков - позволяет повторять элементы на основе массива или объекта.

```vue
<template>
  <ul>
    <li v-for="(item, index) in items" :key="index">
      {{ index }} - {{ item.name }}
    </li>
  </ul>
  
  <div v-for="value in object" :key="value">
    {{ value }}
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { name: 'Яблоко' },
        { name: 'Банан' },
        { name: 'Апельсин' }
      ],
      object: {
        title: 'Мой заголовок',
        description: 'Мое описание'
      }
    }
  }
}
</script>
```

### v-on
Обработка событий - позволяет слушать DOM события и запускать JavaScript код при их возникновении.

```vue
<template>
  <button v-on:click="counter++">Добавить</button>
  <button @click="handleClick">Обработать клик</button>
  <form @submit.prevent="onSubmit">
    <input @keyup.enter="onEnter" />
    <input @keyup.esc="onEscape" />
  </form>
</template>

<script>
export default {
  data() {
    return {
      counter: 0
    }
  },
  methods: {
    handleClick(event) {
      console.log('Кнопка нажата', event)
    },
    onSubmit() {
      console.log('Форма отправлена')
    },
    onEnter() {
      console.log('Нажата клавиша Enter')
    },
    onEscape() {
      console.log('Нажата клавиша Escape')
    }
  }
}
</script>
```

### v-bind
Динамическая привязка атрибутов - позволяет связать атрибут или свойство элемента с динамическим значением.

```vue
<template>
  <img v-bind:src="imageSrc" :alt="imageAlt" />
  <div :class="{ active: isActive, 'text-danger': hasError }">
    Динамический класс
  </div>
  <button :disabled="isDisabled">Кнопка</button>
</template>

<script>
export default {
  data() {
    return {
      imageSrc: '/path/to/image.jpg',
      imageAlt: 'Описание изображения',
      isActive: true,
      hasError: false,
      isDisabled: true
    }
  }
}
</script>
```

### v-model
Двустороннее связывание данных - обеспечивает двустороннее связывание между элементом ввода и состоянием данных.

```vue
<template>
  <input v-model="message" placeholder="Введите сообщение">
  <p>Сообщение: {{ message }}</p>
  
  <textarea v-model="textarea"></textarea>
  
  <input type="checkbox" v-model="checked">
  <label>Флажок: {{ checked }}</label>
  
  <input type="radio" v-model="picked" value="один"> Один
  <input type="radio" v-model="picked" value="два"> Два
  
  <select v-model="selected">
    <option disabled value="">Выберите один</option>
    <option>А</option>
    <option>Б</option>
    <option>В</option>
  </select>
</template>

<script>
export default {
  data() {
    return {
      message: '',
      textarea: '',
      checked: false,
      picked: '',
      selected: ''
    }
  }
}
</script>
```

### v-slot
Слоты - позволяют авторам компонентов определять места, куда могут быть помещены содержимое.

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <template #header>
      <h1>Заголовок</h1>
    </template>
    
    <p>Содержимое по умолчанию</p>
    
    <template #footer>
      <p>Футер</p>
    </template>
  </ChildComponent>
</template>
```

## Модификаторы директив

Многие директивы поддерживают модификаторы - точки, которые указывают директиве обработать что-то по-другому:

```vue
<template>
  <!-- Модификаторы v-on -->
  <form @submit.prevent="onSubmit">
    <input @keyup.enter="onEnter" />
    <input @click.stop="onClick" />
  </form>
  
  <!-- Модификаторы v-model -->
  <input v-model.lazy="value" />
  <input v-model.number="value" />
  <input v-model.trim="value" />
</template>
```

## Пользовательские директивы

Vue позволяет регистрировать пользовательские директивы. Существуют два типа: директивы компонента и директивы на уровне экземпляра.

```javascript
// Регистрация глобальной пользовательской директивы
Vue.directive('focus', {
  mounted(el) {
    el.focus()
  }
})

// Использование в шаблоне
<template>
  <input v-focus />
</template>
```

Также можно создавать локальные директивы в компоненте:

```javascript
export default {
  directives: {
    focus: {
      mounted(el) {
        el.focus()
      }
    }
  }
}
```

## Жизненный цикл пользовательской директивы

- `created` - вызывается один раз, когда директива привязана к элементу
- `beforeMount` - вызывается перед тем, как родительский компонент будет смонтирован
- `mounted` - вызывается после монтирования элемента
- `beforeUpdate` - вызывается до обновления содержащего компонента
- `updated` - вызывается после обновления содержащего компонента
- `beforeUnmount` - вызывается до размонтирования директивы
- `unmounted` - вызывается, когда директива удаляется

## Примеры полезных пользовательских директив

### Директива для автоскрытия элемента

```javascript
export default {
  directives: {
    clickOutside: {
      beforeMount(el, binding) {
        el.clickOutsideEvent = function(event) {
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
  }
}
```

### Директива для подсветки синтаксиса

```javascript
import hljs from 'highlight.js'

export default {
  directives: {
    highlight: {
      mounted(el, binding) {
        const code = binding.value
        el.innerHTML = hljs.highlightAuto(code).value
      },
      updated(el, binding) {
        const code = binding.value
        el.innerHTML = hljs.highlightAuto(code).value
      }
    }
  }
}
```

## Лучшие практики

1. **Используйте v-show для часто переключаемых элементов**, а v-if для редко меняющихся условий
2. **Всегда используйте ключ в v-for**, особенно при наличии v-if в том же элементе
3. **Будьте осторожны с v-html**, особенно при работе с пользовательским контентом
4. **Предпочитайте v-model в простых случаях**, но используйте v-bind и v-on для более сложной логики
5. **Пользовательские директивы должны использоваться умеренно**, когда другие подходы (компоненты, плагины) не подходят

## Сравнение с другими фреймворками

Директивы в Vue отличаются от подходов в других фреймворках:

- В Angular есть похожая концепция директив, но с более сложной архитектурой
- В React нет встроенных директив, все реализуется через JSX и компоненты
- В Svelte директивы более интегрированы в синтаксис языка

> [!tip] 
> Для более глубокого понимания директив в Vue рекомендуется ознакомиться с [[Встроенные-директивы]] и [[Пользовательские-директивы]]

> [!note] 
> Директивы - это мощный инструмент, но не стоит ими злоупотреблять. Часто компоненты являются более подходящим решением для сложной логики.

## См. также

- [[Директивы-в-Angular]]
- [[Директивы-в-Svelte]]
- [[Пользовательские-директивы]]
- [[Встроенные-директивы]]