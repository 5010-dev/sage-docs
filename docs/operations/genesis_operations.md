# Sage.ai Genesis Operations

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: 운영팀, 경영진

---

## Genesis Phase Overview

### Timeline: M3 Week 12 (Public Launch)

**D-Day**: MVP Public Launch Day

### Objectives

1. **Zero-Downtime Launch** - 가동률 99%+ 유지
2. **First 100 Users** - 첫 100명 온보딩 성공
3. **Viral Moment** - Product Hunt 상위 5위
4. **Community Activation** - Discord 200+ members

---

## Pre-Launch (D-7 to D-1)

### D-7: Final Checks

#### Technical
- [ ] Production 배포 완료 (ECS Fargate)
- [ ] Database 마이그레이션 테스트
- [ ] Sentry 알림 테스트
- [ ] CloudWatch 대시보드 확인
- [ ] SSL 인증서 유효성 확인
- [ ] DNS 설정 확인

#### Content
- [ ] Discord 채널 최종 정리
- [ ] Twitter 론칭 트윗 예약 (Buffer/Hootsuite)
- [ ] Product Hunt 페이지 초안 작성
- [ ] 블로그 포스트 최종 검토

### D-5: Waitlist Activation

- [ ] Waitlist 이메일 발송 (100명)
- [ ] Early Access 초대 코드 생성
- [ ] Discord 초대 링크 공유

### D-3: Team Preparation

- [ ] 운영팀 24시간 대기 체계 구축
- [ ] Emergency Runbook 리뷰
- [ ] 커뮤니케이션 채널 테스트 (Slack/Discord)

### D-1: Final Readiness

- [ ] 최종 QA 테스트
- [ ] 베타 테스터 피드백 최종 반영
- [ ] Press Kit 준비 (로고, 스크린샷, 카피)
- [ ] 론칭 트윗 최종 검토

---

## Launch Day (D-Day)

### 00:00 - 시스템 상태 확인

```bash
# Health Check
curl https://api.sage.ai/health

# Database Connection
psql $DATABASE_URL -c "SELECT 1"

# Redis Connection
redis-cli -u $REDIS_URL ping

# CloudWatch Metrics
aws cloudwatch get-metric-statistics --namespace AWS/ECS ...
```

### 09:00 - Product Hunt 런칭

**Post Template**:
```
Title: Sage.ai - Your 24/7 AI Investment Mentor

Tagline: Warren Buffett's wisdom + Claude AI + Real-time market data

Description:
Meet Wallet Buffett (월렛 버핏) - an AI investment mentor that:
✅ Gives you data-driven insights (zero hallucination)
✅ Tracks AI recommendations with Shadow Portfolios
✅ Alerts you when markets move

Built for crypto investors who want more than just price alerts.

Tech Stack: Nest.js + Claude Sonnet 4 + React

Try it free: https://app.sage.ai

[Screenshot 1: Chat with Wallet Buffett]
[Screenshot 2: Shadow Portfolio]
[Screenshot 3: Market Alert]
```

**Engagement Plan**:
- 09:00: Post 올리기
- 09:30: 팀원들 upvote (5명)
- 10:00: 베타 테스터 upvote 요청
- 12:00: 댓글 적극 응답
- 15:00: 중간 순위 체크, Twitter 공유
- 18:00: 최종 순위 확인

### 10:00 - Twitter 론칭 스레드

```
🚀 Introducing Sage.ai

Your 24/7 AI investment mentor for crypto

Imagine having Warren Buffett as your personal advisor.
Now imagine he never sleeps, knows real-time market data,
and NEVER hallucinates.

That's Wallet Buffett (월렛 버핏)

🧵 1/7

---

🤖 Powered by Claude Sonnet 4

We built a multi-agent system that:
- Fetches real-time data (CoinGecko, Fear & Greed)
- Cross-validates every number
- Gives you insights, not just information

Zero hallucination rate in 2-week beta test.

2/7

---

📊 Shadow Portfolios

"Can AI really make money?"

Track it yourself with Shadow Portfolios.
- Click "Add to Portfolio" on AI recommendations
- See real-time performance
- Compare vs benchmark (BTC)

Transparent. Verifiable. Trustworthy.

3/7

---

🔔 Proactive Alerts

Markets move fast. We've got you covered.

15-min auto analysis detects:
- ±5% BTC moves
- ±7% ETH moves
- Fear & Greed shifts

Get notified via:
- Push (PWA)
- Discord

Never miss a moment.

4/7

---

🆓 Free to start

- 6 coins (BTC, ETH, SOL, BNB, DOGE, XRP)
- Unlimited chats
- 1 Shadow Portfolio

Pro ($19.99/mo):
- 3 Shadow Portfolios
- Priority alerts

Premium ($49.99/mo):
- All features + API access

5/7

---

🧪 Built by a team of 5

- AI-native development (Claude Code + Cursor)
- 3 months from idea to launch
- Tech stack: Nest.js + React + Claude 4

We're shipping fast and iterating based on your feedback.

6/7

---

Try Sage.ai today 👇

https://app.sage.ai

Join our Discord:
https://discord.gg/sage-ai

Questions? I'm here! 🙋‍♂️

7/7

#SageAI #CryptoInvesting #AI
```

### 11:00 - Discord 활성화

**Announcements 채널**:
```
@everyone

🎉 We're LIVE! 🎉

Sage.ai is now publicly available!

Join us in building the future of AI-powered investing.

🔗 https://app.sage.ai

Let's go! 🚀
```

