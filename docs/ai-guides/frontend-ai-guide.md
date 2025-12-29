# Frontend Development AI Guide

> **Version**: 3.0
> **Last Updated**: December 26, 2025
> **Author**: Sam
> **Target Audience**: Frontend Developers, Claude Code

---

## 1. AI-Powered Frontend Development

### 1.1 Development Tools

```typescript
interface FrontendTools {
  claudeCode: {
    purpose: string[];
    useCases: string[];
  };
  figmaMCP: {
    purpose: string[];
    useCases: string[];
  };
  githubCopilot: {
    purpose: string[];
    useCases: string[];
  };
}

const tools: FrontendTools = {
  claudeCode: {
    purpose: ["React components", "Hooks", "State management", "Architecture"],
    useCases: ["Component creation", "SSE streaming", "Form validation", "Chart integration"]
  },
  figmaMCP: {
    purpose: ["UI/UX design"],
    useCases: ["Design system creation", "Component design", "Prototype review"]
  },
  githubCopilot: {
    purpose: ["Real-time code suggestions"],
    useCases: ["Inline completion", "Refactoring", "Test generation"]
  }
};
```

### 1.2 Development Workflow

```mermaid
graph LR
    A[Design in Figma] --> B[Generate Component]
    B --> C[Add State Management]
    C --> D[Integrate API]
    D --> E[Test & Review]
```

---

## 2. Common Development Tasks

### 2.1 Creating a React Component

**Component Requirements**:
```typescript
interface ComponentSpec {
  name: string;
  props: {
    symbol: string;
    action: "buy" | "sell";
    entryPrice: number;
    currentPrice: number;
  };
  styling: {
    framework: "Tailwind CSS";
    components: "shadcn/ui";
  };
  features: {
    responsive: boolean;
    calculations: string[];
    conditionalStyling: boolean;
  };
}
```

**Component Architecture**:
```mermaid
graph TB
    A[PortfolioCard] --> B[Card Container]
    B --> C[Symbol & Action]
    B --> D[Price Info]
    B --> E[P&L Display]
    E --> F{Profit/Loss}
    F -->|Profit| G[Green Styling]
    F -->|Loss| H[Red Styling]
```

**Prompt Template**:
```
Create a React component for Shadow Portfolio card with the following:

Requirements:
- Show coin symbol, action (buy/sell), price, current P&L
- Use shadcn/ui Card component
- Use Tailwind CSS for styling
- Props: { symbol: string, action: 'buy' | 'sell', entryPrice: number, currentPrice: number }
- Calculate P&L automatically
- Green for profit, red for loss
- Mobile responsive

Example:
┌─────────────────────────────┐
│ BTC                         │
│ Buy @ $43,250               │
│ Current: $44,100            │
│ P&L: +$850 (+1.96%)         │
└─────────────────────────────┘
```

**Expected Implementation**:
```tsx
import { Card, CardContent } from '@/components/ui/card';
import { cn } from '@/lib/utils';

interface PortfolioCardProps {
  symbol: string;
  action: 'buy' | 'sell';
  entryPrice: number;
  currentPrice: number;
}

export function PortfolioCard({ symbol, action, entryPrice, currentPrice }: PortfolioCardProps) {
  const pnl = currentPrice - entryPrice;
  const pnlPercent = ((pnl / entryPrice) * 100).toFixed(2);
  const isProfit = pnl > 0;

  return (
    <Card>
      <CardContent className="p-4">
        <div className="flex justify-between items-start">
          <div>
            <h3 className="text-lg font-bold">{symbol}</h3>
            <p className="text-sm text-gray-600">
              {action === 'buy' ? 'Buy' : 'Sell'} @ ${entryPrice.toLocaleString()}
            </p>
            <p className="text-sm text-gray-600">
              Current: ${currentPrice.toLocaleString()}
            </p>
          </div>
        </div>

        <div className={cn(
          'mt-4 flex justify-between items-center',
          isProfit ? 'text-green-600' : 'text-red-600'
        )}>
          <span className="font-bold">
            P&L: {isProfit ? '+' : ''}${pnl.toFixed(2)} ({isProfit ? '+' : ''}{pnlPercent}%)
          </span>
        </div>
      </CardContent>
    </Card>
  );
}
```

