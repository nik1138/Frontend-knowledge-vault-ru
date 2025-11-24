---
aliases: [DevTools Extension, Расширения для разработчиков]
tags: [browser-extension, devtools, chrome-extension, web-extension, typescript, debugging]
---

# DevTools-расширения: Инструменты разработчика для браузера

## Обзор

DevTools-расширения - это специальный тип расширений браузера, которые интегрируются с встроенными инструментами разработчика (DevTools). Они позволяют разработчикам расширять функциональность DevTools, добавлять новые панели, инспектировать специфические данные и улучшать процесс отладки веб-приложений.

## Архитектура DevTools-расширений

DevTools-расширения состоят из нескольких компонентов:

1. **devtools.html** - точка входа для DevTools
2. **devtools.ts** - основной скрипт для взаимодействия с DevTools
3. **panel.html** - пользовательский интерфейс для новой панели
4. **panel.ts** - логика для пользовательской панели
5. **content-script.ts** - для взаимодействия с веб-страницей
6. **background.ts** - для управления общим состоянием

## Объявление в манифесте

```json
{
  "manifest_version": 3,
  "name": "Пример DevTools расширения",
  "version": "1.0.0",
  "devtools_page": "devtools.html",
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content-script.js"]
    }
  ],
  "permissions": [
    "activeTab",
    "storage"
  ]
}
```

## Структура DevTools-расширения

### 1. devtools.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>DevTools Extension</title>
</head>
<body>
  <script src="devtools.js"></script>
</body>
</html>
```

### 2. devtools.ts

```typescript
// devtools.ts
import { PanelController } from './panel-controller';

// Инициализация DevTools расширения
function initializeDevTools(): void {
  // Проверяем, открыты ли DevTools
  if (typeof chrome !== 'undefined' && chrome.devtools) {
    console.log('DevTools расширение инициализировано');
    
    // Добавляем новую панель в DevTools
    chrome.devtools.panels.create(
      'Моё расширение', // Название панели
      'icon16.png',     // Иконка (опционально)
      'panel.html',     // HTML файл для содержимого панели
      (panel) => {
        console.log('Панель DevTools создана');
        
        // Обработка событий панели
        panel.onShown.addListener(() => {
          console.log('Панель показана');
          // Здесь можно инициализировать UI панели
        });
        
        panel.onHidden.addListener(() => {
          console.log('Панель скрыта');
          // Здесь можно освободить ресурсы
        });
      }
    );
    
    // Работа с текущей вкладкой
    const currentTabId = chrome.devtools.inspectedWindow.tabId;
    console.log('Инспектируемая вкладка:', currentTabId);
    
    // Мониторинг изменений в DOM
    chrome.devtools.inspectedWindow.onResourceAdded.addListener((resource) => {
      console.log('Ресурс добавлен:', resource);
    });
    
    // Выполнение кода в контексте веб-страницы
    chrome.devtools.inspectedWindow.eval(
      'document.title',
      (result, isException) => {
        if (isException) {
          console.error('Ошибка при выполнении кода:', isException);
        } else {
          console.log('Заголовок страницы:', result);
        }
      }
    );
  } else {
    console.error('DevTools API недоступен');
  }
}

// Инициализация при загрузке
document.addEventListener('DOMContentLoaded', initializeDevTools);
```

### 3. panel.html

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Панель DevTools</title>
  <link rel="stylesheet" href="panel.css">
</head>
<body>
  <div class="devtools-panel">
    <header class="panel-header">
      <h1>Инспектор моего расширения</h1>
      <div class="panel-controls">
        <button id="refresh-btn" class="btn btn-primary">Обновить</button>
        <button id="clear-btn" class="btn btn-secondary">Очистить</button>
      </div>
    </header>
    
    <main class="panel-main">
      <div class="panel-section">
        <h3>Информация о странице</h3>
        <div id="page-info" class="info-container">
          <p>Загрузка информации...</p>
        </div>
      </div>
      
      <div class="panel-section">
        <h3>События расширения</h3>
        <div id="events-log" class="log-container">
          <!-- События будут добавляться сюда -->
        </div>
      </div>
      
      <div class="panel-section">
        <h3>Кастомные данные</h3>
        <div id="custom-data" class="data-container">
          <!-- Кастомные данные будут отображаться здесь -->
        </div>
      </div>
    </main>
  </div>
  
  <script src="panel.js"></script>
</body>
</html>
```

