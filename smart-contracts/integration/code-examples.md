# Code Examples

## Overview

This page provides complete, production-ready code examples for common WHACKROCK integration patterns. All examples include error handling, gas optimization, and best practices.

## Complete Fund Creation Example

```javascript
// fund-creator.js - Complete fund creation with validation
import { ethers } from 'ethers';

class FundCreator {
    constructor(registryAddress, wallet) {
        this.registryAddress = registryAddress;
        this.wallet = wallet;
        this.registry = new ethers.Contract(registryAddress, REGISTRY_ABI, wallet);
    }
    
    async createFund(params) {
        console.log('🚀 Starting fund creation process...');
        
        try {
            // Step 1: Validate all parameters
            await this.validateParameters(params);
            
            // Step 2: Prepare creation fee
            await this.prepareCreationFee();
            
            // Step 3: Estimate gas cost
            const gasCost = await this.estimateGasCost(params);
            console.log(`Estimated gas cost: ${ethers.formatEther(gasCost)} ETH`);
            
            // Step 4: Create fund
            const result = await this.executeFundCreation(params);
            
            // Step 5: Verify deployment
            await this.verifyDeployment(result.fundAddress);
            
            console.log('🎉 Fund creation successful!');
            return result;
            
        } catch (error) {
            console.error('❌ Fund creation failed:', error.message);
            throw new Error(`Fund creation failed: ${error.message}`);
        }
    }
    
    async validateParameters(params) {
        console.log('Validating parameters...');
        
        // Get registry constraints
        const [allowedTokens, maxTokens, maxFee, isSymbolTaken] = await Promise.all([
            this.registry.getRegistryAllowedTokens(),
            this.registry.maxInitialAllowedTokensLength(),
            this.registry.totalAumFeeBpsForFunds(),
            this.registry.isSymbolTaken(params.vaultSymbol)
        ]);
        
        // Validate token count
        if (params.fundAllowedTokens.length === 0) {
            throw new Error('Fund must have at least one token');
        }
        
        if (params.fundAllowedTokens.length > maxTokens) {
            throw new Error(`Too many tokens. Maximum: ${maxTokens}, provided: ${params.fundAllowedTokens.length}`);
        }
        
        // Validate tokens are allowed
        for (const token of params.fundAllowedTokens) {
            if (!allowedTokens.includes(token)) {
                throw new Error(`Token ${token} not in registry allowlist`);
            }
        }
        
        // Validate weights
        if (params.initialTargetWeights.length !== params.fundAllowedTokens.length) {
            throw new Error('Weights array length must match tokens array length');
        }
        
        const totalWeight = params.initialTargetWeights.reduce((a, b) => a + b, 0);
        if (totalWeight !== 10000) {
            throw new Error(`Weights must sum to 10000, got ${totalWeight}`);
        }
        
        // Check for zero weights
        if (params.initialTargetWeights.some(w => w === 0)) {
            throw new Error('Zero weights not allowed');
        }
        
        // Validate symbol uniqueness
        if (isSymbolTaken) {
            throw new Error(`Symbol "${params.vaultSymbol}" already taken`);
        }
        
        // Validate fee rate
        if (params.agentSetTotalAumFeeBps > maxFee) {
            throw new Error(`Fee rate ${params.agentSetTotalAumFeeBps} exceeds maximum ${maxFee}`);
        }
        
        // Validate addresses
        const zeroAddress = ethers.ZeroAddress;
        if (params.initialAgent === zeroAddress) {
            throw new Error('Initial agent cannot be zero address');
        }
        
        if (params.agentAumFeeWallet === zeroAddress) {
            throw new Error('Agent fee wallet cannot be zero address');
        }
        
        console.log('✅ All parameters valid');
    }
    
    async prepareCreationFee() {
        const feeAmount = await this.registry.protocolFundCreationFeeUsdcAmount();
        
        if (feeAmount > 0) {
            const usdcAddress = await this.registry.USDC_TOKEN();
            const usdc = new ethers.Contract(usdcAddress, [
                'function balanceOf(address) view returns (uint256)',
                'function approve(address,uint256) external',
                'function allowance(address,address) view returns (uint256)'
            ], this.wallet);
            
            const balance = await usdc.balanceOf(this.wallet.address);
            if (balance < feeAmount) {
                throw new Error(`Insufficient USDC balance. Need ${ethers.formatUnits(feeAmount, 6)}, have ${ethers.formatUnits(balance, 6)}`);
            }
            
            const allowance = await usdc.allowance(this.wallet.address, this.registryAddress);
            if (allowance < feeAmount) {
                console.log(`Approving ${ethers.formatUnits(feeAmount, 6)} USDC...`);
                const tx = await usdc.approve(this.registryAddress, feeAmount);
                await tx.wait();
                console.log('✅ USDC approved');
            }
        }
    }
    
    async estimateGasCost(params) {
        try {
            const gasEstimate = await this.registry.createWhackRockFund.estimateGas(
                params.initialAgent,
                params.fundAllowedTokens,
                params.initialTargetWeights,
                params.vaultName,
                params.vaultSymbol,
                params.vaultURI,
                params.description,
                params.agentAumFeeWallet,
                params.agentSetTotalAumFeeBps
            );
            
            const feeData = await this.wallet.provider.getFeeData();
            return gasEstimate * feeData.gasPrice;
        } catch (error) {
            console.warn('Gas estimation failed, using default estimate');
            return ethers.parseEther('0.01'); // Default estimate
        }
    }
    
    async executeFundCreation(params) {
        console.log('Creating fund...');
        
        const tx = await this.registry.createWhackRockFund(
            params.initialAgent,
            params.fundAllowedTokens,
            params.initialTargetWeights,
            params.vaultName,
            params.vaultSymbol,
            params.vaultURI,
            params.description,
            params.agentAumFeeWallet,
            params.agentSetTotalAumFeeBps
        );
        
        console.log(`Transaction sent: ${tx.hash}`);
        const receipt = await tx.wait();
        
        // Extract fund address from event
        const fundCreatedEvent = receipt.logs.find(log => {
            try {
                const parsed = this.registry.interface.parseLog(log);
                return parsed.name === 'WhackRockFundCreated';
            } catch {
                return false;
            }
        });
        
        if (!fundCreatedEvent) {
            throw new Error('Fund creation event not found in transaction receipt');
        }
        
        const parsedEvent = this.registry.interface.parseLog(fundCreatedEvent);
        
        return {
            fundAddress: parsedEvent.args.fundAddress,
            fundId: parsedEvent.args.fundId,
            transactionHash: tx.hash,
            blockNumber: receipt.blockNumber,
            gasUsed: receipt.gasUsed
        };
    }
    
    async verifyDeployment(fundAddress) {
        console.log('Verifying deployment...');
        
        const fund = new ethers.Contract(fundAddress, [
            'function name() view returns (string)',
            'function symbol() view returns (string)',
            'function agent() view returns (address)',
            'function totalNAVInAccountingAsset() view returns (uint256)'
        ], this.wallet.provider);
        
        try {
            const [name, symbol, agent, nav] = await Promise.all([
                fund.name(),
                fund.symbol(),
                fund.agent(),
                fund.totalNAVInAccountingAsset()
            ]);
            
            console.log(`✅ Fund verified:`);
            console.log(`  Name: ${name}`);
            console.log(`  Symbol: ${symbol}`);
            console.log(`  Agent: ${agent}`);
            console.log(`  Initial NAV: ${ethers.formatEther(nav)} WETH`);
            
        } catch (error) {
            throw new Error(`Fund deployment verification failed: ${error.message}`);
        }
    }
}

// Usage example
async function createAIFund() {
    const creator = new FundCreator(REGISTRY_ADDRESS, wallet);
    
    const fundParams = {
        initialAgent: aiAgentAddress,
        fundAllowedTokens: [WBTC_ADDRESS, ETH_ADDRESS, USDC_ADDRESS],
        initialTargetWeights: [4000, 4000, 2000], // 40%, 40%, 20%
        vaultName: "AI Growth Fund",
        vaultSymbol: "AIGF",
        vaultURI: "ipfs://QmAIFundMetadata...",
        description: "AI-managed growth strategy focusing on digital assets",
        agentAumFeeWallet: aiAgentFeeWallet,
        agentSetTotalAumFeeBps: 200 // 2% annual fee
    };
    
    const result = await creator.createFund(fundParams);
    console.log('New fund created:', result.fundAddress);
    
    return result;
}
```

