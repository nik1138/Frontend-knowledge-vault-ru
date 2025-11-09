# Plugin Structure

## Определение
Структура плагина Vue определяет, как создавать и организовывать плагины для расширения функциональности Vue приложений. Плагин может добавлять глобальные компоненты, директивы, методы, изменять поведение Vue или интегрироваться с внешними библиотеками.

## Базовая структура плагина

### Простой объект-плагин
```javascript
// myPlugin.js
export const MyPlugin = {
  // Уникальный идентификатор плагина
  id: 'my-plugin',
  
  // Метод установки
  install(app, options = {}) {
    // Установка начинается здесь
    console.log('Мой плагин установлен')
    
    // Добавление глобальных возможностей
    app.config.globalProperties.$myMethod = function() {
      return 'Мой глобальный метод'
    }
  }
}

// Использование
import { createApp } from 'vue'
import { MyPlugin } from './plugins/myPlugin'

const app = createApp(App)
app.use(MyPlugin, { option1: 'value1' })
```

## Структура полного плагина

```javascript
// advancedPlugin.js
export default {
  id: 'advanced-plugin',
  
  // Метод установки
  install(app, options = {}) {
    // Валидация опций
    const config = {
      apiUrl: options.apiUrl || 'https://api.example.com',
      debug: options.debug || false,
      ...options
    }
    
    // Добавление глобальных свойств
    app.config.globalProperties.$api = createApiService(config)
    
    // Регистрация компонентов
    registerComponents(app, config)
    
    // Регистрация директив
    registerDirectives(app, config)
    
    // Установка хуков
    setupHooks(app, config)
    
    if (config.debug) {
      console.log('Advanced plugin installed with config:', config)
    }
  }
}

function createApiService(config) {
  return {
    get: (endpoint) => fetch(`${config.apiUrl}${endpoint}`),
    post: (endpoint, data) => fetch(`${config.apiUrl}${endpoint}`, {
      method: 'POST',
      body: JSON.stringify(data)
    })
  }
}

function registerComponents(app, config) {
  // Регистрация глобальных компонентов
  app.component('GlobalModal', GlobalModal)
  app.component('GlobalButton', GlobalButton)
}

function registerDirectives(app, config) {
  // Регистрация глобальных директив
  app.directive('tooltip', createTooltipDirective(config))
}

function setupHooks(app, config) {
  // Установка глобальных хуков
  app.config.errorHandler = (err, instance, info) => {
    console.error('Plugin error:', err)
  }
}
```

## Типы плагинов

### 1. Функциональные плагины

```javascript
// functionalPlugin.js
export function createMyPlugin(options) {
  return {
    install(app) {
      // Логика установки
      app.provide('myService', createService(options))
    }
  }
}

// Использование
app.use(createMyPlugin({ apiKey: '123' }))
```

### 2. Классы-плагины

```javascript
// classPlugin.js
export class MyPlugin {
  constructor(options = {}) {
    this.options = options
    this.installed = false
  }
  
  install(app) {
    if (this.installed) return
    this.installed = true
    
    // Логика установки
    app.config.globalProperties.$myPlugin = this
  }
  
  doSomething() {
    // Публичный метод
  }
}

// Использование
const plugin = new MyPlugin(options)
app.use(plugin)
```

## Структура плагина с модулями

```javascript
// modularPlugin.js
import { authService } from './modules/auth'
import { notificationService } from './modules/notifications'
import { cacheService } from './modules/cache'

class ModularPlugin {
  constructor(options = {}) {
    this.options = options
    this.services = {}
  }
  
  install(app) {
    // Инициализация сервисов
    this.services.auth = authService(this.options.auth)
    this.services.notifications = notificationService(this.options.notifications)
    this.services.cache = cacheService(this.options.cache)
    
    // Добавление в приложение
    app.config.globalProperties.$plugin = this
    app.provide('plugin', this)
  }
  
  getAuth() {
    return this.services.auth
  }
  
  notify(message) {
    return this.services.notifications.show(message)
  }
}

export default new ModularPlugin()
```

## Плагин с асинхронной установкой

```javascript
// asyncPlugin.js
export const asyncPlugin = {
  async install(app, options) {
    // Асинхронная инициализация
    const config = await loadPluginConfig(options.configUrl)
    
    // Установка после загрузки конфига
    app.config.globalProperties.$pluginConfig = config
    
    // Регистрация зависимостей
    registerPluginComponents(app, config)
  }
}

async function loadPluginConfig(url) {
  const response = await fetch(url)
  return response.json()
}
```

