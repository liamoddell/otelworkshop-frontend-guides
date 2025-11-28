# Lab 6: Track User Actions & Workflows

**Duration**: 20 minutes

---

## Overview

In this lab, you'll learn about Faro's User Actions feature, which tracks both automatic interactions and custom workflows. You'll understand what's captured automatically and how to create custom user actions for business-critical flows.

**What you'll learn:**
- Automatic user actions (page views, clicks)
- Manual user action tracking with `startUserAction()`
- How user actions auto-complete
- Tracking multi-step workflows

---

## Learning Objectives

By the end of this lab, you will:
- Understand automatic vs manual user actions
- Know how to use `faro.api.startUserAction()`
- Understand how the 100ms auto-completion timer works
- Be able to track custom workflows
- Query user actions in Grafana Cloud

---

## User Actions: Automatic vs Manual

### Automatic User Actions (Already Enabled)
From Lab 2, Faro already tracks:
- **Page Views**: Route navigation
- **Clicks**: Button/link clicks

These happen **automatically** with no code needed.

### Manual User Actions (This Lab)
Use `startUserAction()` for:
- Multi-step workflows (checkout, bet placement)
- Custom business flows
- Operations spanning multiple events

**Key difference**: Manual actions let you name and track specific workflows.

---

## How User Actions Work

### Auto-Completion Mechanism

User actions **automatically complete** after **100ms of inactivity**:

1. You call `startUserAction('workflow_name')`
2. The action starts tracking
3. Related events (clicks, API calls) reset the 100ms timer
4. After 100ms with no activity → action completes automatically

**No manual stop needed!** The action ends on its own.

### Example Timeline:
```
0ms:    startUserAction('bet_placement')
50ms:   User clicks button → timer resets to 100ms
200ms:  API call starts → timer resets to 100ms
850ms:  API completes → timer resets to 100ms
950ms:  Action completes automatically (100ms after last event)
```

---

## Quick Start (Files to Modify)

**You'll be editing 2 files in this lab:**

1. **`app/events/[id]/page.tsx`** - Track bet placement workflow
2. **`app/account/page.tsx`** - Track deposit workflow

---

## Detailed Steps

### Part 1: Verify Automatic Tracking

**No changes needed** - automatic tracking is already enabled from Lab 2!

**File:** `webapp/lib/faro.ts`

```typescript
instrumentations: [
  ...getWebInstrumentations({
    captureConsole: false,
  }),
  new TracingInstrumentation(),
],
```

**Test it:**
1. Navigate between pages → Automatic `page_view` actions
2. Click buttons → Automatic `click` actions

---

### Part 2: Track Bet Placement Workflow

**Goal:** Create a custom user action that tracks the entire bet placement flow from button click to completion.

**File:** `webapp/app/events/[id]/page.tsx`

#### Step 1: Import is already done

The `getFaro` import should already be present from Lab 3.

#### Step 2: Start user action at beginning of bet placement

**Find the bet placement handler** (search for `const handlePlaceBet`).

**Add at the very beginning of the function** (before validation):

```typescript
const handlePlaceBet = async () => {
  const faro = getFaro();
  faro?.api.startUserAction('bet_placement_workflow', {
    eventId: event.id,
    sport: event.sport,
  });

  setError('');
  setSuccess('');

  // ... rest of function
```

**That's it!** The action will auto-complete 100ms after the bet is placed (or after any error).

**Why this works:**
- Action starts when user initiates bet
- All subsequent events (validation, API call, state updates) reset the 100ms timer
- 100ms after the last event (success message or error), action completes automatically
- Duration captures the entire workflow time

**Complete function should start like:**

```typescript
const handlePlaceBet = async () => {
  const faro = getFaro();
  faro?.api.startUserAction('bet_placement_workflow', {
    eventId: event.id,
    sport: event.sport,
  });

  setError('');
  setSuccess('');

  if (!user) {
    setError('Please log in to place bets');
    setTimeout(() => router.push('/login'), 2000);
    return;
  }

  // ... rest of validation and bet placement logic
};
```

---

### Part 3: Track Deposit Workflow

**Goal:** Track the deposit flow as a custom user action.

**File:** `webapp/app/account/page.tsx`

#### Step 1: Import is already done

The `getFaro` import should already be present from Lab 3.

#### Step 2: Start user action at beginning of deposit handler

**Find the deposit handler** (search for `const handleDeposit`).

**Add at the beginning:**

```typescript
const handleDeposit = async () => {
  const faro = getFaro();
  faro?.api.startUserAction('deposit_workflow', {
    depositMethod: 'credit_card',
  });

  if (!user) return;

  setError('');
  setSuccess('');

  const amount = parseFloat(depositAmount);

  // ... rest of function
```

**That's it!** The deposit workflow action will auto-complete after the transaction finishes.

---

## Verification

### Step 1: Generate User Actions

Test both automatic and manual tracking:

1. **Automatic actions** (no code needed):
   - Navigate between pages → `page_view`
   - Click buttons → `click`

2. **Manual actions** (from this lab):
   - Place bets → `bet_placement_workflow`
   - Make deposits → `deposit_workflow`

### Step 2: View in Frontend Observability

1. Go to **Frontend Observability** → Your app
2. Click **"User Journey"** tab
3. You'll see:
   - Automatic `page_view` and `click` actions
   - Your custom `bet_placement_workflow` actions
   - Your custom `deposit_workflow` actions
4. Click on a workflow action to see:
   - Duration (entire workflow time)
   - Custom attributes (eventId, sport, etc.)
   - User context

