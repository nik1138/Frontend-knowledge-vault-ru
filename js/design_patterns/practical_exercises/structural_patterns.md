# Практические упражнения: Структурные паттерны проектирования

## Упражнение 1: Адаптер (Adapter)

### Задача
Создайте систему интеграции различных API платежных систем с использованием паттерна адаптер для унификации интерфейса.

### Требования
1. Различные платежные системы с разными интерфейсами
2. Адаптеры для унификации интерфейса
3. Клиентский код, работающий с унифицированным интерфейсом
4. Обработка ошибок и логирование

### Решение
```javascript
// Интерфейс платежной системы
class PaymentProcessor {
  processPayment(amount, currency, cardData) {
    throw new Error('Метод processPayment должен быть реализован');
  }
  
  refundPayment(transactionId, amount) {
    throw new Error('Метод refundPayment должен быть реализован');
  }
  
  getTransactionStatus(transactionId) {
    throw new Error('Метод getTransactionStatus должен быть реализован');
  }
}

// Сторонняя платежная система Stripe (имитация)
class StripeAPI {
  constructor() {
    this.transactions = new Map();
  }
  
  makeCharge(amount, currency, source) {
    console.log(`Stripe: Обработка платежа ${amount} ${currency}`);
    
    // Имитация случайных ошибок
    if (Math.random() < 0.1) {
      throw new Error('Stripe: Ошибка обработки платежа');
    }
    
    const transactionId = `stripe_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    const transaction = {
      id: transactionId,
      amount,
      currency,
      source,
      status: 'success',
      timestamp: new Date()
    };
    
    this.transactions.set(transactionId, transaction);
    return transaction;
  }
  
  createRefund(chargeId, amount) {
    console.log(`Stripe: Обработка возврата ${amount} для транзакции ${chargeId}`);
    
    const transaction = this.transactions.get(chargeId);
    if (!transaction) {
      throw new Error('Stripe: Транзакция не найдена');
    }
    
    const refundId = `refund_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    return {
      id: refundId,
      chargeId,
      amount,
      status: 'success',
      timestamp: new Date()
    };
  }
  
  retrieveCharge(chargeId) {
    const transaction = this.transactions.get(chargeId);
    if (!transaction) {
      throw new Error('Stripe: Транзакция не найдена');
    }
    return transaction;
  }
}

// Сторонняя платежная система PayPal (имитация)
class PayPalAPI {
  constructor() {
    this.payments = new Map();
  }
  
  processPayment(paymentDetails) {
    console.log(`PayPal: Обработка платежа ${paymentDetails.amount} ${paymentDetails.currency}`);
    
    // Имитация случайных ошибок
    if (Math.random() < 0.15) {
      throw new Error('PayPal: Ошибка обработки платежа');
    }
    
    const paymentId = `paypal_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    const payment = {
      id: paymentId,
      ...paymentDetails,
      state: 'approved',
      create_time: new Date().toISOString()
    };
    
    this.payments.set(paymentId, payment);
    return payment;
  }
  
  refundSale(saleId, refundDetails) {
    console.log(`PayPal: Обработка возврата для транзакции ${saleId}`);
    
    const payment = this.payments.get(saleId);
    if (!payment) {
      throw new Error('PayPal: Платеж не найден');
    }
    
    const refundId = `paypal_refund_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    return {
      id: refundId,
      sale_id: saleId,
      ...refundDetails,
      state: 'completed',
      create_time: new Date().toISOString()
    };
  }
  
  getPaymentDetails(paymentId) {
    const payment = this.payments.get(paymentId);
    if (!payment) {
      throw new Error('PayPal: Платеж не найден');
    }
    return payment;
  }
}

// Адаптер для Stripe
class StripeAdapter extends PaymentProcessor {
  constructor(stripeAPI) {
    super();
    this.stripe = stripeAPI;
  }
  
  processPayment(amount, currency, cardData) {
    try {
      const result = this.stripe.makeCharge(amount, currency, cardData);
      return {
        transactionId: result.id,
        amount: result.amount,
        currency: result.currency,
        status: result.status,
        timestamp: result.timestamp
      };
    } catch (error) {
      console.error('Ошибка Stripe:', error.message);
      throw new Error(`Ошибка обработки платежа через Stripe: ${error.message}`);
    }
  }
  
  refundPayment(transactionId, amount) {
    try {
      const result = this.stripe.createRefund(transactionId, amount);
      return {
        refundId: result.id,
        transactionId: result.chargeId,
        amount: result.amount,
        status: result.status,
        timestamp: result.timestamp
      };
    } catch (error) {
      console.error('Ошибка возврата Stripe:', error.message);
      throw new Error(`Ошибка возврата платежа через Stripe: ${error.message}`);
    }
  }
  
  getTransactionStatus(transactionId) {
    try {
      const transaction = this.stripe.retrieveCharge(transactionId);
      return {
        transactionId: transaction.id,
        status: transaction.status,
        amount: transaction.amount,
        currency: transaction.currency
      };
    } catch (error) {
      console.error('Ошибка получения статуса Stripe:', error.message);
      throw new Error(`Ошибка получения статуса транзакции через Stripe: ${error.message}`);
    }
  }
}

// Адаптер для PayPal
class PayPalAdapter extends PaymentProcessor {
  constructor(paypalAPI) {
    super();
    this.paypal = paypalAPI;
  }
  
  processPayment(amount, currency, cardData) {
    try {
      const paymentDetails = {
        amount,
        currency,
        card_number: cardData.number,
        card_expiry: cardData.expiry,
        card_cvv: cardData.cvv
      };
      
      const result = this.paypal.processPayment(paymentDetails);
      return {
        transactionId: result.id,
        amount: result.amount,
        currency: result.currency,
        status: result.state,
        timestamp: new Date(result.create_time)
      };
    } catch (error) {
      console.error('Ошибка PayPal:', error.message);
      throw new Error(`Ошибка обработки платежа через PayPal: ${error.message}`);
    }
  }
  
  refundPayment(transactionId, amount) {
    try {
      const refundDetails = {
        amount
      };
      
      const result = this.paypal.refundSale(transactionId, refundDetails);
      return {
        refundId: result.id,
        transactionId: result.sale_id,
        amount: result.amount,
        status: result.state,
        timestamp: new Date(result.create_time)
      };
    } catch (error) {
      console.error('Ошибка возврата PayPal:', error.message);
      throw new Error(`Ошибка возврата платежа через PayPal: ${error.message}`);
    }
  }
  
  getTransactionStatus(transactionId) {
    try {
      const payment = this.paypal.getPaymentDetails(transactionId);
      return {
        transactionId: payment.id,
        status: payment.state,
        amount: payment.amount,
        currency: payment.currency
      };
    } catch (error) {
      console.error('Ошибка получения статуса PayPal:', error.message);
      throw new Error(`Ошибка получения статуса транзакции через PayPal: ${error.message}`);
    }
  }
}

// Фабрика адаптеров
class PaymentAdapterFactory {
  static createAdapter(type) {
    switch (type.toLowerCase()) {
      case 'stripe':
        return new StripeAdapter(new StripeAPI());
      case 'paypal':
        return new PayPalAdapter(new PayPalAPI());
      default:
        throw new Error(`Неизвестный тип платежной системы: ${type}`);
    }
  }
}

// Клиентский код
class PaymentService {
  constructor() {
    this.processors = new Map();
  }
  
  addPaymentProcessor(name, processor) {
    if (!(processor instanceof PaymentProcessor)) {
      throw new Error('Процессор должен быть экземпляром PaymentProcessor');
    }
    this.processors.set(name, processor);
  }
  
  processPayment(processorName, amount, currency, cardData) {
    const processor = this.processors.get(processorName);
    if (!processor) {
      throw new Error(`Платежный процессор ${processorName} не найден`);
    }
    
    console.log(`Обработка платежа через ${processorName}: ${amount} ${currency}`);
    return processor.processPayment(amount, currency, cardData);
  }
  
  refundPayment(processorName, transactionId, amount) {
    const processor = this.processors.get(processorName);
    if (!processor) {
      throw new Error(`Платежный процессор ${processorName} не найден`);
    }
    
    console.log(`Обработка возврата через ${processorName}: ${amount}`);
    return processor.refundPayment(transactionId, amount);
  }
  
  getTransactionStatus(processorName, transactionId) {
    const processor = this.processors.get(processorName);
    if (!processor) {
      throw new Error(`Платежный процессор ${processorName} не найден`);
    }
    
    return processor.getTransactionStatus(transactionId);
  }
}

// Пример использования
const paymentService = new PaymentService();

// Добавляем адаптеры для разных платежных систем
paymentService.addPaymentProcessor('stripe', PaymentAdapterFactory.createAdapter('stripe'));
paymentService.addPaymentProcessor('paypal', PaymentAdapterFactory.createAdapter('paypal'));

// Данные карты для тестирования
const cardData = {
  number: '4242424242424242',
  expiry: '12/25',
  cvv: '123'
};

try {
  // Обработка платежа через Stripe
  console.log('=== Платеж через Stripe ===');
  const stripeResult = paymentService.processPayment('stripe', 100, 'USD', cardData);
  console.log('Результат:', stripeResult);
  
  // Проверка статуса транзакции
  const stripeStatus = paymentService.getTransactionStatus('stripe', stripeResult.transactionId);
  console.log('Статус:', stripeStatus);
  
  // Возврат платежа
  const stripeRefund = paymentService.refundPayment('stripe', stripeResult.transactionId, 50);
  console.log('Возврат:', stripeRefund);
  
} catch (error) {
  console.error('Ошибка обработки платежа через Stripe:', error.message);
}

try {
  // Обработка платежа через PayPal
  console.log('\n=== Платеж через PayPal ===');
  const paypalResult = paymentService.processPayment('paypal', 150, 'EUR', cardData);
  console.log('Результат:', paypalResult);
  
  // Проверка статуса транзакции
  const paypalStatus = paymentService.getTransactionStatus('paypal', paypalResult.transactionId);
  console.log('Статус:', paypalStatus);
  
} catch (error) {
  console.error('Ошибка обработки платежа через PayPal:', error.message);
}

// Попытка использовать несуществующий процессор
try {
  paymentService.processPayment('unknown', 100, 'USD', cardData);
} catch (error) {
  console.error('Ошибка:', error.message);
}
```

## Упражнение 2: Фасад (Facade)

### Задача
Создайте фасад для упрощения работы с комплексной системой управления заказами интернет-магазина.

### Требования
1. Упрощенный интерфейс для работы с заказами
2. Интеграция с различными подсистемами (инвентарь, платежи, доставка)
3. Обработка ошибок и логирование
4. Поддержка различных типов заказов

### Решение
```javascript
// Подсистема управления инвентарем
class InventorySystem {
  constructor() {
    this.products = new Map([
      ['laptop', { name: 'Ноутбук', price: 1000, stock: 10 }],
      ['mouse', { name: 'Мышь', price: 25, stock: 50 }],
      ['keyboard', { name: 'Клавиатура', price: 75, stock: 30 }]
    ]);
  }
  
  checkAvailability(productId, quantity) {
    const product = this.products.get(productId);
    if (!product) {
      throw new Error(`Продукт ${productId} не найден`);
    }
    
    return product.stock >= quantity;
  }
  
  reserveProduct(productId, quantity) {
    const product = this.products.get(productId);
    if (!product) {
      throw new Error(`Продукт ${productId} не найден`);
    }
    
    if (product.stock < quantity) {
      throw new Error(`Недостаточно товара ${product.name} в наличии`);
    }
    
    product.stock -= quantity;
    console.log(`Зарезервировано ${quantity} шт. ${product.name}`);
    return true;
  }
  
  releaseProduct(productId, quantity) {
    const product = this.products.get(productId);
    if (product) {
      product.stock += quantity;
      console.log(`Освобождено ${quantity} шт. ${product.name}`);
    }
  }
  
  getProductInfo(productId) {
    return this.products.get(productId);
  }
}

// Подсистема обработки платежей
class PaymentSystem {
  constructor() {
    this.transactions = new Map();
  }
  
  processPayment(amount, paymentMethod, cardData) {
    console.log(`Обработка платежа ${amount} через ${paymentMethod}`);
    
    // Имитация случайных ошибок
    if (Math.random() < 0.1) {
      throw new Error('Ошибка обработки платежа');
    }
    
    const transactionId = `txn_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    const transaction = {
      id: transactionId,
      amount,
      method: paymentMethod,
      status: 'completed',
      timestamp: new Date()
    };
    
    this.transactions.set(transactionId, transaction);
    return transactionId;
  }
  
  refundPayment(transactionId, amount) {
    console.log(`Обработка возврата ${amount} для транзакции ${transactionId}`);
    
    const transaction = this.transactions.get(transactionId);
    if (!transaction) {
      throw new Error('Транзакция не найдена');
    }
    
    const refundId = `refund_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    return refundId;
  }
}

// Подсистема доставки
class ShippingSystem {
  constructor() {
    this.shipments = new Map();
  }
  
  calculateShippingCost(weight, destination) {
    // Простая формула расчета стоимости доставки
    const baseCost = 10;
    const weightCost = weight * 2;
    const distanceCost = destination.includes('дальний') ? 20 : 5;
    
    return baseCost + weightCost + distanceCost;
  }
  
  createShipment(orderId, address, items) {
    console.log(`Создание доставки для заказа ${orderId}`);
    
    const totalWeight = items.reduce((sum, item) => sum + (item.weight || 1), 0);
    const cost = this.calculateShippingCost(totalWeight, address);
    
    const shipmentId = `ship_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    const shipment = {
      id: shipmentId,
      orderId,
      address,
      items,
      cost,
      status: 'pending',
      trackingNumber: `TRK${Math.random().toString(36).substr(2, 12).toUpperCase()}`,
      estimatedDelivery: new Date(Date.now() + 3 * 24 * 60 * 60 * 1000) // 3 дня
    };
    
    this.shipments.set(shipmentId, shipment);
    return shipment;
  }
  
  updateShipmentStatus(shipmentId, status) {
    const shipment = this.shipments.get(shipmentId);
    if (shipment) {
      shipment.status = status;
      console.log(`Статус доставки ${shipmentId} обновлен: ${status}`);
    }
  }
  
  getShipmentInfo(shipmentId) {
    return this.shipments.get(shipmentId);
  }
}

