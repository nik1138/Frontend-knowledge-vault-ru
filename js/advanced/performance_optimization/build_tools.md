# Оптимизация сборки в JavaScript

## Введение

Оптимизация процесса сборки играет ключевую роль в создании быстрых и эффективных веб-приложений. Современные инструменты сборки позволяют автоматизировать множество задач, включая минификацию кода, оптимизацию изображений, разделение кода и многое другое. В этом разделе мы рассмотрим различные техники оптимизации сборки с использованием популярных инструментов.

## Webpack оптимизация

Настройка Webpack для максимальной производительности:

```javascript
// Webpack оптимизация
class WebpackOptimization {
  // Базовая оптимизированная конфигурация
  static getBaseConfig() {
    return {
      mode: 'production',
      entry: {
        main: './src/index.js',
        vendor: './src/vendor.js'
      },
      output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
        chunkFilename: '[name].[contenthash].chunk.js',
        clean: true // Автоматическая очистка dist директории
      },
      optimization: {
        // Минификация JavaScript
        minimize: true,
        minimizer: [
          new TerserPlugin({
            terserOptions: {
              compress: {
                drop_console: true, // Удаляем console.log
                drop_debugger: true, // Удаляем debugger
                pure_funcs: ['console.log'] // Удаляем конкретные функции
              },
              mangle: true, // Минификация имен переменных
              format: {
                comments: false // Удаляем комментарии
              }
            }
          })
        ],
        
        // Code splitting
        splitChunks: {
          chunks: 'all',
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
        },
        
        // Runtime chunk для оптимизации загрузки
        runtimeChunk: {
          name: 'runtime'
        }
      },
      
      plugins: [
        // Очистка dist директории
        new CleanWebpackPlugin(),
        
        // Минификация CSS
        new MiniCssExtractPlugin({
          filename: '[name].[contenthash].css',
          chunkFilename: '[id].[contenthash].css'
        }),
        
        // Оптимизация изображений
        new ImageMinimizerPlugin({
          minimizer: {
            implementation: ImageMinimizerPlugin.imageminMinify,
            options: {
              plugins: [
                ['gifsicle', { interlaced: true }],
                ['jpegtran', { progressive: true }],
                ['optipng', { optimizationLevel: 5 }],
                ['svgo', {
                  plugins: [
                    {
                      name: 'preset-default',
                      params: {
                        overrides: {
                          removeViewBox: false,
                          addAttributesToSVGElement: {
                            params: {
                              attributes: [
                                { xmlns: 'http://www.w3.org/2000/svg' }
                              ]
                            }
                          }
                        }
                      }
                    }
                  ]
                }]
              ]
            }
          }
        }),
        
        // Анализ бандла
        new BundleAnalyzerPlugin({
          analyzerMode: 'static',
          openAnalyzer: false,
          reportFilename: 'bundle-report.html'
        })
      ],
      
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
              loader: 'babel-loader',
              options: {
                presets: [
                  ['@babel/preset-env', {
                    useBuiltIns: 'usage',
                    corejs: 3,
                    targets: '> 0.25%, not dead'
                  }]
                ],
                cacheDirectory: true // Кэширование для ускорения сборки
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
            test: /\.(png|jpe?g|gif|svg)$/i,
            type: 'asset/resource',
            generator: {
              filename: 'images/[name].[contenthash][ext]'
            }
          }
        ]
      }
    };
  }
  
  // Оптимизация для development
  static getDevConfig() {
    return {
      mode: 'development',
      devtool: 'eval-source-map',
      optimization: {
        removeAvailableModules: false,
        removeEmptyChunks: false,
        splitChunks: false,
        minimize: false,
        concatenateModules: false
      },
      cache: {
        type: 'filesystem', // Кэширование на файловой системе
        buildDependencies: {
          config: [__filename]
        }
      },
      devServer: {
        hot: true, // Hot Module Replacement
        compress: true, // Сжатие gzip
        historyApiFallback: true,
        open: true
      }
    };
  }
  
  // Продвинутая оптимизация
  static getAdvancedOptimization() {
    return {
      optimization: {
        // Удаление неиспользуемого кода
        usedExports: true,
        sideEffects: false,
        
        // Оптимизация внутренних зависимостей
        innerGraph: true,
        
        // Минификация с продвинутыми настройками
        minimizer: [
          new TerserPlugin({
            parallel: true, // Параллельная минификация
            terserOptions: {
              compress: {
                arrows: true,
                collapse_vars: true,
                comparisons: true,
                computed_props: true,
                hoist_funs: true,
                hoist_props: true,
                hoist_vars: true,
                inline: true,
                loops: true,
                negate_iife: true,
                properties: true,
                reduce_funcs: true,
                reduce_vars: true,
                switches: true,
                toplevel: true,
                typeofs: true,
                booleans: true,
                if_return: true,
                sequences: true,
                unused: true,
                conditionals: true,
                dead_code: true,
                evaluate: true
              }
            }
          })
        ],
        
        // Оптимизация CSS
        splitChunks: {
          cacheGroups: {
            styles: {
              name: 'styles',
              type: 'css/mini-extract',
              chunks: 'all',
              enforce: true
            }
          }
        }
      },
      
      // Оптимизация загрузки модулей
      resolve: {
        alias: {
          '@': path.resolve(__dirname, 'src'),
          'components': path.resolve(__dirname, 'src/components')
        },
        extensions: ['.js', '.jsx', '.json'],
        modules: [
          'node_modules',
          path.resolve(__dirname, 'src')
        ]
      },
      
      // Внешние зависимости
      externals: {
        'react': 'React',
        'react-dom': 'ReactDOM'
      }
    };
  }
}

// Пример полной конфигурации webpack
const webpackConfig = {
  ...WebpackOptimization.getBaseConfig(),
  ...WebpackOptimization.getAdvancedOptimization(),
  
  plugins: [
    ...WebpackOptimization.getBaseConfig().plugins,
    
    // Определение переменных окружения
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.API_URL': JSON.stringify(process.env.API_URL || 'http://localhost:3000')
    }),
    
    // Копирование статических файлов
    new CopyWebpackPlugin({
      patterns: [
        { from: 'public', to: '.' }
      ]
    }),
    
    // Генерация HTML файлов
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeRedundantAttributes: true,
        useShortDoctype: true,
        removeEmptyAttributes: true,
        removeStyleLinkTypeAttributes: true,
        keepClosingSlash: true,
        minifyJS: true,
        minifyCSS: true,
        minifyURLs: true
      }
    })
  ]
};
```

