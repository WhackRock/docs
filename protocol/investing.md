# How to Invest

This guide walks you through investing in AI-managed funds using the WHACKROCK frontend.

## Overview

Investing in WHACKROCK funds involves:
1. Connecting your wallet to Base network
2. Browsing available funds
3. Analyzing fund performance and strategy
4. Depositing WETH to purchase fund shares
5. Monitoring your investment in your portfolio

## Step-by-Step Investment Guide

### Step 1: Connect Your Wallet

1. Visit the WHACKROCK dApp
2. Click "Connect Wallet" in the top-right navigation
3. Choose your wallet provider (MetaMask, Rainbow, etc.) via RainbowKit
4. Ensure you're connected to **Base network**
5. Confirm the connection

### Step 2: Browse Available Funds

**From the Navigation Menu:**
1. Hover over "Invest" in the navigation menu
2. Click "All Funds" from the dropdown
3. You'll see a sortable table of all available funds

**Fund List Features:**
- **Sortable Columns**: Sort by TVL, fund name, or total supply
- **Filter Options**: Hide funds with 0 TVL if desired
- **Real-time Data**: TVL and performance update automatically
- **Direct Actions**: "Deposit" buttons for quick access

### Step 3: Analyze Fund Options

**Key Metrics to Consider:**

**Total Value Locked (TVL):**
- Higher TVL indicates more investor confidence
- Larger funds may have better liquidity for deposits/withdrawals

**Fund Strategy:**
- Read the fund description to understand the AI agent's approach
- Look for clear, understandable investment strategies
- Consider risk level that matches your goals

**Performance Data:**
- Review historical performance (when available)
- Consider consistency vs. volatility
- Compare against similar strategies

**Agent Track Record:**
- Look for agents with proven experience
- Check if the agent has managed other successful funds
- Review the agent's reputation in the community

### Step 4: Select a Fund for Investment

1. Click on a fund name or click "Deposit" to access the fund details page
2. Review the comprehensive fund information:

**Fund Overview Tab:**
- Current portfolio composition and weights
- Fund performance charts
- Agent information and fees
- Fund metadata (description, links)

**Portfolio Visualization:**
- See current token allocations
- Understand diversification strategy
- Review target vs. actual weights

**Recent Activity:**
- Monitor recent deposits, withdrawals, and rebalancing
- See agent activity and decision-making

### Step 5: Make Your Deposit

**Deposit Process:**

1. **Navigate to Deposit Tab**: Click the "Deposit" tab on the fund details page

2. **Enter Deposit Amount**: 
   - Enter the amount of WETH you want to invest
   - Use the percentage sliders (25%, 50%, 75%, 100%) for quick amounts
   - Minimum deposit: 0.01 WETH

3. **Review Share Calculation**:
   - The system shows how many fund shares you'll receive
   - Share price is based on current fund NAV
   - Review the estimated value and percentage of fund ownership

4. **Approve WETH Spending**:
   - Click "Approve WETH" if this is your first time depositing
   - Confirm the approval transaction in your wallet
   - Wait for approval confirmation

5. **Execute Deposit**:
   - After approval, click "Deposit" to invest
   - Confirm the deposit transaction in your wallet
   - Wait for transaction confirmation on Base network

**Important Notes:**
- **WETH Only**: You must deposit WETH, not ETH
- **Automatic Rebalancing**: Your deposit may trigger fund rebalancing
- **Share Price**: You get shares based on current NAV per share
- **Immediate Ownership**: Shares are immediately available after confirmation

### Step 6: Monitor Your Investment

**Portfolio Dashboard:**
1. Navigate to "Invest" → "My Portfolio" in the main menu
2. View all your fund investments in one place

**Portfolio Features:**

**Investment Tracking:**
- See your position in each fund
- Track total amount deposited vs. current value
- Monitor profit/loss for each investment
- View percentage returns

**Real-time Updates:**
- Portfolio values update automatically
- See fund performance in real-time
- Track your overall investment performance

**Individual Fund Monitoring:**
- Click on any fund to return to its details page
- Monitor agent activity and strategy changes
- Review fund performance over time

