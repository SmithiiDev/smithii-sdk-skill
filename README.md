# Smithii SDK Skill

> **Let your Agent handle it.** Paste this into Claude Code, Cursor, or any agent
> that supports the [agent skills standard](https://agentskills.io):
>
> ```
> add smithii-sdk skill https://raw.githubusercontent.com/SmithiiDev/smithii-sdk-skill/main/SKILL.md
> ```

Claude Code / agent skill for [`@smithii/sdk`](https://www.npmjs.com/package/@smithii/sdk) —
the framework-agnostic TypeScript SDK powering [Smithii Tools](https://tools.smithii.io).

Once installed, your agent can build pump.fun bundlers, anti-MEV volume bots,
token creators, multisenders, and the rest of the Smithii toolset without
hallucinating method signatures or limits.

## What it covers

- **Solana** — pump.fun bundlers, PumpSwap, Launchlab, Bonk (LetsBonk), Moonit,
  anti-MEV, token creator, multisender, token manager, vesting, claim,
  market maker, Mantis cross-chain, payment program reader
- **EVM** — token creator, multisender, holder snapshot (Base, Ethereum, BSC,
  Polygon, Arbitrum, Avalanche, Blast)
- **SUI** — token creator, snapshot

## Manual install

If your agent doesn't support the install phrase format, clone the repo into
your skills directory:

```bash
git clone https://github.com/SmithiiDev/smithii-sdk-skill ~/.claude/skills/smithii-sdk
```

## Structure

```
smithii-sdk-skill/
├── SKILL.md                    # invocation triggers + agent instructions + inline examples
├── resources/
│   ├── api-reference.md        # complete API surface (every client)
│   └── invariants.md           # on-chain invariants, math, slippage, limits
└── examples/                   # runnable examples per tool
```

## Links

- **Smithii**: https://smithii.io
- **Smithii Tools**: https://tools.smithii.io
- **SDK on npm**: https://www.npmjs.com/package/@smithii/sdk
- **SDK GitHub**: https://github.com/SmithiiDev/smithii-sdk
- **Discord**: https://discord.gg/3AFfGDfmk7

## License

MIT
