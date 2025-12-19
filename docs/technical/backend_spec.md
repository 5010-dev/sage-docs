# Sage.ai Backend Specification

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: Backend 개발자

---

## 기술 스택

### Core

| 컴포넌트 | 기술 | 버전 | 선택 이유 |
|---------|------|------|-----------|
| **Runtime** | Node.js | 20 LTS | 안정성, 생태계 |
| **언어** | TypeScript | 5.x | 타입 안정성 |
| **프레임워크** | Nest.js | 10.x | 모듈러 구조, DI, TypeScript 네이티브 |
| **ORM** | Prisma | 5.x | 타입 안정성, 직관적 마이그레이션 |
| **Database** | PostgreSQL | 16 | JSON 지원, 안정성, 확장성 |
| **Cache** | Redis | 7.x | 세션, 캐싱, BullMQ 백엔드 |

### Async & Scheduling

| 컴포넌트 | 기술 | 용도 |
|---------|------|------|
| **Job Queue** | BullMQ | 5.x | Memory 추출, 알림 발송 |
| **Cron Jobs** | @nestjs/schedule | - | 가격 폴링 (15분) |

### External Services

| 서비스 | 용도 | API |
|--------|------|-----|
| **Anthropic Claude** | AI 멘토링 | @anthropic-ai/sdk |
| **CoinGecko** | 시장 데이터 | REST API |
| **Alternative.me** | Fear & Greed Index | REST API |
| **Discord** | 알림 | Webhook |

---

## 아키텍처

### Layered Architecture (Clean Lite)

```
┌─────────────────────────────────────┐
│         Presentation Layer          │  ← Controllers (HTTP/SSE)
├─────────────────────────────────────┤
│          Application Layer          │  ← Services (Business Logic)
├─────────────────────────────────────┤
│           Domain Layer              │  ← Entities, DTOs
├─────────────────────────────────────┤
│        Infrastructure Layer         │  ← Prisma, Redis, External APIs
└─────────────────────────────────────┘
```

### Folder Structure

```
src/
├── main.ts                      # 앱 엔트리포인트
├── app.module.ts                # 루트 모듈
│
├── common/                      # 공통 유틸리티
│   ├── filters/                 # Exception Filters
│   ├── guards/                  # Auth Guards
│   ├── interceptors/            # Logging, Transform
│   ├── pipes/                   # Validation Pipes
│   └── decorators/              # Custom Decorators
│
├── config/                      # 설정
│   ├── database.config.ts
│   ├── redis.config.ts
│   └── anthropic.config.ts
│
├── modules/                     # 기능 모듈
│   ├── auth/                    # 인증
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.module.ts
│   │   ├── dto/
│   │   └── guards/
│   │
│   ├── chat/                    # 채팅
│   │   ├── chat.controller.ts
│   │   ├── chat.service.ts
│   │   ├── chat.module.ts
│   │   ├── dto/
│   │   └── entities/
│   │
│   ├── ai-agents/               # AI 에이전트
│   │   ├── manager.agent.ts     # 라우팅
│   │   ├── analyst.agent.ts     # 데이터 수집
│   │   ├── persona.agent.ts     # 월렛 버핏
│   │   ├── risk.agent.ts        # 교차 검증
│   │   ├── orchestrator.service.ts
│   │   └── ai-agents.module.ts
│   │
│   ├── market/                  # 시장 데이터
│   │   ├── market.controller.ts
│   │   ├── market.service.ts
│   │   ├── market.module.ts
│   │   ├── coingecko.client.ts
│   │   └── fear-greed.client.ts
│   │
│   ├── portfolio/               # 섀도우 포트폴리오
│   │   ├── portfolio.controller.ts
│   │   ├── portfolio.service.ts
│   │   ├── portfolio.module.ts
│   │   ├── dto/
│   │   └── entities/
│   │
│   ├── notifications/           # 알림
│   │   ├── notifications.controller.ts
│   │   ├── notifications.service.ts
│   │   ├── notifications.module.ts
│   │   ├── push.service.ts
│   │   └── discord.service.ts
│   │
│   ├── scheduler/               # 주기적 작업
│   │   ├── price-polling.service.ts
│   │   ├── market-analyzer.service.ts
│   │   └── scheduler.module.ts
│   │
│   └── jobs/                    # Background Jobs
│       ├── memory-extraction.processor.ts
│       ├── notification.processor.ts
│       └── jobs.module.ts
│
└── prisma/                      # Prisma Schema & Migrations
    ├── schema.prisma
    └── migrations/
```

