---
aliases: [WebXR, WebXR API, VR/AR в браузере]
tags: [vr, ar, typescript, webxr, webvr, xr-api, browser-api]
---

# WebXR API для VR/AR приложений

## Введение в WebXR

WebXR - это современный веб-стандарт, который предоставляет API для создания VR (виртуальной реальности) и AR (дополненной реальности) приложений в браузере. WebXR заменил устаревший WebVR API и предлагает более широкие возможности для работы с различными устройствами XR.

## Основные концепции WebXR

### XR-сессии

XR-сессия - это основной контекст для работы с VR/AR устройствами. Она определяет режим работы (иммерсивный VR, AR или inline) и предоставляет доступ к отслеживанию позиции и ориентации.

```typescript
// Проверка поддержки WebXR
if ('xr' in navigator) {
  console.log('WebXR поддерживается');
} else {
  console.log('WebXR не поддерживается');
}

// Запрос сессии
async function initXRSession() {
  try {
    // Проверка доступности сессии
    const isSupported = await navigator.xr.isSessionSupported('immersive-vr');
    
    if (isSupported) {
      // Запрос сессии
      const session = await navigator.xr.requestSession('immersive-vr');
      
      // Настройка сессии
      await setupXRSession(session);
    } else {
      console.warn('Immersive VR не поддерживается');
    }
  } catch (error) {
    console.error('Ошибка при запросе XR-сессии:', error);
  }
}
```

### XR-пространства

XR-пространства определяют систему координат для отслеживания позиции и ориентации.

```typescript
// Ссылочные пространства
async function setupXRSession(session: XRSession) {
  // Запрос ссылочного пространства
  const refSpace = await session.requestReferenceSpace('local');
  
  // Для AR может использоваться 'bounded-floor' или 'unbounded'
  // const arRefSpace = await session.requestReferenceSpace('bounded-floor');
  
  // Для комнатного масштаба - 'unbounded'
  // const roomScaleRefSpace = await session.requestReferenceSpace('unbounded');
  
  // Начало сеанса отслеживания
  session.requestAnimationFrame((time, frame) => {
    onXRFrame(time, frame, session, refSpace);
  });
}
```

## Создание WebXR приложения

### Основная структура

```typescript
import * as THREE from 'three';

class WebXRApp {
  private renderer: THREE.WebGLRenderer;
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;
  private xrSession: XRSession | null = null;
  private xrRefSpace: XRReferenceSpace | null = null;
  private controllers: (XRInputSource | null)[] = [null, null];
  
  constructor() {
    this.initThreeJS();
  }
  
  private initThreeJS() {
    // Создание Three.js компонентов
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(this.renderer.domElement);
    
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(0x333333);
    
    // Камера будет управляться WebXR
    this.camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
  }
  
  public async initXR() {
    if (!navigator.xr) {
      console.error('WebXR не поддерживается');
      return;
    }
    
    try {
      // Проверка поддержки режима
      const isSupported = await navigator.xr.isSessionSupported('immersive-vr');
      
      if (isSupported) {
        // Запрос сессии
        this.xrSession = await navigator.xr.requestSession('immersive-vr');
        
        // Настройка рендерера для WebXR
        this.renderer.xr.enabled = true;
        this.renderer.xr.setSession(this.xrSession);
        
        // Получение ссылочного пространства
        this.xrRefSpace = await this.xrSession.requestReferenceSpace('local');
        
        // Настройка контроллеров
        this.setupControllers();
        
        // Запуск анимации
        this.renderer.setAnimationLoop(this.animate.bind(this));
      } else {
        console.warn('Immersive VR не поддерживается');
      }
    } catch (error) {
      console.error('Ошибка инициализации WebXR:', error);
    }
  }
  
  private setupControllers() {
    if (!this.xrSession) return;
    
    // Обработка подключения контроллеров
    this.xrSession.addEventListener('inputsourceschange', (event) => {
      // Обновление списка источников ввода
      const inputSources = this.xrSession?.inputSources || [];
      
      // Назначение контроллеров
      for (let i = 0; i < inputSources.length; i++) {
        if (i < 2) { // Обычно максимум 2 контроллера
          this.controllers[i] = inputSources[i];
        }
      }
    });
  }
  
  private animate(timestamp: number, frame?: XRFrame) {
    if (frame && this.xrRefSpace) {
      // Получение позиции и ориентации головы
      const pose = frame.getViewerPose(this.xrRefSpace);
      
      if (pose) {
        // Обновление камеры на основе позы
        const view = pose.views[0]; // Обычно только один вид для VR
        if (view) {
          // Обновление матрицы камеры
          this.camera.matrix.fromArray(view.transform.matrix);
          this.camera.matrix.decompose(
            this.camera.position,
            this.camera.quaternion,
            this.camera.scale
          );
          
          // Обновление проекционной матрицы
          this.camera.projectionMatrix.fromArray(view.projectionMatrix);
        }
      }
      
      // Обработка контроллеров
      this.handleControllers(frame);
    }
    
    // Рендеринг сцены
    this.renderer.render(this.scene, this.camera);
  }
  
  private handleControllers(frame: XRFrame) {
    if (!this.xrSession || !this.xrRefSpace) return;
    
    for (const inputSource of this.xrSession.inputSources) {
      if (inputSource.targetRayMode === 'tracked-pointer') {
        const targetRayPose = frame.getPose(inputSource.targetRaySpace, this.xrRefSpace);
        
        if (targetRayPose) {
          // Обработка позы контроллера
          console.log('Поза контроллера:', targetRayPose.transform);
          
          // Обработка нажатий кнопок
          if (inputSource.gamepad) {
            for (let i = 0; i < inputSource.gamepad.buttons.length; i++) {
              const button = inputSource.gamepad.buttons[i];
              if (button.pressed) {
                console.log(`Кнопка ${i} нажата`);
              }
            }
          }
        }
      }
    }
  }
}

// Использование WebXR приложения
const app = new WebXRApp();
app.initXR();
```

