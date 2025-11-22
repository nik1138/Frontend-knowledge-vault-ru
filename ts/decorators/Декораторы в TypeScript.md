# Декораторы в TypeScript

Декораторы - это специальный вид объявления, который может быть присоединен к объявлению класса, методу, аксессору, свойству или параметру. Декораторы используют форму @expression, где expression должен вычисляться в функцию, которая будет вызвана во время выполнения с информацией о декорированном объявлении.

## Содержание раздела

1. [[ts/decorators/class-decorators|Декораторы классов]] - Применение декораторов к классам
2. [[ts/decorators/method-decorators|Декораторы методов]] - Применение декораторов к методам
3. [[ts/decorators/property-decorators|Декораторы свойств]] - Применение декораторов к свойствам
4. [[ts/decorators/accessor-decorators|Декораторы аксессоров]] - Применение декораторов к геттерам и сеттерам
5. [[ts/decorators/parameter-decorators|Декораторы параметров]] - Применение декораторов к параметрам
6. [[ts/decorators/composition|Композиция декораторов]] - Комбинирование нескольких декораторов
7. [[ts/decorators/factories|Фабрики декораторов]] - Создание параметризованных декораторов
8. [[ts/decorators/metadata|Метаданные и рефлексия]] - Работа с метаданными во время выполнения

## Введение

Декораторы - это мощная возможность TypeScript, которая позволяет добавлять аннотации и метапрограммировать код. Они представляют собой функции, которые могут изменять поведение классов и их членов во время определения.

## Основные концепции

### 1. Типы декораторов

В TypeScript существует пять типов декораторов:

```typescript
// Декоратор класса
@ClassDecorator
class MyClass {
  // Декоратор свойства
  @PropertyDecorator
  property: string;
  
  // Декоратор метода
  @MethodDecorator
  method(@ParameterDecorator param: string) {
    // ...
  }
  
  // Декоратор аксессора
  @AccessorDecorator
  get accessor() {
    return this._value;
  }
  
  set accessor(value: string) {
    this._value = value;
  }
}
```

### 2. Функции декораторов

Каждый тип декоратора имеет свою сигнатуру функции:

```typescript
// Декоратор класса
function classDecorator(constructor: Function) {
  // ...
}

// Декоратор свойства
function propertyDecorator(target: any, propertyKey: string) {
  // ...
}

// Декоратор метода
function methodDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // ...
}

// Декоратор аксессора
function accessorDecorator(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  // ...
}

// Декоратор параметра
function parameterDecorator(
  target: any,
  propertyKey: string | undefined,
  parameterIndex: number
) {
  // ...
}
```

### 3. Порядок применения декораторов

Декораторы применяются в определенном порядке:

1. **Для параметров**: Слева направо
2. **Для методов, аксессоров и свойств**: Сверху вниз
3. **Для классов**: Снизу вверх

```typescript
@First()
@Second()
class Example {
  @First()
  @Second()
  property: string;
  
  @First()
  @Second()
  method(@First() @Second() param: string) {
    // ...
  }
}
```

## Практическое применение

### 1. Создание фреймворка

Декораторы широко используются в фреймворках для создания декларативного API:

```typescript
// Пример из NestJS
import { Controller, Get, Post, Body } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string[] {
    return ['Cat 1', 'Cat 2'];
  }
  
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }
}
```

### 2. Валидация данных

Декораторы позволяют создавать декларативные системы валидации:

```typescript
class User {
  @Required()
  @MinLength(2)
  name: string;
  
  @Required()
  @Email()
  email: string;
  
  @MinLength(6)
  password: string;
}

const user = new User();
const result = validate(user); // Проверяет все декорированные правила
```

### 3. Аспектно-ориентированное программирование

Декораторы позволяют реализовать сквозную функциональность:

```typescript
class Service {
  @Log()
  @Cache({ ttl: 60000 })
  @Retry({ maxAttempts: 3 })
  @Timing()
  async fetchData(id: string): Promise<any> {
    // Основная логика метода
  }
}
```

