---
aliases: ["Vue I18n", "Интернационализация Vue", "Vue i18n библиотека"]
tags: [vue, i18n, интернационализация, программирование]
---

# Vue-I18n

## Обзор

Vue I18n - это официальная библиотека интернационализации для Vue.js приложений. Она позволяет легко управлять переводами, форматировать числа, даты и валюты в соответствии с локальными стандартами, а также поддерживать мультиязычные интерфейсы.

## Установка

```bash
npm install vue-i18n
```

Для Vue 3 необходимо использовать версию 9.x, а для Vue 2 - версию 8.x.

## Базовая настройка

### Для Vue 3

```javascript
import { createApp } from 'vue'
import { createI18n } from 'vue-i18n'

import App from './App.vue'

// Подготовка переводов
const messages = {
  ru: {
    message: {
      hello: 'Привет мир!',
      greeting: 'Привет, {name}!'
    }
  },
  en: {
    message: {
      hello: 'Hello world!',
      greeting: 'Hello, {name}!'
    }
  }
}

// Создание экземпляра i18n
const i18n = createI18n({
  locale: 'ru', // локаль по умолчанию
  fallbackLocale: 'en', // резервная локаль
  messages, // сообщения
})

const app = createApp(App)

app.use(i18n)
app.mount('#app')
```

### Для Vue 2

```javascript
import Vue from 'vue'
import VueI18n from 'vue-i18n'

Vue.use(VueI18n)

const messages = {
  ru: {
    message: {
      hello: 'Привет мир!',
      greeting: 'Привет, {name}!'
    }
  },
  en: {
    message: {
      hello: 'Hello world!',
      greeting: 'Hello, {name}!'
    }
  }
}

const i18n = new VueI18n({
  locale: 'ru',
  fallbackLocale: 'en',
  messages
})

new Vue({
  i18n,
  // ... другие опции
}).$mount('#app')
```

## Использование в компонентах

### Через композиционный API (Vue 3)

```vue
<template>
  <div>
    <h1>{{ t('message.hello') }}</h1>
    <p>{{ t('message.greeting', { name: userName }) }}</p>
    <button @click="changeLocale('ru')">Русский</button>
    <button @click="changeLocale('en')">English</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useI18n } from 'vue-i18n'

const { t, locale } = useI18n()

const userName = ref('Алексей')

const changeLocale = (newLocale) => {
  locale.value = newLocale
}
</script>
```

### Через Options API

```vue
<template>
  <div>
    <h1>{{ $t('message.hello') }}</h1>
    <p>{{ $t('message.greeting', { name: userName }) }}</p>
    <button @click="changeLocale('ru')">Русский</button>
    <button @click="changeLocale('en')">English</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userName: 'Алексей'
    }
  },
  methods: {
    changeLocale(newLocale) {
      this.$i18n.locale = newLocale
    }
  }
}
</script>
```

## Практические рекомендации для российских разработчиков

### Работа с падежами

В русском языке важную роль играют падежи. Для корректной работы с падежами можно использовать плагины или вспомогательные функции:

```javascript
// Пример функции для склонения существительных
const declineNoun = (count, one, two, five) => {
  const n = Math.abs(count) % 100
  const n1 = n % 10
  if (n > 10 && n < 20) return five
  if (n1 > 1 && n1 < 5) return two
  if (n1 === 1) return one
  return five
}

// Использование в шаблоне
const messages = {
  ru: {
    items: {
      count: (count) => `${count} ${declineNoun(count, 'товар', 'товара', 'товаров')}`
    }
  }
}
```

### Форматирование дат и чисел

Vue I18n предоставляет встроенные возможности для форматирования:

```javascript
const i18n = createI18n({
  locale: 'ru-RU',
  fallbackLocale: 'en-US',
  messages,
  datetimeFormats: {
    'ru-RU': {
      short: {
        year: 'numeric',
        month: 'short',
        day: 'numeric'
      },
      long: {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        weekday: 'long'
      }
    }
  },
  numberFormats: {
    'ru-RU': {
      currency: {
        style: 'currency',
        currency: 'RUB',
        minimumFractionDigits: 2
      },
      decimal: {
        style: 'decimal',
        minimumFractionDigits: 2
      }
    }
  }
})
```

## Плагины и расширения

Для улучшения процесса интернационализации можно использовать дополнительные плагины:

- [[Плагины-перевода]]
- [[Локализация-компонентов]]

## Лучшие практики

1. **Структура файлов переводов**: Организуйте переводы по модулям/компонентам
2. **Проверка синтаксиса**: Используйте линтеры для проверки синтаксиса переводов
3. **Инструменты для переводчиков**: Интегрируйте с профессиональными переводческими инструментами
4. **Тестирование локализации**: Создавайте тесты для проверки корректности отображения переводов

## См. также

- [[Мультиязычность]]
- [[RTL-поддержка]]
- [[Локализация-компонентов]]
- [[Плагины-перевода]]