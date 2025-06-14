# Security Best Practices

## Overview

This guide provides comprehensive security recommendations for all participants in the WHACKROCK ecosystem: protocol administrators, fund creators, agents, investors, and integrators.

## For Protocol Administrators

### Registry Management

#### Multi-Signature Setup
```javascript
// Use Gnosis Safe for registry ownership
const gnosisSafe = await deployGnosisSafe({
    owners: [ADMIN_1, ADMIN_2, ADMIN_3],
    threshold: 2, // Require 2 of 3 signatures
    fallbackHandler: FALLBACK_HANDLER
});

await registry.transferOwnership(gnosisSafe.address);
```

#### Token Allowlist Curation
```javascript
class TokenCurator {
    async evaluateToken(tokenAddress) {
        const checks = {
            // Technical checks
            hasValidContract: await this.checkContractExists(tokenAddress),
            isERC20Compliant: await this.checkERC20Compliance(tokenAddress),
            hasReasonableSupply: await this.checkSupplyLimits(tokenAddress),
            
            // Security checks
            isNotMintable: await this.checkMintability(tokenAddress),
            hasNoBlacklist: await this.checkBlacklistFunctions(tokenAddress),
            hasReputableDev: await this.checkDeveloperReputation(tokenAddress),
            
            // Liquidity checks
            hasMainnetLiquidity: await this.checkDEXLiquidity(tokenAddress),
            hasPriceFeeds: await this.checkPriceOracles(tokenAddress)
        };
        
        return this.assessTokenRisk(checks);
    }
    
    async checkERC20Compliance(tokenAddress) {
        const token = new ethers.Contract(tokenAddress, ERC20_ABI, provider);
        
        try {
            // Test required functions
            await token.name();
            await token.symbol();
            await token.decimals();
            await token.totalSupply();
            
            return true;
        } catch {
            return false;
        }
    }
    
    async checkMintability(tokenAddress) {
        const code = await provider.getCode(tokenAddress);
        
        // Check for mint function signatures
        const mintSignatures = [
            'mint(address,uint256)',
            'mint(uint256)',
            '_mint(address,uint256)'
        ];
        
        return !mintSignatures.some(sig => 
            code.includes(ethers.id(sig).slice(2, 10))
        );
    }
}
```

#### Fee Parameter Management
```javascript
// Gradual fee adjustments to prevent ecosystem shock
async function updateFeesGradually(newTotalAumFeeBps) {
    const current = await registry.totalAumFeeBpsForFunds();
    const maxChangePerUpdate = 50; // 0.5% max change
    
    if (Math.abs(newTotalAumFeeBps - current) > maxChangePerUpdate) {
        throw new Error('Fee change too large - implement gradually');
    }
    
    await registry.updateRegistryParameters(
        await registry.USDC_TOKEN(),
        await registry.whackRockRewardsAddress(),
        await registry.protocolFundCreationFeeUsdcAmount(),
        newTotalAumFeeBps, // New fee limit
        await registry.protocolAumFeeRecipientForFunds(),
        await registry.maxAgentDepositFeeBps()
    );
}
```

### Upgrade Management

#### Timelock Implementation
```javascript
class UpgradeManager {
    constructor(timelockAddress, registryAddress) {
        this.timelock = new ethers.Contract(timelockAddress, TIMELOCK_ABI, signer);
        this.registry = new ethers.Contract(registryAddress, REGISTRY_ABI, signer);
        this.minDelay = 7 * 24 * 3600; // 7 days
    }
    
    async proposeUpgrade(newImplementation, upgradeData = '0x') {
        const upgradeCalldata = this.registry.interface.encodeFunctionData(
            'upgradeTo',
            [newImplementation]
        );
        
        // Schedule upgrade with timelock
        const salt = ethers.randomBytes(32);
        await this.timelock.schedule(
            this.registry.address,
            0,
            upgradeCalldata,
            ethers.ZeroHash,
            salt,
            this.minDelay
        );
        
        console.log('Upgrade scheduled. Can execute after:', new Date(Date.now() + this.minDelay * 1000));
        return salt;
    }
    
    async executeUpgrade(salt) {
        const upgradeCalldata = this.registry.interface.encodeFunctionData('upgradeTo', [newImplementation]);
        
        await this.timelock.execute(
            this.registry.address,
            0,
            upgradeCalldata,
            ethers.ZeroHash,
            salt
        );
        
        console.log('Upgrade executed successfully');
    }
}
```

