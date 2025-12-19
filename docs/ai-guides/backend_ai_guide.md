# Backend Development AI Guide

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: Backend 개발자

---

## AI-Powered Backend Development

### Tools

- **Claude Code**: 복잡한 로직, 아키텍처 설계
- **Cursor**: 빠른 CRUD, 보일러플레이트
- **GitHub Copilot**: 실시간 코드 제안

---

## Prompts for Common Tasks

### 1. Creating a New Module

**Prompt**:
```
Create a Nest.js module for shadow portfolio management with the following:

Requirements:
- Controller with REST endpoints (POST /api/shadow-trades, GET /api/shadow-trades)
- Service with business logic
- Prisma integration (ShadowTrade model)
- DTOs with validation (class-validator)
- Unit tests (Jest)
- Follow existing patterns in src/modules/chat

Prisma Schema:
model ShadowTrade {
  id        String   @id @default(uuid())
  userId    String
  symbol    String
  action    String
  price     Decimal
  createdAt DateTime @default(now())
}
```

**Expected Output**:
- `portfolio.module.ts`
- `portfolio.controller.ts`
- `portfolio.service.ts`
- `dto/create-trade.dto.ts`
- `portfolio.service.spec.ts`

---

### 2. Implementing AI Agent

**Prompt**:
```
Implement the Analyst Agent for Sage.ai multi-agent system:

Agent Purpose: Fetch and analyze market data

Tools to use:
1. get_price(symbol: string) → { price: number, change24h: number }
2. get_fear_greed() → { value: number, classification: string }

Requirements:
- Use @anthropic-ai/sdk
- Model: claude-haiku-4-20250303
- Force tool use (tool_choice: "required")
- Return structured facts (JSON)
- Handle tool errors gracefully

Example input: "What's happening with Bitcoin?"
Example output: { facts: { BTC: { price: 43250, change_24h: -5.2 }, fear_greed: 25 }, summary: "..." }
```

**Expected Code**:
```typescript
// ai-agents/analyst.agent.ts
import Anthropic from '@anthropic-ai/sdk';

export class AnalystAgent {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
  }

  async analyze(query: string): Promise<AnalystResponse> {
    const response = await this.client.messages.create({
      model: 'claude-haiku-4-20250303',
      max_tokens: 500,
      tools: [
        {
          name: 'get_price',
          description: 'Get current price and 24h change for a coin',
          input_schema: {
            type: 'object',
            properties: {
              symbol: { type: 'string', enum: ['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'] }
            },
            required: ['symbol']
          }
        },
        {
          name: 'get_fear_greed',
          description: 'Get current Fear & Greed Index (0-100)',
          input_schema: { type: 'object', properties: {} }
        }
      ],
      tool_choice: { type: 'any' },
      messages: [{ role: 'user', content: query }]
    });

    // Process tool calls and return facts
    // ...
  }
}
```

---

### 3. Adding Validation

**Prompt**:
```
Add validation to this DTO using class-validator:

export class CreateTradeDto {
  symbol: string;
  action: string;
  price: number;
}

Requirements:
- symbol must be one of: BTC, ETH, SOL, BNB, DOGE, XRP
- action must be: buy or sell
- price must be positive number
- Add API documentation annotations (@ApiProperty)
```

**Expected Code**:
```typescript
import { ApiProperty } from '@nestjs/swagger';
import { IsIn, IsNumber, Min } from 'class-validator';

export class CreateTradeDto {
  @ApiProperty({
    description: 'Coin symbol',
    enum: ['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'],
    example: 'BTC'
  })
  @IsIn(['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'])
  symbol: string;

  @ApiProperty({
    description: 'Trade action',
    enum: ['buy', 'sell'],
    example: 'buy'
  })
  @IsIn(['buy', 'sell'])
  action: string;

  @ApiProperty({
    description: 'Trade price',
    example: 43250.50,
    minimum: 0
  })
  @IsNumber()
  @Min(0)
  price: number;
}
```

---

### 4. Writing Tests

**Prompt**:
```
Write Jest unit tests for this service:

class PortfolioService {
  constructor(private prisma: PrismaService) {}

  async createTrade(userId: string, dto: CreateTradeDto) {
    return this.prisma.shadowTrade.create({
      data: {
        userId,
        symbol: dto.symbol,
        action: dto.action,
        price: dto.price
      }
    });
  }

  async getTrades(userId: string) {
    return this.prisma.shadowTrade.findMany({
      where: { userId },
      orderBy: { createdAt: 'desc' }
    });
  }
}

Requirements:
- Mock PrismaService
- Test happy path
- Test error cases
- Coverage > 80%
```

