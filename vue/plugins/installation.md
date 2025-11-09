# Plugin Installation

## Определение
Установка плагинов в Vue - это процесс интеграции сторонней функциональности в приложение Vue с помощью метода `app.use()`. Это позволяет расширять возможности приложения, добавлять глобальные компоненты, директивы, методы и другую функциональность.

## Базовая установка

### Простая установка
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

const app = createApp(App)

// Установка плагина
app.use(MyPlugin)

app.mount('#app')
```

### Установка с опциями
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

const app = createApp(App)

// Установка с опциями
app.use(MyPlugin, {
  apiKey: 'your-api-key',
  debug: true,
  timeout: 5000
})

app.mount('#app')
```

## Порядок установки

### Порядок имеет значение
```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Установка Vuex до компонентов, использующих store
app.use(store)

// Установка Vue Router до компонентов, использующих роутинг
app.use(router)

// Установка пользовательских плагинов
app.use(myPlugin)

// Установка UI фреймворка
app.use(uiFramework)

app.mount('#app')
```

## Условная установка

### Установка на основе условий
```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// Условная установка
if (process.env.NODE_ENV === 'development') {
  app.use(devToolsPlugin)
}

if (process.env.VUE_APP_ANALYTICS === 'enabled') {
  app.use(analyticsPlugin, {
    trackingId: process.env.VUE_APP_ANALYTICS_ID
  })
}

// Установка для определенных режимов
if (import.meta.env.MODE === 'ssr') {
  app.use(ssrPlugin)
} else {
  app.use(clientPlugin)
}

app.mount('#app')
```

### Установка с проверками
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import { checkPluginCompatibility } from './utils/pluginUtils'

const app = createApp(App)

try {
  if (checkPluginCompatibility(MyPlugin)) {
    app.use(MyPlugin)
  } else {
    console.warn('Plugin is not compatible with current setup')
  }
} catch (error) {
  console.error('Plugin installation failed:', error)
}

app.mount('#app')
```

## Асинхронная установка

### Асинхронная подготовка перед установкой
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

async function initApp() {
  // Асинхронная загрузка конфигурации
  const config = await loadAppConfig()
  
  const app = createApp(App)
  
  // Установка плагина с загруженной конфигурацией
  app.use(MyPlugin, {
    ...config.pluginOptions
  })
  
  app.mount('#app')
}

initApp().catch(console.error)

async function loadAppConfig() {
  const response = await fetch('/api/config')
  return response.json()
}
```

### Установка с ожиданием
```javascript
import { createApp } from 'vue'
import App from './App.vue'
import AuthPlugin from './plugins/authPlugin'

async function setupAuthPlugin(app) {
  // Загрузка данных аутентификации
  const authData = await loadAuthData()
  
  // Установка плагина с аутентификационными данными
  app.use(AuthPlugin, {
    token: authData.token,
    user: authData.user
  })
}

async function initApp() {
  const app = createApp(App)
  
  await setupAuthPlugin(app)
  
  app.mount('#app')
}

initApp().catch(console.error)
```

## Установка в разных окружениях

### SSR установка
```javascript
// server.js
import { createApp } from 'vue'
import { renderToString } from '@vue/server-renderer'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

export async function render(url, manifest) {
  const app = createApp(App)
  
  // Установка плагинов, совместимых с SSR
  app.use(MyPlugin, { 
    isServer: true,
    // Определенные опции для сервера
  })
  
  const appContent = await renderToString(app)
  
  return { appContent }
}
```

```javascript
// client.js
import { createApp } from 'vue'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

const app = createApp(App)

// Установка плагинов для клиента
app.use(MyPlugin, {
  isServer: false,
  // Определенные опции для клиента
})

app.mount('#app')
```

## Установка в модулях

### В модулях Nuxt
```javascript
// plugins/myPlugin.client.js (Nuxt 3)
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.use(MyPlugin, {
    // Опции плагина
  })
  
  return {
    provide: {
      myPlugin: {} // Доступ к плагину через $nuxt.$myPlugin
    }
  }
})
```

### В функциях высшего порядка
```javascript
// pluginInstaller.js
export function createInstaller(plugin, options) {
  return (app) => {
    app.use(plugin, options)
  }
}

// Использование
import { createApp } from 'vue'
import { createInstaller } from './pluginInstaller'
import MyPlugin from './plugins/myPlugin'

const app = createApp(App)

const installMyPlugin = createInstaller(MyPlugin, { debug: true })
installMyPlugin(app)

app.mount('#app')
```

## Установка с обертыванием

