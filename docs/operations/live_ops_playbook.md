# Sage.ai Live Ops Playbook

> **문서 버전**: 1.0
> **최종 수정**: 2025년 12월 19일
> **작성자**: Sam
> **대상 독자**: 운영팀, 커뮤니티 매니저

---

## Live Ops Overview

### Definition

**Live Ops (Live Operations)**: 런칭 후 서비스를 실시간으로 운영하며 사용자 참여, 리텐션, 수익화를 지속적으로 개선하는 활동

### Goals

1. **Engagement** - 사용자 활성화 (DAU/MAU 30%+)
2. **Retention** - 재방문율 증가 (D7 40%+, D30 20%+)
3. **Monetization** - 유료 전환 (Free → Pro 5%+)
4. **Community** - 커뮤니티 성장 (Discord DAU 1,000+)

---

## Daily Operations

### Morning Routine (09:00-10:00)

#### 1. Metrics Review

```bash
# Daily Dashboard Check
- MAU / DAU
- New signups
- Churn rate
- Discord activity
- Twitter mentions
- Sentry errors
```

**Dashboard URL**: https://analytics.sage.ai/daily

#### 2. Community Check

**Discord**:
- #general-chat 활동 확인
- #feedback 새 피드백 리뷰
- #help 미답변 질문 확인

**Twitter**:
- @SageAI 멘션 확인
- DM 답변
- 트렌딩 토픽 확인

#### 3. System Health

**Sentry**:
- 지난 24시간 에러 수
- Critical 이슈 확인

**CloudWatch**:
- API 응답 속도
- ECS CPU/Memory
- RDS connections

### Afternoon Routine (14:00-15:00)

#### 1. Content Publishing

- Twitter 포스트 (1-2개)
- Discord 시장 업데이트
- 블로그 포스트 (주 2회)

#### 2. User Engagement

- Discord AMA (주 1회, 수요일)
- Twitter 투표/질문
- 사용자 후기 리트윗

### Evening Routine (18:00-19:00)

#### 1. Market Monitoring

- 급변 감지 (±5% BTC, ±7% ETH)
- Fear & Greed 변화
- 주요 뉴스 체크

#### 2. Proactive Alerts

- 급변 시 Discord 알림
- Twitter 긴급 분석 포스트
- PWA Push 발송

---

## Weekly Operations

### Monday: Week Planning

- [ ] 주간 목표 설정 (MAU, 가입자, Discord 활동)
- [ ] 콘텐츠 캘린더 리뷰
- [ ] 이벤트 계획 (챌린지, 프로모션)

### Tuesday: User Feedback Review

- [ ] #feedback 채널 정리
- [ ] Top 3 피드백 선정
- [ ] 개발팀과 우선순위 논의

### Wednesday: AMA Session (Discord)

**Time**: 20:00-21:00 KST

**Format**:
```
@everyone

🎙️ Weekly AMA with Wallet Buffett 🎙️

오늘 밤 8시, 월렛 버핏과 함께하는 Q&A 시간!

질문 있으신 분들은 #wallet-buffett-qa에 남겨주세요.

See you there! 🚀
```

**Example Questions**:
- "지금 비트코인 매수 타이밍인가요?"
- "섀도우 포트폴리오 수익률이 마이너스인데 어떻게 하나요?"
- "알트코인 추천해주세요"

### Thursday: Content Creation Day

- [ ] 블로그 포스트 작성 (1개)
- [ ] Twitter 스레드 준비 (주말용)
- [ ] YouTube 스크립트 (Phase 2+)

### Friday: Week Review

- [ ] 주간 지표 리뷰
  - MAU / DAU 달성률
  - Conversion rate
  - Churn rate
  - Discord activity
- [ ] Next week 액션 아이템

---

## Monthly Operations

### Month Start (1일)

- [ ] 월간 목표 설정
- [ ] 이벤트 캘린더 확정
- [ ] 마케팅 예산 배정

### Mid-Month (15일)

- [ ] 진행 상황 체크
- [ ] 필요 시 전략 조정
- [ ] 다음 달 계획 초안

### Month End (30일)

- [ ] 월간 리포트 작성
- [ ] 팀 회고 (Retrospective)
- [ ] 다음 달 OKR 설정

---

## Events & Campaigns

### Monthly Investment Challenge

**Concept**: 섀도우 포트폴리오 수익률 경쟁

**Timeline**: 매월 1일 - 말일

**Rules**:
1. 참가자는 Discord #challenges 채널에 등록
2. 월렛 버핏 조언 기반 섀도우 트레이드만 인정
3. 최고 수익률 3명에게 보상

