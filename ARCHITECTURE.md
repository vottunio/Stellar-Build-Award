# Technical Architecture Document

## Wirex – Vottun Stellar SDK

*A Modular, Reusable Integration Framework for Stellar Payments*

*SCF Integration Track – Build Program*

---

## 1. Overview

This document describes the technical architecture of the Wirex–Vottun Stellar SDK, a fully modular, developer-facing software development kit designed as reusable infrastructure for the Stellar ecosystem.

The SDK abstracts core Stellar network functionality and regulated payment integrations into a coherent, extensible TypeScript framework. It enables developers to build real-world, payment-enabled applications on Stellar without managing low-level blockchain complexity.

The SDK is built under the SCF Integration Track, positioning it as a building block that other SCF applicants and ecosystem developers can directly reuse or extend. Wirex payment infrastructure is included as a reference integration; the SDK itself is provider-agnostic, open, and extensible.

---

## 2. Technology Stack

The SDK is built entirely in TypeScript to ensure type safety, broad ecosystem compatibility, and alignment with the Stellar developer community.

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Language | TypeScript 5.x | Type-safe SDK development |
| Stellar SDK | @stellar/stellar-sdk (latest) | Core Stellar operations: accounts, transactions, assets |
| Soroban Client | @stellar/stellar-sdk (Soroban module) | Smart contract invocation and RPC interaction |
| HTTP Client | Axios / native fetch | REST API communication with Horizon and external services |
| WebSocket | Native WebSocket + Horizon SSE | Real-time event streaming from Stellar network |
| Build System | tsup / Rollup | ESM and CJS output for broad compatibility |
| Testing | Vitest + Stellar Testnet | Unit, integration, and end-to-end testing |
| Package | npm registry | Distributed as @wirex-vottun/stellar-sdk |

**Primary dependency:** The SDK builds on top of `@stellar/stellar-sdk`, the official JavaScript/TypeScript library maintained by the Stellar Development Foundation. This library provides direct access to the Horizon REST API, Soroban RPC, transaction primitives, keypair management, and asset operations. The SDK wraps and extends this library with higher-level abstractions, error normalization, and integration patterns.

---

## 3. High-Level Architecture

The architecture is organized into four layers. Each layer has a well-defined responsibility and communicates only with adjacent layers.

### 3.1 Architecture Layers

```
+----------------------------------------------------------------------+
|  LAYER 1: CLIENT APPLICATIONS                                       |
|  Non-custodial wallets | Fintech apps | Enterprise payment systems  |
+----------------------------------------------------------------------+
                              |
                              v
+----------------------------------------------------------------------+
|  LAYER 2: WIREX-VOTTUN STELLAR SDK                                  |
|                                                                      |
|  +------------+  +-------------+  +------------+  +---------------+  |
|  | Wallet     |  | Transaction |  | Smart      |  | Configuration |  |
|  | Management |  | Lifecycle   |  | Contract   |  | Module        |  |
|  +------------+  +-------------+  +------------+  +---------------+  |
|  +------------+  +-------------+                                     |
|  | API Client |  | WebSocket & |                                     |
|  | Module     |  | Streaming   |                                     |
|  +------------+  +-------------+                                     |
+----------------------------------------------------------------------+
                    |                        |
                    v                        v
+-------------------------------+  +-----------------------------------+
|  LAYER 3: STELLAR NETWORK     |  |  LAYER 4: EXTERNAL SERVICES      |
|                               |  |                                   |
|  Horizon REST API             |  |  Wirex Payment APIs (reference)   |
|  Soroban RPC                  |  |  Additional providers (pluggable) |
|  Stellar Assets (XLM/USDC/   |  |                                   |
|  EURC)                        |  |                                   |
|  Testnet + Mainnet            |  |                                   |
+-------------------------------+  +-----------------------------------+
```

### 3.2 Data Flow

The following describes the end-to-end data flow for a typical payment transaction:

```
Client App
    |
    |  1. Initialize SDK with network config (testnet/mainnet)
    |  2. Create or load Stellar account via Wallet Module
    v
Wallet Module --> @stellar/stellar-sdk --> Horizon API
    |
    |  3. Build transaction (payment, trustline, contract call)
    v
Transaction Module --> stellar-sdk TransactionBuilder --> Horizon /transactions
    |
    |  4. Client-side signing (keypair never leaves client)
    |  5. Submit signed XDR to Horizon
    |  6. Monitor transaction lifecycle (pending -> confirmed/failed)
    v
WebSocket Module --> Horizon SSE streams --> Real-time status updates
    |
    |  7. Optional: Trigger off-chain settlement via API Client
    v
API Client Module --> Wirex APIs (or other provider) --> Settlement confirmation
```

