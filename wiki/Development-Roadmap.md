# Sage.ai Development Roadmap

```
문서 정보
├── 버전: 1.0
├── 최종 수정: 2025년 12월 19일
├── 작성자: Sam
└── 대상 독자: 개발팀, 경영진, 투자자
```

---

## 1. Roadmap Overview

### 1.1 Timeline

```
로드맵 타임라인
│
├── 2025 Dec ──────────────────────────────> 2027 Dec
│       │                                          │
│       MVP (3개월)    Scale    Monetize    Ecosystem
│       └─────┬─────┘  └──┬──┘  └───┬───┘  └────┬────┘
│          2026 Q1    2026 Q2-Q3  2026 Q4     2027+
```

### 1.2 Vision Progression

| Phase | 시기 | 핵심 가치 | 목표 |
|-------|------|----------|------|
| MVP | 2026 Q1 | PMF 검증 | MAU 5,000 |
| Scale | 2026 Q2-Q3 | 글로벌 확장 | MAU 50,000 |
| Monetize | 2026 Q4 | 수익화 | MRR $30,000 |
| Ecosystem | 2027+ | 생태계 | MAU 1,000,000 |

---

## 2. Phase 1: MVP (3개월, 2026 Q1)

### 2.1 목표

**"핵심 가치 검증 - 월렛 버핏 AI 멘토의 신뢰 확보"**

**전략**: 기능별로 점진적으로 출시하여 빠른 피드백 루프 확보

### 2.2 Month 1: Foundation (기반 구축)

#### Week 1-2: 프로젝트 셋업 & 인프라

```
인프라 구축 체크리스트
├── [ ] 프로젝트 구조 설계
├── [ ] AWS 계정 & 인프라 설정
│   ├── ECS Fargate 클러스터
│   ├── RDS PostgreSQL 16
│   ├── ElastiCache Redis 7.x
│   └── S3 + CloudFront
├── [ ] GitHub Repository & CI/CD
│   ├── GitHub Actions 워크플로우
│   └── 자동 테스트 & 배포
└── [ ] 개발 환경 설정
    ├── pnpm workspace
    ├── Nest.js 백엔드 boilerplate
    └── React + Vite 프론트엔드 boilerplate
```

#### Week 3-4: 인증 & 기본 UI

```
인증 & UI 체크리스트
├── [ ] Auth.js 구현 (Google OAuth)
├── [ ] User 모델 & Prisma 스키마
├── [ ] 기본 라우팅 (React Router)
├── [ ] shadcn/ui 컴포넌트 세팅
├── [ ] WhyBitcoinFallen.com 개발
│   ├── 실시간 BTC 가격
│   ├── Fear & Greed 게이지
│   ├── AI 미리보기
│   └── Sage.ai CTA
└── [ ] Sage.ai 랜딩 페이지
    ├── Hero 섹션
    ├── 3가지 핵심 기능 소개
    └── 성과 카드 (섀도우 포트폴리오 예시)
```

**마일스톤**: 인증 시스템 완성 + 2개 사이트 배포

### 2.3 Month 2: 점진적 기능 출시

#### Release 1: 기본 채팅 (Week 5-6)

```typescript
interface Release1Features {
  claudeIntegration: {
    sdk: "@anthropic-ai/sdk";
    streaming: "Nest.js SSE";
  };
  multiAgentSystem: {
    version: "v0.1";
    agents: [
      { name: "Manager Agent", model: "Haiku 4", role: "라우팅" },
      { name: "Analyst Agent", model: "Haiku 4", role: "데이터 수집" },
      { name: "Persona Agent", model: "Sonnet 4", role: "월렛 버핏 응답" },
      { name: "Risk Agent", model: "Haiku 4", role: "교차 검증" }
    ];
  };
  dataIntegration: {
    coins: ["BTC", "ETH", "SOL", "BNB", "DOGE", "XRP"];
    apis: ["CoinGecko", "Alternative.me Fear & Greed"];
    caching: "Redis 5분";
  };
}
```

**출시 정보**:
- 대상: 베타 테스터 10명 초대
- 목표: 환각률 <1% 검증, 응답 속도 2초 확인
- 피드백: Discord #feedback 채널 운영

**마일스톤**: 첫 베타 유저와 대화 성공 + 환각 제로 검증

#### Release 2: 섀도우 포트폴리오 (Week 7-8)

