# Инструментарий и сборка в JavaScript

## Введение

Современная разработка на JavaScript требует использования множества инструментов для повышения продуктивности, обеспечения качества кода и автоматизации рутинных задач. Инструментарий включает в себя транспайлеры, бандлеры, линтеры, тестовые фреймворки и системы сборки, которые помогают создавать сложные веб-приложения.

## Основные категории инструментов

### 1. Транспайлеры

Транспайлеры преобразуют код из одной версии JavaScript в другую, обеспечивая совместимость с различными браузерами.

#### Babel

Babel - самый популярный транспайлер в экосистеме JavaScript, преобразующий современный код в совместимый с более старыми браузерами.

```javascript
// .babelrc - конфигурация Babel
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": ["> 1%", "last 2 versions"]
        },
        "useBuiltIns": "usage",
        "corejs": 3
      }
    ],
    "@babel/preset-react"
  ],
  "plugins": [
    "@babel/plugin-proposal-class-properties",
    "@babel/plugin-proposal-optional-chaining"
  ]
}

// package.json - зависимости для Babel
{
  "devDependencies": {
    "@babel/core": "^7.22.0",
    "@babel/cli": "^7.22.0",
    "@babel/preset-env": "^7.22.0",
    "@babel/preset-react": "^7.22.0",
    "@babel/plugin-proposal-class-properties": "^7.18.0",
    "@babel/plugin-proposal-optional-chaining": "^7.21.0"
  },
  "scripts": {
    "build": "babel src --out-dir lib",
    "watch": "babel src --out-dir lib --watch"
  }
}

// Пример кода до транспиляции (ES2022+)
class User {
  // Приватные поля
  #name;
  #email;
  
  constructor(name, email) {
    this.#name = name;
    this.#email = email;
  }
  
  // Геттеры
  get name() {
    return this.#name;
  }
  
  // Опциональная цепочка
  getDomain() {
    return this.#email?.split('@')?.[1] ?? 'unknown';
  }
  
  // Асинхронные методы
  async fetchProfile() {
    try {
      const response = await fetch(`/api/users/${this.#name}`);
      return await response.json();
    } catch (error) {
      console.error('Failed to fetch profile:', error);
      return null;
    }
  }
}

// Пример кода после транспиляции (ES5)
"use strict";

function _classCallCheck(instance, Constructor) { 
  if (!(instance instanceof Constructor)) { 
    throw new TypeError("Cannot call a class as a function"); 
  } 
}

function _defineProperties(target, props) { 
  for (var i = 0; i < props.length; i++) { 
    var descriptor = props[i]; 
    descriptor.enumerable = descriptor.enumerable || false; 
    descriptor.configurable = true; 
    if ("value" in descriptor) descriptor.writable = true; 
    Object.defineProperty(target, descriptor.key, descriptor); 
  } 
}

function _createClass(Constructor, protoProps, staticProps) { 
  if (protoProps) _defineProperties(Constructor.prototype, protoProps); 
  if (staticProps) _defineProperties(Constructor, staticProps); 
  return Constructor; 
}

var User = /*#__PURE__*/function () {
  function User(name, email) {
    _classCallCheck(this, User);
    
    // Приватные поля преобразуются в WeakMap
    _name.set(this, {
      writable: true,
      value: void 0
    });
    _email.set(this, {
      writable: true,
      value: void 0
    });
    
    _classPrivateFieldSet(this, _name, name);
    _classPrivateFieldSet(this, _email, email);
  }
  
  _createClass(User, [{
    key: "name",
    get: function get() {
      return _classPrivateFieldGet(this, _name);
    }
  }, {
    key: "getDomain",
    value: function getDomain() {
      var _ref, _ref2;
      
      return (_ref = (_ref2 = _classPrivateFieldGet(this, _email)) === null || _ref2 === void 0 ? void 0 : _ref2.split('@')) === null || _ref === void 0 ? void 0 : _ref[1] !== null && _ref[1] !== void 0 ? _ref[1] : 'unknown';
    }
  }, {
    key: "fetchProfile",
    value: function fetchProfile() {
      return _asyncToGenerator( /*#__PURE__*/_regeneratorRuntime().mark(function _callee() {
        var response;
        return _regeneratorRuntime().wrap(function _callee$(_context) {
          // ... транспилированный async/await код
        }, _callee, this);
      }))();
    }
  }]);
  
  return User;
}();

