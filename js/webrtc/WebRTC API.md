# WebRTC API

## Введение

WebRTC (Web Real-Time Communication) - это технология, которая позволяет веб-приложениям осуществлять прямую передачу данных, аудио и видео между браузерами без необходимости в промежуточных серверах. В этой главе мы рассмотрим все аспекты WebRTC: RTCPeerConnection, RTCDataChannel, RTCDTMFSender, а также сигнализацию, STUN/TURN серверы и другие важные компоненты.

## Содержание

- [[Основы WebRTC]]
- [[RTCPeerConnection]]
- [[RTCDataChannel]]
- [[RTCDTMFSender]]
- [[Сигнализация]]
- [[STUN и TURN серверы]]
- [[Обработка медиа потоков]]
- [[Безопасность WebRTC]]
- [[Примеры использования]]

## Основы WebRTC

### Введение в WebRTC

WebRTC - это технология для создания приложений реального времени в браузере. Она позволяет обмениваться аудио, видео и произвольными данными между браузерами без дополнительных плагинов.

```javascript
// Класс для проверки поддержки WebRTC
class WebRTCBasics {
    static isSupported() {
        return !!(window.RTCPeerConnection && 
                  window.RTCIceCandidate && 
                  window.RTCSessionDescription);
    }
    
    static getSupportedConstraints() {
        return navigator.mediaDevices.getSupportedConstraints();
    }
    
    static async checkPermissions() {
        if (!WebRTCBasics.isSupported()) {
            throw new Error('WebRTC не поддерживается в этом браузере');
        }
        
        try {
            // Проверка доступа к медиаустройствам
            const stream = await navigator.mediaDevices.getUserMedia({ 
                video: true, 
                audio: true 
            });
            
            // Остановка треков для освобождения камер/микрофонов
            stream.getTracks().forEach(track => track.stop());
            
            return true;
        } catch (error) {
            console.error('Ошибка проверки разрешений:', error);
            return false;
        }
    }
    
    // Получение информации о медиаустройствах
    static async getMediaDevices() {
        if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
            throw new Error('enumerateDevices не поддерживается');
        }
        
        const devices = await navigator.mediaDevices.enumerateDevices();
        return {
            cameras: devices.filter(device => device.kind === 'videoinput'),
            microphones: devices.filter(device => device.kind === 'audioinput'),
            speakers: devices.filter(device => device.kind === 'audiooutput')
        };
    }
}

// Пример использования
console.log('WebRTC поддерживается:', WebRTCBasics.isSupported());
WebRTCBasics.checkPermissions().then(hasPermission => {
    console.log('Разрешения получены:', hasPermission);
});

WebRTCBasics.getMediaDevices().then(devices => {
    console.log('Камеры:', devices.cameras);
    console.log('Микрофоны:', devices.microphones);
    console.log('Колонки:', devices.speakers);
});
```

### Класс для управления WebRTC соединениями

```javascript
// Основной класс для управления WebRTC
class WebRTCManager {
    constructor(config = {}) {
        this.config = {
            iceServers: [
                { urls: 'stun:stun.l.google.com:19302' },
                { urls: 'stun:stun1.l.google.com:19302' }
            ],
            iceTransportPolicy: 'all',
            bundlePolicy: 'balanced',
            rtcpMuxPolicy: 'require',
            ...config
        };
        
        this.peerConnection = null;
        this.dataChannels = new Map();
        this.remoteStreams = new Map();
        this.localStreams = new Map();
        this.isStarted = false;
        this.isInitiator = false;
        this.roomId = null;
        
        this.eventListeners = new Map();
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.onicecandidate = null;
        this.oniceconnectionstatechange = null;
        this.onsignalingstatechange = null;
        this.ontrack = null;
        this.ondatachannel = null;
        this.onconnectionstatechange = null;
        this.onicegatheringstatechange = null;
    }
    
    // Создание RTCPeerConnection
    createPeerConnection() {
        if (this.peerConnection) {
            this.peerConnection.close();
        }
        
        this.peerConnection = new RTCPeerConnection(this.config);
        
        // Установка обработчиков событий
        this.peerConnection.onicecandidate = (event) => {
            if (this.onicecandidate) {
                this.onicecandidate(event);
            }
        };
        
        this.peerConnection.oniceconnectionstatechange = () => {
            if (this.oniceconnectionstatechange) {
                this.oniceconnectionstatechange(this.peerConnection.iceConnectionState);
            }
        };
        
        this.peerConnection.onsignalingstatechange = () => {
            if (this.onsignalingstatechange) {
                this.onsignalingstatechange(this.peerConnection.signalingState);
            }
        };
        
        this.peerConnection.ontrack = (event) => {
            if (this.ontrack) {
                this.ontrack(event);
            }
        };
        
        this.peerConnection.ondatachannel = (event) => {
            if (this.ondatachannel) {
                this.ondatachannel(event);
            }
        };
        
        this.peerConnection.onconnectionstatechange = () => {
            if (this.onconnectionstatechange) {
                this.onconnectionstatechange(this.peerConnection.connectionState);
            }
        };
        
        this.peerConnection.onicegatheringstatechange = () => {
            if (this.onicegatheringstatechange) {
                this.onicegatheringstatechange(this.peerConnection.iceGatheringState);
            }
        };
        
        return this.peerConnection;
    }
    
    // Добавление локального потока
    async addLocalStream(stream) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        stream.getTracks().forEach(track => {
            this.peerConnection.addTrack(track, stream);
        });
        
        const streamId = stream.id;
        this.localStreams.set(streamId, stream);
        
        return streamId;
    }
    
    // Удаление локального потока
    removeLocalStream(streamId) {
        const stream = this.localStreams.get(streamId);
        if (stream) {
            stream.getTracks().forEach(track => {
                const sender = this.peerConnection.getSenders().find(s => s.track === track);
                if (sender) {
                    this.peerConnection.removeTrack(sender);
                }
            });
            this.localStreams.delete(streamId);
        }
    }
    
    // Создание DataChannel
    createDataChannel(label, options = {}) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        const channel = this.peerConnection.createDataChannel(label, {
            ordered: true,
            maxRetransmits: 3,
            ...options
        });
        
        this.setupDataChannel(channel);
        return channel;
    }
    
    // Настройка DataChannel
    setupDataChannel(channel) {
        channel.onopen = () => {
            console.log('Data channel открыт:', channel.label);
        };
        
        channel.onclose = () => {
            console.log('Data channel закрыт:', channel.label);
            this.dataChannels.delete(channel.label);
        };
        
        channel.onerror = (error) => {
            console.error('Ошибка в data channel:', error);
        };
        
        channel.onmessage = (event) => {
            this.handleDataChannelMessage(channel.label, event.data);
        };
        
        this.dataChannels.set(channel.label, channel);
    }
    
    // Обработка сообщений от DataChannel
    handleDataChannelMessage(channelLabel, data) {
        console.log(`Сообщение от ${channelLabel}:`, data);
    }
    
    // Отправка сообщения через DataChannel
    sendToChannel(label, data) {
        const channel = this.dataChannels.get(label);
        if (channel && channel.readyState === 'open') {
            channel.send(typeof data === 'string' ? data : JSON.stringify(data));
        } else {
            throw new Error(`Data channel ${label} не открыт`);
        }
    }
    
    // Создание предложения (offer)
    async createOffer(options = {}) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        const offer = await this.peerConnection.createOffer(options);
        await this.peerConnection.setLocalDescription(offer);
        
        return offer;
    }
    
    // Создание ответа (answer)
    async createAnswer(options = {}) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        const answer = await this.peerConnection.createAnswer(options);
        await this.peerConnection.setLocalDescription(answer);
        
        return answer;
    }
    
    // Установка удаленного описания
    async setRemoteDescription(description) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        await this.peerConnection.setRemoteDescription(description);
    }
    
    // Добавление ICE кандидата
    async addIceCandidate(candidate) {
        if (!this.peerConnection) {
            throw new Error('Peer connection не создана');
        }
        
        await this.peerConnection.addIceCandidate(candidate);
    }
    
    // Начало соединения
    async start(isInitiator = false) {
        this.isInitiator = isInitiator;
        
        if (!this.peerConnection) {
            this.createPeerConnection();
        }
        
        if (isInitiator) {
            // Инициатор создает offer
            const offer = await this.createOffer();
            return offer;
        } else {
            // Не инициатор ждет offer
            // Обработка будет происходить через обработчики событий
        }
        
        this.isStarted = true;
    }
    
    // Завершение соединения
    close() {
        if (this.peerConnection) {
            this.peerConnection.close();
            this.peerConnection = null;
        }
        
        // Закрытие всех data channels
        this.dataChannels.forEach(channel => {
            if (channel.readyState === 'open') {
                channel.close();
            }
        });
        this.dataChannels.clear();
        
        // Остановка всех локальных потоков
        this.localStreams.forEach(stream => {
            stream.getTracks().forEach(track => track.stop());
        });
        this.localStreams.clear();
        
        this.isStarted = false;
    }
    
    // Получение состояния соединения
    getConnectionState() {
        if (!this.peerConnection) {
            return 'closed';
        }
        
        return {
            connectionState: this.peerConnection.connectionState,
            iceConnectionState: this.peerConnection.iceConnectionState,
            signalingState: this.peerConnection.signalingState,
            iceGatheringState: this.peerConnection.iceGatheringState
        };
    }
}

// Использование WebRTCManager
const webrtcManager = new WebRTCManager({
    iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        // Добавьте TURN серверы при необходимости
    ]
});
```