### Step 3: Query with LogQL

**View all user actions (automatic + manual):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="user_action"
```

**View only your custom workflows:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="user_action"
| type=~"bet_placement_workflow|deposit_workflow"
```

**View bet placement workflows only:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="user_action"
| type="bet_placement_workflow"
```

**Calculate average bet placement duration:**
```logql
avg_over_time({service_name="<your-name>-betwise"}
  | logfmt
  | kind="user_action"
  | type="bet_placement_workflow"
  | unwrap value_duration [5m])
```

**Compare workflow durations:**
```logql
avg by (type) (
  avg_over_time({service_name="<your-name>-betwise"}
    | logfmt
    | kind="user_action"
    | type=~"bet_placement_workflow|deposit_workflow"
    | unwrap value_duration [5m])
)
```

### Step 4: Analyze the Data

Look for:
- **Workflow duration**: How long do bets/deposits take?
- **Custom attributes**: Enriched with eventId, sport, etc.
- **User context**: Linked to specific users
- **Session correlation**: Grouped by session_id

**Success Criteria:**
- You see automatic `page_view` and `click` actions
- Custom `bet_placement_workflow` actions appear with correct duration
- Custom `deposit_workflow` actions appear
- Attributes include eventId, sport, etc.
- Actions are linked to user sessions

---

## Understanding the 100ms Timer

### Why 100ms?

This captures the **complete workflow** including:
- Initial click
- Validation
- API calls
- State updates
- UI feedback

**Example: Bet Placement**
```
0ms:     User clicks "Place Bet"
         ↓ startUserAction() called
50ms:    Validation runs → timer resets
100ms:   setState triggers → timer resets
600ms:   Simulated API delay → timer resets
850ms:   Bet created → timer resets
900ms:   Success message → timer resets
1000ms:  Action completes (100ms after last event)
         Duration: 1000ms
```

### When Actions Complete

Actions complete when:
- ✅ 100ms pass with no related events
- ✅ All async operations finish
- ✅ UI updates complete

Actions do NOT complete when:
- ❌ Events keep happening (timer keeps resetting)
- ❌ API calls are in progress
- ❌ State updates are pending

---

## Best Practices

### When to Use Manual User Actions

**Good use cases:**
- Multi-step workflows (checkout, registration, bet placement)
- Operations with async logic
- Business-critical flows you want to measure
- Workflows spanning multiple user interactions

**Bad use cases:**
- Single click events (use automatic tracking)
- Very short operations (< 100ms might not capture fully)
- High-frequency operations (too much data)

### Naming Conventions

**Good names:**
- `bet_placement_workflow` - Descriptive, snake_case
- `deposit_workflow`
- `registration_flow`

**Bad names:**
- `action1` - Not descriptive
- `BetPlacement` - Inconsistent casing
- `bet` - Too generic

### Adding Context with Attributes

**Include relevant business data:**
```typescript
faro?.api.startUserAction('bet_placement_workflow', {
  eventId: event.id,
  sport: event.sport,
  betType: selectedBetType,
  // Don't include: stake amount (privacy), passwords, PII
});
```

**Good attributes:**
- IDs (eventId, userId)
- Categories (sport, league)
- Workflow metadata (source, trigger)

**Avoid:**
- Sensitive data (amounts, passwords)
- PII (full names, addresses)
- High-cardinality values (timestamps, random IDs)

---

## Automatic vs Manual: When to Use Each

### Automatic User Actions
**Use for:** Understanding user navigation and interaction patterns
- Where do users click?
- What pages do they visit?
- How do they navigate the app?

**Examples:**
- Click heatmaps
- Navigation flow analysis
- Page popularity tracking

### Manual User Actions
**Use for:** Tracking business-critical workflows
- How long do key operations take?
- Do workflows complete successfully?
- Where do users encounter friction?

**Examples:**
- Checkout completion time
- Registration funnel duration
- Payment success rate

### Combined Analysis

**Query:** Complete user journey (automatic + manual)
```logql
{service_name="<your-name>-betwise"}
| logfmt
| user_id="<user-id>"
| kind=~"user_action|event"
```

Shows:
- `page_view` → User lands on event page
- `click` → User clicks bet button
- `bet_placement_workflow` → Complete bet flow (850ms)
- `bet_placed` event → Business outcome

---

## Summary

**User Actions are Working!**

In this lab, you successfully:
- Understood automatic vs manual user actions
- Learned how `startUserAction()` works
- Implemented bet placement workflow tracking
- Implemented deposit workflow tracking
- Learned about the 100ms auto-completion mechanism

**What This Enables:**
- **Workflow optimization**: Measure multi-step operations
- **User experience**: Find friction points in flows
- **Business metrics**: Track critical workflow completion
- **Complete journey tracking**: Automatic + manual = full picture

**Key Concepts Learned:**
- `faro.api.startUserAction(name, attributes)` - Start tracking a workflow
- Auto-completion after 100ms of inactivity
- Events reset the timer automatically
- Custom attributes enrich workflow data
- Automatic tracking (clicks, page views) complements manual actions

**The Complete Picture:**
- **Automatic actions**: HOW users navigate (page views, clicks)
- **Manual actions**: WHAT workflows they complete (bet placement, deposits)
- **Custom events** (Lab 3): WHICH business moments occur (bet_placed)
- **Measurements** (Lab 5): HOW LONG operations take (bet_placement_duration)
- **Together**: Complete understanding of user behavior and business impact

---

**Next Lab:** Lab 7 - Explore all your data in Grafana Cloud!
