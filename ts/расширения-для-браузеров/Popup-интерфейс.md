---
aliases: [Popup Interface, Всплывающий интерфейс]
tags: [browser-extension, popup, chrome-extension, web-extension, typescript, user-interface]
---

# Popup-интерфейс: Всплывающее окно расширения

## Обзор

Popup-интерфейс - это всплывающее окно, которое открывается при клике на иконке расширения в адресной строке браузера. Это основной способ взаимодействия пользователя с расширением, позволяющий управлять функциями расширения, просматривать информацию и настраивать параметры.

## Структура Popup-интерфейса

Popup-интерфейс состоит из трех компонентов:

1. HTML-файл с разметкой
2. CSS-файл со стилями
3. JavaScript/TypeScript файл с логикой

### Объявление в манифесте

```json
{
  "manifest_version": 3,
  "action": {
    "default_popup": "popup.html",
    "default_title": "Название расширения"
  }
}
```

## Пример структуры Popup

### HTML-файл (popup.html)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Popup расширения</title>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="popup-container">
    <header class="popup-header">
      <h1>Моё расширение</h1>
      <button id="close-btn" class="close-btn">✕</button>
    </header>
    
    <main class="popup-main">
      <section class="controls-section">
        <div class="toggle-container">
          <label for="extension-toggle" class="toggle-label">
            <input type="checkbox" id="extension-toggle" class="toggle-input">
            <span class="toggle-switch"></span>
            <span class="toggle-text">Активировать расширение</span>
          </label>
        </div>
        
        <div class="settings-container">
          <h3>Настройки</h3>
          <div class="setting-item">
            <label for="theme-select">Тема оформления:</label>
            <select id="theme-select" class="theme-select">
              <option value="light">Светлая</option>
              <option value="dark">Темная</option>
            </select>
          </div>
          
          <div class="setting-item">
            <label for="highlight-color">Цвет выделения:</label>
            <input type="color" id="highlight-color" class="color-input" value="#FFFF00">
          </div>
        </div>
      </section>
      
      <section class="stats-section">
        <h3>Статистика</h3>
        <div class="stats-grid">
          <div class="stat-item">
            <span class="stat-label">Посещено страниц:</span>
            <span id="visited-pages-count" class="stat-value">0</span>
          </div>
          <div class="stat-item">
            <span class="stat-label">Выделено элементов:</span>
            <span id="highlighted-elements-count" class="stat-value">0</span>
          </div>
        </div>
      </section>
    </main>
    
    <footer class="popup-footer">
      <button id="save-settings-btn" class="btn btn-primary">Сохранить настройки</button>
      <button id="reset-settings-btn" class="btn btn-secondary">Сбросить</button>
    </footer>
  </div>
  
  <script src="popup.js"></script>
</body>
</html>
```

### CSS-файл (popup.css)

```css
:root {
  --primary-color: #4CAF50;
  --secondary-color: #2196F3;
  --background-color: #ffffff;
  --text-color: #333333;
  --border-color: #e0e0e0;
  --success-color: #4CAF50;
  --warning-color: #FF9800;
  --error-color: #F44336;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  width: 350px;
  background-color: var(--background-color);
  color: var(--text-color);
}

.popup-container {
  display: flex;
  flex-direction: column;
  min-height: 400px;
  padding: 16px;
}

.popup-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 16px;
  border-bottom: 1px solid var(--border-color);
  margin-bottom: 16px;
}

.popup-header h1 {
  font-size: 1.2rem;
  font-weight: 600;
}

.close-btn {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: var(--text-color);
  padding: 4px;
  border-radius: 4px;
}

.close-btn:hover {
  background-color: #f0f0f0;
}

.controls-section,
.stats-section {
  margin-bottom: 16px;
}

.controls-section h3,
.stats-section h3 {
  font-size: 1rem;
  margin-bottom: 12px;
  color: var(--text-color);
}

.toggle-container {
  margin-bottom: 16px;
}

.toggle-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
}

.toggle-input {
  display: none;
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: 44px;
  height: 22px;
  margin-right: 8px;
}

