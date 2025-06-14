# Fund Constants

## Overview

WhackRockFund uses carefully chosen constants to ensure security, efficiency, and proper economic incentives. All constants are immutable and cannot be changed after deployment.

## Basis Points System

### TOTAL_WEIGHT_BASIS_POINTS
```solidity
uint256 public constant TOTAL_WEIGHT_BASIS_POINTS = 10000;
```
- **Value**: 10000
- **Purpose**: Represents 100% for all percentage calculations
- **Usage**: Portfolio weights, fee calculations, slippage

{% hint style="info" %}
**Basis Points**: 1 basis point = 0.01%, so 10000 = 100%
{% endhint %}

## Fee Distribution

### AGENT_AUM_FEE_SHARE_BPS
```solidity
uint256 public constant AGENT_AUM_FEE_SHARE_BPS = 6000;
```
- **Value**: 6000 (60%)
- **Purpose**: Agent's share of collected AUM fees
- **Rationale**: Incentivizes agents to grow and manage funds effectively

### PROTOCOL_AUM_FEE_SHARE_BPS
```solidity
uint256 public constant PROTOCOL_AUM_FEE_SHARE_BPS = 4000;
```
- **Value**: 4000 (40%)
- **Purpose**: Protocol's share of collected AUM fees
- **Rationale**: Sustains protocol development and operations

**Fee Split Validation**:
```solidity
AGENT_AUM_FEE_SHARE_BPS + PROTOCOL_AUM_FEE_SHARE_BPS = 10000 ✓
```

## Trading Parameters

### DEFAULT_SLIPPAGE_BPS
```solidity
uint256 public constant DEFAULT_SLIPPAGE_BPS = 50;
```
- **Value**: 50 (0.5%)
- **Purpose**: Maximum acceptable slippage during swaps
- **Application**: Protects against MEV and sandwich attacks
- **Trade-off**: Higher = more likely to succeed, lower = better price

### SWAP_DEADLINE_OFFSET
```solidity
uint256 public constant SWAP_DEADLINE_OFFSET = 15 minutes;
```
- **Value**: 900 seconds
- **Purpose**: Time window for swap execution
- **Usage**: `block.timestamp + SWAP_DEADLINE_OFFSET`
- **Rationale**: Prevents indefinite pending transactions

### DEFAULT_POOL_STABILITY
```solidity
bool public constant DEFAULT_POOL_STABILITY = false;
```
- **Value**: false (use volatile pools)
- **Purpose**: Default Aerodrome pool type for swaps
- **Options**:
  - `true`: Stable pools (for correlated assets)
  - `false`: Volatile pools (for uncorrelated assets)

## Deposit Security

### MINIMUM_SHARES_LIQUIDITY
```solidity
uint256 private constant MINIMUM_SHARES_LIQUIDITY = 1000;
```
- **Value**: 1000
- **Visibility**: Private (internal use only)
- **Purpose**: Minimum shares for first deposit
- **Security**: Prevents share price manipulation attacks

### MINIMUM_INITIAL_DEPOSIT
```solidity
uint256 public constant MINIMUM_INITIAL_DEPOSIT = 0.01 ether;
```
- **Value**: 0.01 ETH (10^16 wei)
- **Purpose**: Required for first deposit into empty fund
- **Security**: Makes share price attacks economically unfeasible
- **User Impact**: ~$20-40 USD at typical ETH prices

### MINIMUM_DEPOSIT
```solidity
uint256 public constant MINIMUM_DEPOSIT = 0.01 ether;
```
- **Value**: 0.01 ETH (10^16 wei)
- **Purpose**: Minimum for all deposits
- **Rationale**: Prevents dust attacks and ensures meaningful investments

{% hint style="warning" %}
The minimum deposit requirements are critical security features and should not be circumvented
{% endhint %}

## Rebalancing Parameters

### REBALANCE_DEVIATION_THRESHOLD_BPS
```solidity
uint256 public constant REBALANCE_DEVIATION_THRESHOLD_BPS = 100;
```
- **Value**: 100 (1%)
- **Purpose**: Triggers automatic rebalancing
- **Behavior**: Rebalance when any asset deviates >1% from target
- **Trade-off**: Lower = more rebalancing (more gas), higher = larger tracking error

## Usage Examples

### Calculating Fees
```solidity
// Annual fee of 2% (200 basis points)
uint256 annualFeeBps = 200;
uint256 dailyFee = (nav * annualFeeBps) / (TOTAL_WEIGHT_BASIS_POINTS * 365);
```

### Checking Rebalance Need
```solidity
// Check if 3.5% deviation exceeds 1% threshold
uint256 deviationBps = 350; // 3.5%
bool needsRebalance = deviationBps > REBALANCE_DEVIATION_THRESHOLD_BPS; // true
```

### Setting Slippage
```solidity
// Calculate minimum output with 0.5% slippage
uint256 expectedOutput = 1000 * 10**18;
uint256 minOutput = expectedOutput * (TOTAL_WEIGHT_BASIS_POINTS - DEFAULT_SLIPPAGE_BPS) / TOTAL_WEIGHT_BASIS_POINTS;
// minOutput = 995 * 10**18
```

## Design Rationale

### Security Constants
- **Minimum Deposits**: Prevent economic attacks while remaining accessible
- **Share Liquidity**: Makes manipulation prohibitively expensive

### Economic Constants
- **Fee Split**: Balances agent incentives with protocol sustainability
- **Rebalance Threshold**: Minimizes unnecessary trading while maintaining allocations

### Operational Constants
- **Slippage**: Protects users while ensuring execution
- **Deadlines**: Prevents stuck transactions

## Comparison with Industry Standards

| Parameter | WhackRock | Traditional Funds | DeFi Average |
|-----------|-----------|-------------------|--------------|
| Min Investment | 0.01 ETH | $1,000-10,000 | 0-0.1 ETH |
| Slippage Tolerance | 0.5% | N/A | 0.3-3% |
| Rebalance Threshold | 1% | 5-10% | 1-5% |
| Fee Structure | 60/40 split | Variable | 80/20 - 100/0 |

## Gas Optimization

Using constants provides several benefits:
1. **No SLOAD**: Constants are inlined at compile time
2. **Predictable Costs**: No storage reads needed
3. **Compiler Optimization**: Better optimization opportunities

Example gas savings:
```solidity
// Using constant (cheap)
uint256 shares = amount * TOTAL_WEIGHT_BASIS_POINTS / nav;

// Using storage variable (expensive)
uint256 shares = amount * totalWeightBps / nav; // SLOAD cost
```

## Future Considerations

While constants cannot be changed, future fund versions might adjust:
- Lower minimum deposits as gas costs decrease
- Dynamic slippage based on market conditions
- Flexible rebalancing thresholds per asset
- Alternative fee structures

{% hint style="info" %}
Constants are part of the fund's immutable contract. New funds can be deployed with different constants if needed.
{% endhint %}

## Related Documentation

- [State Variables](state-variables.md) - Mutable contract state
- [Portfolio Management](portfolio-mgmt.md) - How constants affect rebalancing
- [Fee Collection](fee-collection.md) - Fee calculation details