## Vite оптимизация

Настройка Vite для быстрой сборки:

```javascript
// Vite оптимизация
class ViteOptimization {
  // Базовая конфигурация Vite
  static getBaseConfig() {
    return {
      // Vite использует esbuild для транспиляции - очень быстро
      optimizeDeps: {
        include: ['lodash-es', 'axios', 'react', 'react-dom'],
        exclude: ['some-heavy-library']
      },
      
      build: {
        // Rollup опции
        rollupOptions: {
          output: {
            manualChunks: {
              vendor: ['react', 'react-dom'],
              utils: ['lodash-es', 'moment'],
              ui: ['@mui/material', '@emotion/react']
            }
          }
        },
        
        // Минификация
        minify: 'esbuild', // Быстрее чем terser
        
        // CSS оптимизация
        cssCodeSplit: true,
        
        // Сжатие brotli
        brotliSize: true,
        
        // Размер чанка для предупреждений
        chunkSizeWarningLimit: 1000
      },
      
      // Development сервер
      server: {
        host: true,
        port: 3000,
        hmr: true, // Hot Module Replacement
        cors: true,
        proxy: {
          '/api': {
            target: 'http://localhost:8000',
            changeOrigin: true,
            secure: false
          }
        }
      },
      
      // Preview сервер
      preview: {
        port: 5000
      }
    };
  }
  
  // Продвинутая оптимизация Vite
  static getAdvancedConfig() {
    return {
      build: {
        // Таргеты для современных браузеров
        target: 'es2015',
        
        // Полифиллы
        polyfillModulePreload: true,
        
        // Оптимизация изображений
        assetsInlineLimit: 4096, // Inline assets smaller than 4kb
        
        // Оптимизация CSS
        css: {
          preprocessorOptions: {
            scss: {
              additionalData: `@import "src/styles/variables.scss";`
            }
          }
        },
        
        // Rollup продвинутые опции
        rollupOptions: {
          external: ['some-external-library'],
          output: {
            assetFileNames: (assetInfo) => {
              let extType = assetInfo.name.split('.').at(1);
              if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(extType)) {
                extType = 'img';
              } else if (/woff|woff2/.test(extType)) {
                extType = 'fonts';
              }
              return `assets/${extType}/[name]-[hash][extname]`;
            },
            chunkFileNames: 'assets/js/[name]-[hash].js',
            entryFileNames: 'assets/js/[name]-[hash].js'
          }
        }
      },
      
      // Оптимизация зависимостей
      optimizeDeps: {
        esbuildOptions: {
          target: 'es2020'
        }
      },
      
      // Плагины
      plugins: [
        // React поддержка
        react(),
        
        // PWA поддержка
        VitePWA({
          registerType: 'autoUpdate',
          devOptions: {
            enabled: false
          },
          workbox: {
            globPatterns: ['**/*.{js,css,html,ico,png,svg}']
          },
          manifest: {
            name: 'My App',
            short_name: 'App',
            description: 'My awesome app',
            theme_color: '#ffffff'
          }
        })
      ]
    };
  }
  
  // Оптимизация для production
  static getProductionConfig() {
    return {
      build: {
        // Включаем sourcemaps для production
        sourcemap: true,
        
        // Оптимизация изображений
        assetsInlineLimit: 0, // Не inline assets в production
        
        // Размер чанка для предупреждений
        chunkSizeWarningLimit: 500,
        
        // Оптимизация CSS
        css: {
          minify: true
        }
      }
    };
  }
}

// Пример полной конфигурации vite
const viteConfig = {
  ...ViteOptimization.getBaseConfig(),
  ...ViteOptimization.getAdvancedConfig(),
  ...ViteOptimization.getProductionConfig()
};

// Динамическая конфигурация в зависимости от режима
export default defineConfig(({ command, mode }) => {
  if (command === 'serve') {
    return {
      ...ViteOptimization.getBaseConfig(),
      // Development специфичные настройки
      define: {
        __APP_ENV__: JSON.stringify(mode)
      }
    };
  } else {
    // Build специфичные настройки
    return {
      ...ViteOptimization.getBaseConfig(),
      ...ViteOptimization.getProductionConfig()
    };
  }
});
```

