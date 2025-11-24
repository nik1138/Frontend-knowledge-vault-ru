---
aliases: [Только для чтения, Неизменяемый]
tags: [typescript, utility-types, readonly]
---

# Readonly

Утилита `Readonly` в TypeScript используется для создания нового типа, в котором все свойства исходного типа становятся доступными только для чтения. Это предотвращает изменение значений свойств после создания объекта, что способствует созданию неизменяемых (immutable) структур данных.

## Определение

```typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

Здесь `readonly` означает, что каждое свойство становится доступным только для чтения.

## Примеры использования

### Создание конфигурации

При создании конфигурационных объектов часто полезно сделать их неизменяемыми, чтобы избежать случайного изменения настроек:

```typescript
interface AppConfig {
    apiUrl: string;
    timeout: number;
    retries: number;
}

const config: Readonly<AppConfig> = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    retries: 3
};

// config.apiUrl = "https://new-api.example.com"; // Ошибка: нельзя изменить readonly свойство
```

### Неизменяемые данные

При работе с данными, которые не должны изменяться после создания, `Readonly` помогает предотвратить непреднамеренные изменения:

```typescript
interface User {
    id: number;
    name: string;
    email: string;
}

function processUserData(userData: Readonly<User>) {
    // Данные не могут быть изменены внутри этой функции
    console.log(`Обработка данных пользователя: ${userData.name}`);
    // userData.name = "Новое имя"; // Ошибка: нельзя изменить readonly свойство
}

const user: User = { id: 1, name: "Иван", email: "ivan@example.com" };
processUserData(user);
```

### Глубокая неизменяемость

Для глубокой неизменяемости (вложенные объекты также становятся readonly) можно создать кастомную утилиту:

```typescript
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

interface NestedConfig {
    server: {
        host: string;
        port: number;
    };
    database: {
        url: string;
        name: string;
    };
}

const deepConfig: DeepReadonly<NestedConfig> = {
    server: {
        host: "localhost",
        port: 3000
    },
    database: {
        url: "mongodb://localhost:27017",
        name: "mydb"
    }
};

// deepConfig.server.host = "newhost"; // Ошибка: все вложенные свойства тоже readonly
```

## Связанные концепции

- [[Partial]] - делает все свойства необязательными
- [[Required]] - делает все свойства обязательными
- [[Pick]] - позволяет выбрать только определенные свойства
- [[Omit]] - позволяет исключить определенные свойства

## Практические советы

- Используйте `Readonly` для создания неизменяемых структур данных, особенно в функциях, которые не должны изменять входные параметры.
- Будьте осторожны с `Readonly` при работе с массивами - `Readonly<Array<T>>` делает методы изменения массива недоступными.
- Для массивов используйте `ReadonlyArray<T>` для более точного контроля.

## Заключение

`Readonly` - важная утилита для создания неизменяемых типов, что способствует более надежному и предсказуемому коду. Она особенно полезна при работе с конфигурациями и данными, которые не должны изменяться после создания.