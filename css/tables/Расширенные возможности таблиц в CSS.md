---
aliases: ["Таблицы в CSS", "Расширенные таблицы", "CSS таблицы"]
tags: [css, tables, layout, advanced]
---

# Расширенные возможности таблиц в CSS

## Введение

CSS предоставляет мощные инструменты для стилизации HTML-таблиц, позволяя создавать сложные и функциональные табличные представления данных. В этом руководстве мы рассмотрим расширенные возможности стилизации таблиц, включая управление границами, выравнивание, прокрутку и адаптивность.

## Основные элементы таблицы

Для начала вспомним основные HTML-элементы таблицы:

- `table` — контейнер всей таблицы
- `thead` — заголовок таблицы
- `tbody` — тело таблицы
- `tfoot` — подвал таблицы
- `tr` — строка таблицы
- `th` — ячейка заголовка
- `td` — ячейка данных

## Свойства для стилизации таблиц

### border-collapse

Свойство `border-collapse` определяет, как границы таблицы и ячеек взаимодействуют друг с другом:

```css
table {
  border-collapse: separate; /* Границы ячеек отдельны */
  border-spacing: 0; /* Расстояние между границами ячеек */
}

table.collapsed-borders {
  border-collapse: collapse; /* Границы объединяются */
}
```

### empty-cells

Свойство `empty-cells` управляет отображением границ и фона пустых ячеек:

```css
table {
  empty-cells: show; /* Показывать границы пустых ячеек */
}

table.hide-empty {
  empty-cells: hide; /* Не показывать границы пустых ячеек */
}
```

### table-layout

Свойство `table-layout` определяет, как вычисляются ширины столбцов таблицы:

```css
table {
  table-layout: auto; /* Ширина столбцов определяется содержимым */
}

table.fixed {
  table-layout: fixed; /* Ширина столбцов определяется шириной таблицы и столбцов */
}
```

## Стилизация заголовков таблицы

Для стилизации заголовков таблицы можно использовать элементы `thead` и псевдоклассы:

```css
table thead th {
  background-color: #f2f2f2;
  font-weight: bold;
  text-align: left;
  padding: 10px;
  border-bottom: 2px solid #ddd;
}

table thead tr:first-child th {
  border-top: 2px solid #bbb;
}
```

## Зебра-эффект (чередование строк)

Для создания чередующихся цветов строк используем псевдоклассы:

```css
table tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

table tbody tr:hover {
  background-color: #f5f5f5;
}
```

## Фиксированные заголовки таблицы

Для создания таблиц с фиксированными заголовками и прокручиваемым телом:

```css
.scrollable-table-container {
  height: 300px;
  overflow-y: auto;
}

.scrollable-table-container table {
  width: 100%;
  border-collapse: collapse;
}

.scrollable-table-container thead th {
  position: sticky;
  top: 0;
  background-color: #fff;
  z-index: 10;
}
```

## Адаптивные таблицы

Для создания адаптивных таблиц можно использовать несколько подходов:

### Горизонтальная прокрутка

```css
.responsive-table {
  overflow-x: auto;
  width: 100%;
}

.responsive-table table {
  min-width: 600px; /* Минимальная ширина, при которой появляется прокрутка */
}
```

### Преобразование в карточки на мобильных устройствах

```css
@media (max-width: 768px) {
  .card-table thead,
  .card-table tbody tr:first-child {
    display: none;
  }

  .card-table tbody tr {
    display: block;
    margin-bottom: 15px;
    border: 1px solid #ddd;
    padding: 10px;
  }

  .card-table tbody td {
    display: block;
    text-align: right;
    padding: 5px;
  }

  .card-table tbody td::before {
    content: attr(data-label) ": ";
    float: left;
    font-weight: bold;
  }
}
```

## Примеры расширенных таблиц

### Таблица с фиксированной шириной столбцов

```css
.fixed-columns-table {
  table-layout: fixed;
  width: 100%;
}

.fixed-columns-table .col-1 {
  width: 20%;
}

.fixed-columns-table .col-2 {
  width: 30%;
}

.fixed-columns-table .col-3 {
  width: 50%;
}
```

### Таблица с группировкой строк

```css
.grouped-rows-table tbody tr.group-start {
  border-top: 2px solid #333;
  font-weight: bold;
}

.grouped-rows-table tbody tr.group-end {
  border-bottom: 2px solid #333;
}
```

## Лучшие практики

1. **Используйте семантические элементы**: `thead`, `tbody`, `tfoot` для структурирования таблицы
2. **Обеспечьте доступность**: добавьте атрибуты `scope`, `headers`, `caption`
3. **Оптимизируйте производительность**: используйте `table-layout: fixed` для больших таблиц
4. **Поддерживайте читаемость**: обеспечьте достаточный контраст и отступы

## Связанные темы

- [[Адаптивные таблицы: техники и лучшие практики]]
- [[Стилизация таблиц с нуля: от структуры до оформления]]
- [[CSS Selectors]]
- [[CSS Layout]]