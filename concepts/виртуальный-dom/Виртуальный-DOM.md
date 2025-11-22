---
aliases: [Виртуальный DOM, Virtual DOM, Реакт-рендеринг, DOM оптимизация]
tags: [frontend, virtual-dom, react, performance, rendering, diffing]
---

# Виртуальный DOM в фронтенд-разработке

## Обзор

Виртуальный DOM (Virtual DOM) - это концепция программирования, при которой в памяти создается легковесная копия реального DOM. Эта концепция используется во многих современных фреймворках, таких как React, Vue и других, для оптимизации производительности при обновлении пользовательского интерфейса.

## Основные принципы виртуального DOM

### 1. Архитектура виртуального DOM

```javascript
// Определение структуры виртуального узла
class VirtualNode {
    constructor(tag, props = {}, children = []) {
        this.tag = tag;
        this.props = props;
        this.children = children;
        this.key = props.key || null;
    }
    
    // Метод для создания реального DOM-элемента
    toDOM() {
        if (this.tag === 'TEXT') {
            return document.createTextNode(this.props.nodeValue);
        }
        
        const element = document.createElement(this.tag);
        
        // Установка атрибутов
        Object.keys(this.props).forEach(propName => {
            if (propName === 'children' || propName === 'nodeValue') return;
            
            if (propName.startsWith('on')) {
                // Обработчики событий
                const eventName = propName.slice(2).toLowerCase();
                element.addEventListener(eventName, this.props[propName]);
            } else {
                // Обычные атрибуты
                element.setAttribute(propName, this.props[propName]);
            }
        });
        
        // Добавление дочерних элементов
        this.children.forEach(child => {
            element.appendChild(child.toDOM());
        });
        
        return element;
    }
}

// Пример использования
const virtualElement = new VirtualNode('div', { class: 'container' }, [
    new VirtualNode('h1', { class: 'title' }, [
        new VirtualNode('TEXT', { nodeValue: 'Заголовок' })
    ]),
    new VirtualNode('p', { class: 'description' }, [
        new VirtualNode('TEXT', { nodeValue: 'Описание' })
    ])
]);
```

### 2. Алгоритм сравнения (Diffing Algorithm)

```javascript
// Алгоритм сравнения виртуальных деревьев
class VirtualDOMDiff {
    static diff(oldTree, newTree) {
        const patches = [];
        this.walk(oldTree, newTree, patches, 0);
        return patches;
    }
    
    static walk(oldNode, newNode, patches, index) {
        const patch = [];
        
        if (!oldNode) {
            // Узел добавлен
            patch.push({ type: 'CREATE', node: newNode });
        } else if (!newNode) {
            // Узел удален
            patch.push({ type: 'REMOVE' });
        } else if (oldNode.tag !== newNode.tag) {
            // Узел заменен
            patch.push({ type: 'REPLACE', node: newNode });
        } else if (oldNode.tag === 'TEXT') {
            // Обновление текстового узла
            if (oldNode.props.nodeValue !== newNode.props.nodeValue) {
                patch.push({ 
                    type: 'TEXT', 
                    text: newNode.props.nodeValue 
                });
            }
        } else {
            // Обновление атрибутов
            const attrPatches = this.diffProps(oldNode.props, newNode.props);
            if (attrPatches.length > 0) {
                patch.push({ type: 'ATTRS', attrs: attrPatches });
            }
            
            // Рекурсивное сравнение дочерних элементов
            this.diffChildren(
                oldNode.children, 
                newNode.children, 
                patch, 
                index
            );
        }
        
        if (patch.length > 0) {
            patches[index] = patch;
        }
    }
    
    static diffProps(oldProps, newProps) {
        const patches = [];
        
        // Обновление существующих атрибутов
        for (const key in oldProps) {
            if (!(key in newProps)) {
                patches.push({ type: 'REMOVE_ATTR', key });
            } else if (oldProps[key] !== newProps[key]) {
                patches.push({ type: 'UPDATE_ATTR', key, value: newProps[key] });
            }
        }
        
        // Добавление новых атрибутов
        for (const key in newProps) {
            if (!(key in oldProps)) {
                patches.push({ type: 'ADD_ATTR', key, value: newProps[key] });
            }
        }
        
        return patches;
    }
    
    static diffChildren(oldChildren, newChildren, patch, index) {
        const childPatches = [];
        
        // Сравнение по индексам (упрощенная стратегия)
        const count = Math.max(oldChildren.length, newChildren.length);
        
        for (let i = 0; i < count; i++) {
            const oldChild = oldChildren[i];
            const newChild = newChildren[i];
            
            if (oldChild && newChild && oldChild.key && newChild.key) {
                // Сравнение по ключу (ключевая оптимизация)
                const oldChildIndex = oldChildren.findIndex(c => c.key === newChild.key);
                if (oldChildIndex !== -1) {
                    this.walk(oldChildren[oldChildIndex], newChild, childPatches, i);
                } else {
                    // Новый элемент
                    childPatches.push({ type: 'CREATE', node: newChild });
                }
            } else {
                // Сравнение по индексу
                this.walk(oldChild, newChild, childPatches, i);
            }
        }
        
        if (childPatches.length > 0) {
            patch.push({ type: 'CHILDREN', patches: childPatches });
        }
    }
}
```