```
섀도우 포트폴리오 개발
├── [ ] 대화에서 프로필 자동 추론
│   ├── 투자 성향 (공격적/보수적)
│   ├── 관심 코인
│   └── 투자 목표
├── [ ] 섀도우 트레이드 모델
├── [ ] AI 시그널 감지 (buy/sell)
├── [ ] "담아보기" 버튼 (메시지 → 포트폴리오)
├── [ ] 현재가 조회 & 저장
├── [ ] 수익률 계산 로직
└── [ ] 포트폴리오 대시보드
```

**출시 정보**:
- 대상: 베타 테스터 20명으로 확장
- 목표: 섀도우 트레이드 10+ 추적
- 피드백: 수익률 계산 정확도, UX 개선

**마일스톤**: 첫 섀도우 포트폴리오 수익률 공개

### 2.4 Month 3: 능동적 분석 & 공식 런칭

#### Release 3: 능동적 분석 (Week 9-10)

```typescript
interface MarketAlertConfig {
  polling: {
    scheduler: "@nestjs/schedule";
    interval: "15분";
    queue: "BullMQ";
  };
  thresholds: {
    btc: "±5%";
    eth: "±7%";
    altcoins: "±10%";
    fearGreed: "±15";
  };
  notifications: {
    pwa: "Web Push API";
    discord: "Webhook";
    deeplink: "/chat/new?context=xxx";
  };
}
```

**마일스톤**: 첫 실시간 급변 알림 발송 성공

#### Public Launch (Week 11-12)

```
런칭 준비 체크리스트
├── [ ] 최종 QA 테스트
├── [ ] 성능 최적화
│   ├── 응답 속도 2초 이내 확인
│   ├── 컨텍스트 로드 0.5초 확인
│   └── 환각률 <1% 검증
├── [ ] ECS Fargate Auto Scaling 설정
├── [ ] 모니터링 설정 (Sentry, CloudWatch)
├── [ ] Product Hunt 페이지 준비
└── [ ] Twitter 론칭 스레드 준비
```

**런칭 정보**:
- D-Day: Week 12 월요일
- 전략: WhyBitcoinFallen.com + Product Hunt + Reddit
- 목표: 첫 100명 가입, Discord 200+ members

**마일스톤**: MVP 공식 런칭 + 첫 주 MAU 500+

### 2.5 점진적 출시의 장점

```
점진적 출시 전략
│
├── 빠른 피드백 루프
│   ├── Release 1 → 채팅 UX 검증 (2주)
│   ├── Release 2 → 포트폴리오 검증 (2주)
│   └── Release 3 → 알림 검증 (2주)
│
├── 위험 분산
│   ├── 한번에 모든 기능 출시 시 버그 폭발 위험
│   └── 기능별로 안정화 후 다음 단계 진행
│
├── 사용자 성장
│   ├── Release 1: 10명 (코어 피드백)
│   ├── Release 2: 20명 (확장 테스트)
│   └── Release 3: 100명 (Public Beta)
│
└── 마케팅 모멘텀
    ├── "계속 성장하는 서비스" 이미지
    └── 각 기능 출시마다 Discord/Twitter 업데이트
```

### 2.6 Phase 1 Success Criteria

| 지표 | 목표 | 측정 방법 |
|------|------|----------|
| 베타 테스터 | 10-20명 | 직접 모집 |
| WhyBitcoinFallen 방문자 | 일 1,000+ | Google Analytics |
| Sage.ai MAU | 500+ | 활성 사용자 수 |
| 환각률 | <1% | 사용자 신고 / 전체 응답 |
| Shadow Portfolio 추적 | 10+ 트레이드 | DB 기록 |
| NPS | 40+ | 베타 테스터 설문 |
| 응답 속도 | 2초 이내 | Sentry Traces |
| 가동률 | 99%+ | CloudWatch Uptime |

---

## 3. Phase 2: Scale (2026 Q2-Q3, 6개월)

### 3.1 목표

**"글로벌 확장 & 제품 고도화"**

### 3.2 Q2 (4-6월): 글로벌 & 멀티 페르소나

#### 멀티 페르소나 추가

```typescript
interface PersonaConfig {
  satoshiSage: {
    name: "사토시 현자 (Satoshi Sage)";
    philosophy: "비트코인 원리주의";
    focus: ["탈중앙화", "보안 중심", "장기 HODLing"];
  };
  alphaHunter: {
    name: "알파 헌터 (Alpha Hunter)";
    philosophy: "단기 트레이딩";
    focus: ["기술적 분석", "차트/지표", "고위험 고수익"];
  };
}
```

#### 멀티 언어 지원

```
언어 지원 체크리스트
├── [ ] i18n 설정 (react-i18next)
├── [ ] 4개 언어 지원
│   ├── 한국어 (기본)
│   ├── 영어
│   ├── 일본어
│   └── 중국어 (간체)
├── [ ] 페르소나별 언어 맞춤형 응답
└── [ ] 각 언어별 랜딩 페이지
```

