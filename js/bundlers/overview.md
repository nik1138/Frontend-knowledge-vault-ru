# Bundlers API

## Введение

Сборщики модулей (bundlers) - это инструменты, которые объединяют модули JavaScript и другие ресурсы в единые файлы для браузера. В этой главе мы рассмотрим популярные сборщики: Webpack, Rollup, Parcel, Vite и другие, их конфигурацию, плагины и лучшие практики использования.

## Содержание

- [[Webpack]]
- [[Rollup]]
- [[Parcel]]
- [[Vite]]
- [[ESBuild]]
- [[Сравнение сборщиков]]
- [[Плагины и расширения]]
- [[Оптимизация бандлов]]
- [[Горячая замена модулей]]

## Webpack

Webpack - один из самых популярных и гибких сборщиков модулей. Он создает граф зависимостей и объединяет все модули в один или несколько бандлов.

### Основы Webpack

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    // Точка входа
    entry: {
        main: './src/index.js',
        vendor: './src/vendor.js'
    },
    
    // Выходная конфигурация
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[contenthash].js',
        chunkFilename: '[name].[contenthash].chunk.js',
        clean: true // Очистка папки dist перед сборкой
    },
    
    // Режим сборки
    mode: process.env.NODE_ENV || 'development',
    
    // Дебаг-информация
    devtool: process.env.NODE_ENV === 'production' ? 'source-map' : 'eval-source-map',
    
    // Модули и правила загрузки
    module: {
        rules: [
            // JavaScript
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
            
            // CSS
            {
                test: /\.css$/,
                use: [
                    process.env.NODE_ENV === 'production' 
                        ? MiniCssExtractPlugin.loader 
                        : 'style-loader',
                    'css-loader'
                ]
            },
            
            // Изображения
            {
                test: /\.(png|jpg|gif|svg)$/,
                type: 'asset/resource',
                generator: {
                    filename: 'images/[name].[hash][ext]'
                }
            },
            
            // Шрифты
            {
                test: /\.(woff|woff2|eot|ttf|otf)$/,
                type: 'asset/resource',
                generator: {
                    filename: 'fonts/[name].[hash][ext]'
                }
            }
        ]
    },
    
    // Плагины
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            minify: process.env.NODE_ENV === 'production'
        }),
        
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css'
        })
    ],
    
    // Оптимизация
    optimization: {
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    chunks: 'all',
                },
                common: {
                    minChunks: 2,
                    chunks: 'all',
                    enforce: true
                }
            }
        },
        
        runtimeChunk: 'single'
    },
    
    // Разработка
    devServer: {
        static: './dist',
        hot: true,
        port: 3000,
        open: true
    }
};
```

### Продвинутая конфигурация Webpack

```javascript
// webpack.advanced.config.js
const path = require('path');
const webpack = require('webpack');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

// Условная конфигурация
const isProduction = process.env.NODE_ENV === 'production';
const isDevelopment = !isProduction;

