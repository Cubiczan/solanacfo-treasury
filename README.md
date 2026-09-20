# SolanaCFO Treasury

**On-Chain Solana Treasury Management** with Multi-Agent Council Deliberation, built for [Superteam](https://superteam.ca) and the Solana ecosystem.

## Architecture

```
  DAO Members ──▶ Cognito ──▶ API Gateway ──▶ Treasury Management
                                         ──▶ Portfolio Analyzer
                                         ──▶ Risk Monitor
                                         ──▶ Yield Strategist
                                         ──▶ Governance Analyst
                                         ──▶ Treasury CFO
                                         ──▶ Council Deliberation (5 agents)
                                         ──▶ On-Chain Monitor (event-driven)
                                         ──▶ Governance Executor (on-chain)
                                         ──▶ Voice Briefings (Deepgram)

  5-Agent Council: Portfolio | Risk | Yield | Governance | CFO
            ▼
    Council Synthesizer ──▶ Final Recommendation

  On-Chain (Solana):
  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │ SOL Balances  │  │ SPL Tokens   │  │ Governance    │
  │ via RPC       │  │ via Anchor   │  │ Proposals     │
  └──────────────┘  └──────────────┘  └──────────────┘
```

## Features

- **5-Agent Council**: Portfolio Analyzer, Risk Monitor, Yield Strategist, Governance Analyst, Treasury CFO (`agents/portfolio_analyzer.py`, `agents/risk_monitor_agent.py`, `agents/yield_strategist.py`, `agents/governance_analyst.py`, `agents/treasury_cfo.py`)
- **Solana Integration**: RPC calls for SOL/SPL balances, token accounts, transaction history (`lambdas/common/solana_client.py`)
- **Anchor Program**: On-chain treasury governance with token-weighted voting (Rust) (`programs/src/lib.rs`)
- **DeFi Analytics**: TVL, APY, impermanent loss, liquidation risk calculations (`lambdas/common/defi_analytics.py`)
- **Governance Executor**: Create/execute proposals on-chain with quorum thresholds (`lambdas/governance_executor/app.py`)
- **Voice Briefings**: Deepgram TTS treasury status updates (`lambdas/common/deepgram_client.py`)
- **Event-Driven Monitoring**: SQS-triggered alerts for large transfers and governance events (`lambdas/risk_monitor/app.py`)
- **Observability**: PRISMtrace on BlockConvey for Bedrock calls across the council (`lambdas/common/prism_observability.py`)

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | AWS SAM, Python 3.12, Lambda |
| Blockchain | Solana (RPC), Anchor (Rust) |
| LLM | Amazon Bedrock (Claude 3.5 Sonnet) |
| Voice | Deepgram Nova-2 STT + Aura TTS |
| Auth | Amazon Cognito |
| Database | DynamoDB (6 tables) |
| On-Chain | Anchor framework, SPL Token operations |

## Deployment

```bash
# Backend
sam build && sam deploy --guided

# Anchor program (requires Solana CLI + Anchor)
cd programs && anchor build && anchor deploy --provider.cluster localnet
```

## License

MIT

## Propagation notes (wave B)

- **Row 7 (Sentinel-style circuit breaker) — reversed.** The reversal
  condition fires: Sentinel needs a dense, rolling action stream with
  correlated-failure structure to detect, while the treasury's autonomous
  surface is episodic — scheduled lambdas
  (`lambdas/council_deliberation/app.py` and the analyst lambdas) plus a
  human-in-the-loop council vote per action. The repository also carries no
  breaker, pause, or halt machinery to extend (`grep` across `lambdas/` and
  `agents/` finds none). Reopens if the desk runs a high-frequency
  autonomous action loop rather than deliberated, scheduled cycles.
