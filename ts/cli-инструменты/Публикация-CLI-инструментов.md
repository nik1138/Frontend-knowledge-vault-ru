---
aliases: [Публикация CLI инструментов, npm CLI пакеты]
tags: [typescript, cli, npm, publishing, nodejs]
---

# Публикация CLI инструментов

Публикация CLI инструментов в npm позволяет делиться своими инструментами с сообществом и использовать их в различных проектах. В этом документе мы рассмотрим процесс создания, настройки и публикации CLI инструментов с использованием TypeScript.

## Подготовка проекта

### Структура проекта

Для создания CLI инструмента рекомендуется следующая структура проекта:

```
my-cli-tool/
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts          # Точка входа CLI
│   ├── commands/         # Файлы команд
│   │   ├── build.ts
│   │   └── deploy.ts
│   └── utils/            # Вспомогательные функции
│       └── logger.ts
├── bin/
│   └── my-cli-tool       # Исполняемый файл
├── dist/                 # Скомпилированные файлы (после сборки)
└── README.md
```

### Настройка package.json

```json
{
  "name": "@your-username/my-cli-tool",
  "version": "1.0.0",
  "description": "Описание вашего CLI инструмента",
  "bin": {
    "my-cli-tool": "./bin/my-cli-tool"
  },
  "files": [
    "bin/",
    "dist/"
  ],
  "scripts": {
    "build": "tsc",
    "dev": "ts-node src/index.ts",
    "prepublishOnly": "npm run build"
  },
  "keywords": [
    "cli",
    "typescript",
    "command-line",
    "tool"
  ],
  "author": "Ваше имя",
  "license": "MIT",
  "dependencies": {
    "commander": "^9.0.0",
    "chalk": "^4.1.0"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/commander": "^2.12.0",
    "typescript": "^4.8.0",
    "ts-node": "^10.0.0"
  }
}
```

### Настройка tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "tests"
  ]
}
```

## Создание исполняемого файла

### Точка входа CLI

Создайте основной файл CLI в `src/index.ts`:

```typescript
#!/usr/bin/env node

import { Command } from 'commander';
import chalk from 'chalk';
import { version } from '../package.json';
import { buildCommand } from './commands/build';
import { deployCommand } from './commands/deploy';

const program = new Command();

program
  .name('my-cli-tool')
  .description(chalk.blue('Мощный CLI инструмент для разработки'))
  .version(version, '-v, --version', chalk.green('Показать версию инструмента'))
  .addHelpText('after', chalk.gray(`
Примеры:
  $ my-cli-tool build
  $ my-cli-tool build --watch
  $ my-cli-tool deploy production --confirm
  `));

// Регистрация команд
program.addCommand(buildCommand);
program.addCommand(deployCommand);

// Обработка ошибок
program.on('command:*', () => {
  console.error(chalk.red('Неверная команда'));
  process.exitCode = 1;
});

program.parseAsync().catch((error) => {
  console.error(chalk.red('Ошибка:'), error.message);
  process.exitCode = 1;
});
```

### Создание исполняемого файла в bin/

Создайте файл `bin/my-cli-tool`:

```bash
#!/usr/bin/env node

// Проверка версии Node.js
const semver = require('semver');
const { engines } = require('../package.json');

if (!semver.satisfies(process.version, engines.node)) {
  console.error(
    `Требуется Node.js версии ${engines.node}, но установлена ${process.version}`
  );
  process.exit(1);
}

// Запуск скомпилированного CLI инструмента
require('../dist/index.js');
```

Не забудьте сделать файл исполняемым:

```bash
chmod +x bin/my-cli-tool
```

## Сборка проекта

### Компиляция TypeScript

```bash
npm run build
```

После компиляции все файлы из `src/` будут скомпилированы в `dist/`, и CLI инструмент будет готов к использованию.

### Проверка перед публикацией

Перед публикацией рекомендуется протестировать CLI инструмент локально:

```bash
# Установка пакета локально в режиме разработки
npm link

# Тестирование CLI
my-cli-tool --help
my-cli-tool build --watch

# После тестирования отвязать пакет
npm unlink -g my-cli-tool
```

## Публикация в npm

### Подготовка к публикации

1. Убедитесь, что у вас есть аккаунт npm
2. Войдите в систему:
   ```bash
   npm login
   ```

3. Проверьте имя пакета на уникальность
4. Убедитесь, что все файлы включены в публикацию (см. поле `files` в package.json)

### Публикация

```bash
npm publish
```

Для публикации приватного пакета:

```bash
npm publish --access restricted
```

## Управление версиями

### Семантическое версионирование

CLI инструменты должны следовать семантическому версионированию (SemVer):
- `MAJOR.MINOR.PATCH` (например, 1.2.3)
- `MAJOR` - для несовместимых изменений
- `MINOR` - для добавления новой функциональности
- `PATCH` - для исправления ошибок

### Автоматическое обновление версии

```bash
# Повышение PATCH версии (1.0.0 -> 1.0.1)
npm version patch

