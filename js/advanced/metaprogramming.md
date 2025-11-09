# Метапрограммирование в JavaScript: Proxy и Reflect

## Введение

Метапрограммирование - это техника программирования, при которой программы обладают способностью обрабатывать другие программы как данные. В JavaScript метапрограммирование стало возможным благодаря введению объектов Proxy и Reflect в ES6.

Proxy позволяет перехватывать и переопределять операции над объектами, а Reflect предоставляет методы для выполнения тех же операций, что и операторы JavaScript, но в виде функций.

## Proxy

Proxy - это объект, который используется для определения пользовательского поведения для фундаментальных операций (например, поиск свойства, присваивание, перечисление, вызов функции и т.д.).

### Основы Proxy

```javascript
// Создание Proxy
const target = {
  name: "Иван",
  age: 30
};

const handler = {
  // Перехват чтения свойства
  get(target, property) {
    console.log(`Чтение свойства: ${property}`);
    return target[property];
  },
  
  // Перехват записи свойства
  set(target, property, value) {
    console.log(`Запись свойства: ${property} = ${value}`);
    target[property] = value;
    return true; // Указывает, что присваивание прошло успешно
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name);  // Чтение свойства: name \n Иван
proxy.age = 31;           // Запись свойства: age = 31
```

### Ловушки (Traps) Proxy

Proxy поддерживает множество ловушек для перехвата различных операций:

```javascript
const handler = {
  // Перехват чтения свойства
  get(target, property, receiver) {
    console.log(`get: ${property}`);
    return Reflect.get(target, property, receiver);
  },
  
  // Перехват записи свойства
  set(target, property, value, receiver) {
    console.log(`set: ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  },
  
  // Перехват проверки наличия свойства (in оператор)
  has(target, property) {
    console.log(`has: ${property}`);
    return Reflect.has(target, property);
  },
  
  // Перехват удаления свойства
  deleteProperty(target, property) {
    console.log(`delete: ${property}`);
    return Reflect.deleteProperty(target, property);
  },
  
  // Перехват получения дескриптора свойства
  getOwnPropertyDescriptor(target, property) {
    console.log(`getOwnPropertyDescriptor: ${property}`);
    return Reflect.getOwnPropertyDescriptor(target, property);
  },
  
  // Перехват определения нового свойства
  defineProperty(target, property, descriptor) {
    console.log(`defineProperty: ${property}`);
    return Reflect.defineProperty(target, property, descriptor);
  },
  
  // Перехват получения прототипа
  getPrototypeOf(target) {
    console.log('getPrototypeOf');
    return Reflect.getPrototypeOf(target);
  },
  
  // Перехват установки прототипа
  setPrototypeOf(target, prototype) {
    console.log('setPrototypeOf');
    return Reflect.setPrototypeOf(target, prototype);
  },
  
  // Перехват проверки расширяемости объекта
  isExtensible(target) {
    console.log('isExtensible');
    return Reflect.isExtensible(target);
  },
  
  // Перехват предотвращения расширения объекта
  preventExtensions(target) {
    console.log('preventExtensions');
    return Reflect.preventExtensions(target);
  },
  
  // Перехват получения ключей объекта
  ownKeys(target) {
    console.log('ownKeys');
    return Reflect.ownKeys(target);
  },
  
  // Перехват вызова функции
  apply(target, thisArg, argumentsList) {
    console.log('apply');
    return Reflect.apply(target, thisArg, argumentsList);
  },
  
  // Перехват создания экземпляра через new
  construct(target, argumentsList, newTarget) {
    console.log('construct');
    return Reflect.construct(target, argumentsList, newTarget);
  }
};
```

### Практические примеры использования Proxy

#### 1. Валидация свойств

```javascript
function createValidatedObject(schema) {
  return new Proxy({}, {
    set(target, property, value) {
      // Проверка наличия свойства в схеме
      if (!(property in schema)) {
        throw new Error(`Свойство "${property}" не разрешено`);
      }
      
      // Проверка типа значения
      const expectedType = schema[property];
      if (typeof value !== expectedType) {
        throw new Error(`Свойство "${property}" должно быть типа ${expectedType}`);
      }
      
      target[property] = value;
      return true;
    }
  });
}

