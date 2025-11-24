---
aliases: [Исключение полей, Пропуск свойств]
tags: [typescript, utility-types, omit]
---

# Omit

Утилита `Omit` в TypeScript позволяет создать новый тип, исключая определенные свойства из существующего типа. Это противоположная операция по сравнению с [[Pick]], где выбираются только указанные свойства. `Omit` полезен, когда вы хотите работать с типом, но исключить из него определенные поля, например, чувствительные данные или поля, которые не должны быть изменены.

## Определение

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

Здесь `Exclude<keyof T, K>` исключает ключи `K` из общего набора ключей `T`, а затем `Pick` создает тип с оставшимися ключами.

## Примеры использования

### Исключение чувствительных данных

При передаче данных пользователю часто нужно исключить чувствительные поля, такие как пароли:

```typescript
interface User {
    id: number;
    name: string;
    email: string;
    password: string;
    role: string;
    createdAt: Date;
}

// Для публичного представления исключаем пароль
type PublicUser = Omit<User, 'password'>;

function getPublicUser(userId: number): PublicUser {
    const user = getUserById(userId);
    // Исключаем пароль из возвращаемого объекта
    const { password, ...publicUser } = user;
    return publicUser;
}

// Использование
const publicUserData: PublicUser = getPublicUser(1);
// publicUserData не содержит поля password
```

### Создание типов для обновления

При обновлении сущностей часто нужно исключить поля, которые не должны изменяться, например, `id`:

```typescript
interface Product {
    id: number;
    name: string;
    description: string;
    price: number;
    createdAt: Date;
    updatedAt: Date;
}

// Для обновления исключаем id и даты создания
type ProductUpdate = Omit<Product, 'id' | 'createdAt' | 'updatedAt'>;

function updateProduct(id: number, updates: ProductUpdate): Product {
    const product = getProductById(id);
    return {
        ...product,
        ...updates,
        updatedAt: new Date()
    };
}

// Использование
const updates: ProductUpdate = {
    name: "Новое название",
    price: 99.99
};
// id, createdAt и updatedAt не могут быть изменены через этот тип
const updatedProduct = updateProduct(1, updates);
```

### DTO для разных слоев приложения

При создании DTO (Data Transfer Objects) для разных слоев приложения можно исключить ненужные поля:

```typescript
interface FullOrder {
    id: number;
    userId: number;
    items: OrderItem[];
    total: number;
    status: OrderStatus;
    paymentInfo: PaymentInfo;
    shippingAddress: Address;
    billingAddress: Address;
    createdAt: Date;
    updatedAt: Date;
}

// Для внутреннего использования исключаем платежную информацию
type InternalOrder = Omit<FullOrder, 'paymentInfo'>;

// Для отчетов исключаем детали адресов и внутреннюю информацию
type OrderReport = Omit<FullOrder, 'paymentInfo' | 'shippingAddress' | 'billingAddress' | 'items'>;
```

## Связанные концепции

- [[Pick]] - противоположная утилита, выбирает определенные свойства
- [[Partial]] - делает все свойства необязательными
- [[Required]] - делает все свойства обязательными
- [[Readonly]] - делает свойства доступными только для чтения

## Практические советы

- Используйте `Omit` для создания типов, соответствующих конкретным сценариям использования, исключая ненужные или чувствительные поля.
- `Omit` особенно полезен при работе с API, где нужно ограничить объем передаваемых данных.
- Комбинируйте `Omit` с другими утилитами для создания более сложных типов.

## Заключение

`Omit` - важная утилита для создания типов с исключенными свойствами. Она позволяет создавать более безопасные и точные типы, соответствующие конкретным сценариям использования, и улучшает читаемость кода.