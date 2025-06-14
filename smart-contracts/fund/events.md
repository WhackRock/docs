# Fund Events

## Overview

WhackRockFund emits comprehensive events for all significant operations, enabling real-time monitoring, analysis, and integration with external systems.

## Investment Events

### WETHDepositedAndSharesMinted

```solidity
event WETHDepositedAndSharesMinted(
    address indexed depositor,
    address indexed receiver,
    uint256 wethDeposited,
    uint256 sharesMinted,
    uint256 navBeforeDepositWETH,
    uint256 totalSupplyBeforeDeposit,
    uint256 wethValueInUSDC
)
```

**Emitted When**: WETH is deposited and shares are minted

**Indexed Parameters**:
- `depositor`: Address that provided the WETH
- `receiver`: Address that received the shares

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `wethDeposited` | `uint256` | Amount of WETH invested |
| `sharesMinted` | `uint256` | Shares created for investor |
| `navBeforeDepositWETH` | `uint256` | Fund NAV before deposit |
| `totalSupplyBeforeDeposit` | `uint256` | Share supply before deposit |
| `wethValueInUSDC` | `uint256` | WETH/USDC exchange rate |

**Use Cases**:
- Track investment flow
- Calculate share prices
- Monitor fund growth
- USD-denominated reporting

### BasketAssetsWithdrawn

```solidity
event BasketAssetsWithdrawn(
    address indexed owner,
    address indexed receiver,
    uint256 sharesBurned,
    address[] tokensWithdrawn,
    uint256[] amountsWithdrawn,
    uint256 navBeforeWithdrawalWETH,
    uint256 totalSupplyBeforeWithdrawal,
    uint256 totalWETHValueOfWithdrawal,
    uint256 wethValueInUSDC
)
```

**Emitted When**: Shares are burned and assets withdrawn

**Indexed Parameters**:
- `owner`: Address that owned the shares
- `receiver`: Address that received the assets

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `sharesBurned` | `uint256` | Shares destroyed |
| `tokensWithdrawn` | `address[]` | Tokens transferred out |
| `amountsWithdrawn` | `uint256[]` | Amounts of each token |
| `navBeforeWithdrawalWETH` | `uint256` | Fund NAV before withdrawal |
| `totalSupplyBeforeWithdrawal` | `uint256` | Share supply before withdrawal |
| `totalWETHValueOfWithdrawal` | `uint256` | Total value in WETH terms |
| `wethValueInUSDC` | `uint256` | WETH/USDC exchange rate |

**Use Cases**:
- Track redemptions
- Analyze withdrawal patterns
- Calculate asset allocations
- Monitor fund outflows

## Portfolio Management Events

### TargetWeightsUpdated

```solidity
event TargetWeightsUpdated(
    address indexed agent,
    address[] tokens,
    uint256[] weights,
    uint256 timestamp
)
```

**Emitted When**: Agent updates target portfolio weights

**Indexed Parameters**:
- `agent`: Agent that made the change

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `tokens` | `address[]` | Token addresses |
| `weights` | `uint256[]` | New target weights (BPS) |
| `timestamp` | `uint256` | When change occurred |

**Use Cases**:
- Track strategy changes
- Monitor agent activity
- Analyze portfolio evolution
- Strategy backtesting

### RebalanceCheck

```solidity
event RebalanceCheck(
    bool needsRebalance,
    uint256 maxDeviationBPS,
    uint256 currentNAV_AA
)
```

**Emitted When**: System checks if rebalancing is needed

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `needsRebalance` | `bool` | Whether rebalancing triggered |
| `maxDeviationBPS` | `uint256` | Largest allocation deviation |
| `currentNAV_AA` | `uint256` | Current NAV in WETH |

**Use Cases**:
- Monitor allocation drift
- Track rebalancing triggers
- Analyze portfolio maintenance
- Gas usage optimization

### RebalanceCycleExecuted

```solidity
event RebalanceCycleExecuted(
    uint256 navBeforeRebalanceAA,
    uint256 navAfterRebalanceAA,
    uint256 blockTimestamp,
    uint256 wethValueInUSDC
)
```

**Emitted When**: A complete rebalancing cycle finishes

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `navBeforeRebalanceAA` | `uint256` | NAV before trades |
| `navAfterRebalanceAA` | `uint256` | NAV after trades |
| `blockTimestamp` | `uint256` | When rebalancing completed |
| `wethValueInUSDC` | `uint256` | WETH/USDC rate |

