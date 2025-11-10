---
aliases: ["DOM селекторы", "HTML селекторы"]
tags: [html, javascript, dom, selectors, frontend]
---

# DOM Селекторы в JavaScript

DOM (Document Object Model) селекторы позволяют находить и манипулировать элементами HTML-документа. Это основа для интерактивности веб-страниц.

## Основные методы поиска элементов

### document.getElementById

Находит элемент по уникальному идентификатору `id`.

```javascript
const element = document.getElementById('myId');
```

> [!info] Особенность
> Метод возвращает **один** элемент или `null`, если элемент не найден. Идентификатор должен быть уникальным на странице.

### document.getElementsByClassName

Находит элементы по имени CSS-класса.

```javascript
const elements = document.getElementsByClassName('myClass');
```

> [!tip] Возвращаемое значение
> Возвращает `HTMLCollection` — **живую** коллекцию элементов.

### document.getElementsByTagName

Находит элементы по имени тега.

```javascript
const divs = document.getElementsByTagName('div');
```

> [!info] Возвращаемое значение
> Возвращает `HTMLCollection` — **живую** коллекцию элементов.

### document.querySelector

Находит **первый** элемент, соответствующий CSS-селектору.

```javascript
const element = document.querySelector('.myClass');
const button = document.querySelector('button[type="submit"]');
```

> [!warning] Возвращаемое значение
> Возвращает **один** элемент или `null`, если элемент не найден.

### document.querySelectorAll

Находит **все** элементы, соответствующие CSS-селектору.

```javascript
const elements = document.querySelectorAll('.myClass');
const inputs = document.querySelectorAll('input[type="text"]');
```

> [!tip] Возвращаемое значение
> Возвращает `NodeList` — **статическую** коллекцию элементов.

## NodeList vs HTMLCollection

| Характеристика | NodeList | HTMLCollection |
|---|---|---|
| Возвращается из | `querySelectorAll` | `getElementsBy...` |
| Тип коллекции | Статическая | Живая |
| Поддержка `forEach` | Да | Нет (до ES2015) |

> [!note] Живые и статические коллекции
> - **Живые** коллекции обновляются при изменении DOM
> - **Статические** коллекции не изменяются после создания

## Производительность и выбор метода

| Метод | Производительность | Когда использовать |
|---|---|---|
| `getElementById` | Наибыстрейший | Когда известен уникальный `id` |
| `querySelector` | Быстрый | Для сложных селекторов |
| `getElementsBy...` | Быстрый | Для получения живых коллекций |
| `querySelectorAll` | Медленнее | Для получения статических коллекций |

## Практические примеры

### Пример 1: Поиск формы и её элементов

```javascript
const form = document.getElementById('contactForm');
const inputs = form.querySelectorAll('input');
const submitButton = form.querySelector('button[type="submit"]');
```

### Пример 2: Поиск по атрибутам

```javascript
// Найти все ссылки с определённым атрибутом
const externalLinks = document.querySelectorAll('a[data-external]');
```

### Пример 3: Комбинированный поиск

```javascript
// Найти все кнопки в конкретном контейнере
const container = document.getElementById('buttonContainer');
const buttons = container.querySelectorAll('button');
```

## Расширенные селекторы

Современные селекторы CSS позволяют создавать сложные запросы:

```javascript
// Псевдоклассы
document.querySelectorAll('li:nth-child(odd)');

// Атрибуты
document.querySelectorAll('[data-role="button"]');

// Комбинации
document.querySelectorAll('.container > .item:first-child');
```

## Лучшие практики

1. **Используйте `getElementById`** для поиска по `id` — это быстрейший метод
2. **Используйте `querySelector`/`querySelectorAll`** для сложных селекторов
3. **Избегайте избыточного поиска** — кэшируйте результаты
4. **Проверяйте существование** элементов перед работой с ними

```javascript
const element = document.querySelector('.myElement');
if (element) {
  // Работа с элементом
}
```

## Связанные темы

- [[html/events]] — Работа с событиями DOM
- [[html/manipulation]] — Манипуляции с DOM-элементами
- [[css/selectors]] — CSS-селекторы
- [[js/dom-api]] — DOM API в JavaScript

## Заключение

DOM-селекторы — ключ к интерактивности веб-страниц. Понимание различий между методами позволяет эффективно находить элементы и создавать производительные приложения.

> [!summary] Ключевые моменты
> - `getElementById` — самый быстрый для поиска по `id`
> - `querySelector`/`querySelectorAll` — гибкие методы с поддержкой CSS-селекторов
> - `HTMLCollection` — живая коллекция, `NodeList` — статическая
> - Выбирайте метод в зависимости от задачи и производительности