# Sage.ai API Documentation Strategy

```
Document Info
├── Version: 1.0
├── Last Updated: 2025-12-19
├── Author: Sam
├── Status: Production Ready
└── Audience: Backend Developers, API Consumers
```

---

## 1. Documentation Philosophy

```
"API First, Documentation Always"
```

### 1.1 Core Principles

| Principle | Description |
|-----------|-------------|
| Self-Documenting Code | Code itself becomes documentation |
| Auto-Generated Docs | Swagger/OpenAPI automatic generation |
| Living Documentation | Auto-updates when code changes |
| Developer-Friendly | Example-driven, quick start guides |

---

## 2. Tools & Technologies

| Tool | Purpose | URL |
|------|---------|-----|
| Swagger UI | Interactive API docs | `/api/docs` |
| OpenAPI 3.0 | API spec definition | `/api/docs-json` |
| Nest.js Swagger | Auto-generation | `@nestjs/swagger` |
| Redoc | Static docs (optional) | `/api/redoc` |

---

## 3. OpenAPI Specification

### 3.1 Nest.js Setup

```typescript
// apps/backend/src/main.ts
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Swagger configuration
  const config = new DocumentBuilder()
    .setTitle('Sage.ai API')
    .setDescription('AI Investment Mentor Platform API')
    .setVersion('1.0')
    .addTag('auth', 'Authentication endpoints')
    .addTag('chat', 'Chat & messaging endpoints')
    .addTag('market', 'Market data endpoints')
    .addTag('portfolio', 'Shadow portfolio endpoints')
    .addTag('notifications', 'Notification endpoints')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api/docs', app, document);

  await app.listen(3000);
}
```

### 3.2 Controller Documentation

```typescript
// modules/chat/chat.controller.ts
import { ApiTags, ApiOperation, ApiResponse, ApiParam, ApiQuery } from '@nestjs/swagger';

@ApiTags('chat')
@Controller('api/chats')
export class ChatController {
  @Post()
  @ApiOperation({
    summary: 'Create a new chat',
    description: 'Creates a new chat session for the authenticated user'
  })
  @ApiResponse({
    status: 201,
    description: 'Chat created successfully',
    type: ChatDto
  })
  @ApiResponse({
    status: 401,
    description: 'Unauthorized'
  })
  async createChat(@Req() req): Promise<ChatDto> {
    return this.chatService.create(req.user.id);
  }

  @Get(':id/messages')
  @ApiOperation({
    summary: 'Get chat messages',
    description: 'Retrieves all messages in a chat with pagination'
  })
  @ApiParam({
    name: 'id',
    description: 'Chat ID',
    type: 'string',
    example: '550e8400-e29b-41d4-a716-446655440000'
  })
  @ApiQuery({
    name: 'limit',
    required: false,
    type: 'number',
    description: 'Number of messages to retrieve',
    example: 20
  })
  @ApiQuery({
    name: 'offset',
    required: false,
    type: 'number',
    description: 'Offset for pagination',
    example: 0
  })
  @ApiResponse({
    status: 200,
    description: 'Messages retrieved successfully',
    type: [MessageDto]
  })
  async getMessages(
    @Param('id') id: string,
    @Query('limit') limit = 20,
    @Query('offset') offset = 0
  ): Promise<MessageDto[]> {
    return this.chatService.getMessages(id, limit, offset);
  }
}
```

### 3.3 DTO Documentation

```typescript
// modules/chat/dto/message.dto.ts
import { ApiProperty } from '@nestjs/swagger';

export class MessageDto {
  @ApiProperty({
    description: 'Message ID',
    example: '550e8400-e29b-41d4-a716-446655440000'
  })
  id: string;

  @ApiProperty({
    description: 'Chat ID this message belongs to',
    example: '660e8400-e29b-41d4-a716-446655440000'
  })
  chatId: string;

  @ApiProperty({
    description: 'Message role',
    enum: ['user', 'assistant'],
    example: 'user'
  })
  role: 'user' | 'assistant';

  @ApiProperty({
    description: 'Message content',
    example: 'What do you think about Bitcoin?'
  })
  content: string;

  @ApiProperty({
    description: 'AI trading signal (if any)',
    required: false,
    example: { action: 'buy', symbol: 'BTC', confidence: 0.8 }
  })
  signal?: AISignal;

  @ApiProperty({
    description: 'Message creation timestamp',
    example: '2024-01-15T10:30:00Z'
  })
  createdAt: Date;
}

export class AISignal {
  @ApiProperty({
    description: 'Trading action',
    enum: ['buy', 'sell'],
    example: 'buy'
  })
  action: 'buy' | 'sell';

  @ApiProperty({
    description: 'Coin symbol',
    enum: ['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP'],
    example: 'BTC'
  })
  symbol: string;

  @ApiProperty({
    description: 'AI confidence level (0-1)',
    example: 0.8,
    minimum: 0,
    maximum: 1
  })
  confidence: number;
}
```

