---
aliases: [Yargs, yargs]
tags: [typescript, cli, yargs, nodejs]
---

# Yargs

Yargs - это мощная библиотека для создания Command-line интерфейсов в Node.js. Она предоставляет богатый набор возможностей для парсинга аргументов командной строки и создания удобных CLI приложений.

## Введение в Yargs

Yargs автоматически разбирает аргументы командной строки и предоставляет удобный способ определения опций, команд и аргументов. Он также генерирует справочную информацию и обрабатывает ошибки.

### Установка

```bash
npm install yargs
npm install @types/yargs # для TypeScript поддержки
```

## Основы использования

### Базовая структура

```typescript
#!/usr/bin/env node

import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';

const argv = yargs(hideBin(process.argv))
  .scriptName('my-app')
  .usage('$0 <cmd> [args]')
  .command('build', 'Сборка проекта', (yargs) => {
    return yargs.option('watch', {
      alias: 'w',
      type: 'boolean',
      description: 'Наблюдение за изменениями'
    });
  }, (argv) => {
    console.log('Сборка проекта...');
    if (argv.watch) {
      console.log('Включено наблюдение за изменениями');
    }
  })
  .help()
  .argv;
```

### Определение опций

```typescript
const argv = yargs(hideBin(process.argv))
  .option('verbose', {
    alias: 'v',
    type: 'boolean',
    description: 'Подробный вывод'
  })
  .option('port', {
    alias: 'p',
    type: 'number',
    description: 'Порт для сервера',
    default: 3000
  })
  .option('config', {
    alias: 'c',
    type: 'string',
    description: 'Путь к конфигурационному файлу'
  })
  .argv;

console.log('Опции:', {
  verbose: argv.verbose,
  port: argv.port,
  config: argv.config
});
```

## Продвинутое использование

### Валидация опций

```typescript
const argv = yargs(hideBin(process.argv))
  .option('port', {
    alias: 'p',
    type: 'number',
    description: 'Порт для сервера',
    default: 3000
  })
  .check((argv) => {
    if (argv.port && (argv.port < 1 || argv.port > 65535)) {
      throw new Error('Порт должен быть в диапазоне от 1 до 65535');
    }
    return true;
  })
  .argv;
```

### Аргументы командной строки

```typescript
const argv = yargs(hideBin(process.argv))
  .command('process <input> [output]', 'Обработка файлов', (yargs) => {
    return yargs
      .positional('input', {
        describe: 'Входной файл',
        type: 'string'
      })
      .positional('output', {
        describe: 'Выходной файл',
        type: 'string',
        default: 'output.txt'
      });
  }, (argv) => {
    console.log(`Обработка: ${argv.input} -> ${argv.output}`);
  })
  .argv;
```

### Подкоманды

```typescript
const argv = yargs(hideBin(process.argv))
  .command('deploy', 'Развертывание приложения', (yargs) => {
    return yargs
      .command('staging', 'Развертывание в staging', (yargs) => {
        return yargs.option('force', {
          alias: 'f',
          type: 'boolean',
          description: 'Принудительное развертывание'
        });
      }, (argv) => {
        console.log('Развертывание в staging...');
        if (argv.force) {
          console.log('Принудительное развертывание включено');
        }
      })
      .command('production', 'Развертывание в production', (yargs) => {
        return yargs.option('confirm', {
          alias: 'c',
          type: 'boolean',
          description: 'Подтверждение развертывания'
        });
      }, (argv) => {
        if (argv.confirm) {
          console.log('Развертывание в production...');
        } else {
          console.error('Требуется подтверждение для развертывания в production');
        }
      });
  })
  .argv;
```

## Интеграция с TypeScript

### Типизация опций

```typescript
import yargs from 'yargs';
import { hideBin } from 'yargs/helpers';

interface BuildOptions {
  _: (string | number)[];
  $0: string;
  watch: boolean;
  output?: string;
  minify: boolean;
}

const argv = yargs(hideBin(process.argv))
  .command('build', 'Сборка проекта', (yargs) => {
    return yargs
      .option('watch', {
        alias: 'w',
        type: 'boolean',
        description: 'Наблюдение за изменениями',
        default: false
      })
      .option('output', {
        alias: 'o',
        type: 'string',
        description: 'Путь к выходному файлу'
      })
      .option('minify', {
        alias: 'm',
        type: 'boolean',
        description: 'Минификация результата',
        default: false
      });
  }, (argv: BuildOptions) => {
    console.log('Сборка с опциями:', {
      watch: argv.watch,
      output: argv.output,
      minify: argv.minify
    });
  })
  .argv as BuildOptions;
```

### Типизация аргументов

