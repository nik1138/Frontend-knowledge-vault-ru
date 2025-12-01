---
aliases: [Keep-alive, Кеширование компонентов]
tags: [vue, keep-alive, frontend, performance, ru-2025]
---

# Keep-alive

Keep-alive - это встроенный компонент Vue, который позволяет кэшировать экземпляры компонентов, предотвращая их повторное создание и уничтожение при переключении между ними. Это особенно полезно для повышения производительности в приложениях с частыми переключениями между компонентами.

## Основы использования

Компонент `<keep-alive>` оборачивает динамический компонент или компонент с условным рендерингом:

```vue
<template>
  <div>
    <button @click="activeComponent = 'ComponentA'">Компонент A</button>
    <button @click="activeComponent = 'ComponentB'">Компонент B</button>
    
    <keep-alive>
      <component :is="activeComponent" />
    </keep-alive>
  </div>
</template>

<script>
import ComponentA from './ComponentA.vue'
import ComponentB from './ComponentB.vue'

export default {
  components: {
    ComponentA,
    ComponentB
  },
  data() {
    return {
      activeComponent: 'ComponentA'
    }
  }
}
</script>
```

## Атрибуты Keep-alive

### `include`
Определяет компоненты, которые должны быть кэшированы (по имени компонента или регулярному выражению):

```vue
<keep-alive include="ComponentA,ComponentB">
  <component :is="currentComponent" />
</keep-alive>

<!-- Или с использованием массива -->
<keep-alive :include="['ComponentA', 'ComponentB']">
  <component :is="currentComponent" />
</keep-alive>
```

### `exclude`
Определяет компоненты, которые не должны быть кэшированы:

```vue
<keep-alive exclude="ComponentC">
  <component :is="currentComponent" />
</keep-alive>
```

### `max`
Ограничивает максимальное количество кэшируемых компонентов:

```vue
<keep-alive :max="5">
  <component :is="currentComponent" />
</keep-alive>
```

## Жизненный цикл компонентов в Keep-alive

Компоненты, обернутые в `<keep-alive>`, получают дополнительные хуки жизненного цикла:

- `activated` - вызывается при активации компонента
- `deactivated` - вызывается при деактивации компонента

```vue
<script>
export default {
  name: 'CachedComponent',
  activated() {
    console.log('Компонент активирован')
    // Здесь можно возобновить таймеры, подписки и т.д.
  },
  deactivated() {
    console.log('Компонент деактивирован')
    // Здесь можно приостановить таймеры, отписаться от событий и т.д.
  }
}
</script>
```

## Практическое применение в российских реалиях 2025

Keep-alive особенно востребован в следующих сценариях:

1. **Табы и навигация** - для сохранения состояния вкладок
2. **Многошаговые формы** - для сохранения введенных данных
3. **Пошаговые инструкции** - для сохранения прогресса пользователя
4. **Карточки товаров** - для сохранения состояния фильтров и поиска
5. **Чат-интерфейсы** - для сохранения состояния сессий

## Пример реализации с учетом российских реалий

```vue
<template>
  <div class="tab-container">
    <nav class="tab-nav">
      <button
        v-for="tab in tabs"
        :key="tab.name"
        :class="{ active: currentTab === tab.name }"
        @click="switchTab(tab.name)"
      >
        {{ tab.title }}
      </button>
    </nav>
    
    <keep-alive :include="cachedTabs" :max="maxCachedTabs">
      <component 
        :is="currentTabComponent" 
        :key="currentTab"
        :locale="locale"
        @data-updated="handleDataUpdate"
      />
    </keep-alive>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import UserProfileTab from './tabs/UserProfileTab.vue'
import UserOrdersTab from './tabs/UserOrdersTab.vue'
import UserSettingsTab from './tabs/UserSettingsTab.vue'

export default {
  name: 'RussianTabContainer',
  components: {
    UserProfileTab,
    UserOrdersTab,
    UserSettingsTab
  },
  props: {
    initialTab: {
      type: String,
      default: 'UserProfileTab'
    },
    locale: {
      type: String,
      default: 'ru-RU'
    }
  },
  setup(props) {
    const currentTab = ref(props.initialTab)
    const cachedTabs = ref(['UserProfileTab', 'UserOrdersTab'])
    const maxCachedTabs = ref(3)
    
    const tabs = [
      { name: 'UserProfileTab', title: 'Профиль' },
      { name: 'UserOrdersTab', title: 'Заказы' },
      { name: 'UserSettingsTab', title: 'Настройки' }
    ]
    
    const currentTabComponent = computed(() => {
      return tabs.find(tab => tab.name === currentTab.value)?.name || 'UserProfileTab'
    })
    
    const switchTab = (tabName) => {
      currentTab.value = tabName
    }
    
    const handleDataUpdate = (data) => {
      // Обработка обновления данных из вкладки
      console.log('Данные обновлены в вкладке', currentTab.value, data)
    }
    
    // Отслеживание изменения вкладки для целей аналитики
    watch(currentTab, (newTab, oldTab) => {
      // Отправка события в систему аналитики
      if (window.ym) {
        window.ym(12345678, 'reachGoal', `switch_to_${newTab}`)
      }
    })
    
    return {
      currentTab,
      currentTabComponent,
      tabs,
      cachedTabs,
      maxCachedTabs,
      switchTab,
      handleDataUpdate
    }
  }
}
</script>

<style scoped>
.tab-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.tab-nav {
  display: flex;
  border-bottom: 1px solid #ddd;
}

.tab-nav button {
  padding: 12px 20px;
  border: none;
  background: none;
  cursor: pointer;
  border-bottom: 2px solid transparent;
}

.tab-nav button.active {
  border-bottom-color: #007bff;
  font-weight: bold;
}

.tab-content {
  flex: 1;
  padding: 20px;
}
</style>
```

## Совместимость с другими элементами

Keep-alive работает корректно с другими специальными элементами Vue:

- [[Teleport]] - кэшированные компоненты могут использовать телепорт
- [[Suspense]] - совместим с асинхронными компонентами
- [[Component]] - часто используется вместе с динамическими компонентами

## Особенности и ограничения

1. **Память** - кэшированные компоненты остаются в памяти, что может увеличить потребление
2. **Состояние** - все данные компонента сохраняются при переключении
3. **События** - компоненты продолжают реагировать на события даже в неактивном состоянии
4. **SSR** - требует особого внимания при рендеринге на стороне сервера

## Управление кэшем программно

Vue предоставляет доступ к кэшированным компонентам через `$attrs`:

```javascript
// Доступ к экземпляру кэшированного компонента
this.$refs.keepAliveRef.$vnode.componentInstance

// Очистка кэша
this.$refs.keepAliveRef.cache = new Map()
this.$refs.keepAliveRef.keys = []
```

## Связанные темы

- [[Component]] - для понимания динамических компонентов
- [[Teleport]] - для понимания работы с DOM вне обычного потока
- [[Suspense]] - для обработки асинхронных компонентов
- [[Vue компоненты]]
- [[Vue производительность]]

## Лучшие практики

1. Используйте `max` для ограничения потребления памяти
2. Очищайте кэш при необходимости (например, при выходе пользователя)
3. Учитывайте особенности работы с асинхронными данными в кэшированных компонентах
4. Используйте `activated` и `deactivated` для управления ресурсами
5. Не кэшируйте компоненты с чувствительными данными без необходимости