### Обертывание плагина
```javascript
// wrappedPlugin.js
export function installWrappedPlugin(app, plugin, options = {}) {
  const { beforeInstall, afterInstall } = options
  
  if (beforeInstall) {
    beforeInstall(app, options)
  }
  
  app.use(plugin, options.pluginOptions)
  
  if (afterInstall) {
    afterInstall(app, options)
  }
}

// Использование
import { createApp } from 'vue'
import App from './App.vue'
import MyPlugin from './plugins/myPlugin'

const app = createApp(App)

installWrappedPlugin(app, MyPlugin, {
  beforeInstall: (app) => {
    console.log('Подготовка к установке плагина')
  },
  afterInstall: (app) => {
    console.log('Плагин установлен')
  },
  pluginOptions: {
    apiKey: '123'
  }
})

app.mount('#app')
```

## Управление установкой

### Проверка установки
```javascript
// pluginManager.js
class PluginManager {
  constructor() {
    this.installed = new Set()
  }
  
  installOnce(app, plugin, options = {}) {
    if (this.installed.has(plugin)) {
      console.warn(`Plugin ${plugin.id || 'unknown'} already installed`)
      return this
    }
    
    this.installed.add(plugin)
    app.use(plugin, options)
    return this
  }
  
  isInstalled(plugin) {
    return this.installed.has(plugin)
  }
}

// Использование
const pluginManager = new PluginManager()
const app = createApp(App)

pluginManager
  .installOnce(app, MyPlugin, { debug: true })
  .installOnce(app, AnotherPlugin)

app.mount('#app')
```

## Ошибки при установке

### Обработка ошибок установки
```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

const plugins = [
  { plugin: MyPlugin, options: { debug: true } },
  { plugin: AnotherPlugin, options: {} }
]

plugins.forEach(({ plugin, options }) => {
  try {
    app.use(plugin, options)
    console.log(`Плагин ${plugin.id || 'unknown'} установлен успешно`)
  } catch (error) {
    console.error(`Ошибка установки плагина:`, error)
    // Можно продолжить установку других плагинов
  }
})

app.mount('#app')
```

### Проверка зависимостей
```javascript
// dependencyCheck.js
export function installWithDependencies(app, plugin, options = {}) {
  if (plugin.dependencies) {
    plugin.dependencies.forEach(dep => {
      if (!app.config.globalProperties[`$${dep}`]) {
        console.warn(`Зависимость ${dep} не найдена для плагина ${plugin.id}`)
      }
    })
  }
  
  app.use(plugin, options)
}
```

## Удаление плагинов

### "Удаление" плагина (ограниченно)
Vue не предоставляет встроенного метода для "удаления" плагинов, но можно сбросить изменения:

```javascript
// uninstallPlugin.js
export function uninstallPlugin(app, pluginId) {
  // Сброс глобальных свойств
  if (app.config.globalProperties.$[pluginId]) {
    delete app.config.globalProperties.$[pluginId]
  }
  
  // Возвращение к значениям по умолчанию
  // (требует знания, что добавил плагин)
}
```

## Совместимость с TypeScript

```typescript
// types.ts
import { App } from 'vue'

interface PluginOptions {
  apiKey: string
  debug?: boolean
}

declare module '@vue/runtime-core' {
  interface ComponentCustomProperties {
    $myPlugin: any
  }
}

// plugin.ts
export default {
  install(app: App, options: PluginOptions) {
    app.config.globalProperties.$myPlugin = createService(options)
  }
}
```

## Лучшие практики установки

1. **Устанавливайте плагины до монтирования приложения**
2. **Проверяйте совместимость версий**
3. **Обрабатывайте ошибки установки**
4. **Используйте условную установку на основе окружения**
5. **Документируйте зависимости между плагинами**

## Тестирование установки плагинов
```javascript
import { createApp } from '@vue/test-utils'
import App from '@/App.vue'
import MyPlugin from '@/plugins/myPlugin'

describe('Plugin Installation', () => {
  test('устанавливает плагин без ошибок', () => {
    const app = createApp(App)
    
    expect(() => {
      app.use(MyPlugin)
    }).not.toThrow()
  })
  
  test('добавляет ожидаемые свойства', () => {
    const app = createApp(App)
    app.use(MyPlugin)
    
    expect(app.config.globalProperties.$myMethod).toBeDefined()
  })
  
  test('работает с опциями', () => {
    const app = createApp(App)
    
    expect(() => {
      app.use(MyPlugin, { debug: true })
    }).not.toThrow()
  })
})
```

## Связь с другими концепциями
- [[plugins/structure]] - структура плагинов
- [[plugins/global-api]] - добавление глобального API
- [[state-management/State Management]] - установка плагинов управления состоянием
- [[routing/Routing]] - установка роутинг плагинов