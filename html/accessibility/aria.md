# HTML Доступность: ARIA и продвинутые техники

ARIA (Accessible Rich Internet Applications) - это набор атрибутов, которые позволяют разработчикам улучшить доступность веб-контента и веб-приложений. ARIA особенно полезен для динамического контента и сложных пользовательских интерфейсов.

## Что такое ARIA?

ARIA - это спецификация W3C, которая определяет способы:
- Улучшения доступности существующих элементов
- Добавления семантической информации к элементам
- Определения ролей, состояний и свойств для элементов интерфейса

### Основные категории ARIA атрибутов

1. **Роли (Roles)** - определяют тип элемента (кнопка, меню, диалог и т.д.)
2. **Свойства (Properties)** - описывают элемент (заголовок, описание, владелец и т.д.)
3. **Состояния (States)** - описывают текущее состояние элемента (активный, выбран, скрыт и т.д.)

## ARIA роли

### Структурные роли

```html
<!-- Основные структурные роли -->
<div role="banner">Шапка сайта</div>
<div role="main">Основное содержимое</div>
<div role="contentinfo">Подвал сайта</div>
<div role="navigation">Навигация</div>
<div role="search">Блок поиска</div>
```

### Виджетные роли

```html
<!-- Кнопка-имитация -->
<div role="button" tabindex="0" onclick="doAction()" onkeydown="handleKeyDown(event)">
    Нажми меня
</div>

<!-- Переключатель -->
<div role="switch" 
     aria-checked="false" 
     tabindex="0"
     onclick="toggleSwitch()"
     onkeydown="handleSwitchKeyDown(event)">
    <span class="switch-indicator"></span>
</div>

<!-- Прогресс бар -->
<div role="progressbar" 
     aria-valuenow="50" 
     aria-valuemin="0" 
     aria-valuemax="100"
     aria-label="Прогресс загрузки">
    <div style="width: 50%; height: 20px; background: #007acc;"></div>
</div>
```

### Семантические роли

```html
<!-- Статья -->
<div role="article" aria-labelledby="article-title">
    <h2 id="article-title">Заголовок статьи</h2>
    <p>Содержимое статьи...</p>
</div>

<!-- Кнопка -->
<span role="button" tabindex="0" onclick="openModal()">Открыть модальное окно</span>

<!-- Ссылка -->
<div role="link" tabindex="0" onclick="navigateToPage()">Ссылка</div>
```

## ARIA состояния и свойства

### Состояния

```html
<!-- Активные/неактивные элементы -->
<button aria-pressed="false">Переключатель</button>
<button aria-pressed="true">Активный переключатель</button>

<!-- Выбранные/невыбранные элементы -->
<li role="tab" aria-selected="true">Активная вкладка</li>
<li role="tab" aria-selected="false">Неактивная вкладка</li>

<!-- Развернутые/свернутые элементы -->
<div role="treeitem" aria-expanded="true">Развернутый элемент</div>
<div role="treeitem" aria-expanded="false">Свернутый элемент</div>

<!-- Открытые/закрытые элементы -->
<div role="dialog" aria-modal="true" aria-hidden="false">Открытое модальное окно</div>
```

### Свойства

```html
<!-- Описание элемента -->
<button aria-describedby="help-text">Помощь</button>
<div id="help-text">Эта кнопка выполняет важное действие</div>

<!-- Заголовок элемента -->
<div role="region" aria-labelledby="region-title">
    <h2 id="region-title">Область с заголовком</h2>
    <p>Содержимое области...</p>
</div>

<!-- Владелец элемента -->
<input type="text" aria-owns="suggestions" aria-autocomplete="list">
<div id="suggestions" role="listbox">
    <div role="option">Вариант 1</div>
    <div role="option">Вариант 2</div>
</div>
```

## Продвинутые ARIA паттерны

### Раскрывающийся список

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Раскрывающийся список с ARIA</title>
    <style>
        .dropdown {
            position: relative;
            display: inline-block;
        }
        
        .dropdown-button {
            padding: 10px;
            background: #f0f0f0;
            border: 1px solid #ccc;
            cursor: pointer;
        }
        
        .dropdown-list {
            position: absolute;
            top: 100%;
            left: 0;
            background: white;
            border: 1px solid #ccc;
            display: none;
            min-width: 200px;
            z-index: 1000;
        }
        
        .dropdown-list[aria-hidden="false"] {
            display: block;
        }
        
        .dropdown-option {
            padding: 8px;
            cursor: pointer;
        }
        
        .dropdown-option:hover,
        .dropdown-option:focus {
            background: #e0e0e0;
        }
    </style>
</head>
<body>
    <h1>Пример раскрывающегося списка</h1>
    
    <div class="dropdown">
        <div class="dropdown-button" 
             role="combobox" 
             aria-haspopup="listbox" 
             aria-expanded="false" 
             aria-owns="dropdown-list"
             tabindex="0">
            Выберите опцию
        </div>
        
        <div class="dropdown-list" 
             role="listbox" 
             id="dropdown-list" 
             aria-labelledby="dropdown-button"
             aria-hidden="true">
            <div class="dropdown-option" 
                 role="option" 
                 tabindex="-1"
                 onclick="selectOption(this)"
                 onkeydown="handleOptionKeydown(event)">
                Опция 1
            </div>
            <div class="dropdown-option" 
                 role="option" 
                 tabindex="-1"
                 onclick="selectOption(this)"
                 onkeydown="handleOptionKeydown(event)">
                Опция 2
            </div>
            <div class="dropdown-option" 
                 role="option" 
                 tabindex="-1"
                 onclick="selectOption(this)"
                 onkeydown="handleOptionKeydown(event)">
                Опция 3
            </div>
        </div>
    </div>
    
    <script>
        const dropdownButton = document.querySelector('.dropdown-button');
        const dropdownList = document.querySelector('.dropdown-list');
        const options = document.querySelectorAll('.dropdown-option');
        
        // Переключение видимости списка
        dropdownButton.addEventListener('click', toggleDropdown);
        dropdownButton.addEventListener('keydown', handleButtonKeydown);
        
        function toggleDropdown() {
            const isExpanded = dropdownButton.getAttribute('aria-expanded') === 'true';
            dropdownButton.setAttribute('aria-expanded', !isExpanded);
            dropdownList.setAttribute('aria-hidden', isExpanded);
        }
        
        function handleButtonKeydown(event) {
            switch(event.key) {
                case 'Enter':
                case ' ':
                case 'ArrowDown':
                    event.preventDefault();
                    toggleDropdown();
                    if (!dropdownButton.getAttribute('aria-expanded') === 'true') {
                        options[0].focus();
                    }
                    break;
                case 'ArrowUp':
                    event.preventDefault();
                    toggleDropdown();
                    if (!dropdownButton.getAttribute('aria-expanded') === 'true') {
                        options[options.length - 1].focus();
                    }
                    break;
            }
        }
        
        function selectOption(option) {
            dropdownButton.textContent = option.textContent;
            toggleDropdown();
        }
        
        function handleOptionKeydown(event) {
            const currentOption = event.target;
            let nextOption;
            
            switch(event.key) {
                case 'ArrowDown':
                    nextOption = currentOption.nextElementSibling || options[0];
                    nextOption.focus();
                    break;
                case 'ArrowUp':
                    nextOption = currentOption.previousElementSibling || options[options.length - 1];
                    nextOption.focus();
                    break;
                case 'Enter':
                case ' ':
                    event.preventDefault();
                    selectOption(currentOption);
                    break;
                case 'Escape':
                    toggleDropdown();
                    dropdownButton.focus();
                    break;
            }
        }
    </script>