## Реализация простой системы виртуального DOM

### 1. Основной рендерер

```javascript
// Простая реализация виртуального DOM
class SimpleVirtualDOM {
    constructor() {
        this.rootDOM = null;
        this.rootVirtual = null;
    }
    
    render(virtualNode, container) {
        if (!this.rootDOM) {
            // Первый рендер
            this.rootVirtual = virtualNode;
            this.rootDOM = virtualNode.toDOM();
            container.appendChild(this.rootDOM);
        } else {
            // Обновление
            this.update(virtualNode, container);
        }
    }
    
    update(newVirtual, container) {
        const patches = VirtualDOMDiff.diff(this.rootVirtual, newVirtual);
        this.applyPatches(this.rootDOM, patches);
        this.rootVirtual = newVirtual;
    }
    
    applyPatches(node, patches) {
        if (!patches || patches.length === 0) return;
        
        patches.forEach((patch, index) => {
            if (!patch) return;
            
            patch.forEach(operation => {
                this.applyPatch(node, operation, index);
            });
        });
    }
    
    applyPatch(node, operation, index) {
        switch (operation.type) {
            case 'CREATE':
                const newNode = operation.node.toDOM();
                node.appendChild(newNode);
                break;
                
            case 'REMOVE':
                node.remove();
                break;
                
            case 'REPLACE':
                const replacementNode = operation.node.toDOM();
                node.parentNode.replaceChild(replacementNode, node);
                break;
                
            case 'TEXT':
                node.textContent = operation.text;
                break;
                
            case 'ATTRS':
                operation.attrs.forEach(attrOp => {
                    switch (attrOp.type) {
                        case 'REMOVE_ATTR':
                            node.removeAttribute(attrOp.key);
                            break;
                        case 'UPDATE_ATTR':
                        case 'ADD_ATTR':
                            node.setAttribute(attrOp.key, attrOp.value);
                            break;
                    }
                });
                break;
                
            case 'CHILDREN':
                // Рекурсивное применение патчей к дочерним элементам
                const childNodes = Array.from(node.childNodes);
                operation.patches.forEach((childPatch, childIndex) => {
                    if (childPatch && childNodes[childIndex]) {
                        this.applyPatches(childNodes[childIndex], [childPatch]);
                    }
                });
                break;
        }
    }
}

// Использование
const vdom = new SimpleVirtualDOM();
const container = document.getElementById('app');

// Первый рендер
const initialTree = new VirtualNode('div', { class: 'app' }, [
    new VirtualNode('h1', { class: 'title' }, [
        new VirtualNode('TEXT', { nodeValue: 'Добро пожаловать!' })
    ])
]);

vdom.render(initialTree, container);

// Обновление
setTimeout(() => {
    const updatedTree = new VirtualNode('div', { class: 'app updated' }, [
        new VirtualNode('h1', { class: 'title' }, [
            new VirtualNode('TEXT', { nodeValue: 'Обновленный заголовок!' })
        ]),
        new VirtualNode('p', { class: 'subtitle' }, [
            new VirtualNode('TEXT', { nodeValue: 'Новый абзац' })
        ])
    ]);
    
    vdom.render(updatedTree, container);
}, 2000);
```

### 2. JSX-подобный синтаксис

