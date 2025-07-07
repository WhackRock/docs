# Creating a Fund

This guide walks you through creating an AI-managed investment fund using the WHACKROCK frontend.

## Overview

Creating a fund on WHACKROCK involves:
1. Connecting your wallet to Base network
2. Configuring fund details and strategy
3. Setting up your AI agent integration
4. Selecting portfolio tokens and weights
5. Paying creation fee and deploying

## Step-by-Step Guide

### Step 1: Connect Your Wallet

1. Visit the WHACKROCK dApp
2. Click "Connect Wallet" in the top-right navigation
3. Choose your wallet provider (MetaMask, Rainbow, etc.) via RainbowKit
4. Ensure you're connected to **Base network**
5. Confirm the connection

### Step 2: Navigate to Fund Creation

1. In the navigation menu, hover over "Fund Manager"
2. Click "Create Fund" from the dropdown
3. You'll see the comprehensive fund creation form

### Step 3: Configure Fund Details

**Fund Information:**
- **Fund Name**: Choose a descriptive name for your fund
- **Symbol**: Create a ticker symbol (e.g., "AIFUND", "BTCAI")
- **Fund URI**: Link to your fund's information page or social media
- **Description**: Explain your AI agent's investment strategy

**Example:**
```
Fund Name: AI Momentum Strategy
Symbol: AIMOMENTUM
Fund URI: https://twitter.com/my_ai_agent
Description: AI agent that analyzes market momentum and social sentiment to optimize portfolio allocations across DeFi blue chips.
```

### Step 4: Agent Configuration

**Agent Setup:**
- **Initial Agent Address**: Enter your AI agent's wallet address
  - Leave blank to set yourself as the initial agent
  - You can change this later
- **AUM Fee Wallet**: Address that receives management fees
  - Leave blank to use your connected wallet
- **Total AUM Fee (%)**: Annual management fee percentage
  - Maximum: 10%
  - Standard range: 1-5%

**Important Notes:**
- The agent address will have permission to rebalance the portfolio
- Only this agent can set target weights and trigger rebalancing
- You (as fund owner) can change the agent anytime

### Step 5: AI Agent Integration

**GAME SDK Plugin Section:**
The form includes a dedicated "AI Agent Integration" section with information about the WhackRock GAME plugin:

1. **Review the Plugin Card**: The form displays a comprehensive card explaining the GAME plugin capabilities
2. **Access Documentation**: Click "View Plugin Documentation →" to open the [GAME SDK Plugin Repository](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)
3. **Plugin Features**: Your AI agent will use this plugin to:
   - Monitor portfolio weights and NAV
   - Set target allocations in basis points
   - Execute automated rebalancing
   - Collect management fees

**Blue Information Box:**
The portfolio composition section also includes a blue information box with an embedded link to the GAME plugin documentation, helping you understand how to integrate your AI agent after fund creation.

### Step 6: Portfolio Composition

**Token Selection:**
1. Initially, no tokens are selected. Click "Add / Edit Tokens" to open the token selection modal
2. Browse the approved token list (Base network tokens only)
3. Select your desired tokens by checking the boxes
4. Click "Save Selection" to add them to your portfolio
5. Recommended: Select 2-6 tokens for proper diversification

**Weight Allocation:**
Once tokens are selected, you'll see weight input fields:

1. Enter target weight percentages for each token
2. The system tracks your total in real-time
3. Weights must sum to exactly 100% (shown in green when correct)
4. Example allocation:
   - WETH: 40%
   - USDC: 30% 
   - VIRTUAL: 20%
   - cbBTC: 10%

**Weight Guidelines:**
- Total must equal exactly 100% to proceed
- No minimum per token, but diversification recommended
- Consider liquidity when setting weights
- AI agents can modify these weights later

### Step 7: Protocol Fee Payment

**Creation Fee:**
1. The system displays the creation fee in USDC
2. You'll need to approve USDC spending first
3. Click "Approve USDC" and confirm the transaction
4. Wait for approval confirmation

### Step 8: Deploy Your Fund

**Final Deployment:**
1. After USDC approval, the "Create Fund" button becomes active
2. Review all your settings one final time
3. Click "Create Fund" to deploy your fund contract
4. Confirm the transaction in your wallet (this deploys the fund)
5. Wait for confirmation on Base network
6. Your new fund will appear in the fund manager dashboard

## After Fund Creation

### Smart Contract Information

Your newly created fund is deployed using the WhackRock smart contract template:
- **Source Code**: [WhackRock Treasury Template Repository](https://github.com/WhackRock/whackrock-treasury-template)
- **Fund Contract**: Based on `src/funds/WhackRockFundV6_UniSwap_TWAP.sol`
- **Network**: Deployed on Base blockchain

### Immediate Next Steps

1. **Set Up Your AI Agent**: 
   - Install the [GAME SDK Plugin](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)
   - Configure environment variables with your fund address
   - Test agent connectivity

2. **Fund Your Strategy**:
   - Make the first deposit to initialize the fund
   - Minimum deposit: 0.01 WETH
   - This triggers initial rebalancing to target weights

3. **Monitor Performance**:
   - Track your fund's NAV and performance
   - Monitor agent activity and rebalancing
   - Collect management fees as they accrue

### Agent Integration Example

```python
# Example agent setup
from whackrock_plugin_gamesdk.whackrock_fund_manager_gamesdk import WhackRockFundManagerSDK

# Initialize SDK with your fund
fund_sdk = WhackRockFundManagerSDK(
    web3_provider="https://mainnet.base.org",
    fund_contract_address="YOUR_FUND_ADDRESS",
    private_key="YOUR_AGENT_PRIVATE_KEY",
    account_address="YOUR_AGENT_ADDRESS"
)

# Check current portfolio
weights = fund_sdk.get_current_weights()
print(f"Current weights: {weights}")
```

## Best Practices

### Fund Setup
- **Clear Strategy**: Articulate your AI agent's investment approach
- **Reasonable Fees**: Keep management fees competitive (1-5%)
- **Diversification**: Don't over-concentrate in single assets
- **Testing**: Test your agent on testnet first

### Agent Management
- **Regular Monitoring**: Check agent performance regularly
- **Risk Management**: Implement stop-losses and position limits
- **Transparency**: Keep investors informed about strategy changes
- **Documentation**: Maintain clear records of agent decisions

### Legal Considerations
- **Disclaimer**: Include appropriate risk disclaimers
- **Compliance**: Ensure compliance with local regulations
- **Terms**: Consider creating terms of service for investors

## Troubleshooting

### Common Issues

**"Transaction Failed"**
- Check you have enough USDC for creation fee
- Ensure weights sum to exactly 100%
- Verify all required fields are filled

**"Token Not Found"**
- Only approved tokens can be selected
- Make sure you're on Base network
- Try refreshing the token list

**"Agent Address Invalid"**
- Ensure agent address is a valid Ethereum address
- Agent address cannot be the zero address
- You can leave blank to set yourself initially

### Getting Help

- Check the [Technical Documentation](../smart-contracts/fund/overview.md)
- Review [AI Integration Guide](ai-integration.md)
- Join the community Discord for support

## Next Steps

After creating your fund:
1. [Set up AI agent integration](ai-integration.md)
2. [Understand fee collection](../smart-contracts/fund/fee-collection.md)
3. [Learn about portfolio management](../smart-contracts/fund/portfolio-mgmt.md)