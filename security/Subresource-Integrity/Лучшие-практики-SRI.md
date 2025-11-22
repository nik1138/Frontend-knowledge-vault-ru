---
aliases: ["Лучшие практики SRI", "Рекомендации по SRI", "Оптимизация Subresource Integrity"]
tags: [security, sri, best-practices, web-security, optimization]
created: 2025-11-18
updated: 2025-11-18
---

# Лучшие практики SRI

## Обзор

Лучшие практики Subresource Integrity (SRI) представляют собой набор рекомендаций и подходов, направленных на максимальную эффективность и безопасность при реализации проверки целостности ресурсов. Правильное применение этих практик позволяет обеспечить надежную защиту от компрометации CDN и других внешних источников, при этом минимизируя влияние на производительность и удобство сопровождения.

## Выбор алгоритма хеширования

### Рекомендуемые алгоритмы

#### SHA-384 (рекомендуется)

- **Баланс безопасности и совместимости**: Достаточная стойкость при хорошей поддержке
- **Разумная длина хеша**: Меньше, чем SHA-512, но безопаснее SHA-256
- **Соответствие стандартам**: Рекомендуется в большинстве руководств по безопасности

```html
<script src="https://cdn.example.com/library.js"
        integrity="sha384-MzWYY2K+A2LhHvdSsQ5DbwFhFhN8t1+N56X8qe27QY26a823L7k8G+o32z51d"
        crossorigin="anonymous">
</script>
```

#### SHA-512 (для критических приложений)

- **Максимальная стойкость**: Наивысший уровень защиты
- **Более длинные хеши**: Значительно длиннее значения, увеличивает размер HTML
- **Использование для чувствительных данных**: Рекомендуется для финансовых и медицинских приложений

#### Избегайте SHA-1

- **Криптографически небезопасен**: Уязвим к коллизиям
- **Не поддерживается браузерами**: Современные браузеры не поддерживают SHA-1 для SRI

### Множественные алгоритмы

Можно указать несколько хешей, чтобы обеспечить совместимость:

```html
<script src="https://cdn.example.com/library.js"
        integrity="sha384-first-hash sha512-second-hash"
        crossorigin="anonymous">
</script>
```

Браузер будет использовать первый совпадающий хеш.

## Автоматизация процесса SRI

### Интеграция в сборочные системы

#### Webpack

```javascript
// webpack.config.js
const SriPlugin = require('webpack-subresource-integrity');

module.exports = {
  output: {
    crossOriginLoading: 'anonymous'
  },
  plugins: [
    new SriPlugin({
      hashFuncNames: ['sha384'],
      enabled: process.env.NODE_ENV === 'production'
    })
  ]
};
```

#### Gulp

```javascript
// gulpfile.js
const gulp = require('gulp');
const sri = require('gulp-sri');
const htmlReplace = require('gulp-html-replace');

gulp.task('sri', function() {
  return gulp.src('dist/*.js')
    .pipe(sri({ algorithm: 'sha384' }))
    .pipe(gulp.dest('dist'));
});

gulp.task('html-sri', function() {
  // Замена плейсхолдеров в HTML файле сгенерированными хешами
  return gulp.src('src/index.html')
    .pipe(htmlReplace({
      'js-integrity': 'sha384-generated-hash-value',
      'css-integrity': 'sha384-another-generated-hash-value'
    }))
    .pipe(gulp.dest('dist'));
});
```

#### NPM Scripts

```json
{
  "scripts": {
    "build": "webpack --mode=production",
    "generate-sri": "node scripts/generate-sri.js",
    "postbuild": "npm run generate-sri"
  }
}
```

### Автоматическая генерация хешей