---

## 4. Stellar Building Blocks Used

The SDK integrates directly with the following core Stellar components. Each integration uses official Stellar interfaces and follows production best practices.

### 4.1 Horizon REST API

Horizon is the primary interface for interacting with the Stellar network. The SDK uses the following Horizon endpoints:

| Endpoint | SDK Usage | Module |
|----------|-----------|--------|
| GET /accounts/{id} | Account queries, balance retrieval, sequence numbers | Wallet Management |
| POST /transactions | Transaction submission (signed XDR) | Transaction Lifecycle |
| GET /transactions/{hash} | Transaction status and confirmation tracking | Transaction Lifecycle |
| GET /accounts/{id}/operations | Operation history and lifecycle monitoring | Transaction Lifecycle |
| GET /accounts/{id}/payments | Payment history for account reconciliation | API Client |
| GET /fee_stats | Dynamic fee estimation before transaction construction | Transaction Lifecycle |
| GET /assets | Asset discovery and metadata retrieval | Wallet Management |
| SSE /accounts/{id} | Real-time account activity streaming | WebSocket Module |
| SSE /transactions | Real-time transaction event streaming | WebSocket Module |

### 4.2 Soroban RPC

Soroban is Stellar's smart contract platform. The SDK provides high-level abstractions over the Soroban RPC interface:

| RPC Method | SDK Usage | Module |
|------------|-----------|--------|
| simulateTransaction | Estimate resource costs and validate contract calls before submission | Smart Contract |
| sendTransaction | Submit Soroban transactions to the network | Smart Contract |
| getTransaction | Poll transaction status until confirmation or failure | Smart Contract |
| getContractData | Read contract storage values (ledger entries) | Smart Contract |
| getLedgerEntries | Query specific ledger keys for contract state inspection | Smart Contract |

### 4.3 Stellar Assets

The SDK natively supports the following Stellar assets for payment and settlement operations:

| Asset | Type | SDK Support |
|-------|------|-------------|
| XLM | Native asset | Balance queries, payments, fee reserves, account funding |
| USDC | Issued asset (Centre/Circle) | Trustline management, stablecoin payments, settlement |
| EURC | Issued asset (Centre/Circle) | Trustline management, EUR-denominated payments, settlement |
| Custom assets | Any Stellar issued asset | Generic trustline and payment support via asset code + issuer |

### 4.4 Stellar Streaming (Server-Sent Events)

Horizon provides Server-Sent Events (SSE) endpoints for real-time data. The SDK wraps these into a managed subscription system with automatic reconnection and typed event handlers:

- **Account streams:** monitor balance changes, trustline updates, and incoming payments
- **Transaction streams:** track transaction confirmations across the network
- **Ledger streams:** monitor network activity for operational dashboards

### 4.5 Network Environments

The SDK supports both Stellar environments through the Configuration Module:

| Environment | Horizon URL | Soroban RPC URL | Network Passphrase |
|-------------|-------------|-----------------|-------------------|
| Testnet | https://horizon-testnet.stellar.org | https://soroban-testnet.stellar.org | Test SDF Network ; September 2015 |
| Mainnet | https://horizon.stellar.org | https://soroban-rpc.mainnet.stellar.org (or provider) | Public Global Stellar Network ; September 2015 |

---

## 5. SDK Module Architecture

The SDK is organized into six independent but interoperable modules. Each module is designed to be usable standalone or as part of the full SDK. All modules are written in TypeScript and expose strongly typed interfaces.

### 5.1 Wallet Management Module

**Purpose:** Provide a secure, non-custodial abstraction for Stellar account operations, key handling, and asset management.

**Stellar Components Used:**

- `@stellar/stellar-sdk`: Keypair class for key generation and derivation
- Horizon `GET /accounts/{id}` for account queries and balance retrieval
- Horizon asset endpoints for trustline and issued asset management

**Key Features:**

| Feature | Description | Stellar Integration |
|---------|-------------|-------------------|
| Account creation | Generate new Stellar keypairs and fund accounts via friendbot (testnet) or explicit funding (mainnet) | Keypair.random() + Horizon account creation |
| Balance retrieval | Query XLM and issued asset balances for any account | GET /accounts/{id} -> balances array |
| Trustline management | Establish, modify, or remove trustlines for USDC, EURC, or custom assets | ChangeTrust operation via TransactionBuilder |
| External signer support | Interface for external wallet providers to sign transactions | Transaction XDR passed to external signer, signed XDR returned |
| Key management | Secure key generation and storage abstractions (keys never leave the client) | Keypair class, no server-side key storage |

