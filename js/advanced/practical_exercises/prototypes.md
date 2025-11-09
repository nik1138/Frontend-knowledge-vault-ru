# Упражнения по прототипам и наследованию

## Упражнение 1: Создание иерархии классов
Создайте иерархию классов для системы управления транспортными средствами:

```javascript
// Задание: Создайте иерархию классов для транспортных средств
// 1. Базовый класс Vehicle
// 2. Классы Car и Motorcycle, наследующие от Vehicle
// 3. Класс Truck, наследующий от Car
// 4. Используйте прототипы и современный синтаксис классов

// Базовый класс с использованием прототипов
function VehiclePrototype(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
  this.isRunning = false;
}

VehiclePrototype.prototype.start = function() {
  this.isRunning = true;
  console.log(`${this.make} ${this.model} запущен`);
};

VehiclePrototype.prototype.stop = function() {
  this.isRunning = false;
  console.log(`${this.make} ${this.model} остановлен`);
};

VehiclePrototype.prototype.getInfo = function() {
  return `${this.year} ${this.make} ${this.model}`;
};

// Современный синтаксис классов
class Vehicle {
  constructor(make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
    this.isRunning = false;
  }
  
  start() {
    this.isRunning = true;
    console.log(`${this.make} ${this.model} запущен`);
  }
  
  stop() {
    this.isRunning = false;
    console.log(`${this.make} ${this.model} остановлен`);
  }
  
  getInfo() {
    return `${this.year} ${this.make} ${this.model}`;
  }
}

// Наследование с использованием прототипов
function CarPrototype(make, model, year, doors) {
  VehiclePrototype.call(this, make, model, year);
  this.doors = doors;
}

CarPrototype.prototype = Object.create(VehiclePrototype.prototype);
CarPrototype.prototype.constructor = CarPrototype;

CarPrototype.prototype.honk = function() {
  console.log("Бип-бип!");
};

// Современное наследование
class Car extends Vehicle {
  constructor(make, model, year, doors) {
    super(make, model, year);
    this.doors = doors;
  }
  
  honk() {
    console.log("Бип-бип!");
  }
  
  getInfo() {
    return `${super.getInfo()} с ${this.doors} дверьми`;
  }
}

class Motorcycle extends Vehicle {
  constructor(make, model, year, engineSize) {
    super(make, model, year);
    this.engineSize = engineSize; // в см³
  }
  
  wheelie() {
    console.log("Мотоцикл едет на заднем колесе!");
  }
  
  getInfo() {
    return `${super.getInfo()} с двигателем ${this.engineSize} см³`;
  }
}

class Truck extends Car {
  constructor(make, model, year, doors, capacity) {
    super(make, model, year, doors);
    this.capacity = capacity; // в тоннах
  }
  
  load(weight) {
    if (weight <= this.capacity * 1000) {
      console.log(`Груз весом ${weight} кг загружен`);
    } else {
      console.log(`Превышена грузоподъемность! Максимум ${this.capacity * 1000} кг`);
    }
  }
  
  getInfo() {
    return `${super.getInfo()} с грузоподъемностью ${this.capacity} тонн`;
  }
}

// Тестирование
const car = new Car("Toyota", "Camry", 2022, 4);
const motorcycle = new Motorcycle("Harley-Davidson", "Street 750", 2021, 749);
const truck = new Truck("Volvo", "FH16", 2023, 2, 20);

console.log(car.getInfo());
car.start();
car.honk();

console.log(motorcycle.getInfo());
motorcycle.start();
motorcycle.wheelie();

console.log(truck.getInfo());
truck.start();
truck.load(15000);
truck.load(25000);
```

#javascript #prototypes #inheritance #classes #advanced