---
aliases: [FormData и загрузка файлов, FormData в HTML, Загрузка файлов в HTML]
tags: [html, networking, javascript, web-development, file-upload]
---

# FormData и загрузка файлов

## Введение в FormData

FormData - это встроенный объект JavaScript, который позволяет легко создавать пары ключ/значение для отправки с использованием XMLHttpRequest или [[Fetch API]]. Он особенно полезен при отправке форм и загрузке файлов, так как автоматически устанавливает правильный заголовок `Content-Type` с подходящей границей (boundary) для multipart/form-data.

## Основы использования FormData

### Создание FormData

```javascript
// Создание пустого FormData
const formData = new FormData();

// Создание FormData из существующей формы
const formElement = document.getElementById('myForm');
const formDataFromForm = new FormData(formElement);
```

### Добавление данных

```javascript
const formData = new FormData();

// Добавление простых значений
formData.append('name', 'Иван');
formData.append('email', 'ivan@example.com');

// Добавление файла
const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];
formData.append('avatar', file, file.name); // файл с указанием имени

// Добавление Blob-объекта
const blob = new Blob(['Hello, world!'], { type: 'text/plain' });
formData.append('textFile', blob, 'hello.txt');
```

## Загрузка файлов с использованием FormData

### Простая загрузка файла

```html
<form id="uploadForm">
  <input type="file" id="fileInput" name="file" />
  <button type="submit">Загрузить файл</button>
</form>
```

```javascript
document.getElementById('uploadForm').addEventListener('submit', async (event) => {
  event.preventDefault();
  
  const formData = new FormData();
  const fileInput = document.getElementById('fileInput');
  const file = fileInput.files[0];
  
  if (!file) {
    alert('Пожалуйста, выберите файл');
    return;
  }
  
  formData.append('file', file);
  
  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
    
    if (response.ok) {
      const result = await response.json();
      console.log('Файл успешно загружен:', result);
    } else {
      console.error('Ошибка загрузки файла');
    }
  } catch (error) {
    console.error('Ошибка сети:', error);
  }
});
```

### Загрузка нескольких файлов

```javascript
async function uploadMultipleFiles() {
  const formData = new FormData();
  const fileInput = document.getElementById('multipleFiles');
  
  for (let i = 0; i < fileInput.files.length; i++) {
    formData.append('files', fileInput.files[i]);
  }
  
  const response = await fetch('/api/upload-multiple', {
    method: 'POST',
    body: formData
  });
  
  return response.json();
}
```

## Отслеживание прогресса загрузки

### С использованием XMLHttpRequest

```javascript
function uploadWithProgress(file) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    
    formData.append('file', file);
    
    // Отслеживание прогресса отправки
    xhr.upload.onprogress = function(event) {
      if (event.lengthComputable) {
        const percentComplete = (event.loaded / event.total) * 100;
        console.log(`Прогресс загрузки: ${percentComplete.toFixed(2)}%`);
        
        // Обновление индикатора прогресса в UI
        updateProgressBar(percentComplete);
      }
    };
    
    xhr.onload = function() {
      if (xhr.status === 200) {
        const response = JSON.parse(xhr.responseText);
        resolve(response);
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Сетевая ошибка'));
    };
    
    xhr.open('POST', '/api/upload');
    xhr.send(formData);
  });
}

function updateProgressBar(percent) {
  const progressBar = document.getElementById('progressBar');
  progressBar.style.width = `${percent}%`;
  progressBar.textContent = `${percent.toFixed(0)}%`;
}
```

### С использованием Fetch API

```javascript
// Fetch API не предоставляет встроенного способа отслеживания прогресса отправки
// Поэтому для отслеживания прогресса лучше использовать XMLHttpRequest
// Но можно создать обертку для совместимости:

function fetchWithProgress(url, formData, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.upload.onprogress = onProgress;
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.response));
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Сетевая ошибка'));
    };
    
    xhr.open('POST', url);
    xhr.send(formData);
  });
}

// Использование
const formData = new FormData();
formData.append('file', fileInput.files[0]);

fetchWithProgress('/api/upload', formData, (event) => {
  if (event.lengthComputable) {
    const percentComplete = (event.loaded / event.total) * 100;
    console.log(`Прогресс: ${percentComplete}%`);
  }
})
.then(result => console.log('Загружено:', result))
.catch(error => console.error('Ошибка:', error));
```