# Повышение MINOR версии (1.0.0 -> 1.1.0)
npm version minor

# Повышение MAJOR версии (1.0.0 -> 2.0.0)
npm version major
```

## Продвинутые возможности

### Поддержка автодополнения

Для улучшения пользовательского опыта можно добавить поддержку автодополнения:

```typescript
// В файле src/index.ts
program
  .configureOutput({
    outputError: (str, write) => write(chalk.red(str))
  })
  .configureHelp({
    helpWidth: 80,
    sortSubcommands: true,
    sortOptions: true
  })
  .showHelpAfterError()
  .addHelpCommand('help [command]', 'Показать справку по команде');
```

### Конфигурационные файлы

Для более сложных CLI инструментов можно добавить поддержку конфигурационных файлов:

```typescript
import fs from 'fs';
import path from 'path';

function loadConfig(): any {
  const configPaths = [
    './.myclirc',
    './.myclirc.json',
    './config.json',
    path.join(process.env.HOME || '', '.myclirc')
  ];

  for (const configPath of configPaths) {
    if (fs.existsSync(configPath)) {
      return JSON.parse(fs.readFileSync(configPath, 'utf8'));
    }
  }

  return {}; // Возвращаем пустой объект, если конфиг не найден
}
```

### Плагинная архитектура

Для расширяемых CLI инструментов можно реализовать плагинную архитектуру:

```typescript
interface Plugin {
  name: string;
  register: (program: Command) => void;
}

function loadPlugins(program: Command): void {
  // Поиск и загрузка плагинов
  const pluginDir = path.join(process.cwd(), 'plugins');
  
  if (fs.existsSync(pluginDir)) {
    const plugins = fs.readdirSync(pluginDir)
      .filter(file => file.endsWith('.js'))
      .map(file => path.join(pluginDir, file));
    
    for (const pluginPath of plugins) {
      const plugin: Plugin = require(pluginPath);
      plugin.register(program);
    }
  }
}
```

## Лучшие практики публикации

### Безопасность

1. Проверяйте зависимости на наличие уязвимостей:
   ```bash
   npm audit
   npm audit fix
   ```

2. Используйте `.npmignore` или поле `files` в `package.json` для исключения ненужных файлов

3. Не публикуйте чувствительные данные (ключи, пароли и т.д.)

### Оптимизация размера

1. Минимизируйте количество зависимостей
2. Используйте `dependencies` только для необходимых пакетов
3. Рассмотрите возможность использования `peerDependencies` для больших библиотек

### Документация

1. Обязательно включайте качественный README.md
2. Описывайте все команды и опции
3. Приводите примеры использования

### Поддержка

1. Указывайте поле `bugs` в package.json для сообщений об ошибках
2. Указывайте поле `homepage` для дополнительной документации
3. Рассмотрите возможность использования GitHub Issues для поддержки

## Примеры успешных CLI инструментов

### Create React App

```json
{
  "bin": {
    "create-react-app": "./index.js"
  },
  "files": [
    "index.js",
    "package.json",
    "tempalte/**/*"
  ]
}
```

### TypeScript Compiler

```json
{
  "bin": {
    "tsc": "./bin/tsc",
    "tsserver": "./bin/tsserver"
  }
}
```

## Отладка и тестирование

### Unit тестирование

Для тестирования CLI инструментов рекомендуется использовать Jest:

```typescript
// tests/cli.test.ts
import { spawn } from 'child_process';

describe('CLI инструмент', () => {
  it('должен отображать справку', (done) => {
    const cli = spawn('node', ['dist/index.js', '--help']);
    
    let output = '';
    cli.stdout.on('data', (data) => {
      output += data.toString();
    });
    
    cli.on('close', (code) => {
      expect(code).toBe(0);
      expect(output).toContain('Usage:');
      done();
    });
  });
});
```

### Интеграционное тестирование

```typescript
// tests/integration.test.ts
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

describe('Интеграционные тесты', () => {
  it('должен выполнить команду build', async () => {
    const { stdout, stderr } = await execAsync('my-cli-tool build --output test-dist');
    
    expect(stderr).toBe('');
    expect(stdout).toContain('Сборка завершена успешно');
  });
});
```

## Заключение

Публикация CLI инструментов в npm требует внимательного подхода к настройке проекта, документации и тестированию. Следуя лучшим практикам, вы можете создать качественный инструмент, который будет полезен сообществу разработчиков.

Помните о важности поддержки и обновления вашего инструмента, а также о необходимости предоставления качественной документации и примеров использования.

См. также:
- [[Command-line-интерфейсы]]
- [[Commander-js]]
- [[Yargs]]
- [[Интеграция-с-CLI-библиотеками]]
- [[Node.js-с-TypeScript]]
- [[TypeScript-инструменты-и-библиотеки]]
- [[NPM-менеджер-пакетов]]
- [[Сборка-и-оптимизация-TypeScript]]