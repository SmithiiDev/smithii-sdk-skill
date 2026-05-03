# Smithii SDK Skill

Claude Code / agent skill for [`@smithii/sdk`](https://www.npmjs.com/package/@smithii/sdk) —
the framework-agnostic TypeScript SDK powering [Smithii Tools](https://tools.smithii.io).

Install in any agent that supports the [agent skills standard](https://agentskills.io)
to give it deep, accurate knowledge of the SDK API surface.

## What it covers

- **Solana** — pump.fun bundlers, PumpSwap, Launchlab, Bonk, Moonit, anti-MEV,
  token creator, multisender, token manager, vesting, claim, market maker,
  Mantis cross-chain, payment program reader
- **EVM** — token creator, multisender, holder snapshot (Base, ETH, BSC,
  Polygon, Arbitrum, Avalanche, Blast)
- **SUI** — token creator, snapshot

## Install in Claude Code

```bash
# from this repo
git clone https://github.com/SmithiiDev/smithii-sdk-skill ~/.claude/skills/smithii-sdk
```

Or follow the [Solana Skills CLI](https://www.solanaskills.com) instructions.

## Structure

```
smithii-sdk-skill/
├── SKILL.md                    # invocation triggers + agent instructions
├── resources/
│   ├── api-reference.md        # complete API surface (mirror of CLAUDE.md)
│   └── invariants.md           # on-chain invariants, math, slippage, limits
├── examples/                   # runnable examples per tool
├── templates/                  # boilerplate setups
└── docs/                       # troubleshooting and FAQs
```

## Links

- **Smithii**: https://smithii.io
- **Smithii Tools**: https://tools.smithii.io
- **SDK on npm**: https://www.npmjs.com/package/@smithii/sdk
- **SDK GitHub**: https://github.com/SmithiiDev/smithii-sdk
- **Discord**: https://discord.gg/3AFfGDfmk7

## License

MIT