```javascript
// scripts/generate-sri.js
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

function generateSRIHash(filePath) {
  const fileBuffer = fs.readFileSync(filePath);
  const hash = crypto.createHash('sha384').update(fileBuffer).digest('base64');
  return `sha384-${hash}`;
}

function updateHTMLWithSRI(htmlPath, jsPath, cssPath) {
  let html = fs.readFileSync(htmlPath, 'utf8');
  
  if (jsPath) {
    const jsHash = generateSRIHash(jsPath);
    html = html.replace(/js-integrity-placeholder/g, jsHash);
  }
  
  if (cssPath) {
    const cssHash = generateSRIHash(cssPath);
    html = html.replace(/css-integrity-placeholder/g, cssHash);
  }
  
  fs.writeFileSync(htmlPath, html);
}

// Пример использования
updateHTMLWithSRI('./dist/index.html', './dist/bundle.js', './dist/style.css');
```

## Управление зависимостями с SRI

### Пакетные менеджеры

#### Использование с npm/yarn

```json
{
  "scripts": {
    "postinstall": "node scripts/generate-sri-for-deps.js"
  }
}
```

```javascript
// scripts/generate-sri-for-deps.js
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

// Генерация SRI для установленных зависимостей
function generateSRIForDependencies() {
  const nodeModulesPath = './node_modules';
  const dependencies = [
    { name: 'jquery', file: 'dist/jquery.min.js' },
    { name: 'bootstrap', file: 'dist/js/bootstrap.min.js' }
  ];
  
  const sriMap = {};
  
  dependencies.forEach(dep => {
    const filePath = path.join(nodeModulesPath, dep.name, dep.file);
    if (fs.existsSync(filePath)) {
      const fileBuffer = fs.readFileSync(filePath);
      const hash = crypto.createHash('sha384').update(fileBuffer).digest('base64');
      sriMap[`${dep.name}/${dep.file}`] = `sha384-${hash}`;
    }
  });
  
  // Сохранение в файл для дальнейшего использования
  fs.writeFileSync('./sri-hashes.json', JSON.stringify(sriMap, null, 2));
}

generateSRIForDependencies();
```

### CDN-специфичные хеши

Для публичных библиотек часто уже существуют готовые хеши:

```javascript
// sri-hashes.json
{
  "jquery@3.6.0": "sha384-/xJ6Jo+JZVY2p9jZmJ/k8WJbR8vL1EYkNQ0h46xHjc6jJ8Z7k8G+o32z51d",
  "bootstrap@5.1.3": "sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
}
```

## Оптимизация производительности

### Минимизация влияния на загрузку

#### Асинхронная загрузка с SRI

```html
<!-- Асинхронная загрузка скрипта с проверкой целостности -->
<script src="https://cdn.example.com/library.js"
        integrity="sha384-expected-hash-value"
        crossorigin="anonymous"
        async>
</script>

<!-- Отложенная загрузка -->
<script src="https://cdn.example.com/library.js"
        integrity="sha384-expected-hash-value"
        crossorigin="anonymous"
        defer>
</script>
```

#### Приоритизация критических ресурсов

```html
<!-- Критические стили с высоким приоритетом -->
<link rel="preload" href="https://cdn.example.com/critical.css" 
      as="style" 
      integrity="sha384-critical-css-hash"
      crossorigin="anonymous">

<link rel="stylesheet" href="https://cdn.example.com/critical.css"
      integrity="sha384-critical-css-hash"
      crossorigin="anonymous">
```

### Кэширование хешей

Хранение сгенерированных хешей в кэше сборки:

```javascript
// cache-sri.js
const fs = require('fs');
const crypto = require('crypto');

class SRIHashCache {
  constructor(cacheFile = './sri-cache.json') {
    this.cacheFile = cacheFile;
    this.cache = this.loadCache();
  }
  
  loadCache() {
    try {
      return JSON.parse(fs.readFileSync(this.cacheFile, 'utf8'));
    } catch (e) {
      return {};
    }
  }
  
  saveCache() {
    fs.writeFileSync(this.cacheFile, JSON.stringify(this.cache, null, 2));
  }
  
  getHash(filePath) {
    const fileStats = fs.statSync(filePath);
    const cacheKey = `${filePath}-${fileStats.mtimeMs}`;
    
    if (this.cache[cacheKey]) {
      return this.cache[cacheKey];
    }
    
    const fileBuffer = fs.readFileSync(filePath);
    const hash = crypto.createHash('sha384').update(fileBuffer).digest('base64');
    const integrity = `sha384-${hash}`;
    
    this.cache[cacheKey] = integrity;
    this.saveCache();
    
    return integrity;
  }
}

module.exports = SRIHashCache;
```

## Обработка ошибок и резервные варианты

### Резервные стратегии

#### Локальные резервные копии

```html
<script>
// Загрузка с CDN с резервной стратегией
function loadScriptWithFallback(url, integrity, fallbackUrl, callback) {
  const script = document.createElement('script');
  script.src = url;
  script.integrity = integrity;
  script.crossOrigin = 'anonymous';
  
  script.onload = callback;
  script.onerror = function() {
    console.warn(`Загрузка с ${url} не удалась, используем резервный вариант`);
    const fallbackScript = document.createElement('script');
    fallbackScript.src = fallbackUrl;
    fallbackScript.onload = callback;
    document.head.appendChild(fallbackScript);
  };
  
  document.head.appendChild(script);
}

// Использование
loadScriptWithFallback(
  'https://cdn.example.com/library.js',
  'sha384-expected-hash',
  '/local-fallback/library.js',
  () => console.log('Скрипт успешно загружен')
);
</script>
```

#### Проверка загрузки

```javascript
// Проверка успешной загрузки библиотеки
function waitForLibrary(libName, timeout = 5000) {
  return new Promise((resolve, reject) => {
    const startTime = Date.now();
    
    function check() {
      if (window[libName] || (libName === 'jQuery' && window.$)) {
        resolve(window[libName]);
      } else if (Date.now() - startTime > timeout) {
        reject(new Error(`${libName} не загрузилась в течение ${timeout}мс`));
      } else {
        setTimeout(check, 100);
      }
    }
    
    check();
  });
}

// Использование
waitForLibrary('jQuery')
  .then($ => console.log('jQuery загружен'))
  .catch(err => console.error('Ошибка загрузки jQuery:', err));
```

### Мониторинг сбоев SRI

```javascript
// Отслеживание сбоев проверки целостности
window.addEventListener('error', function(e) {
  if (e.target !== window && e.target.integrity) {
    console.error('Ошибка SRI:', {
      src: e.target.src,
      integrity: e.target.integrity,
      tagName: e.target.tagName
    });
    
    // Отправка отчета о сбое в систему мониторинга
    reportSRIFailure(e.target.src, e.target.integrity);
  }
}, true);

function reportSRIFailure(resourceUrl, integrityValue) {
  // Отправка отчета в систему мониторинга
  fetch('/api/sri-failures', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      url: resourceUrl,
      integrity: integrityValue,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent
    })
  });
}
```

## Совместимость и поддержка

### Работа с устаревшими браузерами

```html
<script>
// Проверка поддержки SRI и условная загрузка
function supportsSRI() {
  return 'integrity' in document.createElement('script');
}

if (supportsSRI()) {
  // Загрузка с SRI для современных браузеров
  document.write(`
    <script src="https://cdn.example.com/library.js"
            integrity="sha384-supported-hash"
            crossorigin="anonymous"><\/script>
  `);
} else {
  // Загрузка без SRI для старых браузеров
  document.write(`
    <script src="https://cdn.example.com/library.js"><\/script>
  `);
}
</script>
```

### Гибридные стратегии

