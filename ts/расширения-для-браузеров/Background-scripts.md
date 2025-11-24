---
aliases: [Background Script, Фоновые скрипты]
tags: [browser-extension, background-script, chrome-extension, web-extension, typescript, service-worker]
---

# Background Scripts: Управление логикой расширения

## Обзор

Background Scripts (в Manifest V3 это Service Worker) - это компонент расширения, который работает в фоновом режиме и управляет основной логикой расширения. Они обеспечивают постоянное состояние и позволяют обрабатывать события, такие как клики на иконке расширения, сообщения от других частей расширения и системные события.

## Архитектура Background Scripts в Manifest V3

В Manifest V3 фоновые скрипты реализованы как Service Worker, что отличается от традиционных фоновых страниц в Manifest V2:

- **Service Worker**: Живет только во время обработки событий
- **Более низкое потребление ресурсов**: Автоматически останавливается после выполнения задачи
- **Ограниченные API**: Некоторые API недоступны (DOM, localStorage и др.)

## Объявление в манифесте

```json
{
  "manifest_version": 3,
  "background": {
    "service_worker": "background.js"
  }
}
```

## Пример TypeScript Background Script

```typescript
// background.ts
interface ExtensionState {
  isActive: boolean;
  lastVisitedUrls: string[];
  settings: {
    autoHighlight: boolean;
    theme: 'light' | 'dark';
  };
}

// Инициализация состояния
let state: ExtensionState = {
  isActive: true,
  lastVisitedUrls: [],
  settings: {
    autoHighlight: true,
    theme: 'light'
  }
};

// Обработка установки расширения
chrome.runtime.onInstalled.addListener(() => {
  console.log('Расширение установлено');
  
  // Инициализация начальных настроек
  chrome.storage.sync.set({
    isActive: true,
    settings: {
      autoHighlight: true,
      theme: 'light'
    }
  });
  
  // Установка контекстного меню
  chrome.contextMenus.create({
    id: 'highlight-text',
    title: 'Выделить текст',
    contexts: ['selection']
  });
});

// Обработка активации расширения
chrome.runtime.onStartup.addListener(() => {
  console.log('Расширение запущено');
  loadStateFromStorage();
});

// Загрузка состояния из storage
async function loadStateFromStorage(): Promise<void> {
  try {
    const result = await chrome.storage.sync.get([
      'isActive', 
      'lastVisitedUrls', 
      'settings'
    ]);
    
    state.isActive = result.isActive ?? true;
    state.lastVisitedUrls = result.lastVisitedUrls ?? [];
    state.settings = result.settings ?? {
      autoHighlight: true,
      theme: 'light'
    };
  } catch (error) {
    console.error('Ошибка при загрузке состояния:', error);
  }
}

// Обработка сообщений от content scripts
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  console.log('Получено сообщение в background script:', request);
  
  switch(request.action) {
    case 'CONTENT_SCRIPT_LOADED':
      handleContentScriptLoaded(request.data, sender);
      sendResponse({ status: 'received' });
      break;
      
    case 'TOGGLE_EXTENSION':
      toggleExtension();
      sendResponse({ status: 'toggled', isActive: state.isActive });
      break;
      
    case 'GET_STATE':
      sendResponse({ state });
      break;
      
    case 'SAVE_SETTINGS':
      saveSettings(request.data);
      sendResponse({ status: 'saved' });
      break;
      
    default:
      console.warn('Неизвестное действие:', request.action);
      sendResponse({ status: 'unknown action' });
  }
  
  // Возвращаем true для асинхронного ответа
  return true;
});

// Обработка клика на иконке расширения
chrome.action.onClicked.addListener(async (tab) => {
  if (!tab.id) return;
  
  // Отправка сообщения в активную вкладку
  try {
    const response = await chrome.tabs.sendMessage(tab.id, {
      action: 'TOGGLE_HIGHLIGHT'
    });
    console.log('Ответ от content script:', response);
  } catch (error) {
    console.error('Ошибка при отправке сообщения в вкладку:', error);
    
    // Если вкладка не поддерживает сообщения, открываем popup
    chrome.action.openPopup();
  }
});

// Обработка кликов в контекстном меню
chrome.contextMenus.onClicked.addListener(async (info, tab) => {
  if (info.menuItemId === 'highlight-text' && tab?.id && info.selectionText) {
    try {
      await chrome.tabs.sendMessage(tab.id, {
        action: 'HIGHLIGHT_SELECTED_TEXT',
        text: info.selectionText
      });
    } catch (error) {
      console.error('Ошибка при выделении текста:', error);
    }
  }
});

// Обработка изменений вкладок
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete' && tab.url) {
    // Обновление списка посещенных URL
    if (!state.lastVisitedUrls.includes(tab.url)) {
      state.lastVisitedUrls.unshift(tab.url);
      state.lastVisitedUrls = state.lastVisitedUrls.slice(0, 10); // Ограничение до 10 последних
      saveStateToStorage();
    }
    
    // Автоматическое выполнение действий на новых страницах
    if (state.settings.autoHighlight) {
      chrome.tabs.sendMessage(tabId, { action: 'AUTO_HIGHLIGHT' });
    }
  }
});

// Обработка активации вкладки
chrome.tabs.onActivated.addListener(async (activeInfo) => {
  try {
    const tab = await chrome.tabs.get(activeInfo.tabId);
    if (tab.url) {
      console.log('Активирована вкладка:', tab.url);
    }
  } catch (error) {
    console.error('Ошибка при получении информации о вкладке:', error);
  }
});

// Обработка установки расширения
function handleContentScriptLoaded(data: { url: string }, sender: chrome.runtime.MessageSender): void {
  console.log('Content script загружен на странице:', data.url);
  
  // Можем выполнить дополнительные действия
  chrome.action.setBadgeText({
    text: state.isActive ? 'ON' : 'OFF',
    tabId: sender.tab?.id
  });
  
  chrome.action.setBadgeBackgroundColor({
    color: state.isActive ? '#4CAF50' : '#F44336',
    tabId: sender.tab?.id
  });
}

// Переключение состояния расширения
function toggleExtension(): void {
  state.isActive = !state.isActive;
  
  // Обновление состояния во всех вкладках
  chrome.tabs.query({}, (tabs) => {
    tabs.forEach(tab => {
      if (tab.id) {
        chrome.tabs.sendMessage(tab.id, {
          action: 'EXTENSION_TOGGLED',
          isActive: state.isActive
        }).catch(() => {
          // Игнорируем ошибки для вкладок, которые не могут принимать сообщения
        });
      }
    });
  });
  
  saveStateToStorage();
}

// Сохранение настроек
async function saveSettings(newSettings: Partial<ExtensionState['settings']>): Promise<void> {
  state.settings = { ...state.settings, ...newSettings };
  await saveStateToStorage();
}

// Сохранение состояния в storage
async function saveStateToStorage(): Promise<void> {
  try {
    await chrome.storage.sync.set({
      isActive: state.isActive,
      lastVisitedUrls: state.lastVisitedUrls,
      settings: state.settings
    });
  } catch (error) {
    console.error('Ошибка при сохранении состояния:', error);
  }
}

// Регулярная очистка устаревших данных
setInterval(() => {
  // Очистка старых URL, оставляем только последние 50
  if (state.lastVisitedUrls.length > 50) {
    state.lastVisitedUrls = state.lastVisitedUrls.slice(0, 50);
    saveStateToStorage();
  }
}, 30 * 60 * 1000); // Каждые 30 минут
```

