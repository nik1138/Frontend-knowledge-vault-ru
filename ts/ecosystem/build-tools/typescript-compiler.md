# TypeScript Compiler (tsc)

TypeScript Compiler (tsc) - это официальный компилятор TypeScript, который преобразует TypeScript код в JavaScript, выполняя проверку типов, транспиляцию синтаксиса и генерацию дополнительных файлов (sourcemaps, declaration файлы и т.д.).

## Основные возможности

### Компиляция и проверка типов
```bash
# Основная компиляция
tsc

# Компиляция с указанием конфигурации
tsc --project tsconfig.json

# Компиляция конкретного файла
tsc src/main.ts

# Компиляция с опциями
tsc --target ES2020 --module commonjs --outDir dist src/main.ts
```

### Опции командной строки
```typescript
// Пример конфигурации через командную строку
tsc \
  --target ES2020 \          # Уровень ECMAScript
  --module ESNext \          # Система модулей  
  --lib ES2020,DOM \         # Библиотеки для компиляции
  --outDir dist \            # Выходная директория
  --rootDir src \            # Корневая директория исходников
  --strict \                 # Строгая проверка типов
  --esModuleInterop \        # Совместимость ES модулей
  --skipLibCheck \           # Пропустить проверки .d.ts файлов
  --forceConsistentCasingInFileNames  # Проверка регистра имен файлов
```

## Конфигурация компилятора

### tsconfig.json структура
```json
{
  "compilerOptions": {
    /* Основные опции */
    "target": "ES2020",                          /* Уровень ECMAScript для вывода */
    "module": "ESNext",                         /* Система модулей для вывода */
    "lib": ["ES2020", "DOM", "DOM.Iterable"],   /* Целевые библиотеки */
    "allowJs": true,                            /* Позволить JS файлы в компиляции */
    "checkJs": false,                           /* Проверить JS файлы при компиляции */
    
    /* Системы модулей */
    "moduleResolution": "node",                 /* Стратегия разрешения модулей */
    "baseUrl": ".",                            /* Базовый каталог для некорректных модулей */
    "paths": {                                 /* Псевдонимы для путей к модулям */
      "@/*": ["src/*"],
      "@utils/*": ["src/utils/*"]
    },
    "rootDirs": ["src", "generated"],          /* Логические корневые каталоги */
    "typeRoots": ["./node_modules/@types"],    /* Каталоги для поиска типов */
    "types": ["node", "jest"],                 /* Включить только указанные типы */
    
    /* Объекты исходного кода */
    "declaration": true,                       /* Генерировать .d.ts файлы */
    "declarationMap": true,                    /* Генерировать .d.ts map файлы */
    "sourceMap": true,                         /* Генерировать .map файлы */
    "outFile": "./dist/bundle.js",             /* Вывести все в один файл */
    "outDir": "./dist",                        /* Вывести в директорию */
    "composite": true,                         /* Включить проектные ссылки */
    "tsBuildInfoFile": "./dist/.tsbuildinfo",  /* Файл для инкрементных билдов */
    "removeComments": true,                    /* Удалить комментарии из вывода */
    "noEmit": false,                           /* Не выводить файлы */
    "importHelpers": true,                     /* Импортировать хелперы из tslib */
    "downlevelIteration": true,                /* Предоставить полную поддержку итерации */
    
    /* Строгие проверки */
    "strict": true,                            /* Включить все строгие проверки */
    "noImplicitAny": true,                     /* Ошибка при неявном any */
    "strictNullChecks": true,                  /* Строгая проверка null/undefined */
    "strictFunctionTypes": true,               /* Строгая проверка типов функций */
    "strictBindCallApply": true,               /* Строгая проверка bind/call/apply */
    "strictPropertyInitialization": true,      /* Строгая проверка инициализации свойств */
    "noImplicitThis": true,                    /* Ошибка при неявном this */
    "useUnknownInCatchVariables": true,        /* Catch переменные как unknown */
    
    /* Дополнительные проверки */
    "noUnusedLocals": true,                    /* Ошибка при неиспользуемых локальных переменных */
    "noUnusedParameters": true,                /* Ошибка при неиспользуемых параметрах */
    "noImplicitReturns": true,                 /* Каждый путь кода должен возвращать значение */
    "noFallthroughCasesInSwitch": true,        /* Ошибка при пропуске break в switch */
    "noUncheckedIndexedAccess": true,          /* Доступ к индексу возвращает T | undefined */
    "noImplicitOverride": true,                /* Требовать override при переопределении */
    "allowUnreachableCode": false,             /* Ошибка при недостижимом коде */
    
    /* Совместимость */
    "allowSyntheticDefaultImports": true,      /* Импортировать не-CommonJS как default импорт */
    "esModuleInterop": true,                   /* Включить совместимость ES Module */
    "forceConsistentCasingInFileNames": true,  /* Чувствительность к регистру имен файлов */
    
    /* Дополнительные возможности */
    "experimentalDecorators": true,            /* Поддержка ES7 декораторов */
    "emitDecoratorMetadata": true,             /* Выпуск метаданных для декораторов */
    "resolveJsonModule": true,                 /* Импортировать .json файлы */
    "allowArbitraryExtensions": false,         /* Позволить произвольные расширения файлов */
    "noEmitOnError": true,                     /* Не выводить при наличии ошибок */
    
    /* Оптимизации */
    "incremental": true,                       /* Включить инкрементную компиляцию */
    "assumeChangesOnlyAffectDirectDependencies": true, /* Быстрая инкрементная компиляция */
    "diagnostics": true                        /* Включить диагностику производительности */
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ],
  "references": [
    { "path": "./shared" }
  ]
}
```

