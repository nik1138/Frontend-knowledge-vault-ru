---
aliases: [BEM, БЭМ, Блок-Элемент-Модификатор]
tags: [css, methodology, bem, frontend]
---

# BEM методология - Блок-Элемент-Модификатор

## Введение в BEM

**BEM** (Блок-Элемент-Модификатор) — это методология разработки CSS, придуманная Яндексом, которая помогает создавать переиспользуемые компоненты и упрощает обслуживание CSS-кода. BEM позволяет разбивать пользовательский интерфейс на независимые блоки, создавать масштабируемые библиотеки компонентов и упрощать совместную работу над проектами.

BEM расшифровывается как:
- **Блок** (Block) — логически независимый компонент страницы
- **Элемент** (Element) — составная часть блока, выполняющая определенную функцию
- **Модификатор** (Modifier) — сущность, определяющая внешний вид, состояние или поведение блока/элемента

## Основные принципы BEM

### Блоки

Блок — это функционально независимый компонент страницы, который может быть повторно использован. Блоки не влияют друг на друга, их можно переносить с места на место, добавлять и удалять.

**Примеры блоков:**
- `header`
- `menu`
- `button`
- `input`

```css
/* Правильно - имена блоков в нижнем регистре */
.header { }
.menu { }
.button { }

/* Неправильно */
.Header { } /* не соблюдается регистр */
.main-button { } /* неоднозначное имя */
```

### Элементы

Элемент — это составная часть блока, реализующая определенную функцию. Элемент не может использоваться в отрыве от блока.

**Синтаксис:**
```
block-name__element-name
```

```css
/* Правильно - элементы связаны с блоком */
.menu__item { }
.user__avatar { }
.button__text { }

/* Неправильно */
.menu__item__subitem { } /* слишком глубокая вложенность */
.header__button { } /* элемент не принадлежит блоку */
```

### Модификаторы

Модификатор — это сущность, определяющая внешний вид, состояние или поведение блока или элемента. Модификаторы могут быть булевыми или ключ-значение.

**Синтаксис:**
- Булевый модификатор: `block-name_modifier-name`
- Ключ-значение: `block-name_modifier-name_modifier-value`

```css
/* Булевый модификатор */
.button_disabled { } /* кнопка отключена */
.menu_vertical { } /* вертикальное меню */

/* Ключ-значение */
.button_theme_islands { } /* тема островов */
.size_s { } /* маленький размер */
.button_size_m { } /* средняя кнопка */
```

## Структура именования

### Правила именования

1. **Только латинские буквы, цифры и дефисы**
2. **Нижний регистр**
3. **Разделители:**
   - Блок-элемент: `__`
   - Блок-модификатор: `_`
   - Элемент-модификатор: `_`

```css
/* Правильно */
.article-card { }
.article-card__title { }
.article-card_theme_dark { }
.article-card__title_size_large { }

/* Неправильно */
.ArticleCard { } /* верхний регистр */
.articleCard__title { } /* camelCase */
.article-card__title__subtitle { } /* двойное подчеркивание */
```

### Глубина вложенности

BEM не ограничивает глубину вложенности HTML-элементов, но ограничивает глубину именования CSS-классов. Рекомендуется использовать не более одного уровня вложенности элементов.

```html
<!-- HTML может быть глубоким -->
<div class="menu">
  <ul class="menu__list">
    <li class="menu__item">
      <div class="menu__item-wrapper">
        <span class="menu__item-text">Текст</span>
      </div>
    </li>
  </ul>
</div>
```

```css
/* Но CSS-имена остаются плоскими */
.menu__item { }
.menu__item-wrapper { }
.menu__item-text { }
```

## Практические примеры

### Простой блок

```html
<div class="search-form">
  <input class="search-form__input" type="text" placeholder="Поиск...">
  <button class="search-form__button">Найти</button>
</div>
```

```css
.search-form { 
  display: flex;
  position: relative;
}

.search-form__input { 
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
}

.search-form__button { 
  padding: 10px 15px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
}

.search-form__button:hover { 
  background: #0056b3;
}
```

### Блок с модификаторами

```html
<button class="button button_theme_islands button_size_large">Крупная кнопка</button>
<button class="button button_theme_islands button_disabled">Отключенная кнопка</button>
```

```css
.button { 
  display: inline-block;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  text-align: center;
  transition: background-color 0.2s;
}

/* Модификаторы темы */
.button_theme_islands { 
  background: #007bff;
  color: white;
}

.button_theme_islands:hover:not(.button_disabled) { 
  background: #0056b3;
}

/* Модификаторы размера */
.button_size_large { 
  padding: 12px 24px;
  font-size: 16px;
}

/* Булевый модификатор - отключенная кнопка */
.button_disabled { 
  opacity: 0.6;
  cursor: not-allowed;
}
```

### Комплексный пример

```html
<div class="card card_theme_dark">
  <div class="card__header">
    <h3 class="card__title">Заголовок карточки</h3>
    <span class="card__subtitle card__subtitle_hidden">Подзаголовок</span>
  </div>
  <div class="card__content">
    <p class="card__text">Содержимое карточки</p>
    <img class="card__image card__image_position_right" src="image.jpg" alt="Изображение">
  </div>
  <div class="card__footer">
    <button class="button button_theme_primary">Действие</button>
    <button class="button button_theme_secondary">Отмена</button>
  </div>
</div>
```

