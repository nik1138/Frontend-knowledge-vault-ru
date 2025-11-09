# Паттерны CSS Grid-раскладки

CSS Grid Layout предоставляет мощный способ создания двумерных раскладок. Паттерны Grid-раскладки — это повторяющиеся решения для распространенных задач веб-дизайна, позволяющие эффективно организовывать элементы на странице.

## Основные паттерны сеток

### Головоломка (Jigsaw Pattern)

Паттерн "головоломка" создает непредсказуемую, интересную раскладку с элементами разных размеров, напоминающую кусочки пазла.

```css
.jigsaw {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(4, 200px);
  gap: 10px;
}

.jigsaw .item:nth-child(1) { grid-column: span 2; grid-row: span 2; }
.jigsaw .item:nth-child(2) { grid-column: 3 / -1; grid-row: 1 / 3; }
.jigsaw .item:nth-child(3) { grid-column: 1 / 3; grid-row: 3 / 5; }
```

### Газета (Newspaper Pattern)

Этот паттерн имитирует традиционную газетную раскладку с колонками текста и изображениями.

```css
.newspaper {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

### Ласточка (Masonry Pattern)

Хотя CSS Grid не создает настоящую мasonry-раскладку (для этого лучше использовать CSS Masonry), можно приблизиться к этому эффекту:

```css
.masonry {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 10px; /* Маленькие строки для гибкости */
}

.masonry .item {
  grid-row-end: span var(--row-span);
}
```

### Магистраль (Staircase Pattern)

Создает эффект лестницы, где каждый следующий элемент смещен относительно предыдущего:

```css
.staircase {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-auto-rows: 100px;
  gap: 10px;
}

.staircase .item:nth-child(1) { grid-column: 1; grid-row: 1; }
.staircase .item:nth-child(2) { grid-column: 2; grid-row: 2; }
.staircase .item:nth-child(3) { grid-column: 3; grid-row: 3; }
.staircase .item:nth-child(4) { grid-column: 4; grid-row: 4; }
.staircase .item:nth-child(5) { grid-column: 5; grid-row: 5; }
```

## Адаптивные Grid-паттерны

Для создания адаптивных раскладок используем функции `minmax()`, `auto-fit` и `auto-fill`:

```css
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* Адаптивная раскладка с изменением структуры */
@media (max-width: 768px) {
  .responsive-grid {
    grid-template-columns: 1fr;
  }
}
```

## Комбинация Grid и Flexbox паттернов

Grid и Flexbox дополняют друг друга. Используйте Grid для основной раскладки, а Flexbox — для выравнивания внутри ячеек:

```css
.main-layout {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-rows: auto 1fr auto;
  grid-template-columns: 250px 1fr;
  min-height: 100vh;
}

.header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
}

.main {
  grid-area: main;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.footer {
  grid-area: footer;
  display: flex;
  justify-content: center;
  align-items: center;
}
```

## Современные возможности

### Subgrid

Subgrid позволяет дочерним элементам наследовать сетку родительского элемента:

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(4, auto);
}

.child {
  display: subgrid;
  grid-column: span 2;
  grid-row: span 2;
}
```

### Container Queries в контексте Grid

Контейнерные запросы позволяют создавать адаптивные Grid-раскладки на основе размера родительского контейнера:

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 15px;
  }
}

@container (min-width: 700px) {
  .card-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

## Лучшие практики построения Grid-паттернов

1. **Используйте именованные области** для сложных макетов:

```css
.layout {
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}
```

2. **Определяйте минимальные и максимальные размеры** для адаптивности:

```css
.grid {
  grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
}
```

3. **Используйте логические свойства** для международных проектов:

```css
.grid {
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}
```

## Частые ошибки при работе с Grid-паттернами

1. **Смешивание Grid и Flexbox без необходимости** — используйте Grid для двумерных раскладок, Flexbox — для одномерных
2. **Неправильное определение размеров ячеек** — не используйте фиксированные размеры, когда возможна адаптивность
3. **Игнорирование доступности** — убедитесь, что порядок элементов в HTML логичен, даже если визуальный порядок изменен через Grid
4. **Перегруженные значения grid-template** — разделяйте свойства для лучшей читаемости

## Сравнение с другими методами раскладки

| Паттерн | Grid | Flexbox | Float |
|--------|------|---------|-------|
| Двумерная раскладка | ✅ | ❌ | ❌ |
| Адаптивность | ✅ | ✅ | ❌ |
| Выравнивание элементов | ✅ | ✅ | ❌ |
| Сложные макеты | ✅ | ⚠️ | ❌ |

Для получения дополнительной информации см. [[CSS Grid]], [[CSS Flexbox]], [[CSS Layout]] и [[Responsive Design]].

#css #grid #layout #patterns #web-development #frontend