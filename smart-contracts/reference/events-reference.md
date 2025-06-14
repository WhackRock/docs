# Events Reference

## Overview

This comprehensive reference documents all events emitted by WHACKROCK contracts, their parameters, use cases, and integration patterns. Events are critical for tracking protocol activity and building responsive applications.

## Registry Events

### WhackRockFundCreated

```solidity
event WhackRockFundCreated(
    uint256 indexed fundId,
    address indexed fundAddress,
    address indexed creator,
    address initialAgent,
    string vaultName,
    string vaultSymbol,
    string vaultURI,
    string description,
    address[] allowedTokens,
    uint256[] targetWeights,
    address agentAumFeeWallet,
    uint256 agentTotalAumFeeBps,
    uint256 timestamp
);
```

**Emitted When**: A new fund is successfully created via registry

**Indexed Parameters** (for efficient filtering):
- `fundId`: Sequential fund identifier
- `fundAddress`: Address of the deployed fund contract
- `creator`: Address that created the fund

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `initialAgent` | `address` | Agent managing the fund |
| `vaultName` | `string` | ERC20 name of fund shares |
| `vaultSymbol` | `string` | ERC20 symbol of fund shares |
| `vaultURI` | `string` | Metadata URI |
| `description` | `string` | Fund description |
| `allowedTokens` | `address[]` | Tokens the fund can hold |
| `targetWeights` | `uint256[]` | Initial target weights (BPS) |
| `agentAumFeeWallet` | `address` | Agent's fee recipient |
| `agentTotalAumFeeBps` | `uint256` | Total AUM fee rate |
| `timestamp` | `uint256` | Block timestamp |

**Integration Example**:
```javascript
// Listen for new funds
registry.on('WhackRockFundCreated', (fundId, fundAddress, creator, agent, name, symbol, uri, description, tokens, weights, feeWallet, feeRate, timestamp) => {
    console.log(`New fund: ${name} (${symbol})`);
    console.log(`Address: ${fundAddress}`);
    console.log(`Agent: ${agent}`);
    console.log(`Fee Rate: ${feeRate / 100}%`);
    
    // Add to fund database
    addFundToDatabase({
        id: fundId.toNumber(),
        address: fundAddress,
        name,
        symbol,
        creator,
        agent,
        tokens,
        weights: weights.map(w => w.toNumber()),
        feeRate: feeRate.toNumber(),
        createdAt: new Date(timestamp.toNumber() * 1000)
    });
});

// Query historical fund creations
const filter = registry.filters.WhackRockFundCreated();
const events = await registry.queryFilter(filter, 0); // From genesis

events.forEach(event => {
    const { fundId, fundAddress, creator } = event.args;
    console.log(`Fund ${fundId}: ${fundAddress} by ${creator}`);
});
```

### RegistryAllowedTokenAdded

```solidity
event RegistryAllowedTokenAdded(address indexed token);
```

**Emitted When**: Token is added to registry allowlist

**Parameters**:
- `token` (indexed): Address of the added token

**Use Cases**:
- Update UI token selection lists
- Notify users of new investment options
- Track protocol expansion

**Integration Example**:
```javascript
registry.on('RegistryAllowedTokenAdded', async (token) => {
    console.log(`New token added: ${token}`);
    
    // Fetch token metadata
    const tokenContract = new ethers.Contract(token, ERC20_ABI, provider);
    const [name, symbol, decimals] = await Promise.all([
        tokenContract.name(),
        tokenContract.symbol(),
        tokenContract.decimals()
    ]);
    
    // Update token list
    updateTokenList({
        address: token,
        name,
        symbol,
        decimals,
        addedAt: new Date()
    });
});
```

### RegistryAllowedTokenRemoved

```solidity
event RegistryAllowedTokenRemoved(address indexed token);
```

**Emitted When**: Token is removed from registry allowlist

**Parameters**:
- `token` (indexed): Address of the removed token

**Use Cases**:
- Remove token from UI selection
- Alert users with existing positions
- Track protocol risk management

**Integration Example**:
```javascript
registry.on('RegistryAllowedTokenRemoved', (token) => {
    console.log(`Token removed: ${token}`);
    
    // Remove from UI
    removeTokenFromList(token);
    
    // Alert users with positions
    alertUsersWithToken(token, 'Token removed from protocol allowlist');
});
```

### MaxInitialAllowedTokensLengthUpdated

