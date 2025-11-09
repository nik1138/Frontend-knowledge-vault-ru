# DOM API

## Введение

DOM (Document Object Model) - это кроссплатформенное и языконезависимое соглашение для представления и взаимодействия с HTML и XML документами. DOM представляет документ в виде дерева узлов, где каждый узел является объектом, представляющим часть документа.

## Структура DOM

DOM представляет HTML-документ в виде иерархии узлов:

```
document
├── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── header
        ├── main
        └── footer
```

## Основные методы работы с DOM

### Выбор элементов

```javascript
// По ID
const element = document.getElementById('myId');

// По тегу
const elements = document.getElementsByTagName('div');

// По классу
const elements = document.getElementsByClassName('myClass');

// Современные методы
const element = document.querySelector('#myId');
const elements = document.querySelectorAll('.myClass');
```

### Создание и удаление элементов

```javascript
// Создание элемента
const newDiv = document.createElement('div');
newDiv.className = 'new-element';
newDiv.textContent = 'Новый элемент';

// Создание текстового узла
const textNode = document.createTextNode('Текстовый узел');

// Удаление элемента
const element = document.getElementById('toBeRemoved');
element.parentNode.removeChild(element);

// Или более современный способ
element.remove();
```

### Изменение содержимого

```javascript
// Изменение текста
element.textContent = 'Новый текст';

// Изменение HTML
element.innerHTML = '<strong>Новый HTML</strong>';

// Изменение атрибутов
element.setAttribute('data-value', 'new-value');
element.removeAttribute('data-old');

// Работа с классами
element.classList.add('new-class');
element.classList.remove('old-class');
element.classList.toggle('active');
element.classList.contains('some-class');
```

## Примеры из промышленной разработки

### Динамическая загрузка контента

```javascript
// Пример динамической загрузки контента
async function loadContent(url, containerId) {
    try {
        const response = await fetch(url);
        const html = await response.text();
        
        const container = document.getElementById(containerId);
        container.innerHTML = html;
        
        // Вызов колбэка после загрузки
        if (typeof onContentLoaded === 'function') {
            onContentLoaded(container);
        }
    } catch (error) {
        console.error('Ошибка загрузки контента:', error);
    }
}

// Использование
loadContent('/api/content', 'content-container');
```

### Динамическое создание формы

```javascript
// Функция для создания формы
function createForm(fields) {
    const form = document.createElement('form');
    
    fields.forEach(field => {
        const label = document.createElement('label');
        label.textContent = field.label;
        
        const input = document.createElement('input');
        input.type = field.type || 'text';
        input.name = field.name;
        input.required = field.required || false;
        
        form.appendChild(label);
        form.appendChild(input);
        form.appendChild(document.createElement('br'));
    });
    
    const submitBtn = document.createElement('button');
    submitBtn.type = 'submit';
    submitBtn.textContent = 'Отправить';
    
    form.appendChild(submitBtn);
    
    return form;
}

// Использование
const formFields = [
    { label: 'Имя:', name: 'name', required: true },
    { label: 'Email:', name: 'email', type: 'email', required: true },
    { label: 'Сообщение:', name: 'message', type: 'textarea' }
];

const form = createForm(formFields);
document.body.appendChild(form);
```

## Лучшие практики

### 1. Использование делегирования событий

```javascript
// Вместо добавления обработчиков на каждый элемент
document.querySelectorAll('.button').forEach(button => {
    button.addEventListener('click', handleClick);
});

// Используйте делегирование
document.addEventListener('click', function(event) {
    if (event.target.classList.contains('button')) {
        handleClick(event);
    }
});
```

### 2. Кэширование DOM-элементов

```javascript
// Плохо - каждый раз обращение к DOM
function updateCounter() {
    document.getElementById('counter').textContent = 
        parseInt(document.getElementById('counter').textContent) + 1;
}

// Хорошо - кэширование элемента
const counterElement = document.getElementById('counter');
function updateCounter() {
    counterElement.textContent = parseInt(counterElement.textContent) + 1;
}
```

### 3. Использование DocumentFragment для множественных изменений

```javascript
// Плохо - множество перерисовок
const container = document.getElementById('container');
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Элемент ${i}`;
    container.appendChild(div); // Вызывает перерисовку при каждом добавлении
}

// Хорошо - одна перерисовка
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Элемент ${i}`;
    fragment.appendChild(div);
}
container.appendChild(fragment);
```

## Современные API

### Intersection Observer API

```javascript
// Для определения видимости элементов
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            console.log('Элемент стал видимым');
            // Загрузка изображения или другого контента
            loadImage(entry.target);
        }
    });
});

document.querySelectorAll('.lazy-image').forEach(img => {
    observer.observe(img);
});
```

### Resize Observer API

```javascript
// Для отслеживания изменений размера элементов
const resizeObserver = new ResizeObserver(entries => {
    for (let entry of entries) {
        console.log('Размер элемента изменился:', entry.contentRect);
        // Обновление макета или других зависимостей
    }
});

resizeObserver.observe(document.getElementById('resizable-element'));
```

## Безопасность

При работе с DOM особенно важно учитывать безопасность:

```javascript
// НЕБЕЗОПАСНО - уязвимость XSS
const userInput = '<script>alert("XSS")</script>';
element.innerHTML = userInput;

// БЕЗОПАСНО - используем textContent
element.textContent = userInput;

// Или используем шаблонизацию с экранированием
function safeHTML(htmlString) {
    const temp = document.createElement('div');
    temp.textContent = htmlString;
    return temp.innerHTML;
}
```

## Теги

#javascript #dom #api #frontend #security