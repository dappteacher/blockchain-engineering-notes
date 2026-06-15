# Solidity Gas Optimization: 10 Practical Techniques Every Smart Contract Developer Should Know

> Practical gas optimization techniques for Solidity developers, including storage optimization, calldata usage, custom errors, immutable variables, storage packing, benchmarking, and performance measurement.

## Table of Contents

* [Introduction](#introduction)
* [Why Gas Optimization Matters](#why-gas-optimization-matters)
* [Understanding Where Gas Is Spent](#understanding-where-gas-is-spent)
* [1. Use `calldata` Instead of `memory`](#1-use-calldata-instead-of-memory)
* [2. Use `external` Instead of `public`](#2-use-external-instead-of-public)
* [3. Cache Storage Variables](#3-cache-storage-variables)
* [4. Use Custom Errors](#4-use-custom-errors)
* [5. Use `unchecked` Arithmetic](#5-use-unchecked-arithmetic)
* [6. Pack Storage Variables Efficiently](#6-pack-storage-variables-efficiently)
* [7. Minimize Storage Writes](#7-minimize-storage-writes)
* [8. Avoid Unnecessary External Calls](#8-avoid-unnecessary-external-calls)
* [9. Use Immutable Variables](#9-use-immutable-variables)
* [10. Use Constants for Fixed Values](#10-use-constants-for-fixed-values)
* [Measure Optimizations Instead of Guessing](#measure-optimizations-instead-of-guessing)
* [When Not to Optimize](#when-not-to-optimize)
* [Gas Analyzer Project](#gas-analyzer-project)
* [Final Thoughts](#final-thoughts)
* [About the Author](#about-the-author)

---

## Introduction

Smart contract security is often the primary focus of blockchain development—and rightly so. A single vulnerability can put millions of dollars at risk.

However, there is another aspect of smart contract quality that directly affects users, protocols, and businesses every day:

**Gas Efficiency**

Every unnecessary storage read, storage write, memory allocation, or redundant operation increases transaction costs. For protocols processing thousands of transactions per day, even small inefficiencies can translate into substantial costs over time.

Gas optimization is not about writing clever code merely to save a few gas units. It is about understanding how the Ethereum Virtual Machine (EVM) works and making informed engineering decisions that improve efficiency without sacrificing:

* Security
* Readability
* Maintainability

In this article, we'll explore practical Solidity gas optimization techniques, discuss when they matter, and review the trade-offs every smart contract engineer should understand.

---

## Why Gas Optimization Matters

Gas represents the computational resources required to execute transactions on Ethereum and other EVM-compatible networks.

Every operation has a cost:

* Reading storage
* Writing storage
* Executing arithmetic
* Calling external contracts
* Allocating memory
* Emitting events

The more work a transaction performs, the more gas it consumes.

Although Layer 2 solutions have significantly reduced transaction costs, gas optimization remains important because it improves:

### Better User Experience

Lower transaction costs make protocols more accessible.

### Reduced Operational Costs

Protocols performing frequent transactions can save significant amounts over time.

### Improved Scalability

Efficient contracts allow more operations within block gas limits.

### Better Protocol Design

Optimized contracts often reflect a deeper understanding of EVM internals and system architecture.

---

## Understanding Where Gas Is Spent

Before optimizing anything, it's important to understand where most gas consumption originates.

The largest contributors are typically:

1. Storage writes (`SSTORE`)
2. Storage reads (`SLOAD`)
3. External calls
4. Contract deployment
5. Memory allocation
6. Event emissions

Storage operations are usually the most expensive.

Because of this, many effective optimizations focus on reducing storage interactions.

---

# 1. Use `calldata` Instead of `memory`

One of the simplest and most effective optimizations is choosing `calldata` when parameters do not need modification.

### Less Efficient

```solidity
function process(uint256[] memory values) external {
    // logic
}
```

### More Efficient

```solidity
function process(uint256[] calldata values) external {
    // logic
}
```

### Why?

When using `memory`, Solidity copies the entire array from calldata into memory.

Using `calldata` avoids this copy operation and allows direct access to transaction input data.

This becomes increasingly beneficial for:

* Large arrays
* Complex structs
* Dynamic data structures

---

# 2. Use `external` Instead of `public`

When a function is only intended to be called from outside the contract, prefer `external`.

### Less Efficient

```solidity
function getUser(uint256 id)
    public
    view
    returns (User memory)
{
    return users[id];
}
```

### More Efficient

```solidity
function getUser(uint256 id)
    external
    view
    returns (User memory)
{
    return users[id];
}
```

### Why?

For externally called functions, `external` allows Solidity to read arguments directly from calldata without additional memory copies.

This is particularly beneficial when passing:

* Arrays
* Strings
* Structs

### When Not to Use It

If the function must also be called internally, use `public`.

```solidity
function calculateFee()
    public
    view
    returns (uint256)
{
    return fee;
}

function process() external {
    uint256 currentFee = calculateFee();
}
```

Calling an external function internally requires:

```solidity
this.calculateFee();
```

which introduces an external call and increases gas costs.

### Rule of Thumb

| Usage Pattern       | Visibility |
| ------------------- | ---------- |
| External only       | `external` |
| Internal + External | `public`   |

---

# 3. Cache Storage Variables

Storage reads are expensive.

### Less Efficient

```solidity
for (uint256 i = 0; i < users.length; i++) {
    // logic
}
```

### More Efficient

```solidity
uint256 length = users.length;

for (uint256 i = 0; i < length; i++) {
    // logic
}
```

### Why?

The value is loaded from storage once and reused from memory.

This becomes increasingly valuable inside large loops.

---

# 4. Use Custom Errors

Before Solidity 0.8.4, developers commonly used revert strings.

### Traditional Approach

```solidity
require(msg.sender == owner, "Not owner");
```

### Modern Approach

```solidity
error NotOwner();

if (msg.sender != owner) {
    revert NotOwner();
}
```

### Benefits

* Smaller deployment bytecode
* Lower runtime gas costs
* More structured error handling
* Better scalability for large codebases

Custom errors are now considered a best practice in modern Solidity development.

---

# 5. Use `unchecked` Arithmetic

Since Solidity 0.8.x, arithmetic overflow checks are enabled by default.

### Standard Increment

```solidity
i++;
```

### Optimized Increment

```solidity
unchecked {
    ++i;
}
```

### Why?

The compiler skips overflow checks, reducing execution costs.

### Important

Only use `unchecked` when overflow is mathematically impossible or otherwise controlled.

Security should never be sacrificed for micro-optimizations.

---

# 6. Pack Storage Variables Efficiently

Storage layout directly affects gas costs.

### Poor Packing

```solidity
struct User {
    uint256 balance;
    uint128 score;
    uint64 timestamp;
}
```

### Better Packing

```solidity
struct User {
    uint128 balance;
    uint64 score;
    uint64 timestamp;
}
```

### Why?

The EVM stores data in 32-byte storage slots.

Properly arranged variables can share slots, reducing storage consumption and lowering gas costs.

### General Rule

Group smaller variables together whenever possible.

---

# 7. Minimize Storage Writes

Storage writes are among the most expensive EVM operations.

### Less Efficient

```solidity
counter++;
counter++;
counter++;
```

### More Efficient

```solidity
counter += 3;
```

### Better Yet

Perform calculations in memory first:

```solidity
uint256 newCounter = counter;
newCounter += 3;

counter = newCounter;
```

### Why?

Reducing the number of storage writes often yields significant gas savings.

---

# 8. Avoid Unnecessary External Calls

External calls introduce overhead and complexity.

### Less Efficient

```solidity
token.balanceOf(user);
token.balanceOf(user);
```

### More Efficient

```solidity
uint256 balance = token.balanceOf(user);
```

### Benefits

* Lower gas usage
* Reduced execution complexity
* Improved readability

Repeated external calls should be minimized whenever possible.

---

# 9. Use Immutable Variables

Values that never change after deployment should be declared `immutable`.

### Instead Of

```solidity
address public owner;
```

### Use

```solidity
address public immutable owner;
```

### Constructor Initialization

```solidity
constructor() {
    owner = msg.sender;
}
```

### Why?

Immutable variables are embedded directly into contract bytecode.

Reading them is cheaper than reading from storage.

### Common Candidates

* Owner addresses
* Treasury addresses
* Protocol configuration values
* External contract references

---

# 10. Use Constants for Fixed Values

### Instead Of

```solidity
uint256 public fee = 100;
```

### Use

```solidity
uint256 public constant FEE = 100;
```

### Why?

Constants are compiled directly into bytecode and require no storage slot.

Benefits include:

* Lower deployment costs
* Lower runtime gas consumption

---

# Measure Optimizations Instead of Guessing

One of the most common mistakes developers make is assuming an optimization is beneficial without measuring it.

Gas optimization should be evidence-based.

Useful tools include:

* Foundry Gas Reports
* Forge Snapshot
* Hardhat Gas Reporter
* Tenderly
* Slither

### Example Benchmark Results

| Optimization        | Before | After  | Improvement |
| ------------------- | ------ | ------ | ----------- |
| calldata parameters | 42,100 | 38,300 | 9%          |
| Custom Errors       | 28,600 | 27,100 | 5%          |
| Storage Packing     | 51,200 | 43,000 | 16%         |
| Immutable Variables | 24,000 | 22,800 | 5%          |

Actual savings depend on:

* Contract architecture
* Compiler version
* Execution path

The lesson is simple:

> Measure first. Optimize second.

---

# When Not to Optimize

Optimization is valuable, but it is not always the highest priority.

Excessive optimization can introduce:

* Reduced readability
* Increased complexity
* More difficult audits
* Higher maintenance costs
* Greater risk of subtle bugs

For example, saving 50 gas units by introducing obscure logic is rarely worthwhile if it makes the code significantly harder to understand.

Professional smart contract engineering requires balancing:

1. Security
2. Readability
3. Maintainability
4. Gas Efficiency

**Security should always come first.**

---

# Gas Analyzer Project

While researching and experimenting with optimization techniques, I found that many discussions around gas savings lacked reproducible benchmarks.

To better understand the impact of different patterns, I started building an open-source repository focused on practical gas optimization experiments and benchmarking.

### Goals

* Reproducible examples
* Real gas measurements
* Side-by-side comparisons
* Educational references
* Foundry-based benchmarking workflows

Repository:

**[https://github.com/dappteacher/gas-analyzer](https://github.com/dappteacher/gas-analyzer)**

As the repository evolves, additional benchmarks and optimization patterns will be added to cover increasingly complex smart contract scenarios.

---

# Final Thoughts

Gas optimization is an important skill for every Solidity developer, but it should never become an obsession.

The most effective engineers:

* Understand where gas is spent
* Measure performance objectively
* Optimize intentionally
* Never compromise security

If there is one takeaway from this article, it is this:

> Optimize intentionally, measure everything, and never compromise security for marginal gas savings.

Efficient contracts:

* Create better user experiences
* Lower operational costs
* Demonstrate deeper EVM understanding

As Ethereum and EVM ecosystems continue to evolve, the ability to balance security, maintainability, and performance will remain one of the defining characteristics of high-quality smart contract engineering.

---

# About the Author

**Yaghoub Adelzadeh**
Senior Blockchain Engineer | Smart Contract Architect | Security Researcher

Focused on:

* Smart Contract Security
* DeFi Protocol Design
* Blockchain Infrastructure
* EVM Optimization
* Web3 Engineering

I share practical engineering insights, audit lessons, and open-source tools for developers building the next generation of decentralized applications.

### Connect

* GitHub: [https://github.com/dappteacher](https://github.com/dappteacher)
* Gas Analyzer: [https://github.com/dappteacher/gas-analyzer](https://github.com/dappteacher/gas-analyzer)
* LinkedIn: Add your LinkedIn profile here

---

