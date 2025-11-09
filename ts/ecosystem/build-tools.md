# Build Tools in TypeScript Ecosystem

Инструменты сборки играют ключевую роль в TypeScript экосистеме, обеспечивая компиляцию, преобразование, оптимизацию и упаковку кода. Эти инструменты превращают TypeScript исходники в исполняемый JavaScript код.

## Основные инструменты сборки

### [TypeScript Compiler (tsc)](typescript-compiler.md)
Официальный компилятор TypeScript, преобразующий .ts файлы в JavaScript.

### [Webpack](webpack.md)
Модульный сборщик, который может работать с TypeScript через loaders.

### [Rollup](rollup.md)
Сборщик библиотек, оптимизированный для ES Modules.

### [Vite](vite.md)
Современный сборщик, обеспечивающий быструю разработку и сборку.

### [Parcel](parcel.md)
Нулевая конфигурация сборщик с встроенной поддержкой TypeScript.

### [Esbuild](esbuild.md)
Очень быстрый JavaScript и TypeScript bundler и minifier.

### [SWC (Speedy Web Compiler)](swc.md)
Быстрый компилятор, написанный на Rust, как альтернатива tsc и Babel.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │            Build Tools Ecosystem                        │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Compiler      │               │   Module Bundlers      │                                    │  Fast Builders  │
  │  Core (tsc)     │               │                        │                                    │                 │
  │                 │               │ - Webpack              │                                    │ - Esbuild       │
  │ - Type          │◄──────────────┤ - Rollup               │─────────────────────────────────────►│ - SWC           │
  │   Checking        │               │ - Vite                 │                                    │ - Sucrase       │
  │ - Transpilation   │               │ - Parcel               │                                    │ - LightningCSS  │
  │ - Declaration     │               │ - Snowpack             │                                    │ - Rome          │
  │   Files           │               │ - Configuration        │                                    │ - Compilation   │
  │ - Source Maps     │               │ - Plugins              │                                    │   Speed         │
  └─────────────────┘               │ - Code Splitting         │                                    └─────────────────┘
        │                           └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │        Preprocessors          │
        │                           │                               │
        │                           │ - Babel                        │
        │                           │ - TypeScript + Babel         │
        │                           │ - Transpilation Pipelines      │
        │                           │ - Polyfills & Targets        │
        │                           │ - JSX Processing               │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                        Optimization Tools                       │
        │                           │                                                                 │
        │                           │ - Minification (Terser)                                         │
        │                           │ - Tree Shaking                                                  │
        │                           │ - Dead Code Elimination                                         │
        │                           │ - Bundle Analysis                                              │
        │                           │ - Code Splitting                                               │
        │                           │ - Asset Optimization                                            │
        │                           │ - Compression                                                  │
        └───────────────────────────┤ - Performance Metrics                                           │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## TypeScript Compiler (tsc)

### Базовая настройка
```json
// tsconfig.json
{
  "compilerOptions": {
    /* Основные опции */
    "target": "ES2020",                          /* Уровень ECMAScript */
    "module": "ESNext",                         /* Система модулей */
    "lib": ["ES2020", "DOM"],                   /* Целевые библиотеки */
    
    /* Системы модулей */
    "moduleResolution": "node",                 /* Стратегия разрешения модулей */
    "baseUrl": ".",                            /* Базовый путь для некорректных модулей */
    "paths": {                                 /* Псевдонимы для путей */
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    },
    
    /* Типизация */
    "strict": true,                            /* Включить все строгие проверки */
    "esModuleInterop": true,                   /* Совместимость ES Modules */
    "skipLibCheck": true,                      /* Пропускать проверки .d.ts */
    "forceConsistentCasingInFileNames": true,  /* Строгая проверка регистра файлов */
    
    /* Вывод */
    "outDir": "./dist",                        /* Выходная директория */
    "rootDir": "./src",                        /* Корневая директория исходников */
    "removeComments": true,                    /* Удалять комментарии */
    "sourceMap": true,                         /* Генерировать sourcemaps */
    
    /* Declaration файлы */
    "declaration": true,                       /* Генерировать .d.ts файлы */
    "declarationMap": true,                    /* Генерировать .d.ts map файлы */
    "emitDeclarationOnly": false,              /* Только declaration файлы */
    
    /* Advanced */
    "isolatedModules": true,                   /* Каждый файл как модуль ES */
    "allowSyntheticDefaultImports": true,      /* Позволить synthetic default imports */
    "resolveJsonModule": true,                 /* Разрешать import .json файлов */
    "noEmitOnError": true                      /* Не выводить при ошибках */
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

### Продвинутые настройки
```json
{
  "compilerOptions": {
    /* Производительность */
    "incremental": true,                       /* Инкрементная компиляция */
    "composite": true,                         /* Поддержка проектных ссылок */
    "tsBuildInfoFile": "./.tsbuildinfo",       /* Файл для инкрементной компиляции */
    
    /* Проектные ссылки */
    "declarationMap": true,
    "sourceMap": true,
    "inlineSources": true,
    
    /* Экспериментальные возможности */
    "experimentalDecorators": true,            /* Decorators поддержка */
    "emitDecoratorMetadata": true,             /* Metadata для декораторов */
    
    /* Опциональные проверки */
    "noUnusedLocals": true,                    /* Ошибка при неиспользуемых locals */
    "noUnusedParameters": true,                /* Ошибка при неиспользуемых параметрах */
    "noImplicitReturns": true,                 /* Все кодовые пути должны возвращать */
    "noFallthroughCasesInSwitch": true,        /* Ошибка при fallthrough в switch */
    
    /* Strict mode */
    "strictNullChecks": true,                  /* Строгая проверка null/undefined */
    "strictFunctionTypes": true,               /* Строгая проверка типов функций */
    "strictBindCallApply": true,               /* Строгая проверка bind/call/apply */
    "strictPropertyInitialization": true,      /* Проверка инициализации свойств */
    "noImplicitThis": true,                    /* Явная проверка использования this */
    "useUnknownInCatchVariables": true         /* Catch переменные как unknown */
  }
}
```

## Webpack и TypeScript

### Конфигурация Webpack для TypeScript
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',  // использует tsconfig.json
        exclude: /node_modules/,
      },
      {
        test: /\.js$/,
        use: 'babel-loader',  // можно использовать Babel после TypeScript
        exclude: /node_modules/,
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
    }
  },
  devtool: 'source-map' // sourcemaps для отладки TypeScript
};
```