## Работа с различными типами данных

### Текстовые данные

```javascript
const formData = new FormData();

// Добавление текстовых значений
formData.append('title', 'Мой заголовок');
formData.append('description', 'Описание файла');

// Добавление числовых значений (преобразуются в строки)
formData.append('size', 1024);
```

### Файлы и Blob-объекты

```javascript
const formData = new FormData();

// Добавление файла из input
const fileInput = document.getElementById('fileInput');
formData.append('file', fileInput.files[0]);

// Добавление Blob-объекта
const jsonBlob = new Blob([JSON.stringify({ name: 'data' })], { type: 'application/json' });
formData.append('data', jsonBlob, 'data.json');
```

### Получение данных из FormData

```javascript
const formData = new FormData();
formData.append('name', 'Иван');
formData.append('email', 'ivan@example.com');

// Получение значения по ключу
console.log(formData.get('name')); // 'Иван'

// Получение всех значений по ключу (если добавлено несколько раз)
formData.append('tags', 'javascript');
formData.append('tags', 'html');
console.log(formData.getAll('tags')); // ['javascript', 'html']

// Проверка наличия ключа
console.log(formData.has('name')); // true

// Перебор всех пар ключ-значение
for (const [key, value] of formData.entries()) {
  console.log(key, value);
}
```

## Практические примеры

### Загрузка аватара с предварительным просмотром

```html
<div>
  <input type="file" id="avatarInput" accept="image/*" />
  <img id="preview" style="max-width: 200px; max-height: 200px;" />
  <div id="progressBar" style="width: 0%; height: 20px; background: #4CAF50;"></div>
  <button id="uploadBtn">Загрузить аватар</button>
</div>
```

```javascript
document.getElementById('avatarInput').addEventListener('change', function(event) {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = function(e) {
      document.getElementById('preview').src = e.target.result;
    };
    reader.readAsDataURL(file);
  }
});

document.getElementById('uploadBtn').addEventListener('click', async function() {
  const fileInput = document.getElementById('avatarInput');
  const file = fileInput.files[0];
  
  if (!file) {
    alert('Пожалуйста, выберите файл');
    return;
  }
  
  const formData = new FormData();
  formData.append('avatar', file);
  
  try {
    const response = await fetch('/api/upload-avatar', {
      method: 'POST',
      body: formData
    });
    
    if (response.ok) {
      alert('Аватар успешно загружен!');
    } else {
      alert('Ошибка загрузки аватара');
    }
  } catch (error) {
    console.error('Ошибка:', error);
    alert('Произошла ошибка при загрузке');
  }
});
```

### Многопоточная загрузка файлов

```javascript
async function uploadFilesConcurrently(files) {
  const uploadPromises = Array.from(files).map(file => {
    const formData = new FormData();
    formData.append('file', file);
    
    return fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
  });
  
  try {
    const responses = await Promise.all(uploadPromises);
    const results = await Promise.all(
      responses.map(response => response.json())
    );
    
    console.log('Все файлы загружены:', results);
    return results;
  } catch (error) {
    console.error('Ошибка загрузки файлов:', error);
    throw error;
  }
}
```

## Безопасность при загрузке файлов

### Проверка типа файла на клиенте

