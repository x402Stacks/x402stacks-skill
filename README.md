# x402-stacks Agent Skill

Agent skill for implementing HTTP 402 payment-required endpoints on Stacks blockchain using the x402-stacks SDK.

## What is x402-stacks?

x402-stacks enables **automatic HTTP-level payments** for APIs using STX or sBTC tokens on Stacks blockchain. This skill helps agents quickly implement micropayment systems using the x402 protocol.

## Installation

Install this skill using the [skills CLI](https://www.npmjs.com/package/skills):

```bash
# Install to Claude Code (or your preferred agent)
npx skills add tony1908/x402stacks-skill --skill x402-stacks

# Install globally
npx skills add tony1908/x402stacks-skill --skill x402-stacks -g

# List available skills first
npx skills add tony1908/x402stacks-skill --list
```

## What This Skill Provides

- **Clear wallet setup guidance**: Load existing wallet or generate new one
- **Quick start examples**: Client and server implementations
- **Common mistakes**: Avoid pitfalls with CAIP-2 format, timeouts, etc.
- **Quick reference table**: All key functions at a glance
- **Free facilitator**: Pre-configured with https://facilitator.stacksx402.com

## When to Use

Use this skill when:
- Implementing HTTP 402 payment-required endpoints on Stacks blockchain
- Setting up micropayment clients with STX or sBTC tokens
- Handling x402 protocol wallet setup

## Features

- ✅ Wallet handling (load existing vs. generate new)
- ✅ Installation command (`npm i x402-stacks`)
- ✅ Free facilitator URL
- ✅ Quick reference for common functions
- ✅ Payment flow diagram
- ✅ Common mistakes section

## Supported Agents

This skill works with any agent that supports the [Agent Skills specification](https://github.com/skills/spec), including:

- Claude Code
- OpenCode
- Cursor
- Cline
- Codex
- Continue
- Windsurf
- And 30+ more

## Related Resources

- [x402-stacks SDK](https://www.npmjs.com/package/x402-stacks)
- [Stacks Blockchain](https://www.stacks.co/)
- [Free Facilitator](https://facilitator.stacksx402.com)
- [Testnet Faucet](https://explorer.stacks.co/sandbox/faucet?chain=testnet)

## License

MIT
