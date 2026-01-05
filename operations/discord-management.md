# Sage.ai Discord Community Management

```
Document Info
├── Version: 1.0
├── Last Updated: 2025-12-22
├── Author: Sam
├── Status: Production Ready
└── Audience: Community Managers, Operations Team
```

---

## 1. Discord Server Overview

### 1.1 Server Purpose

```
Sage.ai Discord Server
├── Primary Goals
│   ├── Community Building: Crypto investors gathering place
│   ├── Product Feedback: Direct user voice channel
│   ├── Market Alerts: Real-time AI analysis notifications
│   └── Support: User help and onboarding
│
└── Core Value Proposition
    └── "AI 멘토와 함께하는 투자 커뮤니티"
```

### 1.2 Target Audience

| Segment | Description | Needs |
|---------|-------------|-------|
| Crypto Beginners | First-time investors | Education, guidance, safe space |
| Active Traders | Daily/weekly traders | Market alerts, quick analysis |
| Long-term Holders | HODLers | Macro insights, patience reinforcement |
| AI Enthusiasts | Tech-savvy users | Product updates, feature requests |

---

## 2. Channel Structure

### 2.1 Complete Channel Map

```
Sage.ai Discord Server
│
├── INFORMATION
│   ├── #welcome
│   │   └── Server intro, rules, role selection
│   ├── #announcements
│   │   └── Official updates (admin only)
│   ├── #rules
│   │   └── Community guidelines
│   └── #faq
│       └── Common questions, self-service
│
├── MARKET ALERTS (Core Feature)
│   ├── #btc-alerts
│   │   └── Bitcoin price movements, AI analysis
│   ├── #eth-alerts
│   │   └── Ethereum updates
│   ├── #altcoin-alerts
│   │   └── SOL, BNB, DOGE, XRP alerts
│   ├── #fear-greed-index
│   │   └── Daily Fear & Greed updates
│   └── #breaking-news
│       └── Major market events
│
├── COMMUNITY
│   ├── #general
│   │   └── Free chat, introductions
│   ├── #market-discussion
│   │   └── Price talk, predictions, opinions
│   ├── #wallet-buffett-quotes
│   │   └── AI mentor wisdom, daily quotes
│   ├── #my-portfolio
│   │   └── Share shadow portfolio results
│   └── #memes
│       └── Crypto memes, fun content
│
├── LEARNING
│   ├── #crypto-101
│   │   └── Beginner education
│   ├── #investment-strategies
│   │   └── DCA, risk management, etc.
│   └── #ask-anything
│       └── Q&A with community
│
├── PRODUCT
│   ├── #feature-requests
│   │   └── User suggestions
│   ├── #bug-reports
│   │   └── Issue reporting
│   ├── #beta-testers
│   │   └── Beta access, early features
│   └── #changelog
│       └── Product updates
│
├── SUPPORT
│   ├── #help
│   │   └── General assistance
│   └── #ticket-support
│       └── Private ticket system
│
└── VOICE
    ├── Lounge
    │   └── Casual voice chat
    └── AMA Room
        └── Live sessions with team
```

### 2.2 Channel Permissions Matrix

| Channel | @everyone | Member | Verified | Moderator | Admin |
|---------|-----------|--------|----------|-----------|-------|
| #welcome | Read | Read | Read | Read/Write | Full |
| #announcements | Read | Read | Read | Read | Write |
| #general | - | Read/Write | Read/Write | Full | Full |
| #btc-alerts | - | Read | Read | Read | Write |
| #feature-requests | - | Read | Read/Write | Full | Full |
| #beta-testers | - | - | - | Read/Write | Full |
| #help | - | Read/Write | Read/Write | Full | Full |

---

## 3. Role System

### 3.1 Role Hierarchy

```
Role Structure
│
├── Admin (Red)
│   ├── Server owner, full permissions
│   └── Sam, Core team
│
├── Moderator (Orange)
│   ├── Channel management, mute/kick
│   └── Community moderators
│
├── Team (Blue)
│   ├── Official Sage.ai team
│   └── Product, Dev, Marketing
│
├── Beta Tester (Purple)
│   ├── Early access to features
│   └── Private beta channels
│
├── Verified (Green)
│   ├── Completed onboarding
│   ├── Access to all community channels
│   └── Linked Sage.ai account
│
├── Member (Gray)
│   ├── Basic access
│   └── Limited channels until verified
│
└── @everyone (Default)
    └── Read-only access to info channels
```

