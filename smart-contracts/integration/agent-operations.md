# Agent Operations Guide

## Overview

This guide covers the complete workflow for AI agents and fund managers operating WhackRock funds. Agents are responsible for portfolio strategy, rebalancing, and fee collection.

## Agent Responsibilities

### Core Functions
- **Portfolio Management**: Set and adjust target weights
- **Rebalancing**: Trigger rebalancing when needed
- **Fee Collection**: Collect management fees
- **Strategy Implementation**: Execute investment strategies

### Access Control
- Only the current agent can perform management functions
- Fund owner can change the agent
- Agent permissions are immediately active

## Portfolio Management

### 1. Setting Target Weights

```javascript
class FundAgent {
    constructor(fundAddress, wallet) {
        this.fundAddress = fundAddress;
        this.wallet = wallet;
        this.fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
    }
    
    async setWeights(newWeights, rebalanceImmediately = true) {
        console.log('Setting new target weights...');
        
        // Validate weights
        const totalWeight = newWeights.reduce((a, b) => a + b, 0);
        if (totalWeight !== 10000) {
            throw new Error(`Weights must sum to 10000, got ${totalWeight}`);
        }
        
        try {
            let tx;
            if (rebalanceImmediately) {
                tx = await this.fund.setTargetWeightsAndRebalanceIfNeeded(newWeights);
                console.log('Weights set with immediate rebalancing');
            } else {
                tx = await this.fund.setTargetWeights(newWeights);
                console.log('Weights set without rebalancing');
            }
            
            const receipt = await tx.wait();
            console.log(`✅ Transaction confirmed: ${tx.hash}`);
            
            return receipt;
        } catch (error) {
            console.error('Failed to set weights:', error.message);
            throw error;
        }
    }
    
    async getCurrentWeights() {
        const [currentWeights, tokenAddresses, tokenSymbols] = await this.fund.getCurrentCompositionBPS();
        
        return tokenAddresses.map((addr, i) => ({
            token: addr,
            symbol: tokenSymbols[i],
            weight: currentWeights[i]
        }));
    }
    
    async getTargetWeights() {
        const [targetWeights, tokenAddresses, tokenSymbols] = await this.fund.getTargetCompositionBPS();
        
        return tokenAddresses.map((addr, i) => ({
            token: addr,
            symbol: tokenSymbols[i],
            weight: targetWeights[i]
        }));
    }
}
```

### 2. Strategy Implementation

```javascript
class StrategyAgent extends FundAgent {
    constructor(fundAddress, wallet, strategy) {
        super(fundAddress, wallet);
        this.strategy = strategy;
    }
    
    async executeStrategy() {
        console.log(`Executing ${this.strategy.name} strategy...`);
        
        const marketData = await this.getMarketData();
        const newWeights = await this.strategy.calculateWeights(marketData);
        
        await this.setWeights(newWeights);
        
        console.log('Strategy execution complete');
    }
    
    async getMarketData() {
        // Implementation would fetch real market data
        const tokens = await this.getCurrentWeights();
        
        return tokens.map(token => ({
            address: token.token,
            symbol: token.symbol,
            price: Math.random() * 1000, // Mock price
            volume: Math.random() * 1000000,
            momentum: Math.random() * 0.2 - 0.1 // -10% to +10%
        }));
    }
}

// Example strategy implementations
const momentumStrategy = {
    name: 'Momentum',
    calculateWeights: async (marketData) => {
        // Allocate more to assets with positive momentum
        const totalMomentum = marketData.reduce((sum, token) => 
            sum + Math.max(0, token.momentum), 0
        );
        
        return marketData.map(token => {
            const positiveMomentum = Math.max(0, token.momentum);
            return Math.floor((positiveMomentum / totalMomentum) * 10000);
        });
    }
};

const equalWeightStrategy = {
    name: 'Equal Weight',
    calculateWeights: async (marketData) => {
        const weight = Math.floor(10000 / marketData.length);
        const weights = new Array(marketData.length).fill(weight);
        
        // Adjust last weight for rounding
        weights[weights.length - 1] = 10000 - weight * (marketData.length - 1);
        
        return weights;
    }
};
```

