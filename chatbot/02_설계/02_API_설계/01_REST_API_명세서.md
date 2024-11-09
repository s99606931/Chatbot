# ERP 챗봇 REST API 명세서

## 1. API 개요

### 1.1 기본 정보
```yaml
Base URL: https://api.erp-chatbot.com/v1
인증 방식: Bearer Token (JWT)
응답 형식: JSON
문자 인코딩: UTF-8
```

### 1.2 공통 응답 형식
```json
{
    "status": "success|error",
    "data": {},
    "message": "응답 메시지",
    "timestamp": "2024-01-01T00:00:00Z"
}
```

### 1.3 에러 코드
```yaml
HTTP 상태 코드:
  400: Bad Request (잘못된 요청)
  401: Unauthorized (인증 실패)
  403: Forbidden (권한 없음)
  404: Not Found (리소스 없음)
  429: Too Many Requests (요청 횟수 초과)
  500: Internal Server Error (서버 오류)

비즈니스 에러 코드:
  AUTH001: 인증 토큰 만료
  AUTH002: 유효하지 않은 토큰
  CHAT001: 채팅 세션 만료
  CHAT002: 메시지 처리 실패
  ERP001: ERP 시스템 연동 오류
```

## 2. 인증 API

### 2.1 로그인
```http
POST /auth/login
Content-Type: application/json

Request:
{
    "email": "user@company.com",
    "password": "string",
    "mfa_code": "string"  // 선택적
}

Response:
{
    "status": "success",
    "data": {
        "access_token": "string",
        "refresh_token": "string",
        "expires_in": 3600,
        "token_type": "Bearer"
    }
}
```

### 2.2 토큰 갱신
```http
POST /auth/refresh
Authorization: Bearer {refresh_token}

Response:
{
    "status": "success",
    "data": {
        "access_token": "string",
        "expires_in": 3600
    }
}
```

## 3. 채팅 API

### 3.1 채팅 세션 생성
```http
POST /chat/sessions
Authorization: Bearer {token}
Content-Type: application/json

Request:
{
    "title": "string",  // 선택적
    "context": {        // 선택적
        "module": "hr|finance|asset",
        "sub_module": "string"
    }
}

Response:
{
    "status": "success",
    "data": {
        "session_id": "string",
        "title": "string",
        "created_at": "2024-01-01T00:00:00Z"
    }
}
```

### 3.2 메시지 전송
```http
POST /chat/sessions/{session_id}/messages
Authorization: Bearer {token}
Content-Type: application/json

Request:
{
    "content": "string",
    "attachments": [     // 선택적
        {
            "type": "file|image",
            "url": "string"
        }
    ]
}

Response:
{
    "status": "success",
    "data": {
        "message_id": "string",
        "content": "string",
        "created_at": "2024-01-01T00:00:00Z",
        "references": [   // 관련 문서/데이터 참조
            {
                "type": "document|data",
                "title": "string",
                "url": "string"
            }
        ]
    }
}
```

### 3.3 채팅 이력 조회
```http
GET /chat/sessions/{session_id}/messages
Authorization: Bearer {token}

Parameters:
- page: integer (default: 1)
- size: integer (default: 20)
- start_date: string (YYYY-MM-DD)
- end_date: string (YYYY-MM-DD)

Response:
{
    "status": "success",
    "data": {
        "messages": [
            {
                "message_id": "string",
                "sender_type": "user|bot",
                "content": "string",
                "created_at": "2024-01-01T00:00:00Z"
            }
        ],
        "pagination": {
            "current_page": 1,
            "total_pages": 10,
            "total_items": 100
        }
    }
}
```

## 4. ERP 데이터 API

### 4.1 인사 정보 조회
```http
GET /erp/hr/employee/{employee_id}
Authorization: Bearer {token}

Response:
{
    "status": "success",
    "data": {
        "employee": {
            "id": "string",
            "name": "string",
            "department": "string",
            "position": "string",
            "join_date": "2024-01-01",
            "status": "active|inactive"
        }
    }
}
```

### 4.2 급여 정보 조회
```http
GET /erp/payroll/statements
Authorization: Bearer {token}

Parameters:
- year: integer
- month: integer

Response:
{
    "status": "success",
    "data": {
        "statement": {
            "year": 2024,
            "month": 1,
            "base_salary": 5000000,
            "allowances": [
                {
                    "type": "string",
                    "amount": 100000
                }
            ],
            "deductions": [
                {
                    "type": "string",
                    "amount": 50000
                }
            ],
            "net_salary": 5050000
        }
    }
}
```

## 5. AI 분석 API

### 5.1 의도 분석
```http
POST /ai/analyze/intent
Authorization: Bearer {token}
Content-Type: application/json

Request:
{
    "text": "string",
    "context": {  // 선택적
        "previous_messages": [
            {
                "role": "user|bot",
                "content": "string"
            }
        ]
    }
}

Response:
{
    "status": "success",
    "data": {
        "intent": {
            "type": "string",
            "confidence": 0.95,
            "entities": [
                {
                    "type": "string",
                    "value": "string",
                    "position": {
                        "start": 0,
                        "end": 10
                    }
                }
            ]
        }
    }
}
```

### 5.2 문서 검색
```http
POST /ai/search
Authorization: Bearer {token}
Content-Type: application/json

Request:
{
    "query": "string",
    "filters": {  // 선택적
        "document_type": ["manual", "policy", "guide"],
        "date_range": {
            "start": "2024-01-01",
            "end": "2024-12-31"
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
                "document_id": "string",
                "title": "string",
                "content": "string",
                "relevance_score": 0.95,
                "metadata": {
                    "type": "string",
                    "created_at": "2024-01-01T00:00:00Z",
                    "author": "string"
                }
            }
        ]
    }
}
```

## 6. 용어 설명

### 6.1 인증 용어
- **Bearer Token**: JWT 기반의 인증 토큰
- **Refresh Token**: 액세스 토큰 갱신용 토큰
- **MFA (Multi-Factor Authentication)**: 다중 인증

### 6.2 API 용어
- **Endpoint**: API 호출 주소
- **Rate Limiting**: API 호출 횟수 제한
- **Pagination**: 페이지 단위 데이터 조회 