**Use Cases**:
- Measure rebalancing costs
- Track portfolio efficiency
- Monitor slippage impact
- Performance analysis

### FundTokenSwapped

```solidity
event FundTokenSwapped(
    address indexed tokenFrom,
    uint256 amountFrom,
    address indexed tokenTo,
    uint256 amountTo
)
```

**Emitted When**: Individual token swaps occur during rebalancing

**Indexed Parameters**:
- `tokenFrom`: Token being sold
- `tokenTo`: Token being bought

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `amountFrom` | `uint256` | Amount sold |
| `amountTo` | `uint256` | Amount received |

**Use Cases**:
- Track individual trades
- Calculate swap rates
- Monitor DEX efficiency
- Slippage analysis

## Management Events

### AgentUpdated

```solidity
event AgentUpdated(
    address indexed oldAgent,
    address indexed newAgent
)
```

**Emitted When**: Fund owner changes the agent

**Indexed Parameters**:
- `oldAgent`: Previous agent address
- `newAgent`: New agent address

**Use Cases**:
- Track agent changes
- Monitor governance
- Agent performance attribution
- Access control auditing

### AgentAumFeeCollected

```solidity
event AgentAumFeeCollected(
    address indexed agentFeeWallet,
    uint256 agentSharesMinted,
    address indexed protocolFeeRecipient,
    uint256 protocolSharesMinted,
    uint256 totalFeeValueInAccountingAsset,
    uint256 navAtFeeCalculation,
    uint256 totalSharesAtFeeCalculation,
    uint256 timestamp
)
```

**Emitted When**: AUM fees are collected and distributed

**Indexed Parameters**:
- `agentFeeWallet`: Agent's fee recipient
- `protocolFeeRecipient`: Protocol's fee recipient

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `agentSharesMinted` | `uint256` | Shares minted to agent |
| `protocolSharesMinted` | `uint256` | Shares minted to protocol |
| `totalFeeValueInAccountingAsset` | `uint256` | Total fee value in WETH |
| `navAtFeeCalculation` | `uint256` | NAV when fees calculated |
| `totalSharesAtFeeCalculation` | `uint256` | Share supply when fees calculated |
| `timestamp` | `uint256` | When fees were collected |

**Use Cases**:
- Track fee revenue
- Calculate effective rates
- Monitor agent compensation
- Fee performance analysis

## Emergency Events

### EmergencyWithdrawal

```solidity
event EmergencyWithdrawal(
    address indexed token,
    uint256 amount
)
```

**Emitted When**: Fund owner executes emergency withdrawal

**Indexed Parameters**:
- `token`: Token that was withdrawn

**Data Parameters**:
- `amount`: Amount withdrawn

**Use Cases**:
- Monitor emergency actions
- Security incident tracking
- Governance oversight
- Risk management

## Event Monitoring Examples

### Web3.js Event Listening

```javascript
// Listen for deposits
fund.events.WETHDepositedAndSharesMinted()
    .on('data', (event) => {
        const { depositor, wethDeposited, sharesMinted } = event.returnValues;
        console.log(`Deposit: ${depositor} invested ${wethDeposited} WETH for ${sharesMinted} shares`);
    });

// Listen for rebalancing
fund.events.RebalanceCycleExecuted()
    .on('data', (event) => {
        const { navBeforeRebalanceAA, navAfterRebalanceAA } = event.returnValues;
        const impact = (navAfterRebalanceAA - navBeforeRebalanceAA) / navBeforeRebalanceAA;
        console.log(`Rebalance completed, NAV impact: ${impact * 100}%`);
    });
```

### Ethers.js Filtering

```javascript
// Get all deposits by a specific user
const depositFilter = fund.filters.WETHDepositedAndSharesMinted(userAddress);
const deposits = await fund.queryFilter(depositFilter);

// Get fee collections in last month
const oneMonth = 30 * 24 * 3600;
const feeFilter = fund.filters.AgentAumFeeCollected();
const fees = await fund.queryFilter(feeFilter, -oneMonth * 6500); // Approximate blocks

// Calculate total fees collected
const totalFees = fees.reduce((sum, event) => 
    sum + BigInt(event.args.totalFeeValueInAccountingAsset), 0n
);
```

### GraphQL Integration

