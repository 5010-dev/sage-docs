# Sage.ai Frontend Specification

```
Document Info
├── Version: 1.0
├── Last Updated: 2025-12-19
├── Author: Sam
├── Status: Production Ready
└── Audience: Frontend Developers
```

---

## 1. Technology Stack

### 1.1 Core Technologies

```typescript
interface TechnologyStack {
  core: {
    framework: "React 18.3";
    buildTool: "Vite 5.x";
    language: "TypeScript 5.x";
    routing: "React Router 6.x";
  };
  stateManagement: {
    clientState: "Zustand 4.x";
    serverState: "TanStack Query 5.x";
  };
  styling: {
    framework: "Tailwind CSS 3.x";
    components: "shadcn/ui";
    icons: "Lucide React";
  };
  pwa: {
    serviceWorker: "Vite PWA Plugin";
    notifications: "Web Push API";
  };
}
```

### 1.2 Stack Selection Rationale

| Component | Technology | Version | Reason |
|-----------|------------|---------|--------|
| Framework | React | 18.3 | Ecosystem, stability, Hooks |
| Build Tool | Vite | 5.x | Fast HMR, ESM native |
| Language | TypeScript | 5.x | Type safety |
| Routing | React Router | 6.x | SPA routing |
| Client State | Zustand | 4.x | Sidebar, modal, UI state |
| Server State | TanStack Query | 5.x | API data, caching, sync |
| CSS Framework | Tailwind CSS | 3.x | Rapid development |
| Component Library | shadcn/ui | - | Copy-paste components |
| Icons | Lucide React | - | Consistent icon set |

---

## 2. Architecture

### 2.1 Component Hierarchy

```
Application Architecture
│
├── App
│   ├── Layouts
│   │   ├── AuthLayout
│   │   │   ├── WhyBitcoinFallen
│   │   │   └── Sage.ai Landing
│   │   │
│   │   └── AppLayout
│   │       ├── Sidebar
│   │       ├── Header
│   │       └── Main Content
│   │
│   ├── Pages
│   │   ├── WhyBitcoinFallen
│   │   ├── Landing
│   │   ├── Chat
│   │   ├── Portfolio
│   │   ├── Notifications
│   │   └── Settings
│   │
│   ├── Features
│   │   ├── chat/
│   │   │   ├── ChatList
│   │   │   ├── ChatWindow
│   │   │   ├── MessageInput
│   │   │   └── MessageBubble
│   │   │
│   │   ├── portfolio/
│   │   │   ├── PortfolioOverview
│   │   │   ├── TradeList
│   │   │   └── PerformanceChart
│   │   │
│   │   └── market/
│   │       ├── PriceTicker
│   │       ├── FearGreedGauge
│   │       └── CoinCard
│   │
│   └── Shared Components
│       ├── Button
│       ├── Card
│       ├── Dialog
│       └── Loading
```

### 2.2 Folder Structure

```
apps/frontend/
│
├── public/
│   ├── manifest.json          # PWA manifest
│   ├── sw.js                  # Service worker
│   └── icons/
│
├── src/
│   ├── main.tsx               # Entry point
│   ├── App.tsx                # Root component
│   │
│   ├── pages/                 # Route pages
│   │   ├── WhyBitcoinFallen.tsx
│   │   ├── Landing.tsx
│   │   ├── Chat.tsx
│   │   ├── Portfolio.tsx
│   │   ├── Notifications.tsx
│   │   └── Settings.tsx
│   │
│   ├── features/              # Feature modules
│   │   ├── chat/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── store/
│   │   │   └── types.ts
│   │   │
│   │   ├── portfolio/
│   │   └── market/
│   │
│   ├── components/            # Shared components
│   │   ├── ui/                # shadcn/ui components
│   │   ├── layouts/
│   │   └── common/
│   │
│   ├── lib/                   # Utilities
│   │   ├── api.ts             # API client
│   │   ├── sse.ts             # SSE client
│   │   └── utils.ts
│   │
│   ├── hooks/                 # Global hooks
│   │   ├── useAuth.ts
│   │   └── usePush.ts
│   │
│   ├── store/                 # Zustand stores
│   │   ├── auth.store.ts
│   │   ├── ui.store.ts
│   │   └── chat.store.ts
│   │
│   └── styles/
│       ├── globals.css
│       └── tailwind.css
│
└── vite.config.ts
```

