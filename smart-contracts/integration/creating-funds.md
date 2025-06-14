# Creating Funds

## Overview

This guide covers the complete process of creating WhackRock funds, from parameter selection to post-deployment setup. Creating a fund is permissionless but requires careful parameter selection.

## Prerequisites

- USDC for creation fee (if set by protocol)
- Understanding of target tokens
- Strategy for weight allocation
- Agent address (can be yourself or an AI agent)

## Fund Creation Process

### 1. Parameter Planning

Before calling the contract, plan your fund parameters:

```javascript
const fundParameters = {
    // Management
    initialAgent: "0x...",           // Who manages the fund
    agentAumFeeWallet: "0x...",      // Where agent fees go
    agentSetTotalAumFeeBps: 200,     // 2% annual fee
    
    // Portfolio
    fundAllowedTokens: ["0x...", "0x..."],  // Token addresses
    initialTargetWeights: [6000, 4000],     // 60%, 40%
    
    // Metadata
    vaultName: "AI Growth Fund",
    vaultSymbol: "AIGF",
    vaultURI: "ipfs://...",          // Optional metadata
    description: "AI-managed growth strategy"
};
```

### 2. Validate Parameters

```javascript
async function validateFundParameters(params) {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, provider);
    
    // Check token allowlist
    const allowedTokens = await registry.getRegistryAllowedTokens();
    for (const token of params.fundAllowedTokens) {
        if (!allowedTokens.includes(token)) {
            throw new Error(`Token ${token} not in allowlist`);
        }
    }
    
    // Validate weights
    const totalWeight = params.initialTargetWeights.reduce((a, b) => a + b, 0);
    if (totalWeight !== 10000) {
        throw new Error(`Weights must sum to 10000, got ${totalWeight}`);
    }
    
    // Check fee limits
    const maxFee = await registry.totalAumFeeBpsForFunds();
    if (params.agentSetTotalAumFeeBps > maxFee) {
        throw new Error(`Fee ${params.agentSetTotalAumFeeBps} exceeds maximum ${maxFee}`);
    }
    
    // Check symbol uniqueness
    const isSymbolTaken = await registry.isSymbolTaken(params.vaultSymbol);
    if (isSymbolTaken) {
        throw new Error(`Symbol ${params.vaultSymbol} already taken`);
    }
    
    console.log('✅ All parameters valid');
}
```

### 3. Handle Creation Fee

```javascript
async function handleCreationFee(wallet) {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, wallet);
    const feeAmount = await registry.protocolFundCreationFeeUsdcAmount();
    
    if (feeAmount > 0) {
        const usdcAddress = await registry.USDC_TOKEN();
        const usdc = new ethers.Contract(usdcAddress, [
            "function balanceOf(address) view returns (uint256)",
            "function approve(address,uint256) external",
            "function allowance(address,address) view returns (uint256)"
        ], wallet);
        
        // Check balance
        const balance = await usdc.balanceOf(wallet.address);
        if (balance < feeAmount) {
            throw new Error(`Insufficient USDC. Need ${feeAmount}, have ${balance}`);
        }
        
        // Check/set approval
        const allowance = await usdc.allowance(wallet.address, REGISTRY_ADDRESS);
        if (allowance < feeAmount) {
            console.log(`Approving ${feeAmount} USDC...`);
            const approveTx = await usdc.approve(REGISTRY_ADDRESS, feeAmount);
            await approveTx.wait();
        }
        
        console.log(`✅ USDC fee (${feeAmount}) ready`);
    } else {
        console.log('✅ No creation fee required');
    }
}
```

### 4. Create the Fund

```javascript
async function createFund(params, wallet) {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, wallet);
    
    console.log('Creating fund...');
    const tx = await registry.createWhackRockFund(
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
    console.log(`✅ Fund created in block ${receipt.blockNumber}`);
    
    // Extract fund address from event
    const fundCreatedEvent = receipt.logs.find(log => {
        try {
            const parsed = registry.interface.parseLog(log);
            return parsed.name === 'WhackRockFundCreated';
        } catch {
            return false;
        }
    });
    
    if (!fundCreatedEvent) {
        throw new Error('Fund creation event not found');
    }
    
    const parsedEvent = registry.interface.parseLog(fundCreatedEvent);
    const fundAddress = parsedEvent.args.fundAddress;
    
    console.log(`📍 Fund deployed at: ${fundAddress}`);
    return {
        address: fundAddress,
        fundId: parsedEvent.args.fundId,
        transaction: tx.hash
    };
}
```

## Complete Creation Flow