```javascript
function validateFile(file) {
  // Проверка расширения файла
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif'];
  const fileExtension = '.' + file.name.split('.').pop().toLowerCase();
  
  if (!allowedExtensions.includes(fileExtension)) {
    throw new Error('Недопустимый тип файла');
  }
  
  // Проверка MIME-типа
  const allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];
  if (!allowedMimeTypes.includes(file.type)) {
    throw new Error('Недопустимый MIME-тип файла');
  }
  
  // Проверка размера файла (например, не более 5 МБ)
  const maxSize = 5 * 1024 * 1024; // 5 МБ в байтах
  if (file.size > maxSize) {
    throw new Error('Файл слишком большой');
  }
  
  return true;
}

// Использование
try {
  const file = document.getElementById('fileInput').files[0];
  validateFile(file);
  
  const formData = new FormData();
  formData.append('file', file);
  
  // Отправка файла
} catch (error) {
  console.error('Ошибка валидации файла:', error.message);
}
```

### Защита от вредоносных файлов

> [!warning] Важно
> Клиентская валидация не обеспечивает полной безопасности. Всегда проводите проверку файлов на сервере.

## Практические рекомендации для российских разработчиков (2025)

### Работа с российскими API и сервисами

При загрузке файлов в российские сервисы:
- Обратите внимание на требования к типам файлов
- Учитывайте ограничения на размер файлов
- Проверяйте политику CORS
- Используйте HTTPS для передачи файлов

### Совместимость с российскими браузерами

FormData поддерживается во всех современных браузерах, включая российские (Яндекс.Браузер, Mail.Ru и др.), но при поддержке старых браузеров может потребоваться полифилл.

### Ограничения и особенности

- Файлы не могут быть загружены с использованием GET-запросов
- FormData автоматически устанавливает правильный Content-Type
- При загрузке больших файлов учитывайте ограничения сервера
- Используйте chunked upload для очень больших файлов

## Современные возможности

### Загрузка файлов с Drag & Drop

```javascript
const dropZone = document.getElementById('dropZone');

dropZone.addEventListener('dragover', (event) => {
  event.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('drag-over');
});

dropZone.addEventListener('drop', (event) => {
  event.preventDefault();
  dropZone.classList.remove('drag-over');
  
  const files = event.dataTransfer.files;
  
  // Обработка файлов
  const formData = new FormData();
  for (let i = 0; i < files.length; i++) {
    formData.append('files', files[i]);
  }
  
  // Отправка файлов
  uploadFiles(formData);
});
```

### Компрессия изображений перед загрузкой

```javascript
function compressImage(file, quality = 0.8) {
  return new Promise((resolve) => {
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    const img = new Image();
    
    img.onload = function() {
      // Установка размеров canvas
      canvas.width = img.width;
      canvas.height = img.height;
      
      // Рисование изображения на canvas
      ctx.drawImage(img, 0, 0);
      
      // Преобразование в Blob с компрессией
      canvas.toBlob(resolve, 'image/jpeg', quality);
    };
    
    img.src = URL.createObjectURL(file);
  });
}

// Использование
async function uploadCompressedImage() {
  const fileInput = document.getElementById('imageInput');
  const file = fileInput.files[0];
  
  if (file && file.type.startsWith('image/')) {
    const compressedFile = await compressImage(file);
    const formData = new FormData();
    formData.append('image', compressedFile, 'compressed_' + file.name);
    
    // Отправка сжатого изображения
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
    
    return response.json();
  }
}
```

## Полезные ресурсы

- [[Fetch-API-в-HTML]] - для отправки FormData с использованием Fetch API
- [[XMLHttpRequest-в-HTML]] - для отправки FormData с использованием XMLHttpRequest
- [[Обработка-ответов-и-ошибок]] - обработка ответов от сервера при загрузке файлов
- [[CORS-и-безопасность-запросов]] - вопросы безопасности при загрузке файлов

## Заключение

FormData - мощный инструмент для работы с формами и загрузкой файлов в веб-приложениях. Его правильное использование позволяет создавать удобные интерфейсы для загрузки файлов с отслеживанием прогресса и валидацией данных.

> [!tip] Совет
> Всегда проводите валидацию файлов на сервере, независимо от клиентской проверки, для обеспечения безопасности.

> [!info] Примечание
> FormData автоматически устанавливает правильный заголовок Content-Type для multipart/form-data, что упрощает отправку файлов.