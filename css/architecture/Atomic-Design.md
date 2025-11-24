---
aliases: [Atomic Design, Атомарный дизайн]
tags: [css, architecture, methodology, scalable, components]
---

# Atomic Design

Atomic Design — это методология проектирования интерфейсов, разработанная Брэдом Фростом (Brad Frost). Она основана на идее, что интерфейсы можно строить из небольших, переиспользуемых компонентов, подобно тому, как атомы объединяются в молекулы, молекулы — в организмы, и так далее.

## Общая концепция

Atomic Design делит интерфейс на пять уровней:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Atomic Design Hierarchy                      │
├─────────────────────────────────────────────────────────────────┤
│  1. Атомы (Atoms)                                              │
│     • Базовые элементы: цвета, шрифты, кнопки, формы           │
├─────────────────────────────────────────────────────────────────┤
│  2. Молекулы (Molecules)                                        │
│     • Комбинации атомов: поля ввода с метками, формы поиска    │
├─────────────────────────────────────────────────────────────────┤
│  3. Организмы (Organisms)                                       │
│     • Комбинации молекул: шапка сайта, навигация, карточки     │
├─────────────────────────────────────────────────────────────────┤
│  4. Шаблоны (Templates)                                         │
│     • Структуры страниц: макеты с местами для контента         │
├─────────────────────────────────────────────────────────────────┤
│  5. Страницы (Pages)                                            │
│     • Конкретные реализации шаблонов: главная, каталог, etc.   │
└─────────────────────────────────────────────────────────────────┘
```

## Уровни Atomic Design

### 1. Атомы (Atoms)

Атомы — это базовые строительные блоки интерфейса. Они не могут быть разделены на более мелкие компоненты.

**Примеры атомов:**
- Цвета и палитры
- Шрифты и типографика
- Формы и поля ввода
- Кнопки
- Иконки

```css
/* atoms.buttons.css */
.a-btn {
  display: inline-block;
  padding: 0.5rem 1rem;
  border: 1px solid transparent;
  border-radius: 0.25rem;
  font-size: 1rem;
  line-height: 1.5;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}

.a-btn--primary {
  @extend .a-btn;
  background-color: #007bff;
  color: white;
  border-color: #007bff;
}

.a-btn--secondary {
  @extend .a-btn;
  background-color: #6c757d;
  color: white;
  border-color: #6c757d;
}

