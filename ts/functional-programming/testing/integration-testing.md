# Интеграционное тестирование функционального кода

Интеграционное тестирование в функциональном программировании фокусируется на проверке взаимодействия между компонентами и модулями. В отличие от императивного программирования, функциональный подход упрощает интеграционное тестирование благодаря отсутствию изменяемого состояния и побочных эффектов.

## Содержание

- [Интеграционное тестирование функционального кода](#интеграционное-тестирование-функционального-кода)
  - [Содержание](#содержание)
  - [Основы интеграционного тестирования](#основы-интеграционного-тестирования)
  - [Тестирование композиции модулей](#тестирование-композиции-модулей)
  - [Тестирование потоков данных](#тестирование-потоков-данных)
  - [Тестирование с внешними зависимостями](#тестирование-с-внешними-зависимостями)
  - [Тестирование обработчиков событий](#тестирование-обработчиков-событий)
  - [Тестирование систем с состоянием](#тестирование-систем-с-состоянием)
  - [Тестирование асинхронных операций](#тестирование-асинхронных-операций)
  - [Лучшие практики](#лучшие-практики)
    - [1. Изоляция внешних зависимостей](#1-изоляция-внешних-зависимостей)
    - [2. Тестирование потоков данных](#2-тестирование-потоков-данных)
    - [3. Тестирование сценариев ошибок](#3-тестирование-сценариев-ошибок)
    - [4. Тестирование производительности](#4-тестирование-производительности)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Основы интеграционного тестирования

Интеграционное тестирование в функциональном программировании имеет свои особенности:

1. **Минимизация побочных эффектов** - Легче изолировать и тестировать взаимодействия
2. **Чистые интерфейсы** - Компоненты взаимодействуют через чистые функции
3. **Композиция тестируемости** - Интеграция тестируемых компонентов остается тестируемой
4. **Предсказуемость** - Отсутствие изменяемого состояния делает поведение предсказуемым

```typescript
// Пример системы с чистыми интерфейсами
interface UserRepository {
  findById: (id: number) => Promise<User | null>;
  save: (user: User) => Promise<User>;
}

interface EmailService {
  sendWelcomeEmail: (user: User) => Promise<boolean>;
}

interface Logger {
  info: (message: string) => void;
  error: (message: string) => void;
}

// Сервис пользователей
class UserService {
  constructor(
    private userRepository: UserRepository,
    private emailService: EmailService,
    private logger: Logger
  ) {}
  
  async createUser(userData: CreateUserRequest): Promise<User | null> {
    this.logger.info(`Создание пользователя: ${userData.email}`);
    
    try {
      const existingUser = await this.userRepository.findById(userData.id);
      if (existingUser) {
        this.logger.info(`Пользователь уже существует: ${userData.email}`);
        return existingUser;
      }
      
      const user: User = {
        id: userData.id,
        name: userData.name,
        email: userData.email,
        createdAt: new Date()
      };
      
      const savedUser = await this.userRepository.save(user);
      this.logger.info(`Пользователь создан: ${savedUser.email}`);
      
      const emailSent = await this.emailService.sendWelcomeEmail(savedUser);
      if (emailSent) {
        this.logger.info(`Приветственный email отправлен: ${savedUser.email}`);
      } else {
        this.logger.error(`Не удалось отправить приветственный email: ${savedUser.email}`);
      }
      
      return savedUser;
    } catch (error) {
      this.logger.error(`Ошибка создания пользователя: ${error}`);
      return null;
    }
  }
}

// Модели данных
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface CreateUserRequest {
  id: number;
  name: string;
  email: string;
}

// Тестирование интеграции
describe("Интеграционное тестирование UserService", () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;
  let mockEmailService: jest.Mocked<EmailService>;
  let mockLogger: jest.Mocked<Logger>;
  
  beforeEach(() => {
    mockUserRepository = {
      findById: jest.fn(),
      save: jest.fn()
    };
    
    mockEmailService = {
      sendWelcomeEmail: jest.fn()
    };
    
    mockLogger = {
      info: jest.fn(),
      error: jest.fn()
    };
    
    userService = new UserService(
      mockUserRepository,
      mockEmailService,
      mockLogger
    );
  });
  
  test("должен создать нового пользователя и отправить email", async () => {
    const userData: CreateUserRequest = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    const user: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com",
      createdAt: new Date()
    };
    
    mockUserRepository.findById.mockResolvedValue(null);
    mockUserRepository.save.mockResolvedValue(user);
    mockEmailService.sendWelcomeEmail.mockResolvedValue(true);
    
    const result = await userService.createUser(userData);
    
    expect(result).toEqual(user);
    expect(mockUserRepository.findById).toHaveBeenCalledWith(1);
    expect(mockUserRepository.save).toHaveBeenCalledWith(user);
    expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(user);
    expect(mockLogger.info).toHaveBeenCalledWith("Создание пользователя: ivan@example.com");
    expect(mockLogger.info).toHaveBeenCalledWith("Пользователь создан: ivan@example.com");
    expect(mockLogger.info).toHaveBeenCalledWith("Приветственный email отправлен: ivan@example.com");
  });
  
  test("должен вернуть существующего пользователя", async () => {
    const userData: CreateUserRequest = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    const existingUser: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com",
      createdAt: new Date()
    };
    
    mockUserRepository.findById.mockResolvedValue(existingUser);
    
    const result = await userService.createUser(userData);
    
    expect(result).toEqual(existingUser);
    expect(mockUserRepository.findById).toHaveBeenCalledWith(1);
    expect(mockUserRepository.save).not.toHaveBeenCalled();
    expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
  });
  
  test("должен обработать ошибку при сохранении пользователя", async () => {
    const userData: CreateUserRequest = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    mockUserRepository.findById.mockResolvedValue(null);
    mockUserRepository.save.mockRejectedValue(new Error("Ошибка БД"));
    
    const result = await userService.createUser(userData);
    
    expect(result).toBeNull();
    expect(mockLogger.error).toHaveBeenCalledWith(
      expect.stringContaining("Ошибка создания пользователя")
    );
  });
});
```

## Тестирование композиции модулей

Композиция модулей - ключевая концепция функционального программирования, требующая тщательного тестирования.

```typescript
// Модули для композиции
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

interface OrderItem {
  productId: number;
  quantity: number;
}

interface Order {
  id: number;
  items: OrderItem[];
  total: number;
}

// Сервисы
class ProductService {
  constructor(private products: Product[]) {}
  
  findProduct(id: number): Product | undefined {
    return this.products.find(p => p.id === id);
  }
  
  getProductsByCategory(category: string): Product[] {
    return this.products.filter(p => p.category === category);
  }
}

class OrderService {
  constructor(private productService: ProductService) {}
  
  calculateOrderTotal(items: OrderItem[]): number {
    return items.reduce((total, item) => {
      const product = this.productService.findProduct(item.productId);
      return total + (product ? product.price * item.quantity : 0);
    }, 0);
  }
  
  createOrder(items: OrderItem[]): Order {
    const total = this.calculateOrderTotal(items);
    return {
      id: Date.now(),
      items,
      total
    };
  }
}

class ReportService {
  constructor(private orderService: OrderService) {}
  
  generateSalesReport(orders: Order[]): SalesReport {
    const totalRevenue = orders.reduce((sum, order) => sum + order.total, 0);
    const totalOrders = orders.length;
    
    return {
      totalRevenue,
      totalOrders,
      averageOrderValue: totalOrders > 0 ? totalRevenue / totalOrders : 0
    };
  }
}

interface SalesReport {
  totalRevenue: number;
  totalOrders: number;
  averageOrderValue: number;
}

// Интеграционные тесты композиции
describe("Интеграционное тестирование композиции модулей", () => {
  let productService: ProductService;
  let orderService: OrderService;
  let reportService: ReportService;
  
  beforeEach(() => {
    const products: Product[] = [
      { id: 1, name: "Ноутбук", price: 50000, category: "Электроника" },
      { id: 2, name: "Смартфон", price: 30000, category: "Электроника" },
      { id: 3, name: "Книга", price: 500, category: "Книги" }
    ];
    
    productService = new ProductService(products);
    orderService = new OrderService(productService);
    reportService = new ReportService(orderService);
  });
  
  test("должен корректно создавать заказы и генерировать отчеты", () => {
    const order1 = orderService.createOrder([
      { productId: 1, quantity: 1 }, // Ноутбук - 50000
      { productId: 3, quantity: 2 }  // Книги - 1000
    ]);
    
    const order2 = orderService.createOrder([
      { productId: 2, quantity: 1 } // Смартфон - 30000
    ]);
    
    const report = reportService.generateSalesReport([order1, order2]);
    
    expect(order1.total).toBe(51000); // 50000 + 1000
    expect(order2.total).toBe(30000);
    expect(report.totalRevenue).toBe(81000); // 51000 + 30000
    expect(report.totalOrders).toBe(2);
    expect(report.averageOrderValue).toBe(40500); // 81000 / 2
  });
  
  test("должен корректно обрабатывать несуществующие продукты", () => {
    const order = orderService.createOrder([
      { productId: 999, quantity: 1 }, // Несуществующий продукт
      { productId: 1, quantity: 1 }   // Ноутбук - 50000
    ]);
    
    expect(order.total).toBe(50000); // Только стоимость ноутбука
  });
  
  test("должен генерировать отчет для пустого списка заказов", () => {
    const report = reportService.generateSalesReport([]);
    
    expect(report.totalRevenue).toBe(0);
    expect(report.totalOrders).toBe(0);
    expect(report.averageOrderValue).toBe(0);
  });
});
```

## Тестирование потоков данных

Потоки данных в функциональном программировании представляют собой цепочки преобразований, которые требуют комплексного тестирования.

```typescript
// Модели данных для потоков
interface RawUserData {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  birthDate: string;
  salary: string;
}

interface ProcessedUser {
  id: number;
  fullName: string;
  email: string;
  age: number;
  salary: number;
}

interface UserReport {
  totalUsers: number;
  averageAge: number;
  averageSalary: number;
  salaryDistribution: Record<string, number>;
}

// Функции обработки данных
const parseId = (data: RawUserData): RawUserData & { id: number } => ({
  ...data,
  id: parseInt(data.id, 10)
});

const formatName = (data: RawUserData & { id: number }): RawUserData & { id: number, fullName: string } => ({
  ...data,
  fullName: `${data.firstName.trim()} ${data.lastName.trim()}`
});

const calculateAge = (data: RawUserData & { id: number, fullName: string }): RawUserData & { id: number, fullName: string, age: number } => ({
  ...data,
  age: new Date().getFullYear() - new Date(data.birthDate).getFullYear()
});

const parseSalary = (data: RawUserData & { id: number, fullName: string, age: number }): ProcessedUser => ({
  ...data,
  salary: parseFloat(data.salary)
});

// Композиция обработки
const processUser = (data: RawUserData): ProcessedUser => 
  pipe(parseId, formatName, calculateAge, parseSalary)(data);

const processUsers = (users: RawUserData[]): ProcessedUser[] => 
  users.map(processUser);

// Функции генерации отчетов
const calculateAverage = (numbers: number[]): number => {
  if (numbers.length === 0) return 0;
  return numbers.reduce((sum, num) => sum + num, 0) / numbers.length;
};

const categorizeSalary = (salary: number): string => {
  if (salary < 30000) return "низкая";
  if (salary < 70000) return "средняя";
  return "высокая";
};

const generateSalaryDistribution = (users: ProcessedUser[]): Record<string, number> => {
  const distribution: Record<string, number> = { низкая: 0, средняя: 0, высокая: 0 };
  
  users.forEach(user => {
    const category = categorizeSalary(user.salary);
    distribution[category] = (distribution[category] || 0) + 1;
  });
  
  return distribution;
};

const generateUserReport = (users: ProcessedUser[]): UserReport => {
  const ages = users.map(user => user.age);
  const salaries = users.map(user => user.salary);
  
  return {
    totalUsers: users.length,
    averageAge: calculateAverage(ages),
    averageSalary: calculateAverage(salaries),
    salaryDistribution: generateSalaryDistribution(users)
  };
};

// Утилиты для композиции
const pipe = <A, B, C, D, E>(f: (a: A) => B, g: (b: B) => C, h: (c: C) => D, i: (d: D) => E): ((a: A) => E) => 
  (a: A) => i(h(g(f(a))));

// Интеграционные тесты потоков данных
describe("Интеграционное тестирование потоков данных", () => {
  const rawUsers: RawUserData[] = [
    {
      id: "1",
      firstName: "Иван  ",
      lastName: "  Иванов",
      email: "ivan@example.com",
      birthDate: "1990-05-15",
      salary: "50000.50"
    },
    {
      id: "2",
      firstName: "Мария",
      lastName: "Петрова",
      email: "maria@example.com",
      birthDate: "1985-12-03",
      salary: "45000.00"
    },
    {
      id: "3",
      firstName: "Алексей",
      lastName: "Сидоров",
      email: "alex@example.com",
      birthDate: "1992-03-20",
      salary: "80000.75"
    }
  ];
  
  test("должен корректно обрабатывать поток данных пользователей", () => {
    const processedUsers = processUsers(rawUsers);
    
    expect(processedUsers).toHaveLength(3);
    
    const ivan = processedUsers[0];
    expect(ivan.id).toBe(1);
    expect(ivan.fullName).toBe("Иван Иванов");
    expect(ivan.email).toBe("ivan@example.com");
    expect(ivan.age).toBe(35); // 2025 - 1990
    expect(ivan.salary).toBe(50000.50);
  });
  
  test("должен корректно генерировать отчеты", () => {
    const processedUsers = processUsers(rawUsers);
    const report = generateUserReport(processedUsers);
    
    expect(report.totalUsers).toBe(3);
    expect(report.averageAge).toBe(32); // (35 + 40 + 33) / 3 = 36
    expect(report.averageSalary).toBe(58333.75); // (50000.50 + 45000 + 80000.75) / 3
    expect(report.salaryDistribution).toEqual({
      низкая: 0,
      средняя: 2,
      высокая: 1
    });
  });
  
  test("должен корректно обрабатывать пустые данные", () => {
    const processedUsers = processUsers([]);
    const report = generateUserReport([]);
    
    expect(processedUsers).toHaveLength(0);
    expect(report.totalUsers).toBe(0);
    expect(report.averageAge).toBe(0);
    expect(report.averageSalary).toBe(0);
    expect(report.salaryDistribution).toEqual({ низкая: 0, средняя: 0, высокая: 0 });
  });
  
  test("должен корректно обрабатывать некорректные данные", () => {
    const invalidUsers: RawUserData[] = [
      {
        id: "invalid",
        firstName: "Тест",
        lastName: "Тестов",
        email: "test@example.com",
        birthDate: "invalid-date",
        salary: "invalid-salary"
      }
    ];
    
    // В реальной реализации здесь были бы Either монады для обработки ошибок
    // Для этого примера предполагаем базовую обработку
    const processedUsers = processUsers(invalidUsers);
    
    expect(processedUsers[0].id).toBeNaN();
    expect(processedUsers[0].age).toBeNaN();
    expect(processedUsers[0].salary).toBeNaN();
  });
});
```

## Тестирование с внешними зависимостями

Внешние зависимости в функциональном программировании инкапсулируются в зависимости, что упрощает их тестирование.

```typescript
// Интерфейсы для внешних зависимостей
interface HttpClient {
  get: <T>(url: string) => Promise<T>;
  post: <T>(url: string, data: any) => Promise<T>;
}

interface Database {
  query: <T>(sql: string, params: any[]) => Promise<T[]>;
  execute: (sql: string, params: any[]) => Promise<void>;
}

interface Cache {
  get: <T>(key: string) => Promise<T | null>;
  set: <T>(key: string, value: T, ttl?: number) => Promise<void>;
  delete: (key: string) => Promise<void>;
}

interface Config {
  apiUrl: string;
  databaseUrl: string;
  cacheTtl: number;
}

// Сервис с внешними зависимостями
class UserServiceWithDependencies {
  constructor(
    private httpClient: HttpClient,
    private database: Database,
    private cache: Cache,
    private config: Config
  ) {}
  
  async getUser(id: number): Promise<User | null> {
    // Попытка получить из кэша
    const cachedUser = await this.cache.get<User>(`user:${id}`);
    if (cachedUser) {
      return cachedUser;
    }
    
    // Получение из базы данных
    const dbUsers = await this.database.query<User>(
      "SELECT * FROM users WHERE id = ?", 
      [id]
    );
    
    if (dbUsers.length > 0) {
      const user = dbUsers[0];
      // Сохранение в кэш
      await this.cache.set(`user:${id}`, user, this.config.cacheTtl);
      return user;
    }
    
    // Получение из внешнего API
    try {
      const apiUser = await this.httpClient.get<User>(
        `${this.config.apiUrl}/users/${id}`
      );
      
      // Сохранение в базу данных и кэш
      await this.database.execute(
        "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
        [apiUser.id, apiUser.name, apiUser.email]
      );
      
      await this.cache.set(`user:${id}`, apiUser, this.config.cacheTtl);
      return apiUser;
    } catch (error) {
      return null;
    }
  }
  
  async updateUser(user: User): Promise<boolean> {
    try {
      // Обновление в базе данных
      await this.database.execute(
        "UPDATE users SET name = ?, email = ? WHERE id = ?",
        [user.name, user.email, user.id]
      );
      
      // Обновление в кэше
      await this.cache.set(`user:${user.id}`, user, this.config.cacheTtl);
      
      // Отправка уведомления во внешнюю систему
      await this.httpClient.post(
        `${this.config.apiUrl}/notifications`,
        { type: "user_updated", userId: user.id }
      );
      
      return true;
    } catch (error) {
      return false;
    }
  }
}

// Моки для внешних зависимостей
class MockHttpClient implements HttpClient {
  private responses: Record<string, any> = {};
  
  mockGet<T>(url: string, response: T): void {
    this.responses[`GET:${url}`] = response;
  }
  
  mockPost<T>(url: string, response: T): void {
    this.responses[`POST:${url}`] = response;
  }
  
  async get<T>(url: string): Promise<T> {
    const response = this.responses[`GET:${url}`];
    if (response) {
      return Promise.resolve(response);
    }
    throw new Error(`No mock response for GET ${url}`);
  }
  
  async post<T>(url: string, data: any): Promise<T> {
    const response = this.responses[`POST:${url}`];
    if (response) {
      return Promise.resolve(response);
    }
    throw new Error(`No mock response for POST ${url}`);
  }
}

class MockDatabase implements Database {
  private tables: Record<string, any[]> = {
    users: []
  };
  
  async query<T>(sql: string, params: any[]): Promise<T[]> {
    if (sql.includes("SELECT * FROM users WHERE id = ?")) {
      const id = params[0];
      return this.tables.users.filter(user => user.id === id) as T[];
    }
    return [];
  }
  
  async execute(sql: string, params: any[]): Promise<void> {
    if (sql.includes("INSERT INTO users")) {
      const [id, name, email] = params;
      this.tables.users.push({ id, name, email });
    } else if (sql.includes("UPDATE users")) {
      const [name, email, id] = params;
      const index = this.tables.users.findIndex(user => user.id === id);
      if (index !== -1) {
        this.tables.users[index] = { id, name, email };
      }
    }
  }
}

class MockCache implements Cache {
  private store: Record<string, { value: any; expires: number }> = {};
  
  async get<T>(key: string): Promise<T | null> {
    const item = this.store[key];
    if (item && item.expires > Date.now()) {
      return item.value;
    }
    delete this.store[key];
    return null;
  }
  
  async set<T>(key: string, value: T, ttl: number = 300000): Promise<void> {
    this.store[key] = {
      value,
      expires: Date.now() + ttl
    };
  }
  
  async delete(key: string): Promise<void> {
    delete this.store[key];
  }
}

// Интеграционные тесты с внешними зависимостями
describe("Интеграционное тестирование с внешними зависимостями", () => {
  let userService: UserServiceWithDependencies;
  let mockHttpClient: MockHttpClient;
  let mockDatabase: MockDatabase;
  let mockCache: MockCache;
  let config: Config;
  
  beforeEach(() => {
    mockHttpClient = new MockHttpClient();
    mockDatabase = new MockDatabase();
    mockCache = new MockCache();
    config = {
      apiUrl: "https://api.example.com",
      databaseUrl: "postgresql://localhost:5432/testdb",
      cacheTtl: 300000
    };
    
    userService = new UserServiceWithDependencies(
      mockHttpClient,
      mockDatabase,
      mockCache,
      config
    );
  });
  
  test("должен получать пользователя из кэша", async () => {
    const user: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    await mockCache.set("user:1", user);
    
    const result = await userService.getUser(1);
    
    expect(result).toEqual(user);
  });
  
  test("должен получать пользователя из базы данных", async () => {
    const user: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    // Имитация записи в базу данных
    await mockDatabase.execute(
      "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
      [user.id, user.name, user.email]
    );
    
    const result = await userService.getUser(1);
    
    expect(result).toEqual(user);
    // Проверка, что пользователь был сохранен в кэш
    const cachedUser = await mockCache.get<User>("user:1");
    expect(cachedUser).toEqual(user);
  });
  
  test("должен получать пользователя из внешнего API", async () => {
    const user: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    mockHttpClient.mockGet("https://api.example.com/users/1", user);
    
    const result = await userService.getUser(1);
    
    expect(result).toEqual(user);
    // Проверка, что пользователь был сохранен в базу данных и кэш
    const dbUsers = await mockDatabase.query<User>("SELECT * FROM users WHERE id = ?", [1]);
    expect(dbUsers[0]).toEqual(user);
    
    const cachedUser = await mockCache.get<User>("user:1");
    expect(cachedUser).toEqual(user);
  });
  
  test("должен обновлять пользователя", async () => {
    const user: User = {
      id: 1,
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    const updatedUser: User = {
      id: 1,
      name: "Иван Петров",
      email: "ivan.petrov@example.com"
    };
    
    // Предварительное заполнение данных
    await mockDatabase.execute(
      "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
      [user.id, user.name, user.email]
    );
    
    mockHttpClient.mockPost("https://api.example.com/notifications", { success: true });
    
    const result = await userService.updateUser(updatedUser);
    
    expect(result).toBe(true);
    
    // Проверка обновления в базе данных
    const dbUsers = await mockDatabase.query<User>("SELECT * FROM users WHERE id = ?", [1]);
    expect(dbUsers[0]).toEqual(updatedUser);
    
    // Проверка обновления в кэше
    const cachedUser = await mockCache.get<User>("user:1");
    expect(cachedUser).toEqual(updatedUser);
  });
  
  test("должен возвращать null для несуществующего пользователя", async () => {
    mockHttpClient.mockGet("https://api.example.com/users/999", null);
    
    const result = await userService.getUser(999);
    
    expect(result).toBeNull();
  });
});
```

## Тестирование обработчиков событий

Обработчики событий в функциональном стиле представляют собой чистые функции, что упрощает их тестирование.

```typescript
// Модели событий
interface Event {
  type: string;
  payload: any;
  timestamp: number;
  correlationId: string;
}

interface UserCreatedEvent extends Event {
  type: "USER_CREATED";
  payload: {
    userId: number;
    name: string;
    email: string;
  };
}

interface OrderPlacedEvent extends Event {
  type: "ORDER_PLACED";
  payload: {
    orderId: number;
    userId: number;
    items: OrderItem[];
    total: number;
  };
}

interface EmailSentEvent extends Event {
  type: "EMAIL_SENT";
  payload: {
    emailId: string;
    recipient: string;
    subject: string;
  };
}

// Обработчики событий
class EventHandler {
  private handlers: Record<string, ((event: Event) => Promise<void>)[]> = {};
  
  register(type: string, handler: (event: Event) => Promise<void>) {
    if (!this.handlers[type]) {
      this.handlers[type] = [];
    }
    this.handlers[type].push(handler);
  }
  
  async dispatch(event: Event): Promise<void> {
    const handlers = this.handlers[event.type] || [];
    await Promise.all(handlers.map(handler => handler(event)));
  }
}

// Сервисы для обработки событий
class EmailService {
  async sendWelcomeEmail(user: { name: string; email: string }): Promise<string> {
    // Симуляция отправки email
    return `email-${Date.now()}`;
  }
  
  async sendOrderConfirmation(order: { orderId: number; email: string }): Promise<string> {
    // Симуляция отправки email
    return `email-${Date.now()}`;
  }
}

class NotificationService {
  constructor(private emailService: EmailService, private eventHandler: EventHandler) {
    this.setupEventHandlers();
  }
  
  private setupEventHandlers() {
    this.eventHandler.register("USER_CREATED", this.handleUserCreated.bind(this));
    this.eventHandler.register("ORDER_PLACED", this.handleOrderPlaced.bind(this));
  }
  
  private async handleUserCreated(event: UserCreatedEvent): Promise<void> {
    const emailId = await this.emailService.sendWelcomeEmail({
      name: event.payload.name,
      email: event.payload.email
    });
    
    // Генерация события отправки email
    const emailSentEvent: EmailSentEvent = {
      type: "EMAIL_SENT",
      payload: {
        emailId,
        recipient: event.payload.email,
        subject: "Добро пожаловать!"
      },
      timestamp: Date.now(),
      correlationId: event.correlationId
    };
    
    await this.eventHandler.dispatch(emailSentEvent);
  }
  
  private async handleOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    const emailId = await this.emailService.sendOrderConfirmation({
      orderId: event.payload.orderId,
      email: "user@example.com" // В реальной реализации получаем email пользователя
    });
    
    // Генерация события отправки email
    const emailSentEvent: EmailSentEvent = {
      type: "EMAIL_SENT",
      payload: {
        emailId,
        recipient: "user@example.com",
        subject: `Подтверждение заказа #${event.payload.orderId}`
      },
      timestamp: Date.now(),
      correlationId: event.correlationId
    };
    
    await this.eventHandler.dispatch(emailSentEvent);
  }
}

// Интеграционные тесты обработчиков событий
describe("Интеграционное тестирование обработчиков событий", () => {
  let eventHandler: EventHandler;
  let emailService: EmailService;
  let notificationService: NotificationService;
  
  beforeEach(() => {
    eventHandler = new EventHandler();
    emailService = new EmailService();
    notificationService = new NotificationService(emailService, eventHandler);
  });
  
  test("должен обрабатывать событие создания пользователя", async () => {
    const userCreatedEvent: UserCreatedEvent = {
      type: "USER_CREATED",
      payload: {
        userId: 1,
        name: "Иван Иванов",
        email: "ivan@example.com"
      },
      timestamp: Date.now(),
      correlationId: "corr-123"
    };
    
    const sendWelcomeEmailSpy = jest.spyOn(emailService, "sendWelcomeEmail")
      .mockResolvedValue("email-456");
    
    const dispatchSpy = jest.spyOn(eventHandler, "dispatch");
    
    await eventHandler.dispatch(userCreatedEvent);
    
    expect(sendWelcomeEmailSpy).toHaveBeenCalledWith({
      name: "Иван Иванов",
      email: "ivan@example.com"
    });
    
    expect(dispatchSpy).toHaveBeenCalledWith(
      expect.objectContaining({
        type: "EMAIL_SENT",
        payload: expect.objectContaining({
          recipient: "ivan@example.com",
          subject: "Добро пожаловать!"
        }),
        correlationId: "corr-123"
      })
    );
  });
  
  test("должен обрабатывать событие размещения заказа", async () => {
    const orderPlacedEvent: OrderPlacedEvent = {
      type: "ORDER_PLACED",
      payload: {
        orderId: 1001,
        userId: 1,
        items: [{ productId: 1, quantity: 2 }],
        total: 10000
      },
      timestamp: Date.now(),
      correlationId: "corr-789"
    };
    
    const sendOrderConfirmationSpy = jest.spyOn(emailService, "sendOrderConfirmation")
      .mockResolvedValue("email-999");
    
    const dispatchSpy = jest.spyOn(eventHandler, "dispatch");
    
    await eventHandler.dispatch(orderPlacedEvent);
    
    expect(sendOrderConfirmationSpy).toHaveBeenCalledWith({
      orderId: 1001,
      email: "user@example.com"
    });
    
    expect(dispatchSpy).toHaveBeenCalledWith(
      expect.objectContaining({
        type: "EMAIL_SENT",
        payload: expect.objectContaining({
          recipient: "user@example.com",
          subject: "Подтверждение заказа #1001"
        }),
        correlationId: "corr-789"
      })
    );
  });
  
  test("должен обрабатывать несколько обработчиков для одного события", async () => {
    const additionalHandler = jest.fn();
    eventHandler.register("USER_CREATED", additionalHandler);
    
    const userCreatedEvent: UserCreatedEvent = {
      type: "USER_CREATED",
      payload: {
        userId: 1,
        name: "Иван Иванов",
        email: "ivan@example.com"
      },
      timestamp: Date.now(),
      correlationId: "corr-111"
    };
    
    await eventHandler.dispatch(userCreatedEvent);
    
    expect(additionalHandler).toHaveBeenCalledWith(userCreatedEvent);
  });
});
```

## Тестирование систем с состоянием

Даже в функциональном программировании иногда требуется работа с состоянием, но это делается через чистые функции.

```typescript
// State монада для работы с состоянием
class StateMonad<S, A> {
  constructor(private run: (state: S) => [A, S]) {}
  
  static of<S, A>(value: A): StateMonad<S, A> {
    return new StateMonad(state => [value, state]);
  }
  
  static get<S>(): StateMonad<S, S> {
    return new StateMonad(state => [state, state]);
  }
  
  static put<S>(state: S): StateMonad<S, void> {
    return new StateMonad(() => [undefined, state]);
  }
  
  static modify<S>(fn: (state: S) => S): StateMonad<S, void> {
    return new StateMonad(state => [undefined, fn(state)]);
  }
  
  map<B>(fn: (a: A) => B): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return [fn(value), newState];
    });
  }
  
  flatMap<B>(fn: (a: A) => StateMonad<S, B>): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return fn(value).run(newState);
    });
  }
  
  evaluate(initialState: S): A {
    const [result] = this.run(initialState);
    return result;
  }
  
  execute(initialState: S): S {
    const [, newState] = this.run(initialState);
    return newState;
  }
}

// Модели состояния
interface ShoppingCart {
  items: CartItem[];
  total: number;
}

interface CartItem {
  productId: number;
  quantity: number;
  price: number;
}

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

// Операции со state
const getCart = (): StateMonad<ShoppingCart, ShoppingCart> => {
  return StateMonad.get<ShoppingCart>();
};

const setCart = (cart: ShoppingCart): StateMonad<ShoppingCart, void> => {
  return StateMonad.put(cart);
};

const addItem = (item: CartItem): StateMonad<ShoppingCart, boolean> => {
  return StateMonad.get<ShoppingCart>().flatMap(cart => {
    const existingItemIndex = cart.items.findIndex(i => i.productId === item.productId);
    
    if (existingItemIndex >= 0) {
      // Обновление существующего элемента
      const updatedItems = [...cart.items];
      updatedItems[existingItemIndex] = {
        ...updatedItems[existingItemIndex],
        quantity: updatedItems[existingItemIndex].quantity + item.quantity
      };
      
      const newTotal = updatedItems.reduce((sum, i) => sum + i.price * i.quantity, 0);
      const newCart = { items: updatedItems, total: newTotal };
      
      return StateMonad.put(newCart).map(() => true);
    } else {
      // Добавление нового элемента
      const newItems = [...cart.items, item];
      const newTotal = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);
      const newCart = { items: newItems, total: newTotal };
      
      return StateMonad.put(newCart).map(() => true);
    }
  });
};

const removeItem = (productId: number): StateMonad<ShoppingCart, void> => {
  return StateMonad.get<ShoppingCart>().flatMap(cart => {
    const newItems = cart.items.filter(item => item.productId !== productId);
    const newTotal = newItems.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const newCart = { items: newItems, total: newTotal };
    
    return StateMonad.put(newCart);
  });
};

const clearCart = (): StateMonad<ShoppingCart, void> => {
  return StateMonad.put({ items: [], total: 0 });
};

const checkout = (userId: number): StateMonad<ShoppingCart, CheckoutResult> => {
  return StateMonad.get<ShoppingCart>().flatMap(cart => {
    if (cart.items.length === 0) {
      return StateMonad.of({ success: false, error: "Корзина пуста" });
    }
    
    // Проверка наличия товаров (в реальной реализации обращение к ProductService)
    const allItemsAvailable = cart.items.every(item => item.quantity > 0);
    
    if (!allItemsAvailable) {
      return StateMonad.of({ success: false, error: "Некоторые товары недоступны" });
    }
    
    // Создание заказа (в реальной реализации обращение к OrderService)
    const orderId = Date.now();
    
    // Очистка корзины
    return clearCart().map(() => ({
      success: true,
      orderId,
      total: cart.total,
      items: cart.items
    }));
  });
};

interface CheckoutResult {
  success: boolean;
  orderId?: number;
  total?: number;
  items?: CartItem[];
  error?: string;
}

// Интеграционные тесты систем с состоянием
describe("Интеграционное тестирование систем с состоянием", () => {
  const initialState: ShoppingCart = {
    items: [],
    total: 0
  };
  
  test("должен корректно добавлять товары в корзину", () => {
    const item1: CartItem = { productId: 1, quantity: 2, price: 100 };
    const item2: CartItem = { productId: 2, quantity: 1, price: 200 };
    
    const program = addItem(item1)
      .flatMap(() => addItem(item2));
    
    const result = program.evaluate(initialState);
    const finalState = program.execute(initialState);
    
    expect(result).toBe(true);
    expect(finalState.items).toHaveLength(2);
    expect(finalState.total).toBe(400); // 2 * 100 + 1 * 200
  });
  
  test("должен корректно обновлять количество существующих товаров", () => {
    const initialStateWithItems: ShoppingCart = {
      items: [{ productId: 1, quantity: 1, price: 100 }],
      total: 100
    };
    
    const item: CartItem = { productId: 1, quantity: 2, price: 100 };
    
    const program = addItem(item);
    
    const result = program.evaluate(initialStateWithItems);
    const finalState = program.execute(initialStateWithItems);
    
    expect(result).toBe(true);
    expect(finalState.items).toHaveLength(1);
    expect(finalState.items[0].quantity).toBe(3);
    expect(finalState.total).toBe(300); // 3 * 100
  });
  
  test("должен корректно удалять товары из корзины", () => {
    const initialStateWithItems: ShoppingCart = {
      items: [
        { productId: 1, quantity: 2, price: 100 },
        { productId: 2, quantity: 1, price: 200 }
      ],
      total: 400
    };
    
    const program = removeItem(1);
    
    const finalState = program.execute(initialStateWithItems);
    
    expect(finalState.items).toHaveLength(1);
    expect(finalState.items[0].productId).toBe(2);
    expect(finalState.total).toBe(200);
  });
  
  test("должен корректно очищать корзину", () => {
    const initialStateWithItems: ShoppingCart = {
      items: [
        { productId: 1, quantity: 2, price: 100 },
        { productId: 2, quantity: 1, price: 200 }
      ],
      total: 400
    };
    
    const program = clearCart();
    
    const finalState = program.execute(initialStateWithItems);
    
    expect(finalState.items).toHaveLength(0);
    expect(finalState.total).toBe(0);
  });
  
  test("должен корректно обрабатывать оформление заказа", () => {
    const initialStateWithItems: ShoppingCart = {
      items: [
        { productId: 1, quantity: 2, price: 100 },
        { productId: 2, quantity: 1, price: 200 }
      ],
      total: 400
    };
    
    const program = checkout(1);
    
    const result = program.evaluate(initialStateWithItems);
    const finalState = program.execute(initialStateWithItems);
    
    expect(result.success).toBe(true);
    expect(result.orderId).toBeDefined();
    expect(result.total).toBe(400);
    expect(result.items).toEqual(initialStateWithItems.items);
    
    // Проверка, что корзина очищена
    expect(finalState.items).toHaveLength(0);
    expect(finalState.total).toBe(0);
  });
  
  test("должен возвращать ошибку при оформлении пустого заказа", () => {
    const program = checkout(1);
    
    const result = program.evaluate(initialState);
    
    expect(result.success).toBe(false);
    expect(result.error).toBe("Корзина пуста");
  });
  
  test("должен корректно обрабатывать сложные сценарии", () => {
    const item1: CartItem = { productId: 1, quantity: 1, price: 100 };
    const item2: CartItem = { productId: 2, quantity: 2, price: 50 };
    
    const complexProgram = addItem(item1)
      .flatMap(() => addItem(item2))
      .flatMap(() => removeItem(1))
      .flatMap(() => checkout(1));
    
    const result = complexProgram.evaluate(initialState);
    const finalState = complexProgram.execute(initialState);
    
    expect(result.success).toBe(true);
    expect(result.total).toBe(100); // 2 * 50
    expect(result.items).toEqual([{ productId: 2, quantity: 2, price: 50 }]);
    
    // Проверка финального состояния
    expect(finalState.items).toHaveLength(0);
    expect(finalState.total).toBe(0);
  });
});
```

## Тестирование асинхронных операций

Асинхронные операции в функциональном стиле тестируются с использованием монад и чистых функций.

```typescript
// Future монада для асинхронных операций
class FutureMonad<A> {
  constructor(private computation: () => Promise<A>) {}
  
  static of<A>(value: A): FutureMonad<A> {
    return new FutureMonad(() => Promise.resolve(value));
  }
  
  static fromPromise<A>(promise: Promise<A>): FutureMonad<A> {
    return new FutureMonad(() => promise);
  }
  
  map<B>(fn: (a: A) => B): FutureMonad<B> {
    return new FutureMonad(() => this.computation().then(fn));
  }
  
  flatMap<B>(fn: (a: A) => FutureMonad<B>): FutureMonad<B> {
    return new FutureMonad(() => 
      this.computation().then(a => fn(a).computation())
    );
  }
  
  runFuture(): Promise<A> {
    return this.computation();
  }
  
  catchError<B>(fn: (error: any) => B): FutureMonad<A | B> {
    return new FutureMonad(() => 
      this.computation().catch(fn)
    );
  }
}

// Either монада для обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

// Модели для асинхронных операций
interface ApiResult<T> {
  status: number;
  data?: T;
  error?: string;
}

interface User {
  id: number;
  name: string;
  email: string;
}

interface Product {
  id: number;
  name: string;
  price: number;
}

interface Order {
  id: number;
  userId: number;
  items: OrderItem[];
  total: number;
}

interface OrderItem {
  productId: number;
  quantity: number;
  price: number;
}

// Сервисы с асинхронными операциями
class UserServiceAsync {
  async getUser(id: number): Promise<ApiResult<User>> {
    // Симуляция API вызова
    return new Promise(resolve => {
      setTimeout(() => {
        if (id <= 0) {
          resolve({ status: 404, error: "Пользователь не найден" });
        } else {
          resolve({
            status: 200,
            data: {
              id,
              name: `Пользователь ${id}`,
              email: `user${id}@example.com`
            }
          });
        }
      }, 100);
    });
  }
  
  async createUser(userData: Omit<User, "id">): Promise<ApiResult<User>> {
    // Симуляция API вызова
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          status: 201,
          data: {
            id: Date.now(),
            name: userData.name,
            email: userData.email
          }
        });
      }, 100);
    });
  }
}

