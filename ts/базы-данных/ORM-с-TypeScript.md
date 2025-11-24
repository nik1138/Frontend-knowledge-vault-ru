---
aliases: [ORM в TypeScript, Объектно-реляционное отображение, ORM TypeScript]
tags: [typescript, database, orm, typeorm, sequelize, mongoose]
---

# ORM с TypeScript

## Введение

ORM (Object-Relational Mapping) - это технология программирования, которая позволяет работать с реляционными базами данных с помощью объектно-ориентированного подхода. При использовании TypeScript ORM обеспечивают дополнительные преимущества благодаря строгой типизации, что делает разработку более безопасной и предсказуемой.

## Популярные ORM для TypeScript

### 1. TypeORM

TypeORM - это мощный ORM для TypeScript и JavaScript, поддерживающий ES7+ декораторы и наследование. Он работает с MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, WebSQL и другими базами данных.

#### Установка

```bash
npm install typeorm reflect-metadata mysql2 # или другая база данных
npm install @types/node --save-dev
```

#### Базовая настройка

```typescript
import { createConnection } from 'typeorm';
import { User } from './entity/User';

createConnection({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'test',
  password: 'test',
  database: 'test',
  entities: [User],
  synchronize: true,
  logging: false,
}).then(async connection => {
  console.log('Connected to database');
}).catch(error => console.log(error));
```

#### Определение сущности

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ nullable: true })
  age?: number;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

#### Работа с сущностями

```typescript
import { getRepository } from 'typeorm';
import { User } from './entity/User';

// Создание нового пользователя
const user = new User();
user.email = 'john@example.com';
user.name = 'John Doe';
user.age = 30;

const userRepository = getRepository(User);
await userRepository.save(user);

// Поиск пользователей
const users = await userRepository.find({
  where: {
    age: 30
  },
  order: {
    createdAt: 'DESC'
  }
});

// Поиск по ID
const user = await userRepository.findOne(1);
```

### 2. Sequelize

Sequelize - это promise-based ORM для Node.js, который поддерживает PostgreSQL, MySQL, MariaDB, SQLite и MSSQL. Он полностью поддерживает TypeScript через определения типов.

#### Установка

```bash
npm install sequelize
npm install pg pg-hstore # для PostgreSQL
npm install mysql2 # для MySQL
npm install sqlite3 # для SQLite
npm install tedious # для MSSQL
npm install @types/sequelize --save-dev
```

#### Базовая настройка

```typescript
import { Sequelize } from 'sequelize';

const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql' /* one of 'mysql' | 'mariadb' | 'postgres' | 'mssql' */
});
```

#### Определение модели

```typescript
import { Model, DataTypes } from 'sequelize';

interface UserAttributes {
  id: number;
  email: string;
  name: string;
  age?: number;
  createdAt?: Date;
  updatedAt?: Date;
}

interface UserCreationAttributes extends Optional<UserAttributes, 'id' | 'createdAt' | 'updatedAt'> {}

class User extends Model<UserAttributes, UserCreationAttributes> implements UserAttributes {
  public id!: number;
  public email!: string;
  public name!: string;
  public age?: number;
  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;

  // Методы ассоциаций будут определены здесь
}

User.init({
  id: {
    type: DataTypes.INTEGER.UNSIGNED,
    autoIncrement: true,
    primaryKey: true,
  },
  email: {
    type: DataTypes.STRING(128),
    allowNull: false,
    unique: true,
  },
  name: {
    type: DataTypes.STRING(128),
    allowNull: false,
  },
  age: {
    type: DataTypes.INTEGER,
    allowNull: true,
  },
  createdAt: {
    type: DataTypes.DATE,
    allowNull: false,
    defaultValue: DataTypes.NOW,
  },
  updatedAt: {
    type: DataTypes.DATE,
    allowNull: false,
    defaultValue: DataTypes.NOW,
  },
}, {
  tableName: 'users',
  sequelize, // передаем экземпляр подключения
});
```

#### Работа с моделями

```typescript
// Создание записи
const user = await User.create({
  email: 'john@example.com',
  name: 'John Doe',
  age: 30
});

// Поиск записей
const users = await User.findAll({
  where: {
    age: {
      [Op.gt]: 18
    }
  },
  order: [
    ['createdAt', 'DESC']
  ]
});

// Обновление записи
await user.update({
  name: 'John Smith'
});

// Удаление записи
await user.destroy();
```

