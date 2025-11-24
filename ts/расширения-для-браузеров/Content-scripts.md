---
aliases: [Content Script, Контент-скрипты]
tags: [browser-extension, content-script, chrome-extension, web-extension, typescript]
---

# Content Scripts: Внедрение функциональности в веб-страницы

## Обзор

Content Scripts - это JavaScript файлы, которые выполняются в контексте веб-страницы и могут читать/изменять DOM элементы. Они являются ключевым компонентом расширений браузера, позволяющим взаимодействовать с содержимым веб-сайтов.

## Основные характеристики Content Scripts

- Выполняются в изолированной среде (isolated world)
- Имеют доступ к DOM веб-страницы
- Не имеют доступа к переменным и функциям, определенным веб-сайтом
- Могут обмениваться сообщениями с фоновыми скриптами

## Структура объявления Content Scripts в манифесте

```json
{
  "manifest_version": 3,
  "content_scripts": [
    {
      "matches": ["https://*/*", "http://*/*"],
      "js": ["content-script.js"],
      "css": ["content-style.css"],
      "run_at": "document_idle",
      "all_frames": false
    }
  ]
}
```

### Параметры Content Scripts

- `matches`: URL-шаблоны, определяющие, на каких страницах запускать скрипт
- `js`: массив JavaScript файлов для внедрения
- `css`: массив CSS файлов для внедрения
- `run_at`: момент выполнения (document_start, document_end, document_idle)
- `all_frames`: запускать во всех фреймах или только в основном окне

## Пример TypeScript Content Script

```typescript
// content-script.ts
interface Message {
  action: string;
  data?: any;
}

// Проверка наличия элементов на странице
function checkPageElements(): void {
  const headings = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
  console.log(`Найдено заголовков: ${headings.length}`);
  
  // Добавление кастомного класса к заголовкам
  headings.forEach((heading, index) => {
    heading.classList.add('custom-extension-highlight');
  });
}

// Отправка сообщения в фоновый скрипт
function sendMessageToBackground(message: Message): void {
  chrome.runtime.sendMessage(message, (response) => {
    console.log('Получен ответ от фонового скрипта:', response);
  });
}

// Получение сообщений из фонового скрипта
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  console.log('Получено сообщение в content script:', request);
  
  switch(request.action) {
    case 'HIGHLIGHT_HEADINGS':
      checkPageElements();
      sendResponse({ status: 'success', message: 'Заголовки выделены' });
      break;
    case 'GET_PAGE_INFO':
      const pageInfo = {
        url: window.location.href,
        title: document.title,
        headingsCount: document.querySelectorAll('h1, h2, h3, h4, h5, h6').length
      };
      sendResponse(pageInfo);
      break;
    default:
      sendResponse({ status: 'error', message: 'Неизвестное действие' });
  }
  
  // Возвращаем true для асинхронного ответа
  return true;
});

// Выполнение основной логики при загрузке страницы
document.addEventListener('DOMContentLoaded', () => {
  console.log('Content script загружен и готов к работе');
  
  // Отправка сообщения о загрузке
  sendMessageToBackground({
    action: 'CONTENT_SCRIPT_LOADED',
    data: { url: window.location.href }
  });
});
```

## Работа с DOM в Content Scripts

Content Scripts имеют полный доступ к DOM веб-страницы:

```typescript
// Пример работы с DOM
function enhancePage(): void {
  // Добавление кастомного элемента
  const customElement = document.createElement('div');
  customElement.id = 'extension-enhancement';
  customElement.innerHTML = '<p>Это добавлено расширением!</p>';
  customElement.style.cssText = `
    position: fixed;
    top: 10px;
    right: 10px;
    background: #007acc;
    color: white;
    padding: 10px;
    border-radius: 5px;
    z-index: 10000;
  `;
  
  document.body.appendChild(customElement);
  
  // Мониторинг изменений DOM
  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      if (mutation.type === 'childList') {
        console.log('DOM изменился:', mutation.addedNodes.length, 'элементов добавлено');
      }
    });
  });
  
  observer.observe(document.body, {
    childList: true,
    subtree: true
  });
}
```

## Изолированная среда выполнения

Content Scripts работают в изолированной среде, что означает:

```typescript
// Веб-сайт определяет переменную
// window.myVariable = 'from website';

// Content Script не имеет доступа к переменным веб-сайта
// console.log(window.myVariable); // undefined

// Но Content Script может определять свои переменные
// window.extensionVariable = 'from extension';

// И веб-сайт не имеет доступа к переменным расширения
```

## Взаимодействие с фоновыми скриптами

Content Scripts взаимодействуют с фоновыми скриптами через API сообщений:

```typescript
// Отправка сообщения
chrome.runtime.sendMessage({
  action: 'USER_ACTION',
  payload: { buttonId: 'highlight-btn', state: 'clicked' }
});

// Получение сообщения
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'UPDATE_CONTENT') {
    // Обновление содержимого страницы
    updatePageContent(request.data);
    sendResponse({ success: true });
  }
});
```

## Практические примеры использования

### 1. Выделение ключевых слов

```typescript
function highlightKeywords(keywords: string[]): void {
  const walker = document.createTreeWalker(
    document.body,
    NodeFilter.SHOW_TEXT,
    {
      acceptNode: (node) => {
        return keywords.some(keyword => 
          node.nodeValue?.toLowerCase().includes(keyword.toLowerCase())
        ) ? NodeFilter.FILTER_ACCEPT : NodeFilter.FILTER_REJECT;
      }
    }
  );
  
  const textNodes: Text[] = [];
  let node;
  
  while (node = walker.nextNode()) {
    textNodes.push(node as Text);
  }
  
  textNodes.forEach(textNode => {
    let html = textNode.nodeValue || '';
    
    keywords.forEach(keyword => {
      const regex = new RegExp(`(${keyword})`, 'gi');
      html = html.replace(regex, '<mark class="extension-highlight">$1</mark>');
    });
    
    if (html !== textNode.nodeValue) {
      const span = document.createElement('span');
      span.innerHTML = html;
      textNode.parentNode?.replaceChild(span, textNode);
    }
  });
}
```

### 2. Сбор данных со страницы

```typescript
interface PageData {
  title: string;
  url: string;
  links: string[];
  images: string[];
  textContent: string;
}

function collectPageData(): PageData {
  return {
    title: document.title,
    url: window.location.href,
    links: Array.from(document.querySelectorAll('a')).map(a => a.href),
    images: Array.from(document.querySelectorAll('img')).map(img => img.src),
    textContent: document.body.innerText
  };
}
```

## Ограничения и лучшие практики

### Ограничения

- Нет доступа к переменным и функциям веб-сайта
- Ограниченный доступ к API браузера
- Ограничения на внедрение скриптов из-за CSP

### Лучшие практики

1. Минимизировать объем внедряемого кода
2. Использовать `run_at: "document_idle"` для улучшения производительности
3. Правильно обрабатывать сообщения между контекстами
4. Использовать TypeScript для лучшей типизации и безопасности

## Связанные темы

- [[Manifest-v3]]
- [[Background-scripts]]
- [[Popup-интерфейс]]

## Теги

#browser-extension #content-script #chrome-extension #web-extension #typescript #dom