## Complete Investment Example

```javascript
// investor.js - Complete investment flow with risk management
class SmartInvestor {
    constructor(wallet) {
        this.wallet = wallet;
        this.provider = wallet.provider;
        this.maxSlippageBPS = 50; // 0.5%
        this.maxGasPriceGwei = 50;
    }
    
    async investWithRiskChecks(fundAddress, wethAmount, options = {}) {
        console.log('🎯 Starting smart investment process...');
        
        try {
            // Step 1: Analyze fund
            const analysis = await this.analyzeFund(fundAddress);
            console.log('Fund analysis complete');
            
            // Step 2: Risk assessment
            await this.assessRisk(analysis, wethAmount, options);
            
            // Step 3: Check gas prices
            await this.checkGasPrice();
            
            // Step 4: Prepare WETH
            await this.prepareWETH(wethAmount);
            
            // Step 5: Calculate investment
            const calculation = await this.calculateInvestment(fundAddress, wethAmount);
            console.log(`Expected shares: ${calculation.expectedShares}`);
            
            // Step 6: Execute investment
            const result = await this.executeInvestment(fundAddress, wethAmount, calculation);
            
            // Step 7: Verify investment
            await this.verifyInvestment(fundAddress, result);
            
            console.log('🎉 Investment successful!');
            return result;
            
        } catch (error) {
            console.error('❌ Investment failed:', error.message);
            throw error;
        }
    }
    
    async analyzeFund(fundAddress) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.provider);
        
        const [
            name,
            symbol,
            agent,
            nav,
            totalShares,
            feeRate,
            [currentWeights, tokenAddresses, tokenSymbols],
            [targetWeights]
        ] = await Promise.all([
            fund.name(),
            fund.symbol(),
            fund.agent(),
            fund.totalNAVInAccountingAsset(),
            fund.totalSupply(),
            fund.agentAumFeeBps(),
            fund.getCurrentCompositionBPS(),
            fund.getTargetCompositionBPS()
        ]);
        
        const sharePrice = totalShares > 0 ? nav * BigInt(1e18) / totalShares : BigInt(1e18);
        
        return {
            basic: { name, symbol, agent, nav, totalShares, sharePrice, feeRate },
            portfolio: {
                tokens: tokenAddresses.map((addr, i) => ({
                    address: addr,
                    symbol: tokenSymbols[i],
                    currentWeight: currentWeights[i],
                    targetWeight: targetWeights[i],
                    deviation: Math.abs(currentWeights[i] - targetWeights[i])
                }))
            }
        };
    }
    
    async assessRisk(analysis, amount, options) {
        const risks = [];
        
        // Check fund size (avoid funds that are too small)
        const minFundSize = options.minFundSizeWETH || 1;
        const fundSizeWETH = parseFloat(ethers.formatEther(analysis.basic.nav));
        
        if (fundSizeWETH < minFundSize) {
            risks.push(`Fund too small: ${fundSizeWETH} WETH < ${minFundSize} WETH minimum`);
        }
        
        // Check fee rate
        const maxFeeRate = options.maxFeeRateBPS || 500; // 5%
        if (analysis.basic.feeRate > maxFeeRate) {
            risks.push(`Fee rate too high: ${analysis.basic.feeRate / 100}% > ${maxFeeRate / 100}%`);
        }
        
        // Check portfolio balance
        const maxDeviation = Math.max(...analysis.portfolio.tokens.map(t => t.deviation));
        if (maxDeviation > 500) { // 5%
            risks.push(`Portfolio imbalanced: ${maxDeviation / 100}% max deviation`);
        }
        
        // Check agent activity (simplified)
        if (analysis.basic.agent === ethers.ZeroAddress) {
            risks.push('No active agent managing the fund');
        }
        
        // Check investment size relative to fund
        const amountWETH = parseFloat(ethers.formatEther(amount));
        const investmentPercentage = (amountWETH / fundSizeWETH) * 100;
        
        if (investmentPercentage > 25) {
            risks.push(`Large investment: ${investmentPercentage.toFixed(1)}% of fund`);
        }
        
        if (risks.length > 0 && !options.ignoreRisks) {
            throw new Error(`Risk assessment failed:\n${risks.join('\n')}`);
        }
        
        if (risks.length > 0) {
            console.warn('⚠️  Risks identified:', risks);
        }
    }
    
    async checkGasPrice() {
        const feeData = await this.provider.getFeeData();
        const gasPriceGwei = Number(feeData.gasPrice) / 1e9;
        
        if (gasPriceGwei > this.maxGasPriceGwei) {
            throw new Error(`Gas price too high: ${gasPriceGwei.toFixed(1)} gwei > ${this.maxGasPriceGwei} gwei`);
        }
        
        console.log(`✅ Gas price acceptable: ${gasPriceGwei.toFixed(1)} gwei`);
    }
    
    async prepareWETH(amount) {
        const weth = new ethers.Contract(WETH_ADDRESS, [
            'function balanceOf(address) view returns (uint256)',
            'function deposit() payable',
            'function approve(address,uint256) external'
        ], this.wallet);
        
        const balance = await weth.balanceOf(this.wallet.address);
        
        if (balance < amount) {
            const needed = amount - balance;
            console.log(`Converting ${ethers.formatEther(needed)} ETH to WETH...`);
            
            const ethBalance = await this.provider.getBalance(this.wallet.address);
            if (ethBalance < needed) {
                throw new Error(`Insufficient ETH balance: need ${ethers.formatEther(needed)}, have ${ethers.formatEther(ethBalance)}`);
            }
            
            const tx = await weth.deposit({ value: needed });
            await tx.wait();
            console.log('✅ ETH converted to WETH');
        }
        
        console.log('✅ WETH balance sufficient');
    }
    
    async calculateInvestment(fundAddress, amount) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.provider);
        
        const nav = await fund.totalNAVInAccountingAsset();
        const totalShares = await fund.totalSupply();
        
        let expectedShares;
        if (totalShares === 0n) {
            expectedShares = amount;
        } else {
            expectedShares = (amount * totalShares) / nav;
        }
        
        const sharePrice = totalShares > 0 ? nav * BigInt(1e18) / totalShares : BigInt(1e18);
        
        return {
            amount: ethers.formatEther(amount),
            expectedShares: ethers.formatEther(expectedShares),
            sharePrice: ethers.formatEther(sharePrice),
            nav: ethers.formatEther(nav),
            totalShares: ethers.formatEther(totalShares)
        };
    }
    
    async executeInvestment(fundAddress, amount, calculation) {
        console.log('Executing investment...');
        
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.wallet);
        const weth = new ethers.Contract(WETH_ADDRESS, [
            'function approve(address,uint256) external'
        ], this.wallet);
        
        // Approve WETH
        console.log('Approving WETH...');
        const approveTx = await weth.approve(fundAddress, amount);
        await approveTx.wait();
        
        // Execute deposit
        console.log('Depositing WETH...');
        const depositTx = await fund.deposit(amount, this.wallet.address);
        const receipt = await depositTx.wait();
        
        return {
            transactionHash: depositTx.hash,
            blockNumber: receipt.blockNumber,
            gasUsed: receipt.gasUsed,
            calculation
        };
    }
    
    async verifyInvestment(fundAddress, result) {
        const fund = new ethers.Contract(fundAddress, FUND_ABI, this.provider);
        
        const actualShares = await fund.balanceOf(this.wallet.address);
        const expectedShares = ethers.parseEther(result.calculation.expectedShares);
        
        // Allow for small differences due to timing
        const difference = actualShares > expectedShares ? 
            actualShares - expectedShares : 
            expectedShares - actualShares;
        
        const toleranceBPS = 10; // 0.1%
        const tolerance = (expectedShares * BigInt(toleranceBPS)) / 10000n;
        
        if (difference > tolerance) {
            console.warn(`⚠️  Share amount differs from expected:`);
            console.warn(`  Expected: ${ethers.formatEther(expectedShares)}`);
            console.warn(`  Actual: ${ethers.formatEther(actualShares)}`);
        } else {
            console.log(`✅ Shares received: ${ethers.formatEther(actualShares)}`);
        }
    }
}

// Usage example
async function smartInvest() {
    const investor = new SmartInvestor(wallet);
    
    const result = await investor.investWithRiskChecks(
        fundAddress,
        ethers.parseEther('0.5'), // 0.5 WETH
        {
            minFundSizeWETH: 2,      // Minimum 2 WETH fund size
            maxFeeRateBPS: 300,      // Maximum 3% fee
            ignoreRisks: false       // Don't ignore risk warnings
        }
    );
    
    console.log('Investment result:', result);
}
```