```javascript
// Функция для создания виртуальных узлов (аналог React.createElement)
function createElement(tag, props, ...children) {
    // Обработка текстовых узлов
    const processedChildren = children.flat().map(child => {
        if (typeof child === 'string' || typeof child === 'number') {
            return new VirtualNode('TEXT', { nodeValue: String(child) });
        } else if (child instanceof VirtualNode) {
            return child;
        } else {
            return new VirtualNode('TEXT', { nodeValue: String(child) });
        }
    });
    
    return new VirtualNode(tag, props || {}, processedChildren);
}

// Псевдо-JSX трансформация
const MyComponent = (props) => {
    return createElement('div', { class: 'component', key: props.id }, [
        createElement('h2', { class: 'component-title' }, props.title),
        createElement('p', { class: 'component-content' }, props.content)
    ]);
};

// Использование компонента
const componentTree = MyComponent({ 
    id: 'comp1', 
    title: 'Заголовок компонента', 
    content: 'Содержимое компонента' 
});
```

## Продвинутая реализация с компонентами

### 1. Классовые компоненты

```javascript
// Базовый класс компонента
class Component {
    constructor(props) {
        this.props = props;
        this.state = {};
        this.virtualDOM = null;
    }
    
    setState(newState) {
        this.state = { ...this.state, ...newState };
        this.update();
    }
    
    // Метод должен быть реализован в дочерних классах
    render() {
        throw new Error('Метод render() должен быть реализован');
    }
    
    // Получение виртуального дерева компонента
    getVirtualTree() {
        return this.render();
    }
    
    // Обновление компонента
    update() {
        if (this.virtualDOM) {
            const newTree = this.getVirtualTree();
            const patches = VirtualDOMDiff.diff(this.virtualDOM, newTree);
            // Здесь должен быть вызов обновления DOM
            this.virtualDOM = newTree;
        }
    }
}

// Пример использования
class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
    }
    
    render() {
        return createElement('div', { class: 'counter' }, [
            createElement('h3', {}, `Счетчик: ${this.state.count}`),
            createElement('button', { 
                onClick: () => this.setState({ count: this.state.count + 1 }) 
            }, 'Увеличить'),
            createElement('button', { 
                onClick: () => this.setState({ count: this.state.count - 1 }) 
            }, 'Уменьшить')
        ]);
    }
}

// Использование компонента
const counter = new Counter({ initialCount: 0 });
```

### 2. Функциональные компоненты с хуками

```javascript
// Реализация системы хуков
class HookSystem {
    constructor() {
        this.hooks = [];
        this.hookIndex = 0;
        this.component = null;
    }
    
    init(component) {
        this.component = component;
        this.hookIndex = 0;
    }
    
    useState(initialValue) {
        const hookIndex = this.hookIndex++;
        
        if (this.hooks[hookIndex]) {
            // Возвращаем существующее состояние
            return this.hooks[hookIndex];
        } else {
            // Инициализируем новое состояние
            const setState = (newState) => {
                if (typeof newState === 'function') {
                    this.hooks[hookIndex][0] = newState(this.hooks[hookIndex][0]);
                } else {
                    this.hooks[hookIndex][0] = newState;
                }
                
                // Перерендер компонента
                this.component.forceUpdate();
            };
            
            this.hooks[hookIndex] = [initialValue, setState];
            return this.hooks[hookIndex];
        }
    }
    
    useEffect(effect, deps) {
        const hookIndex = this.hookIndex++;
        
        if (this.hooks[hookIndex]) {
            const [prevDeps, cleanup] = this.hooks[hookIndex];
            
            if (!this.isEqual(deps, prevDeps)) {
                if (cleanup) cleanup();
                
                const newCleanup = effect();
                this.hooks[hookIndex] = [deps, newCleanup];
            }
        } else {
            const cleanup = effect();
            this.hooks[hookIndex] = [deps, cleanup];
        }
    }
    
    isEqual(a, b) {
        if (!a || !b) return a === b;
        if (a.length !== b.length) return false;
        return a.every((item, index) => item === b[index]);
    }
    
    reset() {
        this.hookIndex = 0;
    }
}

const hookSystem = new HookSystem();

// Функциональный компонент с хуками
function CounterWithHooks(props) {
    hookSystem.init(this);
    
    const [count, setCount] = hookSystem.useState(props.initialCount || 0);
    const [name, setName] = hookSystem.useState('Counter');
    
    // Эффект при изменении счетчика
    hookSystem.useEffect(() => {
        document.title = `${name}: ${count}`;
        console.log(`Счетчик обновлен до ${count}`);
        
        return () => {
            console.log('Очистка эффекта счетчика');
        };
    }, [count]);
    
    return createElement('div', { class: 'counter' }, [
        createElement('h3', {}, `${name}: ${count}`),
        createElement('button', { 
            onClick: () => setCount(c => c + 1) 
        }, 'Увеличить'),
        createElement('button', { 
            onClick: () => setCount(c => c - 1) 
        }, 'Уменьшить'),
        createElement('input', { 
            value: name,
            onInput: (e) => setName(e.target.value)
        })
    ]);
}
```