// Подсистема уведомлений
class NotificationSystem {
  sendEmail(to, subject, message) {
    console.log(`Отправка email на ${to}: ${subject}`);
    // Имитация отправки email
    return true;
  }
  
  sendSMS(phone, message) {
    console.log(`Отправка SMS на ${phone}: ${message}`);
    // Имитация отправки SMS
    return true;
  }
  
  sendNotification(userId, type, message) {
    console.log(`Отправка уведомления пользователю ${userId} (${type}): ${message}`);
    // Имитация отправки уведомления
    return true;
  }
}

// Фасад для управления заказами
class OrderFacade {
  constructor() {
    this.inventory = new InventorySystem();
    this.payment = new PaymentSystem();
    this.shipping = new ShippingSystem();
    this.notifications = new NotificationSystem();
    this.orders = new Map();
  }
  
  // Создание заказа
  createOrder(userId, items, shippingAddress, paymentMethod, cardData) {
    console.log(`Создание заказа для пользователя ${userId}`);
    
    try {
      // 1. Проверка наличия товаров
      console.log('1. Проверка наличия товаров...');
      for (const item of items) {
        if (!this.inventory.checkAvailability(item.productId, item.quantity)) {
          throw new Error(`Товар ${item.productId} отсутствует в нужном количестве`);
        }
      }
      
      // 2. Резервирование товаров
      console.log('2. Резервирование товаров...');
      for (const item of items) {
        this.inventory.reserveProduct(item.productId, item.quantity);
      }
      
      // 3. Расчет общей стоимости
      console.log('3. Расчет стоимости...');
      let totalAmount = 0;
      const orderItems = items.map(item => {
        const productInfo = this.inventory.getProductInfo(item.productId);
        const itemTotal = productInfo.price * item.quantity;
        totalAmount += itemTotal;
        return {
          ...item,
          name: productInfo.name,
          price: productInfo.price,
          total: itemTotal
        };
      });
      
      // 4. Расчет стоимости доставки
      console.log('4. Расчет стоимости доставки...');
      const shipment = this.shipping.createShipment(
        `order_${Date.now()}`, 
        shippingAddress, 
        orderItems
      );
      totalAmount += shipment.cost;
      
      // 5. Обработка платежа
      console.log('5. Обработка платежа...');
      const transactionId = this.payment.processPayment(
        totalAmount, 
        paymentMethod, 
        cardData
      );
      
      // 6. Создание заказа
      console.log('6. Создание заказа...');
      const orderId = `order_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
      const order = {
        id: orderId,
        userId,
        items: orderItems,
        totalAmount,
        shippingAddress,
        transactionId,
        shipmentId: shipment.id,
        status: 'confirmed',
        createdAt: new Date()
      };
      
      this.orders.set(orderId, order);
      
      // 7. Отправка уведомлений
      console.log('7. Отправка уведомлений...');
      this.notifications.sendNotification(
        userId, 
        'order_confirmation', 
        `Ваш заказ ${orderId} подтвержден. Сумма: $${totalAmount}`
      );
      
      this.notifications.sendEmail(
        'customer@example.com',
        'Подтверждение заказа',
        `Ваш заказ ${orderId} успешно оформлен.`
      );
      
      console.log(`Заказ ${orderId} успешно создан`);
      return order;
      
    } catch (error) {
      console.error('Ошибка создания заказа:', error.message);
      
      // Откат резервирования товаров
      try {
        for (const item of items) {
          this.inventory.releaseProduct(item.productId, item.quantity);
        }
      } catch (rollbackError) {
        console.error('Ошибка отката резервирования:', rollbackError.message);
      }
      
      throw error;
    }
  }
  
  // Отмена заказа
  cancelOrder(orderId) {
    console.log(`Отмена заказа ${orderId}`);
    
    const order = this.orders.get(orderId);
    if (!order) {
      throw new Error(`Заказ ${orderId} не найден`);
    }
    
    try {
      // 1. Возврат платежа
      console.log('1. Обработка возврата платежа...');
      this.payment.refundPayment(order.transactionId, order.totalAmount);
      
      // 2. Освобождение зарезервированных товаров
      console.log('2. Освобождение товаров...');
      for (const item of order.items) {
        this.inventory.releaseProduct(item.productId, item.quantity);
      }
      
      // 3. Отмена доставки
      console.log('3. Отмена доставки...');
      this.shipping.updateShipmentStatus(order.shipmentId, 'cancelled');
      
      // 4. Обновление статуса заказа
      order.status = 'cancelled';
      
      // 5. Отправка уведомлений
      console.log('4. Отправка уведомлений...');
      this.notifications.sendNotification(
        order.userId,
        'order_cancellation',
        `Ваш заказ ${orderId} отменен. Средства будут возвращены.`
      );
      
      console.log(`Заказ ${orderId} успешно отменен`);
      return true;
      
    } catch (error) {
      console.error('Ошибка отмены заказа:', error.message);
      throw error;
    }
  }
  
  // Получение информации о заказе
  getOrderInfo(orderId) {
    const order = this.orders.get(orderId);
    if (!order) {
      throw new Error(`Заказ ${orderId} не найден`);
    }
    
    const shipment = this.shipping.getShipmentInfo(order.shipmentId);
    
    return {
      ...order,
      shipment
    };
  }
  
  // Получение списка заказов пользователя
  getUserOrders(userId) {
    const userOrders = [];
    this.orders.forEach(order => {
      if (order.userId === userId) {
        userOrders.push(this.getOrderInfo(order.id));
      }
    });
    
    return userOrders;
  }
}

// Пример использования
const orderFacade = new OrderFacade();

// Данные для заказа
const orderItems = [
  { productId: 'laptop', quantity: 1 },
  { productId: 'mouse', quantity: 2 }
];

const shippingAddress = 'г. Москва, ул. Примерная, д. 1 (дальний регион)';
const paymentMethod = 'credit_card';
const cardData = {
  number: '4242424242424242',
  expiry: '12/25',
  cvv: '123'
};

try {
  // Создание заказа
  console.log('=== Создание заказа ===');
  const order = orderFacade.createOrder(
    'user123',
    orderItems,
    shippingAddress,
    paymentMethod,
    cardData
  );
  
  console.log('Созданный заказ:', order);
  
  // Получение информации о заказе
  console.log('\n=== Информация о заказе ===');
  const orderInfo = orderFacade.getOrderInfo(order.id);
  console.log('Информация о заказе:', orderInfo);
  
  // Получение заказов пользователя
  console.log('\n=== Заказы пользователя ===');
  const userOrders = orderFacade.getUserOrders('user123');
  console.log('Заказы пользователя:', userOrders);
  
  // Отмена заказа
  console.log('\n=== Отмена заказа ===');
  const cancellationResult = orderFacade.cancelOrder(order.id);
  console.log('Результат отмены:', cancellationResult);
  
} catch (error) {
  console.error('Ошибка:', error.message);
}

// Попытка создания заказа с недостающим товаром
try {
  console.log('\n=== Попытка заказа недостающего товара ===');
  const invalidOrder = orderFacade.createOrder(
    'user456',
    [{ productId: 'laptop', quantity: 100 }], // Слишком много ноутбуков
    shippingAddress,
    paymentMethod,
    cardData
  );
} catch (error) {
  console.error('Ошибка:', error.message);
}
```

## Упражнение 3: Компоновщик (Composite)

### Задача
Создайте систему управления файловой структурой с использованием паттерна компоновщик для работы с файлами и папками единообразно.

### Требования
1. Единый интерфейс для файлов и папок
2. Возможность рекурсивного обхода структуры
3. Подсчет размера и количества элементов
4. Поиск элементов по имени

### Решение
```javascript
// Компонент - базовый класс для файлов и папок
class FileSystemComponent {
  constructor(name) {
    this.name = name;
  }
  
  getName() {
    return this.name;
  }
  
  getSize() {
    throw new Error('Метод getSize должен быть реализован');
  }
  
  getType() {
    throw new Error('Метод getType должен быть реализован');
  }
  
  add(component) {
    throw new Error('Метод add не поддерживается для этого типа');
  }
  
  remove(component) {
    throw new Error('Метод remove не поддерживается для этого типа');
  }
  
  getChild(index) {
    throw new Error('Метод getChild не поддерживается для этого типа');
  }
  
  getChildren() {
    throw new Error('Метод getChildren не поддерживается для этого типа');
  }
  
  findByName(name) {
    return this.name === name ? this : null;
  }
  
  search(pattern) {
    const results = [];
    if (this.name.includes(pattern)) {
      results.push(this);
    }
    return results;
  }
  
  getPath() {
    return this.name;
  }
  
  print(indent = 0) {
    throw new Error('Метод print должен быть реализован');
  }
}

// Лист - файл
class File extends FileSystemComponent {
  constructor(name, size, extension = '') {
    super(name);
    this.size = size;
    this.extension = extension;
  }
  
  getSize() {
    return this.size;
  }
  
  getType() {
    return 'file';
  }
  
  getExtension() {
    return this.extension;
  }
  
  search(pattern) {
    const results = super.search(pattern);
    return results;
  }
  
  print(indent = 0) {
    const spaces = '  '.repeat(indent);
    console.log(`${spaces}- ${this.name} (${this.size} bytes)`);
  }
}

// Композит - папка
class Folder extends FileSystemComponent {
  constructor(name) {
    super(name);
    this.children = [];
  }
  
  getSize() {
    return this.children.reduce((total, child) => total + child.getSize(), 0);
  }
  
  getType() {
    return 'folder';
  }
  
  add(component) {
    this.children.push(component);
  }
  
  remove(component) {
    const index = this.children.indexOf(component);
    if (index !== -1) {
      this.children.splice(index, 1);
    }
  }
  
  getChild(index) {
    return this.children[index];
  }
  
  getChildren() {
    return [...this.children];
  }
  
  getChildCount() {
    return this.children.length;
  }
  
  findByName(name) {
    // Проверяем себя
    if (super.findByName(name)) {
      return this;
    }
    
    // Проверяем детей
    for (const child of this.children) {
      const found = child.findByName(name);
      if (found) {
        return found;
      }
    }
    
    return null;
  }
  
  search(pattern) {
    let results = super.search(pattern);
    
    // Ищем в детях
    for (const child of this.children) {
      results = results.concat(child.search(pattern));
    }
    
    return results;
  }
  
  getPath() {
    return this.name;
  }
  
  getFullPath() {
    return '/' + this.getChildrenPaths().join('/');
  }
  
  getChildrenPaths() {
    return this.children.map(child => child.getPath());
  }
  
  print(indent = 0) {
    const spaces = '  '.repeat(indent);
    console.log(`${spaces}+ ${this.name} (${this.getChildCount()} items, ${this.getSize()} bytes)`);
    
    for (const child of this.children) {
      child.print(indent + 1);
    }
  }
  
  // Дополнительные методы для работы с файловой системой
  getFiles() {
    const files = [];
    for (const child of this.children) {
      if (child.getType() === 'file') {
        files.push(child);
      } else if (child.getType() === 'folder') {
        files.push(...child.getFiles());
      }
    }
    return files;
  }
  
  getFolders() {
    const folders = [];
    for (const child of this.children) {
      if (child.getType() === 'folder') {
        folders.push(child);
        folders.push(...child.getFolders());
      }
    }
    return folders;
  }
  
  getFilesByExtension(extension) {
    return this.getFiles().filter(file => file.getExtension() === extension);
  }
  
  getLargestFiles(count = 5) {
    return this.getFiles()
      .sort((a, b) => b.getSize() - a.getSize())
      .slice(0, count);
  }
}

// Расширенная папка с дополнительными возможностями
class AdvancedFolder extends Folder {
  constructor(name) {
    super(name);
    this.permissions = 'rw-r--r--';
    this.owner = 'user';
    this.createdAt = new Date();
  }
  
  setPermissions(permissions) {
    this.permissions = permissions;
  }
  
  getPermissions() {
    return this.permissions;
  }
  
  setOwner(owner) {
    this.owner = owner;
  }
  
  getOwner() {
    return this.owner;
  }
  
  getMetadata() {
    return {
      name: this.name,
      type: this.getType(),
      size: this.getSize(),
      items: this.getChildCount(),
      permissions: this.permissions,
      owner: this.owner,
      createdAt: this.createdAt,
      modifiedAt: new Date()
    };
  }
  
  print(indent = 0) {
    const spaces = '  '.repeat(indent);
    const metadata = this.getMetadata();
    console.log(`${spaces}+ ${this.name} [${metadata.permissions}] (${metadata.items} items, ${metadata.size} bytes)`);
    
    for (const child of this.children) {
      child.print(indent + 1);
    }
  }
}

// Фабрика для создания файловых компонентов
class FileSystemFactory {
  static createFile(name, size, extension = '') {
    return new File(name, size, extension);
  }
  
  static createFolder(name) {
    return new Folder(name);
  }
  
  static createAdvancedFolder(name) {
    return new AdvancedFolder(name);
  }
  
  static createProjectStructure() {
    // Создаем структуру проекта
    const root = this.createAdvancedFolder('my-project');
    
    // Папка src
    const src = this.createFolder('src');
    const components = this.createFolder('components');
    const utils = this.createFolder('utils');
    
    // Файлы в src
    src.add(this.createFile('index.js', 1024, 'js'));
    src.add(this.createFile('app.js', 2048, 'js'));
    
    // Компоненты
    components.add(this.createFile('Header.js', 512, 'js'));
    components.add(this.createFile('Footer.js', 384, 'js'));
    components.add(this.createFile('Button.js', 768, 'js'));
    
    // Утилиты
    utils.add(this.createFile('helpers.js', 640, 'js'));
    utils.add(this.createFile('api.js', 896, 'js'));
    
    src.add(components);
    src.add(utils);
    
    // Папка assets
    const assets = this.createFolder('assets');
    const images = this.createFolder('images');
    const styles = this.createFolder('styles');
    
    images.add(this.createFile('logo.png', 4096, 'png'));
    images.add(this.createFile('background.jpg', 10240, 'jpg'));
    images.add(this.createFile('icon.svg', 256, 'svg'));
    
    styles.add(this.createFile('main.css', 2048, 'css'));
    styles.add(this.createFile('responsive.css', 1536, 'css'));
    
    assets.add(images);
    assets.add(styles);
    
    // Файлы в корне
    root.add(this.createFile('package.json', 512, 'json'));
    root.add(this.createFile('README.md', 1024, 'md'));
    root.add(this.createFile('.gitignore', 256, ''));
    
    // Добавляем папки в корень
    root.add(src);
    root.add(assets);
    
    return root;
  }
}

// Пример использования
console.log('=== Создание файловой структуры ===');
const project = FileSystemFactory.createProjectStructure();

// Вывод структуры
console.log('Структура проекта:');
project.print();

// Получение информации
console.log('\n=== Информация о проекте ===');
console.log(`Общий размер: ${project.getSize()} bytes`);
console.log(`Количество элементов: ${project.getChildCount()}`);

// Поиск элементов
console.log('\n=== Поиск элементов ===');
const foundComponent = project.findByName('Button.js');
if (foundComponent) {
  console.log(`Найден элемент: ${foundComponent.getName()} (${foundComponent.getSize()} bytes)`);
}

const searchResults = project.search('js');
console.log(`Найдено ${searchResults.length} элементов с 'js' в имени:`);
searchResults.forEach(item => console.log(`  - ${item.getName()}`));

// Работа с файлами
console.log('\n=== Работа с файлами ===');
const allFiles = project.getFiles();
console.log(`Всего файлов: ${allFiles.length}`);

const jsFiles = project.getFilesByExtension('js');
console.log(`JavaScript файлов: ${jsFiles.length}`);

const largestFiles = project.getLargestFiles(3);
console.log('Самые большие файлы:');
largestFiles.forEach(file => {
  console.log(`  - ${file.getName()}: ${file.getSize()} bytes`);
});

// Работа с папками
console.log('\n=== Работа с папками ===');
const allFolders = project.getFolders();
console.log(`Всего папок: ${allFolders.length}`);

allFolders.forEach(folder => {
  console.log(`  - ${folder.getName()}: ${folder.getChildCount()} items, ${folder.getSize()} bytes`);
});

// Добавление новых элементов
console.log('\n=== Добавление новых элементов ===');
const newFolder = FileSystemFactory.createFolder('tests');
newFolder.add(FileSystemFactory.createFile('test.js', 128, 'js'));
newFolder.add(FileSystemFactory.createFile('integration.test.js', 256, 'js'));

project.add(newFolder);

console.log('Структура после добавления tests:');
project.print();

// Получение метаданных расширенной папки
console.log('\n=== Метаданные корневой папки ===');
if (project.getMetadata) {
  console.log(project.getMetadata());
}

// Удаление элементов
console.log('\n=== Удаление элементов ===');
const readmeFile = project.findByName('README.md');
if (readmeFile) {
  // Найдем родительскую папку README.md
  const removeFile = (folder, fileName) => {
    const children = folder.getChildren();
    for (let i = 0; i < children.length; i++) {
      if (children[i].getName() === fileName) {
        folder.remove(children[i]);
        return true;
      } else if (children[i].getType() === 'folder') {
        if (removeFile(children[i], fileName)) {
          return true;
        }
      }
    }
    return false;
  };
  
  removeFile(project, 'README.md');
  console.log('Файл README.md удален');
}

console.log('Структура после удаления README.md:');
project.print();
```

## Упражнение 4: Заместитель (Proxy)

### Задача
Создайте систему контроля доступа к ресурсам с использованием паттерна заместитель для управления правами доступа и кэширования.

### Требования
1. Заместитель для контроля доступа к ресурсам
2. Заместитель для кэширования ресурсов
3. Заместитель для ленивой инициализации
4. Система пользователей с различными правами

### Решение
```javascript
// Интерфейс ресурса
class Resource {
  getContent() {
    throw new Error('Метод getContent должен быть реализован');
  }
  
  updateContent(newContent) {
    throw new Error('Метод updateContent должен быть реализован');
  }
  
  getMetadata() {
    throw new Error('Метод getMetadata должен быть реализован');
  }
}

// Реальный ресурс
class RealResource extends Resource {
  constructor(name, content = '') {
    super();
    this.name = name;
    this.content = content;
    this.createdAt = new Date();
    this.lastModified = new Date();
    this.accessCount = 0;
    console.log(`Создан реальный ресурс: ${this.name}`);
  }
  
  getContent() {
    this.accessCount++;
    this.lastAccessed = new Date();
    console.log(`Доступ к содержимому ресурса: ${this.name}`);
    return this.content;
  }
  
  updateContent(newContent) {
    this.content = newContent;
    this.lastModified = new Date();
    this.accessCount++;
    console.log(`Обновлено содержимое ресурса: ${this.name}`);
  }
  
  getMetadata() {
    return {
      name: this.name,
      size: this.content.length,
      createdAt: this.createdAt,
      lastModified: this.lastModified,
      lastAccessed: this.lastAccessed,
      accessCount: this.accessCount
    };
  }
}

// Пользователь системы
class User {
  constructor(id, name, role) {
    this.id = id;
    this.name = name;
    this.role = role; // 'admin', 'editor', 'viewer'
  }
  
  getId() {
    return this.id;
  }
  
  getName() {
    return this.name;
  }
  
  getRole() {
    return this.role;
  }
  
  hasPermission(requiredRole) {
    const roles = ['viewer', 'editor', 'admin'];
    const userRoleIndex = roles.indexOf(this.role);
    const requiredRoleIndex = roles.indexOf(requiredRole);
    
    return userRoleIndex >= requiredRoleIndex;
  }
}

// Система аутентификации
class AuthenticationService {
  constructor() {
    this.users = new Map([
      [1, new User(1, 'Alice', 'admin')],
      [2, new User(2, 'Bob', 'editor')],
      [3, new User(3, 'Charlie', 'viewer')]
    ]);
    this.currentUser = null;
  }
  
  login(userId) {
    const user = this.users.get(userId);
    if (user) {
      this.currentUser = user;
      console.log(`Пользователь ${user.getName()} вошел в систему`);
      return true;
    }
    return false;
  }
  
  logout() {
    if (this.currentUser) {
      console.log(`Пользователь ${this.currentUser.getName()} вышел из системы`);
      this.currentUser = null;
    }
  }
  
  getCurrentUser() {
    return this.currentUser;
  }
  
  isAuthenticated() {
    return this.currentUser !== null;
  }
}

// Заместитель контроля доступа
class AccessProxy extends Resource {
  constructor(realResource, authService) {
    super();
    this.realResource = realResource;
    this.authService = authService;
  }
  
  getContent() {
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('viewer')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на чтение`);
    }
    