## Режимы компиляции

### Strict Mode
```json
{
  "compilerOptions": {
    "strict": true,
    // Это включает все следующие проверки:
    "noImplicitAny": true,
    "noImplicitThis": true, 
    "alwaysStrict": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "strictBindCallApply": true,
    "useUnknownInCatchVariables": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### Build Mode
```json
// tsconfig.base.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "incremental": true,
    "tsBuildInfoFile": "./tsconfig.tsbuildinfo"
  }
}

// tsconfig.app.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist/app"
  },
  "references": [
    { "path": "../shared" }
  ]
}
```

## Практические сценарии использования

### Компиляция в разные цели
```json
{
  "compilerOptions": {
    "target": "ES2017",        // для Node.js 12+
    "module": "commonjs",      // для Node.js
    "moduleResolution": "node",
    "outDir": "./dist-node",
    "lib": ["ES2017", "DOM"]
  }
}
```

```json
{
  "compilerOptions": {
    "target": "ES2020",        // для современных браузеров
    "module": "ESNext",        // для модулей браузера  
    "moduleResolution": "node",
    "outDir": "./dist-browser",
    "lib": ["ES2020", "DOM", "DOM.Iterable"]
  }
}
```

### Компиляция библиотеки
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,           // генерировать .d.ts файлы
    "declarationMap": true,        // маппинги для .d.ts файлов
    "sourceMap": false,            // не нужен в библиотеках (только для .d.ts)
    "importHelpers": true,         // использовать tslib
    "downlevelIteration": true,     // полная поддержка итерации
    "stripInternal": true,         // удалять @internal комментарии
    "noEmitOnError": true,
    "composite": true,
    "removeComments": false        // оставлять JSDoc комментарии
  }
}
```

## Интеграция с инструментами

### tsc с webpack
```javascript
// webpack.config.js
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  devServer: {
    hot: true
  },
  plugins: [
    new ForkTsCheckerWebpackPlugin({
      typescript: {
        configFile: path.resolve(__dirname, 'tsconfig.json'),
        diagnosticOptions: {
          semantic: true,
          syntactic: true
        }
      }
    })
  ]
};
```

### Watch mode
```bash
# Компиляция в режиме отслеживания изменений
tsc --watch

# или
tsc -w
```

