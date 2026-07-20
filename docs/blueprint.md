# SRS Language Buddy — Bot specification

**Archetype:** education

**Voice:** warm and encouraging — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for language learning with spaced repetition. Users create/import word decks, set daily new-card limits, and complete study sessions with SRS-based scheduling. Sessions include word reviews with 4-grade feedback, progress tracking, and private data storage. Daily reminders and stats help maintain learning consistency.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram users learning foreign languages
- People seeking efficient spaced repetition practice

## Success criteria

- Users complete daily study sessions with SRS-based scheduling
- Persistent progress tracking across sessions
- Private data storage per user

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open onboarding menu with deck creation, settings, and study options
- **Create Deck** (button, actor: user, callback: deck:create) — Initiate deck creation flow
  - inputs: deck title, description
  - outputs: deck created confirmation
- **Study** (button, actor: user, callback: study:start) — Begin study session with due cards and new cards
  - inputs: deck selection
  - outputs: card queue with front/back interaction
- **Import Starter Deck** (button, actor: user, callback: deck:import) — Choose from built-in language decks
  - inputs: language selection
  - outputs: import confirmation

## Flows

### Onboarding
_Trigger:_ /start

1. Display welcome message
2. Offer deck creation/import options
3. Set daily new-card limit
4. Configure reminders

_Data touched:_ User, DailySettings

### Study Session
_Trigger:_ Study button

1. Compose session queue (due cards + new cards)
2. Show word front
3. Display answer with feedback buttons
4. Update SRS metadata
5. Proceed to next card

_Data touched:_ Card, Session

### Daily Reminder
_Trigger:_ Scheduled time

1. Check due cards for user
2. Send reminder message with study button
3. Track reminder delivery

_Data touched:_ User, Card

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with private settings
  - fields: telegram_id, notification_prefs, daily_settings
- **Deck** _(retention: persistent)_ — Private word collection
  - fields: title, description, card_ids
- **Card** _(retention: persistent)_ — Word/translation pair with SRS metadata
  - fields: front, back, example, easiness, interval, due_date, repetition_count
- **Session** _(retention: session)_ — Active study session state
  - fields: card_queue, current_position, progress
- **DailySettings** _(retention: persistent)_ — User's daily learning parameters
  - fields: new_cards_per_day, reminder_time, notifications_enabled

## Integrations

- **Telegram** (required) — Private chat messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete decks
- Edit/delete cards
- Adjust daily new-card limit
- Enable/disable reminders
- View progress statistics

## Notifications

- Daily study reminder at configured time
- Session progress updates during study
- Empty session guidance when no cards are available

## Permissions & privacy

- All data strictly private per user
- No cross-user data sharing
- Deck and card data encrypted at rest

## Edge cases

- Empty session when no cards exist
- User stops mid-session and resumes later
- Card due date falls outside current session window

## Required tests

- Session resumption after interruption
- SRS algorithm updates card intervals correctly
- Private data isolation between users

## Assumptions

- Default new cards per day set to 20
- Built-in starter decks for common language pairs
- Private deck visibility by default
