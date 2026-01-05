# Sage.ai Branding Guide

> **문서 버전**: 2.0
> **최종 수정**: 2025년 12월 22일
> **작성자**: Sam
> **대상 독자**: 마케팅팀, 디자이너, 콘텐츠 크리에이터

---

## 1. Brand Overview

### 1.1 Brand Identity

```typescript
interface BrandIdentity {
  name: "Sage.ai";
  meaning: {
    sage: "현자, 지혜로운 사람 (영어)";
    ai: "AI 기술 + 도메인 확장자";
  };
  tagline: "Between the zeros and ones";
  taglineMeaning: "디지털 세계(0과 1) 속에서 진짜 가치를 찾다";
}
```

### 1.2 Brand Positioning

#### Mission Statement

> "모든 개인 투자자에게 월가 수준의 AI 멘토를"

#### Vision

**2030년까지 전 세계 100만 투자자의 신뢰받는 AI 투자 파트너가 되다**

---

## 2. Core Values

### 2.1 Brand Values Framework

```typescript
interface BrandValues {
  trust: {
    hallucination: "환각 제로 시스템";
    verification: "섀도우 포트폴리오로 투명한 성과 검증";
    datadriven: "데이터 기반 의사결정";
  };
  wisdom: {
    insight: "단순 정보가 아닌 통찰 제공";
    philosophy: "워렌 버핏의 투자 철학";
    longterm: "장기적 관점의 조언";
  };
  care: {
    availability: "24시간 대응";
    proactive: "능동적 멘탈 케어";
    personalization: "개인화된 맞춤 서비스";
  };
}
```

---

## 3. Brand Persona: Wallet Buffett

### 3.1 Character Profile

```typescript
interface WalletBuffettPersona {
  name: "월렛 버핏 (Wallet Buffett)";
  age: "50대 중반";
  profession: "AI 투자 멘토";
  personality: {
    analytical: "냉철하지만 따뜻한";
    teaching: "훈계하지만 격려하는";
    experienced: "워렌 버핏 철학 + 암호화폐 이해";
  };
}
```

### 3.2 Communication Style

#### Tone Guidelines

| 특성 | 설명 |
|------|------|
| **존대와 하대 혼용** | "자네", "~일세", "~하게" |
| **비유와 은유** | 투자 철학을 쉬운 이야기로 전달 |
| **직설적이지만 따뜻함** | 잘못된 판단은 지적하되, 격려도 함께 |

#### Sample Messages

**긍정적 상황**
```
"자네, 좋은 판단이었네. 공포에 질렸을 때
용기를 낸 것이 결과로 이어졌군."
```

**부정적 상황**
```
"자네, 그건 투자가 아니라 도박일세.
정신 차리게. 시장은 항상 기회를 주지 않아."
```

**조언**
```
"지금은 탐욕 지수가 85를 넘었네.
모두가 흥분할 때 한 발 물러서는 게 현명하지 않겠나?"
```

### 3.3 Do's and Don'ts

```typescript
interface CommunicationRules {
  forbidden: [
    "지금 사세요 (직접적 매매 신호)",
    "내일 10% 오릅니다 (확정적 예측)",
    "레버리지 거래 추천",
    "와! 대박! (감정적 표현)"
  ];
  recommended: [
    "~할 수 있다 (조건부 표현)",
    "과거 데이터에 따르면... (근거 제시)",
    "현물 투자 위주로 (안전 우선)",
    "자네, 차분하게... (이성적 톤)"
  ];
}
```

---

## 4. Visual Identity

### 4.1 Color Palette

#### Primary Colors

```typescript
interface PrimaryColors {
  sageGold: {
    hex: "#D4AF37";
    usage: "로고, CTA, 강조";
  };
  deepNavy: {
    hex: "#1A237E";
    usage: "배경, 텍스트";
  };
  wisdomGray: {
    hex: "#37474F";
    usage: "보조 배경";
  };
}
```

#### Secondary Colors

```typescript
interface SecondaryColors {
  trustBlue: {
    hex: "#1976D2";
    usage: "링크, 정보";
  };
  successGreen: {
    hex: "#388E3C";
    usage: "수익, 긍정";
  };
  warningAmber: {
    hex: "#F57C00";
    usage: "경고, 주의";
  };
  dangerRed: {
    hex: "#D32F2F";
    usage: "손실, 위험";
  };
}
```

### 4.2 Typography

#### Font Stack

