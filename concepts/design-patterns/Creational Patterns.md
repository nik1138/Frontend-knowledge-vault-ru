---
aliases: [Creational Design Patterns, Creation Patterns, Factory Pattern, Singleton Pattern]
tags: [design-patterns, oop, architecture, creational]
---

# Порождающие паттерны проектирования

## Обзор порождающих паттернов

Порождающие паттерны проектирования - это группа паттернов, которые занимаются процессом создания объектов. Они абстрагируют процесс инстанцирования и помогают сделать систему независимой от способа создания, композиции и представления объектов. Эти паттерны особенно полезны, когда система должна быть независимой от процесса создания объектов и когда нужно делегировать создание объектов подклассам.

## Паттерн Singleton

### Описание
Singleton гарантирует, что у класса есть только один экземпляр, и предоставляет глобальную точку доступа к нему. Это один из самых известных и часто используемых паттернов, хотя и вызывает много споров из-за потенциальных проблем с тестированием и нарушением принципов SOLID.

### Структура реализации

```java
// Java реализация Singleton с потокобезопасностью
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {
        // Приватный конструктор предотвращает создание экземпляра извне
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
    
    // Методы класса
    public void doSomething() {
        System.out.println("Выполняется действие в Singleton");
    }
}
```

```javascript
// JavaScript реализация Singleton
class Singleton {
    constructor() {
        if (Singleton.instance) {
            return Singleton.instance;
        }
        
        // Инициализация экземпляра
        this.data = [];
        
        Singleton.instance = this;
        return this;
    }
    
    addItem(item) {
        this.data.push(item);
    }
    
    getData() {
        return this.data;
    }
}

// Использование
const instance1 = new Singleton();
const instance2 = new Singleton();
console.log(instance1 === instance2); // true
```

```python
# Python реализация Singleton
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            instance = super().__call__(*args, **kwargs)
            cls._instances[cls] = instance
        return cls._instances[cls]

class Logger(metaclass=SingletonMeta):
    def __init__(self):
        self.logs = []
    
    def log(self, message):
        self.logs.append(message)
        print(f"LOG: {message}")

# Использование
logger1 = Logger()
logger2 = Logger()
print(logger1 is logger2)  # True
```

### Применение Singleton
- Логгеры
- Базы данных
- Кэши
- Настройки приложения
- Счетчики

### Преимущества и недостатки

**Преимущества:**
- Гарантированно один экземпляр
- Глобальная точка доступа
- Контролируемое хранение глобального состояния

**Недостатки:**
- Нарушение принципа единственной ответственности
- Сложности с тестированием
- Скрытые зависимости
- Проблемы с многопоточностью

## Паттерн Factory Method

### Описание
Factory Method определяет интерфейс для создания объекта, но позволяет подклассам решать, какой класс инстанцировать. Фабрика откладывает создание объектов на время выполнения.

### Структура реализации

```java
// Интерфейс продукта
interface Button {
    void render();
    void onClick();
}

// Конкретные продукты
class WindowsButton implements Button {
    public void render() {
        System.out.println("Рендеринг Windows кнопки");
    }
    
    public void onClick() {
        System.out.println("Windows кнопка нажата");
    }
}

class HTMLButton implements Button {
    public void render() {
        System.out.println("Рендеринг HTML кнопки");
    }
    
    public void onClick() {
        System.out.println("HTML кнопка нажата");
    }
}

// Базовый класс фабрики
abstract class Dialog {
    // Шаблонный метод
    public void renderWindow() {
        Button okButton = createButton();
        okButton.render();
    }
    
    // Фабричный метод
    protected abstract Button createButton();
}

// Конкретные фабрики
class WindowsDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new WindowsButton();
    }
}

class WebDialog extends Dialog {
    @Override
    protected Button createButton() {
        return new HTMLButton();
    }
}
```