```solidity
event MaxInitialAllowedTokensLengthUpdated(uint256 newLength);
```

**Emitted When**: Maximum tokens per fund is changed

**Parameters**:
- `newLength`: New maximum token count

### RegistryParamsUpdated

```solidity
event RegistryParamsUpdated(
    address usdcTokenAddress,
    address whackRockRewardsAddr,
    uint256 protocolCreationFeeUsdc,
    uint256 totalAumFeeBps,
    address protocolAumRecipient,
    uint256 maxAgentDepositFeeBpsAllowed
);
```

**Emitted When**: Registry parameters are updated

**Use Cases**:
- Monitor protocol governance decisions
- Update fee displays
- Alert users to parameter changes

## Fund Events

### Investment Events

#### WETHDepositedAndSharesMinted

```solidity
event WETHDepositedAndSharesMinted(
    address indexed depositor,
    address indexed receiver,
    uint256 wethDeposited,
    uint256 sharesMinted,
    uint256 navBeforeDepositWETH,
    uint256 totalSupplyBeforeDeposit,
    uint256 wethValueInUSDC
);
```

**Emitted When**: WETH is deposited and shares are minted

**Indexed Parameters**:
- `depositor`: Address that provided WETH
- `receiver`: Address that received shares

**Data Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `wethDeposited` | `uint256` | Amount of WETH invested |
| `sharesMinted` | `uint256` | Shares created for investor |
| `navBeforeDepositWETH` | `uint256` | Fund NAV before deposit |
| `totalSupplyBeforeDeposit` | `uint256` | Share supply before deposit |
| `wethValueInUSDC` | `uint256` | WETH price in USDC |

**Integration Example**:
```javascript
fund.on('WETHDepositedAndSharesMinted', (depositor, receiver, wethDeposited, sharesMinted, navBefore, supplyBefore, wethUSDC, event) => {
    const sharePrice = navBefore.gt(0) ? navBefore.mul(ethers.constants.WeiPerEther).div(supplyBefore) : ethers.constants.WeiPerEther;
    
    console.log(`Investment: ${ethers.formatEther(wethDeposited)} WETH`);
    console.log(`Shares: ${ethers.formatEther(sharesMinted)}`);
    console.log(`Share Price: ${ethers.formatEther(sharePrice)} WETH`);
    
    // Update portfolio tracking
    updateUserPortfolio(receiver, {
        fundAddress: fund.address,
        wethInvested: wethDeposited,
        sharesReceived: sharesMinted,
        sharePrice: sharePrice,
        timestamp: event.blockNumber
    });
    
    // Track fund metrics
    updateFundMetrics(fund.address, {
        newNAV: navBefore.add(wethDeposited),
        newTotalSupply: supplyBefore.add(sharesMinted),
        lastActivity: Date.now()
    });
});
```

