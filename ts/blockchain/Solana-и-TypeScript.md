---
aliases: [Solana и TypeScript, Solana TypeScript]
tags: [blockchain, solana, typescript, smart-contracts, web3]
---

# Solana и TypeScript

## Обзор

Solana - высокопроизводительная блокчейн-платформа, поддерживающая создание децентрализованных приложений (dApps). TypeScript предоставляет строгую типизацию при разработке Solana приложений, что улучшает надежность и читаемость кода.

## Настройка проекта

Для работы с Solana и TypeScript установите необходимые зависимости:

```bash
npm init -y
npm install @solana/web3.js @solana/spl-token
npm install --save-dev typescript @types/node ts-node
```

## Основные концепции

### 1. Подключение к Solana

```typescript
import { Connection, clusterApiUrl, PublicKey } from '@solana/web3.js';

// Подключение к основной сети
const connection = new Connection(clusterApiUrl('mainnet-beta'), 'confirmed');

// Подключение к тестовой сети
const testnetConnection = new Connection(clusterApiUrl('devnet'), 'confirmed');
```

### 2. Работа с ключами

```typescript
import { Keypair } from '@solana/web3.js';

// Создание нового ключевого пары
const keypair = Keypair.generate();
console.log('Публичный ключ:', keypair.publicKey.toString());
console.log('Секретный ключ:', keypair.secretKey);

// Восстановление из секретного ключа
const secretKey = Uint8Array.from([/* ... */]);
const restoredKeypair = Keypair.fromSecretKey(secretKey);
```

### 3. Получение баланса

```typescript
async function getBalance(publicKey: PublicKey): Promise<number> {
  const balance = await connection.getBalance(publicKey);
  return balance / 1000000000; // Конвертация из lamports в SOL
}

// Использование
const publicKey = new PublicKey('...');
const balance = await getBalance(publicKey);
console.log(`Баланс: ${balance} SOL`);
```

## Работа с транзакциями

### Создание и отправка транзакции

```typescript
import {
  Transaction,
  SystemProgram,
  sendAndConfirmTransaction
} from '@solana/web3.js';

async function sendSOL(from: Keypair, to: PublicKey, amount: number) {
  const transaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: from.publicKey,
      toPubkey: to,
      lamports: amount * 1000000000 // Конвертация в lamports
    })
  );

  const signature = await sendAndConfirmTransaction(
    connection,
    transaction,
    [from]
  );

  console.log('Транзакция подписана:', signature);
}
```

## Работа с программами Solana

### Взаимодействие с SPL Token

```typescript
import {
  createMint,
  getOrCreateAssociatedTokenAccount,
  mintTo,
  transfer
} from '@solana/spl-token';

// Создание нового токена
async function createNewToken() {
  const mint = await createMint(
    connection,
    payer,
    mintAuthority,
    freezeAuthority,
    decimals
  );
  
  return mint;
}

// Перевод токенов
async function transferTokens(
  source: PublicKey,
  destination: PublicKey,
  owner: Keypair,
  amount: number
) {
  const signature = await transfer(
    connection,
    payer,
    source,
    destination,
    owner,
    amount
  );
  
  console.log('Перевод подписан:', signature);
}
```

## Примеры использования

### Проверка существования аккаунта

```typescript
async function accountExists(publicKey: PublicKey): Promise<boolean> {
  try {
    const accountInfo = await connection.getAccountInfo(publicKey);
    return accountInfo !== null;
  } catch (error) {
    return false;
  }
}
```

### Получение информации о транзакции

```typescript
async function getTransactionDetails(signature: string) {
  const transaction = await connection.getTransaction(signature);
  if (transaction) {
    console.log('Блок:', transaction.slot);
    console.log('Комиссия:', transaction.meta?.fee);
  }
}
```

## Продвинутые темы

### Работа с RPC

```typescript
// Пользовательские RPC вызовы
const response = await connection._rpcRequest('getRecentBlockhash', []);
```

### Использование Anchor фреймворка

```typescript
// Anchor позволяет создавать сложные программы с автоматической генерацией TypeScript SDK
import { Program, AnchorProvider } from '@project-serum/anchor';

// Подключение к программе
const provider = AnchorProvider.env();
const program = new Program(idl, provider);
```

## Лучшие практики

1. **Хранение ключей в безопасном месте**
2. **Проверка баланса перед транзакцией**
3. **Использование try-catch для обработки ошибок**
4. **Тестирование на тестовых сетях**
5. **Оптимизация количества RPC вызовов**

## Связанные темы

- [[Смарт-контракты]]
- [[Web3-библиотеки]]
- [[Кошельки]]
- [[Ethereum-и-TypeScript]]

## Внешние ресурсы

- [Документация Solana Web3.js](https://solana-labs.github.io/solana-web3.js/)
- [Solana Developer Documentation](https://docs.solana.com/developing)
- [Anchor Documentation](https://book.anchor-lang.com/)