# HabitKeeper — Bot specification

**Archetype:** workflow

**Voice:** encouraging and concise — write every user-facing message, button label, error, and empty state in this voice.

A private Telegram bot that helps users build and maintain habit streaks with one-tap tracking, gentle reminders, and weekly summaries. Each user has their own private habit space with stats like current streak, best streak, and completion percentage.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individuals who want a lightweight, private habit tracker inside Telegram
- Non-technical users who prefer simple one-tap interactions and friendly notifications

## Success criteria

- Users can create and track multiple habits with one-tap interactions
- Users receive timely and non-intrusive reminders based on their local time
- Users can view weekly summaries and streak statistics
- Users can pause/resume habits and adjust past entries without breaking streaks

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with habit overview and quick actions
- **+ New habit** (button, actor: user, callback: habit:new) — Start creating a new habit with name, frequency, and reminder time setup
- **/list** (command, actor: user, command: /list) — Show all habits at a glance with streaks and next reminder times
- **/week** (command, actor: user, command: /week) — Show a weekly calendar view of all habits with completion status
- **Mark Done** (button, actor: user, callback: habit:mark_done) — Mark the current day as done for a habit in a reminder message
- **Skip** (button, actor: user, callback: habit:skip) — Mark the current day as skipped for a habit in a reminder message
- **Fail** (button, actor: user, callback: habit:fail) — Mark the current day as failed for a habit in a reminder message
- **Snooze 1h** (button, actor: user, callback: habit:snooze_1h) — Postpone the current reminder by one hour

## Flows

### Onboarding
_Trigger:_ /start

1. Detect user timezone from Telegram metadata
2. Show brief welcome message
3. Prompt to create first habit or set timezone manually

_Data touched:_ User

### Create Habit
_Trigger:_ habit:new

1. Prompt for habit name
2. Select frequency type (daily, weekdays, N-times-per-week)
3. Choose reminder time (default 09:00)
4. Optionally add description
5. Confirm and add to habit list

_Data touched:_ Habit

### Daily Reminder
_Trigger:_ scheduled_reminder

1. Send reminder message with habit name and current streak
2. Show 'Mark Done' and 'Skip/Fail' buttons
3. Handle user action (Done/Skip/Fail/Snooze)

_Data touched:_ HabitDayEntry, Statistics

### Manual Adjustment
_Trigger:_ habit:edit

1. Show week view of habit
2. Allow changing past day entries to Done/Skipped/Failed/Empty
3. Update streak and best streak calculations

_Data touched:_ HabitDayEntry, Statistics

### Pause/Resume
_Trigger:_ habit:pause

1. Toggle habit status between active and paused
2. Stop/start sending reminders

_Data touched:_ Habit

### Weekly Summary
_Trigger:_ /week

1. Generate 7-day calendar view per habit
2. Show completion icons and overall weekly rate
3. Display milestone messages if applicable

_Data touched:_ HabitDayEntry, Statistics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with timezone and preferences
  - fields: telegram_id, timezone, preferences
- **Habit** _(retention: persistent)_ — User-created habit with tracking parameters
  - fields: id, name, description, frequency_type, target_n, reminder_time, active, creation_date, milestone_thresholds
- **HabitDayEntry** _(retention: persistent)_ — Daily tracking entry for a habit
  - fields: habit_id, date, status, action_timestamp
- **Statistics** _(retention: derived)_ — Derived metrics for habit performance
  - fields: current_streak, best_streak, total_completed, completion_rate, week_summary

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set default reminder time
- Configure milestone thresholds
- Enable/disable snooze functionality
- Set frequency interpretation rules for N-times-per-week habits

## Notifications

- Daily reminders with quick-action buttons
- Weekly summary with calendar view
- Milestone celebration messages (7, 14, 30, 100 days)
- Snooze confirmation messages

## Permissions & privacy

- All user data is private to the Telegram account
- No data sharing or export in MVP
- User can only access their own habits and history

## Edge cases

- User changes timezone after creating habits
- User marks a day as Done then later changes it to Skipped
- User has multiple habits with overlapping reminder times
- User pauses a habit and later resumes it mid-week

## Required tests

- Verify one-tap marking works without double-counting
- Test timezone-aware reminders across DST changes
- Validate streak calculations after manual adjustments
- Ensure pause/resume correctly stops and resumes reminders

## Assumptions

- Users will prefer even distribution of N-times-per-week reminders
- Skipped days should not break streaks by default
- Milestone messages should be optional and non-intrusive
- Users will want to edit past entries without breaking streaks