### 4. panel.ts

```typescript
// panel.ts
interface PageData {
  url: string;
  title: string;
  domElements: number;
  scripts: string[];
  styles: string[];
}

interface EventLog {
  timestamp: number;
  type: string;
  message: string;
  data?: any;
}

class DevToolsPanel {
  private pageData: PageData | null = null;
  private eventLog: EventLog[] = [];
  private elements: {
    pageInfo: HTMLElement;
    eventsLog: HTMLElement;
    customData: HTMLElement;
    refreshBtn: HTMLButtonElement;
    clearBtn: HTMLButtonElement;
  };

  constructor() {
    this.initializeElements();
    this.setupEventListeners();
    this.loadInitialData();
    this.setupMessageListener();
  }

  private initializeElements(): void {
    this.elements = {
      pageInfo: document.getElementById('page-info') as HTMLElement,
      eventsLog: document.getElementById('events-log') as HTMLElement,
      customData: document.getElementById('custom-data') as HTMLElement,
      refreshBtn: document.getElementById('refresh-btn') as HTMLButtonElement,
      clearBtn: document.getElementById('clear-btn') as HTMLButtonElement
    };
  }

  private setupEventListeners(): void {
    // Обработчик кнопки обновления
    this.elements.refreshBtn?.addEventListener('click', () => {
      this.loadInitialData();
    });

    // Обработчик кнопки очистки
    this.elements.clearBtn?.addEventListener('click', () => {
      this.clearEventLog();
    });
  }

  private async loadInitialData(): Promise<void> {
    try {
      // Загрузка информации о странице через DevTools API
      this.logEvent('info', 'Загрузка информации о странице...');
      
      // Выполнение кода в контексте веб-страницы
      const [url, title, elementCount] = await Promise.all([
        this.executeInPageContext('window.location.href'),
        this.executeInPageContext('document.title'),
        this.executeInPageContext('document.querySelectorAll("*").length')
      ]);
      
      this.pageData = {
        url: url as string,
        title: title as string,
        domElements: elementCount as number,
        scripts: [],
        styles: []
      };
      
      // Загрузка дополнительной информации
      this.pageData.scripts = await this.getScripts();
      this.pageData.styles = await this.getStyles();
      
      this.updatePageInfo();
      this.logEvent('success', 'Информация о странице загружена');
    } catch (error) {
      console.error('Ошибка при загрузке данных:', error);
      this.logEvent('error', `Ошибка загрузки: ${error.message}`);
    }
  }

  private async executeInPageContext(code: string): Promise<any> {
    return new Promise((resolve, reject) => {
      chrome.devtools.inspectedWindow.eval(
        code,
        (result, isException) => {
          if (isException) {
            reject(new Error(`Ошибка выполнения: ${isException.value || isException.description}`));
          } else {
            resolve(result);
          }
        }
      );
    });
  }

  private async getScripts(): Promise<string[]> {
    const scripts = await this.executeInPageContext(`
      Array.from(document.querySelectorAll('script')).map(script => script.src || script.innerHTML.substring(0, 100))
    `);
    return Array.isArray(scripts) ? scripts : [];
  }

  private async getStyles(): Promise<string[]> {
    const styles = await this.executeInPageContext(`
      Array.from(document.querySelectorAll('link[rel="stylesheet"], style')).map(el => el.href || el.innerHTML.substring(0, 100))
    `);
    return Array.isArray(styles) ? styles : [];
  }

  private updatePageInfo(): void {
    if (!this.pageData || !this.elements.pageInfo) return;
    
    this.elements.pageInfo.innerHTML = `
      <div class="page-info-item">
        <strong>URL:</strong> <span>${this.pageData.url}</span>
      </div>
      <div class="page-info-item">
        <strong>Заголовок:</strong> <span>${this.pageData.title}</span>
      </div>
      <div class="page-info-item">
        <strong>DOM элементов:</strong> <span>${this.pageData.domElements}</span>
      </div>
      <div class="page-info-item">
        <strong>Скрипты:</strong> <span>${this.pageData.scripts.length}</span>
      </div>
      <div class="page-info-item">
        <strong>Стили:</strong> <span>${this.pageData.styles.length}</span>
      </div>
    `;
  }

  private logEvent(type: string, message: string, data?: any): void {
    const event: EventLog = {
      timestamp: Date.now(),
      type,
      message,
      data
    };
    
    this.eventLog.unshift(event); // Добавляем в начало
    
    // Ограничиваем количество событий
    if (this.eventLog.length > 100) {
      this.eventLog = this.eventLog.slice(0, 100);
    }
    
    this.updateEventLog();
  }

  private updateEventLog(): void {
    if (!this.elements.eventsLog) return;
    
    this.elements.eventsLog.innerHTML = this.eventLog.map(event => {
      const time = new Date(event.timestamp).toLocaleTimeString();
      const className = `log-entry log-${event.type}`;
      
      return `
        <div class="${className}">
          <span class="log-time">[${time}]</span>
          <span class="log-message">${event.message}</span>
          ${event.data ? `<span class="log-data">${JSON.stringify(event.data)}</span>` : ''}
        </div>
      `;
    }).join('');
  }

  private clearEventLog(): void {
    this.eventLog = [];
    this.updateEventLog();
    this.logEvent('info', 'Журнал событий очищен');
  }

  private setupMessageListener(): void {
    // Слушаем сообщения от content script или background script
    chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
      console.log('Получено сообщение в DevTools панели:', request);
      
      switch (request.action) {
        case 'EXTENSION_EVENT':
          this.logEvent(request.type || 'info', request.message, request.data);
          break;
          
        case 'PAGE_DATA_UPDATE':
          this.handlePageDataUpdate(request.data);
          break;
          
        default:
          console.warn('Неизвестное действие:', request.action);
      }
      
      sendResponse({ status: 'received' });
    });
  }

  private handlePageDataUpdate(data: any): void {
    this.logEvent('info', 'Данные страницы обновлены', data);
    
    // Обновляем отображение при необходимости
    if (data.url && this.pageData) {
      this.pageData.url = data.url;
      this.updatePageInfo();
    }
  }
}

// Инициализация DevTools панели при загрузке
document.addEventListener('DOMContentLoaded', () => {
  new DevToolsPanel();
});
```