---

### 2.2 Using TanStack Query

**Data Fetching Architecture**:
```typescript
interface QueryHookSpec {
  hookName: string;
  endpoint: string;
  method: "GET" | "POST" | "PUT" | "DELETE";
  refetchInterval?: number;
  caching: {
    staleTime: number;
    cacheTime: number;
  };
  mutations: {
    enabled: boolean;
    invalidateQueries: string[];
  };
}
```

**Query Flow**:
```mermaid
graph LR
    A[Component Mount] --> B[useQuery]
    B --> C{Cache Hit?}
    C -->|Yes| D[Return Cached]
    C -->|No| E[Fetch from API]
    E --> F[Update Cache]
    F --> D
```

**Prompt Template**:
```
Create a custom React hook for fetching shadow trades using TanStack Query:

Requirements:
- Hook name: useShadowTrades
- API endpoint: GET /api/shadow-trades
- Use axios
- Automatic refetch every 60 seconds
- Return trades, isLoading, error
- Include mutation hook for creating trades (useMutation)

Example usage:
const { trades, isLoading } = useShadowTrades();
const { mutate } = useCreateShadowTrade();
```

**Expected Implementation**:
```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import axios from 'axios';

interface ShadowTrade {
  id: string;
  symbol: string;
  action: 'buy' | 'sell';
  price: number;
  createdAt: string;
}

export function useShadowTrades() {
  return useQuery({
    queryKey: ['shadow-trades'],
    queryFn: async () => {
      const { data } = await axios.get<ShadowTrade[]>('/api/shadow-trades');
      return data;
    },
    refetchInterval: 60 * 1000  // 60 seconds
  });
}

export function useCreateShadowTrade() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (trade: Omit<ShadowTrade, 'id' | 'createdAt'>) => {
      const { data } = await axios.post('/api/shadow-trades', trade);
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['shadow-trades'] });
    }
  });
}
```

---

### 2.3 Building SSE Chat Component

**SSE Streaming Requirements**:
```typescript
interface StreamingChatSpec {
  protocol: "Server-Sent Events";
  api: "EventSource";
  stateManagement: {
    streaming: "Zustand";
    messages: "TanStack Query";
  };
  features: {
    tokenByToken: boolean;
    typingIndicator: boolean;
    autoScroll: boolean;
    errorHandling: boolean;
    reconnection: {
      enabled: true;
      strategy: "Exponential backoff";
      maxRetries: 3;
      timeout: 30000; // 30 seconds
    };
  };
}
```

**Streaming Flow with Reconnection**:
```mermaid
graph TB
    A[User Sends Message] --> B[Open EventSource]
    B --> C{Receive Event}
    C -->|token| D[Append to UI]
    C -->|done| E[Close Connection]
    C -->|error| F{Retry Count < 3?}
    F -->|Yes| G[Wait Exponentially]
    G --> B
    F -->|No| H[Show Error]
    D --> C
```

**Prompt Template**:
```
Create a React chat component with Server-Sent Events (SSE) streaming:

Requirements:
- Use EventSource for SSE
- Display streaming message token by token
- Show typing indicator while streaming
- Auto-scroll to bottom
- Handle connection errors
- Use Zustand for temporary streaming state
- Implement reconnection logic:
  * Exponential backoff: 1s → 2s → 4s
  * Maximum 3 retry attempts
  * 30-second timeout
  * User-friendly error message after max retries

UI:
┌──────────────────────────────┐
│ [User Message]               │
│ [Assistant Message...]▋      │  ← Streaming
│ [Send Input]                 │
└──────────────────────────────┘
```

