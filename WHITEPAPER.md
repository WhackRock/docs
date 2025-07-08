# WhackRock Protocol: A Trust Layer for AI-Managed Investment Funds

**Version 1.1**  
**July 2025**

---

## Abstract

WhackRock Protocol establishes a decentralized infrastructure for AI-managed investment funds on Base, enabling AI agents to manage tokenized investment vehicles while ensuring investor protection through immutable smart contracts. The protocol integrates with GAME SDK (Virtuals Protocol) to provide Model Context Protocol (MCP) tooling for agentic portfolio management.

The WROCK token captures value from the entire ecosystem through protocol fee distribution. As total assets under management (AUM) grow across all funds, WROCK stakers receive 40% of all management fees collected, creating direct alignment between protocol growth and token value. With planned token-gating for fund creation and management, WROCK becomes essential infrastructure for participating in AI investments.

AI agents operate within strict security constraints: they can only trade pre-specified tokens defined at fund creation, cannot add new assets or withdraw funds, and are limited to portfolio weight adjustments and rebalancing. All constraints are enforced at the smart contract level, ensuring investor assets remain secure while enabling sophisticated autonomous strategies.

---

## 1. Introduction

The emergence of agentic investment strategies has created significant opportunities, but current implementations operate in black boxes without verifiable performance tracking, create centralized points of failure, and lack standardized frameworks for comparison.

WhackRock Protocol solves these challenges through transparent on-chain operations, standardized smart contract infrastructure, and MCP-enabled agent tooling. Every fund action is recorded immutably, creating accountability and trust in agentic portfolio management.

---

## 2. Protocol Architecture

### 2.1 Core Components

The protocol consists of three main components working in harmony:

**Fund Registry**: A central directory maintaining all WhackRock funds, enabling discovery, performance tracking, and agent verification.

**Fund Contracts**: Individual ERC20-based investment vehicles implementing portfolio management, automatic rebalancing, fee collection, and TWAP-based pricing for accurate valuations.

**Agent Integration Layer**: The MCP-enabled connection between GAME SDK agents and fund contracts, providing standardized tooling, permission controls, and audit trails for all agentic decisions.

### 2.2 GAME SDK and MCP Integration with Security Constraints

WhackRock leverages GAME SDK with MCP tooling to power agents while maintaining strict security through smart contract enforcement. Agents receive limited permissions at deployment, restricting them to portfolio weight adjustments and rebalancing triggers only.

Critical security constraints are hardcoded: agents cannot withdraw funds, add new tokens, change fund parameters, or execute arbitrary transactions. Token lists are immutable after fund creation, ensuring agents can only trade pre-approved, highly liquid assets that passed security validation. This sandbox approach enables sophisticated agentic strategies while protecting investor assets.

---

## 3. Fund Creation and Management

### 3.1 Creating a Fund with GAME Agents

Fund creation requires configuring a GAME agent with specific investment strategies and deploying an immutable fund contract. The process involves selecting verified tokens (which cannot be changed post-deployment), setting initial portfolio weights, and configuring security parameters like slippage protection and rebalance thresholds.

Each fund operates with hardcoded constraints: agents receive only weight adjustment and rebalancing permissions, token lists remain immutable, and all security parameters are enforced at the contract level. This ensures consistent security across all funds while allowing diverse investment strategies.

### 3.2 Agentic Fund Management Operations

GAME agents utilize MCP tools to operate within strict boundaries enforced by smart contracts. They continuously monitor market conditions for whitelisted tokens, calculate optimal portfolio weights, and trigger rebalancing when allocations deviate from targets.

The security model is absolute: agents cannot add tokens, withdraw funds, or override any security parameters. Every action requires smart contract validation, trades execute through secure DEX integrations, and fund assets never leave the contract's custody. Users maintain full control with the ability to withdraw their proportional share at any time.

---

## 4. Investment and Portfolio Management

### 4.1 Investor Participation

Investors participate by depositing WETH into funds and receiving ERC20 shares representing proportional ownership. Share pricing dynamically adjusts based on NAV, with automatic minting on deposits and burning on withdrawals. Investors maintain full liquidity with no lock-ups, receiving their proportional share of all fund assets upon withdrawal. TWAP oracles ensure fair pricing for all transactions.

### 4.2 Portfolio Rebalancing

Funds automatically rebalance when portfolio weights deviate beyond configured thresholds (typically 1-5%). The rebalancing process monitors weight deviations, executes trades through Uniswap V3, and restores target allocations. TWAP pricing and slippage protection ensure fair execution while protecting against MEV attacks.

---

## 5. Economic Model and WROCK Value Accrual

### 5.1 Fee Structure and Distribution

The protocol generates revenue through annual management fees on all funds (up to 10% maximum, typically 2-5%). These fees are collected by minting new fund shares and distributing them according to a fixed split: 60% to the agent managing the fund and 40% to the protocol.

The 40% protocol share flows directly to WROCK stakers, creating a perpetual value stream that scales with total AUM. As more funds launch and grow, WROCK stakers receive an increasing share of management fees across the entire ecosystem. This mechanism ensures that WROCK appreciation is directly tied to protocol adoption and success

### 5.2 Staking and Protocol Rewards

WROCK staking provides the primary value accrual mechanism for token holders. The staking contract implements flexible lock periods from 6 months to 2 years, with longer locks earning bonus points multipliers (0% for 6 months, 50% for 9 months, 100% for 1 year).

Stakers earn rewards from two sources: protocol fee distributions (40% of all fund management fees) and points accumulation for future airdrops. The time-weighted points system ensures fair distribution based on both stake amount and duration, creating incentives for long-term protocol alignment.

Importantly, future updates will introduce token-gating requirements where fund creators and agents must maintain minimum staked WROCK balances to participate in the ecosystem, creating additional demand pressure and value accrual for the token

### 5.3 Points System and Airdrop Mechanism

The protocol implements a points-based reward system where users accumulate points through staking activities. The staking contract uses a time-weighted calculation where 1 WROCK token staked for 365 days earns 1 base point, with bonus multipliers for longer lock periods (up to 100% bonus for 1-year locks).

Accumulated points can be redeemed for reward tokens through the PointsRedeemer contract, which manages airdrop distributions. The redemption rate is configurable by protocol governance, allowing flexible reward mechanisms as the ecosystem grows. This creates additional value accrual for WROCK stakers beyond protocol fee distributions

### 5.4 Token-Gated Participation (Future Enhancement)

The protocol will introduce token-gating requirements to align long-term incentives and ensure quality control. Fund creators and agents will need to maintain minimum staked WROCK balances to participate in the ecosystem. This mechanism creates additional demand for WROCK while ensuring participants have skin in the game.

These staking requirements will scale with fund size and performance, creating natural quality filters and aligning agent incentives with protocol success. Token-gating ensures only committed participants can deploy agents with MCP tooling access

---

## 6. Security and Risk Management

### 6.1 Smart Contract Security

Security is enforced through immutable design choices and access control hierarchies. Token lists are fixed at deployment with no ability to add or modify assets. Only verified tokens with proven liquidity profiles are allowed, preventing manipulation through illiquid assets.

The contracts implement strict separation of concerns: agents can only adjust weights and trigger rebalances, while only shareholders can withdraw funds. This prevents any possibility of fund drainage by compromised agents. Time-locked parameter changes and emergency pause mechanisms provide additional safety layers.

### 6.2 Agent Constraints

Agents operate within a hardcoded sandbox limited to three functions: setting target weights, triggering rebalances, and collecting protocol-defined fees. The smart contracts explicitly prevent agents from adding tokens, withdrawing funds, or executing arbitrary transactions. All agent actions are logged on-chain, creating transparent audit trails for performance analysis and security monitoring of MCP tool usage

---

## 7. Governance and Decentralization

### 7.1 Protocol Development

WhackRock Protocol development follows a community-driven approach with protocol upgrades, parameter adjustments, and agent certification processes managed through decentralized governance mechanisms. Future implementations will introduce formal governance structures as the protocol matures and decentralizes further

### 7.2 Ecosystem Growth Dynamics

The WhackRock ecosystem creates natural growth dynamics through aligned incentives. As successful funds attract more AUM, they generate higher fees that flow to WROCK stakers. This creates a virtuous cycle where protocol success directly benefits token holders.

Competition among agents drives innovation and performance improvements. Transparent on-chain performance tracking allows investors to easily compare agentic strategies and migrate between funds, ensuring only the best agents thrive. This market-driven selection process continuously improves the quality of available MCP-powered investment strategies

---

## 8. Technical Implementation

### 8.1 Smart Contract Architecture

The fund contracts expose a minimal interface split between public functions (deposit, withdraw) and agent-restricted functions (setTargetWeights, triggerRebalance). Critical security is achieved through what the contracts don't implement: no functions exist for adding tokens, transferring funds directly, or modifying security parameters.

### 8.2 GAME Agent and MCP Integration

GAME agents utilize MCP tools to generate investment signals and execute them through authenticated contract calls. The integration ensures agents can only call their permitted functions, with all actions verified through cryptographic signatures and rate limiting to prevent abuse

---

## 9. WROCK Token Value Proposition

### 9.1 Direct Value Accrual Mechanisms

**Protocol Fee Distribution**: WROCK stakers receive 40% of all management fees collected across the entire fund ecosystem. As AUM grows from millions to billions, this creates a substantial and increasing revenue stream for token holders.

**Token-Gated Access**: Future implementation of staking requirements for fund creation and management creates persistent demand for WROCK tokens. Larger funds and more successful agents will require higher stake amounts, creating natural buy pressure.

**Points-Based Rewards**: Time-weighted staking earns points redeemable for additional token distributions, creating compound returns for long-term holders. The flexible redemption mechanism allows protocol governance to optimize reward distribution as the ecosystem evolves.

### 9.2 Network Effects and Growth Dynamics

WROCK benefits from powerful network effects: more funds attract more investors, generating more fees for stakers, which attracts more funds in a virtuous cycle. As the protocol becomes the standard for AI-managed investments, WROCK captures value from the entire ecosystem's growth.

The token's value is fundamentally tied to protocol adoption. Every new fund launched, every dollar of AUM added, and every successful investment strategy developed directly benefits WROCK holders through increased fee generation and ecosystem participation requirements

---

## 10. Future Developments

### 10.1 Protocol Roadmap

Near-term developments focus on implementing token-gating mechanisms, expanding supported assets, and optimizing gas costs through Layer 2 integration. The protocol will introduce graduated staking requirements based on fund size and performance metrics.

Medium-term goals include cross-chain expansion to capture broader DeFi opportunities, institutional custody integration for traditional finance participation, and advanced agent capabilities through continued GAME SDK development.

### 10.2 Ecosystem Expansion

The protocol aims to become the standard infrastructure for agentic investments across DeFi. Strategic partnerships with agent developers, integration with major DeFi protocols, and MCP tooling enhancements will drive adoption. As regulatory frameworks develop, WhackRock will adapt to enable compliant institutional participation while maintaining decentralization

---

## 11. Conclusion

WhackRock Protocol creates a new safer future for investment management where agents operate their sophisticated strategies using our MCP tooling within secure, transparent smart contracts. The WROCK token captures value from this revolution through direct fee distribution, ecosystem participation requirements, and network effects that compound with growth.

As AUM scales from current levels to billions under management, WROCK holders benefit proportionally from every dollar managed by the protocol. The combination of immediate fee distribution, future token-gating, and points-based rewards creates multiple value accrual mechanisms that align with long-term protocol success.

By solving the trust problem in agentic investments, WhackRock enables a future where anyone can access sophisticated MCP-powered strategies previously reserved for institutions, while token holders capture the value created by this democratization of finance



---

**Repository**: [https://github.com/WhackRock/whackrock-treasury-template](https://github.com/WhackRock/whackrock-treasury-template)  
**Documentation**: [https://github.com/WhackRock/docs](https://github.com/WhackRock/docs)  
**Website**: [https://whackrock.ai](https://whackrock.ai)

*This whitepaper is subject to updates as the protocol evolves. Please refer to the official documentation for the most current information.*