var _name = new WeakMap();
var _email = new WeakMap();
```

#### TypeScript

TypeScript - надмножество JavaScript, добавляющее статическую типизацию.

```typescript
// tsconfig.json - конфигурация TypeScript
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}

// Пример кода на TypeScript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

type UserRole = 'admin' | 'user' | 'guest';

class UserService {
  private users: Map<number, User> = new Map();
  
  // Типизированные параметры и возвращаемые значения
  addUser(user: User): User {
    this.users.set(user.id, user);
    return user;
  }
  
  getUser(id: number): User | undefined {
    return this.users.get(id);
  }
  
  // Generic функция
  filterUsers<T extends keyof User>(key: T, value: User[T]): User[] {
    return Array.from(this.users.values()).filter(user => user[key] === value);
  }
  
  // Асинхронный метод с типизированным возвращаемым значением
  async fetchUsers(): Promise<User[]> {
    try {
      const response = await fetch('/api/users');
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const users: User[] = await response.json();
      return users;
    } catch (error) {
      console.error('Failed to fetch users:', error);
      return [];
    }
  }
}

// Утилитарные типы
type PartialUser = Partial<User>; // Все поля опциональны
type RequiredUser = Required<User>; // Все поля обязательны
type UserKeys = keyof User; // Типы ключей объекта User
type UserWithoutId = Omit<User, 'id'>; // User без поля id

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

// Mapped types
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};
```

### 2. Бандлеры

Бандлеры объединяют модули и зависимости в один или несколько файлов для оптимизации загрузки в браузере.

#### Webpack

Webpack - один из самых популярных бандлеров в экосистеме JavaScript.

```javascript
// webpack.config.js - конфигурация Webpack
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  mode: 'production', // или 'development'
  entry: {
    main: './src/index.js',
    vendor: './src/vendor.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader'
        ]
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource'
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource'
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true
      }
    }),
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css'
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  },
  devServer: {
    static: './dist',
    hot: true,
    open: true,
    port: 3000
  }
};

// Пример структуры проекта
/*
src/
├── index.js          // Точка входа
├── index.html        // HTML шаблон
├── components/
│   ├── Header.js
│   ├── Footer.js
│   └── App.js
├── styles/
│   ├── main.css
│   └── components.css
└── utils/
    ├── api.js
    └── helpers.js
*/

// src/index.js - пример точки входа
import './styles/main.css';
import { App } from './components/App';
import { initializeApp } from './utils/helpers';

// Асинхронная загрузка модулей
async function loadApp() {
  const { Header } = await import('./components/Header');
  const { Footer } = await import('./components/Footer');
  
  document.body.appendChild(Header());
  document.body.appendChild(App());
  document.body.appendChild(Footer());
}

// Инициализация приложения
initializeApp().then(() => {
  loadApp().catch(error => {
    console.error('Failed to load app:', error);
  });
});

// Webpack magic comments для управления загрузкой
const loadModule = async () => {
  // Предзагрузка модуля
  const module = await import(
    /* webpackPreload: true */ 
    './heavy-module'
  );
  
  // Ленивая загрузка с именем чанка
  const lazyModule = await import(
    /* webpackChunkName: "analytics" */ 
    './analytics'
  );
  
  return { module, lazyModule };
};
```

#### Vite

Vite - современный бандлер, использующий нативные ES-модули для быстрой разработки.

```javascript
// vite.config.js - конфигурация Vite
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils')
    }
  },
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'axios']
        }
      }
    }
  },
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  css: {
    modules: {
      localsConvention: 'camelCase'
    },
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
});

// package.json - зависимости для Vite
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^4.3.0",
    "@vitejs/plugin-react": "^4.0.0",
    "sass": "^1.62.0"
  }
}