const userSchema = {
  name: 'string',
  age: 'number',
  active: 'boolean'
};

const user = createValidatedObject(userSchema);

user.name = "Иван";     // OK
user.age = 30;          // OK
user.active = true;     // OK

// user.email = "test@example.com";  // Ошибка: Свойство "email" не разрешено
// user.age = "тридцать";            // Ошибка: Свойство "age" должно быть типа number
```

#### 2. Создание "умных" объектов с логированием

```javascript
function createLoggedObject(obj, logger = console.log) {
  return new Proxy(obj, {
    get(target, property, receiver) {
      logger(`GET: ${String(property)}`);
      return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
      logger(`SET: ${String(property)} = ${value}`);
      return Reflect.set(target, property, value, receiver);
    },
    
    deleteProperty(target, property) {
      logger(`DELETE: ${String(property)}`);
      return Reflect.deleteProperty(target, property);
    }
  });
}

const obj = createLoggedObject({ name: "Иван", age: 30 });

console.log(obj.name);  // GET: name \n Иван
obj.age = 31;           // SET: age = 31
delete obj.name;        // DELETE: name
```

#### 3. Реализация "приватных" свойств

```javascript
function createPrivateObject(obj) {
  const privateProperties = new WeakMap();
  privateProperties.set(obj, {});
  
  return new Proxy(obj, {
    get(target, property, receiver) {
      // Проверка, является ли свойство приватным (начинается с #)
      if (typeof property === 'string' && property.startsWith('#')) {
        const privateObj = privateProperties.get(target);
        return privateObj[property];
      }
      
      return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
      // Проверка, является ли свойство приватным
      if (typeof property === 'string' && property.startsWith('#')) {
        const privateObj = privateProperties.get(target);
        privateObj[property] = value;
        return true;
      }
      
      return Reflect.set(target, property, value, receiver);
    }
  });
}

const obj = createPrivateObject({ publicProp: "публичное" });
obj.publicProp = "новое публичное";
obj.#privateProp = "приватное";