    console.log(`Пользователь ${user.getName()} (${user.getRole()}) получил доступ к ресурсу`);
    return this.realResource.getContent();
  }
  
  updateContent(newContent) {
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('editor')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на запись`);
    }
    
    console.log(`Пользователь ${user.getName()} (${user.getRole()}) обновил ресурс`);
    return this.realResource.updateContent(newContent);
  }
  
  getMetadata() {
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('viewer')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на просмотр метаданных`);
    }
    
    return this.realResource.getMetadata();
  }
}

// Заместитель кэширования
class CacheProxy extends Resource {
  constructor(realResource, cacheExpiry = 5000) { // 5 секунд по умолчанию
    super();
    this.realResource = realResource;
    this.cache = null;
    this.cacheExpiry = cacheExpiry;
    this.cacheTime = 0;
  }
  
  getContent() {
    const now = Date.now();
    
    // Проверяем, есть ли действительный кэш
    if (this.cache && (now - this.cacheTime) < this.cacheExpiry) {
      console.log('Получено содержимое из кэша');
      return this.cache;
    }
    
    // Получаем свежие данные и кэшируем их
    console.log('Кэш устарел, получение свежих данных');
    this.cache = this.realResource.getContent();
    this.cacheTime = now;
    
    return this.cache;
  }
  