## For Fund Creators

### Secure Fund Configuration

#### Parameter Validation
```javascript
class SecureFundCreator {
    validateFundParameters(params) {
        const validationResults = {
            // Token validation
            tokenCount: this.validateTokenCount(params.fundAllowedTokens),
            tokenDiversity: this.validateTokenDiversity(params.fundAllowedTokens),
            weightDistribution: this.validateWeightDistribution(params.initialTargetWeights),
            
            // Fee validation
            feeReasonableness: this.validateFeeRate(params.agentSetTotalAumFeeBps),
            agentReputation: this.validateAgent(params.initialAgent),
            
            // Metadata validation
            metadataCompleteness: this.validateMetadata(params)
        };
        
        const issues = Object.entries(validationResults)
            .filter(([key, result]) => !result.valid)
            .map(([key, result]) => `${key}: ${result.issue}`);
            
        if (issues.length > 0) {
            throw new Error(`Validation failed:\n${issues.join('\n')}`);
        }
        
        return true;
    }
    
    validateTokenDiversity(tokens) {
        const SAME_CATEGORY_LIMIT = 2; // Max 2 tokens from same category
        
        const categories = tokens.map(token => this.categorizeToken(token));
        const categoryCounts = categories.reduce((acc, cat) => {
            acc[cat] = (acc[cat] || 0) + 1;
            return acc;
        }, {});
        
        const overweight = Object.entries(categoryCounts)
            .filter(([cat, count]) => count > SAME_CATEGORY_LIMIT);
            
        return {
            valid: overweight.length === 0,
            issue: overweight.length > 0 ? 
                `Too many tokens in categories: ${overweight.map(([cat]) => cat).join(', ')}` : 
                null
        };
    }
    
    validateWeightDistribution(weights) {
        const MAX_SINGLE_WEIGHT = 5000; // 50% max for any single asset
        const MIN_WEIGHT = 500; // 5% minimum for diversification
        
        const overweight = weights.filter(w => w > MAX_SINGLE_WEIGHT);
        const underweight = weights.filter(w => w < MIN_WEIGHT);
        
        if (overweight.length > 0) {
            return { valid: false, issue: `Weights too concentrated: ${overweight}` };
        }
        
        if (underweight.length > 0) {
            return { valid: false, issue: `Weights too small: ${underweight}` };
        }
        
        return { valid: true };
    }
}
```

#### Agent Selection
```javascript
class AgentEvaluator {
    async evaluateAgent(agentAddress) {
        const evaluation = {
            // Technical checks
            codeVerification: await this.checkContractCode(agentAddress),
            keyManagement: await this.assessKeyManagement(agentAddress),
            
            // Historical performance
            previousFunds: await this.getPreviousFunds(agentAddress),
            performanceMetrics: await this.calculatePerformanceMetrics(agentAddress),
            
            // Reputation factors
            communityReputation: await this.checkCommunityReputation(agentAddress),
            securityIncidents: await this.checkSecurityHistory(agentAddress)
        };
        
        return this.calculateAgentScore(evaluation);
    }
    
    async checkContractCode(agentAddress) {
        const code = await provider.getCode(agentAddress);
        
        if (code === '0x') {
            return { type: 'EOA', riskLevel: 'medium', note: 'Private key managed' };
        } else {
            // Contract agent - analyze code
            return await this.analyzeContractAgent(agentAddress);
        }
    }
    
    async analyzeContractAgent(agentAddress) {
        // Check for common security patterns
        const securityChecks = {
            hasAccessControl: await this.checkAccessControlPatterns(agentAddress),
            hasUpgradeability: await this.checkUpgradePatterns(agentAddress),
            hasEmergencyStop: await this.checkEmergencyStopPatterns(agentAddress),
            isVerified: await this.checkSourceVerification(agentAddress)
        };
        
        return {
            type: 'Contract',
            riskLevel: this.assessContractRisk(securityChecks),
            details: securityChecks
        };
    }
}
```

### Fund Launch Security

