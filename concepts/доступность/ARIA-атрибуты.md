---
aliases: [ARIA, ARIA-атрибуты, доступные интерактивные элементы]
tags: [доступность, a11y, html, javascript]
---

# ARIA-атрибуты

## Обзор

ARIA (Accessible Rich Internet Applications) - это набор атрибутов, которые помогают сделать веб-контент и веб-приложения более доступными для людей с ограниченными возможностями. ARIA-атрибуты обеспечивают дополнительную информацию вспомогательным технологиям, таким как скринридеры, позволяя им лучше понимать структуру и функциональность интерфейса.

## Основные категории ARIA-атрибутов

### 1. Роли (Roles)
Определяют тип элемента и его функцию в интерфейсе.

#### Структурные роли
- `banner` - заголовок страницы
- `main` - основное содержимое
- `navigation` - навигационная область
- `complementary` - дополнительное содержимое
- `contentinfo` - нижний колонтитул

#### Виджет-роли
- `button` - кнопка
- `checkbox` - чекбокс
- `dialog` - диалоговое окно
- `menu` - меню
- `tab` - вкладка
- `tabpanel` - панель вкладки
- `tooltip` - всплывающая подсказка

```html
<!-- Пример использования структурных ролей -->
<header role="banner">
  <h1>Мой веб-сайт</h1>
</header>

<nav role="navigation" aria-label="Основная навигация">
  <ul>
    <li><a href="/">Главная</a></li>
    <li><a href="/about">О нас</a></li>
  </ul>
</nav>

<main role="main">
  <p>Основное содержимое страницы</p>
</main>

<footer role="contentinfo">
  <p>&copy; 2023 Мой сайт. Все права защищены.</p>
</footer>
```

### 2. Свойства (Properties)
Описывают состояние элемента или его характеристики.

- `aria-label` - предоставляет доступное имя элементу
- `aria-labelledby` - связывает элемент с другим элементом, который служит его меткой
- `aria-describedby` - указывает на элемент, который описывает текущий элемент
- `aria-details` - указывает на дополнительную информацию об элементе

```html
<!-- Пример использования свойств -->
<button aria-label="Закрыть модальное окно">✕</button>

<div id="tooltip1">Дополнительная информация</div>
<input type="text" aria-describedby="tooltip1" placeholder="Введите текст">

<h2 id="label1">Форма регистрации</h2>
<form aria-labelledby="label1">
  <input type="text" placeholder="Имя">
</form>
```

### 3. Состояния (States)
Описывают текущее состояние элемента, которое может изменяться программно.

- `aria-checked` - состояние чекбокса или радио-кнопки
- `aria-expanded` - состояние расширения/свертывания
- `aria-hidden` - видимость для вспомогательных технологий
- `aria-selected` - состояние выбора элемента
- `aria-pressed` - состояние нажатия кнопки (для toggle-кнопок)

```html
<!-- Пример использования состояний -->
<div role="button" 
     aria-pressed="false" 
     onclick="togglePressed(this)">
  Нажми меня
</div>

<script>
function togglePressed(element) {
  const isPressed = element.getAttribute('aria-pressed') === 'true';
  element.setAttribute('aria-pressed', !isPressed);
}
</script>
```

## Практические рекомендации по использованию ARIA

### 1. Используйте семантический HTML, когда это возможно

> [!tip] Совет
> Всегда используйте нативные HTML-элементы, если они подходят для задачи, вместо добавления ARIA-ролей к элементам, не предназначенным для этой роли.

```html
<!-- Правильно: используем нативную кнопку -->
<button type="button" onclick="doSomething()">Нажми меня</button>

<!-- Неправильно: добавляем роль к div -->
<div role="button" onclick="doSomething()" tabindex="0">Нажми меня</div>
```

### 2. Не перегружайте интерфейс ARIA-атрибутами

Избегайте избыточного использования ARIA-атрибутов, особенно когда нативный HTML уже обеспечивает нужную семантику.

```html
<!-- Избыточно -->
<h1 role="heading" aria-level="1">Заголовок</h1>

<!-- Правильно -->
<h1>Заголовок</h1>
```

### 3. Обеспечьте правильное взаимодействие с клавиатурой

Когда вы используете ARIA-роли для интерактивных элементов, обязательно обеспечьте возможность управления с клавиатуры.

```html
<!-- Правильно: добавляем tabindex и обработчики клавиш -->
<div role="button" 
     tabindex="0" 
     onclick="activate()"
     onkeydown="if(event.key === 'Enter' || event.key === ' ') activate()">
  Интерактивный элемент
</div>
```

## Распространенные паттерны ARIA

### 1. Аккордеон

```html
<div class="accordion">
  <div class="accordion-item">
    <h3>
      <button role="button" 
              aria-expanded="false" 
              aria-controls="panel1"
              onclick="toggleAccordion(this)">
        Заголовок аккордеона
      </button>
    </h3>
    <div id="panel1" role="region" aria-labelledby="accordion-button" hidden>
      <p>Содержимое аккордеона...</p>
    </div>
  </div>
</div>

<script>
function toggleAccordion(button) {
  const isExpanded = button.getAttribute('aria-expanded') === 'true';
  button.setAttribute('aria-expanded', !isExpanded);
  
  const panel = document.getElementById(button.getAttribute('aria-controls'));
  panel.hidden = !panel.hidden;
}
</script>
```

