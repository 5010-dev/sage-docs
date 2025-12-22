# Sage.ai MVP 스펙 (3개월)

```
문서 정보
├── 버전: 2.0
├── 최종 수정일: 2025-12-19
├── 목적: 3개월 내 PMF 검증을 위한 MVP 스펙
├── 대상 독자: 개발팀, 프로덕트 팀
└── 상태: Production Ready
```

---

## 1. MVP 개요

### 1.1 핵심 가치

```
Sage.ai MVP
├── 슬로건: "AI 투자 멘토 월렛 버핏과 함께하는 코인 투자"
│
└── 핵심 기능 3가지
    ├── 1. 월렛 버핏과의 대화
    │   ├── 실시간 시장 데이터 기반 AI 멘토 상담
    │   └── 대화를 통해 자연스럽게 유저 성향 파악
    │
    ├── 2. 섀도우 포트폴리오
    │   └── "버핏 따라가기" - AI 추천 시점에 담고 수익률 추적
    │
    └── 3. 능동적 분석
        └── 시장 급변 시 AI가 먼저 알려줌 (PWA Push + Discord)
```

### 1.2 MVP vs Full 스펙 비교

| 기능 | MVP (8주) | Full (Phase 2+) |
|------|-----------|-----------------|
| AI 페르소나 | 월렛 버핏 1명 | 3명 (월렛 버핏, 사토시 현자, 알파 헌터) |
| 채팅 구조 | 1:1 대화 | 그룹 채팅 (단톡방) + 1:1 DM |
| AI 대화 시작 | 사용자가 먼저 | AI들이 먼저 떠들기 시작 |
| 온보딩 | 없음 (대화로 파악) | 없음 |
| 채팅 컨텍스트 | 최근 20개 메시지만 | RAG + pgvector 장기 기억 |
| 캐싱 | Redis 단순 캐싱 | Semantic Caching |
| 분석 | 15분 Cron 전체 | 이벤트 기반 트리거 |
| 알림 | 순차 발송 + 딥링크 | SNS+SQS Fan-out |
| 보안 | Regex 필터 | + LLM Guardrail |
| 검색 | 없음 | Hybrid Search |
| 구조 | 단일 Next.js | Turborepo 모노레포 |

### 1.3 MVP 서비스 구성

| 서비스 | 도메인 | 설명 |
|--------|--------|------|
| WhyBitcoinFallen.com | whybitcoinfallen.com | 바이럴 마케팅용 사이트 |
| Sage.ai 랜딩 | sage-ai.com | 서비스 소개 페이지 |
| Sage.ai 앱 | app.sage-ai.com | 메인 AI 멘토링 플랫폼 |

### 1.4 지원 자산 (6종)

| 코인 | 심볼 | 선정 이유 |
|------|------|-----------|
| Bitcoin | BTC | 시장 대장, 필수 |
| Ethereum | ETH | #2, DeFi/NFT 기반 |
| Solana | SOL | 고성능 L1, 높은 관심 |
| BNB | BNB | 바이낸스 생태계 |
| Dogecoin | DOGE | 밈코인 대표, 고변동성 |
| XRP | XRP | 결제 특화, 기관 관심 |

---

## 2. 대화 기반 유저 파악

### 2.1 개념

```
MVP 방식 (채팅으로 파악)
│
├── 바로 채팅 진입
│   └── 온보딩 설문 없음
│
├── 대화 중 자연스럽게 파악
│   └── 사용자: "비트코인 처음 사보려는데 어떻게 해?"
│
├── AI 추론
│   └── experience_level = beginner
│
└── 맞춤 응답
    └── 월렛 버핏: "자네, 처음 시작하는구먼! 먼저 기초부터 알려주지..."
```

### 2.2 유저 프로필 자동 업데이트

```typescript
// 대화에서 추출할 정보
interface InferredProfile {
  experienceLevel: 'beginner' | 'intermediate' | 'advanced' | null;
  investmentStyle: 'conservative' | 'moderate' | 'aggressive' | null;
  riskTolerance: 'low' | 'medium' | 'high' | null;
  interestedAssets: string[];  // 언급된 코인들
  mentionedTopics: string[];   // 관심 주제들
}

// Claude 응답에 프로필 추론 포함
{
  "message": "자네, 처음 시작하는구먼...",
  "inferredProfile": {
    "experienceLevel": "beginner",
    "confidence": 0.85
  },
  "signal": null
}
```

### 2.3 추론 기준

| 필드 | 추론 신호 |
|------|-----------|
| **experienceLevel** | |
| beginner | "처음", "초보", "어떻게 시작", "뭐부터" |
| intermediate | "조금 해봤는데", "작년에 샀다가", "DCA는 알아" |
| advanced | "레버리지", "온체인 분석", "청산 가격" |
| **investmentStyle** | |
| conservative | "안전하게", "원금 보존", "적금 대신" |
| moderate | "적당히", "분산 투자" |
| aggressive | "10배", "올인", "단타" |
| **interestedAssets** | 언급된 코인명 수집 |

### 2.4 시스템 프롬프트 반영

```typescript
// 초기 (정보 없음)
const systemPrompt = `
당신은 월렛 버핏입니다.
아직 이 사용자에 대해 아는 것이 없습니다.
대화를 통해 자연스럽게 파악하세요.
`;

// 몇 번 대화 후 (정보 축적)
const systemPrompt = `
당신은 월렛 버핏입니다.

[사용자 정보 - 대화에서 파악됨]
- 투자 경험: 초보 (확신도 85%)
- 관심 코인: BTC, ETH
- 특징: 안전한 투자 선호, 장기 관점

이 정보를 바탕으로 맞춤 조언을 제공하세요.
`;
```

---

## 3. 컨텍스트 관리 (단순화)

### 3.1 기본 원칙

```
MVP 컨텍스트 전략: 단순하게
│
├── 복잡한 방식 (제거)
│   ├── 채팅 내 압축
│   ├── 세션 종료 시 요약
│   ├── 유저 레벨 기억 축적
│   ├── Claude Haiku 요약 호출
│   ├── 비용: 세션당 $0.001
│   └── 복잡도: 높음
│
└── MVP 방식 (채택)
    ├── 최근 20개 메시지만 유지
    ├── 그 이전은 버림
    ├── 추론된 프로필만 저장
    ├── 추가 AI 호출 없음
    ├── 비용: $0
    └── 복잡도: 낮음
```

### 3.2 컨텍스트 윈도우 구조