## RTCPeerConnection

RTCPeerConnection - это основной класс для управления WebRTC соединением.

### Основы RTCPeerConnection

```javascript
// Класс для расширенного управления RTCPeerConnection
class AdvancedPeerConnection {
    constructor(config) {
        this.config = config;
        this.pc = new RTCPeerConnection(config);
        this.statsInterval = null;
        this.bandwidthController = new BandwidthController(this.pc);
        this.qualityController = new QualityController(this.pc);
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        // Обработчик ICE кандидатов
        this.pc.onicecandidate = (event) => {
            if (event.candidate) {
                this.handleIceCandidate(event.candidate);
            } else {
                console.log('ICE gathering завершен');
                this.handleIceGatheringComplete();
            }
        };
        
        // Обработчик изменения состояния соединения
        this.pc.onconnectionstatechange = () => {
            this.handleConnectionStateChange(this.pc.connectionState);
        };
        
        // Обработчик изменения состояния ICE
        this.pc.oniceconnectionstatechange = () => {
            this.handleIceConnectionStateChange(this.pc.iceConnectionState);
        };
        
        // Обработчик изменения сигнального состояния
        this.pc.onsignalingstatechange = () => {
            this.handleSignalingStateChange(this.pc.signalingState);
        };
        
        // Обработчик прибытия медиа треков
        this.pc.ontrack = (event) => {
            this.handleRemoteTrack(event.track, event.streams[0]);
        };
        
        // Обработчик прибытия data channel
        this.pc.ondatachannel = (event) => {
            this.handleDataChannel(event.channel);
        };
        
        // Обработчик изменения состояния сбора ICE
        this.pc.onicegatheringstatechange = () => {
            this.handleIceGatheringStateChange(this.pc.iceGatheringState);
        };
        
        // Обработчик субъективного качества соединения
        this.pc.onnegotiationneeded = () => {
            this.handleNegotiationNeeded();
        };
    }
    
    handleIceCandidate(candidate) {
        // Отправка ICE кандидата через сигнализацию
        this.sendSignalingMessage({
            type: 'ice-candidate',
            candidate: candidate
        });
    }
    
    handleIceGatheringComplete() {
        // ICE gathering завершен, можно отправлять SDP
        this.sendSignalingMessage({
            type: 'ice-gathering-complete'
        });
    }
    
    handleConnectionStateChange(state) {
        console.log('Состояние соединения изменилось:', state);
        
        switch (state) {
            case 'connected':
                console.log('Соединение установлено');
                this.startStatsCollection();
                break;
            case 'disconnected':
                console.log('Соединение потеряно');
                break;
            case 'failed':
                console.log('Соединение не удалось');
                this.handleConnectionFailure();
                break;
            case 'closed':
                console.log('Соединение закрыто');
                this.stopStatsCollection();
                break;
        }
    }
    
    handleIceConnectionStateChange(state) {
        console.log('Состояние ICE соединения изменилось:', state);
        
        switch (state) {
            case 'checking':
                console.log('Проверка соединения...');
                break;
            case 'connected':
                console.log('ICE соединение установлено');
                break;
            case 'completed':
                console.log('ICE соединение завершено');
                break;
            case 'failed':
                console.log('ICE соединение не удалось');
                this.handleIceFailure();
                break;
        }
    }
    
    handleSignalingStateChange(state) {
        console.log('Сигнальное состояние изменилось:', state);
    }
    
    handleRemoteTrack(track, stream) {
        console.log('Прибыл удаленный трек:', track.kind, track.id);
        
        if (stream) {
            console.log('Прибыл удаленный поток:', stream.id);
        }
        
        // Добавление трека в отображение
        this.addRemoteTrackToDisplay(track);
    }
    
    handleDataChannel(channel) {
        console.log('Прибыл data channel:', channel.label);
        
        // Настройка канала
        this.setupDataChannel(channel);
    }
    
    handleIceGatheringStateChange(state) {
        console.log('Состояние сбора ICE изменилось:', state);
    }
    
    handleNegotiationNeeded() {
        console.log('Требуется переговоры');
        this.performRenegotiation();
    }
    
    // Отправка сигнальных сообщений
    sendSignalingMessage(message) {
        // В реальном приложении это будет через WebSocket, Socket.io или другие средства сигнализации
        console.log('Отправка сигнального сообщения:', message);
        
        // Пример отправки через WebSocket
        if (this.signalingSocket) {
            this.signalingSocket.send(JSON.stringify(message));
        }
    }
    
    // Добавление удаленного трека к отображению
    addRemoteTrackToDisplay(track) {
        const elementId = `remote-${track.kind}-${track.id}`;
        const element = document.getElementById(elementId);
        
        if (element && element.srcObject) {
            element.srcObject.addTrack(track);
        } else {
            // Создание нового элемента для отображения
            const mediaElement = track.kind === 'video' ? 
                document.createElement('video') : 
                document.createElement('audio');
                
            mediaElement.id = elementId;
            mediaElement.autoplay = true;
            mediaElement.playsInline = true;
            
            mediaElement.srcObject = new MediaStream([track]);
            
            // Добавление к документу
            document.body.appendChild(mediaElement);
        }
    }
    
    // Настройка DataChannel
    setupDataChannel(channel) {
        channel.onopen = () => {
            console.log('Data channel открыт:', channel.label);
        };
        
        channel.onclose = () => {
            console.log('Data channel закрыт:', channel.label);
        };
        
        channel.onerror = (error) => {
            console.error('Ошибка data channel:', error);
        };
        
        channel.onmessage = (event) => {
            this.handleDataChannelMessage(event.data);
        };
    }
    
    handleDataChannelMessage(data) {
        console.log('Получено сообщение через data channel:', data);
    }
    
    // Управление полосой пропускания
    setBandwidthConstraints(constraints) {
        return this.bandwidthController.applyConstraints(constraints);
    }
    
    // Управление качеством видео
    setVideoQualityConstraints(constraints) {
        return this.qualityController.applyVideoConstraints(constraints);
    }
    
    // Сбор статистики
    startStatsCollection(interval = 1000) {
        if (this.statsInterval) {
            clearInterval(this.statsInterval);
        }
        
        this.statsInterval = setInterval(async () => {
            try {
                const stats = await this.pc.getStats();
                this.processStats(stats);
            } catch (error) {
                console.error('Ошибка сбора статистики:', error);
            }
        }, interval);
    }
    
    stopStatsCollection() {
        if (this.statsInterval) {
            clearInterval(this.statsInterval);
            this.statsInterval = null;
        }
    }
    
    processStats(stats) {
        // Обработка статистики
        stats.forEach(report => {
            if (report.type === 'inbound-rtp') {
                console.log('Прием RTP:', {
                    packetsReceived: report.packetsReceived,
                    bytesReceived: report.bytesReceived,
                    jitter: report.jitter,
                    packetsLost: report.packetsLost
                });
            } else if (report.type === 'outbound-rtp') {
                console.log('Отправка RTP:', {
                    packetsSent: report.packetsSent,
                    bytesSent: report.bytesSent,
                    roundTripTime: report.roundTripTime
                });
            } else if (report.type === 'candidate-pair') {
                console.log('Пара кандидатов:', {
                    state: report.state,
                    nominated: report.nominated,
                    writable: report.writable
                });
            }
        });
    }
    
    // Обработка сбоя соединения
    handleConnectionFailure() {
        console.log('Обработка сбоя соединения');
        // Здесь может быть попытка восстановления соединения
        this.attemptRecovery();
    }
    
    handleIceFailure() {
        console.log('Обработка сбоя ICE');
        // Может потребоваться использование TURN сервера
    }
    
    // Попытка восстановления соединения
    async attemptRecovery() {
        try {
            // Попытка пересоздания соединения
            const offer = await this.pc.createOffer({
                iceRestart: true
            });
            
            await this.pc.setLocalDescription(offer);
            
            // Отправка нового offer через сигнализацию
            this.sendSignalingMessage({
                type: 'offer',
                sdp: offer.sdp
            });
        } catch (error) {
            console.error('Ошибка восстановления соединения:', error);
        }
    }
    
    // Переговоры
    async performRenegotiation() {
        if (this.pc.signalingState !== 'stable') {
            // Ждем завершения текущих переговоров
            return;
        }
        
        try {
            const offer = await this.pc.createOffer();
            await this.pc.setLocalDescription(offer);
            
            this.sendSignalingMessage({
                type: 'offer',
                sdp: offer.sdp
            });
        } catch (error) {
            console.error('Ошибка переговоров:', error);
        }
    }
    
    // Закрытие соединения
    close() {
        this.stopStatsCollection();
        this.pc.close();
    }
}

// Класс для управления полосой пропускания
class BandwidthController {
    constructor(peerConnection) {
        this.pc = peerConnection;
    }
    
    async applyConstraints(constraints) {
        const senders = this.pc.getSenders();
        
        for (const sender of senders) {
            if (sender.track && sender.track.kind === 'video') {
                await this.applyVideoBandwidthConstraint(sender, constraints.video);
            } else if (sender.track && sender.track.kind === 'audio') {
                await this.applyAudioBandwidthConstraint(sender, constraints.audio);
            }
        }
    }
    
    async applyVideoBandwidthConstraint(sender, bandwidth) {
        const parameters = sender.getParameters();
        
        if (!parameters.encodings) {
            parameters.encodings = [{}];
        }
        
        if (bandwidth) {
            parameters.encodings[0].maxBitrate = bandwidth.max * 1000; // в битах
            parameters.encodings[0].minBitrate = bandwidth.min * 1000;
            parameters.encodings[0].scaleResolutionDownBy = bandwidth.scalingFactor || 1;
        }
        
        await sender.setParameters(parameters);
    }
    
    async applyAudioBandwidthConstraint(sender, bandwidth) {
        const parameters = sender.getParameters();
        
        if (!parameters.encodings) {
            parameters.encodings = [{}];
        }
        
        if (bandwidth) {
            parameters.encodings[0].maxBitrate = bandwidth.max * 1000;
            parameters.encodings[0].minBitrate = bandwidth.min * 1000;
        }
        
        await sender.setParameters(parameters);
    }
}

// Класс для управления качеством видео
class QualityController {
    constructor(peerConnection) {
        this.pc = peerConnection;
    }
    
    async applyVideoConstraints(constraints) {
        const senders = this.pc.getSenders();
        
        for (const sender of senders) {
            if (sender.track && sender.track.kind === 'video') {
                await this.applyVideoQuality(sender, constraints);
            }
        }
    }
    
    async applyVideoQuality(sender, constraints) {
        const parameters = sender.getParameters();
        
        if (constraints.resolution) {
            // Управление разрешением (если поддерживается)
            if (parameters.encodings && parameters.encodings[0]) {
                parameters.encodings[0].scaleResolutionDownBy = 
                    constraints.resolution.scalingFactor || 1;
            }
        }
        
        if (constraints.frameRate) {
            // Управление частотой кадров
            if (parameters.encodings && parameters.encodings[0]) {
                parameters.encodings[0].maxFramerate = constraints.frameRate.max;
                parameters.encodings[0].minFramerate = constraints.frameRate.min;
            }
        }
        
        await sender.setParameters(parameters);
    }
}
```

