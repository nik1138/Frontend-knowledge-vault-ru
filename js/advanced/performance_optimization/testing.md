# Оптимизация тестирования в JavaScript

## Введение

Оптимизация процесса тестирования играет важную роль в обеспечении качества кода и производительности приложений. Эффективные тесты помогают выявлять проблемы на ранних стадиях разработки, улучшают надежность кода и ускоряют процесс разработки. В этом разделе мы рассмотрим различные техники оптимизации тестирования в JavaScript.

## Оптимизация unit тестов

Эффективное написание и выполнение unit тестов:

```javascript
// Оптимизация unit тестов
class UnitTestOptimization {
  // Создание тестового фреймворка с оптимизациями
  static createOptimizedTestFramework() {
    const tests = [];
    const beforeEachHooks = [];
    const afterEachHooks = [];
    const beforeAllHooks = [];
    const afterAllHooks = [];
    
    let testStats = {
      passed: 0,
      failed: 0,
      skipped: 0,
      totalTime: 0
    };
    
    return {
      // Описание теста
      test(description, testFunction, options = {}) {
        tests.push({
          description,
          testFunction,
          options,
          skip: options.skip || false,
          only: options.only || false
        });
      },
      
      // Группа тестов
      describe(groupName, groupFunction) {
        console.log(`Группа тестов: ${groupName}`);
        groupFunction();
      },
      
      // Before each hook
      beforeEach(hookFunction) {
        beforeEachHooks.push(hookFunction);
      },
      
      // After each hook
      afterEach(hookFunction) {
        afterEachHooks.push(hookFunction);
      },
      
      // Before all hook
      beforeAll(hookFunction) {
        beforeAllHooks.push(hookFunction);
      },
      
      // After all hook
      afterAll(hookFunction) {
        afterAllHooks.push(hookFunction);
      },
      
      // Асинхронное выполнение тестов
      async run() {
        console.log('Запуск тестов...');
        const startTime = performance.now();
        
        // Выполняем before all hooks
        for (const hook of beforeAllHooks) {
          await hook();
        }
        
        // Фильтруем тесты (если есть .only)
        const filteredTests = tests.filter(t => !t.only || tests.some(tt => tt.only));
        const onlyTests = filteredTests.filter(t => t.only);
        const testsToRun = onlyTests.length > 0 ? onlyTests : filteredTests;
        
        // Выполняем тесты
        for (const test of testsToRun) {
          if (test.skip) {
            console.log(`Пропущен: ${test.description}`);
            testStats.skipped++;
            continue;
          }
          
          try {
            // Выполняем before each hooks
            for (const hook of beforeEachHooks) {
              await hook();
            }
            
            const testStartTime = performance.now();
            await test.testFunction();
            const testTime = performance.now() - testStartTime;
            
            console.log(`✓ ${test.description} (${testTime.toFixed(2)}ms)`);
            testStats.passed++;
            
          } catch (error) {
            console.log(`✗ ${test.description}`);
            console.error(error);
            testStats.failed++;
          } finally {
            // Выполняем after each hooks
            for (const hook of afterEachHooks) {
              await hook();
            }
          }
        }
        
        // Выполняем after all hooks
        for (const hook of afterAllHooks) {
          await hook();
        }
        
        const totalTime = performance.now() - startTime;
        testStats.totalTime = totalTime;
        
        this.printStats();
        return testStats;
      },
      
      // Печать статистики
      printStats() {
        console.log('\n=== Статистика тестов ===');
        console.log(`Пройдено: ${testStats.passed}`);
        console.log(`Провалено: ${testStats.failed}`);
        console.log(`Пропущено: ${testStats.skipped}`);
        console.log(`Всего: ${testStats.passed + testStats.failed + testStats.skipped}`);
        console.log(`Время выполнения: ${testStats.totalTime.toFixed(2)}ms`);
        
        if (testStats.failed === 0) {
          console.log('🎉 Все тесты пройдены!');
        } else {
          console.log('❌ Есть проваленные тесты');
        }
      },
      
      // Получение статистики
      getStats() {
        return { ...testStats };
      }
    };
  }
  
  // Создание mock объектов для тестирования
  static createMockFactory() {
    const mocks = new Map();
    
    return {
      // Создание mock функции
      createMockFunction(implementation) {
        const calls = [];
        const results = [];
        
        const mockFunction = function(...args) {
          calls.push([...args]);
          
          let result;
          if (implementation) {
            result = implementation.apply(this, args);
          }
          
          results.push(result);
          return result;
        };
        
        mockFunction.calls = calls;
        mockFunction.results = results;
        mockFunction.called = false;
        mockFunction.callCount = 0;
        
        // Методы для проверки вызовов
        mockFunction.wasCalled = () => calls.length > 0;
        mockFunction.getCallCount = () => calls.length;
        mockFunction.getCall = (index) => calls[index];
        mockFunction.getCallArgs = (index) => calls[index] || [];
        mockFunction.wasCalledWith = (...expectedArgs) => {
          return calls.some(callArgs => 
            expectedArgs.every((arg, index) => callArgs[index] === arg)
          );
        };
        
        // Установка возвращаемого значения
        mockFunction.mockReturnValue = (value) => {
          implementation = () => value;
        };
        
        // Установка реализации
        mockFunction.mockImplementation = (fn) => {
          implementation = fn;
        };
        
        return mockFunction;
      },
      
      // Создание mock объекта
      createMockObject(methods = []) {
        const mockObject = {};
        
        methods.forEach(methodName => {
          mockObject[methodName] = this.createMockFunction();
        });
        
        return mockObject;
      },
      
      // Создание spy (наблюдателя)
      createSpy(target, methodName) {
        const originalMethod = target[methodName];
        const mockFunction = this.createMockFunction();
        
        target[methodName] = function(...args) {
          mockFunction(...args);
          return originalMethod.apply(this, args);
        };
        
        target[methodName].mock = mockFunction;
        return target[methodName];
      }
    };
  }
  
  // Оптимизация тестовых данных
  static createTestDataFactory() {
    return {
      // Генерация тестовых данных
      generateTestData(type, count = 10) {
        const data = [];
        
        for (let i = 0; i < count; i++) {
          switch (type) {
            case 'user':
              data.push({
                id: i + 1,
                name: `User ${i + 1}`,
                email: `user${i + 1}@example.com`,
                age: Math.floor(Math.random() * 50) + 18
              });
              break;
            case 'product':
              data.push({
                id: i + 1,
                name: `Product ${i + 1}`,
                price: Math.random() * 100,
                category: ['Electronics', 'Clothing', 'Books'][Math.floor(Math.random() * 3)]
              });
              break;
            case 'order':
              data.push({
                id: i + 1,
                userId: Math.floor(Math.random() * 100) + 1,
                productId: Math.floor(Math.random() * 50) + 1,
                quantity: Math.floor(Math.random() * 10) + 1,
                date: new Date(Date.now() - Math.random() * 365 * 24 * 60 * 60 * 1000)
              });
              break;
          }
        }
        
        return data;
      },
      
      // Создание фикстур
      createFixture(data) {
        return JSON.parse(JSON.stringify(data));
      },
      
      // Генерация больших наборов данных
      generateLargeDataset(size = 10000) {
        const data = [];
        for (let i = 0; i < size; i++) {
          data.push({
            id: i,
            value: Math.random(),
            timestamp: Date.now() - Math.random() * 1000000
          });
        }
        return data;
      }
    };
  }
}

// Пример использования оптимизированного тестирования
class Calculator {
  add(a, b) {
    return a + b;
  }
  
  multiply(a, b) {
    return a * b;
  }
  
  divide(a, b) {
    if (b === 0) {
      throw new Error('Division by zero');
    }
    return a / b;
  }
}

// Создаем тестовый фреймворк
const testFramework = UnitTestOptimization.createOptimizedTestFramework();
const mockFactory = UnitTestOptimization.createMockFactory();
const testDataFactory = UnitTestOptimization.createTestDataFactory();

// Тесты для калькулятора
testFramework.describe('Calculator', () => {
  let calculator;
  
  testFramework.beforeEach(() => {
    calculator = new Calculator();
  });
  
  testFramework.test('should add two numbers correctly', () => {
    const result = calculator.add(2, 3);
    if (result !== 5) {
      throw new Error(`Expected 5, but got ${result}`);
    }
  });
  
  testFramework.test('should multiply two numbers correctly', () => {
    const result = calculator.multiply(3, 4);
    if (result !== 12) {
      throw new Error(`Expected 12, but got ${result}`);
    }
  });
  
  testFramework.test('should throw error when dividing by zero', () => {
    try {
      calculator.divide(10, 0);
      throw new Error('Should have thrown an error');
    } catch (error) {
      if (error.message !== 'Division by zero') {
        throw error;
      }
    }
  });
  
  testFramework.test('should divide numbers correctly', () => {
    const result = calculator.divide(10, 2);
    if (result !== 5) {
      throw new Error(`Expected 5, but got ${result}`);
    }
  });
});

// Асинхронные тесты
testFramework.describe('Async Operations', () => {
  testFramework.test('should handle async operations', async () => {
    const asyncOperation = () => {
      return new Promise(resolve => {
        setTimeout(() => resolve('success'), 100);
      });
    };
    
    const result = await asyncOperation();
    if (result !== 'success') {
      throw new Error(`Expected 'success', but got ${result}`);
    }
  });
  
  testFramework.test('should handle async errors', async () => {
    const asyncOperation = () => {
      return new Promise((_, reject) => {
        setTimeout(() => reject(new Error('async error')), 50);
      });
    };
    
    try {
      await asyncOperation();
      throw new Error('Should have thrown an error');
    } catch (error) {
      if (error.message !== 'async error') {
        throw error;
      }
    }
  });
});

// Тестирование с mock объектами
testFramework.describe('Mock Testing', () => {
  testFramework.test('should use mock functions', () => {
    const mockFunction = mockFactory.createMockFunction((a, b) => a + b);
    
    const result = mockFunction(2, 3);
    
    if (result !== 5) {
      throw new Error('Mock function should return 5');
    }
    
    if (!mockFunction.wasCalled()) {
      throw new Error('Mock function should have been called');
    }
    
    if (mockFunction.getCallCount() !== 1) {
      throw new Error('Mock function should have been called once');
    }
    
    if (!mockFunction.wasCalledWith(2, 3)) {
      throw new Error('Mock function should have been called with 2, 3');
    }
  });
});
```

