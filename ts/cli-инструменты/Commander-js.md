---
aliases: [Commander.js, commander]
tags: [typescript, cli, commander, nodejs]
---

# Commander.js

Commander.js - это популярная библиотека для создания Command-line интерфейсов в Node.js. Она предоставляет простой и интуитивно понятный API для определения команд, опций и аргументов.

## Введение в Commander.js

Commander.js позволяет разработчикам быстро создавать полнофункциональные CLI приложения с минимальным количеством кода. Она автоматически генерирует справку и обрабатывает аргументы командной строки.

### Установка

```bash
npm install commander
# Для TypeScript также установите типы:
npm install @types/commander
```

## Основы использования

### Базовая структура

```typescript
#!/usr/bin/env node

import { Command } from 'commander';
import { version } from '../package.json';

const program = new Command();

program
  .name('my-app')
  .description('CLI приложение для демонстрации Commander.js')
  .version(version);

program.parse();
```

### Добавление команд

```typescript
program
  .command('build')
  .description('Сборка проекта')
  .option('-w, --watch', 'наблюдение за изменениями')
  .option('-o, --output <path>', 'путь к выходному файлу')
  .action((options) => {
    console.log('Выполняется сборка проекта...');
    if (options.watch) {
      console.log('Включено наблюдение за изменениями');
    }
    if (options.output) {
      console.log(`Выходной путь: ${options.output}`);
    }
  });
```

## Продвинутое использование

### Подкоманды

```typescript
const deployCommand = program
  .command('deploy')
  .description('Команды для развертывания');

deployCommand
  .command('staging')
  .description('Развертывание в staging окружение')
  .option('-f, --force', 'принудительное развертывание')
  .action((options) => {
    console.log('Развертывание в staging...');
    if (options.force) {
      console.log('Принудительное развертывание включено');
    }
  });

deployCommand
  .command('production')
  .description('Развертывание в production окружение')
  .option('-c, --confirm', 'подтверждение развертывания')
  .action((options) => {
    if (options.confirm) {
      console.log('Развертывание в production...');
    } else {
      console.error('Требуется подтверждение для развертывания в production');
    }
  });
```

### Валидация аргументов

```typescript
import { Argument } from 'commander';

program
  .command('process')
  .description('Обработка данных')
  .argument('<type>', 'тип данных: json, xml, csv')
  .argument('<input-file>', 'входной файл')
  .option('-o, --output <output-file>', 'выходной файл')
  .action((type, inputFile, options) => {
    // Валидация типа данных
    const validTypes = ['json', 'xml', 'csv'];
    if (!validTypes.includes(type)) {
      throw new Error(`Неверный тип данных. Допустимые значения: ${validTypes.join(', ')}`);
    }

    console.log(`Обработка ${type} файла: ${inputFile}`);
    if (options.output) {
      console.log(`Результат будет сохранен в: ${options.output}`);
    }
  });
```

### Пользовательские валидаторы

```typescript
function parsePort(value: string): number {
  const parsedValue = parseInt(value, 10);
  if (isNaN(parsedValue)) {
    throw new Error('Порт должен быть числом');
  }
  if (parsedValue < 1 || parsedValue > 65535) {
    throw new Error('Порт должен быть в диапазоне от 1 до 65535');
  }
  return parsedValue;
}

program
  .command('serve')
  .description('Запуск сервера')
  .option('-p, --port <number>', 'порт для сервера', parsePort)
  .option('-h, --host <host>', 'хост для сервера', 'localhost')
  .action((options) => {
    console.log(`Сервер запущен на ${options.host}:${options.port}`);
  });
```

## Интеграция с TypeScript

### Типизация опций

```typescript
interface BuildOptions {
  watch: boolean;
  output?: string;
  minify: boolean;
}

program
  .command('build')
  .description('Сборка проекта')
  .option('-w, --watch', 'наблюдение за изменениями')
  .option('-o, --output <path>', 'путь к выходному файлу')
  .option('-m, --minify', 'минификация результата')
  .action((options: BuildOptions) => {
    // TypeScript теперь знает типы опций
    console.log('Сборка с опциями:', {
      watch: options.watch,
      output: options.output,
      minify: options.minify
    });
  });
```

### Типизация аргументов

```typescript
interface ProcessArgs {
  type: string;
  inputFile: string;
}

interface ProcessOptions {
  output?: string;
  verbose: boolean;
}

program
  .command('process')
  .description('Обработка данных')
  .argument('<type>', 'тип данных: json, xml, csv')
  .argument('<input-file>', 'входной файл')
  .option('-o, --output <output-file>', 'выходной файл')
  .option('-v, --verbose', 'подробный вывод')
  .action((type: string, inputFile: string, options: ProcessOptions) => {
    const args: ProcessArgs = { type, inputFile };
    console.log('Аргументы:', args);
    console.log('Опции:', options);
  });
```

## Практические примеры

### CLI для управления задачами

```typescript
interface TaskOptions {
  priority: 'low' | 'medium' | 'high';
  assignee?: string;
}

program
  .command('task')
  .description('Управление задачами')
  .argument('<action>', 'действие: add, complete, list')
  .argument('[task-name]', 'название задачи')
  .option('-p, --priority <level>', 'приоритет задачи', 'medium')
  .option('-a, --assignee <name>', 'назначить задачу')
  .action((action, taskName, options: TaskOptions) => {
    switch (action) {
      case 'add':
        if (!taskName) {
          throw new Error('Необходимо указать название задачи');
        }
        console.log(`Добавление задачи: ${taskName}`);
        console.log(`Приоритет: ${options.priority}`);
        if (options.assignee) {
          console.log(`Назначена: ${options.assignee}`);
        }
        break;
        
      case 'complete':
        if (!taskName) {
          throw new Error('Необходимо указать название задачи');
        }
        console.log(`Задача завершена: ${taskName}`);
        break;
        
      case 'list':
        console.log('Список задач...');
        break;
        
      default:
        throw new Error(`Неизвестное действие: ${action}`);
    }
  });
```

## Лучшие практики

1. **Организация команд**
   - Используйте подкоманды для сложных приложений
   - Называйте команды понятными именами
   - Группируйте связанные функции в подкоманды

2. **Обработка ошибок**
   - Валидируйте входные данные
   - Обеспечивайте понятные сообщения об ошибках
   - Используйте exit codes для указания ошибок

3. **Документирование**
   - Добавляйте описания ко всем командам и опциям
   - Используйте примеры в справке

```typescript
program
  .command('example')
  .description('Пример команды с хорошим описанием')
  .option('-v, --verbose', 'подробный вывод')
  .addHelpText('after', `
Примеры:
  $ my-cli example
  $ my-cli example --verbose
  `);
```

## Заключение

Commander.js предоставляет мощный и гибкий способ создания CLI приложений в Node.js с поддержкой TypeScript. Она упрощает обработку аргументов, валидацию и создание интуитивно понятного интерфейса командной строки.

См. также:
- [[Command-line-интерфейсы]]
- [[Yargs]]
- [[Интеграция-с-CLI-библиотеками]]
- [[Публикация-CLI-инструментов]]
- [[Node.js-с-TypeScript]]
- [[TypeScript-инструменты-и-библиотеки]]