## Rollup оптимизация

Настройка Rollup для библиотек:

```javascript
// Rollup оптимизация
class RollupOptimization {
  // Базовая конфигурация Rollup
  static getBaseConfig() {
    return {
      input: 'src/index.js',
      output: [
        {
          file: 'dist/bundle.cjs.js',
          format: 'cjs',
          sourcemap: true,
          exports: 'named'
        },
        {
          file: 'dist/bundle.esm.js',
          format: 'es',
          sourcemap: true
        },
        {
          file: 'dist/bundle.umd.js',
          format: 'umd',
          name: 'MyLibrary',
          sourcemap: true,
          plugins: [
            terser({
              compress: {
                pure_getters: true,
                unsafe: true,
                unsafe_comps: true,
                warnings: false
              },
              mangle: {
                properties: {
                  regex: /^__/,
                }
              },
              output: {
                comments: false
              }
            })
          ]
        }
      ],
      plugins: [
        // Поддержка Node.js модулей
        nodeResolve({
          browser: true,
          preferBuiltins: false
        }),
        
        // CommonJS модули
        commonjs({
          include: /node_modules/,
          namedExports: {
            'react': Object.keys(require('react'))
          }
        }),
        
        // Компиляция TypeScript
        typescript({
          tsconfig: './tsconfig.json',
          sourceMap: true,
          inlineSources: true
        }),
        
        // Минификация
        terser({
          module: true,
          warnings: true,
          mangle: {
            properties: {
              regex: /^__/,
            }
          },
          output: {
            comments: false
          }
        }),
        
        // Генерация файлов деклараций для TypeScript
        dts()
      ],
      
      // Внешние зависимости
      external: [
        'react',
        'react-dom',
        'lodash',
        'moment'
      ]
    };
  }
  
  // Оптимизация tree shaking
  static getTreeShakingConfig() {
    return {
      plugins: [
        // Удаление неиспользуемого кода
        terser({
          module: true,
          warnings: true,
          mangle: {
            properties: {
              regex: /^__/,
            }
          },
          output: {
            comments: false
          },
          compress: {
            pure_getters: true,
            unsafe: true,
            unsafe_comps: true,
            warnings: false,
            passes: 2 // Два прохода для лучшей оптимизации
          }
        })
      ],
      
      // Настройки для оптимального tree shaking
      treeshake: {
        moduleSideEffects: false,
        propertyReadSideEffects: false,
        tryCatchDeoptimization: false,
        unknownGlobalSideEffects: false
      }
    };
  }
  
  // Множественные точки входа
  static getMultiEntryConfig() {
    return {
      input: {
        main: 'src/index.js',
        utils: 'src/utils.js',
        components: 'src/components/index.js'
      },
      output: [
        {
          dir: 'dist/es',
          format: 'es',
          sourcemap: true
        },
        {
          dir: 'dist/cjs',
          format: 'cjs',
          sourcemap: true
        }
      ]
    };
  }
  
  // Оптимизация для разных целей
  static getTargetConfigs() {
    return [
      // Modern browsers
      {
        input: 'src/index.js',
        output: {
          file: 'dist/modern/index.js',
          format: 'es'
        },
        plugins: [
          nodeResolve({ browser: true }),
          terser({
            ecma: 2017,
            module: true
          })
        ]
      },
      
      // Legacy browsers
      {
        input: 'src/index.js',
        output: {
          file: 'dist/legacy/index.js',
          format: 'umd',
          name: 'MyLibrary'
        },
        plugins: [
          nodeResolve({ browser: true }),
          babel({
            presets: [
              ['@babel/preset-env', {
                targets: '> 0.25%, not dead'
              }]
            ]
          }),
          terser()
        ]
      }
    ];
  }
}

// Пример полной конфигурации rollup
const rollupConfig = [
  {
    ...RollupOptimization.getBaseConfig(),
    ...RollupOptimization.getTreeShakingConfig()
  },
  ...RollupOptimization.getTargetConfigs()
];

export default rollupConfig;
```