```typescript
interface ProcessArgs {
  _: (string | number)[];
  $0: string;
  input: string;
  output: string;
}

const argv = yargs(hideBin(process.argv))
  .command('process <input> [output]', 'Обработка файлов', (yargs) => {
    return yargs
      .positional('input', {
        describe: 'Входной файл',
        type: 'string'
      })
      .positional('output', {
        describe: 'Выходной файл',
        type: 'string',
        default: 'output.txt'
      });
  }, (argv: ProcessArgs) => {
    console.log(`Обработка: ${argv.input} -> ${argv.output}`);
  })
  .argv as ProcessArgs;
```

## Продвинутые возможности

### Пользовательские парсеры

```typescript
const argv = yargs(hideBin(process.argv))
  .option('tags', {
    alias: 't',
    type: 'string',
    description: 'Список тегов',
    coerce: (arg: string) => arg.split(',').map(tag => tag.trim())
  })
  .argv;

// Теперь argv.tags будет массивом строк
console.log('Теги:', argv.tags);
```

### Автодополнение

```typescript
const argv = yargs(hideBin(process.argv))
  .completion('completion', 'Генерация скрипта автодополнения')
  .option('environment', {
    alias: 'e',
    type: 'string',
    choices: ['development', 'staging', 'production'],
    description: 'Окружение для развертывания'
  })
  .argv;
```

### Асинхронные команды

```typescript
const argv = yargs(hideBin(process.argv))
  .command('async-command', 'Асинхронная команда', () => {}, async (argv) => {
    try {
      console.log('Выполнение асинхронной команды...');
      // Выполнение асинхронной операции
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Асинхронная команда завершена');
    } catch (error) {
      console.error('Ошибка в асинхронной команде:', error);
      process.exit(1);
    }
  })
  .argv;
```

## Практические примеры

### CLI для управления конфигурацией

```typescript
interface ConfigOptions {
  _: (string | number)[];
  $0: string;
  get?: string;
  set?: string;
  remove?: string;
  list: boolean;
}

const argv = yargs(hideBin(process.argv))
  .usage('$0 <command> [options]')
  .command('config', 'Управление конфигурацией', (yargs) => {
    return yargs
      .option('get', {
        alias: 'g',
        type: 'string',
        description: 'Получить значение конфигурации'
      })
      .option('set', {
        alias: 's',
        type: 'string',
        description: 'Установить значение конфигурации (ключ=значение)'
      })
      .option('remove', {
        alias: 'r',
        type: 'string',
        description: 'Удалить ключ конфигурации'
      })
      .option('list', {
        alias: 'l',
        type: 'boolean',
        description: 'Список всех значений конфигурации',
        default: false
      });
  }, async (argv: ConfigOptions) => {
    if (argv.get) {
      console.log(`Получение значения для: ${argv.get}`);
      // Логика получения значения
    } else if (argv.set) {
      const [key, value] = argv.set.split('=');
      if (!key || !value) {
        throw new Error('Неверный формат установки. Используйте ключ=значение');
      }
      console.log(`Установка ${key}=${value}`);
      // Логика установки значения
    } else if (argv.remove) {
      console.log(`Удаление ключа: ${argv.remove}`);
      // Логика удаления значения
    } else if (argv.list) {
      console.log('Список всех значений конфигурации...');
      // Логика вывода списка
    } else {
      throw new Error('Необходимо указать действие: --get, --set, --remove или --list');
    }
  })
  .help()
  .argv as ConfigOptions;
```

## Лучшие практики

1. **Организация команд**
   - Используйте подкоманды для сложных приложений
   - Называйте команды понятными именами
   - Группируйте связанные функции

2. **Обработка ошибок**
   - Используйте `.check()` для валидации аргументов
   - Обеспечивайте понятные сообщения об ошибках
   - Используйте exit codes для указания ошибок

3. **Документирование**
   - Добавляйте описания ко всем командам и опциям
   - Используйте примеры в справке

```typescript
yargs(hideBin(process.argv))
  .usage('$0 <cmd> [args]')
  .example('$0 build --watch', 'Сборка проекта с наблюдением за изменениями')
  .example('$0 deploy production --confirm', 'Развертывание в production с подтверждением')
  .epilog('Для дополнительной информации посетите: https://example.com')
  .argv;
```

## Сравнение с Commander.js

| Особенность | Yargs | Commander.js |
|-------------|-------|--------------|
| Сложность API | Более сложный | Более простой |
| Функциональность | Более богатый | Более минималистичный |
| Размер бандла | Больше | Меньше |
| Изучение | Дольше | Быстрее |

## Заключение

Yargs предоставляет мощный и гибкий способ создания CLI приложений в Node.js с поддержкой TypeScript. Он особенно полезен для сложных CLI с множеством опций и команд.

См. также:
- [[Command-line-интерфейсы]]
- [[Commander-js]]
- [[Интеграция-с-CLI-библиотеками]]
- [[Публикация-CLI-инструментов]]
- [[Node.js-с-TypeScript]]
- [[TypeScript-инструменты-и-библиотеки]]