## RTCDataChannel

RTCDataChannel позволяет передавать произвольные данные между участниками WebRTC-соединения.

### Основы RTCDataChannel

```javascript
// Класс для управления RTCDataChannel
class DataChannelManager {
    constructor(peerConnection) {
        this.pc = peerConnection;
        this.channels = new Map();
        this.channelQueues = new Map(); // Очереди для недоступных каналов
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        this.pc.ondatachannel = (event) => {
            this.handleIncomingChannel(event.channel);
        };
    }
    
    // Создание нового канала
    createChannel(label, options = {}) {
        const defaultOptions = {
            ordered: true,
            maxRetransmits: null, // или число для ограничения переповторов
            maxPacketLifeTime: null, // или число в миллисекундах
            protocol: '',
            negotiated: false,
            id: null
        };
        
        const channelOptions = { ...defaultOptions, ...options };
        const channel = this.pc.createDataChannel(label, channelOptions);
        
        this.setupChannel(channel);
        return channel;
    }
    
    // Создание канала с предопределенным ID
    createNegotiatedChannel(label, id, options = {}) {
        const channelOptions = {
            ...options,
            negotiated: true,
            id: id
        };
        
        const channel = this.pc.createDataChannel(label, channelOptions);
        this.setupChannel(channel);
        return channel;
    }
    
    // Настройка канала
    setupChannel(channel) {
        channel.onopen = () => {
            console.log(`Канал открыт: ${channel.label}`);
            this.handleChannelOpen(channel);
        };
        
        channel.onclose = () => {
            console.log(`Канал закрыт: ${channel.label}`);
            this.handleChannelClose(channel);
        };
        
        channel.onerror = (error) => {
            console.error(`Ошибка канала ${channel.label}:`, error);
            this.handleChannelError(channel, error);
        };
        
        channel.onmessage = (event) => {
            this.handleChannelMessage(channel, event.data);
        };
        
        channel.onbufferedamountlow = () => {
            console.log(`Буфер канала ${channel.label} ниже порога`);
            this.handleBufferedAmountLow(channel);
        };
        
        this.channels.set(channel.label, channel);
        
        // Отправка сообщений из очереди
        if (this.channelQueues.has(channel.label)) {
            this.sendQueuedMessages(channel);
        }
    }
    
    handleChannelOpen(channel) {
        // Канал готов к отправке данных
        console.log(`Канал ${channel.label} готов к использованию`);
        
        // Проверяем наличие сообщений в очереди
        if (this.channelQueues.has(channel.label)) {
            this.sendQueuedMessages(channel);
        }
    }
    
    handleChannelClose(channel) {
        this.channels.delete(channel.label);
        console.log(`Канал ${channel.label} удален из менеджера`);
    }
    
    handleChannelError(channel, error) {
        console.error(`Ошибка в канале ${channel.label}:`, error);
        // Здесь может быть логика восстановления или уведомления пользователя
    }
    
    handleChannelMessage(channel, data) {
        try {
            // Пытаемся распарсить как JSON, если не удается, то как обычный текст
            let parsedData;
            try {
                parsedData = JSON.parse(data);
            } catch {
                parsedData = data;
            }
            
            // Генерация события
            this.emit('message', {
                channel: channel.label,
                data: parsedData,
                timestamp: Date.now()
            });
        } catch (error) {
            console.error('Ошибка обработки сообщения:', error);
        }
    }
    
    handleBufferedAmountLow(channel) {
        // Уровень буфера снова приемлемый, можно отправлять больше данных
        if (this.channelQueues.has(channel.label)) {
            this.sendQueuedMessages(channel);
        }
    }
    
    handleIncomingChannel(channel) {
        console.log(`Прибыл входящий канал: ${channel.label}`);
        this.setupChannel(channel);
    }
    
    // Отправка сообщения
    send(channelLabel, data, options = {}) {
        const channel = this.channels.get(channelLabel);
        
        if (!channel) {
            // Канал еще не готов, помещаем в очередь
            this.queueMessage(channelLabel, data, options);
            return false;
        }
        
        if (channel.readyState !== 'open') {
            // Канал не открыт, помещаем в очередь
            this.queueMessage(channelLabel, data, options);
            return false;
        }
        
        // Проверка буфера, чтобы не переполнить
        if (options.highWaterMark && channel.bufferedAmount > options.highWaterMark) {
            this.queueMessage(channelLabel, data, options);
            return false;
        }
        
        try {
            const message = typeof data === 'string' ? data : JSON.stringify(data);
            channel.send(message);
            return true;
        } catch (error) {
            console.error('Ошибка отправки сообщения:', error);
            this.queueMessage(channelLabel, data, options);
            return false;
        }
    }
    
    // Помещение сообщения в очередь
    queueMessage(channelLabel, data, options) {
        if (!this.channelQueues.has(channelLabel)) {
            this.channelQueues.set(channelLabel, []);
        }
        
        const queue = this.channelQueues.get(channelLabel);
        queue.push({ data, options, timestamp: Date.now() });
        
        // Ограничение размера очереди
        if (queue.length > 100) { // максимальная длина очереди
            queue.shift(); // удаление старейшего сообщения
            console.warn(`Очередь канала ${channelLabel} переполнена`);
        }
    }
    
    // Отправка сообщений из очереди
    sendQueuedMessages(channel) {
        if (!this.channelQueues.has(channel.label)) {
            return;
        }
        
        const queue = this.channelQueues.get(channel.label);
        
        while (queue.length > 0 && channel.readyState === 'open') {
            const queuedMessage = queue.shift();
            const sent = this.send(channel.label, queuedMessage.data, queuedMessage.options);
            
            if (!sent) {
                // Если не удалось отправить, возвращаем в начало очереди
                queue.unshift(queuedMessage);
                break;
            }
        }
        
        // Очистка очереди если она пуста
        if (queue.length === 0) {
            this.channelQueues.delete(channel.label);
        }
    }
    
    // Отправка сообщения с подтверждением
    sendWithAck(channelLabel, data, timeout = 5000) {
        return new Promise((resolve, reject) => {
            const messageId = this.generateMessageId();
            
            // Добавляем ID сообщения
            const messageWithId = {
                id: messageId,
                data: data,
                timestamp: Date.now()
            };
            
            // Устанавливаем таймер ожидания подтверждения
            const timeoutId = setTimeout(() => {
                reject(new Error(`Таймаут подтверждения сообщения ${messageId}`));
            }, timeout);
            
            // Отправляем сообщение
            const sent = this.send(channelLabel, messageWithId);
            
            if (!sent) {
                clearTimeout(timeoutId);
                reject(new Error('Не удалось отправить сообщение'));
                return;
            }
            
            // Ждем подтверждение
            const ackListener = (event) => {
                if (event.data && event.data.type === 'ack' && event.data.id === messageId) {
                    clearTimeout(timeoutId);
                    this.removeListener('message', ackListener);
                    resolve(event.data);
                }
            };
            
            this.addListener('message', ackListener);
        });
    }
    
    // Отправка подтверждения
    sendAck(channelLabel, messageId) {
        const ackMessage = {
            type: 'ack',
            id: messageId,
            timestamp: Date.now()
        };
        
        this.send(channelLabel, ackMessage);
    }
    
    // Получение статистики по каналам
    getChannelStats() {
        const stats = {};
        
        this.channels.forEach((channel, label) => {
            stats[label] = {
                label: channel.label,
                readyState: channel.readyState,
                bufferedAmount: channel.bufferedAmount,
                maxRetransmits: channel.maxRetransmits,
                ordered: channel.ordered,
                protocol: channel.protocol,
                messagesSent: channel.messagesSent || 0,
                messagesReceived: channel.messagesReceived || 0
            };
        });
        
        return stats;
    }
    
    // Закрытие канала
    closeChannel(channelLabel) {
        const channel = this.channels.get(channelLabel);
        if (channel) {
            channel.close();
            this.channels.delete(channelLabel);
        }
    }
    
    // Закрытие всех каналов
    closeAllChannels() {
        this.channels.forEach((channel, label) => {
            channel.close();
        });
        
        this.channels.clear();
        this.channelQueues.clear();
    }
    
    // Генерация ID сообщения
    generateMessageId() {
        return Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
    
    // Методы для событий (упрощенная реализация)
    listeners = {};
    
    addListener(event, listener) {
        if (!this.listeners[event]) {
            this.listeners[event] = [];
        }
        this.listeners[event].push(listener);
    }
    
    removeListener(event, listener) {
        if (this.listeners[event]) {
            const index = this.listeners[event].indexOf(listener);
            if (index > -1) {
                this.listeners[event].splice(index, 1);
            }
        }
    }
    
    emit(event, data) {
        if (this.listeners[event]) {
            this.listeners[event].forEach(listener => listener(data));
        }
    }
}

// Использование DataChannelManager
class VideoCallApp {
    constructor() {
        this.pc = new RTCPeerConnection({
            iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
        });
        
        this.dataChannelManager = new DataChannelManager(this.pc);
        this.setupCall();
    }
    
    setupCall() {
        // Создание каналов для разных типов данных
        const controlChannel = this.dataChannelManager.createChannel('control', {
            ordered: true,
            maxRetransmits: 3
        });
        
        const dataChannel = this.dataChannelManager.createChannel('data', {
            ordered: false,
            maxRetransmits: null
        });
        
        const fileChannel = this.dataChannelManager.createChannel('files', {
            ordered: true,
            maxPacketLifeTime: 500
        });
        
        // Пример использования
        controlChannel.onopen = () => {
            // Отправка служебных сообщений
            this.dataChannelManager.send('control', {
                type: 'call_start',
                timestamp: Date.now()
            });
        };
        
        dataChannel.onmessage = (event) => {
            console.log('Получено сообщение:', event.data);
        };
    }
    
    // Отправка чата
    sendChatMessage(message) {
        return this.dataChannelManager.send('data', {
            type: 'chat',
            message: message,
            timestamp: Date.now()
        });
    }
    
    // Отправка файлов
    sendFile(file) {
        return this.dataChannelManager.send('files', {
            type: 'file',
            name: file.name,
            size: file.size,
            data: file // В реальности файл нужно читать и отправлять чанками
        });
    }
}
```

