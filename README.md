# Fitness Debt Ledger

Note: This README also serves as my AGENTS.md file, and serves as a record of the prompt, business logic and changelog. Please make updates to this file whenever a significant piece of functionality is updated or added.

This codebase is going to reflect a game that I play with my girlfriend, where unhealthy foods that I consume cost "money" in the form of pushups. We currently use the Chinese RMB to USD exchange rate, at a 7:1 conversion. For example, if I want to drink a beer that costs $1 USD, I will have to do 7 push-ups (or similar exercise) to 'pay it off'. There are a few other rules to note:

1. If I don't have any money 'saved up', then when I buy a beer I will have to pay double the amount, as a cost of buying on 'credit'
2. If I have any debt left at the end of the day, that I haven't paid off, the amount doubles overnight.
3. Even I have accrued 0 debt in a day, each day comes with a 50RMB management fee. The management fee is added at the START of the new day (after doubling), not before. This means the fee itself is never doubled.
4. Different types of exercise earns different amount of points, but in general it is a 1:1 rate. For example, one pushup is one RMB worth of money, as is one sit-up. However, one minute of planking is worth 50RMB (or 30). The system should allow you to add/update the parameters of the values, but start with sensible defaults (such as one pushup = 1RMB, 1 curl = 1RMB, 1 bench = 1RMB).

What I want you to do is to build a simple tracker of my debt where I can a) record new transactions, increasing my total debt count b) record exercise towards paying off the debt and c) tracking the overnight accrual of additional interests and management fees. d) for later functionality, I would like to be able to track trends over time, but for the initial version let's not start that. For now, let's use a very simple, mobile first UI where everything is stored locally using localStorage. What I would like is a simple, raw HTML file without any fancy front-end frameworks like React, where I can record my activity and edit the amounts that a particular piece of exercise is worth.

## Clarified Requirements (2026-01-11)

**Day Rollover**: Automatic at midnight - the app detects when a new day starts and applies interest/fees
**Plank Value**: 30 RMB per minute
**Savings**: No savings tracking - balance cannot go positive, exercise only pays down existing debt
**Management Fee**: Always applied (50 RMB/day), even at zero balance

## Task List

- [x] Clarify requirements
- [x] Create HTML structure with mobile-first design
- [x] Implement localStorage for data persistence
- [x] Add transaction recording (increase debt)
- [x] Add exercise recording (pay off debt)
- [x] Implement automatic midnight rollover with doubling logic
- [x] Add exercise value configuration UI

## Implementation Notes

Created `index.html` - a standalone HTML file with no external dependencies that includes:

**Features Implemented:**

- Mobile-first responsive design with clean UI
- Three-tab interface: Transactions, Exercise, Settings
- Real-time debt display at the top
- Purchase tracking with USD to RMB conversion (7:1)
- Double cost penalty when making purchases while in debt
- Exercise recording with configurable values per exercise type
- Automatic midnight detection and day rollover
- Day-end processing: adds 50 RMB management fee, then doubles total debt
- Transaction history with visual indicators (red for debt, green for payments, orange for system)
- Exercise value configuration (editable in Settings tab)
- All data persisted to localStorage
- Reset functionality

**How the Day Rollover Works:**
The app checks the last stored date vs. current date on every page load. If days have passed, it applies the day-end logic for each day:

1. Double yesterday's debt
2. Add 50 RMB management fee for the new day (not doubled)
3. Reset credit to 0
4. Record as a system transaction

**Exercise Types Included:**

- Push-ups: 1 RMB per rep (default)
- Sit-ups: 1 RMB per rep (default)
- Curls: 1 RMB per rep (default)
- Bench Press: 1 RMB per rep (default)
- Plank: 30 RMB per minute (default)

All values are configurable in the Settings tab.

## Updates (2026-01-11)

**UI Redesign - Treasury/Bank Ledger Theme:**

