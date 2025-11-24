---
aliases: [Web3-библиотеки, Web3 Libraries]
tags: [blockchain, web3, libraries, typescript, ethereum, solana, api]
---

# Web3-библиотеки

## Обзор

Web3-библиотеки обеспечивают интерфейс между приложениями и блокчейн-сетями. Они позволяют разработчикам взаимодействовать с блокчейн-сетями, отправлять транзакции, читать данные из блокчейна и взаимодействовать со смарт-контрактами.

## Основные Web3-библиотеки

### 1. Ethers.js (для Ethereum)

Ethers.js - это полная реализация кошелька Ethereum и утилит для работы с Ethereum.

#### Установка
```bash
npm install ethers
```

#### Основные возможности
- Подключение к Ethereum сетям
- Управление аккаунтами и транзакциями
- Взаимодействие со смарт-контрактами
- Поддержка ENS

#### Пример использования
```typescript
import { ethers } from 'ethers';

// Подключение к провайдеру
const provider = new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/YOUR_PROJECT_ID');

// Подключение к контракту
const contractABI = [
  "function balanceOf(address owner) view returns (uint256)",
  "function transfer(address to, uint256 amount)"
];

const contractAddress = '0x...';
const contract = new ethers.Contract(contractAddress, contractABI, provider);

// Вызов метода
async function getBalance(address: string) {
  const balance = await contract.balanceOf(address);
  return ethers.formatEther(balance);
}
```

### 2. Web3.js (для Ethereum)

Web3.js - это JavaScript библиотека для взаимодействия с Ethereum узлами.

#### Установка
```bash
npm install web3
```

#### Пример использования
```typescript
import Web3 from 'web3';

const web3 = new Web3('https://mainnet.infura.io/v3/YOUR_PROJECT_ID');

// Получение баланса
async function getBalance(address: string) {
  const balance = await web3.eth.getBalance(address);
  return web3.utils.fromWei(balance, 'ether');
}

// Взаимодействие с контрактом
const contract = new web3.eth.Contract(abi, contractAddress);
```

### 3. Solana Web3.js

Библиотека для взаимодействия с Solana сетью.

#### Установка
```bash
npm install @solana/web3.js
```

#### Пример использования
```typescript
import { Connection, PublicKey } from '@solana/web3.js';

const connection = new Connection('https://api.mainnet-beta.solana.com');

async function getBalance(publicKey: PublicKey) {
  const balance = await connection.getBalance(publicKey);
  return balance / 1000000000; // Конвертация из lamports в SOL
}
```

## Современные Web3-библиотеки

### 1. Viem (для Ethereum)

Viem - это современная, легковесная библиотека для взаимодействия с Ethereum.

#### Установка
```bash
npm install viem
```

#### Пример использования
```typescript
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';

const client = createPublicClient({
  chain: mainnet,
  transport: http('https://mainnet.infura.io/v3/YOUR_PROJECT_ID')
});

// Чтение данных из блокчейна
const blockNumber = await client.getBlockNumber();
```

### 2. Solana Web3.js с Anchor

Для работы с Solana программами (смарт-контрактами) используется комбинация Solana Web3.js и Anchor.

```typescript
import { Program, AnchorProvider } from '@project-serum/anchor';
import { Connection, PublicKey, clusterApiUrl } from '@solana/web3.js';

const connection = new Connection(clusterApiUrl('mainnet-beta'));
const provider = new AnchorProvider(connection, wallet, {});

const program = new Program(idl, provider);
```

## Сравнение библиотек

| Библиотека | Поддержка сети | Типизация | Размер | Особенности |
|------------|----------------|-----------|---------|-------------|
| Ethers.js | Ethereum | Отличная | Средний | Полная функциональность, активная поддержка |
| Web3.js | Ethereum | Хорошая | Большой | Долгая история, много возможностей |
| Solana Web3.js | Solana | Отличная | Средний | Оптимизирована для Solana |
| Viem | Ethereum | Отличная | Маленький | Современный подход, легковесная |

## Интеграция с фреймворками

