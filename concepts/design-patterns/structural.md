# Структурные шаблоны проектирования

## Определение структурных шаблонов

Структурные шаблоны проектирования (structural design patterns) — это шаблоны, которые помогают определять отношения между объектами и классами, позволяя создавать более гибкие и эффективные структуры. Они фокусируются на том, как объекты составляются в более крупные структуры, сохраняя при этом гибкость и эффективность этих структур.

Структурные шаблоны обеспечивают масштабируемость и поддержку кода, позволяя легко изменять взаимодействия между компонентами системы. Эти шаблоны особенно полезны при создании сложных систем, где важно управлять зависимостями и обеспечивать слабую связанность между компонентами.

См. также: [[Creational Patterns]], [[Behavioral Patterns]]

## Адаптер (Adapter)

### Определение
Шаблон Адаптер позволяет объектам с несовместимыми интерфейсами работать вместе. Он действует как обертка между двумя интерфейсами, преобразуя интерфейс одного класса в интерфейс, ожидаемый клиентом.

### Когда использовать
- Когда нужно использовать существующий класс, но его интерфейс не соответствует требованиям
- Когда нужно создать повторно используемый класс, который совместим с различными интерфейсами
- Для интеграции сторонних библиотек без изменения их исходного кода

### Пример реализации
```javascript
class OldCalculator {
    operate(num1, num2, operation) {
        if (operation === 'add') {
            return num1 + num2;
        } else if (operation === 'sub') {
            return num1 - num2;
        }
        return 0;
    }
}

class NewCalculator {
    add(num1, num2) {
        return num1 + num2;
    }
    
    sub(num1, num2) {
        return num1 - num2;
    }
}

class CalculatorAdapter {
    constructor() {
        this.calculator = new NewCalculator();
    }
    
    operate(num1, num2, operation) {
        switch(operation) {
            case 'add':
                return this.calculator.add(num1, num2);
            case 'sub':
                return this.calculator.sub(num1, num2);
            default:
                return 0;
        }
    }
}
```

### Преимущества и недостатки
**Преимущества:**
- Повышает повторное использование существующего кода
- Позволяет интегрировать несовместимые интерфейсы
- Соблюдает принцип открытости/закрытости

**Недостатки:**
- Увеличивает сложность системы
- Может усложнить отладку
- Добавляет дополнительный уровень абстракции

## Мост (Bridge)

### Определение
Шаблон Мост разделяет абстракцию и реализацию так, чтобы они могли изменяться независимо. Он позволяет изменять реализацию без изменения клиентского кода, использующего абстракцию.

### Когда использовать
- Когда нужно избежать постоянной привязки абстракции к конкретной реализации
- Когда изменения в реализации не должны влиять на клиентский код
- Для создания платформ-независимых библиотек

### Пример реализации
```javascript
class Device {
    isEnabled() { throw new Error('Not implemented'); }
    enable() { throw new Error('Not implemented'); }
    disable() { throw new Error('Not implemented'); }
    getVolume() { throw new Error('Not implemented'); }
    setVolume(percent) { throw new Error('Not implemented'); }
}

class RemoteControl {
    constructor(device) {
        this.device = device;
    }
    
    togglePower() {
        if (this.device.isEnabled()) {
            this.device.disable();
        } else {
            this.device.enable();
        }
    }
    
    volumeUp() {
        this.device.setVolume(this.device.getVolume() + 10);
    }
    
    volumeDown() {
        this.device.setVolume(this.device.getVolume() - 10);
    }
}

class AdvancedRemoteControl extends RemoteControl {
    mute() {
        this.device.setVolume(0);
    }
}
```

## Компоновщик (Composite)

### Определение
Шаблон Компоновщик позволяет клиентскому коду обрабатывать отдельные объекты и составные структуры одинаково. Он создает древовидную структуру, где узлы могут быть как листьями, так и составными элементами.

### Когда использовать
- Когда нужно представлять иерархию "часть-целое" объектов
- Когда клиентский код должен единообразно обрабатывать простые и составные объекты
- Для рекурсивных структур данных

### Пример реализации
```javascript
class Graphic {
    draw() { throw new Error('Not implemented'); }
    add(graphic) { throw new Error('Not implemented'); }
    remove(graphic) { throw new Error('Not implemented'); }
    getChild(index) { throw new Error('Not implemented'); }
}

class Circle extends Graphic {
    constructor(color) {
        super();
        this.color = color;
    }
    
    draw() {
        console.log(`Drawing a ${this.color} circle`);
    }
}

class CompositeGraphic extends Graphic {
    constructor() {
        super();
        this.children = [];
    }
    
    add(graphic) {
        this.children.push(graphic);
    }
    
    remove(graphic) {
        const index = this.children.indexOf(graphic);
        if (index !== -1) {
            this.children.splice(index, 1);
        }
    }
    
    draw() {
        this.children.forEach(child => child.draw());
    }
}
```

## Декоратор (Decorator)

### Определение
Шаблон Декоратор позволяет динамически добавлять объектам новые функциональные возможности, оборачивая их в полезные "обертки". Это альтернатива созданию подклассов для расширения функциональности.

### Когда использовать
- Когда нужно добавлять обязанности объектам динамически и прозрачно
- Когда нельзя расширить функциональность с помощью наследования
- Для избежания создания большого количества подклассов

