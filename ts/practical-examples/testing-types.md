---
aliases: ["Типизация в тестировании", "TS Testing Types"]
tags: [typescript, testing, types, patterns]
---

# Типизация в тестировании

## Введение

Типизация в тестировании - важный аспект разработки надежных приложений. Правильно спроектированные типы для тестов обеспечивают безопасность типов, улучшают автодополнение в IDE и уменьшают количество ошибок при написании тестов.

## Базовые паттерны типизации тестов

### Типизация тест-кейсов

```typescript
// Тип для описания тест-кейса
interface TestCase<TInput = any, TOutput = any> {
  name: string;
  input: TInput;
  expected: TOutput;
  description?: string;
}

// Тип для описания тест-сьюта
interface TestSuite<TInput = any, TOutput = any> {
  name: string;
  testCases: TestCase<TInput, TOutput>[];
  setup?: () => void | Promise<void>;
  teardown?: () => void | Promise<void>;
}

// Простой тест-раннер с типами
async function runTestSuite<TInput, TOutput>(
  suite: TestSuite<TInput, TOutput>,
  testFunction: (input: TInput) => TOutput | Promise<TOutput>
): Promise<void> {
  console.log(`Running test suite: ${suite.name}`);
  
  if (suite.setup) {
    await suite.setup();
  }
  
  try {
    for (const testCase of suite.testCases) {
      try {
        const result = await testFunction(testCase.input);
        
        if (JSON.stringify(result) === JSON.stringify(testCase.expected)) {
          console.log(`✓ ${testCase.name}`);
        } else {
          console.error(`✗ ${testCase.name}: expected ${JSON.stringify(testCase.expected)}, got ${JSON.stringify(result)}`);
        }
      } catch (error) {
        console.error(`✗ ${testCase.name}: error occurred - ${error}`);
      }
    }
  } finally {
    if (suite.teardown) {
      await suite.teardown();
    }
  }
}

// Пример использования
const add = (a: number, b: number): number => a + b;

const addTestSuite: TestSuite<[number, number], number> = {
  name: 'Add function tests',
  testCases: [
    { name: 'should add positive numbers', input: [2, 3], expected: 5 },
    { name: 'should add negative numbers', input: [-1, -1], expected: -2 },
    { name: 'should add zero', input: [5, 0], expected: 5 }
  ]
};

// runTestSuite(addTestSuite, ([a, b]) => add(a, b));
```

### Типизация для mock-объектов

```typescript
// Тип для создания mock-объекта
type Mock<T> = {
  [K in keyof T]: T[K] extends (...args: infer A) => infer R
    ? jest.Mock<R, A>
    : T[K];
} & {
  // Методы для управления mock-объектом
  $reset: () => void;
  $clear: () => void;
};

// Функция для создания mock-объекта
function createMock<T>(implementation?: Partial<T>): Mock<T> {
  const mock: any = {};
  
  // Добавляем методы управления mock
  mock.$reset = () => {
    Object.keys(mock).forEach(key => {
      if (jest.isMockFunction(mock[key])) {
        mock[key].mockClear();
      }
    });
  };
  
  mock.$clear = () => {
    Object.keys(mock).forEach(key => {
      if (jest.isMockFunction(mock[key])) {
        mock[key].mockReset();
      }
    });
  };
  
  // Добавляем реализации, если они предоставлены
  if (implementation) {
    Object.entries(implementation).forEach(([key, value]) => {
      if (typeof value === 'function') {
        mock[key] = jest.fn(value) as any;
      } else {
        mock[key] = value;
      }
    });
  }
  
  return mock;
}

// Пример использования с сервисом
interface UserService {
  getUser(id: number): Promise<User>;
  updateUser(id: number, data: Partial<User>): Promise<User>;
  deleteUser(id: number): Promise<boolean>;
}

const mockUserService: Mock<UserService> = createMock<UserService>({
  getUser: async (id: number) => ({ id, name: 'Test User', email: 'test@example.com', password: 'secret', createdAt: new Date(), updatedAt: new Date() }),
  updateUser: async (id: number, data: Partial<User>) => ({ id, name: 'Updated User', email: 'updated@example.com', password: 'secret', createdAt: new Date(), updatedAt: new Date(), ...data }),
  deleteUser: async (id: number) => true
});

// Использование в тесте
// describe('UserComponent', () => {
//   it('should fetch and display user', async () => {
//     const user = await mockUserService.getUser(1);
//     expect(user.name).toBe('Test User');
//     expect(mockUserService.getUser).toHaveBeenCalledWith(1);
//   });
// });
```

## Продвинутые примеры

### Типизация для асинхронных тестов

