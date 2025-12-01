---
aliases: ["CSS Modules", "Модульные стили CSS", "Vue CSS Modules"]
tags: [vue, css, стилизация, модули, компоненты, css-modules]
---

# CSS-модули в Vue

## Введение

CSS-модули - это система, которая локализует имена классов CSS в области видимости компонента, предотвращая конфликты имен и обеспечивая инкапсуляцию стилей. В отличие от скоуп-стилей Vue, CSS-модули работают на уровне сборки и создают уникальные имена классов для каждого компонента.

## Основы использования

### Подключение CSS-модулей в Vue

Для использования CSS-модулей в компоненте Vue нужно добавить атрибут `module` к тегу `<style>`:

```vue
<template>
  <div :class="$style.container">
    <h1 :class="$style.title">Заголовок компонента</h1>
    <p :class="$style.text">Текст параграфа</p>
  </div>
</template>

<script>
export default {
  name: 'MyComponent'
}
</script>

<style module>
.container {
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 8px;
}

.title {
  color: #333;
  font-size: 24px;
  margin-bottom: 10px;
}

.text {
  color: #666;
  line-height: 1.6;
}
</style>
```

При компиляции Vue создаст уникальные имена классов и предоставит доступ к ним через объект `$style`:

```javascript
// Результат компиляции
{
  "container": "MyComponent_container__1A2B3",
  "title": "MyComponent_title__4C5D6",
  "text": "MyComponent_text__7E8F9"
}
```

### Альтернативный синтаксис с именованными модулями

Можно задать имя модуля для более явного доступа:

```vue
<template>
  <div :class="styles.container">
    <h1 :class="styles.title">Заголовок</h1>
    <p :class="styles.text">Текст</p>
  </div>
</template>

<style module="styles">
.container {
  padding: 20px;
  background-color: #f5f5f5;
}

.title {
  color: #333;
  font-size: 24px;
}

.text {
  color: #666;
  line-height: 1.6;
}
</style>
```

## Преимущества CSS-модулей

1. **Полная изоляция стилей** - уникальные имена классов предотвращают конфликты
2. **Явное связывание стилей с компонентами** - через объекты стилей
3. **Поддержка динамических классов** - через JavaScript
4. **Совместимость с существующими инструментами** - Webpack, Vite и т.д.
5. **Возможность передачи стилей между компонентами** - через пропсы

## Продвинутые возможности

### Работа с динамическими классами

```vue
<template>
  <div :class="[$style.container, $style[size]]">
    <h1 :class="$style.title">Заголовок</h1>
    <p :class="[$style.text, isHighlighted ? $style.highlighted : '']">
      {{ content }}
    </p>
  </div>
</template>

<script>
export default {
  name: 'DynamicComponent',
  props: {
    size: {
      type: String,
      default: 'medium',
      validator: value => ['small', 'medium', 'large'].includes(value)
    },
    content: String,
    isHighlighted: Boolean
  }
}
</script>

<style module>
.container {
  padding: 20px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.small { width: 200px; }
.medium { width: 300px; }
.large { width: 400px; }

.title {
  color: #333;
  font-size: 20px;
  margin-bottom: 10px;
}

.text {
  color: #666;
  line-height: 1.5;
}

.highlighted {
  background-color: #fffacd;
  font-weight: bold;
}
</style>
```

### Использование CSS-переменных в модулях

```vue
<template>
  <div :class="$style.card" :style="{ '--card-bg': backgroundColor }">
    <h3 :class="$style.title">{{ title }}</h3>
    <p :class="$style.content">{{ content }}</p>
  </div>
</template>

<script>
export default {
  name: 'StyledCard',
  props: {
    title: String,
    content: String,
    backgroundColor: {
      type: String,
      default: '#ffffff'
    }
  }
}
</script>

<style module>
.card {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 16px;
  background-color: var(--card-bg);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.title {
  color: #333;
  font-size: 18px;
  margin-bottom: 8px;
}

.content {
  color: #666;
  line-height: 1.5;
}
</style>
```

### Комбинация с обычными классами

```vue
<template>
  <div :class="[$style.container, 'global-container', customClass]">
    <h1 :class="[$style.title, 'text-bold']">Заголовок</h1>
    <p :class="$style.description">{{ description }}</p>
  </div>
</template>

<script>
export default {
  name: 'CombinedStyles',
  props: {
    description: String,
    customClass: String
  }
}
</script>

<style module>
.container {
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.title {
  color: #42b983;
  margin-bottom: 10px;
}

.description {
  color: #666;
  line-height: 1.6;
}
</style>
```

## Настройка CSS-модулей

### В Vite

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  css: {
    modules: {
      localsConvention: 'camelCase' // Преобразование имен классов в camelCase
    }
  }
})
```

### В Vue CLI

```javascript
// vue.config.js
module.exports = {
  css: {
    modules: true,
    extract: false
  }
}
```

### Конфигурация Webpack

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'vue-style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: '[name]_[local]_[hash:base64:5]'
              }
            }
          }
        ]
      }
    ]
  }
}
```

## Практические рекомендации

### 1. Использование camelCase для имен классов