// Пример использования HMR (Hot Module Replacement)
// src/main.js
import { createApp } from './app';

let app = createApp();

if (import.meta.hot) {
  import.meta.hot.accept('./app', (newModule) => {
    // Удаление старого приложения
    app.unmount();
    
    // Создание нового приложения
    app = newModule.createApp();
    app.mount('#app');
  });
  
  import.meta.hot.dispose(() => {
    app.unmount();
  });
}
```

#### Rollup

Rollup - бандлер, оптимизированный для создания библиотек.

```javascript
// rollup.config.js - конфигурация Rollup
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import { terser } from 'rollup-plugin-terser';
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: [
    {
      file: 'dist/bundle.cjs.js',
      format: 'cjs',
      sourcemap: true
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'esm',
      sourcemap: true
    },
    {
      file: 'dist/bundle.umd.js',
      format: 'umd',
      name: 'MyLibrary',
      sourcemap: true,
      plugins: [terser()]
    }
  ],
  plugins: [
    resolve({
      browser: true
    }),
    commonjs(),
    typescript({
      tsconfig: './tsconfig.json'
    }),
    babel({
      babelHelpers: 'bundled',
      exclude: 'node_modules/**'
    })
  ],
  external: [
    'react',
    'react-dom'
  ]
};

// package.json - конфигурация для библиотеки
{
  "name": "my-awesome-library",
  "version": "1.0.0",
  "main": "dist/bundle.cjs.js",
  "module": "dist/bundle.esm.js",
  "browser": "dist/bundle.umd.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "rollup -c",
    "dev": "rollup -c -w"
  }
}
```

### 3. Линтеры и форматтеры

Линтеры и форматтеры обеспечивают единообразие кода и предотвращают ошибки.

#### ESLint

ESLint - самый популярный линтер для JavaScript.

```javascript
// .eslintrc.js - конфигурация ESLint
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:import/errors',
    'plugin:import/warnings'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 12,
    sourceType: 'module'
  },
  plugins: [
    'react',
    '@typescript-eslint',
    'import'
  ],
  rules: {
    'indent': ['error', 2],
    'linebreak-style': ['error', 'unix'],
    'quotes': ['error', 'single'],
    'semi': ['error', 'always'],
    'no-unused-vars': 'warn',
    'no-console': 'warn',
    'react/prop-types': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off'
  },
  settings: {
    react: {
      version: 'detect'
    },
    'import/resolver': {
      node: {
        paths: ['src'],
        extensions: ['.js', '.jsx', '.ts', '.tsx']
      }
    }
  }
};

// .eslintignore - файлы для игнорирования
/*
node_modules/
dist/
build/
*.min.js
coverage/
*/

// package.json - скрипты для ESLint
{
  "scripts": {
    "lint": "eslint src --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint src --ext .js,.jsx,.ts,.tsx --fix",
    "lint:check": "eslint src --ext .js,.jsx,.ts,.tsx --max-warnings 0"
  },
  "devDependencies": {
    "eslint": "^8.40.0",
    "@typescript-eslint/eslint-plugin": "^5.59.0",
    "@typescript-eslint/parser": "^5.59.0",
    "eslint-plugin-react": "^7.32.0",
    "eslint-plugin-import": "^2.27.0"
  }
}

// Пример кода с ошибками ESLint
function badFunction() {
  var unusedVar = 'unused'; // no-unused-vars
  console.log('Debug info'); // no-console
  return undefined;
}

// Исправленный код
function goodFunction() {
  // Удалены неиспользуемые переменные
  // Удалены console.log для production
  return null;
}

// Конфигурация для разных окружений
// .eslintrc.dev.js - для разработки
module.exports = {
  extends: ['./.eslintrc.js'],
  rules: {
    'no-console': 'off', // Разрешаем console в разработке
    'no-debugger': 'off' // Разрешаем debugger в разработке
  }
};

// .eslintrc.prod.js - для production
module.exports = {
  extends: ['./.eslintrc.js'],
  rules: {
    'no-console': 'error', // Запрещаем console в production
    'no-debugger': 'error' // Запрещаем debugger в production
  }
};
```

#### Prettier

Prettier - автоматический форматтер кода.

```javascript
// .prettierrc - конфигурация Prettier
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "lf"
}