---

## 3. State Management

### 3.1 Zustand (Client State)

#### 3.1.1 UI State

```typescript
// store/ui.store.ts
interface UIState {
  sidebarOpen: boolean;
  toggleSidebar: () => void;

  currentModal: 'none' | 'settings' | 'portfolio-create';
  openModal: (modal: string) => void;
  closeModal: () => void;
}

export const useUIStore = create<UIState>((set) => ({
  sidebarOpen: true,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),

  currentModal: 'none',
  openModal: (modal) => set({ currentModal: modal }),
  closeModal: () => set({ currentModal: 'none' })
}));
```

#### 3.1.2 Chat State

```typescript
// store/chat.store.ts
interface ChatState {
  activeChatId: string | null;
  setActiveChatId: (id: string | null) => void;

  streamingMessage: string;
  setStreamingMessage: (message: string) => void;
  clearStreamingMessage: () => void;
}
```

### 3.2 TanStack Query (Server State)

#### 3.2.1 Chats

```typescript
// hooks/useChats.ts
export function useChats() {
  return useQuery({
    queryKey: ['chats'],
    queryFn: () => api.get('/api/chats').then(res => res.data),
    staleTime: 5 * 60 * 1000  // 5 minutes
  });
}

export function useCreateChat() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: () => api.post('/api/chats').then(res => res.data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['chats'] });
    }
  });
}
```

#### 3.2.2 Messages

```typescript
// hooks/useMessages.ts
export function useMessages(chatId: string) {
  return useQuery({
    queryKey: ['chats', chatId, 'messages'],
    queryFn: () => api.get(`/api/chats/${chatId}/messages`).then(res => res.data),
    enabled: !!chatId
  });
}
```

#### 3.2.3 Market Data

```typescript
// hooks/useMarket.ts
export function usePrices() {
  return useQuery({
    queryKey: ['market', 'prices'],
    queryFn: () => api.get('/api/market/prices').then(res => res.data),
    refetchInterval: 60 * 1000  // Refetch every minute
  });
}

export function useFearGreed() {
  return useQuery({
    queryKey: ['market', 'fear-greed'],
    queryFn: () => api.get('/api/market/fear-greed').then(res => res.data),
    refetchInterval: 5 * 60 * 1000  // 5 minutes
  });
}
```

---

## 4. SSE (Server-Sent Events)

### 4.1 Chat Streaming Client

```typescript
// lib/sse.ts
export class SSEClient {
  private eventSource: EventSource | null = null;

  connect(chatId: string, message: string, onToken: (token: string) => void) {
    const url = `/api/chats/${chatId}/messages?message=${encodeURIComponent(message)}`;

    this.eventSource = new EventSource(url, { withCredentials: true });

    this.eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);

      if (data.type === 'token') {
        onToken(data.content);
      } else if (data.type === 'done') {
        this.close();
      }
    };

    this.eventSource.onerror = (error) => {
      console.error('SSE error:', error);
      this.close();
    };
  }

  close() {
    if (this.eventSource) {
      this.eventSource.close();
      this.eventSource = null;
    }
  }
}
```

### 4.2 Usage in Component

```typescript
// features/chat/components/ChatWindow.tsx
export function ChatWindow({ chatId }: { chatId: string }) {
  const [streamingMessage, setStreamingMessage] = useState('');
  const sseClient = useRef(new SSEClient());

  const sendMessage = async (content: string) => {
    setStreamingMessage('');

    sseClient.current.connect(
      chatId,
      content,
      (token) => {
        setStreamingMessage((prev) => prev + token);
      }
    );
  };

  return (
    <div>
      {/* Messages */}
      <MessageList chatId={chatId} />

      {/* Streaming message */}
      {streamingMessage && (
        <MessageBubble role="assistant" content={streamingMessage} />
      )}

      {/* Input */}
      <MessageInput onSend={sendMessage} />
    </div>
  );
}
```

