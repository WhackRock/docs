# WhackRockFund Overview

## Introduction

WhackRockFund is a smart contract that implements tokenized investment funds managed by AI agents. Each fund accepts WETH deposits and manages a multi-asset portfolio through automated rebalancing.

## Key Features

### ERC20 Tokenized Shares
- Fund shares are ERC20 tokens representing ownership
- Shares minted on WETH deposits, burned on withdrawals
- Share price based on fund NAV

### Multi-Asset Portfolios  
- Supports multiple approved ERC20 tokens
- Target weights set by authorized agent
- Automatic rebalancing when weights deviate >1%

### WETH-Only Deposits
- Investors deposit WETH only (not ETH)
- Minimum deposit: 0.01 WETH
- Withdrawals provide proportional basket of all fund assets

### Agent Management
- Authorized agent can set target weights and trigger rebalancing
- Agent cannot withdraw funds, only manage allocations
- Fund owner can change agent anytime

## Contract Details

- **Version**: WhackRockFundV6_UniSwap_TWAP
- **License**: BUSL-1.1  
- **Solidity**: ^0.8.20
- **DEX Integration**: Uniswap V3 with TWAP oracles

## Inheritance Structure

```solidity
contract WhackRockFund is 
    IWhackRockFund,          // Interface
    ERC20,                   // Token functionality  
    Ownable,                 // Access control
    UniswapV3TWAPOracle,     // Price feeds
    IUniswapV3SwapCallback   // DEX integration
```

## Core Mechanics

### Net Asset Value (NAV)
- Total fund value calculated in WETH (accounting asset)
- Includes all token balances converted to WETH via TWAP oracles
- Used for share price calculations

### Share Price 
- First deposit: 1:1 with WETH deposited
- Subsequent deposits: `shares = deposit * totalShares / NAV`
- Withdrawals burn shares proportionally

### Rebalancing Process
1. Agent sets new target weights in basis points (must sum to 10000)
2. System calculates deviations from current weights
3. If deviation >1%, rebalancing can be triggered
4. Swaps execute through Uniswap V3 pools
5. TWAP oracles provide slippage protection

### Fee Structure
- Maximum AUM fee: 10% annually (1000 basis points)
- Default split: 60% agent, 40% protocol (6000/4000 basis points)
- Fees collected by minting new shares to recipients

## Fund Operations

### Deposits
- Only WETH accepted (not ETH)
- Minimum: 0.01 WETH
- Shares minted proportional to NAV
- Automatic rebalancing triggered after deposits

### Withdrawals  
- Burn shares to receive proportional basket
- Receive all fund tokens, not just WETH
- ERC20 allowance mechanism supported

### Agent Functions
- `setTargetWeights`: Update portfolio targets
- `setTargetWeightsAndRebalanceIfNeeded`: Atomic weight update + rebalancing
- `triggerRebalance`: Manual rebalancing execution

### Owner Functions
- `setAgent`: Change authorized agent
- `collectAgentManagementFee`: Collect accrued fees

## Key Constraints

- WETH-only deposits (no direct ETH)
- Basket withdrawals only (no single-asset)
- Agent authorization required for portfolio changes
- 1% rebalancing threshold minimum
- 10% maximum annual AUM fees

## Related Documentation

- [Investment Operations](investment-ops.md) - Deposit/withdraw details
- [Portfolio Management](portfolio-mgmt.md) - Rebalancing mechanics
- [Fee Collection](fee-collection.md) - Fee structure details