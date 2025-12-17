# Sage.ai 아키텍처 재구성 계획

## 개요

**현재 상태**: Next.js 15 모놀리스 기반 문서 (실제 코드 없음)
**목표 상태**: 완전 분리된 Python 백엔드 + React SPA 프론트엔드 + 헥사고널 아키텍처
**타임라인**: 8주 MVP 유지

### 핵심 결정 요약

| 항목 | 최종 결정 | 이유 |
|------|----------|------|
| **백엔드** | Django + DRF | 빠른 개발 속도 (Admin, ORM, Allauth 내장) |
| **프론트엔드** | Vite + React (SPA) | Next.js 완전 제거, 완전 분리 아키텍처 |
| **ORM** | Django ORM | Django 생태계 통합, 마이그레이션 자동 생성 |
| **인증** | Django-Allauth | Google OAuth 5분 설정 |
| **스트리밍** | Django Channels | SSE 지원 (Claude 실시간 응답) |
| **SEO** | vite-plugin-seo-prerender | Next.js 없이 해결 가능 |

---

## 1. 핵심 기술 스택 결정

### 1.1 백엔드: **Django + Django REST Framework** ✅

**선택 근거 (8주 MVP에 최적)**:
- ✅ **빠른 개발 속도**: Admin 패널, 인증, ORM이 기본 제공 → 초기 개발 2배 빠름
- ✅ **Django Admin**: 사용자, 메시지, 포트폴리오 관리 UI 즉시 제공 (디버깅/관리 용이)
- ✅ **Django ORM**: 성숙한 ORM, 마이그레이션 도구, 헥사고널 아키텍처 Repository 패턴 적용 가능
- ✅ **Django REST Framework**: API 직렬화, 인증, 권한, OpenAPI 문서 생성 (drf-spectacular)
- ✅ **Django-Allauth**: Google OAuth 5분 설정
- ✅ **배터리 포함**: 8주 타임라인에 프레임워크 조합 시간 낭비 없음

**비동기 처리 전략**:
- Django 5.1+ ASGI 지원 (비동기 뷰 가능)
- Claude 스트리밍: Django Channels (WebSocket/SSE)
- 외부 API 호출: `httpx` (async HTTP 클라이언트) + 비동기 뷰
- ORM: 대부분 동기 사용 (조회는 빠름), 필요시 `sync_to_async()` 래퍼

**FastAPI 대비 Trade-off**:
- ❌ 순수 async 성능은 낮음 (하지만 MVP 스케일에서는 무의미)
- ❌ Pydantic 런타임 검증 없음 (DRF serializers로 대체)
- ✅ Admin 패널로 개발 속도 2배 향상
- ✅ 프레임워크 성숙도 (16년 역사 vs FastAPI 5년)

### 1.2 ORM: **Django ORM** ✅

**중요**: Drizzle ORM은 TypeScript 전용 → Python 백엔드에서 사용 불가

**Django ORM 선택 근거**:
- ✅ **Django 통합**: 프레임워크 네이티브, 마이그레이션 자동 생성 (`python manage.py makemigrations`)
- ✅ **성숙도**: 16년 역사, 프로덕션 검증 완료, 광범위한 문서
- ✅ **Admin 자동 연동**: 모델 정의만 하면 Admin UI 즉시 생성
- ✅ **헥사고널 아키텍처 적용 가능**: Repository 패턴으로 ORM 추상화 가능
- ✅ **QuerySet API**: 직관적이고 강력한 쿼리 빌더 (`.filter()`, `.select_related()`, `.prefetch_related()`)

**Django ORM으로 Repository 패턴 구현**:
```python
# domain/ports/repositories.py (인터페이스)
class UserRepository(ABC):
    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass

# infrastructure/database/repositories.py (구현)
class DjangoUserRepository(UserRepository):
    def find_by_email(self, email: str) -> Optional[User]:
        try:
            user_model = DjangoUser.objects.get(email=email)
            return self._to_domain(user_model)  # Django 모델 → 도메인 엔티티
        except DjangoUser.DoesNotExist:
            return None
```