### 3.2 Role Acquisition

| Role | How to Get | Auto/Manual |
|------|------------|-------------|
| Member | Join server | Auto |
| Verified | React to #welcome + Link account | Auto (bot) |
| Beta Tester | Apply in #beta-testers | Manual approval |
| Moderator | Invitation from admin | Manual |
| Team | Internal team only | Manual |
| Admin | Owner only | Manual |

---

## 4. Bot Integration

### 4.1 Required Bots

```
Bot Stack
│
├── Sage Bot (Custom)
│   ├── Market alerts from Lambda
│   ├── Fear & Greed updates
│   ├── Daily Wallet Buffett quotes
│   ├── Account linking verification
│   └── /ask command (mini AI chat)
│
├── MEE6 or Carl-bot
│   ├── Auto-moderation
│   ├── Welcome messages
│   ├── Role reaction
│   └── Leveling system
│
├── Ticket Tool
│   └── Private support tickets
│
└── Dyno or Wick
    ├── Anti-raid protection
    ├── Anti-spam
    └── Logging
```

### 4.2 Sage Bot Commands

```typescript
interface SageBotCommands {
  // Market Commands
  "/price <symbol>": "Get current price (BTC, ETH, etc.)";
  "/fear": "Current Fear & Greed Index";
  "/alerts on/off": "Toggle personal alert DMs";

  // AI Commands
  "/ask <question>": "Quick question to Wallet Buffett (limited)";
  "/quote": "Random Wallet Buffett wisdom";

  // Account Commands
  "/link": "Link Discord to Sage.ai account";
  "/portfolio": "View shadow portfolio summary";

  // Utility
  "/help": "Bot command list";
  "/invite": "Invite link to server";
}
```

### 4.3 Alert Webhook Format

```typescript
// Lambda -> Discord Webhook payload
interface DiscordAlertPayload {
  embeds: [{
    title: string;        // "BTC Alert: -5.2%"
    description: string;  // AI analysis summary
    color: number;        // Red (bearish) or Green (bullish)
    fields: [
      { name: "Current Price", value: "$67,500", inline: true },
      { name: "24h Change", value: "-5.2%", inline: true },
      { name: "Fear & Greed", value: "25 (Extreme Fear)", inline: true }
    ];
    footer: {
      text: "Wallet Buffett AI Analysis";
      icon_url: "https://sage.ai/avatar.png";
    };
    timestamp: string;    // ISO timestamp
  }];
  components: [{
    type: 1,
    components: [{
      type: 2,              // Button
      style: 5,             // Link
      label: "Full Analysis on Sage.ai";
      url: "https://app.sage.ai/chat/new?context=market_alert&symbol=BTC";
    }]
  }];
}
```

---

## 5. Content Strategy

### 5.1 Daily Content Calendar

| Time (KST) | Channel | Content | Responsible |
|------------|---------|---------|-------------|
| 09:00 | #fear-greed-index | Daily Fear & Greed update | Sage Bot |
| 09:30 | #wallet-buffett-quotes | Daily wisdom quote | Sage Bot |
| 12:00 | #market-discussion | Lunch market summary | Moderator |
| 18:00 | #crypto-101 | Educational tip of the day | Content Team |
| 21:00 | #general | Community engagement post | Moderator |
| Real-time | #btc-alerts | Price movement alerts | Sage Bot (Lambda) |

### 5.2 Weekly Content

| Day | Content | Channel |
|-----|---------|---------|
| Monday | Weekly market outlook | #market-discussion |
| Wednesday | AMA session (bi-weekly) | Voice: AMA Room |
| Friday | Weekly portfolio performance recap | #my-portfolio |
| Sunday | Week ahead preview | #announcements |

### 5.3 Wallet Buffett Quote Examples

```
Daily Wisdom Quotes (Rotation)
│
├── "자네, 시장이 공포에 질렸을 때가 바로 기회일세."
├── "단기 변동에 흔들리지 말게. 나무가 아닌 숲을 보게나."
├── "투자는 마라톤이지, 100미터 달리기가 아닐세."
├── "분할 매수의 지혜를 잊지 말게나."
├── "레버리지는 양날의 검일세. 조심하게."
├── "시장은 단기적으로는 투표 기계, 장기적으로는 저울일세."
├── "남들이 탐욕스러울 때 두려워하고, 두려워할 때 탐욕스러워지게."
└── "가격은 지불하는 것이고, 가치는 얻는 것일세."
```

---

