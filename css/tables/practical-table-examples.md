---
aliases: ["Практические примеры таблиц", "Таблицы с фильтрами и сортировкой", "Примеры сложных таблиц"]
tags: [css, tables, examples, filters, sorting]
---

# Практические примеры сложных таблиц с фильтрами и сортировкой

## Введение

В современных веб-приложениях часто требуются интерактивные таблицы с возможностью фильтрации и сортировки данных. В этой статье мы рассмотрим, как стилизовать такие таблицы с помощью CSS, при этом не забывая о функциональной части, реализуемой с помощью JavaScript.

## Пример: Таблица сотрудников с сортировкой

### HTML-структура

```html
<div class="table-container">
  <div class="table-controls">
    <input type="text" id="searchInput" placeholder="Поиск по таблице...">
    <select id="departmentFilter">
      <option value="">Все отделы</option>
      <option value="IT">IT</option>
      <option value="HR">HR</option>
      <option value="Finance">Финансы</option>
    </select>
  </div>
  
  <table class="employee-table">
    <thead>
      <tr>
        <th data-sort="name">Имя <span class="sort-indicator"></span></th>
        <th data-sort="position">Должность</th>
        <th data-sort="department">Отдел</th>
        <th data-sort="salary">Зарплата</th>
        <th data-sort="date">Дата найма</th>
      </tr>
    </thead>
    <tbody>
      <!-- Данные таблицы -->
    </tbody>
  </table>
</div>
```

### CSS-стили

```css
.table-container {
  max-width: 100%;
  margin: 20px auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.table-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.table-controls input,
.table-controls select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.employee-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.employee-table th,
.employee-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.employee-table th {
  background-color: #f8f9fa;
  font-weight: 600;
  position: sticky;
  top: 0;
  z-index: 10;
}

.employee-table th:hover {
  background-color: #e9ecef;
  cursor: pointer;
}

.employee-table tbody tr {
  transition: background-color 0.2s ease;
}

.employee-table tbody tr:hover {
  background-color: #f8f9fa;
}

.employee-table tbody tr.filtered {
  display: none;
}

.employee-table .sort-indicator {
  display: inline-block;
  width: 0;
  height: 0;
  margin-left: 5px;
  vertical-align: middle;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
}

.employee-table .sort-indicator.asc {
  border-bottom: 6px solid #007bff;
}

.employee-table .sort-indicator.desc {
  border-top: 6px solid #007bff;
}

.employee-table .salary {
  text-align: right;
  font-family: 'Courier New', monospace;
}

.employee-table .date {
  white-space: nowrap;
}
```

## Пример: Финансовая таблица с фильтрами

### HTML-структура

```html
<div class="financial-table-container">
  <div class="financial-filters">
    <div class="filter-group">
      <label for="periodFilter">Период:</label>
      <select id="periodFilter">
        <option value="month">Месяц</option>
        <option value="quarter">Квартал</option>
        <option value="year">Год</option>
      </select>
    </div>
    
    <div class="filter-group">
      <label for="categoryFilter">Категория:</label>
      <select id="categoryFilter">
        <option value="">Все категории</option>
        <option value="income">Доходы</option>
        <option value="expenses">Расходы</option>
      </select>
    </div>
  </div>
  
  <table class="financial-table">
    <thead>
      <tr>
        <th>Дата</th>
        <th>Категория</th>
        <th>Описание</th>
        <th>Доход</th>
        <th>Расход</th>
        <th>Баланс</th>
      </tr>
    </thead>
    <tbody>
      <!-- Данные таблицы -->
    </tbody>
  </table>
</div>
```

### CSS-стили

```css
.financial-table-container {
  max-width: 100%;
  margin: 20px auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.financial-filters {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  align-items: end;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.filter-group label {
  font-weight: 500;
  font-size: 14px;
  color: #495057;
}

.filter-group select {
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 14px;
  background-color: #fff;
}

.financial-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.financial-table th,
.financial-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #e9ecef;
}

.financial-table th {
  background-color: #f8f9fa;
  font-weight: 600;
  position: sticky;
  top: 0;
  z-index: 10;
}

.financial-table .income {
  color: #28a745;
  font-weight: 500;
}

.financial-table .expenses {
  color: #dc3545;
  font-weight: 500;
}

.financial-table .balance-positive {
  color: #28a745;
  font-weight: 600;
}

.financial-table .balance-negative {
  color: #dc3545;
  font-weight: 600;
}

.financial-table .currency {
  text-align: right;
  font-family: 'Courier New', monospace;
}
```