---

## Database Schema

### Users

```prisma
model User {
  id            String    @id @default(uuid())
  email         String    @unique
  name          String?
  image         String?
  tier          String    @default("free")  // free, pro, premium

  // Preferences (inferred from chat)
  riskProfile   String?   // conservative, moderate, aggressive
  interests     Json?     // ["BTC", "ETH"]

  chats         Chat[]
  shadowTrades  ShadowTrade[]
  pushSubscriptions PushSubscription[]

  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}
```

### Chats

```prisma
model Chat {
  id        String    @id @default(uuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  title     String    @default("새 대화")
  messages  Message[]

  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  @@index([userId, createdAt])
}
```

### Messages

```prisma
model Message {
  id        String   @id @default(uuid())
  chatId    String
  chat      Chat     @relation(fields: [chatId], references: [id], onDelete: Cascade)

  role      String   // user, assistant
  content   String   @db.Text

  // AI Signal (for shadow portfolio)
  signal    Json?    // { action: "buy" | "sell", symbol: "BTC", confidence: 0.8 }

  createdAt DateTime @default(now())

  @@index([chatId, createdAt])
}
```

### ShadowTrades

```prisma
model ShadowTrade {
  id        String   @id @default(uuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  symbol    String   // BTC, ETH, SOL, BNB, DOGE, XRP
  action    String   // buy, sell
  price     Decimal  @db.Decimal(18, 8)
  quantity  Decimal  @db.Decimal(18, 8) @default(1.0)

  // Reference to message that triggered this
  messageId String?

  createdAt DateTime @default(now())

  @@index([userId, createdAt])
  @@index([symbol])
}
```

### PushSubscriptions

```prisma
model PushSubscription {
  id        String   @id @default(uuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  endpoint  String   @unique
  auth      String
  p256dh    String

  createdAt DateTime @default(now())

  @@index([userId])
}
```

### Notifications

```prisma
model Notification {
  id        String   @id @default(uuid())
  userId    String?  // null = broadcast

  type      String   // market_alert, portfolio_update, system
  title     String
  message   String   @db.Text
  data      Json?    // { symbol: "BTC", change: -5.2 }

  read      Boolean  @default(false)
  sentAt    DateTime @default(now())

  @@index([userId, sentAt])
}
```

---

## API Endpoints

### Authentication

```
POST   /api/auth/google          # Google OAuth callback
POST   /api/auth/logout          # Logout
GET    /api/auth/session         # Get current session
```

### Chat

```
POST   /api/chats                # Create new chat
GET    /api/chats                # List user's chats
GET    /api/chats/:id            # Get chat details
DELETE /api/chats/:id            # Delete chat

POST   /api/chats/:id/messages   # Send message (SSE response)
GET    /api/chats/:id/messages   # Get messages (pagination)
```

### Market

```
GET    /api/market/prices        # Get current prices (6 coins)
GET    /api/market/fear-greed    # Get Fear & Greed Index
GET    /api/market/history/:symbol  # Get price history (24h)
```

### Portfolio

```
POST   /api/shadow-trades        # Create shadow trade
GET    /api/shadow-trades        # List user's trades
GET    /api/shadow-trades/performance  # Calculate performance
DELETE /api/shadow-trades/:id    # Delete trade
```

### Notifications

```
POST   /api/push/subscribe       # Subscribe to push notifications
POST   /api/push/unsubscribe     # Unsubscribe
GET    /api/notifications        # Get notification history
PATCH  /api/notifications/:id/read  # Mark as read
```

---

## Multi-Agent System

### Agent Architecture

