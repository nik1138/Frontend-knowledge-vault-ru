# Поддержка TypeScript в IDE

TypeScript предоставляет отличную поддержку в большинстве современных IDE и редакторов кода. Это включает автодополнение, проверку ошибок, навигацию по коду и другие возможности, значительно улучшающие опыт разработки.

## Поддержка в основных IDE

### Visual Studio Code

Visual Studio Code имеет встроенную поддержку TypeScript благодаря той же команде Microsoft, которая разрабатывает TypeScript.

#### Установка и настройка

TypeScript обычно включается в VS Code по умолчанию, но вы можете убедиться в этом:

```json
// settings.json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always",
  "typescript.preferences.renameShorthandProperties": false,
  "typescript.preferences.useAliasesForRenames": true
}
```

#### Основные функции

1. **Автодополнение** - IntelliSense предоставляет предложения на основе типов:
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

// При вводе user. VS Code покажет доступные свойства: id, name, email
```

2. **Навигация по коду**:
   - `Ctrl+Click` (Cmd+Click на macOS) для перехода к определению
   - `F12` для перехода к определению
   - `Ctrl+Shift+F12` для поиска всех ссылок

3. **Переименование**:
   - `F2` для переименования символа с автоматическим обновлением всех ссылок

4. **Диагностика ошибок**:
   - Ошибки отображаются в редакторе и в панели проблем
   - Подсветка потенциальных проблем до выполнения кода

#### Расширения для TypeScript

- **ESLint** - интеграция с ESLint для проверки типов и кода
- **Prettier** - форматирование TypeScript кода
- **Path Intellisense** - автодополнение путей к файлам
- **Bracket Pair Colorizer** - цветовая подсветка парных скобок

### WebStorm / IntelliJ IDEA

JetBrains IDE предоставляют мощную поддержку TypeScript:

#### Настройка TypeScript

1. Открыть `Settings` → `Languages & Frameworks` → `TypeScript`
2. Убедиться, что TypeScript установлен локально или указан путь к глобальному установочному файлу

#### Основные функции

1. **Проверка типов в реальном времени**
2. **Создание шаблонов TypeScript** для частых операций
3. **Переименование с учетом типов**
4. **Извлечение переменных/функций с правильной типизацией**

#### Конфигурационный файл
```json
// jsconfig.json или tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2017",
    "lib": ["es2017"],
    "strict": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "allowJs": true,
    "outDir": "dist"
  },
  "exclude": ["node_modules", "dist"]
}
```

### Vim/Neovim

Современные конфигурации Vim/Neovim могут предоставить уровень поддержки TypeScript, близкий к полноценным IDE:

#### С использованием Language Server Protocol (LSP)

1. **Установка tsserver**:
```bash
npm install -g typescript typescript-language-server
```

2. **Настройка с использованием nvim-lspconfig (для Neovim)**:
```lua
local lspconfig = require('lspconfig')
lspconfig.tsserver.setup{
  on_attach = function(client, bufnr)
    -- Конфигурация на ваше усмотрение
  end,
}
```

3. **Плагины для дополнительных функций**:
   - `coc.nvim` - автодополнение и LSP
   - `typescript-vim` - синтаксическая подсветка
   - `denite-typescript` - навигация по TypeScript

### Sublime Text

С помощью плагинов Sublime Text может обеспечить хорошую поддержку TypeScript:

1. **TypeScript Syntax** - синтаксическая подсветка
2. **TypeScript Plugin** - интеграция с TypeScript Language Service

### Atom

Хотя развитие Atom замедлилось, он все еще поддерживает TypeScript:

1. **atom-typescript** - плагин для TypeScript
2. **autocomplete-typescript** - автодополнение
3. **linter-tslint** - проверка кода

## TypeScript Language Service

### Основные возможности

TypeScript Language Service - это ядро функций IntelliSense, предоставляемое TypeScript. Он включает:

1. **Проверка синтаксиса и типов**
2. **Автодополнение**
3. **Навигация по коду**
4. **Поиск ссылок**
5. **Переименование**
6. **Сигнатура функций**
7. **Информация о типах**

### Конфигурация для IDE

Файл `tsconfig.json` влияет на работу IDE:

```json
{
  "compilerOptions": {
    // Базовые опции
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    
    // Опции строгости
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    
    // Опции связывания
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    
    // Опции вывода
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": false,
    
    // Опции взаимодействия с JavaScript
    "allowJs": true,
    "checkJs": false,
    
    // Опции для экосистемы
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

## Плагины и расширения

### Для VS Code

#### TypeScript Importer
Автоматически добавляет импорты при использовании классов и функций.

#### Bracket Pair Colorizer
Цветовая подсветка парных скобок для лучшей читаемости.

#### Path Intellisense
Автодополнение для путей к файлам.

#### Prettier
Автоформатирование кода в соответствии с установленными правилами.

### Для WebStorm

#### Vue.js (если используете Vue с TypeScript)
Поддержка синтаксиса Vue в TypeScript файлах.

#### Node.js
Расширенная поддержка Node.js API.

## Настройки для командной разработки

### Единый стиль кода

```json
// .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 2
indent_style = space
insert_final_newline = true
max_line_length = 80
trim_trailing_whitespace = true

[*.ts]
indent_size = 2
```

### Совместимость с ESLint и Prettier

```json
// .eslintrc.json
{
  "parser": "@typescript-eslint/parser",
  "plugins": [
    "@typescript-eslint"
  ],
  "extends": [
    "eslint:recommended",
    "@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-parameter-properties": "off",
    "@typescript-eslint/no-use-before-define": "off",
    "prefer-template": "error",
    "@typescript-eslint/camelcase": "off",
    "@typescript-eslint/no-explicit-any": "off",
    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        "argsIgnorePattern": "^_",
        "varsIgnorePattern": "^_"
      }
    ]
  }
}
```

## Отладка TypeScript

### В браузере

С помощью source maps, отладка TypeScript кода в инструментах разработчика браузера:

```json
// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSourceMap": false,
    "sourceRoot": "/"
  }
}
```

### В Node.js

```json
// tsconfig.json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSources": true
  }
}
```

И запуск с помощью ts-node или компиляция перед отладкой:
```bash
node --inspect-brk -r ts-node/register src/index.ts
```

## Современные тенденции

### Встроенная проверка типов в IDE

Современные IDE теперь не просто отображают ошибки, но и предлагают решения:
- Быстрые исправления (Quick Fixes)
- Преобразования кода
- Предложения рефакторинга

### Интеграция с GitHub Copilot

AI-помощники интегрируются с TypeScript для более точных предложений, используя информацию о типах.

### Remote Development

- Поддержка TypeScript в удаленных средах разработки
- Облачные IDE с полноценной поддержкой TypeScript

## Лучшие практики

1. **Используйте строгую конфигурацию TypeScript** (`"strict": true`)
2. **Установите единые настройки для команды** через .editorconfig, .eslintrc, etc.
3. **Регулярно обновляйте TypeScript** для получения новых возможностей и исправлений
4. **Используйте Path mapping** для сокращения длинных путей импорта
5. **Настройте автосохранение** и форматирование при сохранении

## Устранение неполадок

### Если IDE не распознает типы

1. Перезапустите TypeScript Server (в VS Code: `TypeScript: Restart TS Server`)
2. Проверьте файл `tsconfig.json`
3. Убедитесь, что установлены нужные `@types` пакеты

### Медленная работа IDE

1. Убедитесь, что `tsconfig.json` правильно исключает ненужные файлы
2. Рассмотрите использование `files` вместо `include` для больших проектов
3. Временно отключите ненужные расширения

## Связь с другими концепциями

- [[configuration/index]] - настройка TypeScript, влияющая на IDE
- [[tooling-workflows/index]] - рабочие процессы с TypeScript инструментами
- [[error-handling/type-checking-errors]] - обработка ошибок проверки типов
- [[modules/module-resolution]] - разрешение модулей, влияющее на автодополнение