## Оптимизация производительности тестов

Ускорение выполнения тестов:

```javascript
// Оптимизация производительности тестов
class TestPerformanceOptimization {
  // Параллельное выполнение тестов
  static createParallelTestRunner() {
    return {
      async runTestsInParallel(tests, maxConcurrency = 4) {
        const results = [];
        const executing = [];
        
        for (let i = 0; i < tests.length; i++) {
          const test = tests[i];
          const promise = this.runTest(test).then(result => {
            results[i] = result;
          });
          
          executing.push(promise);
          
          if (executing.length >= maxConcurrency) {
            await Promise.race(executing);
            // Удаляем завершенные промисы
            const index = executing.findIndex(p => p === promise);
            if (index !== -1) {
              executing.splice(index, 1);
            }
          }
        }
        
        await Promise.all(executing);
        return results;
      },
      
      async runTest(test) {
        const startTime = performance.now();
        
        try {
          await test.function();
          return {
            name: test.name,
            status: 'passed',
            time: performance.now() - startTime
          };
        } catch (error) {
          return {
            name: test.name,
            status: 'failed',
            error: error.message,
            time: performance.now() - startTime
          };
        }
      },
      
      // Группировка тестов по времени выполнения
      groupTestsByDuration(tests) {
        const fastTests = [];
        const mediumTests = [];
        const slowTests = [];
        
        tests.forEach(test => {
          if (test.averageDuration < 100) {
            fastTests.push(test);
          } else if (test.averageDuration < 1000) {
            mediumTests.push(test);
          } else {
            slowTests.push(test);
          }
        });
        
        return { fastTests, mediumTests, slowTests };
      }
    };
  }
  
  // Кэширование тестовых данных
  static createTestDataCache() {
    const cache = new Map();
    const ttl = 5 * 60 * 1000; // 5 минут
    
    return {
      set(key, data, customTTL = ttl) {
        cache.set(key, {
          data,
          timestamp: Date.now(),
          ttl: customTTL
        });
      },
      
      get(key) {
        const item = cache.get(key);
        if (!item) return undefined;
        
        if (Date.now() - item.timestamp > item.ttl) {
          cache.delete(key);
          return undefined;
        }
        
        return item.data;
      },
      
      clear() {
        cache.clear();
      },
      
      // Создание тестовых данных с кэшированием
      createCachedTestData(factoryFunction, key, ...args) {
        let data = this.get(key);
        if (!data) {
          data = factoryFunction(...args);
          this.set(key, data);
        }
        return data;
      }
    };
  }
  
  // Оптимизация базы данных для тестов
  static createTestDatabaseOptimizer() {
    const inMemoryDatabases = new Map();
    
    return {
      // Создание in-memory базы данных
      createInMemoryDatabase(name) {
        if (inMemoryDatabases.has(name)) {
          return inMemoryDatabases.get(name);
        }
        
        const db = {
          data: new Map(),
          collections: new Map(),
          
          collection(name) {
            if (!this.collections.has(name)) {
              this.collections.set(name, new Map());
            }
            return this.collections.get(name);
          },
          
          async insert(collectionName, document) {
            const collection = this.collection(collectionName);
            const id = document.id || Date.now().toString();
            collection.set(id, { ...document, id });
            return { ...document, id };
          },
          
          async find(collectionName, query = {}) {
            const collection = this.collection(collectionName);
            const results = [];
            
            for (const [id, doc] of collection) {
              let matches = true;
              for (const [key, value] of Object.entries(query)) {
                if (doc[key] !== value) {
                  matches = false;
                  break;
                }
              }
              if (matches) {
                results.push(doc);
              }
            }
            
            return results;
          },
          
          async update(collectionName, query, update) {
            const collection = this.collection(collectionName);
            let updatedCount = 0;
            
            for (const [id, doc] of collection) {
              let matches = true;
              for (const [key, value] of Object.entries(query)) {
                if (doc[key] !== value) {
                  matches = false;
                  break;
                }
              }
              
              if (matches) {
                const updatedDoc = { ...doc, ...update };
                collection.set(id, updatedDoc);
                updatedCount++;
              }
            }
            
            return { modifiedCount: updatedCount };
          },
          
          async delete(collectionName, query) {
            const collection = this.collection(collectionName);
            let deletedCount = 0;
            
            for (const [id, doc] of collection) {
              let matches = true;
              for (const [key, value] of Object.entries(query)) {
                if (doc[key] !== value) {
                  matches = false;
                  break;
                }
              }
              
              if (matches) {
                collection.delete(id);
                deletedCount++;
              }
            }
            
            return { deletedCount };
          },
          
          clear() {
            this.collections.clear();
          }
        };
        
        inMemoryDatabases.set(name, db);
        return db;
      },
      
      // Получение существующей базы данных
      getDatabase(name) {
        return inMemoryDatabases.get(name);
      },
      
      // Очистка всех баз данных
      clearAllDatabases() {
        inMemoryDatabases.forEach(db => db.clear());
      }
    };
  }
  
  // Оптимизация загрузки модулей
  static createModuleLoader() {
    const loadedModules = new Map();
    const loadingModules = new Map();
    
    return {
      async loadModule(modulePath) {
        // Проверяем кэш
        if (loadedModules.has(modulePath)) {
          return loadedModules.get(modulePath);
        }
        
        // Проверяем загрузку
        if (loadingModules.has(modulePath)) {
          return loadingModules.get(modulePath);
        }
        
        // Начинаем загрузку
        const loadPromise = import(modulePath).then(module => {
          loadedModules.set(modulePath, module);
          loadingModules.delete(modulePath);
          return module;
        }).catch(error => {
          loadingModules.delete(modulePath);
          throw error;
        });
        
        loadingModules.set(modulePath, loadPromise);
        return loadPromise;
      },
      
      // Предзагрузка модулей
      async preloadModules(modulePaths) {
        return Promise.all(
          modulePaths.map(path => this.loadModule(path))
        );
      },
      
      // Очистка кэша модулей
      clearModuleCache() {
        loadedModules.clear();
        loadingModules.clear();
      }
    };
  }
}

// Пример использования оптимизаций производительности тестов
class UserService {
  constructor(database) {
    this.db = database;
  }
  
  async createUser(userData) {
    return await this.db.insert('users', userData);
  }
  
  async findUser(query) {
    const users = await this.db.find('users', query);
    return users[0] || null;
  }
  
  async updateUser(userId, updateData) {
    return await this.db.update('users', { id: userId }, updateData);
  }
  
  async deleteUser(userId) {
    return await this.db.delete('users', { id: userId });
  }
}

// Создаем оптимизированные инструменты
const parallelRunner = TestPerformanceOptimization.createParallelTestRunner();
const dataCache = TestPerformanceOptimization.createTestDataCache();
const dbOptimizer = TestPerformanceOptimization.createTestDatabaseOptimizer();
const moduleLoader = TestPerformanceOptimization.createModuleLoader();

// Тесты с оптимизациями производительности
const performanceTests = [
  {
    name: 'Create user test',
    function: async () => {
      const db = dbOptimizer.createInMemoryDatabase('test');
      const userService = new UserService(db);
      
      const userData = dataCache.createCachedTestData(
        () => ({ name: 'John Doe', email: 'john@example.com' }),
        'user_data'
      );
      
      const user = await userService.createUser(userData);
      if (!user.id) {
        throw new Error('User should have an ID');
      }
    }
  },
  {
    name: 'Find user test',
    function: async () => {
      const db = dbOptimizer.getDatabase('test');
      const userService = new UserService(db);
      
      const user = await userService.findUser({ email: 'john@example.com' });
      if (!user) {
        throw new Error('User should be found');
      }
    }
  },
  {
    name: 'Update user test',
    function: async () => {
      const db = dbOptimizer.getDatabase('test');
      const userService = new UserService(db);
      
      const result = await userService.updateUser('1', { name: 'Jane Doe' });
      if (result.modifiedCount !== 1) {
        throw new Error('User should be updated');
      }
    }
  }
];

// Параллельное выполнение тестов
async function runPerformanceTests() {
  console.log('Запуск оптимизированных тестов...');
  const results = await parallelRunner.runTestsInParallel(performanceTests, 2);
  
  console.log('\nРезультаты:');
  results.forEach(result => {
    console.log(`${result.status === 'passed' ? '✓' : '✗'} ${result.name} (${result.time.toFixed(2)}ms)`);
    if (result.status === 'failed') {
      console.log(`  Ошибка: ${result.error}`);
    }
  });
}
```