### React с Web3
```typescript
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

function useWeb3() {
  const [account, setAccount] = useState<string | null>(null);
  const [balance, setBalance] = useState<string | null>(null);
  
  useEffect(() => {
    const connectWallet = async () => {
      if (typeof window.ethereum !== 'undefined') {
        const provider = new ethers.BrowserProvider(window.ethereum);
        const accounts = await provider.send('eth_requestAccounts', []);
        setAccount(accounts[0]);
        
        const balance = await provider.getBalance(accounts[0]);
        setBalance(ethers.formatEther(balance));
      }
    };
    
    connectWallet();
  }, []);
  
  return { account, balance };
}
```

### Next.js с Web3
```typescript
// pages/api/web3.ts
import { ethers } from 'ethers';
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
  const blockNumber = await provider.getBlockNumber();
  
  res.status(200).json({ blockNumber });
}
```

## Лучшие практики использования

### 1. Управление провайдерами
```typescript
// Создание провайдера с обработкой ошибок
async function createProvider() {
  try {
    if (typeof window.ethereum !== 'undefined') {
      // MetaMask или другой инжектированный провайдер
      const provider = new ethers.BrowserProvider(window.ethereum);
      await provider.send('eth_requestAccounts', []);
      return provider;
    } else {
      // Публичный провайдер
      return new ethers.JsonRpcProvider('https://mainnet.infura.io/v3/YOUR_PROJECT_ID');
    }
  } catch (error) {
    console.error('Ошибка подключения к провайдеру:', error);
    throw error;
  }
}
```

### 2. Обработка транзакций
```typescript
async function sendTransaction(contract: ethers.Contract, method: string, params: any[]) {
  try {
    const tx = await contract[method](...params);
    const receipt = await tx.wait();
    
    if (receipt?.status === 1) {
      console.log('Транзакция успешна:', receipt.hash);
      return receipt;
    } else {
      throw new Error('Транзакция не удалась');
    }
  } catch (error) {
    console.error('Ошибка транзакции:', error);
    throw error;
  }
}
```

### 3. Кэширование данных
```typescript
// Простое кэширование для избегания повторных вызовов
const cache = new Map();

async function getCachedData(key: string, fetcher: () => Promise<any>, ttl: number = 60000) {
  const cached = cache.get(key);
  
  if (cached && Date.now() - cached.timestamp < ttl) {
    return cached.data;
  }
  
  const data = await fetcher();
  cache.set(key, { data, timestamp: Date.now() });
  
  return data;
}
```

## Инструменты и утилиты

### 1. Wagmi
Современная библиотека для интеграции Web3 в React приложения.

```bash
npm install wagmi viem
```

### 2. RainbowKit
UI компоненты для подключения кошельков.

```bash
npm install @rainbow-me/rainbowkit
```

### 3. Web3Modal
Модальное окно для подключения кошельков.

```bash
npm install @web3modal/ethereum @web3modal/html
```

## Безопасность

### 1. Защита приватных ключей
```typescript
// Никогда не храните приватные ключи в открытом виде в коде
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY!, provider);
```

### 2. Проверка транзакций
```typescript
// Всегда проверяйте параметры транзакции перед отправкой
function validateTransaction(to: string, amount: string) {
  if (!ethers.isAddress(to)) {
    throw new Error('Неверный адрес получателя');
  }
  
  const parsedAmount = ethers.parseEther(amount);
  if (parsedAmount <= 0) {
    throw new Error('Неверная сумма');
  }
}
```

## Связанные темы

- [[Ethereum-и-TypeScript]]
- [[Solana-и-TypeScript]]
- [[Смарт-контракты]]
- [[Кошельки]]

## Внешние ресурсы

- [Ethers.js Documentation](https://docs.ethers.org/)
- [Web3.js Documentation](https://web3js.readthedocs.io/)
- [Solana Web3.js Documentation](https://solana-labs.github.io/solana-web3.js/)
- [Viem Documentation](https://viem.sh/)
- [Wagmi Documentation](https://wagmi.sh/)