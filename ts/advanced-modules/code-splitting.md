# Разделение кода в TypeScript

Разделение кода (Code Splitting) - это техника оптимизации, которая позволяет разбить большой бандл на меньшие части, загружаемые по мере необходимости. Это улучшает время загрузки приложения и уменьшает объем передаваемых данных.

## Содержание

1. [Основы разделения кода](#основы-разделения-кода)
2. [Динамические импорты](#динамические-импорты)
3. [Роутинг и разделение кода](#роутинг-и-разделение-кода)
4. [Практические примеры](#практические-примеры)
5. [Продвинутые техники](#продвинутые-техники)
6. [Оптимизация и лучшие практики](#оптимизация-и-лучшие-практики)
7. [Мониторинг и анализ](#мониторинг-и-анализ)

## Основы разделения кода

Разделение кода позволяет загружать только необходимый код для текущего состояния приложения, уменьшая начальный размер бандла и улучшая производительность.

### Проблемы без разделения кода

```typescript
// Без разделения кода - всё в одном бандле
// app.ts
import { HomePage } from './pages/Home';
import { ProfilePage } from './pages/Profile';
import { AdminPage } from './pages/Admin';
import { SettingsPage } from './pages/Settings';
import { ReportsPage } from './pages/Reports';
import { AnalyticsPage } from './pages/Analytics';
// ... ещё 50 страниц

// Все страницы загружаются сразу, даже если пользователь
// посещает только домашнюю страницу
```

### Преимущества разделения кода

1. **Уменьшение начального размера бандла**
2. **Улучшение времени загрузки**
3. **Эффективное использование кэша**
4. **Параллельная загрузка чанков**
5. **Ленивая загрузка неиспользуемого кода**

### Как работает разделение кода

```typescript
// С разделением кода - динамические импорты
// app.ts
async function loadHomePage() {
  const { HomePage } = await import('./pages/Home');
  return HomePage;
}

async function loadProfilePage() {
  const { ProfilePage } = await input('./pages/Profile');
  return ProfilePage;
}

// Только нужный код загружается по требованию
```

## Динамические импорты

Динамические импорты - основа разделения кода в TypeScript и JavaScript.

### Базовый синтаксис

```typescript
// Статический импорт
import { add } from './math';

// Динамический импорт
const mathModule = await import('./math');
const result = mathModule.add(2, 3);

// Или с then/catch
import('./math')
  .then(module => {
    const result = module.add(2, 3);
  })
  .catch(error => {
    console.error('Failed to load module:', error);
  });
```

### Типизация динамических импортов

```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export default class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// main.ts
interface MathModule {
  add: (a: number, b: number) => number;
  multiply: (a: number, b: number) => number;
  default: new () => { add: (a: number, b: number) => number };
}

async function loadMath(): Promise<MathModule> {
  return await import('./math');
}

// Использование
const math = await loadMath();
const sum = math.add(2, 3);
const calc = new math.default();
```

### Обработка ошибок

```typescript
async function safeImport<T>(modulePath: string): Promise<T | null> {
  try {
    const module = await import(modulePath);
    return module;
  } catch (error) {
    console.error(`Failed to import ${modulePath}:`, error);
    return null;
  }
}

// С повторными попытками
async function importWithRetry<T>(
  modulePath: string, 
  maxAttempts: number = 3
): Promise<T> {
  let lastError: unknown;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const module = await import(modulePath);
      if (attempt > 1) {
        console.log(`Module loaded on attempt ${attempt}`);
      }
      return module;
    } catch (error) {
      lastError = error;
      console.warn(`Attempt ${attempt} failed for ${modulePath}:`, error);
      
      if (attempt < maxAttempts) {
        await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
      }
    }
  }
  
  throw new Error(`Failed to import ${modulePath} after ${maxAttempts} attempts: ${lastError}`);
}
```

## Роутинг и разделение кода

Разделение кода особенно эффективно при использовании с роутингом.

### React Router с разделением кода

```typescript
// routes.tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// Ленивая загрузка компонентов
const HomePage = lazy(() => import('./pages/Home'));
const ProfilePage = lazy(() => import('./pages/Profile'));
const AdminPage = lazy(() => import('./pages/Admin'));
const SettingsPage = lazy(() => import('./pages/Settings'));

// Компонент загрузки
const LoadingSpinner = () => <div>Loading...</div>;

function AppRoutes() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/profile" element={<ProfilePage />} />
        <Route path="/admin" element={<AdminPage />} />
        <Route path="/settings" element={<SettingsPage />} />
      </Routes>
    </Suspense>
  );
}
```

### Webpack magic comments

```typescript
// Использование Webpack magic comments для управления чанками
const HomePage = lazy(() => import(
  /* webpackChunkName: "home-page" */
  './pages/Home'
));

const ProfilePage = lazy(() => import(
  /* webpackChunkName: "profile-page" */
  /* webpackPreload: true */
  './pages/Profile'
));

const AdminPage = lazy(() => import(
  /* webpackChunkName: "admin-page" */
  /* webpackPrefetch: true */
  './pages/Admin'
));

// Предзагрузка критических чанков
const CriticalComponent = lazy(() => import(
  /* webpackChunkName: "critical" */
  /* webpackPreload: true */
  './components/CriticalComponent'
));

// Предзапрос не критических чанков
const HeavyComponent = lazy(() => import(
  /* webpackChunkName: "heavy" */
  /* webpackPrefetch: true */
  './components/HeavyComponent'
));
```

### Next.js с разделением кода

```typescript
// pages/index.tsx
import dynamic from 'next/dynamic';

// Ленивая загрузка компонентов
const HeavyComponent = dynamic(() => import('../components/HeavyComponent'));
const ChartComponent = dynamic(() => import('../components/ChartComponent'));

// С отключением SSR
const ClientOnlyComponent = dynamic(
  () => import('../components/ClientOnlyComponent'),
  { ssr: false }
);

// С компонентом загрузки
const LoadingComponent = () => <div>Loading...</div>;

const AdvancedComponent = dynamic(
  () => import('../components/AdvancedComponent'),
  { loading: () => <LoadingComponent /> }
);

export default function HomePage() {
  return (
    <div>
      <h1>Home Page</h1>
      <HeavyComponent />
      <ChartComponent />
      <ClientOnlyComponent />
      <AdvancedComponent />
    </div>
  );
}
```

## Практические примеры

### 1. Плагинная архитектура

```typescript
// plugin-manager.ts
interface Plugin {
  name: string;
  version: string;
  initialize(): Promise<void>;
  execute(data: any): any;
}

class PluginManager {
  private plugins: Map<string, Plugin> = new Map();
  
  async loadPlugin(pluginName: string): Promise<Plugin | null> {
    try {
      // Динамическая загрузка плагина
      const module = await import(`./plugins/${pluginName}`);
      const pluginClass = module.default;
      const plugin: Plugin = new pluginClass();
      
      await plugin.initialize();
      this.plugins.set(plugin.name, plugin);
      
      return plugin;
    } catch (error) {
      console.error(`Failed to load plugin ${pluginName}:`, error);
      return null;
    }
  }
  
  async executePlugin(name: string, data: any): Promise<any> {
    const plugin = this.plugins.get(name);
    if (plugin) {
      return plugin.execute(data);
    }
    throw new Error(`Plugin ${name} not found`);
  }
}

// plugins/analytics.ts
export default class AnalyticsPlugin implements Plugin {
  name = 'analytics';
  version = '1.0.0';
  
  async initialize(): Promise<void> {
    console.log('Analytics plugin initialized');
  }
  
  execute(data: any): any {
    // Логика аналитики
    return { ...data, tracked: true };
  }
}

// plugins/notifications.ts
export default class NotificationsPlugin implements Plugin {
  name = 'notifications';
  version = '1.0.0';
  
  async initialize(): Promise<void> {
    console.log('Notifications plugin initialized');
  }
  
  execute(data: any): any {
    // Логика уведомлений
    return { ...data, notified: true };
  }
}

// Использование
const pluginManager = new PluginManager();
const analytics = await pluginManager.loadPlugin('analytics');
const result = await pluginManager.executePlugin('analytics', { data: 'test' });
```

### 2. Условная загрузка функциональности

```typescript
// feature-loader.ts
interface Feature {
  name: string;
  isEnabled: boolean;
  load(): Promise<void>;
  execute(data: any): any;
}

class FeatureLoader {
  private features: Map<string, Feature> = new Map();
  private loadedFeatures: Set<string> = new Set();
  
  async loadFeature(featureName: string, condition: boolean): Promise<boolean> {
    // Проверяем условие загрузки
    if (!condition) {
      return false;
    }
    
    // Проверяем, загружена ли уже функциональность
    if (this.loadedFeatures.has(featureName)) {
      return true;
    }
    
    try {
      // Динамическая загрузка функциональности
      const module = await import(`./features/${featureName}`);
      const featureClass = module.default;
      const feature: Feature = new featureClass();
      
      await feature.load();
      this.features.set(featureName, feature);
      this.loadedFeatures.add(featureName);
      
      return true;
    } catch (error) {
      console.error(`Failed to load feature ${featureName}:`, error);
      return false;
    }
  }
  
  async executeFeature(featureName: string, data: any): Promise<any> {
    const feature = this.features.get(featureName);
    if (feature && feature.isEnabled) {
      return feature.execute(data);
    }
    throw new Error(`Feature ${featureName} not available`);
  }
}

// features/debug.ts
export default class DebugFeature implements Feature {
  name = 'debug';
  isEnabled = process.env.NODE_ENV === 'development';
  
  async load(): Promise<void> {
    console.log('Debug feature loaded');
  }
  
  execute(data: any): any {
    console.log('Debug data:', data);
    return { ...data, debug: true };
  }
}

// features/analytics.ts
export default class AnalyticsFeature implements Feature {
  name = 'analytics';
  isEnabled = true;
  
  async load(): Promise<void> {
    // Инициализация аналитики
    console.log('Analytics feature loaded');
  }
  
  execute(data: any): any {
    // Отправка аналитики
    return { ...data, tracked: true };
  }
}

// Использование
const featureLoader = new FeatureLoader();
await featureLoader.loadFeature('debug', process.env.NODE_ENV === 'development');
await featureLoader.loadFeature('analytics', true);

if (process.env.NODE_ENV === 'development') {
  await featureLoader.executeFeature('debug', { message: 'test' });
}
```

### 3. Локализация с разделением кода

```typescript
// i18n.ts
interface TranslationModule {
  default: Record<string, string>;
}

class I18nManager {
  private currentLocale: string = 'en';
  private translations: Map<string, Record<string, string>> = new Map();
  
  async loadLocale(locale: string): Promise<void> {
    if (this.translations.has(locale)) {
      this.currentLocale = locale;
      return;
    }
    
    try {
      // Динамическая загрузка переводов
      const module: TranslationModule = await import(`./locales/${locale}.json`);
      this.translations.set(locale, module.default);
      this.currentLocale = locale;
    } catch (error) {
      console.error(`Failed to load locale ${locale}:`, error);
      // Загружаем английский как резервный вариант
      if (locale !== 'en') {
        await this.loadLocale('en');
      }
    }
  }
  
  t(key: string, params?: Record<string, any>): string {
    const translations = this.translations.get(this.currentLocale);
    if (!translations) {
      return key;
    }
    
    let translation = translations[key] || key;
    
    // Подстановка параметров
    if (params) {
      Object.keys(params).forEach(param => {
        translation = translation.replace(`{{${param}}}`, params[param]);
      });
    }
    
    return translation;
  }
}

// locales/en.json
{
  "welcome": "Welcome",
  "hello_user": "Hello, {{name}}!",
  "items_count": "You have {{count}} items"
}

// locales/ru.json
{
  "welcome": "Добро пожаловать",
  "hello_user": "Привет, {{name}}!",
  "items_count": "У вас {{count}} элементов"
}

// Использование
const i18n = new I18nManager();
await i18n.loadLocale('ru');
console.log(i18n.t('welcome')); // Добро пожаловать
console.log(i18n.t('hello_user', { name: 'Иван' })); // Привет, Иван!
```

## Продвинутые техники

### 1. Предзагрузка и предзапрос

```typescript
// preload-manager.ts
class PreloadManager {
  private preloadQueue: string[] = [];
  private prefetched: Set<string> = new Set();
  
  // Добавление в очередь предзагрузки
  preload(modulePath: string): void {
    if (!this.prefetched.has(modulePath)) {
      this.preloadQueue.push(modulePath);
    }
  }
  
  // Предзагрузка всех модулей в очереди
  async preloadAll(): Promise<void> {
    const promises = this.preloadQueue.map(async (path) => {
      try {
        await import(path);
        this.prefetched.add(path);
        console.log(`Preloaded module: ${path}`);
      } catch (error) {
        console.error(`Failed to preload module ${path}:`, error);
      }
    });
    
    await Promise.all(promises);
    this.preloadQueue = [];
  }
  
  // Предзапрос модуля (низкий приоритет)
  prefetch(modulePath: string): void {
    if (this.prefetched.has(modulePath)) {
      return;
    }
    
    // Используем link rel="prefetch"
    const link = document.createElement('link');
    link.rel = 'prefetch';
    link.href = modulePath;
    document.head.appendChild(link);
    
    this.prefetched.add(modulePath);
  }
}

// Использование
const preloadManager = new PreloadManager();

// Предзагрузка критических модулей
preloadManager.preload('./critical-module');
setTimeout(() => {
  preloadManager.preloadAll();
}, 1000);

// Предзапрос не критических модулей
preloadManager.prefetch('./heavy-module');
preloadManager.prefetch('./analytics-module');
```

### 2. Интеллектуальное разделение кода

```typescript
// smart-code-splitting.ts
interface ModuleInfo {
  path: string;
  priority: 'critical' | 'high' | 'medium' | 'low';
  dependencies?: string[];
  size?: number;
}

class SmartCodeSplitter {
  private moduleRegistry: Map<string, ModuleInfo> = new Map();
  private loadedModules: Set<string> = new Set();
  
  registerModule(name: string, info: ModuleInfo): void {
    this.moduleRegistry.set(name, info);
  }
  
  async loadModule(name: string): Promise<any> {
    const moduleInfo = this.moduleRegistry.get(name);
    if (!moduleInfo) {
      throw new Error(`Module ${name} not registered`);
    }
    
    if (this.loadedModules.has(name)) {
      return import(moduleInfo.path);
    }
    
    // Загружаем зависимости
    if (moduleInfo.dependencies) {
      await Promise.all(
        moduleInfo.dependencies.map(dep => this.loadModule(dep))
      );
    }
    
    // Загружаем сам модуль
    const module = await import(moduleInfo.path);
    this.loadedModules.add(name);
    
    return module;
  }
  
  // Загрузка модулей по приоритету
  async loadByPriority(priority: 'critical' | 'high' | 'medium' | 'low'): Promise<void> {
    const modules = Array.from(this.moduleRegistry.entries())
      .filter(([_, info]) => info.priority === priority)
      .map(([name, _]) => name);
    
    await Promise.all(modules.map(name => this.loadModule(name)));
  }
}

// Регистрация модулей
const codeSplitter = new SmartCodeSplitter();
codeSplitter.registerModule('auth', {
  path: './modules/auth',
  priority: 'critical',
  size: 10240 // 10KB
});

codeSplitter.registerModule('analytics', {
  path: './modules/analytics',
  priority: 'low',
  dependencies: ['auth'],
  size: 5120 // 5KB
});

codeSplitter.registerModule('reports', {
  path: './modules/reports',
  priority: 'high',
  dependencies: ['auth'],
  size: 20480 // 20KB
});

// Использование
await codeSplitter.loadByPriority('critical');
// Загружены критические модули

setTimeout(() => {
  codeSplitter.loadByPriority('high');
  // Загружены модули высокого приоритета
}, 2000);
```

### 3. Адаптивное разделение кода

```typescript
// adaptive-code-splitting.ts
interface DeviceInfo {
  isMobile: boolean;
  connectionSpeed: 'slow' | 'medium' | 'fast';
  memory: 'low' | 'medium' | 'high';
}

class AdaptiveCodeSplitter {
  private deviceInfo: DeviceInfo;
  
  constructor() {
    this.deviceInfo = this.getDeviceInfo();
  }
  
  private getDeviceInfo(): DeviceInfo {
    return {
      isMobile: /mobile/i.test(navigator.userAgent),
      connectionSpeed: this.getConnectionSpeed(),
      memory: this.getMemoryInfo()
    };
  }
  
  private getConnectionSpeed(): 'slow' | 'medium' | 'fast' {
    // @ts-ignore
    if (navigator.connection) {
      // @ts-ignore
      const connection = navigator.connection;
      const downlink = connection.downlink || 0;
      
      if (downlink < 1) return 'slow';
      if (downlink < 5) return 'medium';
      return 'fast';
    }
    
    return 'medium';
  }
  
  private getMemoryInfo(): 'low' | 'medium' | 'high' {
    // @ts-ignore
    if (navigator.deviceMemory) {
      // @ts-ignore
      const memory = navigator.deviceMemory;
      
      if (memory < 2) return 'low';
      if (memory < 4) return 'medium';
      return 'high';
    }
    
    return 'medium';
  }
  
  async loadModule(modulePath: string, size: number): Promise<any> {
    // Адаптируем стратегию загрузки под устройство
    if (this.deviceInfo.isMobile || this.deviceInfo.connectionSpeed === 'slow') {
      // Для мобильных и медленных соединений
      if (size > 50000) { // 50KB
        console.log('Large module detected, using progressive loading');
        // Используем прогрессивную загрузку
        return this.loadModuleProgressively(modulePath);
      }
    }
    
    // Нормальная загрузка
    return import(modulePath);
  }
  
  private async loadModuleProgressively(modulePath: string): Promise<any> {
    // Показываем индикатор прогресса
    console.log('Loading large module progressively...');
    
    // Загружаем модуль с индикатором прогресса
    const module = await import(modulePath);
    console.log('Module loaded successfully');
    
    return module;
  }
}

// Использование
const adaptiveSplitter = new AdaptiveCodeSplitter();
const largeModule = await adaptiveSplitter.loadModule('./large-module', 100000); // 100KB
```

## Оптимизация и лучшие практики

### 1. Размер чанков

```typescript
// webpack.config.js с оптимизацией размера чанков
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 20000, // Минимальный размер чанка 20KB
      maxSize: 244000, // Максимальный размер чанка 244KB
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          priority: 10,
          reuseExistingChunk: true
        },
        common: {
          minChunks: 2,
          chunks: 'all',
          priority: 5,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

### 2. Предзагрузка критических путей

```typescript
// critical-path-preloading.ts
class CriticalPathPreloader {
  private criticalPaths: string[] = [];
  
  addCriticalPath(path: string): void {
    this.criticalPaths.push(path);
  }
  
  async preloadCriticalPaths(): Promise<void> {
    // Предзагружаем критические модули немедленно
    const criticalImports = this.criticalPaths.map(path => import(path));
    await Promise.all(criticalImports);
  }
  
  startBackgroundPreloading(): void {
    // Начинаем фоновую предзагрузку после загрузки критических модулей
    setTimeout(async () => {
      const backgroundModules = [
        './modules/analytics',
        './modules/notifications',
        './modules/logging'
      ];
      
      for (const modulePath of backgroundModules) {
        try {
          await import(modulePath);
          console.log(`Background module preloaded: ${modulePath}`);
        } catch (error) {
          console.warn(`Failed to preload ${modulePath}:`, error);
        }
      }
    }, 2000); // Начинаем через 2 секунды
  }
}

// Использование
const preloader = new CriticalPathPreloader();
preloader.addCriticalPath('./modules/auth');
preloader.addCriticalPath('./modules/router');

await preloader.preloadCriticalPaths();
preloader.startBackgroundPreloading();
```

### 3. Кэширование и версионирование

```typescript
// module-cache.ts
class ModuleCache {
  private cache: Map<string, { module: any; timestamp: number }> = new Map();
  private cacheTimeout: number = 5 * 60 * 1000; // 5 минут
  
  async loadModule(modulePath: string, version?: string): Promise<any> {
    const cacheKey = version ? `${modulePath}?v=${version}` : modulePath;
    const cached = this.cache.get(cacheKey);
    
    // Проверяем кэш
    if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
      console.log(`Module loaded from cache: ${modulePath}`);
      return cached.module;
    }
    
    // Загружаем модуль
    try {
      const module = await import(modulePath);
      
      // Сохраняем в кэш
      this.cache.set(cacheKey, {
        module,
        timestamp: Date.now()
      });
      
      console.log(`Module loaded and cached: ${modulePath}`);
      return module;
    } catch (error) {
      // При ошибке загрузки пробуем использовать кэш
      if (cached) {
        console.warn(`Failed to load module ${modulePath}, using cached version`);
        return cached.module;
      }
      
      throw error;
    }
  }
  
  clearCache(): void {
    this.cache.clear();
  }
  
  clearExpired(): void {
    const now = Date.now();
    for (const [key, value] of this.cache.entries()) {
      if (now - value.timestamp >= this.cacheTimeout) {
        this.cache.delete(key);
      }
    }
  }
}

// Использование
const moduleCache = new ModuleCache();
const module = await moduleCache.loadModule('./my-module', '1.2.3');
```

## Мониторинг и анализ

### 1. Метрики производительности

```typescript
// performance-monitoring.ts
interface LoadMetrics {
  moduleName: string;
  loadTime: number;
  size: number;
  success: boolean;
  error?: string;
}

class CodeSplittingMonitor {
  private metrics: LoadMetrics[] = [];
  
  async loadModuleWithMetrics<T>(moduleName: string, loader: () => Promise<T>): Promise<T> {
    const startTime = performance.now();
    let size = 0;
    
    try {
      const module = await loader();
      
      const endTime = performance.now();
      const loadTime = endTime - startTime;
      
      // Оцениваем размер модуля (приблизительно)
      size = this.estimateModuleSize(moduleName);
      
      this.metrics.push({
        moduleName,
        loadTime,
        size,
        success: true
      });
      
      console.log(`Module ${moduleName} loaded in ${loadTime.toFixed(2)}ms (${size} bytes)`);
      
      return module;
    } catch (error) {
      const endTime = performance.now();
      const loadTime = endTime - startTime;
      
      this.metrics.push({
        moduleName,
        loadTime,
        size,
        success: false,
        error: error instanceof Error ? error.message : String(error)
      });
      
      console.error(`Failed to load module ${moduleName}:`, error);
      throw error;
    }
  }
  
  private estimateModuleSize(moduleName: string): number {
    // Простая эвристика для оценки размера
    // В реальности можно использовать более сложные методы
    const baseSize = 10000; // 10KB базовый размер
    const nameLength = moduleName.length;
    return baseSize + (nameLength * 100);
  }
  
  getMetrics(): LoadMetrics[] {
    return [...this.metrics];
  }
  
  getAverageLoadTime(): number {
    if (this.metrics.length === 0) return 0;
    const total = this.metrics.reduce((sum, metric) => sum + metric.loadTime, 0);
    return total / this.metrics.length;
  }
  
  getSuccessRate(): number {
    if (this.metrics.length === 0) return 0;
    const successCount = this.metrics.filter(m => m.success).length;
    return (successCount / this.metrics.length) * 100;
  }
}

// Использование
const monitor = new CodeSplittingMonitor();

const homeModule = await monitor.loadModuleWithMetrics(
  'home-page',
  () => import('./pages/Home')
);

const profileModule = await monitor.loadModuleWithMetrics(
  'profile-page',
  () => import('./pages/Profile')
);

console.log('Average load time:', monitor.getAverageLoadTime().toFixed(2) + 'ms');
console.log('Success rate:', monitor.getSuccessRate().toFixed(2) + '%');
```

### 2. Аналитика использования

```typescript
// usage-analytics.ts
class CodeSplittingAnalytics {
  private usageStats: Map<string, { 
    loadCount: number; 
    totalLoadTime: number;
    lastUsed: number;
  }> = new Map();
  
  trackModuleLoad(moduleName: string, loadTime: number): void {
    const stats = this.usageStats.get(moduleName) || {
      loadCount: 0,
      totalLoadTime: 0,
      lastUsed: 0
    };
    
    stats.loadCount++;
    stats.totalLoadTime += loadTime;
    stats.lastUsed = Date.now();
    
    this.usageStats.set(moduleName, stats);
  }
  
  getMostUsedModules(limit: number = 10): string[] {
    return Array.from(this.usageStats.entries())
      .sort((a, b) => b[1].loadCount - a[1].loadCount)
      .slice(0, limit)
      .map(([name, _]) => name);
  }
  
  getSlowestModules(limit: number = 10): string[] {
    return Array.from(this.usageStats.entries())
      .map(([name, stats]) => ({
        name,
        avgLoadTime: stats.totalLoadTime / stats.loadCount
      }))
      .sort((a, b) => b.avgLoadTime - a.avgLoadTime)
      .slice(0, limit)
      .map(item => item.name);
  }
  
  getUnusedModules(thresholdDays: number = 30): string[] {
    const threshold = Date.now() - (thresholdDays * 24 * 60 * 60 * 1000);
    
    return Array.from(this.usageStats.entries())
      .filter(([_, stats]) => stats.lastUsed < threshold)
      .map(([name, _]) => name);
  }
  
  report(): void {
    console.log('=== Code Splitting Analytics Report ===');
    console.log('Most used modules:', this.getMostUsedModules(5));
    console.log('Slowest modules:', this.getSlowestModules(5));
    console.log('Unused modules (30 days):', this.getUnusedModules(30));
  }
}

// Использование
const analytics = new CodeSplittingAnalytics();

// Оборачиваем загрузку модулей
async function loadModule(moduleName: string, loader: () => Promise<any>) {
  const startTime = Date.now();
  const module = await loader();
  const loadTime = Date.now() - startTime;
  
  analytics.trackModuleLoad(moduleName, loadTime);
  
  return module;
}

// Загрузка модулей с аналитикой
await loadModule('home-page', () => import('./pages/Home'));
await loadModule('profile-page', () => import('./pages/Profile'));

// Генерируем отчет
analytics.report();
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Оптимизация для сборки
- [[ts/advanced-modules/dynamic-imports|Динамические импорты]] - Динамическая загрузка модулей
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора

## Рекомендации по изучению

1. Практикуйтесь в базовом разделении кода с динамическими импортами
2. Освойте интеграцию с роутингом (React Router, Next.js)
3. Изучите Webpack magic comments и управление чанками
4. Практикуйтесь в создании плагинных архитектур
5. Освойте предзагрузку и предзапрос модулей
6. Изучите мониторинг и аналитику разделения кода
7. Практикуйтесь в адаптивном разделении кода

## Следующие шаги

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[TypeScript с React|TypeScript с React]] - Типобезопасная разработка React-приложений
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей