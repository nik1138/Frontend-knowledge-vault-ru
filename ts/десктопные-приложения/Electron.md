---
aliases: [Electron Framework, Electron.js]
tags: [framework, desktop-applications, javascript, typescript]
---

# Electron

## Обзор

Electron - это фреймворк с открытым исходным кодом, разработанный GitHub, который позволяет создавать кроссплатформенные десктопные приложения с использованием веб-технологий: HTML, CSS и JavaScript (или TypeScript). Он позволяет разработчикам использовать уже знакомые веб-технологии для создания нативных приложений, работающих на Windows, macOS и Linux.

## Архитектура Electron

Electron использует архитектуру с двумя процессами:

- **Main Process** (главный процесс): Управляет жизненным циклом приложения, создает окна и взаимодействует с системой.
- **Renderer Process** (рендер-процесс): Отвечает за отображение пользовательского интерфейса в окне браузера.

### Main Process

```typescript
import { app, BrowserWindow } from 'electron';
import * as path from 'path';

let mainWindow: BrowserWindow | null;

function createWindow() {
  mainWindow = new BrowserWindow({
    height: 800,
    webPreferences: {
      nodeIntegration: true,
      contextIsolation: false,
      preload: path.join(__dirname, 'preload.js'),
    },
    width: 1200,
  });

  if (app.isPackaged) {
    mainWindow.loadFile(path.join(__dirname, '../renderer/index.html'));
  } else {
    mainWindow.loadURL('http://localhost:3000');
    mainWindow.webContents.openDevTools();
  }

  mainWindow.on('closed', () => {
    mainWindow = null;
  });
}

app.whenReady().then(() => {
  createWindow();

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow();
    }
  });
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});
```

### Renderer Process

```typescript
// renderer.ts
import { ipcRenderer } from 'electron';

// Пример взаимодействия с main процессом
const button = document.getElementById('myButton');
button?.addEventListener('click', () => {
  ipcRenderer.send('button-clicked', 'Hello from renderer');
});

// Слушаем ответ от main процесса
ipcRenderer.on('response', (event, data) => {
  console.log('Получено сообщение от main:', data);
});
```

## Преимущества Electron

- **Кроссплатформенность**: Одна кодовая база для Windows, macOS и Linux
- **Широкое сообщество**: Большое количество плагинов и готовых решений
- **Простота разработки**: Использование привычных веб-технологий
- **Интеграция с Node.js**: Доступ к системным ресурсам и файловой системе

## Недостатки Electron

- **Высокое потребление памяти**: Приложения Electron могут использовать больше памяти по сравнению с нативными приложениями
- **Размер исполняемого файла**: Результатирующий бинарный файл может быть довольно большим
- **Производительность**: В некоторых случаях может быть ниже, чем у нативных приложений

## Практические рекомендации

### 1. Безопасность

```typescript
// preload.ts
import { contextBridge, ipcRenderer } from 'electron';

// Безопасный экспорт API в renderer процесс
contextBridge.exposeInMainWorld('electronAPI', {
  sendMessage: (message: string) => ipcRenderer.invoke('send-message', message),
  onMessage: (callback: (event: any, message: string) => void) => {
    ipcRenderer.on('message-response', callback);
  }
});
```

### 2. Оптимизация производительности

- Используйте `contextIsolation: true` для безопасности
- Минимизируйте использование Node.js API в renderer процессе
- Оптимизируйте загрузку ресурсов

### 3. Работа с файловой системой

```typescript
// main.ts
import { dialog, ipcMain } from 'electron';
import * as fs from 'fs';

ipcMain.handle('open-file-dialog', async () => {
  const result = await dialog.showOpenDialog({
    properties: ['openFile'],
    filters: [
      { name: 'Text Files', extensions: ['txt'] },
      { name: 'All Files', extensions: ['*'] }
    ]
  });
  
  if (!result.canceled) {
    const content = fs.readFileSync(result.filePaths[0], 'utf-8');
    return content;
  }
  
  return null;
});
```

## Связанные темы

- [[Tauri]] - Альтернативный фреймворк для создания десктопных приложений
- [[Десктопные-паттерны]] - Паттерны проектирования для десктопных приложений
- [[Оптимизация-для-десктопа]] - Рекомендации по оптимизации десктопных приложений
- [[Node.js с TypeScript]] - Использование Node.js в десктопных приложениях
- [[Webpack]] - Сборка Electron-приложений

## Заключение

Electron остается одним из самых популярных решений для создания кроссплатформенных десктопных приложений с использованием веб-технологий. Несмотря на некоторые недостатки, он предоставляет мощную платформу для быстрой разработки приложений с богатым пользовательским интерфейсом.