## Сигнализация

Сигнализация в WebRTC - это процесс обмена метаданными (SDP, ICE кандидатами) между участниками соединения.

### Система сигнализации

```javascript
// Класс для управления сигнализацией
class SignalingManager {
    constructor() {
        this.socket = null;
        this.roomId = null;
        this.peerId = null;
        this.handlers = new Map();
        this.setupDefaultHandlers();
    }
    
    // Подключение к сигнальному серверу
    connect(serverUrl, options = {}) {
        return new Promise((resolve, reject) => {
            this.socket = new WebSocket(serverUrl);
            
            this.socket.onopen = () => {
                console.log('Сигнальное соединение установлено');
                
                // Генерация ID участника
                this.peerId = this.generatePeerId();
                
                // Отправка приветственного сообщения
                this.send({
                    type: 'join',
                    peerId: this.peerId,
                    roomId: options.roomId || null
                });
                
                resolve(this.socket);
            };
            
            this.socket.onclose = (event) => {
                console.log('Сигнальное соединение закрыто:', event.code, event.reason);
                this.handleConnectionClose(event);
            };
            
            this.socket.onerror = (error) => {
                console.error('Ошибка сигнального соединения:', error);
                reject(error);
            };
            
            this.socket.onmessage = (event) => {
                this.handleMessage(JSON.parse(event.data));
            };
        });
    }
    
    // Генерация ID участника
    generatePeerId() {
        return 'peer_' + Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
    
    // Отправка сообщения
    send(message) {
        if (this.socket && this.socket.readyState === WebSocket.OPEN) {
            this.socket.send(JSON.stringify(message));
        } else {
            throw new Error('Сигнальное соединение не установлено');
        }
    }
    
    // Обработка входящих сообщений
    handleMessage(message) {
        const { type, ...data } = message;
        
        if (this.handlers.has(type)) {
            this.handlers.get(type)(data);
        } else {
            console.warn('Неизвестный тип сигнального сообщения:', type);
        }
    }
    
    setupDefaultHandlers() {
        // Обработчик приветственного сообщения
        this.handlers.set('welcome', (data) => {
            this.peerId = data.peerId;
            this.roomId = data.roomId;
            console.log('Добро пожаловать! ID:', this.peerId);
        });
        
        // Обработчик списка участников
        this.handlers.set('peers', (data) => {
            console.log('Участники комнаты:', data.peers);
            this.emit('peersList', data.peers);
        });
        
        // Обработчик нового участника
        this.handlers.set('peerJoined', (data) => {
            console.log('Новый участник:', data.peerId);
            this.emit('peerJoined', data);
        });
        
        // Обработчик ушедшего участника
        this.handlers.set('peerLeft', (data) => {
            console.log('Участник покинул комнату:', data.peerId);
            this.emit('peerLeft', data);
        });
        
        // Обработчик SDP предложения
        this.handlers.set('offer', (data) => {
            console.log('Получен offer от:', data.from);
            this.emit('offer', data);
        });
        
        // Обработчик SDP ответа
        this.handlers.set('answer', (data) => {
            console.log('Получен answer от:', data.from);
            this.emit('answer', data);
        });
        
        // Обработчик ICE кандидата
        this.handlers.set('iceCandidate', (data) => {
            console.log('Получен ICE кандидат от:', data.from);
            this.emit('iceCandidate', data);
        });
        
        // Обработчик завершения сбора ICE
        this.handlers.set('iceGatheringComplete', (data) => {
            console.log('ICE gathering завершен от:', data.from);
            this.emit('iceGatheringComplete', data);
        });
        
        // Обработчик сообщений чата
        this.handlers.set('chat', (data) => {
            this.emit('chatMessage', data);
        });
        
        // Обработчик служебных сообщений
        this.handlers.set('control', (data) => {
            this.emit('controlMessage', data);
        });
    }
    
    // Отправка SDP предложения
    sendOffer(toPeerId, sdp, roomId = null) {
        this.send({
            type: 'offer',
            to: toPeerId,
            from: this.peerId,
            sdp: sdp,
            roomId: roomId || this.roomId
        });
    }
    
    // Отправка SDP ответа
    sendAnswer(toPeerId, sdp, roomId = null) {
        this.send({
            type: 'answer',
            to: toPeerId,
            from: this.peerId,
            sdp: sdp,
            roomId: roomId || this.roomId
        });
    }
    
    // Отправка ICE кандидата
    sendIceCandidate(toPeerId, candidate, roomId = null) {
        this.send({
            type: 'iceCandidate',
            to: toPeerId,
            from: this.peerId,
            candidate: candidate,
            roomId: roomId || this.roomId
        });
    }
    
    // Отправка сообщения чата
    sendChatMessage(message, toPeerId = null) {
        const chatMessage = {
            type: 'chat',
            from: this.peerId,
            message: message,
            timestamp: Date.now()
        };
        
        if (toPeerId) {
            chatMessage.to = toPeerId;
            chatMessage.roomId = this.roomId;
        } else {
            chatMessage.roomId = this.roomId;
        }
        
        this.send(chatMessage);
    }
    
    // Присоединение к комнате
    joinRoom(roomId) {
        this.send({
            type: 'joinRoom',
            roomId: roomId,
            peerId: this.peerId
        });
    }
    
    // Покидание комнаты
    leaveRoom() {
        this.send({
            type: 'leaveRoom',
            roomId: this.roomId,
            peerId: this.peerId
        });
    }
    
    // Закрытие соединения
    close() {
        if (this.socket) {
            this.socket.close();
        }
    }
    
    // Обработчик закрытия соединения
    handleConnectionClose(event) {
        this.emit('connectionClosed', event);
        
        // Попытка переподключения
        if (event.code !== 1000) { // 1000 - нормальное закрытие
            this.attemptReconnect();
        }
    }
    
    // Попытка переподключения
    attemptReconnect(maxRetries = 5) {
        let retryCount = 0;
        
        const reconnect = () => {
            if (retryCount >= maxRetries) {
                console.error('Достигнуто максимальное количество попыток переподключения');
                return;
            }
            
            console.log(`Попытка переподключения... (${retryCount + 1}/${maxRetries})`);
            
            // Увеличиваем задержку с каждой попыткой
            setTimeout(() => {
                this.connect(this.currentServerUrl)
                    .then(() => {
                        console.log('Переподключение успешно');
                        // Восстанавливаем состояние комнаты
                        if (this.roomId) {
                            this.joinRoom(this.roomId);
                        }
                    })
                    .catch(error => {
                        retryCount++;
                        reconnect();
                    });
            }, Math.pow(2, retryCount) * 1000); // Экспоненциальная задержка
        };
        
        reconnect();
    }
    
    // Методы для событий (упрощенная реализация)
    eventListeners = {};
    
    on(event, listener) {
        if (!this.eventListeners[event]) {
            this.eventListeners[event] = [];
        }
        this.eventListeners[event].push(listener);
    }
    
    off(event, listener) {
        if (this.eventListeners[event]) {
            const index = this.eventListeners[event].indexOf(listener);
            if (index > -1) {
                this.eventListeners[event].splice(index, 1);
            }
        }
    }
    
    emit(event, data) {
        if (this.eventListeners[event]) {
            this.eventListeners[event].forEach(listener => listener(data));
        }
    }
}

// Пример использования сигнального менеджера
class VideoConferenceApp {
    constructor() {
        this.signaling = new SignalingManager();
        this.webrtc = new WebRTCManager();
        this.peers = new Map();
        this.setupEventHandlers();
    }
    
    async initialize() {
        // Подключение к сигнальному серверу
        await this.signaling.connect('ws://localhost:8080');
        
        // Присоединение к комнате
        this.signaling.joinRoom('conference-room-123');
        
        // Получение медиа потока
        const stream = await navigator.mediaDevices.getUserMedia({
            video: true,
            audio: true
        });
        
        // Добавление потока к WebRTC
        await this.webrtc.addLocalStream(stream);
        
        // Отображение локального видео
        this.displayLocalVideo(stream);
    }
    
    setupEventHandlers() {
        // Обработчики сигнализации
        this.signaling.on('offer', (data) => {
            this.handleOffer(data);
        });
        
        this.signaling.on('answer', (data) => {
            this.handleAnswer(data);
        });
        
        this.signaling.on('iceCandidate', (data) => {
            this.handleIceCandidate(data);
        });
        
        this.signaling.on('peerJoined', (data) => {
            this.handlePeerJoined(data.peerId);
        });
        
        this.signaling.on('peerLeft', (data) => {
            this.handlePeerLeft(data.peerId);
        });
        
        // Обработчики WebRTC
        this.webrtc.onicecandidate = (event) => {
            if (event.candidate) {
                this.signaling.sendIceCandidate(
                    this.getTargetPeerId(),
                    event.candidate
                );
            }
        };
        
        this.webrtc.ontrack = (event) => {
            this.displayRemoteVideo(event.track, event.streams[0]);
        };
        
        this.webrtc.onconnectionstatechange = (state) => {
            console.log('Состояние соединения:', state);
        };
    }
    
    async handlePeerJoined(peerId) {
        if (!this.peers.has(peerId)) {
            // Создание нового соединения с этим участником
            const peerConnection = this.webrtc.createPeerConnection();
            this.peers.set(peerId, peerConnection);
            
            // Добавление нашего потока к новому участнику
            const localStreams = this.webrtc.localStreams;
            for (const [streamId, stream] of localStreams) {
                await peerConnection.addStream(stream);
            }
            
            // Создание и отправка offer
            const offer = await peerConnection.createOffer();
            await peerConnection.setLocalDescription(offer);
            
            this.signaling.sendOffer(peerId, offer);
        }
    }
    
    handlePeerLeft(peerId) {
        const peerConnection = this.peers.get(peerId);
        if (peerConnection) {
            peerConnection.close();
            this.peers.delete(peerId);
        }
        
        // Удаление видео участника из интерфейса
        this.removeRemoteVideo(peerId);
    }
    
    async handleOffer(data) {
        const peerConnection = this.peers.get(data.from) || 
                              this.webrtc.createPeerConnection();
        
        this.peers.set(data.from, peerConnection);
        
        await peerConnection.setRemoteDescription(data.sdp);
        
        const answer = await peerConnection.createAnswer();
        await peerConnection.setLocalDescription(answer);
        
        this.signaling.sendAnswer(data.from, answer);
    }
    
    async handleAnswer(data) {
        const peerConnection = this.peers.get(data.from);
        if (peerConnection) {
            await peerConnection.setRemoteDescription(data.sdp);
        }
    }
    
    async handleIceCandidate(data) {
        const peerConnection = this.peers.get(data.from);
        if (peerConnection) {
            await peerConnection.addIceCandidate(data.candidate);
        }
    }
    
    displayLocalVideo(stream) {
        const video = document.getElementById('local-video') || document.createElement('video');
        video.id = 'local-video';
        video.autoplay = true;
        video.muted = true;
        video.playsInline = true;
        video.srcObject = stream;
        
        if (!document.getElementById('local-video')) {
            document.body.appendChild(video);
        }
    }
    
    displayRemoteVideo(track, stream) {
        const videoId = `remote-video-${track.id}`;
        let video = document.getElementById(videoId);
        
        if (!video) {
            video = document.createElement('video');
            video.id = videoId;
            video.autoplay = true;
            video.playsInline = true;
            document.body.appendChild(video);
        }
        
        if (video.srcObject !== stream) {
            video.srcObject = stream;
        }
    }
    
    removeRemoteVideo(peerId) {
        const video = document.getElementById(`remote-video-${peerId}`);
        if (video) {
            video.remove();
        }
    }
    
    getTargetPeerId() {
        // В простом случае возвращаем первого попавшегося участника
        // В реальном приложении логика будет сложнее
        return this.peers.keys().next().value;
    }
}
```

