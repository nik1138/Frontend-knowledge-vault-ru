---
aliases: ["RTL поддержка", "Справа налево", "Языки RTL"]
tags: [vue, i18n, rtl, интернационализация, css]
---

# RTL-поддержка

## Обзор

RTL (Right-to-Left) поддержка - это способность интерфейса корректно отображаться для языков, которые читаются справа налево, таких как арабский, иврит, персидский и другие. Хотя в России основным языком является LTR (слева направо), поддержка RTL важна для международных приложений и может быть полезна при интеграции с ближневосточными рынками.

## Основы RTL в веб-разработке

### HTML атрибуты

Для включения режима RTL в HTML используется атрибут `dir`:

```html
<html dir="rtl">
  <!-- или -->
  <body dir="rtl">
    <!-- контент -->
  </body>
</html>
```

### CSS свойства

CSS предоставляет несколько свойств для работы с RTL:

```css
/* Основные свойства для RTL */
.rtl-content {
  direction: rtl;
  text-align: right;
}

/* Использование логических свойств (предпочтительно) */
.rtl-container {
  padding-inline-start: 1rem;  /* вместо padding-left */
  padding-inline-end: 1rem;    /* вместо padding-right */
  margin-inline-start: auto;   /* вместо margin-left */
  margin-inline-end: auto;     /* вместо margin-right */
}
```

## Реализация RTL в Vue.js приложениях

### 1. Динамическое переключение направления

```vue
<template>
  <div 
    :dir="currentDirection" 
    :class="['app-container', `dir-${currentDirection}`]"
  >
    <header class="app-header">
      <LanguageSelector />
      <RTLToggle />
    </header>
    <main class="app-main">
      <router-view />
    </main>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import LanguageSelector from '@/components/LanguageSelector.vue'
import RTLToggle from '@/components/RTLToggle.vue'

const { locale } = useI18n()

// Определение направления на основе языка
const currentDirection = computed(() => {
  const rtlLanguages = ['ar', 'he', 'fa', 'ur', 'ps', 'sd', 'prs', 'ku', 'ug', 'dv', 'ha']
  return rtlLanguages.includes(locale.value) ? 'rtl' : 'ltr'
})
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  transition: direction 0.2s ease;
}

[dir="rtl"] .app-container {
  /* Специфичные стили для RTL */
}

[dir="ltr"] .app-container {
  /* Специфичные стили для LTR */
}
</style>
```

### 2. Компонент переключения RTL

```vue
<template>
  <div class="rtl-toggle">
    <button 
      @click="toggleRTL"
      :aria-label="t('accessibility.toggleRTL')"
      :class="['toggle-btn', { active: isRTL }]"
    >
      <span v-if="isRTL">RTL</span>
      <span v-else>LTR</span>
    </button>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'

const { locale, t } = useI18n()

// Определяем, является ли текущий язык RTL
const isRTL = computed(() => {
  const rtlLanguages = ['ar', 'he', 'fa', 'ur', 'ps', 'sd', 'prs', 'ku', 'ug', 'dv', 'ha']
  return rtlLanguages.includes(locale.value)
})

const toggleRTL = () => {
  // Переключение между RTL и LTR для текущего языка
  // В реальном приложении может зависеть от текущего языка
  document.documentElement.dir = isRTL.value ? 'ltr' : 'rtl'
}
</script>

<style scoped>
.rtl-toggle {
  display: inline-block;
}

.toggle-btn {
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  background-color: #f5f5f5;
  cursor: pointer;
  border-radius: 4px;
}

.toggle-btn.active {
  background-color: #007bff;
  color: white;
}
</style>
```

## CSS для RTL поддержки

### 1. Использование логических свойств

Вместо физических свойств (left/right) используйте логические:

```css
/* ПЛОХО - физические свойства */
.bad-example {
  margin-left: 1rem;
  padding-right: 2rem;
  text-align: right;
}

/* ХОРОШО - логические свойства */
.good-example {
  margin-inline-start: 1rem;      /* вместо margin-left */
  margin-inline-end: 1rem;        /* вместо margin-right */
  padding-inline-start: 2rem;     /* вместо padding-left */
  padding-inline-end: 2rem;       /* вместо padding-right */
  text-align: start;              /* вместо left, будет left в LTR и right в RTL */
  text-align: end;                /* вместо right, будет right в LTR и left в RTL */
}

/* Дополнительные логические свойства */
.logical-properties {
  border-inline-start: 1px solid #000;  /* вместо border-left */
  border-inline-end: 1px solid #000;    /* вместо border-right */
  inset-inline-start: 10px;             /* вместо left */
  inset-inline-end: 10px;               /* вместо right */
}
```

### 2. CSS классы для RTL

```css
/* Базовые стили */
.rtl-base {
  direction: rtl;
  text-align: right;
}

.rtl-base .float-container::after {
  display: table;
  clear: both;
  content: "";
}

/* Специфичные стили для RTL */
[dir="rtl"] .rtl-specific {
  margin-right: 0;
  margin-left: auto;
}

[dir="rtl"] .icon-flip {
  transform: scaleX(-1);
}
```

### 3. Использование CSS-переменных для RTL

```css
:root {
  --margin-start: 1rem;
  --margin-end: 1rem;
  --padding-start: 1rem;
  --padding-end: 1rem;
}

[dir="rtl"] {
  --margin-start: 1rem;
  --margin-end: 1rem;
  --padding-start: 1rem;
  --padding-end: 1rem;
}

.rtl-aware-element {
  margin-inline-start: var(--margin-start);
  margin-inline-end: var(--margin-end);
  padding-inline-start: var(--padding-start);
  padding-inline-end: var(--padding-end);
}
```

## Vue.js компоненты с поддержкой RTL

### 1. Навигационное меню

```vue
<template>
  <nav class="rtl-aware-nav">
    <ul class="nav-list">
      <li v-for="item in navItems" :key="item.id" class="nav-item">
        <router-link 
          :to="item.path" 
          class="nav-link"
        >
          {{ t(item.label) }}
        </router-link>
      </li>
    </ul>
  </nav>
</template>

<script setup>
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const navItems = [
  { id: 1, path: '/', label: 'nav.home' },
  { id: 2, path: '/about', label: 'nav.about' },
  { id: 3, path: '/services', label: 'nav.services' },
  { id: 4, path: '/contact', label: 'nav.contact' }
]
</script>

<style scoped>
.rtl-aware-nav {
  width: 100%;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.nav-list {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  /* Используем логические свойства для выравнивания */
  padding-inline: 1rem;
}

.nav-item {
  margin-inline-end: 2rem;
}

.nav-link {
  display: block;
  padding: 1rem 0.5rem;
  text-decoration: none;
  color: #333;
  position: relative;
}

.nav-link:hover {
  color: #007bff;
}

.nav-link::after {
  content: '';
  position: absolute;
  bottom: 0;
  /* Используем логические свойства для позиционирования */
  inset-inline-start: 50%;
  width: 0;
  height: 2px;
  background-color: #007bff;
  transition: all 0.3s ease;
}

.nav-link:hover::after {
  inset-inline-start: 0;
  width: 100%;
}
</style>
```

### 2. Карточка продукта с RTL

