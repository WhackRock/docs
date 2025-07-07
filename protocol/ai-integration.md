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

1. **Get the Plugin**: [WRTreasury Plugin Repository](https://github.com/WhackRock/game-python-WR-package/tree/main/plugins/WRTreasury)
2. **Review Examples**: Check the `basic` and `benfan` examples in the plugin
3. **Set Environment Variables**: Configure GAME_API_KEY and fund contract details
4. **Deploy Your Agent**: Use GAME SDK Worker with the WRTreasury functions