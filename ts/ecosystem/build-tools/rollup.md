# Rollup in TypeScript Ecosystem

Rollup - это модульный упаковщик, оптимизированный для создания библиотек, особенно тех, которые будут использовать ES модули. Rollup особенно хорош для tree-shaking и создания компактных бандлов.

## Основы использования Rollup

### Установка и настройка
```bash
npm install --save-dev rollup @rollup/plugin-typescript rollup-plugin-terser
```

### Базовая конфигурация
```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import { terser } from 'rollup-plugin-terser';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default {
  input: 'src/index.ts',  // точка входа
  output: [
    {
      // ES Modules версия
      file: 'dist/library.esm.js',
      format: 'es',
      sourcemap: true
    },
    {
      // CommonJS версия
      file: 'dist/library.cjs.js',
      format: 'cjs',
      sourcemap: true
    },
    {
      // IIFE версия (для браузеров)
      file: 'dist/library.iife.js',
      format: 'iife',
      name: 'Library',  // глобальное имя в браузере
      sourcemap: true
    },
    {
      // UMD версия (универсальная)
      file: 'dist/library.umd.js',
      format: 'umd',
      name: 'Library',
      globals: {
        // внешние зависимости (не включать в бандл)
        'vue': 'Vue',
        'react': 'React'
      },
      sourcemap: true
    }
  ],
  plugins: [
    typescript({ tsconfig: './tsconfig.rollup.json' }),
    resolve({
      browser: true, // разрешение для браузера
    }),
    commonjs(), // преобразование CommonJS в ES Modules
    terser() // минификация для production
  ],
  external: [  // внешние зависимости (не включать в бандл)
    'vue',
    'react',
    'react-dom'
  ]
};
```

### tsconfig.rollup.json
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "module": "ESNext",           // Rollup лучше работает с ESNext
    "target": "ES2020",           // поддержка необходимых возможностей
    "declaration": true,          // генерировать .d.ts файлы
    "declarationDir": "./dist/types",  // директория для declaration файлов
    "emitDeclarationOnly": true,  // только declaration файлы (не JS)
    "noEmit": false,             // генерировать declaration файлы
    "outDir": "./temp"           // временный вывод для транспиляции
  },
  "include": [
    "src/**/*"
  ]
}
```

## Продвинутая конфигурация

### Многофайловой вывод
```javascript
// rollup.config.js - продвинутая конфигурация
import typescript from '@rollup/plugin-typescript';
import { glob } from 'glob';

// Автоматическое определение точек входа
const inputEntries = {};
const packages = glob.sync('src/packages/*/index.ts');

packages.forEach(pkgPath => {
  const pkgName = pkgPath.match(/src\/packages\/(.+)\/index\.ts/)?.[1];
  if (pkgName) {
    inputEntries[pkgName] = pkgPath;
  }
});

// Также включаем основной index
inputEntries.index = 'src/index.ts';

export default {
  input: inputEntries,
  output: [
    {
      dir: 'dist/esm',
      format: 'esm',
      preserveModules: true,  // сохранить структуру модулей
      preserveModulesRoot: 'src'
    },
    {
      dir: 'dist/cjs',
      format: 'cjs',
      preserveModules: true,
      preserveModulesRoot: 'src'
    }
  ],
  plugins: [
    typescript({ 
      tsconfig: './tsconfig.library.json',
      declarationDir: './dist/types'
    })
  ],
  external: [
    /^node:.*/,  // все node внутренние модули
    ...Object.keys(require('./package.json').dependencies || {})
  ]
};
```

### Tree-shaking оптимизации
```javascript
// rollup.config.js - оптимизации tree-shaking
export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'es',
    // опции для tree-shaking
    compact: false,  // компактный вывод (для минификации)
    sourcemap: true
  },
  plugins: [
    typescript(),
    {
      name: 'treeshake-hints',
      resolveId(id) {
        // помощь Rollup в tree-shaking
        if (id.endsWith('.development.ts')) {
          return { id, moduleSideEffects: false };
        }
      }
    }
  ],
  treeshake: {
    moduleSideEffects: false,  // отключить побочные эффекты модулей
    propertyReadSideEffects: false,
    tryCatchDeoptimization: false
  }
};
```

### Условная компиляция
```typescript
// conditional.ts - условная логика для разных целей
export const isDevelopment = process.env.NODE_ENV === 'development';
export const isBrowser = typeof window !== 'undefined';