## Преимущества декораторов

### 1. Декларативность

Декораторы позволяют описывать поведение в декларативной форме:

```typescript
// Без декораторов - императивный стиль
function getUser(id: string) {
  console.log(`Calling getUser with id: ${id}`);
  try {
    const result = performGetUser(id);
    console.log(`getUser completed successfully`);
    return result;
  } catch (error) {
    console.error(`getUser failed:`, error);
    throw error;
  }
}

// С декораторами - декларативный стиль
@Log()
@HandleErrors()
async function getUser(id: string) {
  return performGetUser(id);
}
```

### 2. Переиспользуемость

Декораторы можно использовать в разных местах:

```typescript
// Декоратор для кэширования
function cache(ttl: number) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Реализация кэширования
  };
}

// Использование в разных классах
class UserService {
  @cache(30000)
  getUser(id: string) { /* ... */ }
}

class ProductService {
  @cache(60000)
  getProduct(id: string) { /* ... */ }
}
```

### 3. Композиция

Декораторы можно комбинировать для создания сложного поведения:

```typescript
class Service {
  @validate
  @authorize({ roles: ['admin'] })
  @log
  @cache({ ttl: 30000 })
  @retry({ maxAttempts: 3 })
  updateSettings(settings: Settings) {
    // Бизнес-логика
  }
}
```

## Настройка TypeScript для декораторов

Для использования декораторов необходимо включить соответствующие опции в `tsconfig.json`:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

- `experimentalDecorators`: Включает поддержку декораторов
- `emitDecoratorMetadata`: Включает генерацию метаданных для декораторов

## Ограничения и лучшие практики

### 1. Производительность

Сложные декораторы могут замедлять выполнение:

```typescript
// Избегайте тяжелых вычислений в фабриках декораторов
function heavyDecorator() {
  // Тяжелые вычисления здесь выполняются при определении класса
  const heavyData = performHeavyComputation();
  
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Легкие операции здесь выполняются при вызове метода
    return descriptor;
  };
}

// Лучше:
function lightDecorator(heavyData: any) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // Используем предварительно вычисленные данные
    return descriptor;
  };
}

// Вычисляем данные один раз
const heavyData = performHeavyComputation();
```

### 2. Читаемость

Слишком много декораторов может ухудшить читаемость:

```typescript
// Слишком много декораторов
class Service {
  @validate
  @authorize({ roles: ['admin'] })
  @log
  @cache({ ttl: 30000 })
  @retry({ maxAttempts: 3 })
  @timing
  @metrics({ namespace: 'service' })
  complexMethod() { /* ... */ }
}

// Лучше - группировка по функциональности
class Service {
  @businessLogic
  @infrastructure
  complexMethod() { /* ... */ }
}
```

### 3. Тестирование

Декораторы могут усложнить тестирование:

```typescript
// Тестируем оригинальный метод
const service = new Service();
const originalMethod = service.complexMethod;
// Мокаем декораторы или тестируем поведение декораторов отдельно
```

## Связи с другими концепциями

Декораторы тесно связаны с другими аспектами TypeScript:

- [[ts/advanced-generics/Продвинутые_дженерики|Продвинутые дженерики]] - Многие декораторы используют дженерики
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Декораторы могут изменять типы
- [[ts/advanced-types/conditional-types|Условные типы]] - Для создания типобезопасных декораторов
- [[ts/type-system/type-inference|Вывод типов]] - Метаданные помогают в выводе типов

## Рекомендации по изучению

1. Начните с простых декораторов классов и методов
2. Освойте фабрики декораторов для параметризации
3. Практикуйтесь в комбинировании декораторов
4. Изучите работу с метаданными и рефлексией
5. Создавайте собственные декораторы для решения конкретных задач
6. Изучите реализации декораторов в популярных фреймворках

## Следующие шаги

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Сложные паттерны работы с модулями
- [[ts/advanced-generics/Продвинутые_дженерики|Продвинутые дженерики]] - Сложные паттерны с дженериками
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Встроенные и пользовательские типы-помощники