class ProductServiceAsync {
  async getProduct(id: number): Promise<ApiResult<Product>> {
    // Симуляция API вызова
    return new Promise(resolve => {
      setTimeout(() => {
        if (id <= 0) {
          resolve({ status: 404, error: "Продукт не найден" });
        } else {
          resolve({
            status: 200,
            data: {
              id,
              name: `Продукт ${id}`,
              price: Math.floor(Math.random() * 1000) + 100
            }
          });
        }
      }, 100);
    });
  }
  
  async getProducts(): Promise<ApiResult<Product[]>> {
    // Симуляция API вызова
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          status: 200,
          data: [
            { id: 1, name: "Ноутбук", price: 50000 },
            { id: 2, name: "Смартфон", price: 30000 },
            { id: 3, name: "Книга", price: 500 }
          ]
        });
      }, 100);
    });
  }
}

class OrderServiceAsync {
  constructor(
    private userService: UserServiceAsync,
    private productService: ProductServiceAsync
  ) {}
  
  async createOrder(userId: number, items: OrderItem[]): Promise<ApiResult<Order>> {
    // Проверка пользователя
    const userResult = await this.userService.getUser(userId);
    if (userResult.status !== 200) {
      return { status: 400, error: "Неверный пользователь" };
    }
    
    // Проверка продуктов
    let total = 0;
    for (const item of items) {
      const productResult = await this.productService.getProduct(item.productId);
      if (productResult.status !== 200) {
        return { status: 400, error: `Продукт ${item.productId} не найден` };
      }
      total += productResult.data!.price * item.quantity;
    }
    
    // Создание заказа
    const order: Order = {
      id: Date.now(),
      userId,
      items,
      total
    };
    
    return { status: 201, data: order };
  }
}