// .prettierignore - файлы для игнорирования
/*
node_modules/
dist/
build/
*.min.js
package-lock.json
yarn.lock
*/

// package.json - скрипты для Prettier
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  },
  "devDependencies": {
    "prettier": "^2.8.0"
  }
}

// Интеграция ESLint и Prettier
// .eslintrc.js
module.exports = {
  extends: [
    'eslint:recommended',
    'prettier' // Должен быть последним
  ],
  plugins: [
    'prettier'
  ],
  rules: {
    'prettier/prettier': 'error'
  }
};

// Пример до форматирования
const   badFormatting    =     {
  a:1,b:2,
  c:   3
}    ;

function messyFunction(  param1   ,param2
){
  if(param1>param2
  ){
    return param1+
    param2;
  }
}

// Пример после форматирования
const goodFormatting = {
  a: 1,
  b: 2,
  c: 3,
};

function cleanFunction(param1, param2) {
  if (param1 > param2) {
    return param1 + param2;
  }
}
```

### 4. Тестовые фреймворки

Тестовые фреймворки обеспечивают автоматизированное тестирование кода.

#### Jest

Jest - популярный тестовый фреймворк от Facebook.

```javascript
// jest.config.js - конфигурация Jest
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}'
  ]
};

// package.json - зависимости для Jest
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "jest-environment-jsdom": "^29.5.0",
    "@testing-library/jest-dom": "^5.16.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/user-event": "^14.4.0"
  }
}

// Пример unit теста
// src/utils/calculator.test.js
import { add, subtract, multiply, divide } from './calculator';

describe('Calculator functions', () => {
  describe('add', () => {
    test('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });
    
    test('should add negative numbers', () => {
      expect(add(-2, -3)).toBe(-5);
    });
    
    test('should handle zero', () => {
      expect(add(5, 0)).toBe(5);
    });
  });
  
  describe('subtract', () => {
    test('should subtract two numbers', () => {
      expect(subtract(5, 3)).toBe(2);
    });
    
    test('should handle negative results', () => {
      expect(subtract(3, 5)).toBe(-2);
    });
  });
  
  describe('divide', () => {
    test('should divide two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });
    
    test('should throw error when dividing by zero', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
    });
  });
});

// src/utils/calculator.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export function multiply(a, b) {
  return a * b;
}

export function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

// Пример теста компонента React
// src/components/Button.test.js
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import '@testing-library/jest-dom';
import { Button } from './Button';

describe('Button component', () => {
  test('renders button with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  test('calls onClick handler when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = jest.fn();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByText('Click me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByText('Click me')).toBeDisabled();
  });
  
  test('matches snapshot', () => {
    const { asFragment } = render(<Button>Click me</Button>);
    expect(asFragment()).toMatchSnapshot();
  });
});

// Mocking примеры
// __mocks__/axios.js
export default {
  get: jest.fn(() => Promise.resolve({ data: {} })),
  post: jest.fn(() => Promise.resolve({ data: {} }))
};

// src/api/userApi.test.js
import axios from 'axios';
import { getUser, createUser } from './userApi';

jest.mock('axios');

describe('User API', () => {
  beforeEach(() => {
    axios.get.mockClear();
    axios.post.mockClear();
  });
  
  test('getUser should call axios.get with correct URL', async () => {
    axios.get.mockResolvedValue({ data: { id: 1, name: 'John' } });
    
    const user = await getUser(1);
    
    expect(axios.get).toHaveBeenCalledWith('/api/users/1');
    expect(user).toEqual({ id: 1, name: 'John' });
  });
  
  test('createUser should call axios.post with correct data', async () => {
    const userData = { name: 'John', email: 'john@example.com' };
    axios.post.mockResolvedValue({ data: { id: 1, ...userData } });
    
    const user = await createUser(userData);
    
    expect(axios.post).toHaveBeenCalledWith('/api/users', userData);
    expect(user).toEqual({ id: 1, ...userData });
  });
});
```

#### Cypress

Cypress - инструмент для end-to-end тестирования.

```javascript
// cypress.config.js - конфигурация Cypress
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.js',
    fixturesFolder: 'cypress/fixtures',
    screenshotsFolder: 'cypress/screenshots',
    videosFolder: 'cypress/videos',
    video: true,
    screenshotOnRunFailure: true,
    setupNodeEvents(on, config) {
      // Имплементация node event listeners
    }
  },
  env: {
    apiUrl: 'http://localhost:8000/api'
  }
});

