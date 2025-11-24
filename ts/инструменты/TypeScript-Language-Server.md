---
aliases: [TypeScript Language Server, TSServer, Сервер языка TypeScript]
tags: [typescript, lsp, language-server, tsserver, ide, инструменты]
---

# TypeScript Language Server

TypeScript Language Server (TSServer) - это сервер языка, реализующий Language Server Protocol (LSP), который обеспечивает интеллектуальные возможности редактора для TypeScript кода. Он предоставляет такие функции, как автодополнение, навигация по коду, проверка ошибок и рефакторинг.

## Архитектура Language Server Protocol

LSP позволяет редакторам и IDE взаимодействовать с серверами языка через стандартный протокол. TypeScript Language Server использует эту архитектуру для предоставления возможностей редактирования:

- **Клиент**: Редактор или IDE (VSCode, Vim, Emacs и т.д.)
- **Сервер**: TypeScript Language Server, обрабатывающий запросы
- **Протокол**: JSON-RPC сообщения по LSP спецификации

## Функциональные возможности

### 1. Автодополнение (Completion)

Предоставляет предложения для автодополнения на основе контекста:

```typescript
interface User {
  name: string;
  age: number;
}

const user: User = { name: 'John', age: 30 };
user. // При нажатии точки показываются 'name' и 'age'
```

### 2. Проверка ошибок в реальном времени

Немедленно обнаруживает ошибки типизации:

```typescript
const add = (a: number, b: number): number => a + b;
add('hello', 'world'); // Ошибка: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 3. Навигация по коду

- **Переход к определению** (Go to Definition)
- **Поиск всех ссылок** (Find All References)
- **Показ документации** (Hover)

### 4. Рефакторинг

- Переименование символов
- Извлечение методов/переменных
- Добавление импортов
- Организация импортов

## Запуск и конфигурация

### В командной строке

TSServer можно запустить напрямую:

```bash
npx typescript-language-server --stdio
```

### Через tsserver напрямую

```bash
npx tsserver
```

## Интеграция с редакторами

### VSCode

VSCode использует встроенный TypeScript Language Server. Для настройки:

```json
// .vscode/settings.json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.suggest.autoImports": true,
  "typescript.tsserver.log": "verbose",
  "typescript.tsserver.maxTsServerMemory": 4096,
  "typescript.surveys.enabled": false
}
```

### Vim/Neovim

С использованием coc.nvim:

```vim
" В .vimrc или init.vim
Plug 'neoclide/coc.nvim', {'branch': 'release'}

" Конфигурация TypeScript LSP
let g:coc_global_extensions = ['coc-tsserver', 'coc-eslint']
```

### Emacs

С использованием lsp-mode:

```elisp
(use-package lsp-mode
  :ensure t
  :commands lsp
  :hook (typescript-mode . lsp)
  :init
  (setq lsp-clients-typescript-server-args '("--stdio" "--tsserver-path" "node_modules/.bin/tsserver")))
```

## Настройка tsconfig.json для TSServer

Для оптимальной работы TSServer, правильно настройте `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "plugins": [
      {
        "name": "typescript-styled-plugin"
      }
    ]
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

## Плагины для TSServer

### TypeScript Styled Plugin

Для поддержки styled-components:

```bash
npm install --save-dev typescript-styled-plugin styled-components
```

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "name": "typescript-styled-plugin"
      }
    ]
  }
}
```

### GraphQL TypeScript Plugin

Для генерации типов из GraphQL:

```bash
npm install --save-dev graphql-codegen typescript-graphql-plugin
```

## Производительность TSServer

### Мониторинг производительности

Для отслеживания производительности включите логирование:

```json
// .vscode/settings.json
{
  "typescript.tsserver.trace": "verbose"
}
```

### Оптимизация производительности

1. **Ограничьте размер проекта**: Используйте `include` и `exclude` в `tsconfig.json`
2. **Используйте Composite Projects**: Для больших монорепозиториев
3. **Ограничьте использование `/// <reference>`**: Предпочтительно использовать импорты ES6

```json
{
  "compilerOptions": {
    "incremental": true,
    "composite": true
  }
}
```

## Устранение неполадок

### Перезапуск TSServer

В VSCode:
1. `Ctrl+Shift+P` (или `Cmd+Shift+P`)
2. Выберите "TypeScript: Restart TS Server"

### Проверка версии TypeScript

Убедитесь, что используется правильная версия TypeScript:

```bash
npx tsc --version
```

И в редакторе должна использоваться та же версия.

### Очистка кэша

Иногда помогает очистка кэша TSServer:

```bash
rm -rf node_modules/.cache
rm -rf .tsbuildinfo
```

## Альтернативные реализации

### VTSLS (Volar TypeScript Server)

Новый сервер языка от команды Volar, особенно полезный для Vue.js проектов с TypeScript:

```bash
npm install --save-dev @volar/typescript
```

### Biome

Новый инструмент, который может заменить как линтер, так и форматер, и включает поддержку TypeScript:

```bash
npm install --save-dev @biomejs/biome
```

## Интеграция с CI/CD

Для проверки типов в процессе CI:

```yaml
# .github/workflows/typecheck.yml
name: Type Check
on: [push, pull_request]
jobs:
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npx tsc --noEmit
```

## Практические советы

- Используйте локальную версию TypeScript, а не глобальную
- Регулярно обновляйте TypeScript и Language Server
- Настройте правильные пути в `tsconfig.json`
- Используйте `--noEmit` для проверки типов без компиляции
- Рассмотрите использование `composite` и `incremental` проектов для больших кодовых баз

## Связанные темы

- [[ESLint]]
- [[Prettier]]
- [[TypeScript-плагины]]
- [[Редакторы-и-IDE]]
