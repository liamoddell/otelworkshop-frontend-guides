# Lab 5: Monitor Performance & Web Vitals

**Duration**: 10 minutes

---

## Overview

Faro automatically collects Web Vitals (LCP, FID, CLS), but we can add custom measurements for:
- **Business operations**: How long do bets take?
- **API calls**: Backend performance
- **Feature performance**: Specific user actions

---

## Part 1: Custom Performance Measurements

### 1. Measure Bet Placement Duration

Edit: `webapp/app/events/[id]/page.tsx`

**Find the bet placement handler** (search for `const handlePlaceBet`):

Add performance measurement. Your complete handler should look like:

```typescript
const handlePlaceBet = async () => {
  if (!user || !selectedBetType || stakeAmount <= 0) return;

  setError('');
  const selectedOdds = getOddsForBetType(selectedBetType);
  const potentialReturn = stakeAmount * selectedOdds;

  const faro = getFaro();
  const startTime = performance.now();

  if (stakeAmount > user.balance) {
    setError('Insufficient balance');
    return;
  }

  if (Math.random() < 0.05) {
    const error = new Error('Failed to place bet');
    setError('Failed to place bet. Please try again.');

    faro?.api.pushError(error, {
      context: {
        eventId: event.id,
        stake: stakeAmount,
        betType: selectedBetType,
        errorType: 'bet_placement_failure',
      },
    });

    return;
  }

  await new Promise((resolve) => setTimeout(resolve, 800));

  const bet: Bet = {
    id: Date.now().toString(),
    eventId: event.id,
    userId: user.id,
    betType: selectedBetType,
    stake: stakeAmount,
    odds: selectedOdds,
    potentialReturn,
    timestamp: new Date().toISOString(),
    status: 'pending',
  };

  addBet(bet);
  updateUser({ ...user, balance: user.balance - stakeAmount });

  faro?.api.pushEvent('bet_placed', {
    eventId: event.id,
    betType: selectedBetType,
    stake: stakeAmount,
    odds: selectedOdds,
    potentialReturn,
    sport: event.sport,
    league: event.league,
  });

  const duration = performance.now() - startTime;
  faro?.api.pushMeasurement({
    type: 'bet_placement_duration',
    values: {
      duration,
    },
    context: {
      eventId: event.id,
      stake: stakeAmount,
      betType: selectedBetType,
    },
  });

  alert('Bet placed successfully!');
  setStakeAmount(0);
  setSelectedBetType(null);
};
```

**Why measure performance?**
- **User experience**: Slow operations = user frustration
- **Business impact**: Slow checkout = abandoned carts
- **Capacity planning**: Identify bottlenecks

**What to measure:**
- **Critical user paths**: Registration, betting, payments
- **API response times**: Backend latency
- **Heavy operations**: Large data loads, complex calculations

---

## Part 2: Understanding Web Vitals

### Automatic Web Vitals (Already Tracked!)

Faro automatically captures Core Web Vitals:

**1. LCP (Largest Contentful Paint)**
- **What**: Time until largest content element is visible
- **Target**: < 2.5 seconds
- **Impact**: User perception of load speed
- **Example**: Event list loading on homepage

**2. FID (First Input Delay)**
- **What**: Time from first user interaction to browser response
- **Target**: < 100 milliseconds
- **Impact**: Perceived responsiveness
- **Example**: Time from clicking "Place Bet" to button responding

**3. CLS (Cumulative Layout Shift)**
- **What**: Visual stability - unexpected layout shifts
- **Target**: < 0.1
- **Impact**: User frustration from moving elements
- **Example**: Button moving as content loads

**4. TTFB (Time to First Byte)**
- **What**: Server response time
- **Target**: < 600 milliseconds
- **Impact**: Initial load speed

---

## Part 3: View Performance Data in Grafana

### 1. Generate Performance Data

In your browser:
1. Navigate through different pages
2. Place several bets (generates `bet_placement_duration` measurements)
3. Interact with forms and buttons

### 2. View Web Vitals

In Grafana Cloud:
1. Go to **Frontend Observability** → Your app
2. Click **"Web Vitals"** tab
3. You should see:
   - **LCP scores** over time
   - **FID measurements**
   - **CLS values**
   - Performance trends by page

### 3. View Custom Measurements

1. Go to **"Explore"** in Grafana Cloud
2. Select your Faro data source (Loki)
3. Click **"Drilldown Logs"** to browse measurement fields
4. Query for custom measurements:

```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="measurement"
| type="bet_placement_duration"
```

**Calculate average duration:**
```logql
avg_over_time({service_name="<your-name>-betwise"}
  | logfmt
  | kind="measurement"
  | type="bet_placement_duration"
  | unwrap value_duration [5m])
```

5. View duration trends:
   - Check `value_duration` field for timing data
   - Check `duration` field (also contains the same value)
   - Analyze slowest operations by filtering for high durations

### 4. Analyze Performance by Page

Filter Web Vitals by:
- **Page URL**: Which pages are slowest?
- **Browser**: Chrome vs Firefox performance
- **Device**: Mobile vs Desktop
- **Time of day**: Peak traffic performance

---

## Performance Optimization Tips

### Using Faro Data to Improve Performance

**1. Identify Slow Pages**
- Sort pages by LCP
- Focus on pages with high traffic + poor performance

**2. Track Improvements**
- Measure before optimization
- Deploy changes
- Compare metrics after deployment

**3. Set Performance Budgets**
```typescript
// Example: Alert if bet placement takes > 2 seconds
if (duration > 2000) {
  faro?.api.pushEvent('performance_budget_exceeded', {
    operation: 'bet_placement',
    duration,
    threshold: 2000,
  });
}
```

---

## What Have We Accomplished?

🎉 **Performance Monitoring is Active!**

You now have:
- ✅ Automatic Web Vitals tracking (LCP, FID, CLS)
- ✅ Custom operation timing (bet placement)
- ✅ Performance trends over time
- ✅ Actionable performance data
- ✅ User experience insights

**Next**: Lab 6 - Track User Actions & Workflows!