## 6. Moderation Guidelines

### 6.1 Rules & Policies

```
Community Rules
│
├── 1. Respect & Kindness
│   ├── No personal attacks
│   ├── No discrimination
│   └── Constructive criticism only
│
├── 2. No Financial Advice
│   ├── Sage.ai provides AI analysis, not financial advice
│   ├── "DYOR" (Do Your Own Research)
│   └── No pump & dump coordination
│
├── 3. No Spam
│   ├── No promotional links without permission
│   ├── No repetitive messages
│   └── No unsolicited DMs
│
├── 4. No Scams
│   ├── Report suspicious DMs immediately
│   ├── Admins will NEVER DM first
│   └── No impersonation
│
├── 5. Stay On Topic
│   ├── Use appropriate channels
│   └── Keep discussions relevant
│
└── 6. No NSFW
    └── Family-friendly content only
```

### 6.2 Moderation Actions

| Offense | 1st | 2nd | 3rd |
|---------|-----|-----|-----|
| Off-topic | Warning | Mute 1h | Mute 24h |
| Spam | Delete + Warning | Mute 24h | Ban |
| Promotion | Delete | Mute 24h | Ban |
| Harassment | Mute 24h | Ban | - |
| Scam/Phishing | Immediate Ban | - | - |
| Impersonation | Immediate Ban | - | - |

### 6.3 Escalation Process

```
Moderation Escalation
│
├── Level 1: Auto-mod (Bot)
│   ├── Spam detection
│   ├── Link filtering
│   └── Banned word filter
│
├── Level 2: Moderator
│   ├── Manual review
│   ├── Warnings & mutes
│   └── Escalate to admin if needed
│
└── Level 3: Admin
    ├── Ban decisions
    ├── Appeals review
    └── Policy updates
```

---

## 7. Growth & Engagement

### 7.1 Onboarding Flow

```
New Member Journey
│
├── Step 1: Join Server
│   └── See #welcome, #rules
│
├── Step 2: React to Rules
│   └── Bot assigns "Member" role
│
├── Step 3: Introduction (Optional)
│   └── Post in #general
│
├── Step 4: Link Account
│   └── /link command -> Sage.ai OAuth
│   └── Bot assigns "Verified" role
│
├── Step 5: Explore
│   └── Access to all community channels
│
└── Step 6: Engage
    └── Participate in discussions, alerts
```

### 7.2 Engagement Tactics

| Tactic | Description | Goal |
|--------|-------------|------|
| Daily Alerts | Market movement notifications | Retention |
| Quote of the Day | Wallet Buffett wisdom | Brand reinforcement |
| Portfolio Sharing | #my-portfolio results | Social proof |
| AMA Sessions | Live Q&A with team | Trust building |
| Beta Access | Exclusive early features | Loyalty |
| Contests | Trading competitions (paper) | Engagement |
| Referral Rewards | Invite bonuses | Growth |

### 7.3 KPIs

| Metric | Target (MVP) | Target (6 months) |
|--------|--------------|-------------------|
| Total Members | 500 | 5,000 |
| Daily Active | 50 (10%) | 500 (10%) |
| Messages/Day | 100 | 1,000 |
| Verified Rate | 30% | 50% |
| Sage.ai Conversion | 20% | 30% |

---

## 8. Integration with Sage.ai

### 8.1 Account Linking Overview

```
Google 계정 ≠ Discord 계정 (별개 계정 연동)
│
├── Sage.ai 계정 (Google OAuth로 가입)
│   └── sam@gmail.com
│
├── Discord 계정 (별도)
│   └── sam#1234 (discord_id: 123456789)
│
└── 연동 후 DB
    └── users: {
          id: "user-123",
          email: "sam@gmail.com",      // Google에서
          discord_id: "123456789"       // Discord에서
        }
```

### 8.2 Account Linking Flow

```
회원가입 -> Discord 연동 플로우
│
├── 1. 유저가 Sage.ai 회원가입 (Google OAuth)
│
├── 2. 앱 내 설정 페이지에서 "Discord 연동" 버튼 클릭
│   └── Discord OAuth2 로그인 팝업
│
├── 3. 유저가 자신의 Discord 계정으로 로그인 & 승인
│   └── "Sage.ai가 Discord 계정에 접근하려 합니다"
│
├── 4. Discord Access Token 획득
│   └── 유저의 Discord ID, Username 저장
│
├── 5. Bot API로 역할 부여
│   └── PUT /guilds/{guild_id}/members/{user_id}/roles/{role_id}
│
└── 6. 완료 - 유저가 "Verified" 역할 획득
```

