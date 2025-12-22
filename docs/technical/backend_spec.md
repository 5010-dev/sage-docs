# Sage.ai Backend Specification

> Document Version: 2.0
> Last Modified: 2025-12-22
> Author: Sam
> Target Audience: Backend Developers

---

## 1. Technical Stack

### 1.1 Core Components

```typescript
interface CoreStack {
  runtime: {
    name: "Node.js";
    version: "20 LTS";
    reason: "Stability and ecosystem";
  };
  language: {
    name: "TypeScript";
    version: "5.x";
    reason: "Type safety";
  };
  framework: {
    name: "Nest.js";
    version: "10.x";
    reason: "Modular structure, DI, TypeScript native";
  };
  orm: {
    name: "Prisma";
    version: "5.x";
    reason: "Type safety, intuitive migrations";
  };
  database: {
    name: "PostgreSQL";
    version: "16";
    reason: "JSON support, stability, scalability";
  };
  cache: {
    name: "Redis";
    version: "7.x";
    reason: "Sessions, caching, BullMQ backend";
  };
}
```

### 1.2 Async & Scheduling

```typescript
interface AsyncComponents {
  jobQueue: {
    name: "BullMQ";
    version: "5.x";
    purpose: "Memory extraction, notification sending";
  };
  cronJobs: {
    name: "@nestjs/schedule";
    purpose: "Price polling (15-minute intervals)";
  };
}
```

### 1.3 External Services

```typescript
interface ExternalServices {
  ai: {
    provider: "Anthropic Claude";
    api: "@anthropic-ai/sdk";
  };
  marketData: {
    provider: "CoinGecko";
    type: "REST API";
  };
  fearGreed: {
    provider: "Alternative.me";
    type: "REST API";
  };
  notifications: {
    provider: "Discord";
    type: "Webhook";
  };
}
```

---

## 2. Architecture

### 2.1 Layered Architecture (Clean Lite)

```mermaid
graph TD
    A[Presentation Layer] --> B[Application Layer]
    B --> C[Domain Layer]
    C --> D[Infrastructure Layer]

    A1[Controllers - HTTP/SSE] --> A
    B1[Services - Business Logic] --> B
    C1[Entities, DTOs] --> C
    D1[Prisma, Redis, External APIs] --> D
```

### 2.2 Folder Structure

```
src/
├── main.ts                      # App entry point
├── app.module.ts                # Root module
│
├── common/                      # Common utilities
│   ├── filters/                 # Exception Filters
│   ├── guards/                  # Auth Guards
│   ├── interceptors/            # Logging, Transform
│   ├── pipes/                   # Validation Pipes
│   └── decorators/              # Custom Decorators
│
├── config/                      # Configuration
│   ├── database.config.ts
│   ├── redis.config.ts
│   └── anthropic.config.ts
│
├── modules/                     # Feature modules
│   ├── auth/                    # Authentication
│   ├── chat/                    # Chat functionality
│   ├── ai-agents/               # AI agents
│   ├── market/                  # Market data
│   ├── portfolio/               # Shadow portfolio
│   ├── notifications/           # Notifications
│   ├── scheduler/               # Scheduled tasks
│   └── jobs/                    # Background Jobs
│
└── prisma/                      # Prisma Schema & Migrations
```

---

## 3. Database Schema

### 3.1 Users Table

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

### 3.2 Chats Table

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

### 3.3 Messages Table

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

### 3.4 Shadow Trades Table

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

### 3.5 Push Subscriptions Table

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

### 3.6 Notifications Table

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

## 4. API Endpoints

### 4.1 Authentication Endpoints

```typescript
interface AuthEndpoints {
  googleOAuth: "POST /api/auth/google";
  logout: "POST /api/auth/logout";
  getSession: "GET /api/auth/session";
}
```

### 4.2 Chat Endpoints

