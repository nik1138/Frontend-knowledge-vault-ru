---
aliases: ["Виртуальный DOM реализация", "Реализация виртуального DOM", "Создание Virtual DOM"]
tags: [frontend, virtual-dom, javascript, implementation]
---

# Реализация Virtual DOM

## Введение

Virtual DOM (виртуальный DOM) - это концепция программирования, при которой виртуальное представление UI хранится в памяти и синхронизируется с реальным DOM при помощи библиотеки, такой как [[Виртуальный-DOM]]. Virtual DOM позволяет оптимизировать обновления интерфейса, минимизируя количество дорогостоящих операций с реальным DOM.

## Архитектура Virtual DOM

Каждая реализация Virtual DOM включает в себя:

1. **VNode (Virtual Node)** - объект, представляющий узел DOM
2. **Diffing алгоритм** - механизм сравнения изменений между деревьями
3. **Patching (патчинг)** - применение различий к реальному DOM

## Базовая реализация VNode

```javascript
class VNode {
  constructor(tag, props, children) {
    this.tag = tag;
    this.props = props;
    this.children = children;
    this.key = props?.key || null;
  }

  // Создание VNode из реального DOM-элемента
  static fromDOM(element) {
    const tag = element.tagName.toLowerCase();
    const props = {};
    
    // Собираем атрибуты
    for (let attr of element.attributes) {
      props[attr.name] = attr.value;
    }
    
    // Собираем дочерние элементы
    const children = [];
    for (let child of element.childNodes) {
      if (child.nodeType === Node.ELEMENT_NODE) {
        children.push(VNode.fromDOM(child));
      } else if (child.nodeType === Node.TEXT_NODE) {
        children.push(child.textContent);
      }
    }
    
    return new VNode(tag, props, children);
  }
}
```

## Основные методы для работы с VNode

```javascript
// Рендер VNode в реальный DOM
function render(vnode) {
  if (typeof vnode === 'string') {
    return document.createTextNode(vnode);
  }

  const element = document.createElement(vnode.tag);
  
  // Установка атрибутов
  for (let [key, value] of Object.entries(vnode.props || {})) {
    if (key.startsWith('on')) {
      // Обработка обработчиков событий
      const event = key.slice(2).toLowerCase();
      element.addEventListener(event, value);
    } else if (key === 'className') {
      element.className = value;
    } else if (key === 'style') {
      // Поддержка объекта стилей
      if (typeof value === 'object') {
        Object.assign(element.style, value);
      } else {
        element.style.cssText = value;
      }
    } else {
      element.setAttribute(key, value);
    }
  }

  // Рекурсивный рендер дочерних элементов
  if (Array.isArray(vnode.children)) {
    for (let child of vnode.children) {
      element.appendChild(render(child));
    }
  }

  return element;
}
```

## Алгоритм обновления (Diffing)

```javascript
// Сравнение двух VNode и обновление реального DOM
function patch(oldVNode, newVNode, parentElement) {
  if (!oldVNode && newVNode) {
    // Добавление нового узла
    parentElement.appendChild(render(newVNode));
  } else if (oldVNode && !newVNode) {
    // Удаление узла
    parentElement.removeChild(parentElement.lastChild);
  } else if (!sameVNode(oldVNode, newVNode)) {
    // Замена узла
    parentElement.replaceChild(render(newVNode), findRealDOM(oldVNode, parentElement));
  } else if (oldVNode.tag) {
    // Обновление узла
    updateElement(findRealDOM(oldVNode, parentElement), oldVNode, newVNode);
  }
}

// Проверка, являются ли VNode одинаковыми
function sameVNode(oldVNode, newVNode) {
  return oldVNode.tag === newVNode.tag && oldVNode.key === newVNode.key;
}

// Обновление атрибутов и дочерних элементов
function updateElement(element, oldVNode, newVNode) {
  // Обновление атрибутов
  const oldProps = oldVNode.props || {};
  const newProps = newVNode.props || {};

  // Удаление старых атрибутов
  for (let key in oldProps) {
    if (!(key in newProps)) {
      if (key.startsWith('on')) {
        element.removeEventListener(key.slice(2).toLowerCase(), oldProps[key]);
      } else {
        element.removeAttribute(key);
      }
    }
  }

  // Добавление/обновление новых атрибутов
  for (let key in newProps) {
    if (oldProps[key] !== newProps[key]) {
      if (key.startsWith('on')) {
        element.removeEventListener(key.slice(2).toLowerCase(), oldProps[key]);
        element.addEventListener(key.slice(2).toLowerCase(), newProps[key]);
      } else if (key === 'className') {
        element.className = newProps[key];
      } else if (key === 'style') {
        if (typeof newProps[key] === 'object') {
          Object.assign(element.style, newProps[key]);
        } else {
          element.style.cssText = newProps[key];
        }
      } else {
        element.setAttribute(key, newProps[key]);
      }
    }
  }

  // Обновление дочерних элементов
  updateChildren(element, oldVNode.children, newVNode.children);
}
```