```vue
<style module>
/* Хорошо: имена классов в camelCase */
.cardContainer { /* ... */ }
.titleText { /* ... */ }
.buttonPrimary { /* ... */ }

/* Плохо: имена классов в kebab-case */
.card-container { /* ... */ }
.title-text { /* ... */ }
.button-primary { /* ... */ }
</style>
```

### 2. Комбинация с препроцессорами

```vue
<template>
  <div :class="$style.container">
    <h1 :class="$style.title">Заголовок</h1>
  </div>
</template>

<style module lang="scss">
$primary-color: #42b983;
$secondary-color: #f5f5f5;

.container {
  padding: 20px;
  background-color: $secondary-color;
  
  .title {
    color: $primary-color;
    font-weight: bold;
    
    &:hover {
      color: darken($primary-color, 10%);
    }
  }
}
</style>
```

### 3. Работа с дочерними компонентами

```vue
<!-- ParentComponent.vue -->
<template>
  <div :class="$style.parent">
    <ChildComponent :styles="childStyles" />
  </div>
</template>

<script>
export default {
  name: 'ParentComponent',
  computed: {
    childStyles() {
      return {
        container: this.$style.childContainer,
        title: this.$style.childTitle
      }
    }
  }
}
</script>

<style module>
.parent {
  padding: 20px;
}

.childContainer {
  padding: 10px;
  border: 1px solid #ccc;
}

.childTitle {
  color: #333;
  font-size: 16px;
}
</style>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div :class="styles.container">
    <h2 :class="styles.title">Дочерний компонент</h2>
  </div>
</template>

<script>
export default {
  name: 'ChildComponent',
  props: {
    styles: {
      type: Object,
      default: () => ({})
    }
  }
}
</script>
```

### 4. Тестирование компонентов с CSS-модулями

```javascript
// tests/MyComponent.spec.js
import { mount } from '@vue/test-utils'
import MyComponent from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders with correct styles', () => {
    const wrapper = mount(MyComponent)
    
    // Проверка, что классы были применены
    const container = wrapper.find('[class*="container"]')
    expect(container.exists()).toBe(true)
    
    // Проверка конкретного стилизованного элемента
    const title = wrapper.find('[class*="title"]')
    expect(title.exists()).toBe(true)
  })
})
```

## Сравнение с другими подходами

| Функция | Скоуп-стили | CSS-модули | Глобальные стили |
|---------|-------------|------------|------------------|
| Изоляция стилей | ✅ | ✅ | ❌ |
| Явное связывание | ❌ | ✅ | ❌ |
| Динамические классы | ✅ | ✅ | ✅ |
| Поддержка препроцессоров | ✅ | ✅ | ✅ |
| Производительность | Высокая | Высокая | Средняя |
| Легкость использования | Высокая | Средняя | Высокая |

## Адаптация под российские реалии

### Поддержка кириллических шрифтов

```vue
<style module>
.container {
  /* Учет особенностей кириллических шрифтов */
  font-family: 'Roboto', 'Arial', 'Helvetica', sans-serif;
  line-height: 1.7; /* Увеличенный интерлиньяж для лучшей читаемости */
  letter-spacing: 0.2px; /* Небольшое увеличение интервала между буквами */
}

/* Стили для печати документов на русском языке */
.printable {
  font-size: 12pt;
  line-height: 1.5;
  color: #000;
  background: #fff;
}
</style>
```

### Учет требований доступности

```vue
<style module>
.accessibleButton {
  /* Обеспечение контраста для доступности */
  background-color: #42b983;
  color: white;
  border: 2px solid transparent;
  padding: 8px 16px;
  border-radius: 4px;
  
  /* Визуальная индикация фокуса */
  &:focus {
    outline: 2px solid #409eff;
    outline-offset: 2px;
  }
  
  /* Поддержка увеличенного размера текста */
  font-size: 16px;
  min-height: 44px; /* Для удобства касания на мобильных устройствах */
}
</style>
```

## Лучшие практики

1. **Использование семантических имен классов** - улучшает читаемость кода
2. **Консистентность в именовании** - используйте единый стиль именования
3. **Минимизация переопределения стилей** - избегайте избыточного дублирования
4. **Документирование сложных стилей** - особенно при передаче через пропсы
5. **Тестирование визуальных компонентов** - особенно при использовании CSS-модулей

## Совместимость с инструментами

CSS-модули поддерживаются всеми основными инструментами сборки Vue:

- Vite (с @vitejs/plugin-vue)
- Vue CLI (с vue-loader)
- Webpack (с css-loader)
- Nuxt.js (в версиях 3+)

## Заключение

CSS-модули предоставляют мощный механизм изоляции стилей в компонентном Vue-приложении. Они обеспечивают полную изоляцию имен классов, что особенно важно в больших приложениях с множеством компонентов. При правильном использовании CSS-модули позволяют создавать более надежные и предсказуемые стилизационные решения.

## См. также

- [[Скоуп-стили]]
- [[Глобальные-стили]]
- [[Препроцессоры]]
- [[PostCSS]]
- [[Vue компоненты]]

## Источники

- [Официальная документация CSS Modules](https://github.com/css-modules/css-modules)
- [Vue.js Documentation - CSS Modules](https://vuejs.org/api/sfc-css-features.html#css-modules)
- [CSS Modules в Vue 3](https://v3.vuejs.org/api/sfc-style.html#using-css-modules)