  updateContent(newContent) {
    // Обновляем реальный ресурс
    const result = this.realResource.updateContent(newContent);
    
    // Инвалидируем кэш
    this.cache = null;
    this.cacheTime = 0;
    
    return result;
  }
  
  getMetadata() {
    return this.realResource.getMetadata();
  }
  
  // Метод для очистки кэша
  clearCache() {
    this.cache = null;
    this.cacheTime = 0;
    console.log('Кэш очищен');
  }
  
  // Метод для получения информации о кэше
  getCacheInfo() {
    const now = Date.now();
    const age = this.cache ? now - this.cacheTime : 0;
    const isValid = this.cache && age < this.cacheExpiry;
    
    return {
      hasCache: !!this.cache,
      age: age,
      isValid: isValid,
      expiry: this.cacheExpiry
    };
  }
}

// Заместитель ленивой инициализации
class LazyProxy extends Resource {
  constructor(resourceFactory) {
    super();
    this.resourceFactory = resourceFactory;
    this.realResource = null;
    this.initialized = false;
  }
  
  initialize() {
    if (!this.initialized) {
      console.log('Ленивая инициализация ресурса...');
      this.realResource = this.resourceFactory();
      this.initialized = true;
    }
  }
  
  getContent() {
    this.initialize();
    return this.realResource.getContent();
  }
  