## AR-функциональность

### AR-сессия

```typescript
// Инициализация AR-сессии
async function initARSession() {
  try {
    const isSupported = await navigator.xr.isSessionSupported('immersive-ar');
    
    if (isSupported) {
      const session = await navigator.xr.requestSession('immersive-ar', {
        requiredFeatures: ['hit-test', 'dom-overlay'],
        domOverlay: { root: document.body }
      });
      
      // Настройка AR-сессии
      const refSpace = await session.requestReferenceSpace('local');
      const viewerRefSpace = await session.requestReferenceSpace('viewer');
      
      // Создание хит-теста для определения поверхностей
      const hitTestSource = await session.requestHitTestSource({
        space: viewerRefSpace
      });
      
      // Начало отслеживания
      session.requestAnimationFrame((time, frame) => {
        onARFrame(time, frame, session, refSpace, hitTestSource);
      });
    }
  } catch (error) {
    console.error('Ошибка при инициализации AR-сессии:', error);
  }
}

// Обработка AR-кадра
function onARFrame(
  time: number, 
  frame: XRFrame, 
  session: XRSession, 
  refSpace: XRReferenceSpace,
  hitTestSource: XRHitTestSource
) {
  const pose = frame.getViewerPose(refSpace);
  
  if (pose) {
    // Выполнение хит-теста
    const hitTestResults = frame.getHitTestResults(hitTestSource);
    
    if (hitTestResults.length > 0) {
      const hit = hitTestResults[0];
      const pose = hit.getPose(refSpace);
      
      if (pose) {
        // Позиция точки касания в пространстве
        console.log('Точка касания:', pose.transform);
        
        // Здесь можно размещать 3D-объекты в точке касания
      }
    }
  }
  
  // Рендеринг
  renderer.render(scene, camera);
}
```

## Управление сессией

### Жизненный цикл сессии

```typescript
class XRSessionManager {
  private session: XRSession | null = null;
  private referenceSpace: XRReferenceSpace | null = null;
  private animationFrame: number | null = null;
  private onFrameCallback: ((time: number, frame: XRFrame) => void) | null = null;
  
  public async startSession(mode: XRSessionMode) {
    try {
      // Проверка поддержки режима
      const isSupported = await navigator.xr!.isSessionSupported(mode);
      
      if (!isSupported) {
        throw new Error(`Режим ${mode} не поддерживается`);
      }
      
      // Запрос сессии с дополнительными возможностями
      this.session = await navigator.xr!.requestSession(mode, {
        optionalFeatures: ['local-floor', 'bounded-floor', 'hand-tracking', 'layers']
      });
      
      // Настройка сессии
      await this.configureSession();
      
      // Запуск отслеживания
      this.startFrameLoop();
    } catch (error) {
      console.error('Ошибка запуска XR-сессии:', error);
    }
  }
  
  private async configureSession() {
    if (!this.session) return;
    
    // Установка renderState
    await this.session.updateRenderState({
      baseLayer: new XRWebGLLayer(this.session, renderer.getContext())
    });
    
    // Запрос ссылочного пространства
    this.referenceSpace = await this.session.requestReferenceSpace('local');
  }
  
  private startFrameLoop() {
    if (!this.session || !this.referenceSpace) return;
    
    const frameCallback = (time: number, frame: XRFrame) => {
      if (this.onFrameCallback) {
        this.onFrameCallback(time, frame);
      }
      
      // Продолжение цикла
      this.animationFrame = this.session!.requestAnimationFrame(frameCallback);
    };
    
    this.animationFrame = this.session.requestAnimationFrame(frameCallback);
  }
  
  public stopSession() {
    if (this.animationFrame !== null) {
      if (this.session) {
        this.session.cancelAnimationFrame(this.animationFrame);
      }
      this.animationFrame = null;
    }
    
    if (this.session) {
      this.session.end();
      this.session = null;
    }
  }
  
  public setOnFrameCallback(callback: (time: number, frame: XRFrame) => void) {
    this.onFrameCallback = callback;
  }
}
```