#### Gradual Rollout Strategy
```javascript
class SecureFundLaunch {
    async launchFundSecurely(fundParams) {
        // Phase 1: Limited testing
        await this.createTestFund(fundParams);
        await this.performInitialTesting();
        
        // Phase 2: Small-scale launch
        const fund = await this.createProductionFund(fundParams);
        await this.implementDepositLimits(fund, ethers.parseEther('10')); // 10 ETH max initially
        
        // Phase 3: Monitor and adjust
        await this.monitorEarlyOperations(fund);
        
        // Phase 4: Full launch
        await this.removeDepositLimits(fund);
        
        return fund;
    }
    
    async performInitialTesting() {
        const tests = [
            this.testDepositsAndWithdrawals,
            this.testRebalancing,
            this.testFeeCollection,
            this.testEmergencyFunctions
        ];
        
        for (const test of tests) {
            await test();
        }
    }
}
```

## For Agents

### Key Management

#### Hardware Wallet Integration
```javascript
class SecureAgentManager {
    constructor(config) {
        if (config.useHardwareWallet) {
            this.signer = this.setupHardwareWallet(config.walletType);
        } else {
            this.signer = this.setupHotWallet(config.privateKey);
        }
    }
    
    setupHardwareWallet(walletType) {
        switch (walletType) {
            case 'ledger':
                return new LedgerSigner(provider, "m/44'/60'/0'/0/0");
            case 'trezor':
                return new TrezorSigner(provider, "m/44'/60'/0'/0/0");
            default:
                throw new Error('Unsupported hardware wallet');
        }
    }
    
    // Implement key rotation
    async rotateKeys(fundAddress, newAgentAddress) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.currentSigner);
        
        // Verify new key is different
        if (newAgentAddress.toLowerCase() === this.signer.address.toLowerCase()) {
            throw new Error('New agent address must be different');
        }
        
        // This requires fund owner to execute
        console.log('Key rotation requires fund owner approval');
        console.log(`New agent address: ${newAgentAddress}`);
        
        // Fund owner executes: fund.setAgent(newAgentAddress)
    }
}
```

#### Operational Security
```javascript
class OperationalSecurity {
    constructor(fundAddress, signer) {
        this.fund = new ethers.Contract(fundAddress, FUND_ABI, signer);
        this.securityConfig = {
            maxGasPrice: 50e9, // 50 gwei
            maxRebalanceFrequency: 3600, // 1 hour minimum between rebalances
            emergencyStopFile: './emergency.stop',
            maxSlippageBPS: 100 // 1% max slippage
        };
        
        this.lastRebalanceTime = 0;
        this.emergencyMode = false;
    }
    
    async executeSecureRebalance() {
        // Security checks before rebalancing
        await this.performSecurityChecks();
        
        try {
            await this.fund.triggerRebalance();
            this.lastRebalanceTime = Date.now();
        } catch (error) {
            await this.handleRebalanceError(error);
        }
    }
    
    async performSecurityChecks() {
        // Check gas price
        const gasPrice = await provider.getGasPrice();
        if (gasPrice > this.securityConfig.maxGasPrice) {
            throw new Error(`Gas price too high: ${gasPrice / 1e9} gwei`);
        }
        
        // Check frequency limit
        const timeSinceLastRebalance = Date.now() - this.lastRebalanceTime;
        if (timeSinceLastRebalance < this.securityConfig.maxRebalanceFrequency * 1000) {
            throw new Error('Rebalancing too frequent');
        }
        
        // Check emergency stop
        if (this.checkEmergencyStop()) {
            throw new Error('Emergency stop active');
        }
        
        // Verify we're still the agent
        const currentAgent = await this.fund.agent();
        if (currentAgent.toLowerCase() !== this.signer.address.toLowerCase()) {
            throw new Error('No longer the fund agent');
        }
    }
    
    checkEmergencyStop() {
        try {
            const fs = require('fs');
            return fs.existsSync(this.securityConfig.emergencyStopFile);
        } catch {
            return false;
        }
    }
}
```

### Strategy Security

#### Strategy Validation
```javascript
class StrategyValidator {
    validateStrategy(strategy) {
        const checks = {
            weightConstraints: this.checkWeightConstraints(strategy),
            riskLimits: this.checkRiskLimits(strategy),
            dataSource: this.validateDataSources(strategy),
            backtesting: this.validateBacktesting(strategy)
        };
        
        return this.assessStrategyRisk(checks);
    }
    
    checkWeightConstraints(strategy) {
        // Ensure strategy respects risk limits
        const constraints = {
            maxSingleAsset: 0.4,     // 40% max
            minDiversification: 3,    // At least 3 assets
            maxConcentration: 0.7     // Top 2 assets max 70%
        };
        
        return this.validateConstraints(strategy, constraints);
    }
    
    validateDataSources(strategy) {
        // Ensure strategy uses reliable data
        const approvedSources = [
            'coingecko.com',
            'coinmarketcap.com',
            'dexscreener.com'
        ];
        
        return strategy.dataSources.every(source => 
            approvedSources.some(approved => source.includes(approved))
        );
    }
}
```

