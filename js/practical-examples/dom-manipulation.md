---
aliases: ["Работа с DOM", "Манипуляции с DOM", "DOM элементы"]
tags: [js, dom, frontend, practical-examples]
---

# Работа с DOM элементами

## Поиск элементов

### Классические методы поиска

```javascript
// Поиск по ID
const element = document.getElementById('myId');

// Поиск по классу (возвращает HTMLCollection)
const elements = document.getElementsByClassName('myClass');

// Поиск по тегу (возвращает HTMLCollection)
const divs = document.getElementsByTagName('div');

// Поиск по CSS селектору (возвращает первый элемент)
const element = document.querySelector('.myClass #myId');

// Поиск по CSS селектору (возвращает NodeList)
const elements = document.querySelectorAll('div.myClass');
```

### Современные методы поиска

```javascript
// Поиск элементов с дополнительными условиями
const visibleElements = Array.from(document.querySelectorAll('.item'))
  .filter(el => el.offsetParent !== null); // Только видимые элементы

// Поиск по data-атрибутам
const buttons = document.querySelectorAll('[data-action="submit"]');

// Поиск дочерних элементов
const parent = document.querySelector('.parent');
const children = parent.querySelectorAll('.child');
```

## Модификация элементов

### Изменение содержимого

```javascript
// Изменение текста
const element = document.querySelector('#title');
element.textContent = 'Новый заголовок';
element.innerHTML = '<strong>Новый заголовок с HTML</strong>';

// Изменение атрибутов
element.setAttribute('data-status', 'active');
element.removeAttribute('data-status');
element.id = 'newId';

// Работа с классами
element.classList.add('active');
element.classList.remove('inactive');
element.classList.toggle('highlighted');
element.classList.contains('active'); // проверка наличия класса
```

### Создание и удаление элементов

```javascript
// Создание новых элементов
const newDiv = document.createElement('div');
newDiv.className = 'new-element';
newDiv.textContent = 'Новый элемент';

// Добавление элемента в DOM
document.body.appendChild(newDiv);

// Создание элемента с атрибутами
const link = document.createElement('a');
link.href = 'https://example.com';
link.textContent = 'Ссылка';
link.target = '_blank';

// Более эффективный способ создания элементов с помощью шаблонов
const template = document.createElement('template');
template.innerHTML = `
  <div class="card">
    <h3>Заголовок карточки</h3>
    <p>Описание карточки</p>
    <button>Кнопка</button>
  </div>
`;

const card = template.content.cloneNode(true);
document.body.appendChild(card);
```

### Изменение стилей

```javascript
// Прямое изменение стилей
element.style.color = 'red';
element.style.backgroundColor = '#fff';
element.style.setProperty('border-radius', '5px');

// Добавление CSS класса для стилизации
element.classList.add('highlighted');

// Удаление стилей
element.style.removeProperty('color');
```

## События

### Добавление обработчиков событий

```javascript
// Стандартный способ
const button = document.querySelector('#myButton');
button.addEventListener('click', function(event) {
  console.log('Кнопка нажата');
  console.log(event.target); // элемент, на котором произошло событие
});

// События с передачей данных
button.addEventListener('click', (event) => {
  const action = event.target.dataset.action;
  console.log(`Выполнено действие: ${action}`);
});

// События с параметрами
button.addEventListener('click', (event) => {
  event.preventDefault(); // предотвращение стандартного поведения
  event.stopPropagation(); // остановка всплытия события
});
```

### Распространенные события

```javascript
// События мыши
element.addEventListener('click', handler);
element.addEventListener('dblclick', handler);
element.addEventListener('mouseenter', handler);
element.addEventListener('mouseleave', handler);
element.addEventListener('mousedown', handler);
element.addEventListener('mouseup', handler);

// События клавиатуры
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') {
    closeModal();
  }
});

// События формы
const input = document.querySelector('#textInput');
input.addEventListener('input', (e) => {
  console.log('Текст изменился:', e.target.value);
});

input.addEventListener('focus', (e) => {
  console.log('Поле получило фокус');
});

input.addEventListener('blur', (e) => {
  console.log('Поле потеряло фокус');
});

// События документа
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM полностью загружен');
});

window.addEventListener('load', () => {
  console.log('Все ресурсы загружены');
});

window.addEventListener('resize', () => {
  console.log('Размер окна изменен');
});
```

