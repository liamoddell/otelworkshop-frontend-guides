# Lab 4: Track Errors & Exceptions

**Duration**: 10 minutes

---

## Overview

Faro automatically captures JavaScript errors, but we can manually report errors for:
- **Better context**: Add business-specific data
- **Validation errors**: Track form issues
- **Business logic errors**: Failed transactions
- **API failures**: Backend integration issues

---

## Part 1: Track Validation Errors

### 1. Registration Form Validation

Edit: `webapp/app/register/page.tsx`

**Find the email validation check** (search for `!formData.email.includes('@')`):

Replace the existing error handling with:

```typescript
if (!formData.email.includes('@')) {
  setError('Please enter a valid email address');

  const faro = getFaro();
  faro?.api.pushError(new Error('Email validation failed'), {
    context: {
      errorType: 'validation',
      formField: 'email',
    },
  });

  return;
}
```

**Why track validation errors?**
- **UX improvements**: Which fields confuse users?
- **Form optimization**: Reduce abandonment
- **Data quality**: Track invalid submissions

**Error context best practices:**
- Include form field names
- Add validation rule that failed
- Don't send actual user input (privacy)

---

## Part 2: Track Bet Placement Failures

### 1. Bet Placement Error

Edit: `webapp/app/events/[id]/page.tsx`

**Find the bet placement failure simulation** (search for `Math.random() < 0.05`):

Replace the existing error handling:

```typescript
if (Math.random() < 0.05) {
  const error = new Error('Failed to place bet');
  setError('Failed to place bet. Please try again.');

  const faro = getFaro();
  faro?.api.pushError(error, {
    context: {
      eventId: event.id,
      stake: stakeAmount,
      betType: selectedBetType,
      odds: selectedOdds,
      errorType: 'bet_placement_failure',
    },
  });

  return;
}
```

**Why track bet failures?**
- **Revenue impact**: Lost bets = lost revenue
- **System reliability**: Track failure rate over time
- **User experience**: High-value bets failing = user churn

**Critical context to include:**
- **Stake amount**: How much revenue was lost?
- **Event details**: Which events have issues?
- **User context**: Already captured by Faro

---

## Part 3: Track Payment Failures

### 1. Deposit Processing Error

Edit: `webapp/app/account/page.tsx`

**Find the deposit failure simulation** (search for `Math.random() < 0.03`):

Replace the existing error handling:

```typescript
if (Math.random() < 0.03) {
  const error = new Error('Payment processing failed');
  setError('Payment processing failed. Please try again.');

  const faro = getFaro();
  faro?.api.pushError(error, {
    context: {
      amount,
      currentBalance: user?.balance,
      errorType: 'payment_failure',
      paymentMethod: 'credit_card',
    },
  });

  return;
}
```

**Why track payment failures?**
- **Critical business impact**: Payment failures block revenue
- **Customer support**: Proactive outreach to affected users
- **Payment provider issues**: Track by provider/method

---

## Part 4: View Errors in Grafana Cloud

### 1. Generate Errors

In your browser:
1. **Invalid email**: Try registering with "test" (no @)
2. **Bet failures**: Place multiple bets (5% will fail)
3. **Payment failures**: Make multiple deposits (3% will fail)

### 2. View in Grafana Cloud

**Option 1: Frontend Observability (if Errors tab exists)**
1. Go to **Frontend Observability** → Your app
2. Look for **"Errors"** section
3. View error frequency and messages

**Option 2: Use Explore with LogQL**
1. Go to **Explore** in Grafana Cloud
2. Select your Faro data source (Loki)
3. Click **"Drilldown Logs"** to browse available error fields
4. Use LogQL to query errors:

```logql
{service_name="<your-name>-betwise"}
| logfmt
| level="error" or kind="error"
```

**Filter by error type:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType="bet_placement_failure"
```

**Count errors by type:**
```logql
sum by (context_errorType) (
  count_over_time({service_name="<your-name>-betwise"} | logfmt | context_errorType!="null" [5m])
)
```

**Note:** Faro logs use **logfmt** format, not JSON. Error context fields appear as `context_<field>` (e.g., `context_errorType`, `context_stake`, `context_eventId`)

### 3. Analyze Error Context

Expand any error log entry to see:
- **Error message**: "Failed to place bet"
- **Context data**: `context_stake`, `context_eventId`, `context_betType`, `context_errorType`
- **User information**:
  - From setUser(): `user_id`, `user_email`
  - May also see: `event_data_user.email` or similar dot notation fields
- **Timestamp**: When did it happen?

### 4. Query Errors by Type

Use LogQL to group and analyze errors:

**View all errors with types:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType != "null"
```

**Find validation errors:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType="validation"
```

**Find payment failures:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType="payment_failure"
```

---

## Error Tracking Best Practices

### What to Include in Error Context

✅ **Do include:**
- Business identifiers (order ID, bet ID)
- Amounts and values
- User actions leading to error
- System state (balance, inventory)

❌ **Don't include:**
- Passwords
- Credit card numbers
- Personal identifiable information (full names, addresses)
- Session tokens

### Error Severity Levels

Consider adding severity:
```typescript
faro?.api.pushError(error, {
  context: {
    severity: 'critical', // or 'high', 'medium', 'low'
    amount: 1000, // High-value transaction
    errorType: 'payment_failure',
  },
});
```

---

## What Have We Accomplished?

🎉 **Error Tracking with Context!**

You now have:
- ✅ Validation error tracking
- ✅ Transaction failure monitoring
- ✅ Payment error alerts
- ✅ Rich context for debugging
- ✅ User-specific error analysis

**Next**: Lab 5 - Monitor performance metrics!
