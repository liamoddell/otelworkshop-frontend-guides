# Lab 1: Run the Application

**Duration**: 15 minutes

---

## Objectives
- Understand the BetWise application user journeys
- Run the application locally
- Experience built-in error scenarios

## Steps

### 1. Access the Workshop Environment

Open Theia IDE in your browser: **http://localhost:3000**

You should see:
```
/home/project/
├── webapp/              ← You'll work here
├── webapp-reference/    ← Reference solution
└── README.md
```

### 2. Open Integrated Terminal

In Theia:
- **Menu**: Terminal → New Terminal
- Or click the terminal panel at the bottom

### 3. Verify Node.js 20

```bash
node --version
# Should show: v20.18.1
```

### 4. Start the BetWise Application

```bash
cd /home/project/webapp
npm run dev
```

Wait for the message: `✓ Ready in ...ms`

### 5. Explore the Application

Open **http://localhost:3001** in your browser

**Test these user journeys:**

1. **Browse Events**
   - Homepage shows available sports events
   - Filter by sport type (Football, Basketball, Tennis)

2. **Register an Account**
   - Click "Register" button
   - Fill in: Name, Email, Password
   - Starting balance: $1000
   - **Note**: 10% of registrations will fail (simulated error)

3. **Place a Bet**
   - Click on any event
   - Select bet type (Home Win / Draw / Away Win)
   - Enter stake amount
   - Review potential return
   - Place bet
   - **Note**: 5% of bets will fail (simulated error)

4. **Make a Deposit**
   - Click "Account" in navigation
   - View transaction history
   - Enter deposit amount
   - Submit deposit
   - **Note**: 3% of deposits will fail (simulated error)

### 6. Understanding Error Scenarios

The app includes realistic error scenarios:

| Scenario | Trigger | Failure Rate | Purpose |
|----------|---------|--------------|---------|
| Registration Error | Form submission | 10% | Track signup funnel drop-off |
| Bet Placement Error | Place bet | 5% | Monitor transaction failures |
| Payment Error | Deposit funds | 3% | Alert on payment issues |
| Validation Errors | Invalid input | 100% | Improve UX with error context |

**Key Observation**: Currently, these errors happen silently. There's no visibility into:
- How many users are affected?
- Which errors occur most frequently?
- What user actions lead to errors?

This is what we'll solve with Grafana Faro!

---

**Next**: Lab 2 - Configure Grafana Cloud & Basic Instrumentation