## Обработка ввода

### Контроллеры и взаимодействие

```typescript
class XRInputHandler {
  private inputSources: Map<XRInputSource, XRJointSpace | XRRaySpace> = new Map();
  private gripSpaces: Map<XRInputSource, XRSpace> = new Map();
  private targetRaySpaces: Map<XRInputSource, XRSpace> = new Map();
  
  constructor(private session: XRSession, private referenceSpace: XRReferenceSpace) {
    this.setupInputListeners();
  }
  
  private setupInputListeners() {
    this.session.addEventListener('inputsourceschange', (event) => {
      // Обработка добавления новых источников ввода
      for (const inputSource of event.added) {
        this.addInputSource(inputSource);
      }
      
      // Обработка удаления источников ввода
      for (const inputSource of event.removed) {
        this.removeInputSource(inputSource);
      }
    });
  }
  
  private addInputSource(inputSource: XRInputSource) {
    // Для контроллеров с трекингом руки
    if (inputSource.hand) {
      // Использование hand-tracking
      for (const finger of ['thumb', 'index', 'middle', 'ring', 'pinky']) {
        for (let joint = 0; joint < 4; joint++) {
          const jointName = `${finger}-${joint}` as XRHandJoint;
          const jointSpace = inputSource.hand.get(jointName);
          
          if (jointSpace) {
            this.inputSources.set(inputSource, jointSpace);
          }
        }
      }
    } else if (inputSource.targetRayMode === 'tracked-pointer') {
      // Для контроллеров
      this.targetRaySpaces.set(inputSource, inputSource.targetRaySpace);
      
      if (inputSource.gripSpace) {
        this.gripSpaces.set(inputSource, inputSource.gripSpace);
      }
    }
  }
  
  private removeInputSource(inputSource: XRInputSource) {
    this.inputSources.delete(inputSource);
    this.gripSpaces.delete(inputSource);
    this.targetRaySpaces.delete(inputSource);
  }
  
  public processInput(frame: XRFrame) {
    for (const [inputSource, space] of this.inputSources) {
      const pose = frame.getPose(space, this.referenceSpace);
      
      if (pose) {
        // Обработка позы
        this.handlePose(inputSource, pose);
      }
    }
    
    // Обработка контроллеров
    for (const inputSource of this.session.inputSources) {
      if (inputSource.gamepad) {
        this.processGamepad(inputSource);
      }
    }
  }
  
  private handlePose(inputSource: XRInputSource, pose: XRPose) {
    // Позиция и ориентация источника ввода
    const position = new THREE.Vector3();
    const quaternion = new THREE.Quaternion();
    
    position.setFromMatrixPosition(new THREE.Matrix4().fromArray(pose.transform.matrix));
    quaternion.setFromRotationMatrix(new THREE.Matrix4().fromArray(pose.transform.matrix));
    
    console.log(`Позиция ${inputSource.handedness}:`, position);
    console.log(`Ориентация ${inputSource.handedness}:`, quaternion);
  }
  
  private processGamepad(inputSource: XRInputSource) {
    if (!inputSource.gamepad) return;
    
    const gamepad = inputSource.gamepad;
    
    // Обработка кнопок
    for (let i = 0; i < gamepad.buttons.length; i++) {
      const button = gamepad.buttons[i];
      
      if (button.pressed) {
        console.log(`Кнопка ${i} на ${inputSource.handedness} контроллере нажата`);
      }
      
      if (button.touched) {
        console.log(`Кнопка ${i} на ${inputSource.handedness} контроллере тронута`);
      }
    }
    
    // Обработка осей (например, джойстики)
    for (let i = 0; i < gamepad.axes.length; i += 2) {
      const x = gamepad.axes[i];
      const y = gamepad.axes[i + 1];
      
      if (Math.abs(x) > 0.1 || Math.abs(y) > 0.1) {
        console.log(`Ось ${i/2} на ${inputSource.handedness} контроллере: x=${x}, y=${y}`);
      }
    }
  }
}
```

## Оптимизация WebXR приложений

### Эффективная обработка кадров