```
API 호출 시 컨텍스트
│
├── [시스템 프롬프트]
│   ├── 월렛 버핏 페르소나
│   ├── 유저 프로필 (추론된 정보)
│   └── 현재 시장 데이터
│
├── [최근 20개 메시지]  <-- 원본 그대로
│   ├── user: "비트코인 어때?"
│   ├── assistant: "자네, 지금..."
│   ├── user: "더 살까?"
│   ├── assistant: "분할 매수를..."
│   └── ... (최대 20개)
│
└── [현재 질문]
```

### 3.3 구현

```typescript
// lib/ai/context.ts

const MAX_MESSAGES = 20;

async function buildChatContext(chatId: string): Promise<Message[]> {
  // 최근 20개 메시지만 조회 (단순!)
  const messages = await db.query.messages.findMany({
    where: eq(messages.chatId, chatId),
    orderBy: desc(messages.createdAt),
    limit: MAX_MESSAGES,
  });

  // 시간순 정렬 (오래된 것 먼저)
  return messages.reverse();
}

// API 호출
async function chat(chatId: string, userMessage: string) {
  const recentMessages = await buildChatContext(chatId);
  const userProfile = await getUserProfile(userId);
  const marketData = await getMarketData();

  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    system: buildSystemPrompt(userProfile, marketData),
    messages: [
      ...recentMessages,
      { role: 'user', content: userMessage }
    ],
  });

  return response;
}
```

### 3.4 유저 프로필 (유지)

대화에서 추론된 정보는 여전히 저장합니다. 단, 세션 요약은 제거.

```typescript
// db/schema/userProfiles.ts

export const userProfiles = pgTable('user_profiles', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull().unique(),

  // 대화에서 추론된 정보 (유지)
  experienceLevel: varchar('experience_level', { length: 20 }),
  investmentStyle: varchar('investment_style', { length: 20 }),
  riskTolerance: varchar('risk_tolerance', { length: 20 }),
  interestedAssets: jsonb('interested_assets').default([]),
  confidenceScores: jsonb('confidence_scores').default({}),

  // conversationHistory 제거됨 (세션 요약 없음)

  updatedAt: timestamp('updated_at').defaultNow(),
});
```

### 3.5 20개 제한의 의미

```
일반적인 대화 패턴
├── 하루 평균 대화: 5-10회 왕복 (10-20개 메시지)
└── 대부분의 경우 하루 대화는 컨텍스트 내에 유지됨

20개 초과 시:
├── 오래된 메시지부터 컨텍스트에서 제외
├── DB에는 계속 저장 (히스토리 조회용)
└── AI는 최근 대화만 기억

Phase 2에서 추가 가능:
├── RAG로 과거 대화 검색
└── "3달 전에 뭐라 했지?" 질문 대응
```

### 3.6 새 채팅 시작 시

```
새 채팅의 컨텍스트
│
├── [시스템 프롬프트]
│   ├── 월렛 버핏 페르소나
│   ├── 유저 프로필: "초보자, 보수적, BTC/ETH 관심"  <-- 이건 유지!
│   └── 현재 시장 데이터
│
├── [메시지 없음]  <-- 새 채팅이므로
│
└── [현재 질문]
    └── "안녕하세요"

--> AI는 유저가 "초보자"라는 건 알지만,
    이전 채팅에서 "분할 매수 논의했다"는 건 모름

--> 이 정도면 MVP로 충분!
```

---

## 4. 단순화된 기술 스택

### 4.1 기술 스택 Overview

```
MVP 기술 스택
│
├── Main App (Sage.ai)
│   ├── Next.js 15 (App Router)
│   ├── React 19
│   ├── Tailwind CSS 4
│   ├── shadcn/ui
│   └── Zustand (상태관리)
│
├── Sites (랜딩 + 바이럴)
│   ├── Vite + React + TypeScript
│   ├── Tailwind CSS
│   ├── TanStack Query
│   ├── Framer Motion
│   └── Recharts
│
├── Backend
│   ├── Next.js API Routes
│   ├── Drizzle ORM
│   └── Auth.js v5
│
├── AI
│   ├── Claude 3.5 Sonnet (메인 대화)
│   ├── Claude 3.5 Haiku (알림 요약, 바이럴 AI 코멘트)
│   └── Vercel AI SDK (스트리밍)
│
├── Database
│   ├── PostgreSQL (RDS)
│   └── Redis (ElastiCache)
│
├── Infrastructure
│   ├── ECS Fargate (메인 앱)
│   ├── S3 + CloudFront (Sites)
│   ├── Lambda (분석 워커)
│   └── EventBridge (15분 Cron)
│
└── External APIs
    ├── CoinGecko (가격)
    ├── CryptoPanic (뉴스)
    └── Alternative.me (Fear and Greed)
```

### 4.2 기술 선정 상세

| 카테고리 | 기술 | 버전 | MVP 선정 이유 |
|----------|------|------|---------------|
| Runtime | Node.js | 22 LTS | 최신 안정 버전 |
| Language | TypeScript | 5.7+ | 타입 안전성 |
| Framework | Next.js | 15.5+ | App Router, 스트리밍 지원 |
| Sites | Vite | 6.x | 빠른 빌드, React SPA |
| UI | React | 19 | Server Components |
| Styling | Tailwind CSS | 4.0 | 빠른 개발 |
| Components | shadcn/ui | latest | 복사해서 쓰는 컴포넌트 |
| State | Zustand | 5.x | 단순함, 가벼움 |
| Data Fetching | TanStack Query | 5.x | 캐싱, 자동 리패치 |
| Animation | Framer Motion | 11.x | 선언적 애니메이션 |
| Charts | Recharts | 2.x | React 친화적 차트 |
| ORM | Drizzle | 0.38+ | 타입 안전, SQL 친화 |
| Auth | Auth.js | 5.x | Google OAuth 쉬움 |
| AI SDK | Vercel AI SDK | 4.x | 스트리밍 추상화 |

---

## 5. 단순화된 아키텍처

### 5.1 전체 아키텍처

```
MVP Architecture
│
├── [CloudFront CDN]
│   │
│   ├── [WhyBitcoinFallen]     [Landing]          [App ALB]
│   │   (Vite+React/S3)        (Vite+React/S3)        │
│   │                                                  │
│   │                                           [ECS Fargate]
│   │                                            (Next.js)
│   │                                                 │
│   └──────────────────────────────────────────────────┤
│                                                      │
│   ┌──────────────┬──────────────┬──────────────┬─────┴────┐
│   │              │              │              │          │
│   v              v              v              v          v
│ [PostgreSQL]  [Redis]     [Claude API]   [CoinGecko]   [CryptoPanic]
│   (RDS)     (ElastiCache)                              [Fear&Greed]
│
└── Background Processing
    ├── [EventBridge]  -->  [Lambda]  -->  [Discord Webhook]
    │   (15분 Cron)        (분석기)        [PWA Push]
    └──────────────────────────────────────────────────────
```