### 5. content-script.ts (для взаимодействия с веб-страницей)

```typescript
// content-script.ts
interface DevToolsMessage {
  action: string;
  type?: string;
  message: string;
  data?: any;
}

class ContentScriptForDevTools {
  private isConnected: boolean = false;

  constructor() {
    this.initialize();
  }

  private initialize(): void {
    // Проверяем, подключено ли расширение к DevTools
    this.checkDevToolsConnection();
    
    // Мониторим изменения в DOM
    this.setupDOMMonitoring();
    
    // Мониторим сетевые запросы (если разрешено)
    this.setupNetworkMonitoring();
    
    // Отправляем сообщение о загрузке
    this.sendMessageToDevTools({
      action: 'EXTENSION_LOADED',
      message: 'Content script загружен',
      data: {
        url: window.location.href,
        timestamp: Date.now()
      }
    });
  }

  private checkDevToolsConnection(): void {
    // Проверяем подключение к DevTools через background script
    chrome.runtime.sendMessage({
      action: 'CHECK_DEVTOOLS_CONNECTION',
      tabId: chrome.devtools?.inspectedWindow?.tabId
    }, (response) => {
      this.isConnected = response?.connected || false;
      console.log('Соединение с DevTools:', this.isConnected);
    });
  }

  private setupDOMMonitoring(): void {
    const observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        // Отправляем информацию о значительных изменениях DOM в DevTools
        if (mutation.type === 'childList' && mutation.addedNodes.length > 0) {
          const addedElements = Array.from(mutation.addedNodes)
            .filter(node => node.nodeType === Node.ELEMENT_NODE)
            .map(node => ({
              tag: (node as Element).tagName,
              id: (node as Element).id,
              class: (node as Element).className
            }));
          
          if (addedElements.length > 0) {
            this.sendMessageToDevTools({
              action: 'DOM_CHANGED',
              type: 'info',
              message: 'Обнаружены изменения в DOM',
              data: {
                addedElements,
                timestamp: Date.now()
              }
            });
          }
        }
      });
    });
    
    observer.observe(document.body, {
      childList: true,
      subtree: true
    });
  }

  private setupNetworkMonitoring(): void {
    // Перехватываем fetch запросы
    const originalFetch = window.fetch;
    window.fetch = async (...args) => {
      const startTime = Date.now();
      
      try {
        const response = await originalFetch.apply(this, args);
        
        // Отправляем информацию о запросе в DevTools
        this.sendMessageToDevTools({
          action: 'NETWORK_REQUEST',
          type: 'info',
          message: 'Fetch запрос выполнен',
          data: {
            url: args[0] instanceof Request ? args[0].url : args[0],
            method: args[0] instanceof Request ? args[0].method : (args[1]?.method || 'GET'),
            status: response.status,
            duration: Date.now() - startTime,
            timestamp: Date.now()
          }
        });
        
        return response;
      } catch (error) {
        this.sendMessageToDevTools({
          action: 'NETWORK_ERROR',
          type: 'error',
          message: 'Ошибка fetch запроса',
          data: {
            url: args[0] instanceof Request ? args[0].url : args[0],
            error: error.message,
            timestamp: Date.now()
          }
        });
        
        throw error;
      }
    };
  }

  private sendMessageToDevTools(message: DevToolsMessage): void {
    // Отправляем сообщение в background script, который перенаправит в DevTools
    chrome.runtime.sendMessage({
      action: 'FORWARD_TO_DEVTOOLS',
      message,
      tabId: chrome.devtools?.inspectedWindow?.tabId
    }).catch(error => {
      // Игнорируем ошибки, если DevTools не открыты
      console.debug('Не удалось отправить сообщение в DevTools:', error);
    });
  }
}

// Инициализация content script
new ContentScriptForDevTools();
```