## Структура плагина с подключаемыми модулями

```javascript
// extensiblePlugin.js
export class ExtensiblePlugin {
  constructor(options = {}) {
    this.options = options
    this.modules = new Map()
  }
  
  // Регистрация модуля
  registerModule(name, module) {
    this.modules.set(name, module)
    return this
  }
  
  install(app) {
    // Установка основных возможностей
    app.config.globalProperties.$plugin = this
    
    // Установка модулей
    for (const [name, module] of this.modules) {
      if (module.install) {
        module.install(app, this.options[name])
      }
    }
  }
}

// Создание плагина
const plugin = new ExtensiblePlugin()

// Регистрация модулей
plugin.registerModule('auth', authModule)
       .registerModule('storage', storageModule)
       .registerModule('api', apiModule)

export default plugin
```

## Плагин с API для других модулей

```javascript
// apiPlugin.js
export const apiPlugin = {
  install(app, options) {
    const apiClient = createApiClient(options)
    
    // Глобальный API клиент
    app.config.globalProperties.$api = apiClient
    
    // Поддержка инъекции для других плагинов
    app.provide('apiClient', apiClient)
    
    // API для расширения
    app.provide('extendApi', (extension) => {
      Object.assign(apiClient, extension)
    })
  }
}

function createApiClient(options) {
  return {
    get: (url) => fetch(`${options.baseURL}${url}`),
    post: (url, data) => fetch(`${options.baseURL}${url}`, {
      method: 'POST',
      body: JSON.stringify(data)
    }),
    
    // Метод для расширения
    extend: (extension) => {
      Object.assign(this, extension)
      return this
    }
  }
}
```

## Лучшие практики структурирования

### 1. Разделение ответственности
```
plugins/
├── index.js          # Экспорт всех плагинов
├── myPlugin/
│   ├── index.js      # Основной файл плагина
│   ├── components/   # Компоненты плагина
│   ├── directives/   # Директивы плагина
│   ├── composables/  # Композаблы плагина
│   └── types/        # Типы TypeScript (если используется)
```

### 2. Валидация опций
```javascript
export const validatedPlugin = {
  install(app, options = {}) {
    // Валидация опций
    const defaults = {
      debug: false,
      timeout: 5000,
      retries: 3
    }
    
    const config = { ...defaults, ...options }
    
    // Проверка обязательных опций
    if (config.apiKey && typeof config.apiKey !== 'string') {
      throw new Error('apiKey must be a string')
    }
    
    // Установка
    app.config.globalProperties.$validatedPlugin = { config }
  }
}
```

## Тестирование структуры плагинов

```javascript
// tests/plugin.test.js
import { createApp } from 'vue'
import { myPlugin } from '@/plugins/myPlugin'

describe('MyPlugin', () => {
  test('должен установиться без ошибок', () => {
    const app = createApp({})
    expect(() => app.use(myPlugin)).not.toThrow()
  })
  
  test('должен добавить глобальный метод', () => {
    const app = createApp({})
    app.use(myPlugin)
    
    expect(app.config.globalProperties.$myMethod).toBeDefined()
  })
  
  test('должен работать с опциями', () => {
    const app = createApp({})
    const options = { debug: true }
    
    expect(() => app.use(myPlugin, options)).not.toThrow()
  })
})
```

## Совместимость и версионирование

```javascript
// versionedPlugin.js
export const versionedPlugin = {
  version: '1.0.0', // Версия плагина
  requiredVueVersion: '^3.0.0', // Требуемая версия Vue
  
  install(app, options) {
    // Проверка версии Vue
    const vueVersion = app.version
    if (!isVersionCompatible(vueVersion, this.requiredVueVersion)) {
      throw new Error(`Plugin requires Vue ${this.requiredVueVersion}, got ${vueVersion}`)
    }
    
    // Основная установка
    setupPlugin(app, options)
  }
}
```

## Связь с другими концепциями
- [[plugins/installation]] - установка плагинов
- [[composables/Composables]] - интеграция с композаблами
- [[components/Components]] - регистрация компонентов через плагины
- [[state-management/State Management]] - интеграция с системами состояния