**Expected Implementation**:
```tsx
import { useState, useRef, useEffect } from 'react';
import { useStreamingStore } from '@/store/streaming.store';

export function ChatWindow({ chatId }: { chatId: string }) {
  const { streamingMessage, setStreamingMessage, clearStreamingMessage } = useStreamingStore();
  const [isStreaming, setIsStreaming] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const eventSourceRef = useRef<EventSource | null>(null);

  // Reconnection state
  const reconnectAttemptsRef = useRef(0);
  const maxReconnectAttempts = 3;
  const reconnectDelayRef = useRef(1000); // Start with 1s
  const timeoutIdRef = useRef<number | null>(null);

  const connectSSE = (content: string) => {
    clearStreamingMessage();
    setIsStreaming(true);
    setError(null);

    const url = `/api/chats/${chatId}/messages?message=${encodeURIComponent(content)}`;
    eventSourceRef.current = new EventSource(url);

    // Timeout for 30 seconds
    timeoutIdRef.current = window.setTimeout(() => {
      eventSourceRef.current?.close();
      handleReconnect(content);
    }, 30000);

    eventSourceRef.current.onmessage = (event) => {
      const data = JSON.parse(event.data);

      if (data.type === 'token') {
        setStreamingMessage((prev) => prev + data.content);
        reconnectAttemptsRef.current = 0; // Reset on successful message
        reconnectDelayRef.current = 1000;
      } else if (data.type === 'done') {
        setIsStreaming(false);
        clearStreamingMessage();
        if (timeoutIdRef.current) clearTimeout(timeoutIdRef.current);
        eventSourceRef.current?.close();
      }
    };

    eventSourceRef.current.onerror = () => {
      if (timeoutIdRef.current) clearTimeout(timeoutIdRef.current);
      eventSourceRef.current?.close();
      handleReconnect(content);
    };
  };

  const handleReconnect = (content: string) => {
    if (reconnectAttemptsRef.current >= maxReconnectAttempts) {
      setIsStreaming(false);
      setError('연결에 실패했습니다. 다시 시도해주세요.');
      clearStreamingMessage();
      return;
    }

    reconnectAttemptsRef.current += 1;

    // Exponential backoff: 1s → 2s → 4s
    setTimeout(() => {
      connectSSE(content);
    }, reconnectDelayRef.current);

    reconnectDelayRef.current *= 2;
  };

  const sendMessage = (content: string) => {
    reconnectAttemptsRef.current = 0;
    reconnectDelayRef.current = 1000;
    connectSSE(content);
  };

  // Auto-scroll
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [streamingMessage]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (timeoutIdRef.current) clearTimeout(timeoutIdRef.current);
      eventSourceRef.current?.close();
    };
  }, []);

  return (
    <div className="flex flex-col h-screen">
      <div className="flex-1 overflow-y-auto p-4">
        {/* Static messages */}
        {/* ... */}

        {/* Error message */}
        {error && (
          <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
            {error}
          </div>
        )}

        {/* Streaming message */}
        {streamingMessage && (
          <div className="bg-gray-100 rounded-lg p-4">
            <p>{streamingMessage}<span className="animate-pulse">▋</span></p>
          </div>
        )}

        <div ref={messagesEndRef} />
      </div>

      <MessageInput onSend={sendMessage} disabled={isStreaming} />
    </div>
  );
}
```

---

### 2.4 Creating a Chart Component

**Chart Requirements**:
```typescript
interface ChartSpec {
  library: "Recharts";
  chartType: "Line";
  data: {
    xAxis: "date";
    yAxis: "value";
    series: ["portfolio", "benchmark"];
  };
  features: {
    tooltip: boolean;
    legend: boolean;
    responsive: boolean;
    valueFormatting: boolean;
  };
}
```

**Chart Architecture**:
```mermaid
graph TB
    A[PerformanceChart] --> B[ResponsiveContainer]
    B --> C[LineChart]
    C --> D[Portfolio Line]
    C --> E[Benchmark Line]
    C --> F[Axes & Grid]
    C --> G[Tooltip]
```

**Prompt Template**:
```
Create a performance chart using Recharts:

Requirements:
- Show portfolio value over time
- Compare with benchmark (BTC)
- Two lines: Portfolio (blue), BTC (orange)
- X-axis: dates, Y-axis: value ($)
- Tooltip showing values
- Responsive (mobile-friendly)
- Data: { date: string, portfolio: number, benchmark: number }[]
```