## Интеграционное тестирование

Оптимизация интеграционных тестов:

```javascript
// Оптимизация интеграционного тестирования
class IntegrationTestOptimization {
  // Создание тестовой среды
  static createTestEnvironment() {
    const services = new Map();
    const mocks = new Map();
    const spies = new Map();
    
    return {
      // Регистрация сервиса
      registerService(name, service) {
        services.set(name, service);
      },
      
      // Получение сервиса
      getService(name) {
        return services.get(name);
      },
      
      // Создание mock сервиса
      createMockService(name, methods = {}) {
        const mockService = {};
        
        Object.keys(methods).forEach(methodName => {
          const mockFunction = UnitTestOptimization.createMockFactory().createMockFunction(methods[methodName]);
          mockService[methodName] = mockFunction;
        });
        
        mocks.set(name, mockService);
        services.set(name, mockService);
        return mockService;
      },
      
      // Создание spy для метода
      createSpy(serviceName, methodName) {
        const service = services.get(serviceName);
        if (!service) {
          throw new Error(`Service ${serviceName} not found`);
        }
        
        const originalMethod = service[methodName];
        const mockFunction = UnitTestOptimization.createMockFactory().createMockFunction();
        
        service[methodName] = function(...args) {
          mockFunction(...args);
          return originalMethod.apply(this, args);
        };
        
        const spyKey = `${serviceName}.${methodName}`;
        spies.set(spyKey, {
          service,
          methodName,
          mockFunction,
          originalMethod
        });
        
        return mockFunction;
      },
      
      // Получение spy
      getSpy(serviceName, methodName) {
        const spyKey = `${serviceName}.${methodName}`;
        const spy = spies.get(spyKey);
        return spy ? spy.mockFunction : null;
      },
      
      // Очистка среды
      reset() {
        services.clear();
        mocks.clear();
        spies.forEach(spy => {
          spy.service[spy.methodName] = spy.originalMethod;
        });
        spies.clear();
      },
      
      // Создание тестовой транзакции
      createTestTransaction() {
        const operations = [];
        
        return {
          addOperation(operation) {
            operations.push(operation);
          },
          
          async execute() {
            const results = [];
            for (const operation of operations) {
              try {
                const result = await operation();
                results.push({ success: true, result });
              } catch (error) {
                results.push({ success: false, error: error.message });
              }
            }
            return results;
          },
          
          async rollback() {
            // В реальной реализации здесь будет логика отката
            console.log('Откат транзакции');
          }
        };
      }
    };
  }
  
  // Создание тестовых сценариев
  static createTestScenario() {
    const steps = [];
    const setupSteps = [];
    const teardownSteps = [];
    
    return {
      // Добавление шага настройки
      addSetupStep(stepFunction, description) {
        setupSteps.push({ function: stepFunction, description });
      },
      
      // Добавление шага теста
      addStep(stepFunction, description) {
        steps.push({ function: stepFunction, description });
      },
      
      // Добавление шага очистки
      addTeardownStep(stepFunction, description) {
        teardownSteps.push({ function: stepFunction, description });
      },
      
      // Выполнение сценария
      async execute() {
        const results = {
          setup: [],
          steps: [],
          teardown: []
        };
        
        try {
          // Выполняем шаги настройки
          for (const step of setupSteps) {
            try {
              await step.function();
              results.setup.push({ description: step.description, status: 'passed' });
            } catch (error) {
              results.setup.push({ 
                description: step.description, 
                status: 'failed', 
                error: error.message 
              });
              throw error; // Прерываем выполнение при ошибке настройки
            }
          }
          
          // Выполняем основные шаги
          for (const step of steps) {
            try {
              await step.function();
              results.steps.push({ description: step.description, status: 'passed' });
            } catch (error) {
              results.steps.push({ 
                description: step.description, 
                status: 'failed', 
                error: error.message 
              });
            }
          }
          
        } finally {
          // Выполняем шаги очистки
          for (const step of teardownSteps) {
            try {
              await step.function();
              results.teardown.push({ description: step.description, status: 'passed' });
            } catch (error) {
              results.teardown.push({ 
                description: step.description, 
                status: 'failed', 
                error: error.message 
              });
            }
          }
        }
        
        return results;
      },
      
      // Получение статистики
      getStats(results) {
        const totalSteps = results.steps.length;
        const passedSteps = results.steps.filter(s => s.status === 'passed').length;
        const failedSteps = results.steps.filter(s => s.status === 'failed').length;
        
        return {
          total: totalSteps,
          passed: passedSteps,
          failed: failedSteps,
          successRate: totalSteps > 0 ? (passedSteps / totalSteps) * 100 : 0
        };
      }
    };
  }
  
  // Создание тестовых данных для интеграционных тестов
  static createIntegrationTestData() {
    return {
      // Создание тестовой пользовательской сессии
      createUserSession(userData = {}) {
        return {
          userId: userData.id || Math.floor(Math.random() * 10000),
          sessionId: `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
          createdAt: new Date(),
          expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 часа
          userData: {
            name: userData.name || 'Test User',
            email: userData.email || 'test@example.com',
            ...userData
          }
        };
      },
      
      // Создание тестового HTTP запроса
      createHttpRequest(method, url, body = null, headers = {}) {
        return {
          method: method.toUpperCase(),
          url,
          headers: {
            'Content-Type': 'application/json',
            ...headers
          },
          body: body ? JSON.stringify(body) : null
        };
      },
      
      // Создание тестового HTTP ответа
      createHttpResponse(status, data = null, headers = {}) {
        return {
          status,
          statusText: this.getStatusText(status),
          headers: {
            'Content-Type': 'application/json',
            ...headers
          },
          data: data ? JSON.stringify(data) : null,
          json: () => data
        };
      },
      
      getStatusText(status) {
        const statusTexts = {
          200: 'OK',
          201: 'Created',
          400: 'Bad Request',
          401: 'Unauthorized',
          404: 'Not Found',
          500: 'Internal Server Error'
        };
        return statusTexts[status] || 'Unknown';
      },
      
      // Создание тестовой очереди сообщений
      createMessageQueue() {
        const queue = [];
        const subscribers = new Map();
        
        return {
          publish(topic, message) {
            queue.push({ topic, message, timestamp: Date.now() });
            
            // Уведомляем подписчиков
            if (subscribers.has(topic)) {
              subscribers.get(topic).forEach(callback => {
                try {
                  callback(message);
                } catch (error) {
                  console.error('Ошибка в подписчике:', error);
                }
              });
            }
          },
          
          subscribe(topic, callback) {
            if (!subscribers.has(topic)) {
              subscribers.set(topic, new Set());
            }
            subscribers.get(topic).add(callback);
          },
          
          unsubscribe(topic, callback) {
            if (subscribers.has(topic)) {
              subscribers.get(topic).delete(callback);
            }
          },
          
          getMessages(topic) {
            return queue.filter(msg => msg.topic === topic);
          },
          
          clear() {
            queue.length = 0;
            subscribers.clear();
          }
        };
      }
    };
  }
}