// Использование
if (isDevelopment) {
  console.log('Debug mode');
}
```

```javascript
// rollup.config.js - условная компиляция
import replace from '@rollup/plugin-replace';

export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/bundle.js',
    format: 'es'
  },
  plugins: [
    replace({
      preventAssignment: true,
      'process.env.NODE_ENV': JSON.stringify('production'),
      'typeof window': '"object"'  // для браузерных целей
    }),
    typescript()
  ]
};
```

## Создание библиотеки с Rollup

### Пример библиотеки
```typescript
// src/index.ts
export { default as Component } from './component';
export { service as apiService } from './services/api';
export * from './utils/helpers';  // переэкспорт утилит
export type { Config } from './types/config';  // переэкспорт типов
```

```typescript
// src/component.ts
export interface ComponentOptions {
  element: HTMLElement;
  data: Record<string, any>;
}

export default class Component {
  constructor(private options: ComponentOptions) {}
  
  render(): void {
    this.options.element.innerHTML = JSON.stringify(this.options.data);
  }
  
  destroy(): void {
    this.options.element.innerHTML = '';
  }
}
```

### Конфигурация для библиотеки
```javascript
// rollup.config.js для библиотеки
import typescript from '@rollup/plugin-typescript';
import { terser } from 'rollup-plugin-terser';
import dts from 'rollup-plugin-dts';  // плагин для .d.ts бандла

// Основная конфигурация
const jsConfig = {
  input: 'src/index.ts',
  output: [
    { file: 'dist/index.es.js', format: 'es' },
    { file: 'dist/index.cjs.js', format: 'cjs' },
    { 
      file: 'dist/index.umd.js', 
      format: 'umd', 
      name: 'MyLibrary',
      globals: {
        'external-dep': 'ExternalDep'
      }
    }
  ],
  plugins: [
    typescript({ 
      tsconfig: './tsconfig.build.json',
    }),
    terser({
      module: true,
      compress: {
        hoist_vars: true,
        module: true
      },
      mangle: {
        properties: {
          regex: /^__/  // манглировать только свойства, начинающиеся с __
        }
      },
      output: {
        comments: false
      }
    })
  ],
  external: [
    'external-dep'  // не включать в бандл
  ]
};

// Конфигурация для declaration файлов
const dtsConfig = {
  input: 'src/index.ts',
  output: [{ file: 'dist/index.d.ts', format: 'es' }],
  plugins: [dts()]
};

export default [jsConfig, dtsConfig];
```

### tsconfig.build.json для библиотеки
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": false,  // не нужен для библиотек
    "outDir": "./dist/temp",  // временный вывод
    "rootDir": "./src",
    
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    
    "importHelpers": true,  // использовать tslib helpers
    "downlevelIteration": true,
    "stripInternal": true,   // удалять @internal комментарии
    "removeComments": true   // не включать комментарии в бандл
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts",
    "**/*.test.ts"
  ]
}
```

## Продвинутые паттерны

### Многоформатный вывод
```javascript
// rollup.config.js - создание нескольких форматов с одной конфигурации
import typescript from '@rollup/plugin-typescript';
import json from '@rollup/plugin-json';
import resolve from '@rollup/plugin-node-resolve';

const packageJson = require('./package.json');

const baseConfig = {
  input: 'src/index.ts',
  plugins: [
    json(),
    resolve({
      preferBuiltins: true
    }),
    typescript({
      tsconfig: './tsconfig.json',
      declaration: true,
      declarationDir: './dist/types'
    })
  ],
  external: [
    ...Object.keys(packageJson.dependencies || {}),
    ...Object.keys(packageJson.peerDependencies || {})
  ]
};

// Конфигурации для разных форматов
const configs = [
  // ES Modules
  {
    ...baseConfig,
    output: {
      file: packageJson.module,  // package.json: "module": "./dist/index.esm.js"
      format: 'es',
      sourcemap: true
    }
  },
  // CommonJS
  {
    ...baseConfig,
    output: {
      file: packageJson.main,    // package.json: "main": "./dist/index.cjs.js"  
      format: 'cjs',
      sourcemap: true
    }
  },
  // UMD
  {
    ...baseConfig,
    output: {
      file: 'dist/umd/index.js',
      format: 'umd',
      name: 'MyLibrary',
      globals: {
        'external-dep': 'ExternalDep'
      },
      sourcemap: true
    }
  }
];

export default configs;
```