**대안 비교**:
- SQLAlchemy 2.0: 프레임워크 독립적이지만 Django 생태계와 분리 (Admin 활용 불가)
- Prisma Python: 타입 세이프하지만 베타, 생태계 미성숙
- Tortoise ORM: Async 우선이지만 커뮤니티 작고 Django와 별도

### 1.3 인증: **Django-Allauth** ✅

- **Django-Allauth**: Google OAuth2 + 세션 기반 인증 (설정 5분)
- **Django 세션 프레임워크**: `SESSION_COOKIE_HTTPONLY=True`, `SESSION_COOKIE_SECURE=True`, `SESSION_COOKIE_SAMESITE='Lax'`
- **DRF Token/Session Auth**: REST API 인증 (쿠키 세션 or Token)
- **커스터마이징 용이**: AllAuth 시그널로 사용자 프로필 자동 생성

### 1.4 프론트엔드: **Vite 6 + React 19 + TanStack Query** (Next.js 제거) ✅

**Next.js 완전 제거 이유**:
- ❌ SSR 불필요: 메인 앱은 인증 후 사용 (SEO 무관)
- ❌ 백엔드 분리 원칙: Next.js API Routes 사용 안 함 → Vite SPA가 더 단순
- ✅ 빠른 개발 경험: Vite의 HMR (Hot Module Replacement)이 Next.js보다 빠름
- ✅ 배포 단순성: S3 + CloudFront 정적 호스팅 (Next.js Vercel 종속 없음)

**프론트엔드 스택**:
- **빌드 도구**: Vite 6 (CRA 대비 10배 빠름)
- **프레임워크**: React 19 (최신 안정 버전, 2024년 12월)
- **상태 관리**:
  - 서버 상태 → **TanStack Query** (캐싱, 자동 리페칭, 낙관적 업데이트)
  - UI 상태 → **Zustand** (경량 3KB, Redux 대비 90% 코드 감소)
- **API 클라이언트**: `drf-spectacular` + `openapi-typescript-codegen` (Django REST Framework OpenAPI 스펙에서 자동 생성)
- **스타일링**: Tailwind CSS 4 (현재 계획 유지)
- **컴포넌트**: shadcn/ui (현재 계획 유지)

**SEO 전략** (Next.js 없이 해결):
1. **랜딩 페이지** (WhyBitcoinFallen.com, Sage.ai 랜딩):
   - Vite 빌드 시 `vite-plugin-seo-prerender` 사용 (정적 HTML 생성)
   - 또는 Prerender.io / Rendertron (크롤러 감지 시 서버 사이드 렌더링)
   - `<meta>` 태그 최적화 (react-helmet-async)
2. **메인 앱** (app.sage-ai.com):
   - 로그인 후 사용 → SEO 불필요
   - `<meta>` 태그는 기본만 설정

---

## 2. 헥사고널 아키텍처 구조

### 2.1 레이어 의존성

```
Presentation → Application → Domain ← Infrastructure
 (Django DRF)  (Use Cases)  (Entities)  (Adapters)
```

### 2.2 백엔드 폴더 구조 (Django + 헥사고널)