// Пример интеграционного теста
class OrderService {
  constructor(userService, paymentService, inventoryService) {
    this.userService = userService;
    this.paymentService = paymentService;
    this.inventoryService = inventoryService;
  }
  
  async createOrder(userId, items) {
    // Проверяем пользователя
    const user = await this.userService.findUser({ id: userId });
    if (!user) {
      throw new Error('User not found');
    }
    
    // Проверяем наличие товаров
    for (const item of items) {
      const product = await this.inventoryService.getProduct(item.productId);
      if (!product || product.stock < item.quantity) {
        throw new Error(`Insufficient stock for product ${item.productId}`);
      }
    }
    
    // Обрабатываем платеж
    const totalAmount = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const paymentResult = await this.paymentService.processPayment(userId, totalAmount);
    if (!paymentResult.success) {
      throw new Error('Payment failed');
    }
    
    // Создаем заказ
    const order = {
      id: `order_${Date.now()}`,
      userId,
      items,
      totalAmount,
      status: 'completed',
      createdAt: new Date()
    };
    
    return order;
  }
}

// Создаем тестовую среду
const testEnv = IntegrationTestOptimization.createTestEnvironment();
const testData = IntegrationTestOptimization.createIntegrationTestData();

// Настраиваем mock сервисы
testEnv.createMockService('userService', {
  findUser: (query) => {
    if (query.id === 1) {
      return { id: 1, name: 'John Doe', email: 'john@example.com' };
    }
    return null;
  }
});