</body>
</html>
```

### Вкладки (Tabs)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Вкладки с ARIA</title>
    <style>
        .tabs {
            display: flex;
            border-bottom: 1px solid #ccc;
        }
        
        .tab {
            padding: 10px 20px;
            cursor: pointer;
            border: 1px solid transparent;
            border-bottom: none;
            background: #f0f0f0;
            margin-right: 2px;
        }
        
        .tab[aria-selected="true"] {
            background: white;
            border-color: #ccc;
        }
        
        .tab-panel {
            padding: 20px;
            border: 1px solid #ccc;
            border-top: none;
            display: none;
        }
        
        .tab-panel[aria-hidden="false"] {
            display: block;
        }
    </style>
</head>
<body>
    <h1>Пример вкладок с ARIA</h1>
    
    <div role="tablist" aria-label="Пример вкладок">
        <div role="tab" 
             aria-selected="true" 
             aria-controls="panel-1" 
             class="tab"
             tabindex="0"
             onclick="activateTab(event.target)"
             onkeydown="handleTabKeydown(event)">
            Вкладка 1
        </div>
        <div role="tab" 
             aria-selected="false" 
             aria-controls="panel-2" 
             class="tab"
             tabindex="-1"
             onclick="activateTab(event.target)"
             onkeydown="handleTabKeydown(event)">
            Вкладка 2
        </div>
        <div role="tab" 
             aria-selected="false" 
             aria-controls="panel-3" 
             class="tab"
             tabindex="-1"
             onclick="activateTab(event.target)"
             onkeydown="handleTabKeydown(event)">
            Вкладка 3
        </div>
    </div>
    
    <div id="panel-1" 
         role="tabpanel" 
         aria-labelledby="tab-1"
         class="tab-panel"
         aria-hidden="false">
        Содержимое первой вкладки
    </div>
    <div id="panel-2" 
         role="tabpanel" 
         aria-labelledby="tab-2"
         class="tab-panel"
         aria-hidden="true">
        Содержимое второй вкладки
    </div>
    <div id="panel-3" 
         role="tabpanel" 
         aria-labelledby="tab-3"
         class="tab-panel"
         aria-hidden="true">
        Содержимое третьей вкладки
    </div>
    
    <script>
        function activateTab(tabElement) {
            // Снять выделение с текущей вкладки
            document.querySelectorAll('[role="tab"]').forEach(tab => {
                tab.setAttribute('aria-selected', 'false');
                tab.setAttribute('tabindex', '-1');
            });
            
            // Выделить выбранную вкладку
            tabElement.setAttribute('aria-selected', 'true');
            tabElement.setAttribute('tabindex', '0');
            tabElement.focus();
            
            // Скрыть все панели
            document.querySelectorAll('[role="tabpanel"]').forEach(panel => {
                panel.setAttribute('aria-hidden', 'true');
            });
            
            // Показать соответствующую панель
            const panelId = tabElement.getAttribute('aria-controls');
            document.getElementById(panelId).setAttribute('aria-hidden', 'false');
        }
        
        function handleTabKeydown(event) {
            const currentTab = event.target;
            let newTab;
            
            switch(event.key) {
                case 'ArrowRight':
                    newTab = currentTab.nextElementSibling || document.querySelector('[role="tab"]:first-child');
                    break;
                case 'ArrowLeft':
                    newTab = currentTab.previousElementSibling || document.querySelector('[role="tab"]:last-child');
                    break;
                case 'Home':
                    newTab = document.querySelector('[role="tab"]:first-child');
                    break;
                case 'End':
                    newTab = document.querySelector('[role="tab"]:last-child');
                    break;
                default:
                    return;
            }
            
            event.preventDefault();
            activateTab(newTab);
        }
    </script>
</body>
</html>
```

### Модальное окно

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Модальное окно с ARIA</title>
    <style>
        .modal-backdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }
        
        .modal {
            background: white;
            padding: 20px;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
            position: relative;
        }
        
        .modal-close {
            position: absolute;
            top: 10px;
            right: 10px;
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
        }
        
        .hidden {
            display: none;
        }
    </style>
</head>
<body>
    <h1>Страница с модальным окном</h1>
    <p>Это основное содержимое страницы.</p>
    <button onclick="openModal()">Открыть модальное окно</button>
    
    <div id="modal-backdrop" class="modal-backdrop hidden" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal">
            <button class="modal-close" onclick="closeModal()" aria-label="Закрыть">&times;</button>
            <h2 id="modal-title">Заголовок модального окна</h2>
            <p>Содержимое модального окна.</p>
            <button onclick="closeModal()">Закрыть</button>
        </div>
    </div>
    
    <script>
        const modal = document.getElementById('modal-backdrop');
        const body = document.body;
        let originalAriaHidden;
        
        function openModal() {
            // Сохранить исходное состояние aria-hidden для основного содержимого
            const mainContent = document.querySelector('main') || document.body.firstElementChild;
            originalAriaHidden = mainContent.getAttribute('aria-hidden');
            mainContent.setAttribute('aria-hidden', 'true');
            
            modal.classList.remove('hidden');
            
            // Установить фокус на модальное окно
            modal.focus();
            
            // Добавить обработчик для закрытия по Escape
            document.addEventListener('keydown', handleEscapeKey);
        }
        
        function closeModal() {
            modal.classList.add('hidden');
            
            // Восстановить исходное состояние aria-hidden для основного содержимого
            const mainContent = document.querySelector('main') || document.body.firstElementChild;
            if (originalAriaHidden) {
                mainContent.setAttribute('aria-hidden', originalAriaHidden);
            } else {
                mainContent.removeAttribute('aria-hidden');
            }
            
            // Удалить обработчик
            document.removeEventListener('keydown', handleEscapeKey);
        }
        
        function handleEscapeKey(event) {
            if (event.key === 'Escape') {
                closeModal();
            }
        }
        
        // Обеспечить захват фокуса внутри модального окна
        modal.addEventListener('keydown', function(event) {
            if (event.key === 'Tab') {
                const focusableElements = modal.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                
                if (focusableElements.length === 0) return;
                
                const firstElement = focusableElements[0];
                const lastElement = focusableElements[focusableElements.length - 1];
                
                if (event.shiftKey) {
                    if (document.activeElement === firstElement) {
                        lastElement.focus();
                        event.preventDefault();
                    }
                } else {
                    if (document.activeElement === lastElement) {
                        firstElement.focus();
                        event.preventDefault();
                    }
                }
            }
        });
    </script>
</body>
</html>
```

## ARIA live области

ARIA live области используются для динамического обновления контента, чтобы вспомогательные технологии могли его обнаружить.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA Live области</title>
</head>
<body>
    <h1>Пример ARIA Live областей</h1>
    
    <!-- Политный (polite) - обновления будут объявлены, когда пользователь не занят -->
    <div id="status-updates" aria-live="polite" aria-atomic="false">
        <!-- Обновления статуса будут добавляться сюда -->
    </div>
    
    <!-- Настойчивый (assertive) - обновления будут объявлены немедленно -->
    <div id="alerts" aria-live="assertive" aria-atomic="true">
        <!-- Критические уведомления -->
    </div>
    
    <button onclick="addStatusUpdate()">Добавить статус</button>
    <button onclick="addAlert()">Добавить уведомление</button>
    
    <script>
        function addStatusUpdate() {
            const statusDiv = document.getElementById('status-updates');
            const timestamp = new Date().toLocaleTimeString();
            const newUpdate = document.createElement('p');
            newUpdate.textContent = `Обновление в ${timestamp}`;
            statusDiv.appendChild(newUpdate);
        }
        
        function addAlert() {
            const alertsDiv = document.getElementById('alerts');
            alertsDiv.textContent = `Критическое уведомление в ${new Date().toLocaleTimeString()}`;
        }
    </script>
</body>
</html>
```

## Лучшие практики ARIA

### 1. Используйте нативные HTML элементы, когда это возможно

```html
<!-- Правильно - используйте нативные элементы -->
<button>Кнопка</button>
<a href="/page">Ссылка</a>
<input type="text" placeholder="Текст">
<nav>Навигация</nav>

<!-- Избегайте ARIA, когда есть нативные элементы -->
<div role="button">Не используйте это</div>
```

### 2. Обеспечьте клавиатурную навигацию

```html
<!-- Элементы с ARIA ролями должны быть доступны с клавиатуры -->
<div role="button" tabindex="0" onclick="doSomething()" onkeydown="handleKeyDown(event)">
    Кнопка-имитация
</div>

<script>
function handleKeyDown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        doSomething();
    }
}
</script>
```

### 3. Обновляйте ARIA атрибуты при изменении состояния

```html
<button id="toggle-btn" 
        role="switch" 
        aria-checked="false"
        onclick="toggleState()">
    Переключатель
</button>

<script>
function toggleState() {
    const button = document.getElementById('toggle-btn');
    const currentState = button.getAttribute('aria-checked') === 'true';
    button.setAttribute('aria-checked', !currentState);
}
</script>
```

## Проверка доступности с ARIA

### Инструменты для проверки

1. **AXE (Accessibility Engine)** - для автоматизированной проверки
2. **WAVE** - веб-инструмент для проверки доступности
3. **Lighthouse** - встроенная проверка в Chrome DevTools
4. **NVDA или JAWS** - скринридеры для ручного тестирования

### Пример проверки с AXE

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Проверка доступности</title>
    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
</head>
<body>
    <div id="content">
        <h1>Заголовок</h1>
        <button role="button" aria-label="Закрыть">X</button>
    </div>
    
    <script>
        // Проверка доступности при загрузке страницы
        window.addEventListener('load', function() {
            axe.run('#content', {
                runOnly: {
                    type: 'tag',
                    values: ['wcag2a', 'wcag2aa']
                }
            }, function(err, results) {
                if (err) {
                    console.error('Ошибка проверки:', err);
                    return;
                }
                
                if (results.violations.length) {
                    console.log('Найдены нарушения доступности:');
                    results.violations.forEach(violation => {
                        console.log(violation.help, violation.nodes.map(node => node.html));
                    });
                } else {
                    console.log('Доступность в порядке!');
                }
            });
        });
    </script>