### 8.3 Discord Developer Portal Setup

```
Discord Application 설정
│
├── 1. Application 생성
│   └── https://discord.com/developers/applications
│   └── 이름: "Sage.ai"
│
├── 2. OAuth2 설정
│   ├── Redirects: https://app.sage.ai/api/auth/discord/callback
│   └── Scopes: identify, guilds.join
│
├── 3. Bot 생성
│   ├── Bot 탭 → "Add Bot"
│   ├── Token 복사 → 환경변수 저장
│   └── Privileged Gateway Intents: Server Members Intent 활성화
│
└── 4. Bot 서버 초대
    ├── OAuth2 → URL Generator
    ├── Scopes: bot
    ├── Permissions: Manage Roles
    └── 생성된 URL로 서버에 봇 초대
```

### 8.4 Database Schema

```typescript
// users 테이블에 Discord 필드 추가
export const users = pgTable('users', {
  id: uuid('id').primaryKey(),

  // Google OAuth
  email: varchar('email', { length: 255 }).notNull(),
  name: varchar('name', { length: 255 }),

  // Discord 연동 (optional)
  discordId: varchar('discord_id', { length: 50 }),
  discordUsername: varchar('discord_username', { length: 100 }),
  discordLinkedAt: timestamp('discord_linked_at'),

  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});
```

### 8.5 Backend Implementation

```typescript
// Discord OAuth callback 처리
// POST /api/auth/discord/callback

const DISCORD_BOT_TOKEN = process.env.DISCORD_BOT_TOKEN;
const GUILD_ID = 'YOUR_SERVER_ID';        // Discord 서버 ID
const VERIFIED_ROLE_ID = 'YOUR_ROLE_ID';  // Verified 역할 ID

async function handleDiscordCallback(code: string, userId: string) {
  // 1. Access Token 획득
  const tokenResponse = await fetch('https://discord.com/api/oauth2/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      client_id: process.env.DISCORD_CLIENT_ID,
      client_secret: process.env.DISCORD_CLIENT_SECRET,
      grant_type: 'authorization_code',
      code,
      redirect_uri: 'https://app.sage.ai/api/auth/discord/callback',
    }),
  });

  const { access_token } = await tokenResponse.json();

  // 2. Discord 유저 정보 획득
  const userResponse = await fetch('https://discord.com/api/users/@me', {
    headers: { Authorization: `Bearer ${access_token}` },
  });

  const discordUser = await userResponse.json();

  // 3. DB에 Discord ID 저장
  await db.update(users)
    .set({
      discordId: discordUser.id,
      discordUsername: `${discordUser.username}#${discordUser.discriminator}`,
      discordLinkedAt: new Date(),
    })
    .where(eq(users.id, userId));

  // 4. Bot API로 역할 부여
  await assignVerifiedRole(discordUser.id);

  return { success: true, discordUsername: discordUser.username };
}

// Bot API로 역할 부여
async function assignVerifiedRole(discordUserId: string) {
  const response = await fetch(
    `https://discord.com/api/guilds/${GUILD_ID}/members/${discordUserId}/roles/${VERIFIED_ROLE_ID}`,
    {
      method: 'PUT',
      headers: {
        Authorization: `Bot ${DISCORD_BOT_TOKEN}`,
        'Content-Type': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error('Failed to assign Discord role');
  }
}

// 연동 해제
async function unlinkDiscord(userId: string) {
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId),
  });

  if (user?.discordId) {
    // 역할 제거
    await fetch(
      `https://discord.com/api/guilds/${GUILD_ID}/members/${user.discordId}/roles/${VERIFIED_ROLE_ID}`,
      {
        method: 'DELETE',
        headers: { Authorization: `Bot ${DISCORD_BOT_TOKEN}` },
      }
    );

    // DB에서 Discord 정보 제거
    await db.update(users)
      .set({
        discordId: null,
        discordUsername: null,
        discordLinkedAt: null,
      })
      .where(eq(users.id, userId));
  }
}
```

### 8.6 Frontend Implementation

```typescript
// components/DiscordLinkButton.tsx

