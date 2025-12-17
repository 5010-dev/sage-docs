# Sage.ai - AI 투자 멘토 플랫폼

## 월렛 버핏과 함께하는 암호화폐 투자

> **"모든 개인 투자자에게 월가 수준의 AI 멘토를"**

[![MVP Development](https://img.shields.io/badge/status-8%EC%A3%BC%20MVP%20%EA%B0%9C%EB%B0%9C-orange)](https://github.com/sage-ai/sage-platform)
[![Version](https://img.shields.io/badge/version-3.0-blue)]()
[![License](https://img.shields.io/badge/license-Proprietary-red)](LICENSE)

---

## 핵심 가치 제안

**Sage.ai**는 워렌 버핏의 투자 철학을 구현한 AI 멘토 '월렛 버핏'이 24시간 당신의 암호화폐 투자를 돕는 플랫폼입니다.

### 3가지 핵심 기능

1. **월렛 버핏과의 대화**
   - Claude 3.5 기반 실시간 AI 멘토링
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

## 빠른 시작

### 주요 문서

| 문서 | 설명 | 대상 |
|------|------|------|
| **[01-SAGE-overview-kr.md](01-SAGE-overview-kr.md)** | 사업 기획서 (v2.2) | 경영진, 투자자 |
| **[06-MVP-Spec-ko.md](06-MVP-Spec-ko.md)** | MVP 개발 스펙 (8주) | 개발팀 |

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
| **백엔드** | Django 5.1+ + Django REST Framework |
| **프론트엔드** | Vite 6 + React 19 + TypeScript (완전 분리 SPA) |
| **아키텍처** | Hexagonal (Ports & Adapters) + 완전 백엔드/프론트엔드 분리 |
| **ORM** | Django ORM (PostgreSQL) |
| **인증** | Django-Allauth (Google OAuth) |
| **AI** | Claude 3.5 Sonnet + Haiku (Anthropic Python SDK) |
| **스트리밍** | Django Channels (SSE) |
| **Database** | PostgreSQL (RDS) + Redis (ElastiCache) |
| **감사 로그** | 별도 audit_log 테이블 (모든 중요 작업 추적) |
| **Infrastructure** | ECS Fargate + S3/CloudFront |
| **비동기 작업** | Celery + Redis (15분 자동 시장 분석) |
| **API 문서** | drf-spectacular (OpenAPI 자동 생성) |

### AI 에이전트 시스템

```
[사용자 질문]
    ↓
[Manager Agent] - 의도 파악 및 라우팅
    ↓
[Analyst Agent] - 뉴스/가격 API로 Fact 수집
    ↓
[Persona Agent] - 월렛 버핏이 철학 기반 해석
    ↓
[Risk Agent] - 오류/편향 검증
    ↓
[최종 응답]
```

### 컨텍스트 관리 (단순화)

- **최근 20개 메시지만** 유지 (RAG는 Phase 2)
- **대화에서 프로필 추론** (온보딩 설문 없음)
- **추가 AI 호출 없음** - 비용 최적화

---

## 8주 MVP 로드맵

### Phase 1: 기반 구축 (Week 1-2)

- 프로젝트 셋업 + AWS 인프라
- Django-Allauth 인증 (Google OAuth)
- WhyBitcoinFallen.com 개발 (Vite + React)
- Sage.ai 랜딩 페이지 개발

### Phase 2: 핵심 채팅 (Week 3-4)

- Claude API 연동 + Django Channels SSE 스트리밍
- 월렛 버핏 페르소나 구현
- 시장 데이터 연동 (CoinGecko, CryptoPanic)
- 유저 프로필 자동 추론

### Phase 3: 포트폴리오 + 분석 (Week 5-6)

- 섀도우 포트폴리오 시스템
- AI 시그널 + [담아보기] 기능
- 뉴스 + Fear & Greed 통합

### Phase 4: 능동적 분석 + 마무리 (Week 7-8)

- Celery Beat 15분 자동 시장 분석
- PWA Push + Discord 알림
- 딥링크 시스템 (/chat/new?context=xxx)
- QA + ECS Fargate 배포 (Django + Celery worker)
- 베타 런칭

---

## 2026년 분기별 목표

| 분기 | 핵심 목표 | 주요 지표 |
|------|-----------|-----------|
| **Q1** | MVP 검증 | WhyBitcoinFallen 일 방문자 10K+<br>Sage.ai MAU 5K+ |
| **Q2** | 글로벌 확장 | 멀티 채팅 + 4개 언어<br>MAU 20K+ |
| **Q3** | 지표 개선 | 유료 구독 런칭<br>MAU 40K+ |
| **Q4** | ICO 본격화 | 생태계 확장<br>MAU 100K+ |

### 2026년 말 예상 성과

- WhyBitcoinFallen.com 일 방문자: 100,000+
- Sage.ai MAU: 10,000+
- 유료 구독자: 500+ (전환율 5%)
- Discord DAU: 1,000+
- Shadow Portfolio ROI: 벤치마크 +3%p

---

## 기술적 우수성

### AI-Native 개발

- Claude Code + Cursor 활용으로 10배 빠른 개발
- 멀티 에이전트 오케스트레이션으로 환각 제로화
- Tool 강제 사용으로 수치 오류 원천 차단

### 성능 목표

- 채팅 응답 속도: 2초 이내 (스트리밍 시작)
- 능동적 분석: 15분 간격 자동 시장 분석
- 컨텍스트 로드: 20개 메시지 (0.5초 이내)

### 확장성 설계

- ECS Fargate: 1만 유저까지 안정적
- Redis 캐싱: API 비용 절감 (시장 데이터 5분, AI 코멘트 5분)
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

1. **[06-MVP-Spec-ko.md](06-MVP-Spec-ko.md)** 읽기
2. 현재 주차 계획 확인
3. 기술 스택 및 폴더 구조 숙지

### AI-Native Development

- Claude 3.5 Sonnet: 메인 대화, Manager/Persona/Risk Agent
- Claude 3.5 Haiku: 알림 요약, 바이럴 AI 코멘트
- Anthropic Python SDK: 직접 통합

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

| 지표 | 목표 (8주 후) |
|------|---------------|
| **베타 테스터** | 10-20명 |
| **WhyBitcoinFallen 방문자** | 일 1,000+ |
| **Sage.ai MAU** | 500+ |
| **환각률** | <1% |
| **Shadow Portfolio 추적** | 10+ 트레이드 |

---

_"자네, 시장이 공포에 질렸을 때가 바로 기회일세."_ - 월렛 버핏

**Built with**: Django 5.1+ + DRF, Vite 6 + React 19, Tailwind CSS 4, Claude 3.5 Sonnet & Haiku
**Architecture**: Hexagonal (Ports & Adapters), Complete Backend/Frontend Separation
**Target**: 8주 MVP 완성, 2026년 MAU 10K+
**Version**: 3.0 (2025년 12월 17일) - Django Migration
