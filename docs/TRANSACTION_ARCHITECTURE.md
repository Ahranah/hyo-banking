# 트랜젝션 처리 아키텍처 상세 설명

## 목차
1. [개요](#개요)
2. [트랜젝션 처리 흐름](#트랜젝션-처리-흐름)
3. [데이터 모델](#데이터-모델)
4. [동시성 제어 전략](#동시성-제어-전략)
5. [보안 및 검증](#보안-및-검증)
6. [성능 최적화](#성능-최적화)
7. [에러 처리](#에러-처리)
8. [모니터링 및 로깅](#모니터링-및-로깅)

---

## 개요

원터치 효(孝)뱅킹의 트랜젝션 처리는 다음 핵심 원칙을 따릅니다:

### 설계 원칙
- **ACID 보장**: 모든 금융 거래는 원자성, 일관성, 격리성, 지속성을 보장
- **멱등성(Idempotency)**: 동일한 요청을 여러 번 처리해도 결과가 같음
- **보안 우선**: 모든 단계에서 인증/인가 검증
- **감사 가능성**: 모든 트랜젝션은 추적 가능하고 감사 가능

### 트랜젝션 유형
1. **출금 (Withdraw)**: 계좌에서 현금 인출
2. **입금 (Deposit)**: 계좌에 현금 입금
3. **이체 (Transfer)**: 계좌 간 자금 이동
4. **잔액 조회 (Balance Check)**: 계좌 잔액 확인

---

## 트랜젝션 처리 흐름

### 전체 프로세스

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ 1. 매크로    │────>│ 2. QR 생성    │────>│ 3. QR 검증    │────>│ 4. 거래 실행  │
│    생성      │     │                │     │               │     │              │
└─────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
   (Mobile)             (Mobile)              (ATM)              (ATM)

각 단계별 상세:

1단계: 매크로 생성
  - 사용자가 모바일에서 거래 내용 입력
  - 클라이언트 측 유효성 검증
  - 서버로 전송 및 저장
  - 매크로 ID 반환

2단계: QR 생성
  - 매크로 ID로 QR 요청
  - JWT 토큰 생성 (일회용, 3분 유효)
  - QR 이미지 생성
  - 모바일에 QR 표시

3단계: QR 검증
  - ATM에서 QR 스캔
  - JWT 토큰 검증
  - QR 일회성 확인
  - 매크로 정보 반환

4단계: 거래 실행
  - 매크로의 트랜젝션들을 순서대로 실행
  - 각 단계별 검증 (잔액, 한도 등)
  - 하나라도 실패 시 전체 롤백
  - 성공 시 완료 처리
```

### 1단계: 매크로 생성

#### API 명세
```
POST /api/macros
Authorization: Bearer {accessToken}
Content-Type: application/json

Request Body:
{
  "accountFrom": "123-456-789012",
  "transactions": [
    {
      "type": "withdraw",
      "amount": 100000,
      "order": 1
    },
    {
      "type": "balance_check",
      "order": 2
    }
  ]
}

Response (201 Created):
{
  "macroId": "macro-uuid-123",
  "userId": "user-456",
  "accountFrom": "123-456-789012",
  "status": "CREATED",
  "transactions": [...],
  "createdAt": "2025-12-02T18:00:00Z",
  "expiresAt": "2025-12-02T18:10:00Z"
}
```

#### 처리 로직
1. **인증 확인**: JWT 토큰으로 사용자 인증
2. **입력 검증**: 계좌번호 형식, 금액 범위, 거래 유형 등
3. **권한 확인**: 해당 계좌의 소유자인지 확인
4. **비즈니스 규칙 검증**:
   - 금액은 1,000원 단위
   - 1회 최대 500만원
   - 하루 최대 거래 횟수/금액 확인
5. **매크로 저장**: DB에 매크로 및 트랜젝션 저장
6. **만료 시간 설정**: 생성 시점부터 10분 유효

### 2단계: QR 코드 생성

#### API 명세
```
GET /api/macros/{macroId}/qrcode
Authorization: Bearer {accessToken}

Response (200 OK):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "qrImageBase64": "data:image/png;base64,iVBORw0KGgo...",
  "expiresAt": "2025-12-02T18:03:00Z",
  "macroId": "macro-uuid-123"
}
```

#### 처리 로직
1. **매크로 조회**: 매크로 ID로 DB 조회
2. **권한 확인**: 요청자가 매크로 생성자인지 확인
3. **상태 확인**: 매크로가 CREATED 상태인지 확인
4. **JWT 토큰 생성**:
   ```
   Claims:
   - macroId: 매크로 ID
   - userId: 사용자 ID
   - timestamp: 생성 시각
   - nonce: 일회성 보장을 위한 랜덤 값
   Expiry: 3분
   ```
5. **QR 이미지 생성**: 300x300 PNG 이미지
6. **QR 토큰 저장**: 사용 여부 추적을 위해 DB 저장

### 3단계: QR 검증

#### API 명세
```
POST /api/qr/verify
Content-Type: application/json

Request Body:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response (200 OK):
{
  "macroId": "macro-uuid-123",
  "userId": "user-456",
  "accountFrom": "123-456-789012",
  "transactions": [
    {
      "type": "WITHDRAW",
      "amount": 100000,
      "order": 1
    }
  ]
}
```

#### 처리 로직
1. **분산 락 획득**: 동시 검증 방지 (Redis 사용)
2. **JWT 검증**:
   - 서명 확인
   - 만료 시간 확인
   - Claims 파싱
3. **QR 토큰 조회**: DB에서 토큰 조회
4. **일회성 확인**: 
   - 이미 사용된 토큰인지 확인
   - 낙관적 락으로 동시 사용 방지
5. **매크로 검증**:
   - 매크로 존재 확인
   - 사용자 ID 일치 확인
   - 상태가 CREATED인지 확인
   - 만료 시간 확인
6. **사용 처리**:
   - QR 토큰을 used=true로 변경
   - 매크로 상태를 VERIFIED로 변경
7. **분산 락 해제**

### 4단계: 거래 실행

#### API 명세
```
POST /api/atm/execute
Content-Type: application/json

Request Body:
{
  "macroId": "macro-uuid-123"
}

Response (200 OK):
{
  "macroId": "macro-uuid-123",
  "status": "SUCCESS",
  "results": [
    {
      "transactionId": "tx-789",
      "type": "WITHDRAW",
      "success": true,
      "amount": 100000,
      "balanceBefore": 500000,
      "balanceAfter": 400000,
      "timestamp": "2025-12-02T18:05:00Z"
    }
  ],
  "completedAt": "2025-12-02T18:05:01Z"
}
```

#### 처리 로직
1. **매크로 조회 (비관적 락)**:
   - SELECT ... FOR UPDATE
   - 동시 실행 방지
2. **상태 확인**: VERIFIED 상태인지 확인
3. **상태 변경**: EXECUTING으로 변경
4. **트랜젝션 순차 실행**:
   - order 순서대로 정렬
   - 각 트랜젝션 실행
   - 실패 시 즉시 중단 및 롤백
5. **완료 처리**:
   - 모든 트랜젝션 성공 시 COMPLETED
   - 실패 시 FAILED 및 에러 메시지 저장
6. **감사 로그 기록**
7. **메트릭 수집**

---

## 데이터 모델

### ERD (Entity Relationship Diagram)

```
┌──────────────────┐
│     Macro        │
├──────────────────┤
│ id (PK)          │
│ user_id          │
│ account_from     │
│ status           │
│ created_at       │
│ expires_at       │
│ verified_at      │
│ execution_*_at   │
│ error_message    │
└──────────────────┘
        │
        │ 1
        │
        │ N
        ▼
┌──────────────────────┐
│ MacroTransaction     │
├──────────────────────┤
│ id (PK)              │
│ macro_id (FK)        │
│ type                 │
│ amount               │
│ account_to           │
│ order                │
│ status               │
│ executed_at          │
│ transaction_id       │
│ error_message        │
└──────────────────────┘
        │
        │ 1
        │
        │ 0..1
        ▼
┌──────────────────┐
│  Transaction     │
├──────────────────┤
│ id (PK)          │
│ account_number   │
│ type             │
│ amount           │
│ balance_before   │
│ balance_after    │
│ timestamp        │
│ macro_tx_id      │
│ description      │
└──────────────────┘

┌──────────────────┐
│    QRToken       │
├──────────────────┤
│ id (PK)          │
│ token (UNIQUE)   │
│ macro_id         │
│ created_at       │
│ expires_at       │
│ used             │
│ used_at          │
│ version          │
└──────────────────┘

┌──────────────────┐
│    Account       │
├──────────────────┤
│ id (PK)          │
│ account_number   │
│ user_id          │
│ balance          │
│ daily_limit_*    │
│ version          │
└──────────────────┘
```

### 테이블 상세

#### macros
```sql
CREATE TABLE macros (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    account_from VARCHAR(20) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    verified_at TIMESTAMP,
    execution_started_at TIMESTAMP,
    execution_completed_at TIMESTAMP,
    error_message TEXT,
    INDEX idx_user_status_created (user_id, status, created_at),
    INDEX idx_expires_at (expires_at)
);
```

#### macro_transactions
```sql
CREATE TABLE macro_transactions (
    id VARCHAR(36) PRIMARY KEY,
    macro_id VARCHAR(36) NOT NULL,
    type VARCHAR(20) NOT NULL,
    amount BIGINT,
    account_to VARCHAR(20),
    order_num INT NOT NULL,
    status VARCHAR(20) NOT NULL,
    executed_at TIMESTAMP,
    transaction_id VARCHAR(36),
    error_message TEXT,
    FOREIGN KEY (macro_id) REFERENCES macros(id),
    INDEX idx_macro_order (macro_id, order_num)
);
```

#### qr_tokens
```sql
CREATE TABLE qr_tokens (
    id VARCHAR(36) PRIMARY KEY,
    token TEXT NOT NULL,
    macro_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    used BOOLEAN NOT NULL DEFAULT FALSE,
    used_at TIMESTAMP,
    version BIGINT NOT NULL DEFAULT 0,
    UNIQUE INDEX idx_token_hash (MD5(token)),
    INDEX idx_macro_id (macro_id),
    INDEX idx_used (used, expires_at)
);
```

#### accounts
```sql
CREATE TABLE accounts (
    id VARCHAR(36) PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL UNIQUE,
    user_id VARCHAR(36) NOT NULL,
    balance BIGINT NOT NULL,
    daily_withdrawal_limit BIGINT NOT NULL,
    daily_transfer_limit BIGINT NOT NULL,
    version BIGINT NOT NULL DEFAULT 0,
    INDEX idx_account_number (account_number),
    INDEX idx_user_id (user_id)
);
```

#### transactions
```sql
CREATE TABLE transactions (
    id VARCHAR(36) PRIMARY KEY,
    account_number VARCHAR(20) NOT NULL,
    type VARCHAR(20) NOT NULL,
    amount BIGINT NOT NULL,
    balance_before BIGINT NOT NULL,
    balance_after BIGINT NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    macro_transaction_id VARCHAR(36),
    description TEXT,
    INDEX idx_account_timestamp (account_number, timestamp DESC),
    INDEX idx_macro_tx (macro_transaction_id)
);
```

---

## 동시성 제어 전략

### 1. 비관적 락 (Pessimistic Locking)

#### 사용 위치
- 계좌 잔액 변경 시
- 매크로 실행 시
- 이체 시 양쪽 계좌

#### 구현
```sql
-- JPA에서 자동 생성
SELECT * FROM accounts WHERE account_number = '123-456-789012' FOR UPDATE;
SELECT * FROM macros WHERE id = 'macro-123' FOR UPDATE;
```

#### 장점
- 확실한 데이터 일관성
- 동시 수정 방지

#### 단점
- 성능 오버헤드
- 데드락 가능성

#### 데드락 방지
```
이체 시 두 계좌를 락할 때:
1. 계좌번호를 알파벳 순으로 정렬
2. 항상 같은 순서로 락 획득
3. 타임아웃 설정 (5초)
```

### 2. 낙관적 락 (Optimistic Locking)

#### 사용 위치
- QR 토큰 사용 처리
- 계좌 정보 업데이트

#### 구현
```sql
-- version 컬럼 사용
UPDATE qr_tokens 
SET used = TRUE, version = version + 1
WHERE id = 'qr-123' AND version = 5;

-- 0개 업데이트 → OptimisticLockException
```

#### 장점
- 높은 동시성
- 데드락 없음

#### 단점
- 충돌 시 재시도 필요

### 3. 분산 락 (Distributed Lock)

#### 사용 위치
- QR 검증 (멀티 ATM 환경)

#### 구현 (Redis)
```
Redis SETNX 명령:
SET lock:qr:verify:{token} "1" EX 3 NX

성공 → 락 획득
실패 → 다른 ATM이 처리 중
```

#### 타임아웃
- 락 TTL: 3초
- 락 획득 대기: 0초 (즉시 실패)

---

## 보안 및 검증

### 1. 다층 보안 구조

```
Layer 1: 네트워크
- HTTPS 강제
- CORS 설정
- Rate Limiting

Layer 2: 인증/인가
- JWT 토큰
- 사용자 권한 확인
- 계좌 소유권 확인

Layer 3: 입력 검증
- 형식 검증
- 비즈니스 규칙 검증
- XSS, SQL Injection 방지

Layer 4: 트랜젝션 검증
- 잔액 확인
- 한도 확인
- 이상 거래 탐지

Layer 5: 감사 로그
- 모든 요청 기록
- 보안 이벤트 기록
- 추적 가능성 확보
```

### 2. JWT 토큰 구조

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "macroId": "macro-uuid-123",
    "userId": "user-456",
    "timestamp": 1701540000000,
    "nonce": "random-uuid",
    "exp": 1701540180
  },
  "signature": "..."
}
```

**보안 특징**:
- 3분 짧은 만료 시간
- Nonce로 일회성 보장
- 서명으로 위변조 방지

### 3. 이상 거래 탐지 규칙

| 규칙 ID | 조건 | 심각도 | 조치 |
|---------|------|--------|------|
| HIGH_AMOUNT | 100만원 이상 | MEDIUM | 알림 |
| FREQUENT_TX | 5분 내 5회 이상 | HIGH | 알림 + 차단 검토 |
| UNUSUAL_TIME | 심야 시간 (23:00~06:00) | LOW | 로그 |
| DAILY_LIMIT_90 | 일일 한도 90% 이상 | LOW | 알림 |
| MULTIPLE_FAIL | 5분 내 3회 실패 | HIGH | 계정 잠금 |

---

## 성능 최적화

### 1. 데이터베이스 최적화

#### 인덱스 전략
```sql
-- 매크로 조회 (사용자별, 상태별)
CREATE INDEX idx_user_status_created ON macros(user_id, status, created_at);

-- 거래 내역 조회 (계좌별, 시간순)
CREATE INDEX idx_account_timestamp ON transactions(account_number, timestamp DESC);

-- QR 토큰 조회
CREATE INDEX idx_token_hash ON qr_tokens(MD5(token));
CREATE INDEX idx_used ON qr_tokens(used, expires_at);
```

#### 쿼리 최적화
```java
// N+1 문제 해결: Fetch Join
@Query("SELECT m FROM Macro m " +
       "LEFT JOIN FETCH m.transactions " +
       "WHERE m.id = :id")
Optional<Macro> findByIdWithTransactions(String id);

// Batch Size 설정
@BatchSize(size = 10)
private List<MacroTransaction> transactions;
```

### 2. 캐싱 전략

#### 캐시 대상
- 계좌 정보 (1분 TTL)
- 일일 한도 계산 (10분 TTL)
- 사용자 프로필 (30분 TTL)

#### 구현
```java
@Cacheable(value = "accounts", key = "#accountNumber")
public Account findByNumber(String accountNumber) { ... }

@CacheEvict(value = "accounts", key = "#account.accountNumber")
public void save(Account account) { ... }
```

### 3. 비동기 처리

#### 비동기 대상
- 알림 전송 (SMS, 푸시)
- 감사 로그 기록
- 메트릭 수집

#### 구현
```java
@Async
public CompletableFuture<Void> sendNotification(String userId, String message) {
    // 비동기로 처리
    return CompletableFuture.completedFuture(null);
}
```

### 4. 커넥션 풀 설정

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

---

## 에러 처리

### 예외 계층

```
Exception
└── RuntimeException
    └── BusinessException (추상)
        ├── MacroException
        │   ├── MacroNotFoundException
        │   ├── MacroExpiredException
        │   └── InvalidMacroStatusException
        ├── QRException
        │   ├── QRTokenNotFoundException
        │   ├── QRTokenExpiredException
        │   └── QRTokenAlreadyUsedException
        ├── AccountException
        │   ├── AccountNotFoundException
        │   ├── InsufficientBalanceException
        │   └── DailyLimitExceededException
        └── TransactionException
            ├── TransactionFailedException
            └── UnsupportedTransactionTypeException
```

### 에러 응답 형식

```json
{
  "code": "M002",
  "message": "매크로가 만료되었습니다",
  "timestamp": "2025-12-02T18:00:00Z",
  "path": "/api/macros/123/qrcode",
  "errors": [
    "매크로 유효 기간: 10분"
  ]
}
```

### 재시도 전략

```java
@Retryable(
    value = {TransientDataAccessException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public TransactionResponse executeWithRetry(String macroId) {
    return executionService.executeTransaction(macroId);
}
```

---

## 모니터링 및 로깅

### 1. 로그 레벨

| 레벨 | 용도 | 예시 |
|------|------|------|
| ERROR | 시스템 오류 | DB 연결 실패, 예외 발생 |
| WARN | 비정상이지만 처리 가능 | 일일 한도 근접, QR 만료 |
| INFO | 중요 비즈니스 이벤트 | 거래 시작/완료, QR 검증 |
| DEBUG | 상세 처리 과정 | 각 단계별 상태 |
| TRACE | 매우 상세한 정보 | SQL 쿼리, HTTP 요청 |

### 2. 메트릭

```
거래 관련:
- transaction.executed (Counter)
  - tags: type, success
- transaction.duration (Timer)
  - tags: type
- transaction.amount (Distribution Summary)
  - tags: type

QR 관련:
- qr.generated (Counter)
- qr.verified (Counter)
  - tags: success
- qr.verification.duration (Timer)

계좌 관련:
- account.balance (Gauge)
- daily.withdrawal.used (Gauge)

시스템:
- db.connection.active (Gauge)
- http.server.requests (Timer)
  - tags: uri, method, status
```

### 3. 알림

#### 알림 조건
- 거래 실패율 > 5%
- 평균 응답 시간 > 3초
- DB 커넥션 풀 > 80%
- 이상 거래 탐지 (심각도 HIGH)

#### 알림 채널
- Slack
- Email
- SMS (긴급)

---

## 트랜젝션 시나리오

### 시나리오 1: 정상 출금

```
1. 사용자가 모바일에서 10만원 출금 매크로 생성
   - POST /api/macros
   - 계좌 조회 및 권한 확인
   - 매크로 저장 (CREATED 상태)

2. QR 생성 요청
   - GET /api/macros/{id}/qrcode
   - JWT 토큰 생성 (3분 유효)
   - QR 이미지 생성
   - QR 토큰 저장 (used=false)

3. ATM에서 QR 스캔
   - POST /api/qr/verify
   - JWT 검증
   - QR 토큰 사용 처리 (used=true)
   - 매크로 상태 변경 (VERIFIED)

4. 거래 실행
   - POST /api/atm/execute
   - 매크로 조회 (FOR UPDATE)
   - 계좌 조회 (FOR UPDATE)
   - 잔액 확인: 50만원 > 10만원 ✓
   - 일일 한도 확인: 오늘 20만원 사용, 한도 100만원 ✓
   - 잔액 차감: 50만원 → 40만원
   - 거래 내역 저장
   - 매크로 상태 변경 (COMPLETED)

결과: 성공
```

### 시나리오 2: 잔액 부족

```
1~3. 동일

4. 거래 실행
   - POST /api/atm/execute
   - 매크로 조회 (FOR UPDATE)
   - 계좌 조회 (FOR UPDATE)
   - 잔액 확인: 5만원 < 10만원 ✗
   - InsufficientBalanceException 발생
   - 트랜젝션 롤백
   - 매크로 상태 변경 (FAILED)
   - 에러 메시지 저장: "잔액 부족"

결과: 실패 (잔액 부족)
ATM 화면: "잔액이 부족합니다"
```

### 시나리오 3: QR 재사용 시도

```
1~4. 정상 출금 완료

5. 동일 QR로 재시도
   - POST /api/qr/verify
   - JWT 검증 ✓
   - QR 토큰 조회
   - 사용 여부 확인: used=true ✗
   - QRTokenAlreadyUsedException 발생

결과: 실패 (이미 사용된 QR)
ATM 화면: "이미 사용된 QR 코드입니다"
보안 로그: 재사용 시도 기록
```

### 시나리오 4: 동시 출금 시도

```
Thread 1과 Thread 2가 동시에 같은 계좌에서 각각 30만원 출금
계좌 잔액: 50만원

Thread 1:
- 계좌 조회 (FOR UPDATE) - 락 획득
- 잔액 확인: 50만원 > 30만원 ✓
- 잔액 차감: 50만원 → 20만원
- 저장
- 락 해제

Thread 2:
- 계좌 조회 (FOR UPDATE) - 대기...
- Thread 1 완료 후 락 획득
- 잔액 확인: 20만원 < 30만원 ✗
- InsufficientBalanceException 발생

결과: 
- Thread 1 성공 (30만원 출금)
- Thread 2 실패 (잔액 부족)
- 데이터 일관성 보장
```

---

## 결론

원터치 효(孝)뱅킹의 트랜젝션 처리 시스템은:

✅ **ACID 보장**: Spring @Transactional로 원자성, 일관성, 격리성, 지속성 확보  
✅ **동시성 제어**: 비관적 락, 낙관적 락, 분산 락을 상황에 맞게 활용  
✅ **보안**: 다층 보안 구조와 이상 거래 탐지  
✅ **성능**: 인덱스, 캐싱, 비동기 처리로 최적화  
✅ **관찰성**: 구조화된 로깅과 메트릭으로 모니터링  

이를 통해 **안전하고 빠르며 신뢰할 수 있는** 금융 거래 시스템을 구현합니다.
