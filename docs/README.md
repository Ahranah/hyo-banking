# 원터치 효(孝)뱅킹 - 문서 인덱스

이 디렉토리는 원터치 효(孝)뱅킹 프로젝트의 기술 문서를 포함합니다.

## 📚 문서 목록

### 1. [STAR 분석](../STAR_ANALYSIS.md)
프로젝트의 전체적인 STAR (Situation-Task-Action-Result) 분석 문서입니다.

**내용:**
- **S (Situation)**: 고령층이 ATM 사용 시 겪는 문제점
- **T (Task)**: 프로젝트 목표 및 해결 과제
- **A (Action)**: 솔루션 아키텍처 및 구현 방법
- **R (Result)**: 예상 성과 및 향후 계획
- **트랜젝션 처리 상세 분석**: 매크로 생성부터 실행까지 전체 플로우

**대상 독자:** PM, 기획자, 개발자, 이해관계자 전체

---

### 2. [트랜젝션 처리 아키텍처](./TRANSACTION_ARCHITECTURE.md)
트랜젝션 처리 시스템의 기술적 상세 문서입니다.

**내용:**
- 트랜젝션 처리 흐름 (4단계)
- 데이터 모델 (ERD 및 테이블 스키마)
- 동시성 제어 전략 (비관적 락, 낙관적 락, 분산 락)
- 보안 및 검증
- 성능 최적화
- 에러 처리
- 모니터링 및 로깅
- 실제 시나리오별 처리 과정

**대상 독자:** 백엔드 개발자, 시스템 아키텍트, 데이터베이스 관리자

---

## 🔍 빠른 참조

### 트랜젝션 처리 단계
```
1단계: 매크로 생성 (Mobile)
   ↓
2단계: QR 생성 (Mobile)
   ↓
3단계: QR 검증 (ATM)
   ↓
4단계: 거래 실행 (ATM)
```

### API 엔드포인트

| Method | Endpoint | 설명 | 문서 |
|--------|----------|------|------|
| POST | `/api/macros` | 매크로 생성 | [상세](./TRANSACTION_ARCHITECTURE.md#1단계-매크로-생성) |
| GET | `/api/macros/{id}/qrcode` | QR 생성 | [상세](./TRANSACTION_ARCHITECTURE.md#2단계-qr-코드-생성) |
| POST | `/api/qr/verify` | QR 검증 | [상세](./TRANSACTION_ARCHITECTURE.md#3단계-qr-검증) |
| POST | `/api/atm/execute` | 거래 실행 | [상세](./TRANSACTION_ARCHITECTURE.md#4단계-거래-실행) |

### 주요 기술 스택

**Backend:**
- Spring Boot 3.5.5
- JPA/Hibernate
- Spring Security
- MySQL/H2

**Frontend:**
- Vue.js 3
- Vite
- JavaScript

**보안:**
- JWT 인증
- HTTPS
- QR 일회용 토큰

---

## 💡 개발 가이드

### 새로운 트랜젝션 유형 추가
1. `TransactionType` enum에 새 유형 추가
2. `TransactionExecutionService`에 실행 메서드 구현
3. 유효성 검증 로직 추가
4. 테스트 작성

### 보안 규칙 추가
1. `FraudDetectionService`에 새 규칙 추가
2. 심각도 및 알림 조건 설정
3. 테스트 작성

### 성능 최적화
- 인덱스 추가: [데이터 모델](./TRANSACTION_ARCHITECTURE.md#데이터-모델) 참조
- 캐싱 전략: [성능 최적화](./TRANSACTION_ARCHITECTURE.md#성능-최적화) 참조
- 쿼리 최적화: N+1 문제 해결 가이드 참조

---

## 🔧 문제 해결

### 자주 발생하는 문제

**Q: 동시에 같은 계좌에서 출금하면?**  
A: 비관적 락(FOR UPDATE)으로 순차 처리됩니다. [시나리오 4](./TRANSACTION_ARCHITECTURE.md#시나리오-4-동시-출금-시도) 참조

**Q: QR을 여러 번 사용하면?**  
A: 낙관적 락으로 일회성을 보장합니다. [시나리오 3](./TRANSACTION_ARCHITECTURE.md#시나리오-3-qr-재사용-시도) 참조

**Q: 트랜젝션 중간에 실패하면?**  
A: Spring @Transactional로 자동 롤백됩니다. [시나리오 2](./TRANSACTION_ARCHITECTURE.md#시나리오-2-잔액-부족) 참조

---

## 📞 연락처

문서에 대한 문의사항이나 개선 제안은 Issues 탭을 이용해 주세요.

---

**최종 업데이트**: 2025-12-02  
**프로젝트**: 2025 KB IT's Your Life 해커톤  
**팀**: 원터치 효(孝)뱅킹