## STUN и TURN серверы

STUN (Session Traversal Utilities for NAT) и TURN (Traversal Using Relays around NAT) серверы помогают преодолевать NAT и брандмауэры для установки WebRTC соединений.

### Конфигурация STUN/TURN

```javascript
// Класс для работы с ICE серверами
class ICEServerManager {
    constructor() {
        this.defaultSTUN = [
            { urls: 'stun:stun.l.google.com:19302' },
            { urls: 'stun:stun1.l.google.com:19302' },
            { urls: 'stun:stun2.l.google.com:19302' }
        ];
        
        this.turnServers = [];
        this.stunServers = this.defaultSTUN;
        this.selectedServers = [];
    }
    
    // Добавление TURN сервера
    addTURNServer(urls, username, credential, credentialType = 'password') {
        const turnServer = {
            urls: urls,
            username: username,
            credential: credential,
            credentialType: credentialType
        };
        
        this.turnServers.push(turnServer);
        return turnServer;
    }
    
    // Добавление STUN сервера
    addSTUNServer(urls) {
        const stunServer = { urls: urls };
        this.stunServers.push(stunServer);
        return stunServer;
    }
    
    // Получение конфигурации серверов
    getConfiguration(options = {}) {
        const config = {
            iceServers: [
                ...this.stunServers,
                ...this.turnServers
            ],
            iceTransportPolicy: options.transportPolicy || 'all',
            bundlePolicy: options.bundlePolicy || 'balanced',
            rtcpMuxPolicy: options.rtcpMuxPolicy || 'require'
        };
        
        // Фильтрация серверов по типу
        if (options.stunOnly) {
            config.iceServers = this.stunServers;
        } else if (options.turnOnly) {
            config.iceServers = this.turnServers;
        }
        
        return config;
    }
    
    // Тестирование доступности серверов
    async testServerAvailability(servers = null) {
        const testServers = servers || [...this.stunServers, ...this.turnServers];
        const results = [];
        
        for (const server of testServers) {
            const result = await this.testSingleServer(server);
            results.push({
                server: server,
                available: result.available,
                responseTime: result.responseTime,
                error: result.error
            });
        }
        
        return results;
    }
    
    async testSingleServer(server) {
        return new Promise((resolve) => {
            const pc = new RTCPeerConnection({ iceServers: [server] });
            const start = Date.now();
            
            const timer = setTimeout(() => {
                pc.close();
                resolve({
                    available: false,
                    responseTime: null,
                    error: 'Таймаут тестирования'
                });
            }, 5000); // 5 секунд таймаут
            
            pc.onicecandidate = (event) => {
                if (event.candidate) {
                    clearTimeout(timer);
                    pc.close();
                    resolve({
                        available: true,
                        responseTime: Date.now() - start,
                        error: null
                    });
                }
            };
            
            pc.createDataChannel('test');
            pc.createOffer()
                .then(offer => pc.setLocalDescription(offer))
                .catch(() => {
                    clearTimeout(timer);
                    pc.close();
                    resolve({
                        available: false,
                        responseTime: null,
                        error: 'Ошибка создания offer'
                    });
                });
        });
    }
    
    // Выбор оптимальных серверов
    async selectOptimalServers() {
        const testResults = await this.testServerAvailability();
        
        // Сортировка по времени отклика
        const sortedResults = testResults
            .filter(result => result.available)
            .sort((a, b) => (a.responseTime || Infinity) - (b.responseTime || Infinity));
        
        // Выбор топ-N серверов
        const optimalServers = sortedResults.slice(0, 5).map(result => result.server);
        
        this.selectedServers = optimalServers;
        return optimalServers;
    }
    
    // Получение списка доступных серверов
    getAvailableServers() {
        return {
            stun: this.stunServers,
            turn: this.turnServers,
            selected: this.selectedServers
        };
    }
    
    // Обновление конфигурации в реальном времени
    async updateConfiguration(pc, options = {}) {
        const newConfig = this.getConfiguration(options);
        
        // Обновление конфигурации соединения
        try {
            // В некоторых браузерах можно обновить конфигурацию
            if (pc.setConfiguration) {
                pc.setConfiguration(newConfig);
            }
        } catch (error) {
            console.warn('Не удалось обновить конфигурацию:', error);
            // В некоторых случаях нужно пересоздать соединение
        }
    }
}

// Пример использования
const iceManager = new ICEServerManager();

// Добавление своего STUN сервера
iceManager.addSTUNServer('stun:your-stun-server.com:19302');

// Добавление TURN сервера (требуется аутентификация)
iceManager.addTURNServer(
    'turn:your-turn-server.com:3478',
    'username',
    'credential',
    'password'
);

// Получение оптимальной конфигурации
iceManager.selectOptimalServers().then(optimalServers => {
    console.log('Оптимальные серверы:', optimalServers);
    
    // Использование в RTCPeerConnection
    const pc = new RTCPeerConnection({
        iceServers: optimalServers
    });
});
```