## Важные особенности Service Worker

### Жизненный цикл

Service Worker автоматически запускается при возникновении события и останавливается после его обработки:

```typescript
// Service Worker автоматически "просыпается" при событии
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  // Код выполняется
  console.log('Service Worker активен');
  
  // После завершения обработки события Service Worker может "уснуть"
  sendResponse({ status: 'processed' });
});
```

### Ограниченные API

Некоторые API недоступны в Service Worker:

```typescript
// НЕДОСТУПНО:
// - window, document (DOM API)
// - localStorage
// - alert, confirm
// - fetch (в некоторых случаях)

// ДОСТУПНО:
// - chrome.* API
// - fetch (в большинстве случаев)
// - indexedDB
// - setTimeout, setInterval
```

### Постоянное соединение

Для поддержания соединения с другими частями расширения:

```typescript
// Установка соединения с popup
chrome.runtime.onConnect.addListener((port) => {
  console.log('Подключение к popup установлено');
  
  port.onMessage.addListener((message) => {
    console.log('Получено сообщение от popup:', message);
    
    // Обработка сообщения
    if (message.action === 'GET_FULL_STATE') {
      port.postMessage({ state });
    }
  });
  
  port.onDisconnect.addListener(() => {
    console.log('Подключение к popup закрыто');
  });
});
```

## Обработка ошибок и отладка

### Логирование

```typescript
// В Service Worker console.log работает, но логи видны в специальном разделе
console.log('Сообщение из background script');

// Для отладки Service Worker:
// 1. Открыть chrome://extensions/
// 2. Включить "Режим разработчика"
// 3. Найти ваше расширение и нажать "Service Worker"
```

### Обработка асинхронных операций

```typescript
// Правильная обработка асинхронных операций
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'ASYNC_OPERATION') {
    // Выполняем асинхронную операцию
    performAsyncOperation(request.data)
      .then(result => {
        sendResponse({ success: true, result });
      })
      .catch(error => {
        sendResponse({ success: false, error: error.message });
      });
    
    // Возвращаем true для асинхронного ответа
    return true;
  }
});

async function performAsyncOperation(data: any): Promise<any> {
  // Асинхронная операция
  return await new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() > 0.5) {
        resolve({ data: 'Успешно выполнено', timestamp: Date.now() });
      } else {
        reject(new Error('Случайная ошибка'));
      }
    }, 1000);
  });
}
```

## Лучшие практики

1. **Минимизация времени работы**: Выполняйте только необходимые операции
2. **Использование Storage API**: Для сохранения состояния между сессиями
3. **Обработка ошибок**: Всегда обрабатывайте потенциальные ошибки
4. **Эффективное использование ресурсов**: Не выполняйте тяжелые операции без необходимости

## Связанные темы

- [[Manifest-v3]]
- [[Content-scripts]]
- [[Popup-интерфейс]]

## Теги

#browser-extension #background-script #chrome-extension #web-extension #typescript #service-worker