// cypress/support/e2e.js
import './commands';

// Custom commands
Cypress.Commands.add('login', (email, password) => {
  cy.visit('/login');
  cy.get('[data-cy=email]').type(email);
  cy.get('[data-cy=password]').type(password);
  cy.get('[data-cy=submit]').click();
});

Cypress.Commands.add('resetDatabase', () => {
  cy.request('POST', '/api/test/reset');
});

// cypress/e2e/auth.cy.js
describe('Authentication', () => {
  beforeEach(() => {
    cy.resetDatabase();
  });
  
  it('should allow user to login with valid credentials', () => {
    cy.visit('/login');
    
    cy.get('[data-cy=email]').type('user@example.com');
    cy.get('[data-cy=password]').type('password123');
    cy.get('[data-cy=submit]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('[data-cy=user-menu]').should('be.visible');
  });
  
  it('should show error for invalid credentials', () => {
    cy.visit('/login');
    
    cy.get('[data-cy=email]').type('invalid@example.com');
    cy.get('[data-cy=password]').type('wrongpassword');
    cy.get('[data-cy=submit]').click();
    
    cy.get('[data-cy=error-message]')
      .should('be.visible')
      .and('contain', 'Invalid credentials');
  });
  
  it('should redirect to login page when not authenticated', () => {
    cy.visit('/dashboard');
    cy.url().should('include', '/login');
  });
});

// cypress/e2e/todo.cy.js
describe('Todo App', () => {
  beforeEach(() => {
    cy.login('user@example.com', 'password123');
    cy.visit('/todos');
  });
  
  it('should allow user to create a new todo', () => {
    const todoText = 'Buy groceries';
    
    cy.get('[data-cy=new-todo-input]').type(todoText);
    cy.get('[data-cy=add-todo-button]').click();
    
    cy.get('[data-cy=todo-list]')
      .contains(todoText)
      .should('be.visible');
  });
  
  it('should allow user to mark todo as complete', () => {
    cy.get('[data-cy=todo-item]')
      .first()
      .find('[data-cy=complete-checkbox]')
      .click();
    
    cy.get('[data-cy=todo-item]')
      .first()
      .should('have.class', 'completed');
  });
  
  it('should allow user to delete a todo', () => {
    cy.get('[data-cy=todo-item]')
      .first()
      .find('[data-cy=delete-button]')
      .click();
    
    cy.get('[data-cy=todo-item]')
      .first()
      .should('not.contain', 'Previous todo text');
  });
  
  it('should filter todos correctly', () => {
    // Создание тестовых данных
    cy.get('[data-cy=new-todo-input]').type('Active todo');
    cy.get('[data-cy=add-todo-button]').click();
    
    cy.get('[data-cy=todo-item]')
      .first()
      .find('[data-cy=complete-checkbox]')
      .click();
    
    // Фильтрация активных
    cy.get('[data-cy=filter-active]').click();
    cy.get('[data-cy=todo-item]').should('have.length', 1);
    cy.get('[data-cy=todo-item]')
      .first()
      .should('not.have.class', 'completed');
    
    // Фильтрация завершенных
    cy.get('[data-cy=filter-completed]').click();
    cy.get('[data-cy=todo-item]').should('have.length', 1);
    cy.get('[data-cy=todo-item]')
      .first()
      .should('have.class', 'completed');
  });
});
```

## Современные подходы к сборке

### 1. Монорепозитории

Монорепозитории позволяют управлять несколькими пакетами в одном репозитории.

```javascript
// package.json для монорепозитория
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "build": "lerna run build",
    "test": "lerna run test",
    "lint": "lerna run lint"
  },
  "devDependencies": {
    "lerna": "^7.0.0"
  }
}

