# Economic Security

## Overview

This document analyzes potential economic attack vectors against the WHACKROCK protocol and the defensive mechanisms implemented to prevent them.

## Attack Vector Analysis

### 1. Share Price Manipulation

#### Attack Description
An attacker could attempt to manipulate the share price of a fund by:
1. Creating a fund with minimal initial deposit
2. Depositing dust amounts to manipulate share price
3. Large investors then get unfavorable rates

#### Attack Example
```
1. Attacker creates fund with 0.01 WETH
2. Initial shares: 0.01 WETH = 0.01 shares (1:1 ratio)
3. Attacker sends 10 WETH directly to contract (not via deposit)
4. NAV becomes 10.01 WETH, shares remain 0.01
5. Share price: 10.01 / 0.01 = 1001 WETH per share
6. Next investor pays massive premium
```

#### Defense Mechanisms

**Minimum Initial Deposit**:
```solidity
uint256 public constant MINIMUM_INITIAL_DEPOSIT = 0.01 ether;

// In deposit function
if (totalSupplyBeforeDeposit == 0) {
    if (amountWETHToDeposit < MINIMUM_INITIAL_DEPOSIT) revert E2();
    sharesMinted = amountWETHToDeposit;
    
    // Additional safeguard
    if (sharesMinted < MINIMUM_SHARES_LIQUIDITY && amountWETHToDeposit > 0) {
        sharesMinted = MINIMUM_SHARES_LIQUIDITY;
    }
}
```

**Minimum Liquidity Shares**:
```solidity
uint256 private constant MINIMUM_SHARES_LIQUIDITY = 1000;
```

This ensures a minimum number of shares always exist, making manipulation prohibitively expensive.

**Economic Analysis**:
- Cost to manipulate: >0.01 ETH minimum
- Benefit to attacker: Reduced as fund grows
- Break-even point: Very small for any meaningful fund

### 2. Sandwich Attacks on Rebalancing

#### Attack Description
MEV bots could sandwich fund rebalancing transactions:
1. Detect pending rebalance transaction
2. Front-run with buy orders (pump price)
3. Let fund rebalance at inflated price  
4. Back-run with sell orders (dump price)
5. Extract value from fund

#### Defense Mechanisms

**Slippage Protection**:
```solidity
uint256 public constant DEFAULT_SLIPPAGE_BPS = 50; // 0.5%

// In swap execution
uint256 minAmountOut = expectedAmount * (TOTAL_WEIGHT_BASIS_POINTS - DEFAULT_SLIPPAGE_BPS) / TOTAL_WEIGHT_BASIS_POINTS;

router.swapExactTokensForTokens(
    amountIn,
    minAmountOut,
    path,
    address(this),
    deadline
);
```

**Deadline Protection**:
```solidity
uint256 public constant SWAP_DEADLINE_OFFSET = 15 minutes;

// Transaction must execute within time window
uint256 deadline = block.timestamp + SWAP_DEADLINE_OFFSET;
```

**Rebalancing Threshold**:
```solidity
uint256 public constant REBALANCE_DEVIATION_THRESHOLD_BPS = 100; // 1%
```

Only rebalances when deviation exceeds 1%, reducing unnecessary trades.

### 3. Fee Extraction Attacks

#### Attack Description
Malicious agents could:
1. Set high fee rates (within protocol limits)
2. Collect fees frequently
3. Extract maximum value from fund

#### Defense Mechanisms

**Registry Fee Limits**:
```solidity
// In fund creation
require(_agentSetTotalAumFeeBps <= totalAumFeeBpsForFunds, "Registry: Fund AUM fee exceeds protocol max");
```

**Time-Based Fee Accrual**:
```solidity
// Fees accrue over time, cannot be gamed
uint256 timeElapsed = block.timestamp - lastAgentAumFeeCollectionTimestamp;
uint256 totalFeeValueInAA = (navAtFeeCalc * agentAumFeeBps * timeElapsed) / (TOTAL_WEIGHT_BASIS_POINTS * 365 days);
```

**Non-Dilutive Fee Collection**:
```solidity
// Minting shares maintains NAV per share
uint256 totalSharesToMintForFee = (totalFeeValueInAA * sharesAtFeeCalc) / navAtFeeCalc;
```

**Transparent Fee Structure**:
- Immutable fee rate set at creation
- Public fee collection function
- All fees tracked via events

### 4. Flash Loan Attacks

#### Attack Description
Using flash loans to:
1. Borrow large amounts
2. Manipulate fund prices during rebalancing
3. Extract value before repaying loan

#### Defense Mechanisms

**No Flash Loan Integration**:
- Funds don't implement flash loan interfaces
- No lending/borrowing functionality
- Simple investment-only operations

**Atomic Operations**:
```solidity
// Rebalancing is atomic - no intermediate state exposure
function _rebalance() internal {
    // All swaps happen in single transaction
    // No external calls between critical operations
}
```