console.log(obj.publicProp);  // "новое публичное"
console.log(obj.#privateProp); // "приватное"
```

#### 4. Кэширование вычислений

```javascript
function createCachedObject(obj) {
  const cache = new Map();
  
  return new Proxy(obj, {
    get(target, property, receiver) {
      // Если свойство - функция, создаем кэширующую обертку
      const value = Reflect.get(target, property, receiver);
      
      if (typeof value === 'function') {
        return new Proxy(value, {
          apply(func, thisArg, argumentsList) {
            const cacheKey = `${property}(${JSON.stringify(argumentsList)})`;
            
            if (cache.has(cacheKey)) {
              console.log(`Кэш: ${cacheKey}`);
              return cache.get(cacheKey);
            }
            
            const result = Reflect.apply(func, thisArg, argumentsList);
            cache.set(cacheKey, result);
            console.log(`Вычисление: ${cacheKey}`);
            return result;
          }
        });
      }
      
      return value;
    }
  });
}

const mathObj = createCachedObject({
  factorial(n) {
    if (n <= 1) return 1;
    return n * this.factorial(n - 1);
  },
  
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
});

console.log(mathObj.factorial(5));  // Вычисление: factorial([5]) \n 120
console.log(mathObj.factorial(5));  // Кэш: factorial([5]) \n 120
console.log(mathObj.factorial(6));  // Вычисление: factorial([6]) \n 720
```

## Reflect

Reflect - это встроенный объект, который предоставляет методы для перехватываемых JavaScript операций. В отличие от большинства глобальных объектов, Reflect не является конструктором и не может быть вызван или создан с помощью new.

### Основы Reflect

```javascript
// Reflect методы соответствуют внутренним методам объектов
const obj = { name: "Иван" };

// Эти вызовы эквивалентны
console.log(obj.name);                    // "Иван"
console.log(Reflect.get(obj, 'name'));    // "Иван"

obj.age = 30;                             // undefined
Reflect.set(obj, 'age', 30);              // true

console.log('name' in obj);               // true
console.log(Reflect.has(obj, 'name'));    // true

delete obj.age;                           // true
console.log(Reflect.deleteProperty(obj, 'age')); // true
```

### Использование Reflect с Proxy

```javascript
// Использование Reflect в обработчиках Proxy для перенаправления операций
const target = { name: "Иван" };

const handler = {
  get(target, property, receiver) {
    console.log(`Перехват чтения: ${property}`);
    // Используем Reflect для выполнения стандартной операции
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    console.log(`Перехват записи: ${property} = ${value}`);
    // Используем Reflect для выполнения стандартной операции
    return Reflect.set(target, property, value, receiver);
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name);  // Перехват чтения: name \n Иван
proxy.age = 30;           // Перехват записи: age = 30
```

### Продвинутое использование Reflect

#### 1. Конструкторы с Reflect.construct

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

// Создание экземпляра с помощью Reflect.construct
const user1 = Reflect.construct(User, ["Иван", "ivan@example.com"]);
console.log(user1);  // User { name: "Иван", email: "ivan@example.com" }

// Создание экземпляра с другим конструктором
class Admin extends User {
  constructor(name, email, permissions) {
    super(name, email);
    this.permissions = permissions;
  }
}

const admin = Reflect.construct(User, ["Админ", "admin@example.com"], Admin);
console.log(admin instanceof Admin);  // true
console.log(admin instanceof User);   // true
```

#### 2. Определение свойств с Reflect.defineProperty

```javascript
const obj = {};

// Определение свойства с дескриптором
const success = Reflect.defineProperty(obj, 'name', {
  value: 'Иван',
  writable: false,
  enumerable: true,
  configurable: false
});

console.log(success);  // true
console.log(obj.name); // "Иван"

// Попытка изменения неизменяемого свойства
try {
  obj.name = "Мария";
} catch (error) {
  console.log("Свойство не изменяемо");
}
console.log(obj.name); // "Иван"
```

## Практические паттерны

### 1. Observable объекты

```javascript
function createObservable(target, observer) {
  return new Proxy(target, {
    set(target, property, value, receiver) {
      const oldValue = target[property];
      const result = Reflect.set(target, property, value, receiver);
      
      // Уведомляем наблюдателя об изменении
      if (oldValue !== value) {
        observer(property, value, oldValue);
      }
      
      return result;
    }
  });
}

const obj = createObservable({ name: "Иван" }, (property, newValue, oldValue) => {
  console.log(`Свойство ${property} изменилось с ${oldValue} на ${newValue}`);
});

obj.name = "Мария";  // Свойство name изменилось с Иван на Мария
```

### 2. Автозаполнение объектов

```javascript
function createAutoPopulate(target) {
  return new Proxy(target, {
    get(target, property, receiver) {
      // Если свойство не существует, создаем пустой объект
      if (!(property in target)) {
        target[property] = {};
      }
      
      return Reflect.get(target, property, receiver);
    }
  });
}

const obj = createAutoPopulate({});

// Автоматическое создание вложенных объектов
obj.user.profile.name = "Иван";
obj.user.profile.age = 30;
obj.settings.theme.color = "blue";

console.log(obj);
// {
//   user: { profile: { name: "Иван", age: 30 } },
//   settings: { theme: { color: "blue" } }
// }
```

### 3. Защита от опечаток

```javascript
function createTypoSafeObject(target, allowedProperties) {
  return new Proxy(target, {
    get(target, property, receiver) {
      if (!(property in target) && !allowedProperties.includes(property)) {
        // Поиск похожих свойств
        const similar = allowedProperties.find(prop => 
          levenshteinDistance(prop, property) <= 2
        );
        
        if (similar) {
          throw new Error(`Свойство "${property}" не найдено. Возможно, вы имели в виду "${similar}"?`);
        } else {
          throw new Error(`Свойство "${property}" не найдено.`);
        }
      }
      
      return Reflect.get(target, property, receiver);
    }
  });
}

// Простая реализация расстояния Левенштейна
function levenshteinDistance(str1, str2) {
  const matrix = Array(str2.length + 1).fill().map(() => Array(str1.length + 1).fill(0));
  
  for (let i = 0; i <= str1.length; i++) matrix[0][i] = i;
  for (let j = 0; j <= str2.length; j++) matrix[j][0] = j;
  
  for (let j = 1; j <= str2.length; j++) {
    for (let i = 1; i <= str1.length; i++) {
      const cost = str1[i - 1] === str2[j - 1] ? 0 : 1;
      matrix[j][i] = Math.min(
        matrix[j][i - 1] + 1,     // вставка
        matrix[j - 1][i] + 1,     // удаление
        matrix[j - 1][i - 1] + cost // замена
      );
    }
  }
  
  return matrix[str2.length][str1.length];
}

const user = createTypoSafeObject(
  { name: "Иван", age: 30 },
  ['name', 'age', 'email', 'address']
);

console.log(user.name);   // "Иван"
// user.namme;            // Ошибка: Свойство "namme" не найдено. Возможно, вы имели в виду "name"?
```

## Лучшие практики

### 1. Используйте Reflect для перенаправления операций

```javascript
// Хорошо: используем Reflect для перенаправления
const handler = {
  get(target, property, receiver) {
    console.log(`Доступ к ${property}`);
    return Reflect.get(target, property, receiver);
  }
};

// Плохо: повторяем логику JavaScript
const badHandler = {
  get(target, property, receiver) {
    console.log(`Доступ к ${property}`);
    return target[property]; // Не обрабатывает все случаи
  }
};
```

### 2. Избегайте избыточного использования Proxy

```javascript
// Плохо: Proxy для каждой мелочи
function createOverEngineeredObject(obj) {
  return new Proxy(obj, {
    get(target, property) {
      console.log('Перехват'); // Это будет вызываться слишком часто
      return target[property];
    }
  });
}

// Хорошо: Proxy только когда действительно нужно
function createValidationProxy(obj, schema) {
  return new Proxy(obj, {
    set(target, property, value) {
      if (schema[property] && typeof value !== schema[property]) {
        throw new TypeError(`Неверный тип для ${property}`);
      }
      return Reflect.set(target, property, value);
    }
  });
}
```

### 3. Правильная обработка this

```javascript
const target = {
  prop: 'значение',
  method() {
    return this.prop;
  }
};

const proxy = new Proxy(target, {
  get(target, property, receiver) {
    // Важно передавать receiver для правильной работы this
    return Reflect.get(target, property, receiver);
  }
});

console.log(proxy.method());  // "значение"
```

## Ограничения и соображения производительности

### 1. Производительность

Proxy может быть медленнее обычных объектов, особенно при частом доступе к свойствам:

```javascript
// Измерение производительности
function performanceTest() {
  const normalObj = { value: 0 };
  const proxyObj = new Proxy({ value: 0 }, {
    get(target, prop) {
      return target[prop];
    }
  });
  
  console.time('Обычный объект');
  for (let i = 0; i < 1000000; i++) {
    normalObj.value++;
  }
  console.timeEnd('Обычный объект');
  
  console.time('Proxy объект');
  for (let i = 0; i < 1000000; i++) {
    proxyObj.value++;
  }
  console.timeEnd('Proxy объект');
}

// performanceTest(); // Раскомментируйте для тестирования
```

### 2. Ограничения

```javascript
// Proxy не может перехватить определенные операции
const obj = {};
Object.preventExtensions(obj);

const proxy = new Proxy(obj, {
  // Это не сработает, потому что объект не расширяем
  set(target, property, value) {
    target[property] = value;
    return true;
  }
});

// proxy.newProp = 'значение'; // TypeError в строгом режиме
```

## Заключение

Proxy и Reflect предоставляют мощные возможности для метапрограммирования в JavaScript. Они позволяют создавать гибкие и расширяемые решения, но требуют осторожного использования из-за потенциального влияния на производительность и сложность кода.

Ключевые моменты:
1. Proxy позволяет перехватывать и переопределять фундаментальные операции над объектами
2. Reflect предоставляет функциональный API для тех же операций
3. Используйте эти возможности только когда это действительно необходимо
4. Всегда учитывайте производительность и читаемость кода
5. Используйте Reflect для перенаправления операций в обработчиках Proxy

#javascript #proxy #reflect #metaprogramming #es6 #advanced-concepts