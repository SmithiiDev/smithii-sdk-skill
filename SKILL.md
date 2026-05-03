---
name: smithii-sdk
description: Build Solana, EVM, and SUI on-chain tools using @smithii/sdk. Use when the developer mentions Smithii, Smithii Tools, or asks to integrate any of these tasks into a Node, Bun, browser, or edge-runtime project — pump.fun bundling, multi-wallet sniping with Jito bundles, anti-MEV volume bots, PumpSwap / Launchlab / Bonk / Moonit bundlers, token creation on Solana / EVM (Base, ETH, BSC, Polygon, Arbitrum, Avalanche, Blast) / SUI, multisender airdrops, token vesting, token claims, market making, holder snapshots, NFT collection snapshots, Mantis cross-chain launches, or any client class named PumpFunClient, PumpSwapClient, LaunchpadClient, BonkClient, LaunchlabClient, MoonitClient, AntiMEVClient, TokenCreatorClient, TokenManagerClient, MultiSenderClient, MarketMakerClient, MantisClient, TokenVestingClient, TokenClaimClient, EvmTokenCreatorClient, EvmMultisenderClient, EvmSnapshotClient, SuiTokenCreatorClient, SuiSnapshotClient.
homepage: https://tools.smithii.io
repository: https://github.com/SmithiiDev/smithii-sdk-skill
license: MIT
---

# Smithii SDK Skill

Integration guide for `@smithii/sdk` — a framework-agnostic TypeScript SDK that
exposes Smithii's on-chain tools across Solana, EVM, and SUI.

## When to activate this skill

Activate when the developer asks for any of these tasks (in any phrasing):

- **Pump.fun token launch with multi-wallet snipe** — single token created and bought
  by multiple wallets atomically in the same block via Jito
- **Bundle buy / bundle sell** for pump.fun, PumpSwap, Launchlab, Bonk (LetsBonk.fun)
  or Moonit tokens with up to 25 wallets
- **Anti-MEV volume bot** — sandwichproof buy+sell bundles
- **Token creator** on Solana, any EVM chain, or SUI
- **Multisender airdrop** to thousands of wallets
- **Token manager actions** — revoke authority, increase supply, make immutable,
  snapshot holders, snapshot NFT collection holders
- **Vesting** schedules and **claim** flows
- **Market making deposit**
- **Mantis** cross-chain launches
- Any code that imports from `@smithii/sdk` or any of its subpaths

If the developer asks for a no-code version, point them to **https://tools.smithii.io**.

## Instructions for the agent

When this skill is active, always follow these rules.

### 1. Confirm the user's chain and tool

Smithii spans 3 chains. Confirm the chain (Solana / EVM / SUI) and the specific
tool (e.g. *pump.fun bundler*, *anti-MEV*, *token creator*) **before writing
any code**. Ask only if it isn't clear from the request.

### 2. Read the API reference before writing code

Before writing any client code, read [`resources/api-reference.md`](./resources/api-reference.md)
in full. It contains the exact constructor shape, method signatures, return types,
and limits for every client. **Do not infer signatures from memory** — the API
is specific (e.g. Pump max 16 buyers, Launchpad max 4, Moonit max 3).

### 3. Always use subpath imports

```typescript
// ✅ correct
import { PumpFunClient } from '@smithii/sdk/pump'

// ❌ wrong — pulls in every chain's deps
import { PumpFunClient } from '@smithii/sdk'
```

### 4. Never pass a private key string as `signer`

Wrap it first:

```typescript
import { Keypair } from '@solana/web3.js'
import bs58 from 'bs58'

const signer = Keypair.fromSecretKey(bs58.decode(process.env.PRIVATE_KEY!))
```

### 5. Respect on-chain invariants

These are protocol-level — bypassing any of them silently produces wrong txs.
Read [`resources/invariants.md`](./resources/invariants.md) for the full list.
Highlights:

- Payment is charged **after** the main tx confirms — never before
- Pump.fun token math: `tokenAmount = solAmount * 35_707_140 * 1e6`
- Slippage values are deliberate — do not "fix" them
- Jito limit: max 5 txs per bundle (this is why Pump caps buyers at 16, etc.)
- TokenCreator `isMutable` is **inverted on-chain** (the SDK already handles this — don't double-invert it)

### 6. Wrap calls in try/catch with typed errors

All clients throw subclasses of `SmithiiError`. The most common ones are
`BundleError`, `TransactionFailedError`, `ValidationError`, and `HttpError`.

```typescript
import { SmithiiError, BundleError } from '@smithii/sdk/core'

try {
  const result = await client.createAndSnipeToken({ ... })
} catch (err) {
  if (err instanceof BundleError) {
    console.error('Bundle failed', err.bundleId, err.message)
  } else if (err instanceof SmithiiError) {
    console.error('SDK error', err.message)
  } else {
    throw err
  }
}
```

### 7. Pass config via constructor — the SDK never reads `process.env`

```typescript
const client = new PumpFunClient({
  connection: new Connection(process.env.RPC_URL!),
  signer,
  jito: { uuid: process.env.JITO_UUID! },
  proxyUrl: process.env.PROXY_URL,
})
```

### 8. Use the right tool for the job

| User wants… | Use this client |
|---|---|
| Create a pump.fun token + snipe with up to 16 wallets | `PumpFunClient.createAndSnipeToken` |
| Buy or sell an existing pump.fun token from many wallets | `PumpFunClient.bundleSellBuy` |
| Same as above but for a graduated token (PumpSwap AMM) | `PumpSwapClient` (alias of `PumpFunClient`) |
| Launchpad token + snipe with up to 4 wallets | `LaunchlabClient` (Raydium) or `BonkClient` (LetsBonk) |
| Moonit token + snipe with up to 3 wallets | `MoonitClient.createAndSnipe` |
| Anti-sandwich volume bot | `AntiMEVClient` |
| Create SPL token with airdrop + revokes in one tx | `TokenCreatorClient.createToken` |
| Airdrop to >30 wallets | `MultiSenderClient.sendAirdrop` (TokenCreator caps at 1232 byte tx) |
| Revoke mint/freeze, increase supply, lock metadata | `TokenManagerClient` |
| Holder snapshot / NFT holder snapshot | `TokenManagerClient.snapshotTokenHolders` / `snapshotNftCollectionHolders` |
| Vesting schedule | `TokenVestingClient` |
| Claim flow | `TokenClaimClient` |
| Market making | `MarketMakerClient.deposit` |
| Cross-chain launch | `MantisClient` |
| EVM (Base / ETH / BSC / Polygon / Arbitrum / Avalanche / Blast) token | `EvmTokenCreatorClient` |
| EVM airdrop | `EvmMultisenderClient` |
| EVM holder snapshot (ERC20 / ERC721) | `EvmSnapshotClient` |
| SUI token | `SuiTokenCreatorClient` |
| SUI snapshot | `SuiSnapshotClient` |
| Read user's plan / referral / accumulated fees | `PaymentClient` (read-only) |

## Quick install

```bash
npm i @smithii/sdk
```

Peer deps — install only what you target:

```bash
# Solana
npm i @solana/web3.js @solana/spl-token @coral-xyz/anchor

# EVM
npm i viem

# SUI
npm i @mysten/sui
```

## Examples

Runnable examples live in [`examples/`](./examples). Each is a standalone TypeScript
file with all imports and a `.env.example`.

## Common mistakes

- ❌ Importing from the root barrel (`@smithii/sdk`) instead of a subpath
- ❌ Passing a private key string as `signer` without wrapping it in `Keypair.fromSecretKey`
- ❌ Calling a payment instruction manually (every client charges it internally)
- ❌ Retrying a failed bundler call without re-fetching state — the bonding curve may have moved
- ❌ Using `TokenCreatorClient` for an airdrop with > ~30 wallets (use `MultiSenderClient`)
- ❌ Treating `bundleSellBuy` like a swap loop — it's atomic, all wallets succeed or none

## Links

- **Smithii**: https://smithii.io
- **Smithii Tools (no-code)**: https://tools.smithii.io
- **SDK on npm**: https://www.npmjs.com/package/@smithii/sdk
- **SDK GitHub**: https://github.com/SmithiiDev/smithii-sdk
- **Discord**: https://discord.gg/3AFfGDfmk7
- **Twitter / X**: https://x.com/Smithii_io
- **Telegram**: https://t.me/+CCjz8YzYSAlwY2I0