```css
.card { 
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  max-width: 400px;
}

.card_theme_dark { 
  border-color: #333;
  background-color: #f8f9fa;
}

.card__header { 
  padding: 16px;
  border-bottom: 1px solid #eee;
}

.card__title { 
  margin: 0;
  font-size: 18px;
  font-weight: bold;
}

.card__subtitle { 
  display: block;
  color: #666;
  font-size: 14px;
}

.card__subtitle_hidden { 
  display: none;
}

.card__content { 
  padding: 16px;
}

.card__text { 
  margin: 0 0 12px;
  line-height: 1.5;
}

.card__image { 
  max-width: 100%;
  height: auto;
}

.card__image_position_right { 
  float: right;
  margin-left: 12px;
}

.card__footer { 
  padding: 16px;
  border-top: 1px solid #eee;
  text-align: right;
}
```

## Преимущества BEM

### 1. Масштабируемость
- Легко добавлять новые компоненты
- Упрощается рефакторинг
- Уменьшается вероятность конфликта имен

### 2. Читаемость
- Понятная структура классов
- Легко понять взаимосвязи между элементами
- Упрощается поддержка кода

### 3. Повторное использование
- Блоки можно использовать в разных проектах
- Создание библиотек компонентов
- Унификация интерфейсов

### 4. Совместная работа
- Единый стиль написания CSS
- Легче новичкам вникать в проект
- Унификация подходов

## Недостатки и ограничения

### 1. Длинные имена классов
- Классы могут быть очень длинными
- Увеличивается размер CSS-файлов
- Снижается читаемость в HTML

### 2. Жесткая структура
- Требует строгого следования методологии
- Может быть избыточным для простых проектов
- Не всегда интуитивно понятно

### 3. Сложность при рефакторинге
- Изменение структуры требует обновления всех классов
- Зависимость между HTML и CSS

## Лучшие практики

### 1. Именование блоков
- Используйте понятные имена
- Избегайте семантических имен, если это не семантические элементы
- Думайте в терминах компонентов, а не визуальных элементов

```css
/* Лучше */
.search-form { }
.user-card { }
.product-list { }

/* Хуже */
.red-button { } /* семантика в имени */
.big-title { } /* визуальные характеристики */
```

### 2. Использование модификаторов
- Используйте модификаторы для изменения внешнего вида
- Не создавайте элементы только для стилизации
- Модификаторы должны изменять поведение/внешний вид, а не структуру

### 3. Структура файлов
При работе с BEM рекомендуется организовать CSS-файлы по компонентам:

```
blocks/
├── button/
│   ├── button.css
│   ├── button_theme.css
│   └── button_size.css
├── menu/
│   ├── menu.css
│   ├── menu__item.css
│   └── menu_theme.css
└── header/
    ├── header.css
    └── header__logo.css
```

### 4. Связь с HTML и JavaScript
- Используйте CSS-классы для стилизации
- JavaScript-классы лучше выносить в отдельные атрибуты
- Используйте специальные префиксы для JS-селекторов (например, `js-`)

```html
<!-- Правильно -->
<div class="modal" data-js="modal">
  <button class="modal__close" data-js="modal-close">Закрыть</button>
</div>
```

## Инструменты для работы с BEM

### 1. BEM Tools
- `bem-cli` — командная строка для работы с BEM
- `bem-lint` — проверка соответствия BEM-нотации

### 2. Препроцессоры
BEM особенно эффективен при использовании CSS-препроцессоров:

#### SCSS пример:
```scss
.button {
  display: inline-block;
  padding: 8px 16px;
  
  &_theme {
    &_primary {
      background: #007bff;
      color: white;
    }
    
    &_secondary {
      background: #6c757d;
      color: white;
    }
  }
  
  &_size {
    &_large {
      padding: 12px 24px;
      font-size: 16px;
    }
  }
  
  &__text {
    font-weight: bold;
  }
}
```

### 3. Фреймворки и библиотеки
- [[Bootstrap]] использует BEM-подобную нотацию
- [[Tailwind CSS]] предлагает альтернативный подход
- [[Sass]] и [[Less]] поддерживают вложенные селекторы, упрощающие BEM

## Альтернативы BEM

### 1. OOCSS (Object-Oriented CSS)
- Разделение структуры и оформления
- Создание повторно используемых объектов

### 2. SMACSS (Scalable and Modular Architecture)
- Категоризация CSS-правил
- Пять типов правил: базовые, макет, модуль, состояние, тема

### 3. Atomic CSS
- Создание минимальных стилевых атомов
- Высокая переиспользуемость

### 4. CSS-in-JS
- Инкапсуляция стилей в компонентах
- Динамические стили

## Заключение

BEM — мощная методология для создания масштабируемых и поддерживаемых CSS-архитектур. Она особенно полезна в крупных проектах с командной разработкой. Хотя у BEM есть свои недостатки, такие как длинные имена классов, его преимущества в организации кода и упрощении сопровождения часто перевешивают эти недостатки.

При правильном применении BEM способствует:
- Упрощению понимания кода
- Ускорению разработки
- Упрощению рефакторинга
- Унификации подходов в команде

## Ключевые концепции

- [[CSS архитектура]] — организация CSS-кода в проекте
- [[Семантическая верстка]] — использование правильных HTML-тегов
- [[Препроцессоры CSS]] — расширение возможностей CSS
- [[Модульность CSS]] — разделение кода на независимые части
- [[Переменные CSS]] — управление значениями в CSS