```javascript
async function completeFundCreation(params, wallet) {
    try {
        console.log('🚀 Starting fund creation process...\n');
        
        // Step 1: Validate parameters
        console.log('1. Validating parameters...');
        await validateFundParameters(params);
        
        // Step 2: Handle creation fee
        console.log('\n2. Preparing creation fee...');
        await handleCreationFee(wallet);
        
        // Step 3: Create fund
        console.log('\n3. Creating fund...');
        const result = await createFund(params, wallet);
        
        // Step 4: Verify deployment
        console.log('\n4. Verifying deployment...');
        await verifyFundDeployment(result.address);
        
        console.log('\n🎉 Fund creation complete!');
        return result;
        
    } catch (error) {
        console.error('❌ Fund creation failed:', error.message);
        throw error;
    }
}
```

## Advanced Parameters

### Token Selection Strategy

```javascript
// Example: DeFi Blue Chip Fund
const defiTokens = {
    fundAllowedTokens: [
        "0x...", // WBTC
        "0x...", // ETH
        "0x...", // USDC
        "0x..."  // LINK
    ],
    initialTargetWeights: [2500, 2500, 2500, 2500], // Equal weight
    description: "Diversified DeFi blue chip portfolio"
};

// Example: Growth Fund
const growthTokens = {
    fundAllowedTokens: [
        "0x...", // Small cap token 1
        "0x...", // Small cap token 2
        "0x...", // USDC (stability)
    ],
    initialTargetWeights: [4000, 4000, 2000], // 80% growth, 20% stable
    description: "High growth potential portfolio"
};
```

### Fee Strategy

```javascript
// Conservative fee (attracts more investors)
const conservativeFees = {
    agentSetTotalAumFeeBps: 50,  // 0.5% annual
    description: "Low fee passive strategy"
};

// Active management fee
const activeFees = {
    agentSetTotalAumFeeBps: 200, // 2% annual
    description: "Active AI management"
};

// Performance-oriented (note: no performance fees in current version)
const performanceFees = {
    agentSetTotalAumFeeBps: 100, // 1% base fee
    description: "Base fee with planned performance component"
};
```

### Metadata Best Practices