module.exports = {
    entry: {
        app: './src/main.js'
    },
    
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: isProduction ? '[name].[contenthash:8].js' : '[name].js',
        chunkFilename: isProduction ? '[name].[contenthash:8].chunk.js' : '[name].chunk.js',
        assetModuleFilename: 'assets/[hash][ext][query]',
        publicPath: '/'
    },
    
    mode: isProduction ? 'production' : 'development',
    devtool: isDevelopment ? 'eval-source-map' : 'source-map',
    
    resolve: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
        alias: {
            '@': path.resolve(__dirname, 'src'),
            '@components': path.resolve(__dirname, 'src/components'),
            '@utils': path.resolve(__dirname, 'src/utils')
        }
    },
    
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            ['@babel/preset-env', {
                                targets: '> 0.25%, not dead',
                                useBuiltIns: 'usage',
                                corejs: 3
                            }],
                            '@babel/preset-react'
                        ],
                        plugins: [
                            '@babel/plugin-syntax-dynamic-import',
                            isDevelopment && require.resolve('react-refresh/babel')
                        ].filter(Boolean)
                    }
                }
            },
            
            {
                test: /\.(ts|tsx)$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: 'babel-loader',
                        options: {
                            presets: [
                                ['@babel/preset-env', {
                                    targets: '> 0.25%, not dead'
                                }],
                                '@babel/preset-typescript',
                                '@babel/preset-react'
                            ]
                        }
                    },
                    {
                        loader: 'ts-loader',
                        options: {
                            transpileOnly: true
                        }
                    }
                ]
            },
            
            {
                test: /\.css$/,
                use: [
                    isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            modules: {
                                localIdentName: isProduction 
                                    ? '[hash:base64:8]' 
                                    : '[name]__[local]__[hash:base64:4]'
                            },
                            importLoaders: 1
                        }
                    },
                    'postcss-loader'
                ]
            },
            
            {
                test: /\.s[ac]ss$/,
                use: [
                    isProduction ? MiniCssExtractPlugin.loader : 'style-loader',
                    'css-loader',
                    'sass-loader'
                ]
            },
            
            {
                test: /\.(png|jpe?g|gif|svg|webp)$/i,
                type: 'asset',
                parser: {
                    dataUrlCondition: {
                        maxSize: 8 * 1024 // 8kb
                    }
                },
                generator: {
                    filename: 'images/[name].[contenthash:8][ext]'
                }
            }
        ]
    },
    
    plugins: [
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
            'process.env.API_URL': JSON.stringify(process.env.API_URL)
        }),
        
        isProduction && new MiniCssExtractPlugin({
            filename: '[name].[contenthash:8].css',
            chunkFilename: '[id].[contenthash:8].css'
        }),
        
        isProduction && new BundleAnalyzerPlugin({
            analyzerMode: 'static',
            openAnalyzer: false
        })
    ].filter(Boolean),
    
    optimization: {
        minimize: isProduction,
        minimizer: [
            new TerserPlugin({
                terserOptions: {
                    compress: {
                        drop_console: true, // Удаление console.log в продакшене
                        drop_debugger: true
                    }
                }
            }),
            new CssMinimizerPlugin()
        ],
        
        splitChunks: {
            chunks: 'all',
            cacheGroups: {
                default: {
                    minChunks: 2,
                    priority: -20,
                    reuseExistingChunk: true
                },
                vendor: {
                    test: /[\\/]node_modules[\\/]/,
                    name: 'vendors',
                    priority: -10,
                    chunks: 'all'
                },
                react: {
                    test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
                    name: 'react',
                    priority: 20
                }
            }
        },
        
        runtimeChunk: {
            name: 'runtime'
        }
    },
    
    devServer: {
        static: {
            directory: path.join(__dirname, 'dist')
        },
        hot: true,
        port: 3000,
        open: true,
        historyApiFallback: true,
        client: {
            overlay: {
                errors: true,
                warnings: false
            }
        }
    }
};
```

### Плагины Webpack

```javascript
// custom-webpack-plugins.js
const webpack = require('webpack');

// Плагин для добавления кастомного кода
class CustomCodeInjectionPlugin {
    constructor(options = {}) {
        this.options = {
            code: options.code || '',
            position: options.position || 'before'
        };
    }
    
    apply(compiler) {
        compiler.hooks.emit.tapAsync('CustomCodeInjectionPlugin', (compilation, callback) => {
            // Добавление кода во все JS файлы
            Object.keys(compilation.assets).forEach(filename => {
                if (filename.endsWith('.js')) {
                    const source = compilation.assets[filename].source();
                    let modifiedSource;
                    
                    if (this.options.position === 'before') {
                        modifiedSource = this.options.code + '\n' + source;
                    } else {
                        modifiedSource = source + '\n' + this.options.code;
                    }
                    
                    compilation.assets[filename] = {
                        source: () => modifiedSource,
                        size: () => modifiedSource.length
                    };
                }
            });
            
            callback();
        });
    }
}

// Плагин для анализа размера бандла
class BundleSizeAnalyzerPlugin {
    constructor(options = {}) {
        this.threshold = options.threshold || 1024 * 100; // 100KB по умолчанию
    }
    
    apply(compiler) {
        compiler.hooks.emit.tap('BundleSizeAnalyzerPlugin', (compilation) => {
            let totalSize = 0;
            
            Object.keys(compilation.assets).forEach(filename => {
                const asset = compilation.assets[filename];
                const size = asset.size();
                totalSize += size;
                
                if (size > this.threshold) {
                    console.warn(`⚠️  Файл ${filename} превышает порог: ${(size / 1024).toFixed(2)}KB`);
                }
            });
            
            const totalSizeKB = (totalSize / 1024).toFixed(2);
            console.log(`📦 Общий размер бандла: ${totalSizeKB}KB`);
        });
    }
}

// Плагин для обфускации кода
class CodeObfuscationPlugin {
    constructor(options = {}) {
        this.options = {
            exclude: options.exclude || [],
            obfuscate: options.obfuscate || true
        };
    }
    
