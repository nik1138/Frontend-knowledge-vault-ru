# Порождающие паттерны проектирования

Порождающие паттерны проектирования отвечают за процесс создания объектов. Они обеспечивают гибкость в решении задач, связанных с созданием объектов, и позволяют системе быть независимой от процесса создания, композиции и представления объектов.

## Определение

Порождающие паттерны абстрагируют процесс инстанцирования. Они позволяют сделать систему независимой от способа создания, компоновки и представления объектов. Вместо жесткого кодирования классов, которые нужно создать, паттерны делегируют процесс создания экземпляров другим объектам.

## Паттерн Singleton

Паттерн Singleton гарантирует, что у класса есть только один экземпляр, и предоставляет глобальную точку доступа к нему.

### Пример реализации

```javascript
class Singleton {
    constructor() {
        if (Singleton.instance) {
            return Singleton.instance;
        }
        
        // Инициализация объекта
        this.data = [];
        
        Singleton.instance = this;
        return this;
    }
    
    addData(item) {
        this.data.push(item);
    }
    
    getData() {
        return this.data;
    }
}
```

### Когда использовать
- Когда нужен точный контроль над количеством экземпляров класса
- Когда есть глобальная точка доступа к объекту
- Для управления ресурсами, такими как соединения с базой данных или логгеры

### Преимущества
- Гарантирует наличие только одного экземпляра
- Предоставляет глобальную точку доступа
- Отложенная инициализация при первом доступе

### Недостатки
- Нарушает принцип единственной ответственности
- Сложно тестировать из-за глобального состояния
- Может создать скрытые зависимости в коде

## Паттерн Фабрика (Factory)

Паттерн Factory предоставляет интерфейс для создания экземпляров классов, позволяя подклассам решать, какой класс инстанцировать.

### Пример реализации

```javascript
class Animal {
    speak() {}
}

class Dog extends Animal {
    speak() {
        return "Woof!";
    }
}

class Cat extends Animal {
    speak() {
        return "Meow!";
    }
}

class AnimalFactory {
    static createAnimal(type) {
        switch(type) {
            case 'dog':
                return new Dog();
            case 'cat':
                return new Cat();
            default:
                throw new Error('Unknown animal type');
        }
    }
}

// Использование
const dog = AnimalFactory.createAnimal('dog');
console.log(dog.speak()); // "Woof!"
```

### Когда использовать
- Когда делегируем выбор класса для создания подклассам
- Когда неизвестно заранее, какие типы объектов будут создаваться
- Когда нужно централизовать создание объектов в одном месте

## Абстрактная фабрика (Abstract Factory)

Абстрактная фабрика предоставляет интерфейс для создания семейств связанных или зависимых объектов без указания их конкретных классов.

### Пример реализации

```javascript
// Абстрактные классы
class Button {
    render() {}
}

class Dialog {
    render() {}
}

// Конкретные реализации для Windows
class WindowsButton extends Button {
    render() {
        return "Windows button rendered";
    }
}

class WindowsDialog extends Dialog {
    render() {
        return "Windows dialog rendered";
    }
}

// Конкретные реализации для macOS
class MacButton extends Button {
    render() {
        return "Mac button rendered";
    }
}

class MacDialog extends Dialog {
    render() {
        return "Mac dialog rendered";
    }
}

// Абстрактная фабрика
class GUIFactory {
    createButton() {}
    createDialog() {}
}

class WindowsFactory extends GUIFactory {
    createButton() {
        return new WindowsButton();
    }
    
    createDialog() {
        return new WindowsDialog();
    }
}

class MacFactory extends GUIFactory {
    createButton() {
        return new MacButton();
    }
    
    createDialog() {
        return new MacDialog();
    }
}
```

### Когда использовать
- Когда система не должна зависеть от способа создания, компоновки и представления семейств продуктов
- Когда нужно создавать семейства взаимосвязанных объектов
- Когда необходима гарантия совместимости создаваемых объектов

## Паттерн Строитель (Builder)

Паттерн Builder позволяет создавать сложные объекты пошагово. Паттерн позволяет использовать один и тот же код строительства для получения разных представлений.

### Пример реализации

```javascript
class Pizza {
    constructor() {
        this.size = null;
        this.cheese = false;
        this.pepperoni = false;
        this.mushrooms = false;
    }
    
    toString() {
        return `Pizza size: ${this.size}, Cheese: ${this.cheese}, Pepperoni: ${this.pepperoni}, Mushrooms: ${this.mushrooms}`;
    }
}

class PizzaBuilder {
    constructor() {
        this.pizza = new Pizza();
    }
    
    setSize(size) {
        this.pizza.size = size;
        return this;
    }
    
    addCheese() {
        this.pizza.cheese = true;
        return this;
    }
    
    addPepperoni() {
        this.pizza.pepperoni = true;
        return this;
    }
    
    addMushrooms() {
        this.pizza.mushrooms = true;
        return this;
    }
    
    build() {
        return this.pizza;
    }
}

// Использование
const pizza = new PizzaBuilder()
    .setSize('large')
    .addCheese()
    .addPepperoni()
    .build();

console.log(pizza.toString());
```

### Когда использовать
- Когда нужно создавать сложные объекты пошагово
- Когда конечные объекты могут быть представлены разными представлениями
- Когда алгоритм создания сложного объекта не должен зависеть от частей, из которых состоит объект

## Паттерн Прототип (Prototype)

Паттерн Прототип определяет типы создаваемых объектов с помощью экземпляра-прототипа и создает новые объекты, копируя этот прототип.

### Пример реализации

```javascript
class Prototype {
    constructor(value) {
        this.value = value;
    }
    
    clone() {
        return new Prototype(this.value);
    }
}

class ConcretePrototypeA extends Prototype {
    constructor(value, extraData) {
        super(value);
        this.extraData = extraData;
    }
    
    clone() {
        return new ConcretePrototypeA(this.value, this.extraData);
    }
}

// Использование
const prototype = new ConcretePrototypeA("test", {data: 1});
const clone = prototype.clone();
```

### Когда использовать
- Когда типы создаваемых объектов определяются экземпляром-прототипом
- Когда создание новых экземпляров дешевле клонирования существующих
- Когда динамическое назначение классов создаваемых объектов происходит во время выполнения

## Рекомендации по выбору паттерна

- **Singleton**: Используйте для управления глобальным состоянием (логгеры, кэши, соединения с БД)
- **Фабрика**: Подходит, когда нужно создавать разные типы объектов в зависимости от входных данных
- **Абстрактная фабрика**: Используйте для создания семейств связанных объектов
- **Строитель**: Применяйте для пошагового создания сложных объектов
- **Прототип**: Используйте, когда копирование объекта эффективнее его создания

## Рассмотрения реализации

- Обратите внимание на потокобезопасность при реализации Singleton
- Учитывайте производительность при использовании прототипа
- Правильно используйте наследование в фабричных паттернах
- Рассмотрите использование паттернов с [[dependency injection]] для лучшей тестируемости

## Связанные концепции

- [[structural-patterns]] - структурные паттерны
- [[behavioral-patterns]] - поведенческие паттерны
- [[dependency-injection]] - внедрение зависимостей
- [[solid-principles]] - принципы SOLID

## Заключение

Порождающие паттерны проектирования играют важную роль в создании гибких и поддерживаемых приложений. Выбор правильного паттерна зависит от конкретных требований проекта и помогает избежать распространенных проблем при создании объектов.

#design-patterns #creational-patterns #programming #best-practices