.toggle-switch::before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 2px;
  bottom: 2px;
  background-color: white;
  transition: .4s;
  border-radius: 50%;
}

.toggle-input:checked + .toggle-switch {
  background-color: var(--primary-color);
}

.toggle-input:checked + .toggle-switch::before {
  transform: translateX(22px);
}

.toggle-switch,
.toggle-text {
  user-select: none;
}

.settings-container {
  background-color: #f9f9f9;
  padding: 12px;
  border-radius: 8px;
}

.setting-item {
  margin-bottom: 12px;
}

.setting-item label {
  display: block;
  margin-bottom: 4px;
  font-weight: 500;
  font-size: 0.9rem;
}

.theme-select,
.color-input {
  width: 100%;
  padding: 8px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background-color: white;
}

.color-input {
  height: 36px;
  padding: 4px;
}

.stats-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #eee;
}

.stat-label {
  color: #666;
  font-size: 0.9rem;
}

.stat-value {
  font-weight: 600;
  color: var(--primary-color);
}

.popup-footer {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-top: auto;
  padding-top: 16px;
  border-top: 1px solid var(--border-color);
}

.btn {
  padding: 10px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  transition: background-color 0.2s;
}

.btn-primary {
  background-color: var(--primary-color);
  color: white;
}

.btn-primary:hover {
  background-color: #45a049;
}

.btn-secondary {
  background-color: #f0f0f0;
  color: var(--text-color);
}

.btn-secondary:hover {
  background-color: #e0e0e0;
}

.loading {
  opacity: 0.6;
  pointer-events: none;
}

.error-message {
  color: var(--error-color);
  font-size: 0.8rem;
  margin-top: 4px;
}

.success-message {
  color: var(--success-color);
  font-size: 0.8rem;
  margin-top: 4px;
}
```

## TypeScript-файл (popup.ts)

```typescript
// popup.ts
interface ExtensionState {
  isActive: boolean;
  settings: {
    theme: 'light' | 'dark';
    highlightColor: string;
  };
  stats: {
    visitedPages: number;
    highlightedElements: number;
  };
}

class PopupController {
  private state: ExtensionState = {
    isActive: true,
    settings: {
      theme: 'light',
      highlightColor: '#FFFF00'
    },
    stats: {
      visitedPages: 0,
      highlightedElements: 0
    }
  };

  constructor() {
    this.initializeElements();
    this.loadState();
    this.setupEventListeners();
  }

  private initializeElements(): void {
    // Инициализация элементов DOM
    const extensionToggle = document.getElementById('extension-toggle') as HTMLInputElement;
    const themeSelect = document.getElementById('theme-select') as HTMLSelectElement;
    const highlightColor = document.getElementById('highlight-color') as HTMLInputElement;
    const saveBtn = document.getElementById('save-settings-btn') as HTMLButtonElement;
    const resetBtn = document.getElementById('reset-settings-btn') as HTMLButtonElement;
    const closeBtn = document.getElementById('close-btn') as HTMLButtonElement;

    // Сохраняем элементы для дальнейшего использования
    this.elements = {
      extensionToggle,
      themeSelect,
      highlightColor,
      saveBtn,
      resetBtn,
      closeBtn
    };
  }

  private elements: {
    extensionToggle: HTMLInputElement;
    themeSelect: HTMLSelectElement;
    highlightColor: HTMLInputElement;
    saveBtn: HTMLButtonElement;
    resetBtn: HTMLButtonElement;
    closeBtn: HTMLButtonElement;
  };

  private async loadState(): Promise<void> {
    try {
      // Показываем индикатор загрузки
      this.setLoading(true);
      
      // Запрашиваем состояние из background script
      const response = await chrome.runtime.sendMessage({ action: 'GET_STATE' });
      
      if (response.state) {
        this.state = response.state;
        this.updateUI();
      }
    } catch (error) {
      console.error('Ошибка при загрузке состояния:', error);
      this.showError('Не удалось загрузить состояние расширения');
    } finally {
      this.setLoading(false);
    }
  }