- Complete visual overhaul with official government/bank document aesthetic
- Serif typography (Georgia) for formal appearance
- Navy blue (#1a365d), dark gray (#2d3748), and gold (#b8860b) color scheme
- Official header seal: "Department of Physical Treasury - FITNESS DEBT LEDGER"
- Formal borders, double-border containers, and structured layouts
- Monospace fonts (Courier New) for numbers and forms
- Enhanced transaction history with color-coded entries

**Tab Reordering:**

- Exercise tab moved to first position (most frequently used)
- Order now: Exercise → Transactions → Settings

**Quick-Add Exercise Buttons:**

- Added scrollable list of all exercises below the main exercise form
- Each exercise type has two circular quick-add buttons:
  - Push-ups: 10 / 20 reps
  - Sit-ups: 10 / 20 reps
  - Curls: 10 / 20 reps
  - Bench Press: 10 / 20 reps
  - Plank: 1min / 2min
- One-click exercise logging for efficient debt payment workflow
- Designed for users paying off large debts (e.g., 580 RMB in a day)

**Technical Improvements:**

- Fixed CSS vendor prefix warning (added `appearance: textfield`)
- Refactored exercise recording into shared `recordExercise()` function
- Quick-add buttons use same logic as form submission

## Cloud Sync Feature (2026-01-11)

**Implementation:**

- Integrated with keyvalue.immanuel.co free key-value storage service
- Syncs debt amount only (lightweight, fast)
- User chooses their own custom cloud key for multi-device access

**Features:**

- **Custom Cloud Keys:**
  - Users create their own memorable keys (e.g., "misha", "john123")
  - Keys must be alphanumeric (letters and numbers only)
  - Up to 20 characters allowed
  - Simple validation prevents invalid characters

- **Simplified Onboarding:**
  - Clear "How it works" instructions in info box
  - Single "Start Using This Key" button combines save + sync
  - Immediate feedback when key is activated
  - Advanced options (manual sync, pull from cloud) hidden in collapsible section

- **Auto-Sync:**
  - Automatically syncs debt to cloud after every purchase
  - Automatically syncs after every exercise payment
  - Automatically syncs after overnight day rollover
  - Silent background sync (non-blocking)

- **Manual Controls:**
  - "Manual Sync" button in Advanced section
  - "Pull from Cloud" button to restore debt from cloud
  - Shows old vs new debt when pulling from cloud

- **Sync Status Display:**
  - Shows "Not configured" if no app key set
  - Shows "Synced just now" / "Synced X min ago" after successful sync
  - Shows "Never synced" if key exists but no sync yet
  - Color-coded: green (recent), gray (old), red (never)

**Use Case:**

1. Device A: Enter custom key "misha" and click "Start Using This Key"
   - No existing data found → syncs current debt (0) to cloud
2. Device A: Make transactions → app auto-syncs debt to cloud after each transaction
3. Device B: Enter same key "misha" and click "Start Using This Key"
   - Existing data found → automatically pulls debt from cloud
   - Shows: "Found existing data in cloud: Debt 580 RMB"
4. Both devices now auto-sync on every transaction

**Smart Key Detection:**
When you click "Start Using This Key", the app:

1. Checks if that key already exists in the cloud
2. If YES: Automatically loads the cloud data (prevents overwriting)
3. If NO: Syncs your current local debt to cloud (new key setup)

This prevents accidentally overwriting cloud data when setting up a second device.

**API Integration:**

- POST /api/KeyVal/UpdateValue/{appkey}/fitnessdebt/{value} - Sync debt + lastCheckDate
- GET /api/KeyVal/GetValue/{appkey}/fitnessdebt - Load debt + lastCheckDate
- Value format: pipe-delimited `{debt}|{lastCheckDate}` (e.g., "580|2026-01-11")

**Day Rollover Sync Logic:**
To prevent duplicate day-end charges across multiple devices:

1. Cloud stores both `debt` and `lastCheckDate` as pipe-delimited string
2. When app loads and detects a new day, it first checks cloud for latest `lastCheckDate`
3. If another device already applied today's rollover, skip it
4. If rollover is needed, apply it then sync to cloud
5. This ensures only ONE device applies the management fee + doubling per day

Example scenario:

- Monday 8am: Device A opens → detects new day → applies rollover → syncs "1260|2026-01-11"
- Monday 2pm: Device B opens → detects new day → checks cloud first → sees lastCheckDate already "2026-01-11" → skips rollover
- Result: Fee only applied once, both devices stay in sync

**Data Format:**
Using simple pipe delimiter instead of JSON for better URL compatibility:

- Format: `{debt}|{credit}|{lastCheckDate}`
- Example: `580|0|2026-01-11`
- Avoids URL encoding issues with JSON special characters

## Bug Fixes (2026-01-12)

**Timezone Bug Fix:**

- Fixed critical bug where rollover was happening at 4pm PST instead of midnight
- Root cause: `getDateString()` was using `toISOString()` which returns UTC timezone
- When it was 4pm PST (midnight UTC), the app thought a new day had started
- Solution: Changed to use local timezone with `getFullYear()`, `getMonth()`, `getDate()`
- Now rollovers correctly happen at local midnight regardless of timezone

**Cloud Sync Refresh Button:**

- Added subtle refresh icon (↻) in top-right corner of debt display
- Only visible when cloud sync is configured (when `state.appKey` exists)
- Clicking the icon silently syncs data from cloud
- Shows spinning animation during sync
- Displays toast notification: "✓ Synced from cloud" or "✓ Already up to date"
- Provides quick way to check for updates from other devices without navigating to Settings tab