## Оптимизации виртуального DOM

### 1. Memoization компонентов

```javascript
// Декоратор для мемоизации компонентов
function memo(ComponentClass) {
    return class MemoizedComponent extends ComponentClass {
        shouldComponentUpdate(nextProps, nextState) {
            // Сравнение пропсов
            const propsChanged = Object.keys(this.props).some(
                key => this.props[key] !== nextProps[key]
            );
            
            // Сравнение состояния
            const stateChanged = Object.keys(this.state).some(
                key => this.state[key] !== nextState[key]
            );
            
            return propsChanged || stateChanged;
        }
        
        render() {
            if (this._cachedVirtualTree && !this._needsUpdate) {
                return this._cachedVirtualTree;
            }
            
            this._cachedVirtualTree = super.render();
            this._needsUpdate = false;
            return this._cachedVirtualTree;
        }
        
        setState(newState) {
            this._needsUpdate = true;
            super.setState(newState);
        }
    };
}

// Использование мемоизации
const MemoizedCounter = memo(Counter);

// Функциональный компонент с memo
function MemoizedFunctionalComponent({ value, onChange }) {
    // Простая проверка на изменения
    if (MemoizedFunctionalComponent._lastValue === value) {
        return MemoizedFunctionalComponent._cachedResult;
    }
    
    const result = createElement('div', { class: 'memoized-component' }, [
        createElement('input', { 
            value: value,
            onChange: onChange
        }),
        createElement('span', {}, `Значение: ${value}`)
    ]);
    
    MemoizedFunctionalComponent._lastValue = value;
    MemoizedFunctionalComponent._cachedResult = result;
    
    return result;
}
```

### 2. Оптимизация списка элементов

```javascript
// Оптимизированный рендеринг списков
function renderList(items, renderItem, keyExtractor) {
    const virtualNodes = items.map((item, index) => {
        const key = keyExtractor ? keyExtractor(item, index) : index;
        return renderItem(item, key);
    });
    
    return virtualNodes;
}

// Пример использования
function TodoList({ todos }) {
    return createElement('ul', { class: 'todo-list' }, 
        renderList(
            todos,
            (todo, key) => createElement('li', { 
                key: key,
                class: todo.completed ? 'completed' : ''
            }, [
                createElement('input', { 
                    type: 'checkbox',
                    checked: todo.completed,
                    onChange: () => toggleTodo(todo.id)
                }),
                createElement('span', {}, todo.text)
            ]),
            (todo) => todo.id  // Используем ID как ключ
        )
    );
}

// Алгоритм оптимизации перемещений в списках
class ListDiff {
    static diffLists(oldList, newList, keyExtractor) {
        const patches = [];
        
        // Создаем мапы для быстрого поиска
        const oldKeyMap = new Map();
        oldList.forEach((item, index) => {
            const key = keyExtractor(item, index);
            oldKeyMap.set(key, { item, index });
        });
        
        const newKeyMap = new Map();
        newList.forEach((item, index) => {
            const key = keyExtractor(item, index);
            newKeyMap.set(key, { item, index });
        });
        
        // Определяем типы изменений
        const operations = [];
        
        // Проверяем удаленные элементы
        for (const [key, oldItem] of oldKeyMap) {
            if (!newKeyMap.has(key)) {
                operations.push({ type: 'REMOVE', key, index: oldItem.index });
            }
        }
        
        // Проверяем добавленные и измененные элементы
        newList.forEach((item, newIndex) => {
            const key = keyExtractor(item, newIndex);
            const oldItem = oldKeyMap.get(key);
            
            if (!oldItem) {
                // Новый элемент
                operations.push({ type: 'INSERT', key, item, index: newIndex });
            } else if (oldItem.index !== newIndex) {
                // Перемещенный элемент
                operations.push({ type: 'MOVE', key, from: oldItem.index, to: newIndex });
            }
        });
        
        return operations;
    }
}
```

## Производительность и измерения

### 1. Инструменты измерения производительности

