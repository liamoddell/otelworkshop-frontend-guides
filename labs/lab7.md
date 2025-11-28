# Lab 7: Explore Data in Grafana Cloud

**Duration**: 15 minutes

---

## Service Overview

### 1. Access Your Data in Grafana Cloud

**Frontend Observability:**
- Navigate to **Frontend Observability** → Your app
- View automatic metrics like page views, errors, and Web Vitals
- **User Journey tab**: Timeline view of all custom events
  - See `user_registered`, `bet_placed`, `deposit_completed` events
  - Click on events to see full context
  - Filter by user, time range, or event type

**Explore (For custom queries):**
- Navigate to **Explore** in Grafana Cloud
- Select your Faro data source (Loki)
- Click **"Drilldown Logs"** to browse available fields
- Use LogQL queries to analyze custom events, errors, and measurements

**Important Notes:**
- Faro logs use **logfmt** format (not JSON)
- Always use `| logfmt` parser in queries
- Use `!="null"` instead of `!=""` to filter out null values
- Field names in Grafana Cloud are prefixed:
  - Custom events: `event_data_<field>` (e.g., `event_data_event`, `event_data_stake`)
  - Error context: `context_<field>` (e.g., `context_errorType`, `context_eventId`)
  - Measurements: Filter with `kind="measurement"`, then `type` field for measurement name, `value_<field>` for values (e.g., `value_duration`)
  - User data (two sources):
    - From event payload: `event_data_userId`, `event_data_email`
    - From setUser(): `user_id`, `user_email`, `user_attr_<field>` (e.g., `user_attr_balance`, `user_attr_accountAge`)
  - Some fields may use dot notation: `event_data_user.email`

### 2. What Data is Available:

- **Custom Events**: user_registered, bet_placed, deposit_completed
- **Errors**: Validation errors, bet failures, payment failures
- **Performance Measurements**: bet_placement_duration
- **Web Vitals**: LCP, FID, CLS (automatic)
- **User Context**: User ID, email, custom attributes
- **Traces**: HTTP requests (via TracingInstrumentation)

---

## Analyze User Sessions

**Quick View: Use the User Journey Tab**

The easiest way to view user sessions is in **Frontend Observability** → **User Journey** tab:
- Timeline view of all events
- Filter by user email or ID
- Click events to see full context
- See complete user journey from registration to bets to deposits

### 1. Query Session Data with LogQL

For custom queries and aggregations, use LogQL in Explore:

**View all events for a specific user (by ID):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| user_id="<user-id>"
```

**View all events for a specific user (by email):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| user_email="Keith@David.com"
```

**Track user journey (all events for a user):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event!="null"
| user_id="<user-id>"
```

**Track specific events for a user:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event=~"user_registered|bet_placed|deposit_completed"
| user_id="<user-id>"
```

**Note:** User data appears as both `user_email` (from setUser) and `event_data_email` (from event payload)

### 2. Session Analysis

Query for session patterns:
- **Timeline**: Order events by timestamp
- **Pages visited**: Filter by page URL fields
- **Events triggered**: Filter by `event_data_event` field
- **Errors encountered**: Filter by `context_errorType`
- **Performance metrics**: Filter by `kind="measurement"` and `type` field

### 3. User Behavior Patterns

**Find users with errors:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="error"
| user_id != "null"
```

**Track conversion funnel (all custom events):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event!="null"
```

---

## Create Custom Queries

### Using LogQL in Grafana Explore

Navigate to **Explore** tab to run custom queries:

### 1. Find Users with Bet Placement Errors

```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType="bet_placement_failure"
```

**Use case**: Identify affected users for support outreach

### 2. Track Conversion Funnel

**View registration and bet events:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event=~"user_registered|bet_placed"
```

**Count events by type (conversion funnel):**
```logql
sum by (event_data_event) (
  count_over_time({service_name="<your-name>-betwise"}
    | logfmt
    | event_data_event=~"user_registered|bet_placed|deposit_completed" [5m])
)
```

**Track complete user journey (all events):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event!="null"
```

**Use case**: Calculate registration → first bet → deposit conversion rate

### 3. Monitor High-Value Bets

```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event="bet_placed"
| event_data_stake > "100"
```

**Count bets by sport:**
```logql
sum by (event_data_sport) (
  count_over_time({service_name="<your-name>-betwise"}
    | logfmt
    | event_data_sport!="null" [$__auto])
)
```

**Use case**: Track VIP user activity and popular sports

### 4. Find Slow Operations