```javascript
// Гибридная стратегия загрузки
class HybridResourceLoader {
  constructor(options = {}) {
    this.useSRI = options.useSRI !== false; // по умолчанию true
    this.fallbackTimeout = options.fallbackTimeout || 3000;
  }
  
  async loadScript(src, integrity) {
    if (!this.useSRI || !supportsSRI()) {
      // Простая загрузка для браузеров без поддержки SRI
      return this.loadSimpleScript(src);
    }
    
    return this.loadScriptWithSRI(src, integrity);
  }
  
  loadSimpleScript(src) {
    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = src;
      script.onload = resolve;
      script.onerror = reject;
      document.head.appendChild(script);
    });
  }
  
  loadScriptWithSRI(src, integrity) {
    return new Promise((resolve, reject) => {
      const script = document.createElement('script');
      script.src = src;
      script.integrity = integrity;
      script.crossOrigin = 'anonymous';
      
      // Таймаут для резервной загрузки
      const fallbackTimer = setTimeout(() => {
        console.warn('Таймаут SRI, используем резервный вариант');
        this.loadSimpleScript(src).then(resolve).catch(reject);
      }, this.fallbackTimeout);
      
      script.onload = () => {
        clearTimeout(fallbackTimer);
        resolve();
      };
      
      script.onerror = (error) => {
        clearTimeout(fallbackTimer);
        reject(error);
      };
      
      document.head.appendChild(script);
    });
  }
}

// Использование
const loader = new HybridResourceLoader();
loader.loadScript(
  'https://cdn.example.com/library.js',
  'sha384-expected-hash'
).then(() => {
  console.log('Скрипт загружен');
}).catch(err => {
  console.error('Ошибка загрузки скрипта:', err);
});
```

## Мониторинг и аудит

### Автоматический аудит SRI

```javascript
// Скрипт для аудита SRI на странице
function auditSRI() {
  const scripts = document.querySelectorAll('script[src][integrity]');
  const links = document.querySelectorAll('link[rel="stylesheet"][href][integrity]');
  
  const report = {
    scripts: [],
    stylesheets: [],
    total: 0,
    missing: 0
  };
  
  scripts.forEach(script => {
    report.scripts.push({
      src: script.src,
      integrity: script.integrity,
      hasCrossOrigin: !!script.crossOrigin
    });
    report.total++;
    if (!script.integrity) report.missing++;
  });
  
  links.forEach(link => {
    report.stylesheets.push({
      href: link.href,
      integrity: link.integrity,
      hasCrossOrigin: !!link.crossOrigin
    });
    report.total++;
    if (!link.integrity) report.missing++;
  });
  
  return report;
}

// Использование
const sriAudit = auditSRI();
console.table(sriAudit);
```

### CI/CD интеграция

```yaml
# .github/workflows/sri-check.yml
name: SRI Validation
on: [push, pull_request]

jobs:
  sri-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      
      - name: Install dependencies
        run: npm install
      
      - name: Build project
        run: npm run build
      
      - name: Validate SRI hashes
        run: |
          node scripts/validate-sri.js
```

## Заключение

Лучшие практики SRI позволяют эффективно использовать преимущества Subresource Integrity для повышения безопасности веб-приложений. Автоматизация процесса генерации хешей, правильная обработка ошибок, использование резервных стратегий и регулярный мониторинг обеспечивают надежную защиту от компрометации внешних ресурсов при минимальном влиянии на производительность.

Ключевые моменты для успешной реализации:

1. Использовать SHA-384 как баланс безопасности и производительности
2. Автоматизировать генерацию хешей в процессе сборки
3. Обеспечить резервные стратегии для сбоев
4. Мониторить и аудировать использование SRI
5. Обеспечить совместимость с различными браузерами

> [!tip] Совет
> Интегрируйте проверку SRI в процесс CI/CD для автоматического обнаружения проблем.

> [!warning] Важно
> Не забывайте обновлять хеши при обновлении внешних библиотек, иначе ресурсы не будут загружаться.

Связанные темы: [[Реализация-SRI]], [[Проверка-целостности]], [[Безопасность-CDN]]