## Production Agent Bot

```javascript
// agent-bot.js - Production-ready agent automation
class ProductionAgent {
    constructor(config) {
        this.config = {
            fundAddress: config.fundAddress,
            privateKey: config.privateKey,
            rpcUrl: config.rpcUrl,
            strategy: config.strategy,
            rebalanceThresholdBPS: config.rebalanceThresholdBPS || 100,
            feeCollectionMinWETH: config.feeCollectionMinWETH || 0.01,
            maxGasPriceGwei: config.maxGasPriceGwei || 50,
            checkIntervalMinutes: config.checkIntervalMinutes || 60,
            emergencyStopFile: config.emergencyStopFile || './emergency_stop',
            logLevel: config.logLevel || 'info'
        };
        
        this.provider = new ethers.JsonRpcProvider(this.config.rpcUrl);
        this.wallet = new ethers.Wallet(this.config.privateKey, this.provider);
        this.fund = new ethers.Contract(this.config.fundAddress, FUND_ABI, this.wallet);
        
        this.isRunning = false;
        this.metrics = {
            startTime: null,
            rebalancesExecuted: 0,
            feesCollected: 0,
            errorsEncountered: 0,
            lastActivity: null
        };
        
        this.setupLogger();
    }
    
    setupLogger() {
        // Simple logger implementation
        this.log = {
            info: (msg) => console.log(`[INFO ${new Date().toISOString()}] ${msg}`),
            warn: (msg) => console.warn(`[WARN ${new Date().toISOString()}] ${msg}`),
            error: (msg) => console.error(`[ERROR ${new Date().toISOString()}] ${msg}`)
        };
    }
    
    async start() {
        this.log.info('🤖 Starting Production Agent');
        
        try {
            // Verify agent permissions
            await this.verifyPermissions();
            
            // Initialize metrics
            this.metrics.startTime = Date.now();
            this.isRunning = true;
            
            // Start main loop
            this.startMainLoop();
            
            // Setup graceful shutdown
            this.setupGracefulShutdown();
            
            this.log.info('✅ Production Agent started successfully');
            
        } catch (error) {
            this.log.error(`Failed to start agent: ${error.message}`);
            throw error;
        }
    }
    
    async verifyPermissions() {
        const currentAgent = await this.fund.agent();
        if (currentAgent.toLowerCase() !== this.wallet.address.toLowerCase()) {
            throw new Error(`Not authorized as agent. Current: ${currentAgent}, Wallet: ${this.wallet.address}`);
        }
        
        const balance = await this.provider.getBalance(this.wallet.address);
        if (balance < ethers.parseEther('0.01')) {
            throw new Error(`Insufficient ETH balance for gas: ${ethers.formatEther(balance)}`);
        }
        
        this.log.info(`Agent verified: ${this.wallet.address}`);
        this.log.info(`ETH balance: ${ethers.formatEther(balance)}`);
    }
    
    startMainLoop() {
        const intervalMs = this.config.checkIntervalMinutes * 60 * 1000;
        
        this.mainInterval = setInterval(async () => {
            try {
                // Check for emergency stop
                if (await this.checkEmergencyStop()) {
                    this.log.warn('Emergency stop detected, shutting down...');
                    await this.stop();
                    return;
                }
                
                // Execute main logic
                await this.executeMainLogic();
                
            } catch (error) {
                this.metrics.errorsEncountered++;
                this.log.error(`Main loop error: ${error.message}`);
                
                // Stop if too many errors
                if (this.metrics.errorsEncountered > 10) {
                    this.log.error('Too many errors, stopping agent');
                    await this.stop();
                }
            }
        }, intervalMs);
    }
    
    async executeMainLogic() {
        this.log.info('Executing main logic cycle...');
        
        // Check and execute strategy
        await this.checkAndExecuteStrategy();
        
        // Check and trigger rebalancing
        await this.checkAndRebalance();
        
        // Check and collect fees
        await this.checkAndCollectFees();
        
        // Update metrics
        this.metrics.lastActivity = Date.now();
        
        this.log.info('Main logic cycle complete');
    }
    
    async checkAndExecuteStrategy() {
        try {
            if (!this.config.strategy) return;
            
            this.log.info('Checking strategy execution...');
            
            const shouldExecute = await this.config.strategy.shouldExecute();
            if (shouldExecute) {
                this.log.info('Executing strategy...');
                
                const marketData = await this.getMarketData();
                const newWeights = await this.config.strategy.calculateWeights(marketData);
                
                await this.fund.setTargetWeightsAndRebalanceIfNeeded(newWeights);
                
                this.log.info('✅ Strategy executed successfully');
            }
        } catch (error) {
            this.log.error(`Strategy execution failed: ${error.message}`);
        }
    }
    
    async checkAndRebalance() {
        try {
            this.log.info('Checking rebalancing need...');
            
            const [currentWeights] = await this.fund.getCurrentCompositionBPS();
            const [targetWeights] = await this.fund.getTargetCompositionBPS();
            
            let maxDeviation = 0;
            for (let i = 0; i < currentWeights.length; i++) {
                const deviation = Math.abs(Number(currentWeights[i]) - Number(targetWeights[i]));
                maxDeviation = Math.max(maxDeviation, deviation);
            }
            
            if (maxDeviation > this.config.rebalanceThresholdBPS) {
                // Check gas price
                const feeData = await this.provider.getFeeData();
                const gasPriceGwei = Number(feeData.gasPrice) / 1e9;
                
                if (gasPriceGwei <= this.config.maxGasPriceGwei) {
                    this.log.info(`Triggering rebalance (${maxDeviation / 100}% deviation)`);
                    
                    await this.fund.triggerRebalance();
                    this.metrics.rebalancesExecuted++;
                    
                    this.log.info('✅ Rebalancing completed');
                } else {
                    this.log.warn(`Gas too expensive for rebalancing: ${gasPriceGwei} gwei`);
                }
            } else {
                this.log.info(`No rebalancing needed (${maxDeviation / 100}% deviation)`);
            }
        } catch (error) {
            this.log.error(`Rebalancing check failed: ${error.message}`);
        }
    }
    
    async checkAndCollectFees() {
        try {
            this.log.info('Checking fee collection...');
            
            const nav = await this.fund.totalNAVInAccountingAsset();
            const feeRate = await this.fund.agentAumFeeBps();
            const lastCollection = await this.fund.lastAgentAumFeeCollectionTimestamp();
            
            const timeElapsed = Math.floor(Date.now() / 1000) - Number(lastCollection);
            const totalFeeValue = (nav * feeRate * BigInt(timeElapsed)) / (10000n * BigInt(365 * 24 * 3600));
            
            const minFeeValue = ethers.parseEther(this.config.feeCollectionMinWETH.toString());
            
            if (totalFeeValue >= minFeeValue) {
                // Check gas price
                const feeData = await this.provider.getFeeData();
                const gasPriceGwei = Number(feeData.gasPrice) / 1e9;
                
                if (gasPriceGwei <= this.config.maxGasPriceGwei) {
                    this.log.info(`Collecting fees: ${ethers.formatEther(totalFeeValue)} WETH`);
                    
                    await this.fund.collectAgentManagementFee();
                    this.metrics.feesCollected++;
                    
                    this.log.info('✅ Fees collected');
                } else {
                    this.log.warn(`Gas too expensive for fee collection: ${gasPriceGwei} gwei`);
                }
            } else {
                this.log.info(`Fees too small to collect: ${ethers.formatEther(totalFeeValue)} WETH`);
            }
        } catch (error) {
            this.log.error(`Fee collection check failed: ${error.message}`);
        }
    }
    
    async checkEmergencyStop() {
        try {
            const fs = require('fs');
            return fs.existsSync(this.config.emergencyStopFile);
        } catch {
            return false;
        }
    }
    
    setupGracefulShutdown() {
        const shutdown = async (signal) => {
            this.log.info(`Received ${signal}, shutting down gracefully...`);
            await this.stop();
            process.exit(0);
        };
        
        process.on('SIGINT', shutdown);
        process.on('SIGTERM', shutdown);
    }
    
    async stop() {
        this.log.info('Stopping Production Agent...');
        
        this.isRunning = false;
        
        if (this.mainInterval) {
            clearInterval(this.mainInterval);
        }
        
        // Log final metrics
        const uptime = Date.now() - this.metrics.startTime;
        this.log.info(`Agent stopped. Uptime: ${Math.floor(uptime / 1000)}s`);
        this.log.info(`Rebalances: ${this.metrics.rebalancesExecuted}`);
        this.log.info(`Fees collected: ${this.metrics.feesCollected}`);
        this.log.info(`Errors: ${this.metrics.errorsEncountered}`);
    }
    
    async getMarketData() {
        // Mock implementation - replace with real market data source
        const [weights, addresses, symbols] = await this.fund.getCurrentCompositionBPS();
        
        return addresses.map((addr, i) => ({
            address: addr,
            symbol: symbols[i],
            price: Math.random() * 1000,
            volume: Math.random() * 1000000,
            momentum: Math.random() * 0.2 - 0.1
        }));
    }
    
    getStatus() {
        return {
            isRunning: this.isRunning,
            metrics: this.metrics,
            config: {
                fundAddress: this.config.fundAddress,
                checkInterval: this.config.checkIntervalMinutes,
                rebalanceThreshold: this.config.rebalanceThresholdBPS / 100,
                feeCollectionMin: this.config.feeCollectionMinWETH,
                maxGasPrice: this.config.maxGasPriceGwei
            }
        };
    }
}

// Usage example
async function runProductionAgent() {
    const agent = new ProductionAgent({
        fundAddress: process.env.FUND_ADDRESS,
        privateKey: process.env.PRIVATE_KEY,
        rpcUrl: process.env.RPC_URL,
        rebalanceThresholdBPS: 150,      // 1.5%
        feeCollectionMinWETH: 0.01,      // 0.01 WETH minimum
        maxGasPriceGwei: 30,             // Max 30 gwei
        checkIntervalMinutes: 30,        // Check every 30 minutes
        strategy: {
            shouldExecute: async () => {
                // Execute strategy once per day
                const lastExecution = localStorage.getItem('lastStrategyExecution');
                const dayMs = 24 * 60 * 60 * 1000;
                return !lastExecution || (Date.now() - parseInt(lastExecution)) > dayMs;
            },
            calculateWeights: async (marketData) => {
                // Simple momentum strategy
                const totalMomentum = marketData.reduce((sum, token) => 
                    sum + Math.max(0, token.momentum), 0
                );
                
                return marketData.map(token => {
                    const positiveMomentum = Math.max(0, token.momentum);
                    return Math.floor((positiveMomentum / totalMomentum) * 10000) || 1;
                });
            }
        }
    });
    
    await agent.start();
    
    // Status monitoring
    setInterval(() => {
        console.log('Agent Status:', agent.getStatus());
    }, 5 * 60 * 1000); // Every 5 minutes
}

// Emergency stop function
function createEmergencyStop() {
    const fs = require('fs');
    fs.writeFileSync('./emergency_stop', Date.now().toString());
    console.log('Emergency stop file created');
}

// Run the agent
if (require.main === module) {
    runProductionAgent().catch(console.error);
}
```