### Делегирование событий

```javascript
// Эффективный способ обработки событий для динамических элементов
document.querySelector('#container').addEventListener('click', (event) => {
  if (event.target.matches('.delete-btn')) {
    // Обработка клика по кнопке удаления
    event.target.closest('.item').remove();
  } else if (event.target.matches('.edit-btn')) {
    // Обработка клика по кнопке редактирования
    editItem(event.target.closest('.item'));
  }
});
```

## Практические примеры

### Динамическое создание списка

```javascript
function createList(items, containerId) {
  const container = document.getElementById(containerId);
  
  // Очищаем контейнер
  container.innerHTML = '';
  
  // Создаем элементы списка
  items.forEach(item => {
    const li = document.createElement('li');
    li.className = 'list-item';
    li.textContent = item.text;
    
    // Добавляем обработчик клика
    li.addEventListener('click', () => {
      li.classList.toggle('selected');
    });
    
    container.appendChild(li);
  });
}

// Использование
const items = [
  { text: 'Элемент 1' },
  { text: 'Элемент 2' },
  { text: 'Элемент 3' }
];

createList(items, 'list-container');
```

### Модальное окно

```javascript
class Modal {
  constructor(modalId) {
    this.modal = document.getElementById(modalId);
    this.overlay = this.modal.querySelector('.modal-overlay');
    this.closeBtn = this.modal.querySelector('.modal-close');
    this.init();
  }
  
  init() {
    this.overlay.addEventListener('click', () => this.close());
    this.closeBtn.addEventListener('click', () => this.close());
    
    // Закрытие по клавише Escape
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape' && this.isOpen()) {
        this.close();
      }
    });
  }
  
  open() {
    this.modal.classList.add('active');
    document.body.style.overflow = 'hidden';
  }
  
  close() {
    this.modal.classList.remove('active');
    document.body.style.overflow = '';
  }
  
  isOpen() {
    return this.modal.classList.contains('active');
  }
}

// Использование
const modal = new Modal('myModal');
```

### Drag and Drop

```javascript
function initDragAndDrop(containerSelector) {
  const container = document.querySelector(containerSelector);
  
  container.querySelectorAll('.draggable').forEach(item => {
    item.draggable = true;
    
    item.addEventListener('dragstart', (e) => {
      e.dataTransfer.setData('text/plain', e.target.id);
      e.target.classList.add('dragging');
    });
    
    item.addEventListener('dragend', (e) => {
      e.target.classList.remove('dragging');
    });
  });
  
  container.addEventListener('dragover', (e) => {
    e.preventDefault();
  });
  
  container.addEventListener('drop', (e) => {
    e.preventDefault();
    const draggedId = e.dataTransfer.getData('text/plain');
    const draggedElement = document.getElementById(draggedId);
    const dropTarget = e.target.closest('.droppable');
    
    if (dropTarget) {
      dropTarget.appendChild(draggedElement);
    }
  });
}
```

## Лучшие практики

### Оптимизация производительности

```javascript
// Использование DocumentFragment для множественных изменений
function addMultipleElements(data) {
  const fragment = document.createDocumentFragment();
  
  data.forEach(item => {
    const element = document.createElement('div');
    element.textContent = item.text;
    fragment.appendChild(element);
  });
  
  document.querySelector('#container').appendChild(fragment);
}

// Дебаунсинг для событий, вызываемых часто
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

const debouncedInputHandler = debounce((e) => {
  console.log('Поиск:', e.target.value);
}, 300);

document.querySelector('#searchInput').addEventListener('input', debouncedInputHandler);
```

### Управление памятью

```javascript
// Удаление обработчиков событий при удалении элементов
function removeElementWithHandlers(element) {
  // Удаление всех обработчиков событий
  const clone = element.cloneNode(true);
  element.parentNode.replaceChild(clone, element);
}

// Использование WeakMap для хранения данных элементов
const elementData = new WeakMap();

function setElementData(element, data) {
  elementData.set(element, data);
}

function getElementData(element) {
  return elementData.get(element);
}
```

## Ссылки по теме

- [[js/dom/events]] - События DOM
- [[js/dom/manipulation]] - Манипуляции с DOM
- [[js/browser/api]] - Browser API
- [[js/patterns/observer]] - Паттерн Наблюдатель