#### 그룹 채팅 (AI끼리 토론)

```
그룹 채팅 기능
├── [ ] 멀티 에이전트 토론 시스템
├── [ ] 사용자 질문 → 3명의 AI가 토론
├── [ ] 최종 합의 or 다수결
└── [ ] 그룹 채팅 UI (3개 아바타)
```

**마일스톤**: MAU 20,000 + 4개 언어 런칭

### 3.3 Q3 (7-9월): 지표 개선 & 유료화

#### RAG 장기 기억

```typescript
interface RAGConfig {
  vectorDB: "pgvector (PostgreSQL)";
  embedding: "OpenAI text-embedding-3";
  contextExpansion: "20개 → 무제한";
  search: "유사도 기반 메모리 검색";
}
```

#### 온체인 분석

```
온체인 분석 기능
├── [ ] Etherscan / BscScan API 연동
├── [ ] 고래 지갑 추적 (Whale Alert)
├── [ ] 대량 이동 감지
└── [ ] 온체인 지표 Tool 추가
    ├── get_whale_movements
    └── get_exchange_flows
```

#### 유료 플랜 런칭

| 티어 | 가격 | 기능 |
|------|------|------|
| Free | $0 | 기본 채팅, 1개 포트폴리오 |
| Pro | $19.99 | 무제한 채팅, 3개 포트폴리오 |
| Premium | $49.99 | 모든 기능 + API |

#### 추가 코인 지원

- 10개 추가 (총 16개)
- ADA, DOT, AVAX, MATIC, LINK, UNI, ATOM, LTC, XLM, ALGO

**마일스톤**: MAU 40,000 + 유료 구독자 500명 + MRR $10,000

---

## 4. Phase 3: Monetize (2026 Q4, 3개월)

### 4.1 목표

**"수익 모델 최적화 & 커뮤니티 강화"**

### 4.2 고급 분석 기능

```typescript
interface AdvancedAnalytics {
  portfolioMetrics: [
    "샤프 비율 (Sharpe Ratio)",
    "MDD (Maximum Drawdown)",
    "베타 (Beta)"
  ];
  backtesting: {
    description: "과거 AI 추천 시뮬레이션";
    output: "가상 성과 계산";
  };
  customAlerts: {
    priceTargets: boolean;
    indicatorBased: boolean;
  };
}
```

### 4.3 API 제공 (Premium)

```
API 엔드포인트
├── /api/v1/chat - AI 채팅
├── /api/v1/analysis - 시장 분석
└── /api/v1/portfolios - 포트폴리오 조회
```

### 4.4 커뮤니티 & 소셜

```
Discord 서버 구조
├── #announcements
├── #market-alerts
├── #portfolio-showcase
└── #general-chat
```

**마일스톤**: MAU 100,000 + MRR $30,000 + Discord DAU 1,000

---

## 5. Phase 4: Ecosystem (2027+)

### 5.1 목표

**"생태계 확장 - 암호화폐를 넘어"**

### 5.2 주식 시장 진출 (2027 Q1-Q2)

```typescript
interface StockExpansion {
  personas: [
    { name: "피터 린치", style: "성장주 투자" },
    { name: "벤저민 그레이엄", style: "가치 투자" }
  ];
  dataProviders: ["Yahoo Finance API", "Alpha Vantage", "KRX"];
  tools: ["get_stock_fundamentals", "get_stock_technicals"];
}
```

### 5.3 DeFi 통합 (2027 Q3-Q4)

```
DeFi 기능
├── Uniswap / PancakeSwap 가격 조회
├── Yield Farming APY 추적
├── 스테이킹 추천
└── 가스비 최적화 제안
```

### 5.4 전체 자산군 (2028+)

- 부동산 (REITs)
- 채권 (Bonds)
- 원자재 (Commodities)
- 외환 (Forex)

**비전**: **"모든 자산군을 아우르는 AI 투자 플랫폼"**

---

## 6. Technical Debt Management

### 6.1 정기 리팩토링

| 분기 | 리팩토링 주제 |
|------|-------------|
| 2026 Q2 | Database 인덱스 최적화, N+1 쿼리 제거 |
| 2026 Q3 | 멀티 에이전트 성능 튜닝, Redis 캐싱 전략 재검토 |
| 2026 Q4 | TypeScript strict mode 전환, ESLint 규칙 강화 |
| 2027 Q1 | Monorepo로 전환 (Nx / Turborepo) |

### 6.2 보안 감사

