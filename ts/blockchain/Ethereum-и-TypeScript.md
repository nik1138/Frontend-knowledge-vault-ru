---
aliases: [Ethereum и TypeScript, Ethereum TypeScript]
tags: [blockchain, ethereum, typescript, smart-contracts, web3]
---

# Ethereum и TypeScript

## Обзор

Интеграция TypeScript с Ethereum позволяет разработчикам создавать надежные и типизированные приложения для блокчейна. TypeScript предоставляет статическую типизацию, что особенно важно при работе с финансовыми приложениями, где ошибки могут быть катастрофическими.

## Настройка проекта

Для начала работы с Ethereum и TypeScript необходимо настроить проект с необходимыми зависимостями:

```bash
npm init -y
npm install ethers @types/node
npm install --save-dev typescript @types/node ts-node
```

Создайте файл `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

## Основные концепции

### 1. Подключение к Ethereum

Для взаимодействия с Ethereum сети используется провайдер:

```typescript
import { ethers } from 'ethers';

// Подключение к публичному провайдеру
const provider = new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/YOUR_PROJECT_ID');

// Или к тестовой сети
const testnetProvider = new ethers.JsonRpcProvider('https://sepolia.infura.io/v3/YOUR_PROJECT_ID');
```

### 2. Работа с аккаунтами

```typescript
// Создание нового кошелька
const wallet = ethers.Wallet.createRandom();
console.log('Адрес кошелька:', wallet.address);
console.log('Приватный ключ:', wallet.privateKey);

// Подключение к существующему кошельку
const privateKey = '0x...';
const walletFromPrivateKey = new ethers.Wallet(privateKey, provider);
```

### 3. Взаимодействие со смарт-контрактами

```typescript
// ABI смарт-контракта
const contractABI = [
  // ABI определение
  "function balanceOf(address owner) view returns (uint256)",
  "function transfer(address to, uint256 amount)"
];

// Адрес контракта
const contractAddress = '0x...';

// Подключение к контракту
const contract = new ethers.Contract(contractAddress, contractABI, wallet);

// Вызов метода
const balance = await contract.balanceOf(wallet.address);
```

## Примеры использования

### Получение баланса

```typescript
async function getBalance(address: string): Promise<ethers.BigNumberish> {
  const balance = await provider.getBalance(address);
  return ethers.formatEther(balance);
}

// Использование
const balance = await getBalance('0x...');
console.log(`Баланс: ${balance} ETH`);
```

### Отправка транзакции

```typescript
async function sendTransaction(to: string, amount: string) {
  const tx = {
    to: to,
    value: ethers.parseEther(amount)
  };

  const transactionResponse = await wallet.sendTransaction(tx);
  const receipt = await transactionResponse.wait();
  
  console.log('Транзакция подтверждена в блоке:', receipt?.blockNumber);
}
```

## Продвинутые темы

### Использование Ethers.js v6

Ethers.js v6 предоставляет улучшенный API с лучшей типизацией:

```typescript
import { Contract, providers, Wallet } from 'ethers';

// Улучшенная типизация
interface IERC20 {
  balanceOf: (address: string) => Promise<bigint>;
  transfer: (to: string, amount: bigint) => Promise<ContractTransaction>;
}

// Использование типов для контрактов
const erc20Contract = new Contract(address, abi) as unknown as IERC20;
```

### Работа с событиями

```typescript
// Подписка на события контракта
contract.on('Transfer', (from, to, amount) => {
  console.log(`${from} отправил ${amount} токенов ${to}`);
});
```

## Лучшие практики

1. **Проверка баланса перед транзакцией**
2. **Использование try-catch для обработки ошибок**
3. **Валидация входных данных**
4. **Тестирование на тестовых сетях**

## Связанные темы

- [[Смарт-контракты]]
- [[Web3-библиотеки]]
- [[Кошельки]]
- [[Solana-и-TypeScript]]

## Внешние ресурсы

- [Документация Ethers.js](https://docs.ethers.org/)
- [Ethereum Developer Documentation](https://ethereum.org/en/developers/)