    apply(compiler) {
        if (!this.options.obfuscate) return;
        
        compiler.hooks.emit.tapAsync('CodeObfuscationPlugin', (compilation, callback) => {
            Object.keys(compilation.assets).forEach(filename => {
                if (filename.endsWith('.js') && !this.shouldExclude(filename)) {
                    const source = compilation.assets[filename].source();
                    // Простая "обфускация" (в реальном проекте используйте полноценные инструменты)
                    const obfuscated = this.obfuscateCode(source);
                    
                    compilation.assets[filename] = {
                        source: () => obfuscated,
                        size: () => obfuscated.length
                    };
                }
            });
            
            callback();
        });
    }
    
    shouldExclude(filename) {
        return this.options.exclude.some(pattern => 
            filename.includes(pattern)
        );
    }
    
    obfuscateCode(code) {
        // Простая замена имен переменных (демонстрационный пример)
        return code
            .replace(/\b(var|let|const)\s+(\w+)/g, (match, keyword, name) => {
                if (['i', 'j', 'k'].includes(name)) return match; // Не трогаем счетчики циклов
                return `${keyword} _${Math.random().toString(36).substr(2, 5)}`;
            });
    }
}

module.exports = {
    CustomCodeInjectionPlugin,
    BundleSizeAnalyzerPlugin,
    CodeObfuscationPlugin
};
```

## Rollup

Rollup - это сборщик, оптимизированный для библиотек. Он использует ES6 модули и обеспечивает лучшее tree-shaking.

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import { terser } from 'rollup-plugin-terser';
import replace from '@rollup/plugin-replace';
import alias from '@rollup/plugin-alias';

const isProduction = process.env.NODE_ENV === 'production';

export default {
    input: 'src/index.js',
    
    output: [
        // ES Module
        {
            file: 'dist/my-library.esm.js',
            format: 'es',
            sourcemap: !isProduction
        },
        // CommonJS
        {
            file: 'dist/my-library.cjs.js',
            format: 'cjs',
            sourcemap: !isProduction
        },
        // IIFE для браузеров
        {
            file: 'dist/my-library.js',
            format: 'iife',
            name: 'MyLibrary',
            sourcemap: !isProduction
        },
        // UMD
        {
            file: 'dist/my-library.umd.js',
            format: 'umd',
            name: 'MyLibrary',
            sourcemap: !isProduction
        }
    ],
    
    plugins: [
        alias({
            entries: [
                { find: '@', replacement: 'src' },
                { find: '@utils', replacement: 'src/utils' }
            ]
        }),
        
        resolve({
            browser: true,
            dedupe: ['react', 'react-dom']
        }),
        
        commonjs({
            include: /node_modules/
        }),
        
        babel({
            babelHelpers: 'bundled',
            exclude: 'node_modules/**',
            presets: [
                ['@babel/preset-env', {
                    targets: '> 0.25%, not dead',
                    modules: false // Важно для tree-shaking
                }]
            ]
        }),
        
        replace({
            preventAssignment: true,
            values: {
                'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
            }
        }),
        
        isProduction && terser({
            module: true,
            compress: {
                drop_console: true,
                drop_debugger: true
            },
            mangle: {
                properties: {
                    regex: /^__/ // Обфускация свойств, начинающихся с __
                }
            }
        })
    ].filter(Boolean),
    
    external: [
        // Зависимости, которые не нужно включать в бандл
        'react',
        'react-dom',
        'lodash'
    ],
    
    onwarn(warning, warn) {
        // Фильтрация предупреждений
        if (warning.code === 'CIRCULAR_DEPENDENCY') {
            console.warn('Циклическая зависимость:', warning.cycle);
        } else {
            warn(warning);
        }
    }
};
```

## Parcel

Parcel - это "нулевая конфигурация" сборщик, который автоматически определяет зависимости и типы файлов.

```javascript
// package.json для Parcel
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "dev": "parcel serve src/index.html",
    "build": "parcel build src/index.html",
    "watch": "parcel watch src/index.html"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@parcel/config-default": "^2.0.0",
    "@parcel/transformer-js": "^2.0.0",
    "@parcel/transformer-css": "^2.0.0",
    "@parcel/transformer-html": "^2.0.0",
    "parcel": "^2.0.0"
  },
  "targets": {
    "main": {
      "context": "node",
      "optimize": true
    },
    "browser": {
      "context": "browser",
      "engines": {
        "browsers": "> 0.5%, last 2 versions, not dead"
      }
    }
  }
}
```