### Использование fork-ts-checker-webpack-plugin
```javascript
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  // ... остальная конфигурация
  plugins: [
    new ForkTsCheckerWebpackPlugin({
      typescript: {
        configFile: path.resolve(__dirname, 'tsconfig.json')
      }
    })
  ]
};
```

## Vite и TypeScript

### Конфигурация Vite
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components')
    }
  },
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'src/index.html'),
        nested: resolve(__dirname, 'src/nested/index.html')
      }
    }
  },
  esbuild: {
    // Опции esbuild
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  },
  server: {
    // Опции разработки
    port: 3000,
    open: true
  }
});
```

### Vite специфичные особенности
```typescript
// Vite поддерживает ES Modules nativе
// vite-env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

## Rollup для библиотек

### Конфигурация Rollup
```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default [
  // ES Modules версия
  {
    input: 'src/index.ts',
    output: {
      dir: 'dist/esm',
      format: 'es',
      sourcemap: true
    },
    plugins: [
      typescript({ tsconfig: './tsconfig.lib.json' }),
      resolve(),
      commonjs()
    ]
  },
  // CommonJS версия
  {
    input: 'src/index.ts',
    output: {
      dir: 'dist/cjs',
      format: 'cjs',
      sourcemap: true
    },
    plugins: [
      typescript({ tsconfig: './tsconfig.lib.json' }),
      resolve(),
      commonjs()
    ]
  }
];
```

### Библиотечная конфигурация
```json
// tsconfig.lib.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": true,
    "outDir": "./dist/types",
    "stripInternal": true
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx"
  ]
}
```

## Esbuild и SWC

### Esbuild конфигурация
```typescript
// build.ts - скрипт сборки с esbuild
import { build } from 'esbuild';

await build({
  entryPoints: ['./src/index.ts'],
  bundle: true,
  minify: true,
  sourcemap: true,
  format: 'esm',
  target: 'es2020',
  outfile: './dist/bundle.js',
  plugins: [
    // кастомные плагины
  ]
});
```

### SWC конфигурация
```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true
    },
    "transform": {
      "react": {
        "runtime": "automatic"
      }
    },
    "target": "es2020"
  },
  "module": {
    "type": "es6"
  },
  "sourceMaps": true,
  "minify": false
}
```

## Интеграция с Babel

### TypeScript + Babel
```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: { node: 'current' } }],
    ['@babel/preset-typescript', { allowDeclareFields: true }] // TS специфичные опции
  ],
  plugins: [
    // Babel плагины
    ['@babel/plugin-proposal-decorators', { version: '2022-03' }],
    '@babel/plugin-proposal-class-properties'
  ]
};
```

### Преимущества Babel с TypeScript
- Более быстрая транспиляция (без проверки типов)
- Больше плагинов и возможностей
- Совместимость с большей частью JS экосистемы

## Лучшие практики сборки

### 1. Разные конфигурации для разных целей
```json
// tsconfig.app.json - для приложения
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist/app"
  },
  "include": ["src/app/**/*"]
}

// tsconfig.lib.json - для библиотеки
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist/lib",
    "declaration": true,
    "declarationMap": true
  },
  "include": ["src/lib/**/*"]
}

// tsconfig.test.json - для тестов
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "types": ["jest", "node"]
  },
  "include": ["src/**/*", "__tests__/**/*"]
}
```

