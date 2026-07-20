# VocabSRS Bot — Bot specification

**Archetype:** education

**Voice:** professional and encouraging — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot for language learners that uses spaced repetition to teach vocabulary. Users create decks with prompt/translation cards, manage review sessions with SRS scheduling, track progress metrics, and receive configurable reminders while maintaining full data privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual language learners

## Success criteria

- Users complete daily review sessions with spaced repetition scheduling
- Persistent session state resumes after interruptions
- Private user data remains isolated between accounts

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Onboarding flow: explain SRS basics, select/create deck, set daily new-card limit and timezone
- **/review** (command, actor: user, command: /review) — Start review session with due cards
- **/decks** (command, actor: user, command: /decks) — List and manage decks (create, delete, select)
- **Add card** (button, actor: user, callback: deck:add_card) — Initiate card creation flow for current deck
- **Import starter deck** (button, actor: user, callback: deck:import_starter) — Add pre-built vocabulary sets

## Flows

### Onboarding
_Trigger:_ /start

1. Explain SRS basics
2. Select/create deck
3. Set daily new-card limit
4. Configure timezone

_Data touched:_ User

### Review Session
_Trigger:_ /review

1. Show due card prompt
2. User reveals translation
3. User selects rating (Again/Hard/Good/Easy)
4. Update SRS metadata
5. Queue next card

_Data touched:_ Card, Review session

### Deck Management
_Trigger:_ /decks

1. List decks with stats
2. Create/delete/edit decks
3. Select active deck

_Data touched:_ Deck

### Card Management
_Trigger:_ deck:add_card

1. Input prompt
2. Input translation
3. Add optional example
4. Assign to deck

_Data touched:_ Card

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account with private profile and settings
  - fields: telegram_id, timezone, daily_new_card_limit, reminder_time, notification_frequency
- **Deck** _(retention: persistent)_ — Named collection of cards private to owner
  - fields: name, card_count, due_count, created_at
- **Card** _(retention: persistent)_ — Prompt/translation pair with SRS metadata
  - fields: prompt, translation, example_sentence, ease_factor, interval, repetition_count, due_date, status
- **Review Session** _(retention: session)_ — Interruptible sequence of cards for review
  - fields: current_queue, position, session_progress

## Integrations

- **Telegram** (required) — Bot API messaging for all interactions
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete/edit decks
- Adjust daily new-card limit
- Configure reminder time and frequency
- Manage card content

## Notifications

- Daily review reminder at configurable time
- Nudges for due cards (configurable: off/once-per-day)

## Permissions & privacy

- All data strictly private per user account
- No shared decks or public content

## Edge cases

- Empty deck handling with import suggestions
- Session resumption after interruptions
- No due cards on review session start

## Required tests

- Review session persistence across interruptions
- Deck management with nested operations
- SRS algorithm response handling

## Assumptions

- SM-2 algorithm implementation for SRS
- Default daily limit of 10 new cards
- 20:00 default reminder time
