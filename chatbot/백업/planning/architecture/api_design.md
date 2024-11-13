# API 설계서

## 1. API 개요

### 1.1 기본 정보
- Base URL
- 인증 방식
- 응답 형식

### 1.2 공통 사항
- 에러 코드
- 상태 코드
- 헤더 정보

## 2. API 엔드포인트

### 2.1 챗봇 API
- POST /api/chat/message
- GET /api/chat/history
- DELETE /api/chat/history

### 2.2 데이터 API
- GET /api/data/search
- POST /api/data/update
- GET /api/data/analyze 