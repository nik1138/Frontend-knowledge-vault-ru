---
aliases: [TF.js, TensorFlow.js]
tags: [typescript, machine-learning, tensorflow, browser-ml, neural-networks]
---

# TensorFlow.js в TypeScript: Обработка данных в браузере

## Введение

TensorFlow.js - это библиотека машинного обучения от Google, позволяющая разрабатывать и запускать модели машинного обучения непосредственно в браузере или в Node.js. Использование TypeScript вместе с TensorFlow.js обеспечивает строгую типизацию и улучшенный опыт разработки.

## Установка и настройка

Для начала работы с TensorFlow.js в TypeScript проекте:

```bash
npm install @tensorflow/tfjs @types/tensorflow__tfjs
```

Или для использования в браузере:

```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
```

## Базовое использование

### Создание тензоров

```typescript
import * as tf from '@tensorflow/tfjs';

// Создание тензора из массива
const values: number[] = [1, 2, 3, 4];
const tensor1d = tf.tensor1d(values);

// Создание тензора с указанием типа
const tensor2d = tf.tensor2d([[1, 2], [3, 4]], [2, 2]);

// Создание тензора с типом данных
const floatTensor = tf.tensor2d([[1, 2], [3, 4]], [2, 2], 'float32');
```

### Основные операции

```typescript
// Арифметические операции
const a = tf.tensor1d([1, 2, 3]);
const b = tf.tensor1d([4, 5, 6]);

const sum = a.add(b); // [5, 7, 9]
const product = a.mul(b); // [4, 10, 18]

// Матричное умножение
const matrix1 = tf.tensor2d([[1, 2], [3, 4]]);
const matrix2 = tf.tensor2d([[5, 6], [7, 8]]);
const result = matrix1.matMul(matrix2);

// Операции сдвига и масштабирования
const normalized = tensor1d.sub(2).div(5);
```

## Создание и обучение модели

### Простая модель регрессии

```typescript
// Определение модели
const model = tf.sequential({
  layers: [
    tf.layers.dense({inputShape: [1], units: 1, useBias: true}),
    tf.layers.dense({units: 1, useBias: true})
  ]
});

// Компиляция модели
model.compile({
  optimizer: tf.train.adam(0.01),
  loss: 'meanSquaredError'
});

// Подготовка данных
const xs = tf.tensor2d([1, 2, 3, 4], [4, 1]);
const ys = tf.tensor2d([1, 3, 5, 7], [4, 1]);

// Обучение модели
await model.fit(xs, ys, {
  epochs: 100,
  callbacks: {
    onEpochEnd: async (epoch, logs) => {
      console.log(`Эпоха ${epoch}: потеря = ${logs.loss}`);
    }
  }
});
```

### Классификация

```typescript
// Модель для бинарной классификации
const binaryModel = tf.sequential({
  layers: [
    tf.layers.dense({
      units: 32,
      activation: 'relu',
      inputShape: [10]
    }),
    tf.layers.dense({
      units: 16,
      activation: 'relu'
    }),
    tf.layers.dense({
      units: 1,
      activation: 'sigmoid'
    })
  ]
});

binaryModel.compile({
  optimizer: 'adam',
  loss: 'binaryCrossentropy',
  metrics: ['accuracy']
});
```

## Загрузка предварительно обученных моделей

### Загрузка из URL

```typescript
// Загрузка модели из URL
const model = await tf.loadLayersModel('https://example.com/model.json');

// Загрузка весов модели
const weights = await tf.loadWeights(['https://example.com/weights.bin']);
```

### Загрузка из IndexedDB

```typescript
// Сохранение модели в IndexedDB
await model.save('indexeddb://my-model');

// Загрузка модели из IndexedDB
const loadedModel = await tf.loadLayersModel('indexeddb://my-model');
```

## Работа с изображениями

```typescript
// Загрузка изображения и преобразование в тензор
async function loadImageAsTensor(imageElement: HTMLImageElement): Promise<tf.Tensor3D> {
  const tensor = tf.browser.fromPixels(imageElement);
  
  // Нормализация значений пикселей
  const normalized = tensor.div(255);
  
  // Изменение размера изображения
  const resized = tf.image.resizeBilinear(normalized, [224, 224]);
  
  return resized;
}

// Предсказание с использованием модели
async function predictImage(model: tf.GraphModel, imageElement: HTMLImageElement) {
  const tensor = await loadImageAsTensor(imageElement);
  const prediction = model.predict(tensor.expandDims(0)) as tf.Tensor;
  return await prediction.data();
}
```