**Conceptual Interface:**

```typescript
interface StellarWallet {
  create(): Promise<{ publicKey: string; secretKey: string }>;
  getBalances(publicKey: string): Promise<AssetBalance[]>;
  establishTrustline(asset: StellarAsset, signer: Signer): Promise<TxResult>;
  getAccountInfo(publicKey: string): Promise<AccountInfo>;
}

// Usage example:
const sdk = new StellarSDK({ network: 'mainnet' });
const wallet = sdk.wallet.create();
const balances = await sdk.wallet.getBalances(wallet.publicKey);
// -> [{ asset: 'XLM', balance: '100.00' }, { asset: 'USDC', balance: '50.00' }]
```

### 5.2 Transaction Lifecycle Module

**Purpose:** Abstract the full Stellar transaction lifecycle into a consistent, developer-friendly interface with built-in resilience.

**Stellar Components Used:**

- `@stellar/stellar-sdk`: TransactionBuilder, Operation, Keypair
- Horizon `POST /transactions` for submission
- Horizon `GET /fee_stats` for dynamic fee estimation
- Horizon `GET /transactions/{hash}` for status tracking

**Key Features:**

| Feature | Description | Stellar Integration |
|---------|-------------|-------------------|
| Transaction builder | Fluent API for constructing Stellar transactions with operations | TransactionBuilder with base fee, sequence, and timeout |
| Client-side signing | Sign transaction XDR locally using Keypair or external signer | transaction.sign(keypair) - keys never transmitted |
| Fee estimation | Query network fee statistics and set appropriate base fees | GET /fee_stats -> fee_charged percentiles |
| Submission | Submit signed transaction envelope to Horizon | POST /transactions with XDR payload |
| Lifecycle tracking | Monitor transaction from submission through confirmation or failure | GET /transactions/{hash} + SSE polling |
| Retry logic | Exponential backoff for transient network errors | Automatic resubmission with updated sequence numbers |

**Conceptual Interface:**

```typescript
const tx = sdk.transaction
  .payment({
    destination: 'GDEST...',
    asset: sdk.assets.USDC,
    amount: '100.00'
  })
  .setFee(await sdk.transaction.estimateFee())
  .setTimeout(30)
  .build();

const signed = await tx.sign(keypair);
const result = await sdk.transaction.submit(signed);
// result.status: 'pending' | 'confirmed' | 'failed'
// result.hash: '8a3b...'
// result.ledger: 12345678
```

### 5.3 Smart Contract Interaction Module (Soroban)

**Purpose:** Simplify developer interaction with Soroban smart contracts by providing high-level invocation helpers, parameter encoding, and result decoding.

**Stellar Components Used:**

- Soroban RPC: simulateTransaction, sendTransaction, getTransaction, getContractData
- `@stellar/stellar-sdk`: Contract class, nativeToScVal, scValToNative
- XDR encoding/decoding for Soroban parameters and return values

**Key Features:**

| Feature | Description | Stellar Integration |
|---------|-------------|-------------------|
| Contract invocation | Call any deployed Soroban contract method with typed parameters | Contract.call() + simulateTransaction + sendTransaction |
| Parameter encoding | Automatic conversion from JS/TS types to Soroban ScVal types | nativeToScVal() for Address, i128, Bytes, Map, Vec |
| Result decoding | Parse Soroban return values back to native TypeScript types | scValToNative() on transaction result meta |
| Simulation | Dry-run contract calls to estimate resources and validate logic | simulateTransaction RPC method |
| Error normalization | Consistent error types for contract panics, resource limits, and network failures | Mapped from Soroban diagnostic events and error codes |
| Contract data queries | Read contract storage without submitting a transaction | getContractData / getLedgerEntries RPC methods |

**Conceptual Interface:**

```typescript
const contract = sdk.contracts.load('CCONTRACT_ID...');

// Simulate before submitting
const sim = await contract.simulate('transfer', {
  from: Address.fromString('GSENDER...'),
  to: Address.fromString('GDEST...'),
  amount: BigInt(1000000)
});
// sim.cost: { cpuInsns: 1234, memBytes: 5678 }

// Execute the contract call
const result = await contract.invoke('transfer', {
  from: Address.fromString('GSENDER...'),
  to: Address.fromString('GDEST...'),
  amount: BigInt(1000000)
}, keypair);
// result.returnValue: decoded native type
```