  updateContent(newContent) {
    this.initialize();
    return this.realResource.updateContent(newContent);
  }
  
  getMetadata() {
    this.initialize();
    return this.realResource.getMetadata();
  }
  
  isInitialized() {
    return this.initialized;
  }
}

// Комбинированный заместитель
class CombinedProxy extends Resource {
  constructor(realResource, authService, cacheExpiry = 5000) {
    super();
    this.realResource = realResource;
    this.authService = authService;
    this.cache = null;
    this.cacheTime = 0;
    this.cacheExpiry = cacheExpiry;
  }
  
  getContent() {
    // Проверка доступа
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('viewer')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на чтение`);
    }
    
    const now = Date.now();
    
    // Проверка кэша
    if (this.cache && (now - this.cacheTime) < this.cacheExpiry) {
      console.log(`Пользователь ${user.getName()} получил содержимое из кэша`);
      return this.cache;
    }
    
    // Получение данных с реального ресурса
    console.log(`Пользователь ${user.getName()} получил свежие данные`);
    this.cache = this.realResource.getContent();
    this.cacheTime = now;
    
    return this.cache;
  }
  
  updateContent(newContent) {
    // Проверка доступа
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('editor')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на запись`);
    }
    
    // Обновление реального ресурса
    const result = this.realResource.updateContent(newContent);
    
    // Инвалидация кэша
    this.cache = null;
    this.cacheTime = 0;
    
    console.log(`Пользователь ${user.getName()} обновил ресурс`);
    return result;
  }
  
  getMetadata() {
    const user = this.authService.getCurrentUser();
    if (!user) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!user.hasPermission('viewer')) {
      throw new Error(`У пользователя ${user.getName()} нет прав на просмотр метаданных`);
    }
    
    return this.realResource.getMetadata();
  }
}