// Функциональные обертки для асинхронных операций
const getUserFuture = (userService: UserServiceAsync, id: number): FutureMonad<ApiResult<User>> => {
  return FutureMonad.fromPromise(userService.getUser(id));
};

const getProductFuture = (productService: ProductServiceAsync, id: number): FutureMonad<ApiResult<Product>> => {
  return FutureMonad.fromPromise(productService.getProduct(id));
};

const createOrderFuture = (orderService: OrderServiceAsync, userId: number, items: OrderItem[]): FutureMonad<ApiResult<Order>> => {
  return FutureMonad.fromPromise(orderService.createOrder(userId, items));
};

// Композиция асинхронных операций
const processUserOrder = (
  userService: UserServiceAsync,
  productService: ProductServiceAsync,
  orderService: OrderServiceAsync,
  userId: number,
  productIds: number[]
): FutureMonad<ApiResult<{ user: User; order: Order }>> => {
  return getUserFuture(userService, userId)
    .flatMap(userResult => {
      if (userResult.status !== 200) {
        return FutureMonad.of(userResult as any);
      }
      
      const items: OrderItem[] = productIds.map(id => ({
        productId: id,
        quantity: 1,
        price: 0 // Будет заполнен позже
      }));
      
      return createOrderFuture(orderService, userId, items)
        .map(orderResult => {
          if (orderResult.status === 201) {
            return right({
              user: userResult.data!,
              order: orderResult.data!
            });
          }
          return left(orderResult);
        });
    })
    .catchError(error => left({ status: 500, error: `Ошибка обработки: ${error}` }));
};

