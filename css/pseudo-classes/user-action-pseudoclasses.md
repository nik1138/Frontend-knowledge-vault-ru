---
aliases: ["Псевдоклассы пользовательских действий", "target", "lang", "dir", "user-actions"]
tags: [css, pseudo-classes, user-actions, accessibility]
---

# Псевдоклассы пользовательских действий (target, lang, dir и др.)

## Введение

Псевдоклассы пользовательских действий позволяют применять стили к элементам в зависимости от действий пользователя или специфических атрибутов элементов. Эти псевдоклассы особенно полезны для создания интерактивных интерфейсов, многоязычных сайтов и улучшения доступности.

## Псевдокласс :target

Псевдокласс `:target` применяется к элементу, чей идентификатор совпадает с фрагментом URL (частью после #).

### Основное использование

```css
/* Стили для целевого элемента */
.target-element:target {
  background-color: #fff3cd;
  border: 2px solid #ffc107;
  padding: 15px;
  border-radius: 4px;
  box-shadow: 0 0 10px rgba(255, 193, 7, 0.5);
}

/* Анимация при достижении целевого элемента */
.target-element:target {
  animation: highlight 2s ease;
}

@keyframes highlight {
  0% { background-color: #ffffff; }
  50% { background-color: #fff3cd; }
  100% { background-color: #ffffff; }
}
```

### Практический пример с навигацией

```html
<nav>
  <a href="#section1">Раздел 1</a>
  <a href="#section2">Раздел 2</a>
  <a href="#section3">Раздел 3</a>
</nav>

<section id="section1" class="content-section">
  <h2>Раздел 1</h2>
  <p>Содержимое первого раздела...</p>
</section>

<section id="section2" class="content-section">
  <h2>Раздел 2</h2>
  <p>Содержимое второго раздела...</p>
</section>

<section id="section3" class="content-section">
  <h2>Раздел 3</h2>
  <p>Содержимое третьего раздела...</p>
</section>
```

```css
.content-section {
  padding: 20px;
  margin: 20px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.content-section:target {
  background-color: #e7f3ff;
  border-color: #007bff;
  box-shadow: 0 0 15px rgba(0, 123, 255, 0.3);
  transform: scale(1.02);
  transition: all 0.3s ease;
}

.content-section:target h2 {
  color: #007bff;
  text-decoration: underline;
}
```

### Использование :target для создания табов

```html
<div class="tabs-container">
  <nav class="tabs-nav">
    <a href="#tab1" class="tab-link">Вкладка 1</a>
    <a href="#tab2" class="tab-link">Вкладка 2</a>
    <a href="#tab3" class="tab-link">Вкладка 3</a>
  </nav>
  
  <div id="tab1" class="tab-content">
    <h3>Содержимое вкладки 1</h3>
    <p>Текст первой вкладки...</p>
  </div>
  
  <div id="tab2" class="tab-content">
    <h3>Содержимое вкладки 2</h3>
    <p>Текст второй вкладки...</p>
  </div>
  
  <div id="tab3" class="tab-content">
    <h3>Содержимое вкладки 3</h3>
    <p>Текст третьей вкладки...</p>
  </div>
</div>
```

```css
.tabs-container {
  max-width: 600px;
  margin: 0 auto;
}

.tabs-nav {
  display: flex;
  border-bottom: 2px solid #ddd;
}

.tab-link {
  padding: 10px 20px;
  text-decoration: none;
  color: #333;
  border: 1px solid transparent;
  border-bottom: none;
  border-radius: 4px 4px 0 0;
  margin-right: 5px;
}

.tab-link:target,
.tab-link:target ~ .tab-link {
  background-color: white;
  border-color: #ddd;
}

.tab-content {
  display: none;
  padding: 20px;
  border: 1px solid #ddd;
  border-top: none;
  background-color: white;
}

.tab-content:target {
  display: block;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

## Псевдокласс :lang

Псевдокласс `:lang()` применяется к элементам с определенным языком, заданным атрибутом `lang`.

### Основное использование

```css
/* Применяется к элементам с языком русский */
:lang(ru) {
  font-family: "PT Sans", "Arial", sans-serif;
}

/* Применяется к элементам с языком английский */
:lang(en) {
  font-family: "Open Sans", "Helvetica", sans-serif;
}

/* Различные кавычки для разных языков */
:lang(ru)::before {
  content: "«";
}

:lang(ru)::after {
  content: "»";
}

:lang(en)::before {
  content: """;
}

:lang(en)::after {
  content: """;
}
```

### Практический пример многоязычного контента

```html
<div lang="ru">
  <p>Это русский текст с тире: — Привет, мир!</p>
</div>

<div lang="en">
  <p>This is English text with dashes: — Hello, world!</p>
</div>

<div lang="fr">
  <p>Ceci est un texte français : Bonjour le monde !</p>
</div>
```

```css
:lang(ru) {
  font-family: "PT Sans", "Arial", sans-serif;
  line-height: 1.6;
}

:lang(en) {
  font-family: "Open Sans", "Helvetica", sans-serif;
  line-height: 1.5;
}

:lang(fr) {
  font-family: "Merriweather", "Georgia", serif;
  line-height: 1.7;
}

/* Специфическая стилизация для русского языка */
:lang(ru) p {
  text-indent: 1.5em;
}

/* Специфическая стилизация для французского языка */
:lang(fr) {
  quotes: "«" "»" "‘" "’";
}

:lang(fr) q::before {
  content: open-quote;
}

:lang(fr) q::after {
  content: close-quote;
}

/* Стили для цитат на разных языках */
:lang(ru) blockquote {
  border-left: 4px solid #007bff;
  padding-left: 20px;
  margin-left: 0;
}

:lang(en) blockquote {
  border-left: 4px solid #28a745;
  padding-left: 20px;
  margin-left: 0;
}

:lang(fr) blockquote {
  border-left: 4px solid #dc3545;
  padding-left: 20px;
  margin-left: 0;
}
```

### Использование :lang для форматирования чисел и дат

```css
/* Различные форматы дат для разных языков */
:lang(ru) .date {
  font-variant-numeric: tabular-nums;
}

:lang(en) .date {
  font-variant-numeric: oldstyle-nums;
}

/* Различные форматы чисел */
:lang(ru) .number {
  font-feature-settings: "tnum" 1; /* Табличные числа */
}

:lang(de) .number {
  font-feature-settings: "frac" 1; /* Дроби */
}
```

## Псевдокласс :dir

Псевдокласс `:dir()` применяется к элементам с определенным направлением текста (ltr или rtl).

### Основное использование

```css
/* Для текста слева направо */
:dir(ltr) {
  text-align: left;
}

/* Для текста справа налево */
:dir(rtl) {
  text-align: right;
}

/* Адаптация интерфейса под направление текста */
.nav-item:dir(ltr) {
  margin-right: 10px;
  margin-left: 0;
}

.nav-item:dir(rtl) {
  margin-left: 10px;
  margin-right: 0;
}

/* Различные отступы для разных направлений */
.content:dir(ltr) {
  padding-left: 20px;
  padding-right: 10px;
}

.content:dir(rtl) {
  padding-right: 20px;
  padding-left: 10px;
}
```

### Практический пример с многоязычным интерфейсом

```html
<div dir="ltr" lang="en">
  <h2>English Content</h2>
  <p>This content is in English with left-to-right direction.</p>
  <nav class="navigation">
    <a href="#" class="nav-link">Home</a>
    <a href="#" class="nav-link">About</a>
    <a href="#" class="nav-link">Contact</a>
  </nav>
</div>

<div dir="rtl" lang="ar">
  <h2>محتوى عربي</h2>
  <p>هذا المحتوى باللغة العربية مع اتجاه من اليمين إلى اليسار.</p>
  <nav class="navigation">
    <a href="#" class="nav-link">الرئيسية</a>
    <a href="#" class="nav-link">من نحن</a>
    <a href="#" class="nav-link">اتصل بنا</a>
  </nav>
</div>
```

```css
/* Общие стили */
.navigation {
  display: flex;
  padding: 10px;
  background-color: #f8f9fa;
}

.nav-link {
  padding: 8px 16px;
  text-decoration: none;
  color: #007bff;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.nav-link:hover {
  background-color: #e9ecef;
}

/* Стили для LTR (слева направо) */
[lang="en"] .navigation {
  justify-content: flex-start;
}

[lang="en"] .nav-link {
  margin-right: 10px;
}

/* Стили для RTL (справа налево) */
[lang="ar"] .navigation {
  justify-content: flex-end;
}

[lang="ar"] .nav-link {
  margin-left: 10px;
}

/* Адаптация формы для разных направлений */
.form-container:dir(ltr) {
  text-align: left;
}

.form-container:dir(rtl) {
  text-align: right;
}

.form-group:dir(ltr) {
  margin-right: 0;
  margin-left: 0;
}

.form-group:dir(rtl) {
  margin-left: 0;
  margin-right: 0;
}

.input-field:dir(ltr) {
  padding-left: 10px;
  padding-right: 30px;
}

.input-field:dir(rtl) {
  padding-right: 10px;
  padding-left: 30px;
}
```

## Псевдокласс :not()

Псевдокласс `:not()` позволяет исключить элементы из селектора.

### Основное использование

```css
/* Применяется ко всем элементам p, кроме тех, у которых есть класс intro */
p:not(.intro) {
  color: #666;
  font-style: italic;
}

/* Исключение элементов с определенными атрибутами */
input:not([type="submit"]):not([type="button"]) {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Исключение элементов с определенными состояниями */
button:not(:disabled):not(.disabled) {
  cursor: pointer;
  background-color: #007bff;
  color: white;
}

button:not(:disabled):not(.disabled):hover {
  background-color: #0056b3;
}
```

### Практические примеры

```css
/* Стилизация всех ссылок, кроме тех, что в шапке */
main a:not(.header-link) {
  color: #007bff;
  text-decoration: none;
}

main a:not(.header-link):hover {
  text-decoration: underline;
}

/* Стилизация всех заголовков, кроме h1 */
h2:not(.main-title),
h3:not(.main-title),
h4:not(.main-title),
h5:not(.main-title),
h6:not(.main-title) {
  color: #495057;
  border-left: 3px solid #007bff;
  padding-left: 10px;
}

/* Исключение элементов в зависимости от состояния */
.card:not(.featured):not(.premium) {
  opacity: 0.8;
}

.card:not(.featured):not(.premium):hover {
  opacity: 1;
}
```

## Псевдокласс :has()

Псевдокласс `:has()` позволяет выбирать элементы, которые содержат другие элементы, соответствующие селектору.

### Основное использование (экспериментальная возможность)

```css
/* Выбирает заголовки, после которых идет абзац */
h1:has(+ p),
h2:has(+ p),
h3:has(+ p) {
  margin-bottom: 0.5em;
}

/* Выбирает статьи, содержащие изображения */
article:has(img) {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
}

/* Выбирает ссылки, содержащие изображения */
a:has(img) {
  display: inline-block;
  border: none;
}

/* Выбирает формы с ошибками */
form:has(input:invalid) {
  border: 2px solid #dc3545;
  padding: 15px;
  background-color: #f8d7da;
}
```

## Псевдоклассы для доступности

### :defined

Применяется к элементам пользовательских элементов, которые были определены:

```css
/* Стили для пользовательских элементов после их определения */
my-component:defined {
  display: block;
  padding: 20px;
  border: 1px solid #ccc;
}
```

### :placeholder-shown

Применяется к элементу input или textarea, когда отображается placeholder:

```css
.input-field:placeholder-shown {
  color: #999;
  font-style: italic;
}

.input-field:placeholder-shown:focus {
  outline: 1px solid #007bff;
}
```

## Комплексные примеры

### Многоязычный интерфейс с поддержкой RTL

```html
<div class="multilang-container">
  <header dir="ltr" lang="en">
    <h1>English Website</h1>
    <nav>
      <a href="#" class="nav-link">Home</a>
      <a href="#" class="nav-link">About</a>
      <a href="#" class="nav-link">Contact</a>
    </nav>
  </header>
  
  <main dir="rtl" lang="ar">
    <section id="about">
      <h2>من نحن</h2>
      <p>هذا النص باللغة العربية</p>
    </section>
    <section id="services">
      <h2>الخدمات</h2>
      <p>قائمة بالخدمات المقدمة</p>
    </section>
  </main>
  
  <footer dir="ltr" lang="en">
    <p>&copy; 2023 English Website. All rights reserved.</p>
  </footer>
</div>
```

```css
.multilang-container {
  max-width: 1200px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

/* Стили для английской части (LTR) */
[lang="en"] {
  text-align: left;
  direction: ltr;
}

[lang="en"] nav {
  display: flex;
  gap: 20px;
}

[lang="en"] .nav-link {
  text-decoration: none;
  color: #007bff;
  padding: 5px 10px;
  border-radius: 4px;
}

[lang="en"] .nav-link:hover {
  background-color: #e9ecef;
}

/* Стили для арабской части (RTL) */
[lang="ar"] {
  text-align: right;
  direction: rtl;
  font-family: "Arabic Transparent", "Simplified Arabic", Arial, sans-serif;
}

[lang="ar"] section {
  margin-bottom: 30px;
  padding: 20px;
  border-right: 4px solid #007bff;
  background-color: #f8f9fa;
}

/* Стили для целевых элементов */
[lang="ar"] section:target {
  background-color: #fff3cd;
  border-right-color: #ffc107;
  box-shadow: -5px 0 15px rgba(255, 193, 7, 0.3);
}

/* Адаптация формы под язык */
.form-container[lang="ar"] label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-container[lang="ar"] input {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Стили для языка, отличного от английского */
:not([lang="en"]) .special-note {
  background-color: #d1ecf1;
  border: 1px solid #bee5eb;
  color: #0c5460;
  padding: 10px;
  border-radius: 4px;
  margin: 10px 0;
}
```

### Интерактивная таблица с языковой адаптацией

```html
<table class="multilang-table" lang="ru">
  <thead>
    <tr>
      <th>Название</th>
      <th>Описание</th>
      <th>Цена</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Товар 1</td>
      <td>Описание товара 1</td>
      <td>1000 руб.</td>
    </tr>
    <tr>
      <td>Товар 2</td>
      <td>Описание товара 2</td>
      <td>2000 руб.</td>
    </tr>
  </tbody>
</table>
```

```css
.multilang-table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0;
  font-size: 16px;
  font-family: Arial, sans-serif;
}

.multilang-table th,
.multilang-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.multilang-table th {
  background-color: #f8f9fa;
  font-weight: bold;
}

/* Адаптация для русского языка */
:lang(ru) .multilang-table {
  font-family: "PT Sans", Arial, sans-serif;
}

:lang(ru) .multilang-table th {
  background-color: #e9ecef;
}

/* Адаптация для английского языка */
:lang(en) .multilang-table {
  font-family: "Open Sans", Helvetica, Arial, sans-serif;
}

:lang(en) .multilang-table th {
  background-color: #f8f9fa;
}

/* Зебра-эффект */
.multilang-table tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

.multilang-table tbody tr:hover {
  background-color: #f5f5f5;
}

/* Стили для целевой строки (если используется якорь) */
.multilang-table tbody tr:target {
  background-color: #fff3cd;
  outline: 2px solid #ffc107;
}
```

## Заключение

Псевдоклассы пользовательских действий предоставляют мощные возможности для создания адаптивных, многоязычных и доступных веб-интерфейсов. Их правильное использование позволяет улучшить пользовательский опыт и обеспечить соответствие международным стандартам веб-разработки.

Для изучения других категорий псевдоклассов рекомендуется ознакомиться с [[Псевдоклассы форм и состояний (focus, hover, active, focus-within и др.)]] и [[Псевдоклассы структурного соответствия (nth-child, first-child, last-child, only-child и др.)]].

## Ссылки

- [MDN: Псевдоклассы пользовательских действий](https://developer.mozilla.org/ru/docs/Web/CSS/Pseudo-classes#%D0%BF%D1%81%D0%B5%D0%B2%D0%B4%D0%BE%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D1%8B_%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D1%81%D0%BA%D0%B8%D1%85_%D0%B4%D0%B5%D0%B9%D1%81%D1%82%D0%B2%D0%B8%D0%B9)
- [MDN: :target](https://developer.mozilla.org/ru/docs/Web/CSS/:target)
- [MDN: :lang](https://developer.mozilla.org/ru/docs/Web/CSS/:lang)
- [MDN: :dir](https://developer.mozilla.org/ru/docs/Web/CSS/:dir)
- [CSS-Tricks: :not()](https://css-tricks.com/almanac/selectors/n/not/)