## Обновление дочерних элементов

```javascript
function updateChildren(parentElement, oldChildren, newChildren) {
  const oldLength = oldChildren.length;
  const newLength = newChildren.length;
  const maxLength = Math.max(oldLength, newLength);

  for (let i = 0; i < maxLength; i++) {
    const oldVNode = oldChildren[i];
    const newVNode = newChildren[i];

    if (i >= oldLength) {
      // Добавление новых элементов
      parentElement.appendChild(render(newVNode));
    } else if (i >= newLength) {
      // Удаление лишних элементов
      parentElement.removeChild(findRealDOM(oldVNode, parentElement));
    } else {
      // Обновление существующих элементов
      patch(oldVNode, newVNode, parentElement);
    }
  }
}

// Вспомогательная функция для поиска реального DOM-элемента
function findRealDOM(vnode, parentElement) {
  // В реальной реализации может потребоваться более сложная логика
  // для точного соответствия vnode реальному DOM-элементу
  return parentElement.lastChild;
}
```

## Пример использования

```javascript
// Создание виртуального DOM-дерева
const vdom = new VNode('div', { className: 'container' }, [
  new VNode('h1', { id: 'title' }, ['Заголовок']),
  new VNode('p', {}, ['Текст параграфа']),
  new VNode('button', { 
    onClick: () => console.log('Кнопка нажата'),
    className: 'btn'
  }, ['Нажми меня'])
]);

// Рендер в реальный DOM
const realDOM = render(vdom);
document.body.appendChild(realDOM);

// Обновление DOM
const updatedVDOM = new VNode('div', { className: 'container' }, [
  new VNode('h1', { id: 'title' }, ['Новый заголовок']),
  new VNode('p', {}, ['Обновленный текст параграфа']),
  new VNode('button', { 
    onClick: () => console.log('Кнопка обновлена'),
    className: 'btn btn-primary'
  }, ['Нажми меня'])
]);

patch(vdom, updatedVDOM, document.body.lastChild);
```

## Оптимизации и ключи

Ключи (keys) играют важную роль в оптимизации процесса обновления:

```javascript
// Использование ключей для идентификации элементов
const listVDOM = new VNode('ul', {}, [
  new VNode('li', { key: 'item-1' }, ['Элемент 1']),
  new VNode('li', { key: 'item-2' }, ['Элемент 2']),
  new VNode('li', { key: 'item-3' }, ['Элемент 3'])
]);

// При изменении порядка элементов с ключами Virtual DOM может
// эффективно переместить элементы вместо полной перерисовки
const updatedListVDOM = new VNode('ul', {}, [
  new VNode('li', { key: 'item-3' }, ['Элемент 3']),
  new VNode('li', { key: 'item-1' }, ['Элемент 1']),
  new VNode('li', { key: 'item-2' }, ['Элемент 2'])
]);
```

## Заключение

Реализация Virtual DOM требует понимания основных концепций: VNode, diffing-алгоритма и patching-процесса. Эффективная реализация может значительно улучшить производительность приложений за счет минимизации обновлений реального DOM.

См. также:
- [[Diffing-алгоритмы]]
- [[Virtual-DOM-в-React]]
- [[Virtual-DOM-в-Vue]]
- [[Виртуальный-DOM]]