```typescript
interface ChatEndpoints {
  createChat: "POST /api/chats";
  listChats: "GET /api/chats";
  getChatDetails: "GET /api/chats/:id";
  deleteChat: "DELETE /api/chats/:id";
  sendMessage: "POST /api/chats/:id/messages"; // SSE response
  getMessages: "GET /api/chats/:id/messages"; // with pagination
}
```

### 4.3 Market Endpoints

```typescript
interface MarketEndpoints {
  getCurrentPrices: "GET /api/market/prices"; // 6 coins
  getFearGreed: "GET /api/market/fear-greed";
  getPriceHistory: "GET /api/market/history/:symbol"; // 24h
}
```

### 4.4 Portfolio Endpoints

```typescript
interface PortfolioEndpoints {
  createTrade: "POST /api/shadow-trades";
  listTrades: "GET /api/shadow-trades";
  getPerformance: "GET /api/shadow-trades/performance";
  deleteTrade: "DELETE /api/shadow-trades/:id";
}
```

### 4.5 Notification Endpoints

```typescript
interface NotificationEndpoints {
  subscribePush: "POST /api/push/subscribe";
  unsubscribePush: "POST /api/push/unsubscribe";
  getNotifications: "GET /api/notifications";
  markAsRead: "PATCH /api/notifications/:id/read";
}
```

---

## 5. Multi-Agent System

### 5.1 Agent Architecture Flow

```mermaid
graph TD
    A[User Message] --> B[Manager Agent - Haiku 4]
    B -->|Intent: market_data| C[Analyst Agent - Haiku 4]
    B -->|Intent: advice| D[Analyst + Persona Agent - Sonnet 4]
    B -->|Intent: general| E[Persona Agent - Sonnet 4]
    C --> F[Risk Agent - Haiku 4]
    D --> F
    E --> F
    F --> G[Final Response]
```

### 5.2 Manager Agent

```typescript
interface ManagerResponse {
  intent: 'market_data' | 'advice' | 'portfolio' | 'general';
  entities: {
    symbols?: string[];  // ["BTC", "ETH"]
    timeframe?: string;  // "24h", "7d"
  };
  needsMarketData: boolean;
}

// Example
const exampleInput = "비트코인 지금 어때?";
const exampleOutput: ManagerResponse = {
  intent: "advice",
  entities: { symbols: ["BTC"] },
  needsMarketData: true
};
```

### 5.3 Analyst Agent Tools

```typescript
interface AnalystTools {
  getPricetool: {
    name: "get_price";
    description: "Get current price and 24h change for a coin";
    input_schema: {
      type: "object";
      properties: {
        symbol: { type: "string"; enum: ["BTC", "ETH", "SOL", "BNB", "DOGE", "XRP"] };
      };
      required: ["symbol"];
    };
  };
  getFearGreedTool: {
    name: "get_fear_greed";
    description: "Get current Fear & Greed Index (0-100)";
    input_schema: { type: "object"; properties: {} };
  };
}

interface AnalystOutput {
  facts: {
    BTC: { price: 43250; change_24h: -5.2 };
    fear_greed: 25;
  };
  summary: "BTC is at $43,250 (-5.2% in 24h). Market sentiment is 'Extreme Fear' (25/100).";
}
```

### 5.4 Persona Agent System Prompt

```typescript
const personaSystemPrompt = `
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
`;
```

### 5.5 Risk Agent Validation

```typescript
interface RiskCheck {
  hasHallucination: boolean;      // Does number match Tool data?
  hasDirectSignal: boolean;        // Contains "buy now" type signals?
  hasBias: boolean;                // Excessive optimism/pessimism?
  recommendation: 'approve' | 'revise' | 'reject';
}

// Example validation
const validateResponse = (
  personaResponse: string,
  factData: AnalystOutput
): RiskCheck => {
  // Check for hallucinated prices
  const mentionedPrice = extractPrice(personaResponse);
  const actualPrice = factData.facts.BTC.price;

  if (Math.abs(mentionedPrice - actualPrice) > actualPrice * 0.01) {
    return {
      hasHallucination: true,
      hasDirectSignal: false,
      hasBias: false,
      recommendation: 'revise'
    };
  }

  return {
    hasHallucination: false,
    hasDirectSignal: false,
    hasBias: false,
    recommendation: 'approve'
  };
};
```