**Expected Implementation**:
```tsx
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';

interface PerformanceData {
  date: string;
  portfolio: number;
  benchmark: number;
}

interface PerformanceChartProps {
  data: PerformanceData[];
}

export function PerformanceChart({ data }: PerformanceChartProps) {
  return (
    <ResponsiveContainer width="100%" height={400}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis
          dataKey="date"
          tickFormatter={(date) => new Date(date).toLocaleDateString('en-US', { month: 'short', day: 'numeric' })}
        />
        <YAxis
          tickFormatter={(value) => `$${value.toLocaleString()}`}
        />
        <Tooltip
          formatter={(value: number) => `$${value.toLocaleString()}`}
          labelFormatter={(date) => new Date(date).toLocaleDateString()}
        />
        <Legend />
        <Line
          type="monotone"
          dataKey="portfolio"
          stroke="#3b82f6"
          strokeWidth={2}
          name="Your Portfolio"
        />
        <Line
          type="monotone"
          dataKey="benchmark"
          stroke="#f59e0b"
          strokeWidth={2}
          name="BTC Benchmark"
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

---

### 2.5 Form Validation with React Hook Form

**Form Validation Requirements**:
```typescript
interface FormSpec {
  library: "react-hook-form";
  validation: {
    schema: "Zod";
    rules: {
      symbol: ["required", "enum"];
      action: ["required", "enum"];
      price: ["required", "positive"];
    };
  };
  features: {
    errorMessages: boolean;
    submitHandling: boolean;
    resetOnSuccess: boolean;
    disableWhileSubmitting: boolean;
  };
}
```

**Form Validation Flow**:
```mermaid
graph LR
    A[User Input] --> B[Zod Validation]
    B --> C{Valid?}
    C -->|No| D[Show Errors]
    C -->|Yes| E[Submit]
    E --> F[Reset Form]
```

**Prompt Template**:
```
Create a form for creating shadow trades:

Requirements:
- Use react-hook-form
- Validation: symbol (required, one of BTC/ETH/SOL/BNB/DOGE/XRP), action (buy/sell), price (positive number)
- Show error messages
- Submit handler
- Disable submit while submitting
- Reset form after success
```

**Expected Implementation**:
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  symbol: z.enum(['BTC', 'ETH', 'SOL', 'BNB', 'DOGE', 'XRP']),
  action: z.enum(['buy', 'sell']),
  price: z.number().positive('Price must be positive')
});

type FormData = z.infer<typeof schema>;

export function CreateTradeForm() {
  const { register, handleSubmit, formState: { errors, isSubmitting }, reset } = useForm<FormData>({
    resolver: zodResolver(schema)
  });

  const onSubmit = async (data: FormData) => {
    await axios.post('/api/shadow-trades', data);
    reset();
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label>Symbol</label>
        <select {...register('symbol')}>
          <option value="">Select...</option>
          <option value="BTC">Bitcoin</option>
          <option value="ETH">Ethereum</option>
          {/* ... */}
        </select>
        {errors.symbol && <p className="text-red-500 text-sm">{errors.symbol.message}</p>}
      </div>

      <div>
        <label>Action</label>
        <select {...register('action')}>
          <option value="">Select...</option>
          <option value="buy">Buy</option>
          <option value="sell">Sell</option>
        </select>
        {errors.action && <p className="text-red-500 text-sm">{errors.action.message}</p>}
      </div>

      <div>
        <label>Price</label>
        <input
          type="number"
          step="0.01"
          {...register('price', { valueAsNumber: true })}
        />
        {errors.price && <p className="text-red-500 text-sm">{errors.price.message}</p>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating...' : 'Create Trade'}
      </button>
    </form>
  );
}
```

---

## 3. AI-Assisted Code Review

### 3.1 Review Checklist

```typescript
interface FrontendReviewCriteria {
  performance: {
    checks: [
      "Unnecessary re-renders",
      "Missing memoization (useMemo, useCallback)",
      "Large component bundles"
    ];
  };
  accessibility: {
    checks: [
      "ARIA labels",
      "Keyboard navigation",
      "Screen reader support"
    ];
  };
  ux: {
    checks: [
      "Loading states",
      "Error states",
      "Empty states"
    ];
  };
  codeQuality: {
    checks: [
      "Component composition",
      "Prop drilling",
      "Naming conventions"
    ];
  };
}
```

### 3.2 Code Review Prompt

```
Review this React component for:

1. Performance:
   - Unnecessary re-renders
   - Missing memoization (useMemo, useCallback)
   - Large component bundles

2. Accessibility:
   - ARIA labels
   - Keyboard navigation
   - Screen reader support

3. UX:
   - Loading states
   - Error states
   - Empty states

4. Code Quality:
   - Component composition
   - Prop drilling
   - Naming conventions

Code:
[Paste component code]
```