```javascript
// .parcelrc - конфигурация Parcel
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.js": [
      "...",
      "@parcel/transformer-js"
    ],
    "*.css": [
      "...",
      "@parcel/transformer-css"
    ]
  },
  "optimizers": {
    "*.{js,css,html}": ["@parcel/optimizer-terser", "..."]
  },
  "reporters": ["...", "@parcel/reporter-bundle-analyzer"]
}
```

## Vite

Vite - это современный сборщик, который использует нативные ES модули для быстрой разработки.

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import { resolve } from 'path';
import react from '@vitejs/plugin-react';
import legacy from '@vitejs/plugin-legacy';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
    root: 'src',
    base: '/',
    
    resolve: {
        alias: {
            '@': resolve(__dirname, 'src'),
            '@components': resolve(__dirname, 'src/components'),
            '@utils': resolve(__dirname, 'src/utils')
        }
    },
    
    plugins: [
        react({
            jsxRuntime: 'automatic'
        }),
        
        // Поддержка старых браузеров
        legacy({
            targets: ['defaults', 'not IE 11']
        }),
        
        // PWA
        VitePWA({
            registerType: 'autoUpdate',
            workbox: {
                globPatterns: ['**/*.{js,css,html,ico,png,svg}']
            },
            manifest: {
                name: 'My App',
                short_name: 'App',
                theme_color: '#ffffff',
                icons: [
                    {
                        src: 'pwa-192x192.png',
                        sizes: '192x192',
                        type: 'image/png'
                    }
                ]
            }
        })
    ],
    
    build: {
        outDir: '../dist',
        rollupOptions: {
            input: {
                main: resolve(__dirname, 'src/index.html'),
                nested: resolve(__dirname, 'src/nested/index.html')
            }
        },
        target: 'es2015',
        minify: 'terser',
        cssCodeSplit: true,
        sourcemap: true
    },
    
    server: {
        port: 3000,
        open: true,
        hmr: {
            overlay: true
        }
    },
    
    define: {
        __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
    }
});
```

## ESBuild

ESBuild - это очень быстрый сборщик, написанный на Go.

```javascript
// esbuild.config.js
const esbuild = require('esbuild');

const isDev = process.argv.includes('--dev');

async function build() {
    await esbuild.build({
        entryPoints: ['src/index.js'],
        bundle: true,
        minify: !isDev,
        sourcemap: isDev,
        format: 'esm',
        target: ['es2020'],
        outdir: 'dist',
        define: {
            'process.env.NODE_ENV': JSON.stringify(isDev ? 'development' : 'production')
        },
        plugins: [
            // Плагины для ESBuild
            {
                name: 'custom-plugin',
                setup(build) {
                    build.onResolve({ filter: /^@components\/(.*)/ }, (args) => {
                        return {
                            path: resolve(__dirname, 'src/components', args.path.slice(11))
                        };
                    });
                }
            }
        ]
    });
}

build().catch(() => process.exit(1));
```

## Примеры из промышленной разработки

### Универсальная система сборки

```javascript
// universal-build-system.js
class UniversalBuildSystem {
    constructor(config) {
        this.config = config;
        this.plugins = [];
        this.hooks = new Map();
    }
    
    // Регистрация плагина
    use(plugin) {
        if (typeof plugin === 'function') {
            plugin(this);
        } else if (plugin && typeof plugin.apply === 'function') {
            plugin.apply(this);
        }
        return this;
    }
    
    // Регистрация хука
    hook(name, fn) {
        if (!this.hooks.has(name)) {
            this.hooks.set(name, []);
        }
        this.hooks.get(name).push(fn);
        return this;
    }
    
    // Вызов хука
    async callHook(name, ...args) {
        const hooks = this.hooks.get(name) || [];
        for (const hook of hooks) {
            await hook(...args);
        }
    }
    
    // Основной процесс сборки
    async build() {
        await this.callHook('beforeBuild', this.config);
        
        // Выбор сборщика на основе конфигурации
        const bundler = this.selectBundler();
        const result = await bundler.build(this.config);
        
        await this.callHook('afterBuild', result);
        
        return result;
    }
    
    selectBundler() {
        switch (this.config.bundler) {
            case 'webpack':
                return new WebpackBundler();
            case 'rollup':
                return new RollupBundler();
            case 'vite':
                return new ViteBundler();
            case 'esbuild':
                return new EsbuildBundler();
            default:
                throw new Error(`Неизвестный сборщик: ${this.config.bundler}`);
        }
    }
}

// Абстрактный класс для сборщиков
class Bundler {
    async build(config) {
        throw new Error('Метод build должен быть реализован');
    }
}