### 3. Rebalancing Management

```javascript
class RebalancingAgent extends FundAgent {
    async checkRebalanceStatus() {
        // This calls the internal view function through a static call
        try {
            const result = await this.fund.staticCall._isRebalanceNeeded();
            return {
                needsRebalance: result[0],
                maxDeviationBPS: result[1]
            };
        } catch {
            // Fallback: calculate manually
            return await this.calculateRebalanceNeed();
        }
    }
    
    async calculateRebalanceNeed() {
        const current = await this.getCurrentWeights();
        const target = await this.getTargetWeights();
        
        let maxDeviation = 0;
        
        for (let i = 0; i < current.length; i++) {
            const deviation = Math.abs(current[i].weight - target[i].weight);
            maxDeviation = Math.max(maxDeviation, deviation);
        }
        
        return {
            needsRebalance: maxDeviation > 100, // 1% threshold
            maxDeviationBPS: maxDeviation
        };
    }
    
    async triggerRebalanceIfNeeded() {
        const status = await this.checkRebalanceStatus();
        
        if (status.needsRebalance) {
            console.log(`Triggering rebalance (${status.maxDeviationBPS / 100}% deviation)`);
            
            const tx = await this.fund.triggerRebalance();
            const receipt = await tx.wait();
            
            console.log(`✅ Rebalance completed: ${tx.hash}`);
            return receipt;
        } else {
            console.log(`No rebalancing needed (${status.maxDeviationBPS / 100}% deviation)`);
            return null;
        }
    }
    
    async scheduleRebalancing(checkIntervalMinutes = 60) {
        console.log(`Starting automatic rebalancing every ${checkIntervalMinutes} minutes`);
        
        const interval = setInterval(async () => {
            try {
                await this.triggerRebalanceIfNeeded();
            } catch (error) {
                console.error('Rebalancing failed:', error.message);
            }
        }, checkIntervalMinutes * 60 * 1000);
        
        return interval;
    }
}
```

## Fee Management

### 1. Fee Collection

```javascript
class FeeAgent extends FundAgent {
    async collectFees() {
        console.log('Collecting management fees...');
        
        try {
            const tx = await this.fund.collectAgentManagementFee();
            const receipt = await tx.wait();
            
            // Extract fee information from events
            const feeEvent = receipt.logs.find(log => {
                try {
                    const parsed = this.fund.interface.parseLog(log);
                    return parsed.name === 'AgentAumFeeCollected';
                } catch {
                    return false;
                }
            });
            
            if (feeEvent) {
                const parsed = this.fund.interface.parseLog(feeEvent);
                const agentShares = parsed.args.agentSharesMinted;
                const totalFeeValue = parsed.args.totalFeeValueInAccountingAsset;
                
                console.log(`✅ Fees collected:`);
                console.log(`  Agent shares: ${ethers.formatEther(agentShares)}`);
                console.log(`  Fee value: ${ethers.formatEther(totalFeeValue)} WETH`);
            }
            
            return receipt;
        } catch (error) {
            console.error('Fee collection failed:', error.message);
            throw error;
        }
    }
    
    async calculateAccruedFees() {
        const nav = await this.fund.totalNAVInAccountingAsset();
        const feeRate = await this.fund.agentAumFeeBps();
        const lastCollection = await this.fund.lastAgentAumFeeCollectionTimestamp();
        
        const timeElapsed = Math.floor(Date.now() / 1000) - Number(lastCollection);
        const annualSeconds = 365 * 24 * 3600;
        
        const totalFeeValue = (nav * feeRate * BigInt(timeElapsed)) / (10000n * BigInt(annualSeconds));
        
        return {
            totalFeeValue: ethers.formatEther(totalFeeValue),
            timeElapsedDays: timeElapsed / (24 * 3600),
            annualRate: Number(feeRate) / 100
        };
    }
    
    async shouldCollectFees(minValueWETH = 0.001) {
        const accrued = await this.calculateAccruedFees();
        const minValue = ethers.parseEther(minValueWETH.toString());
        
        return parseFloat(accrued.totalFeeValue) >= parseFloat(ethers.formatEther(minValue));
    }
    
    async scheduleFeCollection(checkIntervalHours = 24) {
        console.log(`Starting automatic fee collection every ${checkIntervalHours} hours`);
        
        const interval = setInterval(async () => {
            try {
                const shouldCollect = await this.shouldCollectFees();
                if (shouldCollect) {
                    await this.collectFees();
                }
            } catch (error) {
                console.error('Fee collection failed:', error.message);
            }
        }, checkIntervalHours * 60 * 60 * 1000);
        
        return interval;
    }
}
```

