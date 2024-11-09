# API 운영 매뉴얼

## 1. API 서버 구성

### 1.1 서버 환경
```yaml
하드웨어:
  CPU: Intel Xeon Scalable
  GPU: NVIDIA RTX 4000 Ada × 1 (device=3)
  Memory: 16GB
  Storage: 100GB

소프트웨어:
  - Spring Boot 3.2
  - JDK 17
  - Docker
  - Nginx (리버스 프록시)
```

### 1.2 컨테이너 구성
```yaml
API 컨테이너:
  api-server:
    image: openjdk:17-slim
    memory: 16GB
    gpu: "device=3"
    ports:
      - "8080:8080"
    volumes:
      - api_logs:/var/log/api
      - api_config:/etc/api
```

## 2. API 모니터링

### 2.1 성능 모니터링
```python
class APIMonitor:
    def __init__(self):
        self.metrics = MetricsCollector()
        self.alert = AlertManager()
        
    async def monitor_performance(self):
        # 1. 메트릭 수집
        metrics = await self.metrics.collect([
            'response_time',
            'error_rate',
            'request_count',
            'active_sessions'
        ])
        
        # 2. 성능 분석
        if metrics.response_time > 2.0:  # 2초 초과
            await self.alert.send_alert(
                level='warning',
                message='API 응답 지연 발생'
            )
            
        if metrics.error_rate > 0.01:  # 1% 초과
            await self.alert.send_alert(
                level='critical',
                message='높은 에러율 감지'
            )
```

### 2.2 로그 모니터링
```yaml
로그 설정:
  수집 대상:
    - API 액세스 로그
    - 에러 로그
    - 성능 로그
    - 보안 감사 로그
    
  보관 기간:
    - 실시간 로그: 7일
    - 중요 로그: 1년
    - 감사 로그: 5년
    
  알림 설정:
    - 에러 발생 시
    - 성능 저하 시
    - 보안 위반 시
```

## 3. API 보안

### 3.1 인증/인가
```python
class APISecurityManager:
    def __init__(self):
        self.auth_provider = KeycloakProvider()
        self.token_validator = JWTValidator()
        
    async def authenticate_request(self, request: Request):
        # 1. 토큰 검증
        token = self.extract_token(request)
        if not await self.token_validator.validate(token):
            raise UnauthorizedError()
            
        # 2. 권한 확인
        permissions = await self.auth_provider.get_permissions(token)
        if not self.check_permissions(permissions, request.resource):
            raise ForbiddenError()
            
        # 3. 접근 로깅
        await self.log_access(request, token)
```

### 3.2 속도 제한
```python
class RateLimiter:
    def __init__(self):
        self.redis = Redis()
        self.config = {
            'default': {'rate': 100, 'per': 60},  # 분당 100회
            'premium': {'rate': 1000, 'per': 60}  # 분당 1000회
        }
        
    async def check_rate_limit(self, user_id: str, tier: str):
        limit = self.config.get(tier, self.config['default'])
        key = f"rate_limit:{user_id}"
        
        # 1. 현재 사용량 확인
        current = await self.redis.get(key) or 0
        
        # 2. 제한 확인
        if current >= limit['rate']:
            raise RateLimitExceededError()
            
        # 3. 사용량 증가
        await self.redis.incr(key)
        await self.redis.expire(key, limit['per'])
```

## 4. API 최적화

### 4.1 캐시 관리
```python
class APICache:
    def __init__(self):
        self.redis = Redis()
        self.cache_config = {
            'default_ttl': 300,  # 5분
            'max_size': 10000,
            'strategies': {
                'list': 'LRU',
                'detail': 'LFU'
            }
        }
        
    async def get_cached_response(self, key: str):
        # 1. 캐시 확인
        cached = await self.redis.get(key)
        if cached:
            return json.loads(cached)
            
        # 2. 캐시 미스 처리
        response = await self.fetch_data(key)
        await self.cache_response(key, response)
        
        return response
```

### 4.2 성능 최적화
```yaml
성능 최적화:
  Connection Pool:
    - 최소 연결: 10
    - 최대 연결: 100
    - 유휴 타임아웃: 300초
    
  Thread Pool:
    - 코어 크기: 16
    - 최대 크기: 32
    - 큐 크기: 100
    
  Batch Processing:
    - 배치 크기: 100
    - 처리 간격: 1초
    - 최대 지연: 5초
```

## 5. 장애 대응

### 5.1 장애 감지
```python
class APIHealthCheck:
    def __init__(self):
        self.health_checks = {
            'database': self.check_database,
            'cache': self.check_cache,
            'external': self.check_external_services
        }
        
    async def check_health(self):
        results = {}
        for service, check in self.health_checks.items():
            try:
                status = await check()
                results[service] = status
            except Exception as e:
                results[service] = {
                    'status': 'down',
                    'error': str(e)
                }
                
        return results
```

### 5.2 장애 복구
```yaml
복구 절차:
  서비스 재시작:
    1. 활성 요청 완료 대기
    2. 신규 요청 거부
    3. 서비스 재시작
    4. 상태 확인
    
  데이터 복구:
    1. 트랜잭션 롤백
    2. 캐시 초기화
    3. 데이터 정합성 검증
    
  서비스 정상화:
    1. 상태 모니터링
    2. 성능 확인
    3. 트래픽 복원
```

## 6. 용어 설명

### 6.1 API 용어
- **엔드포인트**: API가 제공하는 특정 기능의 접근 지점
- **페이로드**: API 요청/응답에 포함되는 데이터
- **미들웨어**: API 요청/응답을 처리하는 중간 계층

### 6.2 보안 용어
- **JWT**: JSON Web Token, 인증 정보를 담은 토큰
- **CORS**: Cross-Origin Resource Sharing, 교차 출처 리소스 공유
- **Rate Limiting**: API 호출 횟수 제한 기능