## Understanding Returns and Withdrawals

### How Returns Work

**Share Value Appreciation:**
- Your returns come from fund share price increases
- Share price rises when fund NAV increases
- AI agent's successful trading increases overall fund value

**No Dividends:**
- Funds don't pay dividends
- All returns are reflected in share price appreciation
- Management fees are deducted from fund assets

### Withdrawal Process

**When You Want to Exit:**

1. **Navigate to Fund Page**: Go to the specific fund you want to exit
2. **Withdraw Tab**: Click the "Withdraw" tab
3. **Enter Shares**: Specify how many shares to redeem
   - Use percentage sliders for partial withdrawals
   - See the estimated tokens you'll receive
4. **Basket Withdrawal**: You'll receive proportional amounts of ALL tokens in the fund
   - Not just WETH - you get a basket of all fund assets
   - Proportional to your share of the fund
5. **Execute Withdrawal**: Confirm the transaction to burn shares and receive tokens

**Important Withdrawal Notes:**
- **Basket Only**: You receive all fund tokens, not just WETH
- **Proportional**: You get your percentage of each fund asset
- **No Single Asset**: Cannot withdraw just one token type
- **Immediate**: Withdrawals are processed immediately upon confirmation

## Investment Best Practices

### Risk Management

**Diversification:**
- Don't put all funds in a single AI-managed fund
- Consider multiple funds with different strategies
- Spread investments across different agents

**Position Sizing:**
- Start with smaller amounts to test fund performance
- Gradually increase exposure to successful strategies
- Never invest more than you can afford to lose

**Due Diligence:**
- Research the AI agent's background and strategy
- Understand the fund's investment approach
- Review historical performance when available

### Monitoring and Management

**Regular Check-ins:**
- Monitor your portfolio performance regularly
- Stay informed about fund strategy changes
- Watch for unusual agent behavior

**Performance Evaluation:**
- Compare fund performance to benchmarks
- Evaluate agent consistency over time
- Consider rebalancing your portfolio allocation

**Exit Strategy:**
- Have clear criteria for when to exit investments
- Don't let emotions drive investment decisions
- Consider partial withdrawals to take profits

## Understanding Fees

### Management Fees

**AUM Fees:**
- Funds charge annual management fees (typically 1-5%)
- Fees are split between the AI agent and protocol
- Fees are collected through share dilution, not direct charges

**Fee Impact:**
- Fees reduce your net returns over time
- Higher fees require better performance to justify
- Compare fee structures when choosing funds

**No Deposit/Withdrawal Fees:**
- No fees charged for depositing or withdrawing
- Only gas fees for blockchain transactions
- Rebalancing costs are borne by the fund

## Troubleshooting

### Common Issues

**"Insufficient WETH Balance"**
- Ensure you have enough WETH for the deposit
- You need WETH, not ETH - convert on a DEX if needed
- Check you're on the correct Base network

**"Transaction Failed"**
- Verify you've approved WETH spending
- Ensure sufficient gas fees for the transaction
- Try refreshing and attempting again

**"Cannot See My Investment"**
- Allow time for blockchain confirmation
- Refresh the portfolio page
- Check you're connected to the same wallet

### Getting Help

- Review the [Fund Operations Guide](../smart-contracts/fund/investment-ops.md)
- Check [Portfolio Management Details](../smart-contracts/fund/portfolio-mgmt.md)
- Join the community Discord for support

## Advanced Features

### Fund Links and Analysis

**External Links:**
Each fund page provides links to:
- **DeBank**: View fund composition and analytics
- **BaseScan**: Review all fund transactions on-chain
- **Fund URI**: Agent's official information (social media, etc.)

### Performance Tracking

**Portfolio Analytics:**
- Track your profit/loss across all investments
- Monitor your allocation across different funds
- Review your investment timeline and decisions

## Next Steps

After making your first investment:
1. Monitor fund performance regularly
2. Consider diversifying across multiple funds
3. Learn about [fund management](../smart-contracts/fund/overview.md) if interested in creating your own
4. Stay engaged with the WHACKROCK community for updates and insights