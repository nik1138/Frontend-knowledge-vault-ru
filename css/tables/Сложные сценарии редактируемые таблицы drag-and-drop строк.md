---
aliases: ["Редактируемые таблицы", "Drag-and-drop строки таблиц", "Интерактивные таблицы"]
tags: [css/tables, javascript, html/tables, ui-patterns]
---

# Сложные сценарии: редактируемые таблицы, drag-and-drop строк

## Введение

Современные веб-приложения часто требуют сложных интерактивных таблиц с возможностью редактирования данных и перетаскивания строк. Эти функции значительно улучшают пользовательский опыт, позволяя эффективно управлять табличными данными.

## Редактируемые таблицы

### Основы редактируемых таблиц

Редактируемые таблицы позволяют пользователям изменять содержимое ячеек непосредственно в таблице без необходимости открывать отдельную форму редактирования.

```html
<table class="editable-table">
  <thead>
    <tr>
      <th>Имя</th>
      <th>Фамилия</th>
      <th>Email</th>
      <th>Должность</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td contenteditable="true">Иван</td>
      <td contenteditable="true">Иванов</td>
      <td contenteditable="true">ivan@example.com</td>
      <td contenteditable="true">Менеджер</td>
    </tr>
    <tr>
      <td contenteditable="true">Мария</td>
      <td contenteditable="true">Петрова</td>
      <td contenteditable="true">maria@example.com</td>
      <td contenteditable="true">Разработчик</td>
    </tr>
  </tbody>
</table>
```

### Стилизация редактируемых ячеек

```css
.editable-table {
  border-collapse: collapse;
  width: 100%;
  margin: 20px 0;
}

.editable-table th,
.editable-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

.editable-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.editable-table td[contenteditable="true"] {
  cursor: text;
  transition: background-color 0.2s;
}

.editable-table td[contenteditable="true"]:focus {
  outline: 2px solid #4a90e2;
  outline-offset: -2px;
  background-color: #f0f8ff;
}

.editable-table td[contenteditable="true"]:hover:not(:focus) {
  background-color: #f9f9f9;
}
```

### Управление редактированием с помощью JavaScript

```javascript
// Обработчик событий для редактируемых ячеек
document.querySelectorAll('.editable-table td[contenteditable]').forEach(cell => {
  cell.addEventListener('focus', function() {
    // Сохраняем оригинальное значение при фокусе
    this.setAttribute('data-previous-value', this.textContent);
  });

  cell.addEventListener('blur', function() {
    const previousValue = this.getAttribute('data-previous-value');
    const currentValue = this.textContent;

    // Проверяем, изменилось ли значение
    if (previousValue !== currentValue) {
      // Вызываем пользовательское событие для обработки изменений
      const event = new CustomEvent('cellEdit', {
        detail: {
          oldValue: previousValue,
          newValue: currentValue,
          cell: this
        }
      });
      this.dispatchEvent(event);
    }
  });

  // Ограничение ввода для определенных столбцов
  cell.addEventListener('input', function() {
    const columnIndex = Array.from(this.parentNode.children).indexOf(this);
    
    // Пример: ограничение ввода только для столбца email
    if (columnIndex === 2) { // Предполагаем, что email в 3-м столбце
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(this.textContent) && this.textContent.trim() !== '') {
        this.style.backgroundColor = '#ffe6e6';
      } else {
        this.style.backgroundColor = '';
      }
    }
  });
});

// Обработчик пользовательского события
document.addEventListener('cellEdit', function(e) {
  const { oldValue, newValue, cell } = e.detail;
  console.log(`Ячейка изменена: ${oldValue} -> ${newValue}`);
  
  // Здесь можно добавить логику сохранения данных на сервер
  saveCellData(cell, newValue);
});
```

### Продвинутые редактируемые таблицы

Для более сложных сценариев можно использовать встроенные элементы формы:

```html
<table class="advanced-editable-table">
  <thead>
    <tr>
      <th>Имя</th>
      <th>Фамилия</th>
      <th>Статус</th>
      <th>Действия</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><input type="text" value="Иван" class="edit-input"></td>
      <td><input type="text" value="Иванов" class="edit-input"></td>
      <td>
        <select class="edit-select">
          <option value="active" selected>Активный</option>
          <option value="inactive">Неактивный</option>
          <option value="pending">Ожидает</option>
        </select>
      </td>
      <td>
        <button class="save-btn">Сохранить</button>
        <button class="cancel-btn">Отмена</button>
      </td>
    </tr>
  </tbody>
</table>
```

```css
.advanced-editable-table {
  border-collapse: collapse;
  width: 100%;
}

.advanced-editable-table th,
.advanced-editable-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

.advanced-editable-table th {
  background-color: #f2f2f2;
}

.edit-input, .edit-select {
  width: 100%;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

.edit-input:focus, .edit-select:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 5px rgba(74, 144, 226, 0.3);
}

.save-btn, .cancel-btn {
  padding: 5px 10px;
  margin-right: 5px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

.save-btn {
  background-color: #4CAF50;
  color: white;
}

.cancel-btn {
  background-color: #f44336;
  color: white;
}
```

## Drag-and-drop строк таблиц

### Основы drag-and-drop

HTML5 предоставляет встроенные API для реализации drag-and-drop функциональности. Вот как можно реализовать перетаскивание строк таблицы:

```html
<table class="draggable-table">
  <thead>
    <tr>
      <th>Порядок</th>
      <th>Задача</th>
      <th>Статус</th>
    </tr>
  </thead>
  <tbody>
    <tr draggable="true" data-id="1">
      <td>1</td>
      <td>Разработка интерфейса</td>
      <td>В процессе</td>
    </tr>
    <tr draggable="true" data-id="2">
      <td>2</td>
      <td>Тестирование</td>
      <td>Ожидает</td>
    </tr>
    <tr draggable="true" data-id="3">
      <td>3</td>
      <td>Документация</td>
      <td>Завершено</td>
    </tr>
  </tbody>
</table>
```

```css
.draggable-table {
  border-collapse: collapse;
  width: 100%;
  margin: 20px 0;
}

.draggable-table th,
.draggable-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

.draggable-table th {
  background-color: #f2f2f2;
}

.draggable-table tr[draggable="true"] {
  cursor: move;
  user-select: none;
}

.draggable-table tr.dragging {
  opacity: 0.5;
  background-color: #e3f2fd;
}

.draggable-table tr.drag-over {
  border-top: 2px solid #4a90e2;
}
```

### JavaScript реализация drag-and-drop

```javascript
document.querySelectorAll('.draggable-table tbody tr').forEach(row => {
  row.addEventListener('dragstart', function(e) {
    // Устанавливаем данные для перетаскивания
    e.dataTransfer.setData('text/plain', this.rowIndex);
    this.classList.add('dragging');
    
    // Визуальный эффект для перетаскивания
    e.dataTransfer.effectAllowed = 'move';
  });

  row.addEventListener('dragend', function() {
    this.classList.remove('dragging');
  });

  // Обработчики для области сброса
  row.addEventListener('dragover', function(e) {
    e.preventDefault(); // Обязательно для разрешения сброса
    e.dataTransfer.dropEffect = 'move';
  });

  row.addEventListener('dragenter', function(e) {
    e.preventDefault();
    this.classList.add('drag-over');
  });

  row.addEventListener('dragleave', function() {
    this.classList.remove('drag-over');
  });

  row.addEventListener('drop', function(e) {
    e.preventDefault();
    this.classList.remove('drag-over');
    
    const draggedRowIndex = parseInt(e.dataTransfer.getData('text/plain'));
    const draggedRow = this.parentNode.children[draggedRowIndex];
    
    // Проверяем, что перетаскиваемая строка не та же самая
    if (draggedRow !== this) {
      // Вставляем перетаскиваемую строку перед текущей строкой
      this.parentNode.insertBefore(draggedRow, this);
      
      // Обновляем нумерацию
      updateRowNumbers();
      
      // Вызываем событие для обновления данных
      const dragEvent = new CustomEvent('rowReordered', {
        detail: {
          movedRow: draggedRow,
          targetRow: this
        }
      });
      document.dispatchEvent(dragEvent);
    }
  });
});

// Функция обновления нумерации строк
function updateRowNumbers() {
  const table = document.querySelector('.draggable-table tbody');
  Array.from(table.rows).forEach((row, index) => {
    row.cells[0].textContent = index + 1;
  });
}
```

