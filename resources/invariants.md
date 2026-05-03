# On-chain invariants

Protocol-level constants and behaviors. **Bypassing any of these silently
produces incorrect transactions.**

## Payment ordering

Payment is charged **after** the main transaction confirms on-chain.
Every client enforces this internally — never call payment manually.

## Pump.fun token math

```
tokenAmount = solAmount * 35_707_140 * 1e6   // 6 decimals
maxSolCost  = solAmount * 1.1 * 1e9          // 10% slippage cap
```

After `PUMP_UPGRADE_TIMESTAMP_MS`, BUY/SELL ixs must include an extra fee
recipient appended at the end. The SDK handles this automatically.

## Slippage values (deliberate — do not "fix")

| Action | Slippage |
|--------|----------|
| Pump.fun BUY | 10% (`maxSolCost = sol * 1.1`) |
| Pump.fun SELL | 0% min-out |
| Moonit first BUY | 25% (2500 bps) |
| Moonit subsequent BUY | 10% (1000 bps) |
| Raydium launchpad creation | 99.99% (`new BN(9999)`) |

## Jito bundle limits

- Max **5 transactions per bundle** (Jito protocol limit)
- Tip: 500_000 lamports placed on the **first tx** of every Pump bundle
- Compute unit price: 200_000 microLamports
- Bundle endpoint: `https://mainnet.block-engine.jito.wtf/api/v1/bundles`
  (used via the Smithii proxy)

## Wallet limits per client

| Client / method | Max wallets |
|---|---|
| `PumpFunClient.createAndSnipeToken` | 16 |
| `PumpFunClient.bundleSellBuy` | 25 |
| `LaunchpadClient.createAndSnipe` | 4 (create + 4 = 5 txs) |
| `MoonitClient.createAndSnipe` | 3 (mint + tip + 3 = 5 txs) |
| `MoonitClient.bundleSwap` | 4 (4 swaps + tip = 5 txs) |
| `MultiSenderClient.sendAirdrop` | `AIRDROPPER_LIMIT` (multi-tx allowed) |

## Token program selection

- Default: `TOKEN_PROGRAM_ID` (legacy SPL)
- `isCashbackCoin: true` on `PumpFunClient` → uses `TOKEN_2022_PROGRAM_ID`
- `TokenCreatorClient.tokenProgram` → opt into Token-2022 explicitly

## TokenCreator mutability

`isMutable` is **inverted on-chain** — the SDK sends `!args.isMutable`.
- `isMutable: true` (your input) → metadata is **locked** on-chain
- `isMutable: false` (your input) → metadata is **mutable** on-chain

This is intentional. The `RevokeMutability` fee is charged when the user
passes `isMutable: true` on create.

## Transaction size

Solana max: **1232 bytes**.

The `TokenCreatorClient` validates this — large airdrops will fail. Use
`MultiSenderClient` instead, which batches into multiple txs.

## Pump.fun pre-generated tokens (vanity)

Pre-generated mints (e.g. ending in `pump`) are validated via
`isTokenPregenerated`. They incur **double fee** in `PumpFunClient` and
`LaunchpadClient`.

## Anti-MEV polling

- Single-wallet mode: backend builds deposit tx, SDK polls confirmation
  (60 attempts × 2s = 2 min max)
- Multi-wallet mode: pure backend orchestration with private keys —
  no client-side tx is built or signed

## SUI deploy fee

Default SUI deploy fee: **7.5 SUI**. Paid before deploy via private
`paySmithii()`. The `skipPayment: true` flag is internal/dev only —
never expose to end users.
