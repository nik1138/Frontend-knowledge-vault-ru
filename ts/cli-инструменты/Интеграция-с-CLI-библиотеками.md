---
aliases: [Интеграция с CLI библиотеками, CLI библиотеки]
tags: [typescript, cli, commander, yargs, integration]
---

# Интеграция с CLI библиотеками

Интеграция с CLI библиотеками позволяет разработчикам создавать мощные и удобные интерфейсы командной строки для своих приложений. В этом документе мы рассмотрим различные подходы к интеграции с популярными библиотеками, такими как Commander.js и Yargs, а также способы их настройки под конкретные задачи.

## Обзор CLI библиотек

### Commander.js

Commander.js - это легковесная библиотека, которая предоставляет простой и интуитивно понятный API для создания CLI приложений. Она идеально подходит для простых и средних по сложности CLI инструментов.

```typescript
import { Command } from 'commander';
import { version } from '../package.json';

const program = new Command();

program
  .name('my-cli-tool')
  .description('Описание CLI инструмента')
  .version(version);

program.parse();
```

### Yargs

Yargs - более мощная библиотека с богатым набором возможностей для парсинга аргументов и создания сложных CLI приложений. Она предоставляет больше возможностей для настройки и автоматической генерации справки.

```typescript
import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';

const argv = yargs(hideBin(process.argv))
  .scriptName('my-app')
  .usage('$0 <cmd> [args]')
  .help()
  .argv;
```

## Интеграция с Commander.js

### Базовая интеграция

Для интеграции с Commander.js необходимо установить библиотеку и настроить основные команды:

```bash
npm install commander
npm install @types/commander --save-dev
```

Затем создайте основной CLI файл:

```typescript
#!/usr/bin/env node

import { Command } from 'commander';
import { version } from '../package.json';
import { buildCommand } from './commands/build';
import { deployCommand } from './commands/deploy';

const program = new Command();

program
  .name('my-cli-tool')
  .description('Мощный CLI инструмент для разработки')
  .version(version);

// Регистрация команд
program.addCommand(buildCommand);
program.addCommand(deployCommand);

program.parse();
```

### Создание отдельных команд

Для лучшей организации кода рекомендуется выносить команды в отдельные файлы:

```typescript
// commands/build.ts
import { Command } from 'commander';

export const buildCommand = new Command('build')
  .description('Сборка проекта')
  .option('-w, --watch', 'наблюдение за изменениями')
  .option('-o, --output <path>', 'путь к выходному файлу')
  .option('-m, --minify', 'минификация результата')
  .action(async (options) => {
    console.log('Выполняется сборка проекта...');
    try {
      // Логика сборки
      await performBuild(options);
      console.log('Сборка завершена успешно');
    } catch (error) {
      console.error('Ошибка при сборке:', error.message);
      process.exit(1);
    }
  });

async function performBuild(options: { watch?: boolean; output?: string; minify?: boolean }) {
  // Реализация логики сборки
  console.log('Опции сборки:', options);
}
```

### Расширенная интеграция с Commander.js

#### Поддержка подкоманд

```typescript
import { Command } from 'commander';

const deployCommand = new Command('deploy')
  .description('Команды для развертывания');

deployCommand
  .command('staging')
  .description('Развертывание в staging окружение')
  .option('-f, --force', 'принудительное развертывание')
  .action(async (options) => {
    console.log('Развертывание в staging...');
    try {
      await performDeploy('staging', options);
    } catch (error) {
      console.error('Ошибка при развертывании:', error.message);
      process.exit(1);
    }
  });

deployCommand
  .command('production')
  .description('Развертывание в production окружение')
  .option('-c, --confirm', 'подтверждение развертывания')
  .action(async (options) => {
    if (!options.confirm) {
      console.error('Требуется подтверждение для развертывания в production');
      process.exit(1);
    }
    
    try {
      await performDeploy('production', options);
    } catch (error) {
      console.error('Ошибка при развертывании:', error.message);
      process.exit(1);
    }
  });

async function performDeploy(environment: string, options: any) {
  console.log(`Развертывание в ${environment} с опциями:`, options);
}
```

