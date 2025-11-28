# Lab 3: Add Custom Events & User Context

**Duration**: 20 minutes

---

## Overview

In this lab, you'll instrument your application with custom business events and user context. You'll learn to:
- Track custom events (registrations, bets, deposits)
- Associate events with specific users
- Send contextual data with each event

**What you'll build:**
- User registration tracking
- Bet placement tracking
- Deposit transaction tracking
- User context (ID, email, balance)

---

## Learning Objectives

By the end of this lab, you will:
- Understand when and why to use custom events
- Know how to use Faro's `pushEvent()` API
- Know how to use Faro's `setUser()` API
- Be able to track business-critical user actions
- Understand how user context enriches observability data

---

## Why Custom Events?

Automatic instrumentation captures standard web metrics, but custom events let you track business-specific actions:

**Business Actions:**
- Bet placements (revenue-generating actions)
- Deposits (payment flow)
- Registration completions (user acquisition)

**What This Enables:**
- **Conversion funnel analysis**: Where do users drop off?
- **Revenue tracking**: Which features drive bets?
- **User segmentation**: High-value vs. low-value users

---

## Quick Start (Files to Modify)

**You'll be editing 3 files in this lab:**

1. **`lib/faro.ts`** - Add `getFaro()` export function
2. **`app/register/page.tsx`** - Track registration + set user context
3. **`app/events/[id]/page.tsx`** - Track bet placements
4. **`app/account/page.tsx`** - Track deposits
5. **`app/login/page.tsx`** - Set user context on login

---

## Detailed Steps

### Part 1: Setup - Update Faro Initialization

**Goal:** Expose the Faro API so components can push custom events.

#### Step 1: Expose Faro API for Custom Events

Edit: `webapp/lib/faro.ts`

**Update the imports to include the `faro` instance:**
```typescript
import { initializeFaro, faro as faroInstance, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';
```

**After the existing `initFaro()` function, add a new export function:**
```typescript
export function getFaro() {
  return faroInstance;
}
```

**Why?** The `faro` instance from the SDK provides the API to push custom events, errors, and measurements. The `getFaro()` function lets components access this API.

**Complete file should look like:**
```typescript
import { initializeFaro, faro as faroInstance, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';

let faroInitialized: ReturnType<typeof initializeFaro> | null = null;

export function initFaro() {
  if (typeof window === 'undefined') {
    return null;
  }

  if (faroInitialized) {
    return faroInitialized;
  }

  faroInitialized = initializeFaro({
    url: process.env.NEXT_PUBLIC_FARO_URL || '',
    app: {
      name: 'betwise-betting-app',
      version: '1.0.0',
      environment: 'workshop',
    },
    instrumentations: [
      ...getWebInstrumentations(),
      new TracingInstrumentation(),
    ],
  });

  return faroInitialized;
}

export function getFaro() {
  return faroInstance;
}
```

---

### Part 2: Track User Registration

**Goal:** Track when users complete registration and set their user context.

**File:** `webapp/app/register/page.tsx`

#### Step 1: Add import at the top
```typescript
import { getFaro } from '@/lib/faro';
```

#### Step 2: Track registration event and set user context

**Find the successful registration section** (search for `setUser(newUser)`).

Add this code **after** the `setUser(newUser)` line:

```typescript
const faro = getFaro();
faro?.api.pushEvent('user_registered', {
  event: 'user_registered',
  userId: newUser.id,
  email: newUser.email,
  startingBalance: newUser.balance.toString(),
});

faro?.api.setUser({
  id: newUser.id,
  email: newUser.email,
  attributes: {
    balance: newUser.balance.toString(),
    accountAge: '0',
  },
});
```

**Note:** We include `event: 'user_registered'` in the attributes so it appears as `event_data_event` in Grafana, making it easy to filter events by type.

**Why include `event: 'user_registered'`?**
The first parameter of `pushEvent()` is the event name, but it doesn't automatically become a filterable field in Grafana. By including it as `event: 'user_registered'` in the attributes, it appears as `event_data_event` in Grafana, making it easy to filter and query by event type.

**Why track this?**
- **Conversion metrics**: How many visitors complete registration?
- **User segmentation**: Track user cohorts over time
- **Funnel analysis**: Registration → First Bet → Retention

**What data to include?**
- **Event name**: Always include `event: 'event_name'` for filtering
- **User ID**: Link events to specific users
- **Contextual data**: Balance, source, etc.
- **Avoid PII**: Don't send passwords, full credit cards

---

### Part 3: Track Bet Placements

**Goal:** Track when users place bets (key revenue event).

**File:** `webapp/app/events/[id]/page.tsx`

#### Step 1: Add import at the top
```typescript
import { getFaro } from '@/lib/faro';
```

#### Step 2: Track bet placement event

**Find the bet creation section** inside the `handlePlaceBet` function (around line 117).

You'll see:
```typescript
addBet(bet);

// Update balance
const newBalance = user.balance - stakeAmount;
```

Add the Faro tracking **after** the `addBet(bet)` line and **before** the balance update:

```typescript
const faro = getFaro();
faro?.api.pushEvent('bet_placed', {
  event: 'bet_placed',
  eventId: event.id,
  betType: selectedBetType,
  stake: stakeAmount.toString(),
  odds: selectedOdds.toString(),
  potentialReturn: potentialReturn.toString(),
  sport: event.sport,
  league: event.league,
});
```