---

## 4. Best Practices

### 4.1 Component Composition

**Component Design Patterns**:
```typescript
interface CompositionPattern {
  bad: {
    approach: "Giant monolithic component";
    issues: ["Hard to test", "Poor reusability", "Difficult to maintain"];
  };
  good: {
    approach: "Small, focused components";
    benefits: ["Easy to test", "Highly reusable", "Simple to understand"];
  };
}
```

**Refactoring Flow**:
```mermaid
graph TB
    A[Large Component] --> B[Identify Responsibilities]
    B --> C[Extract ChatHeader]
    B --> D[Extract MessageList]
    B --> E[Extract MessageInput]
    C --> F[Composed Chat]
    D --> F
    E --> F
```

**Prompt for Refactoring**:
```
Refactor this component into smaller, reusable parts:
- Extract ChatHeader
- Extract MessageList
- Extract MessageInput

[Paste large component]
```

---

### 4.2 State Management

**State Management Strategy**:
```typescript
interface StateStrategy {
  zustand: {
    purpose: "UI state";
    useCases: [
      "Modal visibility",
      "Sidebar state",
      "Theme preferences"
    ];
  };
  tanstackQuery: {
    purpose: "Server state";
    useCases: [
      "API data fetching",
      "Caching",
      "Real-time updates"
    ];
  };
}
```

**Zustand Store Prompt**:
```
Create a Zustand store for UI state:
- sidebarOpen (boolean)
- toggleSidebar
- currentModal ('none' | 'settings' | 'portfolio')
- openModal, closeModal
```

**TanStack Query Prompt**:
```
Create TanStack Query hooks for:
- useChats (list chats)
- useMessages (get messages)
- useCreateChat (mutation)
```

---

### 4.3 TypeScript Types

**Type Generation Requirements**:
```typescript
interface TypeGenerationSpec {
  source: "API Response";
  output: "TypeScript Interfaces";
  features: {
    nested: boolean;
    optional: boolean;
    unions: boolean;
  };
}
```

**Type Generation Prompt**:
```
Generate TypeScript types from this API response:

{
  "id": "chat-123",
  "title": "Bitcoin 투자 상담",
  "createdAt": "2024-01-15T10:30:00Z",
  "messages": [
    {
      "id": "msg-123",
      "role": "user",
      "content": "비트코인 지금 어때?",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ]
}
```

**Expected Types**:
```typescript
interface Chat {
  id: string;
  title: string;
  createdAt: string;
  messages: Message[];
}

interface Message {
  id: string;
  role: "user" | "assistant";
  content: string;
  createdAt: string;
}
```

---

## 5. Performance Optimization

### 5.1 Optimization Checklist

```typescript
interface PerformanceOptimization {
  rendering: {
    techniques: [
      "useMemo for expensive calculations",
      "useCallback for function props",
      "React.memo for pure components"
    ];
  };
  bundleSize: {
    techniques: [
      "Code splitting with lazy()",
      "Dynamic imports",
      "Tree shaking"
    ];
  };
  dataFetching: {
    techniques: [
      "TanStack Query caching",
      "Prefetching",
      "Background refetch"
    ];
  };
}
```

### 5.2 Optimization Flow

```mermaid
graph LR
    A[Identify Bottleneck] --> B{Type?}
    B -->|Rendering| C[Add Memoization]
    B -->|Bundle| D[Code Splitting]
    B -->|Data| E[Query Optimization]
    C --> F[Measure Impact]
    D --> F
    E --> F
```

---

## 6. Feature-Based Architecture & PWA Best Practices

### 6.1 Why Feature-Based Structure?

**선택 근거**:
- **확장성**: 각 기능이 자체 포함됨 (컴포넌트, 훅, 스토어, 타입)
- **코로케이션**: 관련 코드가 함께 위치해 파일 전환 최소화
- **팀 자율성**: 여러 개발자가 독립적으로 기능별 작업 가능
- **명확한 경계**: 기능 간 의존성 파악 쉬움, 결합도 낮음