// Интеграционные тесты асинхронных операций
describe("Интеграционное тестирование асинхронных операций", () => {
  let userService: UserServiceAsync;
  let productService: ProductServiceAsync;
  let orderService: OrderServiceAsync;
  
  beforeEach(() => {
    userService = new UserServiceAsync();
    productService = new ProductServiceAsync();
    orderService = new OrderServiceAsync(userService, productService);
  });
  
  test("должен корректно обрабатывать создание заказа", async () => {
    const result = await createOrderFuture(orderService, 1, [
      { productId: 1, quantity: 1, price: 50000 },
      { productId: 2, quantity: 2, price: 30000 }
    ]).runFuture();
    
    expect(result.status).toBe(201);
    expect(result.data).toBeDefined();
    expect(result.data?.userId).toBe(1);
    expect(result.data?.total).toBe(110000); // 50000 + 2 * 30000
  });
  
  test("должен возвращать ошибку для несуществующего пользователя", async () => {
    const result = await createOrderFuture(orderService, -1, [
      { productId: 1, quantity: 1, price: 50000 }
    ]).runFuture();
    
    expect(result.status).toBe(400);
    expect(result.error).toBe("Неверный пользователь");
  });
  
  test("должен возвращать ошибку для несуществующего продукта", async () => {
    const result = await createOrderFuture(orderService, 1, [
      { productId: -1, quantity: 1, price: 50000 }
    ]).runFuture();
    
    expect(result.status).toBe(400);
    expect(result.error).toBe("Продукт -1 не найден");
  });
  
  test("должен обрабатывать сложные асинхронные сценарии", async () => {
    const result = await processUserOrder(
      userService,
      productService,
      orderService,
      1,
      [1, 2]
    ).runFuture();
    
    if (result._tag === "Right") {
      expect(result.right.user.id).toBe(1);
      expect(result.right.user.name).toBe("Пользователь 1");
      expect(result.right.order.userId).toBe(1);
      expect(result.right.order.total).toBeGreaterThan(0);
    } else {
      fail("Ожидался успешный результат");
    }
  });
  
  test("должен обрабатывать ошибки в сложных сценариях", async () => {
    const result = await processUserOrder(
      userService,
      productService,
      orderService,
      -1,
      [1, 2]
    ).runFuture();
    
    if (result._tag === "Left") {
      expect(result.left.status).toBe(400);
      expect(result.left.error).toBe("Неверный пользователь");
    } else {
      fail("Ожидалась ошибка");
    }
  });
  
  test("должен корректно обрабатывать параллельные операции", async () => {
    const userFuture = getUserFuture(userService, 1);
    const productsFuture = FutureMonad.fromPromise(productService.getProducts());
    
    const [userResult, productsResult] = await Promise.all([
      userFuture.runFuture(),
      productsFuture.runFuture()
    ]);
    
    expect(userResult.status).toBe(200);
    expect(userResult.data?.id).toBe(1);
    
    expect(productsResult.status).toBe(200);
    expect(productsResult.data).toHaveLength(3);
  });
});
```

## Лучшие практики

### 1. Изоляция внешних зависимостей

```typescript
// Хорошо: изоляция внешних зависимостей
interface Dependencies {
  httpClient: HttpClient;
  database: Database;
  cache: Cache;
  logger: Logger;
}