## For Investors

### Due Diligence Framework

#### Pre-Investment Checklist
```javascript
class InvestorDueDiligence {
    async performDueDiligence(fundAddress) {
        const analysis = {
            // Technical analysis
            contractSecurity: await this.analyzeContractSecurity(fundAddress),
            fundMetrics: await this.analyzeFundMetrics(fundAddress),
            
            // Agent analysis
            agentReputation: await this.analyzeAgent(fundAddress),
            strategyAnalysis: await this.analyzeStrategy(fundAddress),
            
            // Portfolio analysis
            diversification: await this.analyzeDiversification(fundAddress),
            riskMetrics: await this.calculateRiskMetrics(fundAddress),
            
            // Historical performance
            performanceHistory: await this.analyzePerformance(fundAddress),
            feeAnalysis: await this.analyzeFees(fundAddress)
        };
        
        return this.generateInvestmentRecommendation(analysis);
    }
    
    async analyzeFundMetrics(fundAddress) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
        
        const metrics = {
            nav: await fund.totalNAVInAccountingAsset(),
            totalShares: await fund.totalSupply(),
            sharePrice: await this.calculateSharePrice(fund),
            age: await this.calculateFundAge(fundAddress),
            volume: await this.calculateTradingVolume(fundAddress)
        };
        
        return {
            size: this.categorizeSize(metrics.nav),
            maturity: this.categorizeMaturity(metrics.age),
            liquidity: this.categorizeLiquidity(metrics.volume),
            stability: this.analyzeSharePriceStability(fundAddress)
        };
    }
    
    async calculateRiskMetrics(fundAddress) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
        const [weights, tokens] = await fund.getCurrentCompositionBPS();
        
        return {
            concentration: this.calculateConcentrationRisk(weights),
            correlation: await this.calculateCorrelationRisk(tokens),
            volatility: await this.calculateVolatilityRisk(tokens),
            liquidityRisk: await this.calculateLiquidityRisk(tokens)
        };
    }
}
```

#### Risk Management
```javascript
class InvestorRiskManagement {
    constructor(maxPortfolioAllocation = 0.1) { // 10% max per fund
        this.maxAllocation = maxPortfolioAllocation;
        this.riskLimits = {
            maxFeeRate: 300,        // 3% max annual fee
            minFundSize: ethers.parseEther('1'), // 1 ETH minimum
            maxConcentration: 0.5,  // 50% max in single asset
            minDiversification: 3   // At least 3 tokens
        };
    }
    
    async validateInvestment(fundAddress, investmentAmount, totalPortfolio) {
        const allocation = Number(investmentAmount) / Number(totalPortfolio);
        
        if (allocation > this.maxAllocation) {
            throw new Error(`Investment exceeds maximum allocation: ${allocation * 100}%`);
        }
        
        const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
        
        // Check fee rate
        const feeRate = await fund.agentAumFeeBps();
        if (feeRate > this.riskLimits.maxFeeRate) {
            throw new Error(`Fee rate too high: ${feeRate / 100}%`);
        }
        
        // Check fund size
        const nav = await fund.totalNAVInAccountingAsset();
        if (nav < this.riskLimits.minFundSize) {
            throw new Error(`Fund too small: ${ethers.formatEther(nav)} ETH`);
        }
        
        // Check diversification
        const [weights] = await fund.getCurrentCompositionBPS();
        const maxWeight = Math.max(...weights.map(w => Number(w)));
        
        if (maxWeight > this.riskLimits.maxConcentration * 10000) {
            throw new Error(`Portfolio too concentrated: ${maxWeight / 100}%`);
        }
        
        if (weights.length < this.riskLimits.minDiversification) {
            throw new Error(`Insufficient diversification: ${weights.length} tokens`);
        }
        
        return true;
    }
}
```

## For Integrators