// Реализация Webpack сборщика
class WebpackBundler extends Bundler {
    async build(config) {
        const webpack = require('webpack');
        const webpackConfig = this.createConfig(config);
        
        return new Promise((resolve, reject) => {
            webpack(webpackConfig, (err, stats) => {
                if (err) {
                    return reject(err);
                }
                
                const info = stats.toJson();
                if (stats.hasErrors()) {
                    console.error(info.errors);
                    return reject(new Error('Сборка Webpack завершена с ошибками'));
                }
                
                if (stats.hasWarnings()) {
                    console.warn(info.warnings);
                }
                
                resolve({
                    stats: info,
                    outputPath: webpackConfig.output.path
                });
            });
        });
    }
    
    createConfig(config) {
        // Создание конфигурации Webpack на основе переданной конфигурации
        return {
            mode: config.mode || 'development',
            entry: config.entry,
            output: {
                path: config.output.path,
                filename: config.output.filename || '[name].js',
                publicPath: config.output.publicPath || '/'
            },
            module: {
                rules: this.createRules(config.rules || [])
            },
            plugins: this.createPlugins(config.plugins || [])
        };
    }
    
    createRules(rules) {
        // Преобразование правил из универсального формата в формат Webpack
        return rules.map(rule => ({
            test: new RegExp(rule.test),
            use: rule.loaders || rule.use
        }));
    }
    
    createPlugins(plugins) {
        // Создание плагинов Webpack
        return plugins.map(plugin => {
            if (typeof plugin === 'string') {
                const [name, ...options] = plugin.split(':');
                const PluginClass = require(name);
                return new PluginClass(...options);
            }
            return plugin;
        });
    }
}

// Реализация Rollup сборщика
class RollupBundler extends Bundler {
    async build(config) {
        const { rollup } = require('rollup');
        const rollupConfig = this.createConfig(config);
        
        const bundle = await rollup(rollupConfig);
        const result = await bundle.write(rollupConfig.output);
        
        return {
            bundle,
            result,
            outputPath: rollupConfig.output.dir
        };
    }
    
    createConfig(config) {
        return {
            input: config.entry,
            output: {
                dir: config.output.path,
                format: config.output.format || 'es',
                sourcemap: config.sourcemap !== false
            },
            plugins: this.createPlugins(config.plugins || [])
        };
    }
    
    createPlugins(plugins) {
        return plugins.map(plugin => {
            if (typeof plugin === 'string') {
                const [name, ...options] = plugin.split(':');
                const pluginFactory = require(name);
                return pluginFactory(...options);
            }
            return plugin;
        });
    }
}

// Использование универсальной системы
const buildSystem = new UniversalBuildSystem({
    bundler: 'webpack',
    mode: 'production',
    entry: './src/index.js',
    output: {
        path: './dist',
        filename: '[name].[contenthash].js'
    },
    rules: [
        {
            test: '\\.js$',
            loaders: ['babel-loader']
        },
        {
            test: '\\.css$',
            loaders: ['style-loader', 'css-loader']
        }
    ]
});

// Регистрация хуков
buildSystem
    .hook('beforeBuild', async (config) => {
        console.log('Начало сборки...');
    })
    .hook('afterBuild', async (result) => {
        console.log('Сборка завершена!');
        console.log('Результат:', result);
    });

// Выполнение сборки
buildSystem.build()
    .then(result => console.log('Сборка успешна:', result))
    .catch(error => console.error('Ошибка сборки:', error));
```

### Горячая замена модулей (HMR)

```javascript
// hmr-system.js
class HMRSystem {
    constructor(options = {}) {
        this.options = {
            port: options.port || 8080,
            timeout: options.timeout || 30000,
            reload: options.reload !== false
        };
        
        this.connections = new Set();
        this.fileWatcher = null;
        this.dependencies = new Map();
    }
    
    // Инициализация HMR
    init() {
        this.createServer();
        this.setupFileWatcher();
        this.injectHMRClient();
    }
    
    createServer() {
        // Создание WebSocket сервера для HMR
        const WebSocket = require('ws');
        this.wss = new WebSocket.Server({ port: this.options.port });
        
        this.wss.on('connection', (ws) => {
            this.connections.add(ws);
            
            ws.on('close', () => {
                this.connections.delete(ws);
            });
            
            // Отправка сообщения о подключении
            ws.send(JSON.stringify({ type: 'connected' }));
        });
    }
    
    setupFileWatcher() {
        const chokidar = require('chokidar');
        
        this.fileWatcher = chokidar.watch('src/**/*', {
            ignored: /[\/\\]\./,
            persistent: true
        });
        
        this.fileWatcher.on('change', (path) => {
            this.handleFileChange(path);
        });
    }
    
