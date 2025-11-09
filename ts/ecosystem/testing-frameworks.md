# Тестирование в TypeScript

Тестирование в TypeScript включает в себя использование различных фреймворков и утилит для проверки типобезопасности и функциональности кода. TypeScript предоставляет мощные возможности для создания надежных тестов.

## Основы тестирования в TypeScript

### Типобезопасные тесты

Одно из главных преимуществ использования TypeScript в тестировании - это типобезопасность:

```typescript
// Тестирование функции с типами
interface User {
  id: number;
  name: string;
  email: string;
}

const createUser = (name: string, email: string): User => ({
  id: Date.now(),
  name,
  email
});

// Тест с типобезопасностью
describe('createUser', () => {
  it('should create a user with correct properties', () => {
    const user = createUser('John Doe', 'john@example.com');
    
    // TypeScript гарантирует типы, что предотвращает ошибки
    expect(user.id).toBeGreaterThan(0);
    expect(user.name).toBe('John Doe');
    expect(user.email).toBe('john@example.com');
    
    // Следующая строка вызовет ошибку TypeScript, если свойство не существует
    // expect(user.nonExistentProp).toBeDefined(); // Ошибка компиляции
  });
});
```

## Популярные тестовые фреймворки

### Jest

Jest - один из самых популярных тестовых фреймворков для JavaScript/TypeScript:

```typescript
// package.json
{
  "devDependencies": {
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0",
    "ts-jest": "^29.0.0"
  }
}

// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
};
```

Пример использования с типами:

```typescript
// math-utils.test.ts
import { add, divide } from './math-utils';

describe('Math utilities', () => {
  describe('add', () => {
    it('should return sum of two numbers', () => {
      const result = add(2, 3);
      expect(result).toBe(5);
      
      // TypeScript проверяет, что add ожидает числа
      // add('2', '3'); // Ошибка компиляции
    });
  });

  describe('divide', () => {
    it('should return quotient of two numbers', () => {
      expect(divide(6, 2)).toBe(3);
    });

    it('should throw error when dividing by zero', () => {
      expect(() => divide(5, 0)).toThrow('Division by zero');
    });
  });
});
```

### Vitest

Vitest - современный тестовый фреймворк, созданный для Vite и ESM:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
});
```

Пример теста с Vitest:

```typescript
// user-service.test.ts
import { describe, it, expect } from 'vitest';
import { UserService } from './user-service';

describe('UserService', () => {
  it('should create a user with generated ID', () => {
    const service = new UserService();
    const user = service.create('John', 'john@example.com');
    
    expect(user.id).toBeDefined();
    expect(user.name).toBe('John');
    expect(user.email).toBe('john@example.com');
  });
});
```

### Mocha + Chai

Классическая комбинация для тестирования:

```typescript
import { expect } from 'chai';
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calc: Calculator;

  beforeEach(() => {
    calc = new Calculator();
  });

  it('should add two numbers correctly', () => {
    const result = calc.add(2, 3);
    expect(result).to.equal(5);
  });

  it('should multiply two numbers correctly', () => {
    const result = calc.multiply(3, 4);
    expect(result).to.equal(12);
  });
});
```

## Типизированные моки

### Мокирование модулей

```typescript
// user-api.ts
export interface User {
  id: number;
  name: string;
}

export const fetchUser = async (id: number): Promise<User> => {
  // реализация
  return { id, name: `User ${id}` };
};

// user-service.ts
import { fetchUser, User } from './user-api';

export class UserService {
  async getUserDetails(id: number): Promise<string> {
    const user = await fetchUser(id);
    return `${user.name} (ID: ${user.id})`;
  }
}

// user-service.test.ts
import { UserService } from './user-service';
import * as userApi from './user-api';

// Типобезопасное мокирование
vi.mock('./user-api');

describe('UserService', () => {
  it('should format user details correctly', async () => {
    // Мокирование функции с правильным типом
    const mockFetchUser = vi.fn<Promise<userApi.User>, [number]>()
      .mockResolvedValue({ id: 1, name: 'John Doe' });
    
    // Замена моком
    vi.spyOn(userApi, 'fetchUser').mockImplementation(mockFetchUser);

    const service = new UserService();
    const result = await service.getUserDetails(1);

    expect(result).toBe('John Doe (ID: 1)');
    expect(mockFetchUser).toHaveBeenCalledWith(1);
  });
});
```

## Тестирование типов

### Type Tests

Для проверки типов можно использовать специальные тесты на уровне типов:

```typescript
// type-tests.ts
import { Equal, Expect } from '@type-challenges/utils';
import { GetUserResponse, ApiError } from './api-types';

