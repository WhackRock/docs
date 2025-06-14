# Investing Guide

## Overview

This guide covers everything you need to know about investing in WhackRock funds, from basic deposits to advanced strategies and fund analysis.

## Basic Investment Process

### 1. Fund Discovery

```javascript
async function discoverFunds() {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, provider);
    
    const fundCount = await registry.getDeployedFundsCount();
    const funds = [];
    
    for (let i = 0; i < fundCount; i++) {
        const address = await registry.getFundAddressByIndex(i);
        const fundInfo = await getFundInfo(address);
        funds.push(fundInfo);
    }
    
    return funds;
}

async function getFundInfo(address) {
    const fund = new ethers.Contract(address, FUND_ABI, provider);
    
    return {
        address,
        name: await fund.name(),
        symbol: await fund.symbol(),
        agent: await fund.agent(),
        nav: await fund.totalNAVInAccountingAsset(),
        totalShares: await fund.totalSupply(),
        feeRate: await fund.agentAumFeeBps()
    };
}
```

### 2. Fund Analysis

```javascript
async function analyzeFund(fundAddress) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    // Basic metrics
    const nav = await fund.totalNAVInAccountingAsset();
    const totalShares = await fund.totalSupply();
    const sharePrice = totalShares > 0 ? nav * BigInt(1e18) / totalShares : 0n;
    
    // Portfolio composition
    const [currentWeights, tokenAddresses, tokenSymbols] = await fund.getCurrentCompositionBPS();
    const [targetWeights] = await fund.getTargetCompositionBPS();
    
    // Fee information
    const feeRate = await fund.agentAumFeeBps();
    const agent = await fund.agent();
    
    return {
        metrics: {
            nav: ethers.formatEther(nav),
            sharePrice: ethers.formatEther(sharePrice),
            totalShares: ethers.formatEther(totalShares),
            feeRate: feeRate / 100 // Convert to percentage
        },
        portfolio: {
            tokens: tokenAddresses.map((addr, i) => ({
                address: addr,
                symbol: tokenSymbols[i],
                currentWeight: currentWeights[i] / 100,
                targetWeight: targetWeights[i] / 100
            }))
        },
        management: {
            agent,
            feeRate: feeRate / 100
        }
    };
}
```

### 3. Investment Calculation

```javascript
async function calculateInvestment(fundAddress, wethAmount) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    const nav = await fund.totalNAVInAccountingAsset();
    const totalShares = await fund.totalSupply();
    
    let expectedShares;
    if (totalShares === 0n) {
        // First investment
        expectedShares = wethAmount;
    } else {
        // Subsequent investment
        expectedShares = (wethAmount * totalShares) / nav;
    }
    
    const sharePrice = totalShares > 0 ? nav * BigInt(1e18) / totalShares : BigInt(1e18);
    
    return {
        wethAmount: ethers.formatEther(wethAmount),
        expectedShares: ethers.formatEther(expectedShares),
        sharePrice: ethers.formatEther(sharePrice),
        ownership: totalShares > 0 ? Number(expectedShares * 10000n / (totalShares + expectedShares)) / 100 : 100
    };
}
```

## Investment Execution

### 1. Prepare WETH

```javascript
async function prepareWETH(wallet, amount) {
    const weth = new ethers.Contract(WETH_ADDRESS, [
        "function deposit() external payable",
        "function balanceOf(address) external view returns (uint256)",
        "function approve(address,uint256) external"
    ], wallet);
    
    const currentBalance = await weth.balanceOf(wallet.address);
    
    if (currentBalance < amount) {
        const needed = amount - currentBalance;
        console.log(`Converting ${ethers.formatEther(needed)} ETH to WETH...`);
        
        const depositTx = await weth.deposit({ value: needed });
        await depositTx.wait();
        console.log('✅ ETH converted to WETH');
    }
    
    return currentBalance + (amount - currentBalance);
}
```

### 2. Execute Investment

```javascript
async function investInFund(fundAddress, wethAmount, wallet) {
    try {
        // Step 1: Prepare WETH
        console.log('1. Preparing WETH...');
        await prepareWETH(wallet, wethAmount);
        
        // Step 2: Approve spending
        console.log('2. Approving WETH spending...');
        const weth = new ethers.Contract(WETH_ADDRESS, [
            "function approve(address,uint256) external"
        ], wallet);
        
        const approveTx = await weth.approve(fundAddress, wethAmount);
        await approveTx.wait();
        
        // Step 3: Calculate expected shares
        console.log('3. Calculating investment...');
        const calculation = await calculateInvestment(fundAddress, wethAmount);
        console.log('Expected shares:', calculation.expectedShares);
        console.log('Share price:', calculation.sharePrice, 'WETH');
        
        // Step 4: Execute deposit
        console.log('4. Executing deposit...');
        const fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
        const depositTx = await fund.deposit(wethAmount, wallet.address);
        
        console.log(`Transaction sent: ${depositTx.hash}`);
        const receipt = await depositTx.wait();
        
        // Step 5: Verify investment
        console.log('5. Verifying investment...');
        const actualShares = await fund.balanceOf(wallet.address);
        console.log('Actual shares received:', ethers.formatEther(actualShares));
        
        return {
            transaction: depositTx.hash,
            sharesReceived: actualShares,
            calculation
        };
        
    } catch (error) {
        console.error('Investment failed:', error.message);
        throw error;
    }
}
```

