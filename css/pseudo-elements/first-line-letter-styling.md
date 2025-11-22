---
aliases: ["Псевдоэлементы first-line и first-letter", "::first-line ::first-letter", "Стилизация текста CSS"]
tags: [css, pseudo-elements, text-styling, first-line, first-letter]
---

# Псевдоэлементы ::first-line и ::first-letter: стилизация текста

## Введение

Псевдоэлементы `::first-line` и `::first-letter` позволяют применять специфические стили к первой строке текста и первой букве абзаца соответственно. Эти псевдоэлементы особенно полезны для создания типографских эффектов, улучшающих визуальное восприятие текста и создающих эстетически привлекательные композиции.

## Псевдоэлемент ::first-line

Псевдоэлемент `::first-line` применяет стили к первой строке текста в блочном элементе. Стили, которые могут быть применены к `::first-line`, ограничены:

- Свойства цвета и фона: `color`, `background-color`, `background-image`
- Свойства шрифта: `font-size`, `font-style`, `font-weight`, `font-family`
- Свойства текста: `text-decoration`, `text-transform`, `letter-spacing`, `word-spacing`, `line-height`
- Свойства выравнивания: `text-align`, `vertical-align`

### Основное использование

```css
.article-paragraph::first-line {
  font-weight: bold;
  color: #007bff;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.news-title::first-line {
  font-size: 1.2em;
  font-weight: 600;
  color: #28a745;
}

.quote::first-line {
  font-style: italic;
  color: #6c757d;
  font-size: 1.1em;
}
```

### Продвинутые примеры

```css
/* Стилизация первой строки с градиентом */
.gradient-first-line::first-line {
  background: linear-gradient(to right, #ff6b6b, #4ecdc4);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  font-weight: bold;
}

/* Комбинация с другими псевдоэлементами */
.combined-styling {
  position: relative;
}

.combined-styling::first-line {
  font-weight: bold;
  color: #007bff;
}

.combined-styling::before {
  content: "→ ";
  color: #007bff;
}
```

### Практические примеры использования

#### Стилизация новостных заголовков

```html
<article class="news-article">
  <h2 class="news-title">Заголовок новости</h2>
  <p class="news-content">Первая строка этого абзаца будет стилизована особым образом, а остальной текст останется без изменений...</p>
</article>
```

```css
.news-content::first-line {
  font-weight: 600;
  color: #0056b3;
  font-size: 1.1em;
}

.news-title {
  font-size: 1.8em;
  margin-bottom: 15px;
}

.news-title::first-line {
  color: #007bff;
  border-bottom: 2px solid #007bff;
  padding-bottom: 5px;
}
```

#### Литературные цитаты

```html
<blockquote class="literary-quote">
  <p>В начале было Слово, и Слово было у Бога, и Слово было Бог. Оно было в начале у Бога. И Слово стало плотию, и обитало с нами, полное благодати и истины.</p>
</blockquote>
```

```css
.literary-quote p::first-line {
  font-size: 1.3em;
  font-weight: 600;
  color: #495057;
  font-family: "Georgia", "Times New Roman", serif;
}

.literary-quote {
  border-left: 4px solid #007bff;
  padding-left: 20px;
  margin: 20px 0;
  font-style: italic;
}
```

## Псевдоэлемент ::first-letter

Псевдоэлемент `::first-letter` применяет стили к первой букве первого слова в блочном элементе. Доступные свойства для `::first-letter` включают:

- Свойства цвета и фона: `color`, `background-color`, `background-image`
- Свойства шрифта: `font-size`, `font-style`, `font-weight`, `font-family`
- Свойства границ: `border`, `border-radius`
- Свойства выравнивания: `float`, `vertical-align`
- Свойства отступов: `margin`, `padding`
- Свойства текста: `text-decoration`, `text-transform`

### Основное использование

```css
.drop-cap::first-letter {
  font-size: 3em;
  font-weight: bold;
  float: left;
  line-height: 1;
  margin-right: 5px;
  color: #007bff;
}

.initial-letter::first-letter {
  font-size: 2.5em;
  font-weight: bold;
  color: #dc3545;
  background-color: #f8d7da;
  padding: 5px;
  border-radius: 4px;
}
```

### Продвинутые примеры