### Smart Contract Integration

#### Defensive Programming
```javascript
class DefensiveIntegration {
    constructor(config) {
        this.retryAttempts = config.retryAttempts || 3;
        this.timeoutMs = config.timeoutMs || 30000;
        this.circuitBreaker = new CircuitBreaker();
    }
    
    async safeContractCall(contractMethod, ...args) {
        return await this.circuitBreaker.execute(async () => {
            for (let attempt = 1; attempt <= this.retryAttempts; attempt++) {
                try {
                    const result = await Promise.race([
                        contractMethod(...args),
                        this.timeout(this.timeoutMs)
                    ]);
                    
                    // Validate result
                    await this.validateResult(result);
                    return result;
                    
                } catch (error) {
                    console.warn(`Attempt ${attempt} failed:`, error.message);
                    
                    if (attempt === this.retryAttempts) {
                        throw error;
                    }
                    
                    await this.delay(1000 * attempt); // Exponential backoff
                }
            }
        });
    }
    
    async validateResult(result) {
        // Validate contract call results
        if (result === null || result === undefined) {
            throw new Error('Invalid contract response');
        }
        
        // Additional validation based on expected result type
        if (typeof result === 'object' && result.hash) {
            // Transaction result - wait for confirmation
            const receipt = await result.wait(1);
            if (receipt.status !== 1) {
                throw new Error('Transaction failed');
            }
        }
    }
    
    timeout(ms) {
        return new Promise((_, reject) => 
            setTimeout(() => reject(new Error('Operation timeout')), ms)
        );
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

class CircuitBreaker {
    constructor(threshold = 5, resetTime = 60000) {
        this.failureThreshold = threshold;
        this.resetTime = resetTime;
        this.failureCount = 0;
        this.lastFailureTime = null;
        this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    }
    
    async execute(operation) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.resetTime) {
                this.state = 'HALF_OPEN';
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await operation();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }
    
    onSuccess() {
        this.failureCount = 0;
        this.state = 'CLOSED';
    }
    
    onFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();
        
        if (this.failureCount >= this.failureThreshold) {
            this.state = 'OPEN';
        }
    }
}
```

### Frontend Security

#### Input Validation
```javascript
class FrontendSecurity {
    static validateAddress(address) {
        if (!ethers.isAddress(address)) {
            throw new Error('Invalid Ethereum address');
        }
        
        if (address === ethers.ZeroAddress) {
            throw new Error('Zero address not allowed');
        }
        
        return address.toLowerCase();
    }
    
    static validateAmount(amount, decimals = 18) {
        try {
            const parsed = ethers.parseUnits(amount.toString(), decimals);
            
            if (parsed <= 0) {
                throw new Error('Amount must be positive');
            }
            
            if (parsed > ethers.parseUnits('1000000', decimals)) {
                throw new Error('Amount too large');
            }
            
            return parsed;
        } catch (error) {
            throw new Error(`Invalid amount: ${error.message}`);
        }
    }
    
    static sanitizeInput(input) {
        if (typeof input !== 'string') {
            throw new Error('Input must be string');
        }
        
        // Remove dangerous characters
        const sanitized = input
            .replace(/[<>\"'&]/g, '')
            .trim()
            .slice(0, 1000); // Max length
            
        return sanitized;
    }
    
    static validateWeights(weights) {
        if (!Array.isArray(weights)) {
            throw new Error('Weights must be array');
        }
        
        if (weights.length === 0 || weights.length > 10) {
            throw new Error('Invalid weights array length');
        }
        
        const total = weights.reduce((sum, w) => {
            const weight = parseInt(w);
            if (isNaN(weight) || weight <= 0 || weight > 10000) {
                throw new Error('Invalid weight value');
            }
            return sum + weight;
        }, 0);
        
        if (total !== 10000) {
            throw new Error(`Weights must sum to 10000, got ${total}`);
        }
        
        return weights.map(w => parseInt(w));
    }
}
```

## Security Monitoring