    handleFileChange(filePath) {
        console.log(`Файл изменен: ${filePath}`);
        
        // Определение затронутых модулей
        const affectedModules = this.getAffectedModules(filePath);
        
        // Отправка обновлений клиентам
        this.broadcast({
            type: 'update',
            modules: affectedModules,
            timestamp: Date.now()
        });
    }
    
    getAffectedModules(filePath) {
        // В реальной реализации здесь будет анализ графа зависимостей
        return [{
            id: filePath,
            path: filePath,
            type: this.getFileType(filePath)
        }];
    }
    
    getFileType(filePath) {
        const ext = filePath.split('.').pop();
        return ['js', 'jsx', 'ts', 'tsx'].includes(ext) ? 'js' : 
               ['css', 'scss', 'sass', 'less'].includes(ext) ? 'css' : 'asset';
    }
    
    injectHMRClient() {
        // Код, который будет внедряться в бандл для HMR клиента
        const hmrClientCode = `
            (function() {
                if (module.hot) {
                    const ws = new WebSocket('ws://localhost:${this.options.port}');
                    
                    ws.onmessage = function(event) {
                        const data = JSON.parse(event.data);
                        
                        switch (data.type) {
                            case 'update':
                                handleUpdate(data.modules);
                                break;
                            case 'reload':
                                if (${this.options.reload}) {
                                    window.location.reload();
                                }
                                break;
                        }
                    };
                    
                    function handleUpdate(modules) {
                        modules.forEach(module => {
                            if (module.type === 'js') {
                                // Обработка JS модуля
                                handleJSUpdate(module);
                            } else if (module.type === 'css') {
                                // Обработка CSS модуля
                                handleCSSUpdate(module);
                            }
                        });
                    }
                    
                    function handleJSUpdate(module) {
                        // В реальной реализации здесь будет логика обновления JS модуля
                        console.log('Обновление JS модуля:', module.path);
                        
                        // Уведомление модуля о необходимости обновления
                        if (module.hot) {
                            module.hot.accept();
                        }
                    }
                    
                    function handleCSSUpdate(module) {
                        // Обновление CSS через замену link тега
                        const links = document.getElementsByTagName('link');
                        for (let link of links) {
                            if (link.href.includes(module.path)) {
                                const newLink = link.cloneNode();
                                newLink.href = module.path + '?t=' + Date.now();
                                link.parentNode.replaceChild(newLink, link);
                                break;
                            }
                        }
                    }
                    
                    // Проверка соединения
                    ws.onclose = function() {
                        console.log('Соединение HMR потеряно');
                    };
                }
            })();
        `;
        
        // Внедрение кода в бандл (реализация зависит от сборщика)
        return hmrClientCode;
    }
    
    broadcast(message) {
        const data = JSON.stringify(message);
        this.connections.forEach(ws => {
            if (ws.readyState === WebSocket.OPEN) {
                ws.send(data);
            }
        });
    }
    
    close() {
        if (this.wss) {
            this.wss.close();
        }
        if (this.fileWatcher) {
            this.fileWatcher.close();
        }
    }
}

// Использование HMR системы
const hmr = new HMRSystem({ port: 8080 });
hmr.init();
```

### Оптимизация бандлов

```javascript
// bundle-optimizer.js
class BundleOptimizer {
    constructor() {
        this.analyzer = null;
        this.optimizationStrategies = new Map();
        this.setupOptimizationStrategies();
    }
    
    setupOptimizationStrategies() {
        // Стратегия удаления неиспользуемого кода
        this.optimizationStrategies.set('tree-shaking', {
            name: 'Tree Shaking',
            apply: (bundle) => this.treeShaking(bundle),
            enabled: true
        });
        
        // Стратегия разделения кода
        this.optimizationStrategies.set('code-splitting', {
            name: 'Code Splitting',
            apply: (bundle) => this.codeSplitting(bundle),
            enabled: true
        });
        
        // Стратегия минификации
        this.optimizationStrategies.set('minification', {
            name: 'Minification',
            apply: (bundle) => this.minification(bundle),
            enabled: true
        });
        
        // Стратегия сжатия
        this.optimizationStrategies.set('compression', {
            name: 'Compression',
            apply: (bundle) => this.compression(bundle),
            enabled: true
        });
    }
    