### 3. Prisma

Prisma - это современный ORM для TypeScript и Node.js, который предоставляет типобезопасный доступ к базе данных с помощью генерации типов на основе схемы.

#### Установка

```bash
npm install prisma @prisma/client
npm install -D prisma
npx prisma init
```

#### Определение схемы

```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String
  age       Int?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  userId    Int
  user      User     @relation(fields: [userId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

#### Использование Prisma

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Создание пользователя
const user = await prisma.user.create({
  data: {
    email: 'john@example.com',
    name: 'John Doe',
    age: 30,
  },
});

// Создание связанного поста
const post = await prisma.post.create({
  data: {
    title: 'My First Post',
    content: 'Hello World',
    user: {
      connect: { id: user.id },
    },
  },
});

// Получение пользователя с постами
const userWithPosts = await prisma.user.findUnique({
  where: { id: user.id },
  include: { posts: true },
});
```

## Сравнение ORM

| ORM | Преимущества | Недостатки |
|-----|-------------|------------|
| TypeORM | - Поддержка декораторов<br>- Хорошая интеграция с TypeScript<br>- Поддержка наследования | - Меньше документации<br>- Меньше сообщество |
| Sequelize | - Стабильный и зрелый<br>- Большое сообщество<br>- Хорошая документация | - Меньше типизации по умолчанию<br>- Более сложная настройка |
| Prisma | - Типобезопасность<br>- Автогенерация типов<br>- Современный API | - Новый инструмент<br>- Зависимость от генерации кода |

## Лучшие практики при использовании ORM в TypeScript

### 1. Используйте строгую типизацию

```typescript
// Плохо
const user: any = await User.findOne({ id: 1 });

// Хорошо
const user: User | null = await User.findOne({ id: 1 });
```

### 2. Определяйте интерфейсы для DTO

```typescript
interface CreateUserDTO {
  email: string;
  name: string;
  age?: number;
}

async function createUser(dto: CreateUserDTO): Promise<User> {
  // Реализация создания пользователя
}
```

### 3. Используйте транзакции для сложных операций

```typescript
import { getRepository, getConnection } from 'typeorm';
import { User } from './entity/User';

async function createUserWithProfile(userData: UserCreateInput, profileData: ProfileCreateInput) {
  const connection = getConnection();
  const queryRunner = connection.createQueryRunner();
  
  await queryRunner.connect();
  await queryRunner.startTransaction();
  
  try {
    const user = new User();
    user.email = userData.email;
    user.name = userData.name;
    
    const savedUser = await queryRunner.manager.save(user);
    
    // Создание профиля пользователя
    const profile = new Profile();
    profile.userId = savedUser.id;
    profile.bio = profileData.bio;
    
    await queryRunner.manager.save(profile);
    
    await queryRunner.commitTransaction();
    return savedUser;
  } catch (error) {
    await queryRunner.rollbackTransaction();
    throw error;
  } finally {
    await queryRunner.release();
  }
}
```

### 4. Используйте миграции

```typescript
import { MigrationInterface, QueryRunner, Table } from 'typeorm';

export class CreateUserTable1620000000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(new Table({
      name: 'users',
      columns: [
        {
          name: 'id',
          type: 'int',
          isPrimary: true,
          isGenerated: true,
          generationStrategy: 'increment',
        },
        {
          name: 'email',
          type: 'varchar',
          isUnique: true,
          isNullable: false,
        },
        {
          name: 'name',
          type: 'varchar',
          isNullable: false,
        },
        {
          name: 'age',
          type: 'int',
          isNullable: true,
        },
        {
          name: 'created_at',
          type: 'timestamp',
          default: 'now()',
        },
        {
          name: 'updated_at',
          type: 'timestamp',
          default: 'now()',
        },
      ],
    }));
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('users');
  }
}
```

## Заключение

ORM в TypeScript обеспечивают безопасность типов, улучшают производительность разработки и упрощают работу с базами данных. Выбор конкретного ORM зависит от требований проекта, предпочтений команды и специфики приложения.

Следующие темы, связанные с ORM:
- [[Типизация-моделей]]
- [[Миграции]]
- [[Запросы]]
- [[Транзакции]]

#typescript #database #orm #typeorm #sequelize #prisma #backend #development