</body>
</html>
```

## Современные возможности ARIA

### ARIA в веб-компонентах
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA в веб-компонентах</title>
</head>
<body>
    <main>
        <accessible-modal title="Доступное модальное окно">
            <modal-content>
                <h2>Содержимое модального окна</h2>
                <p>Это содержимое находится в кастомном элементе с полной поддержкой ARIA.</p>
                <button id="close-modal-btn">Закрыть</button>
            </modal-content>
        </accessible-modal>
    </main>

    <script>
        class AccessibleModal extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const title = this.getAttribute('title') || 'Модальное окно';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .modal-backdrop {
                            position: fixed;
                            top: 0;
                            left: 0;
                            width: 100%;
                            height: 100%;
                            background: rgba(0,0,0,0.5);
                            display: flex;
                            align-items: center;
                            justify-content: center;
                            z-index: 1000;
                        }

                        .modal {
                            background: white;
                            padding: 20px;
                            border-radius: 8px;
                            max-width: 500px;
                            width: 90%;
                            position: relative;
                        }

                        .modal-close {
                            position: absolute;
                            top: 10px;
                            right: 10px;
                            background: none;
                            border: none;
                            font-size: 1.5em;
                            cursor: pointer;
                        }
                    </style>
                    
                    <div class="modal-backdrop" 
                         role="dialog" 
                         aria-modal="true" 
                         aria-labelledby="modal-title"
                         aria-hidden="true">
                        <div class="modal">
                            <button class="modal-close" aria-label="Закрыть модальное окно">&times;</button>
                            <h2 id="modal-title">${title}</h2>
                            <div id="modal-content">
                                <slot name="content"></slot>
                            </div>
                        </div>
                    </div>
                `;
                
                this.backdrop = this.shadowRoot.querySelector('.modal-backdrop');
                this.closeButton = this.shadowRoot.querySelector('.modal-close');
                this.modalContent = this.shadowRoot.querySelector('#modal-content');
                
                this.closeButton.addEventListener('click', () => this.close());
                
                // Обработка закрытия по Escape
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'Escape' && this.isOpen) {
                        this.close();
                    }
                });
                
                // Обработка захвата фокуса
                this.backdrop.addEventListener('keydown', (e) => {
                    if (e.key === 'Tab') {
                        this.trapFocus(e);
                    }
                });
            }
            
            connectedCallback() {
                // Открытие модального окна при добавлении в DOM с атрибутом open
                if (this.hasAttribute('open')) {
                    this.open();
                }
            }
            
            open() {
                this.isOpen = true;
                this.backdrop.setAttribute('aria-hidden', 'false');
                
                // Скрытие основного содержимого для скринридеров
                const mainContent = document.querySelector('main') || document.body;
                mainContent.setAttribute('aria-hidden', 'true');
                
                // Установка фокуса на модальное окно
                this.backdrop.focus();
            }
            
            close() {
                this.isOpen = false;
                this.backdrop.setAttribute('aria-hidden', 'true');
                
                // Восстановление основного содержимого
                const mainContent = document.querySelector('main') || document.body;
                mainContent.removeAttribute('aria-hidden');
            }
            
            trapFocus(e) {
                const focusableElements = this.backdrop.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                
                if (focusableElements.length === 0) return;
                
                const firstElement = focusableElements[0];
                const lastElement = focusableElements[focusableElements.length - 1];
                
                if (e.shiftKey) {
                    if (document.activeElement === firstElement) {
                        lastElement.focus();
                        e.preventDefault();
                    }
                } else {
                    if (document.activeElement === lastElement) {
                        firstElement.focus();
                        e.preventDefault();
                    }
                }
            }
        }
        
        customElements.define('accessible-modal', AccessibleModal);
        
        // Открытие модального окна
        function openModal() {
            const modal = document.querySelector('accessible-modal');
            modal.setAttribute('open', '');
            modal.open();
        }
        
        // Закрытие модального окна
        document.getElementById('close-modal-btn').addEventListener('click', () => {
            const modal = document.querySelector('accessible-modal');
            modal.close();
        });
    </script>
    
    <button onclick="openModal()">Открыть модальное окно</button>