**Oracle Independence**:
- Uses DEX router quotes (no external oracles)
- Prices determined by actual market liquidity
- Harder to manipulate than centralized oracles

### 5. Governance Attacks

#### Attack Description
Attacking governance mechanisms:
1. Acquire governance tokens
2. Propose malicious changes
3. Extract value from protocol

#### Defense Mechanisms

**No Governance Tokens**:
- Protocol doesn't use governance tokens
- Changes require registry owner (likely multi-sig)
- Fund-level changes require fund owner

**Immutable Fund Parameters**:
```solidity
// Critical parameters cannot be changed
address public immutable agentAumFeeWallet;
uint256 public immutable agentAumFeeBps;
address[] public allowedTokens; // Set at creation
```

**Owner-Only Changes**:
```solidity
// Only registry owner can make protocol changes
function updateRegistryParameters(...) external onlyOwner {
    // Update parameters
}
```

### 6. Reentrancy Attacks

#### Attack Description
Exploiting reentrancy in:
1. Deposit functions
2. Withdrawal functions  
3. Fee collection

#### Defense Mechanisms

**Checks-Effects-Interactions Pattern**:
```solidity
function deposit(uint256 amountWETHToDeposit, address receiver) external returns (uint256 sharesMinted) {
    // CHECKS
    if (amountWETHToDeposit < MINIMUM_DEPOSIT) revert E2();
    if (receiver == address(0)) revert E1();
    
    // EFFECTS
    uint256 navBeforeDeposit = totalNAVInAccountingAsset();
    sharesMinted = /* calculation */;
    _mint(receiver, sharesMinted); // State change first
    
    // INTERACTIONS
    IERC20(ACCOUNTING_ASSET).safeTransferFrom(msg.sender, address(this), amountWETHToDeposit);
}
```

**SafeERC20 Usage**:
```solidity
using SafeERC20 for IERC20;

// All token transfers use SafeERC20
IERC20(token).safeTransfer(to, amount);
IERC20(token).safeTransferFrom(from, to, amount);
```

## Economic Incentive Analysis

### 1. Agent Incentives

**Positive Incentives**:
- 60% of AUM fees encourages fund growth
- Performance reputation affects future opportunities
- Long-term thinking over short-term extraction

**Potential Misalignments**:
- Agents might prefer high fees over performance
- No direct performance-based compensation
- Short-term agents might not care about long-term health

**Mitigation**:
- Registry fee caps limit abuse
- Market competition drives reasonable fees
- Reputation system (future enhancement)

### 2. Investor Protection

**Built-in Protections**:
```solidity
// Minimum deposits prevent dust attacks
uint256 public constant MINIMUM_DEPOSIT = 0.01 ether;

// Proportional withdrawals ensure fairness
uint256 tokenAmountToWithdraw = (tokenBalance * sharesToBurn) / totalSupplyBeforeWithdrawal;

// Transparent fee structure
uint256 public immutable agentAumFeeBps;
```

**Market Forces**:
- Competitive fee structures
- Transparent performance tracking
- Easy fund comparison

### 3. Protocol Sustainability

**Revenue Sources**:
- 40% of all AUM fees
- Fund creation fees (if set)
- Potential future governance fees

**Cost Centers**:
- Development and maintenance
- Security audits
- Infrastructure costs

**Economic Model**:
```
Protocol Revenue = Σ(Fund_i_AUM × Fund_i_FeeRate × 0.4) + Creation_Fees
Break-even Point = Fixed_Costs / Average_Fee_Rate_Across_All_Funds
```

## Risk Assessment Framework

### 1. Attack Profitability Analysis

```javascript
class AttackProfitability {
    calculateShareManipulationCost(targetPriceMultiplier) {
        // Cost to manipulate share price by factor of X
        const minInitialDeposit = 0.01; // ETH
        const directTransferRequired = minInitialDeposit * (targetPriceMultiplier - 1);
        const totalCost = minInitialDeposit + directTransferRequired;
        
        return {
            cost: totalCost,
            // Profit only from subsequent investor's overpayment
            maxProfit: "Limited by fund adoption and next investment size",
            profitProbability: "Low - depends on finding victim investor"
        };
    }
    
    calculateSandwichProfit(fundSize, rebalanceAmount, slippageBPS) {
        const maxExtractable = rebalanceAmount * slippageBPS / 10000;
        const mevCost = "Gas fees + opportunity cost";
        
        return {
            maxProfit: maxExtractable,
            cost: mevCost,
            feasibility: slippageBPS > 50 ? "Low" : "Very Low"
        };
    }
}
```

### 2. Defense Effectiveness Metrics

| Attack Type | Defense Mechanism | Effectiveness | Cost to Bypass |
|-------------|-------------------|---------------|-----------------|
| Share Manipulation | Min deposits + liquidity shares | High | >0.01 ETH per attempt |
| Sandwich Attacks | Slippage limits + deadlines | Medium-High | Market liquidity dependent |
| Fee Extraction | Registry caps + time-based | High | Impossible to bypass |
| Flash Loans | No integration + atomicity | High | Attack vector eliminated |
| Reentrancy | CEI pattern + SafeERC20 | High | Would require new vulnerability |