### 2. Fee Optimization

```javascript
class OptimizedFeeAgent extends FeeAgent {
    async collectFeesOptimally(gasThresholdGwei = 20) {
        // Check gas price
        const feeData = await provider.getFeeData();
        const gasPriceGwei = Number(feeData.gasPrice) / 1e9;
        
        if (gasPriceGwei > gasThresholdGwei) {
            console.log(`Gas too expensive: ${gasPriceGwei} gwei > ${gasThresholdGwei} gwei`);
            return null;
        }
        
        // Check if fees are worth collecting
        const accrued = await this.calculateAccruedFees();
        const estimatedGasCost = await this.estimateCollectionCost();
        
        if (parseFloat(accrued.totalFeeValue) < estimatedGasCost) {
            console.log('Fees not worth collecting yet (gas cost > fee value)');
            return null;
        }
        
        return await this.collectFees();
    }
    
    async estimateCollectionCost() {
        try {
            const gasEstimate = await this.fund.collectAgentManagementFee.estimateGas();
            const feeData = await provider.getFeeData();
            const costWei = gasEstimate * feeData.gasPrice;
            
            return parseFloat(ethers.formatEther(costWei));
        } catch {
            return 0.001; // Default estimate
        }
    }
}
```

## Complete Agent Implementation

### 1. Full Agent Class

