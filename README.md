# FunAgency MiniApp Mockup

> Visual + functional reference for the FunAgency Telegram MiniApp. Hand this to the dev team as the source of truth for UX, business rules, and tech requirements.

Open `index.html` in any modern browser. No build step, no dependencies — single HTML file.

```bash
# Local preview
python3 -m http.server 8000
# then open http://localhost:8000
```

---

## Table of Contents
1. [What's in this mockup](#whats-in-this-mockup)
2. [Business rules (CORE)](#business-rules-core)
3. [6 Scenario tabs](#6-scenario-tabs)
4. [State Demo Toggle](#state-demo-toggle)
5. [Network filter logic](#network-filter-logic)
6. [Withdrawal rules (USDT TRC20)](#withdrawal-rules-usdt-trc20)
7. [Tech stack (suggested)](#tech-stack-suggested)
8. [Theme handling](#theme-handling)
9. [DB Schema](#db-schema)
10. [Anti-fraud](#anti-fraud-rules-production-required)
11. [Open questions](#open-questions-for-product)

---

## What's in this mockup

- Single `index.html` (~175KB), vanilla HTML+CSS+JS — no framework, no build.
- 3-column layout for review: **UX notes** (left) · **phone mockup** (center) · **tech notes** (right).
- Multilingual: **VI / RU / EN** — switch live via header buttons.
- 6 scenario tabs + a 7th button for the Settings modal.
- 4 state demo buttons (Normal / Loading / Empty / Error) — see below.

---

## Business rules (CORE)

> These rules are the contract. Don't change without product sign-off.

### Tier system
| Tier    | Active refs | Lifetime % | Boost |
|---------|-------------|------------|-------|
| Starter | 0–2         | 0.5%       | —     |
| Growth  | 3–9         | 1.0%       | —     |
| VIP     | 10+         | 1.5%       | +0.5% |

### Activation (when commission starts counting)
- Ref must spend **≥ $10,000** lifetime AND active **≥ 2 months**.
- Until then ref shows as "tracking" (no $ paid yet).

### Hold + claw-back
- Earned commission **locked 14 days** before becoming withdrawable (anti-fraud).
- Chargeback on a ref → **proportional claw-back** from user balance.

### Tier downgrade
- Active refs drop below tier threshold for **30 days** → red warning banner shows on Home.
- After 7-day grace → auto-downgrade to lower tier.
- New rate applies only to NEW commissions (existing locked commissions keep old rate).

### Bonuses
- **Speed Bonus**: 5 active refs in 90 days → **$1,000 cash** one-shot.
- **Boost**: ×1.5 on new commissions for 12 days (event-based, configurable per user).
- **Streak**: daily share check-in → **+$50** bonus on 7-day streak.

---

## 6 Scenario tabs

| Tab          | Purpose                       | Key UI                                                                                               |
|--------------|-------------------------------|------------------------------------------------------------------------------------------------------|
| 🏠 Home      | Hub: earnings, tier, activity | Tier-down warning, animated hero $620, streak card, tier progress bar w/ Boost, Speed Bonus, 2x2 stat grid, LIVE activity feed, ref link card, native TG share |
| 🌐 Network   | Direct refs (1-tier flat)     | Search input, 6 filter chips, sort cycle, per-ref card w/ start date, Active/Inactive split, Load-more pagination ("Page 1/7 · 127 total") |
| 📊 Stats     | Trends + leaderboard          | 6-month bar chart, MoM growth %, Top-10 leaderboard (gold/silver/bronze)                              |
| 💰 Wallet    | Balance + tx history          | Available + Pending split, infinite TX scroll, YTD summary card                                       |
| 💸 Withdraw  | USDT TRC20 only               | Hero w/ network label, USDT TRC20 limits panel (Min/Max/Fee/KYC), Quick chips $100/$300/$500/MAX, payout settings, recent withdrawals |
| 🔗 Share     | 3 share channels              | QR code, Copy button, Native TG share, pre-written message templates                                  |
| ⚙️ Settings  | Account / payout / etc.       | Modal opened from header. Account info, USDT TRC20 wallet, 3 notification toggles, language picker, support |

---

## State Demo Toggle

Header has 4 buttons that switch the active scenario between visual states. Dev MUST implement all 4 for every screen.

| State      | When to show                                                                 |
|------------|------------------------------------------------------------------------------|
| ✅ Normal   | Default — data loaded successfully                                          |
| ⏳ Loading  | Skeleton shimmer + spinner while fetching `/api/...`                         |
| 📭 Empty    | First-time user, no refs/data yet. Copy is **per-tab** (see `emptyConfig` in `<script>`) |
| ⚠️ Error    | API failure / network drop. Show error code (e.g. `ERR_NETWORK · /api/v1/stats · 504`) + Retry button + "Report issue" |

Per-tab empty state messaging examples:
- Home → 🌱 "Welcome aboard! Invite your first ref to start earning 1% lifetime."
- Network → 👥 "Your network is empty. Share your link to invite the first ref."
- Stats → 📊 "Need 7+ days of data to render chart. Keep inviting."
- Wallet → 💸 "No transactions yet. Commissions appear when refs start spending."
- Withdraw → 🔒 "Minimum withdraw is $20. You have $0 available."
- Share → 🔗 "Your link is ready. Pick QR / copy / native share to start."

---

## Network filter logic

**Search input** — free-text matches `@username`, `$amount`, tier name, or join date.

**Filter chips** (only one active at a time):
| Chip            | Logic                                                            |
|-----------------|------------------------------------------------------------------|
| `All`           | Every ref (default)                                              |
| `🟢 Active`     | Currently spending in last 30 days                               |
| `⚪ Inactive`   | Paused for ≥ 30 days                                             |
| `🏆 Top earner` | Top 3 by lifetime commission                                     |
| `✨ New (30d)`  | Joined in last 30 days, active                                   |
| `⚠️ At risk`    | Status `inactive` (warning to re-engage)                          |

**Sort cycle**: `↓ $ earnings` → `↑ $ earnings` → `🕒 Newest` → `A → Z` → loop.

**Pagination**: 20 per page. Load-more button appends. Mockup demonstrates with sample "Showing 7 / 127 · Page 1/7".

---

## Withdrawal rules (USDT TRC20)

| Setting       | Value                                                |
|---------------|------------------------------------------------------|
| Network       | **TRON (TRC20 only)** — no ERC20/BEP20 in v1         |
| Currency      | USDT                                                 |
| Min amount    | $20 / withdrawal                                     |
| Max amount    | $10,000 / day (rolling 24h)                          |
| Network fee   | ~$1 USDT (deducted from amount, NOT added)           |
| Processing    | 2–15 min on-chain (after manual approval)            |
| KYC threshold | Required for withdrawal ≥ **$1,000**                 |
| Auto-payout   | Optional toggle, triggers when balance ≥ $200        |
| Cooldown      | 7 days after first signup before first withdrawal    |
| Manual review | Required for amounts > $1,000 (additional ~24h hold) |

---

## Tech stack (suggested)

- **Frontend**: Vanilla JS / Vue 3 / React 18 + Tailwind CSS
- **Charts**: Chart.js or ApexCharts (bar chart for Stats)
- **Backend**: Node.js + Express + Postgres 15+
- **Auth**: Telegram `initData` validated with HMAC-SHA256 + `BOT_TOKEN`
- **Hosting**: Cloudflare Pages / Vercel / Netlify (HTTPS required by Telegram)
- **Telegram SDK**: `window.Telegram.WebApp` — use `MainButton`, `HapticFeedback`, `themeParams`, `openTelegramLink`, `expand()`, `ready()`
- **Push notifications**: Bot sends via Telegram Bot API (no service worker needed)
- **Deep link pattern**: `?startapp=ref_USERID` — bot AND MiniApp both receive args at start

---

## Theme handling

Mockup ships in dark theme for visual consistency, but production should detect Telegram theme:

```js
function applyTgTheme() {
  const tg = window.Telegram?.WebApp;
  if (!tg?.themeParams) return;
  const tp = tg.themeParams;
  const root = document.documentElement.style;
  if (tp.bg_color)            root.setProperty('--tg-bg', tp.bg_color);
  if (tp.secondary_bg_color)  root.setProperty('--tg-bg-secondary', tp.secondary_bg_color);
  if (tp.text_color)          root.setProperty('--tg-text', tp.text_color);
  if (tp.hint_color)          root.setProperty('--tg-text-hint', tp.hint_color);
  if (tp.link_color)          root.setProperty('--tg-link', tp.link_color);
  if (tp.button_color)        root.setProperty('--tg-button', tp.button_color);
  // ... see <script> in index.html for full list
  tg.ready();
  tg.expand();
}
applyTgTheme();
window.Telegram?.WebApp?.onEvent?.('themeChanged', applyTgTheme);
```

User-toggled theme overrides (light/dark) propagate via `themeChanged` event.

---

## DB Schema

Essential tables. Production should add audit logs, indexes on `tg_id`, `referrer_id`, `(month, recipient_id)`, etc.

```sql
-- USERS
CREATE TABLE users (
  id            BIGSERIAL PRIMARY KEY,
  tg_id         BIGINT UNIQUE NOT NULL,
  username      TEXT,
  referrer_id   BIGINT REFERENCES users(id),     -- 1-tier direct only
  tier          TEXT CHECK (tier IN ('starter','growth','vip')) DEFAULT 'starter',
  wallet_addr   TEXT,                             -- USDT TRC20 address
  kyc_status    TEXT DEFAULT 'none',              -- none|pending|verified|rejected
  joined_at     TIMESTAMPTZ DEFAULT now()
);

-- CLIENTS (people referred — tracked for spend)
CREATE TABLE clients (
  id            BIGSERIAL PRIMARY KEY,
  referrer_id   BIGINT NOT NULL REFERENCES users(id),
  spend_total   NUMERIC(12,2) DEFAULT 0,          -- lifetime spend
  spend_mo      NUMERIC(12,2) DEFAULT 0,          -- current month spend (rolling)
  status        TEXT CHECK (status IN ('tracking','active','paused','churned')) DEFAULT 'tracking',
  activated_at  TIMESTAMPTZ                       -- when reached $10K + 2mo (commission starts)
);

-- COMMISSIONS (one row per month per client)
CREATE TABLE commissions (
  id            BIGSERIAL PRIMARY KEY,
  client_id     BIGINT NOT NULL REFERENCES clients(id),
  recipient_id  BIGINT NOT NULL REFERENCES users(id),  -- = client.referrer_id (denormalized for query speed)
  amount        NUMERIC(12,2) NOT NULL,
  rate          NUMERIC(5,4) NOT NULL,                  -- 0.005 / 0.01 / 0.015
  month         DATE NOT NULL,                          -- YYYY-MM-01
  status        TEXT CHECK (status IN ('locked','available','clawed_back')) DEFAULT 'locked',
  earned_at     TIMESTAMPTZ DEFAULT now(),
  locked_until  TIMESTAMPTZ NOT NULL                    -- earned_at + 14 days
);

-- PAYOUTS (USDT TRC20 withdrawals)
CREATE TABLE payouts (
  id            BIGSERIAL PRIMARY KEY,
  user_id       BIGINT NOT NULL REFERENCES users(id),
  amount        NUMERIC(12,2) NOT NULL,
  fee           NUMERIC(8,2) DEFAULT 1,                 -- ~$1 TRC20 network fee
  net_amount    NUMERIC(12,2) GENERATED ALWAYS AS (amount - fee) STORED,
  tx_hash       TEXT,                                    -- TRON tx hash (filled when broadcast)
  status        TEXT CHECK (status IN ('pending','approved','processing','done','failed','rejected')) DEFAULT 'pending',
  requested_at  TIMESTAMPTZ DEFAULT now(),
  completed_at  TIMESTAMPTZ
);

-- BOOSTS (Speed Bonus, ×1.5 events, etc.)
CREATE TABLE boosts (
  id            BIGSERIAL PRIMARY KEY,
  user_id       BIGINT NOT NULL REFERENCES users(id),
  type          TEXT NOT NULL,                           -- speed_bonus | rate_boost | streak
  multiplier    NUMERIC(4,2),                            -- 1.5 for ×1.5 boost
  reward        NUMERIC(10,2),                           -- $1000 for speed bonus
  expires_at    TIMESTAMPTZ,
  claimed_at    TIMESTAMPTZ
);
```

---

## Anti-fraud rules (production-required)

- Block self-referral: enforce `users.referrer_id != users.id` at signup.
- Capture **IP + device fingerprint** at signup; flag if N users share fingerprint.
- **Cooldown 7 days** after first signup before first withdrawal.
- **Manual review** for withdrawals > $1,000 (additional ~24h hold).
- **Claw-back** proportional to chargeback ratio.
- Rate limit: max **3 withdrawal requests / day / user**.
- Alert on rapid tier upgrade (e.g. 10 active refs in <7 days).

---

## Open questions for product

> Mockup intentionally leaves these undecided — please confirm before dev sprint planning.

1. **Currency display**: `$` shown — is it USD or USDT? Show local currency (VND/RUB) equivalent?
2. **KYC vendor**: Sumsub / Onfido / Veriff?
3. **Push notifications**: Telegram Bot API only, or also web push for users with `enable_notifications=false`?
4. **Bot ↔ MiniApp split**: which actions stay in the bot vs. MiniApp? (See bot mockup at `localhost:8091`.)
5. **Squad / team feature**: planned in roadmap, or scope-cut for v1?
6. **Tier downgrade timing**: confirmed 30d inactive + 7d grace? Or stricter/looser?
7. **Activation rule** ($10K + 2mo): is this final, or A/B test variants planned?
8. **Auto-payout default**: ON or OFF for new users?

---

## Files

- `index.html` — full mockup, single file, no dependencies (~175KB)
- `README.md` — this file

## Related (not in this repo)

- Bot mockup — `localhost:8091`
- Landing page mockup — `localhost:8092`

---

**Last updated**: 2026-05-03 · **Author**: @tony9X