```
/apps/backend/
├── config/                         # Django 프로젝트 설정
│   ├── __init__.py
│   ├── settings/
│   │   ├── base.py                 # 공통 설정
│   │   ├── dev.py                  # 개발 환경
│   │   └── prod.py                 # 프로덕션 환경
│   ├── urls.py                     # Root URL 설정
│   ├── asgi.py                     # ASGI 진입점 (Channels)
│   └── wsgi.py                     # WSGI 진입점
│
├── domain/                         # 핵심 비즈니스 로직 (Pure Python)
│   ├── entities/                   # 도메인 모델 (dataclass)
│   │   ├── user.py
│   │   ├── chat.py
│   │   ├── message.py
│   │   └── portfolio.py
│   ├── value_objects/              # 불변 값 객체
│   │   ├── email.py
│   │   ├── tier.py
│   │   └── signal.py
│   └── ports/                      # 인터페이스 (ABC)
│       ├── repositories.py         # 데이터베이스 포트
│       ├── ai_service.py           # AI 포트
│       ├── market_data.py          # 외부 API 포트
│       └── notification.py         # 알림 포트
│
├── application/                    # 유즈케이스 (비즈니스 규칙)
│   ├── services/
│   │   ├── chat_service.py             # 채팅 플로우 오케스트레이션
│   │   ├── portfolio_service.py        # 섀도우 포트폴리오 로직
│   │   └── alert_service.py            # 알림 로직
│   └── dto/                        # 데이터 전송 객체
│       ├── chat_request.py
│       └── chat_response.py
│
├── infrastructure/                 # 어댑터 (외부 시스템)
│   ├── ai/                         # Claude 통합
│   │   ├── anthropic_adapter.py
│   │   ├── agents/                 # Multi-agent system
│   │   │   ├── manager.py
│   │   │   ├── analyst.py
│   │   │   ├── persona.py
│   │   │   └── risk.py
│   │   └── prompts/                # 시스템 프롬프트
│   ├── market/                     # 외부 API
│   │   ├── coingecko.py
│   │   ├── cryptopanic.py
│   │   └── fear_greed.py
│   ├── cache/                      # Redis 어댑터
│   │   └── redis_cache.py
│   └── notifications/              # PWA Push + Discord
│       ├── discord_webhook.py
│       └── webpush.py
│
├── apps/                           # Django Apps (Presentation + Infrastructure)
│   ├── users/                      # 사용자 관리
│   │   ├── models.py               # Django 모델 (User)
│   │   ├── admin.py                # Admin 설정
│   │   ├── serializers.py          # DRF Serializers
│   │   ├── views.py                # DRF ViewSets
│   │   ├── urls.py                 # URL 라우팅
│   │   └── repositories.py         # Repository 구현체
│   ├── chats/                      # 채팅 관리
│   │   ├── models.py               # Django 모델 (Chat, Message)
│   │   ├── admin.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── consumers.py            # Django Channels (SSE/WebSocket)
│   │   ├── routing.py              # Channels 라우팅
│   │   ├── urls.py
│   │   └── repositories.py
│   ├── portfolio/                  # 섀도우 포트폴리오
│   │   ├── models.py               # Django 모델 (ShadowTrade)
│   │   ├── admin.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   └── repositories.py
│   └── audit/                      # 감사 로그
│       ├── models.py               # AuditLog 모델
│       ├── admin.py
│       ├── middleware.py           # 자동 로깅 미들웨어
│       └── signals.py              # Django signals
│
├── tests/
│   ├── unit/                       # Domain + Application 테스트
│   ├── integration/                # Infrastructure 테스트
│   └── e2e/                        # API 테스트
│
├── manage.py                       # Django CLI
├── pyproject.toml                  # Poetry 의존성
├── requirements.txt                # Pip 의존성 (Docker용)
└── Dockerfile                      # 프로덕션 컨테이너
```

### 2.3 헥사고널 아키텍처 예시: Chat Service

