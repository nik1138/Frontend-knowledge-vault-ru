# Инструменты и рабочие процессы TypeScript

Эффективная разработка на TypeScript требует понимания и использования широкого спектра инструментов и установления правильных рабочих процессов. Эти элементы обеспечивают высокое качество кода, производительность разработки и надежность приложений.

## Инструменты компиляции и сборки

### 1. TypeScript Compiler (tsc) и его расширенные возможности

```json
// tsconfig.json - расширенная конфигурация
{
  "compilerOptions": {
    // Основные настройки
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    
    // Система модулей
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@types": ["src/types"],
      "@assets/*": ["src/assets/*"]
    },
    
    // Строгая типизация
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    
    // Проверки безопасности
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    
    // Совместимость
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    
    // Вывод и эмиттинг
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "inlineSources": false,
    "removeComments": false,
    
    // Производительность
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo",
    "composite": true,
    "diagnostics": true,
    "extendedDiagnostics": true,
    
    // Экспериментальные возможности
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.spec.ts",
    "**/*.test.ts",
    "tests"
  ],
  "references": [
    { "path": "./tsconfig.node.json" }
  ]
}
```

### 2. Настройка ts-node для разработки

```json
// package.json
{
  "scripts": {
    "dev": "ts-node --transpile-only --project tsconfig.json src/index.ts",
    "dev:watch": "nodemon --exec ts-node --transpile-only src/index.ts",
    "start": "node dist/index.js",
    "build": "tsc",
    "build:watch": "tsc --watch",
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch"
  },
  "devDependencies": {
    "ts-node": "^10.9.1",
    "nodemon": "^2.0.20",
    "@types/node": "^18.11.9",
    "typescript": "^4.9.3"
  }
}
```

```json
// nodemon.json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts", "src/**/*.test.ts"],
  "exec": "ts-node --transpile-only ./src/index.ts"
}
```

### 3. Использование swc для ускорения компиляции

```json
// package.json
{
  "devDependencies": {
    "@swc/cli": "^0.1.57",
    "@swc/core": "^1.3.24",
    "typescript": "^4.9.3"
  },
  "scripts": {
    "build:swc": "swc src -d dist",
    "dev:swc": "swc src -d dist --watch",
    "type-check": "tsc --noEmit"
  }
}
```

```json
// .swcrc
{
  "jsc": {
    "parser": {
      "syntax": "typescript",
      "tsx": true,
      "decorators": true,
      "dynamicImport": true
    },
    "transform": {
      "decoratorMetadata": true,
      "legacyDecorator": true
    },
    "target": "es2020"
  },
  "module": {
    "type": "es6"
  },
  "sourceMaps": true
}
```

## Системы сборки

### 1. Vite с TypeScript

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { resolve } from 'path';
import { execSync } from 'child_process';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@assets': resolve(__dirname, 'src/assets'),
      '@types': resolve(__dirname, 'src/types'),
    },
  },
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.ts'),
      name: 'MyLibrary',
      fileName: (format) => `my-library.${format}.js`,
    },
    rollupOptions: {
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM',
        },
      },
    },
  },
  esbuild: {
    target: 'es2020',
    define: {
      __APP_VERSION__: JSON.stringify(execSync('git describe --tags').toString().trim()),
    },
  },
  optimizeDeps: {
    include: ['@types/lodash'],
  },
  server: {
    port: 3000,
    open: true,
  },
});
```

### 2. Webpack с TypeScript

```typescript
// webpack.config.ts
import path from 'path';
import { Configuration } from 'webpack';
import { TsconfigPathsPlugin } from 'tsconfig-paths-webpack-plugin';

const config: Configuration = {
  entry: './src/index.ts',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    library: {
      name: 'MyLibrary',
      type: 'umd',
    },
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    plugins: [
      new TsconfigPathsPlugin({
        configFile: path.resolve(__dirname, 'tsconfig.json'),
      }),
    ],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              configFile: path.resolve(__dirname, 'tsconfig.json'),
              transpileOnly: true, // Для быстрой разработки
            },
          },
        ],
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    // Другие плагины
  ],
  devtool: 'source-map',
  stats: 'minimal',
};