### 6. background.ts (для управления соединением)

```typescript
// background.ts
interface DevToolsConnection {
  tabId: number;
  connected: boolean;
  timestamp: number;
}

class DevToolsBackgroundManager {
  private connections: Map<number, DevToolsConnection> = new Map();

  constructor() {
    this.setupMessageListener();
    this.setupTabListeners();
  }

  private setupMessageListener(): void {
    chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
      switch (request.action) {
        case 'CHECK_DEVTOOLS_CONNECTION':
          const connected = this.connections.has(request.tabId);
          sendResponse({ connected });
          break;
          
        case 'FORWARD_TO_DEVTOOLS':
          this.forwardMessageToDevTools(request.message, request.tabId);
          sendResponse({ status: 'forwarded' });
          break;
          
        case 'DEVTOOLS_OPENED':
          this.handleDevToolsOpened(request.tabId);
          sendResponse({ status: 'registered' });
          break;
          
        case 'DEVTOOLS_CLOSED':
          this.handleDevToolsClosed(request.tabId);
          sendResponse({ status: 'unregistered' });
          break;
          
        default:
          console.warn('Неизвестное действие в background:', request.action);
      }
      
      return true; // Для асинхронного ответа
    });
  }

  private setupTabListeners(): void {
    // Отслеживаем закрытие вкладок
    chrome.tabs.onRemoved.addListener((tabId) => {
      this.connections.delete(tabId);
    });
    
    // Отслеживаем обновление вкладок
    chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
      if (changeInfo.status === 'complete' && tab.url) {
        // Можно выполнить действия при полной загрузке вкладки
        console.log(`Вкладка ${tabId} полностью загружена`);
      }
    });
  }

  private handleDevToolsOpened(tabId: number): void {
    this.connections.set(tabId, {
      tabId,
      connected: true,
      timestamp: Date.now()
    });
    
    console.log(`DevTools открыты для вкладки ${tabId}`);
  }

  private handleDevToolsClosed(tabId: number): void {
    this.connections.delete(tabId);
    console.log(`DevTools закрыты для вкладки ${tabId}`);
  }

  private forwardMessageToDevTools(message: any, tabId: number): void {
    if (!this.connections.has(tabId)) {
      console.debug(`Нет подключения к DevTools для вкладки ${tabId}`);
      return;
    }
    
    // Отправляем сообщение в DevTools панель
    chrome.runtime.sendMessage({
      action: 'DEVTOOLS_MESSAGE',
      message,
      tabId
    }).catch(error => {
      console.debug('Ошибка отправки в DevTools:', error);
      // Удаляем соединение, если не удается отправить сообщение
      this.connections.delete(tabId);
    });
  }
}

// Инициализация менеджера DevTools
new DevToolsBackgroundManager();
```