```
[User Message]
    ↓
[Manager Agent] (Haiku 4)
    │
    ├─> Intent: market_data    → [Analyst Agent] (Haiku 4)
    ├─> Intent: advice         → [Analyst + Persona] (Sonnet 4)
    └─> Intent: general        → [Persona Agent] (Sonnet 4)
    ↓
[Risk Agent] (Haiku 4) - Cross-validation
    ↓
[Final Response]
```

### Manager Agent (manager.agent.ts)

**Responsibility**: 사용자 의도 파악 및 라우팅

```typescript
interface ManagerResponse {
  intent: 'market_data' | 'advice' | 'portfolio' | 'general';
  entities: {
    symbols?: string[];  // ["BTC", "ETH"]
    timeframe?: string;  // "24h", "7d"
  };
  needsMarketData: boolean;
}
```

**Example**:
```
Input: "비트코인 지금 어때?"
Output: {
  intent: "advice",
  entities: { symbols: ["BTC"] },
  needsMarketData: true
}
```

### Analyst Agent (analyst.agent.ts)

**Responsibility**: 시장 데이터 수집 및 팩트 정리

**Tools**:
```typescript
const tools = [
  {
    name: "get_price",
    description: "Get current price and 24h change for a coin",
    input_schema: {
      type: "object",
      properties: {
        symbol: { type: "string", enum: ["BTC", "ETH", "SOL", "BNB", "DOGE", "XRP"] }
      },
      required: ["symbol"]
    }
  },
  {
    name: "get_fear_greed",
    description: "Get current Fear & Greed Index (0-100)",
    input_schema: { type: "object", properties: {} }
  }
];
```

**Output**:
```json
{
  "facts": {
    "BTC": { "price": 43250, "change_24h": -5.2 },
    "fear_greed": 25
  },
  "summary": "BTC is at $43,250 (-5.2% in 24h). Market sentiment is 'Extreme Fear' (25/100)."
}
```

### Persona Agent (persona.agent.ts)

**Responsibility**: 월렛 버핏 페르소나로 응답 생성

**System Prompt**:
```
You are Wallet Buffett (월렛 버핏), an AI investment mentor inspired by Warren Buffett.

Personality:
- Experienced, calm, and wise
- Uses "자네", "~일세", "~하게" (Korean honorific mixing)
- Provides insights, not just information
- Focuses on long-term value, not short-term speculation

Rules:
- NEVER give direct trading signals ("Buy now", "Sell immediately")
- ALWAYS use conditional language ("~할 수 있다", "~를 고려해볼 만하다")
- ALWAYS cite data from tools (use get_price, get_fear_greed)
- NEVER hallucinate numbers - use tools or say "I don't have that data"

Example:
User: "비트코인 지금 어때?"
Wallet Buffett: "자네, 비트코인이 현재 $43,250에 거래되고 있네 (24시간 -5.2%).
시장은 공포에 질렸군 (Fear & Greed 지수 25). 하지만 기억하게,
남들이 두려워할 때가 바로 기회일세. 장기 관점에서 접근하면 어떻겠나?"
```

### Risk Agent (risk.agent.ts)

**Responsibility**: 교차 검증 및 환각 방지

**Checks**:
```typescript
interface RiskCheck {
  hasHallucination: boolean;      // 숫자가 Tool 데이터와 일치하는가?
  hasDirectSignal: boolean;        // "지금 사세요" 같은 직접 신호가 있는가?
  hasBias: boolean;                // 과도한 낙관/비관이 있는가?
  recommendation: 'approve' | 'revise' | 'reject';
}
```

**Example**:
```typescript
// Persona Agent output
const personaResponse = "비트코인이 $50,000이네요";

// Risk Agent check
const riskCheck = {
  hasHallucination: true,  // Real price: $43,250
  recommendation: 'revise'
};

// Trigger re-generation with correct data
```

---

## Caching Strategy

### Redis Cache Keys