#### BasketAssetsWithdrawn

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
);
```

**Emitted When**: Shares are burned and assets withdrawn

**Integration Example**:
```javascript
fund.on('BasketAssetsWithdrawn', (owner, receiver, sharesBurned, tokens, amounts, navBefore, supplyBefore, totalValue, wethUSDC) => {
    console.log(`Withdrawal: ${ethers.formatEther(sharesBurned)} shares`);
    console.log(`Total Value: ${ethers.formatEther(totalValue)} WETH`);
    
    // Log each token received
    tokens.forEach((token, i) => {
        if (amounts[i].gt(0)) {
            console.log(`  ${token}: ${ethers.formatUnits(amounts[i], 18)}`);
        }
    });
    
    // Update user portfolio
    recordWithdrawal(owner, {
        fundAddress: fund.address,
        sharesBurned,
        assetsReceived: tokens.map((token, i) => ({
            token,
            amount: amounts[i]
        })),
        totalValue
    });
});
```

### Portfolio Management Events

#### TargetWeightsUpdated

```solidity
event TargetWeightsUpdated(
    address indexed agent,
    address[] tokens,
    uint256[] weights,
    uint256 timestamp
);
```

**Emitted When**: Agent updates portfolio target weights

**Integration Example**:
```javascript
fund.on('TargetWeightsUpdated', (agent, tokens, weights, timestamp) => {
    console.log(`Agent ${agent} updated weights:`);
    
    tokens.forEach((token, i) => {
        console.log(`  ${token}: ${weights[i].toNumber() / 100}%`);
    });
    
    // Track strategy changes
    recordStrategyChange({
        fundAddress: fund.address,
        agent,
        newWeights: tokens.map((token, i) => ({
            token,
            weight: weights[i].toNumber()
        })),
        timestamp: timestamp.toNumber()
    });
    
    // Notify subscribers
    notifyStrategyUpdate(fund.address, {
        agent,
        newAllocations: tokens.map((token, i) => ({
            token,
            percentage: weights[i].toNumber() / 100
        }))
    });
});
```

#### RebalanceCheck

```solidity
event RebalanceCheck(
    bool needsRebalance,
    uint256 maxDeviationBPS,
    uint256 currentNAV_AA
);
```

**Emitted When**: System checks if rebalancing is needed

**Integration Example**:
```javascript
fund.on('RebalanceCheck', (needsRebalance, maxDeviation, currentNAV) => {
    const deviationPercent = maxDeviation.toNumber() / 100;
    
    console.log(`Rebalance Check: ${needsRebalance ? 'NEEDED' : 'NOT NEEDED'}`);
    console.log(`Max Deviation: ${deviationPercent}%`);
    console.log(`Current NAV: ${ethers.formatEther(currentNAV)} WETH`);
    
    if (needsRebalance) {
        // Alert monitoring systems
        alertRebalanceNeeded(fund.address, {
            deviation: deviationPercent,
            nav: currentNAV
        });
    }
    
    // Update fund health metrics
    updateFundHealth(fund.address, {
        lastRebalanceCheck: Date.now(),
        maxDeviation: deviationPercent,
        rebalanceNeeded: needsRebalance
    });
});
```

#### RebalanceCycleExecuted

```solidity
event RebalanceCycleExecuted(
    uint256 navBeforeRebalanceAA,
    uint256 navAfterRebalanceAA,
    uint256 blockTimestamp,
    uint256 wethValueInUSDC
);
```

**Emitted When**: Complete rebalancing cycle finishes

**Integration Example**:
```javascript
fund.on('RebalanceCycleExecuted', (navBefore, navAfter, timestamp, wethUSDC) => {
    const rebalanceCost = navBefore.sub(navAfter);
    const costPercentage = rebalanceCost.mul(10000).div(navBefore);
    
    console.log(`Rebalance completed:`);
    console.log(`  NAV Before: ${ethers.formatEther(navBefore)} WETH`);
    console.log(`  NAV After: ${ethers.formatEther(navAfter)} WETH`);
    console.log(`  Cost: ${ethers.formatEther(rebalanceCost)} WETH (${costPercentage.toNumber() / 100}%)`);
    
    // Track rebalancing performance
    recordRebalancePerformance(fund.address, {
        navBefore,
        navAfter,
        cost: rebalanceCost,
        costBPS: costPercentage.toNumber(),
        timestamp: timestamp.toNumber(),
        wethUSDCPrice: wethUSDC
    });
    
    // Alert if cost is high
    if (costPercentage.gt(100)) { // > 1%
        alertHighRebalanceCost(fund.address, {
            cost: rebalanceCost,
            percentage: costPercentage.toNumber() / 100
        });
    }
});
```

#### FundTokenSwapped

```solidity
event FundTokenSwapped(
    address indexed tokenFrom,
    uint256 amountFrom,
    address indexed tokenTo,
    uint256 amountTo
);
```

**Emitted When**: Individual token swaps during rebalancing

**Integration Example**:
```javascript
fund.on('FundTokenSwapped', (tokenFrom, amountFrom, tokenTo, amountTo) => {
    console.log(`Swap: ${ethers.formatEther(amountFrom)} ${tokenFrom} → ${ethers.formatEther(amountTo)} ${tokenTo}`);
    
    // Calculate swap rate
    const rate = amountTo.mul(ethers.constants.WeiPerEther).div(amountFrom);
    
    // Track individual trades
    recordTrade(fund.address, {
        fromToken: tokenFrom,
        toToken: tokenTo,
        amountIn: amountFrom,
        amountOut: amountTo,
        rate: rate,
        timestamp: Date.now()
    });
    
    // Monitor for unusual rates (potential MEV)
    checkSwapRate(tokenFrom, tokenTo, rate);
});
```

### Management Events

#### AgentUpdated

```solidity
event AgentUpdated(address indexed oldAgent, address indexed newAgent);
```

**Emitted When**: Fund owner changes the agent

**Integration Example**:
```javascript
fund.on('AgentUpdated', (oldAgent, newAgent) => {
    console.log(`Agent changed: ${oldAgent} → ${newAgent}`);
    
    // Update fund metadata
    updateFundAgent(fund.address, newAgent);
    
    // Alert stakeholders
    notifyAgentChange(fund.address, {
        oldAgent,
        newAgent,
        timestamp: Date.now()
    });
    
    // Reset agent performance metrics
    resetAgentMetrics(fund.address, newAgent);
});
```

#### AgentAumFeeCollected

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
);
```