```javascript
class CompleteAgent {
    constructor(fundAddress, wallet, config = {}) {
        this.fundAddress = fundAddress;
        this.wallet = wallet;
        this.fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
        
        // Configuration
        this.config = {
            rebalanceThresholdBPS: config.rebalanceThresholdBPS || 100,
            feeCollectionMinWETH: config.feeCollectionMinWETH || 0.001,
            maxGasPriceGwei: config.maxGasPriceGwei || 50,
            strategyCheckInterval: config.strategyCheckInterval || 3600000, // 1 hour
            ...config
        };
        
        this.isRunning = false;
        this.intervals = [];
    }
    
    async start() {
        if (this.isRunning) {
            console.log('Agent already running');
            return;
        }
        
        console.log('🤖 Starting WHACKROCK Agent');
        
        // Verify agent permissions
        const currentAgent = await this.fund.agent();
        if (currentAgent.toLowerCase() !== this.wallet.address.toLowerCase()) {
            throw new Error('Wallet is not the fund agent');
        }
        
        this.isRunning = true;
        
        // Start monitoring loops
        this.intervals.push(
            setInterval(() => this.checkAndRebalance(), this.config.strategyCheckInterval)
        );
        
        this.intervals.push(
            setInterval(() => this.checkAndCollectFees(), 60 * 60 * 1000) // Every hour
        );
        
        console.log('✅ Agent started successfully');
    }
    
    async stop() {
        this.isRunning = false;
        this.intervals.forEach(clearInterval);
        this.intervals = [];
        console.log('🛑 Agent stopped');
    }
    
    async checkAndRebalance() {
        if (!this.isRunning) return;
        
        try {
            console.log('Checking rebalancing need...');
            
            const current = await this.getCurrentWeights();
            const target = await this.getTargetWeights();
            
            let maxDeviation = 0;
            for (let i = 0; i < current.length; i++) {
                const deviation = Math.abs(current[i].weight - target[i].weight);
                maxDeviation = Math.max(maxDeviation, deviation);
            }
            
            if (maxDeviation > this.config.rebalanceThresholdBPS) {
                console.log(`Rebalancing needed: ${maxDeviation / 100}% deviation`);
                await this.fund.triggerRebalance();
                console.log('✅ Rebalancing completed');
            }
        } catch (error) {
            console.error('Rebalancing check failed:', error.message);
        }
    }
    
    async checkAndCollectFees() {
        if (!this.isRunning) return;
        
        try {
            console.log('Checking fee collection...');
            
            // Check gas price
            const feeData = await provider.getFeeData();
            const gasPriceGwei = Number(feeData.gasPrice) / 1e9;
            
            if (gasPriceGwei > this.config.maxGasPriceGwei) {
                console.log(`Gas too expensive: ${gasPriceGwei} gwei`);
                return;
            }
            
            // Check accrued fees
            const nav = await this.fund.totalNAVInAccountingAsset();
            const feeRate = await this.fund.agentAumFeeBps();
            const lastCollection = await this.fund.lastAgentAumFeeCollectionTimestamp();
            
            const timeElapsed = Math.floor(Date.now() / 1000) - Number(lastCollection);
            const totalFeeValue = (nav * feeRate * BigInt(timeElapsed)) / (10000n * BigInt(365 * 24 * 3600));
            
            const minFeeValue = ethers.parseEther(this.config.feeCollectionMinWETH.toString());
            
            if (totalFeeValue >= minFeeValue) {
                console.log(`Collecting fees: ${ethers.formatEther(totalFeeValue)} WETH`);
                await this.fund.collectAgentManagementFee();
                console.log('✅ Fees collected');
            }
        } catch (error) {
            console.error('Fee collection check failed:', error.message);
        }
    }
    
    async executeStrategy(strategy) {
        console.log(`Executing ${strategy.name} strategy...`);
        
        try {
            const marketData = await this.getMarketData();
            const newWeights = await strategy.calculateWeights(marketData);
            
            await this.fund.setTargetWeightsAndRebalanceIfNeeded(newWeights);
            
            console.log('✅ Strategy executed successfully');
        } catch (error) {
            console.error('Strategy execution failed:', error.message);
            throw error;
        }
    }
    
    async getStatus() {
        const nav = await this.fund.totalNAVInAccountingAsset();
        const totalShares = await this.fund.totalSupply();
        const agent = await this.fund.agent();
        const feeRate = await this.fund.agentAumFeeBps();
        
        const current = await this.getCurrentWeights();
        const target = await this.getTargetWeights();
        
        return {
            fundAddress: this.fundAddress,
            nav: ethers.formatEther(nav),
            totalShares: ethers.formatEther(totalShares),
            sharePrice: totalShares > 0 ? ethers.formatEther(nav * BigInt(1e18) / totalShares) : '0',
            agent,
            feeRate: Number(feeRate) / 100,
            isRunning: this.isRunning,
            portfolio: {
                current,
                target
            }
        };
    }
    
    // Helper methods
    async getCurrentWeights() {
        const [weights, addresses, symbols] = await this.fund.getCurrentCompositionBPS();
        return addresses.map((addr, i) => ({
            token: addr,
            symbol: symbols[i],
            weight: Number(weights[i])
        }));
    }
    
    async getTargetWeights() {
        const [weights, addresses, symbols] = await this.fund.getTargetCompositionBPS();
        return addresses.map((addr, i) => ({
            token: addr,
            symbol: symbols[i],
            weight: Number(weights[i])
        }));
    }
    
    async getMarketData() {
        // Mock implementation - replace with real market data
        const tokens = await this.getCurrentWeights();
        return tokens.map(token => ({
            address: token.token,
            symbol: token.symbol,
            price: Math.random() * 1000,
            volume: Math.random() * 1000000,
            momentum: Math.random() * 0.2 - 0.1
        }));
    }
}
```

