# DOM (Document Object Model)

DOM (Document Object Model) - это программный интерфейс для веб-документов. Он представляет HTML или XML документ как древовидную структуру, где каждый узел является элементом документа. DOM позволяет программам и скриптам динамически получать доступ и обновлять содержимое, структуру и стиль документа.

## Структура DOM

DOM представляет HTML-документ как древовидную структуру узлов:
```
document
├── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── h1
        ├── p
        └── div
            └── span
```

## Основные методы работы с DOM

### Поиск элементов

```javascript
// Поиск по ID
const element = document.getElementById('myId');

// Поиск по тегу
const elements = document.getElementsByTagName('div');

// Поиск по классу
const elements = document.getElementsByClassName('myClass');

// Поиск с помощью CSS-селекторов
const element = document.querySelector('.myClass');
const elements = document.querySelectorAll('div > p');
```

### Создание и удаление элементов

```javascript
// Создание нового элемента
const newDiv = document.createElement('div');
newDiv.textContent = 'Новый элемент';

// Создание текстового узла
const textNode = document.createTextNode('Текстовый узел');

// Добавление элемента в документ
document.body.appendChild(newDiv);

// Удаление элемента
const elementToRemove = document.getElementById('toBeRemoved');
elementToRemove.remove();
```

### Изменение содержимого

```javascript
const element = document.getElementById('myElement');

// Изменение текстового содержимого
element.textContent = 'Новый текст';

// Изменение HTML-содержимого
element.innerHTML = '<strong>Новый HTML</strong>';

// Изменение атрибутов
element.setAttribute('class', 'newClass');
element.className = 'anotherClass';

// Изменение стилей
element.style.color = 'red';
element.style.cssText = 'color: blue; font-size: 16px;';
```

## Жизненный цикл документа

### Событие DOMContentLoaded

```javascript
document.addEventListener('DOMContentLoaded', function() {
    // Код выполнится после загрузки DOM
    console.log('DOM полностью загружен');
});
```

### Событие load

```javascript
window.addEventListener('load', function() {
    // Код выполнится после загрузки всех ресурсов
    console.log('Страница полностью загружена');
});
```

## Манипуляции с классами

```javascript
const element = document.getElementById('myElement');

// Добавление класса
element.classList.add('newClass');

// Удаление класса
element.classList.remove('oldClass');

// Переключение класса
element.classList.toggle('active');

// Проверка наличия класса
if (element.classList.contains('someClass')) {
    // делаем что-то
}
```

## Навигация по DOM

```javascript
const element = document.getElementById('myElement');

// Родительский элемент
const parent = element.parentElement;

// Дочерние элементы
const children = element.children;

// Соседние элементы
const nextSibling = element.nextElementSibling;
const prevSibling = element.previousElementSibling;

// Первый и последний дочерние элементы
const firstChild = element.firstElementChild;
const lastChild = element.lastElementChild;
```

## Лучшие практики

### 1. Использование делегирования событий

```javascript
// Плохо: добавление обработчика для каждого элемента
const buttons = document.querySelectorAll('.button');
buttons.forEach(button => {
    button.addEventListener('click', handleClick);
});

// Хорошо: делегирование событий
document.addEventListener('click', function(event) {
    if (event.target.classList.contains('button')) {
        handleClick(event);
    }
});
```

### 2. Буферизация DOM-операций

```javascript
// Плохо: множество операций DOM
const container = document.getElementById('container');
for (let i = 0; i < 100; i++) {
    container.innerHTML += `<div>Item ${i}</div>`;
}

// Хорошо: буферизация
const container = document.getElementById('container');
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    fragment.appendChild(div);
}
container.appendChild(fragment);
```

### 3. Использование современных методов

```javascript
// Используйте querySelector/querySelectorAll вместо старых методов
// Вместо getElementsByTagName лучше использовать:
const elements = document.querySelectorAll('div.myClass');
```

## Современные возможности DOM

### Intersection Observer API

```javascript
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            console.log('Элемент появился в области видимости');
        }
    });
});

observer.observe(document.getElementById('target'));
```

### Resize Observer API

```javascript
const resizeObserver = new ResizeObserver(entries => {
    for (let entry of entries) {
        console.log('Размер элемента изменился:', entry.contentRect);
    }
});

resizeObserver.observe(document.getElementById('resizable'));
```

## Промышленные примеры

### Динамическая загрузка контента

```javascript
class ContentLoader {
    constructor(containerSelector) {
        this.container = document.querySelector(containerSelector);
    }
    
    async loadContent(url) {
        try {
            const response = await fetch(url);
            const html = await response.text();
            
            // Создаем временный элемент для парсинга HTML
            const tempDiv = document.createElement('div');
            tempDiv.innerHTML = html;
            
            // Очищаем контейнер и добавляем новый контент
            this.container.innerHTML = '';
            this.container.appendChild(tempDiv);
        } catch (error) {
            console.error('Ошибка загрузки контента:', error);
        }
    }
}

// Использование
const loader = new ContentLoader('#content');
loader.loadContent('/api/content');
```

### Динамическое создание формы

```javascript
class FormBuilder {
    constructor(config) {
        this.config = config;
        this.form = document.createElement('form');
    }
    
    build() {
        this.config.fields.forEach(fieldConfig => {
            const field = this.createField(fieldConfig);
            this.form.appendChild(field);
        });
        
        return this.form;
    }
    
    createField(config) {
        const wrapper = document.createElement('div');
        wrapper.className = 'form-field';
        
        const label = document.createElement('label');
        label.textContent = config.label;
        label.htmlFor = config.id;
        
        let input;
        switch (config.type) {
            case 'text':
            case 'email':
            case 'password':
                input = document.createElement('input');
                input.type = config.type;
                break;
            case 'textarea':
                input = document.createElement('textarea');
                break;
            case 'select':
                input = document.createElement('select');
                config.options?.forEach(option => {
                    const optionEl = document.createElement('option');
                    optionEl.value = option.value;
                    optionEl.textContent = option.text;
                    input.appendChild(optionEl);
                });
                break;
            default:
                input = document.createElement('input');
                input.type = 'text';
        }
        
        input.id = config.id;
        input.name = config.name;
        input.required = config.required || false;
        
        wrapper.appendChild(label);
        wrapper.appendChild(input);
        
        return wrapper;
    }
}

// Использование
const formConfig = {
    fields: [
        { id: 'name', name: 'name', label: 'Имя', type: 'text', required: true },
        { id: 'email', name: 'email', label: 'Email', type: 'email', required: true },
        { id: 'message', name: 'message', label: 'Сообщение', type: 'textarea' }
    ]
};

const formBuilder = new FormBuilder(formConfig);
const form = formBuilder.build();
document.body.appendChild(form);
```

## Совместимость и производительность

DOM-операции могут быть дорогими, особенно при частых изменениях. Важно учитывать:

1. Минимизировать количество обращений к DOM
2. Использовать фрагменты документа при массовых операциях
3. Избегать частых перерасчетов стилей и компоновки
4. Использовать виртуальные скроллинги для больших списков
5. Учитывать производительность при работе с деревом DOM

#javascript #dom #frontend #web-development #best-practices