# ERP 챗봇 서비스 REST API 명세서

## 1. API 개요
### 1.1 기본 정보
- 기본 URL: `https://api.erp-chatbot.com/v1`
- 인증 방식: Bearer Token
- 응답 형식: JSON
- 문자 인코딩: UTF-8

### 1.2 공통 응답 형식
```json
{
    "status": "success|error",
    "data": {},
    "message": "응답 메시지",
    "timestamp": "2023-12-01T12:00:00Z"
}
```

### 1.3 공통 에러 코드
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error

## 2. 인증 API
### 2.1 로그인
- **POST** `/auth/login`
```json
Request:
{
    "email": "string",
    "password": "string"
}

Response:
{
    "status": "success",
    "data": {
        "accessToken": "string",
        "refreshToken": "string",
        "expiresIn": 3600
    }
}
```

### 2.2 토큰 갱신
- **POST** `/auth/refresh`
```json
Request:
{
    "refreshToken": "string"
}

Response:
{
    "status": "success",
    "data": {
        "accessToken": "string",
        "expiresIn": 3600
    }
}
```

## 3. 채팅 API
### 3.1 채팅 메시지 전송
- **POST** `/chat/messages`
```json
Request:
{
    "message": "string",
    "contextId": "string"
}

Response:
{
    "status": "success",
    "data": {
        "messageId": "string",
        "response": "string",
        "timestamp": "2023-12-01T12:00:00Z"
    }
}
```

### 3.2 채팅 이력 조회
- **GET** `/chat/history`
```json
Parameters:
- page: integer (default: 1)
- size: integer (default: 20)
- contextId: string

Response:
{
    "status": "success",
    "data": {
        "messages": [
            {
                "messageId": "string",
                "sender": "user|bot",
                "content": "string",
                "timestamp": "2023-12-01T12:00:00Z"
            }
        ],
        "pagination": {
            "currentPage": 1,
            "totalPages": 10,
            "totalItems": 200
        }
    }
}
```

## 4. ERP 데이터 API
### 4.1 사용자 정보 조회
- **GET** `/erp/users/{userId}`
```json
Response:
{
    "status": "success",
    "data": {
        "userId": "string",
        "name": "string",
        "department": "string",
        "position": "string",
        "permissions": ["string"]
    }
}
```

### 4.2 문서 검색
- **GET** `/erp/documents/search`
```json
Parameters:
- query: string
- category: string
- startDate: string (YYYY-MM-DD)
- endDate: string (YYYY-MM-DD)
- page: integer
- size: integer

Response:
{
    "status": "success",
    "data": {
        "documents": [
            {
                "documentId": "string",
                "title": "string",
                "category": "string",
                "createdAt": "2023-12-01T12:00:00Z",
                "summary": "string"
            }
        ],
        "pagination": {
            "currentPage": 1,
            "totalPages": 10,
            "totalItems": 100
        }
    }
}
```

## 5. AI 분석 API
### 5.1 텍스트 분석
- **POST** `/ai/analyze`
```json
Request:
{
    "text": "string",
    "analysisType": "sentiment|intent|entity"
}

Response:
{
    "status": "success",
    "data": {
        "analysisResult": {
            "type": "string",
            "confidence": 0.95,
            "details": {}
        }
    }
}
```

### 5.2 추천 응답 생성
- **POST** `/ai/suggest`
```json
Request:
{
    "context": "string",
    "userQuery": "string"
}

Response:
{
    "status": "success",
    "data": {
        "suggestions": [
            {
                "text": "string",
                "confidence": 0.95,
                "source": "string"
            }
        ]
    }
}
```

## 6. 시스템 관리 API
### 6.1 시스템 상태 확인
- **GET** `/system/health`
```json
Response:
{
    "status": "success",
    "data": {
        "status": "healthy|degraded|down",
        "components": {
            "database": "up|down",
            "cache": "up|down",
            "messageQueue": "up|down"
        },
        "metrics": {
            "uptime": 1000,
            "totalRequests": 1000,
            "activeUsers": 100
        }
    }
}
```

### 6.2 로그 조회
- **GET** `/system/logs`
```json
Parameters:
- level: string (error|warn|info)
- startDate: string
- endDate: string
- page: integer
- size: integer

Response:
{
    "status": "success",
    "data": {
        "logs": [
            {
                "timestamp": "2023-12-01T12:00:00Z",
                "level": "string",
                "message": "string",
                "source": "string"
            }
        ],
        "pagination": {
            "currentPage": 1,
            "totalPages": 10,
            "totalItems": 100
        }
    }
}
```

## 7. 버전 관리
- API 버전은 URL의 `/v1`과 같은 형식으로 관리
- 하위 호환성을 보장하며 주요 변경사항 발생 시 버전 업데이트

## 8. 보안 정책
- 모든 API 요청은 HTTPS를 통해 이루어짐
- 인증이 필요한 엔드포인트는 유효한 JWT 토큰 필요
- Rate Limiting: IP당 1000회/시간
- API 키는 주기적으로 갱신 필요

## 9. AI 모델 관리 API
### 9.1 모델 버전 관리
- **GET** `/ai/models`
```json
Response:
{
    "status": "success",
    "data": {
        "models": [
            {
                "modelId": "string",
                "version": "string",
                "type": "intent|response|embedding",
                "status": "active|training|deprecated",
                "metrics": {
                    "accuracy": 0.95,
                    "latency": 100,
                    "lastUpdated": "2024-01-01T00:00:00Z"
                }
            }
        ]
    }
}
```

### 9.2 모델 학습 요청
- **POST** `/ai/models/train`
```json
Request:
{
    "modelId": "string",
    "trainingData": {
        "source": "file|database",
        "dataPath": "string",
        "parameters": {
            "epochs": 10,
            "batchSize": 32,
            "learningRate": 0.001
        }
    }
}

Response:
{
    "status": "success",
    "data": {
        "trainingId": "string",
        "estimatedTime": 3600,
        "status": "queued|running|completed|failed"
    }
}
```

## 10. RAG 시스템 API
### 10.1 문서 인덱싱
- **POST** `/rag/documents/index`
```json
Request:
{
    "documents": [
        {
            "id": "string",
            "content": "string",
            "metadata": {
                "source": "string",
                "category": "string",
                "tags": ["string"]
            }
        }
    ]
}

Response:
{
    "status": "success",
    "data": {
        "indexedCount": 10,
        "failedCount": 0,
        "vectorIds": ["string"]
    }
}
```

### 10.2 컨텍스트 검색
- **POST** `/rag/search`
```json
Request:
{
    "query": "string",
    "filters": {
        "category": "string",
        "dateRange": {
            "start": "2024-01-01",
            "end": "2024-01-31"
        }
    },
    "limit": 5
}

Response:
{
    "status": "success",
    "data": {
        "results": [
            {
                "documentId": "string",
                "content": "string",
                "relevanceScore": 0.95,
                "metadata": {}
            }
        ]
    }
}
```