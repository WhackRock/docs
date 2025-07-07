# WHACKROCK Documentation

WHACKROCK is infrastructure for AI agents to manage decentralized investment funds on blockchain.

## Documentation Sections

### Protocol Overview
- [Protocol Overview](protocol/overview.md) - Introduction to WHACKROCK
- [Architecture](protocol/architecture.md) - System design
- [AI Integration](protocol/ai-integration.md) - How AI agents work with WHACKROCK
- [Economics](protocol/economics.md) - Fee structure and incentives

### Smart Contracts
- [Smart Contracts Overview](smart-contracts/README.md) - Contract architecture
- [Fund Contract](smart-contracts/fund/overview.md) - Individual fund implementation

## Quick Start

### For AI Developers
1. Read [AI Integration](protocol/ai-integration.md)
2. Study [Portfolio Management](smart-contracts/fund/portfolio-mgmt.md)
3. Use the [GAME SDK Plugin](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)

### For Fund Creators
1. Learn about [Fund Contract](smart-contracts/fund/overview.md)
2. Understand [Fee Collection](smart-contracts/fund/fee-collection.md)
3. Read [Economic Model](protocol/economics.md)

### For Investors
1. Check [Investment Operations](smart-contracts/fund/investment-ops.md)
2. Understand the [Protocol Overview](protocol/overview.md)

## Key Features

- **WETH-Only Deposits**: Funds accept WETH deposits and provide proportional basket withdrawals
- **Automated Rebalancing**: Maintains target allocations with 1% deviation threshold
- **Agent-Managed**: AI agents can set target weights and trigger rebalancing
- **GAME SDK Integration**: Python plugin for easy AI agent integration
- **Uniswap V3 Integration**: Uses TWAP oracles and direct pool swaps

## GAME SDK Plugin

The WhackRock GAME SDK plugin provides:
- Portfolio monitoring functions
- Automated rebalancing capabilities  
- Secure agent authorization
- Easy integration with GAME framework

Get started: [WRTreasury Plugin Documentation](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)

## License

WHACKROCK smart contracts are licensed under BUSL-1.1.