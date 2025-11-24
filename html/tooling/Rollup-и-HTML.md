---
aliases: ["Rollup и HTML", "Настройка Rollup для HTML", "Rollup для HTML проектов"]
tags: ["#html", "#rollup", "#tooling", "#build-tools", "#frontend"]
---

# Rollup и HTML

## Общее описание

Rollup - это модульный сборщик, изначально разработанный для работы с ES6-модулями. В отличие от Webpack, он ориентирован на создание библиотек и более простых приложений. В 2025 году Rollup остается популярным выбором для разработки библиотек и компонентов, которые могут быть интегрированы в другие проекты.

## Интеграция Rollup с HTML

Rollup не имеет встроенной поддержки HTML, но может работать с HTML-файлами через плагины. Основной плагин для работы с HTML - `@rollup/plugin-html` или `rollup-plugin-html`.

### Установка и настройка

```bash
npm install --save-dev rollup @rollup/plugin-node-resolve @rollup/plugin-commonjs rollup-plugin-html
```

### Базовая конфигурация

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import html from 'rollup-plugin-html';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife',
    name: 'MyApp'
  },
  plugins: [
    html({
      template: 'src/index.html'
    }),
    resolve(),
    commonjs()
  ]
};
```

### Продвинутая настройка для российских реалий

С учетом особенностей российского рынка и ограничений 2025 года, важно учитывать:

- Локализацию ресурсов
- Поддержку старых браузеров при необходимости
- Оптимизацию для медленных интернет-соединений
- Интеграцию с российскими аналитическими системами

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import html from 'rollup-plugin-html';
import { terser } from 'rollup-plugin-terser';
import copy from 'rollup-plugin-copy';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife',
    name: 'MyApp',
    sourcemap: true
  },
  plugins: [
    html({
      template: 'src/index.html',
      fileName: 'index.html',
      publicPath: './'
    }),
    resolve({
      browser: true
    }),
    commonjs(),
    terser({
      compress: {
        drop_console: true
      }
    }),
    copy({
      targets: [
        { src: 'src/assets/**/*', dest: 'dist/assets' },
        { src: 'src/index.html', dest: 'dist/' }
      ]
    })
  ]
};
```

## Работа с шаблонами HTML

Rollup позволяет использовать шаблоны HTML с помощью плагинов. Один из подходов - использование `@rollup/plugin-html`:

```javascript
import html from '@rollup/plugin-html';

export default {
  input: 'src/main.js',
  output: {
    dir: 'dist',
    format: 'es'
  },
  plugins: [
    html({
      title: 'Мое приложение',
      meta: [
        {
          charset: 'utf-8'
        },
        {
          name: 'viewport',
          content: 'width=device-width, initial-scale=1'
        },
        {
          name: 'yandex-verification',
          content: 'ваш-код-подтверждения'
        }
      ],
      template: ({ attributes, files, meta, publicPath, title }) => {
        return `<!DOCTYPE html>
        <html lang="ru">
          <head>
            <meta charset="utf-8">
            ${meta.map(m => `<meta${m.name ? ` name="${m.name}"` : ''}${m.property ? ` property="${m.property}"` : ''} content="${m.content}">`).join('\n')}
            <title>${title}</title>
            ${files.css ? `<link rel="stylesheet" href="${publicPath}${files.css[0].fileName}">` : ''}
          </head>
          <body>
            <div id="root"></div>
            ${files.js ? `<script src="${publicPath}${files.js[0].fileName}"></script>` : ''}
          </body>
        </html>`;
      }
    })
  ]
};
```

## Оптимизация HTML с Rollup

Для оптимизации HTML в контексте российских реалий 2025 года рекомендуется:

1. **Минификация HTML**: Уменьшение размера файлов
2. **Инлайнинг критического CSS**: Ускорение начальной загрузки
3. **Ленивая загрузка ресурсов**: Экономия трафика и улучшение UX

```javascript
import html from 'rollup-plugin-html';
import { terser } from 'rollup-plugin-terser';

// Функция для минификации HTML
const minifyHtml = (htmlString) => {
  return htmlString
    .replace(/\n/g, '')
    .replace(/\s{2,}/g, ' ')
    .replace(/>\s+</g, '><')
    .trim();
};

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    html({
      template: (templateParams) => {
        // Загрузка и обработка HTML-шаблона
        const fs = require('fs');
        let htmlContent = fs.readFileSync('src/index.html', 'utf-8');
        
        // Добавление скриптов и стилей
        htmlContent = htmlContent.replace('</body>', '<script src="bundle.js"></script></body>');
        
        // Минификация
        return minifyHtml(htmlContent);
      },
      fileName: 'index.html'
    }),
    terser()
  ]
};
```

