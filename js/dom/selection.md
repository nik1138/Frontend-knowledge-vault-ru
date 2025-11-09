# Выбор элементов DOM в JavaScript

## Обзор

Для манипуляции с элементами HTML-страницы в JavaScript, сначала нужно получить ссылки на эти элементы. Существует несколько методов для выбора элементов DOM, каждый из которых подходит для разных сценариев.

## Основные методы выбора элементов

### getElementById()

```javascript
// Выбор элемента по ID
const header = document.getElementById('main-header');
console.log(header); // <h1 id="main-header">Заголовок</h1>

// Если элемент не найден, возвращает null
const notFound = document.getElementById('non-existent');
console.log(notFound); // null
```

### getElementsByClassName()

```javascript
// Выбор элементов по имени класса
const buttons = document.getElementsByClassName('btn');
console.log(buttons); // HTMLCollection элементов с классом 'btn'

// Обращение к конкретному элементу
console.log(buttons[0]); // Первый элемент с классом 'btn'

// Обратите внимание: это живая коллекция, она обновляется при изменении DOM
```

### getElementsByTagName()

```javascript
// Выбор элементов по тегу
const paragraphs = document.getElementsByTagName('p');
console.log(paragraphs); // HTMLCollection всех параграфов

// Выбор всех элементов на странице
const allElements = document.getElementsByTagName('*');
console.log(allElements.length); // Количество всех элементов
```

### querySelector()

```javascript
// Выбор первого элемента, соответствующего CSS-селектору
const firstButton = document.querySelector('.btn');
console.log(firstButton); // Первый элемент с классом 'btn'

// Сложные селекторы
const specialButton = document.querySelector('.btn.primary');
const nestedElement = document.querySelector('div.container ul li:first-child');
const inputWithId = document.querySelector('input[name="username"]');
```

### querySelectorAll()

```javascript
// Выбор всех элементов, соответствующих CSS-селектору
const allButtons = document.querySelectorAll('.btn');
console.log(allButtons); // NodeList элементов с классом 'btn'

// NodeList не является живой коллекцией, он не обновляется при изменении DOM
const allHeaders = document.querySelectorAll('h1, h2, h3');
console.log(allHeaders); // NodeList всех заголовков h1, h2 и h3
```

## Различия между коллекциями

```javascript
// HTMLCollection (возвращается getElementsBy... методами)
const liveCollection = document.getElementsByClassName('item');
console.log(liveCollection.length); // Предположим, 3 элемента

// При добавлении элемента в DOM коллекция автоматически обновляется
const newDiv = document.createElement('div');
newDiv.className = 'item';
document.body.appendChild(newDiv);
console.log(liveCollection.length); // Теперь 4 элемента

// NodeList (возвращается querySelectorAll)
const staticNodeList = document.querySelectorAll('.item');
console.log(staticNodeList.length); // 4 элемента
// При добавлении новых элементов NodeList не обновляется
console.log(staticNodeList.length); // Все еще 4 элемента
```

## Практические примеры

### Выбор элементов формы

```javascript
// Выбор элементов формы по разным атрибутам
const form = document.querySelector('form[name="user-form"]');
const usernameInput = document.querySelector('input[name="username"]');
const emailInput = document.querySelector('#email');
const submitButton = document.querySelector('button[type="submit"]');
const allInputs = document.querySelectorAll('input, textarea, select');

// Проверка, что элементы найдены
if (usernameInput) {
    usernameInput.value = 'Иванов';
}

// Перебор коллекции элементов
allInputs.forEach(input => {
    console.log(input.name, input.type);
});
```

### Выбор элементов с определенными атрибутами

```javascript
// Выбор элементов с определенными атрибутами
const images = document.querySelectorAll('img[data-src]');
const links = document.querySelectorAll('a[href^="https://"]');
const hiddenElements = document.querySelectorAll('[hidden]');
const elementsWithCustomAttr = document.querySelectorAll('[data-role="button"]');

// Выбор элементов по нескольким условиям
const visibleButtons = document.querySelectorAll('button:not([hidden]):not([disabled])');
```

### Выбор дочерних элементов

```javascript
// Выбор дочерних элементов конкретного родителя
const container = document.querySelector('.container');
if (container) {
    const childParagraphs = container.querySelectorAll('p');
    const firstChild = container.querySelector(':first-child');
    const lastChild = container.querySelector(':last-child');
    
    // Выбор элементов внутри конкретного контейнера
    const navItems = container.querySelectorAll('nav ul li');
}
```

### Современные методы и техники

```javascript
// Использование closest() для поиска ближайшего родительского элемента
const button = document.querySelector('.delete-btn');
const listItem = button.closest('li'); // Находит ближайший родительский элемент li
console.log(listItem); // Родительский элемент li, содержащий кнопку

// Использование matches() для проверки соответствия селектору
const element = document.querySelector('.some-element');
if (element && element.matches('.special-class')) {
    console.log('Элемент имеет класс special-class');
}

// Выбор элементов с использованием псевдоклассов
const firstItems = document.querySelectorAll('li:first-child');
const evenRows = document.querySelectorAll('tr:nth-child(even)');
const focusedElement = document.querySelector(':focus');
```

## Практические советы

1. **Используйте `querySelector`/`querySelectorAll`** для сложных селекторов и большей гибкости
2. **Используйте `getElementById`** для поиска по ID - это самый быстрый метод
3. **Проверяйте существование элементов** перед работой с ними
4. **Рассмотрите возможность кэширования** часто используемых элементов
5. **Используйте `closest()` и `matches()`** для более сложных сценариев выбора

```javascript
// Пример кэширования элементов
const elementCache = {
    header: document.getElementById('main-header'),
    navigation: document.querySelector('nav'),
    mainContent: document.querySelector('.main-content'),
    footer: document.querySelector('footer')
};

// Теперь можно обращаться к элементам без повторного поиска
if (elementCache.header) {
    elementCache.header.style.backgroundColor = '#f0f0f0';
}
```