**Prizes**:
- 🥇 1st: Pro 플랜 1년 무료
- 🥈 2nd: Pro 플랜 6개월 무료
- 🥉 3rd: Pro 플랜 3개월 무료

**Announcement (Discord)**:
```
@everyone

🏆 Monthly Investment Challenge 🏆

This month's theme: "Buy the Fear, Sell the Greed"

Rules:
1. Sign up in this channel
2. Use Wallet Buffett's advice
3. Track with Shadow Portfolios

Top 3 ROI winners get Pro Plan FREE!

Join now: 👇
```

### Weekly Twitter Engagement

#### Monday: Market Recap

```
📊 Last Week in Crypto

BTC: $43,250 → $45,100 (+4.3%)
ETH: $2,280 → $2,450 (+7.5%)
Fear & Greed: 25 → 35 (Fear → Neutral)

What's your play this week? 🤔

Vote below! 👇
```

#### Wednesday: Wallet Buffett Wisdom

```
💡 Wallet Buffett says:

"자네, 시장이 공포에 질렸을 때가
바로 기회일세."

BTC Fear & Greed: 25 (Extreme Fear)

Historically, buying at this level has led to
+47% average returns in 6 months.

Are you buying the fear? 🤔
```

#### Friday: User Spotlight

```
🌟 User Spotlight 🌟

Meet @john_doe, who used Wallet Buffett's advice
to achieve +23% returns this month!

Here's his strategy:
- Buy when Fear & Greed < 30
- DCA (Dollar-Cost Averaging)
- HODL for 3+ months

Want to be featured? Share your wins in our Discord!
```

### Quarterly Events

#### Q1: Genesis Launch Party (Virtual)

**Date**: Launch Day

**Platform**: Discord + Twitter Spaces

**Agenda**:
- CEO Welcome
- Product Demo
- AMA with team
- Giveaway (Pro 플랜 10개)

#### Q2: "Invest Like Buffett" Webinar

**Date**: TBD

**Speaker**: 암호화폐 전문가 초대

**Topic**: 워렌 버핏 투자 철학을 암호화폐에 적용하기

#### Q3: Beta Anniversary Event

**Date**: 3 months after launch

**Celebration**:
- 베타 테스터 감사 이벤트
- 섀도우 포트폴리오 Top 10 공개
- Special discount (Pro 플랜 50% off)

---

## Community Management

### Discord Moderation

#### Response Time SLA

| Channel | Response Time |
|---------|--------------|
| #help | < 2시간 |
| #feedback | < 24시간 |
| #general-chat | Best effort |

#### Moderation Guidelines

**Allowed**:
- 투자 토론
- Sage.ai 피드백
- 시장 분석 공유

**Not Allowed**:
- Spam
- 직접적 투자 권유 ("Buy X coin now!")
- 타 서비스 홍보
- 욕설, 비방

**Warning → Mute (1일) → Kick → Ban**

### Twitter Engagement

#### Reply Strategy

**Positive Mentions**:
```
User: "Sage.ai is amazing! Just made my first shadow portfolio"
Reply: "That's awesome! 🎉 Let us know how it performs. We're here to help!"
```

**Feature Requests**:
```
User: "Can you add Dogecoin support?"
Reply: "Thanks for the suggestion! We're focusing on the top 6 coins for MVP,
but Dogecoin is on our roadmap for Q2. Stay tuned!"
```

**Complaints**:
```
User: "AI gave me wrong price data"
Reply: "Oh no! That shouldn't happen. Can you DM us the details?
We take accuracy very seriously and want to fix this ASAP."
```

---

## User Retention Strategies

### Email Campaigns

#### Day 0: Welcome Email

```
Subject: Welcome to Sage.ai! 🎉

Hi [Name],

Welcome! Here's how to get the most out of Sage.ai:

1. Chat with Wallet Buffett
2. Enable notifications
3. Try Shadow Portfolios

Happy investing!
```

#### Day 3: Activation Nudge

```
Subject: Try asking Wallet Buffett this...

Hi [Name],

Quick question for Wallet Buffett:

"What do you think about Bitcoin right now?"

[Click to chat] →

See what he says!
```

#### Day 7: Feature Discovery

```
Subject: Did you know? Shadow Portfolios track AI recommendations

Hi [Name],

You can track Wallet Buffett's recommendations with Shadow Portfolios!

Just click "Add to Portfolio" on any AI buy/sell signal.

[Learn more] →
```

#### Day 30: Re-engagement

```
Subject: We miss you! Here's what you missed...

Hi [Name],

You haven't visited in a while. Here's what's new:

- BTC is up 12% this month
- Fear & Greed hit "Extreme Greed" (time to be cautious?)
- 500+ investors joined our Discord

Come back and say hi! 👋

[Chat with Wallet Buffett] →
```