```python
# 1. Port (인터페이스) - domain/ports/ai_service.py
class AIService(ABC):
    @abstractmethod
    async def generate_response(
        self, messages: list[Message], context: UserContext
    ) -> ChatResponse:
        pass

# 2. Adapter (구현) - infrastructure/ai/anthropic_adapter.py
class AnthropicAIService(AIService):
    async def generate_response(
        self, messages: list[Message], context: UserContext
    ) -> ChatResponse:
        # Claude API 호출 + Multi-agent 오케스트레이션
        intent = await self.manager.analyze_intent(messages[-1])
        facts = await self.analyst.gather_facts(intent)
        draft = await self.persona.generate_response(facts, context)
        final = await self.risk.validate(draft)
        return ChatResponse(message=final, ...)

# 3. Use Case (비즈니스 로직) - application/services/chat_service.py
class ChatService:
    def __init__(
        self,
        ai_service: AIService,  # 주입된 포트
        user_repo: UserRepository,
        message_repo: MessageRepository
    ):
        self.ai_service = ai_service
        ...

    async def send_message(
        self, user_id: UUID, chat_id: UUID, content: str
    ) -> ChatResponse:
        user = await self.user_repo.find_by_id(user_id)
        context = await self._build_context(chat_id, user)
        response = await self.ai_service.generate_response(
            messages=context.messages,
            context=context
        )
        await self.message_repo.save(response.message)
        return response

# 4. Controller (API) - apps/chats/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated

class ChatViewSet(APIView):
    permission_classes = [IsAuthenticated]

    def __init__(self):
        # 의존성 주입 (Django에서는 __init__에서)
        self.chat_service = ChatService(
            ai_service=AnthropicAIService(),
            user_repo=DjangoUserRepository(),
            message_repo=DjangoMessageRepository()
        )

    def post(self, request):
        serializer = ChatRequestSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        response = self.chat_service.send_message(
            user_id=request.user.id,
            chat_id=serializer.validated_data['chat_id'],
            content=serializer.validated_data['message']
        )
        return Response(ChatResponseSerializer(response).data)
```

**장점**:
- 테스트 용이: `AIService` 인터페이스를 Mock으로 대체
- 유연성: Claude → OpenAI 교체 시 비즈니스 로직 변경 없음
- 유지보수: 명확한 책임 분리

---

## 3. 프로젝트 구조: Monorepo (Turborepo)

```
/sage-platform/                        # Root 모노레포
├── apps/
│   ├── backend/                       # Python Django + DRF
│   ├── frontend-app/                  # Sage.ai 메인 앱 (Vite + React)
│   ├── frontend-landing/              # 랜딩 페이지
│   └── frontend-bitcoin-fallen/       # WhyBitcoinFallen.com
│
├── packages/
│   ├── ui/                            # 공유 React 컴포넌트 (shadcn/ui)
│   ├── types/                         # 공유 TypeScript 타입 (API 계약)
│   └── config/                        # 공유 설정 (ESLint, TypeScript)
│
├── infrastructure/                    # IaC (Terraform)
│   ├── terraform/
│   │   ├── modules/                   # ECS, RDS, ElastiCache, S3/CloudFront
│   │   └── environments/              # dev, prod
│   └── scripts/                       # 배포 스크립트
│
├── turbo.json                         # Turborepo 설정
├── package.json                       # Root package.json
├── pnpm-workspace.yaml                # pnpm workspaces
└── README.md
```

**모노레포 선택 이유**:
- 공유 타입: API 계약을 `/packages/types`에서 관리
- 공유 UI: 버튼, 인풋 등 `/packages/ui`에서 재사용
- 원자적 배포: 백엔드 + 프론트엔드 동시 배포
- DX: 단일 `pnpm install`, 통합 CI/CD

---

## 4. 데이터베이스 스키마 & 감사 로그

### 4.1 감사 로그 전략: 별도 테이블 (권장)

```sql
-- 메인 테이블
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    tier VARCHAR(20),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- 감사 로그 테이블 (append-only)
CREATE TABLE audit_log (
    id UUID PRIMARY KEY,
    table_name VARCHAR(50),
    record_id UUID,
    action VARCHAR(10),      -- INSERT, UPDATE, DELETE
    old_values JSONB,         -- 변경 전 상태
    new_values JSONB,         -- 변경 후 상태
    user_id UUID,             -- 누가 변경했는지
    timestamp TIMESTAMP,
    ip_address VARCHAR(45)
);

CREATE INDEX idx_audit_table_record ON audit_log(table_name, record_id);
CREATE INDEX idx_audit_timestamp ON audit_log(timestamp DESC);
```

