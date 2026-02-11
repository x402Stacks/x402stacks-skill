# Tutorial: Build a Paid API with x402-stacks

Create a payment-protected API in minutes using the x402-stacks skill for Claude Code.

## What You'll Build

An echo API that charges 0.01 STX per request using the HTTP 402 Payment Required protocol.

## Prerequisites

- Claude Code installed
- Node.js 18+

## Installation

```bash
# Install the skill
npx skills add tony1908/x402stacks-skill --skill x402-stacks -a claude-code
```

## Create Your API

### 1. Set Up Project

```bash
mkdir echo-api && cd echo-api
npm init -y
npm i express cors dotenv x402-stacks
npm i -D typescript ts-node @types/express @types/cors @types/node
```

Create `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

### 2. Ask Claude Code

```bash
claude
```

Give this prompt:
```
using the x402 stacks skill, create a basic echo api protected with x402,
I want to get paid on stx and charge .01 stx per request
```

Claude creates:
- `src/server.ts` - Express server
- `src/routes.ts` - Echo endpoint with payment protection

### 3. Generate Server Wallet

```bash
node -e "const {generateKeypair}=require('x402-stacks');const w=generateKeypair('testnet');console.log('Address:',w.address,'\nPrivate:',w.privateKey)"
```

Create `.env`:
```bash
PORT=3000
SERVER_ADDRESS=<your-server-address>
NETWORK=testnet
FACILITATOR_URL=https://facilitator.stacksx402.com
```

### 4. Start Server

```bash
npm pkg set scripts.dev="ts-node src/server.ts"
npm run dev
```

## Test It

### 1. Create Client

Ask Claude:
```
create a client.ts that calls /api/echo with message "Hello x402!"
```

### 2. Generate Client Wallet

```bash
node -e "const {generateKeypair}=require('x402-stacks');const w=generateKeypair('testnet');console.log('Address:',w.address,'\nPrivate:',w.privateKey)"
```

### 3. Fund Client Wallet

1. Visit: https://explorer.stacks.co/sandbox/faucet?chain=testnet
2. Paste client address
3. Request testnet STX
4. Wait 1-2 minutes

Add to `.env`:
```bash
CLIENT_PRIVATE_KEY=<your-client-private-key>
```

### 4. Run Client

```bash
npm pkg set scripts.client="ts-node src/client.ts"
npm run client
```

Expected output:
```
Echo: Hello x402!
Payment confirmed!
Transaction: 0x1234...
```

## What Just Happened

1. Client requested `/api/echo`
2. Server responded **402 Payment Required**
3. Client automatically paid 0.01 STX
4. Facilitator settled on blockchain (~10 seconds)
5. Server returned the echo response

All automatic! ✨

## Register on x402scan (Optional)

Make your API discoverable:

```bash
curl -X POST https://scan.stacksx402.com/api/v1/resources \
  -H "Content-Type: application/json" \
  -d '{"url": "https://your-api.com/api/echo"}'
```

Your endpoint must return valid x402 response with `outputSchema` (see skill docs).

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Insufficient funds | Fund wallet via faucet |
| Transaction timeout | Increase timeout to 120000ms |
| Invalid network | Use `STACKS_NETWORKS.TESTNET` not `"testnet"` |

## Next Steps

- Deploy to production (use `NETWORK=mainnet`)
- Add more endpoints: Ask Claude to add new paid routes
- Dynamic pricing: Different prices based on input size
- sBTC support: Accept Bitcoin payments

## Resources

- Skill: https://github.com/tony1908/x402stacks-skill
- SDK: https://www.npmjs.com/package/x402-stacks
- Faucet: https://explorer.stacks.co/sandbox/faucet?chain=testnet
- Registry: https://scan.stacksx402.com

---

**You just built a paid API with one prompt!** 🚀