// lerna.json
{
  "packages": [
    "packages/*"
  ],
  "version": "independent",
  "npmClient": "yarn",
  "useWorkspaces": true
}

// Структура монорепозитория
/*
packages/
├── core/
│   ├── package.json
│   ├── src/
│   └── dist/
├── ui-components/
│   ├── package.json
│   ├── src/
│   └── dist/
├── api-client/
│   ├── package.json
│   ├── src/
│   └── dist/
└── website/
    ├── package.json
    ├── src/
    └── dist/
*/

// packages/core/package.json
{
  "name": "@myorg/core",
  "version": "1.2.3",
  "main": "dist/index.js",
  "module": "dist/index.esm.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "rollup -c",
    "test": "jest"
  },
  "dependencies": {
    "lodash": "^4.17.0"
  }
}

// packages/ui-components/package.json
{
  "name": "@myorg/ui-components",
  "version": "2.1.0",
  "main": "dist/index.js",
  "dependencies": {
    "@myorg/core": "^1.2.0",
    "react": "^18.0.0"
  }
}
```

### 2. Контейнеризация и CI/CD

Использование Docker и CI/CD для автоматизации сборки и развертывания.

```dockerfile
# Dockerfile для Node.js приложения
FROM node:18-alpine AS builder

WORKDIR /app

# Копирование package файлов
COPY package*.json ./
COPY yarn.lock ./

# Установка зависимостей
RUN yarn install --frozen-lockfile --production=false

# Копирование исходного кода
COPY . .

# Сборка приложения
RUN yarn build

# Production stage
FROM node:18-alpine AS production

WORKDIR /app

# Создание непривилегированного пользователя
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Копирование package файлов для production
COPY package*.json ./
COPY yarn.lock ./

# Установка только production зависимостей
RUN yarn install --frozen-lockfile --production

# Копирование собранного приложения
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/public ./public

USER nextjs

EXPOSE 3000

CMD ["node", "dist/server.js"]

# .dockerignore
node_modules
dist
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
```

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'yarn'
    
    - name: Install dependencies
      run: yarn install --frozen-lockfile
    
    - name: Run linting
      run: yarn lint
    
    - name: Run tests
      run: yarn test --coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
    
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18.x'
        cache: 'yarn'
    
    - name: Install dependencies
      run: yarn install --frozen-lockfile
    
    - name: Build application
      run: yarn build
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker tag myapp:${{ github.sha }} myapp:latest
    
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Push Docker image
      run: |
        docker push myapp:${{ github.sha }}
        docker push myapp:latest
    
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      run: |
        # Здесь будет код для развертывания
        echo "Deploying version ${{ github.sha }} to production"
```

## Заключение

Инструментарий и системы сборки в JavaScript постоянно развиваются, предлагая разработчикам мощные средства для повышения продуктивности и качества кода. Ключевые аспекты современного инструментария:

1. **Транспиляция**:
   - Babel для поддержки современных возможностей JavaScript
   - TypeScript для статической типизации

2. **Сборка и бандлинг**:
   - Webpack для сложных приложений
   - Vite для быстрой разработки
   - Rollup для библиотек

3. **Качество кода**:
   - ESLint для статического анализа
   - Prettier для автоматического форматирования

4. **Тестирование**:
   - Jest для unit и интеграционного тестирования
   - Cypress для end-to-end тестирования

5. **Современные практики**:
   - Монорепозитории для управления множеством пакетов
   - Контейнеризация для консистентной среды
   - CI/CD для автоматизации процесса разработки

Выбор инструментов зависит от специфики проекта, но важно понимать принципы работы каждого инструмента и уметь их правильно настраивать и использовать. Регулярное обновление инструментария и следование лучшим практикам помогут создавать современные и качественные веб-приложения.

#javascript #tooling #build-systems #webpack #babel #typescript #eslint #jest #cypress #devops #frontend-development