### Пример реализации
```javascript
class Coffee {
    getCost() { throw new Error('Not implemented'); }
    getDescription() { throw new Error('Not implemented'); }
}

class SimpleCoffee extends Coffee {
    getCost() { return 10; }
    getDescription() { return 'Simple coffee'; }
}

class CoffeeDecorator extends Coffee {
    constructor(coffee) {
        super();
        this.coffee = coffee;
    }
    
    getCost() { return this.coffee.getCost(); }
    getDescription() { return this.coffee.getDescription(); }
}

class MilkDecorator extends CoffeeDecorator {
    getCost() { return this.coffee.getCost() + 2; }
    getDescription() { return `${this.coffee.getDescription()}, milk`; }
}

class SugarDecorator extends CoffeeDecorator {
    getCost() { return this.coffee.getCost() + 1; }
    getDescription() { return `${this.coffee.getDescription()}, sugar`; }
}
```

## Фасад (Facade)

### Определение
Шаблон Фасад предоставляет упрощенный интерфейс к сложной системе классов, библиотеке или фреймворку. Он скрывает сложность системы и предоставляет клиенту простой способ доступа к функциональности.

### Когда использовать
- Когда нужно предоставить простой интерфейс к сложной подсистеме
- Для уменьшения зависимости между клиентским кодом и сложными подсистемами
- Для инкапсуляции сложной логики в одном месте

### Пример реализации
```javascript
class CPU {
    freeze() { console.log('CPU freezing...'); }
    jump(position) { console.log(`CPU jumping to ${position}`); }
    execute() { console.log('CPU executing...'); }
}

class Memory {
    load(position, data) { 
        console.log(`Loading data to memory at ${position}`);
        return data;
    }
}

class HardDrive {
    read(lba, size) { 
        console.log(`Reading from hard drive at ${lba}, size ${size}`);
        return 'OS data';
    }
}

class ComputerFacade {
    constructor() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    start() {
        console.log('Starting computer...');
        this.cpu.freeze();
        const data = this.hardDrive.read(0, 1024);
        this.memory.load(0, data);
        this.cpu.jump(0);
        this.cpu.execute();
        console.log('Computer started successfully!');
    }
}
```

## Приспособленец (Flyweight)

### Определение
Шаблон Приспособленец позволяет уменьшить потребление памяти за счет разделения общих данных между множеством мелких объектов. Он используется для эффективного использования памяти при работе с большим количеством похожих объектов.

### Когда использовать
- Когда приложение использует большое количество объектов
- Когда из-за этого возникают проблемы с памятью
- Когда объекты содержат повторяющиеся данные

### Пример реализации
```javascript
class TreeType {
    constructor(name, color, texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }
}

class TreeTypeFactory {
    constructor() {
        this.treeTypes = new Map();
    }
    
    getTreeType(name, color, texture) {
        const key = `${name}-${color}-${texture}`;
        if (!this.treeTypes.has(key)) {
            this.treeTypes.set(key, new TreeType(name, color, texture));
        }
        return this.treeTypes.get(key);
    }
}

class Tree {
    constructor(x, y, type) {
        this.x = x;
        this.y = y;
        this.type = type; // Ссылка на объект TreeType
    }
    
    draw(canvas) {
        console.log(`Drawing tree of type ${this.type.name} at (${this.x}, ${this.y})`);
    }
}
```

## Заместитель (Proxy)

### Определение
Шаблон Заместитель предоставляет суррогат или заместителя другого объекта для контроля доступа к нему. Он действует как посредник между клиентом и реальным объектом, позволяя управлять взаимодействием с ним.

### Когда использовать
- Для контроля доступа к объекту
- Для ленивой инициализации дорогостоящих объектов
- Для реализации безопасности или аутентификации
- Для кэширования результатов дорогостоящих операций

### Пример реализации
```javascript
class RealImage {
    constructor(filename) {
        this.filename = filename;
        this.loadFromDisk();
    }
    
    loadFromDisk() {
        console.log(`Loading image: ${this.filename}`);
    }
    
    display() {
        console.log(`Displaying image: ${this.filename}`);
    }
}

class ProxyImage {
    constructor(filename) {
        this.filename = filename;
        this.realImage = null;
    }
    
    display() {
        if (!this.realImage) {
            this.realImage = new RealImage(this.filename);
        }
        this.realImage.display();
    }
}
```

## Практические примеры и соображения реализации

При реализации структурных шаблонов важно учитывать следующие аспекты:

1. **Избегайте чрезмерного усложнения**: Структурные шаблоны должны упрощать, а не усложнять код
2. **Поддерживайте принципы SOLID**: Все шаблоны должны соответствовать принципам объектно-ориентированного программирования
3. **Рассматривайте производительность**: Некоторые шаблоны могут добавлять накладные расходы
4. **Документируйте использование**: Убедитесь, что другие разработчики понимают, почему используется конкретный шаблон

## Связи с другими концепциями

- [[Object-Oriented Programming]]: Структурные шаблоны строятся на принципах ООП
- [[SOLID Principles]]: Шаблоны помогают реализовать принципы SOLID
- [[Design Patterns]]: Общая категория, к которой принадлежат структурные шаблоны
- [[Architecture Patterns]]: Часто используются в архитектурных решениях

## Заключение

Структурные шаблоны проектирования играют важную роль в создании гибких, масштабируемых и поддерживаемых программных систем. Правильное применение этих шаблонов позволяет эффективно управлять отношениями между объектами и классами, обеспечивая лучшую архитектуру приложения.

#programming #design-patterns #structural-patterns #architecture #oop