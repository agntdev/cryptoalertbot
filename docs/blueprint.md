# CryptoAlertBot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that lets users privately track custom crypto watchlists and receive price-threshold or percent-change alerts. Users manage watchlists via inline buttons or free-text tickers, with optional daily summaries and quiet hours. Owner dashboard shows user analytics and alert statistics.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Retail crypto traders and holders
- Non-technical users preferring simple inline UI

## Success criteria

- Users can add/remove crypto tickers to their watchlist
- Price-threshold and percent-change alerts trigger correctly with cooldowns
- Daily summaries are delivered at user-selected times with quiet hours respected
- Owner dashboard shows accurate user and alert statistics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu for onboarding or returning users
- **/price** (command, actor: user, command: /price) — Check current price of a specific coin or list of all tracked coins
- **/dashboard** (command, actor: owner, command: /dashboard) — Show owner analytics dashboard with user and alert statistics
- **Add common coin** (button, actor: user, callback: add_common_coin) — Open quick-add menu for common coins like BTC, ETH, TON
- **Manage watchlist** (button, actor: user, callback: manage_watchlist) — View and edit current watchlist entries
- **Set daily summary** (button, actor: user, callback: set_daily_summary) — Configure or disable morning summary notifications
- **Configure quiet hours** (button, actor: user, callback: configure_quiet_hours) — Set time window for suppressing alert notifications

## Flows

### onboarding
_Trigger:_ /start

1. Ask for timezone with quick options
2. Suggest default quiet hours window
3. Confirm preferences

_Data touched:_ User profile

### add_watchlist_entry
_Trigger:_ add_common_coin or free-text ticker

1. Validate ticker
2. Add to watchlist
3. Show confirmation with price

_Data touched:_ Watchlist entry

### price_threshold_alert
_Trigger:_ Price crosses threshold

1. Check threshold condition
2. Send alert notification
3. Set cooldown

_Data touched:_ Alert rule, Event record

### percent_change_alert
_Trigger:_ Percent change in window

1. Calculate change
2. Check against threshold
3. Send alert notification
4. Set cooldown

_Data touched:_ Alert rule, Event record

### daily_summary
_Trigger:_ User's configured time

1. Check if in quiet hours
2. Compile summary data
3. Send digest message

_Data touched:_ Event record

### manage_alerts
_Trigger:_ Manage watchlist entry

1. Show coin details
2. Edit threshold/percent rules
3. Update watchlist

_Data touched:_ Watchlist entry

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Telegram user settings and preferences
  - fields: Telegram user id, Display name, Timezone, Quiet hours window, Daily summary time, Notification mute state, Preferences
- **Watchlist entry** _(retention: persistent)_ — User's tracked cryptocurrency with alert rules
  - fields: User id, Ticker, Friendly name, Enabled flag, Last-known price, Active alerts, Cooldown until
- **Alert rule** _(retention: persistent)_ — Price threshold or percent change alert configuration
  - fields: Rule type, Direction, Target value, Window, Active flag
- **Event record** _(retention: persistent)_ — Record of triggered alerts for analytics
  - fields: Timestamp, User id, Ticker, Rule id, Old price, New price, Percent change, Delivered flag
- **Owner analytics** _(retention: persistent)_ — Aggregated user and alert statistics
  - fields: Total users, Active users (30d), Per-ticker triggers, Per-alert-type triggers

## Integrations

- **Telegram** (required) — Bot API messaging and owner dashboard
- **Price data provider** (required) — Fetch current crypto prices with retries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View user count and active users
- See most-triggered alerts and tokens
- Access analytics dashboard via /dashboard command

## Notifications

- Price threshold alerts
- Percent change alerts
- Daily summary digest
- Price unavailability notices
- Owner analytics updates

## Permissions & privacy

- All user data is private and not shared
- Watchlists and alerts are user-specific
- Owner dashboard shows aggregated statistics only
- Users can mute notifications at any time

## Edge cases

- Price data source failures with retry logic
- User timezone changes after setup
- Multiple alert rules for same coin
- Typo in ticker symbol with fuzzy matching
- Quiet hours overlapping with scheduled summary time

## Required tests

- Verify alert triggers with price threshold
- Test percent change alerts with hysteresis
- Confirm daily summary delivery outside quiet hours
- Validate ticker validation and fuzzy matching
- Test owner dashboard accuracy with multiple users

## Assumptions

- Default quiet hours window is 10PM-8AM
- Default cooldown is 12 hours
- Percent alerts use 1-hour window by default
- Price data provider has 99% uptime
- Users understand basic crypto concepts