---

## 4. API Endpoints Reference

### 4.1 Authentication

#### 4.1.1 POST /api/auth/google

**Description**: Authenticate with Google OAuth

**Request Body**:
```json
{
  "code": "google_oauth_code",
  "redirectUri": "https://app.sage.ai/auth/callback"
}
```

**Response**: `200 OK`
```json
{
  "user": {
    "id": "user-123",
    "email": "user@example.com",
    "name": "John Doe",
    "image": "https://...",
    "tier": "free"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 4.2 Chat

#### 4.2.1 POST /api/chats

**Description**: Create a new chat

**Request Headers**:
```
Authorization: Bearer <token>
```

**Response**: `201 Created`
```json
{
  "id": "chat-123",
  "userId": "user-123",
  "title": "New Chat",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

#### 4.2.2 GET /api/chats

**Description**: List user's chats

**Request Headers**:
```
Authorization: Bearer <token>
```

**Response**: `200 OK`
```json
[
  {
    "id": "chat-123",
    "title": "Bitcoin Investment Consultation",
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T11:00:00Z",
    "messageCount": 15
  }
]
```

#### 4.2.3 POST /api/chats/:id/messages

**Description**: Send a message (SSE response)

**Request Headers**:
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body**:
```json
{
  "content": "What do you think about Bitcoin now?"
}
```

**Response**: `200 OK` (SSE Stream)
```
event: token
data: {"content":"Well"}

event: token
data: {"content":", "}

event: token
data: {"content":"Bitcoin"}

...

event: done
data: {"messageId":"msg-123"}
```

#### 4.2.4 GET /api/chats/:id/messages

**Description**: Get chat messages

**Query Parameters**:
- `limit` (optional): Number of messages (default: 20)
- `offset` (optional): Offset for pagination (default: 0)

**Response**: `200 OK`
```json
[
  {
    "id": "msg-123",
    "chatId": "chat-123",
    "role": "user",
    "content": "What do you think about Bitcoin now?",
    "createdAt": "2024-01-15T10:30:00Z"
  },
  {
    "id": "msg-124",
    "chatId": "chat-123",
    "role": "assistant",
    "content": "Well, Bitcoin is currently trading at $43,250...",
    "signal": {
      "action": "buy",
      "symbol": "BTC",
      "confidence": 0.75
    },
    "createdAt": "2024-01-15T10:30:05Z"
  }
]
```

---

### 4.3 Market

#### 4.3.1 GET /api/market/prices

**Description**: Get current prices for all coins

**Response**: `200 OK`
```json
[
  {
    "symbol": "BTC",
    "name": "Bitcoin",
    "price": 43250.50,
    "change24h": -5.2,
    "marketCap": 846000000000,
    "volume24h": 28500000000
  },
  {
    "symbol": "ETH",
    "name": "Ethereum",
    "price": 2280.75,
    "change24h": -3.8,
    "marketCap": 274000000000,
    "volume24h": 15200000000
  }
]
```

#### 4.3.2 GET /api/market/fear-greed

**Description**: Get Fear & Greed Index

**Response**: `200 OK`
```json
{
  "value": 25,
  "classification": "Extreme Fear",
  "timestamp": "2024-01-15T10:00:00Z",
  "previousValue": 30
}
```

#### 4.3.3 GET /api/market/history/:symbol

**Description**: Get price history for a coin

**Path Parameters**:
- `symbol`: Coin symbol (BTC, ETH, etc.)

**Query Parameters**:
- `period`: Time period (24h, 7d, 30d, 1y)

**Response**: `200 OK`
```json
{
  "symbol": "BTC",
  "period": "24h",
  "data": [
    {
      "timestamp": "2024-01-15T00:00:00Z",
      "price": 45200.00,
      "volume": 1200000000
    },
    {
      "timestamp": "2024-01-15T01:00:00Z",
      "price": 45100.50,
      "volume": 1180000000
    }
  ]
}
```

---

### 4.4 Portfolio

#### 4.4.1 POST /api/shadow-trades

**Description**: Create a shadow trade

**Request Headers**:
```
Authorization: Bearer <token>
```

**Request Body**:
```json
{
  "symbol": "BTC",
  "action": "buy",
  "price": 43250.50,
  "quantity": 1.0,
  "messageId": "msg-124"
}
```

**Response**: `201 Created`
```json
{
  "id": "trade-123",
  "userId": "user-123",
  "symbol": "BTC",
  "action": "buy",
  "price": 43250.50,
  "quantity": 1.0,
  "createdAt": "2024-01-15T10:30:00Z"
}
```

#### 4.4.2 GET /api/shadow-trades

**Description**: List user's shadow trades

**Response**: `200 OK`
```json
[
  {
    "id": "trade-123",
    "symbol": "BTC",
    "action": "buy",
    "price": 43250.50,
    "quantity": 1.0,
    "currentPrice": 44100.00,
    "profitLoss": 849.50,
    "profitLossPercent": 1.96,
    "createdAt": "2024-01-15T10:30:00Z"
  }
]
```

#### 4.4.3 GET /api/shadow-trades/performance

**Description**: Calculate portfolio performance

**Response**: `200 OK`
```json
{
  "totalReturn": 12.5,
  "vsBasemark": 3.2,
  "totalTrades": 15,
  "winRate": 66.7,
  "history": [
    {
      "date": "2024-01-01",
      "value": 10000,
      "basemark": 10000
    },
    {
      "date": "2024-01-15",
      "value": 11250,
      "basemark": 10920
    }
  ]
}
```

---

### 4.5 Notifications

#### 4.5.1 POST /api/push/subscribe

**Description**: Subscribe to push notifications

**Request Body**:
```json
{
  "endpoint": "https://fcm.googleapis.com/...",
  "keys": {
    "auth": "...",
    "p256dh": "..."
  }
}
```

**Response**: `201 Created`
```json
{
  "success": true,
  "subscriptionId": "sub-123"
}
```

#### 4.5.2 GET /api/notifications

**Description**: Get notification history

**Query Parameters**:
- `limit` (optional): Number of notifications (default: 20)
- `offset` (optional): Offset for pagination (default: 0)

**Response**: `200 OK`
```json
[
  {
    "id": "notif-123",
    "type": "market_alert",
    "title": "BTC Alert",
    "message": "Bitcoin dropped -5.2%",
    "data": {
      "symbol": "BTC",
      "change": -5.2
    },
    "read": false,
    "sentAt": "2024-01-15T10:30:00Z"
  }
]
```

---

## 5. Error Handling

### 5.1 Error Response Format

All API errors follow this format:

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "details": [
    {
      "field": "symbol",
      "message": "symbol must be one of: BTC, ETH, SOL, BNB, DOGE, XRP"
    }
  ]
}
```

### 5.2 HTTP Status Codes

| Code | Meaning | Example |
|------|---------|---------|
| 200 | OK | Successful GET request |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing or invalid token |
| 403 | Forbidden | User tier doesn't allow this action |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### 5.3 Error Examples

**400 Bad Request**:
```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "details": [
    {
      "field": "symbol",
      "message": "symbol must be one of: BTC, ETH, SOL, BNB, DOGE, XRP"
    }
  ]
}
```

**401 Unauthorized**:
```json
{
  "statusCode": 401,
  "message": "Unauthorized",
  "error": "Unauthorized"
}
```

**403 Forbidden (Tier Limit)**:
```json
{
  "statusCode": 403,
  "message": "Feature 'unlimited_chats' requires pro tier",
  "error": "Forbidden"
}
```

**429 Too Many Requests**:
```json
{
  "statusCode": 429,
  "message": "Rate limit exceeded: 10 requests per minute",
  "error": "Too Many Requests",
  "retryAfter": 45
}
```

---

## 6. Authentication

### 6.1 Bearer Token

All protected endpoints require a Bearer token:

```http
GET /api/chats
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 6.2 Token Expiration