### Push Notifications

#### Trigger 1: Market Alert

```
Title: 🚨 BTC -5.2%
Body: Bitcoin just dropped -5.2%. Wallet Buffett says: "자네, 공포가 기회일세."
Action: Tap to analyze
```

#### Trigger 2: Portfolio Update

```
Title: 📈 Your Shadow Portfolio +8.5%
Body: Your BTC position is up 8.5% since you added it!
Action: View portfolio
```

#### Trigger 3: Inactivity (7 days)

```
Title: 💬 Wallet Buffett misses you
Body: Market moved a lot this week. Want to chat?
Action: Ask Wallet Buffett
```

---

## Monetization Tactics

### Free → Pro Conversion

#### In-App Prompts

**Trigger**: User creates 2nd shadow portfolio (Free tier limit: 1)

```
🔒 Upgrade to Pro

You've reached the limit of 1 Shadow Portfolio on Free tier.

Upgrade to Pro for:
- 3 Shadow Portfolios
- Priority alerts
- Advanced analytics

[Upgrade to Pro - $19.99/mo] →
```

**Trigger**: User asks 11th question in 1 minute (Rate limit)

```
⏳ Slow down there, investor!

You've hit the Free tier limit of 10 questions per minute.

Upgrade to Pro for unlimited chats.

[Upgrade to Pro - $19.99/mo] →
```

#### Exit Intent (Web)

**Trigger**: User moves mouse to close tab

```
Wait! Before you go...

Try Pro FREE for 7 days

- Unlimited chats
- 3 Shadow Portfolios
- No credit card required

[Start Free Trial] →
```

---

## Analytics & Reporting

### Daily Metrics

```sql
-- Daily Active Users (DAU)
SELECT COUNT(DISTINCT user_id)
FROM messages
WHERE DATE(created_at) = CURRENT_DATE;

-- New Signups
SELECT COUNT(*)
FROM users
WHERE DATE(created_at) = CURRENT_DATE;

-- Churn Rate (D30)
SELECT
  COUNT(DISTINCT u.id) AS churned_users,
  (COUNT(DISTINCT u.id) * 100.0 / total.count) AS churn_rate
FROM users u
LEFT JOIN messages m ON u.id = m.user_id AND m.created_at > NOW() - INTERVAL '30 days'
CROSS JOIN (SELECT COUNT(*) AS count FROM users WHERE created_at < NOW() - INTERVAL '30 days') total
WHERE u.created_at < NOW() - INTERVAL '30 days'
  AND m.id IS NULL;
```

### Weekly Report

**Format**:
```
Week [N] Report (YYYY-MM-DD ~ YYYY-MM-DD)

📊 Key Metrics:
- MAU: 1,250 (+15% WoW)
- DAU: 375 (DAU/MAU: 30%)
- New Signups: 150 (+10% WoW)
- Churn Rate: 5% (-1% WoW)

💰 Monetization:
- Pro Conversions: 5 (conversion rate: 3.3%)
- MRR: $99.95

💬 Community:
- Discord Members: 450 (+50 WoW)
- Discord DAU: 120 (DAU/Members: 26.7%)
- Twitter Followers: 850 (+100 WoW)

📝 Top Feedback:
1. "Add more coins (ADA, DOT)"
2. "Mobile app needed"
3. "Love the shadow portfolio feature!"

🐛 Critical Bugs:
- None this week ✅

🎯 Next Week Focus:
- Launch Monthly Investment Challenge
- Publish blog post on Shadow Portfolios
- Improve onboarding flow (target: +20% activation)
```

---

## Incident Response

### Severity Levels

| Level | Description | Response Time | Escalation |
|-------|-------------|--------------|------------|
| **P0** | Service down, data loss | Immediate | CEO + CTO |
| **P1** | Major feature broken (chat, portfolio) | < 1 hour | CTO |
| **P2** | Minor feature broken (notifications) | < 4 hours | Tech Lead |
| **P3** | Cosmetic issues | < 24 hours | Team discretion |

### Communication Template

#### P0/P1 Incident

**Discord #announcements**:
```
⚠️ Service Issue

We're currently experiencing [issue description].

Our team is actively working on a fix.

ETA: [time]

We'll update you every 30 minutes.

Thank you for your patience!
```

**Twitter**:
```
🛠️ We're aware of an issue affecting [feature].

Our team is on it and working on a fix.

We'll keep you updated. Sorry for the inconvenience!
```

**Resolution Announcement**:
```
✅ Issue Resolved

[Feature] is now back to normal.

What happened: [brief explanation]
What we did: [fix description]

Thank you for your patience!
```

---

**문서 끝**

_"Between the zeros and ones"_