    // Tree Shaking - удаление неиспользуемого кода
    treeShaking(bundle) {
        console.log('Применение Tree Shaking...');
        
        // В реальной реализации здесь будет анализ графа зависимостей
        // и удаление неиспользуемых импортов/экспортов
        
        return bundle; // Возвращаем оптимизированный бандл
    }
    
    // Code Splitting - разделение кода на чанки
    codeSplitting(bundle) {
        console.log('Применение Code Splitting...');
        
        // Логика разделения кода
        const chunks = [];
        let currentChunk = [];
        let currentSize = 0;
        const maxSize = 250 * 1024; // 250KB
        
        // Условная логика разделения (в реальности сложнее)
        bundle.modules.forEach(module => {
            const moduleSize = this.estimateModuleSize(module);
            
            if (currentSize + moduleSize > maxSize && currentChunk.length > 0) {
                chunks.push({
                    modules: currentChunk,
                    size: currentSize,
                    id: `chunk-${chunks.length}`
                });
                currentChunk = [];
                currentSize = 0;
            }
            
            currentChunk.push(module);
            currentSize += moduleSize;
        });
        
        if (currentChunk.length > 0) {
            chunks.push({
                modules: currentChunk,
                size: currentSize,
                id: `chunk-${chunks.length}`
            });
        }
        
        return { ...bundle, chunks };
    }
    
