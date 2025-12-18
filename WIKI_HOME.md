# Sage.ai Documentation

> **"모든 개인 투자자에게 월가 수준의 AI 멘토를"**
> _AI-powered investment mentor for every crypto investor_

---

## 🧠 프로젝트 시놉시스

### "자네, 시장이 공포에 질렸을 때가 바로 기회일세"

암호화폐 시장은 24시간 멈추지 않습니다.
정보는 넘쳐나지만, 누구를 믿어야 할지 알 수 없습니다.
리딩방의 70%는 사기, 범용 AI는 투자를 모릅니다.

**Sage.ai**는 이 문제를 해결합니다.

워렌 버핏의 투자 철학을 구현한 AI 멘토 **월렛 버핏**이
당신의 24시간 투자 파트너가 됩니다.

- **Claude 3.5 기반 멀티 에이전트 시스템**으로 환각을 제로화
- **쉐도우 포트폴리오**로 AI의 실력을 투명하게 검증
- **능동적 알림**으로 시장 급변 시 먼저 다가가는 멘토

**Sage.ai**는 단순한 챗봇이 아닙니다.
정보 과부하 속에서 **통찰**을 제공하고,
검증 불가능한 리딩방 대신 **신뢰**를 구축하며,
혼자 판단하는 외로움 대신 **케어**를 제공하는
**차세대 AI 투자 멘토 플랫폼**입니다.

_"Between fear and greed, lies wisdom."_
_"공포와 탐욕 사이에 지혜가 존재합니다."_

---

## 📍 시작하기

- **[Executive Brief](docs/business/executive-brief.md)** - 1페이지 프로젝트 개요
- **[MVP Definition](docs/business/mvp-definition.md)** - 8주 개발 범위 및 일정
- **[Backend Specification](docs/technical/backend-spec.md)** - 기술 아키텍처 상세

---

## 📚 전체 문서

### 🏢 Business & Strategy

- [Business Plan](docs/business/business-plan.md) - 시장 분석 및 수익 모델
- [MVP Definition](docs/business/mvp-definition.md) - MVP 범위 및 성공 기준
- [Executive Brief](docs/business/executive-brief.md) - 투자자용 1페이지 요약
- [Branding Guide](docs/business/branding-guide.md) - 월렛 버핏 브랜드 정체성
- [AI-Native Team Strategy](docs/business/ai-team-strategy.md) - AI 기반 개발 전략
- [Development Roadmap](docs/business/development-roadmap.md) - 8주 + 2026년 로드맵
- [Market Analysis](docs/business/market-analysis.md) - 경쟁 분석 및 타겟 시장

### 🔧 Technical Architecture

- [Backend Specification](docs/technical/backend-spec.md) - Django + DRF + Clean Architecture Lite + Claude 에이전트
- [Frontend Specification](docs/technical/frontend-spec.md) - React 18.3 + Vite 5 (완전 분리 SPA)
- [Infrastructure Design](docs/technical/infrastructure-spec.md) - AWS ECS Fargate, RDS, Redis
- [Database Schema](docs/technical/database-schema.md) - Django ORM 스키마
- [API Documentation Strategy](docs/technical/api-documentation-strategy.md) - REST API 명세 (drf-spectacular)
- [Security & Compliance](docs/technical/security-compliance.md) - 보안 및 법적 대응

### 🎨 Product Design

- [Product Specification](docs/product/product-spec.md) - 3가지 핵심 기능 상세
- [User Journey Map](docs/product/user-journey-map.md) - 사용자 경험 설계
- [Persona Design](docs/product/persona-design.md) - 월렛 버핏 캐릭터 설계
- [Feature Roadmap](docs/product/feature-roadmap.md) - MVP vs Phase 2 기능 비교
- [UX Design Guide](docs/product/ux-guide.md) - 디자인 원칙 및 컴포넌트

### 🚀 Operations & Marketing

- [Operations Roadmap](docs/operations/operations-roadmap.md) - 2026년 분기별 일정
- [GTM Strategy](docs/operations/gtm-strategy.md) - Dual Hook (바이럴 + 랜딩)
- [Viral Campaign](docs/operations/viral-campaign.md) - WhyBitcoinFallen.com 전략
- [Community Management](docs/operations/community-management.md) - Discord 운영
- [Live Operations Playbook](docs/operations/live-ops-playbook.md) - 출시 후 KPI 추적

### 🤖 AI Development Guides

- [Backend AI Guide](docs/ai-guides/backend-ai-guide.md) - Claude Code로 API Routes 개발
- [Frontend AI Guide](docs/ai-guides/frontend-ai-guide.md) - Cursor로 React 컴포넌트 개발
- [Infrastructure AI Guide](docs/ai-guides/infrastructure-ai-guide.md) - AWS CDK/Terraform 자동화
- [Prompt Engineering Guide](docs/ai-guides/prompt-engineering-guide.md) - 월렛 버핏 프롬프트 튜닝