**Expected Code**:
```typescript
// portfolio.service.spec.ts
import { Test } from '@nestjs/testing';
import { PortfolioService } from './portfolio.service';
import { PrismaService } from '../prisma/prisma.service';

describe('PortfolioService', () => {
  let service: PortfolioService;
  let prisma: PrismaService;

  const mockPrisma = {
    shadowTrade: {
      create: jest.fn(),
      findMany: jest.fn()
    }
  };

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        PortfolioService,
        { provide: PrismaService, useValue: mockPrisma }
      ]
    }).compile();

    service = module.get<PortfolioService>(PortfolioService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  describe('createTrade', () => {
    it('should create a shadow trade', async () => {
      const dto = { symbol: 'BTC', action: 'buy', price: 43250 };
      const expected = { id: '123', userId: 'user-1', ...dto, createdAt: new Date() };

      mockPrisma.shadowTrade.create.mockResolvedValue(expected);

      const result = await service.createTrade('user-1', dto);

      expect(result).toEqual(expected);
      expect(mockPrisma.shadowTrade.create).toHaveBeenCalledWith({
        data: { userId: 'user-1', ...dto }
      });
    });
  });

  describe('getTrades', () => {
    it('should return user trades', async () => {
      const expected = [{ id: '123', symbol: 'BTC', action: 'buy', price: 43250 }];

      mockPrisma.shadowTrade.findMany.mockResolvedValue(expected);

      const result = await service.getTrades('user-1');

      expect(result).toEqual(expected);
      expect(mockPrisma.shadowTrade.findMany).toHaveBeenCalledWith({
        where: { userId: 'user-1' },
        orderBy: { createdAt: 'desc' }
      });
    });
  });
});
```

---

### 5. Optimizing Database Queries

**Prompt**:
```
Optimize this Prisma query that has N+1 problem:

async getChatsWithMessageCount(userId: string) {
  const chats = await this.prisma.chat.findMany({
    where: { userId }
  });

  for (const chat of chats) {
    chat.messageCount = await this.prisma.message.count({
      where: { chatId: chat.id }
    });
  }

  return chats;
}

Rewrite using Prisma's include/select to avoid N+1.
```

**Expected Code**:
```typescript
async getChatsWithMessageCount(userId: string) {
  return this.prisma.chat.findMany({
    where: { userId },
    include: {
      _count: {
        select: { messages: true }
      }
    }
  });
}

// Alternative: Use aggregation
async getChatsWithMessageCount(userId: string) {
  return this.prisma.chat.findMany({
    where: { userId },
    select: {
      id: true,
      title: true,
      createdAt: true,
      _count: {
        select: { messages: true }
      }
    }
  });
}
```

---

## AI Code Review Checklist

### Prompt for Code Review

```
Review this code for:

1. **Security**:
   - SQL injection vulnerabilities
   - XSS risks
   - Auth bypass
   - Secrets exposure

2. **Performance**:
   - N+1 queries
   - Memory leaks
   - Inefficient algorithms

3. **Code Quality**:
   - SOLID principles
   - Naming conventions
   - Error handling
   - Type safety

4. **Testing**:
   - Test coverage
   - Edge cases
   - Mocking best practices

Provide specific line numbers and suggestions for improvement.

Code:
[Paste your code here]
```

---

## Debugging with AI

### Prompt for Bug Analysis

```
I'm getting this error:

Error: Cannot read properties of undefined (reading 'id')
  at PortfolioService.createTrade (portfolio.service.ts:15:30)

Code context:
async createTrade(userId: string, dto: CreateTradeDto) {
  const user = await this.prisma.user.findUnique({ where: { id: userId } });
  return this.prisma.shadowTrade.create({
    data: {
      userId: user.id,  // Line 15
      symbol: dto.symbol,
      action: dto.action,
      price: dto.price
    }
  });
}

What's the issue and how to fix it?
```

**AI Response**:
```
The issue is that `user` can be `null` if the user is not found, but you're accessing `user.id` without checking.

Fix:
async createTrade(userId: string, dto: CreateTradeDto) {
  const user = await this.prisma.user.findUnique({ where: { id: userId } });

  if (!user) {
    throw new NotFoundException(`User ${userId} not found`);
  }

  return this.prisma.shadowTrade.create({
    data: {
      userId: user.id,
      symbol: dto.symbol,
      action: dto.action,
      price: dto.price
    }
  });
}

// OR, simply use userId directly if you trust it:
async createTrade(userId: string, dto: CreateTradeDto) {
  return this.prisma.shadowTrade.create({
    data: {
      userId,  // Use directly
      symbol: dto.symbol,
      action: dto.action,
      price: dto.price
    }
  });
}
```

---

## Best Practices

### 1. Be Specific

❌ Bad: "Create a service"
✅ Good: "Create a Nest.js service for portfolio management with Prisma integration, following the pattern in src/modules/chat/chat.service.ts"

### 2. Provide Context

Always include:
- Existing patterns (e.g., "Follow src/modules/chat structure")
- Tech stack (e.g., "Using Nest.js 10.x + Prisma 5.x")
- Requirements (e.g., "Must include validation, tests, API docs")

### 3. Iterate

Don't expect perfect code on first try. Refine:
```
Prompt 1: "Create a basic service"
Prompt 2: "Add error handling"
Prompt 3: "Add tests"
Prompt 4: "Optimize this query"
```

---

**문서 끝**

_"Between the zeros and ones"_