### 5.2 채팅 플로우 (단순화)

```
채팅 플로우 (MVP)
│
├── 사용자: "비트코인 지금 어때?"
│
├── /api/chat (POST)
│   │
│   ├── 1. 기존 프로필 조회 (있으면)
│   │   └── experience_level, interested_assets 등
│   │
│   ├── 2. 컨텍스트 로드
│   │   └── 최근 20개 메시지 조회
│   │
│   ├── 3. 시장 데이터 조회 (Redis 캐시 또는 API)
│   │   ├── 가격: BTC $67,500 (+2.3%)
│   │   ├── Fear & Greed: 58 (탐욕)
│   │   └── 뉴스: "SEC ETF 관련..."
│   │
│   ├── 4. 시스템 프롬프트 조립
│   │   ├── 월렛 버핏 페르소나
│   │   ├── 유저 컨텍스트 (파악된 정보)
│   │   └── 시장 데이터
│   │
│   └── 5. Claude API 호출 (스트리밍)
│
├── SSE 스트리밍
│
└── 응답 처리
    ├── message: "자네, BTC가 $67,500인데..."
    ├── inferredProfile: { experienceLevel: "beginner", ... }
    ├── signal: { action: "buy", symbol: "BTC", ... }
    │
    ├── 6. 프로필 업데이트 (추론된 정보 있으면)
    └── 7. 메시지 저장
```

### 5.3 알림 플로우 (단순화)

```
알림 플로우 (MVP)
│
├── EventBridge (15분마다)
│
├── Lambda: 시장 분석
│   ├── 1. 현재가 조회 (6종)
│   ├── 2. 이전가와 비교 (Redis)
│   ├── 3. 변동 체크
│   │   ├── BTC: +/-5% -> 트리거
│   │   ├── ETH: +/-7% -> 트리거
│   │   └── 알트: +/-10% -> 트리거
│   └── 4. Fear & Greed 급변 체크 (+/-15)
│
├── 급변 감지 시
│
└── 알림 발송
    ├── 1. Discord Webhook (1회)
    │   └── #market-alerts 채널
    │
    ├── 2. PWA Push (개인별, 순차)
    │   ├── 알림 ON 유저만
    │   └── 딥링크 URL 포함
    │
    └── 3. DB 저장 (alert_history)
        └── 알림 타입, 관련 심볼, 딥링크 정보
```

### 5.4 딥링크 시스템

```
딥링크 구조
│
├── 알림 타입별 딥링크
│   │
│   ├── 1. 시장 급변 알림
│   │   └── /chat/new?context=market_alert&symbol=BTC
│   │       --> 새 채팅 + "BTC 5% 하락했는데 어떻게 생각해?" 자동 컨텍스트 주입
│   │
│   ├── 2. 포트폴리오 관련 알림 (담아둔 코인 급등락)
│   │   └── /chat/new?context=portfolio_alert&symbol=ETH
│   │       --> 새 채팅 + "내가 담아둔 ETH가 움직였네" 자동 컨텍스트 주입
│   │
│   └── 3. AI 인사이트 알림 (일일 브리핑 등)
│       └── /chat/new?context=daily_briefing
│           --> 새 채팅 + 오늘의 시장 요약 자동 시작
│
└── PWA Push 페이로드 구조
```

```json
{
  "title": "BTC -5.2% 급락",
  "body": "월렛 버핏이 분석을 준비했습니다",
  "icon": "/icons/alert-192.png",
  "badge": "/icons/badge-72.png",
  "data": {
    "type": "market_alert",
    "symbol": "BTC",
    "changePercent": -5.2,
    "deepLink": "/chat/new?context=market_alert&symbol=BTC",
    "alertId": "uuid-xxx"
  },
  "actions": [
    { "action": "open", "title": "분석 보기" },
    { "action": "dismiss", "title": "나중에" }
  ]
}
```

```typescript
// service-worker.ts (Push 이벤트 핸들러)

self.addEventListener('push', (event) => {
  const data = event.data?.json();

  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: data.icon,
      badge: data.badge,
      data: data.data,
      actions: data.actions,
    })
  );
});

// 알림 클릭 시 딥링크로 이동
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  const deepLink = event.notification.data?.deepLink;

  if (deepLink) {
    event.waitUntil(
      clients.matchAll({ type: 'window' }).then((windowClients) => {
        // 이미 열린 창이 있으면 포커스 + 이동
        for (const client of windowClients) {
          if ('focus' in client && 'navigate' in client) {
            client.focus();
            return client.navigate(deepLink);
          }
        }
        // 없으면 새 창 열기
        return clients.openWindow(deepLink);
      })
    );
  }
});
```

```typescript
// app/chat/new/page.tsx (딥링크 수신 처리)

export default function NewChatPage() {
  const searchParams = useSearchParams();
  const context = searchParams.get('context');
  const symbol = searchParams.get('symbol');

  // 컨텍스트별 자동 메시지 생성
  const getInitialMessage = () => {
    switch (context) {
      case 'market_alert':
        return `${symbol}가 급변했어요. 지금 상황 분석해주세요.`;
      case 'portfolio_alert':
        return `내가 담아둔 ${symbol}가 움직였는데, 어떻게 해야 할까요?`;
      case 'daily_briefing':
        return `오늘 시장 상황 브리핑 해주세요.`;
      default:
        return null;
    }
  };

  useEffect(() => {
    const initialMessage = getInitialMessage();
    if (initialMessage) {
      // 채팅 생성 후 자동으로 첫 메시지 전송
      createChatAndSendMessage(initialMessage);
    }
  }, [context, symbol]);

  // ...
}
```

---

## 6. 데이터베이스 스키마 (MVP)

### 6.1 ERD

```
MVP Database Schema
│
├── users
│   ├── id (PK)
│   ├── email
│   ├── name
│   ├── image
│   ├── tier
│   ├── push_subscription
│   ├── notifications_on
│   └── created_at
│       │
│       ├──────────────────┐
│       │                  │
│       v                  v
├── user_profiles      chats
│   ├── id (PK)        ├── id (PK)
│   ├── user_id (FK)   ├── user_id (FK)
│   ├── experience_    ├── title
│   │   level          ├── created_at
│   ├── investment_    └── updated_at
│   │   style              │
│   ├── risk_              │
│   │   tolerance          v
│   ├── interested_    messages
│   │   assets         ├── id (PK)
│   ├── confidence_    ├── chat_id (FK)
│   │   scores         ├── role
│   └── updated_at     ├── content
│                      ├── signal (jsonb)
│                      ├── inferred_profile
│                      └── created_at
│
├── shadow_trades      alert_history
│   ├── id (PK)        ├── id (PK)
│   ├── user_id (FK)   ├── user_id (FK)
│   ├── chat_id (FK)   ├── title
│   ├── message_id     ├── body
│   │   (FK)           ├── alert_type
│   ├── symbol         ├── symbol
│   ├── action         ├── change_percent
│   ├── price          ├── deep_link
│   ├── confidence     ├── is_read
│   ├── ai_reason      ├── clicked_at
│   └── created_at     └── created_at
```

