# WhyBitcoinFallen.com - Mystery Campaign

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: 마케팅팀, 경영진

---

## Campaign Overview

### Concept

**"왜 비트코인이 떨어졌을까요?"** - 누구나 한 번쯤 궁금해하는 질문에 AI가 즉시 답변

### Objectives

| 목표 | 지표 | Target (3개월) |
|------|------|---------------|
| **Viral Reach** | 일 방문자 | 1,000+ |
| **Conversion** | Sage.ai 가입 전환율 | 10% |
| **Brand Awareness** | SNS 공유 | 100+ shares/day |

---

## Landing Page Design

### Hero Section

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📉 비트코인, 어제보다 -5.2% 떨어졌습니다

왜일까요?

[AI에게 물어보기 →]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Elements**:
- 실시간 BTC 가격
- 24시간 변동률 (빨간색/초록색)
- 간단한 CTA 버튼

### AI Answer Section

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
월렛 버핏이 답합니다:

"자네, 비트코인이 현재 $43,250에 거래되고 있네.
어제보다 -5.2% 하락했군.

시장은 공포에 질렸어 (Fear & Greed 지수 25).
하지만 기억하게, 남들이 두려워할 때가 바로 기회일세.

지난 데이터를 보면, Fear 지수 20-30 구간에서 매수한
투자자들은 평균 6개월 후 +47% 수익을 올렸지.

물론, 과거가 미래를 보장하지는 않아.
하지만 장기 관점으로 접근한다면 어떨까?"

[더 자세한 분석이 궁금하다면? Sage.ai 시작하기 →]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Fear & Greed Gauge

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
현재 시장 심리

Extreme Fear                    Extreme Greed
    25  ●━━━━━━━━━━━━━━━━━━━━○  100

💡 역사적으로 "Extreme Fear" 구간에서 매수한 사람들은
   평균 3개월 후 +35% 수익을 기록했습니다.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### CTA Section

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
24/7 AI 투자 멘토가 필요하신가요?

월렛 버핏과 함께 더 현명한 투자를

✅ 환각 제로 - 오직 데이터에 기반한 분석
✅ 섀도우 포트폴리오로 AI 추천 검증
✅ 급변 시 자동 알림

[무료로 시작하기 →]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Technical Implementation

### Real-Time BTC Price

```typescript
// lib/coingecko.ts
export async function getBTCPrice() {
  const response = await fetch(
    'https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_change=true'
  );

  const data = await response.json();

  return {
    price: data.bitcoin.usd,
    change24h: data.bitcoin.usd_24h_change
  };
}
```

### AI Response Generation

```typescript
// lib/generateAnswer.ts
import Anthropic from '@anthropic-ai/sdk';

export async function generateAnswer(price: number, change24h: number, fearGreed: number) {
  const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

  const prompt = `You are Wallet Buffett (월렛 버핏), an AI investment mentor.

Current data:
- BTC price: $${price.toLocaleString()}
- 24h change: ${change24h.toFixed(2)}%
- Fear & Greed Index: ${fearGreed}

Explain why Bitcoin's price moved in 3-4 sentences.
Use your signature tone: "자네", "~일세"
Focus on data and market psychology.`;

  const message = await client.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 300,
    messages: [{ role: 'user', content: prompt }]
  });

  return message.content[0].text;
}
```

### Caching Strategy

```typescript
// Refresh every 5 minutes
export const revalidate = 300;

export async function getStaticProps() {
  const btc = await getBTCPrice();
  const fearGreed = await getFearGreedIndex();
  const aiAnswer = await generateAnswer(btc.price, btc.change24h, fearGreed.value);

  return {
    props: { btc, fearGreed, aiAnswer },
    revalidate: 300
  };
}
```

---

## SEO Optimization

### Meta Tags

```html
<head>
  <title>비트코인 왜 떨어졌어요? - AI가 알려드립니다</title>
  <meta name="description" content="실시간 AI 분석으로 비트코인 가격 변동 이유를 알아보세요. 월렛 버핏과 함께하는 현명한 투자." />

  <!-- OG Tags -->
  <meta property="og:title" content="비트코인 왜 떨어졌어요?" />
  <meta property="og:description" content="AI가 실시간으로 분석해드립니다" />
  <meta property="og:image" content="https://whybitcoinfallen.com/og-image.png" />
  <meta property="og:type" content="website" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="비트코인 왜 떨어졌어요?" />
  <meta name="twitter:description" content="AI가 실시간으로 분석해드립니다" />
  <meta name="twitter:image" content="https://whybitcoinfallen.com/twitter-card.png" />
</head>
```

### Keywords

- 비트코인 하락 이유
- 비트코인 왜 떨어짐
- 암호화폐 시장 분석
- AI 투자 조언
- 월렛 버핏