```javascript
// Класс для измерения производительности виртуального DOM
class VDOMPerformanceMonitor {
    constructor() {
        this.metrics = {
            renderTime: [],
            diffTime: [],
            patchTime: [],
            memoryUsage: []
        };
    }
    
    measureRender(renderFunction) {
        const start = performance.now();
        const result = renderFunction();
        const end = performance.now();
        
        this.metrics.renderTime.push(end - start);
        return result;
    }
    
    measureDiff(oldTree, newTree) {
        const start = performance.now();
        const patches = VirtualDOMDiff.diff(oldTree, newTree);
        const end = performance.now();
        
        this.metrics.diffTime.push(end - start);
        return patches;
    }
    
    measurePatch(domNode, patches) {
        const start = performance.now();
        // Применение патчей
        const result = this.applyPatches(domNode, patches);
        const end = performance.now();
        
        this.metrics.patchTime.push(end - start);
        return result;
    }
    
    getAverageMetrics() {
        return {
            avgRenderTime: this.metrics.renderTime.reduce((a, b) => a + b, 0) / this.metrics.renderTime.length,
            avgDiffTime: this.metrics.diffTime.reduce((a, b) => a + b, 0) / this.metrics.diffTime.length,
            avgPatchTime: this.metrics.patchTime.reduce((a, b) => a + b, 0) / this.metrics.patchTime.length
        };
    }
    
    // Получение метрик производительности
    getMetrics() {
        return {
            ...this.getAverageMetrics(),
            renderCount: this.metrics.renderTime.length,
            totalRenderTime: this.metrics.renderTime.reduce((a, b) => a + b, 0)
        };
    }
}

// Использование
const perfMonitor = new VDOMPerformanceMonitor();

function optimizedRender(virtualTree, container) {
    // Измерение времени рендеринга
    const domNode = perfMonitor.measureRender(() => virtualTree.toDOM());
    
    // Очистка контейнера и вставка нового узла
    container.innerHTML = '';
    container.appendChild(domNode);
}

// Использование в реальной системе
class PerformanceVDOM {
    constructor() {
        this.performanceMonitor = new VDOMPerformanceMonitor();
        this.rootDOM = null;
        this.rootVirtual = null;
    }
    
    render(virtualNode, container) {
        if (!this.rootDOM) {
            // Первый рендер
            this.rootVirtual = virtualNode;
            this.rootDOM = this.performanceMonitor.measureRender(() => virtualNode.toDOM());
            container.appendChild(this.rootDOM);
        } else {
            // Обновление с измерением
            const patches = this.performanceMonitor.measureDiff(this.rootVirtual, virtualNode);
            this.applyPatches(this.rootDOM, patches);
            this.rootVirtual = virtualNode;
        }
    }
    
    getPerformanceReport() {
        return this.performanceMonitor.getMetrics();
    }
}
```

### 2. Сравнение с прямым DOM-манипулированием

```javascript
// Пример сравнения производительности
class PerformanceComparison {
    static benchmarkDirectDOM() {
        const container = document.getElementById('test-container');
        container.innerHTML = '';
        
        const start = performance.now();
        
        // Прямое манипулирование DOM
        for (let i = 0; i < 1000; i++) {
            const div = document.createElement('div');
            div.textContent = `Элемент ${i}`;
            div.className = 'test-item';
            container.appendChild(div);
        }
        
        const end = performance.now();
        return end - start;
    }
    
    static benchmarkVirtualDOM() {
        const container = document.getElementById('test-container');
        container.innerHTML = '';
        
        const start = performance.now();
        
        // Создание виртуального DOM
        const virtualElements = [];
        for (let i = 0; i < 1000; i++) {
            virtualElements.push(
                new VirtualNode('div', { class: 'test-item' }, [
                    new VirtualNode('TEXT', { nodeValue: `Элемент ${i}` })
                ])
            );
        }
        
        const virtualContainer = new VirtualNode('div', {}, virtualElements);
        const realDOM = virtualContainer.toDOM();
        container.appendChild(realDOM);
        
        const end = performance.now();
        return end - start;
    }
    
    static runComparison(iterations = 10) {
        const directTimes = [];
        const virtualTimes = [];
        
        for (let i = 0; i < iterations; i++) {
            directTimes.push(this.benchmarkDirectDOM());
            virtualTimes.push(this.benchmarkVirtualDOM());
        }
        
        return {
            direct: {
                avg: directTimes.reduce((a, b) => a + b, 0) / iterations,
                min: Math.min(...directTimes),
                max: Math.max(...directTimes)
            },
            virtual: {
                avg: virtualTimes.reduce((a, b) => a + b, 0) / iterations,
                min: Math.min(...virtualTimes),
                max: Math.max(...virtualTimes)
            }
        };
    }
}

// Запуск сравнения
const comparison = PerformanceComparison.runComparison();
console.log('Сравнение производительности:', comparison);
```