### 6.2 스키마 정의

```typescript
// db/schema/users.ts

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }),
  image: text('image'),
  tier: varchar('tier', { length: 20 }).default('free'),
  pushSubscription: text('push_subscription'),
  notificationsEnabled: boolean('notifications_enabled').default(false),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

// db/schema/userProfiles.ts
// 대화에서 추론된 정보 저장

export const userProfiles = pgTable('user_profiles', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull().unique(),

  // 추론된 프로필
  experienceLevel: varchar('experience_level', { length: 20 }),
  investmentStyle: varchar('investment_style', { length: 20 }),
  riskTolerance: varchar('risk_tolerance', { length: 20 }),
  interestedAssets: jsonb('interested_assets').default([]),
  confidenceScores: jsonb('confidence_scores').default({}),

  updatedAt: timestamp('updated_at').defaultNow(),
});

// db/schema/chats.ts

export const chats = pgTable('chats', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  title: varchar('title', { length: 255 }).default('새 대화'),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});

// db/schema/messages.ts

export const messages = pgTable('messages', {
  id: uuid('id').primaryKey().defaultRandom(),
  chatId: uuid('chat_id').references(() => chats.id, { onDelete: 'cascade' }).notNull(),
  role: varchar('role', { length: 20 }).notNull(),
  content: text('content').notNull(),
  signal: jsonb('signal'),
  inferredProfile: jsonb('inferred_profile'),
  createdAt: timestamp('created_at').defaultNow(),
});

// db/schema/shadowTrades.ts

export const shadowTrades = pgTable('shadow_trades', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  chatId: uuid('chat_id').references(() => chats.id),
  messageId: uuid('message_id').references(() => messages.id),
  symbol: varchar('symbol', { length: 10 }).notNull(),
  action: varchar('action', { length: 10 }).notNull(),
  price: decimal('price', { precision: 20, scale: 8 }).notNull(),
  confidence: decimal('confidence', { precision: 3, scale: 2 }),
  aiReason: text('ai_reason'),
  createdAt: timestamp('created_at').defaultNow(),
});

// db/schema/alertHistory.ts

export const alertHistory = pgTable('alert_history', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id),

  // 알림 내용
  title: varchar('title', { length: 255 }).notNull(),
  body: text('body').notNull(),

  // 알림 타입 및 관련 정보
  alertType: varchar('alert_type', { length: 30 }).notNull(),
  symbol: varchar('symbol', { length: 10 }),
  changePercent: decimal('change_percent', { precision: 5, scale: 2 }),

  // 딥링크
  deepLink: varchar('deep_link', { length: 500 }),

  // 상태
  isRead: boolean('is_read').default(false),
  clickedAt: timestamp('clicked_at'),

  createdAt: timestamp('created_at').defaultNow(),
});
```

---

## 7. API 명세 (MVP)

### 7.1 API 목록

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/auth/[...nextauth]` | Auth.js 인증 |
| POST | `/api/chat` | 채팅 메시지 전송 (스트리밍) |
| GET | `/api/chats` | 채팅 목록 조회 |
| POST | `/api/chats` | 새 채팅 생성 |
| GET | `/api/chats/[chatId]/messages` | 채팅 메시지 조회 |
| GET | `/api/market/price` | 가격 조회 |
| GET | `/api/market/news` | 뉴스 조회 |
| GET | `/api/market/fear-greed` | Fear and Greed 조회 |
| POST | `/api/shadow-trades` | 섀도우 트레이드 추가 |
| GET | `/api/shadow-trades` | 섀도우 트레이드 목록 |
| POST | `/api/push/subscribe` | 푸시 구독 등록 |
| DELETE | `/api/push/subscribe` | 푸시 구독 해제 |
| GET | `/api/notifications` | 알림 히스토리 조회 |
| PATCH | `/api/notifications/[id]/read` | 알림 읽음 처리 |
| PATCH | `/api/notifications/[id]/click` | 알림 클릭 추적 (딥링크) |
| GET | `/api/settings` | 설정 조회 |
| PATCH | `/api/settings` | 설정 업데이트 |
| GET | `/api/profile` | 추론된 프로필 조회 |
| GET | `/api/viral/comment` | 바이럴 사이트용 AI 한마디 |
| GET | `/api/viral/stats` | 바이럴 사이트용 실시간 통계 |

### 7.2 주요 API 상세

#### 7.2.1 POST /api/chat

```typescript
// Request
{
  "chatId": "uuid",
  "message": "비트코인 지금 어때?"
}