```typescript
// Market data (TTL: 5분)
`market:price:${symbol}`           // "43250.50"
`market:fear-greed`                // "25"
`market:history:${symbol}:24h`    // JSON array

// User context (TTL: 1시간)
`user:${userId}:recent-messages`   // Last 20 messages
`user:${userId}:profile`           // Inferred profile

// Rate limiting (TTL: 1분)
`ratelimit:${userId}:chat`         // Request count
```

### Cache Invalidation

- **Market data**: TTL 기반 (5분)
- **User context**: 새 메시지 발생 시 갱신
- **Rate limit**: TTL 기반 (1분)

---

## Background Jobs (BullMQ)

### Job Queues

```typescript
// Memory extraction (낮은 우선순위)
Queue: 'memory-extraction'
Processor: 'memory-extraction.processor.ts'
Trigger: 대화 종료 후 (마지막 메시지 10분 경과)

// Notification (높은 우선순위)
Queue: 'notifications'
Processor: 'notification.processor.ts'
Trigger: 시장 급변 감지 시

// Market analysis (중간 우선순위)
Queue: 'market-analysis'
Processor: 'market-analyzer.processor.ts'
Trigger: 15분마다 (@nestjs/schedule)
```

### Memory Extraction Job

**Purpose**: 대화에서 사용자 프로필 추출 및 업데이트

```typescript
interface MemoryExtractionJob {
  chatId: string;
  userId: string;
  messages: Message[];
}

// Processor
async processMemoryExtraction(job: Job<MemoryExtractionJob>) {
  const { messages, userId } = job.data;

  // Call Haiku 4 to extract profile
  const profile = await this.extractProfile(messages);

  // Update user profile
  await this.prisma.user.update({
    where: { id: userId },
    data: {
      riskProfile: profile.riskProfile,
      interests: profile.interests
    }
  });
}
```

### Notification Job

```typescript
interface NotificationJob {
  type: 'market_alert' | 'portfolio_update';
  userId?: string;  // null = broadcast
  data: {
    symbol: string;
    change: number;
    message: string;
  };
}

// Processor
async processNotification(job: Job<NotificationJob>) {
  const { type, userId, data } = job.data;

  // Send push notification
  await this.pushService.send(userId, {
    title: `${data.symbol} Alert`,
    body: data.message
  });

  // Send Discord webhook
  if (Math.abs(data.change) > 7) {
    await this.discordService.sendAlert(data);
  }

  // Save to DB
  await this.prisma.notification.create({ ... });
}
```

---

## Scheduled Tasks

### Price Polling (15분마다)

```typescript
@Cron('*/15 * * * *')  // Every 15 minutes
async pollPrices() {
  const symbols = ['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'];

  for (const symbol of symbols) {
    const currentPrice = await this.coingeckoClient.getPrice(symbol);
    const previousPrice = await this.redis.get(`market:price:${symbol}`);

    // Cache new price
    await this.redis.setex(`market:price:${symbol}`, 300, currentPrice);

    // Check for sudden change
    if (previousPrice) {
      const change = ((currentPrice - previousPrice) / previousPrice) * 100;

      if (this.isSuddenChange(symbol, change)) {
        // Trigger notification job
        await this.notificationQueue.add('market_alert', {
          type: 'market_alert',
          data: { symbol, change, message: `${symbol} ${change > 0 ? '급등' : '급락'} ${Math.abs(change).toFixed(2)}%` }
        });

        // Generate AI context
        await this.marketAnalysisQueue.add('analyze', { symbol, change });
      }
    }
  }
}

private isSuddenChange(symbol: string, change: number): boolean {
  const thresholds = {
    'BTC': 5,
    'ETH': 7,
    'default': 10
  };
  return Math.abs(change) >= (thresholds[symbol] || thresholds.default);
}
```

---

## Error Handling

### Custom Exceptions

```typescript
// common/exceptions/

export class HallucinationDetectedException extends BadRequestException {
  constructor(actual: number, claimed: number) {
    super(`AI hallucination detected: claimed ${claimed}, actual ${actual}`);
  }
}

export class RateLimitExceededException extends TooManyRequestsException {
  constructor(limit: number) {
    super(`Rate limit exceeded: ${limit} requests per minute`);
  }
}

export class TierLimitException extends ForbiddenException {
  constructor(feature: string, requiredTier: string) {
    super(`Feature '${feature}' requires ${requiredTier} tier`);
  }
}
```

