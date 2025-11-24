---
aliases: ["Отладка в IDE", "IDE Debugging", "Отладка в редакторе"]
tags: [typescript, debugging, ide, vscode, development]
---

# Отладка TypeScript в IDE

Отладка TypeScript в интегрированной среде разработки (IDE) - это важный этап разработки, позволяющий эффективно находить и устранять ошибки. Современные IDE, такие как Visual Studio Code, WebStorm и другие, предоставляют мощные инструменты для отладки TypeScript-приложений.

## Основы отладки в IDE

Отладка TypeScript в IDE включает в себя:
- Установку точек останова
- Пошаговое выполнение кода
- Просмотр значений переменных
- Анализ стека вызовов
- Оценку выражений в контексте выполнения

## Настройка отладки в Visual Studio Code

Visual Studio Code предоставляет встроенную поддержку отладки TypeScript через встроенный отладчик и расширения.

### 1. Файл конфигурации launch.json

Создайте файл `.vscode/launch.json` в корне проекта:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug TypeScript",
      "program": "${workspaceFolder}/dist/index.js",
      "sourceMaps": true,
      "outFiles": ["${workspaceFolder}/dist/**/*.js"],
      "preLaunchTask": "tsc: build - tsconfig.json"
    }
  ]
}
```

### 2. Настройка tsconfig.json

Убедитесь, что в `tsconfig.json` включена генерация Source Map:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,
    "inlineSourceMap": false,
    "inlineSources": true
  }
}
```

### 3. Запуск отладки

- Установите точки останова в TypeScript-файлах
- Выберите конфигурацию отладки
- Нажмите F5 или кнопку "Отладка"

## Основные функции отладки

### Точки останова

Точки останова позволяют приостановить выполнение кода в определенных местах:

```typescript
function calculateTotal(items: { price: number; quantity: number }[]): number {
  let total = 0;
  for (const item of items) {
    total += item.price * item.quantity; // Установите точку останова здесь
  }
  return total;
}
```

### Условные точки останова

Можно задать условие, при котором точка останова будет активирована:

```typescript
for (let i = 0; i < items.length; i++) {
  const item = items[i];
  // Установите условную точку останова: i === 5
  console.log(item);
}
```

### Лог-точки

Лог-точки позволяют выводить сообщения в консоль без прерывания выполнения:

```
Item at index {i} has price {item.price}
```

## Отладка в разных средах

### Node.js приложения

Для отладки Node.js приложений с TypeScript:

```json
{
  "type": "node",
  "request": "launch",
  "name": "Debug Node.js with TypeScript",
  "program": "${workspaceFolder}/src/index.ts",
  "runtimeArgs": ["-r", "ts-node/register"],
  "env": {
    "NODE_ENV": "development"
  }
}
```

### Frontend приложения

Для отладки frontend TypeScript приложений (React, Vue и т.д.) через браузер:

```json
{
  "type": "pwa-chrome",
  "request": "launch",
  "name": "Launch Chrome against localhost",
  "url": "http://localhost:3000",
  "webRoot": "${workspaceFolder}/src",
  "sourceMapPathOverrides": {
    "webpack:///src/*": "${webRoot}/*"
  }
}
```

## Интеграция с Webpack

При использовании Webpack важно правильно настроить Source Map:

```javascript
module.exports = {
  mode: 'development',
  devtool: 'source-map', // или 'inline-source-map'
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  }
};
```

## Продвинутые техники отладки

### Отладка async/await

При отладке асинхронного кода важно понимать, как стек вызовов работает с промисами:

```typescript
async function fetchData(): Promise<any> {
  try {
    const response = await fetch('/api/data'); // Точка останова здесь
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error); // Или здесь
    throw error;
  }
}
```

### Отладка с использованием декораторов

Для сложных приложений с декораторами может потребоваться особый подход к отладке:

```typescript
function logMethod(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function (...args: any[]) {
    console.log(`${propertyKey} with args:`, args); // Точка останова для отладки декоратора
    return originalMethod.apply(this, args);
  };
}
```

## Плагины и расширения

### TypeScript IDE Tools

- **TypeScript Importer** - автоматически добавляет импорты
- **Bracket Pair Colorizer** - выделяет парные скобки
- **Error Lens** - показывает ошибки прямо в коде
- **Path Intellisense** - автодополнение путей к файлам

### Отладочные расширения

- **Debugger for Chrome** - отладка в Chrome
- **Debugger for Firefox** - отладка в Firefox
- **Node.js V8 Inspector** - отладка Node.js приложений

## Полезные сочетания клавиш

- `F5` - начать/продолжить отладку
- `F9` - переключить точку останова
- `F10` - шаг с обходом
- `F11` - шаг с заходом
- `Shift+F11` - шаг с выходом
- `Ctrl+Shift+D` - открыть панель отладки

## Отладка с типами

Особенность отладки TypeScript заключается в том, что IDE может отображать типы переменных в процессе отладки:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function processUser(user: User) {
  // В отладчике можно увидеть тип переменной user
  console.log(user);
}
```

## Решение проблем

### Проблема: Точки останова не срабатывают

Возможные причины:
- Source Map не сгенерированы или недоступны
- Неправильный путь к файлам
- Отладчик запущен для скомпилированного JavaScript, а не TypeScript

Решения:
- Проверьте настройки `sourceMap` в `tsconfig.json`
- Убедитесь, что пути в `launch.json` указаны правильно
- Используйте `preLaunchTask` для компиляции перед запуском отладки

> [!tip] Совет
> Используйте `inlineSources: true` в tsconfig.json, чтобы включить исходный код TypeScript прямо в Source Map файлы.

## Заключение

Отладка TypeScript в IDE - это мощный инструмент, который значительно упрощает процесс разработки. Правильная настройка позволяет эффективно находить и устранять ошибки, анализировать поведение кода и улучшать качество программного обеспечения.

См. также:
- [[Source-map]]
- [[Инструменты-отладки]]
- [[Типизированные-ошибки]]
- [[Логирование-с-типами]]