---

## 5. Key Components

### 5.1 ChatWindow

```typescript
// features/chat/components/ChatWindow.tsx
interface ChatWindowProps {
  chatId: string;
}

export function ChatWindow({ chatId }: ChatWindowProps) {
  const { data: messages } = useMessages(chatId);
  const [streamingMessage, setStreamingMessage] = useState('');

  return (
    <div className="flex flex-col h-screen">
      {/* Header */}
      <ChatHeader chatId={chatId} />

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages?.map((msg) => (
          <MessageBubble key={msg.id} message={msg} />
        ))}

        {streamingMessage && (
          <MessageBubble
            role="assistant"
            content={streamingMessage}
            isStreaming
          />
        )}
      </div>

      {/* Input */}
      <MessageInput onSend={handleSend} />
    </div>
  );
}
```

### 5.2 MessageBubble

```typescript
interface MessageBubbleProps {
  role: 'user' | 'assistant';
  content: string;
  isStreaming?: boolean;
  signal?: AISignal;
}

export function MessageBubble({ role, content, isStreaming, signal }: MessageBubbleProps) {
  const isAssistant = role === 'assistant';

  return (
    <div className={cn('flex', isAssistant ? 'justify-start' : 'justify-end')}>
      <div className={cn(
        'max-w-[80%] rounded-lg px-4 py-2',
        isAssistant ? 'bg-gray-100 text-gray-900' : 'bg-blue-600 text-white'
      )}>
        {/* Avatar */}
        {isAssistant && <Avatar name="Wallet Buffett" />}

        {/* Content */}
        <div className="prose prose-sm">
          <ReactMarkdown>{content}</ReactMarkdown>
        </div>

        {/* AI Signal (Add to Portfolio button) */}
        {signal && (
          <div className="mt-2 p-2 bg-yellow-50 rounded border border-yellow-200">
            <p className="text-sm text-yellow-800">
              {signal.action === 'buy' ? 'Buy' : 'Sell'} Signal: {signal.symbol}
            </p>
            <Button size="sm" onClick={() => handleAddToPortfolio(signal)}>
              Add to Portfolio
            </Button>
          </div>
        )}

        {/* Streaming indicator */}
        {isStreaming && <span className="animate-pulse">|</span>}
      </div>
    </div>
  );
}
```

### 5.3 PortfolioOverview

```typescript
export function PortfolioOverview() {
  const { data: trades } = useShadowTrades();
  const { data: performance } = usePortfolioPerformance();

  return (
    <div className="space-y-6">
      {/* Summary Cards */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <StatCard
          title="Total Return"
          value={`${performance?.totalReturn.toFixed(2)}%`}
          trend={performance?.totalReturn > 0 ? 'up' : 'down'}
        />
        <StatCard
          title="vs Benchmark"
          value={`${performance?.vsBasemark.toFixed(2)}%`}
        />
        <StatCard
          title="Total Trades"
          value={trades?.length || 0}
        />
      </div>

      {/* Performance Chart */}
      <Card>
        <CardHeader>
          <CardTitle>Performance History</CardTitle>
        </CardHeader>
        <CardContent>
          <PerformanceChart data={performance?.history} />
        </CardContent>
      </Card>

      {/* Trade List */}
      <Card>
        <CardHeader>
          <CardTitle>Trade History</CardTitle>
        </CardHeader>
        <CardContent>
          <TradeList trades={trades} />
        </CardContent>
      </Card>
    </div>
  );
}
```

### 5.4 PriceTicker