testEnv.createMockService('paymentService', {
  processPayment: (userId, amount) => {
    return { success: true, transactionId: `txn_${Date.now()}` };
  }
});

testEnv.createMockService('inventoryService', {
  getProduct: (productId) => {
    if (productId === 1) {
      return { id: 1, name: 'Product 1', price: 100, stock: 10 };
    }
    return null;
  }
});

// Создаем сценарий теста
const orderScenario = IntegrationTestOptimization.createTestScenario();

orderScenario.addSetupStep(async () => {
  console.log('Настройка тестовой среды для заказа');
}, 'Setup test environment');

orderScenario.addStep(async () => {
  const orderService = new OrderService(
    testEnv.getService('userService'),
    testEnv.getService('paymentService'),
    testEnv.getService('inventoryService')
  );
  
  const order = await orderService.createOrder(1, [
    { productId: 1, quantity: 2, price: 100 }
  ]);
  
  if (!order.id) {
    throw new Error('Order should have an ID');
  }
  
  if (order.status !== 'completed') {
    throw new Error('Order should be completed');
  }
  
  console.log('Заказ успешно создан:', order.id);
}, 'Create order');

orderScenario.addStep(async () => {
  // Проверяем, что были вызваны все необходимые сервисы
  const userServiceSpy = testEnv.getSpy('userService', 'findUser');
  const paymentServiceSpy = testEnv.getSpy('paymentService', 'processPayment');
  const inventoryServiceSpy = testEnv.getSpy('inventoryService', 'getProduct');
  
  if (!userServiceSpy || !userServiceSpy.wasCalled()) {
    throw new Error('User service should have been called');
  }
  
  if (!paymentServiceSpy || !paymentServiceSpy.wasCalled()) {
    throw new Error('Payment service should have been called');
  }
  
  if (!inventoryServiceSpy || !inventoryServiceSpy.wasCalled()) {
    throw new Error('Inventory service should have been called');
  }
  
  console.log('Все сервисы были вызваны корректно');
}, 'Verify service calls');

