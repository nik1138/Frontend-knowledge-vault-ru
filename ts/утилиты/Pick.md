---
aliases: [Выбор полей, Отбор свойств]
tags: [typescript, utility-types, pick]
---

# Pick

Утилита `Pick` в TypeScript позволяет создать новый тип, выбирая только определенные свойства из существующего типа. Это полезно, когда вам нужно работать только с подмножеством полей объекта, например, при создании форм или передаче данных между слоями приложения.

## Определение

```typescript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

Здесь `K extends keyof T` ограничивает `K` только теми ключами, которые существуют в `T`.

## Примеры использования

### Формы создания сущностей

При создании сущностей часто требуется только часть полей, например, при создании пользователя не нужно указывать `id`:

```typescript
interface User {
    id: number;
    name: string;
    email: string;
    password: string;
    createdAt: Date;
    updatedAt: Date;
}

// Для создания пользователя выбираем только нужные поля
type UserForCreation = Pick<User, 'name' | 'email' | 'password'>;

function createUser(userData: UserForCreation): User {
    return {
        id: generateId(),
        ...userData,
        createdAt: new Date(),
        updatedAt: new Date()
    };
}

// Использование
const newUser = createUser({
    name: "Иван",
    email: "ivan@example.com",
    password: "securepassword"
}); // id, createdAt и updatedAt не требуются
```

### DTO (Data Transfer Objects)

При передаче данных между слоями приложения часто нужно ограничить набор передаваемых полей:

```typescript
interface FullProduct {
    id: number;
    name: string;
    description: string;
    price: number;
    category: string;
    createdAt: Date;
    updatedAt: Date;
    supplierId: number;
}

// Для публичного API передаем только основные поля
type PublicProduct = Pick<FullProduct, 'id' | 'name' | 'price' | 'category'>;

function getPublicProducts(): PublicProduct[] {
    // Получаем полные данные из базы
    const products = fetchFullProducts();
    // Возвращаем только публичные поля
    return products.map(p => ({
        id: p.id,
        name: p.name,
        price: p.price,
        category: p.category
    }));
}
```

### Обновление частичных данных

При обновлении данных можно создать тип, который позволяет обновлять только определенные поля:

```typescript
interface UserProfile {
    id: number;
    firstName: string;
    lastName: string;
    email: string;
    phone: string;
    address: string;
    preferences: UserPreferences;
}

// Для обновления контактной информации
type ContactInfoUpdate = Pick<UserProfile, 'email' | 'phone' | 'address'>;

function updateContactInfo(userId: number, contactInfo: ContactInfoUpdate) {
    // Обновляем только контактную информацию
    return updateUser(userId, contactInfo);
}
```

## Связанные концепции

- [[Omit]] - противоположная утилита, исключает определенные свойства
- [[Partial]] - делает все свойства необязательными
- [[Required]] - делает все свойства обязательными
- [[Readonly]] - делает свойства доступными только для чтения

## Практические советы

- Используйте `Pick` для создания специализированных типов, соответствующих конкретным сценариям использования.
- `Pick` особенно полезен при работе с API, где нужно ограничить объем передаваемых данных.
- Комбинируйте `Pick` с другими утилитами для создания более сложных типов.

## Заключение

`Pick` - мощная утилита для создания типов с ограниченным набором свойств. Она позволяет создавать более точные и безопасные типы, соответствующие конкретным сценариям использования, и улучшает читаемость кода.