### 3. Economic Security Monitoring

```javascript
class SecurityMonitor {
    async checkEconomicHealth(fundAddress) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
        
        const metrics = {
            // Share price stability
            sharePriceVolatility: await this.calculateSharePriceVolatility(fund),
            
            // Fee reasonableness
            feeRate: await fund.agentAumFeeBps(),
            feeCompetitiveness: await this.compareFeeRates(fund),
            
            // Rebalancing efficiency
            rebalanceFrequency: await this.getRebalanceFrequency(fund),
            rebalanceCosts: await this.calculateRebalanceCosts(fund),
            
            // Fund health
            nav: await fund.totalNAVInAccountingAsset(),
            shareSupply: await fund.totalSupply(),
            concentration: await this.calculateConcentration(fund)
        };
        
        return this.assessRiskLevel(metrics);
    }
    
    assessRiskLevel(metrics) {
        let riskScore = 0;
        
        // High fees = higher risk
        if (metrics.feeRate > 300) riskScore += 2; // >3%
        if (metrics.feeRate > 500) riskScore += 3; // >5%
        
        // High volatility = higher risk
        if (metrics.sharePriceVolatility > 0.05) riskScore += 2; // >5%
        
        // Low liquidity = higher risk
        if (parseFloat(ethers.formatEther(metrics.nav)) < 1) riskScore += 2;
        
        return {
            score: riskScore,
            level: riskScore < 3 ? 'LOW' : riskScore < 6 ? 'MEDIUM' : 'HIGH',
            recommendations: this.generateRecommendations(riskScore, metrics)
        };
    }
}
```

## Stress Testing Scenarios

### 1. Market Crash Scenario

**Scenario**: 50% market decline in 24 hours

**Expected Behavior**:
- NAV decreases proportionally
- Share price remains stable relative to assets
- No liquidation cascades (no leverage)
- Rebalancing continues normally

**Potential Issues**:
- DEX liquidity may dry up
- Slippage could exceed limits
- Gas prices may spike

### 2. Flash Crash During Rebalancing

**Scenario**: Price manipulation during fund rebalancing

**Protections**:
- Slippage limits prevent excessive losses
- Deadlines prevent stale transactions
- Atomic operations prevent partial execution

### 3. Agent Key Compromise

**Scenario**: Malicious actor gains agent private key

**Damage Limitation**:
- Cannot withdraw funds directly
- Cannot change fee recipients
- Can only adjust weights and trigger rebalancing
- Owner can immediately replace agent

**Recovery Process**:
```javascript
// Emergency agent replacement
const fund = new ethers.Contract(fundAddress, FUND_ABI, fundOwner);
await fund.setAgent(newTrustedAgent);
```

### 4. High Gas Price Attack

**Scenario**: Attacker spams network to prevent rebalancing

**Mitigation**:
- Agents can wait for lower gas prices
- Rebalancing is not time-critical
- Funds remain functional without rebalancing

## Best Practices for Economic Security

### 1. For Fund Creators

```javascript
// Choose reasonable fee rates
const recommendedFeeRates = {
    passive: 50,     // 0.5% for index funds
    active: 150,     // 1.5% for active management
    highFreq: 200    // 2% for high-frequency strategies
};

// Diversify token allocation
const minTokens = 3;
const maxWeightPerToken = 4000; // 40% max

// Set appropriate minimums
const minFundSize = ethers.parseEther("1.0"); // 1 ETH minimum
```

### 2. For Investors

```javascript
// Risk assessment before investing
async function assessFundRisk(fundAddress) {
    const checks = {
        feeRate: await checkFeeReasonableness(fundAddress),
        agentReputation: await checkAgentHistory(fundAddress),
        fundSize: await checkFundLiquidity(fundAddress),
        diversification: await checkPortfolioDiversification(fundAddress)
    };
    
    return calculateOverallRisk(checks);
}
```

### 3. For Agents

```javascript
// Implement protective measures
class SecureAgent {
    constructor(fundAddress, config) {
        this.maxGasPrice = config.maxGasPrice || 50e9; // 50 gwei
        this.rebalanceThreshold = config.rebalanceThreshold || 150; // 1.5%
        this.emergencyStop = config.emergencyStop || false;
    }
    
    async rebalanceWithProtection() {
        // Check gas price
        const gasPrice = await provider.getGasPrice();
        if (gasPrice > this.maxGasPrice) {
            console.log('Gas too expensive, waiting...');
            return;
        }
        
        // Check emergency conditions
        if (this.emergencyStop) {
            console.log('Emergency stop active');
            return;
        }
        
        // Execute rebalancing
        await this.fund.triggerRebalance();
    }
}
```

## Related Documentation

- [Access Control](access-control.md) - Permission-based security
- [Best Practices](best-practices.md) - Security recommendations
- [Audit Notes](audit-notes.md) - Areas requiring special attention