```javascript
// IPFS metadata structure
const metadata = {
    name: "AI Growth Fund",
    description: "Detailed fund description...",
    image: "ipfs://QmImage...",
    external_url: "https://yoursite.com/fund/123",
    attributes: [
        { trait_type: "Strategy", value: "Growth" },
        { trait_type: "AI Agent", value: "GPT-4" },
        { trait_type: "Risk Level", value: "Medium" }
    ],
    fund_details: {
        inception_date: new Date().toISOString(),
        strategy_description: "...",
        risk_metrics: {},
        benchmark: "..."
    }
};

// Upload to IPFS and use hash as vaultURI
const vaultURI = `ipfs://${ipfsHash}`;
```

## Post-Creation Setup

### 1. Verify Fund Deployment

```javascript
async function verifyFundDeployment(fundAddress) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    // Basic checks
    const name = await fund.name();
    const symbol = await fund.symbol();
    const agent = await fund.agent();
    const nav = await fund.totalNAVInAccountingAsset();
    
    console.log(`Name: ${name}`);
    console.log(`Symbol: ${symbol}`);
    console.log(`Agent: ${agent}`);
    console.log(`Initial NAV: ${ethers.formatEther(nav)} WETH`);
    
    // Verify token configuration
    const tokenCount = await fund.allowedTokens.length;
    console.log(`Configured tokens: ${tokenCount}`);
    
    for (let i = 0; i < tokenCount; i++) {
        const token = await fund.allowedTokens(i);
        const weight = await fund.targetWeights(token);
        console.log(`  ${token}: ${weight / 100}%`);
    }
}
```

### 2. Initial Agent Setup

```javascript
async function setupAgent(fundAddress, agentWallet) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, agentWallet);
    
    // Agent can immediately start managing
    console.log('Agent is ready to:');
    console.log('- Adjust target weights');
    console.log('- Trigger rebalancing');
    console.log('- Collect management fees');
    
    // Example: Verify agent permissions
    const currentAgent = await fund.agent();
    if (currentAgent.toLowerCase() !== agentWallet.address.toLowerCase()) {
        throw new Error('Agent wallet mismatch');
    }
    
    console.log('✅ Agent setup verified');
}
```

### 3. Marketing Fund

```javascript
async function generateFundInfo(fundAddress) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, provider);
    
    const info = {
        address: fundAddress,
        name: await fund.name(),
        symbol: await fund.symbol(),
        agent: await fund.agent(),
        feeRate: await fund.agentAumFeeBps(),
        totalFunds: await registry.getDeployedFundsCount(),
        launchDate: new Date().toISOString(),
        links: {
            contract: `https://basescan.org/address/${fundAddress}`,
            metadata: await fund.baseURI()
        }
    };
    
    console.log('📊 Fund Information:');
    console.log(JSON.stringify(info, null, 2));
    
    return info;
}
```

## Gas Optimization

### Batch Parameter Validation

```javascript
// Validate parameters off-chain before transaction
async function batchValidation(params) {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, provider);
    
    // Use multicall or batch requests
    const [allowedTokens, maxFee, isSymbolTaken] = await Promise.all([
        registry.getRegistryAllowedTokens(),
        registry.totalAumFeeBpsForFunds(),
        registry.isSymbolTaken(params.vaultSymbol)
    ]);
    
    // Validate all at once
    return validateAllParameters(params, { allowedTokens, maxFee, isSymbolTaken });
}
```

### Estimate Gas Costs

```javascript
async function estimateCreationCost(params, wallet) {
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, wallet);
    
    try {
        const gasEstimate = await registry.createWhackRockFund.estimateGas(
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
        
        const gasPrice = await provider.getFeeData();
        const estimatedCost = gasEstimate * gasPrice.gasPrice;
        
        console.log(`Estimated gas: ${gasEstimate.toString()}`);
        console.log(`Estimated cost: ${ethers.formatEther(estimatedCost)} ETH`);
        
        return estimatedCost;
    } catch (error) {
        console.error('Gas estimation failed:', error.message);
        return null;
    }
}
```

## Troubleshooting

### Common Errors

```javascript
function handleCreationError(error) {
    const message = error.message.toLowerCase();
    
    if (message.includes('token not allowed')) {
        return 'One or more tokens are not in the registry allowlist';
    }
    
    if (message.includes('symbol already taken')) {
        return 'Fund symbol must be unique. Try a different symbol.';
    }
    
    if (message.includes('insufficient usdc')) {
        return 'Not enough USDC for creation fee. Check balance and approval.';
    }
    
    if (message.includes('exceeds max fund tokens')) {
        return 'Too many tokens. Check registry max token limit.';
    }
    
    if (message.includes('invalid weights')) {
        return 'Target weights must sum to exactly 10000 (100%)';
    }
    
    return `Unknown error: ${error.message}`;
}
```

### Recovery Strategies

```javascript
// If creation fails, help user recover
async function recoverFromFailure(params, error) {
    console.log('Creation failed, checking recovery options...');
    
    // Check if USDC was spent
    const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, provider);
    const feeAmount = await registry.protocolFundCreationFeeUsdcAmount();
    
    if (feeAmount > 0) {
        console.log('Note: USDC approval may still be active');
        console.log('You can reuse the approval for next attempt');
    }
    
    // Suggest parameter fixes
    const suggestion = handleCreationError(error);
    console.log('Suggestion:', suggestion);
    
    // Auto-fix common issues
    if (error.message.includes('symbol already taken')) {
        const newSymbol = params.vaultSymbol + Date.now().toString().slice(-4);
        console.log(`Try with symbol: ${newSymbol}`);
        return { ...params, vaultSymbol: newSymbol };
    }
    
    return params;
}
```

## Examples

### AI Agent Fund

```javascript
const aiAgentFund = {
    initialAgent: "0xAIAgentAddress",
    fundAllowedTokens: ["0xWBTC", "0xETH", "0xUSDC"],
    initialTargetWeights: [4000, 4000, 2000],
    vaultName: "AI Momentum Fund",
    vaultSymbol: "AIMOM",
    vaultURI: "ipfs://QmAIMetadata",
    description: "AI-driven momentum strategy",
    agentAumFeeWallet: "0xAIAgentFeeWallet",
    agentSetTotalAumFeeBps: 150
};
```

### Conservative Index

```javascript
const indexFund = {
    initialAgent: wallet.address,
    fundAllowedTokens: ["0xWBTC", "0xETH", "0xUSDC", "0xLINK"],
    initialTargetWeights: [2500, 2500, 2500, 2500],
    vaultName: "Crypto Index Fund",
    vaultSymbol: "CIDX",
    vaultURI: "",
    description: "Equal-weighted crypto index",
    agentAumFeeWallet: wallet.address,
    agentSetTotalAumFeeBps: 75
};
```

## Related Documentation

- [Agent Operations](agent-operations.md) - Managing funds after creation
- [Investing](investing.md) - How users invest in your fund
- [Portfolio Management](../fund/portfolio-mgmt.md) - Managing fund portfolios