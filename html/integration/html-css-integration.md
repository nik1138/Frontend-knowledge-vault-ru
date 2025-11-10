# Интеграция HTML и CSS

Руководство по интеграции HTML и CSS, охватывающее способы подключения стилей, принципы каскадности, специфичности и наследования, а также лучшие практики.

## Способы подключения CSS к HTML

Существует три основных способа подключения CSS к HTML:

### 1. Встроенные стили (Inline styles)
```html
<p style="color: blue; font-size: 16px;">Этот текст синий и размером 16px</p>
```

### 2. Внутренние стили (Internal styles)
```html
<head>
  <style>
    p {
      color: blue;
      font-size: 16px;
    }
  </style>
</head>
```

### 3. Внешние стили (External styles)
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```

Для получения дополнительной информации о структуре HTML документа, см. [[html/structure/html-document-structure.md]].

## Приоритеты и каскад

CSS следует принципу каскадности, при котором стили применяются в определенном порядке:

1. **Встроенные стили** - имеют наивысший приоритет
2. **Внутренние стили** - средний приоритет
3. **Внешние стили** - низший приоритет
4. **Браузерные стили** - самые низкие

Приоритет также зависит от специфичности селекторов и использования `!important`.

## Специфичность селекторов

Специфичность определяет, какой CSS стиль будет применен к элементу. Она рассчитывается следующим образом:

- **Тег** (например, `p`, `div`) - 1 очко
- **Класс** (например, `.my-class`) - 10 очков
- **ID** (например, `#my-id`) - 100 очков
- **Встроенные стили** - 1000 очков

Пример:
```css
/* Специфичность: 1 (только тег) */
p { color: red; }

/* Специфичность: 11 (тег + класс) */
p.highlight { color: blue; }

/* Специфичность: 101 (тег + ID) */
p#unique { color: green; }

/* Специфичность: 1000 (встроенный стиль) */
p style="color: yellow;"
```

Для более подробного изучения селекторов, см. [[css/selectors/css-selectors-guide.md]].

## Наследование стилей

Некоторые CSS свойства наследуются дочерними элементами от родительских. Например:

```css
body {
  font-family: Arial, sans-serif;
  color: #333;
}
```

Все элементы внутри `<body>` унаследуют `font-family` и `color`, если только они не переопределены.

Свойства, которые **наследуются**:
- `color`
- `font-*`
- `text-*`
- `visibility`

Свойства, которые **не наследуются**:
- `margin`
- `padding`
- `border`
- `width`
- `height`

## Взаимодействие элементов

CSS позволяет управлять взаимодействием между элементами через:

- **Псевдоклассы**: `:hover`, `:focus`, `:active`
- **Псевдоэлементы**: `::before`, `::after`
- **Комбинаторы**: `>`, `+`, `~`, пробел

Пример:
```css
button:hover {
  background-color: #ccc;
}

.item + .item {
  margin-top: 10px;
}
```

Для изучения псевдоклассов и псевдоэлементов, см. [[css/pseudo-elements/pseudo-classes-elements.md]].

## Стилизация HTML элементов

Каждый HTML элемент может быть стилизован с помощью CSS. Пример стилизации формы:

```html
<form class="my-form">
  <input type="text" class="form-input">
  <button type="submit" class="form-button">Отправить</button>
</form>
```

```css
.my-form {
  padding: 20px;
  border: 1px solid #ddd;
}

.form-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #999;
}

.form-button {
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}
```

Для изучения стилизации конкретных элементов, см. [[html/elements/html-elements-styling.md]].

## CSS-фреймворки и библиотеки

Популярные CSS-фреймворки:

- **Bootstrap** - [[css/frameworks/bootstrap-guide.md]]
- **Tailwind CSS** - [[css/frameworks/tailwind-guide.md]]
- **Bulma**
- **Foundation**

Эти фреймворки ускоряют разработку и обеспечивают согласованный дизайн.

## Лучшие практики интеграции

1. **Разделяйте стили по файлам**:
   - `base.css` - базовые стили
   - `layout.css` - структура страницы
   - `components.css` - компоненты
   - `utilities.css` - вспомогательные классы

2. **Используйте семантические классы**: `.header`, `.navigation`, `.card`, а не `.red-text`, `.big-margin`

3. **Следуйте методологии BEM** (см. [[css/naming-conventions/bem-methodology.md]])

4. **Минимизируйте дублирование стилей**

5. **Оптимизируйте производительность** - минимизация CSS, использование сжатия

## Примеры кода

Пример комплексной интеграции HTML и CSS:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <title>Пример интеграции</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header class="main-header">
    <h1>Заголовок страницы</h1>
    <nav class="main-nav">
      <ul>
        <li><a href="#home">Главная</a></li>
        <li><a href="#about">О нас</a></li>
      </ul>
    </nav>
  </header>
  
  <main class="main-content">
    <section class="content-section">
      <h2>Раздел содержимого</h2>
      <p>Текст параграфа</p>
    </section>
  </main>
</body>
</html>
```

```css
/* base.css */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  line-height: 1.6;
}

/* layout.css */
.main-header {
  background-color: #333;
  color: white;
  padding: 1rem;
}

.main-nav ul {
  list-style: none;
  display: flex;
}

.main-nav a {
  color: white;
  text-decoration: none;
  margin-right: 1rem;
}

.main-content {
  padding: 2rem;
}

/* components.css */
.content-section {
  margin-bottom: 2rem;
}

.content-section h2 {
  color: #007bff;
  margin-bottom: 1rem;
}
```

## Связи с другими файлами

- [[html/structure/html-document-structure.md]] - структура HTML документа
- [[css/basics/css-basics.md]] - основы CSS
- [[css/selectors/css-selectors-guide.md]] - гид по селекторам CSS
- [[css/frameworks/bootstrap-guide.md]] - руководство по Bootstrap
- [[css/frameworks/tailwind-guide.md]] - руководство по Tailwind CSS
- [[css/naming-conventions/bem-methodology.md]] - методология BEM
- [[html/elements/html-elements-styling.md]] - стилизация HTML элементов

## Заключение

Интеграция HTML и CSS - ключевая часть веб-разработки. Понимание принципов каскадности, специфичности и наследования позволяет создавать эффективные и поддерживаемые веб-страницы.

#html #css #integration #web-development #styling #cascading #selectors #best-practices