# Webpack in TypeScript Ecosystem

Webpack - мощный модульный сборщик, который отлично интегрируется с TypeScript для создания оптимизированных приложений. Он позволяет использовать TypeScript в сочетании с другими ресурсами (CSS, изображения, и т.д.) для создания комплексных билдов.

## Основы интеграции TypeScript и Webpack

### Установка зависимостей
```bash
npm install --save-dev webpack webpack-cli typescript ts-loader
# или для разработки
npm install --save-dev @types/webpack webpack-dev-server
```

### Базовая конфигурация
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',  // точка входа - TypeScript файл
  module: {
    rules: [
      {
        test: /\.tsx?$/,           // обработка .ts и .tsx файлов
        use: 'ts-loader',         // использование TypeScript loader
        exclude: /node_modules/,   // исключить node_modules
      },
      // Другие loaders для других типов файлов
      {
        test: /\.css$/i,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],  // приоритетные расширения
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

### Конфигурация TypeScript loader
```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          configFile: path.resolve(__dirname, 'tsconfig.json'),  // указать конкретный tsconfig
          transpileOnly: true,  // быстрая транспиляция без проверки типов (для разработки)
          compilerOptions: {    // переопределить опции компиляции
            module: 'es6'
          }
        },
        exclude: /node_modules/,
      }
    ]
  }
};
```

## Продвинутая конфигурация

### Оптимизация производительности
```javascript
// webpack.config.js - продвинутая конфигурация
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: [
          {
            loader: 'babel-loader',  // сначала Babel для дополнительных трансформаций
            options: {
              presets: ['@babel/preset-env']
            }
          },
          {
            loader: 'ts-loader',    // затем TypeScript
            options: {
              transpileOnly: true,  // только транспиляция, без проверки типов
              configFile: path.resolve(__dirname, 'tsconfig.json')
            }
          }
        ],
        exclude: /node_modules/,
      }
    ]
  },
  plugins: [
    // Плагин для проверки типов в отдельном процессе (улучшает производительность)
    new (require('fork-ts-checker-webpack-plugin'))({
      typescript: {
        configFile: path.resolve(__dirname, 'tsconfig.json'),
        diagnosticOptions: {
          semantic: true,
          syntactic: true
        }
      }
    })
  ],
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx']
  }
};
```

### Многофайловая сборка
```javascript
// webpack.config.js - многоэндпоинтовая сборка
module.exports = {
  entry: {
    app: './src/app.ts',
    admin: './src/admin.ts',
    vendor: ['react', 'react-dom']  // отдельные бандлы для библиотек
  },
  output: {
    filename: '[name].bundle.js',  // [name] заменяется на название entry point
    path: path.resolve(__dirname, 'dist')
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        }
      }
    }
  }
};
```

## TypeScript специфичные особенности

### Работа с declaration файлами
```javascript
// webpack.config.js
module.exports = {
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
    // Правила разрешения модулей TypeScript
    modules: [
      path.resolve(__dirname, 'src'),
      'node_modules'
    ]
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: [{
          loader: 'ts-loader',
          options: {
            configFile: path.resolve(__dirname, 'tsconfig.json'),
            // Опции для работы с declaration файлами
            onlyCompileBundledFiles: true,  // компилировать только файлы из бандла
          }
        }]
      }
    ]
  }
};
```

### Совместимость с разными целями ES
```javascript
// webpack.config.js - настройка под разные цели
module.exports = [
  // Modern bundle (ES2020+, для современных браузеров)
  {
    name: 'modern',
    entry: './src/modern.ts',
    output: {
      filename: 'app.modern.js',
      path: path.resolve(__dirname, 'dist'),
    },
    module: {
      rules: [
        {
          test: /\.ts$/,
          use: {
            loader: 'ts-loader',
            options: {
              configFile: path.resolve(__dirname, 'tsconfig.modern.json')
            }
          }
        }
      ]
    }
  },
  // Legacy bundle (ES5, для старых браузеров)
  {
    name: 'legacy', 
    entry: './src/legacy.ts',
    output: {
      filename: 'app.legacy.js',
      path: path.resolve(__dirname, 'dist'),
    },
    module: {
      rules: [
        {
          test: /\.ts$/,
          use: {
            loader: 'ts-loader',
            options: {
              configFile: path.resolve(__dirname, 'tsconfig.legacy.json')
            }
          }
        }
      ]
    }
  }
];
```

### tsconfig.modern.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "outDir": "./dist/types",
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

### tsconfig.legacy.json
```json
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs", 
    "moduleResolution": "node",
    "lib": ["ES5", "ES2015.Promise", "DOM"],
    "strict": true,
    "esModuleInterop": true, 
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "outDir": "./dist/types",
    "sourceMap": true,
    "downlevelIteration": true,
    "importHelpers": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

## Практические примеры

### React приложение с TypeScript
```javascript
// webpack.config.js для React + TypeScript приложения
const HtmlWebpackPlugin = require('html-webpack-plugin');
const path = require('path');

module.exports = {
  entry: './src/index.tsx',
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils')
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.jsx?$/,
        use: 'babel-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.svg$/,
        use: ['@svgr/webpack', 'url-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    })
  ]
};
```

### TypeScript библиотека
```javascript
// webpack.lib.config.js для библиотеки
module.exports = {
  entry: './src/index.ts',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-library.js',
    library: {
      name: 'MyLibrary',
      type: 'umd',
      umdNamedDefine: true
    },
    globalObject: 'this'
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: {
          loader: 'ts-loader',
          options: {
            configFile: path.resolve(__dirname, 'tsconfig.lib.json')
          }
        },
        exclude: /node_modules/
      }
    ]
  },
  externals: {
    // Не включать эти зависимости в бандл
    'react': 'react',
    'react-dom': 'react-dom'
  }
};
```

### tsconfig.lib.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "importHelpers": true,
    "downlevelIteration": true,
    "stripInternal": true      // удалять @internal комментарии
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "tests"]
}
```

