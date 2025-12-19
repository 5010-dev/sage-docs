# Sage.ai MVP 정의

> **문서 버전**: 1.0  
> **최종 수정**: 2025년 12월 19일  
> **작성자**: Sam  
> **대상 독자**: 전체 팀 (개발, 기획, 마케팅)

---

## MVP 개요

### 정의

**MVP (Minimum Viable Product)**: 3개월 내 PMF(Product-Market Fit) 검증을 위한 최소 기능 제품

### 목표

**"AI 투자 멘토의 핵심 가치를 검증하고, 초기 사용자로부터 피드백을 받는다"**

### 성공 지표

| 지표 | 목표 (3개월 후) | 측정 방법 |
|------|----------------|----------|
| **베타 테스터** | 10-20명 | 직접 모집 |
| **WhyBitcoinFallen 방문자** | 일 1,000+ | Google Analytics |
| **Sage.ai MAU** | 500+ | 활성 사용자 수 |
| **환각률** | <1% | 사용자 신고 / 전체 응답 |
| **Shadow Portfolio 추적** | 10+ 트레이드 | DB 기록 |
| **NPS** | 40+ | 분기별 설문 |

---

## 핵심 기능 (Must Have)

### 1. 월렛 버핏과의 대화

#### 기능 설명
- Claude Sonnet 4 기반 AI 멘토와 실시간 대화
- 시장 데이터 기반 통찰 제공
- 워렌 버핏의 투자 철학 구현

#### 핵심 요구사항
- [x] SSE 스트리밍 응답 (2초 이내 첫 토큰)
- [x] 멀티 에이전트 시스템 (Manager, Analyst, Persona, Risk)
- [x] 실시간 시장 데이터 통합 (CoinGecko)
- [x] 환각 방지 메커니즘 (Tool 강제 + 교차 검증)

#### 범위
- **포함**: 6개 코인 (BTC, ETH, SOL, BNB, DOGE, XRP)
- **포함**: Fear & Greed Index 통합
- **제외**: 다중 페르소나 (사토시 현자, 알파 헌터 등) → Phase 2
- **제외**: 그룹 채팅 (AI끼리 토론) → Phase 2

### 2. 섀도우 포트폴리오

#### 기능 설명
- AI 추천을 가상으로 추적하여 성과 검증
- "담아보기" 버튼으로 사용자가 직접 선택
- 투명한 수익률 공개

#### 핵심 요구사항
- [x] AI 시그널 감지 (buy/sell 추천)
- [x] 현재가 자동 조회 및 저장
- [x] 수익률 계산 로직
- [x] 포트폴리오 대시보드

#### 범위
- **포함**: 기본 수익률 계산 (절대 수익률, 벤치마크 대비)
- **포함**: 최대 3개 포트폴리오 (Pro 플랜)
- **제외**: 복잡한 지표 (샤프 비율, MDD 등) → Phase 2
- **제외**: 거래소 API 연동 (실제 매매) → Phase 2

### 3. 능동적 알림

#### 기능 설명
- 15분마다 시장 자동 분석
- 급변 시 PWA Push + Discord 알림
- 딥링크로 즉시 관련 채팅 시작

#### 핵심 요구사항
- [x] BullMQ 기반 백그라운드 작업
- [x] @nestjs/schedule 주기적 폴링
- [x] PWA 푸시 구독 시스템
- [x] Discord Webhook 통합
- [x] 딥링크 URL 생성

#### 범위
- **포함**: 가격 급변 감지 (±5% BTC, ±7% ETH, ±10% 알트)
- **포함**: Fear & Greed 급변 감지 (±15)
- **제외**: 온체인 분석 (고래 이동 등) → Phase 2
- **제외**: 뉴스 기반 알림 → Phase 2

---

## 제외 기능 (Phase 2+)

### Phase 2 (2026 Q2-Q3)

| 기능 | 이유 | 우선순위 |
|------|------|----------|
| **멀티 페르소나** | 월렛 버핏 하나로 PMF 먼저 검증 | 높음 |
| **그룹 채팅** | 1:1 대화 UX 안정화 우선 | 높음 |
| **RAG 장기 기억** | 20개 메시지로 충분 (초기) | 중간 |
| **온체인 분석** | 외부 데이터 의존도 증가 | 중간 |
| **추가 코인** | 6개로 PMF 검증 후 확장 | 낮음 |