## Примеры из промышленной разработки

### Видеоконференция с полной функциональностью

```javascript
// Полнофункциональное приложение видеоконференции
class ProfessionalVideoConference {
    constructor(options = {}) {
        this.options = {
            maxParticipants: 100,
            enableRecording: true,
            enableScreenshare: true,
            enableChat: true,
            enableWhiteboard: true,
            enableFileSharing: true,
            enableBreakoutRooms: true,
            ...options
        };
        
        this.signaling = new SignalingManager();
        this.iceManager = new ICEServerManager();
        this.dataChannelManager = new DataChannelManager(null); // будет установлен позже
        this.participants = new Map();
        this.localMedia = null;
        this.isModerator = false;
        this.roomSettings = {};
        
        this.setupConference();
    }
    
    async setupConference() {
        // Инициализация сигнализации
        await this.signaling.connect(this.options.signalingServer);
        
        // Настройка медиа
        await this.setupMedia();
        
        // Настройка комнаты
        this.setupRoom();
        
        // Настройка обработчиков
        this.setupEventHandlers();
    }
    
    async setupMedia() {
        // Получение основного медиа потока
        this.localMedia = await navigator.mediaDevices.getUserMedia({
            video: {
                width: { ideal: 1280 },
                height: { ideal: 720 },
                frameRate: { ideal: 30 }
            },
            audio: {
                echoCancellation: true,
                noiseSuppression: true,
                autoGainControl: true
            }
        });
        
        // Настройка дополнительных устройств
        await this.setupAdditionalMedia();
    }
    
    async setupAdditionalMedia() {
        // Получение списка устройств
        const devices = await WebRTCBasics.getMediaDevices();
        
        // Сохранение конфигурации устройств
        this.deviceConfig = {
            cameras: devices.cameras,
            microphones: devices.microphones,
            speakers: devices.speakers
        };
    }
    
    setupRoom() {
        // Создание комнаты с настройками
        this.roomSettings = {
            id: this.generateRoomId(),
            name: this.options.roomName || 'Видеоконференция',
            maxParticipants: this.options.maxParticipants,
            recordingEnabled: this.options.enableRecording,
            screenshareEnabled: this.options.enableScreenshare,
            chatEnabled: this.options.enableChat,
            whiteboardEnabled: this.options.enableWhiteboard,
            fileSharingEnabled: this.options.enableFileSharing,
            breakoutRoomsEnabled: this.options.enableBreakoutRooms
        };
    }
    
    setupEventHandlers() {
        // Обработчики сигнализации
        this.signaling.on('roomCreated', (data) => {
            this.handleRoomCreated(data);
        });
        
        this.signaling.on('participantJoined', (data) => {
            this.handleParticipantJoined(data);
        });
        
        this.signaling.on('participantLeft', (data) => {
            this.handleParticipantLeft(data);
        });
        
        this.signaling.on('mediaStreamAdded', (data) => {
            this.handleMediaStreamAdded(data);
        });
        
        this.signaling.on('mediaStreamRemoved', (data) => {
            this.handleMediaStreamRemoved(data);
        });
        
        this.signaling.on('chatMessage', (data) => {
            this.handleChatMessage(data);
        });
        
        this.signaling.on('fileShare', (data) => {
            this.handleFileShare(data);
        });
        
        this.signaling.on('whiteboardUpdate', (data) => {
            this.handleWhiteboardUpdate(data);
        });
        
        this.signaling.on('moderatorAction', (data) => {
            this.handleModeratorAction(data);
        });
    }
    
    handleRoomCreated(data) {
        this.roomSettings = { ...this.roomSettings, ...data.settings };
        this.isModerator = data.isModerator;
        
        // Отображение интерфейса комнаты
        this.displayRoomInterface();
    }
    
    async handleParticipantJoined(participantData) {
        const participant = new Participant(participantData);
        this.participants.set(participant.id, participant);
        
        // Создание соединения с участником
        await this.createPeerConnection(participant.id);
        
        // Обновление интерфейса
        this.updateParticipantsList();
        this.displayParticipant(participant);
        
        // Отправка своего потока новому участнику
        if (this.localMedia) {
            await this.sendLocalStreamToParticipant(participant.id);
        }
    }
    
    handleParticipantLeft(participantId) {
        const participant = this.participants.get(participantId);
        if (participant) {
            // Закрытие соединения
            if (participant.connection) {
                participant.connection.close();
            }
            
            // Удаление из интерфейса
            this.removeParticipantDisplay(participantId);
            
            // Удаление из списка
            this.participants.delete(participantId);
            this.updateParticipantsList();
        }
    }
    
    async handleMediaStreamAdded(streamData) {
        // Добавление удаленного медиа потока
        this.addRemoteStream(streamData);
    }
    
    handleMediaStreamRemoved(streamData) {
        // Удаление удаленного медиа потока
        this.removeRemoteStream(streamData.streamId);
    }
    
    handleChatMessage(chatData) {
        // Отображение сообщения чата
        this.displayChatMessage(chatData);
    }
    
    handleFileShare(fileData) {
        // Обработка совместного использования файлов
        this.handleReceivedFile(fileData);
    }
    
    handleWhiteboardUpdate(whiteboardData) {
        // Обновление интерактивной доски
        this.updateWhiteboard(whiteboardData);
    }
    
    handleModeratorAction(actionData) {
        // Обработка действий модератора
        switch (actionData.action) {
            case 'muteParticipant':
                this.muteParticipant(actionData.participantId);
                break;
            case 'kickParticipant':
                this.kickParticipant(actionData.participantId);
                break;
            case 'lockRoom':
                this.lockRoom(actionData.locked);
                break;
            case 'changeSettings':
                this.updateRoomSettings(actionData.settings);
                break;
        }
    }
    
    async createPeerConnection(participantId) {
        const config = this.iceManager.getConfiguration();
        const pc = new RTCPeerConnection(config);
        
        // Добавление локального потока
        if (this.localMedia) {
            this.localMedia.getTracks().forEach(track => {
                pc.addTrack(track, this.localMedia);
            });
        }
        
        // Настройка data channels
        this.setupDataChannels(pc, participantId);
        
        // Сохранение соединения
        const participant = this.participants.get(participantId);
        if (participant) {
            participant.connection = pc;
            participant.dataChannels = new Map();
        }
        
        this.setupPeerConnectionHandlers(pc, participantId);
    }
    
    setupDataChannels(pc, participantId) {
        // Создание различных каналов данных
        const channels = {
            chat: pc.createDataChannel('chat', { ordered: true }),
            files: pc.createDataChannel('files', { ordered: false, maxRetransmits: 5 }),
            whiteboard: pc.createDataChannel('whiteboard', { ordered: true }),
            control: pc.createDataChannel('control', { ordered: true, maxRetransmits: 3 })
        };
        
        // Настройка каналов
        Object.entries(channels).forEach(([name, channel]) => {
            this.setupDataChannel(channel, participantId, name);
        });
        
        // Сохранение каналов
        const participant = this.participants.get(participantId);
        if (participant) {
            Object.entries(channels).forEach(([name, channel]) => {
                participant.dataChannels.set(name, channel);
            });
        }
    }
    
    setupDataChannel(channel, participantId, channelName) {
        channel.onopen = () => {
            console.log(`Канал ${channelName} открыт для участника ${participantId}`);
        };
        
        channel.onmessage = (event) => {
            this.handleDataChannelMessage(participantId, channelName, event.data);
        };
        
        channel.onerror = (error) => {
            console.error(`Ошибка канала ${channelName} для участника ${participantId}:`, error);
        };
    }
    
    handleDataChannelMessage(participantId, channelName, data) {
        try {
            const parsedData = JSON.parse(data);
            
            switch (channelName) {
                case 'chat':
                    this.handleChatMessage({
                        from: participantId,
                        message: parsedData.message,
                        timestamp: parsedData.timestamp
                    });
                    break;
                case 'files':
                    this.handleFileShare({
                        from: participantId,
                        file: parsedData.file,
                        progress: parsedData.progress
                    });
                    break;
                case 'whiteboard':
                    this.handleWhiteboardUpdate({
                        from: participantId,
                        action: parsedData.action,
                        data: parsedData.data
                    });
                    break;
            }
        } catch (error) {
            console.error('Ошибка парсинга сообщения:', error);
        }
    }
    
    setupPeerConnectionHandlers(pc, participantId) {
        pc.onicecandidate = (event) => {
            if (event.candidate) {
                this.signaling.send({
                    type: 'iceCandidate',
                    to: participantId,
                    candidate: event.candidate
                });
            }
        };
        
        pc.ontrack = (event) => {
            this.handleRemoteTrack(participantId, event.track, event.streams[0]);
        };
        
        pc.ondatachannel = (event) => {
            this.handleIncomingDataChannel(participantId, event.channel);
        };
        
        pc.onconnectionstatechange = () => {
            this.handleConnectionStateChange(participantId, pc.connectionState);
        };
        
        pc.oniceconnectionstatechange = () => {
            this.handleIceConnectionStateChange(participantId, pc.iceConnectionState);
        };
    }
    
    handleRemoteTrack(participantId, track, stream) {
        const participant = this.participants.get(participantId);
        if (participant) {
            if (!participant.remoteStreams) {
                participant.remoteStreams = new Map();
            }
            
            if (stream && !participant.remoteStreams.has(stream.id)) {
                participant.remoteStreams.set(stream.id, stream);
            }
            
            // Отображение трека
            this.displayRemoteTrack(participantId, track, stream);
        }
    }
    
    handleIncomingDataChannel(participantId, channel) {
        // Настройка входящего канала
        this.setupDataChannel(channel, participantId, channel.label);
        
        // Сохранение в участнике
        const participant = this.participants.get(participantId);
        if (participant) {
            participant.dataChannels.set(channel.label, channel);
        }
    }
    
    handleConnectionStateChange(participantId, state) {
        const participant = this.participants.get(participantId);
        if (participant) {
            participant.connectionState = state;
            this.updateParticipantStatus(participantId, state);
        }
        
        // Логика восстановления соединения
        if (state === 'failed' || state === 'disconnected') {
            this.attemptConnectionRecovery(participantId);
        }
    }
    
    handleIceConnectionStateChange(participantId, state) {
        const participant = this.participants.get(participantId);
        if (participant) {
            participant.iceState = state;
        }
    }
    
    // Методы для функций конференции
    async startScreenShare() {
        try {
            const screenStream = await navigator.mediaDevices.getDisplayMedia({
                video: { cursor: 'always' },
                audio: false
            });
            
            // Замена видео трека экраном
            const videoTrack = screenStream.getVideoTracks()[0];
            
            this.participants.forEach(participant => {
                if (participant.connection) {
                    const sender = participant.connection.getSenders()
                        .find(s => s.track && s.track.kind === 'video');
                    
                    if (sender) {
                        sender.replaceTrack(videoTrack);
                    } else {
                        participant.connection.addTrack(videoTrack, screenStream);
                    }
                }
            });
            
            // Остановка предыдущего видео
            if (this.localMedia) {
                const videoTracks = this.localMedia.getVideoTracks();
                videoTracks.forEach(track => track.stop());
            }
            
            this.screenShareActive = true;
            this.currentMediaStream = screenStream;
            
            // Уведомление других участников
            this.signaling.send({
                type: 'screenShareStarted',
                from: this.signaling.peerId
            });
            
        } catch (error) {
            console.error('Ошибка демонстрации экрана:', error);
        }
    }
    
    stopScreenShare() {
        if (this.screenShareActive && this.currentMediaStream) {
            // Возвращение к камере
            const cameraStream = this.localMedia; // предполагается, что локальный медиа сохранен
            
            this.participants.forEach(participant => {
                if (participant.connection) {
                    const screenSender = participant.connection.getSenders()
                        .find(s => s.track && s.track.kind === 'video' && s.track.label.includes('screen'));
                    
                    if (screenSender) {
                        screenSender.replaceTrack(cameraStream.getVideoTracks()[0]);
                    }
                }
            });
            
            // Остановка стрима экрана
            this.currentMediaStream.getTracks().forEach(track => track.stop());
            this.screenShareActive = false;
            
            // Уведомление других участников
            this.signaling.send({
                type: 'screenShareStopped',
                from: this.signaling.peerId
            });
        }
    }
    
    // Методы для чата
    sendChatMessage(message) {
        const chatMessage = {
            type: 'chat',
            from: this.signaling.peerId,
            message: message,
            timestamp: Date.now(),
            isPrivate: false
        };
        
        // Отправка всем участникам через data channels
        this.participants.forEach((participant, id) => {
            const chatChannel = participant.dataChannels.get('chat');
            if (chatChannel && chatChannel.readyState === 'open') {
                chatChannel.send(JSON.stringify(chatMessage));
            }
        });
        
        // Отображение собственного сообщения
        this.displayChatMessage(chatMessage);
    }
    
    sendPrivateMessage(participantId, message) {
        const privateMessage = {
            type: 'privateChat',
            from: this.signaling.peerId,
            to: participantId,
            message: message,
            timestamp: Date.now()
        };
        
        const participant = this.participants.get(participantId);
        if (participant) {
            const chatChannel = participant.dataChannels.get('chat');
            if (chatChannel && chatChannel.readyState === 'open') {
                chatChannel.send(JSON.stringify(privateMessage));
            }
        }
    }
    
    // Методы для совместного использования файлов
    async shareFile(file, toParticipantId = null) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            
            reader.onload = (event) => {
                const fileData = {
                    name: file.name,
                    size: file.size,
                    type: file.type,
                    data: event.target.result,
                    timestamp: Date.now()
                };
                
                if (toParticipantId) {
                    // Приватная отправка
                    const participant = this.participants.get(toParticipantId);
                    if (participant) {
                        const fileChannel = participant.dataChannels.get('files');
                        if (fileChannel && fileChannel.readyState === 'open') {
                            fileChannel.send(JSON.stringify(fileData));
                            resolve({ success: true, to: toParticipantId });
                        } else {
                            reject(new Error('Канал передачи файлов недоступен'));
                        }
                    }
                } else {
                    // Отправка всем
                    this.participants.forEach((participant, id) => {
                        const fileChannel = participant.dataChannels.get('files');
                        if (fileChannel && fileChannel.readyState === 'open') {
                            fileChannel.send(JSON.stringify(fileData));
                        }
                    });
                    resolve({ success: true, to: 'all' });
                }
            };
            
            reader.onerror = () => reject(new Error('Ошибка чтения файла'));
            reader.readAsDataURL(file); // или ArrayBuffer для бинарных данных
        });
    }
    
    // Методы модератора
    muteParticipant(participantId) {
        if (this.isModerator) {
            this.signaling.send({
                type: 'moderatorAction',
                action: 'muteParticipant',
                participantId: participantId,
                by: this.signaling.peerId
            });
        }
    }
    
    kickParticipant(participantId) {
        if (this.isModerator) {
            this.signaling.send({
                type: 'moderatorAction',
                action: 'kickParticipant',
                participantId: participantId,
                by: this.signaling.peerId
            });
        }
    }
    
    lockRoom(locked) {
        if (this.isModerator) {
            this.signaling.send({
                type: 'moderatorAction',
                action: 'lockRoom',
                locked: locked,
                by: this.signaling.peerId
            });
        }
    }
    
    // Вспомогательные методы
    generateRoomId() {
        return 'room_' + Date.now().toString(36) + Math.random().toString(36).substr(2, 5);
    }
    
    displayRoomInterface() {
        // Создание и отображение интерфейса комнаты
        console.log('Интерфейс комнаты отображен');
    }
    
    updateParticipantsList() {
        // Обновление списка участников в интерфейсе
        console.log('Список участников обновлен');
    }
    
    displayParticipant(participant) {
        // Отображение информации об участнике
        console.log('Участник отображен:', participant.id);
    }
    
    displayRemoteTrack(participantId, track, stream) {
        // Отображение удаленного медиа трека
        console.log('Удаленный трек отображен:', participantId, track.kind);
    }
    
    removeParticipantDisplay(participantId) {
        // Удаление отображения участника
        console.log('Участник удален из отображения:', participantId);
    }
    
    displayChatMessage(message) {
        // Отображение сообщения чата
        console.log('Сообщение чата отображено:', message.message);
    }
    
    handleReceivedFile(fileData) {
        // Обработка полученного файла
        console.log('Файл получен:', fileData.name);
    }
    
    updateWhiteboard(data) {
        // Обновление интерактивной доски
        console.log('Доска обновлена');
    }
    
    updateParticipantStatus(participantId, status) {
        // Обновление статуса участника
        console.log(`Статус участника ${participantId}: ${status}`);
    }
    
    async attemptConnectionRecovery(participantId) {
        // Попытка восстановления соединения с участником
        console.log(`Попытка восстановления соединения с ${participantId}`);
        
        const participant = this.participants.get(participantId);
        if (participant) {
            // Закрытие старого соединения
            if (participant.connection) {
                participant.connection.close();
            }
            
            // Создание нового соединения
            await this.createPeerConnection(participantId);
            
            // Повторная отправка локального потока
            if (this.localMedia) {
                await this.sendLocalStreamToParticipant(participantId);
            }
        }
    }
    
    async sendLocalStreamToParticipant(participantId) {
        const participant = this.participants.get(participantId);
        if (participant && this.localMedia) {
            this.localMedia.getTracks().forEach(track => {
                participant.connection.addTrack(track, this.localMedia);
            });
        }
    }
    
    addRemoteStream(streamData) {
        // Добавление удаленного потока
        console.log('Удаленный поток добавлен:', streamData.streamId);
    }
    
    removeRemoteStream(streamId) {
        // Удаление удаленного потока
        console.log('Удаленный поток удален:', streamId);
    }
    
    // Завершение конференции
    endConference() {
        // Закрытие всех соединений
        this.participants.forEach(participant => {
            if (participant.connection) {
                participant.connection.close();
            }
        });
        
        // Остановка локальных потоков
        if (this.localMedia) {
            this.localMedia.getTracks().forEach(track => track.stop());
        }
        
        // Закрытие сигнального соединения
        this.signaling.close();
        
        // Очистка ресурсов
        this.participants.clear();
    }
}

// Класс участника конференции
class Participant {
    constructor(data) {
        this.id = data.id || this.generateId();
        this.name = data.name || 'Аноним';
        this.avatar = data.avatar || null;
        this.role = data.role || 'participant';
        this.connection = null;
        this.dataChannels = new Map();
        this.remoteStreams = new Map();
        this.connectionState = 'new';
        this.iceState = 'new';
        this.joinedAt = data.joinedAt || Date.now();
        this.isAudioMuted = false;
        this.isVideoMuted = false;
        this.isScreenSharing = false;
    }
    
    generateId() {
        return 'participant_' + Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
    
    updateStatus(newState) {
        this.connectionState = newState;
    }
    
    muteAudio() {
        this.isAudioMuted = true;
    }
    
    unmuteAudio() {
        this.isAudioMuted = false;
    }
    
    muteVideo() {
        this.isVideoMuted = true;
    }
    
    unmuteVideo() {
        this.isVideoMuted = false;
    }
    
    startScreenShare() {
        this.isScreenSharing = true;
    }
    
    stopScreenShare() {
        this.isScreenSharing = false;
    }
}

// Инициализация профессиональной видеоконференции
const conference = new ProfessionalVideoConference({
    signalingServer: 'ws://your-signaling-server.com',
    roomName: 'Важная встреча',
    enableRecording: true,
    enableScreenshare: true,
    enableChat: true,
    enableWhiteboard: true,
    enableFileSharing: true,
    enableBreakoutRooms: false,
    maxParticipants: 50
});
```

