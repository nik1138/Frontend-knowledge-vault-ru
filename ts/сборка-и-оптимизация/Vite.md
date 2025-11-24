---
aliases: [Сборка Vite, Vite конфигурация]
tags: [ts, build, vite, bundler, development]
---

# Vite

Vite - это современный инструмент сборки, созданный для обеспечения более быстрой и гибкой разработки. В отличие от традиционных сборщиков, Vite использует нативные возможности ES-модулей в браузере, что значительно ускоряет время запуска и горячую перезагрузку.

## Архитектура Vite

Vite работает по принципу разделения на:

- **Development server** - использует нативные ES-модули браузера для быстрой загрузки
- **Build system** - использует Rollup для создания продакшен-бандлов

## Основная конфигурация для TypeScript

Файл конфигурации Vite обычно называется `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
    },
  },
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
      },
    },
  },
});
```

## Поддержка TypeScript

Vite по умолчанию поддерживает TypeScript без дополнительной настройки. Однако для полной функциональности рекомендуется создать `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

## Плагины для TypeScript-проектов

### @vitejs/plugin-vue или @vitejs/plugin-react

Для фреймворков Vue или React:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
});
```

### TypeScript-плагины

Для дополнительной обработки TypeScript можно использовать плагины:

```typescript
import { defineConfig } from 'vite';
import dts from 'vite-plugin-dts';

export default defineConfig({
  plugins: [
    dts({
      insertTypesEntry: true,
    }),
  ],
});
```

## Горячая перезагрузка (HMR)

Vite предоставляет отличную поддержку HMR, которая работает на уровне модулей:

```typescript
// При изменении модуля обновляется только он, без перезагрузки всей страницы
import { someFunction } from './utils';

// Изменения в utils.ts автоматически обновятся в браузере
```

## Оптимизация производительности

### Pre-bundling зависимостей

Vite автоматически предварительно собирает зависимости для улучшения производительности:

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: ['some-heavy-package'],
  },
});
```

### Код-сплиттинг

Vite поддерживает динамические импорты для автоматического разделения кода:

```typescript
// Ленивая загрузка
const module = await import('./heavy-module');
```

## Режимы разработки и продакшена

### Development

```bash
vite
```

### Build

```bash
vite build
```

### Preview

```bash
vite preview
```

## Сравнение с Webpack

| Особенность | Vite | Webpack |
|-------------|------|---------|
| Старт разработки | Мгновенный | Медленный |
| HMR | Быстрый | Медленный |
| Продакшен-бандл | На основе Rollup | Собственный |
| Конфигурация | Простая | Сложная |

## Практические советы

1. **Используйте псевдонимы** для упрощения импортов
2. **Настройте tsconfig.json** правильно для работы с Vite
3. **Используйте динамические импорты** для разделения кода
4. **Воспользуйтесь встроенными возможностями** для тестирования и линтинга

## Связанные темы

- [[Webpack]] - традиционный сборщик
- [[Rollup]] - сборщик библиотек
- [[Оптимизация-бандлов]] - техники оптимизации
- [[Tree-shaking]] - удаление неиспользуемого кода