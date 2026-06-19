# Solidity Gas Optimization: 10 Practical Techniques Every Smart Contract Developer Should Know

> A Practical Guide to Reducing Smart Contract Costs Through Better EVM Design and Efficient Solidity Patterns

**Original Medium Article:**

[Read on Medium](https://medium.com/@dappteacher/solidity-gas-optimization-10-practical-techniques-every-smart-contract-developer-should-know-89c47b9c32f4?utm_source=chatgpt.com)

---

## Table of Contents

* [Introduction](#introduction)
* [Why Gas Optimization Matters](#why-gas-optimization-matters)
* [How Gas Works in the EVM](#how-gas-works-in-the-evm)
* [1. Use `calldata` Instead of `memory`](#1-use-calldata-instead-of-memory)
* [2. Prefer `external` Over `public`](#2-prefer-external-over-public)
* [3. Cache Storage Reads](#3-cache-storage-reads)
* [4. Use Custom Errors](#4-use-custom-errors)
* [5. Use `unchecked` Arithmetic](#5-use-unchecked-arithmetic)
* [6. Optimize Storage Packing](#6-optimize-storage-packing)
* [7. Reduce Storage Writes](#7-reduce-storage-writes)
* [8. Use Immutable Variables](#8-use-immutable-variables)
* [9. Use Constants](#9-use-constants)
* [10. Optimize Loops](#10-optimize-loops)
* [How Auditors Evaluate Gas Efficiency](#how-auditors-evaluate-gas-efficiency)
* [Benchmarking and Measurement](#benchmarking-and-measurement)
* [Common Optimization Mistakes](#common-optimization-mistakes)
* [Conclusion](#conclusion)
* [Further Reading](#further-reading)
* [About the Author](#about-the-author)

---

# Introduction

Gas optimization is one of the most practical skills a Solidity developer can develop.

Every smart contract interaction consumes computational resources, and those resources are paid for through gas fees.

While optimization should never compromise security or readability, reducing unnecessary gas consumption can provide significant benefits:

* Lower transaction costs
* Better user experience
* Increased protocol efficiency
* Reduced operational expenses
* Improved scalability

This article explores ten practical optimization techniques that can be applied immediately in modern Solidity development.

---

# Why Gas Optimization Matters

Gas represents the computational effort required to execute EVM operations.

Every action consumes gas:

| Operation                | Relative Cost |
| ------------------------ | ------------- |
| Arithmetic               | Low           |
| Memory Access            | Low           |
| Storage Read (`SLOAD`)   | High          |
| Storage Write (`SSTORE`) | Very High     |
| External Call            | High          |

Understanding these costs helps developers focus on optimizations that matter.

The biggest gains usually come from reducing:

* Storage reads
* Storage writes
* Memory allocations
* External calls

---

# How Gas Works in the EVM

A Solidity contract is compiled into bytecode.

The EVM executes that bytecode through opcodes.

Example:

```text
counter++;
```

becomes something similar to:

```text
SLOAD
ADD
SSTORE
```

The expensive operations are not the arithmetic instructions.

The storage interactions dominate gas consumption.

This is why many optimization techniques focus on reducing storage access.

---

# 1. Use `calldata` Instead of `memory`

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

`memory` copies input data into memory.

`calldata` accesses transaction input data directly.

Benefits become significant for:

* Large arrays
* Strings
* Structs
* Dynamic data

---

# 2. Prefer `external` Over `public`

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

External functions can access calldata directly without unnecessary memory copying.

### When Not To Use It

Use `public` if the function must also be called internally.

---

# 3. Cache Storage Reads

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

Repeated storage reads trigger multiple `SLOAD` operations.

Caching reduces expensive storage access.

---

# 4. Use Custom Errors

### Traditional Approach

```solidity
require(msg.sender == owner, "Not Owner");
```

### Optimized Approach

```solidity
error NotOwner();

if (msg.sender != owner) {
    revert NotOwner();
}
```

### Benefits

* Reduced bytecode size
* Lower deployment cost
* Lower runtime gas usage
* Better error handling

---

# 5. Use `unchecked` Arithmetic

### Standard

```solidity
i++;
```

### Optimized

```solidity
unchecked {
    ++i;
}
```

### Why?

Solidity 0.8+ performs overflow checks automatically.

When overflow is impossible, `unchecked` removes unnecessary verification.

### Warning

Never use this unless overflow is mathematically impossible or fully controlled.

---

# 6. Optimize Storage Packing

The EVM stores data in 32-byte slots.

### Poor Layout

```solidity
struct User {
    uint256 balance;
    uint128 score;
    uint64 timestamp;
}
```

### Better Layout

```solidity
struct User {
    uint128 balance;
    uint64 score;
    uint64 timestamp;
}
```

### Benefits

* Fewer storage slots
* Lower deployment costs
* Lower runtime costs

---

# 7. Reduce Storage Writes

### Less Efficient

```solidity
counter++;
counter++;
counter++;
```

### Better

```solidity
counter += 3;
```

### Best

```solidity
uint256 cached = counter;

cached += 3;

counter = cached;
```

### Why?

Storage writes are among the most expensive EVM operations.

---

# 8. Use Immutable Variables

### Instead Of

```solidity
address public owner;
```

### Use

```solidity
address public immutable owner;
```

### Constructor

```solidity
constructor() {
    owner = msg.sender;
}
```

### Benefits

Immutable values are embedded directly into bytecode.

Reading them is cheaper than reading storage.

---

# 9. Use Constants

### Instead Of

```solidity
uint256 public fee = 100;
```

### Use

```solidity
uint256 public constant FEE = 100;
```

### Benefits

* No storage slot required
* Lower deployment cost
* Lower execution cost

---

# 10. Optimize Loops

### Less Efficient

```solidity
for(uint256 i = 0; i < users.length; i++) {
    // logic
}
```

### More Efficient

```solidity
uint256 length = users.length;

for(uint256 i; i < length;) {

    // logic

    unchecked {
        ++i;
    }
}
```

### Benefits

* Fewer storage reads
* Reduced arithmetic overhead

---

# How Auditors Evaluate Gas Efficiency

Security auditors increasingly review gas efficiency because excessive gas consumption may indicate:

* Poor architecture
* Hidden complexity
* Potential denial-of-service vectors
* Scalability limitations

Common review areas include:

### Storage Usage

* Unnecessary `SLOAD`
* Unnecessary `SSTORE`

### Memory Allocation

* Excessive copying
* Dynamic allocation inefficiencies

### Loops

* Unbounded iteration
* Repeated storage access

### External Calls

* Duplicate interactions
* Unnecessary dependency usage

---

# Benchmarking and Measurement

Optimization should always be measured.

Useful tools include:

## Foundry

```bash
forge test --gas-report
forge snapshot
```

## Hardhat Gas Reporter

Provides automated gas metrics during testing.

## Slither

Can identify common optimization opportunities.

Resource:

[Slither Static Analyzer](https://github.com/crytic/slither?utm_source=chatgpt.com)

## Tenderly

Provides transaction profiling and execution traces.

Resource:

[Tenderly Platform](https://tenderly.co?utm_source=chatgpt.com)

---

# Common Optimization Mistakes

### Over-Optimizing

Saving a few gas units while dramatically reducing readability.

### Ignoring Security

Gas optimization should never weaken protocol security.

### Premature Optimization

Optimize after profiling.

### Not Measuring

Assumptions often lead to incorrect conclusions.

---

# Conclusion

Gas optimization is ultimately about understanding how the EVM executes smart contracts.

The most effective engineers focus on:

* Storage efficiency
* Bytecode efficiency
* Execution efficiency
* Measurement-driven improvements

The goal is not writing clever code.

The goal is building secure, maintainable, and efficient systems.

Remember:

> Measure first. Optimize second. Secure always.

---

# Further Reading

### Related Articles

* [Solidity Gas Optimization: 10 Practical Techniques Every Smart Contract Developer Should Know (Medium)](https://medium.com/@dappteacher/solidity-gas-optimization-10-practical-techniques-every-smart-contract-developer-should-know-89c47b9c32f4?utm_source=chatgpt.com)
* [Understanding Opcode and Bytecode in Solidity: Their Role in Gas Optimization](https://medium.com/@dappteacher/understanding-opcode-and-bytecode-in-solidity-their-role-in-gas-optimization-261a2fc1fd7e?utm_source=chatgpt.com)
* [Gas Analyzer Repository](https://github.com/dappteacher/gas-analyzer?utm_source=chatgpt.com)
* [ERC-4626 Vulnerabilities Every Smart Contract Auditor Should Know](https://medium.com/@dappteacher/erc-4626-vulnerabilities-every-smart-contract-auditor-should-know-51c7172d672f?utm_source=chatgpt.com)

---

# About the Author

## Yaghoub Adelzadeh

**Smart Contract Architect | Blockchain Security Researcher | EVM Optimization Specialist**

Areas of Focus:

* Smart Contract Security
* Gas Optimization
* EVM Internals
* DeFi Architecture
* Protocol Auditing

### Connect

* [GitHub Profile](https://github.com/dappteacher?utm_source=chatgpt.com)
* [Medium Profile](https://medium.com/@dappteacher?utm_source=chatgpt.com)

---

One recommendation: because you already have both the **Opcode & Bytecode** article and this **10 Gas Optimization Techniques** article, link them together aggressively. The opcode article explains *why* gas costs exist, while this article explains *how* to reduce them. Together they create a strong "EVM Performance Engineering" section in your GitHub knowledge base.

---

### Connect

* GitHub: [https://github.com/dappteacher](https://github.com/dappteacher)
* Gas Analyzer: [https://github.com/dappteacher/gas-analyzer](https://github.com/dappteacher/gas-analyzer)
* LinkedIn: [https://linkedin.com/in/dappteacher](https://linkedin.com/in/dappteacher)


---