**SQLAlchemy 구현**:
```python
# infrastructure/database/audit.py
from sqlalchemy import event

@event.listens_for(User, 'after_insert')
def audit_user_insert(mapper, connection, target):
    audit = AuditLog(
        table_name='users',
        record_id=target.id,
        action='INSERT',
        new_values=target.__dict__,
        user_id=get_current_user_id(),
        timestamp=datetime.utcnow()
    )
    connection.execute(audit.insert())
```

**대안**: PostgreSQL 트리거 (더 빠르지만 유연성 낮음)

### 4.2 ERD 핵심 엔티티

- **Users**: 사용자 정보, OAuth ID, tier
- **Chats**: 대화 세션
- **Messages**: 메시지 (user/ai), 타임스탬프
- **UserProfile**: 대화에서 추론한 선호도 (experience_level, risk_tolerance, interested_assets)
- **ShadowPortfolio**: AI 추천 트래킹 (symbol, buy_price, quantity, signal_date)
- **MarketData**: 캐시된 가격/감정 스냅샷
- **Alerts**: 생성된 알림 (sent_at, type, content)
- **AuditLog**: 감사 로그

---

## 5. API 설계 & 실시간 통신

### 5.1 REST API (GraphQL 대신)

**이유**:
- DRF의 OpenAPI 자동 생성 (drf-spectacular)
- HTTP 캐싱 (Redis)이 REST에서 더 쉬움
- 8주 타임라인에 적합 (GraphQL은 스키마/리졸버 설정 복잡)
- Django의 성숙한 생태계 활용

**버전 관리**: `/api/v1/chat`, `/api/v1/portfolio`

**OpenAPI 문서 생성**:
```python
# config/urls.py
from drf_spectacular.views import SpectacularAPIView, SpectacularSwaggerView

urlpatterns = [
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/docs/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
]
```

### 5.2 실시간: Django Channels (SSE/WebSocket)

**SSE vs WebSockets**:
- SSE: 서버 → 클라이언트 단방향, HTTP 기반, 자동 재연결
- WebSockets: 양방향, 프로토콜 업그레이드 필요

**Claude 스트리밍: Django Channels로 SSE 구현**

**Django Channels Consumer**:
```python
# apps/chats/consumers.py
from channels.generic.http import AsyncHttpConsumer
import json

class ChatStreamConsumer(AsyncHttpConsumer):
    async def handle(self, body):
        # SSE 헤더 전송
        await self.send_headers(headers=[
            (b"Content-Type", b"text/event-stream"),
            (b"Cache-Control", b"no-cache"),
            (b"Connection", b"keep-alive"),
        ])

        # Claude 스트리밍
        chat_service = ChatService(...)
        async for chunk in chat_service.stream_response(...):
            data = json.dumps({"text": chunk})
            await self.send_body(f"data: {data}\n\n".encode(), more_body=True)

        # 완료 신호
        await self.send_body(b"data: {\"type\": \"done\"}\n\n", more_body=False)

# apps/chats/routing.py
from django.urls import path
from . import consumers

websocket_urlpatterns = [
    path('ws/chat/stream/', consumers.ChatStreamConsumer.as_asgi()),
]
```

**React 클라이언트** (TanStack Query):
```typescript
import { EventSourcePlus } from 'event-source-plus';

function useChatStream(chatId: string, message: string) {
  const [messages, setMessages] = useState<string[]>([]);

  const startStream = () => {
    const eventSource = new EventSourcePlus(`/api/v1/chat/stream`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ chatId, message }),
      credentials: 'include',  // 쿠키 전송
    });

    eventSource.listen({
      onMessage(msg) {
        const data = JSON.parse(msg.data);
        if (data.text) setMessages(prev => [...prev, data.text]);
      }
    });
  };
}
```

