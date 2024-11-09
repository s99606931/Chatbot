# API 보안 정책

## 1. 인증 체계

### 1.1 다중 인증(MFA) 구현
```java
@Configuration
public class MFAConfig {
    @Bean
    public MFAProvider mfaProvider() {
        return MFAProvider.builder()
            // 1. TOTP 설정
            .withTOTP()
                .issuer("ERP-Chatbot")
                .digits(6)
                .period(30)
                .algorithm(HmacSha256.class)
            .and()
            // 2. SMS 인증
            .withSMS()
                .provider(twilioSMSProvider)
                .expirySeconds(180)
                .retryLimit(3)
            .and()
            // 3. 생체 인증
            .withBiometric()
                .enabled(true)
                .fallbackToPassword(true)
                .providers(Arrays.asList("fingerprint", "face"))
            .build();
    }
}
```

### 1.2 JWT 토큰 관리
```java
@Service
public class TokenService {
    private final String SECRET_KEY = System.getenv("JWT_SECRET");
    private final long ACCESS_TOKEN_VALIDITY = 3600; // 1시간
    private final long REFRESH_TOKEN_VALIDITY = 604800; // 7일

    public String generateToken(UserDetails user, TokenType type) {
        Date now = new Date();
        long validity = type == TokenType.ACCESS ? ACCESS_TOKEN_VALIDITY : REFRESH_TOKEN_VALIDITY;
        
        return Jwts.builder()
            .setSubject(user.getUsername())
            .setIssuedAt(now)
            .setExpiration(new Date(now.getTime() + validity * 1000))
            .signWith(SignatureAlgorithm.HS512, SECRET_KEY)
            .claim("roles", user.getAuthorities())
            .claim("token_type", type)
            .compact();
    }
}
```

## 2. 접근 제어

### 2.1 RBAC (Role Based Access Control)
```yaml
# security/rbac-config.yml
roles:
  ADMIN:
    permissions:
      - "api:*"
      - "system:*"
      - "user:*"
    
  MANAGER:
    permissions:
      - "api:read"
      - "api:write"
      - "user:read"
    
  USER:
    permissions:
      - "api:read"
      - "user:read:self"

resources:
  api:
    endpoints:
      - path: "/api/v1/admin/**"
        roles: ["ADMIN"]
      - path: "/api/v1/chat/**"
        roles: ["USER", "MANAGER", "ADMIN"]
      - path: "/api/v1/erp/**"
        roles: ["MANAGER", "ADMIN"]
```

### 2.2 Rate Limiting
```java
@Configuration
public class RateLimitConfig {
    @Bean
    public RateLimiter rateLimiter() {
        return RateLimiter.builder()
            // 기본 제한
            .forEndpoint("/**")
                .limit(1000)  // 요청 수
                .per(Duration.ofHours(1))  // 시간 단위
            .and()
            // AI 엔드포인트 제한
            .forEndpoint("/api/v1/ai/**")
                .limit(100)
                .per(Duration.ofMinutes(1))
            .and()
            // IP 기반 제한
            .forIpAddress()
                .limit(5000)
                .per(Duration.ofHours(1))
            .build();
    }
}
```

## 3. 데이터 보안

### 3.1 전송 구간 암호화
```yaml
# application.yml
server:
  ssl:
    enabled: true
    protocol: TLS
    enabled-protocols: TLSv1.3
    ciphers:
      - TLS_AES_128_GCM_SHA256
      - TLS_AES_256_GCM_SHA384
    key-store: classpath:keystore.p12
    key-store-password: ${SSL_KEY_STORE_PASSWORD}
    key-store-type: PKCS12
```

### 3.2 민감 정보 처리
```java
@Service
public class DataMaskingService {
    private static final Map<String, Pattern> PATTERNS = Map.of(
        "ssn", Pattern.compile("\\d{6}-\\d{7}"),
        "card", Pattern.compile("\\d{4}-\\d{4}-\\d{4}-\\d{4}"),
        "phone", Pattern.compile("\\d{3}-\\d{3,4}-\\d{4}")
    );

    public String maskSensitiveData(String content, String type) {
        Pattern pattern = PATTERNS.get(type);
        if (pattern == null) return content;

        return pattern.matcher(content).replaceAll(match -> {
            switch (type) {
                case "ssn":
                    return match.group().replaceAll("(?<=.{6}).(?=\\d{6})", "*");
                case "card":
                    return match.group().replaceAll("(?<=.{6}).(?=.{4})", "*");
                case "phone":
                    return match.group().replaceAll("(?<=.{3}).(?=.{4})", "*");
                default:
                    return match.group();
            }
        });
    }
}
```

## 4. 보안 모니터링

### 4.1 보안 감사 로깅
```java
@Aspect
@Component
public class SecurityAuditLogger {
    @Around("@annotation(Audited)")
    public Object logSecurityAudit(ProceedingJoinPoint joinPoint) throws Throwable {
        SecurityAuditLog auditLog = SecurityAuditLog.builder()
            .timestamp(Instant.now())
            .user(SecurityContextHolder.getContext().getAuthentication().getName())
            .action(joinPoint.getSignature().getName())
            .resource(joinPoint.getTarget().getClass().getSimpleName())
            .ipAddress(RequestContextHolder.currentRequestAttributes().getRemoteAddr())
            .build();

        try {
            Object result = joinPoint.proceed();
            auditLog.setStatus("SUCCESS");
            return result;
        } catch (Exception e) {
            auditLog.setStatus("FAILED");
            auditLog.setErrorMessage(e.getMessage());
            throw e;
        } finally {
            auditLogRepository.save(auditLog);
        }
    }
}
```

### 4.2 이상 탐지
```java
@Service
public class SecurityAnomalyDetector {
    @Scheduled(fixedRate = 60000) // 1분마다 실행
    public void detectAnomalies() {
        // 1. 로그인 실패 패턴 분석
        analyzeLoginFailures();
        
        // 2. 비정상 접근 패턴 탐지
        detectAbnormalAccessPatterns();
        
        // 3. API 호출 패턴 분석
        analyzeAPIUsagePatterns();
    }

    private void analyzeLoginFailures() {
        Map<String, Long> failuresByIp = loginAttemptRepository
            .findFailuresByIpLast10Minutes();
            
        failuresByIp.forEach((ip, count) -> {
            if (count > 5) {
                blockIpAddress(ip, Duration.ofHours(1));
                notifySecurityTeam("Excessive login failures", ip);
            }
        });
    }
}
```

## 5. 용어 설명

### 5.1 인증 용어
- **MFA (Multi-Factor Authentication)**: 다중 인증 방식으로, 두 가지 이상의 인증 요소를 사용
- **TOTP (Time-based One-Time Password)**: 시간 기반 일회용 비밀번호
- **JWT (JSON Web Token)**: JSON 기반의 웹 토큰 인증 방식

### 5.2 보안 용어
- **RBAC (Role Based Access Control)**: 역할 기반 접근 제어
- **Rate Limiting**: API 호출 횟수 제한
- **TLS (Transport Layer Security)**: 전송 계층 보안 프로토콜 