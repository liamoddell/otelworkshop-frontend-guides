# Frontend Observability Workshop

**Learn Frontend Observability with Grafana Faro SDK**

This workshop teaches you how to instrument a production-like web application with Grafana Faro, enabling comprehensive observability for frontend applications.

---

## Workshop Structure

### Application: BetWise Betting Platform

A realistic Next.js sports betting application featuring:
- User registration and authentication
- Sports event browsing and filtering
- Real-time betting with odds calculation
- Account management and transaction history
- Built-in error scenarios for observability demos

### Workshop Format

**Duration**: ~90 minutes
**Language**: TypeScript + Next.js 16 (App Router)
**Instrumentation**: Grafana Faro Web SDK v2.0.2

---

## What You'll Learn

✅ **Automatic Instrumentation**
- Page views and navigation tracking
- User session management
- JavaScript error capture
- Web Vitals (LCP, FID, CLS)

✅ **Custom Instrumentation**
- Business event tracking (registrations, bets, deposits)
- User context and identification
- Custom error reporting with rich context
- Performance measurements

✅ **Observability in Practice**
- Analysing user sessions in Grafana Cloud
- Debugging errors with full context
- Monitoring conversion funnels
- Performance optimization

---

## Workshop Labs

Workshop lab instructions are provided via GitHub documentation.

### Lab Overview:

1. **Lab 1**: Run the Application (15 min)
   - Understand user journeys
   - Experience error scenarios

2. **Lab 2**: Configure Grafana Cloud & Basic Instrumentation (30 min)
   - Create Faro application in Grafana Cloud
   - Install and initialize Faro SDK
   - Verify automatic instrumentation

3. **Lab 3**: Add Custom Events (20 min)
   - Track user registrations
   - Monitor bet placements
   - Log deposit transactions

4. **Lab 4**: Track Errors & Exceptions (10 min)
   - Capture validation errors
   - Report API failures with context

5. **Lab 5**: Monitor Performance (10 min)
   - Measure operation durations
   - Analyze Web Vitals

6. **Lab 6**: User Actions (5 min)
   - Automatic and Custom Approaches
   - Why Actions vs. Events
   
7. **Lab 7**: Explore Data in Grafana Cloud (5 min)
   - Build dashboards
   - Error analysis
   - Grafana Assistant

---

## Built-in Error Scenarios

The BetWise app includes realistic error scenarios to demonstrate observability value:

| Scenario | Trigger | Rate | Observability Goal |
|----------|---------|------|-------------------|
| Registration Server Error | Form submission | 10% | Track signup funnel drop-off |
| Bet Placement Failure | Place bet | 5% | Monitor transaction failures |
| Payment Processing Error | Deposit funds | 3% | Alert on payment issues |
| Form Validation Errors | Invalid input | 100% | Improve UX with error context |


---

## Technology Stack

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS v4
- **Observability**: Grafana Faro Web SDK v2.0.2
- **State Management**: LocalStorage (client-side only)

---

## Prerequisites for Workshop

Users will need:

1. **Grafana Cloud Account** (provided by facilitator)
   - Frontend Observability application created
   - Faro collector URL

2. **Development Environment**
   - Workshop container (recommended)
   - OR: Node.js 20+ installed locally

3. **Browser** with DevTools for debugging

---

## Reference Implementation

The `webapp-reference/` directory contains a fully instrumented version with:

- ✅ Faro SDK initialized
- ✅ Custom events for all business actions
- ✅ User context tracking
- ✅ Error reporting with rich context
- ✅ Performance measurements

Users can reference this if they get stuck during the workshop.

---

## Troubleshooting

**Application won't start:**
```bash
# Check port availability
lsof -i :3001

# Clean install
rm -rf node_modules package-lock.json
npm install
```

**No data in Grafana Cloud:**
- Verify `.env.local` has correct `NEXT_PUBLIC_FARO_URL`
- Check browser console for Faro initialization messages
- Ensure Faro collector URL includes `/collect/` path

**Build errors:**
```bash
# Verify Node.js version
node --version  # Should be 20+

# Clean build
rm -rf .next
npm run build
```
---

## License

This workshop is provided for educational purposes.

---

## Support

For questions or issues:
- Check the GitHub documentation for detailed lab instructions
- Review `webapp-reference/` for working examples
- Contact your workshop facilitator
