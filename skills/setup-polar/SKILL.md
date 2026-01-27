---
name: setup-polar
description: |
  Interactive onboarding wizard to set up Polar payment integration from scratch. Use this skill when: (1) User wants to "set up Polar" or "integrate Polar" in their project; (2) User is starting fresh with Polar and needs guided setup; (3) User asks "how do I get started with Polar"; (4) User wants to add payments/subscriptions/checkout to their app using Polar; (5) User needs help creating their first Polar product and checkout. This skill walks through MCP installation, authentication, product creation, and generates framework-specific integration code.
---

# Polar Setup Wizard

Interactive setup guide to get Polar payments working in your project.

## Setup Flow

Follow these steps in order:

### Step 1: Check MCP Availability

First, check if the Polar MCP server is available. Try listing products or organizations using MCP tools.

**If MCP tools work:** Skip to Step 3.

**If MCP not available:** Continue to Step 2.

### Step 2: Install Polar MCP

Guide the user to install the Polar MCP server based on their environment:

**Claude Code:**
```bash
# Production
claude mcp add --transport http "Polar" "https://mcp.polar.sh/mcp/polar-mcp"

# Sandbox (recommended for setup)
claude mcp add --transport http "Polar Sandbox" "https://mcp.polar.sh/mcp/polar-sandbox"
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "Polar Sandbox": {
      "url": "https://mcp.polar.sh/mcp/polar-sandbox"
    }
  }
}
```

After installation, the user must:
1. Restart their editor/CLI
2. Authenticate when prompted (redirects to polar.sh)

Once authenticated, verify MCP works by listing organizations.

### Step 3: Gather Project Context

Ask the user:

1. **Framework**: What framework are you using?
   - Next.js (App Router)
   - Next.js (Pages Router)
   - Express.js
   - FastAPI (Python)
   - Other

2. **Environment**: Start with sandbox or production?
   - Sandbox (recommended for new setups)
   - Production

3. **Product type**: What are you selling?
   - Subscription (recurring)
   - One-time purchase
   - Both

### Step 4: Create Organization (if needed)

Use MCP to check if user has an organization. If not, guide them to create one at:
- Sandbox: https://sandbox.polar.sh/start
- Production: https://polar.sh/start

### Step 5: Create First Product

Use MCP to create a test product. Example for subscription:

```
Create a product with:
- Name: "Pro Plan"
- Monthly price: $29
- Organization: [user's org]
```

Store the returned product ID for checkout integration.

### Step 6: Generate Integration Code

Based on the framework from Step 3, generate the integration code.

#### Next.js (App Router)

**Install dependencies:**
```bash
npm install @polar-sh/sdk @polar-sh/nextjs
```

**Environment variables** (`.env.local`):
```bash
POLAR_ACCESS_TOKEN=  # Get from polar.sh/settings (or sandbox)
POLAR_WEBHOOK_SECRET=  # Created in Step 7
POLAR_ORGANIZATION_ID=[org_id from MCP]
```

**Checkout route** (`app/api/checkout/route.ts`):
```typescript
import { Checkout } from "@polar-sh/nextjs";

export const GET = Checkout({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  successUrl: process.env.NEXT_PUBLIC_URL + "/success?checkout_id={CHECKOUT_ID}",
  server: "sandbox", // Change to "production" when ready
});
```

**Webhook route** (`app/api/webhooks/polar/route.ts`):
```typescript
import { Webhooks } from "@polar-sh/nextjs";

export const POST = Webhooks({
  webhookSecret: process.env.POLAR_WEBHOOK_SECRET!,

  onOrderPaid: async (payload) => {
    // Grant access to your product
    console.log("Order paid:", payload.data.id);
    // TODO: Update your database, send email, etc.
  },

  onSubscriptionCreated: async (payload) => {
    // New subscription started
    console.log("Subscription created:", payload.data.id);
  },

  onSubscriptionCanceled: async (payload) => {
    // Subscription will end at period end
    console.log("Subscription canceled:", payload.data.id);
  },
});
```

**Checkout button** (any component):
```typescript
export function CheckoutButton({ productId }: { productId: string }) {
  return (
    <a href={`/api/checkout?products=${productId}`}>
      Subscribe Now
    </a>
  );
}
```