</body>
</html>
```

### ARIA с WebSockets и динамическим контентом
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA с динамическим контентом</title>
</head>
<body>
    <main>
        <h1>Чат с ARIA-объявлениями</h1>
        
        <div id="chat-container" 
             role="log" 
             aria-label="Чат"
             aria-live="polite"
             aria-atomic="false">
            <div id="messages" 
                 role="list" 
                 aria-label="Сообщения чата"></div>
        </div>
        
        <form id="message-form" role="form" aria-label="Форма отправки сообщения">
            <label for="message-input">Введите сообщение:</label>
            <input type="text" 
                   id="message-input" 
                   name="message" 
                   aria-describedby="message-help"
                   required>
            <div id="message-help" class="sr-only">Введите сообщение и нажмите Enter для отправки</div>
            <button type="submit" aria-describedby="send-help">Отправить</button>
            <div id="send-help" class="sr-only">Отправить сообщение в чат</div>
        </form>
        
        <div id="status-announcer" 
             aria-live="polite" 
             aria-atomic="true"
             class="sr-only"></div>
    </main>

    <script>
        // Подключение к WebSocket
        const ws = new WebSocket('ws://localhost:8080/chat');
        const messagesContainer = document.getElementById('messages');
        const statusAnnouncer = document.getElementById('status-announcer');
        
        ws.onopen = function() {
            announceStatus('Соединение с чатом установлено');
        };
        
        ws.onmessage = function(event) {
            const messageData = JSON.parse(event.data);
            addMessage(messageData);
        };
        
        ws.onerror = function(error) {
            announceStatus('Ошибка соединения с чатом');
        };
        
        ws.onclose = function() {
            announceStatus('Соединение с чатом закрыто');
        };
        
        document.getElementById('message-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const input = document.getElementById('message-input');
            if (input.value.trim()) {
                const message = {
                    text: input.value,
                    timestamp: new Date().toISOString(),
                    user: 'current-user'
                };
                
                ws.send(JSON.stringify(message));
                input.value = '';
                
                announceStatus('Сообщение отправлено');
            }
        });
        
        function addMessage(messageData) {
            const messageElement = document.createElement('div');
            messageElement.className = 'message';
            messageElement.setAttribute('role', 'listitem');
            messageElement.setAttribute('aria-label', `Сообщение от ${messageData.user}`);
            
            const time = new Date(messageData.timestamp).toLocaleTimeString();
            messageElement.innerHTML = `
                <strong>${escapeHtml(messageData.user)}</strong> 
                <small>(${time})</small>
                <p>${escapeHtml(messageData.text)}</p>
            `;
            
            messagesContainer.appendChild(messageElement);
            
            // Прокрутка к новому сообщению
            messageElement.scrollIntoView({ behavior: 'smooth' });
        }
        
        function announceStatus(message) {
            statusAnnouncer.textContent = message;
            
            // Очистка объявления через 5 секунд
            setTimeout(() => {
                if (statusAnnouncer.textContent === message) {
                    statusAnnouncer.textContent = '';
                }
            }, 5000);
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
    </script>
    
    <style>
        #chat-container {
            border: 1px solid #ccc;
            height: 400px;
            overflow-y: auto;
            padding: 10px;
            margin-bottom: 20px;
        }
        
        .message {
            padding: 8px;
            margin-bottom: 10px;
            border-radius: 4px;
            background-color: #f9f9f9;
        }
        
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
        
        form {
            display: flex;
            gap: 10px;
        }
        
        input[type="text"] {
            flex: 1;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        
        button {
            padding: 8px 16px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

### ARIA с IndexedDB и оффлайн-доступностью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>ARIA с оффлайн-функциональностью</title>
</head>
<body>
    <main>
        <h1>Оффлайн-заметки с ARIA</h1>
        
        <div id="app-status" 
             aria-live="polite" 
             aria-atomic="true"
             role="status"></div>
        
        <form id="note-form" role="form" aria-label="Форма создания заметки">
            <div class="form-group">
                <label for="note-title">Заголовок:</label>
                <input type="text" 
                       id="note-title" 
                       name="title" 
                       required
                       aria-describedby="title-help">
                <div id="title-help" class="sr-only">Введите заголовок заметки</div>
            </div>
            
            <div class="form-group">
                <label for="note-content">Содержимое:</label>
                <textarea id="note-content" 
                          name="content" 
                          rows="4" 
                          required
                          aria-describedby="content-help"></textarea>
                <div id="content-help" class="sr-only">Введите содержимое заметки</div>
            </div>
            
            <button type="submit" aria-describedby="save-help">Сохранить</button>
            <div id="save-help" class="sr-only">Сохранить заметку в оффлайн-хранилище</div>
        </form>
        
        <section id="notes-section" 
                 role="region" 
                 aria-labelledby="notes-title"
                 aria-live="polite">
            <h2 id="notes-title">Сохраненные заметки</h2>
            <div id="notes-container" role="list"></div>
        </section>
    </main>

    <script>
        // Подключение к IndexedDB
        const request = indexedDB.open('AccessibleNotesDB', 1);
        let db;
        
        request.onerror = function(event) {
            updateStatus('Ошибка при открытии базы данных', 'error');
        };
        
        request.onsuccess = function(event) {
            db = event.target.result;
            updateStatus('База данных открыта', 'success');
            loadNotes();
        };
        
        request.onupgradeneeded = function(event) {
            db = event.target.result;
            
            if (!db.objectStoreNames.contains('notes')) {
                const store = db.createObjectStore('notes', { keyPath: 'id', autoIncrement: true });
                store.createIndex('title', 'title', { unique: false });
                store.createIndex('date', 'date', { unique: false });
            }
        };
        
        document.getElementById('note-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const title = document.getElementById('note-title').value.trim();
            const content = document.getElementById('note-content').value.trim();
            
            if (title && content) {
                const note = {
                    title: title,
                    content: content,
                    date: new Date().toISOString(),
                    status: 'saved'
                };
                
                saveNote(note);
                this.reset();
            }
        });
        
        function saveNote(note) {
            const transaction = db.transaction(['notes'], 'readwrite');
            const store = transaction.objectStore('notes');
            const request = store.add(note);
            
            request.onsuccess = function() {
                updateStatus('Заметка сохранена', 'success');
                loadNotes();
            };
            
            request.onerror = function() {
                updateStatus('Ошибка при сохранении заметки', 'error');
            };
        }
        
        function loadNotes() {
            const transaction = db.transaction(['notes'], 'readonly');
            const store = transaction.objectStore('notes');
            const request = store.getAll();
            
            request.onsuccess = function() {
                displayNotes(request.result);
            };
        }
        
        function displayNotes(notes) {
            const container = document.getElementById('notes-container');
            container.innerHTML = '';
            
            if (notes.length === 0) {
                container.innerHTML = '<p>Нет сохраненных заметок.</p>';
                return;
            }
            
            // Сортировка по дате
            notes.sort((a, b) => new Date(b.date) - new Date(a.date));
            
            notes.forEach(note => {
                const noteElement = document.createElement('div');
                noteElement.className = 'note';
                noteElement.setAttribute('role', 'listitem');
                noteElement.setAttribute('aria-label', `Заметка: ${note.title}`);
                
                const date = new Date(note.date).toLocaleString('ru-RU');
                
                noteElement.innerHTML = `
                    <h3>${escapeHtml(note.title)}</h3>
                    <time datetime="${note.date}">${date}</time>
                    <div class="note-content">${escapeHtml(note.content)}</div>
                    <div class="note-actions">
                        <button type="button" 
                                onclick="deleteNote(${note.id})"
                                aria-label="Удалить заметку: ${escapeHtml(note.title)}">
                            Удалить
                        </button>
                    </div>
                `;
                
                container.appendChild(noteElement);
            });
        }
        
        function deleteNote(id) {
            if (confirm('Удалить эту заметку?')) {
                const transaction = db.transaction(['notes'], 'readwrite');
                const store = transaction.objectStore('notes');
                const request = store.delete(id);
                
                request.onsuccess = function() {
                    updateStatus('Заметка удалена', 'success');
                    loadNotes();
                };
                
                request.onerror = function() {
                    updateStatus('Ошибка при удалении заметки', 'error');
                };
            }
        }
        
        function updateStatus(message, type = 'info') {
            const statusDiv = document.getElementById('app-status');
            statusDiv.textContent = message;
            statusDiv.className = `status status-${type}`;
            
            // Объявление для скринридера
            announceToScreenReader(message);
            
            // Автоматическое скрытие через 5 секунд
            setTimeout(() => {
                if (statusDiv.textContent === message) {
                    statusDiv.textContent = '';
                    statusDiv.className = 'status';
                }
            }, 5000);
        }
        
        function announceToScreenReader(message) {
            const announcement = document.createElement('div');
            announcement.setAttribute('aria-live', 'polite');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.className = 'sr-only';
            announcement.textContent = message;
            
            document.body.appendChild(announcement);
            
            setTimeout(() => {
                if (document.body.contains(announcement)) {
                    document.body.removeChild(announcement);
                }
            }, 1000);
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
    </script>
    
    <style>
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        
        .note {
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 15px;
            margin-bottom: 10px;
            background-color: #fafafa;
        }
        
        .note h3 {
            margin-top: 0;
        }
        
        .note-actions {
            margin-top: 10px;
            text-align: right;
        }
        
        .status {
            padding: 10px;
            margin-bottom: 15px;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .status-success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .status-error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
        
        button {
            padding: 8px 16px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:focus {
            outline: 2px solid #005a9e;
            outline-offset: 2px;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Доступная таблица с сортировкой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная таблица с сортировкой</title>
</head>
<body>
    <main>
        <h1>Данные пользователей</h1>
        
        <div class="table-controls" role="toolbar" aria-label="Управление таблицей">
            <button id="sort-name" 
                    class="sort-btn" 
                    data-column="name"
                    aria-describedby="sort-help"
                    aria-pressed="false">
                Сортировать по имени
                <span class="sort-indicator" aria-hidden="true"></span>
            </button>
            
            <button id="sort-date" 
                    class="sort-btn" 
                    data-column="date"
                    aria-describedby="sort-help"
                    aria-pressed="false">
                Сортировать по дате
                <span class="sort-indicator" aria-hidden="true"></span>
            </button>
        </div>
        
        <div id="sort-help" class="sr-only">
            Нажмите кнопку сортировки для изменения порядка строк в таблице
        </div>
        
        <table role="grid" aria-label="Таблица пользователей">
            <thead>
                <tr role="row">
                    <th role="columnheader" 
                        id="name-header"
                        aria-sort="none"
                        tabindex="0"
                        aria-describedby="name-desc">
                        Имя
                        <span class="sort-indicator" aria-hidden="true"></span>
                    </th>
                    <th role="columnheader" 
                        id="email-header"
                        aria-sort="none">
                        Email
                    </th>
                    <th role="columnheader" 
                        id="date-header"
                        aria-sort="none"
                        tabindex="0"
                        aria-describedby="date-desc">
                        Дата регистрации
                        <span class="sort-indicator" aria-hidden="true"></span>
                    </th>
                </tr>
            </thead>
            <tbody id="table-body" role="rowgroup">
                <tr role="row">
                    <td role="gridcell">Иван Иванов</td>
                    <td role="gridcell">ivan@example.com</td>
                    <td role="gridcell"><time datetime="2023-01-15">15.01.2023</time></td>
                </tr>
                <tr role="row">
                    <td role="gridcell">Мария Петрова</td>
                    <td role="gridcell">maria@example.com</td>
                    <td role="gridcell"><time datetime="2023-02-20">20.02.2023</time></td>
                </tr>
                <tr role="row">
                    <td role="gridcell">Алексей Сидоров</td>
                    <td role="gridcell">alexey@example.com</td>
                    <td role="gridcell"><time datetime="2023-03-10">10.03.2023</time></td>
                </tr>
            </tbody>
        </table>
    </main>

    <script>
        class AccessibleTableSorter {
            constructor() {
                this.data = [
                    { name: 'Иван Иванов', email: 'ivan@example.com', date: '2023-01-15' },
                    { name: 'Мария Петрова', email: 'maria@example.com', date: '2023-02-20' },
                    { name: 'Алексей Сидоров', email: 'alexey@example.com', date: '2023-03-10' }
                ];
                
                this.currentSort = { column: null, direction: null };
                
                this.init();
            }
            
            init() {
                // Добавляем обработчики сортировки
                document.querySelectorAll('.sort-btn').forEach(btn => {
                    btn.addEventListener('click', (e) => this.handleSort(e.target));
                    btn.addEventListener('keydown', (e) => {
                        if (e.key === 'Enter' || e.key === ' ') {
                            e.preventDefault();
                            this.handleSort(e.target);
                        }
                    });
                });
                
                // Добавляем обработчики для заголовков
                document.querySelectorAll('th[tabindex="0"]').forEach(header => {
                    header.addEventListener('click', (e) => this.handleHeaderSort(e.target));
                    header.addEventListener('keydown', (e) => {
                        if (e.key === 'Enter' || e.key === ' ') {
                            e.preventDefault();
                            this.handleHeaderSort(e.target);
                        }
                    });
                });
            }
            
            handleSort(button) {
                const column = button.getAttribute('data-column');
                this.sort(column);
            }
            
            handleHeaderSort(header) {
                // Определяем колонку по ID заголовка
                const column = header.id.replace('-header', '');
                this.sort(column);
            }
            
            sort(column) {
                // Определяем направление сортировки
                let direction;
                if (this.currentSort.column === column) {
                    // Переключаем направление
                    direction = this.currentSort.direction === 'ascending' ? 'descending' : 'ascending';
                } else {
                    // Новая сортировка - по возрастанию
                    direction = 'ascending';
                }
                
                // Обновляем состояние сортировки
                this.currentSort = { column, direction };
                
                // Сортируем данные
                this.data.sort((a, b) => {
                    let aValue = a[column];
                    let bValue = b[column];
                    
                    // Для дат преобразуем в формат, пригодный для сравнения
                    if (column === 'date') {
                        aValue = new Date(aValue);
                        bValue = new Date(bValue);
                    }
                    
                    let result = 0;
                    if (aValue < bValue) result = -1;
                    if (aValue > bValue) result = 1;
                    
                    return direction === 'ascending' ? result : -result;
                });
                
                // Обновляем интерфейс
                this.updateUI();
                
                // Объявляем сортировку для скринридера
                this.announceSort(column, direction);
            }
            
            updateUI() {
                // Обновляем индикаторы сортировки
                document.querySelectorAll('[aria-sort]').forEach(header => {
                    header.setAttribute('aria-sort', 'none');
                    const indicator = header.querySelector('.sort-indicator');
                    if (indicator) indicator.textContent = '';
                });
                
                // Устанавливаем текущую сортировку
                if (this.currentSort.column) {
                    const header = document.getElementById(`${this.currentSort.column}-header`);
                    if (header) {
                        header.setAttribute('aria-sort', this.currentSort.direction);
                        const indicator = header.querySelector('.sort-indicator');
                        if (indicator) {
                            indicator.textContent = this.currentSort.direction === 'ascending' ? '↑' : '↓';
                        }
                    }
                }
                
                // Обновляем тело таблицы
                const tbody = document.getElementById('table-body');
                tbody.innerHTML = '';
                
                this.data.forEach(item => {
                    const row = document.createElement('tr');
                    row.setAttribute('role', 'row');
                    
                    const nameCell = document.createElement('td');
                    nameCell.setAttribute('role', 'gridcell');
                    nameCell.textContent = item.name;
                    
                    const emailCell = document.createElement('td');
                    emailCell.setAttribute('role', 'gridcell');
                    emailCell.textContent = item.email;
                    
                    const dateCell = document.createElement('td');
                    dateCell.setAttribute('role', 'gridcell');
                    const date = document.createElement('time');
                    date.setAttribute('datetime', item.date);
                    date.textContent = new Date(item.date).toLocaleDateString('ru-RU');
                    dateCell.appendChild(date);
                    
                    row.appendChild(nameCell);
                    row.appendChild(emailCell);
                    row.appendChild(dateCell);
                    
                    tbody.appendChild(row);
                });
            }
            
            announceSort(column, direction) {
                const columnNames = {
                    'name': 'имени',
                    'date': 'дате регистрации'
                };
                
                const directionText = direction === 'ascending' ? 'по возрастанию' : 'по убыванию';
                
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = `Таблица отсортирована по ${columnNames[column] || column} ${directionText}`;
                
                document.body.appendChild(announcement);
                
                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 2000);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleTableSorter();
        });
    </script>
    
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        
        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        
        th {
            background-color: #f2f2f2;
            font-weight: bold;
            position: relative;
        }
        
        .sort-indicator {
            margin-left: 5px;
            font-size: 0.8em;
        }
        
        .sort-btn {
            padding: 8px 12px;
            margin-right: 10px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .sort-btn:focus {
            outline: 2px solid #005a9e;
            outline-offset: 2px;
        }
        
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

### Доступный календарь
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступный календарь</title>
</head>
<body>
    <main>
        <h1>Календарь мероприятий</h1>
        
        <div id="calendar-container" 
             role="application" 
             aria-label="Календарь"
             aria-describedby="calendar-help">
            <div id="calendar-header">
                <button id="prev-month" aria-label="Предыдущий месяц">&lt;</button>
                <h2 id="current-month" aria-live="polite">Ноябрь 2023</h2>
                <button id="next-month" aria-label="Следующий месяц">&gt;</button>
            </div>
            
            <div id="calendar-grid" 
                 role="grid" 
                 aria-labelledby="current-month">
                <div role="rowgroup">
                    <div role="row">
                        <div role="columnheader" aria-label="Воскресенье">Вс</div>
                        <div role="columnheader" aria-label="Понедельник">Пн</div>
                        <div role="columnheader" aria-label="Вторник">Вт</div>
                        <div role="columnheader" aria-label="Среда">Ср</div>
                        <div role="columnheader" aria-label="Четверг">Чт</div>
                        <div role="columnheader" aria-label="Пятница">Пт</div>
                        <div role="columnheader" aria-label="Суббота">Сб</div>
                    </div>
                </div>
                
                <div id="calendar-days" role="rowgroup"></div>
            </div>
        </div>
        
        <div id="calendar-help" class="sr-only">
            Используйте клавиши со стрелками для навигации по дням календаря
        </div>
        
        <div id="selected-date" 
             aria-live="polite" 
             aria-atomic="true"
             class="sr-only"></div>
    </main>

    <script>
        class AccessibleCalendar {
            constructor() {
                this.currentDate = new Date(2023, 10, 1); // Ноябрь 2023
                this.selectedDate = null;
                
                this.init();
            }
            
            init() {
                this.renderCalendar();
                
                // Обработчики навигации
                document.getElementById('prev-month').addEventListener('click', () => this.previousMonth());
                document.getElementById('next-month').addEventListener('click', () => this.nextMonth());
                
                // Клавиатурная навигация
                document.addEventListener('keydown', (e) => {
                    if (this.selectedDate) {
                        this.handleKeyNavigation(e);
                    }
                });
            }
            
            renderCalendar() {
                const firstDay = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth(), 1);
                const lastDay = new Date(this.currentDate.getFullYear(), this.currentDate.getMonth() + 1, 0);
                const startDate = new Date(firstDay);
                startDate.setDate(startDate.getDate() - firstDay.getDay()); // Начало недели (воскресенье)
                
                const calendarDays = document.getElementById('calendar-days');
                calendarDays.innerHTML = '';
                
                // Обновляем заголовок
                document.getElementById('current-month').textContent = 
                    `${this.getMonthName(this.currentDate.getMonth())} ${this.currentDate.getFullYear()}`;
                
                let currentDay = new Date(startDate);
                
                for (let week = 0; week < 6; week++) {
                    const weekRow = document.createElement('div');
                    weekRow.setAttribute('role', 'row');
                    
                    for (let day = 0; day < 7; day++) {
                        const dayCell = document.createElement('div');
                        dayCell.setAttribute('role', 'gridcell');
                        
                        const isCurrentMonth = currentDay.getMonth() === this.currentDate.getMonth();
                        const isToday = this.isToday(currentDay);
                        const isSelected = this.selectedDate && 
                                         currentDay.toDateString() === this.selectedDate.toDateString();
                        
                        dayCell.textContent = currentDay.getDate();
                        dayCell.className = 'calendar-day';
                        
                        if (isCurrentMonth) {
                            dayCell.classList.add('current-month');
                        } else {
                            dayCell.classList.add('other-month');
                            dayCell.setAttribute('aria-hidden', 'true');
                        }
                        
                        if (isToday) {
                            dayCell.classList.add('today');
                            dayCell.setAttribute('aria-label', `Сегодня, ${this.formatDate(currentDay)}`);
                        } else {
                            dayCell.setAttribute('aria-label', this.formatDate(currentDay));
                        }
                        
                        if (isSelected) {
                            dayCell.classList.add('selected');
                            dayCell.setAttribute('aria-selected', 'true');
                        }
                        
                        dayCell.setAttribute('tabindex', isCurrentMonth ? '0' : '-1');
                        
                        // Обработчики событий
                        dayCell.addEventListener('click', () => this.selectDate(currentDay));
                        dayCell.addEventListener('keydown', (e) => {
                            if (e.key === 'Enter' || e.key === ' ') {
                                e.preventDefault();
                                this.selectDate(currentDay);
                            }
                        });
                        
                        weekRow.appendChild(dayCell);
                        currentDay.setDate(currentDay.getDate() + 1);
                    }
                    
                    calendarDays.appendChild(weekRow);
                }
            }
            
            selectDate(date) {
                // Снимаем выделение с предыдущего дня
                if (this.selectedDate) {
                    const prevSelected = document.querySelector('.calendar-day[aria-selected="true"]');
                    if (prevSelected) {
                        prevSelected.setAttribute('aria-selected', 'false');
                        prevSelected.classList.remove('selected');
                    }
                }
                
                // Устанавливаем новую дату
                this.selectedDate = new Date(date);
                
                // Выделяем новый день
                const dayElement = document.querySelector(`.calendar-day[tabindex="0"]:not(.other-month):nth-child(${(date.getDate() - 1) % 7 + 1})`);
                if (dayElement) {
                    dayElement.setAttribute('aria-selected', 'true');
                    dayElement.classList.add('selected');
                    dayElement.focus();
                }
                
                // Объявляем выбранную дату
                this.announceDate(date);
            }
            
            handleKeyNavigation(e) {
                if (!this.selectedDate) return;
                
                const newDate = new Date(this.selectedDate);
                
                switch(e.key) {
                    case 'ArrowLeft':
                        newDate.setDate(newDate.getDate() - 1);
                        e.preventDefault();
                        break;
                    case 'ArrowRight':
                        newDate.setDate(newDate.getDate() + 1);
                        e.preventDefault();
                        break;
                    case 'ArrowUp':
                        newDate.setDate(newDate.getDate() - 7);
                        e.preventDefault();
                        break;
                    case 'ArrowDown':
                        newDate.setDate(newDate.getDate() + 7);
                        e.preventDefault();
                        break;
                    default:
                        return;
                }
                
                // Проверяем, остаемся ли в том же месяце
                if (newDate.getMonth() !== this.currentDate.getMonth()) {
                    // Если перешли в другой месяц, обновляем календарь
                    this.currentDate = new Date(newDate);
                    this.renderCalendar();
                }
                
                this.selectDate(newDate);
            }
            
            previousMonth() {
                this.currentDate.setMonth(this.currentDate.getMonth() - 1);
                this.selectedDate = null;
                this.renderCalendar();
            }
            
            nextMonth() {
                this.currentDate.setMonth(this.currentDate.getMonth() + 1);
                this.selectedDate = null;
                this.renderCalendar();
            }
            
            isToday(date) {
                const today = new Date();
                return date.getDate() === today.getDate() &&
                       date.getMonth() === today.getMonth() &&
                       date.getFullYear() === today.getFullYear();
            }
            
            formatDate(date) {
                return date.toLocaleDateString('ru-RU', {
                    weekday: 'long',
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric'
                });
            }
            
            getMonthName(month) {
                const months = [
                    'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
                    'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'
                ];
                return months[month];
            }
            
            announceDate(date) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = `Выбрана дата: ${this.formatDate(date)}`;
                
                document.body.appendChild(announcement);
                
                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 2000);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleCalendar();
        });
    </script>
    
    <style>
        #calendar-container {
            max-width: 500px;
            margin: 0 auto;
            border: 1px solid #ccc;
            border-radius: 8px;
            padding: 20px;
        }
        
        #calendar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }
        
        #calendar-header h2 {
            margin: 0;
            font-size: 1.2em;
        }
        
        #calendar-header button {
            background: none;
            border: 1px solid #ccc;
            border-radius: 4px;
            padding: 5px 10px;
            cursor: pointer;
        }
        
        #calendar-grid {
            border-collapse: collapse;
        }
        
        [role="row"] {
            display: flex;
        }
        
        [role="columnheader"], .calendar-day {
            flex: 1;
            text-align: center;
            padding: 10px;
            border: 1px solid #eee;
            min-height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .calendar-day:focus {
            outline: 2px solid #007acc;
            outline-offset: -2px;
        }
        
        .current-month {
            background-color: #f9f9f9;
        }
        
        .other-month {
            color: #ccc;
        }
        
        .today {
            background-color: #e6f7ff;
            border-color: #007acc;
        }
        
        .selected {
            background-color: #007acc;
            color: white;
        }
        
        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

## Проверка и тестирование ARIA

### Инструменты для проверки ARIA:
1. **AXE-core** - для автоматизированной проверки ARIA атрибутов
2. **WAVE** - веб-инструмент с подсветкой ARIA ошибок
3. **Accessibility Insights** - расширение с проверкой ARIA
4. **Chrome DevTools** - вкладка Accessibility
5. **NVDA/JAWS/VoiceOver** - ручное тестирование с скринридерами

### Проверка с AXE-core:
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Проверка ARIA с AXE</title>
    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
</head>
<body>
    <main id="content">
        <h1>Тест ARIA</h1>
        
        <div role="button" 
             tabindex="0" 
             aria-label="Кнопка для тестирования"
             onclick="handleClick()">
            Тестовая кнопка
        </div>
        
        <div id="result" role="status" aria-live="polite"></div>
    </main>

    <script>
        function handleClick() {
            document.getElementById('result').textContent = 'Кнопка нажата!';
        }
        
        // Функция проверки ARIA
        function checkAria() {
            axe.run('#content', {
                runOnly: {
                    type: 'rule',
                    values: [
                        'aria-allowed-attr',    // Проверка разрешенных ARIA атрибутов
                        'aria-required-attr',   // Проверка обязательных ARIA атрибутов
                        'aria-required-children', // Проверка обязательных дочерних элементов
                        'aria-required-parent',   // Проверка обязательных родительских элементов
                        'aria-roles',            // Проверка корректных ARIA ролей
                        'aria-valid-attr',       // Проверка валидных ARIA атрибутов
                        'aria-valid-attr-value'  // Проверка валидных значений ARIA атрибутов
                    ]
                }
            }, function(err, results) {
                if (err) {
                    console.error('Ошибка проверки ARIA:', err);
                    return;
                }
                
                if (results.violations.length > 0) {
                    console.log('Найдены ошибки ARIA:');
                    results.violations.forEach(violation => {
                        console.log(`- ${violation.help}:`);
                        violation.nodes.forEach(node => {
                            console.log(`  * ${node.html}`);
                            console.log(`  * ${node.failureSummary}`);
                        });
                    });
                    
                    document.getElementById('result').innerHTML = 
                        `<div style="color: red;">Найдено ${results.violations.length} ошибок ARIA</div>`;
                } else {
                    console.log('ARIA атрибуты корректны!');
                    document.getElementById('result').innerHTML = 
                        '<div style="color: green;">ARIA атрибуты корректны!</div>';
                }
            });
        }
        
        // Проверка при загрузке
        window.addEventListener('load', checkAria);
    </script>
    
    <button onclick="checkAria()">Проверить ARIA</button>
</body>
</html>
```