```css
/* Primary Font */
font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;

/* Headings */
h1 { font-weight: 700; font-size: 32px; }
h2 { font-weight: 700; font-size: 24px; }
h3 { font-weight: 700; font-size: 20px; }

/* Body Text */
body { font-weight: 400; font-size: 16px; line-height: 1.6; }

/* Code/Numbers */
.code { font-family: 'JetBrains Mono', monospace; }
```

#### Typography Specifications

| 용도 | Font | Weight | Size |
|------|------|--------|------|
| **Headings** | Inter | 700 | 32px (H1), 24px (H2), 20px (H3) |
| **Body** | Inter | 400 | 16px |
| **Code/Numbers** | JetBrains Mono | 400 | 14px |

### 4.3 Logo Guidelines

```typescript
interface LogoGuidelines {
  format: "Wordmark + Icon";
  icon: "WB (월렛 버핏 이니셜 심볼)";
  color: "#D4AF37";
  background: ["transparent", "#1A237E"];
  minimumSize: {
    web: "120px width";
    mobile: "80px width";
    favicon: "32x32px";
  };
  prohibited: [
    "로고 변형 (비율, 색상)",
    "그라데이션 적용",
    "그림자 효과",
    "다른 폰트 사용"
  ];
}
```

---

## 5. Tone & Manner

### 5.1 Brand Voice Principles

| 항목 | 설명 |
|------|------|
| **전문적이지만 친근한** | 금융 전문가의 신뢰감 + 편안한 대화 |
| **직설적이지만 따뜻한** | 잘못된 판단은 지적하되, 격려도 함께 |
| **데이터 기반이지만 감성적** | 숫자로 말하되, 감정도 이해 |

### 5.2 Communication Examples

| 상황 | 나쁜 예시 | 좋은 예시 |
|------|-------------|------------|
| **시장 하락** | "패닉하지 마세요!" | "자네, 시장이 공포에 질렸을 때가 바로 기회일세." |
| **수익 발생** | "대박났어요!" | "좋은 판단이었네. 하지만 교만하지 말게. 시장은 언제나 변화하니까." |
| **손실 발생** | "다 잃었네요..." | "손실은 배움의 기회일세. 무엇을 놓쳤는지 함께 돌아보세." |

### 5.3 Channel-Specific Guidelines

```typescript
interface ChannelGuidelines {
  chat: {
    style: "존대와 하대 혼용";
    format: "비유와 은유 사용";
    length: "3-5문장 내외 (간결함)";
  };
  marketing: {
    headline: "짧고 강렬한";
    content: "데이터와 스토리텔링 결합";
    cta: "명확하고 직관적";
  };
  email: {
    tone: "정중하고 공식적";
    structure: "핵심부터 먼저 (BLUF 원칙)";
    clarity: "액션 아이템 명확히";
  };
}
```

---

## 6. UI/UX Design Principles

### 6.1 Design Philosophy

> "복잡한 금융을 단순하게, 차가운 데이터를 따뜻하게"

### 6.2 Core Principles

```typescript
interface DesignPrinciples {
  hierarchy: "정보 위계 명확 - 중요한 정보부터 크고 진하게";
  whitespace: "여백 활용 - 복잡한 차트도 여백으로 숨 쉬게";
  consistency: "일관된 컴포넌트 - shadcn/ui 기반";
  mobilefirst: "모바일 우선 - 작은 화면에서도 가독성";
}
```

### 6.3 Icon System

```typescript
interface IconSystem {
  library: "Lucide Icons";
  style: "Line icons (2px stroke)";
  colors: ["Primary", "Grayscale"];
  custom: [
    "코인 심볼 (BTC, ETH 등)",
    "월렛 버핏 아바타",
    "섀도우 포트폴리오 아이콘"
  ];
}
```

### 6.4 Image Guidelines

| 요소 | 스타일 |
|------|--------|
| **사진 톤** | 따뜻한 느낌 (차가운 금융 이미지 지양) |
| **색감** | Sage Gold 계열 강조 |
| **구도** | 여백 많이, 미니멀 |
| **일러스트** | Flat design, 브랜드 팔레트 내 |

---

## 7. Messaging Framework

### 7.1 Core Messages

```typescript
interface MessagingHierarchy {
  main: "신뢰할 수 있는 AI 투자 멘토, 월렛 버핏";
  sub: {
    hallucination: "에이전트 파이프라인 교차 검증으로 환각 제로";
    performance: "섀도우 포트폴리오로 투명하게 공개";
    proactive: "AI가 먼저 알려드립니다";
  };
}
```

### 7.2 Target-Specific Messaging

#### Beginner Investors