const createUserWithDependencies = async (
  userData: CreateUserRequest,
  deps: Dependencies
): Promise<User | null> => {
  deps.logger.info(`Создание пользователя: ${userData.email}`);
  
  try {
    // Проверка существующего пользователя
    const existingUser = await deps.database.query<User>(
      "SELECT * FROM users WHERE email = ?",
      [userData.email]
    );
    
    if (existingUser.length > 0) {
      deps.logger.info(`Пользователь уже существует: ${userData.email}`);
      return existingUser[0];
    }
    
    // Создание нового пользователя
    const user: User = {
      id: Date.now(),
      name: userData.name,
      email: userData.email,
      createdAt: new Date()
    };
    
    await deps.database.execute(
      "INSERT INTO users (id, name, email, created_at) VALUES (?, ?, ?, ?)",
      [user.id, user.name, user.email, user.createdAt]
    );
    
    deps.logger.info(`Пользователь создан: ${user.email}`);
    return user;
  } catch (error) {
    deps.logger.error(`Ошибка создания пользователя: ${error}`);
    return null;
  }
};

// Тестирование с моками
describe("Тестирование с изолированными зависимостями", () => {
  test("должен создавать нового пользователя", async () => {
    const mockDeps: Dependencies = {
      httpClient: mockHttpClient,
      database: mockDatabase,
      cache: mockCache,
      logger: mockLogger
    };
    
    const userData: CreateUserRequest = {
      name: "Иван Иванов",
      email: "ivan@example.com"
    };
    
    mockDatabase.mockQuery("SELECT * FROM users WHERE email = ?", [], []);
    mockDatabase.mockExecute("INSERT INTO users", undefined);
    
    const result = await createUserWithDependencies(userData, mockDeps);
    
    expect(result).toBeDefined();
    expect(result?.name).toBe("Иван Иванов");
    expect(result?.email).toBe("ivan@example.com");
    expect(mockLogger.info).toHaveBeenCalledWith("Создание пользователя: ivan@example.com");
    expect(mockLogger.info).toHaveBeenCalledWith("Пользователь создан: ivan@example.com");
  });
});
```

### 2. Тестирование потоков данных

```typescript
// Хорошо: тестирование потоков данных
const processDataPipeline = pipe(
  validateInput,
  transformData,
  saveToDatabase,
  sendNotification
);

