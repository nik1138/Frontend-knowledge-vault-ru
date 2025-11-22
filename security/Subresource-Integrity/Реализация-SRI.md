---
aliases: ["Реализация SRI", "Поддержка SRI", "Внедрение Subresource Integrity"]
tags: [security, sri, web-security, integrity, javascript, css]
created: 2025-11-18
updated: 2025-11-18
---

# Реализация SRI

## Обзор

Subresource Integrity (SRI) - это важная функция безопасности, позволяющая браузерам проверять целостность загружаемых ресурсов (скриптов и стилей), предотвращая потенциальные атаки, такие как вредоносное изменение файлов на CDN или атаки посредника (MITM). Реализация SRI включает генерацию хеш-значений для ресурсов и добавление атрибута `integrity` к тегам `<script>` и `<link>`.

## Основы SRI

### Как работает SRI

Subresource Integrity работает по следующему принципу:

1. Разработчик генерирует криптографический хеш для файла ресурса
2. Этот хеш добавляется в атрибут `integrity` тега
3. При загрузке ресурса браузер вычисляет хеш загруженного файла
4. Если хеш загруженного файла не совпадает с указанным в атрибуте `integrity`, браузер отклоняет ресурс

### Поддерживаемые алгоритмы

- **SHA-256**: Наиболее распространенный алгоритм
- **SHA-384**: Более стойкий алгоритм, рекомендуемый для критических приложений
- **SHA-512**: Наиболее стойкий алгоритм из поддерживаемых

## Генерация хеш-значений

### Использование OpenSSL

Для генерации хеша с помощью OpenSSL:

```bash
# Для SHA-256
openssl dgst -sha256 -binary FILENAME.js | openssl base64 -A

# Для SHA-384
openssl dgst -sha384 -binary FILENAME.js | openssl base64 -A

# Для SHA-512
openssl dgst -sha512 -binary FILENAME.js | openssl base64 -A
```

### Использование Node.js

```javascript
const crypto = require('crypto');
const fs = require('fs');

function generateSRIHash(filePath, algorithm = 'sha384') {
  const fileBuffer = fs.readFileSync(filePath);
  const hash = crypto.createHash(algorithm).update(fileBuffer).digest('base64');
  return `${algorithm}-${hash}`;
}

// Пример использования
const integrityHash = generateSRIHash('./script.js', 'sha384');
console.log(integrityHash);
```

### Онлайн-генераторы

Существуют онлайн-инструменты для генерации SRI хешей:

- [SRI Hash Generator](https://www.srihash.org/)
- Встроенные инструменты в сборщиках

## Реализация в HTML

### Для JavaScript файлов

```html
<script src="https://cdn.example.com/library.js" 
        integrity="sha384-abc123..." 
        crossorigin="anonymous">
</script>
```

### Для CSS файлов

```html
<link rel="stylesheet" href="https://cdn.example.com/style.css" 
      integrity="sha384-def456..." 
      crossorigin="anonymous">
```

### Важность атрибута crossorigin

Атрибут `crossorigin="anonymous"` необходим при использовании SRI с ресурсами, загружаемыми с других доменов (CDN). Без него браузер не будет проверять целостность ресурса.

## Интеграция в сборочные процессы

### Webpack

Для автоматической генерации SRI хешей в Webpack можно использовать плагин:

```javascript
// webpack.config.js
const SriPlugin = require('webpack-subresource-integrity');

module.exports = {
  output: {
    crossOriginLoading: 'anonymous'
  },
  plugins: [
    new SriPlugin({
      hashFuncNames: ['sha256', 'sha384']
    })
  ]
};
```

### Gulp

```javascript
// gulpfile.js
const gulp = require('gulp');
const sri = require('gulp-sri');

gulp.task('sri', function() {
  return gulp.src('dist/*.js')
    .pipe(sri())
    .pipe(gulp.dest('dist'));
});
```

### Rollup

```javascript
// rollup.config.js
import sri from 'rollup-plugin-sri';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    sri({
      algorithms: ['sha384']
    })
  ]
};
```

## Работа с CDN

### Публичные библиотеки

Многие публичные CDN предоставляют SRI хеши для популярных библиотек:

```html
<!-- jQuery с SRI -->
<script 
  src="https://code.jquery.com/jquery-3.6.0.min.js"
  integrity="sha384-/xJ6Jo+JZVY2p9jZmJ/k8WJbR8vL1EYkNQ0h46xHjc6jJ8Z7k8G+o32z51d"
  crossorigin="anonymous">
</script>

<!-- Bootstrap CSS с SRI -->
<link 
  rel="stylesheet" 
  href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
  integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
  crossorigin="anonymous">
```

### Собственные ресурсы

Для собственных ресурсов, размещенных на CDN:

1. Сгенерируйте хеш при сборке
2. Сохраните хеш в конфигурационный файл или переменную
3. Используйте шаблон для вставки хеша в HTML

## Автоматизация процесса

### Скрипт для генерации SRI

```bash
#!/bin/bash
# generate-sri.sh

function generate_sri() {
  local file=$1
  local algorithm=${2:-"sha384"}
  
  local hash=$(openssl dgst -$algorithm -binary "$file" | openssl base64 -A)
  echo "${algorithm}-${hash}"
}

# Пример использования
JS_HASH=$(generate_sri "dist/app.js")
CSS_HASH=$(generate_sri "dist/style.css")

echo "JS Integrity: $JS_HASH"
echo "CSS Integrity: $CSS_HASH"
```

### Интеграция с CI/CD

Пример шага в GitHub Actions:

```yaml
- name: Generate SRI hashes
  run: |
    npm run build
    JS_HASH=$(openssl dgst -sha384 -binary dist/bundle.js | openssl base64 -A)
    sed -i "s|INTEGRITY_PLACEHOLDER_JS|sha384-$JS_HASH|g" index.html
```

## Практические примеры

### Пример полного HTML с SRI

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Пример с SRI</title>
    
    <!-- Bootstrap CSS с SRI -->
    <link 
        rel="stylesheet" 
        href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"
        integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3"
        crossorigin="anonymous">
</head>
<body>
    <div class="container">
        <h1>Пример с SRI</h1>
    </div>

    <!-- jQuery с SRI -->
    <script 
        src="https://code.jquery.com/jquery-3.6.0.min.js"
        integrity="sha384-/xJ6Jo+JZVY2p9jZmJ/k8WJbR8vL1EYkNQ0h46xHjc6jJ8Z7k8G+o32z51d"
        crossorigin="anonymous">
    </script>
    
    <!-- Собственный скрипт с SRI -->
    <script 
        src="./app.js"
        integrity="sha384-GENERATED_HASH_HERE"
        crossorigin="anonymous">
    </script>
</body>
</html>
```

### Динамическая загрузка с SRI

```javascript
function loadScriptWithSRI(src, integrity, callback) {
  const script = document.createElement('script');
  script.src = src;
  script.integrity = integrity;
  script.crossOrigin = 'anonymous';
  
  if (callback) {
    script.onload = callback;
  }
  
  document.head.appendChild(script);
}

// Использование
loadScriptWithSRI(
  'https://cdn.example.com/library.js',
  'sha384-expected-hash-value',
  () => console.log('Скрипт загружен с проверкой целостности')
);
```

## Тестирование SRI

### Проверка в браузере

1. Откройте DevTools
2. Перейдите во вкладку Network
3. Найдите ресурс с атрибутом integrity
4. Убедитесь, что загрузка прошла успешно
5. Проверьте консоль на наличие ошибок SRI

### Тестирование с намеренным нарушением

Для тестирования можно временно изменить значение integrity на неверное:

```html
<!-- Это вызовет ошибку в браузере -->
<script 
  src="https://cdn.example.com/library.js"
  integrity="sha384-invalid-hash-value"
  crossorigin="anonymous">
</script>
```

## Заключение

Реализация Subresource Integrity - важный шаг в обеспечении безопасности веб-приложения. Она защищает пользователей от загрузки вредоносных скриптов в случае компрометации CDN или атак посредника. Автоматизация генерации и вставки SRI хешей в процессе сборки делает внедрение этой технологии проще и надежнее.

> [!note] Примечание
> SRI не защищает от всех видов атак, но является важной частью комплексного подхода к безопасности.

> [!tip] Совет
> Внедряйте SRI постепенно, начиная с критически важных библиотек и ресурсов.

Связанные темы: [[Проверка-целостности]], [[Безопасность-CDN]], [[Лучшие-практики-SRI]]