## Advanced Investment Strategies

### 1. Dollar Cost Averaging

```javascript
class DCAInvestor {
    constructor(fundAddress, totalAmount, intervals, wallet) {
        this.fundAddress = fundAddress;
        this.amountPerInterval = totalAmount / BigInt(intervals);
        this.intervals = intervals;
        this.wallet = wallet;
        this.invested = 0;
    }
    
    async executeNext() {
        if (this.invested >= this.intervals) {
            console.log('DCA complete');
            return null;
        }
        
        console.log(`DCA ${this.invested + 1}/${this.intervals}`);
        const result = await investInFund(
            this.fundAddress, 
            this.amountPerInterval, 
            this.wallet
        );
        
        this.invested++;
        return result;
    }
    
    // Schedule automatic DCA
    startAutomatic(intervalDays = 7) {
        const intervalMs = intervalDays * 24 * 60 * 60 * 1000;
        
        const timer = setInterval(async () => {
            try {
                const result = await this.executeNext();
                if (!result) {
                    clearInterval(timer);
                }
            } catch (error) {
                console.error('DCA failed:', error);
            }
        }, intervalMs);
        
        return timer;
    }
}

// Usage
const dca = new DCAInvestor(
    fundAddress,
    ethers.parseEther('1.0'),  // 1 WETH total
    10,                         // 10 intervals
    wallet
);

const timer = dca.startAutomatic(7); // Weekly
```

### 2. Portfolio Allocation

```javascript
async function allocatePortfolio(allocations, totalAmount, wallet) {
    const results = [];
    
    for (const allocation of allocations) {
        const amount = (totalAmount * BigInt(allocation.percentage)) / 100n;
        
        console.log(`Investing ${ethers.formatEther(amount)} WETH in ${allocation.fundName}`);
        
        const result = await investInFund(allocation.fundAddress, amount, wallet);
        results.push({
            fund: allocation.fundName,
            amount: ethers.formatEther(amount),
            shares: ethers.formatEther(result.sharesReceived),
            transaction: result.transaction
        });
    }
    
    return results;
}

// Example allocation
const myPortfolio = [
    { fundAddress: "0x...", fundName: "AI Growth", percentage: 60 },
    { fundAddress: "0x...", fundName: "Stable Index", percentage: 30 },
    { fundAddress: "0x...", fundName: "DeFi Momentum", percentage: 10 }
];

const portfolioResults = await allocatePortfolio(
    myPortfolio,
    ethers.parseEther('5.0'),  // 5 WETH total
    wallet
);
```

### 3. Conditional Investment

```javascript
class ConditionalInvestor {
    constructor(fundAddress, wallet) {
        this.fundAddress = fundAddress;
        this.wallet = wallet;
        this.fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
    }
    
    async investOnPriceDrop(targetPrice, amount) {
        const currentPrice = await this.getCurrentSharePrice();
        
        if (currentPrice <= targetPrice) {
            console.log(`Price target hit: ${ethers.formatEther(currentPrice)} <= ${ethers.formatEther(targetPrice)}`);
            return await investInFund(this.fundAddress, amount, this.wallet);
        }
        
        console.log(`Waiting for price drop: ${ethers.formatEther(currentPrice)} > ${ethers.formatEther(targetPrice)}`);
        return null;
    }
    
    async investOnGoodPerformance(periodDays, minReturn, amount) {
        const performance = await this.calculatePerformance(periodDays);
        
        if (performance >= minReturn) {
            console.log(`Performance target met: ${performance}% >= ${minReturn}%`);
            return await investInFund(this.fundAddress, amount, this.wallet);
        }
        
        return null;
    }
    
    async getCurrentSharePrice() {
        const nav = await this.fund.totalNAVInAccountingAsset();
        const totalShares = await this.fund.totalSupply();
        return totalShares > 0 ? nav * BigInt(1e18) / totalShares : BigInt(1e18);
    }
    
    async calculatePerformance(days) {
        // Implementation would fetch historical data
        // This is a simplified example
        const blocksPerDay = 7200; // Approximate for Base
        const targetBlock = await provider.getBlockNumber() - (days * blocksPerDay);
        
        // Would need to implement historical price tracking
        return 0; // Placeholder
    }
}
```