## Примеры использования DevTools-расширений

### 1. Инспектор React-компонентов

```typescript
// Проверка наличия React на странице
async function checkReactPresence(): Promise<boolean> {
  try {
    const result = await executeInPageContext(`
      (function() {
        // Проверяем наличие React
        const hasReact = typeof window.React !== 'undefined' || 
                        document.querySelector('[data-reactroot], [data-reactid]');
        
        // Проверяем наличие ReactDOM
        const hasReactDOM = typeof window.ReactDOM !== 'undefined';
        
        return { hasReact, hasReactDOM };
      })();
    `);
    
    return result.hasReact;
  } catch (error) {
    console.error('Ошибка проверки React:', error);
    return false;
  }
}
```

### 2. Инспектор CSS-стилей

```typescript
// Получение всех CSS-правил на странице
async function getCSSRules(): Promise<any[]> {
  const rules = await executeInPageContext(`
    (function() {
      const allRules = [];
      
      // Проходим по всем стилевым таблицам
      for (let sheet of document.styleSheets) {
        try {
          const rules = sheet.cssRules || sheet.rules;
          for (let rule of rules) {
            allRules.push({
              selector: rule.selectorText,
              cssText: rule.cssText,
              stylesheet: sheet.href || 'inline'
            });
          }
        } catch (e) {
          // Игнорируем ошибки доступа к чужим доменам
          console.debug('Не удалось получить правила из:', sheet.href);
        }
      }
      
      return allRules;
    })();
  `);
  
  return Array.isArray(rules) ? rules : [];
}
```

## Ограничения и особенности

### 1. Ограничения безопасности

DevTools-расширения имеют те же ограничения, что и обычные расширения:

- Ограничения CORS при доступе к другим доменам
- Ограничения Content Security Policy
- Ограничения на внедрение скриптов

### 2. Жизненный цикл

- DevTools-расширение активируется только когда DevTools открыты
- Панель существует только во время открытия DevTools
- Соединение с веб-страницей может быть потеряно при навигации

### 3. Производительность

- Избегайте тяжелых вычислений в DevTools-панели
- Используйте эффективные методы обновления UI
- Ограничивайте частоту обновлений данных

## Лучшие практики

### 1. Эффективное обновление UI

```typescript
// Используем requestAnimationFrame для плавных обновлений
function updateUI(data: any): void {
  requestAnimationFrame(() => {
    // Обновляем DOM
    renderData(data);
  });
}
```

### 2. Управление памятью

```typescript
// Очищаем ресурсы при закрытии панели
window.addEventListener('beforeunload', () => {
  // Очищаем таймеры
  clearInterval(this.updateInterval);
  
  // Отписываемся от событий
  this.eventSource.close();
  
  // Очищаем данные
  this.cache.clear();
});
```

### 3. Обработка ошибок

```typescript
// Оборачиваем потенциально опасные операции
async function safeExecute<T>(operation: () => Promise<T>): Promise<T | null> {
  try {
    return await operation();
  } catch (error) {
    console.error('Ошибка в DevTools операции:', error);
    return null;
  }
}
```

## Связанные темы

- [[Manifest-v3]]
- [[Background-scripts]]
- [[Content-scripts]]
- [[Popup-интерфейс]]

## Теги

#browser-extension #devtools #chrome-extension #web-extension #typescript #debugging #development-tools