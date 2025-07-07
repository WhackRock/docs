# WHACKROCK Documentation

WHACKROCK is infrastructure for AI agents to manage decentralized investment funds on blockchain.

## Getting Started



### For Fund Creators and AI Developers
**Want to create your own AI-managed fund?**
1. [Creating a Fund](protocol/fund-creation.md) - Complete frontend guide with AI agent setup
2. [Fund Management](smart-contracts/fund/overview.md) - How funds operate
3. [Fee Structure](smart-contracts/fund/fee-collection.md) - Revenue model

**Want to let your AI agents manage funds?**
1. [AI Integration Guide](protocol/ai-integration.md) - Technical integration with frontend monitoring
2. [GAME SDK Plugin](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury) - Python SDK for fund management
3. [Portfolio Management](smart-contracts/fund/portfolio-mgmt.md) - Agent capabilities and constraints

### For Investors
**Want to invest in AI-managed funds?**
1. [How to Invest](protocol/investing.md) - Complete step-by-step investment guide using the dApp
2. [Understanding Returns](smart-contracts/fund/investment-ops.md) - How deposits and withdrawals work
3. [Protocol Overview](protocol/overview.md) - Learn about the system

## Documentation Sections

### Protocol Overview
- [Protocol Overview](protocol/overview.md) - Introduction to WHACKROCK
- [Architecture](protocol/architecture.md) - System design
- [Economics](protocol/economics.md) - Fee structure and incentives

### Smart Contracts
- [Smart Contracts Overview](smart-contracts/README.md) - Contract architecture
- [Fund Contract](smart-contracts/fund/overview.md) - Individual fund implementation
- **Source Code**: [WhackRock Treasury Template Repository](https://github.com/WhackRock/whackrock-treasury-template)

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

## About These Docs

These documentation guides are designed for the WHACKROCK website and focus on practical usage of the frontend dApp. They include:

- **Frontend Integration**: Step-by-step guides using the actual WHACKROCK dApp interface
- **User Experience**: Complete workflows for investors, fund creators, and AI developers
- **GAME Plugin Integration**: Direct links and setup instructions for AI agent development
- **Real-world Usage**: Based on the actual whackrock-dapp implementation on Base network

For technical smart contract details, see the Smart Contracts section.

## License

WHACKROCK smart contracts are licensed under BUSL-1.1.