#### Валидация и пользовательские парсеры

```typescript
import { Command, Argument } from 'commander';

function parsePort(value: string): number {
  const port = parseInt(value, 10);
  if (isNaN(port)) {
    throw new Error('Порт должен быть числом');
  }
  if (port < 1 || port > 65535) {
    throw new Error('Порт должен быть в диапазоне от 1 до 65535');
  }
  return port;
}

const serverCommand = new Command('serve')
  .description('Запуск сервера')
  .option('-p, --port <number>', 'порт для сервера', parsePort)
  .option('-h, --host <host>', 'хост для сервера', 'localhost')
  .option('-s, --secure', 'использовать HTTPS')
  .action(async (options) => {
    console.log(`Сервер запущен на ${options.host}:${options.port}`);
    if (options.secure) {
      console.log('Используется защищенное соединение');
    }
  });
```

## Интеграция с Yargs

### Базовая интеграция

Для интеграции с Yargs установите необходимые пакеты:

```bash
npm install yargs
npm install @types/yargs --save-dev
```

Создайте основной CLI файл:

```typescript
#!/usr/bin/env node

import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';
import { buildHandler } from './handlers/build';
import { deployHandler } from './handlers/deploy';

const argv = yargs(hideBin(process.argv))
  .scriptName('my-cli-tool')
  .usage('$0 <command> [options]')
  .command('build', 'Сборка проекта', (yargs) => {
    return yargs
      .option('watch', {
        alias: 'w',
        type: 'boolean',
        description: 'Наблюдение за изменениями'
      })
      .option('output', {
        alias: 'o',
        type: 'string',
        description: 'Путь к выходному файлу'
      })
      .option('minify', {
        alias: 'm',
        type: 'boolean',
        description: 'Минификация результата'
      });
  }, buildHandler)
  .command('deploy <environment>', 'Развертывание в окружение', (yargs) => {
    return yargs
      .positional('environment', {
        describe: 'Окружение для развертывания',
        type: 'string',
        choices: ['staging', 'production']
      })
      .option('confirm', {
        alias: 'c',
        type: 'boolean',
        description: 'Подтверждение развертывания',
        default: false
      });
  }, deployHandler)
  .help()
  .argv;
```

### Создание обработчиков команд

```typescript
// handlers/build.ts
import { ArgumentsCamelCase } from 'yargs';

interface BuildArgs {
  watch?: boolean;
  output?: string;
  minify?: boolean;
}

export async function buildHandler(argv: ArgumentsCamelCase<BuildArgs>) {
  console.log('Выполняется сборка проекта...');
  try {
    await performBuild({
      watch: argv.watch,
      output: argv.output,
      minify: argv.minify
    });
    console.log('Сборка завершена успешно');
  } catch (error) {
    console.error('Ошибка при сборке:', error.message);
    process.exit(1);
  }
}

async function performBuild(options: BuildArgs) {
  console.log('Опции сборки:', options);
  // Реализация логики сборки
}
```

### Расширенная интеграция с Yargs

#### Валидация и пользовательские парсеры

```typescript
import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';

const argv = yargs(hideBin(process.argv))
  .option('port', {
    alias: 'p',
    type: 'number',
    description: 'Порт для сервера',
    default: 3000
  })
  .option('tags', {
    alias: 't',
    type: 'string',
    description: 'Список тегов',
    coerce: (arg: string) => arg.split(',').map(tag => tag.trim())
  })
  .check((argv) => {
    if (argv.port && (argv.port < 1 || argv.port > 65535)) {
      throw new Error('Порт должен быть в диапазоне от 1 до 65535');
    }
    return true;
  })
  .argv;

console.log('Порт:', argv.port);
console.log('Теги:', argv.tags);
```

#### Подкоманды и вложенные команды