| Token Type | Duration |
|------------|----------|
| Access Token | 1 hour |
| Refresh Token | 30 days |

### 6.3 Refresh Token Flow

```http
POST /api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "refresh_token_here"
}
```

**Response**:
```json
{
  "accessToken": "new_access_token",
  "refreshToken": "new_refresh_token"
}
```

---

## 7. Rate Limiting

### 7.1 Limits by Tier

| Tier | Chat Messages | Market Data | General API |
|------|--------------|-------------|-------------|
| Free | 10/min | 60/min | 100/min |
| Pro | 30/min | 120/min | 300/min |
| Premium | Unlimited | Unlimited | 1000/min |

### 7.2 Rate Limit Headers

```http
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1642248000
```

---

## 8. Pagination

### 8.1 Query Parameters

```
?limit=20&offset=0
```

### 8.2 Response Format

```json
{
  "data": [...],
  "pagination": {
    "limit": 20,
    "offset": 0,
    "total": 150,
    "hasMore": true
  }
}
```

---

## 9. Webhooks (Future)

### 9.1 Discord Webhook Format

```json
{
  "content": "**BTC Alert**",
  "embeds": [
    {
      "title": "Bitcoin -5.2% Drop",
      "description": "Current price: $43,250",
      "color": 16711680,
      "timestamp": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

## 10. Testing API

### 10.1 Using cURL

```bash
# Login
curl -X POST https://api.sage.ai/api/auth/google \
  -H "Content-Type: application/json" \
  -d '{"code":"oauth_code","redirectUri":"..."}'