### Более сложная реализация с SortableJS

Для более сложных сценариев можно использовать библиотеку SortableJS:

```html
<script src="https://cdn.jsdelivr.net/npm/sortablejs@latest/Sortable.min.js"></script>

<table class="sortable-table">
  <thead>
    <tr>
      <th>Порядок</th>
      <th>Название</th>
      <th>Описание</th>
    </tr>
  </thead>
  <tbody>
    <tr data-id="1">
      <td>1</td>
      <td>Задача 1</td>
      <td>Описание задачи 1</td>
    </tr>
    <tr data-id="2">
      <td>2</td>
      <td>Задача 2</td>
      <td>Описание задачи 2</td>
    </tr>
    <tr data-id="3">
      <td>3</td>
      <td>Задача 3</td>
      <td>Описание задачи 3</td>
    </tr>
  </tbody>
</table>
```

```javascript
// Инициализация SortableJS
const tableBody = document.querySelector('.sortable-table tbody');
const sortable = new Sortable(tableBody, {
  animation: 150,
  ghostClass: 'drag-over',
  chosenClass: 'dragging',
  dragClass: 'sortable-drag',
  onEnd: function(evt) {
    // Обновляем нумерацию после перетаскивания
    updateRowNumbers();
    
    // Отправляем обновленный порядок на сервер
    const newOrder = Array.from(tableBody.children).map(row => row.dataset.id);
    saveNewOrder(newOrder);
  }
});

// Добавляем стили для SortableJS
const style = document.createElement('style');
style.textContent = `
  .sortable-drag {
    opacity: 0.5;
    background-color: #e3f2fd;
  }
`;
document.head.appendChild(style);
```

## Сочетание редактируемых таблиц и drag-and-drop

Для создания таблицы с обоими функциями:

```html
<table class="combined-table">
  <thead>
    <tr>
      <th>Порядок</th>
      <th>Название</th>
      <th>Описание</th>
      <th>Действия</th>
    </tr>
  </thead>
  <tbody>
    <tr draggable="true" data-id="1">
      <td>1</td>
      <td contenteditable="true">Задача 1</td>
      <td contenteditable="true">Описание задачи 1</td>
      <td>
        <button class="delete-btn">Удалить</button>
      </td>
    </tr>
    <tr draggable="true" data-id="2">
      <td>2</td>
      <td contenteditable="true">Задача 2</td>
      <td contenteditable="true">Описание задачи 2</td>
      <td>
        <button class="delete-btn">Удалить</button>
      </td>
    </tr>
  </tbody>
</table>
```

```javascript
// Комбинируем обе функции
document.querySelectorAll('.combined-table tbody tr').forEach(row => {
  // Обработка drag-and-drop
  row.addEventListener('dragstart', function(e) {
    e.dataTransfer.setData('text/plain', this.rowIndex);
    this.classList.add('dragging');
  });

  row.addEventListener('dragend', function() {
    this.classList.remove('dragging');
  });

  row.addEventListener('dragover', function(e) {
    e.preventDefault();
  });

  row.addEventListener('drop', function(e) {
    e.preventDefault();
    const draggedRowIndex = parseInt(e.dataTransfer.getData('text/plain'));
    const draggedRow = this.parentNode.children[draggedRowIndex];
    
    if (draggedRow !== this) {
      this.parentNode.insertBefore(draggedRow, this);
      updateRowNumbers();
    }
  });

  // Обработка редактирования
  row.querySelectorAll('td[contenteditable]').forEach(cell => {
    cell.addEventListener('focus', function() {
      this.setAttribute('data-previous-value', this.textContent);
    });

    cell.addEventListener('blur', function() {
      const previousValue = this.getAttribute('data-previous-value');
      const currentValue = this.textContent;

      if (previousValue !== currentValue) {
        console.log(`Ячейка изменена: ${previousValue} -> ${currentValue}`);
      }
    });
  });

  // Обработка кнопки удаления
  row.querySelector('.delete-btn').addEventListener('click', function() {
    if (confirm('Вы уверены, что хотите удалить эту строку?')) {
      row.remove();
      updateRowNumbers();
    }
  });
});
```