## Investment Monitoring

### 1. Position Tracking

```javascript
class PositionTracker {
    constructor(wallet, fundAddresses) {
        this.wallet = wallet;
        this.fundAddresses = fundAddresses;
    }
    
    async getPositions() {
        const positions = [];
        
        for (const address of this.fundAddresses) {
            const fund = new ethers.Contract(address, FUND_ABI, provider);
            
            const shares = await fund.balanceOf(this.wallet.address);
            if (shares > 0) {
                const nav = await fund.totalNAVInAccountingAsset();
                const totalShares = await fund.totalSupply();
                const sharePrice = nav * BigInt(1e18) / totalShares;
                const value = shares * sharePrice / BigInt(1e18);
                
                positions.push({
                    fundAddress: address,
                    fundName: await fund.name(),
                    shares: ethers.formatEther(shares),
                    sharePrice: ethers.formatEther(sharePrice),
                    value: ethers.formatEther(value),
                    percentage: Number(shares * 10000n / totalShares) / 100
                });
            }
        }
        
        return positions;
    }
    
    async getTotalPortfolioValue() {
        const positions = await this.getPositions();
        return positions.reduce((total, pos) => total + parseFloat(pos.value), 0);
    }
    
    async printPortfolio() {
        const positions = await this.getPositions();
        const total = await this.getTotalPortfolioValue();
        
        console.log('\n📊 Portfolio Summary');
        console.log('==================');
        
        for (const pos of positions) {
            const allocation = (parseFloat(pos.value) / total * 100).toFixed(2);
            console.log(`${pos.fundName}: ${pos.value} WETH (${allocation}%)`);
            console.log(`  Shares: ${pos.shares} @ ${pos.sharePrice} WETH`);
        }
        
        console.log(`\nTotal Value: ${total.toFixed(4)} WETH`);
    }
}
```

### 2. Performance Monitoring

```javascript
async function monitorPerformance(fundAddress, wallet) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    // Listen for key events
    fund.on('WETHDepositedAndSharesMinted', (depositor, receiver, wethDeposited, sharesMinted) => {
        if (receiver.toLowerCase() === wallet.address.toLowerCase()) {
            console.log(`✅ Investment confirmed: ${ethers.formatEther(sharesMinted)} shares for ${ethers.formatEther(wethDeposited)} WETH`);
        }
    });
    
    fund.on('RebalanceCycleExecuted', (navBefore, navAfter, timestamp) => {
        const impact = ((navAfter - navBefore) * 10000n) / navBefore;
        console.log(`🔄 Rebalance completed. NAV impact: ${Number(impact) / 100}%`);
    });
    
    fund.on('AgentAumFeeCollected', (agentWallet, agentShares, protocolWallet, protocolShares, totalFeeValue) => {
        console.log(`💰 Fees collected: ${ethers.formatEther(totalFeeValue)} WETH`);
    });
    
    console.log('👂 Monitoring fund events...');
}
```

## Risk Management

### 1. Investment Limits

```javascript
class RiskManager {
    constructor(maxPositionSize, maxFundAllocation) {
        this.maxPositionSize = maxPositionSize;      // Max WETH per fund
        this.maxFundAllocation = maxFundAllocation;  // Max % of portfolio per fund
    }
    
    async validateInvestment(fundAddress, amount, wallet) {
        // Check position size limit
        if (amount > this.maxPositionSize) {
            throw new Error(`Investment ${ethers.formatEther(amount)} exceeds position limit ${ethers.formatEther(this.maxPositionSize)}`);
        }
        
        // Check portfolio allocation
        const tracker = new PositionTracker(wallet, [fundAddress]);
        const currentValue = await tracker.getTotalPortfolioValue();
        const newValue = currentValue + parseFloat(ethers.formatEther(amount));
        
        if (newValue / currentValue > this.maxFundAllocation / 100) {
            throw new Error('Investment would exceed fund allocation limit');
        }
        
        return true;
    }
}

const riskManager = new RiskManager(
    ethers.parseEther('1.0'),  // Max 1 WETH per fund
    25  // Max 25% of portfolio per fund
);
```

### 2. Fund Health Checks