/* atoms.inputs.css */
.a-input {
  display: block;
  width: 100%;
  padding: 0.375rem 0.75rem;
  font-size: 1rem;
  line-height: 1.5;
  color: #495057;
  background-color: #fff;
  background-clip: padding-box;
  border: 1px solid #ced4da;
  border-radius: 0.25rem;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.a-input:focus {
  color: #495057;
  background-color: #fff;
  border-color: #80bdff;
  outline: 0;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}
```

### 2. Молекулы (Molecules)

Молекулы — это комбинации атомов, которые формируют относительно простые, но функциональные компоненты.

**Примеры молекул:**
- Поле ввода с меткой
- Форма поиска
- Карточка с изображением и текстом

```html
<!-- molecules.search-form.html -->
<form class="m-search-form">
  <div class="m-search-form__input-group">
    <label for="search" class="a-input-label">Поиск</label>
    <input type="text" id="search" class="a-input" placeholder="Введите запрос...">
    <button class="a-btn a-btn--primary m-search-form__submit">Найти</button>
  </div>
</form>

<!-- molecules.user-card.html -->
<div class="m-user-card">
  <img src="avatar.jpg" alt="Аватар" class="a-avatar">
  <div class="m-user-card__info">
    <h3 class="a-heading a-heading--h3">Иван Иванов</h3>
    <p class="a-text">Разработчик</p>
  </div>
</div>
```

```css
/* molecules.search-form.css */
.m-search-form {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.m-search-form__input-group {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.m-search-form__submit {
  align-self: flex-start;
  margin-top: 0.25rem;
}

/* molecules.user-card.css */
.m-user-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #e9ecef;
  border-radius: 0.25rem;
  background-color: white;
}

.m-user-card__info {
  flex: 1;
}
```

### 3. Организмы (Organisms)

Организмы — это более сложные компоненты, состоящие из атомов и молекул. Они могут содержать несколько молекул и даже другие организмы.

**Примеры организмов:**
- Шапка сайта (навигация + логотип + поиск)
- Карточка товара (изображение + информация + кнопки)
- Блок комментариев

```html
<!-- organisms.header.html -->
<header class="o-header">
  <div class="o-header__container">
    <div class="o-header__logo">
      <img src="logo.png" alt="Логотип" class="a-logo">
    </div>
    
    <nav class="o-header__nav">
      <ul class="o-nav-list">
        <li><a href="#" class="a-link">Главная</a></li>
        <li><a href="#" class="a-link">О нас</a></li>
        <li><a href="#" class="a-link">Контакты</a></li>
      </ul>
    </nav>
    
    <div class="o-header__search">
      <form class="m-search-form">
        <input type="text" class="a-input" placeholder="Поиск...">
        <button class="a-btn a-btn--primary">Найти</button>
      </form>
    </div>
  </div>
</header>
```

```css
/* organisms.header.css */
.o-header {
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.o-header__container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  max-width: 1200px;
  margin: 0 auto;
}

.o-header__logo {
  flex-shrink: 0;
}

.o-header__nav {
  flex-grow: 1;
  margin: 0 2rem;
}

.o-nav-list {
  display: flex;
  list-style: none;
  padding: 0;
  margin: 0;
  gap: 1.5rem;
}

.o-header__search {
  flex-shrink: 0;
}
```

### 4. Шаблоны (Templates)

Шаблоны — это каркасы страниц, которые показывают расположение компонентов. Они не содержат конкретного контента, а демонстрируют структуру.

**Примеры шаблонов:**
- Шаблон главной страницы
- Шаблон страницы каталога
- Шаблон страницы статьи

```html
<!-- templates.main-page.html -->
<div class="t-main-layout">
  <header class="o-header"></header>
  
  <main class="t-main-content">
    <aside class="o-sidebar">
      <nav class="o-navigation"></nav>
    </aside>
    
    <section class="t-content-area">
      <article class="o-article"></article>
      <div class="o-ads-block"></div>
    </section>
  </main>
  
  <footer class="o-footer"></footer>
</div>
```

```css
/* templates.main-page.css */
.t-main-layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.t-main-content {
  display: flex;
  flex: 1;
  gap: 1rem;
  padding: 1rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.t-content-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
```

### 5. Страницы (Pages)

Страницы — это конкретные реализации шаблонов с реальным контентом. Это финальный уровень, где тестируется вся система компонентов.

## Преимущества Atomic Design

1. **Модульность**: Компоненты можно легко переиспользовать в разных частях интерфейса
2. **Консистентность**: Все элементы следуют единой системе
3. **Поддержка**: Легче вносить изменения в отдельные компоненты
4. **Тестирование**: Можно тестировать компоненты изолированно
5. **Документация**: Каждый компонент может быть задокументирован отдельно

## Atomic Design в российских реалиях 2025 года

В 2025 году Atomic Design особенно актуален в российской веб-разработке по следующим причинам:

- **Государственные проекты**: Требуют строгой архитектуры и документации интерфейсов
- **Крупные корпоративные системы**: Где важна переиспользуемость и масштабируемость
- **E-commerce платформы**: Требуют гибкой системы компонентов для быстрого создания новых страниц
- **Дизайн-системы**: Atomic Design является основой для создания корпоративных дизайн-систем

### Примеры использования в российских компаниях:

- **Сбер**: Использует компонентный подход для своих цифровых продуктов
- **Яндекс**: Применяет атомарный дизайн в своих интерфейсах
- **Тинькофф**: Строит интерфейсы на основе переиспользуемых компонентов

## Интеграция с CSS-архитектурой

Atomic Design хорошо сочетается с другими методологиями:

- [[ITCSS]]: Можно использовать ITCSS для организации CSS-кода компонентов
- [[БЭМ]]: Часто используется для именования классов в компонентах
- [[CSS-архитектура]]: Atomic Design предоставляет структуру для архитектуры интерфейсов

## Инструменты для работы с Atomic Design

- **Storybook**: Для создания библиотек компонентов
- **Figma**: Для проектирования атомарных компонентов
- **Pattern Lab**: Специализированный инструмент для Atomic Design
- **Zeroheight**: Для документирования компонентов

## Заключение

Atomic Design — мощная методология для создания масштабируемых интерфейсов. Она особенно актуальна в российских реалиях 2025 года, где важны переиспользуемость, консистентность и документация интерфейсов. Сочетание Atomic Design с [[CSS-архитектура]] и [[ITCSS]] позволяет создавать надежные и поддерживаемые CSS-архитектуры.

## См. также

- [[ITCSS]]
- [[CSS-архитектура]]
- [[Организация-файлов]]
- [[Масштабируемость]]
- [[БЭМ]]