### Работа с разными целями
```javascript
// rollup.multi-target.config.js
import typescript from '@rollup/plugin-typescript';

const targets = [
  { target: 'es2020', format: 'es', file: 'dist/modern.es.js' },
  { target: 'es2017', format: 'es', file: 'dist/legacy.es.js' },
  { target: 'es5', format: 'cjs', file: 'dist/node.cjs.js' }
];

export default targets.map(target => ({
  input: 'src/index.ts',
  output: {
    file: target.file,
    format: target.format,
    sourcemap: true
  },
  plugins: [
    typescript({
      target: target.target,
      tsconfig: './tsconfig.json'
    })
  ],
  external: Object.keys(require('./package.json').dependencies)
}));
```

## Практические примеры

### Утилиты для веб-компонентов
```typescript
// src/web-components.ts
export interface WebComponentOptions {
  template: string;
  styles?: string;
  observedAttributes?: string[];
}

export class BaseWebComponent extends HTMLElement {
  static options: WebComponentOptions;
  
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }
  
  connectedCallback() {
    if (this.shadowRoot) {
      this.shadowRoot.innerHTML = this.getTemplate();
    }
  }
  
  protected getTemplate(): string {
    return (this.constructor as typeof BaseWebComponent).options.template;
  }
}

// Регистрация компонента
export function registerWebComponent<T extends typeof BaseWebComponent>(
  className: T,
  tagName: string
): void {
  customElements.define(tagName, className);
}
```

### Rollup конфигурация для веб-компонентов
```javascript
// rollup.web-components.config.js
import typescript from '@rollup/plugin-typescript';
import { terser } from 'rollup-plugin-terser';

export default {
  input: 'src/web-components.ts',
  output: {
    file: 'dist/web-components.min.js',
    format: 'iife',  // Immediately Invoked Function Expression для браузеров
    name: 'WebComponentsLibrary',
    sourcemap: true
  },
  plugins: [
    typescript({
      tsconfig: './tsconfig.web.json'
    }),
    terser({
      module: false,  // не использовать как ES Module
      compress: {
        drop_console: true  // удалять console.log в production
      }
    })
  ]
};
```

### TypeScript declaration files
```javascript
// rollup.dts.config.js - только для .d.ts файлов
import dts from 'rollup-plugin-dts';

export default {
  input: 'src/index.ts',
  output: [
    { file: 'dist/index.d.ts', format: 'es' },
    { file: 'dist/index.d.cts', format: 'cjs' }  // для CommonJS consumers
  ],
  plugins: [
    dts({
      compilerOptions: {
        // специфичные опции для declaration файлов
        declaration: true,
        noEmit: false,
        emitDeclarationOnly: true
      }
    })
  ]
};
```

## Лучшие практики

### 1. Оптимизация для tree-shaking
```javascript
// rollup.optimized.config.js
export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'es'
  },
  plugins: [
    typescript(),
    {
      name: 'optimize-treeshaking',
      transform(code, id) {
        // Указание чистых функций (без побочных эффектов)
        if (id.includes('pure-functions')) {
          return {
            code: code + '\n/*#__PURE__*/',
            map: null
          };
        }
      }
    }
  ],
  treeshake: {
    preset: 'smallest',
    moduleSideEffects: false
  }
};
```

### 2. Управление зависимостями
```javascript
// external-dependencies.js - утилита для определения внешних зависимостей
const fs = require('fs');
const path = require('path');

function getExternals() {
  const pkg = JSON.parse(fs.readFileSync(path.join(__dirname, 'package.json'), 'utf-8'));
  
  return [
    ...Object.keys(pkg.dependencies || {}),
    ...Object.keys(pkg.peerDependencies || {}),
    'node:fs',      // Node.js builtin модули
    'node:path',
    'node:util'
  ];
}

export default {
  input: 'src/index.ts',
  external: getExternals(),
  output: {
    file: 'dist/index.js',
    format: 'es'
  },
  plugins: [typescript()]
};
```

### 3. Многофайловой вывод с сохранением структуры
```javascript
// rollup.preserve-modules.config.js
export default {
  input: [
    'src/index.ts',
    'src/utils/index.ts',
    'src/services/index.ts',
    // автоматически определить все точки входа
  ],
  output: {
    dir: 'dist',
    format: 'es',
    preserveModules: true,        // сохранить структуру файлов
    preserveModulesRoot: 'src',   // базовый путь для сохранения структуры
    entryFileNames: '[name].js',  // имена файлов точек входа
    chunkFileNames: '[name].js'   // имена чанков
  },
  plugins: [typescript()]
};
```