```typescript
export function PriceTicker() {
  const { data: prices } = usePrices();

  return (
    <div className="bg-gray-900 text-white py-2 overflow-hidden">
      <div className="flex space-x-8 animate-scroll">
        {prices?.map((coin) => (
          <div key={coin.symbol} className="flex items-center space-x-2 whitespace-nowrap">
            <span className="font-bold">{coin.symbol}</span>
            <span>${coin.price.toLocaleString()}</span>
            <span className={coin.change24h > 0 ? 'text-green-400' : 'text-red-400'}>
              {coin.change24h > 0 ? '+' : ''}{Math.abs(coin.change24h).toFixed(2)}%
            </span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 5.5 FearGreedGauge

```typescript
export function FearGreedGauge() {
  const { data: fearGreed } = useFearGreed();

  const getColor = (value: number) => {
    if (value < 25) return 'text-red-500';
    if (value < 45) return 'text-orange-500';
    if (value < 55) return 'text-yellow-500';
    if (value < 75) return 'text-green-500';
    return 'text-emerald-500';
  };

  const getLabel = (value: number) => {
    if (value < 25) return 'Extreme Fear';
    if (value < 45) return 'Fear';
    if (value < 55) return 'Neutral';
    if (value < 75) return 'Greed';
    return 'Extreme Greed';
  };

  return (
    <Card>
      <CardHeader>
        <CardTitle>Fear & Greed Index</CardTitle>
      </CardHeader>
      <CardContent className="flex flex-col items-center">
        <div className="relative w-48 h-48">
          {/* Gauge SVG */}
          <svg viewBox="0 0 200 200">
            <circle
              cx="100"
              cy="100"
              r="80"
              fill="none"
              stroke="#e5e7eb"
              strokeWidth="20"
            />
            <circle
              cx="100"
              cy="100"
              r="80"
              fill="none"
              stroke="currentColor"
              strokeWidth="20"
              strokeDasharray={`${(fearGreed?.value || 0) * 5.03} 503`}
              className={getColor(fearGreed?.value || 0)}
              transform="rotate(-90 100 100)"
            />
          </svg>

          <div className="absolute inset-0 flex flex-col items-center justify-center">
            <div className={cn('text-4xl font-bold', getColor(fearGreed?.value || 0))}>
              {fearGreed?.value || 0}
            </div>
            <div className="text-sm text-gray-500">
              {getLabel(fearGreed?.value || 0)}
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  );
}
```

---

## 6. PWA & Push Notifications

### 6.1 Service Worker Registration

```typescript
// main.tsx
import { registerSW } from 'virtual:pwa-register';