## Frontend Integration Example

```javascript
// react-fund-manager.jsx - Complete React application
import React, { useState, useEffect, useCallback } from 'react';
import { ethers } from 'ethers';

const FundManager = () => {
    const [wallet, setWallet] = useState(null);
    const [funds, setFunds] = useState([]);
    const [selectedFund, setSelectedFund] = useState(null);
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState('');
    
    // Connect wallet
    const connectWallet = async () => {
        try {
            if (!window.ethereum) {
                throw new Error('MetaMask not installed');
            }
            
            const provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send('eth_requestAccounts', []);
            const signer = await provider.getSigner();
            
            setWallet(signer);
            setError('');
        } catch (err) {
            setError(err.message);
        }
    };
    
    // Load funds
    const loadFunds = useCallback(async () => {
        if (!wallet) return;
        
        setLoading(true);
        try {
            const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, wallet);
            const fundCount = await registry.getDeployedFundsCount();
            
            const fundPromises = [];
            for (let i = 0; i < fundCount; i++) {
                fundPromises.push(loadFundInfo(registry, i));
            }
            
            const fundInfos = await Promise.all(fundPromises);
            setFunds(fundInfos);
        } catch (err) {
            setError(err.message);
        }
        setLoading(false);
    }, [wallet]);
    
    const loadFundInfo = async (registry, index) => {
        const address = await registry.getFundAddressByIndex(index);
        const fund = new ethers.Contract(address, FUND_ABI, wallet);
        
        const [name, symbol, nav, totalShares, agent, feeRate] = await Promise.all([
            fund.name(),
            fund.symbol(),
            fund.totalNAVInAccountingAsset(),
            fund.totalSupply(),
            fund.agent(),
            fund.agentAumFeeBps()
        ]);
        
        return {
            address,
            name,
            symbol,
            nav: ethers.formatEther(nav),
            totalShares: ethers.formatEther(totalShares),
            sharePrice: totalShares > 0 ? ethers.formatEther(nav * BigInt(1e18) / totalShares) : '0',
            agent,
            feeRate: Number(feeRate) / 100,
            isMyFund: agent.toLowerCase() === wallet.address.toLowerCase()
        };
    };
    
    // Investment component
    const InvestmentPanel = ({ fund }) => {
        const [amount, setAmount] = useState('');
        const [investing, setInvesting] = useState(false);
        
        const handleInvest = async () => {
            if (!amount || !fund) return;
            
            setInvesting(true);
            try {
                const wethAmount = ethers.parseEther(amount);
                
                // Prepare WETH
                const weth = new ethers.Contract(WETH_ADDRESS, [
                    'function balanceOf(address) view returns (uint256)',
                    'function deposit() payable',
                    'function approve(address,uint256) external'
                ], wallet);
                
                const balance = await weth.balanceOf(wallet.address);
                if (balance < wethAmount) {
                    const needed = wethAmount - balance;
                    await weth.deposit({ value: needed });
                }
                
                // Approve and invest
                await weth.approve(fund.address, wethAmount);
                
                const fundContract = new ethers.Contract(fund.address, FUND_ABI, wallet);
                const tx = await fundContract.deposit(wethAmount, wallet.address);
                await tx.wait();
                
                setAmount('');
                await loadFunds(); // Refresh data
                
            } catch (err) {
                setError(err.message);
            }
            setInvesting(false);
        };
        
        return (
            <div className="investment-panel">
                <h3>Invest in {fund.name}</h3>
                <div className="fund-stats">
                    <p>NAV: {fund.nav} WETH</p>
                    <p>Share Price: {fund.sharePrice} WETH</p>
                    <p>Fee Rate: {fund.feeRate}%</p>
                </div>
                <div className="investment-form">
                    <input
                        type="number"
                        value={amount}
                        onChange={(e) => setAmount(e.target.value)}
                        placeholder="WETH amount"
                        step="0.01"
                        min="0.01"
                    />
                    <button 
                        onClick={handleInvest}
                        disabled={investing || !amount}
                    >
                        {investing ? 'Investing...' : 'Invest'}
                    </button>
                </div>
            </div>
        );
    };
    
    // Agent management component
    const AgentPanel = ({ fund }) => {
        const [newWeights, setNewWeights] = useState('');
        const [managing, setManaging] = useState(false);
        
        const handleSetWeights = async () => {
            if (!newWeights) return;
            
            setManaging(true);
            try {
                const weights = newWeights.split(',').map(w => parseInt(w.trim()));
                const total = weights.reduce((a, b) => a + b, 0);
                
                if (total !== 10000) {
                    throw new Error('Weights must sum to 10000');
                }
                
                const fundContract = new ethers.Contract(fund.address, FUND_ABI, wallet);
                const tx = await fundContract.setTargetWeightsAndRebalanceIfNeeded(weights);
                await tx.wait();
                
                setNewWeights('');
                
            } catch (err) {
                setError(err.message);
            }
            setManaging(false);
        };
        
        const handleCollectFees = async () => {
            setManaging(true);
            try {
                const fundContract = new ethers.Contract(fund.address, FUND_ABI, wallet);
                const tx = await fundContract.collectAgentManagementFee();
                await tx.wait();
                
            } catch (err) {
                setError(err.message);
            }
            setManaging(false);
        };
        
        return (
            <div className="agent-panel">
                <h3>Manage {fund.name}</h3>
                <div className="management-actions">
                    <div>
                        <input
                            type="text"
                            value={newWeights}
                            onChange={(e) => setNewWeights(e.target.value)}
                            placeholder="e.g., 4000,3000,3000"
                        />
                        <button 
                            onClick={handleSetWeights}
                            disabled={managing}
                        >
                            Set Weights & Rebalance
                        </button>
                    </div>
                    <button 
                        onClick={handleCollectFees}
                        disabled={managing}
                    >
                        Collect Fees
                    </button>
                </div>
            </div>
        );
    };
    
    useEffect(() => {
        if (wallet) {
            loadFunds();
        }
    }, [wallet, loadFunds]);
    
    return (
        <div className="fund-manager">
            <header>
                <h1>WHACKROCK Fund Manager</h1>
                {!wallet ? (
                    <button onClick={connectWallet}>Connect Wallet</button>
                ) : (
                    <span>Connected: {wallet.address?.slice(0, 6)}...</span>
                )}
            </header>
            
            {error && (
                <div className="error-message">
                    Error: {error}
                </div>
            )}
            
            {loading ? (
                <div>Loading funds...</div>
            ) : (
                <div className="funds-grid">
                    {funds.map(fund => (
                        <div key={fund.address} className="fund-card">
                            <h3>{fund.name} ({fund.symbol})</h3>
                            <p>NAV: {fund.nav} WETH</p>
                            <p>Share Price: {fund.sharePrice} WETH</p>
                            <p>Fee: {fund.feeRate}%</p>
                            
                            <button 
                                onClick={() => setSelectedFund(fund)}
                                className="select-button"
                            >
                                {fund.isMyFund ? 'Manage' : 'Invest'}
                            </button>
                        </div>
                    ))}
                </div>
            )}
            
            {selectedFund && (
                <div className="modal-overlay" onClick={() => setSelectedFund(null)}>
                    <div className="modal-content" onClick={e => e.stopPropagation()}>
                        <button 
                            className="close-button"
                            onClick={() => setSelectedFund(null)}
                        >
                            ×
                        </button>
                        
                        {selectedFund.isMyFund ? (
                            <AgentPanel fund={selectedFund} />
                        ) : (
                            <InvestmentPanel fund={selectedFund} />
                        )}
                    </div>
                </div>
            )}
        </div>
    );
};

export default FundManager;
```

## Related Documentation

- [Quick Start](quick-start.md) - Simple integration examples
- [Creating Funds](creating-funds.md) - Fund creation details
- [Investing](investing.md) - Investment strategies
- [Agent Operations](agent-operations.md) - Agent management