### 2. Оптимизация производительности
```typescript
// Ускорение компиляции
{
  "compilerOptions": {
    "incremental": true,           // Инкрементная компиляция
    "tsBuildInfoFile": ".tsbuildinfo", // Кэш компиляции
    "noEmit": true,               // Только проверка типов
    "preserveWatchOutput": true,   // Сохранять вывод в watch режиме
    "diagnostics": true           // Включить диагностические данные
  }
}
```

### 3. Tree Shaking и оптимизация
```javascript
// webpack.config.js для оптимизации
module.exports = {
  mode: 'production',
  optimization: {
    usedExports: true,            // Обнаружение неиспользуемых экспортируемых
    sideEffects: false,           // Указывает, что файлы не имеют побочных эффектов
    minimize: true,               // Включить минимизацию
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,  // отдельная упаковка внешних зависимостей
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

## Современные тенденции

### 1. Fast Refresh и HMR
```typescript
// Поддержка горячей перезагрузки
// vite.config.ts
export default defineConfig({
  server: {
    hmr: {
      overlay: true // показывать ошибки в браузере
    }
  },
  plugins: [
    // HMR плагины
  ]
});
```

### 2. Zero-config инструменты
```json
// package.json с zero-config инструментами
{
  "scripts": {
    "dev": "vite",                    // Vite - нулевая конфигурация
    "build": "vite build",            // билд с умными оптимизациями
    "preview": "vite preview"         // предпросмотр билда
  },
  "type": "module"                    // поддержка ES Modules
}
```

### 3. Bundle analysis
```bash
# Анализ бандла
npx webpack-bundle-analyzer dist/stats.json
npx vite-bundle-visualizer
npx rollup-plugin-visualizer
```

## Практические примеры

### Универсальный билд скрипт
```typescript
// build-system.ts - универсальная система сборки
interface BuildConfig {
  input: string;
  output: string;
  format: 'esm' | 'cjs' | 'umd';
  target: string;
  minify: boolean;
  sourcemap: boolean;
  tsconfig?: string;
}

class BuildSystem {
  constructor(private config: BuildConfig) {}
  
  async build(): Promise<void> {
    if (this.isDevMode()) {
      await this.startDevServer();
    } else {
      await this.compileProduction();
    }
  }
  
  private isDevMode(): boolean {
    return process.argv.includes('--dev') || process.env.NODE_ENV === 'development';
  }
  
  private async startDevServer(): Promise<void> {
    // Логика dev сервера
    console.log(`Starting dev server on ${this.config.output}`);
  }
  
  private async compileProduction(): Promise<void> {
    // Логика продакшн сборки
    console.log(`Building for production to ${this.config.output}`);
    
    // Использовать подходящий инструмент в зависимости от настроек
    if (this.config.format === 'esm') {
      // Использовать Rollup для ESM
    } else if (this.config.format === 'umd') {
      // Использовать Webpack для UMD
    }
  }
}
```

### Мониторинг производительности сборки
```typescript
// build-performance.ts
class BuildPerformanceMonitor {
  private timings = new Map<string, number>();
  
  start(label: string): void {
    this.timings.set(label, Date.now());
  }
  
  end(label: string): number {
    const start = this.timings.get(label);
    if (!start) {
      throw new Error(`No timing found for label: ${label}`);
    }
    
    const duration = Date.now() - start;
    console.log(`${label}: ${duration}ms`);
    this.timings.delete(label);
    return duration;
  }
  
  report(): void {
    console.table(Array.from(this.timings.entries()).map(([name, time]) => ({
      name, time
    })));
  }
}

// Использование
const monitor = new BuildPerformanceMonitor();
monitor.start('typescript-compilation');
// ... компиляция
monitor.end('typescript-compilation');
```

## Сравнение инструментов

| Инструмент | Скорость | Гибкость | Простота | Назначение |
|------------|----------|----------|----------|------------|
| tsc | Средняя | Низкая | Высокая | Компиляция TypeScript |
| Babel + TypeScript | Средняя | Высокая | Средняя | Транспиляция + плагины |
| Webpack | Средняя | Очень высокая | Низкая | Приложения + сложные бандлы |
| Rollup | Высокая | Средняя | Средняя | Библиотеки |
| Vite | Очень высокая | Высокая | Высокая | Современные приложения |
| Esbuild | Очень высокая | Средняя | Средняя | Фаст бандлинг |
| SWC | Очень высокая | Средняя | Средняя | Fast транспиляция |

## Связь с другими концепциями
- [[../compilation/compiler-options]] - подробнее о параметрах компиляции
- [[../modules/module-systems]] - системы модулей в инструментах сборки
- [[../deployment/bundling]] - упаковка приложений
- [[../testing/build-integration]] - интеграция сборки с тестированием
- [[../ecosystem/package-management]] - управление зависимостями в сборке