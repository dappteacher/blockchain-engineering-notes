# Uniswap V2, V3, and V4: A Security Engineer's Guide to the Evolution of AMMs

> Understanding the architectural evolution of Uniswap from Constant Product Pools to Concentrated Liquidity and Hooks

**Original Medium Article:**

[Read on Medium](https://medium.com/@dappteacher/uniswap-v2-v3-and-v4-a-security-engineers-guide-to-the-evolution-of-amms-7611d6cd07d3)

---

## Table of Contents

* [Introduction](#introduction)
* [Why Uniswap Matters](#why-uniswap-matters)
* [The Evolution of AMMs](#the-evolution-of-amms)
* [Uniswap V2](#uniswap-v2)

  * Constant Product Formula
  * Architecture
  * Security Considerations
* [Uniswap V3](#uniswap-v3)

  * Concentrated Liquidity
  * Liquidity Positions as NFTs
  * Security Considerations
* [Uniswap V4](#uniswap-v4)

  * Singleton Architecture
  * Flash Accounting
  * Hooks
  * Security Considerations
* [V2 vs V3 vs V4 Comparison](#v2-vs-v3-vs-v4-comparison)
* [How Auditors Analyze Uniswap-Based Protocols](#how-auditors-analyze-uniswap-based-protocols)
* [Common Vulnerabilities Around AMMs](#common-vulnerabilities-around-amms)
* [Lessons for Smart Contract Engineers](#lessons-for-smart-contract-engineers)
* [Conclusion](#conclusion)
* [Further Reading](#further-reading)
* [About the Author](#about-the-author)

---

# Introduction

Few protocols have influenced decentralized finance as profoundly as Uniswap.

What began as a relatively simple implementation of an Automated Market Maker (AMM) evolved into one of the most important pieces of financial infrastructure in the blockchain ecosystem.

Each major version of Uniswap introduced new innovations:

* V2 introduced permissionless liquidity pools.
* V3 introduced concentrated liquidity.
* V4 introduces programmable liquidity through hooks.

For smart contract engineers and security researchers, understanding these architectural changes is essential because every innovation introduces new assumptions, trade-offs, and attack surfaces.

---

# Why Uniswap Matters

Uniswap fundamentally changed how decentralized trading works.

Before AMMs, decentralized exchanges primarily relied on order books.

Uniswap demonstrated that liquidity could be provided through smart contracts using mathematical pricing models.

Today, many DeFi systems depend on:

* Uniswap liquidity
* Uniswap price discovery
* Uniswap oracles
* Uniswap integrations

As a result, understanding Uniswap is valuable far beyond understanding a single protocol.

---

# The Evolution of AMMs

The progression of Uniswap can be summarized as follows:

```text
Uniswap V1
      ↓
Uniswap V2
      ↓
Uniswap V3
      ↓
Uniswap V4
```

Each version aimed to improve:

* Capital efficiency
* Liquidity utilization
* Customizability
* Gas efficiency

while maintaining decentralization and composability.

---

# Uniswap V2

## Constant Product Formula

Uniswap V2 popularized the constant product market maker:

```text
x * y = k
```

Where:

* x = reserve of token A
* y = reserve of token B
* k = constant

Every swap adjusts reserves while preserving the invariant.

---

## Architecture

```text
Trader
   ↓
Pair Contract
   ↓
Token Reserves
```

Core components include:

* Factory Contract
* Pair Contracts
* Router Contracts

---

## Advantages

* Simplicity
* Predictability
* Strong composability
* Easy integration

---

## Security Considerations

### Flash Loan Manipulation

Large temporary liquidity can distort prices.

### Oracle Manipulation

Protocols relying on spot prices become vulnerable.

### Sandwich Attacks

MEV searchers can front-run and back-run trades.

### Low Liquidity Risks

Small pools can be manipulated relatively easily.

---

# Uniswap V3

## The Capital Efficiency Problem

One limitation of V2 is that liquidity is distributed across the entire price curve.

In practice, most liquidity is rarely utilized.

---

## Concentrated Liquidity

V3 allows liquidity providers to select specific price ranges.

Example:

```text
ETH Price Range:
1500 - 2500 USD
```

Liquidity becomes concentrated where trading activity occurs.

---

## Benefits

* Greater capital efficiency
* Higher fee generation
* Better liquidity utilization

---

## NFT-Based Liquidity Positions

Unlike V2 LP tokens, V3 positions are unique.

Each position includes:

* Price range
* Liquidity amount
* Fee configuration

Therefore positions are represented as NFTs.

---

## Security Considerations

### Range Manipulation

Price movements may force liquidity out of active ranges.

### Oracle Complexity

Oracle calculations became significantly more complex.

### Liquidity Fragmentation

Liquidity distributed across multiple ranges introduces additional risks.

### Increased Accounting Complexity

More complex logic means larger audit surfaces.

---

# Uniswap V4

## Architectural Shift

V4 is not simply an iteration.

It introduces a fundamentally different design philosophy.

---

## Singleton Architecture

Previous versions deployed separate contracts per pool.

V4 uses a singleton architecture.

```text
Pool A
Pool B
Pool C
   ↓
Single Contract
```

### Benefits

* Lower deployment costs
* Improved efficiency
* Reduced gas consumption

---

## Flash Accounting

V4 introduces flash accounting to minimize token transfers during execution.

Benefits include:

* Improved gas efficiency
* Reduced redundant state transitions

---

## Hooks

Hooks are the most significant innovation introduced in V4.

Hooks allow developers to inject custom logic at specific execution points.

Examples:

```text
Before Swap
After Swap

Before Mint
After Mint

Before Burn
After Burn
```

---

## Potential Applications

* Dynamic fees
* Custom oracles
* Automated rebalancing
* TWAMMs
* Compliance systems
* Yield strategies

---

## Security Considerations

### Hook Vulnerabilities

Every hook introduces new trust assumptions.

### Reentrancy Risks

Custom execution paths may create unexpected state transitions.

### Access Control Issues

Improper permissions can compromise pool behavior.

### Accounting Errors

Custom accounting logic increases complexity.

### Integration Risks

External dependencies increase attack surface.

---

# V2 vs V3 vs V4 Comparison

| Feature                | V2  | V3     | V4   |
| ---------------------- | --- | ------ | ---- |
| Constant Product AMM   | ✓   | ✓      | ✓    |
| Concentrated Liquidity | ✗   | ✓      | ✓    |
| NFT Positions          | ✗   | ✓      | ✓    |
| Singleton Architecture | ✗   | ✗      | ✓    |
| Hooks                  | ✗   | ✗      | ✓    |
| Flash Accounting       | ✗   | ✗      | ✓    |
| Capital Efficiency     | Low | High   | High |
| Complexity             | Low | Medium | High |
| Audit Difficulty       | Low | Medium | High |

---

# How Auditors Analyze Uniswap-Based Protocols

Professional security reviews often focus on:

### State Transitions

Can invariants be violated?

### Accounting Models

Can balances become inconsistent?

### Oracle Assumptions

Can prices be manipulated?

### External Dependencies

What happens when dependencies fail?

### Permission Models

Who can change critical parameters?

### Economic Attacks

Can protocol incentives be abused?

---

# Common Vulnerabilities Around AMMs

## Price Manipulation

Attackers exploit insufficient liquidity.

## Oracle Exploitation

Protocols trust manipulated AMM prices.

## Flash Loan Attacks

Temporary capital influences protocol state.

## Sandwich Attacks

Users receive unfavorable execution.

## Liquidity Extraction

Economic incentives become misaligned.

---

# Lessons for Smart Contract Engineers

The evolution of Uniswap demonstrates an important engineering principle:

> Every improvement in flexibility introduces additional complexity.

V2 optimized simplicity.

V3 optimized capital efficiency.

V4 optimizes programmability.

As protocols become more powerful, they also become more difficult to reason about and secure.

Engineers should evaluate whether additional flexibility genuinely justifies the added risk.

---

# Conclusion

The history of Uniswap reflects the broader evolution of decentralized finance.

Each version solved important limitations:

* V2 introduced permissionless AMMs.
* V3 improved capital efficiency.
* V4 enables programmable liquidity.

For security researchers and auditors, understanding these transitions is critical because vulnerabilities increasingly emerge from complex interactions rather than isolated coding mistakes.

The future of AMMs will likely continue moving toward greater flexibility, making security analysis more important than ever.

---

# Further Reading

### Related Articles

* [Uniswap V2, V3, and V4: A Security Engineer's Guide to the Evolution of AMMs (Medium)](https://medium.com/@dappteacher/uniswap-v2-v3-and-v4-a-security-engineers-guide-to-the-evolution-of-amms-7611d6cd07d3?utm_source=chatgpt.com)
* [ERC-4626 Vulnerabilities Every Smart Contract Auditor Should Know](https://medium.com/@dappteacher/erc-4626-vulnerabilities-every-smart-contract-auditor-should-know-51c7172d672f?utm_source=chatgpt.com)
* [The Essential DeFi Design Patterns Every Smart Contract Engineer and Auditor Should Know](https://medium.com/@dappteacher/the-essential-defi-design-patterns-every-smart-contract-engineer-and-auditor-should-know-da4fe63c09e2?utm_source=chatgpt.com)
* [Gas Analyzer Repository](https://github.com/dappteacher/gas-analyzer?utm_source=chatgpt.com)

---

# About the Author

## Yaghoub Adelzadeh

**Smart Contract Architect | Blockchain Security Researcher | DeFi Engineer**

Areas of Focus:

* Smart Contract Security
* DeFi Protocol Design
* AMM Architecture
* Security Auditing
* Gas Optimization

### Connect

* GitHub: [https://github.com/dappteacher](https://github.com/dappteacher)
* Gas Analyzer: [https://github.com/dappteacher/gas-analyzer](https://github.com/dappteacher/gas-analyzer)
* LinkedIn: [https://linkedin.com/in/dappteacher](https://linkedin.com/in/dappteacher)

---