```typescript
class OptimizedXRApp {
  private frameCount = 0;
  private lastFrameTime = 0;
  private frameRate = 0;
  
  public onXRFrame(time: number, frame: XRFrame, session: XRSession, refSpace: XRReferenceSpace) {
    // Измерение частоты кадров
    if (this.lastFrameTime) {
      const deltaTime = time - this.lastFrameTime;
      this.frameRate = 1000 / deltaTime;
      this.frameCount++;
      
      // Логирование каждые 100 кадров
      if (this.frameCount % 100 === 0) {
        console.log(`Частота кадров: ${this.frameRate.toFixed(1)} FPS`);
      }
    }
    this.lastFrameTime = time;
    
    // Получение позы
    const pose = frame.getViewerPose(refSpace);
    if (!pose) return;
    
    // Обработка только необходимых данных
    for (const view of pose.views) {
      // Используем только нужные данные из view
      this.renderView(view, frame);
    }
    
    // Обработка ввода
    this.handleInput(frame);
  }
  
  private renderView(view: XRView, frame: XRFrame) {
    // Обновление камеры для конкретного вида
    const viewport = frame.session.renderState.baseLayer!.getViewport(view);
    if (!viewport) return;
    
    // Установка области отрисовки
    this.renderer.setViewport(viewport.x, viewport.y, viewport.width, viewport.height);
    
    // Обновление проекционной матрицы
    this.camera.projectionMatrix.fromArray(view.projectionMatrix);
    
    // Обновление позиции камеры
    const viewMatrix = new THREE.Matrix4().fromArray(view.transform.inverse.matrix);
    this.camera.matrixWorldInverse.copy(viewMatrix);
    this.camera.matrixWorld.getInverse(this.camera.matrixWorldInverse);
  }
  
  private handleInput(frame: XRFrame) {
    // Оптимизированная обработка ввода
    // Обработка только активных контроллеров
    for (const inputSource of frame.session.inputSources) {
      if (inputSource.gamepad && inputSource.gamepad.buttons.some(b => b.pressed)) {
        // Обработка только при необходимости
        this.processControllerInput(inputSource, frame);
      }
    }
  }
  
  private processControllerInput(inputSource: XRInputSource, frame: XRFrame) {
    // Конкретная обработка ввода
    const targetRayPose = frame.getPose(inputSource.targetRaySpace, this.referenceSpace);
    if (targetRayPose) {
      // Обработка позы луча
      console.log(`Поза луча для ${inputSource.handedness}:`, targetRayPose.transform);
    }
  }
}
```

## Совместимость и резервные решения

### Проверка возможностей

```typescript
class WebXRCompatibility {
  static async checkCapabilities(): Promise<WebXRCapabilities> {
    const capabilities: WebXRCapabilities = {
      isSupported: false,
      immersiveVR: false,
      immersiveAR: false,
      supportsHandTracking: false,
      supportsPlaneDetection: false,
      maxLayers: 1
    };
    
    if (!navigator.xr) {
      return capabilities;
    }
    
    capabilities.isSupported = true;
    
    try {
      // Проверка поддержки VR
      capabilities.immersiveVR = await navigator.xr.isSessionSupported('immersive-vr');
      
      // Проверка поддержки AR
      capabilities.immersiveAR = await navigator.xr.isSessionSupported('immersive-ar');
      
      // Проверка дополнительных возможностей
      if (capabilities.immersiveVR) {
        const session = await navigator.xr.requestSession('immersive-vr');
        const supportedFeatures = session.enabledFeatures || [];
        
        capabilities.supportsHandTracking = supportedFeatures.includes('hand-tracking');
        capabilities.maxLayers = supportedFeatures.includes('layers') ? 2 : 1;
        
        session.end();
      }
      
      if (capabilities.immersiveAR) {
        const session = await navigator.xr.requestSession('immersive-ar');
        const supportedFeatures = session.enabledFeatures || [];
        
        capabilities.supportsPlaneDetection = supportedFeatures.includes('plane-detection');
        
        session.end();
      }
    } catch (error) {
      console.warn('Ошибка при проверке возможностей WebXR:', error);
    }
    
    return capabilities;
  }
}

interface WebXRCapabilities {
  isSupported: boolean;
  immersiveVR: boolean;
  immersiveAR: boolean;
  supportsHandTracking: boolean;
  supportsPlaneDetection: boolean;
  maxLayers: number;
}
```

## Заключение

WebXR API предоставляет мощный и стандартизированный способ создания VR/AR приложений в вебе. Его гибкая архитектура позволяет разрабатывать приложения для различных устройств и сценариев использования.

При разработке WebXR приложений важно:
- Обеспечить корректную обработку жизненного цикла сессии
- Оптимизировать производительность, как описано в [[Оптимизация-рендеринга]]
- Реализовать качественное взаимодействие с пользователем через [[Взаимодействие-с-пользователем]]
- Использовать подходящие библиотеки для графики, такие как [[Three-js]] или [[A-Frame]]

## Ключевые теги
#vr #ar #typescript #webxr #webvr #xr-api #browser-api #virtual-reality #augmented-reality