```typescript
// Тип для асинхронного тест-кейса
interface AsyncTestCase<TInput = any, TOutput = any> {
  name: string;
  input: TInput;
  expected: TOutput | ((result: TOutput) => boolean);
  shouldReject?: boolean;
  description?: string;
}

// Типизированный асинхронный тест-раннер
async function runAsyncTestSuite<TInput, TOutput>(
  suite: TestSuite<TInput, TOutput>,
  testFunction: (input: TInput) => Promise<TOutput>
): Promise<void> {
  console.log(`Running async test suite: ${suite.name}`);
  
  for (const testCase of suite.testCases) {
    try {
      let result: TOutput;
      let errorOccurred = false;
      
      try {
        result = await testFunction(testCase.input);
      } catch (error) {
        errorOccurred = true;
        result = error as any;
      }
      
      if (errorOccurred && testCase.shouldReject) {
        console.log(`✓ ${testCase.name} (correctly rejected)`);
      } else if (!errorOccurred && !testCase.shouldReject) {
        // Проверяем результат
        if (typeof testCase.expected === 'function') {
          if (testCase.expected(result)) {
            console.log(`✓ ${testCase.name}`);
          } else {
            console.error(`✗ ${testCase.name}: custom validation failed`);
          }
        } else if (JSON.stringify(result) === JSON.stringify(testCase.expected)) {
          console.log(`✓ ${testCase.name}`);
        } else {
          console.error(`✗ ${testCase.name}: expected ${JSON.stringify(testCase.expected)}, got ${JSON.stringify(result)}`);
        }
      } else {
        console.error(`✗ ${testCase.name}: unexpected behavior`);
      }
    } catch (error) {
      console.error(`✗ ${testCase.name}: error occurred - ${error}`);
    }
  }
}

// Пример асинхронного теста
const fetchUser = async (id: number): Promise<User> => {
  if (id <= 0) {
    throw new Error('Invalid user ID');
  }
  return { id, name: 'Test User', email: 'test@example.com', password: 'secret', createdAt: new Date(), updatedAt: new Date() };
};

const asyncTestSuite: TestSuite<number, User> = {
  name: 'Async fetchUser tests',
  testCases: [
    { 
      name: 'should fetch user with valid ID', 
      input: 1, 
      expected: (result: User) => result.id === 1 && result.name === 'Test User' 
    },
    { 
      name: 'should reject with invalid ID', 
      input: -1, 
      expected: new Error('Invalid user ID'),
      shouldReject: true
    }
  ]
};

// runAsyncTestSuite(asyncTestSuite, fetchUser);
```

### Типизация для тестирования компонентов

```typescript
// Типы для тестирования React-компонентов
import React from 'react';

interface ComponentTestSetup<Props> {
  component: React.ComponentType<Props>;
  defaultProps?: Props;
  providers?: React.ComponentType<any>[];
}

interface ComponentTest<Props> {
  name: string;
  props: Props;
  expected: (container: HTMLElement) => boolean;
  description?: string;
}

interface ComponentTestSuite<Props> {
  name: string;
  setup: ComponentTestSetup<Props>;
  tests: ComponentTest<Props>[];
}

// Тест-раннер для React-компонентов
async function runComponentTestSuite<Props>(
  suite: ComponentTestSuite<Props>
): Promise<void> {
  console.log(`Running component test suite: ${suite.name}`);
  
  for (const test of suite.tests) {
    try {
      // В реальном приложении здесь будет рендеринг компонента с помощью testing-library
      // const { container } = render(<suite.setup.component {...test.props} />);
      
      // if (test.expected(container)) {
      //   console.log(`✓ ${test.name}`);
      // } else {
      //   console.error(`✗ ${test.name}: component did not match expected output`);
      // }
    } catch (error) {
      console.error(`✗ ${test.name}: error occurred - ${error}`);
    }
  }
}

// Пример тестирования компонента
interface WelcomeProps {
  name: string;
  age?: number;
}

const WelcomeComponent: React.FC<WelcomeProps> = ({ name, age }) => (
  <div>
    <h1>Hello, {name}!</h1>
    {age && <p>Age: {age}</p>}
  </div>
);

const welcomeTestSuite: ComponentTestSuite<WelcomeProps> = {
  name: 'WelcomeComponent tests',
  setup: {
    component: WelcomeComponent,
    defaultProps: { name: 'Test User' }
  },
  tests: [
    {
      name: 'should render user name',
      props: { name: 'John' },
      expected: (container) => container.textContent?.includes('Hello, John!') || false
    },
    {
      name: 'should render age when provided',
      props: { name: 'Jane', age: 25 },
      expected: (container) => container.textContent?.includes('Age: 25') || false
    }
  ]
};
```