export function DiscordLinkButton() {
  const { user } = useAuth();

  const handleLink = () => {
    const params = new URLSearchParams({
      client_id: process.env.NEXT_PUBLIC_DISCORD_CLIENT_ID!,
      redirect_uri: 'https://app.sage.ai/api/auth/discord/callback',
      response_type: 'code',
      scope: 'identify guilds.join',
      state: user.id,  // CSRF 방지용
    });

    window.location.href = `https://discord.com/api/oauth2/authorize?${params}`;
  };

  const handleUnlink = async () => {
    await fetch('/api/auth/discord/unlink', { method: 'POST' });
    // Refresh user data
  };

  if (user.discordId) {
    return (
      <div className="flex items-center justify-between">
        <div>
          <span className="text-green-500">Discord 연결됨</span>
          <span className="ml-2 text-gray-500">{user.discordUsername}</span>
        </div>
        <button onClick={handleUnlink}>연동 해제</button>
      </div>
    );
  }

  return (
    <button onClick={handleLink}>
      Discord 연동하기
    </button>
  );
}
```

### 8.7 Settings Page UI

```
설정 페이지 - 계정 연동
┌─────────────────────────────────────────┐
│  계정 연동                               │
├─────────────────────────────────────────┤
│                                         │
│  Google    sam@gmail.com       연결됨    │
│                                         │
│  Discord   연결 안됨          [연동하기]  │
│                                         │
│  ─────────────────────────────────────  │
│  Discord 연동 시 커뮤니티에서            │
│  Verified 역할을 받을 수 있습니다         │
│                                         │
└─────────────────────────────────────────┘

연동 후:
┌─────────────────────────────────────────┐
│  Discord   sam#1234           연결됨    │
│                              [연동해제]  │
└─────────────────────────────────────────┘
```

### 8.8 Discord Server Permission Setup

```
채널 권한 설정
│
├── @everyone
│   ├── #welcome, #rules, #announcements: 읽기만
│   └── 다른 채널: 접근 불가
│
├── Member (서버 가입 시 자동)
│   ├── #welcome, #rules: 읽기만
│   └── 다른 채널: 읽기만 (쓰기 불가)
│
└── Verified (Sage.ai 연동 시 API로 부여)
    ├── 모든 채널: 읽기/쓰기 가능
    └── #beta-testers 제외
```

### 8.9 Environment Variables

```bash
# Discord OAuth
DISCORD_CLIENT_ID="your_client_id"
DISCORD_CLIENT_SECRET="your_client_secret"

# Discord Bot
DISCORD_BOT_TOKEN="your_bot_token"
DISCORD_GUILD_ID="your_server_id"
DISCORD_VERIFIED_ROLE_ID="your_role_id"

# Frontend
NEXT_PUBLIC_DISCORD_CLIENT_ID="your_client_id"
```

### 8.10 Deep Link Integration

```typescript
// Discord alert -> Sage.ai chat
const alertButton = {
  type: 2,           // Button
  style: 5,          // Link style
  label: "Discuss with Wallet Buffett",
  url: `https://app.sage.ai/chat/new?` +
       `context=discord_alert&` +
       `symbol=${symbol}&` +
       `change=${changePercent}`
};

// Opens new chat with context:
// "Discord에서 BTC -5.2% 알림을 보고 왔어요. 분석해주세요."
```

---

## 9. Crisis Management

### 9.1 Scenario Playbooks

```
Crisis Response Playbooks
│
├── Market Crash (>20% drop)
│   ├── 1. Immediate #announcements post
│   │   └── "시장 급락 안내 - 침착하게 대응하세요"
│   ├── 2. Pin Wallet Buffett calming message
│   ├── 3. Moderators active in #general
│   ├── 4. Disable #market-discussion (if panic)
│   └── 5. Team AMA within 24h
│
├── Server Raid
│   ├── 1. Enable slow mode (30s)
│   ├── 2. Raise verification level
│   ├── 3. Ban raid accounts
│   └── 4. Report to Discord Trust & Safety
│
├── Scam Reports
│   ├── 1. Immediate announcement warning
│   ├── 2. Ban reported accounts
│   ├── 3. Pin "Admins never DM first"
│   └── 4. Individual victim support
│
└── Product Outage
    ├── 1. #announcements status update
    ├── 2. Pin in #help
    ├── 3. Regular updates every 30min
    └── 4. Post-mortem after resolution
```

### 9.2 Communication Templates

**Market Crash Announcement:**
```
[MARKET UPDATE]

시장이 급격하게 움직이고 있습니다.

Wallet Buffett의 조언:
"자네, 패닉 셀링은 최악의 선택일세.
이런 때일수록 침착하게 자신의 투자 원칙을 지키게나."

