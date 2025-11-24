---
aliases: [Neutralino Framework, Neutralino.js]
tags: [framework, desktop-applications, lightweight, typescript]
---

# Neutralino

## Обзор

Neutralino - это легковесный фреймворк с открытым исходным кодом для создания кроссплатформенных десктопных приложений с использованием веб-технологий (HTML, CSS, JavaScript/TypeScript). В отличие от Electron, Neutralino не включает в себя движок Chromium, а использует системный веб-просмотр, что делает приложения значительно легче и быстрее.

## Архитектура Neutralino

Neutralino состоит из двух основных компонентов:

- **Frontend**: Веб-интерфейс (HTML, CSS, JavaScript/TypeScript)
- **Backend**: Нативный движок на C++, обеспечивающий доступ к системным ресурсам

### Основные особенности

- **Маленький размер**: Около 10-20 МБ в сравнении с 100+ МБ у Electron
- **Низкое потребление памяти**: Использование системного веб-просмотра
- **Кроссплатформенность**: Windows, macOS, Linux
- **Нет зависимости от Chromium**: Меньше проблем с лицензированием

## Установка и настройка

### 1. Установка Neutralino CLI

```bash
npm install -g @neutralinojs/neu
```

### 2. Создание нового проекта

```bash
# Создание нового проекта
neu create myapp
cd myapp

# Запуск в режиме разработки
neu run

# Сборка приложения
neu build
```

## Структура проекта

```
myapp/
├── neutralino.config.json    # Конфигурация Neutralino
├── index.html               # Главная страница
├── app.js                   # Основной скрипт приложения
├── styles.css               # Стили приложения
└── resources/               # Дополнительные ресурсы
```

## Пример приложения

### Конфигурация (neutralino.config.json)

```json
{
  "applicationId": "js.neutralino.sample",
  "version": "1.0.0",
  "defaultMode": "window",
  "port": 0,
  "documentRoot": "/resources/",
  "url": "/resources/index.html",
  "enableServer": true,
  "enableNativeAPI": true,
  "tokenSecurity": "one-time",
  "modes": {
    "window": {
      "title": "Neutralino Sample",
      "width": 800,
      "height": 600,
      "minWidth": 400,
      "minHeight": 300,
      "icon": "/resources/icons/appIcon.png"
    }
  },
  "cli": {
    "binaryName": "myapp",
    "resourcesPath": "/resources/",
    "extensionsPath": "/extensions/",
    "clientLibrary": "/resources/js/neutralino.js"
  }
}
```

### Frontend (TypeScript)

```typescript
// app.ts
import { app, window, filesystem, os } from "@neutralinojs/lib";

// Проверка готовности Neutralino
Neutralino.init({
  load: async () => {
    console.log("Neutralino запущен!");
    
    // Пример использования API
    const info = await app.getInfo();
    console.log("Информация о приложении:", info);
    
    // Установка заголовка окна
    await window.setTitle(`Neutralino App v${info.version}`);
  },
  exit: () => {
    console.log("Приложение завершает работу");
  }
});

// Функция для показа информации о системе
async function showSystemInfo() {
  try {
    const platform = await os.getPlatform();
    const homeDir = await os.getHomeDirectory();
    const configDir = await os.getConfigDirectory();
    
    console.log(`Платформа: ${platform}`);
    console.log(`Домашняя директория: ${homeDir}`);
    console.log(`Конфигурационная директория: ${configDir}`);
  } catch (error) {
    console.error("Ошибка при получении информации о системе:", error);
  }
}

// Функция для работы с файловой системой
async function readFileContent(filePath: string) {
  try {
    const fileContent = await filesystem.readFile({
      filePath: filePath
    });
    
    console.log("Содержимое файла:", fileContent.data);
    return fileContent.data;
  } catch (error) {
    console.error("Ошибка при чтении файла:", error);
    return null;
  }
}

// Пример обработки события
document.getElementById('btnSystemInfo')?.addEventListener('click', showSystemInfo);
```

### HTML (index.html)

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Neutralino Sample</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>Добро пожаловать в Neutralino</h1>
        <button id="btnSystemInfo">Показать информацию о системе</button>
        <div id="output"></div>
    </div>
    
    <script src="neutralino.js"></script>
    <script src="app.js"></script>
</body>
</html>
```

## API Neutralino

### 1. Работа с приложением

```typescript
import { app } from "@neutralinojs/lib";

// Получение информации о приложении
const info = await app.getInfo();

// Завершение приложения
await app.exit();

// Перезапуск приложения
await app.restart();
```

### 2. Работа с окнами

```typescript
import { window } from "@neutralinojs/lib";

// Управление окном
await window.setTitle("Новый заголовок");
await window.setSize({ width: 1000, height: 800 });
await window.setFullScreen(true);
await window.focus();
```

### 3. Работа с файловой системой

```typescript
import { filesystem } from "@neutralinojs/lib";

// Чтение файла
const fileContent = await filesystem.readFile({
  filePath: "/path/to/file.txt"
});

// Запись в файл
await filesystem.writeFile({
  filePath: "/path/to/output.txt",
  data: "Содержимое файла"
});

// Создание директории
await filesystem.createDirectory({
  path: "/path/to/new/directory"
});
```

## Преимущества Neutralino

- **Маленький размер приложения**: Около 10-20 МБ
- **Низкое потребление памяти**: Использование системного веб-просмотра
- **Быстрая загрузка**: Быстрее запускается по сравнению с Electron
- **Нет зависимости от Chromium**: Меньше проблем с лицензированием
- **Простота разработки**: Использование привычных веб-технологий

## Недостатки Neutralino

- **Меньшее сообщество**: По сравнению с Electron
- **Ограниченные возможности**: Меньше встроенных функций по сравнению с Electron
- **Меньше готовых решений**: Меньше плагинов и библиотек
- **Меньше документации**: Документация может быть менее полной

## Практические рекомендации

### 1. Оптимизация производительности

```typescript
// Используйте асинхронные вызовы для тяжелых операций
async function processLargeFile(filePath: string) {
  try {
    // Показываем индикатор загрузки
    showLoadingIndicator();
    
    // Выполняем асинхронную операцию
    const result = await filesystem.readFile({ filePath });
    
    // Обновляем интерфейс с результатом
    updateUI(result.data);
  } catch (error) {
    showError(error.message);
  } finally {
    hideLoadingIndicator();
  }
}
```

### 2. Обработка ошибок

```typescript
// Обработка ошибок при вызове API
async function safeAPIOperation() {
  try {
    const result = await someNeutralinoAPI();
    return result;
  } catch (error) {
    console.error('Ошибка при вызове API:', error);
    
    // Логируем ошибку и возвращаем значение по умолчанию
    return null;
  }
}
```

### 3. Работа с событиями

```typescript
import { events } from "@neutralinojs/lib";

// Подписка на события приложения
events.on("windowClose", async () => {
  console.log("Окно закрывается");
  
  // Сохраняем данные перед закрытием
  await saveUserData();
});

// Подписка на пользовательские события
events.on("customEvent", (data) => {
  console.log("Пользовательское событие:", data);
});
```

## Связанные темы

- [[Electron]] - Альтернативный фреймворк для создания десктопных приложений
- [[Tauri]] - Современный фреймворк для десктопных приложений с Rust бэкендом
- [[Десктопные-паттерны]] - Паттерны проектирования для десктопных приложений
- [[Оптимизация-для-десктопа]] - Рекомендации по оптимизации десктопных приложений
- [[Webpack]] - Сборка Neutralino-приложений

## Заключение

Neutralino представляет собой отличную альтернативу Electron для разработчиков, которые ценят производительность и малый размер приложений. Он идеально подходит для легких приложений, где не требуется вся мощь Electron, но нужно использовать веб-технологии для интерфейса.