  private updateUI(): void {
    // Обновляем UI на основе текущего состояния
    if (this.elements.extensionToggle) {
      this.elements.extensionToggle.checked = this.state.isActive;
    }
    
    if (this.elements.themeSelect) {
      this.elements.themeSelect.value = this.state.settings.theme;
    }
    
    if (this.elements.highlightColor) {
      this.elements.highlightColor.value = this.state.settings.highlightColor;
    }
    
    // Обновляем статистику
    this.updateStats();
  }

  private updateStats(): void {
    const visitedCount = document.getElementById('visited-pages-count');
    const highlightedCount = document.getElementById('highlighted-elements-count');
    
    if (visitedCount) {
      visitedCount.textContent = this.state.stats.visitedPages.toString();
    }
    
    if (highlightedCount) {
      highlightedCount.textContent = this.state.stats.highlightedElements.toString();
    }
  }

  private setupEventListeners(): void {
    // Обработчик переключения активности расширения
    if (this.elements.extensionToggle) {
      this.elements.extensionToggle.addEventListener('change', (e) => {
        const isActive = (e.target as HTMLInputElement).checked;
        this.toggleExtension(isActive);
      });
    }

    // Обработчик изменения темы
    if (this.elements.themeSelect) {
      this.elements.themeSelect.addEventListener('change', (e) => {
        const theme = (e.target as HTMLSelectElement).value as 'light' | 'dark';
        this.state.settings.theme = theme;
      });
    }

    // Обработчик изменения цвета выделения
    if (this.elements.highlightColor) {
      this.elements.highlightColor.addEventListener('change', (e) => {
        const color = (e.target as HTMLInputElement).value;
        this.state.settings.highlightColor = color;
      });
    }

    // Обработчик сохранения настроек
    if (this.elements.saveBtn) {
      this.elements.saveBtn.addEventListener('click', () => {
        this.saveSettings();
      });
    }

    // Обработчик сброса настроек
    if (this.elements.resetBtn) {
      this.elements.resetBtn.addEventListener('click', () => {
        this.resetSettings();
      });
    }

    // Обработчик закрытия popup
    if (this.elements.closeBtn) {
      this.elements.closeBtn.addEventListener('click', () => {
        window.close();
      });
    }
  }

  private async toggleExtension(isActive: boolean): Promise<void> {
    try {
      const response = await chrome.runtime.sendMessage({
        action: 'TOGGLE_EXTENSION',
        isActive
      });
      
      if (response.status === 'toggled') {
        this.state.isActive = response.isActive;
        this.showSuccess('Состояние расширения изменено');
      }
    } catch (error) {
      console.error('Ошибка при переключении расширения:', error);
      this.showError('Не удалось изменить состояние расширения');
      // Возвращаем переключатель в исходное состояние
      if (this.elements.extensionToggle) {
        this.elements.extensionToggle.checked = !isActive;
      }
    }
  }

  private async saveSettings(): Promise<void> {
    try {
      this.setLoading(true);
      
      const response = await chrome.runtime.sendMessage({
        action: 'SAVE_SETTINGS',
        data: this.state.settings
      });
      
      if (response.status === 'saved') {
        this.showSuccess('Настройки сохранены');
      }
    } catch (error) {
      console.error('Ошибка при сохранении настроек:', error);
      this.showError('Не удалось сохранить настройки');
    } finally {
      this.setLoading(false);
    }
  }

  private resetSettings(): void {
    // Сброс настроек к значениям по умолчанию
    this.state.settings = {
      theme: 'light',
      highlightColor: '#FFFF00'
    };
    
    this.updateUI();
    this.showSuccess('Настройки сброшены');
  }

  private setLoading(isLoading: boolean): void {
    const popupContainer = document.querySelector('.popup-container');
    if (popupContainer) {
      popupContainer.classList.toggle('loading', isLoading);
    }
  }

  private showError(message: string): void {
    this.showMessage(message, 'error');
  }

  private showSuccess(message: string): void {
    this.showMessage(message, 'success');
  }