## Лучшие практики ARIA

### 1. Приоритет нативным HTML элементам
```html
<!-- Хорошо: использование нативных элементов -->
<button>Нажми меня</button>
<a href="/page">Ссылка</a>
<input type="text" placeholder="Введите текст">
<nav>Навигация</nav>

<!-- Плохо: избыточное использование ARIA -->
<div role="button">Не используйте это</div>
<div role="link">Тоже избегайте</div>
<div role="navigation">Лучше используйте nav</div>
```

### 2. Обновление ARIA атрибутов при изменении состояния
```html
<!-- Переключатель с правильным обновлением состояния -->
<button id="toggle"
        role="switch"
        aria-checked="false"
        onclick="toggleSwitch()">
    <span id="switch-label">Выключено</span>
</button>

<script>
function toggleSwitch() {
    const button = document.getElementById('toggle');
    const label = document.getElementById('switch-label');
    
    const isCurrentlyChecked = button.getAttribute('aria-checked') === 'true';
    const newCheckedState = !isCurrentlyChecked;
    
    button.setAttribute('aria-checked', newCheckedState);
    label.textContent = newCheckedState ? 'Включено' : 'Выключено';
    
    // Объявление изменения для скринридера
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', 'polite');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.style.position = 'absolute';
    announcement.style.left = '-9999px';
    announcement.textContent = newCheckedState ? 'Переключатель включен' : 'Переключатель выключен';
    
    document.body.appendChild(announcement);
    
    setTimeout(() => {
        if (document.body.contains(announcement)) {
            document.body.removeChild(announcement);
        }
    }, 1000);
}
</script>
```