```typescript
interface BeginnerMessaging {
  painPoint: "어디서부터 시작해야 할지 모르겠어요";
  message: "월렛 버핏이 처음부터 차근차근 알려드립니다";
  cta: "무료로 시작하기";
}
```

#### Intermediate Investors

```typescript
interface IntermediateMessaging {
  painPoint: "제 판단이 맞는지 확신이 없어요";
  message: "섀도우 포트폴리오로 AI 조언을 검증해보세요";
  cta: "성과 확인하기";
}
```

#### Advanced Investors

```typescript
interface AdvancedMessaging {
  painPoint: "24시간 시장을 모니터링할 수 없어요";
  message: "15분마다 자동 분석, 급변 시 즉시 알림";
  cta: "프리미엄 체험하기";
}
```

---

## 8. Content Strategy

### 8.1 Blog/Article Guidelines

```typescript
interface ContentStructure {
  topics: [
    "투자 교육 (워렌 버핏 철학)",
    "암호화폐 시장 분석",
    "AI 활용 투자 팁",
    "성공/실패 사례"
  ];
  structure: {
    hook: "흥미로운 질문 또는 통계";
    problem: "독자의 Pain Point";
    solution: "Sage.ai의 해결 방법";
    proof: "데이터 또는 사례";
    cta: "명확한 행동 유도";
  };
  tone: "교육적이지만 지루하지 않게, 스토리텔링 활용";
}
```

### 8.2 Social Media Strategy

#### Twitter/X

```typescript
interface TwitterStrategy {
  format: "짧고 강렬한 1-2줄 + 데이터";
  frequency: "하루 3-5회";
  example: "BTC Fear & Greed 25. 과거 이 구간에서 매수했던 사람들은 평균 +47% 수익. - 월렛 버핏";
}
```

#### Discord

```typescript
interface DiscordChannels {
  announcements: "공지사항";
  marketAlerts: "시장 급변 알림";
  generalChat: "자유 토론";
  portfolioShowcase: "성과 공유";
}
```

#### KakaoTalk

```typescript
interface KakaoStrategy {
  format: "오픈채팅방";
  target: "한국 커뮤니티 중심";
  tone: "친근하고 캐주얼";
}
```

---

## 9. Brand Consistency Checklist

### 9.1 Quality Assurance

```typescript
interface BrandChecklist {
  visual: [
    "브랜드 컬러 사용 (Sage Gold, Deep Navy)",
    "로고 정확히 사용",
    "타이포그래피 일관성 (Inter 폰트)"
  ];
  voice: [
    "월렛 버핏 톤 유지 (자네, ~일세)",
    "데이터 근거 제시",
    "직접적 매매 신호 금지"
  ];
  content: [
    "CTA 명확성",
    "메시지 일관성"
  ];
}
```

---

## 10. Brand Evolution Roadmap

### 10.1 Phase 1 (2026 Q1-Q2): Brand Establishment

```typescript
interface Phase1 {
  focus: [
    "월렛 버핏 캐릭터 강화",
    "일관된 비주얼 아이덴티티",
    "환각 제로 메시지 강조"
  ];
}
```

### 10.2 Phase 2 (2026 Q3-Q4): Brand Expansion

```typescript
interface Phase2 {
  focus: [
    "멀티 페르소나 추가 (사토시 현자, 알파 헌터)",
    "글로벌 메시징 (4개 언어)",
    "커뮤니티 브랜드 강화"
  ];
}
```

### 10.3 Phase 3 (2027+): Brand Evolution

```typescript
interface Phase3 {
  focus: [
    "암호화폐 → 주식 시장 확장",
    "AI 투자 멘토 카테고리 리더",
    "브랜드 파트너십 (거래소, 핀테크)"
  ];
}
```

---

## Appendix

### A. Brand Assets

```typescript
interface BrandAssets {
  required: [
    "로고 파일 (SVG, PNG)",
    "컬러 팔레트 (Figma/Sketch)",
    "폰트 라이선스",
    "아이콘 세트",
    "월렛 버핏 아바타"
  ];
}
```

### B. Reference Brands

```typescript
interface BrandBenchmarks {
  inspiration: {
    robinhood: "친근한 금융";
    duolingo: "페르소나 활용";
    perplexity: "깔끔한 AI UI";
  };
  differentiation: [
    "멘토 포지셔닝 (단순 도구 X)",
    "강력한 페르소나",
    "신뢰를 핵심 가치로"
  ];
}
```

---

**문서 끝**

_"Between the zeros and ones"_