---

## Viral Marketing Strategy

### Reddit Seeding

**Subreddits**:
- r/cryptocurrency
- r/Bitcoin
- r/CryptoMarkets
- r/korea (한국어 버전)

**Post Template**:
```
Title: "Made a simple site that explains why Bitcoin is falling - using AI"

Body:
Got tired of guessing why BTC price moves, so I built this:
https://whybitcoinfallen.com

It uses AI (Claude) to analyze current price + Fear & Greed Index
and gives you a quick explanation in Warren Buffett's investing philosophy.

Feedback welcome!
```

### Twitter Strategy

**Viral Hook**:
```
비트코인 -5.2% 떨어졌는데 이유가 궁금하다면?

AI가 실시간으로 분석해드립니다:
https://whybitcoinfallen.com

#Bitcoin #Crypto #AI
```

**Engagement Tactics**:
- 매일 가격 변동 시 자동 트윗
- 유명 크립토 인플루언서 멘션
- "What do you think?" 질문으로 댓글 유도

### Discord/Telegram

**Crypto 커뮤니티에 공유**:
```
Hey everyone! Made a fun little tool:

🤖 WhyBitcoinFallen.com

Real-time AI analysis of why BTC price is moving
Based on Warren Buffett's investing philosophy
100% free, no signup

Try it and let me know what you think!
```

---

## Conversion Funnel

### Stage 1: Curiosity (Landing)
- **Hook**: "왜 비트코인이 떨어졌어요?"
- **Engagement**: 실시간 가격 + AI 답변
- **Goal**: 페이지에 30초+ 머물기

### Stage 2: Interest (Read AI Answer)
- **Value**: 데이터 기반 통찰 제공
- **Trust**: Fear & Greed 지표 시각화
- **Goal**: AI 답변 끝까지 읽기

### Stage 3: Desire (See CTA)
- **Benefit**: "24/7 AI 멘토", "환각 제로", "섀도우 포트폴리오"
- **Social Proof**: "1,000+ 투자자가 사용 중"
- **Goal**: CTA 버튼 클릭

### Stage 4: Action (Sign Up)
- **Friction 제거**: Google OAuth 원클릭 가입
- **Incentive**: "첫 달 Pro 기능 무료"
- **Goal**: Sage.ai 가입 완료

---

## Analytics & Tracking

### Key Metrics

```javascript
// Google Analytics Events
gtag('event', 'page_view', { page_title: 'WhyBitcoinFallen Home' });
gtag('event', 'ai_answer_read', { duration_seconds: 45 });
gtag('event', 'cta_click', { button_location: 'hero' });
gtag('event', 'signup_conversion', { source: 'whybitcoinfallen' });
```

### Conversion Tracking

| Funnel Step | Event | Target Conversion |
|-------------|-------|-------------------|
| **Visit** | page_view | 100% |
| **Engage** | ai_answer_read | 70% |
| **Click CTA** | cta_click | 30% |
| **Sign Up** | signup_conversion | 10% |

---

## A/B Testing

### Test 1: Headline

**Variant A**: "비트코인 왜 떨어졌어요?"
**Variant B**: "AI가 알려주는 비트코인 급락 이유"

**Hypothesis**: B가 AI 강조로 클릭률 높을 것

### Test 2: CTA Text

**Variant A**: "무료로 시작하기"
**Variant B**: "월렛 버핏과 대화하기"

**Hypothesis**: B가 페르소나 강조로 전환율 높을 것

---

## Launch Checklist

- [ ] 도메인 구매 및 DNS 설정
- [ ] SSL 인증서 설정 (Let's Encrypt)
- [ ] Google Analytics 설정
- [ ] OG 이미지 제작 (1200x630px)
- [ ] Twitter Card 이미지 제작 (1200x675px)
- [ ] CoinGecko API 키 발급
- [ ] Alternative.me API 테스트
- [ ] Claude API 호출 테스트
- [ ] 모바일 반응형 테스트
- [ ] 로딩 속도 최적화 (< 2초)
- [ ] Reddit 계정 karma 쌓기 (spam 방지)
- [ ] Twitter 계정 생성 및 프로필 설정

---

## Success Criteria (3개월)

| 지표 | 목표 | 측정 방법 |
|------|------|----------|
| **일 방문자** | 1,000+ | Google Analytics |
| **평균 체류 시간** | 1분+ | GA Engagement Time |
| **CTA 클릭률** | 30%+ | Event tracking |
| **가입 전환율** | 10%+ | Signup event / page_view |
| **SNS 공유** | 100+ shares/day | Share buttons tracking |
| **Organic Search Traffic** | 30% | GA Acquisition |

---

**문서 끝**

_"Between the zeros and ones"_
