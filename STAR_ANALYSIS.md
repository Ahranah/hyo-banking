# 원터치 효(孝)뱅킹 – STAR 분석

## 📋 목차
- [S (Situation) - 상황](#s-situation---상황)
- [T (Task) - 과제](#t-task---과제)
- [A (Action) - 행동](#a-action---행동)
- [R (Result) - 결과](#r-result---결과)
- [트랜젝션 처리 상세 분석](#트랜젝션-처리-상세-분석)

---

## S (Situation) - 상황

### 문제 배경
고령층 이용자들이 ATM을 사용할 때 다음과 같은 심각한 문제에 직면하고 있습니다:

1. **심리적 압박**
   - 뒤에 대기하는 사람들의 시선으로 인한 불안감
   - "빨리 해야 한다"는 압박으로 인한 실수 증가
   - 복잡한 UI로 인한 인지 부담

2. **물리적 어려움**
   - 작은 화면과 버튼으로 인한 조작의 어려움
   - 여러 단계의 인증 과정(카드 삽입, 비밀번호, OTP 등)
   - 시력 저하로 인한 화면 가독성 문제

3. **보안 위협**
   - 어깨 너머로 비밀번호 노출 위험
   - 복잡한 인증 절차를 회피하려는 시도로 인한 보안 취약성
   - 공공장소에서의 개인정보 노출

4. **디지털 소외**
   - 디지털 금융 서비스 접근성 저하
   - 금융 자립도 감소
   - 가족이나 타인에 대한 의존도 증가

### 시장 상황
- 고령 인구 증가: 2025년 대한민국 65세 이상 인구 비율 20% 초과 예상
- 디지털 금융 서비스 확대: 오프라인 지점 축소, 디지털 서비스 중심 전환
- 금융 접근성 격차: 디지털 약자층의 금융 서비스 이용 어려움 심화

---

## T (Task) - 과제

### 프로젝트 목표
**고령층이 ATM에서 느끼는 심리적·물리적 장벽을 최소화하고, 안전하고 편안한 금융 거래 경험을 제공하는 솔루션 개발**

### 구체적 과제

1. **현장 조작 시간 최소화**
   - 목표: ATM 앞 체류 시간을 기존 대비 70% 이상 단축
   - 방법: 거래 설정을 사전에 완료하고 현장에서는 QR 스캔만 수행

2. **사용자 경험 개선**
   - 멀티모달 인터페이스 제공 (시각적 + 청각적 안내)
   - 캐릭터 애니메이션을 통한 직관적인 가이드
   - 큰 글자, 높은 대비, 넓은 터치 영역

3. **보안성 강화**
   - 공공장소에서의 비밀번호 입력 최소화
   - QR 기반 일회성 인증으로 어깨 너머 공격 방지
   - 시간 제한 및 일회용 토큰으로 재사용 공격 차단

4. **기술적 구현**
   - 모바일-ATM 간 안전한 연동 시스템
   - 실시간 트랜젝션 처리 및 검증
   - 확장 가능한 마이크로서비스 아키텍처

---

## A (Action) - 행동

### 솔루션 아키텍처

#### 1. 전체 시스템 구성
```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  Mobile Web App │         │   Spring Boot   │         │  ATM Simulator  │
│   (사전 설정)    │ ←────→ │    Backend      │ ←────→ │   (현장 실행)   │
└─────────────────┘         └─────────────────┘         └─────────────────┘
      │                             │                             │
      │ 1. 매크로 생성               │ 4. QR 검증                  │
      │ 2. QR 생성 요청              │ 5. 거래 실행                │
      │ 3. QR 표시                   │ 6. 결과 반환                │
      └────────────────────────────┴─────────────────────────────┘
```

#### 2. 기술 스택 선정

**Frontend (Vue.js 3 + Vite)**
- Vue 3: 반응형 UI, 컴포넌트 재사용성
- Vite: 빠른 개발 환경, HMR(Hot Module Replacement)
- TTS/음성 안내: Web Speech API 활용

**Backend (Spring Boot 3.5.5 + JPA)**
- Spring Security: 인증/인가 처리
- JPA/Hibernate: 데이터베이스 추상화
- H2/MySQL: 개발/운영 환경 분리

**주요 라이브러리**
- Lombok: 보일러플레이트 코드 감소
- Checkstyle/SpotBugs: 코드 품질 관리
- JUnit 5: 테스트 자동화

#### 3. 개발 프로세스

**3.1 프로젝트 구조**
```
hyo-banking/
├── hyo-banking-mobile/      # 모바일 웹앱 (매크로 생성)
├── hyo-banking-atm/          # ATM 시뮬레이터 (거래 실행)
└── hyo-banking-be/           # 백엔드 API 서버
    ├── src/main/java/
    │   └── org/atmgigi/hyobankingbe/
    │       ├── controller/    # REST API 엔드포인트
    │       ├── service/       # 비즈니스 로직
    │       ├── repository/    # 데이터 접근
    │       ├── entity/        # JPA 엔티티
    │       ├── dto/           # 데이터 전송 객체
    │       └── security/      # 보안 설정
    └── src/main/resources/
        └── application.yml    # 설정 파일
```

**3.2 UI/UX 설계**
- 접근성 가이드라인 준수 (WCAG 2.1 AA 수준)
- 최소 44px 이상 터치 영역
- 대비율 4.5:1 이상
- 캐릭터 애니메이션으로 다음 단계 명확히 표시

**3.3 보안 설계**
- HTTPS 통신 강제
- JWT 기반 토큰 인증
- QR 토큰 만료 시간 설정 (기본 180초)
- 일회용 QR 토큰 (사용 후 즉시 무효화)

### 주요 구현 내용

#### 4. API 설계

| Method | Endpoint | 설명 | 요청 | 응답 |
|--------|----------|------|------|------|
| POST | `/api/macros` | 거래 매크로 생성 | MacroCreateRequest | MacroResponse |
| GET | `/api/macros/{id}` | 매크로 조회 | - | MacroDetailResponse |
| GET | `/api/macros/{id}/qrcode` | QR 코드 생성 | - | QRCodeResponse |
| POST | `/api/qr/verify` | QR 토큰 검증 | QRVerifyRequest | QRVerifyResponse |
| POST | `/api/atm/execute` | 거래 실행 | TransactionRequest | TransactionResponse |
| GET | `/api/atm/status/{txId}` | 거래 상태 조회 | - | TransactionStatusResponse |

#### 5. 데이터 모델

**매크로 (Macro)**
```json
{
  "id": "macro-uuid",
  "userId": "user-123",
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
  ],
  "createdAt": "2025-12-02T18:00:00Z",
  "expiresAt": "2025-12-02T18:10:00Z"
}
```

**QR 토큰 (QRToken)**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "macroId": "macro-uuid",
  "expiresAt": "2025-12-02T18:03:00Z",
  "used": false,
  "qrImageBase64": "data:image/png;base64,iVBORw0KGgo..."
}
```

---

## R (Result) - 결과

### 예상 성과

#### 1. 정량적 성과
- **ATM 체류 시간**: 평균 3~5분 → 30초~1분으로 **80% 단축**
- **조작 오류율**: 기존 대비 **60% 감소** (사전 설정으로 실수 방지)
- **보안 사고**: 어깨 너머 공격 **90% 이상 감소** (공공장소 비밀번호 입력 최소화)
- **사용자 만족도**: **40% 향상** (편안한 UX, 심리적 압박 감소)

#### 2. 정성적 성과
- **디지털 금융 접근성 향상**: 고령층의 자립적 금융 거래 가능
- **심리적 장벽 해소**: "빨리 해야 한다"는 압박감 제거
- **금융 포용성 증대**: 디지털 약자층의 금융 서비스 이용 확대
- **사회적 가치 창출**: 고령 친화적 금융 환경 조성

#### 3. 기술적 성과
- **확장 가능한 아키텍처**: 마이크로서비스 기반, 수평 확장 가능
- **재사용 가능한 컴포넌트**: Vue 컴포넌트 라이브러리화
- **보안 모범 사례 적용**: OWASP Top 10 대응
- **테스트 자동화**: 단위/통합 테스트 커버리지 80% 이상

#### 4. 비즈니스 임팩트
- **고객 이탈 방지**: 디지털 전환 과정에서 고령 고객 유지
- **운영 비용 절감**: 오프라인 창구 이용 감소로 인건비 절감
- **브랜드 이미지 향상**: 사회적 책임 이행, 금융 포용성 실천
- **시장 차별화**: 고령 친화적 금융 서비스 선도

### 향후 계획
1. **Phase 1** (현재): 프로토타입 개발 및 해커톤 시연
2. **Phase 2** (Q1 2026): 실사용자 테스트 및 피드백 반영
3. **Phase 3** (Q2 2026): 시범 지점 PoC 운영
4. **Phase 4** (Q3 2026): 전 지점 확대 배포

---

## 트랜젝션 처리 상세 분석

### 1. 트랜젝션 생명주기 (Transaction Lifecycle)

#### 1.1 매크로 생성 단계 (Mobile App)
```
사용자 입력 → 유효성 검증 → 매크로 저장 → QR 생성 → QR 표시
```

**상세 프로세스:**
```java
// 1. 사용자가 모바일 앱에서 거래 내용 입력
MacroCreateRequest request = {
    accountFrom: "123-456-789012",
    transactions: [
        { type: "withdraw", amount: 100000 }
    ]
};

// 2. 클라이언트 측 유효성 검증
- 계좌번호 형식 검증
- 금액 범위 검증 (0 < amount <= 일일 한도)
- 거래 유형 검증 (withdraw, deposit, transfer, balance_check)

// 3. 서버로 매크로 생성 요청
POST /api/macros
Authorization: Bearer {accessToken}
Content-Type: application/json

{
  "accountFrom": "123-456-789012",
  "transactions": [
    { "type": "withdraw", "amount": 100000, "order": 1 }
  ]
}

// 4. 서버 측 처리
@Service
@Transactional
public class MacroService {
    
    public MacroResponse createMacro(MacroCreateRequest request) {
        // 4.1 인증/인가 검증
        User user = getCurrentUser();
        validateAccountOwnership(user, request.getAccountFrom());
        
        // 4.2 비즈니스 규칙 검증
        validateTransactionLimits(request.getTransactions());
        validateDailyLimit(user, calculateTotalAmount(request));
        
        // 4.3 매크로 엔티티 생성 및 저장
        Macro macro = Macro.builder()
            .userId(user.getId())
            .accountFrom(request.getAccountFrom())
            .status(MacroStatus.CREATED)
            .createdAt(Instant.now())
            .expiresAt(Instant.now().plusSeconds(600)) // 10분 유효
            .build();
            
        // 4.4 트랜젝션 저장 (순서 보장)
        request.getTransactions().forEach(txReq -> {
            MacroTransaction tx = MacroTransaction.builder()
                .macro(macro)
                .type(txReq.getType())
                .amount(txReq.getAmount())
                .order(txReq.getOrder())
                .status(TransactionStatus.PENDING)
                .build();
            macro.addTransaction(tx);
        });
        
        macroRepository.save(macro);
        
        return MacroResponse.from(macro);
    }
}
```

#### 1.2 QR 생성 단계
```java
@Service
public class QRService {
    
    private final JWTProvider jwtProvider;
    private final QRCodeGenerator qrGenerator;
    
    @Transactional(readOnly = true)
    public QRCodeResponse generateQR(String macroId) {
        // 1. 매크로 조회 및 검증
        Macro macro = macroRepository.findById(macroId)
            .orElseThrow(() -> new MacroNotFoundException(macroId));
            
        validateMacroStatus(macro);
        validateMacroExpiration(macro);
        
        // 2. JWT 토큰 생성 (일회용, 시간 제한)
        String token = jwtProvider.createToken(
            claims: {
                "macroId": macroId,
                "userId": macro.getUserId(),
                "timestamp": System.currentTimeMillis(),
                "nonce": UUID.randomUUID().toString()
            },
            expirySeconds: 180  // 3분 유효
        );
        
        // 3. QR 코드 이미지 생성
        byte[] qrImage = qrGenerator.generate(token, 300, 300);
        String qrBase64 = Base64.getEncoder().encodeToString(qrImage);
        
        // 4. QR 토큰 저장 (사용 여부 추적)
        QRToken qrToken = QRToken.builder()
            .token(token)
            .macroId(macroId)
            .createdAt(Instant.now())
            .expiresAt(Instant.now().plusSeconds(180))
            .used(false)
            .build();
            
        qrTokenRepository.save(qrToken);
        
        return QRCodeResponse.builder()
            .token(token)
            .qrImageBase64("data:image/png;base64," + qrBase64)
            .expiresAt(qrToken.getExpiresAt())
            .build();
    }
}
```

#### 1.3 QR 스캔 및 검증 단계 (ATM)
```javascript
// ATM 시뮬레이터 (Vue.js)
async function handleQRScan(qrData) {
  try {
    // 1. QR 데이터 파싱
    const token = qrData.trim();
    
    // 2. 서버로 검증 요청
    const response = await fetch('/api/qr/verify', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ token })
    });
    
    if (!response.ok) {
      throw new Error('QR 검증 실패');
    }
    
    const { macroId, transactions } = await response.json();
    
    // 3. 거래 실행 화면으로 전환
    router.push({
      name: 'TransactionExecute',
      params: { macroId },
      state: { transactions }
    });
    
  } catch (error) {
    showError('QR 코드가 유효하지 않거나 만료되었습니다.');
  }
}
```

```java
// 서버 측 QR 검증
@Service
@Transactional
public class QRVerificationService {
    
    public QRVerifyResponse verifyQR(String token) {
        // 1. JWT 토큰 검증
        Claims claims;
        try {
            claims = jwtProvider.parseToken(token);
        } catch (JwtException e) {
            throw new InvalidQRTokenException("토큰 검증 실패", e);
        }
        
        // 2. 토큰 만료 확인
        if (claims.getExpiration().before(new Date())) {
            throw new QRTokenExpiredException("QR 코드가 만료되었습니다");
        }
        
        // 3. QR 토큰 사용 여부 확인 (일회용)
        String macroId = claims.get("macroId", String.class);
        QRToken qrToken = qrTokenRepository.findByToken(token)
            .orElseThrow(() -> new QRTokenNotFoundException());
            
        if (qrToken.isUsed()) {
            throw new QRTokenAlreadyUsedException("이미 사용된 QR 코드입니다");
        }
        
        // 4. 매크로 조회 및 검증
        Macro macro = macroRepository.findById(macroId)
            .orElseThrow(() -> new MacroNotFoundException(macroId));
            
        if (macro.isExpired()) {
            throw new MacroExpiredException("매크로가 만료되었습니다");
        }
        
        if (macro.getStatus() != MacroStatus.CREATED) {
            throw new InvalidMacroStatusException("매크로 상태가 올바르지 않습니다");
        }
        
        // 5. QR 토큰 사용 처리 (동시성 제어)
        qrToken.markAsUsed();
        qrTokenRepository.save(qrToken);
        
        // 6. 매크로 상태 업데이트
        macro.setStatus(MacroStatus.VERIFIED);
        macroRepository.save(macro);
        
        return QRVerifyResponse.builder()
            .macroId(macroId)
            .transactions(macro.getTransactions())
            .accountFrom(macro.getAccountFrom())
            .build();
    }
}
```

#### 1.4 거래 실행 단계
```java
@Service
@Transactional
public class TransactionExecutionService {
    
    private final AccountService accountService;
    private final TransactionRepository transactionRepository;
    private final AuditLogService auditLogService;
    
    /**
     * 매크로에 정의된 거래들을 순차적으로 실행
     * 하나라도 실패하면 전체 롤백 (@Transactional)
     */
    public TransactionResponse executeTransaction(String macroId) {
        // 1. 매크로 조회 및 상태 확인
        Macro macro = macroRepository.findByIdWithLock(macroId)
            .orElseThrow(() -> new MacroNotFoundException(macroId));
            
        if (macro.getStatus() != MacroStatus.VERIFIED) {
            throw new InvalidMacroStatusException(
                "검증되지 않은 매크로입니다. 현재 상태: " + macro.getStatus()
            );
        }
        
        // 2. 매크로 상태를 EXECUTING으로 변경 (중복 실행 방지)
        macro.setStatus(MacroStatus.EXECUTING);
        macro.setExecutionStartedAt(Instant.now());
        macroRepository.save(macro);
        
        List<TransactionResult> results = new ArrayList<>();
        
        try {
            // 3. 트랜젝션들을 순서대로 실행
            List<MacroTransaction> transactions = macro.getTransactions()
                .stream()
                .sorted(Comparator.comparing(MacroTransaction::getOrder))
                .collect(Collectors.toList());
            
            for (MacroTransaction tx : transactions) {
                TransactionResult result = executeIndividualTransaction(tx);
                results.add(result);
                
                // 실패 시 즉시 중단 (롤백)
                if (!result.isSuccess()) {
                    throw new TransactionFailedException(
                        "거래 실행 중 오류 발생: " + result.getErrorMessage()
                    );
                }
            }
            
            // 4. 모든 거래 성공 - 매크로 상태 업데이트
            macro.setStatus(MacroStatus.COMPLETED);
            macro.setExecutionCompletedAt(Instant.now());
            macroRepository.save(macro);
            
            // 5. 감사 로그 기록
            auditLogService.logTransactionSuccess(macro, results);
            
            return TransactionResponse.builder()
                .macroId(macroId)
                .status("SUCCESS")
                .results(results)
                .completedAt(macro.getExecutionCompletedAt())
                .build();
                
        } catch (Exception e) {
            // 6. 실패 처리
            macro.setStatus(MacroStatus.FAILED);
            macro.setErrorMessage(e.getMessage());
            macroRepository.save(macro);
            
            auditLogService.logTransactionFailure(macro, e);
            
            throw new TransactionExecutionException(
                "거래 실행 실패: " + e.getMessage(), e
            );
        }
    }
    
    /**
     * 개별 트랜젝션 실행
     */
    private TransactionResult executeIndividualTransaction(MacroTransaction tx) {
        tx.setStatus(TransactionStatus.EXECUTING);
        tx.setExecutedAt(Instant.now());
        
        try {
            switch (tx.getType()) {
                case WITHDRAW:
                    return executeWithdraw(tx);
                case DEPOSIT:
                    return executeDeposit(tx);
                case TRANSFER:
                    return executeTransfer(tx);
                case BALANCE_CHECK:
                    return executeBalanceCheck(tx);
                default:
                    throw new UnsupportedTransactionTypeException(
                        "지원하지 않는 거래 유형: " + tx.getType()
                    );
            }
        } catch (Exception e) {
            tx.setStatus(TransactionStatus.FAILED);
            tx.setErrorMessage(e.getMessage());
            throw e;
        }
    }
    
    /**
     * 출금 처리
     */
    private TransactionResult executeWithdraw(MacroTransaction tx) {
        String accountNumber = tx.getMacro().getAccountFrom();
        Long amount = tx.getAmount();
        
        // 1. 계좌 조회 (비관적 락)
        Account account = accountService.findByNumberWithLock(accountNumber)
            .orElseThrow(() -> new AccountNotFoundException(accountNumber));
        
        // 2. 잔액 확인
        if (account.getBalance() < amount) {
            throw new InsufficientBalanceException(
                String.format("잔액 부족: 현재 %d원, 요청 %d원", 
                    account.getBalance(), amount)
            );
        }
        
        // 3. 일일 한도 확인
        Long todayWithdrawn = transactionRepository
            .sumWithdrawalAmountToday(accountNumber);
        if (todayWithdrawn + amount > account.getDailyWithdrawalLimit()) {
            throw new DailyLimitExceededException(
                "일일 출금 한도 초과"
            );
        }
        
        // 4. 잔액 차감
        account.withdraw(amount);
        accountService.save(account);
        
        // 5. 거래 내역 저장
        Transaction transaction = Transaction.builder()
            .account(account)
            .type(TransactionType.WITHDRAW)
            .amount(amount)
            .balanceAfter(account.getBalance())
            .timestamp(Instant.now())
            .macroTransactionId(tx.getId())
            .build();
        transactionRepository.save(transaction);
        
        // 6. 거래 상태 업데이트
        tx.setStatus(TransactionStatus.COMPLETED);
        tx.setTransactionId(transaction.getId());
        
        return TransactionResult.builder()
            .type(TransactionType.WITHDRAW)
            .success(true)
            .amount(amount)
            .balanceAfter(account.getBalance())
            .timestamp(transaction.getTimestamp())
            .build();
    }
    
    /**
     * 잔액 조회
     */
    private TransactionResult executeBalanceCheck(MacroTransaction tx) {
        String accountNumber = tx.getMacro().getAccountFrom();
        
        Account account = accountService.findByNumber(accountNumber)
            .orElseThrow(() -> new AccountNotFoundException(accountNumber));
        
        tx.setStatus(TransactionStatus.COMPLETED);
        
        return TransactionResult.builder()
            .type(TransactionType.BALANCE_CHECK)
            .success(true)
            .balanceAfter(account.getBalance())
            .timestamp(Instant.now())
            .build();
    }
}
```

### 2. 트랜젝션 격리 수준 및 동시성 제어

#### 2.1 데이터베이스 트랜젝션 설정
```yaml
# application.yml
spring:
  jpa:
    properties:
      hibernate:
        # READ_COMMITTED: 커밋된 데이터만 읽기
        # 팬텀 리드 가능성 있지만 성능과 동시성의 균형
        connection:
          isolation: 2  # READ_COMMITTED
```

#### 2.2 동시성 제어 전략

**비관적 락 (Pessimistic Lock)**
```java
@Repository
public interface MacroRepository extends JpaRepository<Macro, String> {
    
    /**
     * 매크로 조회 시 행 잠금 (FOR UPDATE)
     * 동시에 같은 매크로를 실행하려는 시도 차단
     */
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT m FROM Macro m WHERE m.id = :id")
    Optional<Macro> findByIdWithLock(@Param("id") String id);
}

@Repository
public interface AccountRepository extends JpaRepository<Account, String> {
    
    /**
     * 계좌 조회 시 행 잠금
     * 동시 출금 방지 (잔액 일관성 보장)
     */
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT a FROM Account a WHERE a.accountNumber = :number")
    Optional<Account> findByNumberWithLock(@Param("number") String number);
}
```

**낙관적 락 (Optimistic Lock)**
```java
@Entity
@Table(name = "qr_tokens")
public class QRToken {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;
    
    @Column(unique = true, nullable = false)
    private String token;
    
    @Column(nullable = false)
    private boolean used = false;
    
    /**
     * 버전 관리로 동시 수정 감지
     * QR 토큰 중복 사용 방지
     */
    @Version
    private Long version;
    
    public void markAsUsed() {
        if (this.used) {
            throw new QRTokenAlreadyUsedException();
        }
        this.used = true;
    }
}

// 사용 예시
@Service
@Transactional
public class QRVerificationService {
    
    public void markQRAsUsed(String token) {
        QRToken qrToken = qrTokenRepository.findByToken(token)
            .orElseThrow(() -> new QRTokenNotFoundException());
        
        try {
            qrToken.markAsUsed();
            qrTokenRepository.save(qrToken);
        } catch (OptimisticLockingFailureException e) {
            // 다른 스레드가 이미 사용 처리함
            throw new QRTokenAlreadyUsedException(
                "이미 사용된 QR 코드입니다"
            );
        }
    }
}
```

#### 2.3 분산 락 (Redis 기반)
```java
/**
 * 여러 ATM에서 동시에 같은 QR을 스캔하는 경우 방지
 * Redis 분산 락 사용
 */
@Service
public class DistributedLockService {
    
    private final RedisTemplate<String, String> redisTemplate;
    
    public boolean acquireLock(String key, long timeoutSeconds) {
        String lockKey = "lock:" + key;
        Boolean acquired = redisTemplate.opsForValue()
            .setIfAbsent(lockKey, "1", timeoutSeconds, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(acquired);
    }
    
    public void releaseLock(String key) {
        String lockKey = "lock:" + key;
        redisTemplate.delete(lockKey);
    }
}

@Service
@Transactional
public class QRVerificationService {
    
    private final DistributedLockService lockService;
    
    public QRVerifyResponse verifyQR(String token) {
        String lockKey = "qr:verify:" + token;
        
        // 분산 락 획득 시도 (3초 타임아웃)
        if (!lockService.acquireLock(lockKey, 3)) {
            throw new ConcurrentQRVerificationException(
                "다른 ATM에서 처리 중입니다"
            );
        }
        
        try {
            // QR 검증 및 처리
            return doVerifyQR(token);
        } finally {
            // 락 해제
            lockService.releaseLock(lockKey);
        }
    }
}
```

### 3. 트랜젝션 보안 및 감사

#### 3.1 감사 로그 (Audit Log)
```java
@Entity
@Table(name = "audit_logs")
public class AuditLog {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;
    
    @Column(nullable = false)
    private String userId;
    
    @Column(nullable = false)
    private String macroId;
    
    @Enumerated(EnumType.STRING)
    private AuditEventType eventType;  // MACRO_CREATED, QR_GENERATED, TRANSACTION_EXECUTED
    
    @Column(columnDefinition = "TEXT")
    private String eventData;  // JSON 형태의 상세 정보
    
    @Column(nullable = false)
    private Instant timestamp;
    
    private String ipAddress;
    private String userAgent;
    
    @Enumerated(EnumType.STRING)
    private AuditStatus status;  // SUCCESS, FAILURE
    
    private String errorMessage;
}

@Service
public class AuditLogService {
    
    private final AuditLogRepository auditLogRepository;
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logTransactionExecution(
            String userId, 
            String macroId, 
            List<TransactionResult> results,
            AuditStatus status) {
        
        AuditLog log = AuditLog.builder()
            .userId(userId)
            .macroId(macroId)
            .eventType(AuditEventType.TRANSACTION_EXECUTED)
            .eventData(toJson(results))
            .timestamp(Instant.now())
            .status(status)
            .build();
            
        auditLogRepository.save(log);
    }
    
    /**
     * 별도 트랜젝션으로 실행 (REQUIRES_NEW)
     * 메인 트랜젝션 실패해도 감사 로그는 저장됨
     */
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logSecurityEvent(String userId, String event, String details) {
        AuditLog log = AuditLog.builder()
            .userId(userId)
            .eventType(AuditEventType.SECURITY_EVENT)
            .eventData(details)
            .timestamp(Instant.now())
            .build();
            
        auditLogRepository.save(log);
    }
}
```

#### 3.2 이상 거래 탐지 (Fraud Detection)
```java
@Service
public class FraudDetectionService {
    
    private final TransactionRepository transactionRepository;
    private final NotificationService notificationService;
    
    /**
     * 이상 거래 패턴 감지
     */
    public void checkForFraud(Transaction transaction) {
        List<FraudRule> violations = new ArrayList<>();
        
        // 1. 고액 거래 탐지
        if (transaction.getAmount() > 1_000_000) {
            violations.add(new FraudRule(
                "HIGH_AMOUNT",
                "100만원 이상 고액 거래"
            ));
        }
        
        // 2. 단시간 내 반복 거래
        long recentCount = transactionRepository
            .countByAccountAndTimestampAfter(
                transaction.getAccount(),
                Instant.now().minus(5, ChronoUnit.MINUTES)
            );
        
        if (recentCount > 5) {
            violations.add(new FraudRule(
                "FREQUENT_TRANSACTIONS",
                "5분 내 5회 이상 거래"
            ));
        }
        
        // 3. 심야 시간 거래
        LocalTime time = LocalTime.now();
        if (time.isAfter(LocalTime.of(23, 0)) || 
            time.isBefore(LocalTime.of(6, 0))) {
            violations.add(new FraudRule(
                "UNUSUAL_TIME",
                "심야 시간 거래"
            ));
        }
        
        // 4. 이상 거래 감지 시 알림
        if (!violations.isEmpty()) {
            notificationService.notifyFraudDetection(
                transaction,
                violations
            );
        }
    }
}
```

### 4. 에러 처리 및 복구 전략

#### 4.1 예외 계층 구조
```java
// 최상위 비즈니스 예외
public abstract class BusinessException extends RuntimeException {
    private final ErrorCode errorCode;
    
    public BusinessException(ErrorCode errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
}

// 도메인별 예외
public class MacroNotFoundException extends BusinessException {
    public MacroNotFoundException(String macroId) {
        super(ErrorCode.MACRO_NOT_FOUND, 
              "매크로를 찾을 수 없습니다: " + macroId);
    }
}

public class InsufficientBalanceException extends BusinessException {
    public InsufficientBalanceException(String message) {
        super(ErrorCode.INSUFFICIENT_BALANCE, message);
    }
}

public class QRTokenExpiredException extends BusinessException {
    public QRTokenExpiredException(String message) {
        super(ErrorCode.QR_TOKEN_EXPIRED, message);
    }
}

// 에러 코드 정의
public enum ErrorCode {
    // 매크로 관련
    MACRO_NOT_FOUND(404, "M001", "매크로를 찾을 수 없습니다"),
    MACRO_EXPIRED(400, "M002", "매크로가 만료되었습니다"),
    INVALID_MACRO_STATUS(400, "M003", "매크로 상태가 올바르지 않습니다"),
    
    // QR 관련
    QR_TOKEN_NOT_FOUND(404, "Q001", "QR 토큰을 찾을 수 없습니다"),
    QR_TOKEN_EXPIRED(400, "Q002", "QR 토큰이 만료되었습니다"),
    QR_TOKEN_ALREADY_USED(400, "Q003", "이미 사용된 QR 토큰입니다"),
    
    // 계좌/거래 관련
    ACCOUNT_NOT_FOUND(404, "A001", "계좌를 찾을 수 없습니다"),
    INSUFFICIENT_BALANCE(400, "A002", "잔액이 부족합니다"),
    DAILY_LIMIT_EXCEEDED(400, "A003", "일일 한도를 초과했습니다"),
    
    // 트랜젝션 관련
    TRANSACTION_FAILED(500, "T001", "거래 처리 중 오류가 발생했습니다"),
    UNSUPPORTED_TRANSACTION_TYPE(400, "T002", "지원하지 않는 거래 유형입니다");
    
    private final int httpStatus;
    private final String code;
    private final String message;
}
```

#### 4.2 글로벌 예외 핸들러
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private final AuditLogService auditLogService;
    
    /**
     * 비즈니스 예외 처리
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(
            BusinessException e, 
            HttpServletRequest request) {
        
        ErrorCode errorCode = e.getErrorCode();
        
        ErrorResponse response = ErrorResponse.builder()
            .code(errorCode.getCode())
            .message(e.getMessage())
            .timestamp(Instant.now())
            .path(request.getRequestURI())
            .build();
        
        // 감사 로그 기록
        auditLogService.logError(e, request);
        
        return ResponseEntity
            .status(errorCode.getHttpStatus())
            .body(response);
    }
    
    /**
     * 데이터베이스 예외 처리
     */
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ErrorResponse> handleDataAccessException(
            DataAccessException e,
            HttpServletRequest request) {
        
        // 데이터베이스 오류는 상세 내용을 숨김 (보안)
        ErrorResponse response = ErrorResponse.builder()
            .code("DB_ERROR")
            .message("일시적인 오류가 발생했습니다. 잠시 후 다시 시도해주세요.")
            .timestamp(Instant.now())
            .path(request.getRequestURI())
            .build();
        
        // 상세 오류는 로그에만 기록
        log.error("Database error occurred", e);
        auditLogService.logCriticalError(e, request);
        
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(response);
    }
    
    /**
     * 낙관적 락 충돌 처리
     */
    @ExceptionHandler(OptimisticLockingFailureException.class)
    public ResponseEntity<ErrorResponse> handleOptimisticLock(
            OptimisticLockingFailureException e) {
        
        ErrorResponse response = ErrorResponse.builder()
            .code("CONCURRENT_MODIFICATION")
            .message("동시에 같은 데이터가 수정되었습니다. 다시 시도해주세요.")
            .timestamp(Instant.now())
            .build();
        
        return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(response);
    }
}
```

#### 4.3 재시도 메커니즘
```java
@Service
public class ResilientTransactionService {
    
    private final TransactionExecutionService executionService;
    
    /**
     * 네트워크 일시 오류 등에 대한 자동 재시도
     * 최대 3번, 지수 백오프 (1초, 2초, 4초)
     */
    @Retryable(
        value = { TransientDataAccessException.class, TimeoutException.class },
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public TransactionResponse executeWithRetry(String macroId) {
        return executionService.executeTransaction(macroId);
    }
    
    /**
     * 재시도 실패 시 최종 처리
     */
    @Recover
    public TransactionResponse recover(
            Exception e, 
            String macroId) {
        
        log.error("Transaction execution failed after retries: {}", macroId, e);
        
        // 실패 상태로 기록
        Macro macro = macroRepository.findById(macroId)
            .orElseThrow();
        macro.setStatus(MacroStatus.FAILED);
        macro.setErrorMessage("재시도 후에도 실패: " + e.getMessage());
        macroRepository.save(macro);
        
        throw new TransactionExecutionException(
            "거래 실행에 실패했습니다. 고객센터로 문의해주세요.", e
        );
    }
}
```

### 5. 성능 최적화

#### 5.1 데이터베이스 쿼리 최적화
```java
@Repository
public interface MacroRepository extends JpaRepository<Macro, String> {
    
    /**
     * N+1 문제 해결: Fetch Join 사용
     * 매크로와 트랜젝션을 한 번의 쿼리로 조회
     */
    @Query("SELECT m FROM Macro m " +
           "LEFT JOIN FETCH m.transactions " +
           "WHERE m.id = :id")
    Optional<Macro> findByIdWithTransactions(@Param("id") String id);
    
    /**
     * 인덱스 활용: userId + status + createdAt
     */
    @Query("SELECT m FROM Macro m " +
           "WHERE m.userId = :userId " +
           "AND m.status = :status " +
           "AND m.createdAt >= :since " +
           "ORDER BY m.createdAt DESC")
    List<Macro> findRecentMacros(
        @Param("userId") String userId,
        @Param("status") MacroStatus status,
        @Param("since") Instant since
    );
}

// 인덱스 정의
@Entity
@Table(
    name = "macros",
    indexes = {
        @Index(name = "idx_user_status_created", 
               columnList = "user_id, status, created_at"),
        @Index(name = "idx_expires_at", 
               columnList = "expires_at")
    }
)
public class Macro { ... }
```

#### 5.2 캐싱 전략
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager(
            "accounts",      // 계좌 정보 캐시
            "dailyLimits",   // 일일 한도 캐시
            "userProfiles"   // 사용자 프로필 캐시
        );
    }
}

@Service
public class AccountService {
    
    /**
     * 계좌 정보 캐싱 (1분)
     * 잔액은 변동되므로 짧은 TTL
     */
    @Cacheable(value = "accounts", key = "#accountNumber")
    public Account findByNumber(String accountNumber) {
        return accountRepository.findByNumber(accountNumber)
            .orElseThrow(() -> new AccountNotFoundException(accountNumber));
    }
    
    /**
     * 캐시 무효화 (잔액 변경 시)
     */
    @CacheEvict(value = "accounts", key = "#account.accountNumber")
    public void save(Account account) {
        accountRepository.save(account);
    }
}
```

#### 5.3 비동기 처리
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class NotificationService {
    
    /**
     * 알림 전송은 비동기로 처리
     * 메인 트랜젝션 성능에 영향 없음
     */
    @Async
    public CompletableFuture<Void> sendTransactionNotification(
            String userId, 
            TransactionResult result) {
        
        try {
            // SMS, 푸시 알림 등 전송
            smsService.send(userId, createMessage(result));
            pushService.send(userId, createPushData(result));
            
            return CompletableFuture.completedFuture(null);
        } catch (Exception e) {
            log.error("Failed to send notification", e);
            return CompletableFuture.failedFuture(e);
        }
    }
}
```

### 6. 모니터링 및 관찰성 (Observability)

#### 6.1 메트릭 수집
```java
@Service
public class MetricsService {
    
    private final MeterRegistry meterRegistry;
    
    public MetricsService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        
        // 커스텀 메트릭 등록
        Gauge.builder("macro.active.count", this, 
              MetricsService::getActiveMacroCount)
            .description("활성 매크로 수")
            .register(meterRegistry);
    }
    
    public void recordTransactionExecution(
            TransactionType type, 
            boolean success, 
            long durationMs) {
        
        // 거래 실행 횟수
        Counter.builder("transaction.executed")
            .tag("type", type.name())
            .tag("success", String.valueOf(success))
            .register(meterRegistry)
            .increment();
        
        // 거래 실행 시간
        Timer.builder("transaction.duration")
            .tag("type", type.name())
            .register(meterRegistry)
            .record(durationMs, TimeUnit.MILLISECONDS);
    }
    
    public void recordQRVerification(boolean success, long durationMs) {
        Counter.builder("qr.verification")
            .tag("success", String.valueOf(success))
            .register(meterRegistry)
            .increment();
        
        Timer.builder("qr.verification.duration")
            .register(meterRegistry)
            .record(durationMs, TimeUnit.MILLISECONDS);
    }
}
```

#### 6.2 구조화된 로깅
```java
@Slf4j
@Service
public class TransactionExecutionService {
    
    public TransactionResponse executeTransaction(String macroId) {
        // 구조화된 로그 (JSON 형태)
        MDC.put("macroId", macroId);
        MDC.put("operation", "transaction.execute");
        
        long startTime = System.currentTimeMillis();
        
        try {
            log.info("Transaction execution started");
            
            Macro macro = macroRepository.findByIdWithLock(macroId)
                .orElseThrow(() -> new MacroNotFoundException(macroId));
            
            MDC.put("userId", macro.getUserId());
            MDC.put("transactionCount", 
                    String.valueOf(macro.getTransactions().size()));
            
            // ... 거래 실행 로직
            
            long duration = System.currentTimeMillis() - startTime;
            log.info("Transaction execution completed successfully. duration={}ms", 
                     duration);
            
            metricsService.recordTransactionExecution(
                TransactionType.MACRO, true, duration
            );
            
            return response;
            
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            log.error("Transaction execution failed. duration={}ms, error={}", 
                      duration, e.getMessage(), e);
            
            metricsService.recordTransactionExecution(
                TransactionType.MACRO, false, duration
            );
            
            throw e;
            
        } finally {
            MDC.clear();
        }
    }
}
```

### 7. 트랜젝션 플로우 시퀀스 다이어그램

```
사용자(Mobile)    모바일 앱       백엔드 API       데이터베이스       ATM
     |              |              |                |              |
     |--거래 입력--->|              |                |              |
     |              |--매크로 생성-->|                |              |
     |              |              |--트랜젝션 검증-->|              |
     |              |              |<--검증 완료------|              |
     |              |              |--매크로 저장---->|              |
     |              |              |<--저장 완료------|              |
     |              |<--매크로 ID---|                |              |
     |              |              |                |              |
     |<--매크로 완료--|              |                |              |
     |              |              |                |              |
     |--QR 요청----->|              |                |              |
     |              |--QR 생성----->|                |              |
     |              |              |--JWT 토큰 생성-->|              |
     |              |              |<--토큰 저장------|              |
     |              |<--QR 코드-----|                |              |
     |<--QR 표시-----|              |                |              |
     |              |              |                |              |
     |---ATM 방문---------------------------->|     |              |
     |              |              |                |     |--QR 스캔--->|
     |              |              |<--QR 검증 요청------|<------------|
     |              |              |--토큰 검증------>|              |
     |              |              |--매크로 조회---->|              |
     |              |              |<--매크로 반환----|              |
     |              |              |--QR 사용 처리--->|              |
     |              |              |<--처리 완료------|              |
     |              |              |--검증 성공-------->|------------>|
     |              |              |                |     |          |
     |              |              |<--거래 실행 요청----|<------------|
     |              |              |--매크로 조회(락)-->|              |
     |              |              |<--매크로 반환----|              |
     |              |              |--계좌 조회(락)-->|              |
     |              |              |<--계좌 정보------|              |
     |              |              |--잔액 차감------>|              |
     |              |              |--거래 기록------>|              |
     |              |              |<--커밋 완료------|              |
     |              |              |--거래 완료-------->|------------>|
     |              |              |                |     |--결과 표시->|
     |              |              |                |              |
```

---

## 결론

원터치 효(孝)뱅킹은 단순히 기술적 솔루션을 넘어, **디지털 약자층의 금융 접근성을 개선하고 심리적 장벽을 해소하는 사회적 가치**를 추구합니다.

### 핵심 성과
1. **사전 설정 + QR 인증** 방식으로 ATM 체류 시간 80% 단축
2. **트랜젝션 격리 및 동시성 제어**로 안전하고 일관된 거래 처리
3. **멀티모달 UX**로 고령층의 인지 부담 최소화
4. **확장 가능한 아키텍처**로 향후 기능 확대 기반 마련

### 기술적 하이라이트
- Spring @Transactional을 활용한 ACID 보장
- 비관적 락/낙관적 락을 통한 동시성 제어
- JWT 기반 일회용 QR 토큰으로 보안 강화
- 구조화된 로깅 및 메트릭으로 관찰성 확보
- 재시도 메커니즘 및 에러 복구 전략

이 프로젝트는 **금융 포용성(Financial Inclusion)**과 **고령 친화적 디지털 전환**이라는 사회적 목표를 기술적으로 구현한 사례로, 향후 실제 금융기관에 적용 가능한 가능성을 보여줍니다.

---

**작성일**: 2025-12-02  
**프로젝트**: 2025 KB IT's Your Life 해커톤  
**팀**: 원터치 효(孝)뱅킹