**Why track this?**
- **Revenue analytics**: Which sports/events generate most bets?
- **User behavior**: Average stake amounts, bet patterns
- **Business KPIs**: Daily bet volume, high-value bets

**Key attributes:**
- **eventId**: Which event was bet on?
- **stake**: Bet amount (revenue metric)
- **odds**: Expected return
- **sport/league**: Category analysis

---

### Part 4: Track Deposits

**Goal:** Track when users make deposits (payment flow success).

**File:** `webapp/app/account/page.tsx`

#### Step 1: Add import at the top
```typescript
import { getFaro } from '@/lib/faro';
```

#### Step 2: Track deposit event

**Find the successful deposit** (search for `addTransaction(transaction)`).

Add this code **after** the `addTransaction(transaction)` line:

```typescript
const faro = getFaro();
faro?.api.pushEvent('deposit_completed', {
  event: 'deposit_completed',
  amount: amount.toString(),
  newBalance: newBalance.toString(),
  depositMethod: 'credit_card',
});
```

**Why track this?**
- **Payment success rate**: Monitor payment failures
- **Revenue tracking**: Total deposits over time
- **Fraud detection**: Unusual deposit patterns

---

### Part 5: Add User Context on Login

**Goal:** Set user context when users log in (links all future events to this user).

**Why user context?** Links all events and errors to a specific user, enabling:
- User session replay
- User-specific error tracking
- Lifetime value analysis

**Note:** We already added `setUser()` in the registration flow (Part 2). Now we need to add it for the login flow.

**File:** `webapp/app/login/page.tsx`

#### Step 1: Add import at the top

```typescript
import { getFaro } from '@/lib/faro';
```

#### Step 2: Set user context after successful login

**Find the successful login validation** (around line 46-48).

You'll see:
```typescript
    setLoading(false);
    router.push('/');
  };
```

Add the Faro `setUser()` call **before** the `router.push('/')` line:

```typescript
    const faro = getFaro();
    faro?.api.setUser({
      id: user.id,
      email: user.email,
      attributes: {
        balance: user.balance.toString(),
        accountAge: (Date.now() - new Date(user.createdAt).getTime()).toString(),
      },
    });

    setLoading(false);
    router.push('/');
  };
```

**User attributes best practices:**
- **Don't include**: Passwords, credit card numbers, sensitive PII
- **Do include**: User tier, subscription status, preferences
- **Purpose**: Segmentation and analysis, not identification

---

---

## Verification

### Step 1: Generate Events

Test your instrumentation by performing these actions in your browser:

1. **Register a new user** → Generates `user_registered` event
2. **Place a bet** → Generates `bet_placed` event
3. **Make a deposit** → Generates `deposit_completed` event

### Step 2: View Events in Grafana Cloud

**Option 1: Frontend Observability - User Journey Tab**
1. Go to **Frontend Observability** → Your app
2. Click **"User Journey"** tab
3. You'll see a timeline of user events including:
   - `user_registered` events
   - `bet_placed` events
   - `deposit_completed` events
4. Click on any event to see full details

**Option 2: Explore with LogQL**
1. Go to **Explore** in Grafana Cloud
2. Select your Faro data source (Loki)
3. Click **"Drilldown Logs"** to browse available fields
4. Use LogQL to query custom events:

**View all custom events:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event="user_registered" or event_data_event="bet_placed" or event_data_event="deposit_completed"
```

**View specific event type:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event="bet_placed"
```

**Count events by type:**
```logql
sum by (event_data_event) (
  count_over_time({service_name="<your-name>-betwise"} | logfmt | event_data_event!="null" [5m])
)
```

**Note:** Replace `<your-name>` with your actual service name (e.g., `loddell-betwise`)

5. Expand any log entry to see event attributes like:
   - `event_data_event` (the event name: user_registered, bet_placed, deposit_completed)
   - `event_data_stake`
   - `event_data_odds`
   - `event_data_eventId`
   - `event_data_userId` (from event payload)
   - `event_data_email` (from event payload)

### Step 3: Verify User Context

1. In the LogQL results, expand any log entry
2. Look for the user context fields in the parsed logfmt data
3. User data appears in two places:
   - **From event payload**: `event_data_userId`, `event_data_email`, `event_data_startingBalance`
   - **From setUser() call**: `user_id`, `user_email`, `user_attributes_balance`, `user_attributes_accountAge`

**Note:** Some fields may appear with dot notation like `event_data_user.email` depending on how the data is structured

**Success Criteria:**
- You can see custom events in Grafana Explore
- Events include business data (stake amounts, event IDs, etc.)
- User context (ID, email) is attached to events
- **Bonus**: Check the **User Journey** tab in Frontend Observability to see your events in a timeline view!

---

## Summary

**Custom Business Tracking is Working!**

In this lab, you successfully:
- Exposed the Faro API for custom instrumentation
- Tracked user registrations with business context
- Tracked bet placements (revenue events)
- Tracked deposit transactions (payment flow)
- Set user context to link events to specific users

**What This Enables:**
- Conversion funnel analysis (registration → bet → deposit)
- Revenue tracking by user, sport, and event
- User segmentation and behavior analysis
- Session replay with business context
- **User Journey visualization** in Frontend Observability tab

**Key Concepts Learned:**
- `pushEvent()` - Track custom business events
- `setUser()` - Associate events with users
- Event attributes - Add contextual data to events
- User attributes - Track user properties over time

---

**Next Lab:** Lab 4 - Track errors with rich context