**General Chat 유도**:
```
Hey everyone! 👋

We'd love to hear your first impressions.

What do you think about Wallet Buffett's advice?
Any features you'd like to see?

Drop your thoughts in #feedback!
```

### 12:00 - Reddit 포스트

**r/cryptocurrency**:
```
Title: [Product Launch] Built an AI investment mentor with zero hallucination

Body:
Hey r/cryptocurrency,

After 3 months of development, we're launching Sage.ai today.

**What it is:**
An AI mentor (Wallet Buffett) that gives you investment insights
based on real-time market data + Warren Buffett's philosophy.

**Key features:**
- Zero hallucination (multi-agent validation)
- Shadow Portfolios (track AI recommendations)
- Proactive alerts (15-min market analysis)

**Tech:**
- Nest.js backend
- Claude Sonnet 4 + Haiku 4
- React frontend

**Try it:** https://app.sage.ai (free to start)

Feedback welcome! We're here to answer questions.
```

---

## First 24 Hours Monitoring

### Metrics Dashboard

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| **Signups** | 100+ | - | - |
| **Discord Members** | 200+ | - | - |
| **Product Hunt Upvotes** | 200+ | - | - |
| **Twitter Impressions** | 10,000+ | - | - |
| **Downtime** | 0 | - | - |

### Real-Time Monitoring

```bash
# Every 15 minutes
- Check Sentry errors
- Check ECS task health
- Check RDS connections
- Check Redis memory
- Check API response times
```

### On-Call Rotation

| Time | On-Call Engineer | Backup |
|------|------------------|--------|
| 00:00-08:00 | DevOps Lead | CTO |
| 08:00-16:00 | Backend Lead | CEO |
| 16:00-24:00 | Frontend Lead | CTO |

---

## User Onboarding

### First-Time User Experience (FTUE)

#### Step 1: Google OAuth (5초)
- 원클릭 로그인
- 즉시 채팅 가능

#### Step 2: Welcome Message (자동)
```
👋 안녕하세요! 월렛 버핏입니다.

자네, 암호화폐 투자에 관심이 있군요.

저는 워렌 버핏의 투자 철학을 바탕으로
암호화폐 시장을 분석하는 AI 멘토입니다.

무엇이 궁금한가요?
```

#### Step 3: Guided Questions (선택)
```
💡 이런 질문을 해보세요:

- "비트코인 지금 어때?"
- "이더리움 살까?"
- "포트폴리오 점검해줘"
```

### Activation Emails

**Day 0 (즉시)**:
```
Subject: Welcome to Sage.ai!

Hi [Name],

Welcome to Sage.ai! 🎉

You're now part of a community of smart investors who use AI
to make better decisions.

Here's how to get started:
1. Ask Wallet Buffett about any of the 6 coins we support
2. Try the "Add to Portfolio" feature on AI recommendations
3. Enable notifications to never miss market moves

Questions? Reply to this email or join our Discord.

- The Sage.ai Team
```

**Day 3 (if inactive)**:
```
Subject: Try this: Ask Wallet Buffett about Bitcoin

Hi [Name],

We noticed you haven't had a chance to chat with Wallet Buffett yet.

Here's a quick starter:

"What do you think about Bitcoin right now?"

Click here to try: [Link to chat with pre-filled message]

- The Sage.ai Team
```

---

## Crisis Response

### Scenario 1: Server Overload

**Symptoms**:
- API response time > 5s
- ECS tasks CPU > 90%

**Action**:
1. Trigger auto-scaling (increase to 10 tasks)
2. Enable CloudFront aggressive caching
3. Discord announcement: "We're experiencing high traffic. Scaling up..."

### Scenario 2: AI Hallucination Detected

**Symptoms**:
- User reports wrong price data
- Risk Agent flags mismatch

**Action**:
1. Immediately disable affected message
2. Send apology DM to user
3. Fix prompt/validation
4. Re-deploy within 1 hour

### Scenario 3: Database Connection Pool Exhausted

**Symptoms**:
- "Too many connections" errors
- Cannot create new chats

**Action**:
1. Increase RDS max_connections (temp fix)
2. Identify connection leaks in code
3. Deploy fix
4. Scale RDS instance if needed

---

## First Week Goals

### Week 1 Targets

| Day | Signups | Discord Members | Product Hunt Rank |
|-----|---------|-----------------|-------------------|
| **D+0** | 100 | 200 | Top 5 |
| **D+1** | 150 | 300 | Top 10 |
| **D+2** | 200 | 350 | - |
| **D+3** | 250 | 400 | - |
| **D+7** | 500 | 500 | - |

### Daily Standup

**Time**: Every day 09:00

**Agenda**:
- 어제 주요 지표 리뷰
- 사용자 피드백 요약
- Critical bugs 상태
- 오늘 우선순위

---

## Post-Launch (D+7 to D+30)

### Week 2: Feedback Integration

- [ ] User feedback 분류 (Feature / Bug / UX)
- [ ] Top 3 요청사항 로드맵 반영
- [ ] 주간 업데이트 블로그 포스트

### Week 3-4: Optimization

- [ ] Performance tuning (목표: 응답 속도 < 2초)
- [ ] 환각률 재측정 (목표: < 1%)
- [ ] Conversion funnel 분석

### Month 1 Review

- [ ] MAU 500+ 달성 여부 확인
- [ ] NPS 40+ 달성 여부 확인
- [ ] Product-Market Fit 신호 분석
- [ ] Phase 2 계획 수립

---

**문서 끝**

_"Between the zeros and ones"_
