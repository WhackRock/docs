# Quick Start Integration Guide

## Overview

This guide gets you interacting with WHACKROCK contracts in under 5 minutes. Perfect for developers who want to test basic functionality before diving deeper.

## Prerequisites

- Node.js and npm/yarn
- Basic knowledge of Web3/Ethers.js
- Access to Base network (testnet or mainnet)
- Some ETH for gas fees

## Setup

### 1. Install Dependencies

```bash
npm install ethers
```

### 2. Basic Contract Setup

```javascript
import { ethers } from 'ethers';

// Contract addresses (update with actual deployments)
const REGISTRY_ADDRESS = '0x...'; // Registry contract
const WETH_ADDRESS = '0x4200000000000000000000000000000000000006'; // Base WETH

// ABIs (simplified - include full ABIs in production)
const REGISTRY_ABI = [
    "function createWhackRockFund(address,address[],uint256[],string,string,string,string,address,uint256) external returns (address)",
    "function getRegistryAllowedTokens() external view returns (address[])",
    "event WhackRockFundCreated(uint256 indexed fundId, address indexed fundAddress, address indexed creator, address initialAgent, string vaultName, string vaultSymbol, string vaultURI, string description, address[] allowedTokens, uint256[] targetWeights, address agentAumFeeWallet, uint256 agentTotalAumFeeBps, uint256 timestamp)"
];

const FUND_ABI = [
    "function deposit(uint256,address) external returns (uint256)",
    "function withdraw(uint256,address,address) external",
    "function totalNAVInAccountingAsset() external view returns (uint256)",
    "function balanceOf(address) external view returns (uint256)",
    "function totalSupply() external view returns (uint256)"
];

// Setup provider and wallet
const provider = new ethers.JsonRpcProvider('https://mainnet.base.org');
const wallet = new ethers.Wallet('YOUR_PRIVATE_KEY', provider);

// Contract instances
const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, wallet);
```

## Quick Actions

### 1. Query Available Tokens

```javascript
async function getAvailableTokens() {
    const tokens = await registry.getRegistryAllowedTokens();
    console.log('Available tokens:', tokens);
    return tokens;
}
```

### 2. Create a Simple Fund

```javascript
async function createSimpleFund() {
    const tokens = await getAvailableTokens();
    
    // Create 50/50 fund with first two tokens
    const fundTokens = tokens.slice(0, 2);
    const weights = [5000, 5000]; // 50% each
    
    const tx = await registry.createWhackRockFund(
        wallet.address,              // agent (yourself)
        fundTokens,                  // allowed tokens
        weights,                     // target weights
        "My Test Fund",              // fund name
        "MTF",                       // fund symbol
        "",                          // URI (empty for now)
        "A test fund",               // description
        wallet.address,              // fee recipient
        100                          // 1% annual fee
    );
    
    const receipt = await tx.wait();
    
    // Extract fund address from event
    const event = receipt.logs.find(log => 
        log.fragment?.name === 'WhackRockFundCreated'
    );
    
    const fundAddress = event.args.fundAddress;
    console.log('Fund created at:', fundAddress);
    return fundAddress;
}
```

### 3. Invest in Fund

```javascript
async function investInFund(fundAddress, amountWETH) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, wallet);
    const weth = new ethers.Contract(WETH_ADDRESS, [
        "function approve(address,uint256) external",
        "function balanceOf(address) external view returns (uint256)"
    ], wallet);
    
    // Check WETH balance
    const balance = await weth.balanceOf(wallet.address);
    console.log('WETH balance:', ethers.formatEther(balance));
    
    // Approve WETH
    await weth.approve(fundAddress, amountWETH);
    console.log('WETH approved');
    
    // Deposit
    const tx = await fund.deposit(amountWETH, wallet.address);
    const receipt = await tx.wait();
    
    console.log('Investment successful!');
    
    // Check shares received
    const shares = await fund.balanceOf(wallet.address);
    console.log('Shares received:', ethers.formatEther(shares));
}
```

### 4. Check Fund Status

```javascript
async function checkFundStatus(fundAddress) {
    const fund = new ethers.Contract(fundAddress, FUND_ABI, provider);
    
    const nav = await fund.totalNAVInAccountingAsset();
    const totalShares = await fund.totalSupply();
    const sharePrice = totalShares > 0 ? nav * BigInt(1e18) / totalShares : 0n;
    
    console.log('Fund NAV:', ethers.formatEther(nav), 'WETH');
    console.log('Total Shares:', ethers.formatEther(totalShares));
    console.log('Share Price:', ethers.formatEther(sharePrice), 'WETH per share');
}
```

## Complete Example

Here's a complete script that ties everything together:

```javascript
// complete-example.js
import { ethers } from 'ethers';

async function main() {
    try {
        console.log('🚀 WHACKROCK Quick Start');
        
        // 1. Get available tokens
        console.log('\n1. Fetching available tokens...');
        const tokens = await getAvailableTokens();
        
        // 2. Create a fund
        console.log('\n2. Creating a test fund...');
        const fundAddress = await createSimpleFund();
        
        // 3. Invest in fund
        console.log('\n3. Investing 0.01 WETH...');
        const investAmount = ethers.parseEther('0.01');
        await investInFund(fundAddress, investAmount);
        
        // 4. Check status
        console.log('\n4. Checking fund status...');
        await checkFundStatus(fundAddress);
        
        console.log('\n✅ Quick start complete!');
        console.log('Your fund address:', fundAddress);
        
    } catch (error) {
        console.error('❌ Error:', error.message);
    }
}

main().catch(console.error);
```

## Common Issues & Solutions

### Issue: "Insufficient WETH balance"

**Solution**: Convert ETH to WETH first

```javascript
// Convert ETH to WETH
const weth = new ethers.Contract(WETH_ADDRESS, [
    "function deposit() external payable"
], wallet);

await weth.deposit({ value: ethers.parseEther('0.1') });
```

### Issue: "Token not allowed"

**Solution**: Use only tokens from the allowlist

```javascript
const allowedTokens = await registry.getRegistryAllowedTokens();
console.log('Use only these tokens:', allowedTokens);
```

### Issue: "Symbol already taken"

**Solution**: Use a unique symbol

```javascript
const timestamp = Date.now();
const uniqueSymbol = `MTF${timestamp}`;
```

## Next Steps

Once you've completed the quick start:

1. **[Creating Funds](creating-funds.md)** - Detailed fund creation guide
2. **[Investing](investing.md)** - Advanced investment patterns
3. **[Agent Operations](agent-operations.md)** - Managing funds as an agent
4. **[Code Examples](code-examples.md)** - More complex integrations

## Production Checklist

Before going to production:

- [ ] Use proper error handling
- [ ] Implement gas estimation
- [ ] Add transaction retry logic
- [ ] Use full contract ABIs
- [ ] Implement proper logging
- [ ] Add user input validation
- [ ] Test on testnet first
- [ ] Monitor contract events

## Useful Resources

- **Base RPC URLs**: https://docs.base.org/network-information
- **WETH Contract**: Always verify the correct address for your network
- **Ethers.js Docs**: https://docs.ethers.io/
- **Base Testnet Faucet**: For getting test ETH

## Support

If you run into issues:

1. Check the troubleshooting section in each detailed guide
2. Verify contract addresses are correct for your network
3. Ensure you have sufficient gas fees
4. Join our Discord for community support