**Directory Structure**:
```
src/
├── features/
│   ├── chat/
│   │   ├── components/
│   │   │   ├── ChatWindow.tsx
│   │   │   ├── MessageList.tsx
│   │   │   └── MessageInput.tsx
│   │   ├── hooks/
│   │   │   ├── useChat.ts
│   │   │   └── useSSE.ts
│   │   ├── store/
│   │   │   └── chat.store.ts
│   │   ├── types/
│   │   │   └── chat.types.ts
│   │   └── api/
│   │       └── chat.api.ts
│   ├── portfolio/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── store/
│   └── alerts/
│       ├── components/
│       ├── hooks/
│       └── store/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
└── App.tsx
```

### 6.2 SSE Reconnection Pattern

**선택 근거**:
- **안정성**: 네트워크 불안정 환경에서도 사용자 경험 유지
- **Exponential backoff**: 서버 부하 방지 (1s → 2s → 4s)
- **Max retries**: 무한 재시도 방지 (3회 제한)
- **30-second timeout**: 긴 응답 대기 방지

**Implementation Pattern**:
```typescript
// features/chat/hooks/useSSE.ts
export function useSSE() {
  const reconnectAttemptsRef = useRef(0);
  const maxReconnectAttempts = 3;
  const reconnectDelayRef = useRef(1000); // Start with 1s

  const handleReconnect = (content: string) => {
    if (reconnectAttemptsRef.current >= maxReconnectAttempts) {
      setError('연결에 실패했습니다.');
      return;
    }

    reconnectAttemptsRef.current += 1;
    setTimeout(() => {
      connectSSE(content);
    }, reconnectDelayRef.current);

    reconnectDelayRef.current *= 2; // Exponential backoff
  };
}
```

### 6.3 Deeplink Backend Validation

**선택 근거**:
- **보안**: Frontend에서 deeplink context 생성 시 조작 가능성
- **데이터 일관성**: Backend DB에 저장된 context만 허용
- **TTL 관리**: 24시간 후 자동 만료로 무효 링크 방지

**Flow**:
```mermaid
graph LR
    A[Backend: Create Context] --> B[Store in DB with TTL]
    B --> C[Generate Deeplink with contextId]
    C --> D[User Opens Link]
    D --> E[Frontend: Extract contextId]
    E --> F[Backend: Validate contextId]
    F --> G{Valid?}
    G -->|Yes| H[Load Context]
    G -->|No| I[Show Error]
```

**Implementation**:
```typescript
// features/alerts/hooks/useDeeplink.ts
export function useDeeplink() {
  const navigate = useNavigate();

  useEffect(() => {
    const params = new URLSearchParams(window.location.search);
    const contextId = params.get('contextId');

    if (contextId) {
      // Backend validation
      axios.get(`/api/deeplink/${contextId}`)
        .then(({ data }) => {
          navigate(`/chat/${data.chatId}`, {
            state: { initialMessage: data.message }
          });
        })
        .catch(() => {
          // Invalid or expired contextId
          navigate('/');
        });
    }
  }, []);
}
```

### 6.4 PWA Push Notifications

**선택 근거**:
- **실시간 알림**: 시장 변동성 즉시 전달 (15분 크론과 연동)
- **재방문 유도**: 푸시를 통한 사용자 재참여
- **Native 느낌**: 모바일 앱처럼 동작

**Service Worker Registration**:
```typescript
// src/service-worker.ts
self.addEventListener('push', (event) => {
  const data = event.data?.json();

  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: '/icon-192.png',
      badge: '/badge-72.png',
      data: {
        url: data.url // Deeplink URL
      }
    })
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(
    clients.openWindow(event.notification.data.url)
  );
});
```

**Subscription Flow**:
```typescript
// features/alerts/hooks/usePushNotification.ts
export function usePushNotification() {
  const subscribe = async () => {
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
    });

    // Send subscription to backend
    await axios.post('/api/push/subscribe', {
      subscription
    });
  };

  return { subscribe };
}
```

---

**Document Version**: 3.0
**Last Updated**: December 26, 2025
**Stack**: React 18.3 + Vite 5 + TypeScript
**Architecture**: Feature-Based Structure
**Maintainer**: Sam (dev@5010.tech)

_"Between the zeros and ones"_