```javascript
// JavaScript реализация Factory Method
class Button {
    render() {
        throw new Error('Метод render() должен быть реализован');
    }
    
    onClick() {
        throw new Error('Метод onClick() должен быть реализован');
    }
}

class WindowsButton extends Button {
    render() {
        console.log('Рендеринг Windows кнопки');
    }
    
    onClick() {
        console.log('Windows кнопка нажата');
    }
}

class HTMLButton extends Button {
    render() {
        console.log('Рендеринг HTML кнопки');
    }
    
    onClick() {
        console.log('HTML кнопка нажата');
    }
}

class Dialog {
    renderWindow() {
        const okButton = this.createButton();
        okButton.render();
    }
    
    createButton() {
        throw new Error('Метод createButton() должен быть реализован');
    }
}

class WindowsDialog extends Dialog {
    createButton() {
        return new WindowsButton();
    }
}

class WebDialog extends Dialog {
    createButton() {
        return new HTMLButton();
    }
}

// Использование
const dialog = new WindowsDialog();
dialog.renderWindow(); // Рендеринг Windows кнопки
```

### Применение Factory Method
- Когда класс не может предугадать тип создаваемых объектов
- Когда класс хочет, чтобы его подклассы указали, какие создавать объекты
- Когда родительский класс делегирует создание объектов подклассам

## Паттерн Abstract Factory

### Описание
Abstract Factory предоставляет интерфейс для создания семейств взаимосвязанных или взаимозависимых объектов, не специфицируя их конкретных классов.

### Структура реализации

```java
// Абстрактные продукты
interface Button {
    void paint();
}

interface Checkbox {
    void paint();
}

// Конкретные продукты Windows
class WindowsButton implements Button {
    public void paint() {
        System.out.println("Рендеринг Windows кнопки");
    }
}

class WindowsCheckbox implements Checkbox {
    public void paint() {
        System.out.println("Рендеринг Windows чекбокса");
    }
}

// Конкретные продукты Mac
class MacButton implements Button {
    public void paint() {
        System.out.println("Рендеринг Mac кнопки");
    }
}

class MacCheckbox implements Checkbox {
    public void paint() {
        System.out.println("Рендеринг Mac чекбокса");
    }
}

// Абстрактная фабрика
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Конкретные фабрики
class WindowsFactory implements GUIFactory {
    public Button createButton() {
        return new WindowsButton();
    }
    
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    public Button createButton() {
        return new MacButton();
    }
    
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Клиентский код
class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }
    
    public void paint() {
        this.button.paint();
        this.checkbox.paint();
    }
}
```

```python
# Python реализация Abstract Factory
from abc import ABC, abstractmethod

class Button(ABC):
    @abstractmethod
    def paint(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def paint(self):
        pass

class WindowsButton(Button):
    def paint(self):
        print("Рендеринг Windows кнопки")

class WindowsCheckbox(Checkbox):
    def paint(self):
        print("Рендеринг Windows чекбокса")

class MacButton(Button):
    def paint(self):
        print("Рендеринг Mac кнопки")

class MacCheckbox(Checkbox):
    def paint(self):
        print("Рендеринг Mac чекбокса")

class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()
    
    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()

class Application:
    def __init__(self, factory: GUIFactory):
        self.button = factory.create_button()
        self.checkbox = factory.create_checkbox()
    
    def paint(self):
        self.button.paint()
        self.checkbox.paint()

# Использование
factory = WindowsFactory()
app = Application(factory)
app.paint()
```

### Применение Abstract Factory
- Когда система не должна зависеть от того, как создаются, компонуются и представляются входящие в нее объекты
- Когда система должна конфигурироваться одним из семейств составляющих ее объектов
- Когда нужно создавать семейства связанных объектов

## Паттерн Builder

### Описание
Builder отделяет конструирование сложного объекта от его представления так, что в результате одного и того же процесса конструирования могут получаться разные представления.

### Структура реализации

```java
// Продукт
class Car {
    private String engine;
    private String body;
    private String wheels;
    private boolean gps;
    private boolean tripComputer;
    
    // Конструктор и геттеры
    public static class Builder {
        private String engine;
        private String body;
        private String wheels;
        private boolean gps = false;
        private boolean tripComputer = false;
        
        public Builder setEngine(String engine) {
            this.engine = engine;
            return this;
        }
        
        public Builder setBody(String body) {
            this.body = body;
            return this;
        }
        
        public Builder setWheels(String wheels) {
            this.wheels = wheels;
            return this;
        }
        
        public Builder setGPS(boolean gps) {
            this.gps = gps;
            return this;
        }
        
        public Builder setTripComputer(boolean tripComputer) {
            this.tripComputer = tripComputer;
            return this;
        }
        
        public Car build() {
            return new Car(this);
        }
    }
    
    private Car(Builder builder) {
        this.engine = builder.engine;
        this.body = builder.body;
        this.wheels = builder.wheels;
        this.gps = builder.gps;
        this.tripComputer = builder.tripComputer;
    }
    
    @Override
    public String toString() {
        return String.format("Car{engine='%s', body='%s', wheels='%s', gps=%s, tripComputer=%s}",
                engine, body, wheels, gps, tripComputer);
    }
}

// Использование
Car sportCar = new Car.Builder()
    .setEngine("V8")
    .setBody("Coupe")
    .setWheels("Sport")
    .setGPS(true)
    .setTripComputer(true)
    .build();
```