**Success page** (`app/success/page.tsx`):
```typescript
import { Polar } from "@polar-sh/sdk";

const polar = new Polar({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  server: "sandbox",
});

export default async function SuccessPage({
  searchParams,
}: {
  searchParams: { checkout_id?: string };
}) {
  if (!searchParams.checkout_id) {
    return <div>Invalid checkout</div>;
  }

  const checkout = await polar.checkouts.get({
    id: searchParams.checkout_id,
  });

  if (checkout.status !== "succeeded") {
    return <div>Payment not completed</div>;
  }

  return (
    <div>
      <h1>Thank you for your purchase!</h1>
      <p>Order ID: {checkout.id}</p>
    </div>
  );
}
```

#### Express.js

**Install dependencies:**
```bash
npm install @polar-sh/sdk @polar-sh/express zod
```

**Server setup:**
```typescript
import express from "express";
import { Polar } from "@polar-sh/sdk";
import { Checkout, Webhooks } from "@polar-sh/express";

const app = express();
const polar = new Polar({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  server: "sandbox",
});

// Checkout endpoint
app.get("/checkout", Checkout({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  successUrl: "http://localhost:3000/success",
  server: "sandbox",
}));

// Webhooks - MUST be before express.json()
app.post("/webhooks/polar", Webhooks({
  webhookSecret: process.env.POLAR_WEBHOOK_SECRET!,
  onOrderPaid: async (payload) => {
    console.log("Order paid:", payload.data.id);
  },
}));

app.use(express.json());

app.get("/success", async (req, res) => {
  const checkoutId = req.query.checkout_id as string;
  const checkout = await polar.checkouts.get({ id: checkoutId });
  res.json({ status: checkout.status });
});

app.listen(3000);
```

#### FastAPI (Python)

**Install dependencies:**
```bash
pip install polar-sdk fastapi uvicorn
```

**Server setup:**
```python
import os
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import RedirectResponse
from polar_sdk import Polar
from polar_sdk.webhooks import validate_event, WebhookVerificationError

app = FastAPI()
polar = Polar(
    access_token=os.environ["POLAR_ACCESS_TOKEN"],
    server="sandbox",
)

@app.get("/checkout")
async def checkout(product_id: str):
    checkout = polar.checkouts.create(
        product_id=product_id,
        success_url="http://localhost:8000/success?checkout_id={CHECKOUT_ID}",
    )
    return RedirectResponse(checkout.url)

@app.post("/webhooks/polar")
async def webhooks(request: Request):
    body = await request.body()
    try:
        event = validate_event(
            payload=body,
            headers=dict(request.headers),
            secret=os.environ["POLAR_WEBHOOK_SECRET"],
        )
        if event.type == "order.paid":
            print(f"Order paid: {event.data.id}")
        elif event.type == "subscription.created":
            print(f"Subscription created: {event.data.id}")
    except WebhookVerificationError:
        raise HTTPException(status_code=400, detail="Invalid signature")
    return {"received": True}

@app.get("/success")
async def success(checkout_id: str):
    checkout = polar.checkouts.get(id=checkout_id)
    return {"status": checkout.status}
```

### Step 7: Set Up Webhooks

1. Start local development server
2. Start ngrok:
   ```bash
   ngrok http 3000  # or 8000 for FastAPI
   ```
3. Copy the ngrok URL (e.g., `https://abc123.ngrok.io`)

Use MCP to create a webhook endpoint, or guide user to dashboard:
- Sandbox: https://sandbox.polar.sh → Settings → Webhooks
- Add endpoint: `https://[ngrok-url]/webhooks/polar` (or `/api/webhooks/polar` for Next.js)
- Select events: `order.paid`, `subscription.created`, `subscription.canceled`
- Copy the webhook secret to `.env.local`

### Step 8: Test the Integration

1. Start the dev server
2. Visit checkout URL with product ID:
   - Next.js: `http://localhost:3000/api/checkout?products=[PRODUCT_ID]`
   - Express: `http://localhost:3000/checkout?products=[PRODUCT_ID]`
   - FastAPI: `http://localhost:8000/checkout?product_id=[PRODUCT_ID]`

3. Complete checkout using test card: `4242 4242 4242 4242`

4. Verify webhook received in terminal logs

5. Check success page displays correctly

### Step 9: Next Steps

Once basic setup works, suggest:

1. **Add customer portal** - Let users manage subscriptions
2. **Link to your auth** - Pass `customerExternalId` in checkout
3. **Add benefits** - License keys, file downloads, Discord roles
4. **Go production** - Switch from sandbox to production

Refer to `polar-developer-guide` skill for detailed API reference.