## Совместимость с разными средами

### Node.js специфичные настройки
```javascript
// rollup.node.config.js
import typescript from '@rollup/plugin-typescript';
import nodeResolve from '@rollup/plugin-node-resolve';

export default {
  input: 'src/node-index.ts',
  output: {
    file: 'dist/node.js',
    format: 'cjs',
    exports: 'auto'  // автоматически определять экспорт
  },
  plugins: [
    nodeResolve({
      preferBuiltins: true  // использовать встроенные Node.js модули
    }),
    typescript({
      target: 'ES2018',     // для Node.js 10+
      module: 'CommonJS'    // Node.js использует CommonJS
    })
  ],
  external: [
    'fs', 'path', 'crypto', 'http', 'https',  // Node.js builtin модули
    ...Object.keys(require('./package.json').dependencies)
  ]
};
```

### Браузерные специфичные настройки
```javascript
// rollup.browser.config.js
import typescript from '@rollup/plugin-typescript';
import nodeResolve from '@rollup/plugin-node-resolve';
import replace from '@rollup/plugin-replace';

export default {
  input: 'src/browser-index.ts',
  output: {
    file: 'dist/browser.js',
    format: 'iife',
    name: 'BrowserLibrary'
  },
  plugins: [
    replace({
      preventAssignment: true,
      'process.env.NODE_ENV': JSON.stringify('production'),
      'typeof window': '"object"'
    }),
    nodeResolve({
      browser: true      // разрешение для браузера
    }),
    typescript({
      target: 'ES2017',  // широкая поддержка браузеров
      lib: ['ES2017', 'DOM']  // включить DOM API
    })
  ],
  external: []  // в браузере не использовать external
};
```

## Ошибки и решения

### Частые проблемы
```javascript
// 1. Проблема с external зависимостями
// Если external не указаны правильно
// rollup.config.js
export default {
  external: [
    'external-dep'
    // 'external-dep/sub-module'  // также нужно указать подмодули
  ]
};

// 2. Проблема с declaration файлами
// Обычно требует отдельной сборки
// npm run build:js && npm run build:dts

// 3. Проблема с циклическими зависимостями
// Использовать плагин для обработки циклов
import { nodeResolve } from '@rollup/plugin-node-resolve';

export default {
  plugins: [
    nodeResolve({
      dedupe: ['library-used-in-multiple-places']  // дедуплицировать циклические зависимости
    })
  ]
};
```

## Интеграция с инструментами

### Совместимость с TypeScript Composite Projects
```javascript
// rollup.composite.config.js
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/index.js',
    format: 'es'
  },
  plugins: [
    typescript({
      tsconfig: './tsconfig.json',
      composite: true,        // для composite проектов
      declarationMap: true,   // sourcemaps для declaration файлов
      incremental: true       // инкрементная компиляция
    })
  ]
};
```

### Совместимость с тестированием
```javascript
// rollup.test.config.js - для тестов
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/test-bundle.js',
    format: 'cjs'  // для тест runners, использующих CommonJS
  },
  plugins: [
    typescript({
      tsconfig: './tsconfig.test.json',  // специфичная конфигурация для тестов
      compilerOptions: {
        types: ['jest', 'node']  // типы для тестирования
      }
    })
  ],
  external: [  // зависимости тестирования
    'jest',
    '@jest/globals',
    'sinon'
  ]
};
```

## Заключение

Rollup особенно хорошо подходит для:

1. **Создания библиотек** - с отличной поддержкой tree-shaking
2. **ES Modules** - лучшая оптимизация для современных целей
3. **Tree-shaking** - более эффективное удаление мертвого кода
4. **Создания ядра библиотеки** - до финальной сборки под разные цели

Rollup работает особенно хорошо с TypeScript для создания библиотек, где важна оптимизация размера и дерево сбрасывания (tree-shaking).

## Связь с другими концепциями
- [[typescript-compiler]] - интеграция с TypeScript компиляцией
- [[es-modules]] - ES Modules как основа для оптимальной работы Rollup
- [[bundling-optimization]] - оптимизации для уменьшения размера бандла
- [[library-development]] - создание типобезопасных библиотек
- [[build-tools/webpack]] - сравнение с другими системами сборки