export default config;
```

### 3. Rollup с TypeScript

```typescript
// rollup.config.ts
import { defineConfig } from 'rollup';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import typescript from '@rollup/plugin-typescript';
import { terser } from 'rollup-plugin-terser';
import commonjs from '@rollup/plugin-commonjs';

export default defineConfig({
  input: 'src/index.ts',
  output: [
    {
      file: 'dist/bundle.cjs',
      format: 'cjs',
      sourcemap: true,
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'es',
      sourcemap: true,
    },
    {
      file: 'dist/bundle.umd.js',
      format: 'umd',
      name: 'MyLibrary',
      sourcemap: true,
      globals: {
        react: 'React',
      },
    },
  ],
  external: ['react', 'react-dom'],
  plugins: [
    nodeResolve({
      browser: true,
      extensions: ['.js', '.ts', '.tsx'],
    }),
    commonjs(),
    typescript({
      tsconfig: './tsconfig.json',
      sourceMap: true,
      declaration: true,
      outDir: 'dist/types',
    }),
    terser({
      module: true,
      compress: {
        drop_console: true,
      },
      mangle: true,
    }),
  ],
});
```

## Средства статического анализа

### 1. ESLint с TypeScript

```json
// .eslintrc.json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking",
    "plugin:import/recommended",
    "plugin:import/typescript",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module",
    "project": [
      "./tsconfig.json",
      "./tsconfig.node.json",
      "./tsconfig.test.json"
    ]
  },
  "plugins": [
    "@typescript-eslint",
    "import",
    "unused-imports",
    "simple-import-sort",
    "prefer-arrow"
  ],
  "rules": {
    // Основные правила TypeScript
    "@typescript-eslint/consistent-type-definitions": ["error", "interface"],
    "@typescript-eslint/prefer-readonly": "error",
    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        "argsIgnorePattern": "^_",
        "varsIgnorePattern": "^_",
        "caughtErrorsIgnorePattern": "^_"
      }
    ],
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/no-non-null-assertion": "error",
    "@typescript-eslint/prefer-optional-chain": "error",
    "@typescript-eslint/prefer-nullish-coalescing": "error",
    "@typescript-eslint/strict-boolean-expressions": [
      "error",
      {
        "allowString": false,
        "allowNumber": false,
        "allowNullableObject": false
      }
    ],
    
    // Правила импортов
    "import/order": "off",
    "simple-import-sort/imports": "error",
    "simple-import-sort/exports": "error",
    "import/first": "error",
    "import/newline-after-import": "error",
    "import/no-duplicates": "error",
    
    // Правила для неиспользуемых импортов
    "unused-imports/no-unused-imports": "error",
    "unused-imports/no-unused-vars": [
      "error",
      {
        "vars": "all",
        "varsIgnorePattern": "^_",
        "args": "after-used",
        "argsIgnorePattern": "^_"
      }
    ],
    
    // Правила функций
    "prefer-arrow/prefer-arrow-functions": [
      "error",
      {
        "disallowPrototype": true,
        "singleReturnOnly": false,
        "classPropertiesAllowed": false
      }
    ],
    "@typescript-eslint/prefer-function-type": "error",
    
    // Правила безопасности
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/no-misused-promises": "error",
    "@typescript-eslint/prefer-promise-reject-errors": "error",
    "@typescript-eslint/no-unnecessary-type-assertion": "error",
    
    // Правила стиля
    "@typescript-eslint/semi": ["error"],
    "object-curly-spacing": ["error", "always"],
    "comma-dangle": ["error", "always-multiline"],
    "@typescript-eslint/type-annotation-spacing": "error",
    
    // Правила для тестов
    "@typescript-eslint/no-non-null-asserted-optional-chain": "off",
    "@typescript-eslint/restrict-plus-operands": "error"
  },
  "settings": {
    "import/parsers": {
      "@typescript-eslint/parser": [".ts", ".tsx"]
    },
    "import/resolver": {
      "typescript": {
        "alwaysTryTypes": true,
        "project": ["./tsconfig.json"]
      }
    }
  },
  "overrides": [
    {
      "files": ["*.test.ts", "*.spec.ts", "*.test.tsx", "*.spec.tsx"],
      "rules": {
        "@typescript-eslint/no-explicit-any": "off",
        "@typescript-eslint/no-non-null-assertion": "off",
        "@typescript-eslint/unbound-method": "off"
      }
    },
    {
      "files": ["*.js", "*.jsx"],
      "rules": {
        "@typescript-eslint/no-var-requires": "off"
      }
    }
  ]
}
```

### 2. SonarTS (теперь часть SonarJS)

```json
// sonar-project.properties
sonar.projectKey=my-typescript-project
sonar.projectName=My TypeScript Project
sonar.projectVersion=1.0.0
sonar.sources=src
sonar.tests=tests,__tests__
sonar.exclusions=**/*.test.ts,**/*.spec.ts,**/node_modules/**
sonar.typescript.lcov.reportPaths=coverage/lcov.info
sonar.coverage.exclusions=**/*.test.ts,**/*.spec.ts,**/node_modules/**
```

### 3. Type-coverage для отслеживания покрытия типами

```json
// package.json
{
  "scripts": {
    "type-coverage": "type-coverage --detail --at-least 95 --strict",
    "type-coverage:watch": "nodemon --exec \"npm run type-coverage\" --watch src --ext ts,tsx"
  },
  "devDependencies": {
    "type-coverage": "^2.21.0"
  }
}
```

## Инструменты форматирования

### 1. Prettier с TypeScript

```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "overrides": [
    {
      "files": "*.ts",
      "options": {
        "parser": "typescript"
      }
    },
    {
      "files": "*.tsx",
      "options": {
        "parser": "typescript"
      }
    },
    {
      "files": "*.mts",
      "options": {
        "parser": "typescript"
      }
    },
    {
      "files": "*.cts",
      "options": {
        "parser": "typescript"
      }
    }
  ]
}
```

### 2. Integration с ESLint

```json
// .eslintrc.json (дополнение)
{
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended",
    "prettier"
  ],
  "plugins": [
    "@typescript-eslint",
    "prettier"
  ],
  "rules": {
    "prettier/prettier": "error",
    "arrow-body-style": "off",
    "prefer-arrow-callback": "off"
  }
}
```

## Рабочие процессы CI/CD

### 1. GitHub Actions для TypeScript проектов

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Type check
      run: npm run type-check
    
    - name: Lint
      run: npm run lint
    
    - name: Format check
      run: npm run format:check
    
    - name: Run tests
      run: npm run test:ci
    
    - name: Build
      run: npm run build
    
    - name: Upload coverage reports to Codecov
      uses: codecov/codecov-action@v3
      env:
        CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

  security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run security audit
      run: npm audit --audit-level high
    
    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### 2. Advanced workflow с кэшированием

```yaml
# .github/workflows/advanced-ci.yml
name: Advanced CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      cache-key: ${{ steps.cache.outputs.cache-primary-key }}
    steps:
    - name: Checkout
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Cache node_modules
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Install dependencies
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci --frozen-lockfile

  type-check:
    runs-on: ubuntu-latest
    needs: setup
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Restore node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Generate type coverage report
      run: |
        npx type-coverage --detail --at-least 95 --strict
        echo "Type coverage: $(npx type-coverage --output)" >> $GITHUB_STEP_SUMMARY

  lint-and-format:
    runs-on: ubuntu-latest
    needs: setup
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Restore node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Run ESLint
      run: npm run lint
      
    - name: Check formatting
      run: npm run format:check

  test:
    runs-on: ubuntu-latest
    needs: [setup, type-check]
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        
    - name: Restore node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Run unit tests
      run: npm run test:unit -- --coverage
      
    - name: Run integration tests
      run: npm run test:integration
      
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella

  build:
    runs-on: ubuntu-latest
    needs: [setup, type-check, lint-and-format, test]
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        
    - name: Restore node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Build project
      run: npm run build
      
    - name: Archive build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-artifacts
        path: dist/
        retention-days: 30

  deploy-preview:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    needs: build
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v3
      with:
        name: build-artifacts
        path: dist/
        
    - name: Deploy preview
      run: echo "Deploying preview for PR #${{ github.event.number }}"
```

## Инструменты для монорепозиториев

### 1. Lerna с TypeScript

```json
// lerna.json
{
  "packages": [
    "packages/*",
    "examples/*"
  ],
  "version": "independent",
  "npmClient": "npm",
  "useWorkspaces": true,
  "command": {
    "publish": {
      "registry": "https://registry.npmjs.org/"
    },
    "bootstrap": {
      "hoist": true
    }
  }
}
```

```json
// package.json (корневой)
{
  "name": "root",
  "private": true,
  "workspaces": [
    "packages/*",
    "examples/*"
  ],
  "scripts": {
    "bootstrap": "lerna bootstrap",
    "build": "lerna run build --parallel",
    "test": "lerna run test --parallel",
    "lint": "lerna run lint",
    "clean": "lerna clean --yes && rm -rf node_modules",
    "version": "lerna version",
    "publish": "lerna publish"
  },
  "devDependencies": {
    "lerna": "^6.4.1"
  }
}
```

### 2. Nx для монорепозиториев

```json
// nx.json
{
  "extends": "nx/presets/npm.json",
  "$schema": "./node_modules/nx/schemas/nx-schema.json",
  "tasksRunnerOptions": {
    "default": {
      "runner": "nx/tasks-runners/default",
      "options": {
        "cacheableOperations": ["build", "lint", "test", "e2e"]
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "test": {
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"]
    },
    "lint": {
      "inputs": ["default", "{workspaceRoot}/.eslintrc.json"]
    }
  },
  "namedInputs": {
    "default": ["{projectRoot}/**/*", "sharedGlobals"],
    "production": [
      "default",
      "!{projectRoot}/**/?(*.)+(spec|test).[jt]s?(x)?(.snap)",
      "!{projectRoot}/tsconfig.spec.json",
      "!{projectRoot}/jest.config.[jt]s",
      "!{projectRoot}/.eslintrc.json"
    ],
    "sharedGlobals": []
  }
}
```

## Инструменты для документации

### 1. TypeDoc для генерации документации

```json
// typedoc.json
{
  "entryPoints": ["./src/index.ts"],
  "out": "./docs/api",
  "name": "My TypeScript Library",
  "includeVersion": true,
  "readme": "./README.md",
  "excludeNotExported": true,
  "excludePrivate": true,
  "excludeProtected": true,
  "theme": "default",
  "customFooterText": "© 2023 My Company. All rights reserved.",
  "hideGenerator": true,
  "categorizeByGroup": true,
  "sort": ["source-order", "alphabetical"],
  "visibilityFilters": {
    "protected": false,
    "private": false,
    "inherited": true,
    "external": false
  },
  "exclude": [
    "**/node_modules/**",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/tests/**"
  ],
  "tsconfig": "./tsconfig.json"
}
```

### 2. Storybook для компонентов

```typescript
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/core-common';

const config: StorybookConfig = {
  stories: [
    '../src/**/*.stories.@(js|jsx|ts|tsx)',
    '../src/**/*.mdx'
  ],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-docs',
    {
      name: '@storybook/addon-postcss',
      options: {
        cssLoaderOptions: {
          // When you have styled components
          modules: {
            // Enable CSS modules
            localIdentName: '[local]',
          },
        },
      },
    },
  ],
  framework: {
    name: '@storybook/react-webpack5',
    options: {}
  },
  docs: {
    autodocs: true
  },
  staticDirs: ['../public'],
};

