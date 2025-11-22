---
aliases: ["Стилизация таблиц", "Продвинутые таблицы CSS", "CSS таблицы продвинутый уровень"]
tags: [css, tables, styling, advanced]
---

# CSS-таблицы: продвинутые возможности стилизации

## Введение

CSS предоставляет мощные возможности для стилизации HTML-таблиц, позволяя создавать сложные и эстетически привлекательные представления данных. В этой статье мы рассмотрим продвинутые методы стилизации таблиц, которые выходят за рамки базовых свойств.

## Базовые свойства таблиц

Перед тем как перейти к продвинутым возможностям, важно понимать базовые свойства CSS, применяемые к таблицам:

```css
table {
  border-collapse: collapse; /* Объединяет границы ячеек */
  border-spacing: 0; /* Убирает расстояние между границами ячеек */
  width: 100%; /* Задает ширину таблицы */
  table-layout: fixed; /* Фиксирует ширину столбцов */
}
```

## Продвинутые методы стилизации

### 1. Адаптивные таблицы

Для создания адаптивных таблиц можно использовать несколько подходов:

#### Подход с прокруткой

```css
.responsive-table-container {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

.responsive-table {
  min-width: 800px; /* Предотвращает слишком сильное сжатие */
}
```

#### Подход с трансформацией

```css
@media (max-width: 768px) {
  .table-transform {
    transform: scale(0.8);
    transform-origin: top left;
  }
}
```

### 2. Стилизация заголовков таблиц

```css
th {
  background-color: #f2f2f2;
  position: sticky; /* Фиксирует заголовки при прокрутке */
  top: 0;
  z-index: 10;
  padding: 12px 15px;
  text-align: left;
  font-weight: bold;
  border-bottom: 2px solid #ddd;
}
```

### 3. Зебра-эффект (чередование строк)

```css
tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

tbody tr:hover {
  background-color: #f5f5f5;
  transition: background-color 0.3s ease;
}
```

### 4. Стилизация границ

```css
.table-bordered {
  border: 1px solid #ddd;
}

.table-bordered td,
.table-bordered th {
  border: 1px solid #ddd;
}

.table-bordered thead th {
  border-bottom: 2px solid #ddd;
}
```

## Сложные структуры таблиц

### Объединенные ячейки

```css
/* Горизонтальное объединение ячеек */
td[colspan] {
  text-align: center;
  font-weight: bold;
}

/* Вертикальное объединение ячеек */
td[rowspan] {
  vertical-align: middle;
}
```

### Вложенные таблицы

```css
.nested-table {
  margin: 0;
  padding: 0;
  border: none;
}

.nested-table td {
  padding: 5px;
  border: 1px solid #eee;
}
```

## Стилизация специфических ячеек

### Первые и последние ячейки

```css
/* Стилизация первой колонки */
td:first-child,
th:first-child {
  border-left: none;
  padding-left: 15px;
}

/* Стилизация последней колонки */
td:last-child,
th:last-child {
  border-right: none;
  padding-right: 15px;
}
```

### Ячейки с определенными типами данных

```css
/* Стилизация ячеек с числовыми значениями */
.numeric {
  text-align: right;
  font-family: 'Courier New', monospace;
}

/* Стилизация ячеек с процентами */
.percentage {
  color: #28a745;
}

.percentage.negative {
  color: #dc3545;
}
```

## Анимации и переходы

```css
.table-data {
  transition: all 0.3s ease;
}

.table-data.changed {
  background-color: #fff3cd;
  animation: highlight 2s ease;
}

@keyframes highlight {
  0% { background-color: #fff3cd; }
  100% { background-color: transparent; }
}
```

## Практические примеры

### Таблица с выделением активной строки

```css
.table-interactive tbody tr {
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.table-interactive tbody tr.active {
  background-color: #d4edda;
  border-left: 4px solid #28a745;
}

.table-interactive tbody tr:hover:not(.active) {
  background-color: #f8f9fa;
}
```

### Таблица с цветовой кодировкой

```css
.status-active {
  background-color: #d4edda;
  color: #155724;
}

.status-pending {
  background-color: #fff3cd;
  color: #856404;
}

.status-inactive {
  background-color: #f8d7da;
  color: #721c24;
}
```

## Заключение

Продвинутая стилизация таблиц позволяет создавать не только визуально привлекательные, но и функциональные представления данных. Правильное использование CSS-свойств позволяет улучшить читаемость и восприятие информации в таблицах, что особенно важно для пользовательского опыта.

Для дальнейшего изучения рекомендуется ознакомиться с [[CSS таблицы: практические примеры с фильтрами и сортировкой]] и [[Таблицы и доступность: лучшие практики]].

## Ссылки

- [MDN: CSS таблицы](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_tables)
- [MDN: Свойство table-layout](https://developer.mozilla.org/ru/docs/Web/CSS/table-layout)
- [CSS-Tricks: Guide to Tables](https://css-tricks.com/complete-guide-tables/)