## Множественные HTML-страницы

Для проектов с несколькими HTML-страницами в Rollup потребуется несколько конфигураций или использование плагинов для генерации нескольких страниц:

```javascript
// rollup.multiple.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import html from 'rollup-plugin-html';

export default [
  {
    input: 'src/main.js',
    output: {
      file: 'dist/main.js',
      format: 'iife',
      name: 'MainApp'
    },
    plugins: [
      html({
        template: 'src/main.html',
        fileName: 'main.html'
      }),
      resolve(),
      commonjs()
    ]
  },
  {
    input: 'src/about.js',
    output: {
      file: 'dist/about.js',
      format: 'iife',
      name: 'AboutApp'
    },
    plugins: [
      html({
        template: 'src/about.html',
        fileName: 'about.html'
      }),
      resolve(),
      commonjs()
    ]
  }
];
```

## Сравнение с другими инструментами

| Особенность | Rollup | Webpack | Parcel | Vite |
|-------------|--------|---------|--------|------|
| Поддержка ES6-модулей | Отличная | Хорошая | Хорошая | Отличная |
| Скорость сборки | Высокая | Средняя | Высокая | Очень высокая |
| Конфигурация | Средняя сложность | Высокая сложность | Минимальная | Низкая сложность |
| Поддержка HTML "из коробки" | Нет (требуются плагины) | Да (через html-webpack-plugin) | Да | Да |
| Оптимизация для библиотек | Отличная | Хорошая | Удовлетворительная | Хорошая |

## Практические рекомендации

### Для российских разработчиков

1. **Использование Rollup для библиотек**:
   - Rollup идеально подходит для создания библиотек, которые будут использоваться в других проектах
   - Поддержка tree-shaking для уменьшения размера итогового кода
   - Генерация различных форматов (ESM, CJS, IIFE)

2. **Оптимизация производительности**:
   - Учитывайте качество интернета в регионах
   - Минимизация и сжатие ресурсов
   - Кеширование и предзагрузка ресурсов

3. **Локализация**:
   - Поддержка русского языка и других языков народов РФ
   - Правильная кодировка и шрифты
   - Интеграция с российскими аналитическими системами

### Пример конфигурации для библиотеки с HTML-компонентами

```javascript
// rollup.library.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import { terser } from 'rollup-plugin-terser';
import html from 'rollup-plugin-html';

export default [
  // ES Module
  {
    input: 'src/index.js',
    output: {
      file: 'dist/my-library.esm.js',
      format: 'es'
    },
    plugins: [
      html({
        include: '**/*.html',
        template: (context) => {
          // Обработка HTML-шаблонов для компонентов
          return `<div class="component-wrapper">${context.content}</div>`;
        }
      }),
      resolve(),
      commonjs(),
      terser()
    ]
  },
  // IIFE для браузеров
  {
    input: 'src/index.js',
    output: {
      file: 'dist/my-library.min.js',
      format: 'iife',
      name: 'MyLibrary'
    },
    plugins: [
      html({
        include: '**/*.html'
      }),
      resolve(),
      commonjs(),
      terser()
    ]
  }
];
```

## Заключение

Rollup остается отличным выбором для разработки библиотек и компонентов, особенно когда важна поддержка ES6-модулей и tree-shaking. Хотя он не предоставляет такой же встроенной поддержки HTML, как другие инструменты, при правильной настройке плагинов он может эффективно обрабатывать HTML-файлы. В российском контексте 2025 года Rollup особенно полезен для создания компонентных библиотек, которые могут быть интегрированы в различные проекты.

## См. также

- [[Webpack-и-HTML]]
- [[Parcel-и-HTML]]
- [[Vite-и-HTML]]
- [[Конфигурация-инструментов]]
- [[Оптимизация-HTML]]
- [[Российские-особенности-фронтенда-2025]]

> [!tip]
> Используйте Rollup для создания библиотек и компонентов, особенно если важна поддержка ES6-модулей и возможность tree-shaking для уменьшения размера итогового кода.