export default config;
```

## Инструменты для мониторинга и анализа

### 1. Bundle анализ

```json
// package.json
{
  "scripts": {
    "analyze": "webpack-bundle-analyzer dist/stats.json",
    "analyze:build": "npm run build && npm run analyze"
  },
  "devDependencies": {
    "webpack-bundle-analyzer": "^4.7.0"
  }
}
```

### 2. Performance monitoring

```typescript
// performance-monitoring.ts
export class PerformanceMonitor {
  private marks = new Map<string, number>();
  private measures = new Map<string, { start: string; end: string; duration: number }>();

  mark(name: string): void {
    this.marks.set(name, performance.now());
  }

  measure(name: string, startMark: string, endMark: string): void {
    const startTime = this.marks.get(startMark);
    const endTime = this.marks.get(endMark);

    if (startTime === undefined || endTime === undefined) {
      console.warn(`Marks ${startMark} or ${endMark} not found`);
      return;
    }

    const duration = endTime - startTime;
    this.measures.set(name, { start: startMark, end: endMark, duration });

    console.log(`Performance measure ${name}: ${duration.toFixed(2)}ms`);
  }

  getReport(): string {
    let report = 'Performance Report:\n';
    for (const [name, measure] of this.measures.entries()) {
      report += `${name}: ${measure.duration.toFixed(2)}ms (${measure.start} → ${measure.end})\n`;
    }
    return report;
  }

