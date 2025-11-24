---
aliases: [IDE для TypeScript, Редакторы TypeScript, Среды разработки TypeScript]
tags: [typescript, ide, редакторы, vscode, webstorm, разработка]
---

# Редакторы и IDE для TypeScript

Выбор подходящего редактора или IDE играет ключевую роль в продуктивности при разработке на TypeScript. Современные редакторы обеспечивают отличную поддержку TypeScript с возможностями автодополнения, навигации по коду, проверки ошибок в реальном времени и многим другим.

## Visual Studio Code

VSCode является наиболее популярным выбором для разработки на TypeScript благодаря встроенной поддержке языка и богатой экосистеме расширений.

### Основные возможности

- **Встроенная поддержка TypeScript**: VSCode использует ту же библиотеку, что и TypeScript компилятор
- **Интеллектуальное автодополнение (IntelliSense)**: Предлагает автодополнение, сигнатуры методов, документацию и определения
- **Навигация по коду**: Переход к определению, поиск ссылок, навигация по файлам
- **Рефакторинг**: Переименование, извлечение методов, генерация конструкторов и многое другое
- **Отладка**: Встроенная поддержка отладки TypeScript кода

### Полезные расширения

#### 1. TypeScript и JavaScript (официальное расширение от Microsoft)

Предоставляет расширенные возможности для TypeScript и JavaScript разработки.

#### 2. ESLint

Интеграция ESLint прямо в редакторе для проверки кода:

```json
// .vscode/settings.json
{
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "eslint.run": "onSave"
}
```

#### 3. Prettier - Code formatter

Автоматическое форматирование кода:

```json
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true
}
```

#### 4. Path Intellisense

Автодополнение путей к файлам:

```json
// .vscode/settings.json
{
  "path-intellisense.mappings": {
    "/": "${workspaceRoot}/src"
  }
}
```

#### 5. Import Cost

Отображает размер импортируемых пакетов:

```json
// .vscode/settings.json
{
  "importCost.largePackageSupport": true
}
```

### Настройка рабочей среды

#### Файл .vscode/settings.json

```json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.preferences.useAliasesForRenames": false
}
```

## WebStorm / IntelliJ IDEA

Мощная IDE от JetBrains с отличной поддержкой TypeScript.

### Особенности

- **Типобезопасность в реальном времени**: Обнаружение ошибок до компиляции
- **Мощные возможности рефакторинга**: Безопасное переименование, извлечение методов, миграции
- **Интеграция с фреймворками**: Поддержка React, Angular, Vue.js с TypeScript
- **Отладка**: Расширенные возможности отладки с точками останова и переменными

### Конфигурация TypeScript

В WebStorm:

1. Перейдите в Settings → Languages & Frameworks → TypeScript
2. Укажите путь к TypeScript компилятору (node_modules/.bin/tsc)
3. Установите версию языка и настройки компиляции

## Sublime Text

Легковесный редактор с поддержкой TypeScript через плагины.

### Необходимые плагины

- **TypeScript-Sublime-Plugin**: Обеспечивает базовую поддержку TypeScript
- **SublimeLinter-contrib-tslint**: Интеграция с TSLint (или ESLint)
- **Babel**: Для поддержки синтаксиса TypeScript

## Vim/Neovim

Для любителей Vim существует несколько решений для работы с TypeScript.

### Плагины

#### 1. coc.nvim

Плагин, обеспечивающий LSP поддержку:

```vim
" В .vimrc или init.vim
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```

#### 2. TypeScript REPL

Для интерактивной разработки:

```vim
" Добавить в конфигурацию
autocmd FileType typescript setlocal omnifunc=tsuquyomi#complete
```

## Atom

Хотя Atom больше не поддерживается, он все еще может быть использован с TypeScript:

- **atom-typescript**: Пакет для полной поддержки TypeScript
- **linter-tslint**: Интеграция с TSLint
- **autocomplete-modules**: Автодополнение импортов

## Онлайн-редакторы

### 1. CodeSandbox

Онлайн IDE с поддержкой TypeScript:

- Мгновенный запуск проектов
- Совместная разработка
- Поддержка популярных фреймворков

### 2. StackBlitz

Облачный редактор с поддержкой TypeScript:

- Работает прямо в браузере
- Быстрая загрузка проектов
- Поддержка Node.js и клиентских приложений

## Настройка общей конфигурации

### .editorconfig

Для согласованности форматирования между разными редакторами:

```
root = true

[*.ts]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.tsx]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

### Интеграция с Git

#### .gitignore для редакторов

```
# VSCode
.vscode/

# WebStorm
.idea/

# Vim
*.swp
*.swo
```

## Практические советы

- Используйте одинаковую версию TypeScript в редакторе и проекте
- Настройте автосохранение и форматирование при сохранении
- Установите расширения для вашего фреймворка (React, Angular, Vue)
- Настройте горячие клавиши для часто используемых действий
- Используйте snippets для быстрого написания часто используемых конструкций

### Пример TypeScript snippet

```json
// В VSCode: File > Preferences > Configure User Snippets > typescript.json
{
  "Interface": {
    "prefix": "iface",
    "body": [
      "interface ${1:InterfaceName} {",
      "  ${2:property}: ${3:type};",
      "}"
    ],
    "description": "TypeScript interface"
  }
}
```

## Связанные темы

- [[ESLint]]
- [[Prettier]]
- [[TypeScript-плагины]]
- [[TypeScript-Language-Server]]