orderScenario.addTeardownStep(async () => {
  testEnv.reset();
  console.log('Тестовая среда очищена');
}, 'Cleanup test environment');

// Выполняем сценарий
async function runIntegrationTest() {
  console.log('Запуск интеграционного теста...');
  
  const results = await orderScenario.execute();
  const stats = orderScenario.getStats(results);
  
  console.log('\nРезультаты интеграционного теста:');
  console.log(`Успешно: ${stats.passed}`);
  console.log(`Провалено: ${stats.failed}`);
  console.log(`Процент успеха: ${stats.successRate.toFixed(2)}%`);
  
  if (stats.failed === 0) {
    console.log('🎉 Все интеграционные тесты пройдены!');
  } else {
    console.log('❌ Есть проваленные интеграционные тесты');
  }
  
  return results;
}
```

## Практические рекомендации

1. **Используйте параллельное выполнение** - для ускорения тестов
2. **Применяйте кэширование** - для тестовых данных
3. **Используйте in-memory базы данных** - для тестов
4. **Создавайте mock объекты** - для изоляции тестов
5. **Оптимизируйте загрузку модулей** - с предзагрузкой
6. **Используйте тестовые сценарии** - для сложных интеграционных тестов
7. **Применяйте spies и stubs** - для проверки взаимодействия
8. **Мониторьте производительность** - измеряйте время выполнения

Оптимизация тестирования - важный аспект обеспечения качества кода. Правильное использование различных техник оптимизации помогает создавать надежные и эффективные тесты.

#javascript #testing #optimization #performance #unit-tests #integration-tests #mocks #spies