## Performance и оптимизации

### Инкрементная компиляция
```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "composite": true,
    // Оптимизации для производительности:
    "assumeChangesOnlyAffectDirectDependencies": true
  }
}
```

### Skip type checking
```json
{
  "compilerOptions": {
    // При разработке для ускорения:
    "skipLibCheck": true,  // пропускать тип-чекинг библиотек
    "noEmit": true        // только проверка типов, без вывода
  }
}
```

## Расширенные возможности

### Project References
```json
// tsconfig.json для проекта с ссылками
{
  "references": [
    { "path": "./core" },
    { "path": "./utils" },
    { "path": "./api" }
  ],
  "files": []  // используем references
}
```

### Build Optimizations
```json
{
  "compilerOptions": {
    "importHelpers": true,         // используем tslib helpers
    "removeComments": true,        // удаляем комментарии из вывода
    "newLine": "lf",              // нормализуем переводы строк на LF
    "stripInternal": true,         // удаляем внутренние API (@internal)
    "preserveConstEnums": false,   // компилируем const enums как обычные enums
    "sourceMap": true,             // для отладки
    "inlineSources": true          // встраиваем исходники в sourcemaps
  }
}
```

## Практические примеры

### Скрипт автоматической компиляции
```typescript
// build-script.ts
import * as ts from 'typescript';
import * as path from 'path';

function compileProject(tsconfigPath: string): void {
  const configFile = ts.readConfigFile(tsconfigPath, ts.sys.readFile);
  const config = ts.parseJsonConfigFileContent(
    configFile.config,
    ts.sys,
    path.dirname(tsconfigPath)
  );
  
  const program = ts.createProgram({
    rootNames: config.fileNames,
    options: config.options
  });
  
  const emitResult = program.emit();
  
  const allDiagnostics = ts
    .getPreEmitDiagnostics(program)
    .concat(emitResult.diagnostics);
  
  allDiagnostics.forEach(diagnostic => {
    if (diagnostic.file) {
      const { line, character } = ts.getLineAndCharacterOfPosition(
        diagnostic.file,
        diagnostic.start!
      );
      const message = ts.flattenDiagnosticMessageText(diagnostic.messageText, '\n');
      console.log(`${diagnostic.file.fileName} (${line + 1},${character + 1}): ${message}`);
    } else {
      console.log(ts.flattenDiagnosticMessageText(diagnostic.messageText, '\n'));
    }
  });
  
  const exitCode = emitResult.emitSkipped ? 1 : 0;
  process.exit(exitCode);
}

// Использование
compileProject('./tsconfig.json');
```

## Лучшие практики

1. **Используйте строгий режим** (`"strict": true`) для максимальной безопасности
2. **Отдельные конфигурации** для разных целей (приложение, библиотека, тесты)
3. **Проектные ссылки** для больших кодовых баз
4. **Инкрементную компиляцию** для ускорения разработки
5. **Используйте sourcemaps** в разработке для отладки
6. **Генерируйте declaration файлы** для библиотек
7. **Проверяйте только типы** (`tsc --noEmit`) в CI перед билдом

## Ошибки и ловушки

### Общие проблемы
```typescript
// 1. Проблема с файлами в node_modules
// tsconfig.json
{
  "compilerOptions": {
    "skipLibCheck": true  // решает проблемы с несовместимыми библиотеками
  }
}

// 2. Проблема с путями
{
  "compilerOptions": {
    "baseUrl": ".",      // установите базовый URL
    "paths": {
      "@/*": ["src/*"]  // решает проблемы с абсолютными импортами
    }
  }
}
```

## Связь с другими концепциями
- [[../modules/module-resolution]] - как tsc разрешает модули
- [[../compilation/configuration]] - настройка компиляции
- [[../type-system/type-checking]] - проверка типов компилятором
- [[../ecosystem/build-tools]] - интеграция с другими инструментами сборки
- [[../deployment/targets]] - цели компиляции для разных сред