### 3. Понятные и информативные метки
```html
<!-- Хорошо: информативные ARIA метки -->
<button aria-label="Увеличить шрифт">A+</button>
<button aria-label="Уменьшить шрифт">A-</button>
<button aria-label="Закрыть модальное окно">×</button>

<!-- Плохо: неинформативные метки -->
<button aria-label="Кнопка">A+</button>
<button aria-label="Это кнопка">×</button>
```

## Заключение

ARIA является мощным инструментом для улучшения доступности веб-приложений, особенно для сложных интерактивных элементов. Правильное использование ARIA атрибутов помогает вспомогательным технологиям лучше понимать структуру и поведение веб-страницы. Важно помнить о следующем:

1. Используйте ARIA только когда нативных HTML элементов недостаточно
2. Обновляйте ARIA атрибуты при изменении состояния элементов
3. Обеспечьте клавиатурную навигацию для элементов с ARIA ролями
4. Регулярно тестируйте доступность с помощью инструментов и скринридеров
5. Используйте понятные и информативные ARIA метки
6. Обеспечьте совместимость с современными вспомогательными технологиями
7. Следите за обновлениями спецификаций ARIA
8. Интегрируйте проверки ARIA в процесс разработки
9. Избегайте дублирования нативных семантических возможностей HTML
10. Следите за производительностью при использовании сложных ARIA паттернов

Современные подходы к использованию ARIA включают интеграцию с веб-компонентами, динамическим контентом, оффлайн-функциональностью и другими передовыми веб-технологиями. Это позволяет создавать действительно доступные веб-приложения для всех пользователей.

## Современные продвинутые паттерны ARIA

### 1. ARIA в Single Page Applications (SPA)

