# AI Agent Integration

## Overview

WHACKROCK enables AI agents to manage decentralized investment funds through the GAME SDK. The WRTreasury plugin provides the interface between AI agents and fund contracts.

## GAME Framework Integration

GAME (Generalized Autonomous Machine Economy) by Virtuals Protocol enables AI agents to interact with blockchain protocols autonomously.

### How It Works

1. AI agents use the WRTreasury GAME SDK plugin
2. Agents can monitor portfolio weights and NAV
3. Agents set target weights and trigger rebalancing
4. All operations are secured by agent private keys

## WRTreasury Plugin

The WRTreasury plugin provides functions for AI agents to manage WhackRock funds.

### Available Functions

- `get_current_weights`: Get current portfolio composition in basis points
- `get_target_weights`: Get target portfolio weights
- `check_rebalance_needed`: Check if deviation exceeds 1% threshold
- `trigger_rebalance`: Execute portfolio rebalancing
- `set_target_weights`: Update target weights (optionally with rebalancing)
- `get_fund_info`: Get fund NAV and metadata

### Installation

```bash
pip install game-sdk
```

Then use the plugin from the repository:
```python
from whackrock_plugin_gamesdk.whackrock_fund_manager_gamesdk import WhackRockFundManagerSDK
```

## Agent Operations

### Portfolio Monitoring
```python
# Get current and target weights
current_weights = fund_sdk.get_current_weights()
target_weights = fund_sdk.get_target_weights()

# Check if rebalancing is needed (1% threshold)
result = fund_sdk.compare_weights()
```

### Weight Management
```python
# Set target weights in basis points (sum must equal 10000)
result = fund_sdk.set_target_weights(
    weights_bps=[5000, 3000, 2000],  # 50%, 30%, 20%
    rebalance_if_needed=True
)
```

### Fund Information
```python
# Get fund NAV and metadata
fund_info = fund_sdk.get_fund_info()
```

## Agent Authorization

### Permission Model

- **Fund Owner**: Sets the authorized agent address
- **Agent**: Can set target weights and trigger rebalancing
- **Anyone**: Can deposit WETH, view fund information
- **Share Holders**: Can withdraw proportional basket

### Security

- Only the designated agent can modify portfolio weights
- Agent operations require valid private key signatures
- Fund owner retains the ability to change agents

## Example Implementation

The WRTreasury plugin includes a signal-based strategy example that:

1. Fetches YouTube transcripts for market analysis
2. Uses GAME SDK's LLM to analyze sentiment
3. Converts analysis into portfolio weights
4. Executes rebalancing through the fund contract

```python
def get_target_weights_bps() -> list[int]:
    """Get target weights in basis points for GAME framework integration."""
    weights = asyncio.run(derive_weights())
    return [int(w * 10000) for w in weights]

def get_signal_description() -> str:
    """Get description of the current signal for reporting."""
    weights = asyncio.run(derive_weights())
    return f"Signal: VIRTUAL={weights[0]:.2%}, cbBTC={weights[1]:.2%}, USDC={weights[2]:.2%}"
```

## Getting Started

### Prerequisites

1. **Create Your Fund**: Use the [fund creation guide](fund-creation.md) to deploy your fund via the WHACKROCK frontend
2. **Get Fund Address**: Copy your fund contract address from the fund manager dashboard
3. **Set Up Agent Wallet**: Create a dedicated wallet for your AI agent operations
4. **Configure Agent**: Set your agent address in the fund management interface

### Plugin Setup

1. **Install the Plugin**: Clone the [WRTreasury Plugin Repository](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)
2. **Review Examples**: Study the `basic` and `benfan` examples in the plugin directory
3. **Environment Setup**: Configure your environment variables:
   ```bash
   export GAME_API_KEY=your_game_api_key
   export FUND_CONTRACT_ADDRESS=your_fund_address
   export AGENT_PRIVATE_KEY=your_agent_private_key
   export WEB3_PROVIDER=https://mainnet.base.org
   ```
4. **Test Integration**: Run the basic example to verify connectivity

### Integration with Frontend

**Fund Management Dashboard:**
- Use the fund manager dashboard to monitor your agent's performance
- View real-time portfolio weights and NAV
- Track rebalancing activity and performance metrics
- Update agent address if needed

**Monitoring Tools:**
- **Portfolio Tab**: See current vs target weights
- **Recent Activity**: Monitor agent transactions
- **Performance Charts**: Track fund performance over time
- **External Links**: Use DeBank and BaseScan for detailed analysis