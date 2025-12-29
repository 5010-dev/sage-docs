# Sage.ai - AI 투자 멘토 플랫폼

## 월렛 버핏과 함께하는 암호화폐 투자

> **"모든 개인 투자자에게 월가 수준의 AI 멘토를"**

[![MVP Development](https://img.shields.io/badge/status-3%EA%B0%9C%EC%9B%94%20MVP%20%EA%B0%9C%EB%B0%9C-orange)](https://github.com/sage-ai/sage-platform)
[![Version](https://img.shields.io/badge/version-4.0-blue)]()
[![License](https://img.shields.io/badge/license-Proprietary-red)](LICENSE)

---

## 핵심 가치 제안

**Sage.ai**는 워렌 버핏의 투자 철학을 구현한 AI 멘토 '월렛 버핏'이 24시간 당신의 암호화폐 투자를 돕는 플랫폼입니다.

### 3가지 핵심 기능

1. **월렛 버핏과의 대화**
   - Claude Sonnet 4 기반 실시간 AI 멘토링
   - 대화를 통해 자연스럽게 투자 성향 파악
   - 시장 데이터 + 투자 철학 기반 통찰 제공

2. **섀도우 포트폴리오**
   - "버핏 따라가기" - AI 추천을 가상으로 추적
   - 투명한 성과 검증 (목표: 벤치마크 +3%p)
   - 신뢰 구축의 핵심 기능

3. **능동적 분석**
   - 15분마다 시장 자동 분석 및 컨텍스트 생성
   - 급변 시 PWA Push + Discord 알림
   - 딥링크로 즉시 관련 채팅 시작

---

## 📚 문서 네비게이션

> **모든 프로젝트 문서는 이 저장소에서 관리됩니다** (단일 진실 공급원)

### 빠른 시작

| 문서 | 설명 | 대상 |
|------|------|------|
| **[Wiki Home](wiki/home.md)** | 전체 문서 인덱스 | 모든 팀원 |
| **[CLAUDE.md](CLAUDE.md)** | AI 개발 가이드 (Claude Code) | 개발자 |
| **[AI Team Strategy](docs/business/ai-team-strategy.md)** | AI-First 개발 전략 (Claude Code + GitHub Copilot) | 개발팀, 경영진 |
| **[MVP_technical_paper.md](MVP_technical_paper.md)** | MVP 기술 스택 명세 | 개발팀 |
| **[Business Plan](docs/business/business-plan.md)** | 비즈니스 플랜 | 경영진, 투자자 |
| **[MVP Definition](docs/business/mvp-definition.md)** | MVP 정의 및 범위 | 전체 팀 |

### 문서 디렉토리

- **[docs/business/](docs/business/)** - 사업 계획, 시장 분석, GTM 전략, AI 팀 전략
- **[docs/technical/](docs/technical/)** - 백엔드/프론트엔드/인프라 명세서, DB 스키마, API 문서
- **[docs/operations/](docs/operations/)** - 운영 로드맵, 커뮤니티 관리, 라이브 운영
- **[docs/ai-guides/](docs/ai-guides/)** - AI 기반 개발 가이드 (Backend/Frontend/Infrastructure)

### 서비스 구성 (MVP)

- **WhyBitcoinFallen.com** - 바이럴 마케팅 사이트
- **Sage.ai 랜딩** - 서비스 소개 페이지
- **Sage.ai 앱** - 메인 AI 멘토링 플랫폼

---

## 프로젝트 구조

### 비즈니스 & 전략

| 섹션 | 내용 |
|------|------|
| **비전** | "모든 개인 투자자에게 월가 수준의 AI 멘토를" |
| **문제 정의** | 정보 과부하, 신뢰 간극, 24시간 대응 불가 |
| **솔루션** | 멀티 에이전트 시스템 + 섀도우 포트폴리오 + 능동적 분석 |
| **시장 기회** | 글로벌 암호화폐 투자자 5.6억 명 |
| **수익 모델** | Freemium (Free / Pro $19.99 / Premium $49.99) |
| **GTM 전략** | Dual Hook (WhyBitcoinFallen.com + Sage.ai 랜딩) |

### 기술 아키텍처

| 컴포넌트 | 기술 스택 |
|----------|-----------|
| **백엔드** | Nest.js 10.x (TypeScript 네이티브, 모듈러 구조) |
| **프론트엔드** | React 18.3 + Vite 5 + TypeScript (완전 분리 SPA) |
| **아키텍처** | Layered + Domain 분리 (Clean Lite) |
| **ORM** | Prisma 5.x (타입 안정성, 직관적 마이그레이션) |
| **인증** | Auth.js (@auth/core) with Google OAuth |
| **AI** | Claude Sonnet 4 + Haiku 4 (@anthropic-ai/sdk) |
| **실시간** | Nest.js 내장 SSE (Server-Sent Events) |
| **Database** | PostgreSQL 18 (RDS) + Valkey 8.x (ElastiCache) |
| **상태 관리** | Zustand 4.x (클라이언트) + TanStack Query 5.x (서버) |
| **Infrastructure** | ECS Fargate + S3/CloudFront |
| **비동기 작업** | BullMQ 5.x + Valkey (Memory 추출, 알림 발송) |
| **스케줄링** | @nestjs/schedule (15분 가격 폴링) |
| **모니터링** | Sentry + CloudWatch + OpenTelemetry |

### AI 에이전트 시스템

```
[사용자 질문]
    ↓
[Manager Agent] - 의도 파악 및 라우팅 (Haiku 4)
    ↓
[Analyst Agent] - 뉴스/가격 API로 Fact 수집 (Haiku 4)
    ↓
[Persona Agent] - 월렛 버핏이 철학 기반 해석 (Sonnet 4)
    ↓
[Risk Agent] - 오류/편향 검증 (Haiku 4)
    ↓
[최종 응답]
```

### 컨텍스트 관리 (단순화)

- **최근 20개 메시지만** 유지 (RAG는 Phase 2)
- **대화에서 프로필 추론** (온보딩 설문 없음)
- **추가 AI 호출 없음** - 비용 최적화

---

## 3개월 MVP 로드맵

### Month 1: 기반 구축

- 프로젝트 셋업 + AWS 인프라
- Auth.js 인증 (Google OAuth)
- WhyBitcoinFallen.com 개발 (Vite + React)
- Sage.ai 랜딩 페이지 개발

### Month 2: 핵심 채팅 & 포트폴리오

- Claude API 연동 + Nest.js SSE 스트리밍
- 월렛 버핏 페르소나 구현
- 시장 데이터 연동 (CoinGecko, Alternative.me)
- 유저 프로필 자동 추론
- 섀도우 포트폴리오 시스템

### Month 3: 능동적 분석 & 마무리

- BullMQ 기반 Memory 추출/압축
- @nestjs/schedule 15분 자동 시장 분석
- PWA Push + Discord 알림
- 딥링크 시스템 (/chat/new?context=xxx)
- QA + ECS Fargate 배포
- 베타 런칭

---

## 2026년 분기별 목표

| 분기 | 핵심 목표 | 주요 지표 |
|------|-----------|-----------|
| **Q1** | MVP 검증 | WhyBitcoinFallen 월 방문자 10K+<br>Sage.ai MAU 2K+ |
| **Q2** | 글로벌 확장 | 멀티 채팅 + 4개 언어<br>MAU 5K+ |
| **Q3** | 지표 개선 | 유료 구독 런칭<br>MAU 15K+ |
| **Q4** | ICO 본격화 | 생태계 확장<br>MAU 30K+ |

### 2026년 말 예상 성과

- WhyBitcoinFallen.com 월 방문자: 100,000+
- Sage.ai MAU: 30,000+
- 유료 구독자: 450+ (전환율 1.5%)
- Discord DAU: 1,000+
- Shadow Portfolio ROI: 벤치마크 +3%p

---

## 기술적 우수성

### AI-Native 개발

- Claude Code 활용으로 10배 빠른 개발
- 멀티 에이전트 오케스트레이션으로 환각 제로화
- Tool 강제 사용으로 수치 오류 원천 차단

### 성능 목표

- 채팅 응답 속도: 2초 이내 (스트리밍 시작)
- 능동적 분석: 15분 간격 자동 시장 분석
- 컨텍스트 로드: 20개 메시지 (0.5초 이내)

### 확장성 설계

- ECS Fargate: 1만 유저까지 안정적
- Valkey 캐싱: API 비용 절감 (시장 데이터 5분, AI 코멘트 5분)
- S3 + CloudFront: Sites 정적 호스팅

---

## 핵심 경쟁력 (Moat)

| Moat 유형 | 설명 | 방어력 |
|-----------|------|--------|
| **Persona Moat** | 월렛 버핏 캐릭터 IP | 복제 불가능 |
| **Trust Moat** | 환각 제로 + 쉐도우 포트폴리오 | 신뢰 확보 |
| **Engagement Moat** | 능동적 분석 시스템 (15분 자동 컨텍스트 생성) | 재방문율 증가 |
| **Data Moat** | 쉐도우 포트폴리오 누적 데이터 | 시간에 따라 강화 |

---

## 시장 기회

### 글로벌 암호화폐 시장

- 시장 규모: $2.5조 (2024) → $6.3조 (2030, CAGR 9.7%)
- 사용자: 5.6억 명 (2024) → 8.6억 명 (2025 예상)

### AI 투자자문 시장

- 시장 규모: $84억 (2024) → $693억 (2032, CAGR 30.3%)
- 밀레니얼/Z세대의 41%가 로보어드바이저 선호

### 타겟 시장 규모

- TAM: 글로벌 암호화폐 투자자 5.6억 명
- SAM: AI 투자 도구 관심 투자자 약 1억 명
- SOM: 초기 타겟 4개 권역 유료 전환 가능 약 500만 명

---

## 기여하기

### 개발팀을 위해

1. **[MVP_technical_paper.md](MVP_technical_paper.md)** - 기술 스택 명세
2. **[MVP Spec (KR)](docs/technical/mvp-spec-kr.md)** - MVP 개발 스펙
3. 현재 월 계획 확인
4. 기술 스택 및 폴더 구조 숙지

### AI-Native Development

- Claude Sonnet 4: 메인 대화, Persona Agent
- Claude Haiku 4: 라우팅, 팩트체크, Memory 추출
- @anthropic-ai/sdk: TypeScript 직접 통합
- Nest.js: 멀티 에이전트 병렬 처리

---

## 연락처

- **Technical**: [dev@5010.tech](mailto:dev@5010.tech)
- **Business**: [biz@5010.tech](mailto:biz@5010.tech)
- **Discord**: (베타 테스터 모집 시 공개)

---

## 비전

### Mission

**"모든 개인 투자자에게 월가 수준의 AI 멘토를"**

### 핵심 가치

1. 신뢰 (Trust) - 환각 제로 + 투명한 성과 검증
2. 지혜 (Wisdom) - 단순 정보가 아닌 통찰 제공
3. 케어 (Care) - 24시간 대응 + 능동적 멘탈 케어

### 확장 계획

- Phase 1 (2026 Q1-Q4): 암호화폐 AI 멘토 확립
- Phase 2 (2027): 주식 시장 확장
- Phase 3 (2028+): 전체 자산군 (부동산, 채권 등)

---

## MVP 성공 기준

| 지표 | 목표 (3개월 후) |
|------|----------------|
| **베타 테스터** | 10-20명 |
| **WhyBitcoinFallen 방문자** | 일 1,000+ |
| **Sage.ai MAU** | 500+ |
| **환각률** | <1% |
| **Shadow Portfolio 추적** | 10+ 트레이드 |

---

_"자네, 시장이 공포에 질렸을 때가 바로 기회일세."_ - 월렛 버핏

**Built with**: Nest.js 10.x + Prisma 5.x, React 18.3 + Vite 5, Tailwind CSS 3.x, Claude Sonnet 4 & Haiku 4
**Architecture**: Layered + Domain (Clean Lite), TypeScript Fullstack
**Target**: 3개월 MVP 완성, 2026년 MAU 10K+
**Version**: 4.0 (2025년 12월 19일) - Nest.js + Prisma Stack