**Emitted When**: AUM fees are collected and distributed

**Integration Example**:
```javascript
fund.on('AgentAumFeeCollected', (agentWallet, agentShares, protocolWallet, protocolShares, totalFeeValue, nav, totalShares, timestamp) => {
    const feePercentage = totalFeeValue.mul(10000).div(nav);
    
    console.log(`Fee Collection:`);
    console.log(`  Total Fee: ${ethers.formatEther(totalFeeValue)} WETH`);
    console.log(`  Fee Rate: ${feePercentage.toNumber() / 100}%`);
    console.log(`  Agent Shares: ${ethers.formatEther(agentShares)}`);
    console.log(`  Protocol Shares: ${ethers.formatEther(protocolShares)}`);
    
    // Track fee collection
    recordFeeCollection(fund.address, {
        agentWallet,
        protocolWallet,
        totalFeeValue,
        agentShares,
        protocolShares,
        navAtCollection: nav,
        timestamp: timestamp.toNumber()
    });
    
    // Update agent revenue tracking
    updateAgentRevenue(agentWallet, {
        fundAddress: fund.address,
        feeValue: totalFeeValue.mul(6000).div(10000), // 60% to agent
        shares: agentShares
    });
});
```

#### EmergencyWithdrawal

```solidity
event EmergencyWithdrawal(address indexed token, uint256 amount);
```

**Emitted When**: Fund owner executes emergency withdrawal

**Integration Example**:
```javascript
fund.on('EmergencyWithdrawal', (token, amount) => {
    console.log(`🚨 Emergency Withdrawal: ${ethers.formatEther(amount)} of ${token}`);
    
    // High priority alert
    alertEmergencyAction(fund.address, {
        type: 'EMERGENCY_WITHDRAWAL',
        token,
        amount,
        timestamp: Date.now()
    });
    
    // Flag fund for review
    flagFundForReview(fund.address, 'Emergency withdrawal executed');
    
    // Notify all stakeholders
    notifyStakeholders(fund.address, {
        subject: 'Emergency Action Taken',
        message: `Emergency withdrawal of ${ethers.formatEther(amount)} ${token} tokens executed.`
    });
});
```

## Event Aggregation Patterns

### Multi-Fund Monitoring

```javascript
class FundMonitor {
    constructor(fundAddresses) {
        this.funds = fundAddresses.map(address => 
            new ethers.Contract(address, FUND_ABI, provider)
        );
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.funds.forEach(fund => {
            // Investment tracking
            fund.on('WETHDepositedAndSharesMinted', (depositor, receiver, wethDeposited, sharesMinted) => {
                this.handleInvestment(fund.address, {
                    type: 'DEPOSIT',
                    user: receiver,
                    amount: wethDeposited,
                    shares: sharesMinted
                });
            });
            
            fund.on('BasketAssetsWithdrawn', (owner, receiver, sharesBurned, tokens, amounts) => {
                this.handleInvestment(fund.address, {
                    type: 'WITHDRAWAL',
                    user: owner,
                    shares: sharesBurned,
                    assets: tokens.map((token, i) => ({ token, amount: amounts[i] }))
                });
            });
            
            // Management tracking
            fund.on('TargetWeightsUpdated', (agent, tokens, weights, timestamp) => {
                this.handleStrategyChange(fund.address, {
                    agent,
                    weights: tokens.map((token, i) => ({ token, weight: weights[i] })),
                    timestamp: timestamp.toNumber()
                });
            });
            
            fund.on('RebalanceCycleExecuted', (navBefore, navAfter, timestamp) => {
                this.handleRebalance(fund.address, {
                    navBefore,
                    navAfter,
                    cost: navBefore.sub(navAfter),
                    timestamp: timestamp.toNumber()
                });
            });
        });
    }
    
    handleInvestment(fundAddress, data) {
        // Aggregate investment flow data
        this.updateFundMetrics(fundAddress, 'investment', data);
    }
    
    handleStrategyChange(fundAddress, data) {
        // Track strategy evolution
        this.updateFundMetrics(fundAddress, 'strategy', data);
    }
    
    handleRebalance(fundAddress, data) {
        // Monitor rebalancing efficiency
        this.updateFundMetrics(fundAddress, 'rebalance', data);
    }
}
```