```css
/* Буквица с тенью */
.shadowed-initial::first-letter {
  font-size: 3em;
  font-weight: bold;
  color: #28a745;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  float: left;
  line-height: 1;
  margin: 0 8px 0 0;
}

/* Буквица с градиентом */
.gradient-initial::first-letter {
  font-size: 3em;
  font-weight: bold;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  float: left;
  line-height: 1;
  margin: 0 8px 0 0;
}

/* Буквица с рамкой */
bordered-initial::first-letter {
  font-size: 2.5em;
  font-weight: bold;
  color: #007bff;
  border: 2px solid #007bff;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  float: left;
  margin: 5px 10px 5px 0;
  line-height: 1;
}
```

### Практические примеры использования

#### Литературные абзацы с буквицей

```html
<article class="literary-text">
  <p class="drop-cap-paragraph">Однажды утром, поев каши, Женя пошел в лес. Солнце светило ярко, птицы пели, и все вокруг казалось прекрасным. Он шел по тропинке, любуясь природой, и вдруг услышал странный звук.</p>
  
  <p class="drop-cap-paragraph">Второй абзац также начинается с буквицы, создавая литературный эффект.</p>
</article>
```

```css
.drop-cap-paragraph {
  line-height: 1.6;
  margin-bottom: 1em;
}

.drop-cap-paragraph::first-letter {
  font-size: 3em;
  font-weight: bold;
  color: #007bff;
  float: left;
  line-height: 1;
  margin: 0.1em 0.1em 0 0;
  font-family: "Georgia", "Times New Roman", serif;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.2);
}

/* Обеспечиваем правильное выравнивание остального текста */
.drop-cap-paragraph {
  overflow: hidden; /* Предотвращает наложение текста на буквицу */
}
```

#### Стилизация заголовков статей

```html
<article class="styled-article">
  <h1 class="fancy-title">Прекрасное путешествие</h1>
  <p>Текст статьи начинается с эффектной буквицы...</p>
</article>
```

```css
.fancy-title {
  font-size: 2.5em;
  margin-bottom: 20px;
  position: relative;
}

.fancy-title::first-letter {
  color: #dc3545;
  font-size: 1.5em;
  text-transform: uppercase;
}

.fancy-title::before {
  content: "";
  position: absolute;
  bottom: -5px;
  left: 0;
  width: 50px;
  height: 3px;
  background: linear-gradient(to right, #dc3545, transparent);
}
```

## Комбинация ::first-line и ::first-letter

Можно использовать оба псевдоэлемента одновременно:

```css
.combined-styling {
  line-height: 1.6;
}

.combined-styling::first-line {
  font-weight: bold;
  color: #007bff;
}

.combined-styling::first-letter {
  font-size: 2em;
  color: #dc3545;
  float: left;
  line-height: 1;
  margin-right: 5px;
}
```

### Пример с литературным эффектом

```html
<div class="literary-paragraph">
  <p>В одном сказочном королевстве, где солнце всегда светило ярко, а реки пели свои вечные песни, жил-был один очень интересный человек. Его звали...</p>
</div>
```

```css
.literary-paragraph p {
  font-family: "Georgia", "Times New Roman", serif;
  font-size: 1.1em;
  line-height: 1.8;
  text-align: justify;
}

.literary-paragraph p::first-line {
  font-variant: small-caps;
  letter-spacing: 1px;
  color: #495057;
}

.literary-paragraph p::first-letter {
  font-size: 3.5em;
  font-weight: bold;
  color: #007bff;
  float: left;
  line-height: 1;
  margin: 0.05em 0.1em 0 0;
  font-family: "Palatino", "Georgia", serif;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}
```

## Адаптивность и медиа-запросы

```css
.responsive-drop-cap p::first-letter {
  font-size: 2.5em;
  font-weight: bold;
  color: #007bff;
  float: left;
  line-height: 1;
  margin: 0 8px 0 0;
}

@media (max-width: 768px) {
  .responsive-drop-cap p::first-letter {
    font-size: 2em; /* Уменьшаем размер на мобильных */
    margin: 0 5px 0 0;
  }
}

@media (max-width: 480px) {
  .responsive-drop-cap p::first-letter {
    font-size: 1.5em; /* Еще больше уменьшаем на маленьких экранах */
    float: none; /* Убираем float на очень маленьких экранах */
    display: inline;
  }
}
```

## Совместимость и особенности

### Совместимость с браузерами

