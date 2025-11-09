# TypeScript Compatibility

Совместимость в TypeScript охватывает различные аспекты обеспечения того, что код может работать корректно в разных версиях языка, сред выполнения и с различными системами модулей.

## Основные области совместимости

### [Type Compatibility](type-compatibility.md)
Совместимость типов в системе TypeScript - структурная типизация, утиная типизация, совместимость функций, классов и интерфейсов.

### [Module Compatibility](module-compatibility.md)
Совместимость между различными системами модулей (ES Modules, CommonJS, UMD) и форматами экспорта/импорта.

### [Version Compatibility](version-compatibility.md)
Совместимость между разными версиями TypeScript, ECMAScript и сред выполнения (Node.js, браузеры).

## Архитектура совместимости

```
                    ┌─────────────────────────────────────────────────────────┐
                    │              Compatibility System                         │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        │                                     │                                                           │
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Type          │               │      Module              │                                    │  Version        │
  │ Compatibility   │               │  Compatibility           │                                    │ Compatibility   │
  │                 │               │                          │                                    │                 │
  │ - Structural    │               │ - ES Modules vs CommonJS │                                    │ - TypeScript    │
  │   Typing        │               │ - Import/Export Formats  │                                    │   Versions      │
  │ - Duck Typing   │               │ - Ambient Declarations   │                                    │ - ECMA Script   │
  │ - Function      │               │ - Global Modules         │                                    │   Versions      │
  │   Compatibility │               │ - Universal Bundles      │                                    │ - Runtime       │
  │ - Class         │               │ - Format Conversion      │                                    │   Environments  │
  │   Compatibility │               │ - Conditional Exports    │                                    │ - Dependency    │
  │ - Interface     │               │ - ESM/CJS Interop        │                                    │   Compatibility │
  │   Compatibility │               │                          │                                    │ - Migration     │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │         Cross-Cutting           │
        │                           │        Compatibility            │
        │                           │                                 │
        │                           │ - Build Tool Integration        │
        │                           │ - Linting & Type Checking       │
        │                           │ - Testing Across Versions       │
        │                           │ - Documentation Consistency     │
        │                           │ - CI/CD Pipeline Alignment      │
        │                           └───────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                          Best Practices                              │
        │                           │                                                                         │
        │                           │ - Progressive Enhancement                                               │
        │                           │ - Graceful Degradation                                                  │
        │                           │ - Feature Detection                                                     │
        │                           │ - Backward Compatibility Guarantees                                     │
        │                           │ - Version Pinning Strategies                                            │
        └───────────────────────────┤ - Automated Testing Matrix                                              │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Основные принципы совместимости

### 1. Принцип ласточки (Duck Typing)
> "Если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка"

TypeScript использует структурную типизацию, где совместимость определяется формой значения, а не его именем.

```typescript
interface Quackable {
  quack(): void;
}

class Duck {
  quack() { console.log("Quack!"); }
  swim() { console.log("Swimming"); }
}

class Person {
  quack() { console.log("Person imitating duck"); }
  walk() { console.log("Walking"); }
}

// Оба класса совместимы с интерфейсом Quackable
const duck: Quackable = new Duck();   // OK
const person: Quackable = new Person(); // OK - структурно совместим
```

### 2. Принцип подстановки Лисков
Подтипы должны быть взаимозаменяемы с базовыми типами.

### 3. Принцип прогрессивного улучшения
Код должен работать в более старых версиях, где это возможно, с улучшениями для новых версий.

## Ключевые области совместимости

### Совместимость систем типов
- Структурная vs номинальная типизация
- Вариантность (ковариантность, контравариантность, инвариантность)
- Совместимость объединений и пересечений
- Совместимость функций и методов

### Совместимость модулей
- Интероперабельность между системами модулей
- Совместимость с CommonJS/AMD/UMD
- Conditional Exports в package.json
- Tree-shaking и dead code elimination

### Совместимость версий
- Назад совместимость (backward compatibility)
- Вперед совместимость (forward compatibility) 
- Постепенные миграции
- Поддержка старых сред выполнения

## Практическое применение

### Создание совместимых библиотек
```typescript
// good-library.ts - пример совместимой библиотеки
export interface CompatibleAPI {
  // Использовать стабильные, широко поддерживаемые возможности
  method(value: string): Promise<string>;
  
  // Обеспечить обратную совместимость
  legacyMethod?(value: string): string; // необязательный для новых версий
}

// Универсальная реализация
class GoodLibrary implements CompatibleAPI {
  async method(value: string): Promise<string> {
    // Современная реализация с Promise
    return new Promise(resolve => resolve(value.toUpperCase()));
  }
  
  // Совместимость с более старыми версиями
  legacyMethod?(value: string): string {
    return value.toUpperCase();
  }
}
```

### Проверка совместимости
```typescript
// compatibility-check.ts
function checkTypeCompatibility() {
  // Проверка совместимости типов
  interface Expected {
    name: string;
    age: number;
  }
  
  const provided = {
    name: "Alice",
    age: 30,
    extra: "property" // дополнительные свойства разрешены
  };
  
  // Это работает благодаря структурной типизации
  const compatible: Expected = provided; // OK
}

function checkRuntimeCompatibility() {
  // Проверка совместимости возможностей среды выполнения
  if ('Promise' in window) {
    // Современные возможности
  } else {
    // Резервные варианты
  }
}
```

## Лучшие практики

1. **Тестирование на целевых версиях** - запуск тестов в средах, соответствующих требованиям
2. **Использование флагов компиляции** - настройка tsconfig.json для целевой совместимости  
3. **Постепенные обновления** - миграция функциональности поэтапно
4. **Полифилы и шимы** - обеспечения отсутствующих возможностей
5. **Документирование** - ясные требования к средам выполнения

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - основы совместимости типов
- [[TypeScript Modules]] - система модулей и совместимость
- [[../compilation/configuration]] - конфигурация для совместимости
- [[../ecosystem/tools]] - инструменты для обеспечения совместимости
- [[../migration/guides]] - практические руководства по миграции