```javascript
async function checkFundHealth(fundAddress) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    const checks = {
        hasAssets: false,
        agentActive: false,
        recentActivity: false,
        portfolioBalanced: false
    };
    
    // Check if fund has assets
    const nav = await fund.totalNAVInAccountingAsset();
    checks.hasAssets = nav > 0;
    
    // Check if agent exists and is not zero address
    const agent = await fund.agent();
    checks.agentActive = agent !== ethers.ZeroAddress;
    
    // Check portfolio balance (simplified)
    const [currentWeights, tokenAddresses] = await fund.getCurrentCompositionBPS();
    const totalDeviation = currentWeights.reduce((acc, weight, i) => {
        return acc + Math.abs(weight - 10000 / tokenAddresses.length);
    }, 0);
    checks.portfolioBalanced = totalDeviation < 1000; // Less than 10% total deviation
    
    // TODO: Check recent activity from events
    checks.recentActivity = true; // Placeholder
    
    const healthScore = Object.values(checks).filter(Boolean).length / Object.keys(checks).length;
    
    return {
        checks,
        healthScore,
        recommendation: healthScore >= 0.75 ? 'SAFE' : healthScore >= 0.5 ? 'CAUTION' : 'RISKY'
    };
}
```

## Withdrawal Strategies

### 1. Partial Withdrawal

```javascript
async function partialWithdraw(fundAddress, percentage, wallet) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
    
    const totalShares = await fund.balanceOf(wallet.address);
    const sharesToWithdraw = (totalShares * BigInt(percentage)) / 100n;
    
    console.log(`Withdrawing ${percentage}% (${ethers.formatEther(sharesToWithdraw)} shares)`);
    
    const tx = await fund.withdraw(sharesToWithdraw, wallet.address, wallet.address);
    const receipt = await tx.wait();
    
    console.log(`Withdrawal complete: ${tx.hash}`);
    return receipt;
}
```

### 2. Rebalancing Exit

```javascript
async function exitAndRebalance(currentFunds, targetAllocation, wallet) {
    console.log('Executing portfolio rebalancing...');
    
    // Step 1: Withdraw from overweight positions
    for (const fund of currentFunds) {
        const currentAllocation = fund.currentAllocation;
        const targetAllocation = fund.targetAllocation;
        
        if (currentAllocation > targetAllocation) {
            const excessPercentage = currentAllocation - targetAllocation;
            await partialWithdraw(fund.address, excessPercentage, wallet);
        }
    }
    
    // Step 2: Calculate available WETH
    const weth = new ethers.Contract(WETH_ADDRESS, ['function balanceOf(address) view returns (uint256)'], wallet);
    const availableWETH = await weth.balanceOf(wallet.address);
    
    // Step 3: Invest in underweight positions
    for (const fund of currentFunds) {
        const currentAllocation = fund.currentAllocation;
        const targetAllocation = fund.targetAllocation;
        
        if (currentAllocation < targetAllocation) {
            const neededPercentage = targetAllocation - currentAllocation;
            const investAmount = (availableWETH * BigInt(neededPercentage)) / 100n;
            
            if (investAmount > 0) {
                await investInFund(fund.address, investAmount, wallet);
            }
        }
    }
}
```

## Integration Examples

### React Component

```jsx
import { useState, useEffect } from 'react';
import { ethers } from 'ethers';

function FundInvestment({ fundAddress, wallet }) {
    const [fundInfo, setFundInfo] = useState(null);
    const [investAmount, setInvestAmount] = useState('');
    const [loading, setLoading] = useState(false);
    
    useEffect(() => {
        loadFundInfo();
    }, [fundAddress]);
    
    async function loadFundInfo() {
        const info = await analyzeFund(fundAddress);
        setFundInfo(info);
    }
    
    async function handleInvest() {
        setLoading(true);
        try {
            const amount = ethers.parseEther(investAmount);
            await investInFund(fundAddress, amount, wallet);
            await loadFundInfo(); // Refresh
        } catch (error) {
            alert('Investment failed: ' + error.message);
        }
        setLoading(false);
    }
    
    if (!fundInfo) return <div>Loading...</div>;
    
    return (
        <div>
            <h3>{fundInfo.name}</h3>
            <p>NAV: {fundInfo.metrics.nav} WETH</p>
            <p>Share Price: {fundInfo.metrics.sharePrice} WETH</p>
            <p>Fee Rate: {fundInfo.metrics.feeRate}%</p>
            
            <input
                type="number"
                value={investAmount}
                onChange={(e) => setInvestAmount(e.target.value)}
                placeholder="WETH amount"
            />
            <button onClick={handleInvest} disabled={loading}>
                {loading ? 'Investing...' : 'Invest'}
            </button>
        </div>
    );
}
```

## Related Documentation

- [Creating Funds](creating-funds.md) - How to create new funds
- [Agent Operations](agent-operations.md) - Fund management perspective
- [Portfolio Management](../fund/portfolio-mgmt.md) - Understanding fund operations