  clear(): void {
    this.marks.clear();
    this.measures.clear();
  }
}

// Использование
const perf = new PerformanceMonitor();

perf.mark('start-operation');
// Выполнение операции
perf.mark('end-operation');
perf.measure('operation-duration', 'start-operation', 'end-operation');

console.log(perf.getReport());
```

## Практические примеры из реальной разработки

### 1. Комплексная система CI/CD

```yaml
# .github/workflows/enterprise-workflow.yml
name: Enterprise Workflow

on:
  push:
    branches: [main, develop, release/*]
  pull_request:
    branches: [main, develop]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io

jobs:
  validate:
    runs-on: ubuntu-latest
    outputs:
      skip-tests: ${{ steps.skip_check.outputs.should_skip }}
    steps:
    - name: Skip PR checks for documentation changes
      id: skip_check
      run: |
        if git diff --name-only HEAD~1 | grep -qE '\.(md|txt|docx?)$'; then
          echo "should_skip=true" >> $GITHUB_OUTPUT
        else
          echo "should_skip=false" >> $GITHUB_OUTPUT
        fi
      shell: bash

  security-audit:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run security audit
      run: |
        npm audit --audit-level moderate --json > audit-report.json
        if [ -s audit-report.json ]; then
          HIGH_VULNS=$(jq '.metadata.vulnerabilities.high' audit-report.json)
          if [ "$HIGH_VULNS" -gt 0 ]; then
            echo "Found $HIGH_VULNS high severity vulnerabilities"
            exit 1
          fi
        fi

  code-quality:
    runs-on: ubuntu-latest
    needs: validate
    if: needs.validate.outputs.skip-tests != 'true'
    steps:
    - uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Cache node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Install dependencies
      if: steps.node-modules-cache.outputs.cache-hit != 'true'
      run: npm ci
      
    - name: Type check
      run: npm run type-check
      
    - name: Lint code
      run: npm run lint
      
    - name: Check formatting
      run: npm run format:check
      
    - name: Verify type coverage
      run: npx type-coverage --at-least 95 --strict

  unit-tests:
    runs-on: ubuntu-latest
    needs: [validate, code-quality]
    if: needs.validate.outputs.skip-tests != 'true'
    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]
        os: [ubuntu-latest, windows-latest]
      fail-fast: false
    steps:
    - uses: actions/checkout@v3
      
    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        
    - name: Cache node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ matrix.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Install dependencies
      if: steps.node-modules-cache.outputs.cache-hit != 'true'
      run: npm ci
      
    - name: Run unit tests
      run: npm run test:unit -- --coverage --ci --maxWorkers=2
      
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests-${{ matrix.node-version }}-${{ matrix.os }}
        name: codecov-${{ matrix.os }}-${{ matrix.node-version }}

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    steps:
    - uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
        
    - name: Cache node_modules
      uses: actions/cache@v3
      with:
        path: node_modules
        key: node-modules-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
        TEST_DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb

  build-and-deploy:
    runs-on: ubuntu-latest
    needs: [integration-tests, security-audit]
    if: github.ref == 'refs/heads/main'
    environment: production
    permissions:
      contents: read
      packages: write
    steps:
    - uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        registry-url: https://npm.pkg.github.com/
        
    - name: Login to GitHub Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
        
    - name: Build Docker image
      run: |
        docker build -t ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }} .
        docker tag ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }} ${{ env.REGISTRY }}/${{ github.repository }}:latest
        
    - name: Push Docker image
      run: |
        docker push ${{ env.REGISTRY }}/${{ github.repository }}:${{ github.sha }}
        docker push ${{ env.REGISTRY }}/${{ github.repository }}:latest
```

### 2. Система управления версиями

```json
// package.json
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:alpha": "standard-version --prerelease alpha",
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
  },
  "devDependencies": {
    "standard-version": "^9.5.0",
    "conventional-changelog-cli": "^2.2.2"
  }
}
```

```json
// .versionrc.json
{
  "types": [
    {"type": "feat", "section": "Features"},
    {"type": "fix", "section": "Bug Fixes"},
    {"type": "chore", "hidden": true},
    {"type": "docs", "hidden": true},
    {"type": "style", "hidden": true},
    {"type": "refactor", "section": "Refactors"},
    {"type": "perf", "section": "Performance Improvements"},
    {"type": "test", "hidden": true}
  ],
  "commitUrlFormat": "https://github.com/{{owner}}/{{repository}}/commits/{{hash}}",
  "compareUrlFormat": "https://github.com/{{owner}}/{{repository}}/compare/{{previousTag}}...{{currentTag}}"
}
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки в настройке инструментов

```typescript
// ОШИБКА: Неправильная настройка tsconfig для библиотеки
// {
//   "compilerOptions": {
//     "outDir": "dist",
//     "declaration": true,
//     "declarationMap": false,  // Плохо: нет declaration maps
//     "sourceMap": false,       // Плохо: нет sourcemaps
//     "removeComments": true,   // Плохо: удаляет JSDoc комментарии
//   }
// }

// ХОРОШО: Правильная настройка для библиотеки
// {
//   "compilerOptions": {
//     "outDir": "dist",
//     "declaration": true,
//     "declarationMap": true,   // Хорошо: позволяет отладку типов
//     "sourceMap": true,        // Хорошо: позволяет отладку исходного кода
//     "removeComments": false,  // Хорошо: сохраняет JSDoc
//     "stripInternal": true,    // Лучше: удаляет внутренние комментарии
//   }
// }
```

### 2. Лучшие практики настройки

```json
// .nvmrc
18.17.0
```

```json
// .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

## Лучшие практики

1. **Используйте consistent tooling setup** - во всех проектах компании
2. **Автоматизируйте процессы** - через pre-commit хуки и CI/CD
3. **Кэшируйте зависимости** - для ускорения сборок
4. **Используйте incremental builds** - для быстрой разработки
5. **Мониторьте производительность** - сборок и тестов
6. **Регулярно обновляйте инструменты** - для безопасности и новых возможностей
7. **Документируйте workflow** - для новых членов команды
8. **Используйте semantic versioning** - для управления версиями

## Связи с другими концепциями

- [[build-tools/typescript-compiler]] - основы компиляции TypeScript
- [[architecture/monorepo]] - архитектурные аспекты монорепозиториев
- [[testing/automation]] - автоматизация тестирования
- [[deployment/workflows]] - деплоймент и CI/CD процессы
- [[code-review/tools]] - инструменты для код-ревью