// Фабрика заместителей
class ProxyFactory {
  static createAccessProxy(realResource, authService) {
    return new AccessProxy(realResource, authService);
  }
  
  static createCacheProxy(realResource, cacheExpiry) {
    return new CacheProxy(realResource, cacheExpiry);
  }
  
  static createLazyProxy(resourceFactory) {
    return new LazyProxy(resourceFactory);
  }
  
  static createCombinedProxy(realResource, authService, cacheExpiry) {
    return new CombinedProxy(realResource, authService, cacheExpiry);
  }
}

// Пример использования
const authService = new AuthenticationService();

// Создаем реальный ресурс
const realResource = new RealResource('document.txt', 'Содержимое документа');

console.log('=== Демонстрация контроля доступа ===');

// Создаем заместитель контроля доступа
const accessProxy = ProxyFactory.createAccessProxy(realResource, authService);

// Попытка доступа без аутентификации
try {
  accessProxy.getContent();
} catch (error) {
  console.error('Ошибка:', error.message);
}

// Аутентификация как viewer
authService.login(3); // Charlie - viewer

try {
  const content = accessProxy.getContent();
  console.log('Получено содержимое:', content);
} catch (error) {
  console.error('Ошибка:', error.message);
}

// Попытка обновления как viewer
try {
  accessProxy.updateContent('Новое содержимое');
} catch (error) {
  console.error('Ошибка:', error.message);
}