## Практические применения и паттерны

### 1. Условный рендеринг

```javascript
// Компонент с условным рендерингом
function ConditionalComponent({ isLoggedIn, user }) {
    return createElement('div', { class: 'app' }, [
        isLoggedIn ? 
            createElement('div', { class: 'user-panel' }, [
                createElement('h2', {}, `Добро пожаловать, ${user.name}!`),
                createElement('button', { 
                    onClick: () => logout() 
                }, 'Выйти')
            ]) :
            createElement('div', { class: 'login-panel' }, [
                createElement('h2', {}, 'Пожалуйста, войдите'),
                createElement('button', { 
                    onClick: () => login() 
                }, 'Войти')
            ])
    ]);
}
```

### 2. Ленивая загрузка компонентов

```javascript
// Компонент с ленивой загрузкой
class LazyComponent extends Component {
    constructor(props) {
        super(props);
        this.state = { 
            component: null,
            loading: true,
            error: null
        };
    }
    
    async componentDidMount() {
        try {
            const module = await this.props.loader();
            this.setState({ 
                component: module.default || module,
                loading: false
            });
        } catch (error) {
            this.setState({ error, loading: false });
        }
    }
    
    render() {
        if (this.state.loading) {
            return createElement('div', { class: 'loading' }, 'Загрузка...');
        }
        
        if (this.state.error) {
            return createElement('div', { class: 'error' }, 
                `Ошибка загрузки: ${this.state.error.message}`
            );
        }
        
        if (this.state.component) {
            return createElement(this.state.component, this.props);
        }
        
        return createElement('div', { class: 'empty' }, '');
    }
}

// Использование
const LazyUserProfile = (props) => createElement(LazyComponent, {
    ...props,
    loader: () => import('./UserProfile')
});
```

## Лучшие практики

> [!tip] Использование ключей в списках
> Всегда используйте уникальные и стабильные ключи (keys) при рендеринге списков. Это позволяет алгоритму виртуального DOM эффективно определять, какие элементы изменились, были добавлены или удалены.

### 1. Правильное использование ключей

```javascript
// ПЛОХО: Использование индекса как ключа
function BadList({ items }) {
    return createElement('ul', {},
        items.map((item, index) => 
            createElement('li', { key: index }, item.name)
        )
    );
}

// ХОРОШО: Использование уникального ID
function GoodList({ items }) {
    return createElement('ul', {},
        items.map(item => 
            createElement('li', { key: item.id }, item.name)
        )
    );
}

// ПЛОХО: Генерация ключей каждый раз
function BadKeyGeneration({ items }) {
    return createElement('ul', {},
        items.map(item => 
            createElement('li', { key: Math.random() }, item.name)
        )
    );
}
```

### 2. Избегание ненужных перерендеров

```javascript
// Использование мемоизации для предотвращения ненужных рендеров
function ExpensiveComponent({ data, onUpdate }) {
    // Мемоизируем сложные вычисления
    const processedData = useMemo(() => {
        return data.map(item => ({
            ...item,
            expensiveValue: complexCalculation(item)
        }));
    }, [data]); // Пересчитываем только при изменении data
    
    // Мемоизируем обработчики
    const handleClick = useCallback((itemId) => {
        onUpdate(itemId);
    }, [onUpdate]);
    
    return createElement('div', {},
        processedData.map(item => 
            createElement('div', { 
                key: item.id,
                onClick: () => handleClick(item.id)
            }, item.expensiveValue)
        )
    );
}
```

## Связанные концепции

- [[Производительность]] - как виртуальный DOM влияет на производительность
- [[Реактивность]] - связь с реактивными системами
- [[Алгоритмы]] - алгоритмы сравнения и оптимизации
- [[React]] - реализация виртуального DOM в React
- [[Оптимизация-рендеринга]] - техники оптимизации рендеринга
- [[Функции-высшего-порядка]] - паттерны для компонентов