```graphql
query FundActivity($fundAddress: String!, $from: Int!) {
    wethDepositedAndSharesMinteds(
        where: { fundAddress: $fundAddress, blockNumber_gte: $from }
        orderBy: blockNumber
        orderDirection: desc
    ) {
        depositor
        wethDeposited
        sharesMinted
        navBeforeDepositWETH
        blockNumber
        transactionHash
    }
    
    rebalanceCycleExecuteds(
        where: { fundAddress: $fundAddress, blockNumber_gte: $from }
    ) {
        navBeforeRebalanceAA
        navAfterRebalanceAA
        blockTimestamp
        blockNumber
    }
}
```

## Event-Driven Analytics

### Real-time Dashboard

```javascript
class FundDashboard {
    constructor(fund) {
        this.fund = fund;
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        // Track deposits/withdrawals
        this.fund.on('WETHDepositedAndSharesMinted', this.handleDeposit.bind(this));
        this.fund.on('BasketAssetsWithdrawn', this.handleWithdrawal.bind(this));
        
        // Track portfolio changes
        this.fund.on('TargetWeightsUpdated', this.handleWeightUpdate.bind(this));
        this.fund.on('RebalanceCycleExecuted', this.handleRebalance.bind(this));
        
        // Track fees
        this.fund.on('AgentAumFeeCollected', this.handleFeeCollection.bind(this));
    }
    
    handleDeposit(depositor, receiver, wethDeposited, sharesMinted, ...args) {
        this.updateAUM(wethDeposited, 'deposit');
        this.updateShareCount(sharesMinted, 'mint');
        this.notifyLargeDeposit(wethDeposited);
    }
    
    handleRebalance(navBefore, navAfter, timestamp, wethUSDC) {
        const cost = navBefore - navAfter;
        const costBPS = (cost * 10000n) / navBefore;
        this.trackRebalancingCost(costBPS);
    }
}
```

### Performance Analytics

```javascript
class PerformanceAnalyzer {
    async analyzePeriod(fund, startBlock, endBlock) {
        // Get all relevant events
        const deposits = await fund.queryFilter(
            fund.filters.WETHDepositedAndSharesMinted(), 
            startBlock, 
            endBlock
        );
        
        const withdrawals = await fund.queryFilter(
            fund.filters.BasketAssetsWithdrawn(),
            startBlock,
            endBlock
        );
        
        const rebalances = await fund.queryFilter(
            fund.filters.RebalanceCycleExecuted(),
            startBlock,
            endBlock
        );
        
        // Calculate metrics
        return {
            totalDeposits: this.sumDeposits(deposits),
            totalWithdrawals: this.sumWithdrawals(withdrawals),
            rebalancingCosts: this.calculateRebalancingCosts(rebalances),
            netFlow: this.calculateNetFlow(deposits, withdrawals)
        };
    }
}
```

## Best Practices

### Event Reliability

1. **Handle Reorgs**: Wait for confirmations before considering events final
2. **Missing Events**: Implement catch-up mechanisms for missed events
3. **Duplicate Events**: Use transaction hashes to deduplicate
4. **Rate Limiting**: Implement backoff for RPC providers

### Performance Optimization

1. **Filter Efficiently**: Use indexed parameters for faster queries
2. **Batch Queries**: Query multiple events in single calls
3. **Block Ranges**: Use reasonable ranges to avoid timeouts
4. **Caching**: Store processed events to reduce RPC calls

### Data Processing

1. **Decimal Handling**: Use BigNumber libraries for precise calculations
2. **Time Zones**: Store timestamps in UTC
3. **Currency Conversion**: Use consistent base units (wei)
4. **Event Ordering**: Events within a transaction are deterministically ordered

## Troubleshooting

### Common Issues

1. **Event Not Firing**: Check contract address and ABI
2. **Missing Data**: Verify network and block range
3. **Parsing Errors**: Ensure ABI matches deployed contract
4. **Performance**: Use filters and pagination for large datasets

### Debugging Tips

```javascript
// Log all events for debugging
fund.on('*', (event) => {
    console.log('Event:', event.event, event.args);
});

// Check if contract is at expected address
const code = await provider.getCode(fund.address);
if (code === '0x') {
    console.error('No contract at address');
}

// Verify event exists in ABI
const eventExists = fund.interface.events['WETHDepositedAndSharesMinted'];
console.log('Event in ABI:', !!eventExists);
```

## Related Documentation

- [Investment Operations](investment-ops.md) - Functions that emit investment events
- [Portfolio Management](portfolio-mgmt.md) - Rebalancing events
- [Fee Collection](fee-collection.md) - Fee-related events