## Пример: Таблица с деревом данных

### HTML-структура

```html
<table class="tree-table">
  <thead>
    <tr>
      <th>Название проекта</th>
      <th>Ответственный</th>
      <th>Статус</th>
      <th>Срок</th>
    </tr>
  </thead>
  <tbody>
    <tr data-level="0" class="parent-row">
      <td>
        <span class="expand-icon">▶</span>
        <span class="project-name">Основной проект</span>
      </td>
      <td>Иванов И.И.</td>
      <td><span class="status status-in-progress">В работе</span></td>
      <td>31.12.2023</td>
    </tr>
    <tr data-level="1" class="child-row hidden">
      <td class="sub-project">
        <span class="indent"></span>
        <span class="project-name">Подпроект 1</span>
      </td>
      <td>Петров П.П.</td>
      <td><span class="status status-completed">Завершен</span></td>
      <td>15.11.2023</td>
    </tr>
    <tr data-level="1" class="child-row hidden">
      <td class="sub-project">
        <span class="indent"></span>
        <span class="project-name">Подпроект 2</span>
      </td>
      <td>Сидоров С.С.</td>
      <td><span class="status status-pending">Ожидает</span></td>
      <td>30.11.2023</td>
    </tr>
  </tbody>
</table>
```

### CSS-стили

```css
.tree-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
  font-size: 14px;
}

.tree-table th,
.tree-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #e9ecef;
}

.tree-table th {
  background-color: #f8f9fa;
  font-weight: 600;
  position: sticky;
  top: 0;
  z-index: 10;
}

.tree-table .expand-icon {
  display: inline-block;
  margin-right: 8px;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.tree-table .expand-icon.expanded {
  transform: rotate(90deg);
}

.tree-table .project-name {
  font-weight: 500;
}

.tree-table .sub-project {
  padding-left: 40px !important;
}

.tree-table .indent {
  display: inline-block;
  width: 20px;
}

.tree-table .child-row {
  background-color: #f8f9fa;
}

.tree-table .child-row.hidden {
  display: none;
}

.status {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.status-completed {
  background-color: #d4edda;
  color: #155724;
}

.status-in-progress {
  background-color: #d1ecf1;
  color: #0c5460;
}

.status-pending {
  background-color: #fff3cd;
  color: #856404;
}
```

## Адаптивность таблиц

Для адаптации таблиц под мобильные устройства можно использовать следующие техники:

```css
@media (max-width: 768px) {
  .table-container {
    padding: 10px;
  }
  
  .table-controls {
    flex-direction: column;
    gap: 10px;
  }
  
  .financial-filters {
    flex-direction: column;
    gap: 10px;
    align-items: stretch;
  }
  
  .employee-table,
  .financial-table,
  .tree-table {
    font-size: 12px;
  }
  
  .employee-table th,
  .employee-table td,
  .financial-table th,
  .financial-table td,
  .tree-table th,
  .tree-table td {
    padding: 8px 10px;
  }
  
  /* Альтернативное представление для мобильных */
  .table-mobile {
    display: block;
  }
  
  .table-mobile thead,
  .table-mobile tfoot {
    display: none;
  }
  
  .table-mobile tbody tr {
    display: block;
    margin-bottom: 15px;
    border: 1px solid #ddd;
    border-radius: 4px;
    padding: 10px;
  }
  
  .table-mobile tbody td {
    display: block;
    text-align: right;
    border: none;
    position: relative;
    padding-left: 50%;
  }
  
  .table-mobile tbody td:before {
    content: attr(data-label) ": ";
    position: absolute;
    left: 10px;
    width: 45%;
    text-align: left;
    font-weight: bold;
  }
}
```

## Заключение

Создание сложных таблиц с фильтрами и сортировкой требует тщательной проработки как функциональной, так и визуальной части. CSS позволяет создавать эстетически привлекательные и удобные в использовании таблицы, которые улучшают пользовательский опыт.

Для лучшей доступности рекомендуется ознакомиться с [[Таблицы и доступность: лучшие практики]] и [[CSS-таблицы: продвинутые возможности стилизации]].

## Ссылки

- [MDN: CSS таблицы](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_tables)
- [CSS-Tricks: Responsive Tables](https://css-tricks.com/responsive-data-tables/)
- [A11Y Project: Tables](https://www.a11yproject.com/posts/2019-02-12-what-is-the-purpose-of-the-table-element/)