### Типизация для тестирования API

```typescript
// Типы для тестирования API
interface ApiTestRequest {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  url: string;
  headers?: Record<string, string>;
  body?: any;
}

interface ApiTestResponse {
  status: number;
  headers: Record<string, string>;
  body: any;
}

interface ApiTestCase {
  name: string;
  request: ApiTestRequest;
  expected: {
    status: number;
    body?: any;
    headers?: Record<string, string>;
  };
  description?: string;
}

interface ApiTestSuite {
  name: string;
  baseUrl: string;
  tests: ApiTestCase[];
  setup?: () => Promise<void>;
  teardown?: () => Promise<void>;
}

// Тест-раннер для API
async function runApiTestSuite(suite: ApiTestSuite): Promise<void> {
  console.log(`Running API test suite: ${suite.name}`);
  
  if (suite.setup) {
    await suite.setup();
  }
  
  try {
    for (const testCase of suite.tests) {
      try {
        const response = await fetch(`${suite.baseUrl}${testCase.request.url}`, {
          method: testCase.request.method,
          headers: testCase.request.headers,
          body: testCase.request.body ? JSON.stringify(testCase.request.body) : undefined
        });
        
        const responseBody = await response.json();
        
        if (response.status === testCase.expected.status) {
          if (testCase.expected.body) {
            if (JSON.stringify(responseBody) === JSON.stringify(testCase.expected.body)) {
              console.log(`✓ ${testCase.name}`);
            } else {
              console.error(`✗ ${testCase.name}: body mismatch`);
            }
          } else {
            console.log(`✓ ${testCase.name}`);
          }
        } else {
          console.error(`✗ ${testCase.name}: expected status ${testCase.expected.status}, got ${response.status}`);
        }
      } catch (error) {
        console.error(`✗ ${testCase.name}: error occurred - ${error}`);
      }
    }
  } finally {
    if (suite.teardown) {
      await suite.teardown();
    }
  }
}

// Пример API теста
const userApiTestSuite: ApiTestSuite = {
  name: 'User API tests',
  baseUrl: 'http://localhost:3000/api',
  tests: [
    {
      name: 'should get user by ID',
      request: {
        method: 'GET',
        url: '/users/1'
      },
      expected: {
        status: 200,
        body: { id: 1, name: 'Test User', email: 'test@example.com' }
      }
    },
    {
      name: 'should create new user',
      request: {
        method: 'POST',
        url: '/users',
        headers: { 'Content-Type': 'application/json' },
        body: { name: 'New User', email: 'newuser@example.com' }
      },
      expected: {
        status: 201
      }
    }
  ]
};
```

## Практические примеры

### Типизация для unit-тестов с Jest

```typescript
// Типы для специфичных для Jest тестов
type JestTestCallback = () => void | Promise<void>;

interface TypedDescribe {
  <T>(name: string, fn: (test: TypedTest<T>) => void): void;
}

interface TypedTest<T> {
  (name: string, callback: (data: T) => JestTestCallback): void;
}

// Вспомогательная функция для типизации тестов с общими данными
function createTypedTests<T>(setupData: () => Promise<T>): [TypedDescribe, TypedTest<T>] {
  const typedDescribe: TypedDescribe = (name, fn) => {
    describe(name, () => {
      let testData: T;
      
      beforeAll(async () => {
        testData = await setupData();
      });
      
      fn((testName, callback) => {
        test(testName, () => callback(testData));
      });
    });
  };
  
  return [typedDescribe, (name, callback) => test(name, async () => {
    const testData = await setupData();
    return callback(testData);
  })];
}

// Пример использования
interface TestContext {
  userService: UserService;
  userRepository: Mock<UserRepository>;
  logger: Mock<Logger>;
}

const [typedDescribe, typedTest] = createTypedTests<TestContext>(async () => {
  const mockUserRepository = createMock<UserRepository>();
  const mockLogger = createMock<Logger>();
  
  const userService = new UserService(mockUserRepository, mockLogger);
  
  return {
    userService,
    userRepository: mockUserRepository,
    logger: mockLogger
  };
});

// typedDescribe<TestContext>('UserService', (test) => {
//   test('should create user', async (context) => {
//     const { userService, userRepository } = context;
    
//     const userData = { name: 'Test User', email: 'test@example.com' };
//     const expectedUser = { id: 1, ...userData, password: 'secret', createdAt: new Date(), updatedAt: new Date() };
    
//     userRepository.create.mockResolvedValue(expectedUser);
    
//     const result = await userService.createUser(userData);
    
//     expect(result).toEqual(expectedUser);
//     expect(userRepository.create).toHaveBeenCalledWith(userData);
//   });
// });
```