---

## 6. Caching Strategy

### 6.1 Redis Cache Keys

```typescript
interface CacheKeys {
  // Market data (TTL: 5 minutes)
  marketPrice: `market:price:${symbol}`;           // "43250.50"
  marketFearGreed: `market:fear-greed`;            // "25"
  marketHistory: `market:history:${symbol}:24h`;   // JSON array

  // User context (TTL: 1 hour)
  userRecentMessages: `user:${userId}:recent-messages`;  // Last 20 messages
  userProfile: `user:${userId}:profile`;                 // Inferred profile

  // Rate limiting (TTL: 1 minute)
  rateLimit: `ratelimit:${userId}:chat`;                 // Request count
}
```

### 6.2 Cache Invalidation Strategy

```typescript
interface CacheInvalidation {
  marketData: "TTL-based (5 minutes)";
  userContext: "Update on new message";
  rateLimit: "TTL-based (1 minute)";
}
```

---

## 7. Background Jobs (BullMQ)

### 7.1 Job Queue Configuration

```typescript
interface JobQueues {
  memoryExtraction: {
    name: 'memory-extraction';
    processor: 'memory-extraction.processor.ts';
    trigger: '10 minutes after last message';
    priority: 'low';
  };
  notifications: {
    name: 'notifications';
    processor: 'notification.processor.ts';
    trigger: 'Market sudden change detected';
    priority: 'high';
  };
  marketAnalysis: {
    name: 'market-analysis';
    processor: 'market-analyzer.processor.ts';
    trigger: 'Every 15 minutes (@nestjs/schedule)';
    priority: 'medium';
  };
}
```

### 7.2 Memory Extraction Job

```typescript
interface MemoryExtractionJob {
  chatId: string;
  userId: string;
  messages: Message[];
}

async function processMemoryExtraction(job: Job<MemoryExtractionJob>) {
  const { messages, userId } = job.data;

  // Call Haiku 4 to extract profile
  const profile = await extractProfile(messages);

  // Update user profile
  await prisma.user.update({
    where: { id: userId },
    data: {
      riskProfile: profile.riskProfile,
      interests: profile.interests
    }
  });
}
```

### 7.3 Notification Job

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

async function processNotification(job: Job<NotificationJob>) {
  const { type, userId, data } = job.data;

  // Send push notification
  await pushService.send(userId, {
    title: `${data.symbol} Alert`,
    body: data.message
  });

  // Send Discord webhook if change > 7%
  if (Math.abs(data.change) > 7) {
    await discordService.sendAlert(data);
  }

  // Save to DB
  await prisma.notification.create({ ... });
}
```

---

## 8. Scheduled Tasks

### 8.1 Price Polling Implementation

```typescript
interface PricePollingConfig {
  schedule: "*/15 * * * *";  // Every 15 minutes
  symbols: ["BTC", "ETH", "SOL", "BNB", "DOGE", "XRP"];
  changeThresholds: {
    BTC: 5;     // 5% change triggers alert
    ETH: 7;     // 7% change triggers alert
    default: 10; // 10% for others
  };
}

@Cron('*/15 * * * *')
async function pollPrices() {
  const symbols = ['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'];

  for (const symbol of symbols) {
    const currentPrice = await coingeckoClient.getPrice(symbol);
    const previousPrice = await redis.get(`market:price:${symbol}`);

    // Cache new price
    await redis.setex(`market:price:${symbol}`, 300, currentPrice);

    // Check for sudden change
    if (previousPrice) {
      const change = ((currentPrice - previousPrice) / previousPrice) * 100;

      if (isSuddenChange(symbol, change)) {
        // Trigger notification job
        await notificationQueue.add('market_alert', {
          type: 'market_alert',
          data: { symbol, change, message: `${symbol} ${change > 0 ? 'surge' : 'drop'} ${Math.abs(change).toFixed(2)}%` }
        });
      }
    }
  }
}
```

---

## 9. Error Handling

### 9.1 Custom Exceptions

```typescript
class HallucinationDetectedException extends BadRequestException {
  constructor(actual: number, claimed: number) {
    super(`AI hallucination detected: claimed ${claimed}, actual ${actual}`);
  }
}