---

## 6. 배포 & 인프라

### 6.1 Docker 컨테이너화

**백엔드 Dockerfile** (Django + Gunicorn + Channels):
```dockerfile
# Stage 1: Builder
FROM python:3.12-slim as builder
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .

# Django 마이그레이션 & collectstatic
RUN python manage.py collectstatic --noinput

# Daphne (Channels ASGI 서버) or Gunicorn
CMD ["daphne", "-b", "0.0.0.0", "-p", "8000", "config.asgi:application"]
# 또는 REST API만: CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000"]
```

### 6.2 AWS 인프라

- **백엔드**: ECS Fargate (Docker 컨테이너)
- **데이터베이스**: RDS PostgreSQL 16+ (db.t3.micro)
- **캐시**: ElastiCache Redis 7.x (cache.t3.micro)
- **프론트엔드**: S3 + CloudFront (정적 호스팅)
- **스케줄링**: Celery Beat (15분마다 시장 분석, Django 내장)

### 6.3 CI/CD: GitHub Actions

```yaml
# .github/workflows/deploy-backend.yml
name: Deploy Backend
on:
  push:
    branches: [main]
    paths: ['apps/backend/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t sage-backend:${{ github.sha }} apps/backend
      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login ...
          docker push $ECR_REGISTRY/sage-backend:latest
      - name: Update ECS Service
        run: aws ecs update-service --cluster sage-cluster --service sage-backend --force-new-deployment
```

---

## 7. MVP vs 완전한 헥사고널 (Trade-offs)

### 7.1 MVP에서 연기할 것 (8주 타임라인 준수)

| 기능 | MVP (8주) | Post-MVP (Phase 2) |
|-----|-----------|-------------------|
| Event Sourcing | ❌ 단순 CRUD | ✅ 이벤트 기반 아키텍처 |
| CQRS | ❌ 단일 모델 | ✅ Read/Write 분리 |
| Domain Events | ❌ 직접 호출 | ✅ 이벤트 버스 (Redis Pub/Sub) |
| Unit Tests | ⚠️ 핵심 경로만 | ✅ 80%+ 커버리지 |
| Repository Pattern | ✅ 기본 인터페이스 | ✅ 고급 (캐싱, 트랜잭션) |
| Multi-Agent System | ✅ 4 에이전트 | ✅ 더 많은 에이전트 추가 |

### 7.2 MVP에서도 유지할 것

1. **도메인 순수성**: `/domain` 폴더는 외부 의존성 없음
2. **Ports/Adapters**: Claude → OpenAI 교체 가능하게
3. **의존성 주입**: FastAPI `Depends()`로 모든 서비스 주입
4. **Repository 인터페이스**: SQLAlchemy 구현 교체 가능하게

---

## 8. 구현 순서 (8주)

### Week 1-2: 기초 인프라
- [ ] FastAPI 프로젝트 초기화 + 헥사고널 폴더 구조
- [ ] SQLAlchemy 모델 정의 (users, chats, messages)
- [ ] Alembic 마이그레이션 설정
- [ ] PostgreSQL RDS + Redis ElastiCache 생성 (Terraform)
- [ ] Vite 프로젝트 3개 초기화 (app, landing, bitcoin-fallen)
- [ ] Turborepo 모노레포 설정
- [ ] FastAPI-Users + Google OAuth 구현
- [ ] 쿠키 기반 세션 인증 구현

### Week 3-4: 핵심 채팅
- [ ] Claude Multi-agent 시스템 구현 (Manager, Analyst, Persona, Risk)
- [ ] SSE 스트리밍 엔드포인트 구현 (`/api/v1/chat/stream`)
- [ ] Context 관리 (20 메시지 윈도우)
- [ ] 프론트엔드 채팅 UI (메시지 버블, 입력창)
- [ ] TanStack Query + SSE 클라이언트 구현
- [ ] 외부 API 어댑터 (CoinGecko, CryptoPanic, Fear & Greed)
- [ ] Redis 캐싱 (5/10/30분 TTL)