### Типизация для тестирования с фикстурами

```typescript
// Типы для фикстур
interface Fixture<T> {
  data: T;
  setup?: () => Promise<void>;
  teardown?: () => Promise<void>;
}

interface FixtureCollection {
  [name: string]: Fixture<any>;
}

// Управление фикстурами
class FixtureManager {
  private fixtures: FixtureCollection = {};
  
  add<T>(name: string, fixture: Fixture<T>): void {
    this.fixtures[name] = fixture;
  }
  
  async load<T>(name: string): Promise<T> {
    const fixture = this.fixtures[name];
    if (!fixture) {
      throw new Error(`Fixture ${name} not found`);
    }
    
    if (fixture.setup) {
      await fixture.setup();
    }
    
    return fixture.data;
  }
  
  async unload(name: string): Promise<void> {
    const fixture = this.fixtures[name];
    if (fixture && fixture.teardown) {
      await fixture.teardown();
    }
  }
  
  async loadAll(): Promise<{ [name: string]: any }> {
    const result: { [name: string]: any } = {};
    
    for (const [name, fixture] of Object.entries(this.fixtures)) {
      result[name] = await this.load(name);
    }
    
    return result;
  }
}

// Пример фикстур
const fixtureManager = new FixtureManager();

fixtureManager.add('users', {
  data: [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
  ],
  setup: async () => {
    console.log('Setting up users fixture');
    // Здесь может быть логика для подготовки базы данных
  },
  teardown: async () => {
    console.log('Tearing down users fixture');
    // Здесь может быть логика для очистки базы данных
  }
});

fixtureManager.add('config', {
  data: {
    apiEndpoint: 'http://localhost:3000',
    timeout: 5000
  }
});
```

### Типизация для тестирования производительности

```typescript
// Типы для тестирования производительности
interface PerformanceTest {
  name: string;
  fn: () => any;
  maxExecutionTime: number; // в миллисекундах
  iterations?: number; // количество итераций для усреднения
  description?: string;
}

interface PerformanceTestResult {
  name: string;
  averageTime: number;
  minTime: number;
  maxTime: number;
  passed: boolean;
  details: {
    executionTimes: number[];
    iterations: number;
  };
}

// Тест-раннер для производительности
async function runPerformanceTest(test: PerformanceTest): Promise<PerformanceTestResult> {
  const executionTimes: number[] = [];
  const iterations = test.iterations || 1;
  
  for (let i = 0; i < iterations; i++) {
    const start = performance.now();
    await Promise.resolve(test.fn()); // Оборачиваем в Promise.resolve для поддержки асинхронных функций
    const end = performance.now();
    
    executionTimes.push(end - start);
  }
  
  const averageTime = executionTimes.reduce((a, b) => a + b, 0) / executionTimes.length;
  const minTime = Math.min(...executionTimes);
  const maxTime = Math.max(...executionTimes);
  
  const result: PerformanceTestResult = {
    name: test.name,
    averageTime,
    minTime,
    maxTime,
    passed: averageTime <= test.maxExecutionTime,
    details: {
      executionTimes,
      iterations
    }
  };
  
  if (result.passed) {
    console.log(`✓ ${test.name}: ${averageTime.toFixed(2)}ms (max: ${test.maxExecutionTime}ms)`);
  } else {
    console.error(`✗ ${test.name}: ${averageTime.toFixed(2)}ms (max: ${test.maxExecutionTime}ms)`);
  }
  
  return result;
}

// Пример теста производительности
const performanceTests: PerformanceTest[] = [
  {
    name: 'Array map operation',
    fn: () => {
      const arr = Array.from({ length: 10000 }, (_, i) => i);
      return arr.map(x => x * 2);
    },
    maxExecutionTime: 50, // 50ms
    iterations: 5
  },
  {
    name: 'Object creation',
    fn: () => {
      const obj = {};
      for (let i = 0; i < 1000; i++) {
        obj[`prop${i}`] = i;
      }
      return obj;
    },
    maxExecutionTime: 10, // 10ms
    iterations: 10
  }
];

// Запуск тестов производительности
// performanceTests.forEach(test => runPerformanceTest(test));
```

## Заключение

Типизация в тестировании позволяет создавать более надежные и поддерживаемые тесты. Правильно спроектированные типы обеспечивают безопасность, улучшают автодополнение в IDE и упрощают рефакторинг тестов. Используйте эти паттерны как основу для создания типизированных тестов в ваших приложениях.

См. также: [[state-management-types]] | [[middleware-types]] | [[dependency-injection-types]]