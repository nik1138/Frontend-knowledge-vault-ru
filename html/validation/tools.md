# HTML Валидация: Инструменты проверки

HTML валидация - это процесс проверки HTML кода на соответствие официальным стандартам. Существует множество инструментов, которые помогают разработчикам проверять и поддерживать валидность HTML кода.

## Онлайн инструменты валидации

### W3C Markup Validator

Официальный инструмент от W3C для проверки HTML валидности:

```html
<!-- Пример валидного HTML документа для проверки -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Валидный документ</title>
</head>
<body>
    <header>
        <h1>Валидный HTML документ</h1>
    </header>
    
    <main>
        <p>Этот документ соответствует стандартам HTML5.</p>
        
        <figure>
            <img src="example.jpg" alt="Пример изображения">
            <figcaption>Подпись к изображению</figcaption>
        </figure>
    </main>
    
    <footer>
        <p>Подвал документа</p>
    </footer>
</body>
</html>
```

#### Использование W3C Validator:
1. Перейдите на [validator.w3.org](https://validator.w3.org)
2. Вставьте HTML код, URL страницы или загрузите файл
3. Получите подробный отчет об ошибках и предупреждениях

### HTMLHint

Инструмент для проверки HTML кода с настраиваемыми правилами:

```json
{
  "tagname-lowercase": true,
  "attr-lowercase": true,
  "attr-value-double-quotes": true,
  "id-class-value": "dash",
  "src-require": true,
  "alt-require": true,
  "doctype-first": true,
  "tag-pair": true,
  "spec-char-escape": true,
  "id-unique": true,
  "title-require": true
}
```

#### Установка и использование:

```bash
# Установка HTMLHint
npm install -g htmlhint

# Проверка HTML файла
htmlhint index.html

# Проверка с конфигурацией
htmlhint index.html --config .htmlhintrc

# Проверка всех HTML файлов в директории
htmlhint "**/*.html"
```

## Инструменты командной строки

### HTML Tidy

HTML Tidy - мощный инструмент для проверки и форматирования HTML:

```bash
# Установка HTML Tidy
sudo apt-get install tidy  # Ubuntu/Debian
brew install tidy-html5    # macOS

# Проверка HTML файла
tidy -e index.html

# Проверка с подробным выводом
tidy -f errors.txt -errors index.html

# Форматирование и корректировка HTML
tidy -indent -wrap 0 -modify index.html

# Проверка с настройками
tidy -config tidy.conf index.html
```

#### Пример конфигурационного файла (tidy.conf):

```
# Настройки для строгой валидации HTML5
doctype: html5
input-xml: no
output-xml: no
add-xml-decl: no
indent: auto
indent-attributes: no
indent-spaces: 2
wrap: 80
markup: yes
show-warnings: yes
force-output: no
quiet: no
tidy-mark: no
```

### VNU (Validator.nu)

Сервис валидации HTML от Nu Html Checker:

```bash
# Установка VNU
npm install -g @html-validate/vnu-jar

# Проверка файла
vnu index.html

# Проверка с игнорированием определенных ошибок
vnu --errors-only index.html

# Проверка нескольких файлов
vnu **/*.html
```

## Интеграция в редакторы кода

### VS Code

Расширения для проверки HTML в VS Code:

```json
// settings.json для VS Code
{
  "html.suggest.html5": true,
  "html.autoClosingTags": true,
  "html.autoCreateQuotes": true,
  "html.completion.attributeDefaultValue": "doublequotes",
  
  // Настройки для Emmet
  "emmet.includeLanguages": {
    "html": "html"
  },
  
  // Проверка при сохранении
  "html.validate.scripts": true,
  "html.validate.styles": true
}
```

#### Популярные расширения:
- **HTMLHint** - для проверки по правилам HTMLHint
- **HTML CSS Support** - для поддержки CSS в HTML
- **Auto Rename Tag** - автоматическое переименование парных тегов
- **Prettier** - для форматирования HTML

### Sublime Text

Конфигурация для Sublime Text:

```json
{
  "extensions": ["html", "htm", "shtml", "xhtml"],
  "rules": {
    "tagname-lowercase": true,
    "attr-lowercase": true,
    "attr-value-double-quotes": true,
    "doctype-first": true,
    "tag-pair": true,
    "spec-char-escape": true
  }
}
```

## Интеграция в CI/CD

### GitHub Actions

```yaml
# .github/workflows/html-validation.yml
name: HTML Validation
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install validation tools
        run: |
          npm install -g html-validate
          npm install -g htmlhint
          
      - name: Validate HTML files
        run: |
          html-validate "**/*.html"
          htmlhint "**/*.html"
```

### Git Hooks

#### Pre-commit hook с использованием lint-staged:

```javascript
// package.json
{
  "scripts": {
    "validate:html": "html-validate",
    "lint:html": "htmlhint"
  },
  "lint-staged": {
    "*.html": [
      "html-validate",
      "htmlhint",
      "git add"
    ]
  }
}
```

#### .lintstagedrc.js:

```javascript
module.exports = {
  '*.html': [
    'html-validate',
    'htmlhint',
    'git add'
  ]
};
```

## Автоматизированные инструменты

### HTML Validator (Node.js)

```javascript
// validate-html.js
const fs = require('fs');
const { validate } = require('html-validator');

async function validateHTMLFile(filePath) {
    const htmlContent = fs.readFileSync(filePath, 'utf8');
    
    try {
        const result = await validate({
            data: htmlContent,
            format: 'text'
        });
        
        if (result.includes('Error:')) {
            console.log(`❌ Ошибки валидации в ${filePath}:`);
            console.log(result);
            return false;
        } else {
            console.log(`✅ ${filePath} - валидный HTML`);
            return true;
        }
    } catch (error) {
        console.error(`Ошибка проверки ${filePath}:`, error);
        return false;
    }
}

// Проверка конкретного файла
validateHTMLFile('index.html');
```

### Пакетный скрипт проверки

```javascript
// batch-validate.js
const fs = require('fs');
const path = require('path');
const { validate } = require('html-validator');

async function validateAllHTMLFiles(dirPath) {
    const files = fs.readdirSync(dirPath);
    let allValid = true;
    
    for (const file of files) {
        const filePath = path.join(dirPath, file);
        const stat = fs.statSync(filePath);
        
        if (stat.isDirectory()) {
            const subDirValid = await validateAllHTMLFiles(filePath);
            allValid = allValid && subDirValid;
        } else if (file.endsWith('.html')) {
            const isValid = await validateHTMLFile(filePath);
            allValid = allValid && isValid;
        }
    }
    
    return allValid;
}

async function validateHTMLFile(filePath) {
    const htmlContent = fs.readFileSync(filePath, 'utf8');
    
    try {
        const result = await validate({
            data: htmlContent,
            format: 'json'
        });
        
        if (result.messages && result.messages.length > 0) {
            const errors = result.messages.filter(msg => msg.type === 'error');
            if (errors.length > 0) {
                console.log(`❌ Ошибки в ${filePath}:`);
                errors.forEach(error => {
                    console.log(`  Строка ${error.lastLine}, Колонка ${error.lastColumn}: ${error.message}`);
                });
                return false;
            }
        }
        
        console.log(`✅ ${filePath} - валидный HTML`);
        return true;
    } catch (error) {
        console.error(`Ошибка проверки ${filePath}:`, error);
        return false;
    }
}

// Запуск проверки для всей директории
validateAllHTMLFiles('./html')
    .then(allValid => {
        if (allValid) {
            console.log('\n🎉 Все HTML файлы валидны!');
            process.exit(0);
        } else {
            console.log('\n❌ Найдены ошибки валидации');
            process.exit(1);
        }
    });
```

## Инструменты для специфических задач

### Проверка доступности

```javascript
// Использование axe-core для проверки доступности
const axe = require('axe-core');

// Проверка HTML на доступность
axe.run(document, {
    runOnly: {
        type: 'tag',
        values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']
    }
}, (err, results) => {
    if (err) {
        console.error('Ошибка проверки доступности:', err);
        return;
    }
    
    if (results.violations.length) {
        console.log('Найдены проблемы доступности:');
        results.violations.forEach(violation => {
            console.log(`${violation.help}: ${violation.nodes.length} элементов`);
        });
    } else {
        console.log('Доступность в порядке!');
    }
});
```

### Проверка SEO

```javascript
// Простая проверка основных SEO элементов
function checkSEO(htmlString) {
    const parser = new DOMParser();
    const doc = parser.parseFromString(htmlString, 'text/html');
    
    const issues = [];
    
    // Проверка заголовка
    const title = doc.querySelector('title');
    if (!title || !title.textContent.trim()) {
        issues.push('Отсутствует или пустой заголовок страницы');
    } else if (title.textContent.length > 60) {
        issues.push('Заголовок слишком длинный (>60 символов)');
    }
    
    // Проверка описания
    const description = doc.querySelector('meta[name="description"]');
    if (!description || !description.getAttribute('content')) {
        issues.push('Отсутствует мета-описание');
    } else if (description.getAttribute('content').length > 160) {
        issues.push('Мета-описание слишком длинное (>160 символов)');
    }
    
    // Проверка изображений
    const images = doc.querySelectorAll('img');
    images.forEach((img, index) => {
        if (!img.hasAttribute('alt')) {
            issues.push(`Изображение ${index + 1} не имеет alt атрибута`);
        }
    });
    
    return issues;
}
```

## Настройка проекта для автоматической проверки

### Пример конфигурации (package.json):

```json
{
  "name": "html-validation-project",
  "version": "1.0.0",
  "scripts": {
    "validate": "npm run validate:html && npm run validate:accessibility",
    "validate:html": "html-validate \"./html/**/*.html\"",
    "validate:accessibility": "pa11y \"./html/**/*.html\"",
    "lint": "htmlhint \"./html/**/*.html\"",
    "format": "prettier --write \"./html/**/*.html\"",
    "test": "npm run validate && npm run lint"
  },
  "devDependencies": {
    "html-validate": "^7.0.0",
    "htmlhint": "^1.1.4",
    "pa11y": "^6.1.0",
    "prettier": "^2.8.0"
  }
}
```

### Конфигурационный файл HTML Validate (.htmlvalidate.json):

```json
{
  "extends": ["html-validate:recommended"],
  "rules": {
    "doctype-style": "require",
    "element-permitted-content": "error",
    "element-required-attributes": "error",
    "require-sri": "off",
    "no-implicit-close": "off",
    "prefer-tbody": "warn",
    "attr-quotes": "error",
    "doctype-first": "error",
    "title-required": "error",
    "area-alt": "error",
    "frame-title": "error",
    "image-alt": "error",
    "image-src": "error",
    "input-label": "error",
    "table-header": "error"
  }
}
```

## Заключение

Инструменты валидации HTML являются важной частью процесса разработки, помогая поддерживать высокое качество кода и соответствие стандартам. Автоматизация проверок через CI/CD и редакторы кода позволяет выявлять и исправлять ошибки на ранних стадиях разработки. Регулярное использование этих инструментов улучшает совместимость, доступность и поддерживаемость веб-сайтов.

## Следующие темы
- [[HTML/Валидация/Соответствие стандартам]]
- [[HTML/Валидация/Основы]]
- [[HTML/Доступность]]

## Теги
#html #validation #tools #web-development #quality-assurance #w3c #frontend #ci-cd