// Response (SSE Stream)
data: {"type":"text","content":"자"}
data: {"type":"text","content":"네"}
...
data: {"type":"profile","inferredProfile":{"experienceLevel":"beginner","confidence":0.85}}
data: {"type":"signal","signal":{"action":"buy","symbol":"BTC","confidence":0.75}}
data: {"type":"done"}
```

#### 7.2.2 GET /api/profile

```typescript
// Response
{
  "profile": {
    "experienceLevel": "beginner",
    "investmentStyle": "conservative",
    "riskTolerance": "low",
    "interestedAssets": ["BTC", "ETH"],
    "confidenceScores": {
      "experienceLevel": 0.85,
      "investmentStyle": 0.60
    }
  }
}
```

---

## 8. 폴더 구조 (MVP)

```
sage-ai/
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy.yml
│       └── deploy-viral.yml
│
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   │
│   │   ├── (main)/
│   │   │   ├── chat/
│   │   │   │   ├── [chatId]/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx      # 딥링크 처리
│   │   │   │   └── page.tsx
│   │   │   │
│   │   │   ├── portfolio/
│   │   │   │   └── page.tsx
│   │   │   │
│   │   │   ├── notifications/
│   │   │   │   └── page.tsx
│   │   │   │
│   │   │   ├── settings/
│   │   │   │   ├── page.tsx
│   │   │   │   └── notifications/
│   │   │   │       └── page.tsx      # 알림 설정
│   │   │   │
│   │   │   └── layout.tsx
│   │   │
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   ├── chat/
│   │   │   ├── chats/
│   │   │   ├── market/
│   │   │   ├── shadow-trades/
│   │   │   ├── push/
│   │   │   ├── notifications/
│   │   │   ├── profile/
│   │   │   └── viral/                # 바이럴 사이트용 API
│   │   │       ├── comment/
│   │   │       │   └── route.ts
│   │   │       └── stats/
│   │   │           └── route.ts
│   │   │
│   │   ├── layout.tsx
│   │   └── page.tsx
│   │
│   ├── components/
│   │   ├── ui/
│   │   ├── chat/
│   │   ├── portfolio/
│   │   └── layout/
│   │
│   ├── lib/
│   │   ├── ai/
│   │   │   ├── claude.ts
│   │   │   ├── context.ts
│   │   │   ├── profileInference.ts
│   │   │   └── prompts/
│   │   │       └── walletBuffett.ts
│   │   ├── api/
│   │   ├── db/
│   │   ├── cache/
│   │   ├── auth/
│   │   └── push/
│   │
│   ├── hooks/
│   ├── stores/
│   └── types/
│
├── public/
│   ├── manifest.json                 # PWA 매니페스트
│   ├── sw.js                         # Service Worker
│   └── icons/
│       ├── icon-192.png
│       └── icon-512.png
│
├── lambda/
│   └── market-analyzer/
│
├── sites/
│   ├── why-bitcoin-fallen/           # 바이럴 사이트 (Vite + React)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── api/
│   │   │   ├── App.tsx
│   │   │   └── main.tsx
│   │   ├── public/
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   ├── tailwind.config.js
│   │   └── package.json
│   │
│   └── landing/                      # 랜딩 페이지 (Vite + React)
│       ├── src/
│       │   ├── components/
│       │   ├── sections/
│       │   ├── App.tsx
│       │   └── main.tsx
│       ├── public/
│       ├── index.html
│       ├── vite.config.ts
│       ├── tailwind.config.js
│       └── package.json
│
├── drizzle/
├── .env.example
├── next.config.ts
└── package.json
```

---

## 9. 8주 개발 계획

### 9.1 전체 로드맵

```
8주 MVP 로드맵
│
│  Week 1   Week 2   Week 3   Week 4   Week 5   Week 6   Week 7   Week 8
│    │        │        │        │        │        │        │        │
│    v        v        v        v        v        v        v        v
│  +----+  +----+  +----+  +----+  +----+  +----+  +----+  +----+
│  |셋업|  |인증|  |채팅|  |채팅|  |섀도|  |분석|  |알림|  | QA |
│  |    |  |바이|  |기본|  |시장|  |우  |  |스킬|  |딥링|  |배포|
│  |    |  |럴  |  |    |  |연동|  |    |  |    |  |크  |  |    |
│  +----+  +----+  +----+  +----+  +----+  +----+  +----+  +----+
│
│  |-- PHASE 1 --|  |-- PHASE 2 --|  |-- PHASE 3 --|  |-- PHASE 4 --|
│     기반 구축       핵심 채팅       포트폴리오+       마무리
│                                      분석
```

### 9.2 주차별 상세 계획

---

#### PHASE 1: 기반 구축 (Week 1-2)

```
Week 1: 프로젝트 셋업 + 인프라
│
├── Day 1 (월): 프로젝트 초기화
│   ├── Next.js 15 프로젝트 생성
│   ├── TypeScript, ESLint, Prettier 설정
│   ├── Tailwind CSS 4 + shadcn/ui 설치
│   └── 폴더 구조 잡기
│
├── Day 2 (화): Git + CI 설정
│   ├── GitHub 레포 생성
│   ├── GitHub Actions CI (린트, 타입체크, 빌드)
│   └── PR 템플릿
│
├── Day 3 (수): AWS 인프라
│   ├── RDS PostgreSQL 생성 (db.t3.micro)
│   ├── ElastiCache Redis 생성 (cache.t3.micro)
│   └── 보안그룹 설정
│
├── Day 4 (목): ORM + 캐시 연결
│   ├── Drizzle ORM 설정
│   ├── ioredis 설정
│   └── 연결 테스트
│
├── Day 5 (금): DB 스키마 v1
│   ├── users, user_profiles 테이블
│   ├── chats, messages 테이블
│   └── 마이그레이션 실행
│
└── [산출물]: Next.js + DB + Redis 연결 완료


Week 2: 인증 + 바이럴/랜딩 사이트
│
├── Day 1 (월): Auth.js 설정
│   ├── next-auth v5 설치
│   ├── Google OAuth 설정
│   └── Drizzle 어댑터 연결
│
├── Day 2 (화): 로그인 UI + 미들웨어
│   ├── /login 페이지
│   ├── 인증 미들웨어
│   ├── 보호된 라우트 설정
│   └── 로그인 후 바로 채팅 진입 (온보딩 없음)
│
├── Day 3 (수): WhyBitcoinFallen.com (Vite + React)
│   ├── Vite + React + TypeScript 프로젝트 생성
│   ├── 실시간 BTC 가격 컴포넌트 (TanStack Query -> CoinGecko)
│   ├── Fear & Greed 게이지 컴포넌트 (-> Alternative.me)
│   ├── 간단 차트 (Recharts)
│   ├── AI 미리보기 컴포넌트 (UI만, API는 Week 3 후 연동)
│   ├── 사회적 증거 + CTA 섹션
│   ├── S3 + CloudFront 배포
│   └── 도메인 연결
│
├── Day 4 (목): Sage.ai 랜딩 + 바이럴 API 준비
│   ├── Vite + React + TypeScript 프로젝트 생성
│   ├── Hero 섹션 (Framer Motion 애니메이션)
│   ├── 기능 소개 3가지 카드
│   ├── 스크린샷/데모 섹션
│   ├── 가입 CTA
│   ├── /api/viral/comment, /api/viral/stats 라우트 생성
│   │   (AI 코멘트는 Week 3 Claude 연동 후 활성화)
│   └── S3 + CloudFront 배포
│
├── Day 5 (금): 대시보드 레이아웃
│   ├── 대시보드 레이아웃 컴포넌트
│   │   ├── 사이드바 (채팅 목록)
│   │   ├── 헤더 (유저 정보, 로그아웃)
│   │   └── 메인 영역
│   └── 반응형 (모바일: 햄버거 메뉴)
│
└── [산출물]:
    ├── 로그인 -> 바로 채팅
    ├── WhyBitcoinFallen.com 라이브
    └── Sage.ai 랜딩 라이브
