# TypeScript

TypeScript — это строго типизированное надмножество JavaScript, которое компилируется в обычный JavaScript. Он добавляет опциональные типы, классы, интерфейсы и другие возможности, которые помогают разработчикам писать более надежный и поддерживаемый код.

## Основные характеристики

### Система типов
TypeScript предоставляет мощную систему типов, которая помогает выявлять ошибки на этапе компиляции, а не во время выполнения.

```typescript
// Пример типизации в TypeScript
interface User {
  name: string;
  age: number;
  email?: string; // Опциональное поле
}

function greetUser(user: User): string {
  return `Hello, ${user.name}!`;
}
```

### Интерфейсы и типы
TypeScript поддерживает интерфейсы и типы для определения структуры объектов и контрактов.

```typescript
// Интерфейсы
interface Drawable {
  draw(): void;
}

// Типы
type Status = 'active' | 'inactive' | 'pending';
```

### Дженерики
TypeScript поддерживает дженерики для создания универсальных и переиспользуемых компонентов.

```typescript
// Пример дженерика
function identity<T>(arg: T): T {
  return arg;
}

let output = identity<string>("hello");
```

## Преимущества TypeScript

1. **Раннее обнаружение ошибок** — Типизация помогает выявлять ошибки на этапе разработки
2. **Улучшенная IntelliSense** — Лучшая поддержка автодополнения и навигации в IDE
3. **Читаемость кода** — Типы делают код самодокументируемым
4. **Рефакторинг** — Безопасный рефакторинг с автоматическим обновлением ссылок
5. **Совместимость** — Полная совместимость с JavaScript

## Связанные концепции

- [[SOLID]] - Принципы SOLID особенно эффективно реализуются в TypeScript благодаря системе типов
- [[DRY (Don't Repeat Yourself)]] - Система типов TypeScript помогает избежать дублирования структур данных
- [[KISS (Keep It Simple, Stupid)]] - TypeScript может усложнить код, но улучшает архитектуру
- [[Чистый код]] - TypeScript способствует написанию чистого, типобезопасного кода
- [[Интерфейсы]] - TypeScript активно использует интерфейсы
- [[Дженерики]] - Дженерики являются важной частью системы типов TypeScript
- [[Наследование]] - TypeScript поддерживает наследование классов
- [[Композиция]] - TypeScript поддерживает композицию через интерфейсы и типы
- [[Полиморфизм]] - TypeScript поддерживает полиморфизм через интерфейсы и наследование

## В других технологиях

### В [[react]]
React с TypeScript обеспечивает типобезопасность компонентов:

```typescript
interface ButtonProps {
  text: string;
  onClick: () => void;
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ text, onClick, disabled = false }) => {
  return (
    <button onClick={onClick} disabled={disabled}>
      {text}
    </button>
  );
};
```

### В [[vue]]
Vue 3 имеет отличную поддержку TypeScript:

```typescript
import { defineComponent, ref } from 'vue';

export default defineComponent({
  name: 'HelloWorld',
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  setup() {
    const count = ref<number>(0);
    
    return {
      count
    };
  }
});
```

## Теги
#typescript #javascript #types #programming-languages