## Parcel оптимизация

Настройка Parcel для простой и быстрой сборки:

```javascript
// Parcel оптимизация
class ParcelOptimization {
  // Конфигурация через package.json
  static getPackageJsonConfig() {
    return {
      "name": "my-app",
      "version": "1.0.0",
      "source": "src/index.html",
      "browserslist": "> 0.5%, last 2 versions, not dead",
      "targets": {
        "main": {
          "context": "browser",
          "engines": {
            "browsers": ["> 0.5%, last 2 versions, not dead"]
          }
        }
      },
      "scripts": {
        "start": "parcel serve src/index.html",
        "build": "parcel build src/index.html --dist-dir dist",
        "analyze": "parcel build src/index.html --dist-dir dist --profile"
      },
      "devDependencies": {
        "parcel": "^2.0.0",
        "@parcel/transformer-sass": "^2.0.0",
        "@parcel/optimizer-css": "^2.0.0"
      }
    };
  }
  
  // Конфигурация через .parcelrc
  static getParcelRcConfig() {
    return {
      "extends": "@parcel/config-default",
      "transformers": {
        "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"],
        "*.vue": ["@parcel/transformer-vue"]
      },
      "optimizers": {
        "*.js": ["@parcel/optimizer-terser"],
        "*.css": ["@parcel/optimizer-cssnano"]
      },
      "reporters": [
        "...",
        "@parcel/reporter-bundle-analyzer"
      ]
    };
  }
  
  // Продвинутая конфигурация
  static getAdvancedConfig() {
    return {
      // Оптимизации по умолчанию
      "optimizers": {
        "bundle": [
          "@parcel/optimizer-cssnano",
          "@parcel/optimizer-htmlnano",
          "@parcel/optimizer-image",
          "@parcel/optimizer-svgo",
          "@parcel/optimizer-terser"
        ]
      },
      
      // Кастомные трансформеры
      "transformers": {
        "*.{js,mjs,jsm,jsx,es6,cjs,ts,tsx}": [
          "@parcel/transformer-js",
          "@parcel/transformer-react-refresh-wrap"
        ],
        "*.{css,pcss}": [
          "@parcel/transformer-css"
        ],
        "*.{html,htm}": [
          "@parcel/transformer-html",
          "@parcel/transformer-posthtml"
        ]
      },
      
      // Настройки оптимизаторов
      "optimizer-options": {
        "@parcel/optimizer-terser": {
          "mangle": true,
          "compress": {
            "drop_console": true,
            "drop_debugger": true
          }
        },
        "@parcel/optimizer-cssnano": {
          "preset": [
            "default",
            {
              "cssDeclarationSorter": false
            }
          ]
        }
      }
    };
  }
  
  // CLI оптимизации
  static getCLIOptions() {
    return {
      // Development
      "serve": {
        "port": 1234,
        "host": "localhost",
        "https": false,
        "hmr": true,
        "cache": true,
        "contentHash": false
      },
      
      // Build
      "build": {
        "minify": true,
        "scopeHoist": true,
        "sourceMaps": true,
        "publicUrl": "./",
        "cache": true,
        "contentHash": true,
        "detailedReport": true
      },
      
      // Profile
      "profile": {
        "profile": true,
        "detailedReport": true
      }
    };
  }
}

// Пример использования Parcel с кастомной конфигурацией
class ParcelBuildManager {
  constructor() {
    this.config = ParcelOptimization.getPackageJsonConfig();
  }
  
  // Оптимизированная сборка
  async optimizedBuild() {
    const bundler = new Parcel({
      entries: 'src/index.html',
      defaultConfig: '@parcel/config-default',
      mode: 'production',
      defaultTargetOptions: {
        distDir: 'dist',
        engines: {
          browsers: ['> 0.5%', 'last 2 versions', 'not dead']
        }
      },
      additionalReporters: [
        { packageName: '@parcel/reporter-cli', resolveFrom: __filename },
        { packageName: '@parcel/reporter-bundle-analyzer', resolveFrom: __filename }
      ]
    });
    
    try {
      const { bundleGraph, buildTime } = await bundler.run();
      console.log(`Сборка завершена за ${buildTime}ms`);
      return bundleGraph;
    } catch (error) {
      console.error('Ошибка сборки:', error);
      throw error;
    }
  }
  
  // Development сервер с оптимизациями
  async startDevServer() {
    const bundler = new Parcel({
      entries: 'src/index.html',
      defaultConfig: '@parcel/config-default',
      mode: 'development',
      hmrOptions: {
        port: 1234
      },
      defaultTargetOptions: {
        distDir: '.parcel-cache/dist'
      }
    });
    
    const devServer = await bundler.serve(1234, false, 'localhost');
    console.log('Development сервер запущен на http://localhost:1234');
    return devServer;
  }
}
```

## Практические рекомендации

1. **Используйте современные бандлеры** - Vite, Parcel для быстрой сборки
2. **Применяйте code splitting** - для уменьшения размера начального бандла
3. **Оптимизируйте изображения** - сжатие и современные форматы
4. **Используйте tree shaking** - для удаления неиспользуемого кода
5. **Применяйте минификацию** - для уменьшения размера файлов
6. **Используйте кэширование** - для ускорения повторных сборок
7. **Анализируйте бандл** - регулярно проверяйте состав и размер
8. **Оптимизируйте для production** - разные настройки для dev и prod

Оптимизация сборки - важный аспект создания быстрых и эффективных веб-приложений. Правильная настройка инструментов сборки может значительно улучшить производительность и пользовательский опыт.

#javascript #build-tools #optimization #performance #webpack #vite #rollup #parcel