---

## 🎯 핵심 가치

### 신뢰 (Trust)

**환각 제로 + 투명한 성과 검증**

- 멀티 에이전트 교차 검증으로 Fact 오류 원천 차단
- 쉐도우 포트폴리오로 AI 조언의 수익률을 투명하게 추적
- 목표: ROI +18.5%, 적중률 72.3%, 벤치마크 대비 +3%p

### 지혜 (Wisdom)

**단순 정보가 아닌 통찰 제공**

- 월렛 버핏 페르소나: 워렌 버핏의 투자 철학 구현
- "공포에 사라, 탐욕에 팔아라" - 반직관적 지혜 전달
- 대화 기반 프로필 추론으로 맞춤형 조언

### 케어 (Care)

**24시간 대응 + 능동적 멘탈 케어**

- 시장 급변 시 먼저 알려주는 능동적 알림 (PWA Push + Discord)
- 15분 간격 자동 시장 분석 및 컨텍스트 생성
- 패닉셀, FOMO 방지 멘탈 케어

---

## 🏗️ 기술적 우수성

### AI-Native Development

- **Claude 3.5 Sonnet/Haiku** 멀티 에이전트 오케스트레이션
- **Vercel AI SDK** 스트리밍으로 2초 이내 응답 시작
- **Tool 강제 사용**으로 수치 계산 오류 제로화

### 확장성 설계

- **ECS Fargate** → 1만 유저까지 안정적
- **Redis 캐싱** → API 비용 절감 (시장 데이터 5분, AI 코멘트 5분)
- **S3 + CloudFront** → WhyBitcoinFallen.com 정적 호스팅

### 성능 목표

| 메트릭 | 목표 |
|-------|------|
| 채팅 응답 시작 | 2초 이내 |
| 알림 발송 간격 | 15분 자동 분석 |
| 컨텍스트 로드 | 0.5초 (20개 메시지) |
| 동시 접속자 | 1,000명 (MVP) |

---

## 🚀 8주 MVP 로드맵

### Week 1-2: 기반 구축

- AWS 인프라 (ECS, RDS, Redis)
- Auth.js Google OAuth
- WhyBitcoinFallen.com (Vite + React)
- Sage.ai 랜딩 페이지

### Week 3-4: 핵심 채팅

- Claude API 연동 + SSE 스트리밍
- 월렛 버핏 페르소나 구현
- 시장 데이터 연동 (CoinGecko, CryptoPanic)
- 유저 프로필 자동 추론

### Week 5-6: 포트폴리오 + 분석

- 섀도우 포트폴리오 시스템
- AI 시그널 + [담아보기] 기능
- 뉴스 + Fear & Greed 통합

### Week 7-8: 알림 + 마무리

- PWA Push + Discord 알림
- 딥링크 시스템 (/chat/new?context=xxx)
- QA + ECS Fargate 배포
- **베타 런칭!**

---

## 📊 2026년 비전

### Q1: MVP 검증

- WhyBitcoinFallen 일 방문자 10K+
- Sage.ai MAU 5K+

### Q2: 글로벌 확장

- 멀티 채팅 + 4개 언어 (한/영/일/중)
- MAU 20K+

### Q3: 지표 개선

- 유료 구독 런칭 (Pro $19.99, Premium $49.99)
- MAU 40K+

### Q4: ICO 본격화

- 생태계 확장 + 토큰 상장
- MAU 100K+

**2026년 말 목표**:
- WhyBitcoinFallen 일 방문자 **100,000+**
- Sage.ai MAU **10,000+**
- 유료 구독자 **500+** (전환율 5%)

---

> **"자네, 시장이 공포에 질렸을 때가 바로 기회일세"**
> _When fear grips the market, opportunity reveals itself_

우리는 단순히 AI 챗봇을 만드는 것이 아닙니다.
정보 과부하 속에서 지혜를 찾고,
검증 불가능한 신뢰를 구축하며,
혼자가 아닌 멘토와 함께 성장하는
**새로운 투자 패러다임**을 만들고 있습니다.

**월렛 버핏이 기다립니다.**
**당신은 준비되었나요?**

_"May your portfolio grow deep!"_
_"당신의 포트폴리오가 깊이 성장하기를!"_ 📈

---

**Last Updated**: 2025년 12월 17일
**Version**: 1.0
**Status**: Phase 1 - 핵심 3개 문서 완성 (WIKI_HOME, CLAUDE, backend-spec)