```typescript
import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';

const argv = yargs(hideBin(process.argv))
  .command('config', 'Управление конфигурацией', (yargs) => {
    return yargs
      .command('get <key>', 'Получить значение конфигурации', (yargs) => {
        return yargs.positional('key', {
          describe: 'Ключ для получения значения'
        });
      }, async (argv) => {
        console.log(`Получение значения для ключа: ${argv.key}`);
        // Логика получения значения
      })
      .command('set <key> <value>', 'Установить значение конфигурации', (yargs) => {
        return yargs
          .positional('key', {
            describe: 'Ключ для установки значения'
          })
          .positional('value', {
            describe: 'Значение для установки'
          });
      }, async (argv) => {
        console.log(`Установка значения: ${argv.key}=${argv.value}`);
        // Логика установки значения
      })
      .command('list', 'Список всех значений конфигурации', {}, async () => {
        console.log('Список всех значений конфигурации...');
        // Логика вывода списка
      });
  })
  .help()
  .argv;
```

## Сравнение подходов

### Commander.js vs Yargs

| Критерий | Commander.js | Yargs |
|----------|--------------|-------|
| Простота использования | Высокая | Средняя |
| Функциональность | Базовая | Расширенная |
| Размер бандла | Меньше | Больше |
| Гибкость | Средняя | Высокая |
| Время изучения | Короче | Дольше |

### Когда использовать Commander.js

- Для простых CLI инструментов
- Когда важен размер бандла
- Для быстрого прототипирования
- Когда нужен простой и понятный API

### Когда использовать Yargs

- Для сложных CLI с множеством опций
- Когда требуется богатая функциональность
- Для приложений с вложенными командами
- Когда важна автоматическая генерация справки

## Практические рекомендации

### Организация кода

Для лучшей поддержки кода рекомендуется организовать CLI инструмент следующим образом:

```
src/
├── cli/
│   ├── index.ts          # Основной CLI файл
│   ├── commands/         # Файлы команд
│   │   ├── build.ts
│   │   ├── deploy.ts
│   │   └── serve.ts
│   ├── handlers/         # Обработчики команд
│   │   ├── build-handler.ts
│   │   └── deploy-handler.ts
│   └── utils/            # Вспомогательные функции
│       ├── validators.ts
│       └── formatters.ts
```

### Обработка ошибок

```typescript
import { Command } from 'commander';

const program = new Command();

program
  .command('process')
  .description('Обработка данных')
  .action(async (options) => {
    try {
      await processData(options);
    } catch (error) {
      console.error('Ошибка при обработке данных:', error.message);
      process.exitCode = 1;
    }
  });

async function processData(options: any) {
  // Логика обработки данных
  throw new Error('Пример ошибки');
}
```

### Тестирование CLI

Для тестирования CLI приложений рекомендуется использовать unit тесты для обработчиков команд:

```typescript
// tests/build-handler.test.ts
import { buildHandler } from '../src/handlers/build';
import { ArgumentsCamelCase } from 'yargs';

describe('buildHandler', () => {
  it('should build project with correct options', async () => {
    const argv = {
      watch: true,
      output: './dist',
      minify: false
    } as ArgumentsCamelCase<{ watch?: boolean; output?: string; minify?: boolean }>;
    
    // Мокаем консоль для проверки вывода
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();
    
    await buildHandler(argv);
    
    expect(consoleSpy).toHaveBeenCalledWith('Выполняется сборка проекта...');
    
    consoleSpy.mockRestore();
  });
});
```

## Заключение

Интеграция с CLI библиотеками позволяет создавать мощные и удобные интерфейсы командной строки. Выбор между Commander.js и Yargs зависит от сложности приложения и требований к функциональности. Важно правильно организовать код и предусмотреть обработку ошибок для создания надежного CLI инструмента.

См. также:
- [[Command-line-интерфейсы]]
- [[Commander-js]]
- [[Yargs]]
- [[Публикация-CLI-инструментов]]
- [[Node.js-с-TypeScript]]
- [[TypeScript-инструменты-и-библиотеки]]
