# Vite in TypeScript Ecosystem

Vite - это современный инструмент сборки, который обеспечивает молниеносно быструю разработку через нативную поддержку ES модулей и интеллектуальную предварительную загрузку. Vite особенно хорошо работает с TypeScript, предоставляя отличную интеграцию и мгновенный старт разработки.

## Основные особенности Vite

### Принцип работы
Vite работает по принципу "на лету" обслуживания исходных файлов через нативные ES модули, что позволяет:
- Быстро запускать сервер разработки (почти мгновенно)
- Использовать только фактически нужные файлы (без сборки всего проекта)
- Мгновенно обновлять изменения (Hot Module Replacement)

```typescript
// vite работает напрямую с TypeScript файлами:
// main.ts
import { createApp } from 'vue';
import App from './App.vue';
import './styles.css';

createApp(App).mount('#app');

// Vite понимает и транспилирует TypeScript на лету в браузере
```

### Базовая конфигурация
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';
import { readFileSync } from 'fs';

export default defineConfig({
  plugins: [],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils')
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  },
  server: {
    port: 3000,
    open: true  // автоматически открывать браузер
  }
});
```

## Интеграция с TypeScript

### tsconfig.json для Vite
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    
    /* Bundler mode */
    "moduleResolution": "node",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,      // Vite не использует tsc для вывода
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    
    /* Для разработки с Vite */
    "types": ["vite/client"]  // типы для специфичных возможностей Vite
  },
  "include": ["src"],
  "exclude": ["dist", "node_modules"]
}
```

### vite-env.d.ts
```typescript
/// <reference types="vite/client" />

// Дополнительные типы для Vite специфичных возможностей
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
  readonly VITE_DEBUG_MODE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

// Типы для HMR
interface ImportMeta {
  readonly hot?: {
    accept: (callback: (module: any) => void) => void;
    dispose: (callback: (module: any) => void) => void;
    decline: () => void;
  };
}
```

## Плагины для TypeScript

### @vitejs/plugin-typescript
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import typescript from '@vitejs/plugin-typescript';

export default defineConfig({
  plugins: [
    typescript({
      tsconfig: resolve(__dirname, 'tsconfig.vite.json'),
      // Опции для транспиляции (не проверки типов)
      transpileOnly: false,  // включить проверку типов
      experimentalDecorators: true,
      emitDecoratorMetadata: true
    })
  ]
});
```

### @vitejs/plugin-basic-ssl (для HTTPS)
```typescript
import { defineConfig } from 'vite';
import basicSsl from '@vitejs/plugin-basic-ssl';

export default defineConfig({
  plugins: [
    basicSsl(),
    typescript()
  ],
  server: {
    https: true  // теперь HTTPS по умолчанию
  }
});
```

## Практические примеры

### React с TypeScript
```typescript
// vite.config.ts для React проекта
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@components': resolve(__dirname, './src/components'),
      '@hooks': resolve(__dirname, './src/hooks'),
      '@utils': resolve(__dirname, './src/utils')
    }
  },
  server: {
    port: 3000,
    host: true,  // доступен извне (для мобильной разработки)
    hmr: {
      overlay: true  // показывать ошибки в браузере
    }
  },
  build: {
    target: 'es2020',  // для поддержки современных возможностей
    cssCodeSplit: true,
    rollupOptions: {
      output: {
        manualChunks: {
          // Разделение бандла для лучшей загрузки
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router-dom']
        }
      }
    }
  }
});
```

### Vue с TypeScript
```typescript
// vite.config.ts для Vue проекта
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { resolve } from 'path';

export default defineConfig({
  plugins: [vue({
    template: {
      // Опции для компиляции шаблонов
      compilerOptions: {
        // Настройки компиляции
      }
    }
  })],
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@': resolve(__dirname, 'src'),
    }
  },
  define: {
    // Определения для препроцессинга
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
});
```

## Современные возможности

### Server-Side Rendering (SSR)
```typescript
// vite.config.ts для SSR
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { resolve } from 'path';