# Create chat
curl -X POST https://api.sage.ai/api/chats \
  -H "Authorization: Bearer YOUR_TOKEN"

# Send message
curl -X POST https://api.sage.ai/api/chats/chat-123/messages \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"What do you think about Bitcoin now?"}'
```

### 10.2 Using Postman

**Collection**: [Download Sage.ai Postman Collection](link)

### 10.3 Using JavaScript

```javascript
// Login
const loginResponse = await fetch('https://api.sage.ai/api/auth/google', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ code: 'oauth_code', redirectUri: '...' })
});

const { token } = await loginResponse.json();

// Send message with SSE
const eventSource = new EventSource(
  `https://api.sage.ai/api/chats/chat-123/messages?message=What%20about%20Bitcoin?`,
  { headers: { Authorization: `Bearer ${token}` } }
);

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.type === 'token') {
    console.log(data.content);
  }
};
```

---

## 11. Versioning Strategy

### 11.1 URL Versioning

```
https://api.sage.ai/api/v1/chats
https://api.sage.ai/api/v2/chats  (future)
```

### 11.2 Deprecation Policy

| Policy | Duration |
|--------|----------|
| Announcement | 3 months before deprecation |
| Sunset Header | `Sunset: Sat, 31 Dec 2024 23:59:59 GMT` |
| Minimum Support | 6 months after new version |

---

## 12. API Changelog

### 12.1 v1.0.0 (2026-03-15) - Initial Release

**Features**:
- Authentication (Google OAuth)
- Chat endpoints
- Market data endpoints
- Shadow portfolio endpoints
- Push notification endpoints

---

## 13. Best Practices for API Consumers

### 13.1 Always Handle Errors

```typescript
try {
  const response = await fetch('/api/chats', {
    headers: { Authorization: `Bearer ${token}` }
  });

  if (!response.ok) {
    const error = await response.json();
    console.error('API Error:', error.message);
  }

  const data = await response.json();
} catch (error) {
  console.error('Network Error:', error);
}
```

### 13.2 Implement Retry Logic

```typescript
async function fetchWithRetry(url, options, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.status === 429) {
        const retryAfter = response.headers.get('Retry-After');
        await sleep(retryAfter * 1000);
        continue;
      }
      return response;
    } catch (error) {
      if (i === retries - 1) throw error;
      await sleep(1000 * Math.pow(2, i)); // Exponential backoff
    }
  }
}
```

### 13.3 Use Pagination

```typescript
async function getAllChats() {
  let allChats = [];
  let offset = 0;
  const limit = 20;

  while (true) {
    const response = await fetch(`/api/chats?limit=${limit}&offset=${offset}`);
    const { data, pagination } = await response.json();

    allChats = [...allChats, ...data];

    if (!pagination.hasMore) break;
    offset += limit;
  }

  return allChats;
}
```

---

## 14. SDK (Future)

### 14.1 TypeScript SDK

```bash
npm install @sage-ai/sdk
```

```typescript
import { SageClient } from '@sage-ai/sdk';

const sage = new SageClient({
  apiKey: 'YOUR_API_KEY'
});

// Create chat
const chat = await sage.chats.create();

// Send message
const stream = await sage.chats.sendMessage(chat.id, {
  content: 'What do you think about Bitcoin now?'
});

for await (const token of stream) {
  process.stdout.write(token);
}
```

---

## Appendix A: Swagger UI Screenshot

[Coming Soon]

## Appendix B: Postman Collection

[Download Link]

## Appendix C: API Playground

https://api.sage.ai/api/docs

## Appendix D: Support

| Channel | Link |
|---------|------|
| Email | api@sage.ai |
| Discord | #api-support |
| GitHub Issues | https://github.com/sage-ai/sage/issues |

---

```
Document Footer
├── Last Updated: 2025-12-19
├── Version: 1.0
├── Status: Production Ready
└── Maintainer: Sam (dev@5010.tech)
```