### Global Exception Filter

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    // Log to Sentry
    Sentry.captureException(exception);

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception.message || 'Internal server error'
    });
  }
}
```

---

## Testing Strategy

### Unit Tests (Jest)

**Coverage Target**: 80%+

```typescript
// chat.service.spec.ts
describe('ChatService', () => {
  let service: ChatService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        ChatService,
        { provide: PrismaService, useValue: mockPrisma }
      ]
    }).compile();

    service = module.get<ChatService>(ChatService);
  });

  it('should create a new chat', async () => {
    const result = await service.createChat('user-123');
    expect(result.userId).toBe('user-123');
    expect(result.title).toBe('새 대화');
  });
});
```

### Integration Tests (Supertest)

```typescript
// chat.e2e-spec.ts
describe('Chat (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture = await Test.createTestingModule({
      imports: [AppModule]
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/api/chats (POST)', () => {
    return request(app.getHttpServer())
      .post('/api/chats')
      .set('Authorization', 'Bearer valid-token')
      .expect(201)
      .expect(res => {
        expect(res.body).toHaveProperty('id');
      });
  });
});
```

---

## Performance Targets

| 지표 | 목표 | 측정 방법 |
|------|------|----------|
| **API 응답 속도** | 95th percentile < 200ms | Sentry Traces |
| **SSE 첫 토큰** | < 2초 | Custom metric |
| **Database 쿼리** | < 50ms | Prisma Logging |
| **캐시 히트율** | > 80% | Redis INFO stats |

---

## Security

### Authentication

- **Auth.js** (Google OAuth)
- Session stored in PostgreSQL or JWT
- CSRF protection enabled

### Authorization

```typescript
@UseGuards(AuthGuard, TierGuard)
@RequireTier('pro')
async createPortfolio() {
  // Only Pro/Premium users
}
```

### Input Validation

```typescript
// dto/create-trade.dto.ts
export class CreateTradeDto {
  @IsIn(['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'])
  symbol: string;

  @IsIn(['buy', 'sell'])
  action: string;

  @IsNumber()
  @Min(0)
  price: number;
}
```

### Rate Limiting

```typescript
@UseGuards(ThrottlerGuard)
@Throttle(10, 60)  // 10 requests per minute
async sendMessage() {
  // ...
}
```

---

## Monitoring

### Metrics to Track

```typescript
// Custom metrics
- chat.response_time (histogram)
- ai.hallucination_rate (counter)
- market.api_calls (counter)
- jobs.completed (counter)
- jobs.failed (counter)
```

### Alerting Rules

- API 에러율 > 5% (5분)
- SSE 첫 토큰 > 5초 (1분)
- Database connection pool > 90%
- Redis memory > 80%
- BullMQ queue size > 1000

---

**문서 끝**

_"Between the zeros and ones"_

---

## Appendix

### A. Environment Variables

```env
# Database
DATABASE_URL="postgresql://user:pass@localhost:5432/sage"

# Redis
REDIS_URL="redis://localhost:6379"

# Anthropic
ANTHROPIC_API_KEY="sk-ant-..."

# Auth
GOOGLE_CLIENT_ID="..."
GOOGLE_CLIENT_SECRET="..."
NEXTAUTH_SECRET="..."

# External APIs
COINGECKO_API_KEY="..."
DISCORD_WEBHOOK_URL="..."
```

### B. Prisma Commands

```bash
# Generate Prisma Client
npx prisma generate

# Create migration
npx prisma migrate dev --name add_shadow_trades

# Apply migration
npx prisma migrate deploy

# Open Prisma Studio
npx prisma studio
```

### C. Development Workflow

```bash
# Start dev server
pnpm run start:dev

# Run tests
pnpm run test
pnpm run test:e2e

# Lint & format
pnpm run lint
pnpm run format
```