```javascript
// JavaScript реализация Builder
class Car {
    constructor(builder) {
        this.engine = builder.engine;
        this.body = builder.body;
        this.wheels = builder.wheels;
        this.gps = builder.gps || false;
        this.tripComputer = builder.tripComputer || false;
    }
    
    toString() {
        return `Car{engine='${this.engine}', body='${this.body}', wheels='${this.wheels}', gps=${this.gps}, tripComputer=${this.tripComputer}}`;
    }
}

class CarBuilder {
    constructor() {
        this.engine = null;
        this.body = null;
        this.wheels = null;
        this.gps = false;
        this.tripComputer = false;
    }
    
    setEngine(engine) {
        this.engine = engine;
        return this;
    }
    
    setBody(body) {
        this.body = body;
        return this;
    }
    
    setWheels(wheels) {
        this.wheels = wheels;
        return this;
    }
    
    setGPS(gps) {
        this.gps = gps;
        return this;
    }
    
    setTripComputer(tripComputer) {
        this.tripComputer = tripComputer;
        return this;
    }
    
    build() {
        return new Car(this);
    }
}

// Использование
const sportCar = new CarBuilder()
    .setEngine('V8')
    .setBody('Coupe')
    .setWheels('Sport')
    .setGPS(true)
    .setTripComputer(true)
    .build();
```

```python
# Python реализация Builder
class Car:
    def __init__(self, builder):
        self.engine = builder.engine
        self.body = builder.body
        self.wheels = builder.wheels
        self.gps = builder.gps
        self.trip_computer = builder.trip_computer

class CarBuilder:
    def __init__(self):
        self.engine = None
        self.body = None
        self.wheels = None
        self.gps = False
        self.trip_computer = False
    
    def set_engine(self, engine):
        self.engine = engine
        return self
    
    def set_body(self, body):
        self.body = body
        return self
    
    def set_wheels(self, wheels):
        self.wheels = wheels
        return self
    
    def set_gps(self, gps):
        self.gps = gps
        return self
    
    def set_trip_computer(self, trip_computer):
        self.trip_computer = trip_computer
        return self
    
    def build(self):
        return Car(self)

# Использование
sport_car = (CarBuilder()
             .set_engine("V8")
             .set_body("Coupe")
             .set_wheels("Sport")
             .set_gps(True)
             .set_trip_computer(True)
             .build())
```

### Применение Builder
- Когда нужно создавать сложные объекты пошагово
- Когда процесс создания объекта должен обеспечивать различные представления объекта
- Когда конструируемый объект должен быть immutable

## Паттерн Prototype

### Описание
Prototype позволяет копировать объекты, не вдаваясь в подробности их реализации. Он особенно полезен, когда создание объекта требует значительных ресурсов.

### Структура реализации

```java
interface Prototype {
    Prototype clone();
}

class ConcretePrototype implements Prototype {
    private String name;
    private int value;
    
    public ConcretePrototype(String name, int value) {
        this.name = name;
        this.value = value;
    }
    
    @Override
    public Prototype clone() {
        return new ConcretePrototype(this.name, this.value);
    }
    
    // Геттеры и сеттеры
    public String getName() { return name; }
    public int getValue() { return value; }
    
    @Override
    public String toString() {
        return String.format("ConcretePrototype{name='%s', value=%d}", name, value);
    }
}

// Регистр прототипов
class PrototypeRegistry {
    private Map<String, Prototype> prototypes = new HashMap<>();
    
    public void addPrototype(String key, Prototype prototype) {
        prototypes.put(key, prototype);
    }
    
    public Prototype getPrototype(String key) {
        Prototype prototype = prototypes.get(key);
        return prototype != null ? prototype.clone() : null;
    }
}
```