// Аутентификация как editor
authService.login(2); // Bob - editor

try {
  accessProxy.updateContent('Обновленное содержимое документа');
  const updatedContent = accessProxy.getContent();
  console.log('Обновленное содержимое:', updatedContent);
} catch (error) {
  console.error('Ошибка:', error.message);
}

authService.logout();

console.log('\n=== Демонстрация кэширования ===');

// Создаем заместитель кэширования
const cacheProxy = ProxyFactory.createCacheProxy(realResource, 3000); // 3 секунды

// Первый доступ
console.log('Первый доступ к содержимому:');
const content1 = cacheProxy.getContent();

// Второй доступ (из кэша)
console.log('Второй доступ к содержимому:');
const content2 = cacheProxy.getContent();

// Ждем истечения времени кэша
console.log('Ожидание истечения времени кэша...');
setTimeout(() => {
  console.log('Третий доступ после истечения времени кэша:');
  const content3 = cacheProxy.getContent();
  
  // Обновление содержимого (инвалидация кэша)
  console.log('Обновление содержимого:');
  cacheProxy.updateContent('Содержимое после обновления');
  
  // Доступ после инвалидации кэша
  console.log('Доступ после инвалидации кэша:');
  const content4 = cacheProxy.getContent();
  
  // Информация о кэше
  console.log('Информация о кэше:', cacheProxy.getCacheInfo());
  
}, 3500);