### Real-Time Dashboard

```javascript
class RealtimeDashboard {
    constructor() {
        this.metrics = {
            totalTVL: ethers.BigNumber.from(0),
            dailyVolume: ethers.BigNumber.from(0),
            activeFunds: 0,
            recentActivity: []
        };
        
        this.setupRealtimeTracking();
    }
    
    setupRealtimeTracking() {
        // Track all registry events
        registry.on('WhackRockFundCreated', (fundId, fundAddress, creator, agent, name, symbol) => {
            this.metrics.activeFunds++;
            this.addActivity({
                type: 'FUND_CREATED',
                fundAddress,
                name,
                symbol,
                creator,
                timestamp: Date.now()
            });
        });
        
        // Track all fund investments
        const fundCreatedFilter = registry.filters.WhackRockFundCreated();
        registry.queryFilter(fundCreatedFilter).then(events => {
            events.forEach(event => {
                const fund = new ethers.Contract(event.args.fundAddress, FUND_ABI, provider);
                
                fund.on('WETHDepositedAndSharesMinted', (depositor, receiver, wethDeposited) => {
                    this.metrics.totalTVL = this.metrics.totalTVL.add(wethDeposited);
                    this.metrics.dailyVolume = this.metrics.dailyVolume.add(wethDeposited);
                    
                    this.addActivity({
                        type: 'INVESTMENT',
                        fundAddress: fund.address,
                        amount: wethDeposited,
                        investor: receiver,
                        timestamp: Date.now()
                    });
                });
                
                fund.on('BasketAssetsWithdrawn', (owner, receiver, sharesBurned, tokens, amounts, navBefore, supplyBefore, totalValue) => {
                    this.metrics.totalTVL = this.metrics.totalTVL.sub(totalValue);
                    this.metrics.dailyVolume = this.metrics.dailyVolume.add(totalValue);
                    
                    this.addActivity({
                        type: 'WITHDRAWAL',
                        fundAddress: fund.address,
                        value: totalValue,
                        investor: owner,
                        timestamp: Date.now()
                    });
                });
            });
        });
    }
    
    addActivity(activity) {
        this.metrics.recentActivity.unshift(activity);
        if (this.metrics.recentActivity.length > 100) {
            this.metrics.recentActivity.pop();
        }
        
        // Emit to connected clients
        this.broadcastUpdate();
    }
    
    getDashboardData() {
        return {
            tvl: ethers.formatEther(this.metrics.totalTVL),
            dailyVolume: ethers.formatEther(this.metrics.dailyVolume),
            activeFunds: this.metrics.activeFunds,
            recentActivity: this.metrics.recentActivity.slice(0, 10)
        };
    }
}
```

## Event Storage and Querying

### Database Schema

```sql
-- Registry events
CREATE TABLE fund_created_events (
    id SERIAL PRIMARY KEY,
    fund_id INTEGER NOT NULL,
    fund_address VARCHAR(42) NOT NULL,
    creator VARCHAR(42) NOT NULL,
    initial_agent VARCHAR(42) NOT NULL,
    vault_name VARCHAR(255),
    vault_symbol VARCHAR(10),
    vault_uri TEXT,
    description TEXT,
    allowed_tokens JSON,
    target_weights JSON,
    agent_fee_wallet VARCHAR(42),
    agent_fee_rate INTEGER,
    block_number INTEGER,
    transaction_hash VARCHAR(66),
    timestamp TIMESTAMP
);

-- Fund events
CREATE TABLE investment_events (
    id SERIAL PRIMARY KEY,
    fund_address VARCHAR(42) NOT NULL,
    event_type VARCHAR(20) NOT NULL, -- 'DEPOSIT' or 'WITHDRAWAL'
    user_address VARCHAR(42) NOT NULL,
    weth_amount DECIMAL(36, 18),
    shares_amount DECIMAL(36, 18),
    share_price DECIMAL(36, 18),
    block_number INTEGER,
    transaction_hash VARCHAR(66),
    timestamp TIMESTAMP
);

CREATE TABLE rebalance_events (
    id SERIAL PRIMARY KEY,
    fund_address VARCHAR(42) NOT NULL,
    nav_before DECIMAL(36, 18),
    nav_after DECIMAL(36, 18),
    cost DECIMAL(36, 18),
    cost_bps INTEGER,
    block_number INTEGER,
    transaction_hash VARCHAR(66),
    timestamp TIMESTAMP
);

CREATE TABLE fee_collection_events (
    id SERIAL PRIMARY KEY,
    fund_address VARCHAR(42) NOT NULL,
    agent_wallet VARCHAR(42),
    protocol_wallet VARCHAR(42),
    total_fee_value DECIMAL(36, 18),
    agent_shares DECIMAL(36, 18),
    protocol_shares DECIMAL(36, 18),
    nav_at_collection DECIMAL(36, 18),
    block_number INTEGER,
    transaction_hash VARCHAR(66),
    timestamp TIMESTAMP
);
```