registerSW({
  onNeedRefresh() {
    // Show update prompt
  },
  onOfflineReady() {
    console.log('App ready to work offline');
  }
});
```

### 6.2 Push Subscription

```typescript
// hooks/usePush.ts
export function usePush() {
  const subscribe = async () => {
    if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
      throw new Error('Push notifications not supported');
    }

    const registration = await navigator.serviceWorker.ready;

    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
    });

    // Send subscription to backend
    await api.post('/api/push/subscribe', subscription.toJSON());

    return subscription;
  };

  const unsubscribe = async () => {
    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();

    if (subscription) {
      await subscription.unsubscribe();
      await api.post('/api/push/unsubscribe');
    }
  };

  return { subscribe, unsubscribe };
}
```

### 6.3 Service Worker (sw.js)

```javascript
self.addEventListener('push', (event) => {
  const data = event.data.json();

  const options = {
    body: data.message,
    icon: '/icons/icon-192x192.png',
    badge: '/icons/badge-72x72.png',
    data: {
      url: data.url || '/'
    }
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  event.waitUntil(
    clients.openWindow(event.notification.data.url)
  );
});
```

---

## 7. Routing

### 7.1 Route Configuration

```typescript
// App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/why-bitcoin-fallen" element={<WhyBitcoinFallen />} />
        <Route path="/landing" element={<Landing />} />

        {/* Auth routes */}
        <Route path="/auth/login" element={<Login />} />
        <Route path="/auth/callback" element={<AuthCallback />} />

        {/* Protected routes */}
        <Route element={<ProtectedRoute />}>
          <Route path="/" element={<AppLayout />}>
            <Route index element={<Navigate to="/chat" replace />} />
            <Route path="chat" element={<ChatList />} />
            <Route path="chat/:id" element={<ChatWindow />} />
            <Route path="portfolio" element={<PortfolioOverview />} />
            <Route path="notifications" element={<NotificationList />} />
            <Route path="settings" element={<Settings />} />
          </Route>
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 7.2 Deep Links

```typescript
// Handle deep link from notification
// /chat/new?context=btc-alert-2024-01-15

export function ChatWindow() {
  const [searchParams] = useSearchParams();
  const context = searchParams.get('context');

  useEffect(() => {
    if (context) {
      // Load pre-filled message based on context
      const message = generateMessageFromContext(context);
      sendMessage(message);
    }
  }, [context]);
}
```

---

## 8. Styling

### 8.1 Tailwind Configuration

```javascript
// tailwind.config.js
export default {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        'sage-gold': '#D4AF37',
        'deep-navy': '#1A237E',
        'wisdom-gray': '#37474F'
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace']
      }
    }
  }
};
```

### 8.2 Component Styling (shadcn/ui)

```tsx
// components/ui/button.tsx
import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ variant = 'default', size = 'md', className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        'rounded-md font-medium transition-colors',
        {
          'bg-sage-gold text-white hover:bg-sage-gold/90': variant === 'default',
          'border border-gray-300 hover:bg-gray-50': variant === 'outline',
          'hover:bg-gray-100': variant === 'ghost'
        },
        {
          'px-3 py-1.5 text-sm': size === 'sm',
          'px-4 py-2': size === 'md',
          'px-6 py-3 text-lg': size === 'lg'
        },
        className
      )}
      {...props}
    />
  );
}
```

---

## 9. Performance Optimization

### 9.1 Code Splitting

```typescript
// Lazy load pages
const ChatWindow = lazy(() => import('./pages/Chat'));
const Portfolio = lazy(() => import('./pages/Portfolio'));

<Suspense fallback={<LoadingSpinner />}>
  <ChatWindow />
</Suspense>
```

### 9.2 Image Optimization

```typescript
// Use WebP with fallback
<picture>
  <source srcSet="/images/hero.webp" type="image/webp" />
  <img src="/images/hero.png" alt="Hero" />
</picture>
```

### 9.3 Memoization

```typescript
// Expensive calculation
const performanceData = useMemo(() => {
  return calculatePerformance(trades);
}, [trades]);

// Callback
const handleSendMessage = useCallback((message: string) => {
  sendMessage(chatId, message);
}, [chatId]);
```

---

## 10. Testing

### 10.1 Component Tests (Vitest + Testing Library)

```typescript
// ChatWindow.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('ChatWindow', () => {
  it('should send message on submit', async () => {
    render(<ChatWindow chatId="123" />);

    const input = screen.getByPlaceholderText('Type a message...');
    const sendButton = screen.getByRole('button', { name: 'Send' });

    await userEvent.type(input, 'Hello');
    await userEvent.click(sendButton);

    await waitFor(() => {
      expect(screen.getByText('Hello')).toBeInTheDocument();
    });
  });
});
```

---

## 11. Build & Deployment

### 11.1 Build Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      manifest: {
        name: 'Sage.ai',
        short_name: 'Sage',
        theme_color: '#D4AF37',
        icons: [
          {
            src: '/icons/icon-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          }
        ]
      }
    })
  ],
  build: {
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu']
        }
      }
    }
  }
});
```

### 11.2 Deployment (S3 + CloudFront)

```bash
# Build
pnpm run build

# Upload to S3
aws s3 sync dist/ s3://sage-frontend --delete

# Invalidate CloudFront
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"
```

---

## Appendix A: Key Dependencies

```json
{
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-router-dom": "^6.20.0",
    "@tanstack/react-query": "^5.0.0",
    "zustand": "^4.0.0",
    "axios": "^1.6.0",
    "react-markdown": "^9.0.0",
    "lucide-react": "^0.300.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0",
    "typescript": "^5.3.0",
    "tailwindcss": "^3.4.0",
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.0.0"
  }
}
```

## Appendix B: Environment Variables

```env
VITE_API_URL=https://api.sage.ai
VITE_VAPID_PUBLIC_KEY=xxx
```

## Appendix C: Useful Hooks

```typescript
// hooks/useDebounce.ts
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}
```

---

```
Document Footer
├── Last Updated: 2025-12-19
├── Version: 1.0
├── Status: Production Ready
└── Maintainer: Sam (dev@5010.tech)
```