export default defineConfig({
  plugins: [vue()],
  
  // Конфигурация для SSR
  build: {
    rollupOptions: {
      input: {
        // Входные точки для SSR
        server: resolve(__dirname, 'src/entry-server.ts'),
        client: resolve(__dirname, 'src/entry-client.ts')
      },
      output: {
        // Разные форматы вывода для сервера и клиента
        entryFileNames: chunkInfo => {
          return chunkInfo.name === 'server' 
            ? 'server.js' 
            : 'assets/[name].[hash].js';
        }
      }
    },
    ssr: true  // включить SSR сборку
  },
  
  ssr: {
    noExternal: ['vue']  // не исключать vue из SSR бандла
  }
});
```

### Library Mode
```typescript
// vite.config.ts для библиотеки
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      name: 'MyLibrary',
      fileName: (format) => `my-library.${format}.js`
    },
    rollupOptions: {
      external: [
        'vue',  // не включать vue в бандл библиотеки
        'react',
        'lodash'
      ],
      output: {
        globals: {
          vue: 'Vue',
          react: 'React',
          lodash: '_'
        }
      }
    }
  },
  plugins: [
    typescript({
      // При build не проверять типы (только транспиляция)
      check: false
    })
  ]
});
```

## Локальная разработка

### HMR с TypeScript
```typescript
// Компонент с HMR
import { ref } from 'vue';

const counter = ref(0);

if (import.meta.hot) {
  // Сохранение состояния при HMR
  import.meta.hot.accept((newModule) => {
    // Обновление компонента
    console.log('Module updated:', newModule);
  });
  
  // Сохранение локального состояния
  import.meta.hot.dispose(() => {
    // Очистка при замене модуля
    console.log('Disposing module');
  });
}
```

### Fast Refresh для React
```typescript
// React компонент с Fast Refresh
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// Fast Refresh работает автоматически с Vite и React
```

## Оптимизации производительности

### Pre-bundling
```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: [
      // Предварительно собрать эти зависимости
      'vue',
      'vue-router',
      'axios',
      'lodash-es'
    ],
    exclude: [
      // Исключить из предварительной сборки
      'my-private-package'
    ]
  },
  
  build: {
    rollupOptions: {
      // Кеширование для ускорения повторных сборок
      cache: true,
      
      output: {
        // Оптимизация для.tree-shaking
        manualChunks: {
          vendor: ['vue', 'vue-router']  // отдельный чанк для вендорных зависимостей
        }
      }
    }
  }
});
```

### Conditional Loading
```typescript
// Условная загрузка компонентов
async function loadFeature(featureName: string) {
  switch (featureName) {
    case 'dashboard':
      const { Dashboard } = await import('./features/Dashboard.tsx');
      return Dashboard;
    case 'reports':
      const { Reports } = await import('./features/Reports.tsx');
      return Reports;
    default:
      const { DefaultFeature } = await import('./features/Default.tsx');
      return DefaultFeature;
  }
}

// Использование с Suspense
const LazyComponent = React.lazy(() => 
  import('./HeavyComponent.tsx')
);
```

## Интеграция с TypeScript возможностями

### Типобезопасные HMR
```typescript
// typed-hmr.ts
interface HotModule {
  accept: (callback: (updatedModule: Module) => void) => void;
  dispose: (callback: () => void) => void;
  decline: () => void;
  invalidate: () => void;
}

interface Module {
  [key: string]: any;
}

// Использование в компоненте
if (import.meta.hot) {
  const hot = import.meta.hot as HotModule;
  
  hot.accept((newModule: Module) => {
    console.log('Module updated:', newModule);
  });
  
  hot.dispose(() => {
    console.log('Cleaning up before module disposal');
  });
}
```

### TypeScript в шаблонах Vue (через Volar)
```vue
<!-- Component.vue -->
<template>
  <div>{{ message }}</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

// TypeScript проверки работают в блоке script setup
interface Props {
  initialMessage?: string;
}

const props = withDefaults(defineProps<Props>(), {
  initialMessage: "Hello World"
});

const message = ref<string>(props.initialMessage);

// Функция с типизацией
const updateMessage = (newMessage: string): void => {
  message.value = newMessage;
};

// Vite + Vue + TS = отличная поддержка типов
defineExpose({
  message,
  updateMessage
});
</script>
```

## Совместимость с различными фреймворками

### SolidJS с TypeScript
```typescript
// vite.config.ts для SolidJS
import { defineConfig } from 'vite';
import solidPlugin from 'vite-plugin-solid';

export default defineConfig({
  plugins: [
    solidPlugin({
      typescript: true,
      hot: true
    })
  ],
  build: {
    target: 'esnext',
    polyfillDynamicImport: false
  }
});
```

### SvelteKit с TypeScript
```typescript
// vite.config.ts для SvelteKit
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [sveltekit()],
  kit: {
    adapter: adapter(),
    files: {
      routes: 'src/routes',
      lib: 'src/lib'
    }
  }
});
```

## Лучшие практики

### 1. Раздельные конфигурации
```typescript
// vite.config.shared.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';