```

---

#### PHASE 2: 핵심 채팅 (Week 3-4)

```
Week 3: AI 채팅 기본
│
├── Day 1 (월): Claude API 연동
│   ├── @anthropic-ai/sdk 설치
│   ├── /api/chat 라우트
│   └── 기본 메시지 송수신
│
├── Day 2 (화): 월렛 버핏 페르소나
│   ├── 시스템 프롬프트 v1
│   │   ├── 말투: "자네", "~일세"
│   │   └── 금지: 레버리지, 확정 예측
│   ├── 유저 프로필 추론 프롬프트
│   └── 페르소나 테스트
│
├── Day 3 (수): SSE 스트리밍
│   ├── Vercel AI SDK 연동
│   ├── streamText + toDataStreamResponse
│   └── 클라이언트 useChat
│
├── Day 4 (목): 채팅 UI
│   ├── MessageBubble 컴포넌트
│   ├── ChatInput 컴포넌트
│   └── 스크롤 자동 이동
│
├── Day 5 (금): 채팅 목록
│   ├── ChatList 사이드바
│   ├── 새 채팅 생성
│   └── 채팅 전환
│
└── [산출물]: 월렛 버핏과 스트리밍 대화


Week 4: 채팅 고도화 + 시장 데이터
│
├── Day 1 (월): 대화 히스토리
│   ├── messages 테이블 저장
│   ├── 이전 대화 로드 (최근 20개)
│   └── 채팅 제목 자동 생성
│
├── Day 2 (화): 시장 데이터 연동
│   ├── CoinGecko API (가격 6종)
│   ├── Redis 5분 캐싱
│   └── 프롬프트에 가격 주입
│
├── Day 3 (수): 유저 프로필 추론
│   ├── 응답에서 inferredProfile 추출
│   ├── user_profiles 업데이트
│   ├── confidence 점수 관리
│   └── 프롬프트에 파악된 정보 반영
│
├── Day 4 (목): 채팅 UX 개선
│   ├── 타이핑 인디케이터
│   ├── 메시지 재전송
│   └── 오프라인 처리
│
├── Day 5 (금): 에러 핸들링 + 모바일
│   ├── 타임아웃 처리
│   ├── 로딩 상태
│   └── 모바일 반응형
│
└── [산출물]:
    ├── 시장 데이터 기반 AI 대화
    └── 대화로 유저 성향 파악
```

---

#### PHASE 3: 섀도우 포트폴리오 + 분석 (Week 5-6)

```
Week 5: 섀도우 포트폴리오
│
├── Day 1 (월): AI 시그널 시스템
│   ├── Claude 응답에 signal 구조 추가
│   ├── buy/sell 판단 기준 프롬프트
│   └── confidence 점수
│
├── Day 2 (화): 시그널 UI
│   ├── SignalCard 컴포넌트
│   ├── [담아보기] 버튼
│   └── shadow_trades 테이블
│
├── Day 3 (수): 담아보기 API
│   ├── POST /api/shadow-trades
│   ├── 현재가 자동 조회
│   └── 성공 토스트
│
├── Day 4 (목): 포트폴리오 페이지
│   ├── /portfolio 페이지
│   ├── 수익률 계산 로직
│   └── 트레이드 목록
│
├── Day 5 (금): 성적표 UI
│   ├── 전체 평균 수익률
│   ├── 종목별 수익률
│   └── 채팅에서 성적표 조회
│
└── [산출물]: 추천 -> 담기 -> 수익률 추적


Week 6: 분석 스킬
│
├── Day 1 (월): 뉴스 API
│   ├── CryptoPanic 연동
│   ├── 코인별 뉴스 필터
│   └── Redis 10분 캐싱
│
├── Day 2 (화): Fear & Greed
│   ├── Alternative.me 연동
│   ├── 프롬프트 통합
│   └── Redis 30분 캐싱
│
├── Day 3 (수): "왜 떨어졌어?" 처리
│   ├── 하락 이유 분석
│   ├── 가격 + 뉴스 + Fear&Greed 종합
│   └── 프롬프트 튜닝
│
├── Day 4 (목): 데이터 최적화
│   ├── Promise.all 병렬 조회
│   ├── 부분 실패 처리
│   └── 타임아웃 설정
│
├── Day 5 (금): 품질 테스트
│   ├── 다양한 질문 테스트
│   ├── 응답 품질 평가
│   └── 프롬프트 미세 조정
│
└── [산출물]: 뉴스 + Fear&Greed 종합 분석
```

---

#### PHASE 4: 알림 + 마무리 (Week 7-8)

```
Week 7: 능동적 분석 + 딥링크 (PWA + Discord)
│
├── Day 1 (월): PWA 설정
│   ├── manifest.json
│   ├── Service Worker 기본
│   └── VAPID 키 생성
│
├── Day 2 (화): Web Push 구현
│   ├── 푸시 구독 플로우
│   ├── Service Worker 푸시 수신
│   ├── 알림 페이로드에 딥링크 포함
│   └── notificationclick 이벤트 처리
│
├── Day 3 (수): 딥링크 시스템
│   ├── /chat/new?context=xxx 파라미터 처리
│   ├── 컨텍스트별 자동 메시지 생성
│   │   ├── market_alert: "BTC 급변 분석해줘"
│   │   ├── portfolio_alert: "담아둔 코인 움직임"
│   │   └── daily_briefing: "오늘 브리핑"
│   └── 알림 -> 채팅 자동 시작 플로우
│
├── Day 4 (목): 시장 분석 Lambda + Discord
│   ├── EventBridge 15분 Cron
│   ├── 급변 감지 로직
│   ├── Discord Webhook (Embed)
│   └── PWA 발송 (딥링크 포함)
│
├── Day 5 (금): 알림 설정 + 히스토리
│   ├── /settings/notifications
│   ├── /notifications 앱내 알림함
│   ├── 읽음/안읽음 처리
│   └── 알림 클릭 시 딥링크 이동
│
└── [산출물]:
    ├── 급변 시 Discord + PWA 알림
    └── 알림 클릭 -> 관련 채팅 자동 시작