```vue
<template>
  <div class="product-card">
    <div class="product-image-container">
      <img :src="product.image" :alt="t('product.imageAlt', { name: product.name })" />
    </div>
    <div class="product-details">
      <h3 class="product-name">{{ t('product.name', { name: product.name }) }}</h3>
      <p class="product-description">{{ t('product.description', { desc: product.description }) }}</p>
      <div class="product-price-container">
        <span class="product-price">{{ formatPrice(product.price) }}</span>
        <button class="buy-button">{{ t('product.buy') }}</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useI18n } from 'vue-i18n'
import { computed } from 'vue'

const props = defineProps({
  product: {
    type: Object,
    required: true
  }
})

const { t, locale } = useI18n()

const formatPrice = computed(() => (price) => {
  return new Intl.NumberFormat(locale.value, {
    style: 'currency',
    currency: locale.value === 'ar' ? 'SAR' : 'USD'  // Пример для арабского языка
  }).format(price)
})
</script>

<style scoped>
.product-card {
  display: flex;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s ease;
}

.product-card:hover {
  transform: translateY(-2px);
}

.product-image-container {
  flex: 0 0 30%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f9f9f9;
}

.product-image-container img {
  max-width: 100%;
  max-height: 100%;
  object-fit: cover;
}

.product-details {
  flex: 1;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.product-name {
  margin: 0 0 0.5rem 0;
  font-size: 1.2rem;
  font-weight: bold;
}

.product-description {
  margin: 0 0 1rem 0;
  color: #666;
  flex-grow: 1;
}

.product-price-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.product-price {
  font-size: 1.2rem;
  font-weight: bold;
  color: #e74c3c;
}

.buy-button {
  padding: 0.5rem 1rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.buy-button:hover {
  background-color: #0056b3;
}

/* Специфичные стили для RTL */
[dir="rtl"] .product-card {
  flex-direction: row-reverse;
}

[dir="rtl"] .product-price-container {
  flex-direction: row-reverse;
  gap: 1rem;
}
</style>
```

## Практические рекомендации для российских разработчиков

### 1. Проверка RTL поддержки

При разработке интернационализированных приложений, даже если основной рынок - Россия, важно учитывать:

- Возможность выхода на международные рынки
- Использование приложения русскоязычными пользователями в арабских странах
- Интеграция с международными системами

### 2. Тестирование RTL интерфейсов

```javascript
// tests/rtl-support.spec.js
import { mount } from '@vue/test-utils'
import App from '@/App.vue'

describe('RTL Support', () => {
  test('should render correctly in RTL mode', async () => {
    const wrapper = mount(App, {
      global: {
        plugins: [i18n]
      }
    })
    
    // Переключаем язык на арабский (RTL)
    i18n.global.locale.value = 'ar'
    
    // Проверяем, что атрибут dir установлен правильно
    expect(wrapper.attributes('dir')).toBe('rtl')
    
    // Проверяем специфичные стили
    expect(wrapper.classes()).toContain('dir-rtl')
  })
})
```

### 3. Инструменты для проверки RTL

```javascript
// utils/rtl-detector.js
export function detectRTL(language) {
  const rtlLanguages = [
    'ar', 'ara', // Arabic
    'he', 'heb', // Hebrew
    'fa', 'fas', // Persian/Farsi
    'ur', 'urd', // Urdu
    'ps', 'pus', // Pashto
    'sd', 'snd', // Sindhi
    'prs', 'prc', // Dari Persian
    'ku', 'kur', // Kurdish
    'ug', 'uig', // Uyghur
    'dv', 'div', // Divehi/Maldivian
    'ha', 'hau'  // Hausa
  ]
  
  return rtlLanguages.includes(language.toLowerCase())
}
```

## Лучшие практики

1. **Используйте логические CSS свойства**: Вместо `margin-left/right` используйте `margin-inline-start/end`
2. **Тестируйте с RTL языками**: Регулярно проверяйте интерфейс с RTL языками
3. **Используйте CSS-фреймворки с RTL поддержкой**: Например, Bootstrap 5 имеет встроенную поддержку RTL
4. **Документируйте RTL особенности**: Ведите документацию по специфике RTL отображения
5. **Используйте визуальные индикаторы**: Добавляйте визуальные подсказки для направления текста

## См. также

- [[Vue-I18n]]
- [[Локализация-компонентов]]
- [[Плагины-перевода]]
- [[Мультиязычность]]