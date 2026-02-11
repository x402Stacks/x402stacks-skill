# Tutorial: Building a Paid Echo API with x402-stacks

Learn how to create a payment-protected API in minutes using the x402-stacks skill for Claude Code.

## What You'll Build

A simple echo API that:
- Returns whatever message you send it
- Requires 0.01 STX payment per request
- Uses the HTTP 402 Payment Required protocol
- Automatically handles payments via the Stacks blockchain

## Prerequisites

- Claude Code installed
- Node.js 18+ installed
- A Stacks wallet with testnet STX (we'll help you get this)

## Step 1: Install the x402-stacks Skill

Open your terminal and run:

```bash
# Install the skill for Claude Code
npx skills add x402Stacks/x402stacks-skill --skill x402-stacks -a claude-code

# Verify installation
npx skills list
```

You should see `x402-stacks` in the list of installed skills.

## Step 2: Start a New Project

```bash
# Create project directory
mkdir echo-api-x402
cd echo-api-x402

# Initialize package.json
npm init -y

# Install dependencies
npm install express cors dotenv x402-stacks
npm install -D typescript ts-node @types/express @types/cors @types/node

# Create tsconfig.json
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "es2016",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
EOF

# Create .env file
cat > .env << 'EOF'
PORT=3000
SERVER_ADDRESS=your_stacks_address_here
NETWORK=testnet
FACILITATOR_URL=https://facilitator.stacksx402.com
EOF

# Create src directory
mkdir src
```

## Step 3: Use Claude Code with the Skill

Open Claude Code in your project directory:

```bash
claude
```

Then, give Claude this exact prompt:

```
using the x402 stacks skill, create a basic echo api protected with x402,
I want to get paid on stx and charge .01 stx per request
```

### What Claude Will Create

Claude Code will automatically:

1. **Create `src/server.ts`** - Express server setup
2. **Create `src/routes.ts`** - Echo endpoint with payment middleware
3. **Set up payment protection** - Using `paymentMiddleware` from x402-stacks
4. **Configure pricing** - 0.01 STX per request using `STXtoMicroSTX(0.01)`

### Expected Files

After Claude finishes, you'll have:

```
echo-api-x402/
├── src/
│   ├── server.ts          # Express server
│   └── routes.ts          # Echo route with payment
├── .env                   # Environment variables
├── package.json
└── tsconfig.json
```

## Step 4: Get Your Server Wallet

You need a Stacks address to receive payments.

### Generate a Server Wallet

Run this in your project:

```bash
node -e "
const { generateKeypair } = require('x402-stacks');
const wallet = generateKeypair('testnet');
console.log('Server Wallet Generated!');
console.log('Address:', wallet.address);
console.log('Private Key:', wallet.privateKey);
console.log('\nAdd to .env:');
console.log('SERVER_ADDRESS=' + wallet.address);
"
```

### Update .env

Copy the `SERVER_ADDRESS` from above and update your `.env` file:

```bash
PORT=3000
SERVER_ADDRESS=ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM  # Your address here
NETWORK=testnet
FACILITATOR_URL=https://facilitator.stacksx402.com
```

## Step 5: Start the Server

```bash
# Add start script to package.json
npm pkg set scripts.dev="ts-node src/server.ts"

# Start the server
npm run dev
```

You should see:
```
Server running on http://localhost:3000
```

## Step 6: Test with a Client

### Create a Test Client

Ask Claude Code:

```
create a client.ts file that uses the x402-stacks sdk to call my echo endpoint
and pay automatically. Use a test message "Hello from x402!"
```

Claude will create `src/client.ts` with:
- Wallet setup (load existing or generate new)
- Axios wrapped with payment interceptor
- Automatic payment on 402 response

### Generate Client Wallet

```bash
node -e "
const { generateKeypair } = require('x402-stacks');
const wallet = generateKeypair('testnet');
console.log('Client Wallet Generated!');
console.log('Address:', wallet.address);
console.log('Private Key:', wallet.privateKey);
console.log('\nFund this address at:');
console.log('https://explorer.stacks.co/sandbox/faucet?chain=testnet');
console.log('\nAdd to .env:');
console.log('CLIENT_PRIVATE_KEY=' + wallet.privateKey);
"
```

### Fund Your Client Wallet

1. Copy the client address
2. Visit: https://explorer.stacks.co/sandbox/faucet?chain=testnet
3. Paste your address and request testnet STX
4. Wait ~1-2 minutes for tokens to arrive

### Update .env

Add the client private key:

```bash
PORT=3000
SERVER_ADDRESS=ST1PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM
NETWORK=testnet
FACILITATOR_URL=https://facilitator.stacksx402.com
CLIENT_PRIVATE_KEY=your_client_private_key_here
```

### Run the Client

In a new terminal:

```bash
npm pkg set scripts.client="ts-node src/client.ts"
npm run client
```

Expected output:
```
Using wallet: ST2PQHQKV0RJXZFY1DGX8MNSNYVE3VGZJSRTPGZGM
Requesting echo endpoint (payment will be automatic)...
=== Response ===
Echo: Hello from x402!
Payment confirmed!
Transaction: 0x1234...
```

## Step 7: Verify the Payment

Check the transaction on Stacks Explorer:

```
https://explorer.stacks.co/txid/0x1234...?chain=testnet
```

You should see:
- **From:** Your client address
- **To:** Your server address
- **Amount:** 0.01 STX
- **Status:** Success ✅

## Step 8: Register on x402scan (Optional)

Make your API discoverable to other agents:

```bash
curl -X POST https://scan.stacksx402.com/api/v1/resources \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-deployed-api.com/api/echo"
  }'
```

Your endpoint must return a valid x402 response with `outputSchema` (see skill for details).

## What Just Happened?

1. **Server Started** - Your Echo API is running with payment protection
2. **Client Request** - Client made a GET request to `/api/echo`
3. **402 Response** - Server responded with "Payment Required" + payment details
4. **Automatic Payment** - Client signed and sent a 0.01 STX transaction
5. **Settlement** - Facilitator broadcast the transaction to Stacks blockchain
6. **Confirmation** - After ~10 seconds, transaction confirmed
7. **Access Granted** - Server returned the echo response

All of this happened automatically thanks to the x402-stacks SDK!

## Example Code

### Server (src/routes.ts)

```typescript
import { Router } from "express";
import { paymentMiddleware, STXtoMicroSTX, STACKS_NETWORKS } from "x402-stacks";

const router = Router();

router.get(
  "/api/echo",
  paymentMiddleware({
    amount: STXtoMicroSTX(0.01),
    payTo: process.env.SERVER_ADDRESS!,
    network: STACKS_NETWORKS.TESTNET,
    asset: "STX",
    facilitatorUrl: "https://facilitator.stacksx402.com",
    description: "Echo service - returns your message",
  }),
  (req, res) => {
    const message = req.query.message || "No message provided";
    res.json({
      echo: message,
      paid: true,
      amount: "0.01 STX"
    });
  }
);

export default router;
```

### Client (src/client.ts)

```typescript
import axios from "axios";
import { wrapAxiosWithPayment, privateKeyToAccount } from "x402-stacks";
import dotenv from "dotenv";

dotenv.config();

const account = privateKeyToAccount(process.env.CLIENT_PRIVATE_KEY!, "testnet");

const api = wrapAxiosWithPayment(
  axios.create({
    baseURL: "http://localhost:3000",
    timeout: 60000
  }),
  account
);

async function main() {
  const response = await api.get("/api/echo", {
    params: { message: "Hello from x402!" }
  });

  console.log("Echo:", response.data.echo);
  console.log("Paid:", response.data.paid);
}

main().catch(console.error);
```

## Next Steps

### Deploy to Production

1. **Change to mainnet:**
   ```bash
   NETWORK=mainnet
   FACILITATOR_URL=https://facilitator.stacksx402.com
   ```

2. **Get mainnet STX:**
   - Buy STX on an exchange
   - Send to your server and client addresses

3. **Deploy your server:**
   - Vercel, Railway, Render, etc.
   - Ensure HTTPS enabled

4. **Update facilitator:**
   - Or run your own facilitator

### Add More Endpoints

Ask Claude Code:

```
add another endpoint /api/reverse that reverses a string,
charge 0.005 STX per request
```

Claude will add the new endpoint with the correct pricing.

### Dynamic Pricing

Ask Claude Code:

```
make the price dynamic based on message length:
0.01 STX for <100 chars, 0.02 STX for 100-500 chars,
0.05 STX for >500 chars
```

### sBTC Support

Ask Claude Code:

```
add sBTC payment option to the echo endpoint,
charge 0.00001 sBTC (1000 sats) per request
```

## Troubleshooting

### "Insufficient funds"
- Fund your client wallet via the faucet
- Check balance: https://explorer.stacks.co/address/YOUR_ADDRESS?chain=testnet

### "Transaction timeout"
- Testnet can be slow (~10 min blocks)
- Increase timeout: `timeout: 120000` (2 minutes)

### "Invalid network format"
- Use `STACKS_NETWORKS.TESTNET` or `"stacks:2147483648"`
- NOT plain `"testnet"`

### "Payment verification failed"
- Check facilitator URL is correct
- Ensure your server address is correct in .env

## Resources

- **Skill Repository:** https://github.com/x402Stacks/x402stacks-skill
- **x402-stacks SDK:** https://www.npmjs.com/package/x402-stacks
- **Free Facilitator:** https://facilitator.stacksx402.com
- **Testnet Faucet:** https://explorer.stacks.co/sandbox/faucet?chain=testnet
- **x402scan Registry:** https://scan.stacksx402.com

## Summary

You just built a payment-protected API in minutes using:
- ✅ One skill installation
- ✅ One prompt to Claude Code
- ✅ Automatic payment handling
- ✅ Blockchain settlement
- ✅ Zero payment processing code written

The x402-stacks skill handled all the complexity for you! 🎉

## What's Next?

- Build more complex paid APIs
- Integrate with AI services (OpenAI, Anthropic, etc.)
- Create marketplaces for AI agent services
- Monetize your APIs with micropayments

**Happy building! 🚀**