class RateLimitExceededException extends TooManyRequestsException {
  constructor(limit: number) {
    super(`Rate limit exceeded: ${limit} requests per minute`);
  }
}

class TierLimitException extends ForbiddenException {
  constructor(feature: string, requiredTier: string) {
    super(`Feature '${feature}' requires ${requiredTier} tier`);
  }
}
```

### 9.2 Global Exception Filter

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

## 10. Performance Targets

### 10.1 Performance Metrics

```typescript
interface PerformanceTargets {
  apiResponseTime: {
    metric: "95th percentile";
    target: "< 200ms";
    measurement: "Sentry Traces";
  };
  sseFirstToken: {
    metric: "Time to first token";
    target: "< 2s";
    measurement: "Custom metric";
  };
  databaseQuery: {
    metric: "Query duration";
    target: "< 50ms";
    measurement: "Prisma Logging";
  };
  cacheHitRate: {
    metric: "Hit rate";
    target: "> 80%";
    measurement: "Redis INFO stats";
  };
}
```

---

## 11. Security

### 11.1 Authentication & Authorization

```typescript
interface SecurityConfig {
  authentication: {
    provider: "Auth.js";
    method: "Google OAuth";
    storage: "PostgreSQL or JWT";
    csrfProtection: true;
  };
  authorization: {
    guards: ["AuthGuard", "TierGuard"];
    example: "@RequireTier('pro')";
  };
}
```

### 11.2 Input Validation

```typescript
class CreateTradeDto {
  @IsIn(['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'])
  symbol: string;

  @IsIn(['buy', 'sell'])
  action: string;

  @IsNumber()
  @Min(0)
  price: number;
}
```

### 11.3 Rate Limiting

```typescript
@UseGuards(ThrottlerGuard)
@Throttle(10, 60)  // 10 requests per minute
async sendMessage() {
  // Implementation
}
```

---

## 12. Monitoring

### 12.1 Custom Metrics

```typescript
interface CustomMetrics {
  chatResponseTime: "histogram";
  aiHallucinationRate: "counter";
  marketApiCalls: "counter";
  jobsCompleted: "counter";
  jobsFailed: "counter";
}
```

### 12.2 Alerting Rules

```typescript
interface AlertingRules {
  apiErrorRate: "> 5% for 5 minutes";
  sseFirstToken: "> 5s for 1 minute";
  databaseConnectionPool: "> 90%";
  redisMemory: "> 80%";
  bullmqQueueSize: "> 1000";
}
```

---

## Appendix A: Environment Variables

```bash
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

## Appendix B: Development Commands

```bash
# Development server
pnpm run start:dev              # Start with watch mode
pnpm run start:debug            # Start in debug mode

# Build
pnpm run build                  # Production build
pnpm run start:prod             # Run production build

# Database (Prisma)
npx prisma generate             # Generate Prisma Client
npx prisma migrate dev          # Create and apply migration
npx prisma migrate deploy       # Apply migrations in production
npx prisma studio               # Open Prisma Studio (DB GUI)

# Testing
pnpm run test                   # Run unit tests
pnpm run test:watch             # Run tests in watch mode
pnpm run test:cov               # Generate coverage report
pnpm run test:e2e               # Run e2e tests

# Linting
pnpm run lint                   # Run ESLint
pnpm run format                 # Format with Prettier
```

---

Document Version: 2.0
Last Updated: 2025-12-22
Architecture: Layered + Domain (Clean Lite), TypeScript Fullstack
Maintainer: Sam (dev@5010.tech)