### Continuous Monitoring
```javascript
class SecurityMonitor {
    constructor(contracts) {
        this.contracts = contracts;
        this.alerts = [];
        this.thresholds = {
            maxGasPrice: 100e9,     // 100 gwei
            maxSlippage: 500,       // 5%
            maxRebalanceFreq: 3600, // 1 hour
            minLiquidity: ethers.parseEther('0.1')
        };
    }
    
    startMonitoring() {
        // Monitor gas prices
        setInterval(async () => {
            const gasPrice = await provider.getGasPrice();
            if (gasPrice > this.thresholds.maxGasPrice) {
                this.alert('HIGH_GAS_PRICE', { gasPrice: gasPrice / 1e9 });
            }
        }, 60000); // Every minute
        
        // Monitor fund activities
        this.contracts.funds.forEach(fund => {
            fund.on('RebalanceCycleExecuted', (navBefore, navAfter, timestamp) => {
                const slippage = Math.abs(navAfter - navBefore) * 10000 / navBefore;
                if (slippage > this.thresholds.maxSlippage) {
                    this.alert('HIGH_SLIPPAGE', { fund: fund.address, slippage });
                }
            });
            
            fund.on('EmergencyWithdrawal', (token, amount) => {
                this.alert('EMERGENCY_WITHDRAWAL', { fund: fund.address, token, amount });
            });
        });
    }
    
    alert(type, data) {
        const alert = {
            type,
            data,
            timestamp: new Date().toISOString(),
            severity: this.getAlertSeverity(type)
        };
        
        this.alerts.push(alert);
        this.notifyAdmins(alert);
        
        // Store alert for historical analysis
        this.storeAlert(alert);
    }
    
    getAlertSeverity(type) {
        const severityMap = {
            'HIGH_GAS_PRICE': 'low',
            'HIGH_SLIPPAGE': 'medium',
            'EMERGENCY_WITHDRAWAL': 'high',
            'SUSPICIOUS_ACTIVITY': 'high',
            'CONTRACT_UPGRADE': 'medium'
        };
        
        return severityMap[type] || 'medium';
    }
}
```

## Incident Response

### Emergency Procedures
```javascript
class EmergencyResponse {
    constructor(contracts, adminWallet) {
        this.contracts = contracts;
        this.adminWallet = adminWallet;
        this.emergencyProcedures = new Map();
        
        this.setupEmergencyProcedures();
    }
    
    setupEmergencyProcedures() {
        this.emergencyProcedures.set('COMPROMISED_AGENT', this.handleCompromisedAgent.bind(this));
        this.emergencyProcedures.set('CONTRACT_BUG', this.handleContractBug.bind(this));
        this.emergencyProcedures.set('ECONOMIC_ATTACK', this.handleEconomicAttack.bind(this));
        this.emergencyProcedures.set('ORACLE_FAILURE', this.handleOracleFailure.bind(this));
    }
    
    async handleEmergency(type, fundAddress, details) {
        console.log(`🚨 EMERGENCY: ${type} detected for fund ${fundAddress}`);
        
        const procedure = this.emergencyProcedures.get(type);
        if (!procedure) {
            throw new Error(`No emergency procedure for type: ${type}`);
        }
        
        await procedure(fundAddress, details);
    }
    
    async handleCompromisedAgent(fundAddress, details) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.adminWallet);
        
        // 1. Immediately replace agent (requires fund owner)
        console.log('Replacing compromised agent...');
        await fund.setAgent(EMERGENCY_AGENT_ADDRESS);
        
        // 2. Trigger emergency withdrawal if needed
        if (details.severity === 'high') {
            console.log('Triggering emergency asset protection...');
            // Emergency procedures depend on specific fund owner setup
        }
        
        // 3. Notify all stakeholders
        await this.notifyStakeholders('AGENT_COMPROMISED', fundAddress);
        
        // 4. Begin forensic analysis
        await this.beginForensicAnalysis(fundAddress, details);
    }
    
    async handleContractBug(fundAddress, details) {
        // 1. Pause all operations if possible
        console.log('Pausing fund operations...');
        
        // 2. Assess impact
        const impact = await this.assessBugImpact(fundAddress, details);
        
        // 3. If severe, trigger emergency withdrawals
        if (impact.severity === 'critical') {
            await this.triggerEmergencyWithdrawals(fundAddress);
        }
        
        // 4. Coordinate with security researchers
        await this.coordinateSecurityResponse(details);
    }
}
```

## Related Documentation

- [Access Control](access-control.md) - Permission-based security
- [Economic Security](economic-security.md) - Financial attack vectors
- [Audit Notes](audit-notes.md) - Areas requiring special attention
- [Integration Guide](../integration/quick-start.md) - Secure integration patterns