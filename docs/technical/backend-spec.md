# Sage.ai Backend Specification

> **AI-Powered Investment Mentor Platform - Backend Architecture**

---

**문서 버전**: 1.0
**최종 수정일**: 2025년 12월 17일
**작성자**: Sam
**검토자**: -

---

## 목차

1. [개요](#1-개요)
2. [기술 스택](#2-기술-스택)
3. [아키텍처](#3-아키텍처)
4. [AI 에이전트 시스템](#4-ai-에이전트-시스템)
5. [컨텍스트 관리](#5-컨텍스트-관리)
6. [데이터베이스 스키마](#6-데이터베이스-스키마)
7. [API 명세](#7-api-명세)
8. [외부 API 연동](#8-외부-api-연동)
9. [성능 목표](#9-성능-목표)
10. [보안 및 인증](#10-보안-및-인증)

---

## 1. 개요

### 1.1 프로젝트 개요

**Sage.ai**는 Claude 3.5 기반 멀티 에이전트 시스템을 활용한 AI 투자 멘토 플랫폼입니다.

**핵심 특징**:
- **월렛 버핏 페르소나**: 워렌 버핏의 투자 철학을 구현한 AI 멘토
- **환각 제로화**: Tool 강제 사용 + 멀티 에이전트 교차 검증
- **쉐도우 포트폴리오**: AI 조언의 수익률을 투명하게 추적
- **능동적 알림**: 15분 간격 시장 분석 + PWA Push/Discord 알림

### 1.2 아키텍처 철학

**단순성 > 확장성** (MVP 우선)

- **모노리스 구조**: 마이크로서비스 대신 Next.js API Routes
- **컨텍스트 단순화**: 최근 20개 메시지만 (RAG 없음)
- **프로그레시브 스케일링**: ECS Fargate → 10K 유저까지, 이후 EKS 고려

### 1.3 개발 목표

- **8주 MVP**: Week 1-2 (인프라), Week 3-4 (채팅), Week 5-6 (포트폴리오), Week 7-8 (알림)
- **타겟 유저**: 500 MAU (MVP), 5K MAU (Q1 2026), 10K MAU (Q4 2026)
- **성능**: 2초 이내 첫 토큰, <1% 환각률, 0.5초 컨텍스트 로드

---

## 2. 기술 스택

### 2.1 Overview

```
Backend Stack (Sage.ai)

Runtime & Language
- Node.js 22 LTS
- TypeScript 5.7+

Framework
- Next.js 15.5+ (App Router)
- React 19 (Server Components)

Backend Core
- Next.js API Routes (not Express)
- Drizzle ORM 0.38+
- Auth.js v5

AI & Streaming
- Claude 3.5 Sonnet (메인 대화, Manager/Persona/Risk Agent)
- Claude 3.5 Haiku (알림 요약, 바이럴 AI 코멘트)
- Vercel AI SDK 4.x (SSE 스트리밍 추상화)

Database
- PostgreSQL 16+ (RDS)
- Redis 7.x (ElastiCache)

Infrastructure
- ECS Fargate (컨테이너)
- EventBridge (15분 Cron)
- Lambda (분석 워커)
- S3 + CloudFront (정적 사이트)
```

### 2.2 기술 선정 상세

| 카테고리 | 기술 | 버전 | 선정 이유 |
|---------|------|------|----------|
| **Runtime** | Node.js | 22 LTS | 최신 안정 버전, Edge Runtime 지원 |
| **Language** | TypeScript | 5.7+ | 타입 안전성, Claude Code 친화적 |
| **Framework** | Next.js | 15.5+ | App Router, SSR, API Routes 통합 |
| **ORM** | Drizzle | 0.38+ | 타입 안전, SQL 친화, Zod 통합 |
| **Auth** | Auth.js | 5.x | Google OAuth 간편 통합 |
| **AI SDK** | Vercel AI SDK | 4.x | SSE 스트리밍 추상화, Claude 지원 |
| **Database** | PostgreSQL | 16+ | ACID, JSON 지원, RDS 관리형 |
| **Cache** | Redis | 7.x | 고성능 캐싱, ElastiCache 관리형 |

### 2.3 No 선정 (의도적 제외)

| 기술 | 제외 이유 |
|------|-----------|
| **Express/Fastify** | Next.js API Routes로 충분 |
| **Prisma** | Drizzle이 더 SQL 친화적 |
| **RAG (Vector DB)** | MVP는 20개 메시지만, Phase 2에 추가 |
| **Kafka** | 이벤트 드리븐 불필요, EventBridge로 충분 |
| **Kubernetes** | ECS Fargate로 10K 유저 처리 가능 |

---

## 3. 아키텍처

### 3.1 전체 아키텍처

```
MVP Architecture

                       [CloudFront CDN]
                              |
              +---------------+---------------+
              |               |               |
      [WhyBitcoinFallen]  [Landing]      [App ALB]
       (Vite+React/S3)   (Vite+React/S3)      |
                                       [ECS Fargate]
                                        (Next.js)
                                             |
            +----------------+---------------+----------------+
            |                |               |                |
      [PostgreSQL]       [Redis]        [Claude API]    [External APIs]
         (RDS)        (ElastiCache)                      - CoinGecko
                                                         - CryptoPanic
                                                         - Fear&Greed

  -------------------------------------------------------------------------

      [EventBridge]  -->  [Lambda]  -->  [Discord Webhook]
      (15분 Cron)        (분석기)        [PWA Push]
```

### 3.2 채팅 플로우

```
사용자: "비트코인 지금 어때?"
      |
      v
+----------------------------------------------------------+
|  POST /api/chat                                           |
|                                                           |
|  1. 기존 프로필 조회 (있으면)                             |
|     - experience_level, interested_assets 등             |
|                                                           |
|  2. 컨텍스트 로드                                         |
|     - 최근 20개 메시지 조회                              |
|                                                           |
|  3. 시장 데이터 조회 (Redis 캐시 또는 API)               |
|     - 가격: BTC $67,500 (+2.3%)                          |
|     - Fear & Greed: 58 (탐욕)                            |
|     - 뉴스: "SEC ETF 관련..."                            |
|                                                           |
|  4. 시스템 프롬프트 조립                                 |
|     - 월렛 버핏 페르소나                                 |
|     - 유저 컨텍스트 (파악된 정보)                        |
|     - 시장 데이터                                        |
|                                                           |
|  5. Claude API 호출 (스트리밍)                           |
+----------------------------------------------------------+
      |
      v SSE 스트리밍
+----------------------------------------------------------+
|  응답 처리                                                |
|                                                           |
|  - message: "자네, BTC가 $67,500인데..."                 |
|  - inferredProfile: { experienceLevel: "beginner", ... }  |
|  - signal: { action: "buy", symbol: "BTC", ... }         |
|                                                           |
|  6. 프로필 업데이트 (추론된 정보 있으면)                 |
|  7. 메시지 저장                                          |
+----------------------------------------------------------+
```

### 3.3 알림 플로우

```
EventBridge (15분마다)
      |
      v
+----------------------------------------------------------+
|  Lambda: 시장 분석                                        |
|                                                           |
|  1. 현재가 조회 (6종)                                    |
|  2. 이전가와 비교 (Redis)                                |
|  3. 변동 체크                                            |
|     - BTC: +/-5% -> 트리거                               |
|     - ETH: +/-7% -> 트리거                               |
|     - 알트: +/-10% -> 트리거                             |
|  4. Fear & Greed 급변 체크 (+/-15)                       |
+----------------------------------------------------------+
      |
      | 급변 감지!
      v
+----------------------------------------------------------+
|  알림 발송                                                |
|                                                           |
|  1. Discord Webhook (1회)                                 |
|     - #market-alerts 채널                                |
|                                                           |
|  2. PWA Push (개인별, 순차)                              |
|     - 알림 ON 유저만                                     |
|     - 딥링크 URL 포함                                    |
|                                                           |
|  3. DB 저장 (alert_history)                              |
|     - 알림 타입, 관련 심볼, 딥링크 정보                  |
+----------------------------------------------------------+
```

---

## 4. AI 에이전트 시스템

### 4.1 멀티 에이전트 오케스트레이션

```
[사용자 질문]
    ↓
[Manager Agent] - 의도 파악 및 라우팅 (Sonnet 3.5)
    ↓
[Analyst Agent] - 뉴스/가격 API로 Fact 수집 (Haiku 3.5)
    ↓
[Persona Agent] - 월렛 버핏이 철학 기반 해석 (Sonnet 3.5)
    ↓
[Risk Agent] - 오류/편향 검증 (Sonnet 3.5)
    ↓
[최종 응답]
```

### 4.2 에이전트 구성

| 에이전트 | 모델 | 핵심 기술 | 역할 |
|---------|------|----------|------|
| **Manager** | Sonnet 3.5 | Router, State Manager | 대화 흐름 조율, 적절한 에이전트 연결 |
| **Analyst** | Haiku 3.5 | Search API, Price Fetcher, Calculator | 감정 배제, 시장 현상(Fact) 수집 |
| **Persona** | Sonnet 3.5 | Style Transfer, Reasoning | **월렛 버핏**: 통찰과 가르침 전달 |
| **Risk** | Sonnet 3.5 | Compliance Check, Fact Verification | 사실 오류/법적 위험 최종 확인 |

### 4.3 환각 제로화 전략

1. **Tool 강제 사용**
   ```typescript
   // 수치 계산은 LLM이 아닌 Calculator 도구 사용 의무화
   tools: [
     {
       name: "calculator",
       description: "Perform mathematical calculations",
       required: true, // 가격 비교, 수익률 계산 시 필수
     }
   ]
   ```

2. **Analyst 분리**
   - 월렛 버핏은 '해석'만
   - '팩트 체크'는 별도 Analyst 에이전트

3. **Risk Agent 교차 검증**
   - 최종 응답 전 오류 필터링
   - 법적 위험 문구 감지 ("지금 사라" → "~를 고려해볼 만하다"로 변경)

### 4.4 구현 예시

```typescript
// lib/ai/agents.ts

import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

// Manager Agent
export async function managerAgent(userMessage: string) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    system: `당신은 Manager Agent입니다. 사용자 의도를 파악하고 적절한 에이전트로 라우팅하세요.`,
    messages: [{ role: 'user', content: userMessage }],
  });

  return {
    intent: response.content[0].text,
    nextAgent: 'analyst', // or 'persona', 'risk'
  };
}

// Analyst Agent (Fact만 수집)
export async function analystAgent(query: string) {
  const price = await getPriceFromCoinGecko('BTC');
  const news = await getNewsFromCryptoPanic('BTC');
  const fearGreed = await getFearGreedIndex();

  return {
    facts: {
      price,
      news,
      fearGreed,
    },
    timestamp: new Date().toISOString(),
  };
}

// Persona Agent (월렛 버핏 해석)
export async function personaAgent(facts: any, userProfile: any) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 2048,
    system: `당신은 월렛 버핏입니다. 워렌 버핏의 투자 철학을 구현하세요.

    말투: "자네", "~일세", "~하게"
    철학: 안전마진, 해자, 장기투자, 공포에 사라 탐욕에 팔아라

    [팩트 데이터]
    ${JSON.stringify(facts)}

    [사용자 프로필]
    ${JSON.stringify(userProfile)}
    `,
    messages: [{ role: 'user', content: '지금 시장에 대해 조언해주세요.' }],
  });

  return response.content[0].text;
}

// Risk Agent (최종 검증)
export async function riskAgent(message: string) {
  const dangerousPatterns = [
    /지금 사(라|세요)/,
    /무조건/,
    /100% 확실/,
  ];

  for (const pattern of dangerousPatterns) {
    if (pattern.test(message)) {
      // LLM으로 안전한 표현으로 변경
      const safeMessage = await rewriteToSafeExpression(message);
      return safeMessage;
    }
  }

  return message;
}
```

---

## 5. 컨텍스트 관리

### 5.1 단순화 전략 (MVP)

**복잡한 방식 제거**:
- ❌ 채팅 내 압축
- ❌ 세션 종료 시 요약
- ❌ 유저 레벨 기억 축적
- ❌ Claude Haiku 요약 호출

**MVP 방식**:
- ✅ 최근 20개 메시지만 유지
- ✅ 그 이전은 버림 (DB에는 저장, AI는 모름)
- ✅ 추론된 프로필만 저장
- ✅ 추가 AI 호출 없음

**비용**: $0 (추가 AI 호출 없음)
**복잡도**: 낮음

### 5.2 컨텍스트 윈도우 구조

```
API 호출 시 컨텍스트

[시스템 프롬프트]
  - 월렛 버핏 페르소나
  - 유저 프로필 (추론된 정보)
  - 현재 시장 데이터

[최근 20개 메시지]      <- 원본 그대로
  - user: "비트코인 어때?"
  - assistant: "자네, 지금..."
  - user: "더 살까?"
  - assistant: "분할 매수를..."
  - ... (최대 20개)

[현재 질문]
```

### 5.3 구현

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

### 5.4 20개 제한의 의미

```
일반적인 대화 패턴

하루 평균 대화: 5-10회 왕복 (10-20개 메시지)
→ 대부분의 경우 하루 대화는 컨텍스트 내에 유지됨

20개 초과 시:
- 오래된 메시지부터 컨텍스트에서 제외
- DB에는 계속 저장 (히스토리 조회용)
- AI는 최근 대화만 기억

Phase 2에서 추가 가능:
- RAG로 과거 대화 검색
- "3달 전에 뭐라 했지?" 질문 대응
```

---

## 6. 데이터베이스 스키마

### 6.1 ERD

```
MVP Database Schema

+------------------+       +------------------+
|     users        |       |  user_profiles   |
+------------------+       +------------------+
| id (PK)          |------>| id (PK)          |
| email            |       | user_id (FK)     |
| name             |       | experience_level |
| image            |       | investment_style |
| tier             |       | risk_tolerance   |
| push_subscription|       | interested_assets|
| notifications_on |       | confidence_scores|
| created_at       |       | updated_at       |
+------------------+       +------------------+
        |
        |
        v
+------------------+       +------------------+
|     chats        |       |    messages      |
+------------------+       +------------------+
| id (PK)          |------>| id (PK)          |
| user_id (FK)     |       | chat_id (FK)     |
| title            |       | role             |
| created_at       |       | content          |
| updated_at       |       | signal (jsonb)   |
+------------------+       | inferred_profile |
                           | created_at       |
                           +------------------+

+------------------+       +------------------+
| shadow_trades    |       |  alert_history   |
+------------------+       +------------------+
| id (PK)          |       | id (PK)          |
| user_id (FK)     |       | user_id (FK)     |
| chat_id (FK)     |       | title            |
| message_id (FK)  |       | body             |
| symbol           |       | alert_type       |
| action           |       | symbol           |
| price            |       | change_percent   |
| confidence       |       | deep_link        |
| ai_reason        |       | is_read          |
| created_at       |       | clicked_at       |
+------------------+       | created_at       |
                           +------------------+
```

### 6.2 Drizzle 스키마

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
  alertType: varchar('alert_type', { length: 30 }).notNull(), // market_alert, portfolio_alert, daily_briefing
  symbol: varchar('symbol', { length: 10 }),
  changePercent: decimal('change_percent', { precision: 5, scale: 2 }),

  // 딥링크
  deepLink: varchar('deep_link', { length: 500 }), // /chat/new?context=market_alert&symbol=BTC

  // 상태
  isRead: boolean('is_read').default(false),
  clickedAt: timestamp('clicked_at'), // 클릭 시간 (전환 추적용)

  createdAt: timestamp('created_at').defaultNow(),
});
```

---

## 7. API 명세

### 7.1 API 목록

| Method | Endpoint | 설명 | 인증 |
|--------|----------|------|------|
| POST | `/api/auth/[...nextauth]` | Auth.js 인증 | - |
| POST | `/api/chat` | 채팅 메시지 전송 (SSE 스트리밍) | ✅ |
| GET | `/api/chats` | 채팅 목록 조회 | ✅ |
| POST | `/api/chats` | 새 채팅 생성 | ✅ |
| GET | `/api/chats/[chatId]/messages` | 채팅 메시지 조회 | ✅ |
| GET | `/api/market/price` | 가격 조회 (6종) | - |
| GET | `/api/market/news` | 뉴스 조회 | - |
| GET | `/api/market/fear-greed` | Fear and Greed 조회 | - |
| POST | `/api/shadow-trades` | 섀도우 트레이드 추가 | ✅ |
| GET | `/api/shadow-trades` | 섀도우 트레이드 목록 | ✅ |
| POST | `/api/push/subscribe` | 푸시 구독 등록 | ✅ |
| DELETE | `/api/push/subscribe` | 푸시 구독 해제 | ✅ |
| GET | `/api/notifications` | 알림 히스토리 조회 | ✅ |
| PATCH | `/api/notifications/[id]/read` | 알림 읽음 처리 | ✅ |
| PATCH | `/api/notifications/[id]/click` | 알림 클릭 추적 | ✅ |
| GET | `/api/settings` | 설정 조회 | ✅ |
| PATCH | `/api/settings` | 설정 업데이트 | ✅ |
| GET | `/api/profile` | 추론된 프로필 조회 | ✅ |
| GET | `/api/viral/comment` | 바이럴 사이트용 AI 한마디 | - |
| GET | `/api/viral/stats` | 바이럴 사이트용 실시간 통계 | - |

### 7.2 주요 API 상세

#### POST /api/chat

**Request**:
```typescript
{
  "chatId": "uuid",
  "message": "비트코인 지금 어때?"
}
```

**Response** (SSE Stream):
```
data: {"type":"text","content":"자"}
data: {"type":"text","content":"네"}
...
data: {"type":"profile","inferredProfile":{"experienceLevel":"beginner","confidence":0.85}}
data: {"type":"signal","signal":{"action":"buy","symbol":"BTC","confidence":0.75}}
data: {"type":"done"}
```

#### GET /api/profile

**Response**:
```typescript
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

## 8. 외부 API 연동

### 8.1 Market Data APIs

| API | 용도 | Rate Limit | 캐싱 (Redis TTL) |
|-----|------|-----------|------------------|
| **CoinGecko** | 가격 (6종: BTC, ETH, SOL, BNB, DOGE, XRP) | 50/min | 5분 |
| **CryptoPanic** | 뉴스 | 100/day | 10분 |
| **Alternative.me** | Fear & Greed Index | 무제한 | 30분 |

### 8.2 구현 예시

```typescript
// lib/api/coingecko.ts

export async function getPriceFromCoinGecko(symbol: string): Promise<number> {
  const cacheKey = `price:${symbol}`;

  // Redis 캐시 확인
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // API 호출
  const res = await fetch(
    `https://api.coingecko.com/api/v3/simple/price?ids=${symbol}&vs_currencies=usd`
  );
  const data = await res.json();
  const price = data[symbol].usd;

  // Redis에 5분 캐싱
  await redis.setex(cacheKey, 300, JSON.stringify(price));

  return price;
}
```

---

## 9. 성능 목표

### 9.1 응답 시간

| 메트릭 | 목표 | 측정 방법 |
|-------|------|----------|
| **채팅 응답 시작** (TTFT) | 2초 이내 | SSE 첫 토큰까지 |
| **컨텍스트 로드** | 0.5초 이내 | DB 쿼리 시간 |
| **API 타임아웃** | 10초 | External API 호출 |
| **알림 발송** | 15분 간격 | Lambda Cron 정확도 |

### 9.2 동시 접속자

| Stage | Target Users | Infrastructure |
|-------|-------------|----------------|
| **MVP** | 1,000 | ECS Fargate (2 tasks) |
| **Q1 2026** | 5,000 | ECS Auto Scaling (5 tasks) |
| **Q4 2026** | 10,000 | ECS Auto Scaling (10 tasks) |

### 9.3 환각률

**목표**: <1%

**측정**:
- 사용자 신고 ("이 정보가 틀렸어요" 피드백)
- 자동 검증 (Price API와 LLM 응답 수치 비교)

---

## 10. 보안 및 인증

### 10.1 인증

- **Auth.js v5**: Google OAuth 2.0
- **세션**: JWT (HttpOnly cookie)
- **CSRF**: Next.js 기본 보호

### 10.2 Rate Limiting

```typescript
// middleware.ts

import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, "1 m"), // 분당 10회
});

export async function middleware(req: NextRequest) {
  const ip = req.ip ?? "127.0.0.1";
  const { success } = await ratelimit.limit(ip);

  if (!success) {
    return new Response("Too Many Requests", { status: 429 });
  }

  return NextResponse.next();
}
```

### 10.3 데이터 보호

- **PII 암호화**: 이메일, 이름 (AES-256)
- **API Key 관리**: AWS Secrets Manager
- **HTTPS Only**: CloudFront 강제

---

## 부록

### A. 참고 문서

- [MVP Definition](../business/mvp-definition.md) - 8주 개발 계획
- [Frontend Specification](frontend-spec.md) - Next.js 15 + Vite
- [Infrastructure Specification](infrastructure-spec.md) - AWS 아키텍처
- [Database Schema](database-schema.md) - Drizzle 상세
- [Backend AI Guide](../ai-guides/backend-ai-guide.md) - Claude Code 개발 가이드

### B. 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|-----|------|----------|--------|
| 1.0 | 2025-12-17 | 초안 작성 | Sam |

---

**문서 버전**: 1.0
**최종 수정일**: 2025년 12월 17일
**다음 리뷰 예정**: 2026년 1월 1일 (Phase 1 완료 시)