### 2. Usage Examples

```javascript
// Start a basic agent
const agent = new CompleteAgent(fundAddress, wallet, {
    rebalanceThresholdBPS: 150,  // 1.5% threshold
    feeCollectionMinWETH: 0.01,  // Collect when >0.01 WETH
    maxGasPriceGwei: 30          // Don't transact above 30 gwei
});

await agent.start();

// Execute a momentum strategy
await agent.executeStrategy(momentumStrategy);

// Check agent status
const status = await agent.getStatus();
console.log('Agent Status:', status);

// Stop agent
await agent.stop();
```

## Advanced Agent Features

### 1. Multi-Fund Agent

```javascript
class MultiFundAgent {
    constructor(fundAddresses, wallet) {
        this.agents = fundAddresses.map(address => 
            new CompleteAgent(address, wallet)
        );
    }
    
    async startAll() {
        for (const agent of this.agents) {
            await agent.start();
        }
    }
    
    async executeStrategyOnAll(strategy) {
        const results = [];
        for (const agent of this.agents) {
            try {
                await agent.executeStrategy(strategy);
                results.push({ fund: agent.fundAddress, success: true });
            } catch (error) {
                results.push({ fund: agent.fundAddress, success: false, error: error.message });
            }
        }
        return results;
    }
    
    async getAllStatuses() {
        const statuses = [];
        for (const agent of this.agents) {
            const status = await agent.getStatus();
            statuses.push(status);
        }
        return statuses;
    }
}
```

### 2. Strategy Backtesting

```javascript
class StrategyBacktester {
    constructor(strategy, historicalData) {
        this.strategy = strategy;
        this.historicalData = historicalData;
    }
    
    async backtest(startDate, endDate) {
        const results = [];
        let currentWeights = null;
        
        for (const dataPoint of this.historicalData) {
            if (dataPoint.date >= startDate && dataPoint.date <= endDate) {
                const newWeights = await this.strategy.calculateWeights(dataPoint.marketData);
                
                if (currentWeights) {
                    const returns = this.calculateReturns(currentWeights, dataPoint.returns);
                    results.push({
                        date: dataPoint.date,
                        weights: newWeights,
                        returns: returns
                    });
                }
                
                currentWeights = newWeights;
            }
        }
        
        return this.analyzeResults(results);
    }
    
    calculateReturns(weights, assetReturns) {
        return weights.reduce((totalReturn, weight, i) => {
            return totalReturn + (weight / 10000) * assetReturns[i];
        }, 0);
    }
    
    analyzeResults(results) {
        const totalReturn = results.reduce((acc, r) => acc * (1 + r.returns), 1) - 1;
        const volatility = this.calculateVolatility(results.map(r => r.returns));
        const sharpeRatio = totalReturn / volatility;
        
        return {
            totalReturn,
            volatility,
            sharpeRatio,
            maxDrawdown: this.calculateMaxDrawdown(results)
        };
    }
    
    calculateVolatility(returns) {
        const mean = returns.reduce((a, b) => a + b) / returns.length;
        const variance = returns.reduce((acc, r) => acc + Math.pow(r - mean, 2), 0) / returns.length;
        return Math.sqrt(variance);
    }
    
    calculateMaxDrawdown(results) {
        let peak = 1;
        let maxDrawdown = 0;
        let cumulative = 1;
        
        for (const result of results) {
            cumulative *= (1 + result.returns);
            if (cumulative > peak) {
                peak = cumulative;
            }
            const drawdown = (peak - cumulative) / peak;
            maxDrawdown = Math.max(maxDrawdown, drawdown);
        }
        
        return maxDrawdown;
    }
}
```

## Related Documentation

- [Portfolio Management](../fund/portfolio-mgmt.md) - Detailed rebalancing mechanics
- [Fee Collection](../fund/fee-collection.md) - Fee calculation and distribution
- [Creating Funds](creating-funds.md) - Setting up funds for agent management