## Оптимизация производительности

### Выбор backend

```typescript
// Проверка доступных backend
console.log(tf.getBackend());
console.log(tf.engine().registeredVariables);

// Установка backend (webgl, cpu, wasm)
tf.setBackend('webgl');
tf.ready().then(() => {
  console.log('Backend установлен:', tf.getBackend());
});
```

### Использование tf.tidy для управления памятью

```typescript
// tf.tidy автоматически очищает созданные тензоры
const result = tf.tidy(() => {
  const a = tf.tensor2d([1, 2]);
  const b = tf.tensor2d([3, 4]);
  const c = a.matMul(b);
  return c.sum();
});
```

## Практические примеры

### Пример: Определение рукописных цифр

```typescript
// Загрузка предобученной модели MNIST
const model = await tf.loadLayersModel('https://storage.googleapis.com/tfjs-models/savedmodel/mnist/model.json');

// Функция предсказания
async function predictDigit(imageData: ImageData): Promise<number> {
  // Преобразование изображения в тензор
  const tensor = tf.browser.fromPixels(imageData, 1)
    .resizeNearestNeighbor([28, 28])
    .expandDims(0)
    .cast('float32')
    .div(255);

  // Получение предсказания
  const prediction = model.predict(tensor) as tf.Tensor;
  const predictedDigit = (await prediction.data())[0];
  
  tensor.dispose(); // Освобождение памяти
  
  return predictedDigit;
}
```

## Типизация и интерфейсы

```typescript
// Определение типов для работы с моделями
interface ModelConfig {
  inputShape: number[];
  outputShape: number[];
  hiddenLayers: number[];
}

interface PredictionResult {
  class: number;
  probability: number;
  allProbabilities: number[];
}

// Типизированный класс модели
class TypedModel {
  private model: tf.Sequential;
  
  constructor(config: ModelConfig) {
    this.model = this.buildModel(config);
  }
  
  private buildModel(config: ModelConfig): tf.Sequential {
    const model = tf.sequential();
    
    // Добавление входного слоя
    model.add(tf.layers.dense({
      units: config.hiddenLayers[0],
      inputShape: config.inputShape,
      activation: 'relu'
    }));
    
    // Добавление скрытых слоев
    for (let i = 1; i < config.hiddenLayers.length; i++) {
      model.add(tf.layers.dense({
        units: config.hiddenLayers[i],
        activation: 'relu'
      }));
    }
    
    // Добавление выходного слоя
    model.add(tf.layers.dense({
      units: config.outputShape[0],
      activation: 'softmax'
    }));
    
    return model;
  }
  
  async train(xs: tf.Tensor, ys: tf.Tensor, epochs: number): Promise<void> {
    this.model.compile({
      optimizer: 'adam',
      loss: 'categoricalCrossentropy',
      metrics: ['accuracy']
    });
    
    await this.model.fit(xs, ys, {epochs});
  }
  
  async predict(input: tf.Tensor): Promise<PredictionResult[]> {
    const predictions = this.model.predict(input) as tf.Tensor;
    const data = await predictions.data();
    
    // Преобразование в результаты с вероятностями
    const results: PredictionResult[] = [];
    const batchSize = input.shape[0];
    const numClasses = predictions.shape[1];
    
    for (let i = 0; i < batchSize; i++) {
      let maxIdx = 0;
      let maxVal = data[i * numClasses];
      
      for (let j = 1; j < numClasses; j++) {
        if (data[i * numClasses + j] > maxVal) {
          maxVal = data[i * numClasses + j];
          maxIdx = j;
        }
      }
      
      results.push({
        class: maxIdx,
        probability: maxVal,
        allProbabilities: Array.from(data.slice(i * numClasses, (i + 1) * numClasses))
      });
    }
    
    return results;
  }
}
```

## Заключение

TensorFlow.js в сочетании с TypeScript предоставляет мощную платформу для разработки приложений машинного обучения в браузере. Благодаря строгой типизации TypeScript, разработка становится более надежной и предсказуемой, особенно при работе с сложными структурами данных, какими являются тензоры.

## См. также

- [[Модели-в-браузере]]
- [[Обучение-в-браузере]]
- [[Инференс]]
- [[ONNX-js]]
- [[Типы-данных-в-машинном-обучении]]