Week 8: QA + 배포
│
├── Day 1 (월): 전체 플로우 테스트
│   ├── 회원가입 -> 채팅 (온보딩 없이)
│   ├── 섀도우 포트폴리오 플로우
│   └── 알림 수신 테스트
│
├── Day 2 (화): 버그 수정 + 보안
│   ├── Critical 버그 수정
│   ├── 인증 우회 테스트
│   └── 기본 Rate Limit
│
├── Day 3 (수): 성능 + 프로덕션 준비
│   ├── 페이지 로딩 속도
│   ├── 프로덕션 환경변수
│   └── 도메인 설정
│
├── Day 4 (목): ECS 배포
│   ├── Dockerfile
│   ├── ECS Fargate 배포
│   └── CloudFront + 모니터링
│
├── Day 5 (금): 베타 런칭!
│   ├── 베타 테스터 모집 (10-20명)
│   ├── 피드백 수집 채널
│   └── 실시간 모니터링
│
└── [산출물]: 베타 버전 라이브!
```

---

## 10. 바이럴/랜딩 사이트 상세

### 10.1 WhyBitcoinFallen.com

```
WhyBitcoinFallen.com 구조
│
├── Header
│   ├── [Logo]
│   └── [Sage.ai 가입]
│
├── Hero Section
│   ├── "WHY IS BITCOIN FALLING?"
│   ├── $67,500 (-5.2%)
│   └── [가격 차트]
│
├── Fear & Greed Section
│   ├── Fear & Greed Index: 25 (Extreme Fear)
│   └── [게이지 바]
│
├── Headlines Section
│   ├── "TODAY'S HEADLINES"
│   ├── SEC, 비트코인 ETF 관련 부정적 코멘트
│   ├── 연준 금리 동결 시사
│   └── 고래 지갑에서 대량 이동 감지
│
├── AI Preview Section (핵심!)
│   ├── [월렛 버핏 아바타]
│   ├── "자네, 시장이 공포에 질렸을 때가 바로 기회일세.
│   │    Fear & Greed 25라면... 과거 이 구간에서
│   │    매수했던 사람들은 평균 +47% 수익을 봤지."
│   ├── [이어서 대화하기]
│   └── [전체 분석 보기]
│
├── Social Proof Section
│   ├── 지금 1,247명이 월렛 버핏과 대화 중
│   └── "분할 매수 조언 덕분에 -5% 손실을 +12%로 바꿨어요"
│       - 사용자 김** (3일 전)
│
└── CTA Section
    └── [무료로 AI 멘토 만나기 - 30초면 시작]


전환 유도 요소:
├── 1. AI 미리보기: 월렛 버핏의 실시간 한마디 (호기심 유발)
├── 2. 사회적 증거: "1,247명이 대화 중" (FOMO)
├── 3. 후기: 실제 사용자 경험담
├── 4. 낮은 진입장벽: "30초면 시작"
└── 5. 이어서 대화하기: 클릭 시 Sage.ai 채팅으로 바로 연결

기술 스택:
├── Vite + React + TypeScript
├── Tailwind CSS
├── TanStack Query (API 캐싱)
├── Recharts (차트)
└── S3 + CloudFront 배포

API 연동:
├── CoinGecko: 실시간 가격
├── Alternative.me: Fear & Greed
└── Sage.ai API: AI 한마디 (캐시된 응답, 5분 갱신)
```

#### 10.1.1 프로젝트 구조

```
sites/why-bitcoin-fallen/
│
├── src/
│   ├── components/
│   │   ├── Header.tsx
│   │   ├── PriceDisplay.tsx
│   │   ├── PriceChart.tsx
│   │   ├── FearGreedGauge.tsx
│   │   ├── HeadlinesList.tsx
│   │   ├── AIPreview.tsx
│   │   ├── SocialProof.tsx
│   │   ├── CTAButton.tsx
│   │   └── Footer.tsx
│   ├── hooks/
│   │   ├── useBTCPrice.ts
│   │   ├── useFearGreed.ts
│   │   ├── useAIComment.ts
│   │   └── useActiveUsers.ts
│   ├── api/
│   │   └── index.ts
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── public/
│   ├── favicon.ico
│   └── og-image.png
├── index.html
├── vite.config.ts
├── tailwind.config.js
└── package.json
```

#### 10.1.2 핵심 컴포넌트

```typescript
// src/components/PriceDisplay.tsx
interface PriceDisplayProps {
  price: number;
  change24h: number;
  isLoading: boolean;
}

export function PriceDisplay({ price, change24h, isLoading }: PriceDisplayProps) {
  const isNegative = change24h < 0;

  return (
    <div className="text-center py-12">
      <h1 className="text-4xl font-bold mb-4">WHY IS BITCOIN FALLING?</h1>

      {isLoading ? (
        <div className="animate-pulse h-16 bg-gray-700 rounded w-48 mx-auto" />
      ) : (
        <>
          <div className="text-6xl font-bold mb-2">
            ${price.toLocaleString()}
          </div>
          <div className={`text-2xl ${isNegative ? 'text-red-500' : 'text-green-500'}`}>
            {isNegative ? '' : '+'}{change24h.toFixed(2)}%
          </div>
        </>
      )}
    </div>
  );
}
```

```typescript
// src/components/AIPreview.tsx
interface AIPreviewProps {
  comment: string;
  isLoading: boolean;
}

export function AIPreview({ comment, isLoading }: AIPreviewProps) {
  return (
    <div className="bg-gradient-to-r from-amber-900/30 to-yellow-900/30
                    border border-amber-700/50 rounded-xl p-6">
      <div className="flex items-start gap-4">
        {/* 버핏 아바타 */}
        <div className="w-16 h-16 rounded-full bg-amber-700 flex-shrink-0
                        flex items-center justify-center text-2xl">
          WB
        </div>

        <div className="flex-1">
          <div className="text-amber-400 font-semibold mb-2">월렛 버핏</div>

          {isLoading ? (
            <div className="space-y-2">
              <div className="h-4 bg-gray-700 rounded animate-pulse" />
              <div className="h-4 bg-gray-700 rounded animate-pulse w-3/4" />
            </div>
          ) : (
            <p className="text-gray-200 leading-relaxed">
              "{comment}"
            </p>
          )}
        </div>
      </div>

      {/* CTA 버튼 */}
      <div className="flex gap-3 mt-6">
        <a
          href="https://app.sage-ai.com/chat/new?context=market_alert&symbol=BTC"
          className="flex-1 bg-amber-600 hover:bg-amber-500 text-white
                     py-3 px-4 rounded-lg text-center font-semibold
                     transition-colors"
        >
          이어서 대화하기
        </a>
        <a
          href="https://app.sage-ai.com"
          className="flex-1 border border-amber-600 text-amber-400
                     hover:bg-amber-600/20 py-3 px-4 rounded-lg text-center
                     transition-colors"
        >
          전체 분석 보기
        </a>
      </div>
    </div>
  );
}
```

#### 10.1.3 데이터 Hooks

```typescript
// src/hooks/useBTCPrice.ts
import { useQuery } from '@tanstack/react-query';

interface PriceData {
  price: number;
  change24h: number;
  high24h: number;
  low24h: number;
}

export function useBTCPrice() {
  return useQuery<PriceData>({
    queryKey: ['btc-price'],
    queryFn: async () => {
      const res = await fetch(
        'https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_change=true'
      );
      const data = await res.json();
      return {
        price: data.bitcoin.usd,
        change24h: data.bitcoin.usd_24h_change,
        high24h: data.bitcoin.usd_24h_high,
        low24h: data.bitcoin.usd_24h_low,
      };
    },
    refetchInterval: 30_000, // 30초마다 갱신
    staleTime: 10_000,
  });
}
```

```typescript
// src/hooks/useFearGreed.ts
import { useQuery } from '@tanstack/react-query';