### Phase 3 (2027+)

- 주식 시장 확장
- 실시간 거래소 WebSocket
- 소셜 기능 (성과 공유)
- 모바일 앱 (React Native)

---

## 기술 스택 (MVP)

### 백엔드
- **프레임워크**: Nest.js 10.x
- **ORM**: Prisma 5.x
- **Database**: PostgreSQL 16 + Redis 7.x
- **비동기**: BullMQ 5.x

### 프론트엔드
- **프레임워크**: React 18.3 + Vite 5
- **상태관리**: Zustand 4.x + TanStack Query 5.x
- **스타일링**: Tailwind CSS 3.x

### AI
- **모델**: Claude Sonnet 4 + Haiku 4
- **SDK**: @anthropic-ai/sdk (TypeScript)

### 인프라
- **컨테이너**: AWS ECS Fargate
- **정적 파일**: S3 + CloudFront
- **모니터링**: Sentry + CloudWatch

---

## 사용자 여정 (MVP)

### 신규 사용자

```
[WhyBitcoinFallen.com 방문]
    ↓ (바이럴 훅)
[Sage.ai 랜딩 페이지]
    ↓ (가입 전환)
[Google OAuth 로그인]
    ↓ (온보딩 없음!)
[즉시 채팅 시작]
    ↓
[월렛 버핏과 대화]
    ↓ (AI 추천 발생)
["담아보기" 클릭]
    ↓
[섀도우 포트폴리오 추가]
    ↓ (시간 경과)
[수익률 확인]
    ↓ (만족)
[유료 전환 (Pro $19.99)]
```

### 재방문 사용자

```
[Discord 알림 수신]
"BTC -5.2% 급락"
    ↓ (클릭)
[앱 자동 실행]
    ↓ (딥링크)
[관련 채팅 자동 시작]
"BTC가 급변했는데 어떻게 생각해?"
    ↓
[월렛 버핏 분석 수신]
    ↓
[대화 또는 담아보기]
```

---

## 개발 범위

### 페이지 구성

#### WhyBitcoinFallen.com (바이럴 사이트)
- **페이지 수**: 1개 (SPA)
- **기능**:
  - 실시간 BTC 가격 표시
  - Fear & Greed 게이지
  - AI 미리보기 (월렛 버핏 한마디)
  - Sage.ai 가입 CTA

#### Sage.ai 랜딩
- **페이지 수**: 1개 (SPA)
- **기능**:
  - Hero 섹션
  - 3가지 핵심 기능 소개
  - 성과 카드 (섀도우 포트폴리오 예시)
  - 가입 CTA

#### Sage.ai 앱
- **페이지 수**: 5개
  - `/chat` - 채팅 목록
  - `/chat/:id` - 대화 상세
  - `/portfolio` - 섀도우 포트폴리오
  - `/notifications` - 알림 히스토리
  - `/settings` - 설정

### API 엔드포인트

| Method | Endpoint | 기능 |
|--------|----------|------|
| POST | `/api/auth/google` | Google OAuth |
| POST | `/api/chat` | 채팅 메시지 전송 (SSE) |
| GET | `/api/chats` | 채팅 목록 조회 |
| GET | `/api/chats/:id/messages` | 메시지 조회 |
| GET | `/api/market/price` | 가격 조회 (캐싱) |
| GET | `/api/market/fear-greed` | Fear & Greed 조회 |
| POST | `/api/shadow-trades` | 섀도우 트레이드 추가 |
| GET | `/api/shadow-trades` | 포트폴리오 조회 |
| POST | `/api/push/subscribe` | 푸시 구독 등록 |
| GET | `/api/notifications` | 알림 히스토리 |

---

## 데이터 모델 (핵심만)

### Users
```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  image     String?
  tier      String   @default("free") // free, pro, premium
  createdAt DateTime @default(now())
}
```

### Chats
```prisma
model Chat {
  id        String    @id @default(uuid())
  userId    String
  title     String    @default("새 대화")
  messages  Message[]
  createdAt DateTime  @default(now())
}
```

### Messages
```prisma
model Message {
  id        String   @id @default(uuid())
  chatId    String
  role      String   // user, assistant
  content   String
  signal    Json?    // AI 시그널 (buy/sell)
  createdAt DateTime @default(now())
}
```