## Лучшие практики

### 1. Обработка ошибок и восстановление

```javascript
// Лучшие практики для WebRTC
class WebRTCBestPractices {
    // Обработка ошибок подключения
    static handleConnectionError(error, context = '') {
        const errorInfo = {
            message: error.message,
            name: error.name,
            context: context,
            timestamp: Date.now(),
            userAgent: navigator.userAgent
        };
        
        console.error('WebRTC ошибка:', errorInfo);
        
        // Отправка ошибки для аналитики
        this.reportError(errorInfo);
        
        return errorInfo;
    }
    
    // Восстановление соединения
    static async attemptConnectionRecovery(pc, maxRetries = 3) {
        for (let i = 0; i < maxRetries; i++) {
            try {
                // ICE restart
                const offer = await pc.createOffer({ iceRestart: true });
                await pc.setLocalDescription(offer);
                
                // Уведомление другого участника
                // (требует сигнального обмена)
                
                return true;
            } catch (error) {
                console.warn(`Попытка восстановления ${i + 1} не удалась:`, error);
                
                if (i < maxRetries - 1) {
                    await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
                }
            }
        }
        
        return false;
    }
    
    // Адаптивное качество видео
    static adaptVideoQuality(pc, networkConditions) {
        const senders = pc.getSenders().filter(sender => 
            sender.track && sender.track.kind === 'video'
        );
        
        senders.forEach(sender => {
            const parameters = sender.getParameters();
            
            if (networkConditions.bandwidth < 1000) { // < 1 Mbps
                // Понижение качества
                if (parameters.encodings && parameters.encodings[0]) {
                    parameters.encodings[0].maxBitrate = 500 * 1000; // 500 Kbps
                    parameters.encodings[0].scaleResolutionDownBy = 2; // Половина разрешения
                }
            } else if (networkConditions.bandwidth > 5000) { // > 5 Mbps
                // Повышение качества
                if (parameters.encodings && parameters.encodings[0]) {
                    parameters.encodings[0].maxBitrate = 2000 * 1000; // 2 Mbps
                    parameters.encodings[0].scaleResolutionDownBy = 1; // Полное разрешение
                }
            }
            
            sender.setParameters(parameters).catch(console.error);
        });
    }
    
    // Управление буфером DataChannel
    static manageDataChannelBuffer(channel, highWaterMark = 1024 * 1024) { // 1MB
        channel.onbufferedamountlow = () => {
            console.log('Буфер DataChannel ниже порога');
        };
        
        // Проверка буфера перед отправкой
        const sendMessage = (data) => {
            if (channel.bufferedAmount > highWaterMark) {
                console.warn('Буфер DataChannel переполнен, задержка отправки');
                // Можно использовать очередь или отложить отправку
            }
            
            channel.send(data);
        };
        
        return sendMessage;
    }
    
    // Мониторинг производительности
    static async monitorPerformance(pc, callback, interval = 5000) {
        const intervalId = setInterval(async () => {
            try {
                const stats = await pc.getStats();
                const performanceData = this.extractPerformanceData(stats);
                callback(performanceData);
            } catch (error) {
                console.error('Ошибка мониторинга производительности:', error);
            }
        }, interval);
        
        return intervalId;
    }
    
    static extractPerformanceData(stats) {
        const data = {
            timestamp: Date.now(),
            connectionQuality: 'good', // или 'poor', 'bad'
            bandwidth: 0,
            packetLoss: 0,
            jitter: 0,
            roundTripTime: 0
        };
        
        stats.forEach(report => {
            if (report.type === 'inbound-rtp') {
                data.packetLoss = (report.packetsLost / report.packetsReceived) * 100 || 0;
                data.jitter = report.jitter * 1000 || 0; // в миллисекундах
            } else if (report.type === 'candidate-pair' && report.currentRoundTripTime) {
                data.roundTripTime = report.currentRoundTripTime * 1000 || 0; // в миллисекундах
            } else if (report.type === 'transport') {
                data.bandwidth = report.bytesSent + report.bytesReceived;
            }
        });
        
        // Оценка качества соединения
        if (data.packetLoss > 5 || data.jitter > 50 || data.roundTripTime > 200) {
            data.connectionQuality = 'poor';
        } else if (data.packetLoss > 2 || data.jitter > 30 || data.roundTripTime > 100) {
            data.connectionQuality = 'fair';
        }
        
        return data;
    }
    
    // Отчет об ошибках
    static reportError(errorInfo) {
        // В реальном приложении отправка на сервер аналитики
        console.log('Отчет об ошибке:', errorInfo);
    }
}
```