## Современные подходы

### Использование webpack 5
```javascript
// webpack 5 config
module.exports = {
  mode: 'production',
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.ts$/,
        loader: 'ts-loader',
        options: {
          configFile: path.resolve(__dirname, 'tsconfig.json'),
        },
        exclude: /node_modules/,
      }
    ]
  },
  optimization: {
    // Webpack 5 дерево сбрасывания (tree-shaking) улучшено
    sideEffects: false,
    usedExports: true,
    providedExports: true
  },
  experiments: {
    // Экспериментальные возможности Webpack 5
    outputModule: true  // поддержка ES модулей в output
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    module: true,  // генерировать ES модуль
    library: {
      type: 'module'  // для ES модулей
    }
  }
};
```

### Совместимость с ESM и CommonJS
```javascript
// Универсальная конфигурация
module.exports = [
  // ESM версия
  {
    name: 'esm',
    entry: './src/index.ts',
    output: {
      filename: 'index.esm.js',
      path: path.resolve(__dirname, 'dist/esm'),
      library: { type: 'module' }
    },
    experiments: { outputModule: true },
    // ... остальные настройки
  },
  // CommonJS версия
  {
    name: 'cjs',
    entry: './src/index.ts',
    output: {
      filename: 'index.cjs.js',
      path: path.resolve(__dirname, 'dist/cjs'),
      library: { type: 'commonjs2' }
    },
    // ... остальные настройки
  }
];
```

## Лучшие практики

### 1. Раздельная конфигурация для разработки и продакшена
```javascript
// webpack.dev.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'eval-cheap-module-source-map',  // быстрая отладка
  devServer: {
    contentBase: './dist',
    hot: true
  }
});

// webpack.prod.js
const { merge } = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  mode: 'production',
  devtool: 'source-map',  // полные sourcemaps для продакшена
  optimization: {
    minimize: true
  }
});
```

### 2. Оптимизация для TypeScript
```javascript
module.exports = {
  // ... остальные настройки
  plugins: [
    // Проверка типов в отдельном процессе
    new ForkTsCheckerWebpackPlugin({
      typescript: {
        configFile: path.resolve(__dirname, 'tsconfig.json'),
        build: true,  // использовать incremental compilation
        mode: 'write-references'  // улучшенная производительность
      },
      issue: {
        exclude: [
          { file: '**/node_modules/**' }  // исключить ошибки из node_modules
        ]
      }
    })
  ],
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        types: {
          test: /[\\/]node_modules[\\/](@types|typescript)/,  // отдельный chunk для типов
          name: 'types',
          chunks: 'all',
          enforce: true
        }
      }
    }
  }
};
```

## Интеграция с инструментами

### Совместимость с TypeScript Watch
```bash
# Команда для разработки
webpack --watch --mode development

# Или с TypeScript отдельно
tsc --watch & webpack-dev-server
```

### Интеграция с ESLint
```javascript
// webpack.config.js с ESLint
module.exports = {
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: [
          'ts-loader',
          {
            loader: 'eslint-loader',  // проверка стиля кода
            options: {
              configFile: path.resolve(__dirname, '.eslintrc.js')
            }
          }
        ]
      }
    ]
  }
};
```

## Ошибки и решения

### Распространенные проблемы
```javascript
// 1. Проблема с резолюцией модулей
{
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      // Убедитесь, что пути корректны
      '@': path.resolve(__dirname, 'src')
    },
    modules: [
      'node_modules',
      path.resolve(__dirname, 'src')
    ]
  }
}

// 2. Проблема с проверкой типов
{
  plugins: [
    new ForkTsCheckerWebpackPlugin({
      typescript: {
        configFile: path.resolve(__dirname, 'tsconfig.json'),
        diagnosticOptions: {
          semantic: true,
          syntactic: true
        },
        // Для улучшения производительности
        mode: 'write-references'
      }
    })
  ]
}
```

### Сложности с HMR
```javascript
// tsconfig.json для HMR
{
  "compilerOptions": {
    "module": "ESNext",  // важно для HMR
    "moduleResolution": "node",
    "jsx": "react-jsx",  // реакт специфичные настройки
    // остальные опции
  }
}

// webpack.config.js
module.exports = {
  devServer: {
    hot: true,
    liveReload: false  // отключить live reload если используется HMR
  },
  // остальные настройки
};
```

## Связь с другими концепциями
- [[typescript-compiler]] - как webpack использует tsc через ts-loader
- [[module-resolution]] - как webpack разрешает модули TypeScript
- [[bundling-optimization]] - оптимизация бандлов
- [[build-performance]] - производительность сборки
- [[deployment/bundling]] - деплой собранного кода