## Доступность и семантика

### ARIA атрибуты для редактируемых таблиц

```html
<table 
  class="editable-table"
  role="grid"
  aria-label="Таблица сотрудников с возможностью редактирования">
  <thead>
    <tr role="row">
      <th role="columnheader" scope="col">Имя</th>
      <th role="columnheader" scope="col">Фамилия</th>
      <th role="columnheader" scope="col">Email</th>
    </tr>
  </thead>
  <tbody>
    <tr role="row">
      <td role="gridcell" contenteditable="true" tabindex="0">Иван</td>
      <td role="gridcell" contenteditable="true" tabindex="0">Иванов</td>
      <td role="gridcell" contenteditable="true" tabindex="0">ivan@example.com</td>
    </tr>
  </tbody>
</table>
```

### ARIA атрибуты для drag-and-drop

```html
<tr 
  draggable="true" 
  role="row" 
  aria-grabbed="false"
  tabindex="0"
  data-id="1">
  <td role="gridcell">1</td>
  <td role="gridcell" contenteditable="true">Задача 1</td>
</tr>
```

```javascript
// Обновляем ARIA атрибуты во время drag-and-drop
row.addEventListener('dragstart', function(e) {
  this.setAttribute('aria-grabbed', 'true');
});

row.addEventListener('dragend', function() {
  this.setAttribute('aria-grabbed', 'false');
});
```

## Производительность и оптимизация

### Оптимизация больших таблиц

Для больших таблиц с редактируемыми ячейками и drag-and-drop:

```javascript
// Используем делегирование событий для производительности
document.querySelector('.large-table tbody').addEventListener('focus', function(e) {
  if (e.target.hasAttribute('contenteditable')) {
    e.target.setAttribute('data-previous-value', e.target.textContent);
  }
}, true);

document.querySelector('.large-table tbody').addEventListener('blur', function(e) {
  if (e.target.hasAttribute('contenteditable')) {
    const previousValue = e.target.getAttribute('data-previous-value');
    const currentValue = e.target.textContent;

    if (previousValue !== currentValue) {
      // Обработка изменений
    }
  }
}, true);
```

### Использование requestAnimationFrame

```javascript
let isUpdating = false;

function updateRowNumbers() {
  if (!isUpdating) {
    isUpdating = true;
    requestAnimationFrame(() => {
      const table = document.querySelector('.sortable-table tbody');
      Array.from(table.rows).forEach((row, index) => {
        row.cells[0].textContent = index + 1;
      });
      isUpdating = false;
    });
  }
}
```

## Заключение

Реализация редактируемых таблиц и drag-and-drop строк требует сочетания HTML, CSS и JavaScript. Важно учитывать:

1. Доступность и семантику
2. Производительность при работе с большими объемами данных
3. Обратную совместимость с различными браузерами
4. Обработку ошибок и валидацию данных
5. Сохранение изменений на сервере

Сочетание этих возможностей позволяет создавать мощные интерактивные интерфейсы для управления табличными данными.

## См. также

- [[Продвинутые возможности таблиц: colspan, rowspan, sticky заголовки]]
- [[Таблицы в разных браузерах: особенности и совместимость]]
- [[Адаптивные таблицы: техники и лучшие практики]]
- [[CSS Grid: глубокое погружение]]