### 5.4 API Client Module

**Purpose:** Provide a unified, provider-agnostic API access layer for both Stellar network services and external off-chain services.

**Stellar Components Used:**

- Horizon REST API: all account, transaction, and asset endpoints
- Soroban RPC: JSON-RPC interface for smart contract operations

**Key Features:**

| Feature | Description |
|---------|-------------|
| Unified HTTP client | Standardized request/response models for Horizon, Soroban RPC, and external APIs |
| Authentication management | Token handling, session management, and secure header attachment for off-chain services |
| Error normalization | Consistent error types across Stellar network errors and external API failures |
| Retry and resilience | Configurable retry policies with exponential backoff for transient failures |
| Provider extensibility | Plugin architecture allowing new service providers without modifying SDK core |

The API Client is designed so that additional service providers (beyond Wirex) can be registered without changes to the core SDK. Each provider implements a standardized interface for authentication, request formatting, and error handling.

### 5.5 WebSocket and Streaming Module

**Purpose:** Enable real-time, event-driven application behavior using Horizon Server-Sent Events (SSE) and WebSocket connections for off-chain services.

**Stellar Components Used:**

- Horizon SSE endpoints: /accounts/{id}, /transactions, /payments, /ledgers
- EventSource API for persistent SSE connections with automatic cursor management

**Key Features:**

| Feature | Description | Stellar Integration |
|---------|-------------|-------------------|
| Account event streams | Subscribe to real-time balance changes and incoming payments | Horizon SSE /accounts/{id}/payments |
| Transaction streams | Monitor transaction confirmations across the network | Horizon SSE /transactions with cursor |
| Automatic reconnection | Detect lost connections and reconnect using last cursor position | SSE cursor-based pagination for seamless recovery |
| Event filtering | Selectively process events by type, account, or asset | Client-side filtering on Horizon SSE event payloads |
| Type-safe handlers | Strongly typed event interfaces for all Stellar event types | TypeScript types matching Horizon response schemas |

**Conceptual Interface:**

```typescript
const stream = sdk.streams.accountPayments('GACCOUNT...', {
  onPayment: (event) => {
    console.log(`Received ${event.amount} ${event.asset_code}`);
  },
  onError: (err) => console.error(err),
  cursor: 'now'  // Start from current ledger
});

// Automatic reconnection with cursor tracking
// stream.close() to stop subscription
```

### 5.6 Configuration Module

**Purpose:** Centralize all environment and network configuration, ensuring deterministic behavior across testnet and mainnet deployments.

**Key Features:**

| Feature | Description |
|---------|-------------|
| Environment switching | Toggle between testnet and mainnet with a single configuration parameter |
| Network parameters | Automatic resolution of Horizon URL, Soroban RPC URL, and network passphrase |
| API endpoint configuration | Centralized control over external service base URLs |
| Logging and debugging | Configurable log verbosity for development and production |
| Feature flags | Enable or disable specific SDK capabilities per deployment |

**Conceptual Interface:**

```typescript
const sdk = new StellarSDK({
  network: 'mainnet',              // or 'testnet'
  horizonUrl: 'https://horizon.stellar.org',
  sorobanRpcUrl: 'https://soroban-rpc.mainnet.stellar.org',
  providers: {
    wirex: { apiKey: '...', baseUrl: '...' }
  },
  logging: { level: 'info' }
});

// Mainnet deployment is a configuration change, not a rewrite
```

---

## 6. Reference Integration: Wirex Payments

Wirex payment infrastructure is included as a reference implementation within the SDK. This integration demonstrates how regulated payment flows can connect to Stellar-based applications and serves as a reusable template for other providers.

### 6.1 Payment Settlement Flow

The following describes the end-to-end settlement flow using Stellar as the settlement layer:

```
Step 1: User initiates card payment via client application
    |
Step 2: Client app calls SDK API Client -> Wirex payment API
    |
Step 3: Wirex processes payment authorization (off-chain, regulated)
    |
Step 4: SDK Transaction Module builds settlement transaction:
    |   - Payment operation: USDC or EURC to merchant account
    |   - Memo: settlement reference ID
    |   - Fee: estimated via GET /fee_stats
    |
Step 5: Transaction signed client-side (non-custodial)
    |
Step 6: Signed XDR submitted to Horizon POST /transactions
    |
Step 7: WebSocket Module streams confirmation via SSE
    |
Step 8: Settlement confirmed on Stellar ledger
    -> Transaction hash returned to client and Wirex backend
```