Оба псевдоэлемента имеют отличную поддержку во всех современных браузерах. Для совместимости с более старыми браузерами можно использовать одинарный двоеточие:

```css
/* Совместимая версия */
.element:first-line {
  /* стили для ::first-line */
}

.element:first-letter {
  /* стили для ::first-letter */
}

/* Современная версия */
.element::first-line {
  /* стили для ::first-line */
}

.element::first-letter {
  /* стили для ::first-letter */
}
```

### Особенности поведения

```css
/* ::first-letter применяется только к блочным элементам */
.inline-first-letter {
  display: inline; /* ::first-letter не будет работать */
}

.inline-first-letter-block {
  display: inline-block; /* ::first-letter будет работать */
}

/* ::first-line учитывает только первую форматированную строку */
.long-first-line {
  width: 200px; /* Если текст не помещается в одну строку, ::first-line применяется только к первой строке */
}

.long-first-line::first-line {
  background-color: #f8f9fa;
}
```

## Расширенные примеры

### Стилизация цитат с буквицей

```html
<blockquote class="fancy-quote">
  <p>Мудрость — это не возраст, а состояние души. Человек может быть молод душой, но мудр сердцем.</p>
  <footer>— Известный философ</footer>
</blockquote>
```

```css
.fancy-quote {
  position: relative;
  padding: 20px 20px 20px 60px;
  background-color: #f8f9fa;
  border-left: 4px solid #007bff;
  font-style: italic;
  margin: 20px 0;
  border-radius: 0 4px 4px 0;
}

.fancy-quote::before {
  content: """;
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 4em;
  color: #007bff;
  opacity: 0.3;
  font-family: serif;
}

.fancy-quote p {
  margin: 0;
  position: relative;
  z-index: 1;
}

.fancy-quote p::first-letter {
  font-size: 2em;
  font-weight: bold;
  color: #007bff;
  float: left;
  line-height: 1;
  margin: 0 5px 0 0;
}

.fancy-quote footer {
  margin-top: 10px;
  font-style: normal;
  font-size: 0.9em;
  text-align: right;
  color: #6c757d;
}
```

### Эффект "старинной книги"

```html
<div class="antique-text">
  <p>В далёкие времена, когда мир был полон чудес и тайн, существовало волшебное королевство, где жили добрые волшебники и драконы, которые больше любили спать, чем охранять сокровища.</p>
</div>
```

```css
.antique-text {
  font-family: "Times New Roman", Times, serif;
  line-height: 1.8;
  color: #333;
  background-color: #fdf6e3; /* Цвет старой бумаги */
  padding: 20px;
  border: 1px solid #d4c4a8;
  border-radius: 4px;
}

.antique-text p::first-line {
  font-variant: small-caps;
  letter-spacing: 1px;
  color: #5a5245;
}

.antique-text p::first-letter {
  font-size: 3em;
  font-weight: bold;
  color: #8b4513; /* Цвет сепии */
  float: left;
  line-height: 1;
  margin: 0.05em 0.1em 0 0;
  font-family: "Palatino", serif;
  text-shadow: 1px 1px 1px rgba(0,0,0,0.1);
}

.antique-text {
  position: relative;
}

.antique-text::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    radial-gradient(
      rgba(0,0,0,0.05) 1px, 
      transparent 1px
    );
  background-size: 20px 20px;
  pointer-events: none;
  z-index: 1;
}
```

## Заключение

Псевдоэлементы `::first-line` и `::first-letter` предоставляют мощные возможности для типографской стилизации текста. Их правильное использование позволяет создавать эстетически привлекательные и читаемые текстовые композиции, особенно эффективные в литературных и новостных публикациях.

Для изучения других псевдоэлементов рекомендуется ознакомиться с [[Псевдоэлементы ::before и ::after: продвинутые техники]] и [[Другие псевдоэлементы (selection, placeholder, marker и др.)]].

## Ссылки

- [MDN: ::first-line](https://developer.mozilla.org/ru/docs/Web/CSS/::first-line)
- [MDN: ::first-letter](https://developer.mozilla.org/ru/docs/Web/CSS/::first-letter)
- [CSS-Tricks: :first-line](https://css-tricks.com/almanac/selectors/f/first-line/)
- [CSS-Tricks: :first-letter](https://css-tricks.com/almanac/selectors/f/first-letter/)