### ShadowTrades
```prisma
model ShadowTrade {
  id        String   @id @default(uuid())
  userId    String
  symbol    String   // BTC, ETH, etc.
  action    String   // buy, sell
  price     Decimal
  createdAt DateTime @default(now())
}
```

---

## 제약 사항 & 트레이드오프

### 기술적 제약

| 제약 사항 | 이유 | 완화 방법 |
|----------|------|----------|
| **20개 메시지 컨텍스트** | RAG 구현 복잡도 | Phase 2에서 pgvector 추가 |
| **6개 코인만 지원** | API 비용, 복잡도 | 사용자 피드백 기반 확장 |
| **순차 알림 발송** | SNS+SQS 인프라 부담 | 초기 사용자 500명 이하면 충분 |

### 비즈니스 제약

| 제약 사항 | 이유 | 완화 방법 |
|----------|------|----------|
| **유료 플랜 없음** | MVP에서 검증 후 런칭 | Q3에 Pro/Premium 오픈 |
| **한/영만 지원** | 리소스 제한 | Q2에 일/중 추가 |
| **Discord만 커뮤니티** | 초기 집중 필요 | Q2에 카카오톡 추가 |

---

## 테스트 계획

### 베타 테스터 모집

- **인원**: 10-20명
- **프로필**: 암호화폐 투자 경험 있는 얼리어답터
- **기간**: 2주
- **보상**: 평생 Pro 플랜 무료

### 테스트 시나리오

1. **기본 대화**
   - "비트코인 지금 어때?"
   - "이더리움 살까?"
   - "포트폴리오 점검해줘"

2. **섀도우 포트폴리오**
   - AI 추천 받기
   - "담아보기" 클릭
   - 수익률 확인

3. **알림**
   - Discord에서 급변 알림 수신
   - 푸시 알림 수신
   - 딥링크로 채팅 시작

### 품질 기준

- **응답 속도**: 2초 이내 (첫 토큰)
- **환각률**: <1% (사용자 신고 기준)
- **가동률**: 99% (다운타임 최소화)
- **버그**: P0 버그 0개, P1 버그 3개 이하

---

## 릴리즈 계획

### 릴리즈 단계

#### Alpha (내부 테스트)
- **시기**: M2 말
- **대상**: 팀 내부
- **목적**: 기능 동작 확인

#### Closed Beta
- **시기**: M3 초
- **대상**: 10-20명 베타 테스터
- **목적**: 실사용 피드백

#### Open Beta
- **시기**: M3 중순
- **대상**: Waitlist 사용자 (100명 제한)
- **목적**: 서버 부하 테스트

#### Public Launch
- **시기**: M3 말
- **대상**: 전체 공개
- **목적**: PMF 검증

---

## 성공 기준 & Next Steps

### MVP 성공 기준

| 지표 | 목표 | 판단 |
|------|------|------|
| **베타 테스터 피드백** | NPS 40+ | 만족 시 다음 단계 |
| **환각률** | <1% | 기술 검증 완료 |
| **재방문율** | 40%+ | PMF 신호 |
| **섀도우 포트폴리오 사용** | 30%+ | 핵심 가치 검증 |

### 성공 시 (Phase 2)
- 유료 플랜 런칭
- 멀티 페르소나 개발
- 그룹 채팅 기능
- 글로벌 확장 (일/중 지원)

### 실패 시 (Pivot)
- 기능 재검토
- 타겟 유저 변경
- 비즈니스 모델 수정

---

## Appendix

### A. MVP 범위 체크리스트

#### 반드시 포함
- [x] 월렛 버핏 대화
- [x] 섀도우 포트폴리오
- [x] 능동적 알림
- [x] WhyBitcoinFallen.com
- [x] Sage.ai 랜딩

#### 제외 (Phase 2)
- [ ] 멀티 페르소나
- [ ] 그룹 채팅
- [ ] RAG 장기 기억
- [ ] 온체인 분석
- [ ] 실시간 거래소 WebSocket

### B. 용어 사전

- **PMF**: Product-Market Fit, 제품-시장 적합성
- **NPS**: Net Promoter Score, 순추천고객지수
- **MAU**: Monthly Active Users, 월간 활성 사용자
- **SSE**: Server-Sent Events, 서버 전송 이벤트

---

**문서 끝**

_"Between the zeros and ones"_