### Event Indexer

```javascript
class EventIndexer {
    constructor(database, contracts) {
        this.db = database;
        this.registry = contracts.registry;
        this.lastProcessedBlock = 0;
    }
    
    async start() {
        // Get starting block
        this.lastProcessedBlock = await this.db.getLastProcessedBlock() || 0;
        
        // Process historical events
        await this.processHistoricalEvents();
        
        // Start real-time processing
        this.startRealtimeProcessing();
    }
    
    async processHistoricalEvents() {
        const currentBlock = await provider.getBlockNumber();
        const batchSize = 10000;
        
        for (let fromBlock = this.lastProcessedBlock; fromBlock < currentBlock; fromBlock += batchSize) {
            const toBlock = Math.min(fromBlock + batchSize - 1, currentBlock);
            
            console.log(`Processing blocks ${fromBlock} to ${toBlock}`);
            
            // Process registry events
            await this.processRegistryEvents(fromBlock, toBlock);
            
            // Process fund events
            await this.processFundEvents(fromBlock, toBlock);
            
            // Update last processed block
            await this.db.updateLastProcessedBlock(toBlock);
        }
    }
    
    async processRegistryEvents(fromBlock, toBlock) {
        const filter = this.registry.filters.WhackRockFundCreated();
        const events = await this.registry.queryFilter(filter, fromBlock, toBlock);
        
        for (const event of events) {
            await this.db.saveFundCreatedEvent({
                fundId: event.args.fundId.toNumber(),
                fundAddress: event.args.fundAddress,
                creator: event.args.creator,
                initialAgent: event.args.initialAgent,
                vaultName: event.args.vaultName,
                vaultSymbol: event.args.vaultSymbol,
                vaultURI: event.args.vaultURI,
                description: event.args.description,
                allowedTokens: event.args.allowedTokens,
                targetWeights: event.args.targetWeights.map(w => w.toNumber()),
                agentFeeWallet: event.args.agentAumFeeWallet,
                agentFeeRate: event.args.agentTotalAumFeeBps.toNumber(),
                blockNumber: event.blockNumber,
                transactionHash: event.transactionHash,
                timestamp: new Date(event.args.timestamp.toNumber() * 1000)
            });
        }
    }
    
    async processFundEvents(fromBlock, toBlock) {
        // Get all fund addresses
        const fundAddresses = await this.db.getAllFundAddresses();
        
        for (const fundAddress of fundAddresses) {
            const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
            
            // Process investment events
            const depositFilter = fund.filters.WETHDepositedAndSharesMinted();
            const depositEvents = await fund.queryFilter(depositFilter, fromBlock, toBlock);
            
            for (const event of depositEvents) {
                await this.db.saveInvestmentEvent({
                    fundAddress,
                    eventType: 'DEPOSIT',
                    userAddress: event.args.receiver,
                    wethAmount: ethers.formatEther(event.args.wethDeposited),
                    sharesAmount: ethers.formatEther(event.args.sharesMinted),
                    sharePrice: this.calculateSharePrice(event.args.navBeforeDepositWETH, event.args.totalSupplyBeforeDeposit),
                    blockNumber: event.blockNumber,
                    transactionHash: event.transactionHash,
                    timestamp: new Date((await provider.getBlock(event.blockNumber)).timestamp * 1000)
                });
            }
            
            // Process other event types...
        }
    }
}
```

## Related Documentation

- [Integration Guide](../integration/quick-start.md) - Using events in applications
- [Fund Events](../fund/events.md) - Detailed fund event documentation
- [Registry Events](../registry/events.md) - Detailed registry event documentation