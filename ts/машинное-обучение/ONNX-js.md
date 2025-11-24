---
aliases: [ONNX.js, ONNX Runtime Web]
tags: [typescript, machine-learning, onnx, browser-ml, model-inference, cross-platform]
---

# ONNX.js в TypeScript: Кроссплатформенные модели в браузере

## Введение

ONNX.js (Open Neural Network Exchange) - это библиотека, позволяющая запускать предварительно обученные модели машинного обучения в браузере. ONNX - это открытый стандарт для представления моделей машинного обучения, поддерживающий модели из различных фреймворков (PyTorch, TensorFlow, scikit-learn и др.).

## Установка и настройка

Для работы с ONNX.js в TypeScript проекте:

```bash
npm install onnxruntime-web
```

Или для использования в браузере:

```html
<script src="https://cdn.jsdelivr.net/npm/onnxruntime-web/dist/ort.js"></script>
```

## Базовое использование

### Загрузка модели

```typescript
import * as ort from 'onnxruntime-web';

// Загрузка модели из URL
const session = await ort.InferenceSession.create('./model.onnx');

// Загрузка модели из ArrayBuffer
const response = await fetch('./model.onnx');
const buffer = await response.arrayBuffer();
const sessionFromArrayBuffer = await ort.InferenceSession.create(buffer);
```

### Подготовка данных для инференса

```typescript
// Создание тензора для входных данных
const tensor = new ort.Tensor('float32', new Float32Array([1, 2, 3, 4]), [1, 4]);

// Подготовка данных для модели
interface ModelInput {
  [key: string]: ort.Tensor;
}

const inputs: ModelInput = {
  'input': tensor
};
```

### Выполнение инференса

```typescript
// Выполнение инференса
const output = await session.run(inputs);

// Получение результата
const result = output[session.outputNames[0]];
console.log('Результат инференса:', result.data);
console.log('Форма тензора:', result.dims);
```

## Работа с различными типами моделей

### Классификация изображений

```typescript
// Загрузка модели для классификации изображений
const imageClassificationModel = await ort.InferenceSession.create('./resnet50-v1-7.onnx');

// Функция подготовки изображения
function preprocessImage(image: HTMLImageElement): ort.Tensor {
  // Изменение размера изображения до 224x224 (для ResNet50)
  const canvas = document.createElement('canvas');
  canvas.width = 224;
  canvas.height = 224;
  const ctx = canvas.getContext('2d');
  ctx.drawImage(image, 0, 0, 224, 224);
  
  // Получение данных пикселей
  const imageData = ctx.getImageData(0, 0, 224, 224);
  const pixels = imageData.data;
  
  // Нормализация и преобразование в формат BGR (для ResNet50)
  const floatArray = new Float32Array(3 * 224 * 224);
  for (let i = 0; i < 224 * 224; i++) {
    floatArray[i] = (pixels[i * 4] / 255 - 0.485) / 0.229;       // R
    floatArray[224 * 224 + i] = (pixels[i * 4 + 1] / 255 - 0.456) / 0.224; // G
    floatArray[2 * 224 * 224 + i] = (pixels[i * 4 + 2] / 255 - 0.406) / 0.225; // B
  }
  
  return new ort.Tensor('float32', floatArray, [1, 3, 224, 224]);
}

// Выполнение классификации
async function classifyImage(image: HTMLImageElement): Promise<number[]> {
  const preprocessed = preprocessImage(image);
  const inputs: ModelInput = {
    'data': preprocessed
  };
  
  const output = await imageClassificationModel.run(inputs);
  const result = output[imageClassificationModel.outputNames[0]];
  
  return Array.from(result.data);
}
```

### Обработка текста