console.log('\n=== Демонстрация ленивой инициализации ===');

// Создаем заместитель ленивой инициализации
const lazyProxy = ProxyFactory.createLazyProxy(() => {
  console.log('Создание реального ресурса в ленивой инициализации');
  return new RealResource('lazy-document.txt', 'Лениво инициализированное содержимое');
});

console.log('Заместитель создан, реальный ресурс еще не инициализирован');
console.log('Инициализирован:', lazyProxy.isInitialized());

// Первый доступ (инициализация)
console.log('Первый доступ к содержимому:');
const lazyContent = lazyProxy.getContent();
console.log('Инициализирован:', lazyProxy.isInitialized());

console.log('\n=== Демонстрация комбинированного заместителя ===');

// Создаем комбинированный заместитель
const combinedProxy = ProxyFactory.createCombinedProxy(
  new RealResource('combined-document.txt', 'Комбинированное содержимое'),
  authService,
  2000 // 2 секунды кэша
);

// Аутентификация
authService.login(1); // Alice - admin

// Доступ к содержимому
console.log('Доступ к содержимому через комбинированный заместитель:');
const combinedContent1 = combinedProxy.getContent();
const combinedContent2 = combinedProxy.getContent(); // Из кэша

// Обновление содержимого
console.log('Обновление содержимого:');
combinedProxy.updateContent('Обновленное комбинированное содержимое');

// Доступ после обновления
console.log('Доступ после обновления:');
const combinedContent3 = combinedProxy.getContent();

authService.logout();

// Попытка доступа без аутентификации
try {
  combinedProxy.getContent();
} catch (error) {
  console.error('Ошибка:', error.message);
}