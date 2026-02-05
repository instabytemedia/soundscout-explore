# Optional: Payments (Stripe)

> **Feature Type:** OPTIONAL
> **Complexity:** High
> **Dependencies:** Auth, Environment Variables
> **Agent:** Payments Agent 💳
> **Ziel:** Stripe Integration für One-Time Payments

---

## Voraussetzungen

- Stripe Account: https://dashboard.stripe.com
- Stripe CLI: `brew install stripe/stripe-cli/stripe`

---

## Tasks

### Task P.1: Stripe Setup

**Environment Variables:**

```.env
# Stripe
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

```

**Dependencies:**

```bash
npm install stripe @stripe/stripe-js
```

- [ ] Stripe Account erstellt
- [ ] API Keys in .env
- [ ] Dependencies installiert

### Task P.2: Stripe Client

**Datei:** `lib/stripe/client.ts`

```typescript
import Stripe from "stripe";

if (!process.env.STRIPE_SECRET_KEY) {
  throw new Error("STRIPE_SECRET_KEY is not set");
}

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: "2023-10-16",
  typescript: true,
});
```

**Datei:** `lib/stripe/config.ts`

```typescript

export const PRODUCTS = {
  lifetime: {
    name: "SoundScout Explore Lifetime",
    priceId: process.env.STRIPE_PRICE_LIFETIME!,
    price: 99,
  },
} as const;

```

- [ ] Stripe Client erstellt
- [ ] Config mit Plans/Products


### Task P.3: Checkout Session (One-Time)

**Datei:** `app/api/stripe/checkout/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { createClient } from "@/lib/supabase/server";
import { stripe } from "@/lib/stripe/client";
import { PRODUCTS } from "@/lib/stripe/config";

export async function POST(req: NextRequest) {
  try {
    const supabase = await createClient();
    const { data: { user } } = await supabase.auth.getUser();

    if (!user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const session = await stripe.checkout.sessions.create({
      customer_email: user.email,
      mode: "payment",
      payment_method_types: ["card"],
      line_items: [{ price: PRODUCTS.lifetime.priceId, quantity: 1 }],
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?success=true`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing?canceled=true`,
      metadata: { userId: user.id },
    });

    return NextResponse.json({ url: session.url });
  } catch (error) {
    console.error("Checkout error:", error);
    return NextResponse.json(
      { error: "Failed to create checkout session" },
      { status: 500 }
    );
  }
}
```

- [ ] Checkout Route erstellt


### Task P.4: Webhook Handler

**Datei:** `app/api/stripe/webhook/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";
import { stripe } from "@/lib/stripe/client";
import { createClient } from "@supabase/supabase-js";
import type Stripe from "stripe";

// Use service role for webhook (no user context)
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

export async function POST(req: NextRequest) {
  const body = await req.text();
  const headersList = await headers();
  const signature = headersList.get("stripe-signature")!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error("Webhook signature verification failed");
    return NextResponse.json({ error: "Invalid signature" }, { status: 400 });
  }

  try {
    switch (event.type) {

      case "checkout.session.completed": {
        const session = event.data.object as Stripe.Checkout.Session;
        const userId = session.metadata?.userId;

        if (userId) {
          await supabase
            .from("profiles")
            .update({
              has_lifetime_access: true,
              purchased_at: new Date().toISOString(),
            })
            .eq("id", userId);
        }
        break;
      }

      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error("Webhook handler error:", error);
    return NextResponse.json(
      { error: "Webhook handler failed" },
      { status: 500 }
    );
  }
}
```

- [ ] Webhook Route erstellt

### Task P.5: Database Schema Update

**SQL für profiles Tabelle erweitern:**

```sql
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS stripe_customer_id TEXT;

ALTER TABLE profiles ADD COLUMN IF NOT EXISTS has_lifetime_access BOOLEAN DEFAULT FALSE;
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS purchased_at TIMESTAMPTZ;

```

- [ ] Schema erweitert



### Task P.7: Checkout Button Component

**Datei:** `components/payments/CheckoutButton.tsx`

```typescript
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";

interface CheckoutButtonProps {
  
  children: React.ReactNode;
}

export function CheckoutButton({ children }: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false);

  const handleCheckout = async () => {
    setLoading(true);
    try {
      const res = await fetch("/api/stripe/checkout", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({  }),
      });

      const { url, error } = await res.json();

      if (error) {
        alert(error);
        return;
      }

      window.location.href = url;
    } catch (error) {
      alert("Fehler beim Checkout");
    } finally {
      setLoading(false);
    }
  };

  return (
    <Button onClick={handleCheckout} disabled={loading}>
      {loading ? "Wird geladen..." : children}
    </Button>
  );
}
```

- [ ] Checkout Button Component erstellt

### Task P.8: Stripe CLI für lokales Testing

```bash
# Login
stripe login

# Webhook forwarding
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Test Events
stripe trigger checkout.session.completed

```

- [ ] Stripe CLI funktioniert
- [ ] Webhook Events werden empfangen

---

## Checkpoint

**Payments sind abgeschlossen wenn:**

- [ ] Stripe Keys in .env
- [ ] Checkout Session funktioniert
- [ ] Webhook empfängt Events
- [ ] Database wird aktualisiert

- [ ] User kann bezahlen (Test Mode)

**Test:**
1. Klicke auf Checkout Button
2. Nutze Stripe Test Card: 4242 4242 4242 4242
3. Prüfe Webhook Logs
4. Prüfe Database Update

---

**Diese Datei ist optional. Nur umsetzen wenn Payments benötigt werden.**