- 충동적인 결정을 자제하세요
- 레버리지 포지션을 확인하세요
- 장기적 관점을 유지하세요

자세한 분석: https://app.sage.ai
```

**Scam Warning:**
```
[SCAM ALERT]

DM으로 투자 권유나 링크를 보내는 사람을 조심하세요!

기억하세요:
- Admin/Mod는 절대 먼저 DM하지 않습니다
- 공식 링크: sage.ai만 사용하세요
- 의심되면 즉시 신고하세요

의심스러운 DM을 받으셨다면 스크린샷과 함께 #help에 신고해주세요.
```

---

## 10. Launch Checklist

### 10.1 Pre-Launch

```
Pre-Launch Checklist
│
├── Server Setup
│   ├── [ ] All channels created
│   ├── [ ] Roles configured
│   ├── [ ] Permissions verified
│   ├── [ ] Welcome message written
│   └── [ ] Rules posted
│
├── Bots
│   ├── [ ] Sage Bot deployed & tested
│   ├── [ ] MEE6/Carl-bot configured
│   ├── [ ] Ticket system ready
│   └── [ ] Anti-raid protection active
│
├── Content
│   ├── [ ] #welcome banner/embed
│   ├── [ ] #faq populated
│   ├── [ ] First week content scheduled
│   └── [ ] Wallet Buffett quotes loaded
│
├── Team
│   ├── [ ] Moderators assigned & trained
│   ├── [ ] Admin backup confirmed
│   └── [ ] Response templates ready
│
└── Integration
    ├── [ ] Lambda -> Discord webhook tested
    ├── [ ] Account linking flow tested
    └── [ ] Deep links working
```

### 10.2 Post-Launch

| Timeframe | Action |
|-----------|--------|
| Day 1 | Monitor closely, welcome each member |
| Week 1 | Daily engagement posts, gather feedback |
| Week 2 | First AMA session |
| Month 1 | Review KPIs, adjust strategy |
| Month 3 | Community feedback survey |

---

## 11. Tools & Resources

### 11.1 Recommended Tools

| Tool | Purpose | Cost |
|------|---------|------|
| Discord Server | Community platform | Free |
| MEE6 | Moderation, leveling | Free tier |
| Carl-bot | Reaction roles | Free |
| Ticket Tool | Support tickets | Free |
| Sapphire | Custom bot framework | Free (self-host) |
| Canva | Graphics, banners | Free tier |

### 11.2 Asset Requirements

```
Discord Assets
│
├── Server Icon
│   └── 512x512 PNG, Sage.ai logo
│
├── Server Banner
│   └── 960x540 PNG, branded
│
├── Invite Splash
│   └── 1920x1080 PNG
│
├── Role Icons (Nitro)
│   └── 64x64 each role
│
├── Emoji Pack
│   ├── :sage: - Logo
│   ├── :buffett: - Wallet Buffett
│   ├── :bullish: - Green up
│   ├── :bearish: - Red down
│   ├── :fear: - Fear emoji
│   └── :greed: - Greed emoji
│
└── Stickers (Optional)
    └── Wallet Buffett expressions
```

---

## Appendix A: Invite Link Strategy

```
Invite Links
│
├── Main Invite (Vanity URL)
│   └── discord.gg/sageai
│
├── Tracking Links
│   ├── WhyBitcoinFallen.com -> discord.gg/sageai-wbf
│   ├── Landing page -> discord.gg/sageai-web
│   ├── Twitter -> discord.gg/sageai-x
│   └── App in-product -> discord.gg/sageai-app
│
└── Analytics
    └── Track source conversion in Discord insights
```

## Appendix B: Moderator Training

```
Moderator Training Topics
│
├── 1. Platform Basics
│   ├── Discord features
│   ├── Bot commands
│   └── Admin panel
│
├── 2. Community Guidelines
│   ├── Rules enforcement
│   ├── Escalation process
│   └── Ban appeals
│
├── 3. Sage.ai Product
│   ├── Feature overview
│   ├── Common user questions
│   └── Account linking support
│
├── 4. Crisis Response
│   ├── Raid handling
│   ├── Scam reports
│   └── Market panic
│
└── 5. Tone & Voice
    ├── Friendly, helpful
    ├── Professional
    └── Wallet Buffett personality alignment
```

---

```
Document Footer
├── Last Updated: 2025-12-22
├── Version: 1.0
├── Status: Production Ready
└── Maintainer: Sam (dev@5010.tech)
```
