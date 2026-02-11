---
name: x402-stacks
description: Use when implementing HTTP 402 payment-required endpoints on Stacks blockchain, setting up micropayment clients with STX or sBTC tokens, or handling x402 protocol wallet setup
---

# x402-stacks SDK

## Overview

x402-stacks enables **automatic HTTP-level payments** for APIs using STX or sBTC tokens on Stacks blockchain. Client pays automatically via axios interceptor, server protects endpoints with Express middleware.

**Core principle:** HTTP 402 Payment Required becomes a working protocol.

## Installation

```bash
npm i x402-stacks
```

## Wallet Setup (Two Options Only)

**Critical: Choose ONE approach based on your situation.**

### Option 1: Load Existing Wallet
**When:** You have a private key already

```typescript
import { privateKeyToAccount } from 'x402-stacks';

const account = privateKeyToAccount(
  process.env.PRIVATE_KEY!,
  'testnet'  // or 'mainnet'
);
```

### Option 2: Generate New Wallet
**When:** You don't have a wallet yet

```typescript
import { generateKeypair, privateKeyToAccount } from 'x402-stacks';

// Generate once, save the private key
const keypair = generateKeypair('testnet');
console.log('Save this:', keypair.privateKey);
console.log('Fund at:', `https://explorer.stacks.co/sandbox/faucet?chain=testnet`);

// Use it
const account = privateKeyToAccount(keypair.privateKey, 'testnet');
```

**Important:** After generating, save private key to `.env` file and fund via faucet before making payments.

## Quick Start: Client (Pays Automatically)

```typescript
import axios from 'axios';
import { wrapAxiosWithPayment, privateKeyToAccount } from 'x402-stacks';

const account = privateKeyToAccount(process.env.PRIVATE_KEY!, 'testnet');

const api = wrapAxiosWithPayment(
  axios.create({ baseURL: 'http://localhost:3000', timeout: 60000 }),
  account
);

// Payment happens automatically on 402 response
const response = await api.get('/api/premium-data');
```

## Quick Start: Server (Requires Payment)

```typescript
import express from 'express';
import { paymentMiddleware, STXtoMicroSTX, STACKS_NETWORKS } from 'x402-stacks';

const app = express();

app.get('/api/premium-data',
  paymentMiddleware({
    amount: STXtoMicroSTX(0.00001),
    payTo: process.env.SERVER_ADDRESS!,
    network: STACKS_NETWORKS.TESTNET,  // or STACKS_NETWORKS.MAINNET
    asset: 'STX',
    facilitatorUrl: 'https://facilitator.stacksx402.com',  // Free facilitator
  }),
  (req, res) => {
    res.json({ data: 'Premium content' });
  }
);
```

## Registering on x402scan

**Make your service discoverable** by registering it on x402scan.

### Registration Endpoint

```bash
POST https://x402scan.com/api/v1/resources
Content-Type: application/json

{
  "url": "https://your-service.com/api/your-endpoint"
}
```

### Your Endpoint Must Return

When x402scan validates your URL, it expects HTTP 402 (or 200) with:

```json
{
  "x402Version": 1,
  "name": "My AI Service",
  "image": "https://your-service.com/logo.png",
  "accepts": [{
    "scheme": "exact",
    "network": "stacks",
    "asset": "STX",
    "maxAmountRequired": "1000000",
    "resource": "https://your-service.com/api/your-endpoint",
    "description": "What this service does",
    "mimeType": "application/json",
    "payTo": "SP2...YOUR_ADDRESS",
    "maxTimeoutSeconds": 60,
    "outputSchema": {
      "input": {
        "type": "https",
        "method": "GET",
        "queryParams": {
          "q": { "type": "string", "required": true, "description": "Query param" }
        }
      },
      "output": {
        "type": "object",
        "properties": { "result": { "type": "string" } }
      }
    }
  }]
}
```

### Validation Requirements

| Must Have | Error if Missing |
|-----------|------------------|
| HTTPS URL | `invalid_url` |
| Non-empty `name` | `invalid_name` |
| At least 1 `accepts` entry | `empty_accepts` |
| `network: "stacks"` in all accepts | `invalid_network` |
| `outputSchema` in all accepts | `missing_output_schema` |

### Why outputSchema Matters

The `outputSchema` tells agents HOW to call your service:
- `input.method` - HTTP method (GET/POST)
- `input.queryParams` - Query parameters (for GET)
- `input.bodyFields` - Body structure (for POST)
- `output` - Response format

This is the contract agents use to call your service programmatically.

### Quick Registration Example

```bash
curl -X POST https://x402scan.com/api/v1/resources \
  -H "Content-Type: application/json" \
  -d '{"url": "https://your-service.com/api/endpoint"}'
```

**After registration:** Your service appears in the x402scan directory and gets re-validated every 24h.

## Quick Reference

| Function | Purpose | Returns |
|----------|---------|---------|
| `privateKeyToAccount(key, network)` | Load wallet from key | `{ address, privateKey, network }` |
| `generateKeypair(network)` | Create new wallet | `{ privateKey, publicKey, address }` |
| `wrapAxiosWithPayment(axios, account)` | Auto-pay on 402 | axios instance |
| `paymentMiddleware(config)` | Protect endpoint | Express middleware |
| `STXtoMicroSTX(amount)` | Convert STX to microSTX | bigint |
| `decodePaymentResponse(header)` | Get payment details | `{ transaction, payer, network }` |

## Payment Flow

```
1. Client → GET /api/data → Server
2. Server → 402 + payment-required header → Client
3. Client signs tx (not broadcast)
4. Client → GET + payment-signature → Server
5. Server → Facilitator settles tx → Blockchain
6. Server → 200 + data + payment-response → Client
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Generating wallet every run | Save private key to `.env` and load it |
| Using plain "mainnet" instead of CAIP-2 | Use `STACKS_NETWORKS.MAINNET` or `"stacks:1"` |
| Not funding testnet wallet | Get tokens from faucet first |
| Forgetting timeout for settlement | Set `timeout: 60000` (60 seconds minimum) |
| Mixing STX and microSTX amounts | Use `STXtoMicroSTX()` converter |
| Hardcoding private keys | Always use environment variables |

## Token Support

- **STX** (default): Native Stacks token
- **sBTC**: Bitcoin on Stacks (add `tokenType: 'sBTC'` and `tokenContract`)

## Environment Variables

```bash
# Client
PRIVATE_KEY=your-private-key-hex
NETWORK=testnet

# Server
SERVER_ADDRESS=ST1...
FACILITATOR_URL=https://facilitator.stacksx402.com
```

## When NOT to Use

- Traditional payment flows (use Stripe/PayPal)
- Subscription models (x402 is for pay-per-use)
- High-frequency micro-transactions (<$0.001 - fees make it impractical)
- When users don't have crypto wallets

## Resources

- Testnet Faucet: https://explorer.stacks.co/sandbox/faucet?chain=testnet
- Stacks Explorer: https://explorer.stacks.co/
- Free Facilitator: https://facilitator.stacksx402.com
- x402scan Registry: https://x402scan.com