### 6.2 Key Characteristics

- **Non-custodial:** End-user assets remain in self-custody at all times. The SDK never stores or transmits private keys.
- **Provider-agnostic:** The Wirex integration uses the same API Client interface available to any provider. It is not hardcoded into the SDK.
- **On-chain settlement:** All settlement transactions are recorded on the Stellar ledger with full transparency and near-instant finality.
- **Stablecoin support:** Settlement uses USDC and EURC issued on Stellar, providing price stability for real-world payment use cases.

---

## 7. Security and Compliance Boundaries

The SDK enforces clear security boundaries to ensure safe operation in production environments:

| Principle | Implementation |
|-----------|---------------|
| No key custody | Private keys are never stored, transmitted, or logged by the SDK. All signing is client-side using Keypair or external signers. |
| Client-side signing only | Transaction XDR is constructed by the SDK but signed exclusively on the client device. |
| Regulated services external | KYC, AML, card issuance, and fiat rails remain outside the SDK boundary, handled by providers like Wirex. |
| Orchestration layer only | The SDK acts as an abstraction and orchestration layer. It does not hold funds, manage compliance, or process payments directly. |
| Minimal attack surface | Each module exposes only the interfaces required for its function. Internal implementation details are not accessible. |
| Input validation | All user-provided inputs (addresses, amounts, asset codes) are validated before constructing Stellar operations. |

---

## 8. Deployment and Environment Flow

### 8.1 Supported Environments

| Environment | Horizon | Soroban RPC | Use Case |
|-------------|---------|-------------|----------|
| Testnet | horizon-testnet.stellar.org | soroban-testnet.stellar.org | Development, testing, integration validation |
| Mainnet | horizon.stellar.org | Provider-dependent | Production deployment |

### 8.2 Deployment Flow

The SDK is designed so that mainnet deployment is a configuration change, not a code rewrite:

- Developer configures environment via Configuration Module (network: testnet or mainnet)
- SDK automatically resolves correct Horizon URL, Soroban RPC URL, and network passphrase
- Identical TypeScript APIs are used across both environments
- All modules route requests to the configured network endpoints
- Mainnet release requires only updating the network configuration parameter

---

## 9. Extensibility and Ecosystem Reuse

The SDK is designed as ecosystem infrastructure, not a single-application dependency.

### 9.1 Extensibility Features

- **Modular architecture:** each module can be imported and used independently
- **Provider-agnostic API client:** new payment or service providers can be registered without modifying SDK core
- **Clear separation:** Stellar logic is fully decoupled from external service integrations
- **Generic contract builder:** interact with any Soroban contract, not just predefined ones
- **TypeScript-first:** full type definitions for IDE support, documentation generation, and developer productivity

### 9.2 Ecosystem Use Cases

- SCF applicants can use the SDK as a dependency for their own Stellar projects
- Developers can extend individual modules or contribute new integrations
- Fintech companies can build Stellar payment applications using the SDK as infrastructure
- The Wirex reference integration serves as a template for connecting any regulated payment provider to Stellar

---

## 10. Mainnet Readiness

The SDK is designed for production deployment on Stellar mainnet. The following criteria will be met before the final release:

| Criterion | Approach |
|-----------|----------|
| Mainnet compatibility | Explicit mainnet support in Configuration Module with correct network passphrase and endpoints |
| Error handling | Comprehensive error normalization across Horizon, Soroban RPC, and external API failures |
| Transaction resilience | Retry logic with exponential backoff and sequence number management |
| Testing | Unit tests, integration tests on testnet, and end-to-end tests with mainnet-compatible configuration |
| Documentation | Full API reference, usage guides, and example integrations published alongside the SDK |
| Professional user testing | External developer testing conducted prior to final release (SCF requirement) |
| Open source | SDK published on npm and GitHub with clear contribution guidelines and licensing |

---

## 11. Summary

The Wirex-Vottun Stellar SDK provides a clean, modular, TypeScript-based abstraction over Stellar's core building blocks. It enables developers to build real-world payment applications without managing protocol complexity.

The SDK integrates directly with Horizon REST API, Soroban RPC, Stellar assets (XLM, USDC, EURC), and Server-Sent Events for real-time streaming. Each integration is mapped to specific endpoints and methods from the @stellar/stellar-sdk library.

By combining Stellar-native integrations, production-grade SDK design, a provider-agnostic architecture, and a reference payment integration with Wirex, this project delivers long-term value to the Stellar ecosystem as reusable developer infrastructure.