### 2. Вкладки

```html
<div role="tablist" aria-label="Навигация по вкладкам">
  <button role="tab" 
          aria-selected="true" 
          aria-controls="panel-1"
          id="tab-1"
          onclick="switchTab(this, 'panel-1')">
    Вкладка 1
  </button>
  <button role="tab" 
          aria-selected="false" 
          aria-controls="panel-2"
          id="tab-2"
          onclick="switchTab(this, 'panel-2')">
    Вкладка 2
  </button>
</div>

<div role="tabpanel" 
     id="panel-1" 
     aria-labelledby="tab-1">
  Содержимое первой вкладки
</div>
<div role="tabpanel" 
     id="panel-2" 
     aria-labelledby="tab-2" 
     hidden>
  Содержимое второй вкладки
</div>

<script>
function switchTab(tab, panelId) {
  // Сбрасываем все вкладки
  document.querySelectorAll('[role="tab"]').forEach(t => {
    t.setAttribute('aria-selected', 'false');
  });
  document.querySelectorAll('[role="tabpanel"]').forEach(p => {
    p.hidden = true;
  });
  
  // Активируем выбранную вкладку
  tab.setAttribute('aria-selected', 'true');
  document.getElementById(panelId).hidden = false;
}
</script>
```

### 3. Модальное окно

```html
<button onclick="openModal()">Открыть модальное окно</button>

<div id="modal" 
     class="modal" 
     role="dialog" 
     aria-modal="true" 
     aria-labelledby="modal-title"
     hidden>
  <div class="modal-content">
    <h2 id="modal-title">Заголовок модального окна</h2>
    <p>Содержимое модального окна</p>
    <button onclick="closeModal()">Закрыть</button>
  </div>
</div>

<script>
function openModal() {
  const modal = document.getElementById('modal');
  modal.hidden = false;
  
  // Сохраняем текущий элемент фокуса
  const currentFocus = document.activeElement;
  
  // Устанавливаем фокус на модальное окно
  modal.focus();
  
  // Захватываем фокус внутри модального окна
  modal.addEventListener('keydown', trapFocus);
}

function closeModal() {
  const modal = document.getElementById('modal');
  modal.hidden = true;
  
  // Возвращаем фокус на элемент, который был активен до открытия модального окна
  document.activeElement.blur();
}

function trapFocus(event) {
  const modal = document.getElementById('modal');
  const focusableElements = modal.querySelectorAll('button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])');
  const firstElement = focusableElements[0];
  const lastElement = focusableElements[focusableElements.length - 1];
  
  if (event.key === 'Tab') {
    if (event.shiftKey && document.activeElement === firstElement) {
      event.preventDefault();
      lastElement.focus();
    } else if (!event.shiftKey && document.activeElement === lastElement) {
      event.preventDefault();
      firstElement.focus();
    }
  }
}
</script>
```

## Ошибки и лучшие практики

### 1. Избегайте конфликтов между ARIA и HTML

```html
<!-- Неправильно: противоречие между ARIA и HTML -->
<button role="checkbox" aria-checked="true">Нажми меня</button>

<!-- Правильно: используем правильный элемент -->
<input type="checkbox" id="myCheckbox" checked>
<label for="myCheckbox">Мой чекбокс</label>
```

### 2. Правильное использование aria-hidden

```html
<!-- Используем aria-hidden для скрытия элементов от вспомогательных технологий -->
<div aria-hidden="true">Содержимое видно только зрячим пользователям</div>

<!-- Важно: не используем aria-hidden="true" на фокусируемых элементах -->
<!-- Неправильно -->
<button aria-hidden="true">Скрытая кнопка</button>

<!-- Правильно - скрываем элемент и удаляем возможность фокусировки -->
<button aria-hidden="true" tabindex="-1">Скрытая кнопка</button>
```

### 3. Обновление ARIA-атрибутов динамически

```javascript
// Правильно: обновляем ARIA-атрибуты при изменении состояния
function updateProgressBar(percent) {
  const progressBar = document.getElementById('progress');
  progressBar.setAttribute('aria-valuenow', percent);
  progressBar.style.width = percent + '%';
}
```

## Заключение

ARIA-атрибуты являются мощным инструментом для повышения доступности веб-приложений, особенно при создании сложных интерактивных компонентов. Однако важно использовать их осознанно и в соответствии с установленными паттернами, чтобы обеспечить наилучший опыт для пользователей вспомогательных технологий.

См. также:
- [[Стандарты-WCAG]]
- [[Семантическая-верстка]]
- [[Доступные-компоненты]]

> [!warning] Важно
> ARIA - это инструмент последней инстанции. Сначала используйте семантический HTML, затем добавляйте ARIA только когда это действительно необходимо.