- **2026 Q2**: 외부 보안 감사 (침투 테스트)
- **2026 Q4**: OWASP Top 10 체크
- **2027 Q1**: SOC 2 인증 준비

---

## 7. Infrastructure Evolution

### 7.1 2026년 인프라 변화

| 분기 | MAU | 인프라 |
|------|-----|--------|
| Q1 | 5,000 | ECS Fargate (2 tasks) |
| Q2 | 20,000 | ECS Fargate (5 tasks) + Auto Scaling |
| Q3 | 40,000 | ECS Fargate (10 tasks) + CloudFront 최적화 |
| Q4 | 100,000 | ECS Fargate (20 tasks) + Multi-Region 고려 |

### 7.2 2027년 이후

- **Multi-Region**: US West, Asia (Seoul), EU (Frankfurt)
- **CDN 강화**: CloudFlare Enterprise
- **Database Sharding**: 사용자 기반 샤딩
- **Microservices**: Chat, Portfolio, Notification 분리

---

## 8. Team Growth

### 8.1 팀 확장 계획

| 시기 | 팀 규모 | 새로운 역할 |
|------|---------|------------|
| 2026 Q1 | 5명 | 현재 팀 |
| 2026 Q2 | 8명 | +3명 (Frontend 2, QA 1) |
| 2026 Q3 | 12명 | +4명 (Backend 2, DevOps 1, Product Manager 1) |
| 2026 Q4 | 15명 | +3명 (AI Engineer 1, Data Analyst 1, Designer 1) |
| 2027 Q1 | 20명 | +5명 (각 영역 확장) |

---

## 9. Budget & Resources

### 9.1 2026년 분기별 예산

| 분기 | 인건비 | 인프라 | AI API | 마케팅 | 총액 |
|------|--------|--------|--------|--------|------|
| Q1 | $75K | $6K | $4.5K | $9K | **$94.5K** |
| Q2 | $120K | $12K | $10K | $20K | **$162K** |
| Q3 | $180K | $20K | $20K | $40K | **$260K** |
| Q4 | $240K | $30K | $30K | $60K | **$360K** |
| **Total** | **$615K** | **$68K** | **$64.5K** | **$129K** | **$876.5K** |

---

## 10. Key Metrics Dashboard

### 10.1 2026년 목표 (분기별)

| 지표 | Q1 | Q2 | Q3 | Q4 |
|------|----|----|----|----|
| MAU | 5K | 20K | 40K | 100K |
| 유료 구독자 | 50 | 200 | 500 | 1,500 |
| MRR | $1K | $4K | $10K | $30K |
| ARR | $12K | $48K | $120K | $360K |
| NPS | 40+ | 45+ | 50+ | 55+ |
| Churn Rate | <10% | <7% | <5% | <5% |

---

## 11. Risk Mitigation

### 11.1 기술 리스크

| 리스크 | 완화 전략 | 책임자 |
|--------|----------|--------|
| Claude API 장애 | 캐싱 강화, Rate limit 조정, Anthropic 지원팀 연락 | CTO |
| 데이터베이스 과부하 | Read Replica, Redis 캐싱 | DevOps |
| 환각 증가 | 멀티 에이전트 재검증, Tool 강제 | AI Engineer |

### 11.2 비즈니스 리스크

| 리스크 | 완화 전략 | 책임자 |
|--------|----------|--------|
| 시장 침체 | "공포가 기회" 메시지 강화 | CEO |
| 경쟁사 진입 | 페르소나 IP 강화, 빠른 출시 | Product Manager |
| 규제 변화 | 법률 자문, 투자 교육 포지셔닝 | CEO |

---

## 12. Conclusion

### 12.1 2026년 목표 요약

**"3개월 MVP → 1년 안에 MAU 100,000 달성"**

### 12.2 핵심 마일스톤

```
2026년 마일스톤
├── 2026 Q1: MVP 런칭, PMF 검증 [완료 예정]
├── 2026 Q2: 글로벌 4개 언어, 멀티 페르소나 [목표]
├── 2026 Q3: 유료 플랜 런칭, 수익화 시작 [목표]
├── 2026 Q4: 커뮤니티 확장, 생태계 구축 [목표]
└── 2027+: 주식 시장 진출, 전체 자산군 확장 [비전]
```

### 12.3 비전

**"모든 개인 투자자에게 월가 수준의 AI 멘토를"**

---

## Appendix

### A. Feature Flag 관리

- [링크: Feature Flag Strategy]

### B. 배포 일정

- [링크: Deployment Schedule]

### C. 테스트 전략

- [링크: QA & Testing Strategy]

---

**문서 끝**

_"Between the zeros and ones"_
