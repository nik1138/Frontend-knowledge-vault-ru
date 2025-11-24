---
aliases: [Плагины для TypeScript, Расширения TypeScript]
tags: [typescript, плагины, расширения, инструменты, компиляция]
---

# TypeScript-плагины

TypeScript-плагины - это расширения, которые позволяют изменять и дополнять стандартное поведение компилятора TypeScript. Они позволяют трансформировать код, добавлять пользовательские проверки, генерировать дополнительные артефакты и многое другое.

## Типы плагинов

### 1. Плагины компилятора (Compiler Plugins)

Эти плагины работают на этапе компиляции и могут изменять AST (абстрактное синтаксическое дерево) TypeScript кода. Они позволяют:

- Преобразовывать синтаксис
- Выполнять пользовательские проверки
- Генерировать дополнительный код

### 2. Плагины для транспиляторов

Интеграция с инструментами типа Babel, SWC или esbuild для обработки TypeScript кода.

## Создание простого плагина компилятора

Вот пример базового плагина, который добавляет комментарии к функциям:

```typescript
import * as ts from 'typescript';

function createPlugin(program: ts.Program): ts.TransformerFactory<ts.SourceFile> {
  return (context: ts.TransformationContext) => {
    return (sourceFile: ts.SourceFile) => {
      function visitNode(node: ts.Node): ts.Node {
        // Добавляем комментарий к каждой функции
        if (ts.isFunctionDeclaration(node)) {
          const comment = ts.addSyntheticLeadingComment(
            node,
            ts.SyntaxKind.MultiLineCommentTrivia,
            ` Автосгенерированный комментарий для ${node.name?.getText()}`,
            true
          );
          return ts.visitEachChild(comment, visitNode, context);
        }
        
        return ts.visitEachChild(node, visitNode, context);
      }

      return ts.visitNode(sourceFile, visitNode);
    };
  };
}

export default createPlugin;
```

## Популярные плагины

### 1. ts-patch

Позволяет патчить TypeScript компилятор для добавления новых возможностей:

```bash
npm install -D ts-patch
npx ts-patch install
```

### 2. ts-transformer-keys

Генерирует ключи для типов:

```typescript
import { keys } from 'ts-transformer-keys';

interface User {
  id: number;
  name: string;
  email: string;
}

const userKeys = keys<User>(); // ['id', 'name', 'email']
```

### 3. ts-auto-guard

Автоматически генерирует guard функции для типов:

```typescript
// До
interface User {
  name: string;
  age: number;
}

// После использования плагина
// Генерируется файл с guard функцией:
// export function isUser(obj: unknown): obj is User { ... }
```

### 4. ts-transformer-enumerate

Позволяет создавать перечисления из литеральных типов:

```typescript
type Status = 'pending' | 'success' | 'error';
const Status = enumerate<Status>(); // ['pending', 'success', 'error']
```

## Настройка плагинов в tsconfig.json

Для использования плагинов, необходимо настроить `tsconfig.json`:

```json
{
  "compilerOptions": {
    "plugins": [
      {
        "transform": "ts-transformer-keys/transformer",
        "type": "program"
      },
      {
        "transform": "path/to/custom-transformer",
        "after": true,
        "formatDiagnostics": true
      }
    ]
  }
}
```

## Использование с Webpack

Для использования плагинов с Webpack, можно использовать `ts-loader`:

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              getCustomTransformers: (program) => ({
                before: [require('ts-transformer-keys').default(program)],
              }),
            },
          },
        ],
        exclude: /node_modules/,
      },
    ],
  },
};
```

## Использование с Rollup

Для Rollup можно использовать `@rollup/plugin-typescript` с настройкой трансформеров:

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'esm',
  },
  plugins: [
    typescript({
      transformers: {
        before: [
          require('ts-transformer-keys').default(/* program */),
        ],
      },
    }),
  ],
};
```

## Плагины для генерации кода

### tsoa

Генерирует REST API из TypeScript аннотаций:

```typescript
import { Controller, Get, Route } from 'tsoa';

@Route('users')
export class UsersController extends Controller {
  @Get()
  public async getAll(): Promise<User[]> {
    return [];
  }
}
```

### graphql-codegen

Генерирует TypeScript типы из GraphQL схемы:

```yaml
# codegen.yml
overwrite: true
schema: "src/schema.graphql"
generates:
  src/generated/graphql.ts:
    plugins:
      - "typescript"
      - "typescript-resolvers"
```

## Практические советы

- Плагины могут значительно усложнить процесс компиляции
- Тестирование плагинов требует отдельного внимания
- Некоторые плагины могут конфликтовать друг с другом
- Всегда проверяйте совместимость с версиями TypeScript
- Используйте плагины для автоматизации рутинных задач

## Интеграция с инструментами

### С ESLint

Некоторые плагины могут требовать дополнительных правил ESLint для корректной работы:

```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "plugin:your-plugin/recommended"
  ]
}
```

### С Prettier

Плагины, генерирующие код, могут конфликтировать с Prettier - убедитесь, что сгенерированный код правильно форматируется.

## Связанные темы

- [[ESLint]]
- [[Prettier]]
- [[Редакторы-и-IDE]]
- [[TypeScript-Language-Server]]