// Проверка, что типы соответствуют ожиданиям
type tests = [
  Expect<Equal<GetUserResponse, { id: number; name: string; email: string }>>,
  Expect<Equal<ApiError, { error: string; code: number }>>
];
```

### tsd - тестирование типов

Для проверки типов при компиляции:

```typescript
// user.test-d.ts (файл для тестирования типов)
import { expectType } from 'tsd';
import { UserService } from './user-service';

const userService = new UserService();

// Проверка типа возвращаемого значения
expectType<Promise<{ id: number; name: string; email: string }>>(
  userService.getUser(1)
);
```

## Тестирование асинхронного кода

```typescript
// async-service.test.ts
import { AsyncService } from './async-service';

describe('AsyncService', () => {
  it('should handle async operations', async () => {
    const service = new AsyncService();
    
    const result = await service.processData('test-data');
    
    expect(result).toEqual({
      status: 'success',
      data: 'processed: test-data',
      timestamp: expect.any(Number)
    });
  });

  it('should handle async errors', async () => {
    const service = new AsyncService();
    
    await expect(service.processData('error')).rejects.toThrow('Processing failed');
  });
});
```

## Скриншотные тесты

С использованием типов:

```typescript
// component.test.tsx
import React from 'react';
import { render } from '@testing-library/react';
import { UserCard } from './user-card';
import { User } from './types';

const mockUser: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com'
};

describe('UserCard', () => {
  it('should render user correctly', () => {
    const { asFragment } = render(<UserCard user={mockUser} />);
    expect(asFragment()).toMatchSnapshot();
  });
});
```

## Интеграция с другими инструментами

### ESLint и тестирование

Конфигурация для тестов:

```json
// .eslintrc.json
{
  "overrides": [
    {
      "files": ["*.test.ts", "*.test.tsx", "**/__tests__/**/*"],
      "rules": {
        "@typescript-eslint/no-non-null-assertion": "off",
        "jest/expect-expect": "off"
      }
    }
  ]
}
```

### Покрытие кода

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

## Лучшие практики

### Организация тестов

```typescript
// Рекомендуемая структура
// src/
//   calculator.ts
//   calculator.test.ts
//   types/
//     calculator-types.ts
//     calculator-types.test.ts

// calculator-types.test.ts
import { 
  Expect, 
  Equal, 
  IsTrue,
  IsFalse 
} from '../utils/type-tests';
import { PositiveNumber, EvenNumber } from './calculator-types';

// Проверка типов
type tests = [
  Expect<IsTrue<PositiveNumber<1>>>,
  Expect<IsFalse<PositiveNumber<-1>>>,
  Expect<IsTrue<EvenNumber<2>>>,
  Expect<IsFalse<EvenNumber<3>>>
];
```

### Тестирование граничных значений

```typescript
// edge-cases.test.ts
import { calculateDiscount } from './pricing';

describe('calculateDiscount', () => {
  it('should handle 0% discount', () => {
    expect(calculateDiscount(100, 0)).toBe(100);
  });

  it('should handle 100% discount', () => {
    expect(calculateDiscount(100, 100)).toBe(0);
  });

  it('should handle negative price', () => {
    expect(calculateDiscount(-10, 10)).toBe(-9);
  });

  it('should handle negative discount', () => {
    expect(() => calculateDiscount(100, -10)).toThrow();
  });
});
```

## Современные тенденции

### Property-based testing

С использованием TypeScript:

```typescript
// property-based.test.ts
import fc from 'fast-check';
import { reverseString } from './string-utils';

describe('reverseString', () => {
  it('should preserve length when reversing', () => {
    fc.assert(
      fc.property(
        fc.string(),
        (str) => {
          const reversed = reverseString(str);
          expect(reversed.length).toBe(str.length);
        }
      )
    );
  });

  it('should reverse and reverse again to get original', () => {
    fc.assert(
      fc.property(
        fc.string(),
        (str) => {
          const doubleReversed = reverseString(reverseString(str));
          expect(doubleReversed).toBe(str);
        }
      )
    );
  });
});
```

## Связь с другими концепциями

- [[tooling-workflows/testing-workflows]] - рабочие процессы тестирования
- [[error-handling/testing-errors]] - тестирование обработки ошибок
- [[modules/testing-modules]] - тестирование модулей
- [[type-system/type-guards]] - защита типов в тестах
- [[patterns/test-patterns]] - паттерны тестирования в TypeScript