### Week 5-6: 섀도우 포트폴리오
- [ ] Shadow trade repository 구현
- [ ] Portfolio service (ROI 계산)
- [ ] Claude 응답에서 시그널 추출
- [ ] 프론트엔드 포트폴리오 페이지
- [ ] "섀도우에 추가" 버튼
- [ ] 성능 차트 (Recharts)

### Week 7-8: 능동적 분석 + 배포
- [ ] Celery + Redis 설정 (비동기 작업)
- [ ] Celery Beat 스케줄러 (15분 자동 시장 분석)
- [ ] 시장 분석 태스크 구현 (CoinGecko, CryptoPanic, Fear & Greed)
- [ ] Discord webhook 통합
- [ ] PWA push notifications
- [ ] Deep linking (`/chat/new?context=market_alert`)
- [ ] Docker 빌드 최적화 (Celery worker 컨테이너 추가)
- [ ] ECS Fargate 배포 (Django + Celery worker)
- [ ] S3 + CloudFront 프론트엔드 배포
- [ ] CI/CD 파이프라인 (GitHub Actions)
- [ ] 통합 테스트
- [ ] 성능 테스트 (2초 응답 시간)
- [ ] 기본 보안 감사

---

## 9. Next.js → Python/React 마이그레이션 전략

### 9.1 재사용 가능한 것

✅ **데이터베이스 스키마**: 현재 ERD 그대로 사용
✅ **AI 프롬프트**: Wallet Buffett 페르소나 그대로 사용
✅ **Multi-agent 로직**: Manager/Analyst/Persona/Risk 플로우 그대로 사용
✅ **UI 컴포넌트**: React 컴포넌트 (Tailwind + shadcn/ui) 재사용
✅ **비즈니스 로직**: 채팅 플로우, 포트폴리오 계산 로직 재사용
✅ **외부 API 통합**: CoinGecko, CryptoPanic 등 Python으로 포팅

### 9.2 완전히 재작성할 것

❌ **백엔드 API 레이어**: Next.js API Routes → Django REST Framework
❌ **ORM 레이어**: Drizzle → Django ORM
❌ **인증 레이어**: Auth.js → Django-Allauth
❌ **AI SDK**: Vercel AI SDK → Anthropic Python SDK
❌ **스트리밍**: Vercel AI SDK → Django Channels

**예상 마이그레이션 노력**: ~60% 재작성 (백엔드), ~20% 재작성 (프론트엔드)

---

## 10. 핵심 파일 (구현 우선순위)

### 백엔드 (Django)

1. **`/apps/backend/domain/ports/repositories.py`**
   모든 Repository 인터페이스 정의 (User, Chat, Message, Portfolio). 헥사고널 아키텍처의 핵심.

2. **`/apps/backend/application/services/chat_service.py`**
   채팅 플로우 비즈니스 로직. Multi-agent 오케스트레이션, 컨텍스트 관리, 프로필 추론.

3. **`/apps/backend/infrastructure/ai/anthropic_adapter.py`**
   Claude 통합 + Multi-agent 시스템 (Manager, Analyst, Persona, Risk). AI 기능의 핵심.

4. **`/apps/backend/apps/users/repositories.py`**
   UserRepository의 Django ORM 구현. 다른 모든 Repository의 패턴.

5. **`/apps/backend/apps/chats/views.py`**
   Django REST Framework 채팅 ViewSet. 인증, 직렬화, 비즈니스 로직 호출.

6. **`/apps/backend/apps/chats/consumers.py`**
   Django Channels Consumer. SSE 스트리밍, 실시간 Claude 응답.