export const sharedConfig = defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
    }
  },
  test: {
    environment: 'jsdom',
    globals: true
  }
});
```

```typescript
// vite.config.dev.ts
import { defineConfig } from 'vite';
import { sharedConfig } from './vite.config.shared';

export default defineConfig({
  ...sharedConfig,
  mode: 'development',
  server: {
    port: 3000,
    open: true,
    hmr: true
  }
});
```

```typescript
// vite.config.prod.ts
import { defineConfig } from 'vite';
import { sharedConfig } from './vite.config.shared';

export default defineConfig({
  ...sharedConfig,
  mode: 'production',
  build: {
    minify: 'esbuild',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router']
        }
      }
    }
  }
});
```

### 2. Управление зависимостями
```typescript
// deps.ts - централизованное управление зависимостями
export const EXTERNAL_DEPS = [
  'vue',
  'react',
  'react-dom',
  '@vueuse/core'
] as const;

export const OPTIMIZED_DEPS = [
  'axios',
  'lodash-es',
  'date-fns'
] as const;

// Использование в конфигурации
export default defineConfig({
  optimizeDeps: {
    include: OPTIMIZED_DEPS
  },
  build: {
    rollupOptions: {
      external: EXTERNAL_DEPS
    }
  }
});
```

## Ошибки и решения

### Частые проблемы

#### 1. Проблемы с типами
```typescript
// Если типы не находятся, убедитесь, что:
// 1. Добавлены в tsconfig.json
// 2. Используется правильная версия @vitejs/plugin-*

// tsconfig.json
{
  "compilerOptions": {
    "types": ["vite/client", "node"]  // типы Vite и Node.js
  }
}
```

#### 2. Проблемы с HMR
```typescript
// Убедитесь, что используете последние версии плагинов
// и правильно настроили режим разработки
export default defineConfig({
  plugins: [/* фреймворк-специфичный плагин */],
  server: {
    hmr: {
      overlay: true  // показывать ошибки поверх UI
    }
  }
});
```

### Производительность
```typescript
// Оптимизации для крупных проектов
export default defineConfig({
  cacheDir: '.vite',  // директория для кеша
  
  optimizeDeps: {
    force: true  // форсировать предварительную сборку зависимостей
  },
  
  build: {
    minify: 'esbuild',  // самый быстрый минификатор
    rollupOptions: {
      // Параллельная обработка
      maxParallelFileOps: 8
    }
  }
});
```

## Интеграция с инструментами

### Совместимость с ESLint
```typescript
// vite.config.ts с ESLint
import { defineConfig } from 'vite';
import eslint from 'vite-plugin-eslint';

export default defineConfig({
  plugins: [
    eslint({
      include: ['src/**/*.ts', 'src/**/*.tsx', 'src/**/*.vue']
    })
  ]
});
```

### Совместимость с Prettier
```typescript
// Использование через линтеры или VS Code
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## Сравнение с другими инструментами

| Функция | Vite | Webpack | Parcel |
|---------|------|---------|--------|
| Старт разработки | <1с | 5-30с | 1-5с |
| HMR | Быстр | Медлен | Быстр |
| TypeScript | NATIVELY | Loader | NATIVELY |
| ES Modules | NATIVELY | через Babel | NATIVELY |
| Tree Shaking | Лучше | Хорошо | Средне |
| Конфигурация | Просто | Сложно | Минимум |

## Заключение

Vite в TypeScript экосистеме:

1. **Обеспечивает молниеносную разработку** - благодаря нативной поддержке ES Modules
2. **Предоставляет лучшую производительность** - за счет умной предварительной загрузки
3. **Поддерживает современные стандарты** - ES Modules, TypeScript, JSX
4. **Имеет отличную интеграцию** - с популярными фреймворками
5. **Упрощает настройку** - минимальная конфигурация для большинства случаев

Vite стал стандартом де-факто для современных TypeScript проектов, особенно в сочетании с фреймворками следующего поколения.

## Связь с другими концепциями
- [[typescript-compiler]] - интеграция с tsconfig и проверками типов
- [[build-performance]] - производительность сборки
- [[es-modules]] - ES модули как основа Vite
- [[frameworks-integration]] - интеграция с разными фреймворками
- [[deployment/modern-deployment]] - деплой современных сборок