  private showMessage(message: string, type: 'error' | 'success'): void {
    // Удаляем предыдущие сообщения
    const existingMessage = document.querySelector('.message');
    if (existingMessage) {
      existingMessage.remove();
    }
    
    // Создаем новый элемент сообщения
    const messageElement = document.createElement('div');
    messageElement.className = `message ${type}-message`;
    messageElement.textContent = message;
    
    // Добавляем в footer
    const footer = document.querySelector('.popup-footer');
    if (footer) {
      footer.insertBefore(messageElement, footer.firstChild);
      
      // Автоматически удаляем сообщение через 3 секунды
      setTimeout(() => {
        messageElement.remove();
      }, 3000);
    }
  }
}

// Инициализация контроллера при загрузке popup
document.addEventListener('DOMContentLoaded', () => {
  new PopupController();
});
```

## Особенности Popup-интерфейса

### Ограничения размера

Popup имеет ограничения по размеру, которые зависят от браузера:

```css
/* Рекомендуемый максимальный размер для popup */
body {
  width: 350px; /* Ширина не должна превышать 800px */
  min-height: 200px;
  max-height: 600px; /* Высота не должна быть слишком большой */
}
```

### Взаимодействие с background script

Popup взаимодействует с background script через сообщения:

```typescript
// Отправка сообщения в background
chrome.runtime.sendMessage({
  action: 'GET_STATE'
}, (response) => {
  console.log('Получено состояние:', response);
});

// Получение сообщений от background
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'STATE_UPDATED') {
    // Обновляем UI при изменении состояния
    updateUI(request.data);
  }
});
```

### Работа с Storage API

Popup может напрямую работать с chrome.storage:

```typescript
// Сохранение данных
async function saveUserData(data: any): Promise<void> {
  try {
    await chrome.storage.local.set({ userData: data });
    console.log('Данные сохранены');
  } catch (error) {
    console.error('Ошибка при сохранении данных:', error);
  }
}

// Загрузка данных
async function loadUserData(): Promise<any> {
  try {
    const result = await chrome.storage.local.get(['userData']);
    return result.userData || {};
  } catch (error) {
    console.error('Ошибка при загрузке данных:', error);
    return {};
  }
}
```

## Лучшие практики

### 1. Эффективная загрузка данных

```typescript
// Загружайте только необходимые данные
async function loadMinimalData(): Promise<Partial<ExtensionState>> {
  const result = await chrome.storage.sync.get([
    'isActive',
    'settings'
  ]);
  
  return {
    isActive: result.isActive ?? true,
    settings: result.settings ?? { theme: 'light', highlightColor: '#FFFF00' }
  };
}
```

### 2. Обработка ошибок

```typescript
// Всегда обрабатывайте ошибки при взаимодействии с API
async function safeSendMessage(message: any): Promise<any> {
  try {
    return await chrome.runtime.sendMessage(message);
  } catch (error) {
    console.error('Ошибка при отправке сообщения:', error);
    throw new Error('Не удалось отправить сообщение в background script');
  }
}
```

### 3. Оптимизация производительности

```typescript
// Используйте debounce для частых событий
function debounce<T extends (...args: any[]) => any>(func: T, wait: number): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout;
  return function executedFunction(...args: Parameters<T>): void {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// Пример использования
const debouncedSave = debounce(saveSettings, 300);
```

## Адаптивный дизайн

Popup должен корректно отображаться на разных устройствах:

```css
/* Адаптивные стили для popup */
@media (max-width: 400px) {
  .popup-container {
    width: 100%;
    padding: 12px;
  }
  
  .stats-grid {
    grid-template-columns: 1fr;
  }
}

/* Темная тема */
@media (prefers-color-scheme: dark) {
  :root {
    --background-color: #202124;
    --text-color: #e8eaed;
    --border-color: #5f6368;
  }
}
```

## Связанные темы

- [[Manifest-v3]]
- [[Background-scripts]]
- [[Content-scripts]]

## Теги

#browser-extension #popup #chrome-extension #web-extension #typescript #user-interface #ui