```javascript
// JavaScript реализация Prototype
class Prototype {
    clone() {
        throw new Error('Метод clone() должен быть реализован');
    }
}

class ConcretePrototype extends Prototype {
    constructor(name, value) {
        super();
        this.name = name;
        this.value = value;
    }
    
    clone() {
        return new ConcretePrototype(this.name, this.value);
    }
    
    toString() {
        return `ConcretePrototype{name='${this.name}', value=${this.value}}`;
    }
}

class PrototypeRegistry {
    constructor() {
        this.prototypes = new Map();
    }
    
    addPrototype(key, prototype) {
        this.prototypes.set(key, prototype);
    }
    
    getPrototype(key) {
        const prototype = this.prototypes.get(key);
        return prototype ? prototype.clone() : null;
    }
}

// Использование
const registry = new PrototypeRegistry();
registry.addPrototype('default', new ConcretePrototype('default', 0));

const cloned = registry.getPrototype('default');
console.log(cloned.toString()); // ConcretePrototype{name='default', value=0}
```

```python
# Python реализация Prototype
import copy

class Prototype:
    def clone(self):
        return copy.deepcopy(self)

class ConcretePrototype(Prototype):
    def __init__(self, name, value):
        self.name = name
        self.value = value
    
    def __str__(self):
        return f"ConcretePrototype(name='{self.name}', value={self.value})"

class PrototypeRegistry:
    def __init__(self):
        self._prototypes = {}
    
    def register(self, key, prototype):
        self._prototypes[key] = prototype
    
    def clone(self, key):
        prototype = self._prototypes.get(key)
        return prototype.clone() if prototype else None

# Использование
registry = PrototypeRegistry()
registry.register('default', ConcretePrototype('default', 0))

cloned = registry.clone('default')
print(cloned)  # ConcretePrototype(name='default', value=0)
```

### Применение Prototype
- Когда система должна быть независимой от того, как создаются, компонуются и представляются продукты
- Когда экземпляры класса могут находиться в одном из нескольких возможных состояний
- Когда создание нового экземпляра дороже клонирования

## Сравнение паттернов

| Паттерн | Цель | Когда использовать |
|---------|------|-------------------|
| Singleton | Обеспечить один экземпляр класса | Когда нужен глобальный доступ к состоянию |
| Factory Method | Определить интерфейс создания объекта | Когда подклассы решают, какой класс инстанцировать |
| Abstract Factory | Создать семейство связанных объектов | Когда нужно создавать взаимосвязанные объекты |
| Builder | Пошаговое создание сложного объекта | Когда объект требует много параметров для создания |
| Prototype | Копировать объекты | Когда создание объекта дороже клонирования |

## Практические примеры использования

### В веб-разработке
- Singleton для соединения с базой данных
- Factory для создания компонентов UI
- Builder для сложных объектов запросов/ответов

### В мобильной разработке
- Singleton для менеджера сессий
- Factory для создания элементов интерфейса
- Prototype для копирования настроек

### В enterprise приложениях
- Abstract Factory для создания платформ-независимых интерфейсов
- Builder для сложных бизнес-объектов
- Singleton для глобальных настроек

## Лучшие практики

### Общие рекомендации
- Используйте паттерны осознанно, не применяйте ради применения
- Учитывайте сложность и поддерживаемость
- Пишите тесты для проверки работы паттернов
- Документируйте использование паттернов

### Когда избегать паттернов
- Когда простое решение достаточно
- Когда паттерн усложняет код без видимой пользы
- Когда команда не знакома с паттерном
- Когда сроки проекта ограничены

## Заключение

Порождающие паттерны проектирования предоставляют гибкие решения для создания объектов в различных сценариях. Понимание и правильное применение этих паттернов позволяет создавать более модульные, тестируемые и поддерживаемые системы.

Каждый паттерн решает определенную проблему создания объектов, и выбор подходящего паттерна зависит от конкретных требований проекта. Важно понимать не только как реализовать паттерн, но и когда его использовать и когда от него лучше отказаться.

См. также:
- [[Structural Patterns]]
- [[Behavioral Patterns]]
- [[Design Principles]]
- [[SOLID Principles]]
- [[Object-Oriented Programming]]