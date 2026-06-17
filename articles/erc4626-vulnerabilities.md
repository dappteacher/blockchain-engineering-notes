# ERC-4626 Vulnerabilities Every Smart Contract Auditor Should Know

> A Practical Security Guide to ERC-4626 Vault Risks, Exploitation Vectors, and Audit Methodologies

**Original Medium Article:**

[Read on Medium](https://medium.com/@dappteacher/erc-4626-vulnerabilities-every-smart-contract-auditor-should-know-51c7172d672f)

---

## Table of Contents

* [Introduction](#introduction)
* [What Is ERC-4626?](#what-is-erc-4626)
* [Why ERC-4626 Became Popular](#why-erc-4626-became-popular)
* [Understanding Vault Accounting](#understanding-vault-accounting)
* [Security Assumptions Auditors Must Verify](#security-assumptions-auditors-must-verify)
* [1. Inflation Attacks](#1-inflation-attacks)
* [2. Share Price Manipulation](#2-share-price-manipulation)
* [3. Donation Attacks](#3-donation-attacks)
* [4. Flash Loan Manipulation](#4-flash-loan-manipulation)
* [5. Rounding Vulnerabilities](#5-rounding-vulnerabilities)
* [6. First Depositor Risks](#6-first-depositor-risks)
* [7. Oracle Dependency Risks](#7-oracle-dependency-risks)
* [8. Fee Calculation Vulnerabilities](#8-fee-calculation-vulnerabilities)
* [9. Reentrancy Risks](#9-reentrancy-risks)
* [10. Strategy Integration Risks](#10-strategy-integration-risks)
* [Audit Methodology for ERC-4626 Vaults](#audit-methodology-for-erc-4626-vaults)
* [Checklist for Security Review](#checklist-for-security-review)
* [Conclusion](#conclusion)
* [Further Reading](#further-reading)
* [About the Author](#about-the-author)

---

# Introduction

ERC-4626 has become the standard interface for tokenized vaults in the Ethereum ecosystem.

The standard simplifies vault integration by providing a consistent framework for:

* Deposits
* Withdrawals
* Yield generation
* Share accounting
* Asset management

Protocols such as:

* Yield aggregators
* Liquid staking systems
* Structured products
* Restaking protocols

have increasingly adopted ERC-4626 because of its composability and interoperability.

However, standardization does not eliminate security risks.

Many vulnerabilities observed in production vaults arise from incorrect assumptions about share accounting, asset valuation, strategy integration, and user behavior.

This article examines the most important ERC-4626 security considerations that auditors should understand.

---

# What Is ERC-4626?

ERC-4626 defines a standardized interface for yield-bearing vaults.

A vault accepts an underlying asset and issues shares representing ownership of the vault's assets.

Example:

```text
User Deposits USDC
        ↓
ERC-4626 Vault
        ↓
Receives Vault Shares
```

As the vault generates yield, the value of each share increases.

---

# Why ERC-4626 Became Popular

Before ERC-4626, every vault implementation used custom interfaces.

This created integration challenges for:

* Wallets
* Aggregators
* Protocols
* Frontends

ERC-4626 introduced a common standard that significantly improved composability across DeFi.

Benefits include:

* Standardized integrations
* Better developer experience
* Reduced implementation complexity
* Improved ecosystem interoperability

---

# Understanding Vault Accounting

The security of ERC-4626 systems revolves around one fundamental concept:

## Share Price

The relationship between assets and shares determines ownership.

Simplified formula:

```text
Share Price =
Total Assets / Total Shares
```

Every deposit, withdrawal, donation, strategy gain, and strategy loss ultimately affects this ratio.

Because of this, auditors spend significant time reviewing accounting logic.

---

# Security Assumptions Auditors Must Verify

When reviewing ERC-4626 implementations, auditors should validate:

* Asset accounting
* Share accounting
* Conversion functions
* Strategy interactions
* Fee mechanisms
* Oracle integrations
* Withdrawal logic

Most critical vulnerabilities originate from violations of these assumptions.

---

# 1. Inflation Attacks

## Overview

Inflation attacks occur when attackers manipulate share issuance during low-liquidity conditions.

Most commonly:

1. Attacker becomes first depositor
2. Vault receives direct asset donation
3. Share price becomes distorted
4. Future users receive unfavorable share allocations

### Example

```text
Attacker Deposits 1 Token
          ↓
Receives 1 Share

Attacker Donates 1,000 Tokens
          ↓
Share Price Increases

Victim Deposits
          ↓
Receives Fewer Shares Than Expected
```

---

# 2. Share Price Manipulation

Share price is a primary target because many calculations depend on it.

Potential vectors include:

* Donations
* Flash loans
* Oracle manipulation
* Strategy accounting errors

Auditors should verify that share price calculations remain robust under adversarial conditions.

---

# 3. Donation Attacks

ERC-4626 vaults often rely on:

```solidity
asset.balanceOf(address(this))
```

to determine total assets.

This can be dangerous because anyone can transfer assets directly to the vault.

### Risks

* Artificial share inflation
* Accounting inconsistencies
* Manipulated exchange rates

### Mitigation

Maintain internal accounting instead of relying exclusively on token balances.

---

# 4. Flash Loan Manipulation

Flash loans enable temporary capital access within a single transaction.

Attackers may use flash loans to manipulate:

* Share price
* Asset valuation
* Oracle values
* Deposit calculations

Auditors should evaluate vault behavior under extreme capital fluctuations.

---

# 5. Rounding Vulnerabilities

Rounding errors frequently appear in:

* Deposit calculations
* Withdraw calculations
* Mint functions
* Redeem functions

Examples:

```solidity
convertToAssets()

convertToShares()

previewDeposit()

previewMint()
```

Even small rounding inconsistencies can create:

* Value leakage
* Unfair distributions
* Economic exploits

---

# 6. First Depositor Risks

The first depositor often establishes the initial vault state.

If improperly handled, this can enable:

* Share inflation
* Precision manipulation
* Economic attacks

Common mitigations include:

* Minimum liquidity
* Dead shares
* Bootstrap deposits

---

# 7. Oracle Dependency Risks

Vaults relying on external pricing mechanisms inherit oracle risk.

Examples include:

* Strategy valuation
* Collateral pricing
* Reward accounting

Auditors should review:

* Oracle update frequency
* Failure handling
* Manipulation resistance

---

# 8. Fee Calculation Vulnerabilities

Many ERC-4626 vaults include:

* Deposit fees
* Withdrawal fees
* Performance fees
* Management fees

Common mistakes include:

* Incorrect fee ordering
* Double charging
* Rounding inconsistencies
* Share dilution

Fee logic should be carefully modeled and tested.

---

# 9. Reentrancy Risks

Although ERC-4626 itself does not create reentrancy vulnerabilities, vault implementations often interact with:

* Strategies
* Reward contracts
* External protocols

Auditors should inspect:

```solidity
deposit()

withdraw()

mint()

redeem()
```

for external-call ordering issues.

---

# 10. Strategy Integration Risks

Most production vaults deploy capital into external strategies.

Examples:

```text
Vault
 ↓
Strategy
 ↓
Lending Protocol
 ↓
Yield Generation
```

Additional risks include:

* Strategy insolvency
* Protocol exploits
* Liquidity shortages
* Incorrect accounting

The vault is only as secure as the systems it depends on.

---

# Audit Methodology for ERC-4626 Vaults

Professional audits typically focus on:

### Step 1

Review accounting model.

### Step 2

Verify share issuance logic.

### Step 3

Test inflation attack scenarios.

### Step 4

Analyze donation handling.

### Step 5

Evaluate rounding behavior.

### Step 6

Review strategy integrations.

### Step 7

Test extreme edge cases.

---

# Checklist for Security Review

### Accounting

* [ ] Total assets correctly tracked
* [ ] Total shares correctly tracked
* [ ] Conversion functions consistent

### Deposits

* [ ] Inflation resistant
* [ ] Donation resistant
* [ ] Proper rounding

### Withdrawals

* [ ] Accurate redemption
* [ ] No share dilution
* [ ] Strategy liquidity handled

### External Dependencies

* [ ] Oracle assumptions reviewed
* [ ] Strategy risks assessed
* [ ] Reentrancy mitigations present

### Fees

* [ ] Correct fee ordering
* [ ] Accurate fee accounting
* [ ] No unexpected dilution

---

# Conclusion

ERC-4626 significantly improves vault interoperability across the DeFi ecosystem.

However, the standard introduces unique accounting and economic considerations that require careful review.

Many high-severity findings arise not from traditional coding mistakes but from incorrect assumptions about:

* Share issuance
* Asset valuation
* Strategy integration
* Economic behavior

For auditors, understanding these patterns is essential for identifying risks before they reach production.

---

# Further Reading

### Related Articles

* [ERC-4626 Vulnerabilities Every Smart Contract Auditor Should Know (Medium)](https://medium.com/@dappteacher/erc-4626-vulnerabilities-every-smart-contract-auditor-should-know-51c7172d672f?utm_source=chatgpt.com)
* [The Essential DeFi Design Patterns Every Smart Contract Engineer and Auditor Should Know](https://medium.com/@dappteacher/the-essential-defi-design-patterns-every-smart-contract-engineer-and-auditor-should-know-da4fe63c09e2?utm_source=chatgpt.com)
* [Gas Analyzer Repository](https://github.com/dappteacher/gas-analyzer?utm_source=chatgpt.com)

---

# About the Author

## Yaghoub Adelzadeh

**Smart Contract Architect | Blockchain Security Researcher | DeFi Engineer**

Specializing in:

* Smart Contract Security
* DeFi Architecture
* ERC Standards
* Security Auditing
* Protocol Design

### Connect

* GitHub: [https://github.com/dappteacher](https://github.com/dappteacher)
* Gas Analyzer: [https://github.com/dappteacher/gas-analyzer](https://github.com/dappteacher/gas-analyzer)
* LinkedIn: [https://linkedin.com/in/dappteacher](https://linkedin.com/in/dappteacher)

---

