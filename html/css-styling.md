# Стилизация HTML с помощью CSS

CSS (Cascading Style Sheets) - это язык таблиц стилей, используемый для описания внешнего вида документа, написанного на HTML. Он позволяет отделить структуру документа от его представления, что делает веб-разработку более гибкой и управляемой.

## Методы подключения CSS

Существует три основных способа подключения CSS к HTML-документу:

### 1. Встроенные стили (Inline Styles)
```html
<p style="color: blue; font-size: 16px;">Этот текст синий и размером 16 пикселей.</p>
```

### 2. Внутренние стили (Internal Stylesheet)
```html
<head>
    <style>
        body {
            background-color: lightblue;
        }
        h1 {
            color: navy;
        }
    </style>
</head>
```

### 3. Внешние стили (External Stylesheet)
```html
<head>
    <link rel="stylesheet" href="styles.css">
</head>
```

Файл `styles.css`:
```css
body {
    margin: 0;
    padding: 20px;
}
```

## Приоритеты стилей

Приоритеты CSS-правил определяются по формуле: `!important > inline > ID > class/attribute/pseudo-class > element/type > universal`. Это позволяет разработчикам управлять тем, какие стили применяются, когда есть конфликты.

> [!tip] 
> Используйте `!important` с осторожностью, так как это может усложнить отладку и поддержку кода.

## Специфичность

Специфичность определяет, какой CSS-селектор будет применен к элементу, если несколько селекторов имеют одинаковые свойства. Она вычисляется следующим образом:

- **Инлайновые стили**: 1000
- **ID селекторы**: 100
- **Классы, атрибуты, псевдоклассы**: 10
- **Элементы, псевдоэлементы**: 1

Пример:
```css
#main .content p { /* 100 + 10 + 1 = 111 */ }
div.content p { /* 10 + 10 + 1 = 21 */ }
p { /* 1 */ }
```

## Наследование

Некоторые CSS-свойства наследуются от родительских элементов к дочерним. Например, `color`, `font-family`, `text-align`. Другие, как `margin`, `padding`, `border`, не наследуются.

```css
body {
    font-family: Arial, sans-serif;
    color: #333;
}
/* Все текстовые элементы будут использовать Arial и цвет #333 */
```

## CSS-фреймворки

CSS-фреймворки ускоряют разработку и обеспечивают согласованный дизайн. Популярные фреймворки:

- [[Bootstrap]] - один из самых популярных, с обширной документацией и компонентами
- [[Tailwind CSS]] - utility-first фреймворк для быстрой сборки интерфейсов
- [[Bulma]] - современный фреймворк с чистым CSS
- [[Foundation]] - гибкий и адаптивный фреймворк

## Стилизация элементов формы

HTML-элементы формы требуют особого внимания к стилизации для обеспечения доступности и пользовательского опыта:

```css
input[type="text"], 
input[type="email"], 
textarea {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
}

input:focus, 
textarea:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}
```

## Адаптивность

Адаптивная вёрстка позволяет сайтам корректно отображаться на различных устройствах. Основные инструменты:

- **Медиазапросы**:
```css
@media (max-width: 768px) {
    .container {
        width: 100%;
        padding: 10px;
    }
}
```

- **Flexbox**:
```css
.container {
    display: flex;
    flex-wrap: wrap;
}
.item {
    flex: 1 1 300px;
}
```

- **Grid**:
```css
.grid-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}
```

## Анимации

CSS-анимации позволяют создавать плавные переходы и динамические эффекты:

```css
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

.fade-element {
    animation: fadeIn 1s ease-in-out;
}

/* Плавное изменение цвета фона */
.button {
    background-color: #007bff;
    transition: background-color 0.3s ease;
}
.button:hover {
    background-color: #0056b3;
}
```

## Доступность стилей

Важно учитывать доступность при стилизации:

- Обеспечьте достаточный контраст текста (минимум 4.5:1 для нормального текста)
- Используйте семантические HTML-элементы
- Обеспечьте видимость при фокусе для интерактивных элементов
- Поддерживайте масштабирование текста

```css
/* Обеспечение контраста */
.high-contrast-text {
    color: #000;
    background-color: #fff;
}

/* Видимый фокус */
button:focus {
    outline: 2px solid #007bff;
    outline-offset: 2px;
}
```

## Лучшие практики

- Используйте внешние стили для повторного использования
- Организуйте CSS с помощью методологий, таких как [[BEM (Block Element Modifier)]]
- Минимизируйте дублирование стилей
- Используйте относительные единицы измерения (em, rem, %)
- Оптимизируйте производительность с помощью сжатия CSS
- Проверяйте кроссбраузерность

## Связи с другими файлами

- [[html/structure]] - структура HTML-документа
- [[css/flexbox]] - подробное руководство по Flexbox
- [[css/grid]] - подробное руководство по CSS Grid
- [[css/responsive-design]] - адаптивный дизайн
- [[css/animations]] - продвинутые анимации
- [[accessibility/html]] - доступность в HTML
- [[performance/css-optimization]] - оптимизация CSS

## Теги

#css #html #styling #web-development #frontend #responsive-design #accessibility #animations #best-practices #frameworks