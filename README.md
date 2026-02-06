# Wirex – Vottun Stellar SDK

**Modular SDK for Payment-Enabled Applications on Stellar**

---

## What is this?

The **Wirex–Vottun Stellar SDK** is a modular, non-custodial SDK that simplifies building payment-enabled applications on Stellar.

It abstracts:

- Stellar accounts, assets, and transactions
- Soroban smart contract interactions
- Real-time network events
- Integration with regulated payment infrastructure (via reference implementations)

So developers can focus on **product logic**, not protocol plumbing.

Built as reusable ecosystem infrastructure under the **SCF Integration Track**.

---

## Why it exists

Building real-world payments on Stellar today requires stitching together:

- Horizon APIs
- Soroban contracts
- Transaction lifecycle logic
- Streaming / event handling
- External payment providers

This SDK provides a **single, consistent abstraction layer** for all of the above.

---

## Core Principles

- **Non-custodial by design** — keys never leave the client
- **Stellar-native** — Horizon, Soroban, assets, streaming
- **Modular & extensible** — use one module or all
- **Production-oriented** — testnet → mainnet parity
- **Provider-agnostic** — Wirex included as reference, not hard dependency

---

## Architecture (High Level)

```
Client App (Wallet / Fintech / Backend)
        ↓
Wirex–Vottun Stellar SDK
        ↓
Stellar Network (Horizon, Soroban, Assets)
        ↓
Optional Reference Integrations (e.g. Wirex Payments)
```

Full technical details: [ARCHITECTURE.md](ARCHITECTURE.md)

---

## SDK Modules

### Wallet

- Account creation & querying
- Balance retrieval (XLM + issued assets)
- Trustlines & account utilities
- Client-side key handling only

### Transactions

- Transaction construction & signing
- Fee estimation
- Submission & lifecycle tracking

### Smart Contracts (Soroban)

- Contract invocation helpers
- Parameter encoding / decoding
- Consistent error handling

### API Client

- Unified access to Stellar + external services
- Normalized responses & errors
- Pluggable provider design

### WebSocket / Streaming

- Account activity updates
- Transaction events
- Network streams

### Configuration

- Testnet / mainnet switching
- Environment & endpoint management

---

## Reference Integration: Wirex

Includes a reference integration showing how to connect regulated payment infrastructure to Stellar apps.

- Users retain **self-custody of assets**
- Wirex handles regulated components (payments, card issuance, compliance)
- Stellar is used as the settlement & asset layer

This integration is **optional** and intended as a template for other providers.

---

## Supported Stellar Features

- Horizon APIs
- Soroban smart contracts
- XLM and issued assets (e.g. USDC, EURC)
- Streaming / real-time events
- Testnet and mainnet

---

## Security Model

- No private key custody
- All signing occurs client-side
- Clear separation between on-chain logic (Stellar) and off-chain regulated services

Designed for non-custodial wallets and secure backend services.

---

## Status

**Active development**

- Target: Mainnet-ready release
- Built as part of an SCF Build project
- APIs and docs may evolve until v1.0

---

## Repository Structure

```
/src
  /wallet
  /transactions
  /contracts
  /api-client
  /websocket
  /config
/docs
  ARCHITECTURE.md
  USAGE.md
/examples
```

---

## Getting Started

Coming soon:

- Installation instructions
- Testnet walkthroughs
- Example integrations

---

## Contributing

This project is intended to be open and extensible.

Contribution guidelines will be added once the initial SDK surface is finalized.

---

## Teams

**Wirex** — Global payments infrastructure provider operating at scale across 130+ countries.

**Vottun** — Blockchain infrastructure and SDK provider with production-grade Stellar experience since 2021.

---

## Links

- Project website: [https://www.wirexapp.com/](https://www.wirexapp.com/)
- Technical architecture: [ARCHITECTURE.md](./docs/ARCHITECTURE.md)
