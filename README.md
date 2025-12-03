# 원터치 효(孝)뱅킹 – 우리 부모님 전용 ATM 솔루션
> 『2025 KB IT’s Your Life 해커톤』 – 디지털 약자를 위한 금융 접근성 개선

고령층이 ATM에서 느끼는 **시간 압박·뒷사람 시선** 등의 심리적 장벽을 낮추기 위해,  
집에서 **모바일 사전 매크로**로 거래를 모두 설정하고, 현장 ATM에서는 **QR 한 번**으로 끝내는 프로토타입입니다.  
ATM 화면에는 **캐릭터 애니메이션 + 음성 안내**가 함께 제공되어 직관적이고 편안한 사용 경험을 만듭니다.

---

## 핵심 가치 (Why)
- **심리적 부담 최소화**: 현장 조작을 줄여 ‘빨리 해야 한다’는 압박을 제거
- **실사용 맥락 최적화**: 집에서 미리 설정 → ATM에서는 인증만
- **접근성 강화**: 캐릭터·음성 기반 **멀티모달 안내**로 눈과 손의 부담을 분산

## 주요 기능 (What)
1. **ATM 시뮬레이터(UI/UX)**
   - 실제 ATM과 유사한 화면 흐름
   - 캐릭터(예: 버튼 가리키기, 완료 제스처) 애니메이션 + 음성 안내
2. **모바일 사전 매크로 + QR 연동**
   - 계좌, 금액, 절차(순서)를 **행동 단위 매크로**로 저장
   - 생성된 **QR**을 ATM에서 스캔하면 즉시 실행
3. **간소화된 현장 프로세스**
   - 무카드·OTP 입력 등 다단계를 대폭 축소 → **QR 한 번으로 완료**

## 차별성 (Differentiation)
- ‘쉬운 사용법’만이 아니라, **‘편안한 경험’** 자체를 설계  
- 기존 무카드 출금의 다단계 인증 대신, **사전 설정 + 단일 QR 인증**으로 종결  
- 어깨너머 비밀번호 노출 등 **피싱 리스크도 저감**

## 데모(예정)
- 시연 영상: (추가 예정)  
- 스크린샷: (추가 예정)

---

## 아키텍처 개요 (How)
```
[Mobile Web/App]
└─ 거래 매크로 생성/저장 ─┐
├─ QR 생성/표시 ──▶ [ATM Simulator(Web)]
[Spring Boot Mock API] ◀─────┘ └─ QR 스캔 → 거래 실행
```

### 기술 스택
- **Front**: Vue.js, JavaScript, HTML/CSS
- **Server**: Spring Boot, Jpa
- **기타**: 음성 안내(브라우저 TTS 또는 오디오), QR 생성/스캔 라이브러리

---

## 시작하기 (Getting Started)

### 1) 저장소 구조(제안)
```
/atm-simulator # Vue 3 + Vite (ATM 시뮬레이터)
/mobile-app # Vue 3 + Vite (모바일 사전 매크로/QR)
/server # Spring Boot (QR 검증·매크로 Mock API)
```

### 2) 요구사항
- Node.js 20.19.0 / npm
- JDK 17+

### 3) 설치 & 실행
```
# ATM 시뮬레이터
cd hyo-banking-atm
nvm use
npm ci
npm run dev

# 모바일 앱(웹)
cd ../hyo-banking-mobile
nvm use
npm ci
npm run dev

# 서버(Mock)
cd ../hyo-banking-be
./gradlew bootRun
```
### 4) 환경 변수(예시)

`/server/src/main/resources/application.yml`
```
server:
  port: 8080
app:
  qr:
    expiry-seconds: 180
```
---
## API 스케치(예시)

| Method | Path                    | 설명                |
| -----: | ----------------------- | ----------------- |
|   POST | /api/macros             | 거래 매크로 생성         |
|    GET | /api/macros/{id}        | 매크로 상세 조회         |
|    GET | /api/macros/{id}/qrcode | 매크로 QR 이미지(또는 토큰) |
|   POST | /api/qr/verify          | QR 토큰 검증          |
|   POST | /api/atm/execute        | 검증된 매크로 실행        |


**매크로 예시(JSON)**
```
{
  "accountFrom": "123-456-789012",
  "transactions": [
    { "type": "withdraw", "amount": 100000 },
    { "type": "balance_check" }
  ]
}
```

## 접근성·UX 가이드
- 큰 글자·높은 대비·넓은 터치 영역(44px+) 유지
- 단계별 음성 안내 + 애니메이션 포인터 제공
- 진행 상태/남은 단계를 항상 표시
- ATM 현장에서는 선택지 최소화(이탈 없이 원터치 완료)

## 로드맵
- 캐릭터 동작(버튼 가리키기, 완료 제스처) 1차 세트
- 음성 스크립트/타이밍 싱크
- QR 보안 옵션(만료, 1회용, 서명)
- 사용자 테스트(고령층) 및 피드백 반영
- 시범 지점 PoC 연계(협의)

## 팀 & 역할
- **고창진 (BE Lead)**: 시스템 아키텍처 및 Mock API
- **김서현 (BE & FE)**: QR 생성/인증 로직, 모바일 사전 매크로
- **나동건 (FE Lead)**: 웹 시뮬레이터 핵심 개발 총괄
- **박해세 (BE & UI/UX)**: ATM 화면 레이아웃, 사용자 플로우, 음성 시나리오
- **정혜인 (PM/Design & FE)**: 캐릭터 애니메이션 기획, 문서·발표 자료

## 📖 상세 문서
- **[STAR 분석](./STAR_ANALYSIS.md)**: 프로젝트 전체 분석 (Situation-Task-Action-Result)
- **[트랜젝션 처리 아키텍처](./docs/TRANSACTION_ARCHITECTURE.md)**: 기술적 상세 문서
- **[문서 인덱스](./docs/README.md)**: 모든 문서의 빠른 참조

## 라이선스
- TBD

## 문의
- Issues 탭을 활용해 주세요.