    // Минификация кода
    minification(bundle) {
        console.log('Применение минификации...');
        
        // Простая минификация (в реальности используйте Terser или UglifyJS)
        const minifyCode = (code) => {
            return code
                .replace(/\s+/g, ' ') // Замена множественных пробелов
                .replace(/;\s*/g, ';') // Удаление пробелов после точек с запятой
                .replace(/\/\/.*$/gm, '') // Удаление однострочных комментариев
                .replace(/\/\*[\s\S]*?\*\//g, ''); // Удаление многострочных комментариев
        };
        
        if (bundle.code) {
            bundle.code = minifyCode(bundle.code);
        }
        
        if (bundle.chunks) {
            bundle.chunks = bundle.chunks.map(chunk => ({
                ...chunk,
                code: minifyCode(chunk.code)
            }));
        }
        
        return bundle;
    }
    
    // Сжатие бандла
    compression(bundle) {
        console.log('Применение сжатия...');
        
        // В реальной реализации используйте gzip, brotli и т.д.
        // Здесь просто имитация
        
        const originalSize = this.estimateBundleSize(bundle);
        const compressedSize = originalSize * 0.7; // Условное сжатие на 30%
        
        console.log(`Сжатие: ${originalSize} -> ${compressedSize} байт`);
        
        return {
            ...bundle,
            compressed: true,
            originalSize,
            compressedSize
        };
    }
    
    // Основной метод оптимизации
    optimize(bundle, strategyNames = null) {
        console.log('Начало оптимизации бандла...');
        
        const strategies = strategyNames 
            ? strategyNames.map(name => this.optimizationStrategies.get(name))
            : Array.from(this.optimizationStrategies.values());
        
        let optimizedBundle = { ...bundle };
        
        for (const strategy of strategies) {
            if (strategy && strategy.enabled) {
                console.log(`Применение стратегии: ${strategy.name}`);
                optimizedBundle = strategy.apply(optimizedBundle);
            }
        }
        
        console.log('Оптимизация завершена!');
        return optimizedBundle;
    }
    
    estimateModuleSize(module) {
        // Оценка размера модуля (условная)
        return module.code ? module.code.length : 1024; // 1KB по умолчанию
    }
    
    estimateBundleSize(bundle) {
        // Оценка общего размера бандла
        if (bundle.code) {
            return bundle.code.length;
        }
        
        if (bundle.chunks) {
            return bundle.chunks.reduce((total, chunk) => 
                total + (chunk.code ? chunk.code.length : 0), 0
            );
        }
        
        return 0;
    }
    
    // Анализ бандла
    async analyze(bundle) {
        const analysis = {
            totalSize: this.estimateBundleSize(bundle),
            moduleCount: bundle.modules ? bundle.modules.length : 0,
            chunkCount: bundle.chunks ? bundle.chunks.length : 0,
            optimizationSuggestions: []
        };
        
        // Проверка на потенциальные проблемы
        if (analysis.totalSize > 1024 * 1024) { // > 1MB
            analysis.optimizationSuggestions.push('Бандл превышает 1MB, рекомендуется разделение кода');
        }
        
        if (analysis.moduleCount > 100) {
            analysis.optimizationSuggestions.push('Слишком много модулей в одном бандле');
        }
        
        return analysis;
    }
}

// Использование оптимизатора
const optimizer = new BundleOptimizer();

// Пример бандла для оптимизации
const sampleBundle = {
    modules: [
        { id: 1, code: 'function hello() { console.log("Hello"); }', size: 50 },
        { id: 2, code: 'function world() { console.log("World"); }', size: 50 }
    ],
    code: 'function hello() { console.log("Hello"); } function world() { console.log("World"); }'
};

// Оптимизация
const optimizedBundle = optimizer.optimize(sampleBundle, ['minification', 'tree-shaking']);
const analysis = await optimizer.analyze(optimizedBundle);

console.log('Анализ оптимизации:', analysis);
```

## Лучшие практики

### 1. Организация конфигурации

```javascript
// webpack.base.config.js
const path = require('path');

module.exports = {
    resolve: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
        alias: {
            '@': path.resolve(__dirname, 'src'),
            '@assets': path.resolve(__dirname, 'src/assets'),
            '@components': path.resolve(__dirname, 'src/components'),
            '@utils': path.resolve(__dirname, 'src/utils'),
            '@services': path.resolve(__dirname, 'src/services')
        }
    }
};
```

### 2. Управление зависимостями

```javascript
// Условная загрузка зависимостей
const loadLodash = async () => {
    if (typeof window !== 'undefined') {
        // Для браузера - использовать CDN или отложенно загрузить
        return import('https://cdn.skypack.dev/lodash');
    } else {
        // Для Node.js - использовать локальный импорт
        return import('lodash');
    }
};
```

## Безопасность

При использовании сборщиков важно учитывать безопасность:

```javascript
// secure-build-system.js
class SecureBuildSystem {
    constructor() {
        this.allowedModules = new Set();
        this.suspiciousPatterns = [
            /eval\s*\(/,
            /Function\s*\(/,
            /setTimeout\s*\(\s*["'].*["']/,
            /setInterval\s*\(\s*["'].*["']/,
            /document\.cookie/,
            /localStorage/,
            /sessionStorage/,
            /import\(/, // Динамические импорты могут быть потенциально небезопасны
        ];
    }
    
    // Проверка модуля на безопасность
    async scanModule(modulePath) {
        const fs = require('fs');
        
        try {
            const content = fs.readFileSync(modulePath, 'utf8');
            
            // Проверка на подозрительные паттерны
            for (const pattern of this.suspiciousPatterns) {
                if (pattern.test(content)) {
                    throw new Error(`Найден подозрительный код в ${modulePath}: ${pattern}`);
                }
            }
            
            console.log(`Модуль ${modulePath} безопасен`);
            return true;
        } catch (error) {
            console.error(`Ошибка сканирования модуля ${modulePath}:`, error.message);
            return false;
        }
    }
    
    // Проверка зависимостей
    async scanDependencies() {
        const fs = require('fs');
        const packageJson = JSON.parse(fs.readFileSync('package.json', 'utf8'));
        const dependencies = { ...packageJson.dependencies, ...packageJson.devDependencies };
        
        for (const [name, version] of Object.entries(dependencies)) {
            // В реальности здесь будет проверка через npm audit API или другие сервисы
            console.log(`Проверка зависимости: ${name}@${version}`);
            
            // Проверка на наличие уязвимостей (условно)
            if (this.isVulnerableDependency(name, version)) {
                console.warn(`Зависимость ${name} может содержать уязвимости`);
            }
        }
    }
    
    isVulnerableDependency(name, version) {
        // В реальности используйте npm audit, snyk, или другие инструменты
        const vulnerablePackages = [
            'event-stream', // Известная уязвимость
            'flatmap-stream' // Известная уязвимость
        ];
        
        return vulnerablePackages.includes(name);
    }
    
    // Безопасная загрузка плагинов
    async loadPlugin(pluginPath) {
        // Проверка пути к плагину
        if (!this.isValidPluginPath(pluginPath)) {
            throw new Error(`Недопустимый путь к плагину: ${pluginPath}`);
        }
        
        // Сканирование плагина
        await this.scanModule(pluginPath);
        
        // Загрузка плагина
        return require(pluginPath);
    }
    
    isValidPluginPath(pluginPath) {
        // Проверка, что путь находится в разрешенных директориях
        const allowedPaths = ['node_modules/', 'plugins/', 'build-plugins/'];
        return allowedPaths.some(allowed => pluginPath.startsWith(allowed));
    }
}

// Использование
const secureBuild = new SecureBuildSystem();
await secureBuild.scanDependencies();
```

## Теги

#javascript #bundlers #webpack #rollup #parcel #vite #esbuild #build-tools #modules #frontend