describe("Тестирование потоков данных", () => {
  test("должен корректно обрабатывать валидные данные", async () => {
    const inputData = createValidInput();
    
    const result = await processDataPipeline(inputData);
    
    expect(result.success).toBe(true);
    expect(result.data).toBeDefined();
    expect(mockDatabase.save).toHaveBeenCalledWith(expect.any(Object));
    expect(mockNotificationService.send).toHaveBeenCalledWith(expect.any(Object));
  });
  
  test("должен корректно обрабатывать невалидные данные", async () => {
    const inputData = createInvalidInput();
    
    const result = await processDataPipeline(inputData);
    
    expect(result.success).toBe(false);
    expect(result.error).toBeDefined();
    expect(mockDatabase.save).not.toHaveBeenCalled();
    expect(mockNotificationService.send).not.toHaveBeenCalled();
  });
});
```

### 3. Тестирование сценариев ошибок

```typescript
// Хорошо: тщательное тестирование сценариев ошибок
describe("Тестирование сценариев ошибок", () => {
  test("должен корректно обрабатывать ошибки базы данных", async () => {
    mockDatabase.mockExecute("INSERT INTO users", new Error("Ошибка БД"));
    
    const result = await createUser(userData, mockDeps);
    
    expect(result).toBeNull();
    expect(mockLogger.error).toHaveBeenCalledWith(
      expect.stringContaining("Ошибка создания пользователя")
    );
  });
  
  test("должен корректно обрабатывать ошибки сети", async () => {
    mockHttpClient.mockPost("/api/users", new Error("Ошибка сети"));
    
    const result = await createUser(userData, mockDeps);
    
    expect(result).toBeNull();
    expect(mockLogger.error).toHaveBeenCalledWith(
      expect.stringContaining("Ошибка создания пользователя")
    );
  });
  
  test("должен корректно обрабатывать таймауты", async () => {
    mockHttpClient.mockPost("/api/users", new Promise(() => {})); // Никогда не разрешается
    
    const result = await Promise.race([
      createUser(userData, mockDeps),
      new Promise(resolve => setTimeout(() => resolve(null), 5000))
    ]);
    
    expect(result).toBeNull();
  });
});
```

### 4. Тестирование производительности

```typescript
// Хорошо: тестирование производительности
describe("Тестирование производительности", () => {
  test("должен обрабатывать большое количество запросов", async () => {
    const startTime = Date.now();
    const promises = [];
    
    for (let i = 0; i < 1000; i++) {
      promises.push(createUser(createUserRequest(i), mockDeps));
    }
    
    const results = await Promise.all(promises);
    const endTime = Date.now();
    
    expect(results.filter(r => r !== null)).toHaveLength(1000);
    expect(endTime - startTime).toBeLessThan(10000); // Менее 10 секунд
  });
  
  test("должен эффективно использовать кэш", async () => {
    const cacheHitsBefore = mockCache.getHits();
    
    // Первый запрос - обращение к базе данных
    await getUser(1, mockDeps);
    
    // Второй запрос - обращение к кэшу
    await getUser(1, mockDeps);
    
    const cacheHitsAfter = mockCache.getHits();
    expect(cacheHitsAfter - cacheHitsBefore).toBe(1);
  });
});
```

## Связи с другими концепциями

- [[ts/functional-programming/testing/Тестирование_функционального_кода|Тестирование функционального кода]] - Основная страница раздела
- [[ts/functional-programming/testing/unit-testing|Модульное тестирование]] - Тестирование отдельных функций
- [[ts/functional-programming/pure-functions|Чистые функции]] - Основа для тестируемости
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Структуры для обработки эффектов
- [[ts/functional-programming/composition|Композиция функций]] - Тестирование композиции
- [[ts/testing/e2e-testing|End-to-end тестирование]] - Тестирование полных сценариев
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация функционального кода

## Следующие шаги

- [[ts/testing/e2e-testing|End-to-end тестирование]] - Тестирование полных сценариев
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация функционального кода
- [[Обработка ошибок в TypeScript|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[ts/functional-programming/Функциональное_программирование|Функциональное программирование]] - Обзор всех концепций функционального программирования