7. **`/apps/backend/apps/users/models.py`** + **`/apps/backend/apps/chats/models.py`**
   Django 모델 정의 (User, Chat, Message). Admin 자동 생성, 마이그레이션 기반.

### 프론트엔드 (TypeScript)

1. **`/apps/frontend-app/src/api/client.ts`**
   OpenAPI 자동 생성 클라이언트 래퍼. Axios 인스턴스 + 쿠키 credentials + 에러 핸들링.

2. **`/apps/frontend-app/src/features/chat/hooks/useChatStream.ts`**
   SSE 스트리밍 hook + TanStack Query. 실시간 채팅의 핵심.

3. **`/apps/frontend-app/src/stores/auth.ts`**
   Zustand 인증 상태 (current user, session). 모든 기능에서 사용.

### 인프라

1. **`/infrastructure/terraform/modules/ecs/main.tf`**
   ECS Fargate 클러스터, 태스크 정의, ALB. 백엔드 배포의 핵심.

2. **`/infrastructure/terraform/modules/rds/main.tf`**
   PostgreSQL RDS 인스턴스, 보안 그룹, 파라미터 그룹. 데이터베이스 기반.

---

## 11. 성공 기준

### 기술적 목표
- ✅ 헥사고널 아키텍처 구현 (Ports/Adapters 패턴)
- ✅ Django REST Framework OpenAPI 자동 문서화 (drf-spectacular)
- ✅ TypeScript 클라이언트 자동 생성
- ✅ Django Admin으로 데이터 관리 UI 제공
- ✅ Claude 응답 시간 < 2초 (첫 토큰)
- ✅ Hallucination 비율 < 1%
- ✅ 감사 로그 100% 커버리지 (중요 작업)

### MVP 목표 (8주)
- ✅ Google OAuth 로그인
- ✅ Wallet Buffett 채팅 (Multi-agent)
- ✅ 실시간 시장 데이터 통합 (6개 자산)
- ✅ 섀도우 포트폴리오 트래킹
- ✅ 15분마다 시장 알림 (Discord + PWA)
- ✅ 3개 프론트엔드 배포 (app, landing, bitcoin-fallen)

---

## 12. 최종 권장 사항

### 핵심 결정 사항 ✅

1. **Django + DRF**: 빠른 개발 속도 우선 (Admin, ORM, Allauth 내장)
2. **Django ORM**: 성숙한 생태계, 마이그레이션 자동 생성
3. **Django Channels**: SSE 스트리밍 지원 (Claude 실시간 응답)
4. **Vite + React SPA**: Next.js 완전 제거 (완전 분리 아키텍처)
5. **Turborepo 모노레포**: 공유 타입/컴포넌트, 빠른 반복
6. **별도 감사 로그 테이블**: 유연성 + 비즈니스 로직 통합 가능

### 타임라인 준수 전략

- **Week 1-2**: 인프라 + 인증 (병렬 작업)
- **Week 3-4**: 채팅 + AI (핵심 가치)
- **Week 5-6**: 포트폴리오 (차별화 요소)
- **Week 7-8**: 알림 + 배포 (완성도)

### 리스크 관리

- ⚠️ **헥사고널 오버엔지니어링**: MVP에서는 실용적 접근 (Event Sourcing, CQRS 연기)
- ⚠️ **Django Channels 학습**: SSE 스트리밍 구현 1-2일 학습 필요 (문서 충분)
- ⚠️ **모노레포 초기 설정**: Turborepo 설정에 1-2일 투자 필요
- ✅ **빠른 반복**: Django Admin (데이터 관리 UI 즉시 제공) + DRF 자동 문서화
- ✅ **성숙한 생태계**: Django 16년 역사, 수많은 패키지, Stack Overflow 자료 풍부

---

**이 계획은 2025년 12월 기준 최고의 팀들이 사용하는 Best Practice를 반영하며, 8주 MVP 타임라인을 유지하면서 완전한 헥사고널 아키텍처의 기반을 구축합니다.**