```typescript
// Модель для обработки естественного языка
const nlpModel = await ort.InferenceSession.create('./bert.onnx');

// Токенизация текста (упрощенная версия)
function tokenize(text: string, vocab: Map<string, number>): ort.Tensor {
  // Простая токенизация по пробелам
  const tokens = text.toLowerCase().split(/\s+/);
  const tokenIds = tokens.map(token => vocab.get(token) || 0);
  
  // Добавление специальных токенов
  tokenIds.unshift(vocab.get('[CLS]') || 0);
  tokenIds.push(vocab.get('[SEP]') || 0);
  
  // Паддинг до фиксированной длины
  const maxLength = 128;
  if (tokenIds.length < maxLength) {
    tokenIds.push(...Array(maxLength - tokenIds.length).fill(0));
  }
  
  return new ort.Tensor('int64', new BigInt64Array(tokenIds.map(n => BigInt(n))), [1, maxLength]);
}

// Выполнение NLP задачи
async function processText(text: string, vocab: Map<string, number>): Promise<number[]> {
  const tokens = tokenize(text, vocab);
  const inputs: ModelInput = {
    'input_ids': tokens
  };
  
  const output = await nlpModel.run(inputs);
  const result = output[nlpModel.outputNames[0]];
  
  return Array.from(result.data);
}
```

## Оптимизация производительности

### Выбор Execution Provider

```typescript
// Настройка сессии с определенными параметрами
const sessionOptions: ort.InferenceSession.SessionOptions = {
  executionProviders: ['webgl', 'wasm', 'cpu'], // Приоритет выполнения
  graphOptimizationLevel: 'all'
};

const optimizedSession = await ort.InferenceSession.create(
  './model.onnx',
  sessionOptions
);
```

### Управление памятью

```typescript
// Функция для очистки памяти после инференса
function disposeTensor(tensor: ort.Tensor): void {
  // ONNX.js не требует ручного управления памятью для тензоров
  // но важно понимать, что большие тензоры могут потреблять много памяти
  console.log(`Тензор с формой [${tensor.dims.join('x')}] будет автоматически очищен GC`);
}

// Проверка использования памяти
function logMemoryUsage(): void {
  if ((performance as any).memory) {
    const memory = (performance as any).memory;
    console.log(`Использование памяти: ${memory.usedJSHeapSize / 1024 / 1024} MB`);
  }
}
```

## Работа с несколькими моделями

```typescript
// Класс для управления несколькими ONNX моделями
class ModelManager {
  private models: Map<string, ort.InferenceSession>;
  
  constructor() {
    this.models = new Map();
  }
  
  async loadModel(name: string, url: string): Promise<void> {
    const session = await ort.InferenceSession.create(url);
    this.models.set(name, session);
    console.log(`Модель ${name} загружена`);
  }
  
  async runModel(modelName: string, inputs: ModelInput): Promise<ort.InferenceSession.RunResult> {
    const session = this.models.get(modelName);
    if (!session) {
      throw new Error(`Модель ${modelName} не найдена`);
    }
    
    return await session.run(inputs);
  }
  
  async dispose(): Promise<void> {
    // Очистка всех моделей
    for (const [name, session] of this.models) {
      // ONNX.js не имеет метода dispose для сессий
      console.log(`Модель ${name} будет очищена GC`);
    }
    this.models.clear();
  }
}

// Использование менеджера моделей
const modelManager = new ModelManager();
await modelManager.loadModel('classifier', './classifier.onnx');
await modelManager.loadModel('detector', './detector.onnx');
```

## Типизация и интерфейсы