В современных одностраничных приложениях важно управлять фокусом и объявлениями при динамическом обновлении контента:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>SPA с ARIA</title>
</head>
<body>
    <div id="app" role="main">
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="#home" onclick="navigate('home')">Главная</a></li>
                <li><a href="#profile" onclick="navigate('profile')">Профиль</a></li>
                <li><a href="#settings" onclick="navigate('settings')">Настройки</a></li>
            </ul>
        </nav>

        <!-- Контентная область с ARIA-атрибутами для SPA -->
        <div id="content" 
             role="region" 
             aria-live="polite" 
             aria-label="Основной контент"
             aria-busy="false">
            <div id="home-content">
                <h1>Главная страница</h1>
                <p>Добро пожаловать в наше приложение!</p>
            </div>
        </div>

        <!-- Объявление навигации для скринридеров -->
        <div id="navigation-announcer" 
             aria-live="assertive" 
             aria-atomic="true" 
             class="sr-only">
        </div>
    </div>

    <script>
        class SPANavigator {
            constructor() {
                this.contentContainer = document.getElementById('content');
                this.navigationAnnouncer = document.getElementById('navigation-announcer');
                this.currentPage = 'home';
                
                this.pages = {
                    home: () => this.renderHome(),
                    profile: () => this.renderProfile(),
                    settings: () => this.renderSettings()
                };
            }

            navigate(pageId) {
                if (!this.pages[pageId]) return;

                // Объявление о смене страницы
                this.announceNavigation(pageId);
                
                // Установка состояния загрузки
                this.contentContainer.setAttribute('aria-busy', 'true');
                
                // Задержка для имитации загрузки
                setTimeout(() => {
                    // Рендер новой страницы
                    this.pages[pageId]();
                    
                    // Обновление текущей страницы
                    this.currentPage = pageId;
                    
                    // Сброс состояния загрузки
                    this.contentContainer.setAttribute('aria-busy', 'false');
                    
                    // Установка фокуса на основной контент
                    this.contentContainer.focus();
                }, 300);
            }

            announceNavigation(pageId) {
                const pageNames = {
                    'home': 'Главная страница',
                    'profile': 'Страница профиля',
                    'settings': 'Страница настроек'
                };
                
                this.navigationAnnouncer.textContent = 
                    `Навигация к ${pageNames[pageId] || 'неизвестной странице'}`;
            }

            renderHome() {
                this.contentContainer.innerHTML = `
                    <div id="home-content">
                        <h1>Главная страница</h1>
                        <p>Добро пожаловать в наше приложение!</p>
                        <button onclick="spaNavigator.navigateToProfile()">Перейти к профилю</button>
                    </div>
                `;
            }

            renderProfile() {
                this.contentContainer.innerHTML = `
                    <div id="profile-content">
                        <h1>Профиль пользователя</h1>
                        <form id="profile-form">
                            <div class="form-group">
                                <label for="username">Имя пользователя:</label>
                                <input type="text" id="username" name="username" value="ivan_ivanov">
                            </div>
                            <div class="form-group">
                                <label for="email">Email:</label>
                                <input type="email" id="email" name="email" value="ivan@example.com">
                            </div>
                            <button type="submit">Сохранить</button>
                        </form>
                    </div>
                `;
            }

            renderSettings() {
                this.contentContainer.innerHTML = `
                    <div id="settings-content">
                        <h1>Настройки</h1>
                        <form id="settings-form">
                            <div class="form-group">
                                <label>
                                    <input type="checkbox" name="notifications" checked>
                                    Уведомления
                                </label>
                            </div>
                            <div class="form-group">
                                <label>
                                    <input type="checkbox" name="dark-mode">
                                    Темная тема
                                </label>
                            </div>
                            <button type="button" onclick="spaNavigator.resetApp()">Сбросить настройки</button>
                        </form>
                    </div>
                `;
            }

            navigateToProfile() {
                this.navigate('profile');
            }

            resetApp() {
                this.navigate('home');
            }
        }

        // Глобальный экземпляр навигатора
        const spaNavigator = new SPANavigator();

        function navigate(pageId) {
            spaNavigator.navigate(pageId);
        }
    </script>

    <style>
        #content {
            min-height: 400px;
            padding: 20px;
        }

        .form-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            margin-bottom: 5px;
        }

        input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        button {
            padding: 10px 15px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

### 2. ARIA для сложных интерактивных элементов

#### Доступная сортировка таблицы с ARIA

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная сортировка таблицы</title>
</head>
<body>
    <main>
        <h1>Данные пользователей</h1>

        <table role="grid" aria-label="Таблица пользователей с возможностью сортировки">
            <thead>
                <tr role="row">
                    <th role="columnheader" 
                        id="name-header"
                        aria-sort="ascending"
                        tabindex="0"
                        onclick="sortTable('name', 'ascending')"
                        onkeydown="handleHeaderKeydown(event, 'name')">
                        Имя
                        <span class="sort-indicator" aria-hidden="true">↑</span>
                    </th>
                    <th role="columnheader" 
                        id="email-header"
                        aria-sort="none"
                        tabindex="0"
                        onclick="sortTable('email', 'none')"
                        onkeydown="handleHeaderKeydown(event, 'email')">
                        Email
                        <span class="sort-indicator" aria-hidden="true"></span>
                    </th>
                    <th role="columnheader" 
                        id="date-header"
                        aria-sort="none"
                        tabindex="0"
                        onclick="sortTable('date', 'none')"
                        onkeydown="handleHeaderKeydown(event, 'date')">
                        Дата регистрации
                        <span class="sort-indicator" aria-hidden="true"></span>
                    </th>
                </tr>
            </thead>
            <tbody id="table-body" role="rowgroup">
                <tr role="row">
                    <td role="gridcell">Иван Иванов</td>
                    <td role="gridcell">ivan@example.com</td>
                    <td role="gridcell"><time datetime="2023-01-15">15.01.2023</time></td>
                </tr>
                <tr role="row">
                    <td role="gridcell">Мария Петрова</td>
                    <td role="gridcell">maria@example.com</td>
                    <td role="gridcell"><time datetime="2023-02-20">20.02.2023</time></td>
                </tr>
                <tr role="row">
                    <td role="gridcell">Алексей Сидоров</td>
                    <td role="gridcell">alexey@example.com</td>
                    <td role="gridcell"><time datetime="2023-03-10">10.03.2023</time></td>
                </tr>
            </tbody>
        </table>

        <!-- Объявление сортировки -->
        <div id="sort-announcer" 
             aria-live="polite" 
             aria-atomic="true" 
             class="sr-only">
        </div>
    </main>

    <script>
        class AccessibleTableSorter {
            constructor() {
                this.data = [
                    { name: 'Иван Иванов', email: 'ivan@example.com', date: '2023-01-15' },
                    { name: 'Мария Петрова', email: 'maria@example.com', date: '2023-02-20' },
                    { name: 'Алексей Сидоров', email: 'alexey@example.com', date: '2023-03-10' }
                ];
                
                this.currentSort = { column: 'name', direction: 'ascending' };
                this.sortIndicators = {
                    'ascending': '↑',
                    'descending': '↓',
                    'none': ''
                };
            }

            sort(column) {
                // Определяем направление сортировки
                let direction;
                if (this.currentSort.column === column) {
                    // Переключаем направление
                    direction = this.currentSort.direction === 'ascending' ? 'descending' : 'ascending';
                } else {
                    // Новая сортировка - по возрастанию
                    direction = 'ascending';
                }

                // Обновляем состояние сортировки
                this.currentSort = { column, direction };

                // Сортируем данные
                this.data.sort((a, b) => {
                    let aValue = a[column];
                    let bValue = b[column];

                    // Для дат преобразуем в формат, пригодный для сравнения
                    if (column === 'date') {
                        aValue = new Date(aValue);
                        bValue = new Date(bValue);
                    }

                    let result = 0;
                    if (aValue < bValue) result = -1;
                    if (aValue > bValue) result = 1;

                    return direction === 'ascending' ? result : -result;
                });

                // Обновляем интерфейс
                this.updateUI();

                // Объявляем сортировку для скринридера
                this.announceSort(column, direction);
            }

            updateUI() {
                // Обновляем индикаторы сортировки
                document.querySelectorAll('[role="columnheader"]').forEach(header => {
                    header.setAttribute('aria-sort', 'none');
                    const indicator = header.querySelector('.sort-indicator');
                    if (indicator) indicator.textContent = '';
                });

                // Устанавливаем текущую сортировку
                if (this.currentSort.column) {
                    const header = document.getElementById(`${this.currentSort.column}-header`);
                    if (header) {
                        header.setAttribute('aria-sort', this.currentSort.direction);
                        const indicator = header.querySelector('.sort-indicator');
                        if (indicator) {
                            indicator.textContent = this.sortIndicators[this.currentSort.direction];
                        }
                    }
                }

                // Обновляем тело таблицы
                const tbody = document.getElementById('table-body');
                tbody.innerHTML = '';

                this.data.forEach(item => {
                    const row = document.createElement('tr');
                    row.setAttribute('role', 'row');

                    const nameCell = document.createElement('td');
                    nameCell.setAttribute('role', 'gridcell');
                    nameCell.textContent = item.name;

                    const emailCell = document.createElement('td');
                    emailCell.setAttribute('role', 'gridcell');
                    emailCell.textContent = item.email;

                    const dateCell = document.createElement('td');
                    dateCell.setAttribute('role', 'gridcell');
                    const date = document.createElement('time');
                    date.setAttribute('datetime', item.date);
                    date.textContent = new Date(item.date).toLocaleDateString('ru-RU');
                    dateCell.appendChild(date);

                    row.appendChild(nameCell);
                    row.appendChild(emailCell);
                    row.appendChild(dateCell);

                    tbody.appendChild(row);
                });
            }

            announceSort(column, direction) {
                const columnNames = {
                    'name': 'имени',
                    'email': 'email',
                    'date': 'дате регистрации'
                };

                const directionText = {
                    'ascending': 'по возрастанию',
                    'descending': 'по убыванию'
                };

                const announcement = document.getElementById('sort-announcer');
                announcement.textContent = `Таблица отсортирована по ${columnNames[column] || column} ${directionText[direction] || ''}`;
            }
        }

        // Глобальный экземпляр
        const tableSorter = new AccessibleTableSorter();

        function sortTable(column, currentDirection) {
            tableSorter.sort(column);
        }

        function handleHeaderKeydown(event, column) {
            if (event.key === 'Enter' || event.key === ' ') {
                event.preventDefault();
                sortTable(column);
            }
        }
    </script>

    <style>
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }

        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }

        th {
            background-color: #f2f2f2;
            font-weight: bold;
            position: relative;
            cursor: pointer;
            user-select: none;
        }

        th:hover {
            background-color: #e8e8e8;
        }

        th:focus {
            outline: 2px solid #007acc;
            outline-offset: -2px;
        }

        .sort-indicator {
            margin-left: 5px;
            font-size: 0.8em;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

### 3. ARIA для динамических уведомлений и диалогов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамические уведомления с ARIA</title>
</head>
<body>
    <main>
        <h1>Система уведомлений</h1>
        
        <button onclick="showAlert()">Показать предупреждение</button>
        <button onclick="showError()">Показать ошибку</button>
        <button onclick="showSuccess()">Показать успех</button>
        <button onclick="showModal()">Показать модальное окно</button>
    </main>

    <!-- Контейнер для уведомлений -->
    <div id="notifications" 
         role="region" 
         aria-live="polite" 
         aria-label="Системные уведомления"
         aria-relevant="additions">
    </div>

    <!-- Контейнер для ассертивных уведомлений -->
    <div id="assertive-notifications" 
         role="alert" 
         aria-live="assertive" 
         aria-atomic="true">
    </div>

    <script>
        class AccessibleNotificationSystem {
            constructor() {
                this.notificationsContainer = document.getElementById('notifications');
                this.assertiveContainer = document.getElementById('assertive-notifications');
            }

            showAlert(message, duration = 5000) {
                this.createNotification(message, 'alert', duration);
            }

            showError(message, duration = 7000) {
                this.createNotification(message, 'error', duration);
            }

            showSuccess(message, duration = 3000) {
                this.createNotification(message, 'success', duration);
            }

            createNotification(message, type, duration) {
                const notification = document.createElement('div');
                notification.className = `notification notification-${type}`;
                notification.setAttribute('role', 'status');
                notification.setAttribute('aria-live', 'polite');
                notification.setAttribute('aria-atomic', 'true');
                
                notification.innerHTML = `
                    <div class="notification-content">
                        <span class="notification-message">${message}</span>
                        <button class="notification-close" 
                                aria-label="Закрыть уведомление" 
                                onclick="notificationSystem.closeNotification(this)">
                            ×
                        </button>
                    </div>
                `;

                this.notificationsContainer.appendChild(notification);

                // Автоматическое закрытие
                if (duration > 0) {
                    setTimeout(() => {
                        this.closeNotification(notification.querySelector('.notification-close'));
                    }, duration);
                }

                // Объявление для скринридеров
                this.announceToScreenReader(`${type === 'success' ? 'Успешно' : type === 'error' ? 'Ошибка' : 'Предупреждение'}: ${message}`);
            }

            closeNotification(closeButton) {
                const notification = closeButton.parentElement.parentElement;
                notification.style.opacity = '0';
                notification.style.transform = 'translateX(100%)';
                
                setTimeout(() => {
                    if (notification.parentNode) {
                        notification.parentNode.removeChild(notification);
                    }
                }, 300);
            }

            showAssertion(message) {
                // Ассертивное уведомление - прерывает текущую речь
                this.assertiveContainer.textContent = message;
                
                // Очистка через 5 секунд
                setTimeout(() => {
                    if (this.assertiveContainer.textContent === message) {
                        this.assertiveContainer.textContent = '';
                    }
                }, 5000);
            }

            announceToScreenReader(message) {
                // Создание временного элемента для объявления
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 3000);
            }

            showModal(title, content) {
                // Создание модального окна с правильной ARIA разметкой
                const modal = document.createElement('div');
                modal.className = 'modal-overlay';
                modal.setAttribute('role', 'dialog');
                modal.setAttribute('aria-modal', 'true');
                modal.setAttribute('aria-labelledby', 'modal-title');
                modal.setAttribute('aria-describedby', 'modal-content');
                
                modal.innerHTML = `
                    <div class="modal-content" role="document">
                        <div class="modal-header">
                            <h2 id="modal-title">${title}</h2>
                            <button class="modal-close" 
                                    aria-label="Закрыть модальное окно" 
                                    onclick="notificationSystem.closeModal(this)">
                                ×
                            </button>
                        </div>
                        <div id="modal-content" class="modal-body">
                            ${content}
                        </div>
                        <div class="modal-footer">
                            <button onclick="notificationSystem.closeModal(this)">Закрыть</button>
                        </div>
                    </div>
                `;

                document.body.appendChild(modal);

                // Управление фокусом
                const firstFocusable = modal.querySelector('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
                if (firstFocusable) {
                    firstFocusable.focus();
                }

                // Ловушка фокуса
                this.trapFocus(modal);

                // Сохранение предыдущего активного элемента
                this.previousActiveElement = document.activeElement;
            }

            closeModal(closeButton) {
                const modal = closeButton.closest('.modal-overlay');
                if (modal) {
                    document.body.removeChild(modal);
                    
                    // Возврат фокуса на предыдущий элемент
                    if (this.previousActiveElement) {
                        this.previousActiveElement.focus();
                    }
                }
            }

            trapFocus(modal) {
                const focusableElements = modal.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );

                if (focusableElements.length === 0) return;

                const firstElement = focusableElements[0];
                const lastElement = focusableElements[focusableElements.length - 1];

                modal.addEventListener('keydown', (e) => {
                    if (e.key === 'Tab') {
                        if (e.shiftKey) {
                            if (document.activeElement === firstElement) {
                                lastElement.focus();
                                e.preventDefault();
                            }
                        } else {
                            if (document.activeElement === lastElement) {
                                firstElement.focus();
                                e.preventDefault();
                            }
                        }
                    } else if (e.key === 'Escape') {
                        this.closeModal(modal.querySelector('.modal-close'));
                    }
                });
            }
        }

        // Глобальный экземпляр
        const notificationSystem = new AccessibleNotificationSystem();

        function showAlert() {
            notificationSystem.showAlert('Это важное предупреждение, на которое стоит обратить внимание');
        }

        function showError() {
            notificationSystem.showError('Произошла ошибка при обработке запроса. Пожалуйста, попробуйте снова.');
        }

        function showSuccess() {
            notificationSystem.showSuccess('Операция выполнена успешно!');
        }

        function showModal() {
            notificationSystem.showModal(
                'Подтверждение действия', 
                '<p>Вы уверены, что хотите выполнить это действие? Это действие нельзя будет отменить.</p>'
            );
        }
    </script>

    <style>
        .notification {
            position: relative;
            margin: 10px 0;
            padding: 15px;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            animation: slideIn 0.3s ease-out;
        }

        @keyframes slideIn {
            from { right: -100%; opacity: 0; }
            to { right: 0; opacity: 1; }
        }

        .notification-alert {
            background-color: #fff3cd;
            border-left: 4px solid #ffc107;
            color: #856404;
        }

        .notification-error {
            background-color: #f8d7da;
            border-left: 4px solid #dc3545;
            color: #721c24;
        }

        .notification-success {
            background-color: #d4edda;
            border-left: 4px solid #28a745;
            color: #155724;
        }

        .notification-content {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 100%;
        }

        .notification-close {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            padding: 0;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .notification-close:focus {
            outline: 2px solid #007acc;
        }

        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .modal-content {
            background-color: white;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
            overflow: hidden;
        }

        .modal-header {
            padding: 20px;
            border-bottom: 1px solid #eee;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .modal-header h2 {
            margin: 0;
        }

        .modal-header .modal-close {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            padding: 0;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .modal-body {
            padding: 20px;
        }

        .modal-footer {
            padding: 20px;
            border-top: 1px solid #eee;
            text-align: right;
        }

        .modal-footer button {
            padding: 8px 16px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .sr-only {
            position: absolute;
            width: 1px;
            height: 1px;
            padding: 0;
            margin: -1px;
            overflow: hidden;
            clip: rect(0, 0, 0, 0);
            white-space: nowrap;
            border: 0;
        }
    </style>
</body>
</html>
```

## Следующие темы
- [[HTML/Доступность/Основы]]
- [[HTML/Формы/Доступность]]
- [[HTML/Семантика]]
- [[HTML/Доступность/Стандарты WCAG]]
- [[HTML/Доступность/Тестирование]]
- [[HTML/Доступность/Промышленные примеры]]

## Теги
#html #accessibility #aria #web-development #inclusive-design #wcag #frontend #web-components #websockets #indexeddb #accessibility-testing #best-practices #aria-roles #aria-states #aria-properties #aria-patterns #single-page-applications #spa-accessibility #dynamic-content #live-regions #assertive-aria #aria-sort #aria-grid #aria-dialog #aria-alert #aria-status #aria-atomic #aria-busy #aria-relevant #aria-describedby #aria-labelledby #aria-controls #aria-owns #aria-flowto #aria-activedescendant #aria-haspopup #aria-expanded #aria-autocomplete #aria-required #aria-invalid #aria-readonly #aria-disabled #aria-selected #aria-checked #aria-pressed #aria-level #aria-posinset #aria-setsize #aria-multiselectable #aria-colindex #aria-rowindex #aria-colspan #aria-rowspan #aria-sort #aria-valuemax #aria-valuemin #aria-valuenow #aria-valuetext