interface FearGreedData {
  value: number;
  label: string;
  timestamp: string;
}

export function useFearGreed() {
  return useQuery<FearGreedData>({
    queryKey: ['fear-greed'],
    queryFn: async () => {
      const res = await fetch('https://api.alternative.me/fng/');
      const data = await res.json();
      const fng = data.data[0];
      return {
        value: parseInt(fng.value),
        label: fng.value_classification,
        timestamp: fng.timestamp,
      };
    },
    refetchInterval: 5 * 60_000, // 5분마다 갱신
    staleTime: 60_000,
  });
}
```

### 10.2 Sage.ai 랜딩

```
Sage.ai 랜딩 구조
│
├── Header
│   ├── [Logo]
│   ├── [로그인]
│   └── [무료로 시작하기]
│
├── Hero Section
│   ├── "AI 투자 멘토, 월렛 버핏을 만나보세요"
│   ├── "50년 경력의 노련한 투자 멘토가
│   │    당신의 코인 투자를 도와드립니다"
│   └── [무료로 시작하기]
│
├── Features Section
│   ├── [기능 1] 실시간 분석 - 시장 데이터 기반 조언
│   ├── [기능 2] 섀도우 포트폴리오 - 버핏 추천 따라 수익률 추적
│   └── [기능 3] 능동적 분석 - 급변 시 먼저 알려드려요
│
├── Demo Section
│   └── [스크린샷 or 데모 GIF]
│
├── Testimonials Section
│   └── "버핏의 조언대로 했더니 +15% 수익!"
│       - 베타 테스터 후기
│
└── CTA Section
    └── [지금 바로 시작하기]


기술 스택:
├── Vite + React + TypeScript
├── Tailwind CSS
├── Framer Motion (애니메이션)
└── S3 + CloudFront 배포
```

---

## 11. Phase 2 로드맵 (MVP 이후)

### 11.1 기술 고도화

| 기능 | 설명 | 우선순위 |
|------|------|----------|
| 세션 요약 기억 | 채팅 종료 시 요약 저장, 새 채팅에서 참조 | 높음 |
| RAG + pgvector | 장기 기억 ("3달 전 뭐라 했지?") | 높음 |
| Hybrid Search | 날짜/키워드 검색 정확도 향상 | 중간 |
| Semantic Caching | 유사 질문 캐싱으로 비용 30% 절감 | 중간 |
| 이벤트 기반 트리거 | 15분 Cron -> 가격 변동 시에만 분석 | 중간 |
| LLM Guardrail | Regex + Claude Haiku 2중 방어 | 낮음 |
| SNS+SQS Fan-out | 대규모 알림 발송 | 유저 1만명+ |
| Redis Circuit Breaker | 분산 환경 장애 전파 차단 | 트래픽 증가 시 |

### 11.2 기능 확장

| 기능 | 설명 | 우선순위 |
|------|------|----------|
| AI 멘토 단톡방 | 3명 AI 멘토 그룹채팅 + AI끼리 자발적 토론 | 높음 |
| 다중 페르소나 | 월렛 버핏, 사토시 현자, 알파 헌터 | 높음 |
| 온체인 분석 | 고래 이동, 거래소 흐름 | 높음 |
| 추가 코인 | 6종 -> 20종 | 중간 |
| 프리미엄 플랜 | 결제, 고급 기능 | 중간 |
| 모바일 앱 | React Native | 낮음 |
| 소셜 기능 | 다른 유저 성적표 공유 | 낮음 |

### 11.3 Phase 2 핵심: AI 멘토 단톡방

```
MVP (8주)                          Phase 2
│                                  │
├── [사용자] <--> [월렛 버핏]      ├── [사용자]
│                                  │       │
│   1:1 채팅                       │       +-- [월렛 버핏]
│   사용자가 먼저 말해야 함         │       │
│                                  │       +-- [사토시 현자]
│                                  │       │
│                                  │       +-- [알파 헌터]
│                                  │
│                                  └── 그룹 채팅 (단톡방)
│                                      ├── AI들이 먼저 떠들기 시작
│                                      ├── AI끼리 토론/의견 충돌
│                                      ├── 사용자는 듣다가 참여
│                                      ├── @멘션으로 특정 AI 질문
│                                      └── 1:1 DM도 가능
```

---

## 12. 주차별 체크리스트

| 주차 | 목표 | 완료 기준 | 체크 |
|------|------|-----------|------|
| W1 | 프로젝트 셋업 | Next.js + DB + Redis 연결 | [ ] |
| W2 | 인증 + 사이트 | 로그인 + WhyBitcoinFallen + 랜딩 | [ ] |
| W3 | 채팅 기본 | 월렛 버핏 스트리밍 대화 | [ ] |
| W4 | 채팅 고도화 | 시장 데이터 + 프로필 추론 | [ ] |
| W5 | 섀도우 포트폴리오 | 추천 -> 담기 -> 수익률 | [ ] |
| W6 | 분석 스킬 | 뉴스 + Fear&Greed 종합 | [ ] |
| W7 | 알림 | PWA Push + Discord | [ ] |
| W8 | QA + 배포 | 베타 라이브 | [ ] |

---

## Appendix: 환경 변수

```bash
# .env.example

# Database
DATABASE_URL="postgresql://..."

# Redis
REDIS_URL="redis://..."

# Auth
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="http://localhost:3000"
GOOGLE_CLIENT_ID="..."
GOOGLE_CLIENT_SECRET="..."

# AI
ANTHROPIC_API_KEY="..."

# External APIs
COINGECKO_API_KEY="..."
CRYPTOPANIC_API_KEY="..."

# Push Notifications
NEXT_PUBLIC_VAPID_PUBLIC_KEY="..."
VAPID_PRIVATE_KEY="..."

# Discord
DISCORD_WEBHOOK_URL="..."

# AWS (배포 시)
AWS_REGION="ap-northeast-2"
AWS_ACCESS_KEY_ID="..."
AWS_SECRET_ACCESS_KEY="..."

# S3 Buckets
S3_BUCKET_VIRAL="whybitcoinfallen.com"
S3_BUCKET_LANDING="sage-ai-landing"

# CloudFront
CF_DISTRIBUTION_APP="..."
CF_DISTRIBUTION_VIRAL="..."
CF_DISTRIBUTION_LANDING="..."

# URLs
NEXT_PUBLIC_APP_URL="https://app.sage-ai.com"
VIRAL_SITE_URL="https://whybitcoinfallen.com"
LANDING_SITE_URL="https://sage-ai.com"
```

---

```
문서 정보
├── 최종 수정일: 2025-12-19
├── 버전: 2.0
├── 상태: Production Ready
└── 관리자: Sam (dev@5010.tech)
```