```typescript
// Определение типов для работы с ONNX моделями
interface ONNXModelConfig {
  modelUrl: string;
  inputName: string;
  outputName: string;
  inputShape: number[];
  outputShape: number[];
}

interface ModelPrediction {
  data: Float32Array | Int32Array | BigInt64Array;
  dims: number[];
  type: ort.TensorType;
}

class TypedONNXModel {
  private session: ort.InferenceSession;
  private config: ONNXModelConfig;
  
  constructor(config: ONNXModelConfig) {
    this.config = config;
  }
  
  async load(): Promise<void> {
    this.session = await ort.InferenceSession.create(this.config.modelUrl);
  }
  
  async predict(inputTensor: ort.Tensor): Promise<ModelPrediction> {
    // Проверка формы входных данных
    if (JSON.stringify(inputTensor.dims) !== JSON.stringify(this.config.inputShape)) {
      throw new Error(`Неверная форма входных данных. Ожидается: [${this.config.inputShape}], получено: [${inputTensor.dims}]`);
    }
    
    const inputs: ModelInput = {
      [this.config.inputName]: inputTensor
    };
    
    const output = await this.session.run(inputs);
    const result = output[this.config.outputName];
    
    return {
      data: result.data,
      dims: result.dims,
      type: result.type
    };
  }
  
  async run(inputs: ModelInput): Promise<ort.InferenceSession.RunResult> {
    return await this.session.run(inputs);
  }
}
```

## Сравнение с другими библиотеками

| Характеристика | ONNX.js | TensorFlow.js | Tensor.js |
|----------------|---------|---------------|-----------|
| Формат модели | ONNX | TensorFlow/TF Lite | TensorFlow.js Layers |
| Поддержка фреймворков | Все ONNX-совместимые | TensorFlow | TensorFlow |
| Производительность | Высокая | Высокая | Высокая |
| Размер библиотеки | Меньше | Больше | Средний |
| Типизация | Через TypeScript | Встроена | Встроена |

## Практические примеры

### Пример: Запуск модели YOLO в браузере

```typescript
// Класс для детекции объектов с помощью YOLO
class YOLODetector {
  private model: TypedONNXModel;
  
  constructor(modelPath: string) {
    this.model = new TypedONNXModel({
      modelUrl: modelPath,
      inputName: 'images',
      outputName: 'output',
      inputShape: [1, 3, 640, 640],
      outputShape: [1, 25200, 85]
    });
  }
  
  async load(): Promise<void> {
    await this.model.load();
  }
  
  async detect(image: HTMLImageElement): Promise<ObjectDetectionResult[]> {
    // Предварительная обработка изображения
    const preprocessed = this.preprocessImage(image);
    
    // Выполнение инференса
    const result = await this.model.predict(preprocessed);
    
    // Постобработка результатов
    return this.postprocess(result);
  }
  
  private preprocessImage(image: HTMLImageElement): ort.Tensor {
    // Изменение размера изображения до 640x640
    const canvas = document.createElement('canvas');
    canvas.width = 640;
    canvas.height = 640;
    const ctx = canvas.getContext('2d');
    ctx.drawImage(image, 0, 0, 640, 640);
    
    // Получение данных пикселей
    const imageData = ctx.getImageData(0, 0, 640, 640);
    const pixels = imageData.data;
    
    // Нормализация и преобразование в формат BGR
    const floatArray = new Float32Array(3 * 640 * 640);
    for (let i = 0; i < 640 * 640; i++) {
      floatArray[i] = pixels[i * 4] / 255;         // R
      floatArray[640 * 640 + i] = pixels[i * 4 + 1] / 255; // G
      floatArray[2 * 640 * 640 + i] = pixels[i * 4 + 2] / 255; // B
    }
    
    return new ort.Tensor('float32', floatArray, [1, 3, 640, 640]);
  }
  
  private postprocess(result: ModelPrediction): ObjectDetectionResult[] {
    // Реализация NMS (Non-Maximum Suppression) и другие постобработки
    // ...
    return [];
  }
}

interface ObjectDetectionResult {
  className: string;
  confidence: number;
  bbox: [number, number, number, number]; // [x, y, width, height]
}
```

## Заключение

ONNX.js предоставляет мощную платформу для запуска моделей машинного обучения в браузере, особенно полезную для кроссплатформенных приложений. С помощью TypeScript можно обеспечить строгую типизацию и надежность при работе с моделями, полученными из различных фреймворков.

## См. также

- [[Модели-в-браузере]]
- [[Инференс]]
- [[TensorFlow-js]]
- [[Обучение-в-браузере]]
- [[Формат-ONNX-для-моделей]]