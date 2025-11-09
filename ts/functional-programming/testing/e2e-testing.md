# End-to-End тестирование функционального кода

End-to-End (E2E) тестирование в функциональном программировании фокусируется на проверке полных сценариев использования приложения от начала до конца. В функциональном подходе E2E тесты становятся более предсказуемыми и воспроизводимыми благодаря отсутствию изменяемого состояния и побочных эффектов.

## Содержание

- [End-to-End тестирование функционального кода](#end-to-end-тестирование-функционального-кода)
  - [Содержание](#содержание)
  - [Основы E2E тестирования](#основы-e2e-тестирования)
  - [Тестирование пользовательских сценариев](#тестирование-пользовательских-сценариев)
  - [Тестирование API и интеграций](#тестирование-api-и-интеграций)
  - [Тестирование потоков данных](#тестирование-потоков-данных)
  - [Тестирование обработчиков событий](#тестирование-обработчиков-событий)
  - [Тестирование систем с состоянием](#тестирование-систем-с-состоянием)

## Основы E2E тестирования

E2E тестирование в функциональном программировании имеет свои особенности:

1. **Предсказуемость** - Отсутствие изменяемого состояния делает тесты воспроизводимыми
2. **Изоляция** - Легче создавать изолированные тестовые среды
3. **Композиция** - E2E сценарии строятся из тестируемых компонентов
4. **Наблюдаемость** - Чистые функции проще отлаживать и мониторить

```typescript
// Пример системы для E2E тестирования
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

interface Order {
  id: number;
  userId: number;
  items: OrderItem[];
  total: number;
  status: "pending" | "confirmed" | "shipped" | "delivered";
  createdAt: Date;
}

interface OrderItem {
  productId: number;
  quantity: number;
  price: number;
}

// Сервисы системы
class UserService {
  private users: User[] = [];
  
  async createUser(userData: Omit<User, "id" | "createdAt">): Promise<User> {
    const user: User = {
      id: Date.now(),
      name: userData.name,
      email: userData.email,
      createdAt: new Date()
    };
    
    this.users.push(user);
    return user;
  }
  
  async getUser(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }
  
  async getUserByEmail(email: string): Promise<User | null> {
    return this.users.find(u => u.email === email) || null;
  }
}

class ProductService {
  private products: Product[] = [
    { id: 1, name: "Ноутбук", price: 50000, inStock: true },
    { id: 2, name: "Смартфон", price: 30000, inStock: true },
    { id: 3, name: "Книга", price: 500, inStock: true }
  ];
  
  async getProduct(id: number): Promise<Product | null> {
    return this.products.find(p => p.id === id) || null;
  }
  
  async getProducts(): Promise<Product[]> {
    return [...this.products];
  }
  
  async updateStock(productId: number, inStock: boolean): Promise<boolean> {
    const product = this.products.find(p => p.id === productId);
    if (product) {
      product.inStock = inStock;
      return true;
    }
    return false;
  }
}

class OrderService {
  private orders: Order[] = [];
  private nextOrderId = 1;
  
  constructor(
    private userService: UserService,
    private productService: ProductService
  ) {}
  
  async createOrder(userId: number, items: Omit<OrderItem, "price">[]): Promise<Order | null> {
    const user = await this.userService.getUser(userId);
    if (!user) {
      return null;
    }
    
    const orderItems: OrderItem[] = [];
    let total = 0;
    
    for (const item of items) {
      const product = await this.productService.getProduct(item.productId);
      if (!product || !product.inStock) {
        return null;
      }
      
      const orderItem: OrderItem = {
        productId: item.productId,
        quantity: item.quantity,
        price: product.price
      };
      
      orderItems.push(orderItem);
      total += product.price * item.quantity;
    }
    
    const order: Order = {
      id: this.nextOrderId++,
      userId,
      items: orderItems,
      total,
      status: "pending",
      createdAt: new Date()
    };
    
    this.orders.push(order);
    return order;
  }
  
  async confirmOrder(orderId: number): Promise<boolean> {
    const order = this.orders.find(o => o.id === orderId);
    if (order && order.status === "pending") {
      order.status = "confirmed";
      return true;
    }
    return false;
  }
  
  async getOrder(id: number): Promise<Order | null> {
    return this.orders.find(o => o.id === id) || null;
  }
  
  async getUserOrders(userId: number): Promise<Order[]> {
    return this.orders.filter(o => o.userId === userId);
  }
}

class NotificationService {
  async sendOrderConfirmation(order: Order, user: User): Promise<boolean> {
    // Симуляция отправки уведомления
    console.log(`Отправка подтверждения заказа #${order.id} пользователю ${user.email}`);
    return true;
  }
  
  async sendOrderStatusUpdate(order: Order, user: User, status: string): Promise<boolean> {
    // Симуляция отправки уведомления
    console.log(`Отправка обновления статуса заказа #${order.id} (${status}) пользователю ${user.email}`);
    return true;
  }
}

// Основная система
class ECommerceSystem {
  public userService: UserService;
  public productService: ProductService;
  public orderService: OrderService;
  public notificationService: NotificationService;
  
  constructor() {
    this.userService = new UserService();
    this.productService = new ProductService();
    this.notificationService = new NotificationService();
    this.orderService = new OrderService(this.userService, this.productService);
  }
  
  async registerUser(name: string, email: string): Promise<User> {
    return this.userService.createUser({ name, email });
  }
  
  async browseProducts(): Promise<Product[]> {
    return this.productService.getProducts();
  }
  
  async purchaseProducts(userId: number, items: { productId: number; quantity: number }[]): Promise<Order | null> {
    const order = await this.orderService.createOrder(userId, items);
    if (order) {
      const user = await this.userService.getUser(userId);
      if (user) {
        await this.notificationService.sendOrderConfirmation(order, user);
      }
    }
    return order;
  }
  
  async updateOrderStatus(orderId: number, status: "confirmed" | "shipped" | "delivered"): Promise<boolean> {
    let success = false;
    
    switch (status) {
      case "confirmed":
        success = await this.orderService.confirmOrder(orderId);
        break;
      // Другие статусы могут быть реализованы аналогично
    }
    
    if (success) {
      const order = await this.orderService.getOrder(orderId);
      if (order) {
        const user = await this.userService.getUser(order.userId);
        if (user) {
          await this.notificationService.sendOrderStatusUpdate(order, user, status);
        }
      }
    }
    
    return success;
  }
}

// E2E тесты
describe("E2E тестирование E-Commerce системы", () => {
  let system: ECommerceSystem;
  
  beforeEach(() => {
    system = new ECommerceSystem();
  });
  
  test("Полный сценарий: регистрация пользователя, просмотр продуктов, создание заказа", async () => {
    // 1. Регистрация нового пользователя
    const user = await system.registerUser("Иван Иванов", "ivan@example.com");
    expect(user).toBeDefined();
    expect(user.name).toBe("Иван Иванов");
    expect(user.email).toBe("ivan@example.com");
    
    // 2. Просмотр доступных продуктов
    const products = await system.browseProducts();
    expect(products).toHaveLength(3);
    expect(products.some(p => p.name === "Ноутбук")).toBe(true);
    expect(products.some(p => p.name === "Смартфон")).toBe(true);
    expect(products.some(p => p.name === "Книга")).toBe(true);
    
    // 3. Создание заказа
    const order = await system.purchaseProducts(user.id, [
      { productId: 1, quantity: 1 }, // Ноутбук
      { productId: 3, quantity: 2 }  // 2 книги
    ]);
    
    expect(order).toBeDefined();
    expect(order?.userId).toBe(user.id);
    expect(order?.items).toHaveLength(2);
    expect(order?.total).toBe(51000); // 50000 + 2 * 500
    expect(order?.status).toBe("pending");
    
    // 4. Проверка уведомлений (через моки или логи)
    // В реальной реализации здесь были бы проверки отправки уведомлений
    
    // 5. Обновление статуса заказа
    const statusUpdated = await system.updateOrderStatus(order!.id, "confirmed");
    expect(statusUpdated).toBe(true);
    
    const updatedOrder = await system.orderService.getOrder(order!.id);
    expect(updatedOrder?.status).toBe("confirmed");
  });
  
  test("Сценарий с ошибкой: попытка заказа недоступного продукта", async () => {
    // 1. Регистрация пользователя
    const user = await system.registerUser("Мария Петрова", "maria@example.com");
    
    // 2. Пометка продукта как недоступного
    await system.productService.updateStock(1, false); // Ноутбук недоступен
    
    // 3. Попытка заказа недоступного продукта
    const order = await system.purchaseProducts(user.id, [
      { productId: 1, quantity: 1 } // Недоступный ноутбук
    ]);
    
    expect(order).toBeNull();
  });
  
  test("Сценарий с несуществующим пользователем", async () => {
    // Попытка создания заказа для несуществующего пользователя
    const order = await system.purchaseProducts(999, [
      { productId: 1, quantity: 1 }
    ]);
    
    expect(order).toBeNull();
  });
  
  test("Сценарий с несуществующим продуктом", async () => {
    // 1. Регистрация пользователя
    const user = await system.registerUser("Алексей Сидоров", "alex@example.com");
    
    // 2. Попытка заказа несуществующего продукта
    const order = await system.purchaseProducts(user.id, [
      { productId: 999, quantity: 1 } // Несуществующий продукт
    ]);
    
    expect(order).toBeNull();
  });
});
```

## Тестирование пользовательских сценариев

Пользовательские сценарии в E2E тестировании охватывают полные потоки взаимодействия пользователя с системой.

```typescript
// Модели для пользовательских сценариев
interface CartItem {
  productId: number;
  quantity: number;
}

interface ShoppingCart {
  items: CartItem[];
  total: number;
}

interface UserSession {
  userId: number;
  cart: ShoppingCart;
  isLoggedIn: boolean;
}

// Сервис корзины покупок
class ShoppingCartService {
  private carts: Record<number, ShoppingCart> = {};
  
  async getCart(userId: number): Promise<ShoppingCart> {
    return this.carts[userId] || { items: [], total: 0 };
  }
  
  async addItem(userId: number, productId: number, quantity: number): Promise<ShoppingCart> {
    const cart = await this.getCart(userId);
    const existingItemIndex = cart.items.findIndex(item => item.productId === productId);
    
    if (existingItemIndex >= 0) {
      cart.items[existingItemIndex].quantity += quantity;
    } else {
      cart.items.push({ productId, quantity });
    }
    
    // Пересчет общей суммы (в реальной реализации с ProductService)
    cart.total = cart.items.reduce((sum, item) => sum + item.productId * 100 * item.quantity, 0);
    
    this.carts[userId] = cart;
    return cart;
  }
  
  async removeItem(userId: number, productId: number): Promise<ShoppingCart> {
    const cart = await this.getCart(userId);
    cart.items = cart.items.filter(item => item.productId !== productId);
    
    // Пересчет общей суммы
    cart.total = cart.items.reduce((sum, item) => sum + item.productId * 100 * item.quantity, 0);
    
    this.carts[userId] = cart;
    return cart;
  }
  
  async clearCart(userId: number): Promise<void> {
    this.carts[userId] = { items: [], total: 0 };
  }
}

// Сервис сессий пользователей
class UserSessionService {
  private sessions: Record<string, UserSession> = {};
  
  async createSession(userId: number): Promise<string> {
    const sessionId = `session-${Date.now()}-${Math.random()}`;
    this.sessions[sessionId] = {
      userId,
      cart: { items: [], total: 0 },
      isLoggedIn: true
    };
    return sessionId;
  }
  
  async getSession(sessionId: string): Promise<UserSession | null> {
    return this.sessions[sessionId] || null;
  }
  
  async updateCart(sessionId: string, cart: ShoppingCart): Promise<boolean> {
    const session = await this.getSession(sessionId);
    if (session) {
      session.cart = cart;
      return true;
    }
    return false;
  }
}

// Веб-приложение
class WebApplication {
  constructor(
    private userService: UserService,
    private productService: ProductService,
    private orderService: OrderService,
    private shoppingCartService: ShoppingCartService,
    private userSessionService: UserSessionService
  ) {}
  
  async registerAndLogin(name: string, email: string): Promise<string> {
    const user = await this.userService.createUser({ name, email });
    return this.userSessionService.createSession(user.id);
  }
  
  async browseProducts(sessionId: string): Promise<Product[]> {
    const session = await this.userSessionService.getSession(sessionId);
    if (!session || !session.isLoggedIn) {
      throw new Error("Пользователь не авторизован");
    }
    
    return this.productService.getProducts();
  }
  
  async addToCart(sessionId: string, productId: number, quantity: number): Promise<ShoppingCart> {
    const session = await this.userSessionService.getSession(sessionId);
    if (!session || !session.isLoggedIn) {
      throw new Error("Пользователь не авторизован");
    }
    
    const cart = await this.shoppingCartService.addItem(session.userId, productId, quantity);
    await this.userSessionService.updateCart(sessionId, cart);
    return cart;
  }
  
  async removeFromCart(sessionId: string, productId: number): Promise<ShoppingCart> {
    const session = await this.userSessionService.getSession(sessionId);
    if (!session || !session.isLoggedIn) {
      throw new Error("Пользователь не авторизован");
    }
    
    const cart = await this.shoppingCartService.removeItem(session.userId, productId);
    await this.userSessionService.updateCart(sessionId, cart);
    return cart;
  }
  
  async checkout(sessionId: string): Promise<Order | null> {
    const session = await this.userSessionService.getSession(sessionId);
    if (!session || !session.isLoggedIn) {
      throw new Error("Пользователь не авторизован");
    }
    
    const cart = await this.shoppingCartService.getCart(session.userId);
    if (cart.items.length === 0) {
      throw new Error("Корзина пуста");
    }
    
    const orderItems = cart.items.map(item => ({
      productId: item.productId,
      quantity: item.quantity
    }));
    
    const order = await this.orderService.createOrder(session.userId, orderItems);
    
    if (order) {
      await this.shoppingCartService.clearCart(session.userId);
    }
    
    return order;
  }
}

// E2E тесты пользовательских сценариев
describe("E2E тестирование пользовательских сценариев", () => {
  let app: WebApplication;
  let userService: UserService;
  let productService: ProductService;
  let orderService: OrderService;
  let shoppingCartService: ShoppingCartService;
  let userSessionService: UserSessionService;
  
  beforeEach(() => {
    userService = new UserService();
    productService = new ProductService();
    orderService = new OrderService(userService, productService);
    shoppingCartService = new ShoppingCartService();
    userSessionService = new UserSessionService();
    
    app = new WebApplication(
      userService,
      productService,
      orderService,
      shoppingCartService,
      userSessionService
    );
  });
  
  test("Полный пользовательский сценарий: регистрация, добавление в корзину, оформление заказа", async () => {
    // 1. Регистрация и вход в систему
    const sessionId = await app.registerAndLogin("Иван Иванов", "ivan@example.com");
    expect(sessionId).toMatch(/^session-/);
    
    // 2. Просмотр продуктов
    const products = await app.browseProducts(sessionId);
    expect(products).toHaveLength(3);
    
    // 3. Добавление продуктов в корзину
    let cart = await app.addToCart(sessionId, 1, 1); // Ноутбук
    expect(cart.items).toHaveLength(1);
    expect(cart.items[0].productId).toBe(1);
    expect(cart.items[0].quantity).toBe(1);
    expect(cart.total).toBeGreaterThan(0);
    
    cart = await app.addToCart(sessionId, 3, 2); // 2 книги
    expect(cart.items).toHaveLength(2);
    expect(cart.total).toBeGreaterThan(0);
    
    // 4. Просмотр корзины
    const currentCart = await shoppingCartService.getCart(1); // ID пользователя 1
    expect(currentCart.items).toHaveLength(2);
    
    // 5. Оформление заказа
    const order = await app.checkout(sessionId);
    expect(order).toBeDefined();
    expect(order?.userId).toBe(1);
    expect(order?.items).toHaveLength(2);
    expect(order?.status).toBe("pending");
    
    // 6. Проверка очистки корзины после оформления заказа
    const emptyCart = await shoppingCartService.getCart(1);
    expect(emptyCart.items).toHaveLength(0);
    expect(emptyCart.total).toBe(0);
  });
  
  test("Сценарий: работа с корзиной - добавление, удаление, обновление", async () => {
    // 1. Регистрация и вход
    const sessionId = await app.registerAndLogin("Мария Петрова", "maria@example.com");
    
    // 2. Добавление нескольких продуктов
    await app.addToCart(sessionId, 1, 1); // Ноутбук
    await app.addToCart(sessionId, 2, 2); // 2 смартфона
    await app.addToCart(sessionId, 3, 3); // 3 книги
    
    let cart = await shoppingCartService.getCart(2); // ID пользователя 2
    expect(cart.items).toHaveLength(3);
    
    // 3. Удаление одного продукта
    await app.removeFromCart(sessionId, 2); // Удаление смартфонов
    
    cart = await shoppingCartService.getCart(2);
    expect(cart.items).toHaveLength(2);
    expect(cart.items.some(item => item.productId === 2)).toBe(false);
    
    // 4. Добавление еще одного продукта
    await app.addToCart(sessionId, 1, 1); // Еще один ноутбук
    
    cart = await shoppingCartService.getCart(2);
    expect(cart.items).toHaveLength(2);
    const laptopItem = cart.items.find(item => item.productId === 1);
    expect(laptopItem?.quantity).toBe(2); // 1 + 1 = 2
  });
  
  test("Сценарий с ошибками: неавторизованный доступ", async () => {
    // Попытка доступа к защищенным функциям без авторизации
    await expect(app.browseProducts("invalid-session"))
      .rejects.toThrow("Пользователь не авторизован");
    
    await expect(app.addToCart("invalid-session", 1, 1))
      .rejects.toThrow("Пользователь не авторизован");
    
    await expect(app.checkout("invalid-session"))
      .rejects.toThrow("Пользователь не авторизован");
  });
  
  test("Сценарий с ошибками: оформление пустого заказа", async () => {
    // 1. Регистрация и вход
    const sessionId = await app.registerAndLogin("Алексей Сидоров", "alex@example.com");
    
    // 2. Попытка оформления заказа с пустой корзиной
    await expect(app.checkout(sessionId))
      .rejects.toThrow("Корзина пуста");
  });
  
  test("Сценарий с несколькими сессиями", async () => {
    // 1. Создание нескольких пользователей
    const sessionId1 = await app.registerAndLogin("Пользователь 1", "user1@example.com");
    const sessionId2 = await app.registerAndLogin("Пользователь 2", "user2@example.com");
    
    // 2. Каждый пользователь работает со своей корзиной
    await app.addToCart(sessionId1, 1, 1); // Пользователь 1 добавляет ноутбук
    await app.addToCart(sessionId2, 2, 2); // Пользователь 2 добавляет 2 смартфона
    
    // 3. Проверка изоляции корзин
    const cart1 = await shoppingCartService.getCart(3); // ID пользователя 1
    const cart2 = await shoppingCartService.getCart(4); // ID пользователя 2
    
    expect(cart1.items).toHaveLength(1);
    expect(cart1.items[0].productId).toBe(1);
    
    expect(cart2.items).toHaveLength(1);
    expect(cart2.items[0].productId).toBe(2);
    expect(cart2.items[0].quantity).toBe(2);
  });
});
```

## Тестирование API и интеграций

API и интеграции в функциональных системах тестируются как композиции чистых функций с минимальными побочными эффектами.

```typescript
// Модели для API тестирования
interface ApiRequest {
  method: string;
  url: string;
  headers: Record<string, string>;
  body?: any;
}

interface ApiResponse<T> {
  status: number;
  headers: Record<string, string>;
  body: T;
}

interface ApiError {
  status: number;
  message: string;
  details?: any;
}

// HTTP клиент для тестирования
class HttpClient {
  private baseUrl: string;
  private defaultHeaders: Record<string, string>;
  
  constructor(baseUrl: string, defaultHeaders: Record<string, string> = {}) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = defaultHeaders;
  }
  
  async request<T>(req: ApiRequest): Promise<ApiResponse<T> | ApiError> {
    try {
      // Симуляция HTTP запроса
      const url = `${this.baseUrl}${req.url}`;
      const headers = { ...this.defaultHeaders, ...req.headers };
      
      console.log(`HTTP ${req.method} ${url}`, { headers, body: req.body });
      
      // Симуляция ответа в зависимости от URL и метода
      return await this.simulateResponse<T>(req);
    } catch (error) {
      return {
        status: 500,
        message: "Внутренняя ошибка сервера",
        details: error instanceof Error ? error.message : String(error)
      };
    }
  }
  
  private async simulateResponse<T>(req: ApiRequest): Promise<ApiResponse<T> | ApiError> {
    const url = req.url;
    const method = req.method;
    
    // Симуляция различных ответов
    if (url === "/users" && method === "POST") {
      if (!req.body || !req.body.name || !req.body.email) {
        return {
          status: 400,
          message: "Неверные данные пользователя"
        };
      }
      
      return {
        status: 201,
        headers: { "content-type": "application/json" },
        body: {
          id: Date.now(),
          name: req.body.name,
          email: req.body.email,
          createdAt: new Date().toISOString()
        } as any
      };
    }
    
    if (url === "/users/1" && method === "GET") {
      return {
        status: 200,
        headers: { "content-type": "application/json" },
        body: {
          id: 1,
          name: "Иван Иванов",
          email: "ivan@example.com",
          createdAt: "2023-01-01T00:00:00Z"
        } as any
      };
    }
    
    if (url === "/products" && method === "GET") {
      return {
        status: 200,
        headers: { "content-type": "application/json" },
        body: [
          { id: 1, name: "Ноутбук", price: 50000, inStock: true },
          { id: 2, name: "Смартфон", price: 30000, inStock: true },
          { id: 3, name: "Книга", price: 500, inStock: true }
        ] as any
      };
    }
    
    if (url.startsWith("/orders") && method === "POST") {
      if (!req.body || !req.body.userId || !req.body.items) {
        return {
          status: 400,
          message: "Неверные данные заказа"
        };
      }
      
      return {
        status: 201,
        headers: { "content-type": "application/json" },
        body: {
          id: Date.now(),
          userId: req.body.userId,
          items: req.body.items,
          total: req.body.items.reduce((sum: number, item: any) => sum + item.price * item.quantity, 0),
          status: "pending",
          createdAt: new Date().toISOString()
        } as any
      };
    }
    
    return {
      status: 404,
      message: "Ресурс не найден"
    };
  }
}

// API клиент
class ApiClient {
  constructor(private httpClient: HttpClient) {}
  
  async createUser(userData: { name: string; email: string }): Promise<ApiResponse<any> | ApiError> {
    return this.httpClient.request({
      method: "POST",
      url: "/users",
      headers: { "content-type": "application/json" },
      body: userData
    });
  }
  
  async getUser(id: number): Promise<ApiResponse<any> | ApiError> {
    return this.httpClient.request({
      method: "GET",
      url: `/users/${id}`,
      headers: { "accept": "application/json" }
    });
  }
  
  async getProducts(): Promise<ApiResponse<any> | ApiError> {
    return this.httpClient.request({
      method: "GET",
      url: "/products",
      headers: { "accept": "application/json" }
    });
  }
  
  async createOrder(orderData: { userId: number; items: any[] }): Promise<ApiResponse<any> | ApiError> {
    return this.httpClient.request({
      method: "POST",
      url: "/orders",
      headers: { "content-type": "application/json" },
      body: orderData
    });
  }
}

// Сервис интеграции с внешними API
class ExternalApiService {
  constructor(private httpClient: HttpClient) {}
  
  async sendEmail(recipient: string, subject: string, body: string): Promise<boolean> {
    const response = await this.httpClient.request({
      method: "POST",
      url: "/external/email",
      headers: { "content-type": "application/json" },
      body: { recipient, subject, body }
    });
    
    return response.status === 200;
  }
  
  async processPayment(amount: number, cardNumber: string): Promise<{ success: boolean; transactionId?: string; error?: string }> {
    const response = await this.httpClient.request({
      method: "POST",
      url: "/external/payment",
      headers: { "content-type": "application/json" },
      body: { amount, cardNumber }
    });
    
    if (response.status === 200) {
      return { success: true, transactionId: `txn-${Date.now()}` };
    }
    
    return { success: false, error: "Ошибка обработки платежа" };
  }
}

// E2E тесты API и интеграций
describe("E2E тестирование API и интеграций", () => {
  let httpClient: HttpClient;
  let apiClient: ApiClient;
  let externalApiService: ExternalApiService;
  
  beforeEach(() => {
    httpClient = new HttpClient("https://api.example.com");
    apiClient = new ApiClient(httpClient);
    externalApiService = new ExternalApiService(httpClient);
  });
  
  test("Полный API сценарий: создание пользователя, получение продуктов, создание заказа", async () => {
    // 1. Создание пользователя
    const createUserResponse = await apiClient.createUser({
      name: "Иван Иванов",
      email: "ivan@example.com"
    });
    
    expect(createUserResponse.status).toBe(201);
    if (createUserResponse.status === 201) {
      expect(createUserResponse.body.name).toBe("Иван Иванов");
      expect(createUserResponse.body.email).toBe("ivan@example.com");
      expect(createUserResponse.body.id).toBeDefined();
    }
    
    // 2. Получение списка продуктов
    const getProductsResponse = await apiClient.getProducts();
    
    expect(getProductsResponse.status).toBe(200);
    if (getProductsResponse.status === 200) {
      expect(getProductsResponse.body).toHaveLength(3);
      expect(getProductsResponse.body.some((p: any) => p.name === "Ноутбук")).toBe(true);
    }
    
    // 3. Создание заказа
    const createOrderResponse = await apiClient.createOrder({
      userId: 1,
      items: [
        { productId: 1, quantity: 1, price: 50000 },
        { productId: 3, quantity: 2, price: 500 }
      ]
    });
    
    expect(createOrderResponse.status).toBe(201);
    if (createOrderResponse.status === 201) {
      expect(createOrderResponse.body.userId).toBe(1);
      expect(createOrderResponse.body.items).toHaveLength(2);
      expect(createOrderResponse.body.total).toBe(51000); // 50000 + 2 * 500
    }
  });
  
  test("Сценарий с ошибками API", async () => {
    // 1. Попытка создания пользователя с неверными данными
    const invalidUserResponse = await apiClient.createUser({
      name: "", // Пустое имя
      email: "invalid-email" // Неверный формат email
    });
    
    expect(invalidUserResponse.status).toBe(400);
    if (invalidUserResponse.status === 400) {
      expect(invalidUserResponse.message).toBe("Неверные данные пользователя");
    }
    
    // 2. Попытка получения несуществующего пользователя
    const getUserResponse = await apiClient.getUser(999);
    
    // В симуляции возвращается 404
    expect(getUserResponse.status).toBe(404);
    
    // 3. Попытка создания заказа с неверными данными
    const invalidOrderResponse = await apiClient.createOrder({
      userId: 1,
      items: [] // Пустой список товаров
    });
    
    expect(invalidOrderResponse.status).toBe(400);
    if (invalidOrderResponse.status === 400) {
      expect(invalidOrderResponse.message).toBe("Неверные данные заказа");
    }
  });
  
  test("Интеграционный сценарий: создание заказа с оплатой и уведомлением", async () => {
    // 1. Создание пользователя
    const createUserResponse = await apiClient.createUser({
      name: "Мария Петрова",
      email: "maria@example.com"
    });
    
    // 2. Создание заказа
    const createOrderResponse = await apiClient.createOrder({
      userId: 2,
      items: [
        { productId: 2, quantity: 1, price: 30000 }
      ]
    });
    
    // 3. Обработка платежа
    const paymentResult = await externalApiService.processPayment(30000, "1234-5678-9012-3456");
    
    expect(paymentResult.success).toBe(true);
    expect(paymentResult.transactionId).toBeDefined();
    
    // 4. Отправка уведомления
    const emailSent = await externalApiService.sendEmail(
      "maria@example.com",
      "Подтверждение заказа",
      "Ваш заказ успешно оформлен и оплачен"
    );
    
    expect(emailSent).toBe(true);
  });
  
  test("Сценарий с ошибками интеграции", async () => {
    // 1. Ошибка обработки платежа
    const paymentResult = await externalApiService.processPayment(30000, "invalid-card");
    
    expect(paymentResult.success).toBe(false);
    expect(paymentResult.error).toBe("Ошибка обработки платежа");
    
    // 2. Ошибка отправки email
    const emailSent = await externalApiService.sendEmail(
      "invalid-email",
      "Тест",
      "Тестовое сообщение"
    );
    
    // В симуляции возвращается false для ошибок
    expect(emailSent).toBe(false);
  });
  
  test("Сценарий с параллельными запросами", async () => {
    // Создание нескольких параллельных запросов
    const requests = [
      apiClient.getUser(1),
      apiClient.getProducts(),
      apiClient.createUser({ name: "Пользователь 1", email: "user1@example.com" }),
      apiClient.createUser({ name: "Пользователь 2", email: "user2@example.com" })
    ];
    
    const responses = await Promise.all(requests);
    
    // Проверка всех ответов
    expect(responses[0].status).toBe(200); // Получение пользователя
    expect(responses[1].status).toBe(200); // Получение продуктов
    expect(responses[2].status).toBe(201); // Создание пользователя 1
    expect(responses[3].status).toBe(201); // Создание пользователя 2
    
    // Проверка данных
    if (responses[2].status === 201) {
      expect(responses[2].body.name).toBe("Пользователь 1");
    }
    
    if (responses[3].status === 201) {
      expect(responses[3].body.name).toBe("Пользователь 2");
    }
  });
});
```

## Тестирование потоков данных

Потоки данных в функциональных системах представляют собой цепочки преобразований, которые требуют комплексного E2E тестирования.

```typescript
// Модели для потоков данных
interface RawData {
  id: string;
  timestamp: string;
  values: string[];
  metadata: {
    source: string;
    tags: string[];
  };
}

interface ProcessedData {
  id: number;
  timestamp: Date;
  values: number[];
  source: string;
  tagCount: number;
  processedAt: Date;
}

interface DataReport {
  totalRecords: number;
  averageValue: number;
  sources: Record<string, number>;
  timeRange: {
    start: Date;
    end: Date;
  };
}

// Сервис обработки данных
class DataProcessingService {
  async processRawData(rawData: RawData[]): Promise<ProcessedData[]> {
    return rawData.map(data => ({
      id: parseInt(data.id, 10),
      timestamp: new Date(data.timestamp),
      values: data.values.map(v => parseFloat(v)).filter(v => !isNaN(v)),
      source: data.metadata.source,
      tagCount: data.metadata.tags.length,
      processedAt: new Date()
    }));
  }
  
  async generateReport(processedData: ProcessedData[]): Promise<DataReport> {
    if (processedData.length === 0) {
      return {
        totalRecords: 0,
        averageValue: 0,
        sources: {},
        timeRange: {
          start: new Date(),
          end: new Date()
        }
      };
    }
    
    const allValues = processedData.flatMap(d => d.values);
    const averageValue = allValues.reduce((sum, val) => sum + val, 0) / allValues.length;
    
    const sources: Record<string, number> = {};
    processedData.forEach(d => {
      sources[d.source] = (sources[d.source] || 0) + 1;
    });
    
    const timestamps = processedData.map(d => d.timestamp);
    const start = new Date(Math.min(...timestamps.map(t => t.getTime())));
    const end = new Date(Math.max(...timestamps.map(t => t.getTime())));
    
    return {
      totalRecords: processedData.length,
      averageValue,
      sources,
      timeRange: { start, end }
    };
  }
  
  async saveReport(report: DataReport): Promise<boolean> {
    // Симуляция сохранения отчета
    console.log("Сохранение отчета:", report);
    return true;
  }
}

// Сервис потоков данных
class DataFlowService {
  constructor(
    private dataProcessingService: DataProcessingService
  ) {}
  
  async processDataStream(rawData: RawData[]): Promise<{ processed: ProcessedData[]; report: DataReport }> {
    // 1. Обработка сырых данных
    const processed = await this.dataProcessingService.processRawData(rawData);
    
    // 2. Генерация отчета
    const report = await this.dataProcessingService.generateReport(processed);
    
    // 3. Сохранение отчета
    await this.dataProcessingService.saveReport(report);
    
    return { processed, report };
  }
}

// E2E тесты потоков данных
describe("E2E тестирование потоков данных", () => {
  let dataProcessingService: DataProcessingService;
  let dataFlowService: DataFlowService;
  
  beforeEach(() => {
    dataProcessingService = new DataProcessingService();
    dataFlowService = new DataFlowService(dataProcessingService);
  });
  
  test("Полный поток данных: обработка, генерация отчета, сохранение", async () => {
    // 1. Подготовка тестовых данных
    const rawData: RawData[] = [
      {
        id: "1",
        timestamp: "2023-01-01T10:00:00Z",
        values: ["1.5", "2.3", "3.7"],
        metadata: {
          source: "sensor-1",
          tags: ["temperature", "indoor"]
        }
      },
      {
        id: "2",
        timestamp: "2023-01-01T11:00:00Z",
        values: ["2.1", "3.2", "4.1"],
        metadata: {
          source: "sensor-2",
          tags: ["temperature", "outdoor"]
        }
      },
      {
        id: "3",
        timestamp: "2023-01-01T12:00:00Z",
        values: ["1.8", "2.9", "3.5"],
        metadata: {
          source: "sensor-1",
          tags: ["temperature", "indoor", "critical"]
        }
      }
    ];
    
    // 2. Обработка потока данных
    const result = await dataFlowService.processDataStream(rawData);
    
    // 3. Проверка обработанных данных
    expect(result.processed).toHaveLength(3);
    
    const firstRecord = result.processed[0];
    expect(firstRecord.id).toBe(1);
    expect(firstRecord.values).toEqual([1.5, 2.3, 3.7]);
    expect(firstRecord.source).toBe("sensor-1");
    expect(firstRecord.tagCount).toBe(2);
    expect(firstRecord.timestamp).toBeInstanceOf(Date);
    
    // 4. Проверка отчета
    expect(result.report.totalRecords).toBe(3);
    expect(result.report.averageValue).toBeCloseTo(2.76, 2); // (1.5+2.3+3.7+2.1+3.2+4.1+1.8+2.9+3.5)/9
    
    expect(result.report.sources).toEqual({
      "sensor-1": 2,
      "sensor-2": 1
    });
    
    expect(result.report.timeRange.start).toBeInstanceOf(Date);
    expect(result.report.timeRange.end).toBeInstanceOf(Date);
    
    // 5. Проверка временного диапазона
    expect(result.report.timeRange.start.toISOString()).toBe("2023-01-01T10:00:00.000Z");
    expect(result.report.timeRange.end.toISOString()).toBe("2023-01-01T12:00:00.000Z");
  });
  
  test("Поток данных с некорректными значениями", async () => {
    // 1. Подготовка данных с ошибками
    const rawData: RawData[] = [
      {
        id: "invalid",
        timestamp: "invalid-date",
        values: ["1.5", "invalid", "3.7"],
        metadata: {
          source: "sensor-1",
          tags: ["temperature"]
        }
      },
      {
        id: "2",
        timestamp: "2023-01-01T11:00:00Z",
        values: ["2.1", "3.2"],
        metadata: {
          source: "sensor-2",
          tags: ["temperature"]
        }
      }
    ];
    
    // 2. Обработка потока данных
    const result = await dataFlowService.processDataStream(rawData);
    
    // 3. Проверка обработки ошибок
    expect(result.processed).toHaveLength(2);
    
    const firstRecord = result.processed[0];
    expect(firstRecord.id).toBeNaN(); // Неверный ID
    expect(firstRecord.timestamp.toString()).toBe("Invalid Date"); // Неверная дата
    expect(firstRecord.values).toEqual([1.5, 3.7]); // Неверное значение отфильтровано
    
    const secondRecord = result.processed[1];
    expect(secondRecord.id).toBe(2);
    expect(secondRecord.values).toEqual([2.1, 3.2]);
  });
  
  test("Поток данных с пустыми данными", async () => {
    // 1. Обработка пустого массива
    const result = await dataFlowService.processDataStream([]);
    
    // 2. Проверка обработки пустых данных
    expect(result.processed).toHaveLength(0);
    expect(result.report.totalRecords).toBe(0);
    expect(result.report.averageValue).toBe(0);
    expect(result.report.sources).toEqual({});
  });
  
  test("Поток данных с большими объемами", async () => {
    // 1. Генерация большого объема тестовых данных
    const rawData: RawData[] = [];
    for (let i = 0; i < 1000; i++) {
      rawData.push({
        id: `${i + 1}`,
        timestamp: `2023-01-01T${String(i % 24).padStart(2, '0')}:00:00Z`,
        values: [`${Math.random() * 100}`, `${Math.random() * 100}`],
        metadata: {
          source: `sensor-${(i % 5) + 1}`,
          tags: ["temperature"]
        }
      });
    }
    
    // 2. Измерение производительности
    const startTime = Date.now();
    const result = await dataFlowService.processDataStream(rawData);
    const endTime = Date.now();
    
    // 3. Проверка результатов
    expect(result.processed).toHaveLength(1000);
    expect(result.report.totalRecords).toBe(1000);
    expect(result.report.averageValue).toBeGreaterThan(0);
    expect(endTime - startTime).toBeLessThan(10000); // Менее 10 секунд
    
    // 4. Проверка распределения источников
    const totalSources = Object.values(result.report.sources).reduce((sum, count) => sum + count, 0);
    expect(totalSources).toBe(1000);
  });
  
  test("Поток данных с параллельной обработкой", async () => {
    // 1. Создание нескольких потоков данных
    const stream1 = dataFlowService.processDataStream([
      {
        id: "1",
        timestamp: "2023-01-01T10:00:00Z",
        values: ["1.0", "2.0"],
        metadata: { source: "sensor-1", tags: ["test"] }
      }
    ]);
    
    const stream2 = dataFlowService.processDataStream([
      {
        id: "2",
        timestamp: "2023-01-01T11:00:00Z",
        values: ["3.0", "4.0"],
        metadata: { source: "sensor-2", tags: ["test"] }
      }
    ]);
    
    // 2. Параллельная обработка
    const [result1, result2] = await Promise.all([stream1, stream2]);
    
    // 3. Проверка результатов
    expect(result1.processed[0].id).toBe(1);
    expect(result1.processed[0].values).toEqual([1.0, 2.0]);
    
    expect(result2.processed[0].id).toBe(2);
    expect(result2.processed[0].values).toEqual([3.0, 4.0]);
  });
});
```

## Тестирование обработчиков событий

Обработчики событий в функциональных системах представляют собой чистые функции, что упрощает их E2E тестирование.

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

interface PaymentProcessedEvent extends Event {
  type: "PAYMENT_PROCESSED";
  payload: {
    orderId: number;
    transactionId: string;
    amount: number;
    status: "success" | "failed";
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

// Система обработки событий
class EventProcessor {
  private handlers: Record<string, ((event: Event) => Promise<void>)[]> = {};
  
  registerHandler(type: string, handler: (event: Event) => Promise<void>) {
    if (!this.handlers[type]) {
      this.handlers[type] = [];
    }
    this.handlers[type].push(handler);
  }
  
  async processEvent(event: Event): Promise<void> {
    const handlers = this.handlers[event.type] || [];
    await Promise.all(handlers.map(handler => handler(event)));
  }
  
  async processEvents(events: Event[]): Promise<void> {
    await Promise.all(events.map(event => this.processEvent(event)));
  }
}

// Сервисы для обработки событий
class UserService {
  async createUser(userData: { name: string; email: string }): Promise<User> {
    return {
      id: Date.now(),
      name: userData.name,
      email: userData.email,
      createdAt: new Date()
    };
  }
}

class OrderService {
  async createOrder(orderData: { userId: number; items: OrderItem[] }): Promise<Order> {
    const total = orderData.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    
    return {
      id: Date.now(),
      userId: orderData.userId,
      items: orderData.items,
      total,
      status: "pending",
      createdAt: new Date()
    };
  }
}

class PaymentService {
  async processPayment(amount: number): Promise<{ transactionId: string; success: boolean }> {
    // Симуляция обработки платежа
    return {
      transactionId: `txn-${Date.now()}`,
      success: Math.random() > 0.1 // 90% успешных платежей
    };
  }
}

class EmailService {
  async sendEmail(recipient: string, subject: string, body: string): Promise<string> {
    // Симуляция отправки email
    console.log(`Отправка email: ${recipient}, ${subject}`);
    return `email-${Date.now()}`;
  }
}

// Обработчики событий
class UserEventHandler {
  constructor(
    private userService: UserService,
    private eventProcessor: EventProcessor
  ) {
    this.setupHandlers();
  }
  
  private setupHandlers() {
    this.eventProcessor.registerHandler("USER_CREATED", this.handleUserCreated.bind(this));
  }
  
  private async handleUserCreated(event: UserCreatedEvent): Promise<void> {
    // Отправка приветственного email
    const emailService = new EmailService();
    const emailId = await emailService.sendEmail(
      event.payload.email,
      "Добро пожаловать!",
      `Привет, ${event.payload.name}! Добро пожаловать в наш сервис.`
    );
    
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
    
    await this.eventProcessor.processEvent(emailSentEvent);
  }
}

class OrderEventHandler {
  constructor(
    private orderService: OrderService,
    private eventProcessor: EventProcessor
  ) {
    this.setupHandlers();
  }
  
  private setupHandlers() {
    this.eventProcessor.registerHandler("ORDER_PLACED", this.handleOrderPlaced.bind(this));
  }
  
  private async handleOrderPlaced(event: OrderPlacedEvent): Promise<void> {
    // Обработка платежа
    const paymentService = new PaymentService();
    const paymentResult = await paymentService.processPayment(event.payload.total);
    
    // Генерация события обработки платежа
    const paymentEvent: PaymentProcessedEvent = {
      type: "PAYMENT_PROCESSED",
      payload: {
        orderId: event.payload.orderId,
        transactionId: paymentResult.transactionId,
        amount: event.payload.total,
        status: paymentResult.success ? "success" : "failed"
      },
      timestamp: Date.now(),
      correlationId: event.correlationId
    };
    
    await this.eventProcessor.processEvent(paymentEvent);
    
    // Если платеж успешен, отправляем подтверждение заказа
    if (paymentResult.success) {
      const emailService = new EmailService();
      const emailId = await emailService.sendEmail(
        "user@example.com", // В реальной реализации получаем email пользователя
        `Подтверждение заказа #${event.payload.orderId}`,
        `Ваш заказ #${event.payload.orderId} на сумму ${event.payload.total} успешно оплачен.`
      );
      
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
      
      await this.eventProcessor.processEvent(emailSentEvent);
    }
  }
}

// E2E тесты обработчиков событий
describe("E2E тестирование обработчиков событий", () => {
  let eventProcessor: EventProcessor;
  let userService: UserService;
  let orderService: OrderService;
  let userEventHandler: UserEventHandler;
  let orderEventHandler: OrderEventHandler;
  
  beforeEach(() => {
    eventProcessor = new EventProcessor();
    userService = new UserService();
    orderService = new OrderService();
    userEventHandler = new UserEventHandler(userService, eventProcessor);
    orderEventHandler = new OrderEventHandler(orderService, eventProcessor);
  });
  
  test("Полный сценарий: создание пользователя с отправкой приветственного email", async () => {
    // 1. Создание события создания пользователя
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
    
    // 2. Моки для проверки вызовов
    const emailService = new EmailService();
    const sendEmailSpy = jest.spyOn(emailService, "sendEmail")
      .mockImplementation(async () => "email-456");
    
    // 3. Обработка события
    await eventProcessor.processEvent(userCreatedEvent);
    
    // 4. Проверка вызовов
    expect(sendEmailSpy).toHaveBeenCalledWith(
      "ivan@example.com",
      "Добро пожаловать!",
      "Привет, Иван Иванов! Добро пожаловать в наш сервис."
    );
  });
  
  test("Полный сценарий: размещение заказа с обработкой платежа и отправкой подтверждения", async () => {
    // 1. Создание события размещения заказа
    const orderPlacedEvent: OrderPlacedEvent = {
      type: "ORDER_PLACED",
      payload: {
        orderId: 1001,
        userId: 1,
        items: [
          { productId: 1, quantity: 1, price: 50000 },
          { productId: 2, quantity: 2, price: 500 }
        ],
        total: 51000
      },
      timestamp: Date.now(),
      correlationId: "corr-789"
    };
    
    // 2. Моки для проверки вызовов
    const paymentService = new PaymentService();
    const processPaymentSpy = jest.spyOn(paymentService, "processPayment")
      .mockResolvedValue({ transactionId: "txn-999", success: true });
    
    const emailService = new EmailService();
    const sendEmailSpy = jest.spyOn(emailService, "sendEmail")
      .mockImplementation(async () => "email-111");
    
    // 3. Обработка события
    await eventProcessor.processEvent(orderPlacedEvent);
    
    // 4. Проверка вызовов
    expect(processPaymentSpy).toHaveBeenCalledWith(51000);
    expect(sendEmailSpy).toHaveBeenCalledWith(
      "user@example.com",
      "Подтверждение заказа #1001",
      "Ваш заказ #1001 на сумму 51000 успешно оплачен."
    );
  });
  
  test("Сценарий с неуспешным платежом", async () => {
    // 1. Создание события размещения заказа
    const orderPlacedEvent: OrderPlacedEvent = {
      type: "ORDER_PLACED",
      payload: {
        orderId: 1002,
        userId: 2,
        items: [{ productId: 1, quantity: 1, price: 30000 }],
        total: 30000
      },
      timestamp: Date.now(),
      correlationId: "corr-222"
    };
    
    // 2. Мок для неуспешного платежа
    const paymentService = new PaymentService();
    const processPaymentSpy = jest.spyOn(paymentService, "processPayment")
      .mockResolvedValue({ transactionId: "txn-888", success: false });
    
    const emailService = new EmailService();
    const sendEmailSpy = jest.spyOn(emailService, "sendEmail");
    
    // 3. Обработка события
    await eventProcessor.processEvent(orderPlacedEvent);
    
    // 4. Проверка вызовов
    expect(processPaymentSpy).toHaveBeenCalledWith(30000);
    // Email не должен быть отправлен при неуспешном платеже
    expect(sendEmailSpy).not.toHaveBeenCalled();
  });
  
  test("Сценарий с обработкой нескольких событий", async () => {
    // 1. Создание нескольких событий
    const events: Event[] = [
      {
        type: "USER_CREATED",
        payload: {
          userId: 1,
          name: "Пользователь 1",
          email: "user1@example.com"
        },
        timestamp: Date.now(),
        correlationId: "corr-333"
      },
      {
        type: "USER_CREATED",
        payload: {
          userId: 2,
          name: "Пользователь 2",
          email: "user2@example.com"
        },
        timestamp: Date.now(),
        correlationId: "corr-444"
      },
      {
        type: "ORDER_PLACED",
        payload: {
          orderId: 2001,
          userId: 1,
          items: [{ productId: 1, quantity: 1, price: 10000 }],
          total: 10000
        },
        timestamp: Date.now(),
        correlationId: "corr-555"
      }
    ];
    
    // 2. Моки для проверки вызовов
    const emailService = new EmailService();
    const sendEmailSpy = jest.spyOn(emailService, "sendEmail")
      .mockImplementation(async () => "email-666");
    
    const paymentService = new PaymentService();
    const processPaymentSpy = jest.spyOn(paymentService, "processPayment")
      .mockResolvedValue({ transactionId: "txn-777", success: true });
    
    // 3. Обработка всех событий
    await eventProcessor.processEvents(events);
    
    // 4. Проверка вызовов
    expect(sendEmailSpy).toHaveBeenCalledTimes(3); // 2 приветственных + 1 подтверждение заказа
    expect(processPaymentSpy).toHaveBeenCalledTimes(1);
  });
  
  test("Сценарий с ошибками в обработчиках", async () => {
    // 1. Создание события
    const userCreatedEvent: UserCreatedEvent = {
      type: "USER_CREATED",
      payload: {
        userId: 1,
        name: "Иван Иванов",
        email: "ivan@example.com"
      },
      timestamp: Date.now(),
      correlationId: "corr-888"
    };
    
    // 2. Мок для ошибки отправки email
    const emailService = new EmailService();
    const sendEmailSpy = jest.spyOn(emailService, "sendEmail")
      .mockRejectedValue(new Error("Ошибка отправки email"));
    
    // 3. Обработка события (должна обрабатываться ошибка)
    await expect(eventProcessor.processEvent(userCreatedEvent)).resolves.not.toThrow();
    
    // 4. Проверка, что ошибка была обработана
    expect(sendEmailSpy).toHaveBeenCalledWith(
      "ivan@example.com",
      "Добро пожаловать!",
      expect.any(String)
    );
  });
});
```

## Тестирование систем с состоянием

Даже в функциональных системах иногда требуется работа с состоянием, но это делается через контролируемые механизмы.

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
interface ApplicationState {
  users: User[];
  products: Product[];
  orders: Order[];
  nextUserId: number;
  nextOrderId: number;
}

// Операции со state
const getState = (): StateMonad<ApplicationState, ApplicationState> => {
  return StateMonad.get<ApplicationState>();
};

const setState = (state: ApplicationState): StateMonad<ApplicationState, void> => {
  return StateMonad.put(state);
};

const createUser = (userData: Omit<User, "id" | "createdAt">): StateMonad<ApplicationState, User> => {
  return StateMonad.get<ApplicationState>().flatMap(state => {
    const user: User = {
      id: state.nextUserId,
      name: userData.name,
      email: userData.email,
      createdAt: new Date()
    };
    
    const newState: ApplicationState = {
      ...state,
      users: [...state.users, user],
      nextUserId: state.nextUserId + 1
    };
    
    return StateMonad.put(newState).map(() => user);
  });
};

const createProduct = (productData: Omit<Product, "id" | "inStock">): StateMonad<ApplicationState, Product> => {
  return StateMonad.get<ApplicationState>().flatMap(state => {
    const product: Product = {
      id: state.products.length + 1,
      name: productData.name,
      price: productData.price,
      inStock: true
    };
    
    const newState: ApplicationState = {
      ...state,
      products: [...state.products, product]
    };
    
    return StateMonad.put(newState).map(() => product);
  });
};

const createOrder = (orderData: { userId: number; items: Omit<OrderItem, "price">[] }): StateMonad<ApplicationState, Order | null> => {
  return StateMonad.get<ApplicationState>().flatMap(state => {
    const user = state.users.find(u => u.id === orderData.userId);
    if (!user) {
      return StateMonad.of(null);
    }
    
    const orderItems: OrderItem[] = [];
    let total = 0;
    
    for (const item of orderData.items) {
      const product = state.products.find(p => p.id === item.productId);
      if (!product || !product.inStock) {
        return StateMonad.of(null);
      }
      
      const orderItem: OrderItem = {
        productId: item.productId,
        quantity: item.quantity,
        price: product.price
      };
      
      orderItems.push(orderItem);
      total += product.price * item.quantity;
    }
    
    const order: Order = {
      id: state.nextOrderId,
      userId: orderData.userId,
      items: orderItems,
      total,
      status: "pending",
      createdAt: new Date()
    };
    
    const newState: ApplicationState = {
      ...state,
      orders: [...state.orders, order],
      nextOrderId: state.nextOrderId + 1
    };
    
    return StateMonad.put(newState).map(() => order);
  });
};

const updateUserStock = (productId: number, inStock: boolean): StateMonad