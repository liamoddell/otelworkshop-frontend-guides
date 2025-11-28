# Lab 2: Configure Grafana Cloud & Add Basic Instrumentation

**Duration**: 30 minutes

---

## What is Grafana Faro?

Grafana Faro is a frontend observability SDK that automatically captures:
- Page views and navigation
- User interactions (clicks, form submissions)
- JavaScript errors and exceptions
- Web vitals (performance metrics)
- Custom events and user context

---

## Create a Faro Application in Grafana Cloud

1. **Log in to Grafana Cloud** (credentials provided by facilitator)

2. **Navigate to Frontend Observability**:
   - Click "Frontend Observability" in the left sidebar
   - Click "Create Application"

3. **Configure your application**:
   - Application name: `betwise-<your-name>`
   - Click "Create"
   - Copy the provided collector URL

4. **Configure CORS settings**:
   - In your Faro application settings, add your domain: `http://localhost:3001`
   - This allows your local app to send telemetry data to Grafana Cloud

5. **Create environment file**:
   ```bash
   cd /home/project/webapp
   nano .env.local
   ```

6. **Add your Faro collector URL**:
   ```bash
   NEXT_PUBLIC_FARO_URL=https://faro-collector-prod-us-east-0.grafana.net/collect/YOUR_APP_ID
   ```

---

## Add Faro SDK

**Why do we use the SDK?** The Faro Web SDK provides automatic instrumentation for frontend applications, capturing user interactions, errors, and performance metrics without requiring manual instrumentation.

### 1. Faro SDK Dependencies (Pre-installed)

The Faro SDK is already installed in your `package.json`:

```json
"@grafana/faro-web-sdk": "^2.0.2",
"@grafana/faro-web-tracing": "^2.0.2"
```

**Why tracing?** Tracing captures the journey of requests through your application, showing you:
- How long API calls take
- Which requests fail or are slow
- Dependencies between frontend and backend operations

### 2. Create Faro initialization file (`lib/faro.ts`)

**Why this file?** This initializes the Faro SDK once when your app starts. The `getWebInstrumentations()` automatically enables tracking for page views, errors, and Web Vitals without any additional code.

```typescript
import { initializeFaro, getWebInstrumentations } from '@grafana/faro-web-sdk';
import { TracingInstrumentation } from '@grafana/faro-web-tracing';

let faroInstance: ReturnType<typeof initializeFaro> | null = null;

export function initFaro() {
  // Only run in the browser (Next.js also runs code on the server)
  if (typeof window === 'undefined') {
    return null;
  }

  // Return existing instance if already initialized
  if (faroInstance) {
    return faroInstance;
  }

  faroInstance = initializeFaro({
    url: process.env.NEXT_PUBLIC_FARO_URL || '',
    app: {
      name: 'betwise-betting-app',
      version: '1.0.0',
      environment: 'workshop',
    },
    // getWebInstrumentations() automatically tracks:
    // - Page views
    // - Errors and console logs
    // - Web Vitals (LCP, FID, CLS)
    // - User interactions
    instrumentations: [
      ...getWebInstrumentations(),
      // TracingInstrumentation captures distributed traces:
      // - Fetch/XHR requests (API calls)
      // - Request duration and status
      // - Frontend → Backend correlation
      new TracingInstrumentation(),
    ],
  });

  return faroInstance;
}
```

### 3. Create Faro Provider (`components/FaroProvider.tsx`)

**Why a Provider?** In Next.js, we need a Client Component to run browser-side code. The `useEffect` hook ensures Faro initializes after the page loads.

```typescript
'use client';

import { useEffect } from 'react';
import { initFaro } from '@/lib/faro';

export default function FaroProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    // Initialize Faro when the component mounts (page loads)
    initFaro();
  }, []);

  return <>{children}</>;
}
```

### 4. Update root layout (`app/layout.tsx`)

**Why wrap the app?** By wrapping all children with FaroProvider in the root layout, we ensure Faro tracks every page in your application automatically.

Add the import at the top:
```typescript
import FaroProvider from "@/components/FaroProvider";
```

Then wrap `{children}` with the FaroProvider:
```typescript
export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className="antialiased" suppressHydrationWarning>
        <FaroProvider>
          {children}
        </FaroProvider>
      </body>
    </html>
  );
}
```

**Note:** Next.js will automatically hot-reload these changes - no need to restart the dev server!

---

## Verify in Grafana Cloud

### 1. Generate Traffic

Navigate through the app at http://localhost:3001:
- Browse homepage
- Register a new user
- Place bets
- View account page

**Why?** Each action triggers traces for any fetch/XHR requests.

### 2. Open Grafana Cloud

- Go to **Frontend Observability**
- Select your application: `betwise-<your-name>`

### 3. You Should See:

**📊 Dashboard Tab:**
- Page views count
- User sessions
- Web vitals (LCP, FID, CLS)
- Navigation timings

**🔍 Traces Tab (NEW!):**
- HTTP requests made by the frontend
- Request duration and status codes
- Failed requests highlighted
- Trace spans showing request flow

**Note:** In a real app with a backend, traces would show the full journey from frontend → backend → database. Since BetWise is client-side only, you'll see traces for internal Next.js requests and any external API calls.

---

**Next**: [Lab 3: Add Custom Events & User Context](lab3.md)