```logql
{service_name="<your-name>-betwise"}
| logfmt
| kind="measurement"
| type="bet_placement_duration"
| value_duration > 2000
```

**Use case**: Identify performance issues

### 5. Payment Failures

```logql
{service_name="<your-name>-betwise"}
| logfmt
| context_errorType="payment_failure"
```

**Use case**: Monitor payment failures and calculate lost revenue

---

## Build Dashboards

### 1. Create a Custom Dashboard

1. Click **"Dashboards"** → **"New Dashboard"**
2. Add panels for:

**Business Metrics Panel:**
- Total registrations (count of `user_registered`)
- Total bets placed (count of `bet_placed`)
- Total deposits (sum of `deposit_completed.amount`)

**Error Tracking Panel:**
- Error rate over time
- Top 5 error types
- Errors by page

**Performance Panel:**
- Average bet placement duration
- Web Vitals trends
- Slowest pages

### 2. Share Dashboards

- Share with team members
- Set up as homepage for monitoring
- Export/import dashboard JSON

---

## Set Up Alerts

### 1. Create Alert Rules

In Grafana Cloud, set up alerts for:

**Critical Business Metrics:**
```
Alert: High Bet Failure Rate
Condition: bet_placement_failure count > 10 in 5 minutes
Action: Notify #ops-team in Slack
```

**Performance Degradation:**
```
Alert: Slow Bet Placement
Condition: avg(bet_placement_duration) > 2000ms for 10 minutes
Action: Create PagerDuty incident
```

**Payment Issues:**
```
Alert: Payment Failures Spike
Condition: payment_failure count > 5 in 5 minutes
Action: Email finance team + Slack #payments
```

### 2. Configure Notification Channels

1. **Integrations** → **Contact Points**
2. Add:
   - Slack webhook
   - Email recipients
   - PagerDuty integration
   - Webhook for custom integrations

---

## Advanced Features

### 1. User Journey Analysis

Track complete conversion funnels:
1. **Registration** → `user_registered`
2. **First Bet** → `bet_placed` (within 24 hours)
3. **First Deposit** → `deposit_completed`
4. **Retention** → Active after 7 days

### 2. Cohort Analysis

Compare user groups:
- Users who encountered errors vs. error-free users
- High-value bettors vs. casual users
- Mobile vs. desktop users

### 3. Correlation Analysis

Find patterns:
- Do slow page loads increase errors?
- Do bet failures lead to user churn?
- Which features drive deposits?

---

## Pro Tips for Querying Events

### Using event_data_event for Cleaner Queries

Now that you've added `event: 'event_name'` to your pushEvent calls, you can write cleaner queries:

**Filter by specific event:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event="bet_placed"
```

**Multiple events (regex):**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event=~"bet_placed|deposit_completed"
```

**All custom events:**
```logql
{service_name="<your-name>-betwise"}
| logfmt
| event_data_event!="null"
```

**Best Practice:** Always include `event: 'event_name'` as an attribute when using `pushEvent()` to make filtering and querying easier.

---

## Summary - What You Learned

### ✅ Core Observability Concepts

1. **Automatic Instrumentation**
   - Faro SDK setup and configuration
   - Web Vitals tracking (LCP, FID, CLS)
   - Automatic error capture
   - Distributed tracing (HTTP requests)

2. **Custom Instrumentation**
   - Business event tracking (`pushEvent`)
   - User context management (`setUser`)
   - Error reporting with context (`pushError`)
   - Performance measurements (`pushMeasurement`)

3. **Data Analysis**
   - Session replay and user journeys
   - Custom LogQL queries
   - Dashboard creation
   - Alert configuration

### 🎯 Business Value

You can now:
- **Track revenue metrics**: Bets, deposits, conversions
- **Monitor user experience**: Performance, errors, drop-offs
- **Debug issues faster**: Full context with errors
- **Make data-driven decisions**: Real user behavior insights

### 📊 Key Metrics You're Tracking

**Business Metrics:**
- User registrations
- Bet placements (amount, frequency)
- Deposit transactions
- Conversion funnels

**Technical Metrics:**
- Page load performance (Web Vitals)
- Error rates and types
- API response times (via tracing)
- User session quality

---

## Congratulations! 🎉

You've successfully instrumented a production-like web application with comprehensive frontend observability!

You now have the skills to:
- ✅ Set up frontend observability from scratch
- ✅ Track custom business events
- ✅ Monitor user journeys and conversions
- ✅ Debug issues with rich context
- ✅ Analyze performance and Web Vitals
- ✅ Capture distributed traces
- ✅ Build dashboards and alerts

**Go forth and observe all the things!** 🔭