## Безопасность

При работе с WebRTC важно учитывать безопасность:

```javascript
// Безопасность WebRTC
class WebRTCSecurity {
    // Проверка конфиденциальности соединения
    static verifyConnectionPrivacy(pc) {
        // Проверка, что используются зашифрованные транспорты
        const senders = pc.getSenders();
        const receivers = pc.getReceivers();
        
        const hasSecureTransport = [...senders, ...receivers].every(endpoint => {
            // Проверка, что транспорт использует SRTP и т.д.
            return endpoint.transport && endpoint.transport.state === 'connected';
        });
        
        return hasSecureTransport;
    }
    
    // Санитизация данных перед отправкой
    static sanitizeDataBeforeSend(data) {
        if (typeof data === 'string') {
            // Очистка потенциально опасных строк
            return data
                .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
                .replace(/javascript:/gi, '')
                .replace(/vbscript:/gi, '')
                .replace(/on\w+\s*=/gi, '');
        }
        
        if (typeof data === 'object') {
            // Рекурсивная очистка объекта
            return this.deepSanitize(data);
        }
        
        return data;
    }
    
    static deepSanitize(obj) {
        if (obj === null || typeof obj !== 'object') {
            return obj;
        }
        
        if (Array.isArray(obj)) {
            return obj.map(item => this.deepSanitize(item));
        }
        
        const sanitized = {};
        for (const [key, value] of Object.entries(obj)) {
            sanitized[key] = this.deepSanitize(value);
        }
        
        return sanitized;
    }
    
    // Проверка разрешений перед доступом к медиа
    static async checkMediaPermissions(requiredMedia = { audio: true, video: true }) {
        try {
            const stream = await navigator.mediaDevices.getUserMedia(requiredMedia);
            
            // Немедленная остановка треков
            stream.getTracks().forEach(track => track.stop());
            
            return true;
        } catch (error) {
            console.error('Нет разрешения на доступ к медиа:', error);
            return false;
        }
    }
    
    // Безопасная генерация ID
    static generateSecureId() {
        const array = new Uint8Array(16);
        crypto.getRandomValues(array);
        return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
    }
    
    // Проверка сигнального сервера
    static validateSignalingEndpoint(url) {
        try {
            const parsedUrl = new URL(url);
            
            // Проверка, что используется HTTPS/WSS для безопасности
            if (!['https:', 'wss:'].includes(parsedUrl.protocol)) {
                throw new Error('Небезопасный протокол сигнализации');
            }
            
            // Проверка домена (в реальном приложении - доверенные домены)
            const allowedDomains = [
                'your-trusted-domain.com',
                'localhost',
                '127.0.0.1'
            ];
            
            if (!allowedDomains.includes(parsedUrl.hostname)) {
                throw new Error('Недоверенный домен сигнального сервера');
            }
            
            return true;
        } catch (error) {
            console.error('Ошибка валидации сигнального сервера:', error);
            return false;
        }
    }
    
    // Обработка чувствительных данных
    static handleSensitiveInformation(data) {
        // Не отправлять чувствительные данные через WebRTC
        const sensitiveFields = ['password', 'token', 'secret', 'key'];
        
        if (typeof data === 'object') {
            for (const field of sensitiveFields) {
                if (data.hasOwnProperty(field)) {
                    console.warn(`Попытка отправить чувствительное поле: ${field}`);
                    delete data[field];
                }
            }
        }
        
        return data;
    }
}

// Использование безопасных практик
async function secureWebRTCExample() {
    // Проверка разрешений
    const hasPermission = await WebRTCSecurity.checkMediaPermissions();
    if (!hasPermission) {
        throw new Error('Нет разрешения на доступ к камере и микрофону');
    }
    
    // Проверка сигнального сервера
    const isValidEndpoint = WebRTCSecurity.validateSignalingEndpoint('wss://secure-server.com');
    if (!isValidEndpoint) {
        throw new Error('Небезопасный сигнальный сервер');
    }
    
    // Генерация безопасного ID
    const secureId = WebRTCSecurity.generateSecureId();
    console.log('Безопасный ID сгенерирован:', secureId.substring(0, 8) + '...');
}
```

## Теги

#webrtc #realtime #communication #p2p #webcam #microphone #datachannel #signaling #stun #turn #mediastream #rtcpeerconnection